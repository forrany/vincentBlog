---
title: 树莓派创建热点与 node 脚本自动启动
date: 2019-03-23 15:00:10
tags:
    - Raspberry Pi
    - Node 脚本自启动
    - 创建热点
---

## 功能实现
> 文中所有说到的功能，均已配置好，无需再进行操作，本文只做记录以供之后的调试。
> 使用时：
> 1. 开机，系统会自动创建热点：`PipeRobot`
> 2. PC 或手机等终端连接热点（密码：SJTUROBOT）
> 3. 打开浏览器，输入地址：`192.168.123.251:8081`

树莓派在 PipeRobot 中作为服务器，需要具有创建 AP 热点的能力，供下位机建立连接和控制。

为此，查阅了相关资料，项目使用 create_ap 插件实现快速创建热点。

除此外，Node 脚本搭建的服务器需要能够实现开机启动，项目使用了 PM2 实现。

---
## 热点

### create_ap 创建热点

1. git clone 该项目 `git clonehttps://github.com/oblique/create_ap.git`
2. 进入文件夹`cd create_ap`
3. 编译 `sudo make install 就这样安装好了`
4. 安装依赖 `sudo apt-get install util-linux procps hostapd iproute2 iw haveged dnsmasq`
5. 此时就可以创建热点了，这里用的指令如下：

`sudo create_ap wlan0 eth0 SSID PASSWRD`
其中，SSID 是 WIFI 的名称，PASSWRD 是要设置的密码。

### 开机时自动开启热点

作为机器人控制系统一部分，需要在开机时自动创建热点。`create_ap`同样提供了这样的功能。

可以使用`Systemd service`实现后台开启热点，比如

`systemctl start create_ap` 就是开启热点，当然，我们需要对其配置文件进行编辑，开启我们需要的热点

修改/etc/create_ap.conf
`vim /etc/create_ap.conf`
把其中的两句 SSID 和 PASSHRASE 修改为自己希望的用户名和密码。
```
SSID=PipeRobot
PASSPHRASE=SJTUROBOT
```
**设置开机启动**
`systemctl enable create_ap`

至此，每次重启系统，就可以实现自动创建热点了。

## Node 脚本开机启动

因为对 Linux 脚本不是非常熟悉，Node 脚本的自动执行使用了 PM2 模块进行辅助。

首先全局安装 PM2
`sudo npm install -g pm2`

### 使用 pm2 执行 node 脚本

使用 PM2 运行脚本，首先进入脚本所在文件夹
`cd Public/PipeRobot`

调用 pm2 开启脚本
`pm2 start server.js`

然后就可以看到
```bash
[PM2] Starting /home/pi/app.js in fork_mode (1 instance)
[PM2] Done.
┌──────────┬────┬─────────┬──────┬─────┬────────┬─────────┬────────┬─────┬───────────┬──────┬──────────┐
│ App name │ id │ version │ mode │ pid │ status │ restart │ uptime │ cpu │ mem       │ user │ watching │
├──────────┼────┼─────────┼──────┼─────┼────────┼─────────┼────────┼─────┼───────────┼──────┼──────────┤
│ app      │ 0  │ N/A     │ fork │ 738 │ online │ 0       │ 0s     │ 0%  │ 21.8 MB   │ pi   │ disabled │
└──────────┴────┴─────────┴──────┴─────┴────────┴─────────┴────────┴─────┴───────────┴──────┴──────────┘
```

这时，服务器已经运行起来了，如果想要查看某个脚本的运行状态，可以使用`pm2 show id`查看。

### 开机启动
`pm2 startup`指令会生成一个开机启动的脚本

```
pm2 startup systemd
```

可以看到输入如下

```
[PM2] Init System found: systemd
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u pi --hp /home/pi
```

复制生成的脚本，并执行

```
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u pi --hp /home/p
```

这条指令创建的系统单元会在系统启动的时候开始执行，当系统启动的时候，PM2 就会从这个转储系统的恢复过来，为了创建这个转储空间，运行以下命令：
```
pm2 save
```

这条指令会存储 pm2 当前的状态（当前还在运行我们的服务器`server.js`）在转储系统中，当开机时，就会从系统中恢复。

这样就实现了 Node 脚本的开机启动

## 参考资料
1. [Run your Node.js application on a headless Raspberry Pi](https://dev.to/bogdaaamn/run-your-nodejs-application-on-a-headless-raspberry-pi-4jnn)
2. [create_ap](https://github.com/oblique/create_ap)