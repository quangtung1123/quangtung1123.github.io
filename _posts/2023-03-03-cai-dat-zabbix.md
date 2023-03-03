---
layout: post
title:  "Cài đặt Zabbix"
date:   2022-09-16 10:48:00
permalink: 2023/03/03/cai-dat-zabbix
tags: Monitoring Zabbix
category: Monitoring
img: /assets/cai-dat-zabbix/Hinh1.jpg
summary: Cài đặt Zabbix

---

Cái đặt Zabbix 6.2 trên hệ điều hành AlmaLinux bao gồm các thành phần:
- PHP
- Apache web server
- MySQL/ MariaDB database server

Các bước cụ thể như sau:

## Cấu hình SELinux sang chế độ permissive ##

```
sudo setenforce 0 && sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
cat /etc/selinux/config | grep SELINUX=
```

## Cài đặt Zabbix server, frontend, and agent ##

```
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.2/rhel/9/x86_64/zabbix-release-6.2-3.el9.noarch.rpm
sudo dnf clean all
sudo dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

## Cài đặt và cấu hình database ##

- Cài đặt MariaDB:
```
sudo dnf install mariadb-server mariadb -y
```

- Start and enable the service:
```
sudo systemctl start mariadb && sudo systemctl enable mariadb
```

- Harden the MariaDB instance:
```
sudo mariadb-secure-installation
....
Enter current password for root (enter for none): Press Enter
.....
Switch to unix_socket authentication [Y/n] y
.....
Change the root password? [Y/n] y
New password: 
Re-enter new password:
.....
Remove anonymous users? [Y/n] y
....
Disallow root login remotely? [Y/n] y
....
Remove test database and access to it? [Y/n] y

Reload privilege tables now? [Y/n] y
....
Thanks for using MariaDB!
```

- Login vào MariaDB server và tạo database cho Zabbix:
```
mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
```

- Import dữ liệu khởi tạo schema và data cho database:
```
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

- Disable lựa chọn log_bin_trust_function_creators sau khi importing database schema.
```
# mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

## Cấu hình database cho Zabbix server ##
- Mở file /etc/zabbix/zabbix_server.conf và thay đổi tham số liên quan đến DB:

```
sudo vi /etc/zabbix/zabbix_server.conf
```

```
DBName=zabbix
DBUser=zabbix
DBPassword=StrongDBPassw0rd
```


## Cấu hình Timezone trong PHP ##

```
$ sudo vim /etc/php-fpm.d/zabbix.conf
php_value[date.timezone] = Asia/Ho_Chi_Minh
```


## Start Zabbix server and agent processes ##

```
sudo systemctl restart zabbix-server zabbix-agent httpd php-fpm
sudo systemctl enable zabbix-server zabbix-agent httpd php-fpm
```

## Mở cổng kết nối trên firewall ##

```
sudo firewall-cmd --add-service={http,https} --permanent
sudo firewall-cmd --add-port={10051/tcp,10050/tcp} --permanent
sudo firewall-cmd --reload
```

## Mở Zabbix UI web page ##
- Truy cập Zabbix web UI qua URL http://IP_Address/zabbix/ hoặc http://domain_name/zabbix/ và tiếp tục thực hiện cấu hình theo hướng dẫn của web.
- Tài khoản đăng nhập mặc định là:
```
Username: Admin
Password: zabbix
```

## Khắc phục lỗi ##
- Lỗi "Locale for language "en_US" is not found on the web server", thì thực hiện cài đặt thêm gói ngôn ngữ và khởi động lại php-fpm:

```
sudo dnf install glibc-langpack-en
sudo systemctl restart php-fpm
```


**Tài liệu tham khảo**
- [zabbix](https://www.zabbix.com/download?zabbix=6.2&os_distribution=alma_linux&os_version=9&components=server_frontend_agent&db=mysql&ws=apache)
- [computingforgeeks](https://computingforgeeks.com/install-configure-zabbix-server-on-rocky-almalinux/)
