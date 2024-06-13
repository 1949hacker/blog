---
title: 记录一次构建armv8平台vsftpd Docker镜像的过程
categories: [知识库]
comments: true
date: 2023-09-12 20:53:20
keywords: armv8
tags:
    - Docker
    - armv8
    - vsftpd
---

因客户使用飞腾2000平台需要部署vsftpd，但因系统环境不同存在诸多部署问题，且无适配的armv8版本vsftpd容器，特此专门构建vsftpd镜像并记录构建过程

<!-- more -->

Dockerfile代码如下

```Dockerfile
FROM arm64v8/debian

LABEL maintainer="0@hackerbs.com"

RUN rm /etc/apt/sources.list.d/debian.sources -f

RUN echo "deb http://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware\
deb-src http://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware\
deb http://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware\
deb-src http://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware\
deb http://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware\
deb-src http://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware\
deb http://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware\
deb-src http://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware" > /etc/apt/sources.list

RUN apt update -y && \
	apt install -y openssl netcat-openbsd vsftpd --no-install-recommends && \
	apt autoremove -y && apt clean -y && \
	rm -rf /var/lib/apt/lists/*

# VSFTPD configuration
COPY vsftpd.conf /etc/vsftpd.conf

# VSFTPD pre-configurations
COPY docker-entrypoint.sh /var/tmp/

RUN chmod +x /var/tmp/docker-entrypoint.sh

ENTRYPOINT ["/var/tmp/docker-entrypoint.sh"]

EXPOSE 20/tcp 21/tcp
EXPOSE 4559/tcp 4560/tcp 4561/tcp 4562/tcp 4563/tcp 4564/tcp

HEALTHCHECK --interval=5m --timeout=3s \
  CMD nc -z localhost 21 || exit 1

CMD ["vsftpd"]

# docker build -t hackerbs/vsftpd:latest .
```

docker-entrypoint.sh代码如下

