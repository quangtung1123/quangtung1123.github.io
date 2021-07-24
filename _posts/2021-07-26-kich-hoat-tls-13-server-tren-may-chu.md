---
layout: post
comments: true
title:  "Hướng dẫn kích hoạt TLS 1.3 server trên các loại máy chủ website"
date:   2021-07-26 15:10:00
permalink: 2021/07/26/kich-hoat-tls-13-server-tren-may-chu
mathjax: false
tags: Server TLS1.3 TLS
category: TLS
sc_project: 12494739
sc_security: d785a534
img: /assets/kich-hoat-tls-13-server-tren-may-chu/Hinh1.png
summary: Hướng dẫn kích hoạt TLS 1.3 server trên các loại máy chủ website

---

Trước khi thực hiện quy trình, chúng ta hãy xem TLS 1.3 là gì, nó khác gì với 1.2, lịch sử và khả năng tương thích tại [link này](/2021/07/25/tong-quan-ve-tls-13-nhanh-hon-va-bao-mat-hon).

## Bật TLS 1.3 trong Nginx ##

TLS 1.3 được hỗ trợ bắt đầu từ phiên bản Nginx 1.13 . Nếu bạn đang chạy phiên bản cũ hơn, thì trước tiên, bạn phải nâng cấp.
Tôi giả sử bạn có Nginx 1.13+, thì thực hiện như sau:

- Đăng nhập vào máy chủ Nginx
- Sao lưu cấu hình file nginx.conf
- Sửa đổi nginx.conf bằng cách sử dụng vi hoặc trình soạn thảo yêu thích của bạn

Cấu hình mặc định trong cài đặt SSL sẽ là:

```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
```

- Thêm TLSv1.3 vào cuối dòng:

```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
```

Lưu ý: cấu hình trên sẽ cho phép TLS 1 / 1.1 / 1.2 / 1.3. Nếu bạn muốn bật TLS 1.2 / 1.3 cho an toàn, thì cấu hình của bạn sẽ như sau:

```
ssl_protocols TLSv1.2 TLSv1.3;
```

- Khởi động lại Nginx:

```
systemctl restart nginx
```

## Bật TLS 1.3 trong Apache ##

Bắt đầu từ Apache HTTP 2.4.38, bạn có thể tận dụng TLS 1.3. Nếu bạn vẫn đang sử dụng phiên bản cũ hơn, thì bạn phải nghĩ đến việc nâng cấp phiên bản đó trước.

Cấu hình như sau:

- Đăng nhập vào máy chủ Apache HTTP và tạo bản sao lưu file ssl.conf hoặc nơi có cấu hình SSL
- Xác định dòng SSLProtocoldòng và thêm +TLSv1.3 vào cuối dòng

Ví dụ: dòng sau cho phép TLS 1.2 và TLS 1.3

```
SSLProtocol -all +TLSv1.2 +TLSv1.3
```

- Khởi động lại Apache HTTP

```
systemctl restart httpd
```

## Bật TLS 1.3 trong Cloudflare ##

Một trong những nhà cung cấp CDN đầu tiên triển khai hỗ trợ TLS 1.3. Cloudflare bật nó theo mặc định cho tất cả các trang web.

Tuy nhiên, nếu bạn cần vô hiệu hóa hoặc kiểm tra, thì đây là cách bạn có thể làm điều đó.

- Đăng nhập vào Cloudflare
- Chuyển đến tab SSL / TLS >> Edge certificates
- Cuộn xuống một chút và bạn sẽ thấy tùy chọn TLS 1.3

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-server-tren-may-chu/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

## Bật TLS 1.3 trong Windows server ##

Theo thông tin từ Microsof, TLS 1.3 sẽ được bật mặc định trên Windows Server 2022, còn các phiên bản cũ hơn sẽ không được hỗ trợ.

Một trong những nhà cung cấp CDN đầu tiên triển khai hỗ trợ TLS 1.3.

## Kiểm tra trang web đang sử dụng TLS 1.3? ##

Có nhiều cách để kiểm tra nó:

- [Geekflare TLS Test](https://gf.dev/tls-test) - nhanh chóng tìm ra phiên bản TLS được hỗ trợ:

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-server-tren-may-chu/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- [SSL Labs](https://www.ssllabs.com/ssltest/) - nhập URL HTTPS của bạn và cuộn xuống trên trang kết quả kiểm tra:

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-server-tren-may-chu/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Bạn sẽ thấy tất cả những gì về các giao thức được kích hoạt.

- Google Chrome - nếu bạn đang bật trên các trang web mạng nội bộ, thì bạn có thể kiểm tra nó ngay từ trình duyệt Chrome

  - Khởi chạy Chrome
  - Mở công cụ dành cho nhà phát triển (Developer Tools)
  - Chuyển đến tab Bảo mật (Security)
  - Truy cập URL HTTPS
  - Bên trái, chọn main origin để xem giao thức (protocol):
  
<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-server-tren-may-chu/Hinh4.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**

- [geekflare](https://geekflare.com/enable-tls-1-3/)
- [microsoft](https://docs.microsoft.com/en-us/windows/win32/secauthn/protocols-in-tls-ssl--schannel-ssp-)
- [microsoft-dev](https://devblogs.microsoft.com/premier-developer/microsoft-tls-1-3-support-reference/)

---
