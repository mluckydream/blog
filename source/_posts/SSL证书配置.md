---
title: SSL 证书配置
date: 2019-08-03 21:52:34
layout: 
  - tags
  - categories
categories:
  - Guide
  - Linux
tags:
  - Linux
---

### 需要准备

   腾讯云证书，Nginx 服务，例： 1inc.cn

### 证书申请

 [SSL 证书管理控制台](https://console.cloud.tencent.com/ssl) 中 申请免费证书，选择 DNS验证，在 域名解析 中添加相应记录，再回到验证页面进行验证即可成功签发证书。

### 证书安装

#### 1）在 [SSL 证书管理控制台](https://console.cloud.tencent.com/ssl) 中下载并解压缩 `www.domain.com` 证书文件包到本地目录。

解压缩后，可获得相关类型的证书文件。其中包含 Nginx 文件夹和 CSR 文件：

- **文件夹名称**：Nginx
- 文件夹内容
  - `1_www.domain.com_bundle.crt` 证书文件
  - `2_www.domain.com.key` 私钥文件

#### 2）使用文件传输工具，例如 [filezilla](https://filezilla-project.org/download.php?platform=osx) 将上面两个文件传到登录的服务器上，移动到服务器 Nginx 目录下

#### 3）在 /etc/nginx/sites-available 目录下新建一个配置文件

```shell
sudo vi /etc/nginx/sites-available/1inc.cn
```

```shell
server {
        listen 443;
        ssl on;
        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name www.1inc.cn;
        ssl_certificate 1_www.1inc.cn_bundle.crt;
         ssl_certificate_key 2_www.1inc.cn.key; 
        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```



以下为官方配置文件，可参照更改

```shell
server {
     #SSL 访问端口号为 443
     ssl on; 
     #填写绑定证书的域名
     server_name www.1inc.cn; 
     #证书文件名称
     ssl_certificate 1_www.1inc.cn_bundle.crt; 
     #私钥文件名称
     ssl_certificate_key 2_www.1inc.cn.key; 
     ssl_session_timeout 5m;
     #请按照以下协议配置
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
     #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
     ssl_prefer_server_ciphers on;
     location / {
        #网站主页路径。此路径仅供参考，具体请您按照实际目录操作。
         root /var/www/html/www.1inc.cn; 
         index  index.html index.htm;
     }
 }
```

#### 4）创建链接启动服务器

```shell
sudo ln -s /etc/nginx/sites-available/1inc.cn /etc/nginx/sites-enabled/
```

#### 5）/ sites-enabled /目录中取消默认配置文件的链接：

```shell
sudo unlink /etc/nginx/sites-enabled/default
```

#### 6）检查配置文件，直到不出错

```shell
sudo nginx -t
```



#### 7）最后，重新加载 Nginx 

```shell
sudo systemctl reload nginx
```

#### 8）测试

https://www.1inc.cn



### HTTP 自动跳转 HTTPS 的安全配置（可选）

不妨试一下，上面配置若是以 HTTP 能成功登录吗？

下面将进一步配置服务器，使其自动将 HTTP 的请求重定向到 HTTPS

#### 1) 进入网站配置文件，即上面的 1inc.cn，添加如下配置

```shell
server {
listen 80;
server_name www.1inc.cn; 
return 301 https://$host$request_uri; 
}
```

#### 2）测试

http://www.1inc.cn

以上，无论你从哪里来 1inc， 不迷路！