```shell
#!/bin/bash
set -x

if [ "$1" = "vsftpd" ]; then
	VSFTPDDIR="/etc"
	PIDDIR="/var/run/vsftpd"
	LOGDIR="/var/log/vsftpd"
	SECURECHROOTDIR="/var/run/vsftpd/empty"
	PRIVATEKEY_FILE="/etc/ssl/private/vsftpd.key"
	CERTIFICATE_FILE="/etc/ssl/certs/vsftpd.crt"
	CSR_FILE="/etc/ssl/certs/vsftpd.csr"

	if [ -z "$FTP_SERVER_NAME" ]; then
		export FTP_SERVER_NAME="Welcome to [hackerbs's vsftpd for armv8] service"
	fi

	if [ -z "$FTP_REPOSITORY" ]; then
		export FTP_REPOSITORY="/srv_volume"
	fi

	if [ -z "$FTP_USER" ]; then
		export FTP_USER="admin"
	fi

	if [ -z "$FTP_PASSWORD" ]; then
		export FTP_PASSWORD="$(cat /dev/urandom | tr -dc A-Z-a-z-0-9 | head -c 18)"
	fi

	if [ -z "$PASV_ADDRESS" ]; then
		export PASV_ADDRESS="$(tail -n 1 /etc/hosts | awk '{print $1}')"
	fi

	if [ -z "$PASV_PROMISCUOUS" ]; then
		export PASV_PROMISCUOUS="false"
	fi

	if [ -z "$PASV_MIN_PORT" ]; then
		export PASV_MIN_PORT="4559"
	fi

	if [ -z "$PASV_MAX_PORT" ]; then
		export PASV_MAX_PORT="4564"
	fi

	if [ -z "$USESSL" ]; then
		export USESSL="false"
	fi

	if [ -z "$FORCESSL" ]; then
		export FORCESSL="false"
	fi

	EXIST=1
	grep -qw ^"$FTP_USER" /etc/passwd || EXIST=0

	if [ "$EXIST" -eq 0 ]; then
		# Neccesary directories creation
		mkdir -p "$LOGDIR" "$PIDDIR" "$SECURECHROOTDIR"

		# VSFTPd log file creation
		touch "${LOGDIR}"/vsftpd.log
		touch "${LOGDIR}"/xferlog.log

		# User creation / configuration
		useradd -c "User for send files using vSFTPD" -d "$FTP_REPOSITORY" -m "$FTP_USER" &> /dev/null && echo "FTP user creation [ OK ]" || exit 2
		chown "$FTP_USER". "$FTP_REPOSITORY" &> /dev/null && echo "FTP user directory configuration [ OK ]" || exit 2
		echo -e "$FTP_PASSWORD\\n$FTP_PASSWORD" | passwd "$FTP_USER" &> /dev/null && echo "FTP user password configuration [ OK ]" || exit 2

		sed -i "s/PASV_ADDRESS_CUSTOM/$PASV_ADDRESS/g" "${VSFTPDDIR}/vsftpd.conf"
		sed -i "s/FTP_SERVER_NAME_CUSTOM/$FTP_SERVER_NAME/g" "${VSFTPDDIR}/vsftpd.conf"

		if [ "$PASV_PROMISCUOUS" == "true" ]; then
			echo "pasv_promiscuous=YES" >> "${VSFTPDDIR}/vsftpd.conf"
		fi

		{
			echo "pasv_min_port=$PASV_MIN_PORT"
			echo "pasv_max_port=$PASV_MAX_PORT"
		} >> "${VSFTPDDIR}/vsftpd.conf"

		if [ "$USESSL" == "true" ]; then
			{
				echo "ssl_enable=YES"
				echo "allow_anon_ssl=NO"
				echo "ssl_tlsv1=NO"
				echo "ssl_sslv2=NO"
				echo "ssl_sslv3=NO"
				echo "rsa_cert_file=$CERTIFICATE_FILE"
				echo "rsa_private_key_file=$PRIVATEKEY_FILE"
			} >> "${VSFTPDDIR}/vsftpd.conf"

			if [ -z "$SSL_CERTIFICATE" ]; then
				openssl genrsa -out "$PRIVATEKEY_FILE" 4096 &> /dev/null && echo "Private key generate [ OK ]" || exit 2
				openssl req -subj "/CN=$HOSTNAME/C=ES/ST=Catalunya/L=Barcelona/O=Arroyof Solutions/OU=Sistemas/emailAddress=enzo@arroyof.com" -sha256 -new -key "$PRIVATEKEY_FILE" -out "$CSR_FILE" &> /dev/null && echo "CSR generate [ OK ]" || exit 2
				openssl x509 -req -days 365 -in "$CSR_FILE" -signkey "$PRIVATEKEY_FILE" -sha256 -out "$CERTIFICATE_FILE" &> /dev/null && echo "Self-signed certificate generate [ OK ]" || exit 2
			fi
		fi

		if [ "$FORCESSL" == "false" ]; then
			{
				echo "force_local_logins_ssl=NO"
				echo "force_local_data_ssl=NO"
			} >> "${VSFTPDDIR}/vsftpd.conf"
		fi

		touch "${VSFTPDDIR}/vsftpd.user_list"
	fi

	# VSFTPd standard log container redirection
	tail -f "${LOGDIR}"/vsftpd.log | tee /dev/stdout &
	tail -f "${LOGDIR}"/xferlog.log | tee /dev/stdout &

cat << EOB

****************************************************
*                                                  *
*    Docker image: oscarenzo/vsftpd                *
*    https://gitlab.com/docker-files1/vsftpd       *
*                                                  *
****************************************************

SERVER SETTINGS
---------------
· FTP host: $PASV_ADDRESS
· FTP user: $FTP_USER
· FTP password: $FTP_PASSWORD
· PATH: $FTP_REPOSITORY
· Promiscuous: $PASV_PROMISCUOUS
· SSL enabled: $USESSL
· SSL forced: $FORCESSL
---------------

EOB

"$@" "${VSFTPDDIR}/vsftpd.conf" &
pid="${!}"
wait "${pid}"
fi
```

vsftpd.conf配置文件如下

