# 欢迎阅读 v2raytlswsdocker！

> 鸣谢 wulabing的 V2Ray Nginx+vmess+ws+tls/ http2 over tls 一键安装脚本 https://github.com/wulabing/V2Ray_ws-tls_bash_onekey
> 我这个docker建起来基本完全是按着一键部署脚本做的
> 什么BRR加速我也没有弄 一方面是不感到有什麼用 一方面是docker

### 准备工作
首先准备工作会少一点，但是申请cert和域名是不可避免的
* 准备一个域名，并将A记录添加好。
* [V2ray官方说明](https://www.v2ray.com/)，了解 TLS WebSocket 及 V2ray 相关信息
* 装好docker和docker-compose https://docs.docker.com/install/ https://docs.docker.com/compose/install/

### 申请cert
申请cert我觉得用acme烦死了，又要装依赖又要一步一步跑，我喜欢用quay.io的certbot的docker
```
docker run -it --rm -p 443:443 -p 80:80 --name certbot -v "/etc/letsencrypt:/etc/letsencrypt" -v "/var/lib/letsencrypt:/var/lib/letsencrypt" quay.io/letsencrypt/letsencrypt:latest certonly --standalone -d EXAMPLE.com
```
letsencrypt跑完在/etc/letsencrypt/archive/会有你的域名的证书, docker不吃symbolic link切记

### 弄好混淆的网站 (照抄)
```
mkdir -p /var/www/html;git clone https://github.com/wulabing/3DCEList.git /var/www/html
```

### 改好docker-compose里的相关配置 /docker_compose/docker-compose.yml
主要是volume里cert的位置，timezone你在中国就不用改了，不过可能不同linux有不同位置
timezone 可以用以下命令找到
```
readlink -f /etc/localtime
```
把EXAMPLE.com换成你的domain
```
volumes:
- ./v2ray.config.json:/etc/v2ray/config.json
- ./v2ray.nginx.conf:/etc/nginx/conf.d/v2ray.conf
- /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime
- /var/www/html:/var/www/html
- /etc/letsencrypt/archive/EXAMPLE.com:/etc/nginx/ssl_certificate
```

### 改好nginx里的相关配置 /docker_compose/v2ray.nginx.conf
/etc/nginx/ssl_certificate/是docker裡面的路徑
```
ssl_certificate       /etc/nginx/ssl_certificate/fullchain1.pem;
ssl_certificate_key   /etc/nginx/ssl_certificate/privkey1.pem;
```

一定要改 \#7, \#25和\#26的域名

\#7:
```
server_name           EXAMPLE.com;
```
\#25
```
server_name           EXAMPLE.com;
```
\#26
return 301 https://EXAMPLE.com$request_uri;

### 搞定，在/docker_compose里面跑
```
docker-compose up -d
```
