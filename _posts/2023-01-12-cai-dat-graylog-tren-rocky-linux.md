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

- Sau đó, cài đặt bản phát hành MongoDB bằng lệnh sau:
```
sudo yum install mongodb-org
```

- Khởi động MongoDB và cài đặt khởi động cùng hệ điều hành:
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

- Sau đó, cài đặt bản phát hành Elaticsearch bằng lệnh sau:
```
sudo yum install elasticsearch-oss
```

- Thay đổi File cấu hình Elasticsearch (/etc/elasticsearch/elasticsearch.yml) và thay đổi tên cluster sang graylog và thêm dòng action.auto_create_index: false:
```
sudo tee -a /etc/elasticsearch/elasticsearch.yml > /dev/null <<EOT
cluster.name: graylog
action.auto_create_index: false
EOT
```

- Khởi động Elasticsearch và cài đặt khởi động cùng hệ điều hành:
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl restart elasticsearch.service
sudo systemctl --type=service --state=active
```

## Cài đặt Graylog ##
- Cài đặt Graylog bằng các lệnh sau:
```
sudo rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-5.0-repository_latest.rpm
sudo yum install graylog-server
```

- Chỉnh sửa file cấu hình /etc/graylog/server/server.conf và thêm 2 tham số password_secret và root_password_sha2. Để tạo root_password_sha2 sử dụng lệnh sau:

```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1

```

- Tiếp theo, nhập mật khẩu cần tạo cho tài khoản admin để đăng nhập graylog:
Ví dụ : logtaptrung@12345 là mật khẩu được sử dụng cho tài khoản admin.
Sau khi tạo, mật khẩu sẽ có dạng như sau :
```
bde31aa7fe8fb4827ec9477429fab4dd74f7e0f304df292f0355fa3d55888626
```

- Lưu lại và gán cho root_password_sha2:
```
sed -i 's|root_password_sha2 =|root_password_sha2 = bde31aa7fe8fb4827ec9477429fab4dd74f7e0f304df292f0355fa3d55888626|g' /etc/graylog/server/server.conf
```

- Sửa múi giờ sang giờ Vietnam :
```
sed -i 's|#root_timezone = UTC|root_timezone = Asia/Ho_Chi_Minh|' /etc/graylog/server/server.conf
```

- Để truy cập trang web bằng địa chỉ ip máy Graylog, ta cần sửa địa chỉ ip mặc định thành địa chỉ ip của máy.
```
sed -i 's|#http_bind_address = 127.0.0.1:9000|http_bind_address = 10.10.34.101:9000|' /etc/graylog/server/server.conf
```

- Bước cuối cùng là khởi động Graylog và cài đặt khởi động cùng hệ điều hành:
```
sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service
sudo systemctl --type=service --state=active
grep graylog
```

- Để truy cập graylog qua port 9000, ta cần tiến hành mở port trên firewall:
```
firewall-cmd --zone=public --add-port=9000/tcp --permanent
firewall-cmd --reload
```

- Sau đó kiểm tra lại xem các port của graylog đã xuất hiện chưa
```
ss -lan | egrep "9000|27017|9200|9300"
```

## Cấu hình SELinux ##
Nếu bạn đang sử dụng SELinux trên hệ thống của mình, bạn cần quan tâm đến các cài đặt sau:

- Cho phép máy chủ web truy cập mạng:sudo setsebool -P httpd_can_network_connect 1
- Nếu chính sách trên không tuân thủ chính sách bảo mật của bạn, bạn cũng có thể cho phép truy cập vào từng cổng riêng lẻ:
	- Graylog REST API và giao diện web:sudo semanage port -a -t http_port_t -p tcp 9000
	- Elaticsearch (chỉ khi API HTTP đang được sử dụng):sudo semanage port -a -t http_port_t -p tcp 9200
- Cho phép sử dụng cổng mặc định của MongoDB (27017/tcp):sudo semanage port -a -t mongod_port_t -p tcp 27017

Nếu bạn chạy một môi trường máy chủ duy nhất với proxy NGINX hoặc Apache , thì việc bật API Graylog REST là đủ. Tất cả các quy tắc khác chỉ được yêu cầu trong thiết lập nhiều nút. Việc SELinux bị vô hiệu hóa trong khi cài đặt và kích hoạt nó sau này, yêu cầu bạn phải kiểm tra thủ công các chính sách cho MongoDB, Elaticsearch và Graylog.

## Đăng nhập##

- Mở trình duyệt và nhập đường link đã tạo trong file config của graylog 10.10.34.101:9000, đăng nhập bằng tài khoản admin và mật khẩu thuctapsinh@123

<div class="imgcap">
<div >
    <img src="/assets/cai-dat-graylog-tren-rocky-linux/H1.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

- Sau khi đăng nhập sẽ có giao diện như bên dưới:
<div class="imgcap">
<div >
    <img src="/assets/cai-dat-graylog-tren-rocky-linux/H2.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**
- [graylog](https://go2docs.graylog.org/5-0/downloading_and_installing_graylog/red_hat_installation.htm)
- [cloud365](https://news.cloud365.vn/graylog-lab-phan1-huong-dan-cai-dat-graylog-3-1-tren-centos-7/)