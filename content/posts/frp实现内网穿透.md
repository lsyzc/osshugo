---
title: frp实现多台主机内网穿透


---



## 概述

### frp 是什么？

frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。
具体参见官方文档:https://github.com/fatedier/frp/blob/dev/README_zh.md

## frp的安装与部署

目前可以在 Github 的 [Release](https://github.com/fatedier/frp/releases) 页面中下载到最新版本的客户端和服务端二进制文件，所有文件被打包在一个压缩包中。

解压缩下载的压缩包，将其中的 frpc 拷贝到内网服务所在的机器上，将 frps 拷贝到具有公网 IP 的机器上，放置在任意目录。

### 使用systemctl来控制frps后台自启动

这个方法比较好用，很方便
`sudo vim /lib/systemd/system/frps.service`
在frps.service里写入以下内容

```shell
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target
[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/your/path/frps -c /your/path/frps.ini
[Install]
WantedBy=multi-user.target
```

启动：systemctl start frps

自启动： systemctl  enable frps

## 一台服务器穿透多台主机

frp支持一台公网服务器为多台内网主机提供内网穿透服务，可配置基于域名和基于端口的内网穿透。而只需更改客户端配置文件，服务端不用更改。

+ 通过自定义域名访问内网的web服务

  server端配置：

  ```linux
  [common]
  bind_port = 7000 //服务端监听端口，等待客户端连接
  vhost_http_port = 8080 
  ```

  ~~~linux
  修改客户端配置文件：
  A电脑
  [common]
  server_addr = x.x.x.x
  server_port = 7000  
  [web]
  type = http
  local_ip = 127.0.0.1
  local_port = 80
  custom_domains = abc.test.com
  
  B电脑的配置信息如下
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  [web2]
  type = http
  local_ip = 127.0.0.1
  local_port = 8080
  custom_domains = def.test.com
  ~~~

  将abc.test.com和def.test.com解析到ip x.x.x.x或修改hosts文件

  配置完成后可通过abc.baidu.com:8080访问A电脑的80端口服务
  通过def.baidu.com:8080访问B电脑的8080端口服务

  

+ 基于tcp实现多台客户机的内网穿透

  server端配置：

  ```linux
  [common]
  bind_port = 7000 //服务端监听端口，等待客户端连接
  ```

  客户端:

  ```linux
  A电脑：
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 22
  remote_port = 6000
  B电脑:
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 22
  remote_port = 6001
  ```

  通过x.x.x.x:6000访问A的22端口

  通过x.x.x.x:6001访问B的22端口

  更多功能参考https://frps.cn/11.html
  
  

