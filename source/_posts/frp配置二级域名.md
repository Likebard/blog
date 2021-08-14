---
title: frp+Nginx配置二级域名(http/https)
date: 2021-08-12 10:33:12
categories: DevOps
tags: [frp, nginx]
---

## 预置环境：
本地和公网服务器均安装好frpc+frps服务，公网服务器安装好Nginx。已有阿里云ECS和域名
<!--more-->
## 一、 阿里云官网设置
1. 主域名解析设置
添加一条解析记录，主机记录填*,记录值填ECS的ip地址，其他项默认。这个操作会设置，当访问主域名下所有二级域名时，都会被DNS解析到指定的公网服务器上来。
![阿里云域名解析截图](host.png)

2. 申请免费证书，并绑定到想用到的二级域名上。
![申请ssl截图](aplication_ssl.png)

申请完成后下载证书到本地，并且scp上传到服务器一份。注意因为后续要用nginx，所以下载的证书选择Nginx类型。

## 二、 frp配置
1. 远程服务器frps.ini,添加如下配置
``` markdown
[common]
vhost_http_port = 4080      # frp监听Http请求的端口
vhost_https_port = 4443     # frp监听HTTPS请求的端口
subdomain_host = test.com   # 这里填你要配置的主域名
```
对于设置的4080和4443两个端口，不要忘记去阿里云ECS控制台的安全组里配置这两个端口，不然会被墙。

2. 本地主机frpc.ini，添加如下配置
``` markdown
[web]
type = http
local_port = 3000   # 端口号写你本地web应用的端口号
subdomain = xxx     # 这里写上你的子域名，到时候在浏览器可以输入xxx.test.com来访问你的内网应用

[webs]
type = https
local_port = 3000
subdomain = xxx

plugin = https2http
plugin_local_addr = 127.0.0.1:3000

# HTTPS证书相关的配置
plugin_crt_path = /Users/likebard/program/cert.pem  # 这里使用下载到本地的公钥路径
plugin_key_path = /Users/likebard/program/cert.key  # 下载到本地的私钥路径
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp
```

3. 配置完成后，重启frpc服务，访问Client端和Server端的dashboard，如果看见的结果和以下截图一样，说明配置成功。
![frps管理界面](frps_dashboard.png)
![frpc管理界面](frpc_dashboard.png)

## 三、 ESC上Nginx对二级域名做配置
针对http请求泛域名：
``` nginx
server {
    listen 80;
    server_name *.test.com;     #这里可以使用泛二级域名，也可以像下面配置https那样，指定二级域名
    location / {
        proxy_pass http://127.0.0.1:4080;   # 这是要走80端口后要代理的端口，填上frps监听http请求的端口
        proxy_set_header    Host            $host:80;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_hide_header   X-Powered-By;

    }
}
```

针对https请求指定二级域名：
``` nginx
server {
         listen 443 ssl http2;
         server_name xxx.test.com; #填写绑定证书的域名

         ssl_certificate cert/cert.pem;
         ssl_certificate_key cert/cert.key;
         ssl_session_timeout 5m;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #使用此加密套件。
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; #使用该协议进行配置。
         ssl_prefer_server_ciphers on;
         location / {
             proxy_pass http://127.0.0.1:4080;
             proxy_ssl_server_name on;
             proxy_redirect off;

             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_set_header Host $host;
         }
 }
```

## 最终
重启nginx之后,打开Chrome，就能通过二级域名访问内网web应用了！由于Nginx里配置了proxy_pass代理，因此不需要带上端口号，可以直接访问应用。
我们来梳理一下整个工作流程： 访问https://***.test.com时，之前配置过的dns解析能确保请求到达指定的阿里云ECS上， ECS上的Nginx会根据配置的子域名，来判断应该使用哪一套ssl证书，并转发到ECS指定的端口上（在本文中是4080端口）。4080端口上有frps的监听服务,frps会根据请求的二级域名，去配置表中遍历所有挂载的二级域名，命中后就找到了内网服务器，利用NAT映射，最终访问到我们指定的内网web应用上。
