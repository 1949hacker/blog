---
title: 51单片机入门-点亮LED
categories: [知识库]
date: 2023-09-25 10:14:36
keywords: 51单片机
tags:
    - 嵌入式开发
---

初次接触51单片机，记录下第一个程序，LED跑马灯，源代码如下：

<!-- more -->

```c
#include <reg52.h>//引入52系列单片机头文件

//位定义，sbit 标识符 = 地址值
sbit led1=P2^0;
sbit led2=P2^1;
sbit led3=P2^2;
sbit led4=P2^3;
sbit led5=P2^4;
sbit led6=P2^5;
sbit led7=P2^6;
sbit led8=P2^7;

//延迟函数,单位约10微秒，此处利用的是CPU处理的延时，并不准确
//11.0952MHz测得执行时间为0.000009771593，约为9.771593微秒
void delay(int us)
{
	while(us--);//循环执行，递减
}

void main()
{
	while(1)
	{
		led1=0;//设置低电平点亮LED，断点开始时间0.00042318
		delay(10000);//1秒为1000毫秒1000000微秒，此处为0.1秒，断点结束时间0.09813911
		led1=1;//设置高电平熄灭LED
		delay(10000);
		
		led2=0;
		delay(10000);
		led2=1;
		delay(10000);
		
		led3=0;
		delay(10000);
		led3=1;
		delay(10000);
		
		led4=0;
		delay(10000);
		led4=1;
		delay(10000);
		
		led5=0;
		delay(10000);
		led5=1;
		delay(10000);
		
		led6=0;
		delay(10000);
		led6=1;
		delay(10000);
		
		led7=0;
		delay(10000);
		led7=1;
		delay(10000);
		
		led8=0;
		delay(10000);
		led8=1;
		delay(10000);
	}
}
```