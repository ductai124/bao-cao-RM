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
```php
#Update lại rocky linux bằng câu lệnh
dnf upgrade --refresh -y
#Cài đặt gói Epel và remi

dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

#Cài đặt wget và zip

yum -y install wget
yum -y install unzip

#Gỡ cài đặt nginx bản cũ

systemctl stop nginx

dnf remove nginx

dnf install dnf-utils -y

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
```
* Khởi động nginx và Thiết lập tường lửa:
```php
systemctl enable --now nginx

firewall-cmd --add-port=80/tcp

firewall-cmd --add-service=http

firewall-cmd --runtime-to-permanent
```
* Cấu hình lại nginx
```php
#Cấu hình file sau
vi  /etc/nginx/conf.d/default.conf

#Sửa đoạn code sau 
server {
    #listen       80;
    #server_name  localhost;

#Sửa thành

server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  www.srv.world;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

#Tìm đến đoạn 
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
#Sửa thành
    location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
    }

#Tìm đến dòng
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
#Sửa thành
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

#sau đó thoát ra và restart lại nginx
    
    systemctl restart nginx
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

```php
yum module -y install mariadb:10.3

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

```php

mysql_secure_installation

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
 	yum -y install php php-fpm php-gd php-mysqlnd php-cli php-curl php-zip php-common php-bcmath php-xmlrpc php-json php-mbstring php-xml php-opcache
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
## 3. Cài đặt nukeviet và cấu hình rewrite cho nukeviet
* Cài đặt nukeviet
```php
cd /usr/share/nginx/html

wget https://github.com/nukeviet/nukeviet/releases/download/4.5.01/nukeviet4.5.01setup.zip

unzip nukeviet4.5.01setup.zip

