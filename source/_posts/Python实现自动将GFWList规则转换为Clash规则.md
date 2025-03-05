---
title: Python实现自动将GFWList规则转换为Clash规则
comments: true
categories: [知识库]
date: 2025-03-05 14:50:15
keywords:
		- Python
		- Clash
		- 代理
		- GFWList
		- GEO
		- 分流
tags:
		- Python
		- Clash
		- 代理
		- GFWList
		- GEO
		- 分流
---

## 环境介绍

### 代理客户端为Clash Verge Rev

[直达官方github仓库](https://github.com/clash-verge-rev/clash-verge-rev)

### 机场为狗狗加速

[直达官网](https://down.dginv.click/#/register?code=pGwFvdKk)

使用邀请码享注册优惠`pGwFvdKk`

- 高性能海外机场，免费试用，优惠套餐，解锁流媒体，全球首家支持 Hysteria 协议。

- 海外团队，无跑路风险，高达 50% 返佣

- 集群负载均衡设计，高速专线(兼容老客户端)，极低延迟，无视晚高峰，4K 秒开

- 全球首家 Hysteria 协议机场，现已上线更快的 Hysteria2 协议(Clash Verge 客户端最佳搭配)

- 解锁流媒体及 ChatGPT

|套餐|流量|详情|
|---|---|---|
|标准套餐|160G|限速150M，无审计，年付9折|
|高级套餐|200G|不限速，无审计，年付8折,最新Hysteria协议|
|豪华套餐|500G|不限速,无审计,2年付6折,最新Hysteria2协议,东南亚节点,微信客服技术支持|

### 开发环境

Python 3.13.2

<!-- more -->

## Clash配置文件的介绍

### 其余部分不做介绍,采用机场的配置即可

本文只涉及`rules:`部分,其余部分直接复制下方从狗狗加速官方提取的配置即可

`proxies`和`proxy-groups`部分不需要填写,让机场的配置文件覆盖即可,当然下面部分的配置也可以留空,让机场覆盖,只填写`rules:`部分就可以实现分流

在编辑配置文件之前,先订阅,然后查看当前配置就可以看到机场的`rules:`,可以先将其备份,添加到GFWList的规则之前,也就是程序中的最高优先级规则部分

![20250305152944](https://img.hackerbs.com//20250305152944.png)

```yaml
mode: rule
mixed-port: 7897
allow-lan: false
log-level: info
external-controller: 127.0.0.1:9097
secret: ''
unified-delay: true
tun:
  mtu: 1500
  auto-detect-interface: true
  auto-route: true
  device: Mihomo
  dns-hijack:
  - any:53
  stack: mixed
  strict-route: false
  enable: true
dns:
  enable: true
  use-system-hosts: false
  listen: 127.0.0.1:5335
  default-nameserver:
  - 180.76.76.76
  - 182.254.118.118
  - 8.8.8.8
  - 180.184.2.2
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
  - '*.lan'
  - stun.*.*.*
  - stun.*.*
  - time.windows.com
  - time.nist.gov
  - time.apple.com
  - time.asia.apple.com
  - '*.ntp.org.cn'
  - '*.openwrt.pool.ntp.org'
  - time1.cloud.tencent.com
  - time.ustc.edu.cn
  - pool.ntp.org
  - ntp.ubuntu.com
  - ntp.aliyun.com
  - ntp1.aliyun.com
  - ntp2.aliyun.com
  - ntp3.aliyun.com
  - ntp4.aliyun.com
  - ntp5.aliyun.com
  - ntp6.aliyun.com
  - ntp7.aliyun.com
  - time1.aliyun.com
  - time2.aliyun.com
  - time3.aliyun.com
  - time4.aliyun.com
  - time5.aliyun.com
  - time6.aliyun.com
  - time7.aliyun.com
  - '*.time.edu.cn'
  - time1.apple.com
  - time2.apple.com
  - time3.apple.com
  - time4.apple.com
  - time5.apple.com
  - time6.apple.com
  - time7.apple.com
  - time1.google.com
  - time2.google.com
  - time3.google.com
  - time4.google.com
  - music.163.com
  - '*.music.163.com'
  - '*.126.net'
  - musicapi.taihe.com
  - music.taihe.com
  - songsearch.kugou.com
  - trackercdn.kugou.com
  - '*.kuwo.cn'
  - api-jooxtt.sanook.com
  - api.joox.com
  - joox.com
  - y.qq.com
  - '*.y.qq.com'
  - streamoc.music.tc.qq.com
  - mobileoc.music.tc.qq.com
  - isure.stream.qqmusic.qq.com
  - dl.stream.qqmusic.qq.com
  - aqqmusic.tc.qq.com
  - amobile.music.tc.qq.com
  - '*.xiami.com'
  - '*.music.migu.cn'
  - music.migu.cn
  - '*.msftconnecttest.com'
  - '*.msftncsi.com'
  - localhost.ptlogin2.qq.com
  - '*.*.*.srv.nintendo.net'
  - '*.*.stun.playstation.net'
  - xbox.*.*.microsoft.com
  - '*.ipv6.microsoft.com'
  - '*.*.xboxlive.com'
  - speedtest.cros.wr.pvp.net
  nameserver:
  - 180.76.76.76
  - 119.29.29.29
  - 180.184.1.1
  - 223.5.5.5
  - 8.8.8.8
  - https://223.6.6.6/dns-query#h3=true
  - https://223.5.5.5/dns-query
  - https://cloudflare-dns.com/dns-query
  - https://doh.pub/dns-query
  fallback:
  - https://000000.dns.nextdns.io/dns-query#h3=true
  - https://101.101.101.101/dns-query
  - https://208.67.220.220/dns-query
  - tls://8.8.4.4
  - tls://1.0.0.1:853
  - https://cloudflare-dns.com/dns-query
  - https://dns.google/dns-query
  fallback-filter:
    geoip: true
    ipcidr:
    - 240.0.0.0/4
    - 0.0.0.0/32
    - 127.0.0.1/32
    domain:
    - +.google.com
    - +.facebook.com
    - +.twitter.com
    - +.youtube.com
    - +.xn--ngstr-lra8j.com
    - +.google.cn
    - +.googleapis.cn
    - +.googleapis.com
    - +.gvt1.com
  ipv6: false
external-controller-cors:
  allow-private-network: true
  allow-origins:
  - '*'

profile:
  store-selected: true
```

### rules部分

规则参数:

DOMAIN - 匹配完整域名,不包含子域名.例如:- DOMAIN,example.com,DIRECT

DOMAIN-SUFFIX - 匹配域名后缀,可以匹配子域名.例如:- DOMAIN-SUFFIX,example.com,PROXY

DOMAIN-KEYWORD - 匹配域名中包含的关键字,不限定位置.例如:- DOMAIN-KEYWORD,google,PROXY

PROCESS-NAME(仅 Clash for Windows / Clash Meta 支持) - 匹配进程名,仅适用于 TUN 模式.例如:- PROCESS-NAME,chrome.exe,PROXY

IP-CIDR - 匹配IPv4 地址段,可以带 no-resolve 以防止 Clash 进行 DNS 解析.例如:- IP-CIDR,192.168.1.0/24,DIRECT
如果不想 Clash 解析 IP 地址对应的域名,可以加 no-resolve:- IP-CIDR,192.168.1.0/24,DIRECT,no-resolve

IP-CIDR6 - 匹配IPv6 地址段,用法与 IP-CIDR 类似.例如:- IP-CIDR6,2400:3200::/32,DIRECT

GEOIP - 匹配IP 地址归属地(基于 GeoIP 数据库).例如: - GEOIP,CN,DIRECT
如果要让 Clash 不解析域名,可以加 no-resolve:- GEOIP,CN,DIRECT,no-resolve

MATCH - 匹配所有流量,通常作为最后的兜底规则.例如:- MATCH,PROXY

其中的`DIRECT`为直连,`PROXY`为你的`proxy-groups`的`name`部分,例如狗狗加速的`name: 狗狗加速.com`和`name: 🔥ChatGPT`

```yaml
# 以狗狗加速机场的代理组"🔥ChatGPT"为例
- DOMAIN-KEYWORD,openai,🔥ChatGPT
```

Clash 按规则的先后顺序进行匹配，第一个匹配成功的规则生效，后面的规则不会再被执行。因此，一般规则顺序建议：

- PROCESS-NAME 规则
- DOMAIN 规则
- DOMAIN-SUFFIX 规则
- DOMAIN-KEYWORD 规则
- IP-CIDR / IP-CIDR6 规则
- GEOIP 规则
- MATCH 规则（兜底）

## 代码

```python
import re
import base64
import requests
import ipaddress

# GFWList 地址
GFWLIST_URL = "https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"

# 预置规则 - 来源于上文提到的机场的规则和你自己手动添加的规则
# 此处预置了狗狗加速机场的规则
highest_priority_rules = """rules:
- DOMAIN,subapi.doggysub.uk,DIRECT
- PROCESS-NAME,v2ray,DIRECT
- PROCESS-NAME,xray,DIRECT
- PROCESS-NAME,naive,DIRECT
- PROCESS-NAME,trojan,DIRECT
- PROCESS-NAME,trojan-go,DIRECT
- PROCESS-NAME,ss-local,DIRECT
- PROCESS-NAME,privoxy,DIRECT
- PROCESS-NAME,leaf,DIRECT
- PROCESS-NAME,v2ray.exe,DIRECT
- PROCESS-NAME,xray.exe,DIRECT
- PROCESS-NAME,naive.exe,DIRECT
- PROCESS-NAME,trojan.exe,DIRECT
- PROCESS-NAME,trojan-go.exe,DIRECT
- PROCESS-NAME,ss-local.exe,DIRECT
- PROCESS-NAME,privoxy.exe,DIRECT
- PROCESS-NAME,leaf.exe,DIRECT
- PROCESS-NAME,Surge,DIRECT
- PROCESS-NAME,Surge 2,DIRECT
- PROCESS-NAME,Surge 3,DIRECT
- PROCESS-NAME,Surge 4,DIRECT
- PROCESS-NAME,Surge%202,DIRECT
- PROCESS-NAME,Surge%203,DIRECT
- PROCESS-NAME,Surge%204,DIRECT
- PROCESS-NAME,Thunder,DIRECT
- PROCESS-NAME,DownloadService,DIRECT
- PROCESS-NAME,qBittorrent,DIRECT
- PROCESS-NAME,Transmission,DIRECT
- PROCESS-NAME,fdm,DIRECT
- PROCESS-NAME,aria2c,DIRECT
- PROCESS-NAME,Folx,DIRECT
- PROCESS-NAME,NetTransport,DIRECT
- PROCESS-NAME,uTorrent,DIRECT
- PROCESS-NAME,WebTorrent,DIRECT
- PROCESS-NAME,aria2c.exe,DIRECT
- PROCESS-NAME,BitComet.exe,DIRECT
- PROCESS-NAME,fdm.exe,DIRECT
- PROCESS-NAME,NetTransport.exe,DIRECT
- PROCESS-NAME,qbittorrent.exe,DIRECT
- PROCESS-NAME,Thunder.exe,DIRECT
- PROCESS-NAME,ThunderVIP.exe,DIRECT
- PROCESS-NAME,transmission-daemon.exe,DIRECT
- PROCESS-NAME,transmission-qt.exe,DIRECT
- PROCESS-NAME,uTorrent.exe,DIRECT
- PROCESS-NAME,WebTorrent.exe,DIRECT
- PROCESS-NAME,aDrive.exe,DIRECT
- DOMAIN-SUFFIX,services.googleapis.cn,狗狗加速.com
- DOMAIN-SUFFIX,xn--ngstr-lra8j.com,狗狗加速.com
- DOMAIN,safebrowsing.urlsec.qq.com,DIRECT
- DOMAIN,safebrowsing.googleapis.com,DIRECT
- DOMAIN,developer.apple.com,狗狗加速.com
- DOMAIN-SUFFIX,digicert.com,狗狗加速.com
- DOMAIN,ocsp.apple.com,狗狗加速.com
- DOMAIN,ocsp.comodoca.com,狗狗加速.com
- DOMAIN,ocsp.usertrust.com,狗狗加速.com
- DOMAIN,ocsp.sectigo.com,狗狗加速.com
- DOMAIN,ocsp.verisign.net,狗狗加速.com
- DOMAIN-SUFFIX,apple-dns.net,狗狗加速.com
- DOMAIN,testflight.apple.com,狗狗加速.com
- DOMAIN,sandbox.itunes.apple.com,狗狗加速.com
- DOMAIN,itunes.apple.com,狗狗加速.com
- DOMAIN-SUFFIX,apps.apple.com,狗狗加速.com
- DOMAIN-SUFFIX,blobstore.apple.com,狗狗加速.com
- DOMAIN,cvws.icloud-content.com,狗狗加速.com
- DOMAIN-SUFFIX,mzstatic.com,DIRECT
- DOMAIN-SUFFIX,itunes.apple.com,DIRECT
- DOMAIN-SUFFIX,icloud.com,DIRECT
- DOMAIN-SUFFIX,icloud-content.com,DIRECT
- DOMAIN-SUFFIX,me.com,DIRECT
- DOMAIN-SUFFIX,aaplimg.com,DIRECT
- DOMAIN-SUFFIX,cdn20.com,DIRECT
- DOMAIN-SUFFIX,cdn-apple.com,DIRECT
- DOMAIN-SUFFIX,akadns.net,DIRECT
- DOMAIN-SUFFIX,akamaiedge.net,DIRECT
- DOMAIN-SUFFIX,edgekey.net,DIRECT
- DOMAIN-SUFFIX,mwcloudcdn.com,DIRECT
- DOMAIN-SUFFIX,mwcname.com,DIRECT
- DOMAIN-SUFFIX,apple.com,DIRECT
- DOMAIN-SUFFIX,apple-cloudkit.com,DIRECT
- DOMAIN-SUFFIX,apple-mapkit.com,DIRECT
- DOMAIN,browser-intake-datadoghq.com,🔥ChatGPT
- DOMAIN,chat.openai.com.cdn.cloudflare.net,🔥ChatGPT
- DOMAIN,gemini.google.com,🔥ChatGPT
- DOMAIN,openai-api.arkoselabs.com,🔥ChatGPT
- DOMAIN,openaicom-api-bdcpf8c6d2e9atf6.z01.azurefd.net,🔥ChatGPT
- DOMAIN,openaicomproductionae4b.blob.core.windows.net,🔥ChatGPT
- DOMAIN,production-openaicom-storage.azureedge.net,🔥ChatGPT
- DOMAIN,static.cloudflareinsights.com,🔥ChatGPT
- DOMAIN-SUFFIX,ai.com,🔥ChatGPT
- DOMAIN-SUFFIX,algolia.net,🔥ChatGPT
- DOMAIN-SUFFIX,api.statsig.com,🔥ChatGPT
- DOMAIN-SUFFIX,auth0.com,🔥ChatGPT
- DOMAIN-SUFFIX,chatgpt.com,🔥ChatGPT
- DOMAIN-SUFFIX,chatgpt.livekit.cloud,🔥ChatGPT
- DOMAIN-SUFFIX,client-api.arkoselabs.com,🔥ChatGPT
- DOMAIN-SUFFIX,events.statsigapi.net,🔥ChatGPT
- DOMAIN-SUFFIX,featuregates.org,🔥ChatGPT
- DOMAIN-SUFFIX,host.livekit.cloud,🔥ChatGPT
- DOMAIN-SUFFIX,identrust.com,🔥ChatGPT
- DOMAIN-SUFFIX,intercom.io,🔥ChatGPT
- DOMAIN-SUFFIX,intercomcdn.com,🔥ChatGPT
- DOMAIN-SUFFIX,launchdarkly.com,🔥ChatGPT
- DOMAIN-SUFFIX,oaistatic.com,🔥ChatGPT
- DOMAIN-SUFFIX,oaiusercontent.com,🔥ChatGPT
- DOMAIN-SUFFIX,observeit.net,🔥ChatGPT
- DOMAIN-SUFFIX,openai.com,🔥ChatGPT
- DOMAIN-SUFFIX,openaiapi-site.azureedge.net,🔥ChatGPT
- DOMAIN-SUFFIX,openaicom.imgix.net,🔥ChatGPT
- DOMAIN-SUFFIX,segment.io,🔥ChatGPT
- DOMAIN-SUFFIX,sentry.io,🔥ChatGPT
- DOMAIN-SUFFIX,stripe.com,🔥ChatGPT
- DOMAIN-SUFFIX,turn.livekit.cloud,🔥ChatGPT
- DOMAIN-SUFFIX,sora.com,🔥ChatGPT
- DOMAIN-KEYWORD,openai,🔥ChatGPT
- DOMAIN,api.msn.com,🔥ChatGPT
- DOMAIN,api.statsig.com,🔥ChatGPT
- DOMAIN,assets.msn.com,🔥ChatGPT
- DOMAIN,browser-intake-datadoghq.com,🔥ChatGPT
- DOMAIN,chat.openai.com.cdn.cloudflare.net,🔥ChatGPT
- DOMAIN,copilot.microsoft.com,🔥ChatGPT
- DOMAIN,gateway.bingviz.microsoft.net,🔥ChatGPT
- DOMAIN,gateway.bingviz.microsoftapp.net,🔥ChatGPT
- DOMAIN,in.appcenter.ms,🔥ChatGPT
- DOMAIN,location.microsoft.com,🔥ChatGPT
- DOMAIN,odc.officeapps.live.com,🔥ChatGPT
- DOMAIN,openai-api.arkoselabs.com,🔥ChatGPT
- DOMAIN,openaicomproductionae4b.blob.core.windows.net,🔥ChatGPT
- DOMAIN,production-openaicom-storage.azureedge.net,🔥ChatGPT
- DOMAIN,r.bing.com,🔥ChatGPT
- DOMAIN,self.events.data.microsoft.com,🔥ChatGPT
- DOMAIN,services.bingapis.com,🔥ChatGPT
- DOMAIN,static.cloudflareinsights.com,🔥ChatGPT
- DOMAIN,sydney.bing.com,🔥ChatGPT
- DOMAIN,www.bing.com,🔥ChatGPT
- DOMAIN-SUFFIX,algolia.net,🔥ChatGPT
- DOMAIN-SUFFIX,api.microsoftapp.net,🔥ChatGPT
- DOMAIN-SUFFIX,auth0.com,🔥ChatGPT
- DOMAIN-SUFFIX,bing-shopping.microsoft-falcon.io,🔥ChatGPT
- DOMAIN-SUFFIX,challenges.cloudflare.com,🔥ChatGPT
- DOMAIN-SUFFIX,chatgpt.com,🔥ChatGPT
- DOMAIN-SUFFIX,chatgpt.livekit.cloud,🔥ChatGPT
- DOMAIN-SUFFIX,client-api.arkoselabs.com,🔥ChatGPT
- DOMAIN-SUFFIX,edgeservices.bing.com,🔥ChatGPT
- DOMAIN-SUFFIX,events.statsigapi.net,🔥ChatGPT
- DOMAIN-SUFFIX,featuregates.org,🔥ChatGPT
- DOMAIN-SUFFIX,host.livekit.cloud,🔥ChatGPT
- DOMAIN-SUFFIX,identrust.com,🔥ChatGPT
- DOMAIN-SUFFIX,intercom.io,🔥ChatGPT
- DOMAIN-SUFFIX,intercomcdn.com,🔥ChatGPT
- DOMAIN-SUFFIX,launchdarkly.com,🔥ChatGPT
- DOMAIN-SUFFIX,oaistatic.com,🔥ChatGPT
- DOMAIN-SUFFIX,oaiusercontent.com,🔥ChatGPT
- DOMAIN-SUFFIX,observeit.net,🔥ChatGPT
- DOMAIN-SUFFIX,openai.com,🔥ChatGPT
- DOMAIN-SUFFIX,openaiapi-site.azureedge.net,🔥ChatGPT
- DOMAIN-SUFFIX,openaicom.imgix.net,🔥ChatGPT
- DOMAIN-SUFFIX,segment.io,🔥ChatGPT
- DOMAIN-SUFFIX,sentry.io,🔥ChatGPT
- DOMAIN-SUFFIX,stripe.com,🔥ChatGPT
- DOMAIN-SUFFIX,turn.livekit.cloud,🔥ChatGPT
- DOMAIN-KEYWORD,openaicom-api,🔥ChatGPT
- DOMAIN,ai.google.dev,🔥ChatGPT
- DOMAIN,alkalimakersuite-pa.clients6.google.com,🔥ChatGPT
- DOMAIN,makersuite.google.com,🔥ChatGPT
- DOMAIN-SUFFIX,bard.google.com,🔥ChatGPT
- DOMAIN-SUFFIX,deepmind.com,🔥ChatGPT
- DOMAIN-SUFFIX,deepmind.google,🔥ChatGPT
- DOMAIN-SUFFIX,gemini.google.com,🔥ChatGPT
- DOMAIN-SUFFIX,generativeai.google,🔥ChatGPT
- DOMAIN-SUFFIX,proactivebackend-pa.googleapis.com,🔥ChatGPT
- DOMAIN-SUFFIX,apis.google.com,🔥ChatGPT
- DOMAIN-KEYWORD,colab,🔥ChatGPT
- DOMAIN-KEYWORD,developerprofiles,🔥ChatGPT
- DOMAIN-KEYWORD,generativelanguage,🔥ChatGPT
- DOMAIN,cdn.usefathom.com,🔥ChatGPT
- DOMAIN-SUFFIX,anthropic.com,🔥ChatGPT
- DOMAIN-SUFFIX,claude.ai,🔥ChatGPT
- DOMAIN-SUFFIX,razie.ai,🔥ChatGPT
- DOMAIN-SUFFIX,razie.aws.intellij.net,🔥ChatGPT
- DOMAIN-SUFFIX,etbrains.ai,🔥ChatGPT
- DOMAIN-SUFFIX,meta.com,🔥ChatGPT
- DOMAIN-SUFFIX,cn,DIRECT
- DOMAIN-KEYWORD,-cn,DIRECT
- DOMAIN-SUFFIX,126.com,DIRECT
- DOMAIN-SUFFIX,126.net,DIRECT
- DOMAIN-SUFFIX,127.net,DIRECT
- DOMAIN-SUFFIX,163.com,DIRECT
- DOMAIN-SUFFIX,360buyimg.com,DIRECT
- DOMAIN-SUFFIX,36kr.com,DIRECT
- DOMAIN-SUFFIX,acfun.tv,DIRECT
- DOMAIN-SUFFIX,air-matters.com,DIRECT
- DOMAIN-SUFFIX,aixifan.com,DIRECT
- DOMAIN-KEYWORD,alicdn,DIRECT
- DOMAIN-KEYWORD,alipay,DIRECT
- DOMAIN-KEYWORD,taobao,DIRECT
- DOMAIN-SUFFIX,amap.com,DIRECT
- DOMAIN-SUFFIX,autonavi.com,DIRECT
- DOMAIN-KEYWORD,baidu,DIRECT
- DOMAIN-SUFFIX,bdimg.com,DIRECT
- DOMAIN-SUFFIX,bdstatic.com,DIRECT
- DOMAIN-SUFFIX,bilibili.com,DIRECT
- DOMAIN-SUFFIX,bilivideo.com,DIRECT
- DOMAIN-SUFFIX,caiyunapp.com,DIRECT
- DOMAIN-SUFFIX,clouddn.com,DIRECT
- DOMAIN-SUFFIX,cnbeta.com,DIRECT
- DOMAIN-SUFFIX,cnbetacdn.com,DIRECT
- DOMAIN-SUFFIX,cootekservice.com,DIRECT
- DOMAIN-SUFFIX,csdn.net,DIRECT
- DOMAIN-SUFFIX,ctrip.com,DIRECT
- DOMAIN-SUFFIX,dgtle.com,DIRECT
- DOMAIN-SUFFIX,dianping.com,DIRECT
- DOMAIN-SUFFIX,douban.com,DIRECT
- DOMAIN-SUFFIX,doubanio.com,DIRECT
- DOMAIN-SUFFIX,duokan.com,DIRECT
- DOMAIN-SUFFIX,easou.com,DIRECT
- DOMAIN-SUFFIX,ele.me,DIRECT
- DOMAIN-SUFFIX,feng.com,DIRECT
- DOMAIN-SUFFIX,fir.im,DIRECT
- DOMAIN-SUFFIX,frdic.com,DIRECT
- DOMAIN-SUFFIX,g-cores.com,DIRECT
- DOMAIN-SUFFIX,godic.net,DIRECT
- DOMAIN-SUFFIX,gtimg.com,DIRECT
- DOMAIN,cdn.hockeyapp.net,DIRECT
- DOMAIN-SUFFIX,hongxiu.com,DIRECT
- DOMAIN-SUFFIX,hxcdn.net,DIRECT
- DOMAIN-SUFFIX,iciba.com,DIRECT
- DOMAIN-SUFFIX,ifeng.com,DIRECT
- DOMAIN-SUFFIX,ifengimg.com,DIRECT
- DOMAIN-SUFFIX,ipip.net,DIRECT
- DOMAIN-SUFFIX,iqiyi.com,DIRECT
- DOMAIN-SUFFIX,jd.com,DIRECT
- DOMAIN-SUFFIX,jianshu.com,DIRECT
- DOMAIN-SUFFIX,knewone.com,DIRECT
- DOMAIN-SUFFIX,le.com,DIRECT
- DOMAIN-SUFFIX,lecloud.com,DIRECT
- DOMAIN-SUFFIX,lemicp.com,DIRECT
- DOMAIN-SUFFIX,luoo.net,DIRECT
- DOMAIN-SUFFIX,meituan.com,DIRECT
- DOMAIN-SUFFIX,meituan.net,DIRECT
- DOMAIN-SUFFIX,mi.com,DIRECT
- DOMAIN-SUFFIX,miaopai.com,DIRECT
- DOMAIN-SUFFIX,microsoft.com,DIRECT
- DOMAIN-SUFFIX,microsoftonline.com,DIRECT
- DOMAIN-SUFFIX,miui.com,DIRECT
- DOMAIN-SUFFIX,miwifi.com,DIRECT
- DOMAIN-SUFFIX,mob.com,DIRECT
- DOMAIN-SUFFIX,netease.com,DIRECT
- DOMAIN-SUFFIX,office.com,DIRECT
- DOMAIN-SUFFIX,office365.com,DIRECT
- DOMAIN-KEYWORD,officecdn,DIRECT
- DOMAIN-SUFFIX,oschina.net,DIRECT
- DOMAIN-SUFFIX,ppsimg.com,DIRECT
- DOMAIN-SUFFIX,pstatp.com,DIRECT
- DOMAIN-SUFFIX,qcloud.com,DIRECT
- DOMAIN-SUFFIX,qdaily.com,DIRECT
- DOMAIN-SUFFIX,qdmm.com,DIRECT
- DOMAIN-SUFFIX,qhimg.com,DIRECT
- DOMAIN-SUFFIX,qhres.com,DIRECT
- DOMAIN-SUFFIX,qidian.com,DIRECT
- DOMAIN-SUFFIX,qihucdn.com,DIRECT
- DOMAIN-SUFFIX,qiniu.com,DIRECT
- DOMAIN-SUFFIX,qiniucdn.com,DIRECT
- DOMAIN-SUFFIX,qiyipic.com,DIRECT
- DOMAIN-SUFFIX,qq.com,DIRECT
- DOMAIN-SUFFIX,qqurl.com,DIRECT
- DOMAIN-SUFFIX,rarbg.to,DIRECT
- DOMAIN-SUFFIX,ruguoapp.com,DIRECT
- DOMAIN-SUFFIX,segmentfault.com,DIRECT
- DOMAIN-SUFFIX,sinaapp.com,DIRECT
- DOMAIN-SUFFIX,smzdm.com,DIRECT
- DOMAIN-SUFFIX,snapdrop.net,DIRECT
- DOMAIN-SUFFIX,sogou.com,DIRECT
- DOMAIN-SUFFIX,sogoucdn.com,DIRECT
- DOMAIN-SUFFIX,sohu.com,DIRECT
- DOMAIN-SUFFIX,soku.com,DIRECT
- DOMAIN-SUFFIX,speedtest.net,DIRECT
- DOMAIN-SUFFIX,sspai.com,DIRECT
- DOMAIN-SUFFIX,suning.com,DIRECT
- DOMAIN-SUFFIX,taobao.com,DIRECT
- DOMAIN-SUFFIX,tencent.com,DIRECT
- DOMAIN-SUFFIX,tenpay.com,DIRECT
- DOMAIN-SUFFIX,tianyancha.com,DIRECT
- DOMAIN-SUFFIX,tmall.com,DIRECT
- DOMAIN-SUFFIX,tudou.com,DIRECT
- DOMAIN-SUFFIX,umetrip.com,DIRECT
- DOMAIN-SUFFIX,upaiyun.com,DIRECT
- DOMAIN-SUFFIX,upyun.com,DIRECT
- DOMAIN-SUFFIX,veryzhun.com,DIRECT
- DOMAIN-SUFFIX,weather.com,DIRECT
- DOMAIN-SUFFIX,weibo.com,DIRECT
- DOMAIN-SUFFIX,xiami.com,DIRECT
- DOMAIN-SUFFIX,xiami.net,DIRECT
- DOMAIN-SUFFIX,xiaomicp.com,DIRECT
- DOMAIN-SUFFIX,ximalaya.com,DIRECT
- DOMAIN-SUFFIX,xmcdn.com,DIRECT
- DOMAIN-SUFFIX,xunlei.com,DIRECT
- DOMAIN-SUFFIX,yhd.com,DIRECT
- DOMAIN-SUFFIX,yihaodianimg.com,DIRECT
- DOMAIN-SUFFIX,yinxiang.com,DIRECT
- DOMAIN-SUFFIX,ykimg.com,DIRECT
- DOMAIN-SUFFIX,youdao.com,DIRECT
- DOMAIN-SUFFIX,youku.com,DIRECT
- DOMAIN-SUFFIX,zealer.com,DIRECT
- DOMAIN-SUFFIX,zhihu.com,DIRECT
- DOMAIN-SUFFIX,zhimg.com,DIRECT
- DOMAIN-SUFFIX,zimuzu.tv,DIRECT
- DOMAIN-SUFFIX,zoho.com,DIRECT
- DOMAIN,fastly-download.epicgames.com,DIRECT
- DOMAIN,epicgames-download1.akamaized.net,DIRECT
- DOMAIN,steamcdn-a.akamaihd.net,DIRECT
- DOMAIN-SUFFIX,steamserver.net,DIRECT
- DOMAIN-SUFFIX,cm.steampowered.com,DIRECT
- DOMAIN-SUFFIX,synology.com,DIRECT
- DOMAIN-SUFFIX,dyndns.org,DIRECT
- DOMAIN-SUFFIX,egdownload.fastly-edge.com,DIRECT
- DOMAIN-KEYWORD,amazon,狗狗加速.com
- DOMAIN-KEYWORD,google,狗狗加速.com
- DOMAIN-KEYWORD,gmail,狗狗加速.com
- DOMAIN-KEYWORD,youtube,狗狗加速.com
- DOMAIN-KEYWORD,facebook,狗狗加速.com
- DOMAIN-SUFFIX,fb.me,狗狗加速.com
- DOMAIN-SUFFIX,fbcdn.net,狗狗加速.com
- DOMAIN-KEYWORD,twitter,狗狗加速.com
- DOMAIN-KEYWORD,instagram,狗狗加速.com
- DOMAIN-KEYWORD,dropbox,狗狗加速.com
- DOMAIN-SUFFIX,twimg.com,狗狗加速.com
- DOMAIN-KEYWORD,blogspot,狗狗加速.com
- DOMAIN-SUFFIX,youtu.be,狗狗加速.com
- DOMAIN-KEYWORD,whatsapp,狗狗加速.com
- DOMAIN-SUFFIX,ipleak.net,狗狗加速.com
- DOMAIN-SUFFIX,browserscan.net,狗狗加速.com
- DOMAIN-SUFFIX,surfsharkdns.com,狗狗加速.com
- DOMAIN-SUFFIX,edns.ip-api.com,狗狗加速.com
- DOMAIN-SUFFIX,dnsleaktest.com,狗狗加速.com
- DOMAIN-SUFFIX,dnsleak.com,狗狗加速.com
- DOMAIN-SUFFIX,expressvpn.com,狗狗加速.com
- DOMAIN-SUFFIX,nordvpn.com,狗狗加速.com
- DOMAIN-SUFFIX,surfshark.com,狗狗加速.com
- DOMAIN-SUFFIX,perfect-privacy.com,狗狗加速.com
- DOMAIN-SUFFIX,browserleaks.com,狗狗加速.com
- DOMAIN-SUFFIX,browserleaks.org,狗狗加速.com
- DOMAIN-SUFFIX,browserleaks.net,狗狗加速.com
- DOMAIN-SUFFIX,vpnunlimited.com,狗狗加速.com
- DOMAIN-SUFFIX,whoer.net,狗狗加速.com
- DOMAIN-SUFFIX,whrq.net,狗狗加速.com
- DOMAIN-KEYWORD,admarvel,REJECT
- DOMAIN-KEYWORD,admaster,REJECT
- DOMAIN-KEYWORD,adsage,REJECT
- DOMAIN-KEYWORD,adsmogo,REJECT
- DOMAIN-KEYWORD,adsrvmedia,REJECT
- DOMAIN-KEYWORD,adwords,REJECT
- DOMAIN-KEYWORD,adservice,REJECT
- DOMAIN-SUFFIX,appsflyer.com,REJECT
- DOMAIN-KEYWORD,domob,REJECT
- DOMAIN-SUFFIX,doubleclick.net,REJECT
- DOMAIN-KEYWORD,duomeng,REJECT
- DOMAIN-KEYWORD,dwtrack,REJECT
- DOMAIN-KEYWORD,guanggao,REJECT
- DOMAIN-KEYWORD,lianmeng,REJECT
- DOMAIN-SUFFIX,mmstat.com,REJECT
- DOMAIN-KEYWORD,mopub,REJECT
- DOMAIN-KEYWORD,omgmta,REJECT
- DOMAIN-KEYWORD,openx,REJECT
- DOMAIN-KEYWORD,partnerad,REJECT
- DOMAIN-KEYWORD,pingfore,REJECT
- DOMAIN-KEYWORD,supersonicads,REJECT
- DOMAIN-KEYWORD,uedas,REJECT
- DOMAIN-KEYWORD,umeng,REJECT
- DOMAIN-KEYWORD,usage,REJECT
- DOMAIN-SUFFIX,vungle.com,REJECT
- DOMAIN-KEYWORD,wlmonitor,REJECT
- DOMAIN-KEYWORD,zjtoolbar,REJECT
- DOMAIN-SUFFIX,linkedin.com,狗狗加速.com
- DOMAIN-SUFFIX,licdn.com,狗狗加速.com
- DOMAIN-SUFFIX,9to5mac.com,狗狗加速.com
- DOMAIN-SUFFIX,abpchina.org,狗狗加速.com
- DOMAIN-SUFFIX,adblockplus.org,狗狗加速.com
- DOMAIN-SUFFIX,adobe.com,狗狗加速.com
- DOMAIN-SUFFIX,akamaized.net,狗狗加速.com
- DOMAIN-SUFFIX,alfredapp.com,狗狗加速.com
- DOMAIN-SUFFIX,amplitude.com,狗狗加速.com
- DOMAIN-SUFFIX,ampproject.org,狗狗加速.com
- DOMAIN-SUFFIX,android.com,狗狗加速.com
- DOMAIN-SUFFIX,angularjs.org,狗狗加速.com
- DOMAIN-SUFFIX,aolcdn.com,狗狗加速.com
- DOMAIN-SUFFIX,apkpure.com,狗狗加速.com
- DOMAIN-SUFFIX,appledaily.com,狗狗加速.com
- DOMAIN-SUFFIX,appshopper.com,狗狗加速.com
- DOMAIN-SUFFIX,appspot.com,狗狗加速.com
- DOMAIN-SUFFIX,arcgis.com,狗狗加速.com
- DOMAIN-SUFFIX,archive.org,狗狗加速.com
- DOMAIN-SUFFIX,armorgames.com,狗狗加速.com
- DOMAIN-SUFFIX,aspnetcdn.com,狗狗加速.com
- DOMAIN-SUFFIX,att.com,狗狗加速.com
- DOMAIN-SUFFIX,awsstatic.com,狗狗加速.com
- DOMAIN-SUFFIX,azurewebsites.net,狗狗加速.com
- DOMAIN-SUFFIX,bing.com,狗狗加速.com
- DOMAIN-SUFFIX,bintray.com,狗狗加速.com
- DOMAIN-SUFFIX,bit.com,狗狗加速.com
- DOMAIN-SUFFIX,bit.ly,狗狗加速.com
- DOMAIN-SUFFIX,bitbucket.org,狗狗加速.com
- DOMAIN-SUFFIX,bjango.com,狗狗加速.com
- DOMAIN-SUFFIX,bkrtx.com,狗狗加速.com
- DOMAIN-SUFFIX,blog.com,狗狗加速.com
- DOMAIN-SUFFIX,blogcdn.com,狗狗加速.com
- DOMAIN-SUFFIX,blogger.com,狗狗加速.com
- DOMAIN-SUFFIX,blogsmithmedia.com,狗狗加速.com
- DOMAIN-SUFFIX,blogspot.com,狗狗加速.com
- DOMAIN-SUFFIX,blogspot.hk,狗狗加速.com
- DOMAIN-SUFFIX,bloomberg.com,狗狗加速.com
- DOMAIN-SUFFIX,box.com,狗狗加速.com
- DOMAIN-SUFFIX,box.net,狗狗加速.com
- DOMAIN-SUFFIX,cachefly.net,狗狗加速.com
- DOMAIN-SUFFIX,chromium.org,狗狗加速.com
- DOMAIN-SUFFIX,cl.ly,狗狗加速.com
- DOMAIN-SUFFIX,cloudflare.com,狗狗加速.com
- DOMAIN-SUFFIX,cloudfront.net,狗狗加速.com
- DOMAIN-SUFFIX,cloudmagic.com,狗狗加速.com
- DOMAIN-SUFFIX,cmail19.com,狗狗加速.com
- DOMAIN-SUFFIX,cnet.com,狗狗加速.com
- DOMAIN-SUFFIX,cocoapods.org,狗狗加速.com
- DOMAIN-SUFFIX,comodoca.com,狗狗加速.com
- DOMAIN-SUFFIX,crashlytics.com,狗狗加速.com
- DOMAIN-SUFFIX,culturedcode.com,狗狗加速.com
- DOMAIN-SUFFIX,d.pr,狗狗加速.com
- DOMAIN-SUFFIX,danilo.to,狗狗加速.com
- DOMAIN-SUFFIX,dayone.me,狗狗加速.com
- DOMAIN-SUFFIX,db.tt,狗狗加速.com
- DOMAIN-SUFFIX,deskconnect.com,狗狗加速.com
- DOMAIN-SUFFIX,disq.us,狗狗加速.com
- DOMAIN-SUFFIX,disqus.com,狗狗加速.com
- DOMAIN-SUFFIX,disquscdn.com,狗狗加速.com
- DOMAIN-SUFFIX,dnsimple.com,狗狗加速.com
- DOMAIN-SUFFIX,docker.com,狗狗加速.com
- DOMAIN-SUFFIX,dribbble.com,狗狗加速.com
- DOMAIN-SUFFIX,droplr.com,狗狗加速.com
- DOMAIN-SUFFIX,duckduckgo.com,狗狗加速.com
- DOMAIN-SUFFIX,dueapp.com,狗狗加速.com
- DOMAIN-SUFFIX,dytt8.net,狗狗加速.com
- DOMAIN-SUFFIX,edgecastcdn.net,狗狗加速.com
- DOMAIN-SUFFIX,edgekey.net,狗狗加速.com
- DOMAIN-SUFFIX,edgesuite.net,狗狗加速.com
- DOMAIN-SUFFIX,engadget.com,狗狗加速.com
- DOMAIN-SUFFIX,entrust.net,狗狗加速.com
- DOMAIN-SUFFIX,eurekavpt.com,狗狗加速.com
- DOMAIN-SUFFIX,evernote.com,狗狗加速.com
- DOMAIN-SUFFIX,fabric.io,狗狗加速.com
- DOMAIN-SUFFIX,fast.com,狗狗加速.com
- DOMAIN-SUFFIX,fastly.net,狗狗加速.com
- DOMAIN-SUFFIX,fc2.com,狗狗加速.com
- DOMAIN-SUFFIX,feedburner.com,狗狗加速.com
- DOMAIN-SUFFIX,feedly.com,狗狗加速.com
- DOMAIN-SUFFIX,feedsportal.com,狗狗加速.com
- DOMAIN-SUFFIX,fiftythree.com,狗狗加速.com
- DOMAIN-SUFFIX,firebaseio.com,狗狗加速.com
- DOMAIN-SUFFIX,flexibits.com,狗狗加速.com
- DOMAIN-SUFFIX,flickr.com,狗狗加速.com
- DOMAIN-SUFFIX,flipboard.com,狗狗加速.com
- DOMAIN-SUFFIX,g.co,狗狗加速.com
- DOMAIN-SUFFIX,gabia.net,狗狗加速.com
- DOMAIN-SUFFIX,geni.us,狗狗加速.com
- DOMAIN-SUFFIX,gfx.ms,狗狗加速.com
- DOMAIN-SUFFIX,ggpht.com,狗狗加速.com
- DOMAIN-SUFFIX,ghostnoteapp.com,狗狗加速.com
- DOMAIN-SUFFIX,git.io,狗狗加速.com
- DOMAIN-KEYWORD,github,狗狗加速.com
- DOMAIN-SUFFIX,globalsign.com,狗狗加速.com
- DOMAIN-SUFFIX,gmodules.com,狗狗加速.com
- DOMAIN-SUFFIX,godaddy.com,狗狗加速.com
- DOMAIN-SUFFIX,golang.org,狗狗加速.com
- DOMAIN-SUFFIX,gongm.in,狗狗加速.com
- DOMAIN-SUFFIX,goo.gl,狗狗加速.com
- DOMAIN-SUFFIX,goodreaders.com,狗狗加速.com
- DOMAIN-SUFFIX,goodreads.com,狗狗加速.com
- DOMAIN-SUFFIX,gravatar.com,狗狗加速.com
- DOMAIN-SUFFIX,gstatic.com,狗狗加速.com
- DOMAIN-SUFFIX,gvt0.com,狗狗加速.com
- DOMAIN-SUFFIX,hockeyapp.net,狗狗加速.com
- DOMAIN-SUFFIX,hotmail.com,狗狗加速.com
- DOMAIN-SUFFIX,icons8.com,狗狗加速.com
- DOMAIN-SUFFIX,ifixit.com,狗狗加速.com
- DOMAIN-SUFFIX,ift.tt,狗狗加速.com
- DOMAIN-SUFFIX,ifttt.com,狗狗加速.com
- DOMAIN-SUFFIX,iherb.com,狗狗加速.com
- DOMAIN-SUFFIX,imageshack.us,狗狗加速.com
- DOMAIN-SUFFIX,img.ly,狗狗加速.com
- DOMAIN-SUFFIX,imgur.com,狗狗加速.com
- DOMAIN-SUFFIX,imore.com,狗狗加速.com
- DOMAIN-SUFFIX,instapaper.com,狗狗加速.com
- DOMAIN-SUFFIX,ipn.li,狗狗加速.com
- DOMAIN-SUFFIX,is.gd,狗狗加速.com
- DOMAIN-SUFFIX,issuu.com,狗狗加速.com
- DOMAIN-SUFFIX,itgonglun.com,狗狗加速.com
- DOMAIN-SUFFIX,itun.es,狗狗加速.com
- DOMAIN-SUFFIX,ixquick.com,狗狗加速.com
- DOMAIN-SUFFIX,j.mp,狗狗加速.com
- DOMAIN-SUFFIX,js.revsci.net,狗狗加速.com
- DOMAIN-SUFFIX,jshint.com,狗狗加速.com
- DOMAIN-SUFFIX,jtvnw.net,狗狗加速.com
- DOMAIN-SUFFIX,justgetflux.com,狗狗加速.com
- DOMAIN-SUFFIX,kat.cr,狗狗加速.com
- DOMAIN-SUFFIX,klip.me,狗狗加速.com
- DOMAIN-SUFFIX,libsyn.com,狗狗加速.com
- DOMAIN-SUFFIX,line-apps.com,狗狗加速.com
- DOMAIN-SUFFIX,linode.com,狗狗加速.com
- DOMAIN-SUFFIX,lithium.com,狗狗加速.com
- DOMAIN-SUFFIX,littlehj.com,狗狗加速.com
- DOMAIN-SUFFIX,live.com,狗狗加速.com
- DOMAIN-SUFFIX,live.net,狗狗加速.com
- DOMAIN-SUFFIX,livefilestore.com,狗狗加速.com
- DOMAIN-SUFFIX,llnwd.net,狗狗加速.com
- DOMAIN-SUFFIX,macid.co,狗狗加速.com
- DOMAIN-SUFFIX,macromedia.com,狗狗加速.com
- DOMAIN-SUFFIX,macrumors.com,狗狗加速.com
- DOMAIN-SUFFIX,mashable.com,狗狗加速.com
- DOMAIN-SUFFIX,mathjax.org,狗狗加速.com
- DOMAIN-SUFFIX,medium.com,狗狗加速.com
- DOMAIN-SUFFIX,mega.co.nz,狗狗加速.com
- DOMAIN-SUFFIX,mega.nz,狗狗加速.com
- DOMAIN-SUFFIX,megaupload.com,狗狗加速.com
- DOMAIN-SUFFIX,microsofttranslator.com,狗狗加速.com
- DOMAIN-SUFFIX,mindnode.com,狗狗加速.com
- DOMAIN-SUFFIX,mobile01.com,狗狗加速.com
- DOMAIN-SUFFIX,modmyi.com,狗狗加速.com
- DOMAIN-SUFFIX,msedge.net,狗狗加速.com
- DOMAIN-SUFFIX,myfontastic.com,狗狗加速.com
- DOMAIN-SUFFIX,name.com,狗狗加速.com
- DOMAIN-SUFFIX,nextmedia.com,狗狗加速.com
- DOMAIN-SUFFIX,nsstatic.net,狗狗加速.com
- DOMAIN-SUFFIX,nssurge.com,狗狗加速.com
- DOMAIN-SUFFIX,nyt.com,狗狗加速.com
- DOMAIN-SUFFIX,nytimes.com,狗狗加速.com
- DOMAIN-SUFFIX,omnigroup.com,狗狗加速.com
- DOMAIN-SUFFIX,onedrive.com,狗狗加速.com
- DOMAIN-SUFFIX,onenote.com,狗狗加速.com
- DOMAIN-SUFFIX,ooyala.com,狗狗加速.com
- DOMAIN-SUFFIX,openvpn.net,狗狗加速.com
- DOMAIN-SUFFIX,openwrt.org,狗狗加速.com
- DOMAIN-SUFFIX,orkut.com,狗狗加速.com
- DOMAIN-SUFFIX,osxdaily.com,狗狗加速.com
- DOMAIN-SUFFIX,outlook.com,狗狗加速.com
- DOMAIN-SUFFIX,ow.ly,狗狗加速.com
- DOMAIN-SUFFIX,paddleapi.com,狗狗加速.com
- DOMAIN-SUFFIX,parallels.com,狗狗加速.com
- DOMAIN-SUFFIX,parse.com,狗狗加速.com
- DOMAIN-SUFFIX,pdfexpert.com,狗狗加速.com
- DOMAIN-SUFFIX,periscope.tv,狗狗加速.com
- DOMAIN-SUFFIX,pinboard.in,狗狗加速.com
- DOMAIN-SUFFIX,pinterest.com,狗狗加速.com
- DOMAIN-SUFFIX,pixelmator.com,狗狗加速.com
- DOMAIN-SUFFIX,pixiv.net,狗狗加速.com
- DOMAIN-SUFFIX,playpcesor.com,狗狗加速.com
- DOMAIN-SUFFIX,playstation.com,狗狗加速.com
- DOMAIN-SUFFIX,playstation.com.hk,狗狗加速.com
- DOMAIN-SUFFIX,playstation.net,狗狗加速.com
- DOMAIN-SUFFIX,playstationnetwork.com,狗狗加速.com
- DOMAIN-SUFFIX,pushwoosh.com,狗狗加速.com
- DOMAIN-SUFFIX,rime.im,狗狗加速.com
- DOMAIN-SUFFIX,servebom.com,狗狗加速.com
- DOMAIN-SUFFIX,sfx.ms,狗狗加速.com
- DOMAIN-SUFFIX,shadowsocks.org,狗狗加速.com
- DOMAIN-SUFFIX,sharethis.com,狗狗加速.com
- DOMAIN-SUFFIX,shazam.com,狗狗加速.com
- DOMAIN-SUFFIX,skype.com,狗狗加速.com
- DOMAIN-SUFFIX,smartdns狗狗加速.com.com,狗狗加速.com
- DOMAIN-SUFFIX,smartmailcloud.com,狗狗加速.com
- DOMAIN-SUFFIX,sndcdn.com,狗狗加速.com
- DOMAIN-SUFFIX,sony.com,狗狗加速.com
- DOMAIN-SUFFIX,soundcloud.com,狗狗加速.com
- DOMAIN-SUFFIX,sourceforge.net,狗狗加速.com
- DOMAIN-SUFFIX,spotify.com,狗狗加速.com
- DOMAIN-SUFFIX,squarespace.com,狗狗加速.com
- DOMAIN-SUFFIX,sstatic.net,狗狗加速.com
- DOMAIN-SUFFIX,st.luluku.pw,狗狗加速.com
- DOMAIN-SUFFIX,stackoverflow.com,狗狗加速.com
- DOMAIN-SUFFIX,startpage.com,狗狗加速.com
- DOMAIN-SUFFIX,staticflickr.com,狗狗加速.com
- DOMAIN-SUFFIX,steamcommunity.com,狗狗加速.com
- DOMAIN-SUFFIX,symauth.com,狗狗加速.com
- DOMAIN-SUFFIX,symcb.com,狗狗加速.com
- DOMAIN-SUFFIX,symcd.com,狗狗加速.com
- DOMAIN-SUFFIX,tapbots.com,狗狗加速.com
- DOMAIN-SUFFIX,tapbots.net,狗狗加速.com
- DOMAIN-SUFFIX,tdesktop.com,狗狗加速.com
- DOMAIN-SUFFIX,techcrunch.com,狗狗加速.com
- DOMAIN-SUFFIX,techsmith.com,狗狗加速.com
- DOMAIN-SUFFIX,thepiratebay.org,狗狗加速.com
- DOMAIN-SUFFIX,theverge.com,狗狗加速.com
- DOMAIN-SUFFIX,time.com,狗狗加速.com
- DOMAIN-SUFFIX,timeinc.net,狗狗加速.com
- DOMAIN-SUFFIX,tiny.cc,狗狗加速.com
- DOMAIN-SUFFIX,tinypic.com,狗狗加速.com
- DOMAIN-SUFFIX,tmblr.co,狗狗加速.com
- DOMAIN-SUFFIX,todoist.com,狗狗加速.com
- DOMAIN-SUFFIX,trello.com,狗狗加速.com
- DOMAIN-SUFFIX,trustasiassl.com,狗狗加速.com
- DOMAIN-SUFFIX,tumblr.co,狗狗加速.com
- DOMAIN-SUFFIX,tumblr.com,狗狗加速.com
- DOMAIN-SUFFIX,tweetdeck.com,狗狗加速.com
- DOMAIN-SUFFIX,tweetmarker.net,狗狗加速.com
- DOMAIN-SUFFIX,twitch.tv,狗狗加速.com
- DOMAIN-SUFFIX,txmblr.com,狗狗加速.com
- DOMAIN-SUFFIX,typekit.net,狗狗加速.com
- DOMAIN-SUFFIX,ubertags.com,狗狗加速.com
- DOMAIN-SUFFIX,ublock.org,狗狗加速.com
- DOMAIN-SUFFIX,ubnt.com,狗狗加速.com
- DOMAIN-SUFFIX,ulyssesapp.com,狗狗加速.com
- DOMAIN-SUFFIX,urchin.com,狗狗加速.com
- DOMAIN-SUFFIX,usertrust.com,狗狗加速.com
- DOMAIN-SUFFIX,v.gd,狗狗加速.com
- DOMAIN-SUFFIX,v2ex.com,狗狗加速.com
- DOMAIN-SUFFIX,vimeo.com,狗狗加速.com
- DOMAIN-SUFFIX,vimeocdn.com,狗狗加速.com
- DOMAIN-SUFFIX,vine.co,狗狗加速.com
- DOMAIN-SUFFIX,vivaldi.com,狗狗加速.com
- DOMAIN-SUFFIX,vox-cdn.com,狗狗加速.com
- DOMAIN-SUFFIX,vsco.co,狗狗加速.com
- DOMAIN-SUFFIX,vultr.com,狗狗加速.com
- DOMAIN-SUFFIX,w.org,狗狗加速.com
- DOMAIN-SUFFIX,w3schools.com,狗狗加速.com
- DOMAIN-SUFFIX,webtype.com,狗狗加速.com
- DOMAIN-SUFFIX,wikiwand.com,狗狗加速.com
- DOMAIN-SUFFIX,wikileaks.org,狗狗加速.com
- DOMAIN-SUFFIX,wikimedia.org,狗狗加速.com
- DOMAIN-SUFFIX,wikipedia.com,狗狗加速.com
- DOMAIN-SUFFIX,wikipedia.org,狗狗加速.com
- DOMAIN-SUFFIX,windows.com,狗狗加速.com
- DOMAIN-SUFFIX,windows.net,狗狗加速.com
- DOMAIN-SUFFIX,wire.com,狗狗加速.com
- DOMAIN-SUFFIX,wordpress.com,狗狗加速.com
- DOMAIN-SUFFIX,workflowy.com,狗狗加速.com
- DOMAIN-SUFFIX,wp.com,狗狗加速.com
- DOMAIN-SUFFIX,wsj.com,狗狗加速.com
- DOMAIN-SUFFIX,wsj.net,狗狗加速.com
- DOMAIN-SUFFIX,xda-developers.com,狗狗加速.com
- DOMAIN-SUFFIX,xeeno.com,狗狗加速.com
- DOMAIN-SUFFIX,xiti.com,狗狗加速.com
- DOMAIN-SUFFIX,yahoo.com,狗狗加速.com
- DOMAIN-SUFFIX,yimg.com,狗狗加速.com
- DOMAIN-SUFFIX,ying.com,狗狗加速.com
- DOMAIN-SUFFIX,yoyo.org,狗狗加速.com
- DOMAIN-SUFFIX,ytimg.com,狗狗加速.com
- DOMAIN-SUFFIX,signal.org,狗狗加速.com
- DOMAIN-SUFFIX,whispersystems.org,狗狗加速.com
- DOMAIN-SUFFIX,cangku.moe,狗狗加速.com
- DOMAIN-SUFFIX,tradingview.com,狗狗加速.com
- DOMAIN-SUFFIX,x.com,狗狗加速.com
- DOMAIN-SUFFIX,asmr.one,狗狗加速.com
- DOMAIN-SUFFIX,telegra.ph,狗狗加速.com
- DOMAIN-SUFFIX,telegram.org,狗狗加速.com
- DOMAIN-SUFFIX,t.me,狗狗加速.com
- DOMAIN-SUFFIX,telegram.me,狗狗加速.com
- IP-CIDR,91.108.4.0/22,狗狗加速.com,no-resolve
- IP-CIDR,91.108.8.0/21,狗狗加速.com,no-resolve
- IP-CIDR,91.108.16.0/22,狗狗加速.com,no-resolve
- IP-CIDR,91.108.56.0/22,狗狗加速.com,no-resolve
- IP-CIDR,149.154.160.0/20,狗狗加速.com,no-resolve
- IP-CIDR6,2001:67c:4e8::/48,狗狗加速.com,no-resolve
- IP-CIDR6,2001:b28:f23d::/48,狗狗加速.com,no-resolve
- IP-CIDR6,2001:b28:f23f::/48,狗狗加速.com,no-resolve
- IP-CIDR,120.232.181.162/32,狗狗加速.com,no-resolve
- IP-CIDR,120.241.147.226/32,狗狗加速.com,no-resolve
- IP-CIDR,120.253.253.226/32,狗狗加速.com,no-resolve
- IP-CIDR,120.253.255.162/32,狗狗加速.com,no-resolve
- IP-CIDR,120.253.255.34/32,狗狗加速.com,no-resolve
- IP-CIDR,120.253.255.98/32,狗狗加速.com,no-resolve
- IP-CIDR,180.163.150.162/32,狗狗加速.com,no-resolve
- IP-CIDR,180.163.150.34/32,狗狗加速.com,no-resolve
- IP-CIDR,180.163.151.162/32,狗狗加速.com,no-resolve
- IP-CIDR,180.163.151.34/32,狗狗加速.com,no-resolve
- IP-CIDR,203.208.39.0/24,狗狗加速.com,no-resolve
- IP-CIDR,203.208.40.0/24,狗狗加速.com,no-resolve
- IP-CIDR,203.208.41.0/24,狗狗加速.com,no-resolve
- IP-CIDR,203.208.43.0/24,狗狗加速.com,no-resolve
- IP-CIDR,203.208.50.0/24,狗狗加速.com,no-resolve
- IP-CIDR,220.181.174.162/32,狗狗加速.com,no-resolve
- IP-CIDR,220.181.174.226/32,狗狗加速.com,no-resolve
- IP-CIDR,220.181.174.34/32,狗狗加速.com,no-resolve
- IP-CIDR,24.199.123.28/32,🔥ChatGPT,no-resolve
- IP-CIDR,64.23.132.171/32,🔥ChatGPT,no-resolve
- IP-CIDR,24.199.123.28/32,🔥ChatGPT,no-resolve
- IP-CIDR,64.23.132.171/32,🔥ChatGPT,no-resolve
- DOMAIN,injections.adguard.org,DIRECT
- DOMAIN,local.adguard.org,DIRECT
- DOMAIN-SUFFIX,local,DIRECT
- IP-CIDR,127.0.0.0/8,DIRECT
- IP-CIDR,172.16.0.0/12,DIRECT
- IP-CIDR,192.168.0.0/16,DIRECT
- IP-CIDR,10.0.0.0/8,DIRECT
- IP-CIDR,17.0.0.0/8,DIRECT
- IP-CIDR,100.64.0.0/10,DIRECT
- IP-CIDR,224.0.0.0/4,DIRECT
- IP-CIDR6,fe80::/10,DIRECT
- GEOIP,CN,DIRECT
- MATCH,狗狗加速.com
"""

# 下载 GFWList 并 Base64 解码
response = requests.get(GFWLIST_URL)
decoded_data = base64.b64decode(response.text).decode("utf-8", errors="ignore")

# 解析 GFWList 规则并转换为 Clash 规则格式
rules = []
for line in decoded_data.split("\n"):
    line = line.strip()
    if not line or line.startswith("!") or line.startswith("[") or line.startswith("@@"):  # 忽略注释、无效行和白名单规则
        continue
    if line.startswith("||"):  # 处理 DOMAIN-SUFFIX 规则
        domain = line[2:]
        rules.append(f"DOMAIN-SUFFIX,{domain},狗狗加速.com")
    elif line.startswith("|http") or line.startswith("|https"):  # 处理 URL 规则
        match = re.search(r"\|\w+://([^/]+)", line)
        if match:
            domain = match.group(1)
            rules.append(f"DOMAIN,{domain},狗狗加速.com")
    elif line.startswith("."):  # 处理以点号开头的域名
        domain = line[1:]
        rules.append(f"DOMAIN-SUFFIX,{domain},狗狗加速.com")
    elif re.match(r"^\d+\.\d+\.\d+\.\d+(\/\d+)?$", line):  # 处理 IP 或 IP-CIDR 规则
        try:
            # 如果缺少子网掩码，则添加默认的 /32
            if '/' not in line:
                line += '/32'
            ipaddress.ip_network(line, strict=False)
            rules.append(f"IP-CIDR,{line},狗狗加速.com")
        except ValueError:
            continue  # 跳过无效的 IP 或 IP-CIDR
    else:  # 处理普通域名
        rules.append(f"DOMAIN,{line},狗狗加速.com")

# 记录规则去重
rule_dict = {}
removed_rules = []
invalid_rules = []

# 处理 GFWList 规则
for rule in rules:
    parts = rule.split(",")
    if len(parts) >= 3:
        rule_type, target, policy = parts[0].strip(), parts[1].strip(), parts[2].strip()
        key = f"{rule_type},{target}"

        # 处理 IP-CIDR 规则，确保 IP 和子网掩码合法
        if rule_type in ["IP-CIDR", "IP-CIDR6"]:
            try:
                ipaddress.ip_network(target, strict=False)  # 检测 CIDR 合法性
            except ValueError:
                invalid_rules.append(rule)  # 记录无效的规则
                continue  # 跳过此规则

        if key not in rule_dict:
            rule_dict[key] = rule  # 第一次出现的规则直接保留
        elif policy != "狗狗加速.com":
            removed_rules.append(rule)  # 记录被删除的规则

# 重新整理规则
filtered_rules = list(rule_dict.values())

# 将原始规则与过滤后的规则合并
all_rules = highest_priority_rules.strip() + "\n" + "\n".join([f"- {rule}" for rule in filtered_rules])

# 写入优化后的 YAML 文件
output_file = "rules.yaml"
with open(output_file, "w", encoding="utf-8") as f:
    f.write(all_rules)

# 打印被删除的冲突规则
if removed_rules:
    print("\n=== 以下规则因冲突被删除（仅保留'狗狗加速.com'的规则） ===")
    for rule in removed_rules:
        print(rule)

# 打印无效 IP-CIDR 规则
if invalid_rules:
    print("\n=== 以下无效的 IP-CIDR 规则已被移除 ===")
    for rule in invalid_rules:
        print(rule)

print(f"\n去重完成，优化后的 YAML 文件已保存为: {output_file}")
```

程序会将处理好的规则输出到当前目录,名为`rules.yaml`

执行前确保你能打得开[GFWList](https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt),如果打不开,可以自行替换为打得开的链接

GFWList内容如下,打开如果不是这样的那就说明你找错链接了

![20250305153304](https://img.hackerbs.com//20250305153304.png)

## 规则的使用方法

程序输出的内容直接全选,然后打开Clash Verge Rev的订阅 -> 全局扩展覆写配置 -> 右键选择打开文件(如果点编辑会导致自动检查失败)

然后将程序的内容直接粘贴到文末即可

退出Clash Verge Rev,重新打开,使用TUN模式，刷新订阅，点开你的规则组，例如我的狗狗加速.com,按延迟排序,节点测速

因为规则有7361条,所以节点测速时重复点击,大概5分钟后才会测出正常速度,例如15-新加坡Hy2节点，87ms延迟

如果你和我一样用的狗狗加速,需要访问ChatGPT的话就选`🔥ChatGPT`这组

![20250305153734](https://img.hackerbs.com//20250305153734.png)

![20250305153746](https://img.hackerbs.com//20250305153746.png)

![20250305153755](https://img.hackerbs.com//20250305153755.png)
