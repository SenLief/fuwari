---
title: DXP4800部署内网穿透cloudflared
category: 笔记
tags:
  - cloudflared
  - 内网穿透
  - DXP4800
  - NAS
  - Blog
  - DDNS
published: 2025-03-18 17:21:44
description: 虽然绿联有内网穿透的UGLINK，但是只能用于了绿联自己的应用，而我们自己部署的应用无法使用，所以其他的方式还是需要的。
publish: "true"
shortRepo: fuwari
---
## Cloudflared介绍

>cloudflared 是 Cloudflare 提供的一个命令行工具，用于创建与 Cloudflare 网络的安全隧道，允许您安全地将本地服务暴露到互联网，而无需打开入站端口。 它可以简化远程访问、安全连接和各种类型的开发工作流程。

简单来说，可以利用赛博电子活佛的服务来让没有公网的网络获得提供公网服务的能力，俗称内网穿透，主要的缺点就是慢，不适合内外网传输，适用于网页的管理。

## 前置要求

- 一个域名，托管于[cloudflare][https://www.cloudflare.com/]
- 一个cloudflare账号，免费注册的即可
- 信用卡用于开通Zero Trust

## Cloudflare网页配置

> 需要将域名托管于cloudflare，同时需要开通Zero Trust

1. 配置Tunnels
   ![dxiCGpR37DPB6tU.png](https://cdn.sa.net/2025/03/18/dxiCGpR37DPB6tU.png)
2. 创建隧道，点击`选择cloudflared`
3. 命名，不重复即可
4. 保存`TOKEN`
   ![hzFIiyZTjr4Oo8U.png](https://cdn.sa.net/2025/03/18/hzFIiyZTjr4Oo8U.png)
   命令中`install eyxxxxx`，token就是`ey`开头这一串保存起来，后面会用到。

## Docker Compose部署cloudflared客户端

1. docker中创建项目
2. `compose.yml`文件
```
services:
  cloudflared:
    image: cloudflare/cloudflared:latest 
    container_name: cloudflared
    environment:
      - TZ=Asia/Shanghai 
      - TUNNEL_TOKEN=<TOKEN> # 更换为上面ey开头的token
    restart: unless-stopped
    command: tunnel --no-autoupdate run
    network_mode: "host"
```
3. `compose.yml` 解释
	1. `TUNNEL_TOKEN`：上面`ey`开头的一串字符TOKEN
	2. `network_mode: "host"`: 推荐用`host`模式，这种模式下配置更简单
4. 立即部署即可

## 创建路由
1. 创建NAS访问路由: `http://localhost:9999`
   ![vNXFjzh2LP59p7l.png](https://cdn.sa.net/2025/03/18/vNXFjzh2LP59p7l.png)
   
   **类型选HTTP**，稍等片刻，访问`https://域名`即可访问NAS
2. 创建PT访问路由：`http://localhost:8092`
   ![mlgaqcPypD1GY6k.png](https://cdn.sa.net/2025/03/18/mlgaqcPypD1GY6k.png)
   访问`https://域名`即可访问pt
3. 外网访问ssh
   待续
## 后记
`cloudflared`可以作为`ddns`的补充，在ddns无法使用的时候还是可以应急的。
