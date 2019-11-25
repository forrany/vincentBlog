---
title: Mjpg-Streamer+Node.js实现在树莓派上的监控与拍照
date: 2019-03-09 22:42:45
tags:
    - JavaScript
    - Mjpg-Streamer
    - Raspberry Pi
---

最近在做一个机器人项目，需要将试试捕获安装于机器人身上的视频图像，并能够对机器人进行无线运动控制。作为前端工程师的我，很自然的想到了使用Node作为服务器和机器人的控制中心，通过前端页面实现对机器人控制和视频图像的捕捉。

本文主要对项目中的一个单元：视频图像的捕捉和拍照功能进行开发记录和解析。

实现功能
一： 远程视频图像获取
二： 视频图像清晰度调节
三： 拍照功能

---
## 基于Express的服务器环境搭建

Express是基于Node的一个快速搭建服务器的框架，项目使用Express快速搭建服务器。
### node的安装
首先，更新所有安装列表到最新的状态：
``` bash
pi@raspberrypi:~$ sudo apt-get update
```

升级所有安装包到最新版本：

```bash
pi@raspberrypi:~$ sudo apt-get dist-upgrade
```
接下来，下载和安装node(注意版本号使用_8.x)

```bash
pi@raspberrypi:~$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
```

现在可以安装了：

```bash
pi@raspberrypi:~$ sudo apt-get install -y nodejs
```

测试是否安装成功：

```bash
pi@raspberrypi:~$ node -v
```

### Express安装

使用Node的包管理工具npm来新建项目和安装框架

首先，进入项目目录，并新建工程:
```bash
$ cd Public/WebProject/FisrtPage/
$ npm init -y
```
安装 Express 并将其保存到依赖列表中：
以下命令会将 Express 框架安装在当前目录的 node_modules 目录中
```bash
$ npm install express --save
```
然后，在该项目文件下新建`server.js`文件，引入Express就可以很方便的搭建起一个服务器。具体的内容在后面进行分析。

### Mjpg-Streamer

项目使用的是一个USB摄像头，为了能将图像捕获并通过HTTP转发，项目使用Mjpg-Streamer实现这一功能。
1. 安装必要的库
```bash
sudo apt-get update
sudo apt-get install libjpeg8-dev
sudo apt-get install imagemagick
sudo apt-get install libv4l-dev //
sudo apt-get install cmake //编译工具
```
 2. 为了向后兼容，链接videodev2.h和videodev.h
```bash
sudo ln -s /usr/include/linux/videodev2.h /usr/include/linux/videodev/h
```
注意，这里的`sudo ln -s`是非常重要的操作命令，类似于为a做一个超链接

3. git开源代码到本地，编译进入到home目录，然后开始克隆
```bash
cd ~
sudo git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
```
4. 编译和安装
```bash
sudo make
sudo make install
```
5. 测试和使用

