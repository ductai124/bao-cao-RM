<!--
# h1
## h2
### h3
#### h4
##### h5
###### h6

*in nghiêng*

**bôi đậm**

***vừa in nghiêng vừa bôi đậm***

`inlide code`

```php

echo ("highlight code");

```

[Link test](https://viblo.asia/helps/cach-su-dung-markdown-bxjvZYnwkJZ)

![markdown](https://images.viblo.asia/518eea86-f0bd-45c9-bf38-d5cb119e947d.png)

* mục 3
* mục 2
* mục 1

1. item 1
2. item 2
3. item 3

***
horizonal rules

> text

{@youtube: https://www.youtube.com/watch?v=HndN6P9ke6U}
-->
***
# TÀI LIỆU HƯỚNG CÁC BƯỚC ĐẦU TIÊN CÀI ĐẶT VÀ CẤU HÌNH WEB SERVER BẰNG NGINX ROCKY LINUX TRÊN MÁY ẢO
***
## 1.	Cài đặt nginx
* Tắt SELinux đi đề phòng phát sinh lỗi
```php
sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux
sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config
reboot
```
* Cài đặt nginx bằng câu lệnh sau
```php
dnf -y install nginx
```
*	Cấu hình nginx như sau
```php
vi /etc/nginx/nginx.conf

 Server{
     ...
     server_name www.srv.world;
     ...
 }
 
```
* Khởi động nginx và Thiết lập tường lửa:
```php
systemctl enable --now nginx

firewall-cmd --add-port=80/tcp

firewall-cmd --add-service=http

firewall-cmd --runtime-to-permanent
```
***
## 2. Cài đặt phpmyadmin
* Cài đặt php 7.4
```php
yum module -y enable php:7.4
yum module -y install php:7.4/common
Restart nginx và php-fpm
systemctl restart nginx
systemctl status php-fpm
```
* cài đặt mariadb với câu lệnh sau
yum module -y install mariadb:10.3
```php
#Tạo 1 file mới và chỉnh sửa
vi /etc/my.cnf.d/charset.cnf

[mysqld] 
character-set-server = utf8mb4 

[client] 
default-character-set = utf8mb4

#Khởi động mariadb bằng câu lệnh

systemctl enable --now mariadb

```
* Tiến hành cài đặt ban đầu cho mariadb 
mysql_secure_installation
```php
# set root password
Set root password? [Y/n] y
New password:
Re-enter new password:

# remove anonymous users
Remove anonymous users? [Y/n] y

# disallow root login remotely
Disallow root login remotely? [Y/n] y

# remove test database
Remove test database and access to it? [Y/n] y

# reload privilege tables
Reload privilege tables now? [Y/n] y
```
* Cài đặt các gói cần thiết cho phpmyadmin
```php
# Cài đặt wget và unzip
    yum -y install wget
    yum -y install unzip
# Cài đặt các gói php
 	yum -y install php php-fpm php-gd php-mysqlnd php-cli php-curl php-zip php-common php-bcmath php-imap php-xmlrpc php-json php-mbstring php-xml php-opcache
# Khởi động lại nginx và php-fpm
    systemctl restart nginx

    systemctl start php-fpm

    systemctl enable php-fpm

```
* Cài đặt phpmyadmin
```php
cd /usr/share/nginx/html

wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.zip

unzip phpMyAdmin-5.1.1-all-languages.zip

mv phpMyAdmin-5.1.1-all-languages phpmyadmin

cd /usr/share/nginx/html/phpmyadmin

cp config.sample.inc.php config.inc.php

chown nginx:nginx config.inc.php

systemctl restart nginx

systemctl restart php-fpm

```
***
## 3. Cài đặt nukeviet
* Cài đặt nukeviet
```php
cd /usr/share/nginx/html

wget https://github.com/nukeviet/nukeviet/releases/download/4.5.01/nukeviet4.5.01setup.zip

unzip nukeviet4.5.01setup.zip

chown -R nginx:nginx /usr/share/nginx/html

chmod -R 777 /usr/share/nginx/html

systemctl restart nginx

systemctl restart php-fpm

```
***
## 4. Cài đặt modsecurity
* Chuẩn bị cài đặt các gói cần thiết và gỡ cài đặt nginx
```php
#Update lại rocky linux bằng câu lệnh
dnf upgrade --refresh -y
#Cài đặt gói Epel và remi

dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm

#Gỡ cài đặt nginx bản cũ

systemctl stop nginx

dnf remove nginx

#Tạo đoạn mã chỉ định kho lưu trữ nginx

vi /etc/yum.repos.d/nginx.repo

[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
* Cài đặt nginx phiên bản mới nhất
```php
#Cài đặt nginx phiên bản mới nhất

yum-config-manager --enable nginx-mainline

dnf install nginx

#Khởi động nginx

systemctl start nginx

systemctl enable nginx

#Kiểm tra version hiện tại

nginx -v

#Tạo và di chuyên đến thư mục sau bằng câu lệnh

mkdir /usr/local/src/nginx && cd /usr/local/src/nginx

#Tải xuống mã nguồn nginx và giải nén

wget http://nginx.org/download/nginx-1.21.4.tar.gz

tar -xvzf nginx-1.21.4.tar.gz
```



