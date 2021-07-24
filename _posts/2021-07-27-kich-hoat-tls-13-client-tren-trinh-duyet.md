---
layout: post
comments: true
title:  "Hướng dẫn kích hoạt TLS 1.3 client trên các loại trình duyệt"
date:   2021-07-27 15:10:00
permalink: 2021/07/27/kich-hoat-tls-13-client-tren-trinh-duyet
mathjax: false
tags: Browser TLS1.3 TLS
category: TLS
sc_project: 12494739
sc_security: d785a534
img: /assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh1.png
summary: Hướng dẫn kích hoạt TLS 1.3 client trên các loại trình duyệt

---

Trước khi thực hiện quy trình, chúng ta hãy xem TLS 1.3 là gì, nó khác gì với 1.2, lịch sử và khả năng tương thích tại [link này](/2021/07/25/tong-quan-ve-tls-13-nhanh-hon-va-bao-mat-hon).

Nếu bạn đang sở hữu trang web, thì bạn có thể xem xét kích hoạt nó ngay hôm nay. Hãy xem bài viết trước về [Hướng dẫn kích hoạt TLS 1.3 server trên các loại máy chủ website](/2021/07/26/kich-hoat-tls-13-server-tren-may-chu).

Còn làm thế nào về phía máy khách - các trình duyệt?

Chrome bắt đầu từ phiên bản 63 và Firefox 61 đã bắt đầu hỗ trợ TLS 1.3 và nếu trình duyệt của bạn chưa hỗ trợ điều này, thì bạn đang bỏ lỡ các tính năng về hiệu suất và quyền riêng tư.

## Bật TLS 1.3 trong Chrome ##

- Mở Chrome
- Nhập thông tin sau vào thanh địa chỉ và nhấn Enter

```
chrome://flags/#tls13-variant
```

Đảm bảo nó không bị vô hiệu hóa (disabled). Bạn có thể chọn Mặc định (Default) hoặc Đã bật (Enabled).

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Khởi động lại Chrome để cài đặt có hiệu lực

## Bật TLS 1.3 trong Firefox ##

- Mở Firefox
- Gõ  about:config vào thanh địa chỉ và nhấn Enter
- Nhập  tls.version vào ô tìm kiếm và bạn sẽ thấy những phần sau:

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Đảm  bảo giá trị security.tls.version.max là 4
- Nếu không, hãy nhấp đúp vào nó để sửa đổi thành 4.

## Bật TLS 1.3 trong Safari ##

- Mở Terminal với quyền root

```
sudo su - root
```

- Nhập lệnh sau và nhấn Enter

```
defaults write /Library/Preferences/com.apple.networkd tcp_connect_enable_tls13 1
```

- Khởi động lại Safari để tận hưởng những lợi ích.

## Bật TLS 1.3 trong Microsoft Edge (Chromium) ##

- Mở Microsoft Edge
- Nhập thông tin sau vào thanh địa chỉ và nhấn Enter

```
edge://flags
```

- Tìm kiếm TLS 1.3 và chọn Mặc định (Default) hoặc Kích hoạt (Enabled).

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh3.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

- Khởi động lại Microsoft Edge để cài đặt có hiệu lực


## Bật TLS 1.3 client trong Windows ##

Một số trình duyệt như: Internet Explorer, Microsoft Edge phiên bản cũ sử dụng Windows TLS stack thì bật như sau:

- Nhập inetcpl.cpl vào Run (Win + R) và nhấn phím Enter.
- Cửa sổ Internet Properties sẽ mở. Hãy chuyển sang phần Advanced.
- Trong phần bảo mật, hãy chọn hộp TLS 1.3.
- Khởi động lại trình duyệt.

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh4.png" width = "800">
</div>
<div class="thecap"></div>
</div>

TLS 1.3 mới chỉ được hỗ trợ trên Windows 1903 khi bài viết này phát hành và chỉ cho mục đích thử nghiệm, không phải môi trường sản xuất. Còn đối với Windows server sẽ được hỗ trợ chính thức từ phiên bản Windows server 2022.

## TLS 1.3 có được hỗ trợ trên .NET không? ##

Đối với .NET, [hướng dẫn chính thức](https://docs.microsoft.com/en-us/dotnet/framework/network-programming/tls) tại thời điểm này là dựa vào hệ điều hành cơ bản để cung cấp phiên bản TLS (mặc định sẽ tự động chọn phiên bản mạnh nhất hiện có của giao thức TLS) và tránh mã cứng/chỉ định một phiên bản TLS cố định trong mã ứng dụng.

Bắt đầu với .NET Framework 4.7, cấu hình mặc định là sử dụng phiên bản TLS của OS.

## Kiểm tra trình duyệt đã bật TLS 1.3 ##

Làm cách nào để bạn đảm bảo trình duyệt của mình hỗ trợ phiên bản TLS mới nhất? Có một số công cụ bạn có thể sử dụng và chỉ cần nhấn vào trang sau để kiểm tra nó:

- [Cloudflare](https://www.cloudflare.com/ssl/encrypted-sni/)  - đây là kết quả hiển thị khi trình duyệt hỗ trợ nó:

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh5.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- [SSL Labs](https://www.ssllabs.com/ssltest/viewMyClient.html) -  kết quả kiểm tra:

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh6.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- [How's My SSL](https://www.howsmyssl.com/) - kiểm tra khả năng tương thích giao thức SSL/TLS, hỗ trợ tìm lỗ hổng bảo mật đã biết:

<div class="imgcap">
<div >
    <img src="/assets/kich-hoat-tls-13-client-tren-trinh-duyet/Hinh7.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**

- [geekflare](hhttps://geekflare.com/enable-tls-1-3-in-browsers/)
- [microsoft](https://docs.microsoft.com/en-us/windows/win32/secauthn/protocols-in-tls-ssl--schannel-ssp-)
- [microsoft-dev](https://devblogs.microsoft.com/premier-developer/microsoft-tls-1-3-support-reference/)

---
