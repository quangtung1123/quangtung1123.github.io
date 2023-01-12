---
layout: post
title:  "Cài đặt Graylog trên Rocky Linux"
date:   2023-01-12 10:48:00
permalink: 2023/01/12/cai-dat-graylog-tren-rocky-linux
tags: Log Graylog
category: Log
img: /assets/cai-dat-graylog-tren-rocky-linux/Hinh1.jpg
summary: Cài đặt phần mềm quản lý log tập trung Graylog trên Rocky Linux

---

## Chuẩn bị ##
Graylog 5.0 yêu cầu các phần mềm kèm theo như sau:
- OpenJDK 17 (được nhúng trong tệp cài đặt 5.0)
- Elaticsearch 7.10.2
- MongoDB (5.x hoặc 6.x)

## Cài đặt MongoDB ##
- Đầu tiên, thêm tệp kho lưu trữ /etc/yum.repos.d/mongodb-org.repo với các nội dung sau (thay thế 5.x bằng phiên bản bạn đã chọn):

```
[mongodb-org-5.x]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/Red Hat/$releasever/mongodb-org/5.x/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-5.x.asc
```

Sau đó, cài đặt bản phát hành MongoDB bằng lệnh sau:
```
sudo yum install mongodb-org
```

Khởi động MongoDB và cài đặt khởi động cùng hệ điều hành:
```
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl start mongod.service
```

## Cài đặt Elaticsearch ##
- Graylog 5.0 chỉ có thể được sử dụng với Elaticsearch 7.10.2.
- Đầu tiên, cài đặt khóa GPG:
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

- Thêm tệp kho lưu trữ vào file /etc/yum.repos.d/elasticsearch.repo với nội dung sau:
```
[elasticsearch-7.10.2]
name=Elasticsearch repository for 7.10.2 packages
baseurl=https://artifacts.elastic.co/packages/oss-7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Sau đó, cài đặt bản phát hành Elaticsearch bằng lệnh sau:
```
sudo yum install elasticsearch-oss
```

Thay đổi File cấu hình Elasticsearch (/etc/elasticsearch/elasticsearch.yml) và thay đổi tên cluster sang graylog và thêm dòng action.auto_create_index: false:
```
sudo tee -a /etc/elasticsearch/elasticsearch.yml > /dev/null <<EOT
cluster.name: graylog
action.auto_create_index: false
EOT
```

Khởi động Elasticsearch và cài đặt khởi động cùng hệ điều hành:
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl restart elasticsearch.service
sudo systemctl --type=service --state=active
```

## Cài đặt Graylog ##
Cài đặt Graylog bằng các lệnh sau:
```
sudo rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-5.0-repository_latest.rpm
sudo yum install graylog-server
```

Chỉnh sửa file cấu hình /etc/graylog/server/server.conf và thêm 2 tham số password_secret và root_password_sha2.
Để tạo root_password_sha2 sử dụng lệnh sau:

```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```
Để có thể kết nối với Graylog, bạn nên đặt http_bind_address tên máy chủ công khai hoặc địa chỉ IP công khai của máy mà bạn có thể kết nối.

Bước cuối cùng là khởi động Graylog và cài đặt khởi động cùng hệ điều hành:
```
sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service
sudo systemctl --type=service --state=active
grep graylog
```

## Cấu hình SELinux ##
Nếu bạn đang sử dụng SELinux trên hệ thống của mình, bạn cần quan tâm đến các cài đặt sau:

- Cho phép máy chủ web truy cập mạng:sudo setsebool -P httpd_can_network_connect 1
- Nếu chính sách trên không tuân thủ chính sách bảo mật của bạn, bạn cũng có thể cho phép truy cập vào từng cổng riêng lẻ:
	- Graylog REST API và giao diện web:sudo semanage port -a -t http_port_t -p tcp 9000
	- Elaticsearch (chỉ khi API HTTP đang được sử dụng):sudo semanage port -a -t http_port_t -p tcp 9200
- Cho phép sử dụng cổng mặc định của MongoDB (27017/tcp):sudo semanage port -a -t mongod_port_t -p tcp 27017

Nếu bạn chạy một môi trường máy chủ duy nhất với proxy NGINX hoặc Apache , thì việc bật API Graylog REST là đủ. Tất cả các quy tắc khác chỉ được yêu cầu trong thiết lập nhiều nút. Việc SELinux bị vô hiệu hóa trong khi cài đặt và kích hoạt nó sau này, yêu cầu bạn phải kiểm tra thủ công các chính sách cho MongoDB, Elaticsearch và Graylog.