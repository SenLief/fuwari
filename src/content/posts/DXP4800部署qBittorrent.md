---
title: DXP4800部署qBittorrent
categories:
  - 笔记
tags:
  - Blog
  - 绿联
  - DXP4800
  - qBittorrent
  - pt
date: 2025-03-06 23:00:49
sticky: 
aging: true
aging_days: 30
home_cover: https://cdn.sa.net/2025/03/06/vpbDIXsywjG2Zul.png
publish: "true"
category: 笔记
published: 2025-03-17 22:57:22
shortRepo: astro
path: src/content/posts
---
>NAS不就是用来下载小姐姐的~

## 前置需求

- Docker配置过镜像加速器
- 或配置了代理可以访问Docker Hub仓库

## 使用`Docker Compose`编排项目

### 打开`Docker`,选择`项目`，`创建`。

![iQwEKV5rUu8PjYJ.png](https://cdn.sa.net/2025/03/06/iQwEKV5rUu8PjYJ.png)

- `项目名称`：`qbt`
- `存放路径`：`共享文件夹/docker/qbt` （这个路径需要在docker文件夹中新建qbt文件夹）
- `Compose配置` ：复制粘贴以下内容
```yaml
services:
  qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    environment:
      - PUID=1000 # 更改为你的用户ID
      - PGID=10 # 更改为你的组ID
      - TZ=Asia/Shanghai # 时区，例如 "Asia/Shanghai"
      - WEBUI_PORT=8080 # Web UI的端口
    volumes:
      - ./config:/config # 你的配置文件存储路径
      - ./downloads:/downloads # 下载文件存储路径
      - /volume1/docker/Plex/media/movie:/movie
    ports:
      - 8080:8080 # Web UI端口映射
      - 36881:36881 # BitTorrent TCP端口
      - 36881:36881/udp # BitTorrent UDP端口
    restart: unless-stopped
```
- 立即部署
### `compose.yml`文件自定义配置
其他配置不要动，主要修改2处配置即可
1. `volumes`
	1. `./config:/config`：不要修改
	2. `./downloads:/downloads`：这是默认的下载路径，可以正常挂载。
	3. `/volume1/docker/Plex/media/movie:/movie`：这是自定义挂载的位置，我已有视频的文件`/volume1/docker/Plex/media/movie`，后续的`pt`下载也希望下载到这个文件夹中，那么就可以把这个文件夹挂载到容器中的`/movie`
2. `ports`：可以默认不修改
	1. `8080:8080`：Web UI的端口，默认使用`http://<nas_ip>:8080`访问
	2. `36881:36881`：可以不改，不要使用`6881`端口，大部分站点不通，如果有防火墙记得放行这个端口的`tcp/udp`协议。
## qBittorrent WebUI配置
### 配置`qBittorrent.conf`
默认的打开`http://<nas_ip>:8080`会显示`Unauthorized`，我们需要配置一下配置文件。
1. Docker项目中停止容器（必须）
2. 打开`共享文件夹/docker/qbt/config/qBittorrent/qBittorrent.conf`文件
3. 在`[Preferences]`项下添加以下两项，添加后类似于
```
[Preferences]
WebUI\CSRFProtection=false
WebUI\HostHeaderValidation=false
```
![KrfJAv6yS1Ge8Ru.png](https://cdn.sa.net/2025/03/06/KrfJAv6yS1Ge8Ru.png)
4. 保存后重启容器
5. 访问`http://<nas_ip>:8080`
### 查看账号和密码
新版本的账号和密码并不是以前的固定密码了，需要启动后去`日志`中查看随机生成的密码，在`项目`-`qbt`-`日志`中就能看到随机的密码。
- `账号`：`admin`
- `密码`：日志中的随机密码

## `qBittorrent`基础配置

1. `行为`设置
   ![1fIniQqrseKwDv2.png](https://cdn.sa.net/2025/03/06/1fIniQqrseKwDv2.png)
2. `下载`设置
   ![qpJrhaReGPiH2bc.png](https://cdn.sa.net/2025/03/06/qpJrhaReGPiH2bc.png)
3. `连接`设置
   ![sM3JiQLRw2SvtW5.png](https://cdn.sa.net/2025/03/06/sM3JiQLRw2SvtW5.png)
4. `BitTorrent`配置
   ![sM3JiQLRw2SvtW5.png](https://cdn.sa.net/2025/03/06/sM3JiQLRw2SvtW5.png)
5. `WebUI`配置
   ![lydA5IPJtMGiW1o.png](https://cdn.sa.net/2025/03/06/lydA5IPJtMGiW1o.png)
6. 其他的默认即可，高级的配置自己摸索
## 更换WebUI
默认的WebUI在移动端是显示不齐的，这对在移动端管理添加了很多的麻烦，虽然也有一些app，但是本着无必要无增加实体的原则，还是换一套UI就可以了。
### VueTorrent

项目地址: https://github.com/VueTorrent/VueTorrent
- 桌面端Light
  ![gIK6Rtq7Cxo5lNB.png](https://cdn.sa.net/2025/03/11/gIK6Rtq7Cxo5lNB.png)
- 桌面端Dark
  ![yF4zYkPWMJAnXur.png](https://cdn.sa.net/2025/03/11/yF4zYkPWMJAnXur.png)
- 移动端
  ![ytNuIHSPp3nAZxD.png|300](https://cdn.sa.net/2025/03/11/ytNuIHSPp3nAZxD.png)
### 安装

1. 下载zip包，当前版本`v2.23.0`，[V2.23.0版本下载][https://github.com/VueTorrent/VueTorrent/releases/download/v2.23.0/vuetorrent.zip]
2. 解压zip包，并把解压后的`vuetorrent`文件夹上传到NAS的`qBittrent`容器的`config`目录中。
3. 启用备用UI，填入路径`/config/vuetorrent`，刷新UI即可。
   ![pc5wIHAyLeojBbE.png](https://cdn.sa.net/2025/03/11/pc5wIHAyLeojBbE.png)
## Enjoy
享受PT的乐趣吧，不建议用自带的下载来下载PT，很多的PT站对于客户端是有要求的，虽然自带的下载器也使用`qbt`或者`tr`来模拟，但以防万一，尽量不用。