完成以上步骤之后，可以开始测试一下。 插入摄像头，执行以下命令，分别在两个窗口打开
```bash
sudo mjpg_streamer -i "./input_uvc.so -r 640x480 -q 70 -f 15 -d /dev/video1 -n" -o "./output_http.so -p 8080 -w /usr/local/www"
```
出现一下的内容，表明安装成功
![](http://ww1.sinaimg.cn/large/6f9f3683ly1g0wxsj1795j20dw09gads.jpg)
这样，打开浏览器输入`http://localhost:8080/?action=stream`就可以看到视频图像，其中`localhost`在实际使用中，换成了树莓派的IP地址，树莓派已经提前设置了静态地址，我使用的是`192.168.123.251`，因此，视频的地址就顾定成了：

`http://192.168.123.251:8080/?action=stream`

## 依赖模块
### shelljs
上面使用的Mjpg-Streamer可以通过改变参数实现对清晰度、帧率的调整，比如将上面的图像修改为720P、15帧的指令为：
```bash
mjpg_streamer -i "input_uvc.so -r 1280x720 -f 15 -n" -o "output_http.so "
```
但是这个是在终端中执行的命令，而服务器是使用Node，因此这里使用了shelljs实现在Node运行shell指令。

首先安装shelljs
```bash
npm install shelljs -S
```
有关该模块的具体使用及相关API可以查阅[官网](https://github.com/shelljs/shelljs),本项目中主要使用了两个指令是：
- `shell.exec()` 执行某个指令
- `shell.cd()` 进入某个目录

为了在后台实现不同分辨率图像的转换，专门写一个函数来实现切换，并通过变量`videoStatus`的状态来表示不同的分辨率，与前端相对应的:
* videoStatus: 1-流畅
* videoStatus: 2-清晰
* videoStatus: 3-高清
  
清晰度切换的函数实现如下:
```JavaScript
var videoStatus = 0;  // 1-流畅； 2-清晰； 3-高清， 默认清晰
const videoCommand = [
	'mjpg_streamer -i "./input_uvc.so -r 640x480 -q 70 -f 30 -d /dev/video0 -n" -o "./output_http.so -p 8082 -w /usr/local/www"',
	'mjpg_streamer -i "./input_uvc.so -r 1280x720 -q 70 -f 15 -d /dev/video0 -n" -o "./output_http.so -p 8082 -w /usr/local/www"',
	'mjpg_streamer -i "./input_uvc.so -r 1920x1080 -q 70 -f 15 -d /dev/video0 -n" -o "./output_http.so -p 8082 -w /usr/local/www"'
]
function openVideo(qulity) {
	return new Promise((resolve,reject) => {
		if (shell.exec('pgrep mjpg_streamer').stdout !== '') {
			shell.exec('killall mjpg_streamer');
		}
		let command = videoCommand[qulity];
		shell.cd('~');
		shell.cd('mjpg-streamer/mjpg-streamer-experimental/')
		shell.exec(command, (code, std, err) => {
			console.log('Exit code:', code);
			console.log('Program output:', std);
			console.log('Program stderr:', err);
		})
		videoStatus = +qulity + 1;
		resolve('Success');
	})
}
```
对关键的几条指令进行一下说明：
* `shell.exec('pgrep mjpg_streamer').stdout !== ''`


`pgrep`以名称为依据从运行进程队列中查找进程,并显示查找到进程的id。在shelljs中，stdout是指令的输出，如果不存在进程，则返回为空； 这里加判断的意思主要**在于如果`mjpg`已经在运行，则要杀死该进程（清晰度更换通过重启`mjpg`实现）**
* `let command = videoCommand[qulity]`


具体的程序执行取决于前端的请求，根据`qulity`来开启不同清晰度的摄像头。

接下里，对设置清晰度的请求，设置服务器响应,并开启服务器:
```JavaScript
var express = require('express');
var app = express();
var bodyParse = require("body-parser");

app.post('/VideoSet',(req,res) => {
	if(+req.body.Video === videoStatus) {
		res.end('Same Video Set');
		return;
	}
	switch (req.body.Video) {
		case "1": 
			openVideo(0);
			res.end('Success');
			break;
		case "2":
			openVideo(1);
			res.end('Success');
			break;
		case "3": 
			openVideo(2);
			res.end('Success');
			break;
	}
})

var server = app.listen(8081,function(){
	console.log("Server is running at 8081 port...");
});
```

### serialport

项目除了传统前端的内容，还涉及了对机器人的控制，而机器人控制主要通过基于STM系列的下位机实现。所以RaspberryPi在作为服务器接收到前端控制请求后，需要将控制请求发送至下位机，实现控制，项目中使用了UART串口进行相互连接。

#### 打开RaspberryPi 3B的串口通讯能力
之前项目中，使用了USB转串口模块直接插在RaspberryPI的USB接口上，然后通过`serialport`打开相应的串口实现串口通讯。

本项目中，为了节约USB资源和空间，要使用GPIO口的TX/RX进行UART通讯。RaspberryPi 3B与之前的版本不同，它带了两个串口，分别是：
1. /dev/ttyAMA0：
RPI3配备了蓝牙，为了保证蓝牙的正确使用，/dev/ttyAMA0则不再为GPIO串口服务，而是为蓝牙模块服务。
2. /dev/ttyS0：
被称为"mini uart"，此串口代表了"Physical pin 8|10 BCM pin 14|15Wiring Pi pin 15|16".
但是由于次串口波特率收到cpu频率影响，并不稳定，所以实际上无法被用来串口通信。

正因如此，网络上大部分教程，直接使用/dev/ttyAMA0作为串口的方法就无法使用RPI3了，查了相关资料，通过以下方法解决[(参考自简书R4L)](https://www.jianshu.com/p/26409ddf6a9b)：
>将ttyAMA0和ttyS0互换，那么gpio tx\rx串口映射给ttyAMA0，ttyS0则映射给蓝牙设备。
这样gpio 14、15串口就拥有了稳定，强大的通信功能,而蓝牙串口则无法正常使用。

**1) 激活串口**
```bash
$ sudo nano /boot/config.txt
```
改变使得：enable_uart=1.
若无此参数，则在最后一行添加：enable_uart=1.
重启设备。

**2)查看串口别名**
```bash
ls -l /dev
```
会发现:
lrwxrwxrwx 1 root root 7 Aug 28 07:41 serial0 -> ttyS0
lrwxrwxrwx 1 root root 5 Aug 28 07:41 serial1 -> ttyAMA0

**3)禁用/dev/ttyS0的console功能**
```bash
$ sudo systemctl stop serial-getty@ttyS0.service
$ sudo systemctl disable serial-getty@ttyS0.service
```
并且修改cmdline.txt文件
```bash
$ sudo nano /boot/cmdline.txt
```
删除“console=serial0,115200”，保存并重启