mv ./nukeviet/* /usr/share/nginx/html/

rm -f /usr/share/nginx/html/index.html

rm -rf nukeviet/

chown -R nginx:nginx /usr/share/nginx/html

chmod -R 777 /usr/share/nginx/html

systemctl restart nginx

systemctl restart php-fpm

#Chạy câu lệnh sau đề phòng lỗi session

chown -R nginx:nginx /var/lib/php/session
service php-fpm restart
service nginx restart

#Chỉnh sửa lại user và group

vi /etc/php-fpm.d/www.conf

#Tìm đến 2 dòng

user = apache;

group = apache;

#sửa 

user=nginx; 

group=nginx;

service php-fpm restart
service nginx restart

```
* Cấu hình rewrite
```php
chỉnh sửa file sau

vi  /etc/nginx/conf.d/default.conf

#tìm đén đoạn code sau

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
#Thêm vào phía trên đoạn code này đoạn code cấu hình sau

	if ($request_filename ~ /robots.txt$){
		rewrite ^(.*)$ /robots.php?action=$http_host break;
	}

        # Thiết lập rewrite chỉ có từ NukeViet 4.3.00
	rewrite ^/install/check\.rewrite$ /install/rewrite.php break;

	rewrite ^/(.*?)sitemap\.xml$ /index.php?nv=SitemapIndex break;
	rewrite "^/(.*?)sitemap\-([a-z]{2})\.xml$" /index.php?language=$2&nv=SitemapIndex break;
	rewrite "^/(.*?)sitemap\-([a-z]{2})\.([a-zA-Z0-9-]+)\.xml$" /index.php?language=$2&nv=$3&op=sitemap break;

	if (!-e $request_filename){
		rewrite (.*)(\/|\.html)$ /index.php;
		rewrite /(.*)tag/(.*)$ /index.php;
	}
	
	rewrite ^/seek\/q\=([^?]+)$ /index.php?nv=seek&q=$1 break;
	rewrite ^/search\/q\=([^?]+)$ /index.php?nv=news&op=search&q=$1 break;
	rewrite ^/([a-zA-Z0-9\-]+)\/search\/q\=([^?]+)$ /index.php?nv=$1&op=search&q=$2 break; 
	rewrite ^/([a-zA-Z0-9-\/]+)\/([a-zA-Z0-9-]+)$ /$1/$2/ break; 
	rewrite ^/([a-zA-Z0-9-]+)$ /$1/ break;		

	location ~ ^/admin/([a-z0-9]+)/(.*)$ {
		deny all;
	}

	location ~ ^/(config|includes|install|vendor)/(.*)$ {
		deny all;
	}

	location ~ ^/data/(cache|ip|ip6|logs)/(.*)$ {
		deny all;
	}

	location ~ ^/(assets|uploads|themes)/(.*).(php|ini|tpl|php3|php4|php5|phtml|shtml|inc|pl|py|jsp|sh|cgi)$ {
		deny all;
	}	
#Lưu lại và thoát
#Tiếp đến thiết lập file sau
vi /usr/share/nginx/html/data/config/config_global.php

#Thêm vào sau hàm if đoạn code sau 
$sys_info['supports_rewrite']='rewrite_mode_apache';

#Lưu lại và khởi động lại các dịch vụ
service php-fpm restart
service nginx restart
```
***
## 4. Cài đặt modsecurity
* Chuẩn bị cài đặt các gói cần thiết và gỡ cài đặt nginx

* Cài đặt libmodsecurity3 cho ModSecurity
```php
#Cài đặt git
dnf install git -y

#Clone lại mã nguồn
git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity /usr/local/src/ModSecurity/

cd /usr/local/src/ModSecurity/

#Cài đặt các phụ thuộc và module con

dnf install gcc-c++ flex bison yajl curl-devel zlib-devel pcre-devel autoconf automake git curl make libxml2-devel pkgconfig libtool httpd-devel redhat-rpm-config wget openssl openssl-devel nano

dnf --enablerepo=powertools install doxygen yajl-devel

dnf --enablerepo=remi install GeoIP-devel

git submodule init

git submodule update

#Xây dựng môi trường chạy cấu hình và cài đặt
./build.sh

./configure

make

make install
```
* Cài đặt ModSecurity-nginx Connector
```php

git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git /usr/local/src/ModSecurity-nginx/

cd /usr/local/src/nginx/nginx-1.21.4

./configure --with-compat --add-dynamic-module=/usr/local/src/ModSecurity-nginx

make modules

cp objs/ngx_http_modsecurity_module.so /usr/lib64/nginx/modules/
```
* Tải và cấu hình ModSecurity-nginx Connector với nginx
```php
vi /etc/nginx/nginx.conf

#Thêm vào đầu file
load_module modules/ngx_http_modsecurity_module.so;

#Thêm vào thẻ http{} đoạn code
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/modsec-config.conf;

#Tạo thư mục mới

mkdir -p /etc/nginx/modsec/

#Di chuyên file cấu hình của modsecurity đến thư mục sau

cp /usr/local/src/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf

#Tiến hành chỉnh sửa file sau

/etc/nginx/modsec/modsecurity.conf

#Tại dòng 7
SecRuleEngine On

#Tại dòng 224
SecAuditLogParts ABCEFHJKZ

#Tạo tệp sau và thêm đoạn code vào

/etc/nginx/modsec/modsec-config.conf

#Thêm vào đoạn code
Include /etc/nginx/modsec/modsecurity.conf

#Cuối cùng là sao chép file sau
cp /usr/local/src/ModSecurity/unicode.mapping /etc/nginx/modsec/

#Kiểm tra dịch vụ bằng câu lệnh
nginx -t

#Sau đó restart nginx
systemctl restart nginx

```

* Cài đặt OWASP CRS 3.3
```php
#Tải file về bằng câu lệnh
wget https://github.com/coreruleset/coreruleset/archive/refs/heads/v3.3/master.zip

#Cài đăt giải nén file tải về vào modsec
unzip /etc/nginx/master.zip -d /etc/nginx/modsec

#Tạo 1 bản sao lưu
cp /etc/nginx/modsec/coreruleset-3.3-master/crs-setup.conf.example /etc/nginx/modsec/coreruleset-3.3-master/crs-setup.conf

#Mở file sau lên và thêm vào nhưng dòng code sau để bật các quy tắc
Vi /etc/nginx/modsec/modsec-config.conf

#Thêm vào

Include /etc/nginx/modsec/coreruleset-3.3-master/crs-setup.conf
Include /etc/nginx/modsec/coreruleset-3.3-master/rules/*.conf*


```
* Kiểm tra và khởi động lại nginx

>nginx -t

>systemctl restart nginx

* Tùy chỉnh modscurity
```php
#Truy cập thư mục sau để tùy chỉnh các modsecurity
vi /etc/nginx/modsec/coreruleset-3.3-master/crs-setup.conf/

#Đường dẫn chứa các biến của rules của từng protection
/etc/nginx/modsec/coreruleset-3.3-master/rules/
```
***
## 5. Hướng dẫn thiết lập Test một số các modsecurity
* Kiểm tra chặn xss bằng cách thêm dòng script sau vào sau tên miền 
```php
/index.html?exec=/bin/bash

#Hoặc

/?q=<Script>

#ví dụ:

http://192.168.1.15//index.html?exec=/bin/bash

```
* Mở bảo vệ Dos và kiểm tra
```php
#Sửa file sau
vi /etc/nginx/modsec/coreruleset-3.4-dev/crs-setup.conf

#Tìm đến dòng

#SecAction \
# "id:900700,\
#  phase:1,\
#  nolog,\
#  pass,\
#  t:none,\
#  setvar:'tx.dos_burst_time_slice=60',\  
#  setvar:'tx.dos_counter_threshold=100',\ 
#  setvar:'tx.dos_block_timeout=600'"

#Bỏ hết comment và chỉnh sửa như sau để có thể nhanh chóng kiểm tra
#tx.dos_burst_time_slice=30 
#tx.dos_counter_threshold=10
#setvar:'tx.dos_block_timeout=60
#khi reqquest 10 lần trong 30 giấy sẽ tính là 1 lần quá tải, khi quá tải 2 lần sẽ ban trong 60 giây(request 20 lần trong 30 giấy)

"id:900700,\
 phase:1,\
 nolog,\
 pass,\
 t:none,\
 setvar:'tx.dos_burst_time_slice=30',\  
 setvar:'tx.dos_counter_threshold=10',\ 
 setvar:'tx.dos_block_timeout=60'"


Test Dos protection bằng cách request liên tục vào trang web đến khi nào đủ giới hạn như ví dụ trên
```

* Kiểm tra SQL Injection
```php
#Truy cập vào trang quản trị và nhập vào ô user và password câu lệnh sql injection cơ bản như sau


admin' or '1'='1

#Hoặc

admin' --


```
***
## 6. Những vấn đề còn tồn tại

* Sau khi chặn người dùng Dos đến máy chủ như đã thiết lập thì chưa thể bỏ chặn người dùng được, Người dùng sẽ bị chặn vĩnh viễn cho đến khí restart lại nginx.


## 7. Tạo server mới chứa mysql và thiết lập tường lửa

* Setup truy cập mysql từ máy khác 
```php
#Cài đặt mariadb trên máy chủ mới như các bước trước đó
#truy cập vào mysql
mysql -u root -p
#Tạo tài khoản mới để truy cập từ xa
create user 'tai123'@'%' identified by 'tai0837686717';

GRANT ALL PRIVILEGES ON *.* TO 'tai123'@'192.168.1.15' IDENTIFIED BY 'tai0837686717';

FLUSH PRIVILEGES;
#Hoặc sử dụng tài khoản root để truy cập từ xa

GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.15' IDENTIFIED BY 'tai0837686717' WITH GRANT OPTION;

FLUSH PRIVILEGES;



```
* Sau khi cài đặt máy chứa mysql mariadb thì thực hiện cấu hình tường lửa như sau
```php
#Tạo 1 zone riêng

firewall-cmd --permanent --new-zone=sqlzone

firewall-cmd --reload

#Kiểm tra lại các zone

firewall-cmd --get-zones

#Thêm những luật sau vào zone: Mở ssh, mysql, chỉ cho phép ip 192.168.1.15(ip của máy muốn truy cập) truy cập và mở cổng 3306(cổng mặc định của mysql)

firewall-cmd --zone=sqlzone --add-service=mysql --permanent  
firewall-cmd --zone=sqlzone --add-service=ssh --permanent   
firewall-cmd --zone=sshzone --add-source='192.168.1.15' --permanent
firewall-cmd --zone=sqlzone --add-port=3306/tcp --permanent 
firewall-cmd --zone=sqlzone --change-interface=ens33

#Loại bỏ dịch vụ ssh trên zone public
firewall-cmd --zone=public --remove-service=ssh
firewall-cmd --reload

#kiểm tra lại

firewall-cmd --list-all-zones

#Kiểm tra đã giới hạn ssh và mysql như sau
#Sử dụng 1 máy có ip khác với ip đã thiết lập ở trên

ssh username@192.168.1.12

mysql -u tai123 -p -h 192.168.1.12

```
* Đẩy database qua máy server mới chứa mysql
```php
#Cách 1: Backup để sử dụng cho mariadb cùng phiên bản
#Cài đặt mariadb backup nếu chưa có

dnf -y install mariadb-backup

#Tạo 1 thư mục lưu trữ database backup

mkdir /home/mariadb_backup

#backup lại database(tai0837686717 là mật khẩu root của mariadb đã được thiết lập)

mariabackup --backup --target-dir /home/mariadb_backup -u root -p tai0837686717

#Sau đó đẩy file qua máy server mysql mới lập(với ip của máy đó)

cd /home/

scp -r mariadb_backup/ root@192.168.1.12:/root/mariadb_backup

#Tiếp theo là thao tác trên máy server mysql(192.168.1.12)
#Import database vừa gửi qua vào database hiện có

mariabackup --prepare --target-dir /root/mariadb_backup

mariabackup --copy-back --target-dir /root/mariadb_backup

chown -R mysql. /var/lib/mysql

systemctl restart mariadb.service

#Hoàn thành việc chuyền dữ liệu

#sau đó xóa mariadb ở máy chủ cũ đi

systemctl stop mariadb
rm -rf /var/lib/mysql/*

#Cách 2 backup file .sql
#Backup xuất file .sql

mysqldump --all-databases --user=root --password > /root/testbackup.sql

#Đẩy file qua server mysql mới
cd /root/

scp -r testbackup.sql/ root@192.168.1.12:/root/testbackup.sql

#Import file sql vào trong mariadb

cd /root/
mysql -u root -p < /root/testbackup.sql

```
* Sử dụng tường lửa CSF
```php
#cài đặt gói sau

dnf -y install @perl

#Cài đặt tường lửa CSF

cd /tmp
wget https://download.configserver.com/csf.tgz
tar -zxvf csf.tgz
cd csf
./install.sh
rm -rf /tmp/csf
rm -f /tmp/csf.tgz

#Tắt tường lửa mặc định và bật tường lửa CSF để sử dụng
systemctl stop firewalld
systemctl disable firewalld

systemctl enable csf
systemctl enable lfd
systemctl start csf

#Kiểm tra CSF đã hoạt động chưa
systemctl status csf

#Test iptable

/etc/csf/csftest.pl

#Kích hoạt
vi /etc/csf/csf.conf

#Tìm đến dòng có TESTING và FASTSTART và sửa giá trị như sau

TESTING = 0
FASTSTART = 1
#Thi thoảng tường lửa vẫn chặn ip trong csf.alow nên chỉnh về 1 để không bị chặn
IGNORE_ALLOW = "1"
#Cho phép user truy cập tới (incoming) các TCP port trên server
TCP_IN = "20,21,22,25,53,80,110,143,443,465,587,993,995"
#Ta mở thêm cổng 3306 để cho phép kết nối với mysql
TCP_IN = "20,21,22,25,53,80,110,143,443,465,587,993,995,3306"
#Server kết nối ra (outgoing) tới các TCP port bên ngoài
TCP_OUT = "20,21,22,25,53,80,110,113,443"
#Ta mở thêm công 3306 để cho phép kết nối với mysql
TCP_OUT = "20,21,22,25,53,80,110,113,443,3306"
#Chặn ping 
#Thay đổi giá trị sau
ICMP_IN = "1"
#Thành
ICMP_IN = "0"
#Truy cập vào cổng 22 5 lần trong 300 giấy sẽ bị chặn 300 giây
PORTFLOOD = “22; tcp; 5; 300

#1 Số lệnh cơ bản trong trong lửa CSF
#Start CSF	
csf -s
#Flush/Stop firewall rules (note: lfd may restart csf)	
csf -f
#Restart CSF	
csf -r
#Kiểm tra các rules
csf -l
#Allow an IP	
csf -a ip-address
#Xóa ip khỏi danh sách cho phép
csf -ar ip-address
#Deny IP	
csf -d ip-address
#Bỏ chặn ip
csf -dr ip-address
#Remove and unblock all entries	
csf -df
#Stop firewall	
csf -x
#Enable firewall	
csf -e


#Cho phép ip truy
vi /etc/csf/csf.allow

#Cho phép tcp kết nối inbound cổng 22 ip 192.168.1.2
tcp|in|d=22|s=192.168.1.2

#Cho phép tcp kết nối outbound cổng 22 ip 192.168.1.2
tcp|out|d=22|s=192.168.1.2

#Chặn ip truy cập
vi /etc/csf/csf.deny
#Chặn tcp kết nối inbound cổng 22 ip 192.168.1.12
tcp|in|d=22|s=192.168.1.12


```
## 8. Xây dựng mô hình Replication Slave đồng bộ database từ máy master qua máy slave
* Config máy master chứa database gốc
```php
#Chỉnh sửa file sau
vi /etc/my.cnf.d/mariadb-server.cnf

#add vào dưới [mysql]
log-bin=mysql-bin
# define server ID (setup duy nhất không được để trùng)
server-id=14

#restart lại mariadb
systemctl restart mariadb

#Tạo user cho slave
mysql -u root -p

grant replication slave on *.* to repl_user@'%' identified by 'slave0837686717'; 

flush privileges; 

exit;

```

* Trên máy slave thiết lập như sau
```php
#Chỉnh sửa file sau
vi /etc/my.cnf.d/mariadb-server.cnf

#Chỉnh sửa vào dưới [mysql]

log-bin=mysql-bin
# define server ID (Set ID duy nhất không trùng với master và các slave khác nếu có)
server-id=9
# read only yes
read_only=1
# define own hostname
report-host=192.168.1.14

```
* Quay về lại máy master
```php
#Tạo thư mục mariadb_backup và backup lại dữ liệu vào thư mục đấy
mkdir /home/mariadb_backup
mariabackup --backup --target-dir /home/mariadb_backup -u root -p password

#chuyển file backup qua bên máy slave
rsync -avP /home/mariadb_backup root@192.168.1.9:/root/mariadb_backup
```
* Quay về máy slave
```php
#dừng mariadb và xóa các dữ liệu hiện có
systemctl stop mariadb
rm -rf /var/lib/mysql/*

#Chạy tiếp câu lệnh
mariabackup --prepare --target-dir /root/mariadb_backup

#Nếu hiện dòng sau thì tiếp tục thực hiện các bước tiếp theo còn nếu không hãy xem lại đường dẫn hoặc thư mục backup đã được đẩy qua máy slave hay chưa
[00] 2021-08-03 16:23:30 completed OK!

#Restore lại database
mariabackup --copy-back --target-dir /root/mariadb_backup
chown -R mysql. /var/lib/mysql
systemctl start mariadb



#Chạy câu lệnh sau
cat /root/mariadb_backup/xtrabackup_binlog_info
#Sau khi chạy câu lệnh trên sẽ hiện ra đoạn mã tương tự như sau hay ghi nhớ

mysql-bin.000001        642

#Mở mysql
mysql -u root -p

#giải thích đoạn code
change master to 

#ip máy chủ
master_host='192.168.1.14',

#User đã tạo ở máy master             
master_user='repl_user',

#Mật khẩu
master_password='slave0837686717',

#Giá trị kiểm tra ở bước trên
master_log_file='mysql-bin.000001',
#Giá trị kiểm tra ở bước trên
master_log_pos=642;

#Chạy đoạn code đó như sau
change master to 
master_host='192.168.1.14',
master_user='repl_user',
master_password='slave0837686717',
master_log_file='mysql-bin.000001',
master_log_pos=642;

#Khởi động slave
start slave;

#Kiểm tra trạng thái
show slave status\G 
```
* Kiểm tra
```php
#Bên máy master tạo 1 database để kiểm tra
create database test_database; 

# Tạo bảng
create table test_database.test_table (id int, name varchar(50), address varchar(50), primary key (id)); 
Query OK, 0 rows affected (0.108 sec)

# Thêm dữ liệu
insert into test_database.test_table(id, name, address) values("001", "Rocky Linux", "Hiroshima"); 
Query OK, 1 row affected (0.036 sec)

# kiểm tra bảng
select * from test_database.test_table; 
+----+-------------+-----------+
| id | name        | address   |
+----+-------------+-----------+
|  1 | Rocky Linux | Hiroshima |
+----+-------------+-----------+


#Sau đó qua bên máy slave kiểm tra 
select * from test_database.test_table; 
#Nếu hiện đúng dữ liệu bảng vừa tạo thì đã thành công
```
* Xử lý Promoting a Slave to Master. Giả lập Master bị hỏng, ví dụ: hỏng ổ cứng, rút dây mạng, dừng dịch vụ, ….Chuyển node slave thành node master tạm thời
```php
#Truy cập vào file
vi /etc/my.cnf.d/mariadb-server.cnf
bỏ dòng read-only=1

#Đăng nhập vào mysql bên máy slave
#Xóa cấu hình slave bằng câu lệnh
stop slave;
reset slave all;
exit;
systemctl restart mysql

#Hiện tại node slave này có thể trở thành node master chúng ta có thể cấu hình lại các node slave khác nhận node này làm node master

```
* Khi master đã được sửa chữa xong chúng ta sẽ thiết lập đưa slave về đúng vị trí của nó
```php
#Đầu tiên là backup lại dữ liệu và đẩy lại qua master do trong quá trình node slave làm master có thể là đã có dữ liệu được ghi thêm vào sau đó đẩy qua bên master rồi import lại rồi bắt đầu thiết lập lại vị trí của slave 

vi /etc/my.cnf.d/mariadb-server.cnf
#Thêm lại dòng read-only vào sau [mysql]
read-only=1

#Đăng nhập lại vào mysql bên máy slave
#cấu hình lại slave
change master to 
master_host='192.168.1.14',
master_user='repl_user',
master_password='slave0837686717',
master_log_file='mysql-bin.000001',
master_log_pos=642;
start slave;

#Kiểm tra trạng thái
show slave status\G 
```

* Xử lý backup dữ liệu 1 tiếng 1 lần

```php
#Tạo thư mục lưu trữ các bản backup
mkdir /home/mariadb_backup/

#Đầu tiên viết tools backup database
#Đầu tiên tạo 1 file ở root 
vi mysql-backup.sh

#Sau đó ghi vào file đoạn code sau

TODAY=`date +"%d:%b:%Y:%H:%M"`
DB_BACKUP_PATH='/home/mariadb_backup/'
#################################################################
mkdir -p ${DB_BACKUP_PATH}/folder_${TODAY}
echo "Backup started for database "

mariabackup --backup --target-dir ${DB_BACKUP_PATH}/folder_${TODAY}/backup-mysql-on-${TODAY} -u root -p tai0837686717

if [ $? -eq 0 ]; then
echo "Database backup successfully completed"
else
echo "Error found during backup"
exit 1
fi

#Xóa cách 1: xóa file backup đã tồn tại được 3 tiếng
BACKUP_RETAIN_DAYS=3
DBDELDATE=`date +"%d:%b:%Y:%H:%M" --date="${BACKUP_RETAIN_DAYS} hour ago"`

if [ ! -z ${DB_BACKUP_PATH} ]; then
      cd ${DB_BACKUP_PATH}
      if [ ! -z ${DBDELDATE} ] && [ -d ${DBDELDATE} ]; then
            rm -rf ${DBDELDATE}
      fi
fi

#Xóa cách 2
#Tìm các file đã cũ hơn 8 tiếng và xóa nó đi
find /home/mariadb_backup -name folder_* -cmin +480 | xargs /bin/rm -rf | xargs /bin/rm -f
### End of script ####


#Ghi lại rồi thoát
#Vậy là tools backup đã được tạo
#Kiểm tra file có hoạt động không
bash mysql-backup.sh
#truy cập vào file /home/mariadb_backup/ để kiểm tra
cd /home/mariadb_backup/
ls

#sử dụng crontab để thiết lập 1 tiếng chạy tools 1 lần
crontab -e

#Backup sau moi gio
0 * * * * bash /root/mysql-backup.sh

#sau đó lưu lại và thoát
#Kiểm tra crontab đang hoạt động

crontab -l

#Muốn chỉnh sửa crontab thì tiếp tực crontab -e và sửa trong file hoặc xóa 
#Muốn xóa toàn bộ setup của crontab thì sử dụng câu lệnh crontab -r

#Kiểm tra thư mục /home/mariadb_backup/ sau 1 tiếng để thấy kết quả

cd /home/mariadb_backup/

```

## 9. Mã hóa dữ liệu khi chuyển từ master qua slave

* Đầu tiên tiến hành trên máy master cài đặt ssl
```php
yum -y install mod_sll
```
* Tạo chứng chỉ với OpenSSL
```php
#Tạo 1 thư mục lưu trữ
mkdir -p /etc/mysql/transit/
cd /etc/mysql/transit/

#Tạo key ca
openssl genrsa 2048 > ca-key.pem

openssl req -new -x509 -nodes -days 365000 \
      -key ca-key.pem -out ca.pem

Country Name (2 letter code) [XX]:VN
State or Province Name (full name) []:HaNoi
Locality Name (eg, city) [Default City]:HaNoi
Organization Name (eg, company) [Default Company Ltd]:VinaDes
Organizational Unit Name (eg, section) []:Dau Thau
Common Name (eg, your name or your server's hostname) []:CA
Email Address []:webmaster@vinades.vn

#Tạo key cho server
openssl req -newkey rsa:2048 -days 365000 \
      -nodes -keyout server-key.pem -out server-req.pem

#Tạo chứng chỉ
openssl rsa -in server-key.pem -out server-key.pem

#Nhập thông tin
#Lưu ý Common Name không để trùng nhau nếu không sẽ dẫn đến lỗi
Country Name (2 letter code) [XX]:VN
State or Province Name (full name) []:HaNoi
Locality Name (eg, city) [Default City]:HaNoi
Organization Name (eg, company) [Default Company Ltd]:VinaDes
Organizational Unit Name (eg, section) []:Dau Thau
Common Name (eg, your name or your server's hostname) []:Server
Email Address []:webmaster@vinades.vn

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

#Ký chứng chỉ
openssl x509 -req -in server-req.pem -days 365000 \
      -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
      -out server-cert.pem

#Kiểm tra
openssl verify -CAfile ca.pem server-cert.pem
#Hiển thị như sau là đã thành công còn không có thể xóa hết các file đi và làm lại
server-cert.pem: OK

```
* Tạo key và chứng chỉ cho client
```php
#Tạo khóa
openssl genrsa 2048 > client-key.pem

#Tạo chứng chỉ
openssl req -new -key client-key.pem -out client-req.pem

Country Name (2 letter code) [XX]:VN
State or Province Name (full name) []:HaNoi
Locality Name (eg, city) [Default City]:HaNoi
Organization Name (eg, company) [Default Company Ltd]:VinaDes
Organizational Unit Name (eg, section) []:Dau Thau
Common Name (eg, your name or your server's hostname) []:Client
Email Address []:phamductai123456@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

#Ký chứng chỉ
openssl x509 -req -in client-req.pem -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out client-cert.pem -days 3650 -sha256

#Kiểm tra chứng chỉ
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
```
* Bên máy slave
```php
#Tạo thư mục lưu trữ các chứng chỉ
mkdir -p /etc/mysql/transit/
```
* Quay về master rồi chuyển các chứng chỉ qua bên slave
```php
scp -r /etc/mysql/transit/* root@192.168.1.9:/etc/mysql/transit/
```
* Tại cả máy master và máy slave thực hiện cấp quyền cho file vừa rồi như sau
```php
cd /etc/mysql/transit
$ chown -R mysql:mysql *
$ chmod 600 client-key.pem server-key.pem ca-key.pem
```
* Thực hiện chỉnh sửa file hệ thống của mariadb của cả master và slave
```php
#Chỉnh sửa file sau
vi /etc/my.cnf.d/mariadb-server.cnf

#add vào dưới [mysql]
ssl-ca=/etc/mysql/transit/ca.pem
ssl-cert=/etc/mysql/transit/server-cert.pem
ssl-key=/etc/mysql/transit/server-key.pem

#Khởi động lại mariadb
systemctl restart mariadb
```
* Quay trở lại máy master, đăng nhập vào maridb, yêu cầu người dùng sử dụng ssl 
```php
#Require ssl cho user tai123
#Nếu chưa tạo user có thể tạo user riêng
GRANT ALL PRIVILEGES ON *.* TO 'tai123'@'192.168.1.9' IDENTIFIED BY 'tai0837686717' REQUIRE SSL;

FLUSH PRIVILEGES;
#Khởi động lại mariadb
systemctl restart mariadb
```
* Máy slave kiểm tra
```php
mysql -u tai123 -p -h 192.168.1.14 --ssl-cert client-cert.pem --ssl-key client-key.pem --ssl-ca ca.pem -e 'status'

#Chú ý 2 dòng trạng thái sau nếu như ssl hiển thị tương tự như vậy là 2 máy đã kết nối với nhau có mã hóa
Current user:           tai123@192.168.1.9
SSL:                    Cipher in use is TLS_AES_256_GCM_SHA384

```
* Tiếp tục trên máy slave chính sửa file cấu hình mariadb
```php
vi /etc/my.cnf.d/mariadb-server.cnf
#Thêm vào file đoạn sau hoạc nếu đã có [client] trong file thì chỉ thêm đoạn mã ở dưới [client]
[client]
ssl-ca=/etc/mysql/transit/ca.pem
ssl-cert=/etc/mysql/transit/client-cert.pem
ssl-key=/etc/mysql/transit/client-key.pem

#Sau đó đăng nhập vào mariadb và dừng slave
STOP SLAVE;
#Khởi động lại mariadb
systemctl restart mariadb
```
* Quay lại máy master
```php
GRANT ALL PRIVILEGES ON *.* TO 'repl_user'@'192.168.1.9' IDENTIFIED BY 'slave0837686717';

ALTER USER repl_user@'192.168.1.9' identified by 'slave0837686717' REQUIRE SSL;

FLUSH PRIVILEGES;

#Khởi động lại mariadb

systemctl restart mariadb
```
* Quay lại máy slave
```php
#Kiểm tra
mysql -u repl_user -p -h 192.168.1.14 --ssl -e 'status'

#SSL hiển thị tương tự như sau là đã thành công
Current user:           repl_user@192.168.1.9
SSL:                    Cipher in use is TLS_AES_256_GCM_SHA384


#Đăng nhập vào mariadb sau đó chạy đoạn code sau

CHANGE MASTER TO MASTER_SSL = 1, MASTER_SSL_CA = '/etc/mysql/transit/ca.pem', MASTER_SSL_CERT = '/etc/mysql/transit/client-cert.pem', MASTER_SSL_KEY = '/etc/mysql/transit/client-key.pem';

#Mở lại slave
START SLAVE;
#Kiểm tra slave
SHOW SLAVE STATUS\G

Slave_IO_State: Waiting for master to send event
    Master_Host: 192.168.1.14
    Master_User: repl_user
    Master_Port: 3306
    Connect_Retry: 60
    Master_Log_File: mysql-bin.000024
    Read_Master_Log_Pos: 1401
    Relay_Log_File: mariadb-relay-bin.000004
    Relay_Log_Pos: 1700
    Relay_Master_Log_File: mysql-bin.000024
    Slave_IO_Running: Yes
    Slave_SQL_Running: Yes
    Master_SSL_Allowed: Yes
    Master_SSL_CA_File: /etc/mysql/transit/ca.pem
    Master_SSL_Cert: /etc/mysql/transit/client-cert.pem
    Master_SSL_Key: /etc/mysql/transit/client-key.pem

#Đã thực hiện mã hóa thành công nếu như 3 dòng trạng thái sau hiển thị tương tự
#Slave_IO_Running: Yes
#Slave_SQL_Running: Yes
#Master_SSL_Allowed: Yes

```
* Tiến hành kiểm tra dữ liệu được truyền đi đã mã hóa hay chưa
```php
#Cài đặt gói tcpdump trên máy salve
yum install tcpdump 

#Chạy đoạn code sau trên máy salve để kiểm tra truyền dữ liệu ở cổng 3306
tcpdump -i ens33 -s 0 -l -w - 'src port 3306 or dst port 3306' | strings

#Ở máy Master insert thêm dữ liệu vào bảng test đã tạo ở bài thiết lập mô hình slave rồi sau đó quay ra kiểm tra bên máy salve

#Ta thấy được dữ liệu đã được mã hóa thành công

|`.K
LB.o
X6h"
3I&x d
=t-E
z#i[
'~Vx`
hJ;a
p\![|
cHjU
OS1z
6;rM
xf58
,`C0


^C34 packets captured
34 packets received by filter
0 packets dropped by kernel
```

## 10. Xử lý chuyển slave thành master khi có sự cố
* Giả xử có 1 máy master 192.168.1.14 và 2 máy slave 192.168.1.9 và 192.168.1.12
* Khi máy 192.168.1.14 bị hỏng giải quyết để chuyển 192.168.1.9 thành master máy còn lại vẫn là slave
* Bước 1: thực hiện trên 192.168.1.9
```php
vi /etc/my.cnf.d/mariadb-server.cnf
# Bỏ dòng read-only =1
# Bỏ dòng report-host=192.168.1.14

#Sau đó đăng nhập vào mysql
mysql -u root -p
stop slave;
reset slave all;
exit;
systemctl restart mysql

#backup lại sql
mariabackup --backup --target-dir /home/mariadb_backup_2 -u root -p tai0837686717

#Chuyển file qua bên máy slave 192.168.1.12
rsync -avP /home/mariadb_backup_2 root@192.168.1.12:/root/

```

* Qua bên máy slave 192.168.1.12
```php
#trước khi thiết lập hãy xóa toàn bộ cấu hình slave
mysql -u root -p
stop slave;
reset slave all;
exit;
systemctl restart mysql

vi /etc/my.cnf.d/mariadb-server.cnf
# Thay đổi report-host=192.168.1.14
report-host=192.168.1.9

#Thực hiện lại các bước thiết lập slave
systemctl stop mariadb
rm -rf /var/lib/mysql/*

#Chạy tiếp câu lệnh
mariabackup --prepare --target-dir /root/mariadb_backup_2

#Nếu hiện dòng sau thì tiếp tục thực hiện các bước tiếp theo còn nếu không hãy xem lại đường dẫn hoặc thư mục backup đã được đẩy qua máy slave hay chưa
[00] 2021-08-03 16:23:30 completed OK!

#Restore lại database
mariabackup --copy-back --target-dir /root/mariadb_backup_2
chown -R mysql. /var/lib/mysql
systemctl start mariadb



#Chạy câu lệnh sau
cat /root/mariadb_backup_2/xtrabackup_binlog_info

mysql-bin.000011    328

#Truy cập mysql

mysql -u root -p

#Thiết lập slave
change master to 
master_host='192.168.1.9',
master_user='repl_user',
master_password='slave0837686717',
master_log_file='mysql-bin.000011',
master_log_pos=328;

#Thiết lập ssl
CHANGE MASTER TO MASTER_SSL = 1, MASTER_SSL_CA = '/etc/mysql/transit/ca.pem', MASTER_SSL_CERT = '/etc/mysql/transit/client-cert.pem', MASTER_SSL_KEY = '/etc/mysql/transit/client-key.pem';

#Mở lại slave
START SLAVE;
#Kiểm tra slave
SHOW SLAVE STATUS\G

```

