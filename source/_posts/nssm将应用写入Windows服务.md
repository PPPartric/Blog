---
title: nssm 将应用写入 Windows 服务
date: 2016-10-13 9:42:52
tags: 
- Python
- Node
- nssm
categories: Windows
---

## 安装 
---

[nssm](http://nssm.cc/)是一个 Windows 服务管理器，可以把应用写入服务，以达到自动重启的功能。

目前最新的版本是2.24，[下载地址](http://nssm.cc/download)。 下载完成以后解压到某个路径下，然后将 win64/win32 文件路径（D:\nssm-2.24\win64）添加的环境变量 Path。
<!-- more -->
## 使用

### Node

假设有一个最简单的 node 服务器程序

```javascript
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  res.send('Hello World');
});

app.listen(3000, function() {
  console.log('app is listening at port 3000');
});
```
需要将其写入 Windows 服务。


打开控制台工具。输入:

```shell
$ nssm install node_test
```
之后会显示出GUI界面：

输入**Path**为 node.exe安装路径，**Startup directory** 为应用文件路径，**Arguments** 为启动文件

点击 Install service

然后打开 Windows 的 Services，看到刚才安装的应用已经在 Services 里面了，并且为 “Automatic”，说明它会随着 Windows 启动而自动启动。

右击-“start”启动该服务，或者在命令行中输入

```shell
$ nssm start node_test
```

在浏览器中查看效果

在控制台输入 nssm 查看所有命令，可以看出 nssm 使用极其简单，参考[官方文档](http://nssm.cc/usage)
```shell
$ nssm install [<servicename>]
$ nssm remove [<servicename>]
$ nssm start <servicename>
$ nssm stop <servicename>
$ nssm restart <servicename>
$ nssm edit <servicename>
```

### bat

Windows 中可以使用批处理文件做一些自动化和重复性的工作，bat 文件单击即可运行。

假如还是在刚才 index.js 文件夹下面有一个 bat 文件,文件内容：

```shell
$ node index.js 1> app.log 2>&1
```

我们将 bat 文件写入 Windows Services （记得提前移除刚才写入的node_test服务）

```shell
$ nssm install node_test_bat
```
这次路径直接选择 bat 文件即可，因为它是可执行文件。

启动 bat 文件

```shell
$ nssm start node_test_bat
```
在浏览器中输入 `http://localhost:3000/` 查看效果,发现服务已经成功开启，而且这个时候在 index.js 文件夹里面发现了一个
**app.log** 文件，里面记录着 node 服务器的输入日志：

```shell
$ app is listening at port 3000
```
于是就在 nssm 监控管理 node 服务的同时，还拥有了一个简单的日志系统，是不是很方便。

### Python

将 Python 应用写入 Windows 服务也可以使用上述两种方法。

### nssm 启动 bat 服务报错

遇到 bat 文件双击可以运行，但是写入服务却不能运行的情况。

通过 Google 发现是可能是因为同时安装 Python2 和 Python3 导致的，因此改变 bat 文件为：

```shell
$ py -3 main.py
```

写入服务即可。


## 注意

注意非常不推荐把一个运行一次就结束的程序写入 Windows Services 的，比如 Node 或者 Python 脚本里面只有一句输出

```javascript
console.log('Hello');
```

```python
print('Hello')
```
使用 Windows 自带的 Services 手动启动：

使用 nssm 启动：

```shell
$ node_test: Unexpected status SERVICE_PAUSED in response to START control.
```

因为应用启动后立即结束，所以应用进入**Paused** 的状态。

但是可以把定时任务写入 Services。

如果对 Python 定时任务感兴趣，可以移步[Python定时任务的实现方式](https://lz5z.com/Python%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F/)

