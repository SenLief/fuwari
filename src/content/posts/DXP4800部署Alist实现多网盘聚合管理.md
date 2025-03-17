---
title: DXP4800部署Alist实现多网盘聚合管理
categories:
  - 笔记
tags:
  - Blog
  - DXP4800
  - alist
  - 网盘
  - rclone
date: 2025-03-14 00:21:35
sticky: 
aging: true
aging_days: 30
home_cover: https://cdn.sa.net/2025/03/14/UVpLS3oK5DMcYWl.png
publish: "true"
category: 笔记
shortRepo: astro
published: 2025-03-17 22:57:22
path: src/content/posts
---
## 缘起
绿联系统是有网盘同步的功能的，目前已经实现了`oneDrive`和`百度云盘`的下载，但是创建同步任务是需要钱的，官方也承诺说会继续开发其他的网盘聚合，但是显而易见还是需要钱的，所以倒不如使用`alist`聚合网盘，利用已有的会员网盘，实现更多的网盘聚合管理。
## Docker Compose部署Alist
**不要使用应用中心的alist安装，因为无法更改compose.yml的配置**

### 前置需要
1. docker可以访问到docker hub，已加速或已代理
2. `compose.yml`文件默认放在`共享文件夹/docker/alist`下
3. （进阶）若想挂载网盘到本地文件中管理需要熟悉`linux`和`rclone`

### 编写`compose.yml`文件

#### 文件内容
```
services:
  alist:
    image: 'xhofe/alist:beta'
    container_name: alist
    volumes:
      - './alist:/opt/alist/data'
      - './data:/opt/alist/local'
    ports:
      - '5244:5244'
    environment:
      - PUID=1000
      - PGID=10
      - UMASK=022
    restart: unless-stopped
```
#### 解释
- `'./alist:/opt/alist/data'`：这个是默认的尽量不动
- `'./data:/opt/alist/local'`：如果你需要把网盘内容直接下载到NAS中则挂载这个卷
#### 截图
![sfg42GNkzAwjZ8W.png](https://cdn.sa.net/2025/03/14/sfg42GNkzAwjZ8W.png)
### Alist配置
#### 获取登录用户名和密码
`ssh`登录后输入`docker exec -it alist ./alist admin set NEW_PASSWORD` 替换到其中的`NEW_PASSWD`为自己的密码即可。
#### 添加网盘存储
参考官网教程：[https://alist.nn.ci/zh/guide/drivers/][https://alist.nn.ci/zh/guide/drivers/]
#### 添加本地存储
如果我们想把网盘的内容直接下载到NAS中，或者从NAS中直接上传就需要添加本地存储。
需要配置以下项：
- 存储中`驱动`选择`本机存储`
- `挂载路径`为`/local`，不重复就可以
- `根文件路径`为`/opt/alist/local`，也就是`compose.yml`文件中配置的第二个挂载卷
- 其他按需选择即可
#### 浏览器打开`http://<nas-ip>:5244`即可管理
已经可以实现网盘和本地之间的上传和下载功能。
1. 网盘到本地
   直接在网页端选择网盘文件复制/粘贴到`local`中即可
2. 本地到网盘
   需要先把上传的的文件复制到`共享文件夹/alist/data`下，然后网页端在`local`中找到这个文件复制粘贴到网盘中即可。
## 网盘挂载到本地

>实际上alist我觉得已经够用了，不过如果能像操作本地文件一样在文件管理中管理，那显得确实很酷（但不算是好用）。

#### 原理
利用`rclone`把alist的webdav挂载到本地文件夹，实现用文件管理器来管理，同时利用webui还能实现一些其他的功能。
#### **安全性**
`rclone`我使用是系统自带的`rclone-webdav`，由于挂载需要使用`root`账户运行，为了配置的方便我使用`WebUI`来控制，所以安全性有很大的问题，不建议在公网使用这种方式，配置WebUI认证的时候建议配置强密码。
#### ssh获取root权限
ssh登录后输入`sudo -i`并输入当前账户的密码即可切换到`root`账户。
#### 编写`rclone.service`文件
1. 输入`nano /etc/systemd/system/rclone.service`并粘贴以下内容：
```
[Unit]
Description=Rclone Remote Control Daemon
After=network.target

[Service]
Type=simple
Environment=RCLONE_RC_NO_OPEN_BROWSER=true
Environment=RCLONE_CONFIG=/root/.config/rclone/rclone.conf
ExecStart=/usr/sbin/rclone-webdav rcd --rc-web-gui --rc-user admin --rc-pass 123456 --rc-web-gui-update --rc-web-fetch-url=https://s3.yuudi.dev/rwa/embed/version.json --rc-addr=0.0.0.0:5572
Restart=on-failure
ExecStop=/usr/sbin/rclone-webdav rc core/quit
KillMode=none

[Install]
WantedBy=multi-user.target
```

**重要**：修改文件中的`123456`为强密码！！！
2. 启动systemd.service
   `systemctl daemon-reload`
   `systemctl start rclone.service`
#### 配置`rclone`
1. 浏览器打开`http://<nas-ip>:5572`
2. 输入`admin`和`上面设置的密码`
3. 点击左上方`存储池`，新建`存储池`
4. 搜索`webdav`下一步，注意alist需要给用户开启`webdav`的相关权限，在alist的后台账户可以打开。
5. `url`：`http://<nas-ip>:5244/dav`
6. `user`：alist账户，需要有WebDAV权限的账户
7. `pass`：用户的密码
8. 下一步填写完即可
#### 挂载到本地
1. 点击左上方的`挂载`
2. 新建挂载点
3. `存储池`：选择刚才新建的名称
4. `挂载点`：`/volume1/local/WebDav`，我选择的是`共享文件夹/local/WebDav`，请注意这个文件夹必须存在才能挂载
5. 效果图
   ![7fjLDutEOVMk1Yo.png](https://cdn.sa.net/2025/03/14/7fjLDutEOVMk1Yo.png)

## 结语
可能只是更方面把网盘搬家回来吧，但是效果不太好，主要是下载量大了就会自动断流什么的，能怎么办？充钱呗。

`rclone`挂载的方式不太建议，安全性堪忧，我比较好奇的是绿联系统居然内置了`rclone`！