```shell
# Run in the foreground to keep the container running:
background=NO

# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
anonymous_enable=NO

# Uncomment this to allow local users to log in.
# When SELinux is enforcing check for SE bool ftp_home_dir
local_enable=YES

# Uncomment this to enable any form of FTP write command.
write_enable=YES

# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
local_umask=022

# Uncomment this to allow the anonymous FTP user to upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
# When SELinux is enforcing check for SE bool allow_ftpd_anon_write, allow_ftpd_full_access
anon_upload_enable=NO

# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
anon_mkdir_write_enable=NO

# Activate directory messages - messages given to remote users when they
# go into a certain directory.
dirmessage_enable=YES

# Activate logging of uploads/downloads.
xferlog_enable=YES

# Make sure PORT transfer connections originate from port 20 (ftp-data).
connect_from_port_20=YES

# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
chown_uploads=NO
chown_username=root

# This option is the name of the file to which we write the vsftpd style log file.
# This log is only written if the option xferlog_enable is set, and xferlog_std_format is NOT set.
# Alternatively, it is written if you have set the option dual_log_enable.
# One further complication - if you have set syslog_enable, then this file is not written and output is sent to the system log instead.
vsftpd_log_file=/var/log/vsftpd/vsftpd.log

# This option is the name of the file to which we write the wu-ftpd style transfer log.
# The transfer log is only written if the option xferlog_enable is set, along with xferlog_std_format.
# Alternatively, it is written if you have set the option dual_log_enable.
xferlog_file=/var/log/vsftpd/xferlog.log

# If you want, you can have your log file in standard ftpd xferlog format.
# Note that the default log file location is /var/log/xferlog in this case.
xferlog_std_format=YES

# You may change the default value for timing out an idle session.
idle_session_timeout=600

# You may change the default value for timing out a data connection.
data_connection_timeout=300

# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
#nopriv_user=ftpsecure

# Enable this and the server will recognise asynchronous ABOR requests. Not
# recommended for security (the code is non-trivial). Not enabling it,
# however, may confuse older FTP clients.
#async_abor_enable=YES

# By default the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode. The vsftpd.conf(5) man page explains
# the behaviour when these options are disabled.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
ascii_upload_enable=NO
ascii_download_enable=NO

# You may specify a file of disallowed anonymous e-mail addresses. Apparently
# useful for combatting certain DoS attacks.
#deny_email_enable=YES
# (default follows)
#banned_email_file=/etc/vsftpd/banned_emails

# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
# (Warning! chroot'ing can be very dangerous. If using chroot, make sure that
# the user does not have write access to the top level directory within the
# chroot)
chroot_local_user=YES
#chroot_list_enable=YES
# (default follows)
#chroot_list_file=/etc/vsftpd/chroot_list

# Allow chroot writeable
allow_writeable_chroot=YES

# This option should be the name of a directory which is empty.
# Also, the directory should not be writable by the ftp user.
# This directory is used as a secure chroot() jail at times vsftpd does not require filesystem access.
# Default: /usr/share/empty
secure_chroot_dir=/var/run/vsftpd/empty

# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
ls_recurse_enable=NO

# When "listen" directive is enabled, vsftpd runs in standalone mode and
# listens on IPv4 sockets. This directive cannot be used in conjunction
# with the listen_ipv6 directive.
listen=YES

# This directive enables listening on IPv6 sockets. By default, listening
# on the IPv6 "any" address (::) will accept connections from both IPv6
# and IPv4 clients. It is not necessary to listen on *both* IPv4 and IPv6
# sockets. If you want that (perhaps because you want to listen on specific
# addresses) then you must run two copies of vsftpd with two configuration
# files.
# Make sure, that one of the listen options is commented !!
listen_ipv6=NO

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES

# Logging
log_ftp_protocol=YES
syslog_enable=NO
dual_log_enable=YES

# Variables set at container runtime
ftpd_banner=FTP_SERVER_NAME_CUSTOM
pasv_address=PASV_ADDRESS_CUSTOM
pasv_addr_resolve=NO
pasv_enable=YES
```

将以上三个文件放到同一目录中，使用`docker build -t hackerbs/vsftpd:latest .`构建即可。

git仓库地址：[https://github.com/1949hacker/vsftpd](https://github.com/1949hacker/vsftpd)
