---
layout: post
title:  利用FRP工具实现内网穿透
description: 利用FRP工具实现内网穿透
date:   2022-8-18
tags: FRP Tunnel Github
---

### 一、下载地址

- 文档：[https://gofrp.org/docs/](https://gofrp.org/docs/)
- 源码：[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)

### 二、文件说明

- Linux下载: frp_0.44.0_linux_amd64.tar.gz

- Windows下载: frp_0.44.0_windows_amd64.zip

> frps是服务端，在公网服务器上部署，frpc是客户端，在需要内网穿透的机器上部署

### 三、frps服务端配置

#### 1、编辑frps.ini

- 访问多个web端subdomain_host必须使用域名，通过二级域名访问不同的本地web端，作用类似Nginx里面的service_name

```$xslt
[common]
bind_addr = 0.0.0.0

# 公网服务器暴露的端口
bind_port = 12534

# 公网ip或者域名
subdomain_host = xxx.xxx.xxx
# 客户端连接凭证`
token =xxxxXXXXXxxxx

# 服务器看板访问端口
dashboard_port = 16343
# 服务器看板账户
dashboard_user = geek
dashboard_pwd = geek

# web端暴露的端口（避免和nginx冲突，用nginx代理转发）
vhost_http_port = 12080

# 日志
log_file = ./frps.log
log_level = error
log_max_days = 3

# 最大连接数                                                                                                                                 
max_pool_count = 5
tcp_mux = true
```

#### 2、域名解析

- 域名解析到服务器公网IP

#### 3、启动服务端

- win版

```$xslt
frps.exe -c frps.ini
```
 
- linux版

```$xslt
./frps -c ./frps.ini
```

### 四、frpc客户端配置

- 编辑frps.ini

```$xslt
[common]
# 和frps服务端一致
server_addr = xxx.xxx.xxx
server_port = 12534
# 客户端连接凭证
token =xxxxxXXXXXxxxx

[web]
type = http
local_ip = 127.0.0.1
local_port = 80
custom_domains = geek.xxx.xxx.xxx
locations = /
	
[api]
type = http
local_ip = 127.0.0.1
local_port = 10088
custom_domains = diy.xxx.xxx
locations = /

[ssh]
type = tcp
local_port = 22
local_ip = 127.0.0.1
# 在服务端注册端口，服务端将监听 17022 ssh root@xxx.xxx.xxx -p 17022 即可代理到本机 ssh 登录
remote_port = 17022

[mysql]
type = tcp
local_ip = 127.0.0.1
local_port = 3306
remote_port = 15001

[redis]
type = tcp
local_ip = 127.0.0.1
local_port = 6379
remote_port = 16002
```

#### 1、启动客户端

```$xslt
# win版
frpc.exe -c frpc.ini
```

```$xslt
# linux版
./frpc -c ./frpc.ini
```

### 五、内网穿透配置完成

- 通过域名+端口号访问对应的本地服务

### 六、服务器端后台运行及自动启动

- Linux系统使用systemd配置开机自启

```$xslt
# 新建frps.service文件
vi /etc/systemd/system/frps.service
```

写入以下内容：

```$xslt
[Unit]
Description=frps daemon
After=syslog.target network.target
Wants=network.target
 
[Service]
Type=simple
# 设置frps文件和配置文件目录 
ExecStart=/home/frps/frps -c /home/frps/frps.ini
Restart=always
RestartSec=1min
 
[Install]
WantedBy=multi-user.target
```

### 七、启动并设为开机自启

```$xslt
# 启动
systemctl start frps

# 状态查询
systemctl status frps

# 开机启动
systemctl enable frps
```

