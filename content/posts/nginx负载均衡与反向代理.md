---

title: "nginx负载均衡与反向代理"
tags: ["nginx"]
date: "2019-03-10"
categories: ["nginx","1"]

---



**代理**：也被叫做正向代理，是一个位于客户端和目标服务器之间的代理服务器。

**作用**：客户端将发送的请求和指定的目标服务器提交给代理服务器，然后代理服务器向目标服务器发起请求，并将获得的响应结果返回给客户端的过程。

![image-20211022141222843](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022141222843.png)

**反向代理**：对于客户端而言就是目标服务器。

**作用**：客户端向反向代理服务器发送请求后，反向代理服务器将该请求转发给内部网络上的后端服务器，并将从后端服务器上得到的响应结果返回给客户端。

![image-20211022141244290](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022141244290.png)

►负载均衡(Load Balance)是集群技术（Cluster）的一种应用。**将大量的并发请求分担到多个处理节点，从而提高并发处理能力。 由于单个处理节点的故障不影响整个服务，负载均衡集群同时也实现了高可用。**

►任何的负载均衡技术都要建立某种一对多的映射机制，一个请求的入口映射到多个处理请求的节点，不同的机制形成不同的负载均衡技术，常见的包括：

•手动选择

•DNS轮询

•IP负载均衡

•CDN



**nginx实现负载均衡**

| **配置方式** | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| 轮询方式     | 负载均衡默认设置方式，每个请求按照时间顺序逐一分配到不同的后端服务器进行处理，如果有服务器宕机，会自动剔除 |
| 权重方式     | 利用weight指定轮询的权重比率，与访问率成正比，用于后端服务器性能不均的情况 |
| ip_hash方式  | 每个请求按访问IP的hash结果分配，这样可以使每个访客固定访问一个后端服务器，可以解决Session共享的问题 |
| 第三方模块   | 第三方模块采用fair时，按照每台服务器的响应时间来分配请求，响应时间短的优先分配；若第三方模块采用url_hash时，按照访问url的hash值来分配请求 |

+ 一般轮询

```bash
server {

  listen 80;

  server_name www.test.com;

  location / {

   proxy_pass http://APP;

   }

}

\#配置负载均衡服务器组

Upstream APP {

server 172.18.0.111;

server 172.18.0.112;

}
```



+ 加权轮询

```bash
server {

  listen 80;

  server_name www.test.com;

  location / {

   proxy_pass http://APP;

   }

}

\#配置负载均衡服务器组

Upstream APP {

server 172.18.0.111 weight=1;

server 172.18.0.112 weight=3;

}
```



+ ip hash

```bash
server {

  listen 80;

  server_name www.test.com;

  location / {

   proxy_pass http://APP;

   }

}

\#配置负载均衡服务器组

Upstream APP {

ip_hash;

server 172.18.0.111;

server 172.18.0.112;

}
```




| 案例任务名称 | **配置nginx反向代理和负载均衡**                        |      |      |      |      |
| ------------ | ------------------------------------------------------ | ---- | ---- | ---- | ---- |
| 案例训练目标 | 1、  学会配置nginx反向代理  2、  使用nginx配置负载均衡 |      |      |      |      |
| 包含技能点   | 1、  配置nginx的proxy_pass  2、  配置nginx的upstream   |      |      |      |      |

![image-20211022140301763](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140301763.png)

**案例子任务一、配置nginx反向代理，使用nginx1、APP1、APP2三个容器**

  **步骤1：使用php-apache镜像启动APP1和APP2两个容器**  

1)   #docker network create --subnet=172.18.0.0/16 cluster //创建docker网络，截图 

![image-20211022140336942](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140336942.png)

2)   #docker network ls //查看宿主机上的docker网络类型种类，截图

![image-20211022140403862](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140403862.png)

3)   启动容器APP1，设定地址为172.18.0.111，截图

docker run -d --privileged --net cluster --ip 172.18.0.111 --name APP1 php-apache /usr/sbin/init

![image-20211022140433972](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140433972.png)

4)   启动容器APP2，设定地址为172.18.0.112，截图

docker run -d --privileged --net cluster --ip 172.18.0.112 --name APP2 php-apache /usr/sbin/init

![image-20211022140457288](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140457288.png)

5)   配置容器APP1，编辑首页内容为“site1”，在宿主机访问，截图

![image-20211022140517054](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140517054.png)

6)   配置容器APP1，编辑首页内容为“site2”，在宿主机访问，截图

![image-20211022140536835](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140536835.png)

 

**步骤2：使用nginx镜像启动nginx1容器，配置反向代理**

1)   启动容器nginx1，设定地址为172.18.0.11，截图

docker run -d --privileged --net cluster --ip 172.18.0.11 -p 80:80 --name nginx1 nginx  /usr/sbin/init

![image-20211022140553491](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140553491.png)

2)   在容器nginx1编辑/etc/nginx/nginx.conf文件，重新启动nginx服务，截图

\#配置两台虚拟主机

```bash
server {

  listen 80;

  server_name site1.test.com;

  location / {

   proxy_pass http://172.18.0.111;

  }

}

 

server {

  listen 80;

  server_name site2.test.com;

  location / {

   proxy_pass http://172.18.0.112;

  }
```



3)   }在主机编辑hosts文件，并使用ping命令检查，截图

![image-20211022140846682](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140846682.png)

宿主机的IP地址   site1.test.com

宿主机的IP地址   site2.test.com

宿主机的IP地址   www.test.com

4)   在主机使用浏览器访问site1.test.com，截图

![image-20211022140621936](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140621936.png)

5)   在主机使用浏览器访问site2.test.com，截图

![image-20211022140637540](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140637540.png)

 

**案例子任务二、配置nginx负载均衡，使用nginx1、APP1、APP2三个容器**

**步骤1：保持以上三个容器不变**   

步骤2：使用nginx1容器，配置nginx一般轮询负载均衡**

1)   在容器nginx1编辑/etc/nginx/nginx.conf文件，重新启动nginx服务，截图

\#配置 www.test.com虚拟主机

2)   在主机使用浏览器访问 www.test.com并不断刷新，截图

 

![image-20211022140716115](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140716115.png)

![image-20211022140734154](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140734154.png)

**步骤3：使用nginx1容器，配置nginx IP哈希轮询**

1)   在容器nginx1编辑/etc/nginx/conf.d/default.conf文件，重新启动nginx服务，截图

\#配置 www.test.com虚拟主机

```bash
server {

  listen 80;

  server_name www.test.com;

  location / {

   proxy_pass http://APP;

   }

}

\#配置负载均衡服务器组

Upstream APP {

  ip_hash;

server 172.18.0.111;

  server 172.18.0.112;

}
```



2)   在主机使用浏览器访问 www.test.com并不断刷新，截图

![image-20211022140828405](https://gitee.com/lsyzc/picture/raw/master/picture/image-20211022140828405.png)