4) 交换串口
```bash
$ sudo nano /boot/config.txt
```
在最下面添加：dtoverlay=pi3-miniuart-bt
保存并重启。
此时查看串口别名则发现：
lrwxrwxrwx 1 root root           7 Aug 28 07:41 serial0 -> ttyAMA0
lrwxrwxrwx 1 root root           5 Aug 28 07:41 serial1 -> ttyS0
此时，ttyAMA0串口可以正常用于串口通信，ttyS0则无法被用于串口通信，蓝牙功能失效。

#### 使用`serialport`打开通讯
1) 安装serialport
```node
npm install serialport -S
```

2) 引入serialport，并开启串口
```node
var SerialPort = require('serialport');
const port = new SerialPort('/dev/ttyAMA0', {
	baudRate: 9600
})   //使用串口，与下位机机型通讯
```

3) 串口通讯

`serialport`的api非常简单，使用相关进行通讯即可
```node
port.write('main screen turn on', function (err) {
	if (err) {
		return console.log('Error on write: ', err.message)
	}
	console.log('message written')  //打开串口
})

// Open errors will be emitted as an error event
port.on('error', function (err) {
	console.log('Error: ', err.message)
})

// Switches the port into "flowing mode"
port.on('data', function (data) {
	console.log('Data:', data.toString())  //接收到数据，打印出来
})
```

## 拍照与保存功能

`MJPG-STREAMER`支持保存当前帧，只需要将视频画面地址`http://192.168.123.251:8082/?action=stream`中的最后一个`stream`改为`snapshot`即可。

一开始初步的想法是完全同通前端实现，通过`<img src="http://192.168.123.251:8080/?action=action" />`标签来实现拍照功能，但是这种放有两个问题：
1. 所见非所得，假如在t0时刻拍照为img1，接着点击保存到本地的时候，下载和保存的图片是t1时刻的另一张照片，这是不满足需求的；
2. 图片下载功能通过`<a>`标签＋`download`属性实现，但是chrome浏览器对与跨域的图像无法实现保存，只能在新页面打开。

 因此拍照与保存功能设计成如下的流程：
