---
title: 折腾路由--Padavan私有云
date: 2019-01-14 15:07:50
tags:
     - 路由器 
---

经历了网盘的各种关闭、数据迁移和限速，国内基本只有百度网盘一家独大了。然而百度网盘推出会员、进而超级会员一波骚操作后，实在让人累觉不爱。

最近正好在折腾路由器，上一篇已经把刚买的小米路由器刷成了Padavan固件，就其本身而言，已经很具可玩性了。这次把搭建私有云的过程写下来，也只是防止日后忘记而已，并没有太多的技术含量。

在实验室、家庭中，有一个内网范围的共享平台是非常方便的，这也是觉得比较使用，有必要记录和分享的原因吧，下面进入正题。

## 准备阶段

1. 已刷Padavan固件的路由器，这个已经在上一篇记录，有需要的可以参考[小米路由器刷机指南](https://forrany.github.io/2018/12/20/%E5%B0%8F%E7%B1%B3%E8%B7%AF%E7%94%B1%E5%99%A8R3G%E5%88%B7%E6%9C%BA%E6%8C%87%E5%8D%97/)
2. 移动硬盘

## KodExplorer可道云

KodExplorer可道云和智能路由器真的是绝配，刷Padavan是因为固件本身已经继承了KodExplorer，以及很多其他工具。

这里是KodExplorer的一个在线Demo，有桌面和文件夹两种管理模式，非常Nice。
[可道云在线Demo](http://demo.kodcloud.com/)

## 配置可道云

登陆路由器管理页面，默认地址:192.168.123.1,账号密码:admin
![](http://ww1.sinaimg.cn/large/6f9f3683ly1fz650q87odj20gf0dtjtu.jpg)

固件中已经集成可道云，点击左侧搭建Web环境

![](http://ww1.sinaimg.cn/large/6f9f3683ly1fz651mknbuj208b0jqt98.jpg)

点入以后，按照以下显示操作。按如下操作就可以打开WEB服务器功能和可道云，因为可道云不使用数据库，所以还是很方便的，这个时候就可以直接通过IP+端口的方式访问可道云了，因为集成的是早些版本的可道云，所以建议在升级以后使用。
![](http://ww1.sinaimg.cn/large/6f9f3683ly1fz6521tm3dj20ds0csjua.jpg)

通过以上的步骤，已经可以实现内网的可道云了，只需要IP地址加端口号即可访问。

当然，如果想要外网访问，还需要做一下其他工作,主要有4中方法：

1、方法一：跟电信商要一个公网的IP在路由器中开启端口映射功能
2、方法二：注册花生壳免费账号，通过绑定花生壳来做访问
3、方法三：ngrok内网转发等方式来实现访问
4、更多方法：百度搜索“内网穿透”

这里就在暂时不讨论了。

最后，简单上一下效果吧：
![](http://ww1.sinaimg.cn/large/6f9f3683ly1fz6541js5uj20yo0khasc.jpg)

![](http://ww1.sinaimg.cn/large/6f9f3683ly1fz654abilwj20yu0f742d.jpg)

## opt挂载空间占用100%问题

在使用kod云进行大文件传输的时候，会遇到资源空间用完的问题，这是非常坑的，提示如下
```bash  
【LNMP】: /opt 已用节点空间100%/100%
```
为了解决这个问题，需要将opt挂载到U盘。

这里主要涉及两个点：
1. ext4格式U盘
2. 挂载opt

一般U盘不是ext4格式的，Windows格式化ext4需要一些软件，其实可以在Linux进行格式化，这里介绍对方法进行总结。

### 如何在路由器上格式化 U 盘为 ext4

**一、安装fdisk**
一般梅林固件都会自带的，不用安装
```bash
$ opkg update
$ opkg install fdisk
# 输出Configuring fdisk. 并且没有错误
# fdisk就安装好了
```

**二、查看设备**
```bash
$ fdisk -l 
# 这里先输出系统分区之类的不用管，外置设备一般在最后
Disk /dev/sda: 30.7 GB, 30752000000 bytes
64 heads, 32 sectors/track, 29327 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Device Boot      Start         End      Blocks  Id System
/dev/sda1               2       29327    30029824  83 Linux
```
上面的信息注意看到和你的存储大小一样的设备，我的是/dev/sda，在它里面有个/dev/sda1的分区

**三、删除分区、新建分区**
```bash
$ fdisk /dev/sda # 这是你的设备別打成分区

Welcome to fdisk (util-linux 2.29.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d # 输入d回车，我只有一个分区，它自动选择了，如果你有多个分区，可以多次使用d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): n # 输入n会车，创建分区
Partition type
p   primary (0 primary, 0 extended, 4 free)
e   extended (container for logical partitions)

Select (default p): p # 选择p
Partition number (1-4, default 1): # 回车
First sector (2048-2065023, default 2048): #回车
Last sector, +sectors or +size{K,M,G,T,P} (2048-2065023, default 2065023): # 回车
Created a new partition 1 of type 'Linux' and of size 1007.3 MiB.

Command (m for help): w # 输入w回车，保存并退出
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
经过以上的操作，你可以用fdisk -l命令查看U盘上是否只有一个Linux分区

```bash
$ fdisk -l 
# 找到你的设备 可以看到ID为83就对了
Disk /dev/sda: 30.7 GB, 30752000000 bytes
64 heads, 32 sectors/track, 29327 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Device Boot      Start         End      Blocks  Id System
/dev/sda1               2       29327    30029824  83 Linux
```

**四、格式化分区**
分区已经有了，现在开始格式化，其实现在的分区已经是ext4格式的了，不过我们还是对它进行一下格式化，算是熟悉一下命令，以后直接这样格式化吧
```bash

$ mkfs.ext4 /dev/sda1 
# 如果你的硬盘比较大，256G以上的话，是这个命令：mkfs.ext4 -T largefile /dev/sda1
mke2fs 1.43.3 (04-Sep-2016)
/dev/sda1 contains a ext4 file system labelled 'ONMP'
last mounted on Sun Nov 12 09:21:22 2017
Proceed anyway? (y,n) y # 输入y回车

$ umount /dev/sda1 # 如果出错，可能是因为已经被挂载了，先执行这个卸载
```
这样，U盘就被格式化完了

### 修改opt挂载

当出现opt资源空间不足问题时，服务是无法正常启动的。

我们先查看下空间使用情况，两个指令，分别是:
```bash
df -h  #查看空间
df -i  #这个命令的节点空间查看
``` 

![](http://ww1.sinaimg.cn/large/6f9f3683ly1fz8uovlawtj20k907pq33.jpg)

原来的是100%，是在内存卡上的

ada1 这个是我的u盘

我吧这个opt文件挂载到u盘上

mount /dev/sda1 /opt

然后重启，就可以了

现在你继续用LNMP就 可以了

## opt下载失败/解压失败

曾经遇到这个问题，当出现时，可以手动下载来解决，过程如下。

1、首先在U盘或者SD卡的分区一上建立一个opt 目录：
例如 
```bash
mkdir /media/AiCard_01/opt -p
```
这里的目录可能和你的存储设备不同。

2、重启路由，确定你的opt目录已经正确mount了。
输入 mount ，看到有如下字样
```bash
/dev/mmcblk0p1 on /opt type ext4 (rw,noatime,data=ordered)
```
说明成功了。

3、然后手动下载 opt.tgz 文件，目前有两个下载地址：
```bash
cd /opt
wget https://bitbucket.org/hiboyhiboy ... aster/optupang7.tgz -O opt.tgz
```
或者
```bash

wget https://raw.githubusercontent.co ... aster/optupang7.tgz -O opt.tgz
```


也可以使用 curl
```bash
curl https://bitbucket.org/hiboyhiboy ... aster/optupang7.tgz -o opt.tgz -k
```
或者
```bash
curl https://raw.githubusercontent.co ... aster/optupang7.tgz -o opt.tgz -k
```
这个时候可以看到下载进度条开始慢慢跑了，是的，两个都很慢。

当下载进度条到了100%以后，再把opt 功能打开，


## 参考资料
[1.如何在路由器上格式化 U 盘为 ext4](https://github.com/xzhih/ONMP/wiki/%E5%A6%82%E4%BD%95%E5%9C%A8%E8%B7%AF%E7%94%B1%E5%99%A8%E4%B8%8A%E6%A0%BC%E5%BC%8F%E5%8C%96-U-%E7%9B%98%E4%B8%BA-ext4)
[2.【LNMP】: /opt 已用节点空间100%/100%](https://www.cnblogs.com/dbfedbf/p/7644989.html)
