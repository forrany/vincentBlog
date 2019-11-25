---
title: 小米路由器R3G刷机指南
date: 2018-12-20 13:37:21
tags: 
     - 黑科技 
     - 路由器 
     - 刷机
---
# 小米路由刷机指南

## 1. 开箱并刷机至开发板

为了能够获得更高权限，开箱后必须要刷开发板，这个是官方支持的，没有什么风险，步骤如下：

1. 在miwifi.com官网下载路由器对应的开发版ROM包，并将其放在U盘的根目录下，命名为miwifi.bin
2. 断开小米路由器的电源，将U盘插入路由器的USB接口
3. 按下reset按钮后重新接入电源，待指示灯变为黄色闪烁状态后松开reset键
4. 等待5~8分钟，刷机完成之后系统会自动重启并进入正常的启动状态（指示灯由黄灯常亮变为蓝灯常亮），此时，说明刷机成功完成

## 2. 开启SSH权限

与一般的OPENWRT的路由器不太相同，开启SSH权限需要小米账号以及官方文件的支持。主要有以下步骤:

- 使用小米路由APP登陆管理界面，绑定小米账号
- 到[小米路由官网](http://miwifi.com/miwifi_open.html)下载SSH工具，如下图

![](http://ww1.sinaimg.cn/large/6f9f3683ly1fxnk3n9fgnj211v0f8aeq.jpg)

- 根据官网提示，将SSH工具下载到U盘，开启SSH权限



![](http://ww1.sinaimg.cn/large/6f9f3683ly1fyd5qt2xq0j20nj0b2my6.jpg)

## 3. 刷入Breed

Breed就是一个不死UBOOT，有了它，不用担心刷机成砖的问题，因此首先刷入该文件，为了方便，将之后用的文件全部下载好

- breed 链接：https://pan.baidu.com/s/1zI0e_R6qySDj-TpjqracUw 
  提取码：n41w 
  复制这段内容后打开百度网盘手机App，操作更方便哦
- padavan固件：链接：https://pan.baidu.com/s/14DYiNJWjpN3_S6eCoRHT_Q 
  提取码：9gpj 
  复制这段内容后打开百度网盘手机App，操作更方便哦

首先刷入breed，通过WINSCP与路由连接，小米官方固件下，地址为196.168.31.1，登录名root，密码为刚才ssh工具官网中所给的密码。

![](http://ww1.sinaimg.cn/large/6f9f3683ly1fyd5rw9x4ej20go0aoad1.jpg)

登陆后，将breed.bin文件拖入根目录下的tmp文件夹，打开终端，输入指令`mtd -r write /tmp/breed.bin Bootloader`

1. 刷入后，机器会重新启动，手动设置电脑有线网卡的IP为192.168.1.3
2. 电脑与路由器WAN口连接
3. 用牙签顶住路由的reset键再开机，等到路由的灯狂闪的时候，松开reset键，
4. 电脑上在浏览器中输入192.168.1.1，就进入不死breed的控制台了

## 4. 进入Breed，备份原来的固件

![](http://ww1.sinaimg.cn/large/6f9f3683ly1fyd5sbhuirj20go09sjul.jpg)

进入Breed，选择固件备份，然后将原来的内容进行备份，有备无患。

[备份](https://pan.baidu.com/s/13GxeuCdm1S7cfpP0fuuGnA) 提取码：k2x7 

## 5. 刷入固件

刷入刚才下载的padavan固件

刷入后，路由管理地址变为: `192.168.123.1`  

管理账号和密码均为`admin`

## 6. 补充

gl-inet 6404路由的管理页面为`192.168.8.1`