![](http://ww1.sinaimg.cn/mw690/6f9f3683ly1g1707sek53j20av0bt0sx.jpg)

### 服务器端配置
**1） 获取图片地址**

服务器端要实现保存图片到本地，首先需要获取图片的地址。图片地址为`http://IP:PORT/?action=action`

项目中，将视频画面的地址端口设置为8082，即`PORT=8082`，IP地址则是RaspberryPi本机的地址，在NODE中获取本机地址的方法如下：
```node
function getIPAdress() {
	var interfaces = require('os').networkInterfaces();
	for (var devName in interfaces) {
		var iface = interfaces[devName];
		for (var i = 0; i < iface.length; i++) {
			var alias = iface[i];
			if (alias.family === 'IPv4' && alias.address !== '127.0.0.1' && !alias.internal) {
				return alias.address;
			}
		}
	}
} 
```

**2） 下载图片**

图片下载，使用到了`request`这个模块，首先在项目中安装该模块
```node
npm install request -S
```

接下来，写一个下载图片的函数，创建一个文件`downIMG.js`
```bash
$ vim downIMG.js
```

写下载图片的函数，并将函数导出
```node
const fs = require('fs');
const request = require('request');

const download = function(uri, filename, callback) {
    request.head(uri, function(err, res, body) {
        request(uri).pipe(fs.createWriteStream(__dirname +'/public/SnapShoot/' + filename)).on('close', callback);
    });
};

module.exports = download
```
在项目所在文件夹下，新建一个`SnapShoot`的文件夹。

**3） 引入图片下载函数，服务器实现响应**

在主文件`server.js`中，实现服务器的响应
```node
var download = require('./downIMG');

app.use(express.static(path.join(__dirname, 'public'))); //将public设置为静态资源，这样保存的截图才可以被访问得到
app.use(bodyParse.json({ limit: '1mb' }));  //body-parser 解析json格式数据
app.use(bodyParse.urlencoded({extended:false}));

app.get('/capture', function (req, res) {
	download(snapAddress,'current.png',function(){
		res.send('Capture Sussess');
	})
})   //拍照请求
```
至此，前端只需要通过`<a>`标签配合`download`属性，就可以实现拍照和下载的功能了，样例：
```html
<a target="_blank" id="down" :href="snapAdd" :download="currentDate">点击下载</a>
```

## 完整代码

服务器端包括了 server.js + downIMG.js，以及前端的页面及静态资源。

* downIMG.js

```node
var fs = require('fs'),
    request = require('request');

var download = function(uri, filename, callback) {
    request.head(uri, function(err, res, body) {
        request(uri).pipe(fs.createWriteStream(__dirname +'/public/SnapShoot/' + filename)).on('close', callback);
    });
};

module.exports = download
```

* server.js

```node
const express = require('express');
const app = express();
const bodyParse = require("body-parser");
const shell = require('shelljs');
const download = require('./downIMG');
const path = require('path');
const SerialPort = require('serialport');
const IP = getIPAdress();
const PORT = '8082';
const videoCommand = [
	'mjpg_streamer -i "./input_uvc.so -r 640x480 -q 70 -f 30 -d /dev/video0 -n" -o "./output_http.so -p 8082 -w /usr/local/www"',
	'mjpg_streamer -i "./input_uvc.so -r 1280x720 -q 70 -f 15 -d /dev/video0 -n" -o "./output_http.so -p 8082 -w /usr/local/www"',
	'mjpg_streamer -i "./input_uvc.so -r 1920x1080 -q 70 -f 15 -d /dev/video0 -n" -o "./output_http.so -p 8082 -w /usr/local/www"'
]

const snapAddress = `http://${IP}:${PORT}/?action=snapshot`;

var speed = 50; //运动速度
var videoStatus = 0;  // 1-流畅； 2-清晰； 3-高清， 默认清晰
const port = new SerialPort('/dev/ttyAMA0', {
	baudRate: 9600
})   //使用串口，与下位机机型通讯

port.write('main screen turn on', function (err) {
	if (err) {
		return console.log('Error on write: ', err.message)
	}
	console.log('message written')
})

// Open errors will be emitted as an error event
port.on('error', function (err) {
	console.log('Error: ', err.message)
})

// Switches the port into "flowing mode"
port.on('data', function (data) {
	console.log('Data:', data.toString())
})
 const buf1 = Buffer.alloc(1,1),  //前进
	buf2 = Buffer.alloc(1,2),  //后退
	buf3 = Buffer.alloc(1,3),  //伸张
	buf4 = Buffer.alloc(1,4),  //收缩
	buf5 = Buffer.alloc(1,5);  //停止

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParse.json({ limit: '1mb' }));  //body-parser 解析json格式数据
app.use(bodyParse.urlencoded({extended:false}));
app.post('/VideoSet',(req,res) => {
	if(+req.body.Video === videoStatus) {
		res.end('Same Video Set');
		return;
	}
	switch (req.body.Video) {
		case "1": 
			openVideo(0);
			res.end('Success');
			break;
		case "2":
			openVideo(1);
			res.end('Success');
			break;
		case "3": 
			openVideo(2);
			res.end('Success');
			break;
	}
})
app.get('/capture', function (req, res) {
	download(snapAddress,'current.png',function(){
		res.send('Capture Sussess');
	})
}) 
app.get('/speed',function(req,res) {
	res.send({'speed':speed});
}) 
app.post('/speedSet',(req,res) => {
	speed = req.body.speed;
	res.end('Speed Set Success');
})

app.get('/VideoStatus', function(req, res){
	res.send(videoStatus.toString());
})

app.get('/',function(req,res){
	res.sendFile(__dirname + '/' + "index.html");
})
app.get('/up',function(req,res){
	console.log("up recieve");
	res.send('ok');
	port.write(buf1,'hex');
})
app.get('/down',function(req,res){
	console.log('down recieve');
	res.send('ok');
	port.write(buf2,'hex');
	
})
app.get('/stretch', function (req, res) {
	console.log('stretch');
	res.send('ok');
	port.write(buf3,'hex');

})
app.get('/shrink',function(req,res){
	console.log('shrink recieve');
	res.send('ok');
	port.write(buf4);
})
app.get('/stop',function(req,res){
	console.log('stop!!!');
	res.send('ok');
	port.write(buf5);
})
var server = app.listen(8081,function(){
	console.log("Server is running at 8081 port...");
});

function openVideo(qulity) {
	return new Promise((resolve,reject) => {
		if (shell.exec('pgrep mjpg_streamer').stdout !== '') {
			shell.exec('killall mjpg_streamer');
		}
		let command = videoCommand[qulity];
		shell.cd('~');
		shell.cd('mjpg-streamer/mjpg-streamer-experimental/')
		shell.exec(command, (code, std, err) => {
			console.log('Exit code:', code);
			console.log('Program output:', std);
			console.log('Program stderr:', err);
		})
		videoStatus = +qulity + 1;
		resolve('Success');
	})
}

function getIPAdress() {
	var interfaces = require('os').networkInterfaces();
	for (var devName in interfaces) {
		var iface = interfaces[devName];
		for (var i = 0; i < iface.length; i++) {
			var alias = iface[i];
			if (alias.family === 'IPv4' && alias.address !== '127.0.0.1' && !alias.internal) {
				return alias.address;
			}
		}
	}
} 
```