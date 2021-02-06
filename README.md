<h1 align="center">BiliRoaming PHP Server in Docker</h1>

<div align="center">based on <a href="https://github.com/metowolf/docker-lemp" target="_blank">@metowolf/docker-lemp</a></div></br>


### 组件

|name|pulls|version|layers|image size|
|:---:|:---:|:---:|:---:|:---:|
|[metowolf/php](https://hub.docker.com/r/metowolf/php)|![Pulls Count](https://img.shields.io/docker/pulls/metowolf/php.svg?style=flat-square)|![GitHub release (latest by date)](https://img.shields.io/github/v/tag/metowolf/docker-php?style=flat-square)|![Layers](https://shields.beevelop.com/docker/image/layers/metowolf/php/latest.svg?style=flat-square)|![image size](https://shields.beevelop.com/docker/image/image-size/metowolf/php/latest.svg?style=flat-square)|
|[metowolf/nginx](https://hub.docker.com/r/metowolf/nginx)|![Pulls Count](https://img.shields.io/docker/pulls/metowolf/nginx.svg?style=flat-square)|![GitHub release (latest by date)](https://img.shields.io/github/v/tag/metowolf/docker-nginx?style=flat-square)|![](https://shields.beevelop.com/docker/image/layers/metowolf/nginx/latest.svg?style=flat-square)|![](https://shields.beevelop.com/docker/image/image-size/metowolf/nginx/latest.svg?style=flat-square)|
|[mysql/mysql-server](https://hub.docker.com/r/mysql/mysql-server)|![Pulls Count](https://img.shields.io/docker/pulls/mysql/mysql-server.svg?style=flat-square)||![](https://shields.beevelop.com/docker/image/layers/mysql/mysql-server/latest.svg?style=flat-square)|![](https://shields.beevelop.com/docker/image/image-size/mysql/mysql-server/latest.svg?style=flat-square)|
|[phpmyadmin/phpmyadmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin)|![Pulls Count](https://img.shields.io/docker/pulls/phpmyadmin/phpmyadmin.svg?style=flat-square)||![](https://shields.beevelop.com/docker/image/layers/phpmyadmin/phpmyadmin/latest.svg?style=flat-square)|![](https://shields.beevelop.com/docker/image/image-size/phpmyadmin/phpmyadmin/latest.svg?style=flat-square)|

### 要求

 - 安装 [docker](https://get.docker.com/) 和 [docker-compose](https://docs.docker.com/compose/install/)。
 - 修改安全组放通服务器相关端口（根据 `docker-compose.yml`）

### 用法

1. **克隆此仓库和子模块**
   
   ```bash
   git clone --recursive https://github.com/monsterxcn/docker-lemp-bili.git
   ```

2. **进入仓库目录下，新建 `.env` 和 `docker-compose.yml`**
   
   在 `.env` 文件第 6 行设置 MySQL 密码（root 用户）
   
   在 `docker-compose.yml` 文件第 52 行设置 PhpMyAdmin 映射端口，默认 `8080`。如果修改了奇怪的端口记得在服务器安全组设置中放通该端口，或使用 NginX 反代此端口。
   
   ```bash
   cd docker-lemp-bili
   cp .env.example .env
   cp docker-compose.example.yml docker-compose.yml
   
   nano .env
   nano docker-compose.yml
   ```

3. **配置 NginX**
   
   ```bash
   # 将 HTTPS 证书放至 `etc/nginx/ssl/` 文件夹下
   cp /path/to/server_name.crt etc/nginx/ssl/bili.crt
   cp /path/to/server_name.key etc/nginx/ssl/bili.key
   
   # 修改第 6 行 server_name 为待部署域名
   nano etc/nginx/conf/bili.conf
   ```

4. **配置 MySQL**
   
   ```bash
   # 先启动容器，再从 PhpMyAdmin 操作数据库
   docker-compose up -d
   
   # 打开 http://ip:8080/ 登录 PhpMyAdmin
   ```
   
   使用 PhpMyAdmin 新建用户、数据库导入导出就不用多说了吧。
   
   访问 PhpMyAdmin 的地址与第 2 步中设置相关，默认为 `http://ip:8080/`，若无法访问请检查服务器是否放通指定端口。

5. **配置 BiliRoaming 解析服务**
   
   [@david082321/BiliRoaming-PHP-Server](https://github.com/david082321/BiliRoaming-PHP-Server) 作为子模块放置于 `wwwroot/bili/` 目录下。请参考该项目说明配置。
   
   ```bash
   nano wwwroot/bili/config.php
   ```

6. **重新启动所有服务**
   
   ```bash
   docker-compose restart
   ```

### 高级

 - **使用自定义 HTML 页面作为网址直接访问展示**
   
   根据 [@david082321/BiliRoaming-PHP-Server](https://github.com/david082321/BiliRoaming-PHP-Server) 目前版本代码，修改 `wwwroot/bili/index.php` 第 34 行：
   
   ```php
   // exit(WELCOME);
   header("Location: /home.html");
   ```
   
   在 `wwwroot/bili/` 新建 `home.html` 文件即可将网址直接显示页面展示为其他页面。

 - **使用此项目部署其他站点**
   
   此 Docker 可用于部署其他 PHP 站点，在 `etc/nginx/conf.d/` 目录下新建其他站点配置 `.conf` 文件后重新启动 docker-compose 服务即可，或者使用命令：
   
   ```bash
   docker-compose up -d --no-deps --build
   ```
   
   注意容器内文件路径写法，具体参考现有配置文件。

 - **启用 HTTP3/QUIC**
   
   此 Docker 支持站点使用 HTTP3 QUIC 协议。注意在服务器安全组放通 UDP 443 端口，站点 `.conf` 配置文件写法参考：
   
   ```
   server {
     # Enable QUIC, HTTP/3 and HTTP/2 on IPv4.
     listen 443 ssl http2;
     listen 443 quic;
     
     ssl_certificate      cert.crt;
     ssl_certificate_key  cert.key;
     
     # Add HSTS header
     add_header Strict-Transport-Security "max-age=31536000; preload" always;
     
     # Add Alt-Svc header to negotiate HTTP/3.
     add_header alt-svc 'h3-23=":443"; ma=86400';
     
     # Enable specific TLS versions (TLSv1 and TLSv1.1 are not longer saft, TLSv1.3 is required for QUIC).
     ssl_protocols TLSv1.2 TLSv1.3;
     ssl_ciphers TLS-CHACHA20-POLY1305-SHA256:TLS-AES-256-GCM-SHA384:TLS-AES-128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256;
     ssl_prefer_server_ciphers on;
     ssl_early_data on;
   }
   ```

 - **启用 Brotli**
   
   此 Docker 支持站点使用 Brotli 压缩。站点 `.conf` 配置文件写法参考：
   
   ```
   server {

     # ...

     brotli on;
     brotli_comp_level 6;
     brotli_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript image/svg+xml;
     
     # ...
   }
   ```