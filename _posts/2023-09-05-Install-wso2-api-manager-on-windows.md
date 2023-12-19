---
layout: post
title:  "Cài đặt WS02 API Manager trên Windows"
date:   2023-09-05 14:50:00
permalink: 2023/09/05/Install-wso2-api-manager-on-windows
tags: SSO WS02
category: SSO
img: /assets/Install-wso2-api-manager-on-windows/oauth-app-select.png
summary: Cài đặt WS02 API Manager trên Windows

---

## Tải WS02 ##
- Tải bản cài đặt từ trang web của WS02 (phiên bản hiện tại 9/2023 là 4.2.0): https://wso2.com/api-manager
- Giải nén và để vào thư mục bất kỳ trên Windows, ví dụ D:\wso2am-4.2.0

## Cài đặt OpenJDK ##
- Tải và cài đặt OpenJDK tương ứng với hệ điều hành và phiên bản WS02. Ví dụ: WS02 4.2.0 sẽ tương thích với OpenJDK 17: https://learn.microsoft.com/vi-vn/java/openjdk/download
- Tải xong thì cài đặt vào windows. Lưu ý: chọn Add Path JAVA_HOME khi cài đặt, nếu không chọn thì thêm thủ công sau khi cài đặt.
- Kiểm tra phiên bản java bằng cách mở Command Prompt và chạy lệnh 'java --version'
```
> java --version
openjdk 17.0.8.1 2023-08-24 LTS
OpenJDK Runtime Environment Microsoft-8297089 (build 17.0.8.1+1-LTS)
OpenJDK 64-Bit Server VM Microsoft-8297089 (build 17.0.8.1+1-LTS, mixed mode, sharing)
```

***Lưu ý: tải OpenJDK, chứ không phải JDK***

## Khởi động WSO2 ##
- Vào thư mục bin trong thư mục chứa WS02 ở bước trên
- Chạy file 'api-manager.bat'

## Truy cập vào cấu hình ##
- Truy cập các link phía dưới để sử dụng:
```
- https://localhost:9443/publisher
- https://localhost:9443/carbon
- https://localhost:9443/devportal
```
- Tài khoản, mật khẩu mặc định là admin/admin. Để thay đổi mật khẩu thì vào devportal và vào phần Change password.

## Cấu hình tên miền ##
Để thay đổi từ localhost sang 1 tên miền cụ thể, ví dụ: example.vn, ta thực hiện như sau:
- Mở file <WS02_HOME>/repository/conf/deployment.toml
- Chỉnh sửa thuộc tính hostname và các url có bắt đầu bằng http hoặc https sang tên miền tương ứng:

```
[server]
hostname = "example.vn"

service_url = "https://example.vn:${mgt.transport.https.port}/services/"

http_endpoint = "http://example.vn:${http.nio.port}"
https_endpoint = "https://example.vn:${https.nio.port}"
websub_event_receiver_http_endpoint = "http://example.vn:9021"
websub_event_receiver_https_endpoint = "https://example.vn:8021"

notification_endpoint = "https://example.vn:${mgt.transport.https.port}/internal/data/v1/notify"
```

- Khởi động lại WS02;
- Truy cập các link theo tên miền đã thiết lập để kiểm tra:
```
- https://example.vn:9443/publisher
- https://example.vn:9443/carbon
- https://example.vn:9443/devportal
```

Sau khi thay đổi tên miền thì truy cập vào publisher hoặc devportal sẽ báo lỗi "Hostname configuration in deployment.toml is not reflected in callback url for publisher and devportal". Ta xử lý như sau:
- Đăng nhập vào carbon: https://example.vn:9443/carbon;
- Vào phần Service providers list:

<div class="imgcap">
<div >
    <img src="/assets/Install-wso2-api-manager-on-windows/service-providers.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Click vào nút Edit của API Publisher service provider
<div class="imgcap">
<div >
    <img src="/assets/Install-wso2-api-manager-on-windows/service-providers-list.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Vào phần Inbound Authentication Configuration > OAuth/OpenID Connect Configuration và click vào nút Edit OAuth application.

<div class="imgcap">
<div >
    <img src="/assets/Install-wso2-api-manager-on-windows/oauth-app-select.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Xem phần Callback Url regex và thay đổi từ localhost sang tên miền mới tương ứng:
```
regexp=(https://localhost:9443/publisher/services/auth/callback/login|https://localhost:9443/publisher/services/auth/callback/logout)
```
thay đổi thành:
```
regexp=(https://example.com:9443/publisher/services/auth/callback/login|https://example.com:9443/publisher/services/auth/callback/logout)
```

- Click vào nút Update trong trang Applications Settings, sau đó click nút Update trong trang service provider information để lưu lại.

- Chọn service provider Cho API Developer Portal (apim_devportal) và lặp lại các bước 4 - bước 6 để thay đổi.

## Tham khảo ##
- [wso2](https://is.docs.wso2.com/en/latest/deploy/change-the-hostname/)
- [wso2](https://apim.docs.wso2.com/en/latest/troubleshooting/troubleshooting-invalid-callback-error/)
