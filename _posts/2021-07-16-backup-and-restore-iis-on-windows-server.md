---
layout: post
title:  "IIS - Sao lưu và khôi phục cấu hình IIS trên Windows Server"
date:   2021-07-16 15:10:00
permalink: 2021/07/16/backup-and-restore-iis-on-windows-server
tags: IIS Windows_Server
category: IIS
img: /assets/backup-and-restore-iis-on-windows-server/Hinh0.png
summary: Sao lưu và khôi phục cấu hình IIS trên Windows Server
---

Trên máy chủ nguồn, thực hiện các công việc sau:

1. Sao lưu tất cả các tập tin của trang web.
2. Xuất bất kỳ chứng chỉ SSL nào mà bạn đã cài đặt.
3. Sao lưu cấu hình IIS.

Thực hiện các công việc sau trên máy chủ đích:

1. Khôi phục tất cả các tập tin của trang web.
2. Nhập bất kỳ chứng chỉ SSL nào được yêu cầu.
3. Khôi phục cấu hình IIS.

Có một số yêu cầu chính mà bạn phải xem xét:

1. Cả hai máy chủ phải chạy cùng một phiên bản IIS.
2. Cấu hình của IIS phải giống nhau trên cả hai máy chủ.
3. Các bước được mô tả ở trên phải được thực hiện theo thứ tự đã nêu, tức là các chứng chỉ phải được nhập trước khi cấu hình IIS được khôi phục.
4. Nếu bạn không chạy máy chủ web như một phần của miền Active Directory, bạn phải đặc biệt chú ý đến các tài khoản người dùng được cấu hình. Cụ thể là tài khoản người dùng  application pool phải có quyền truy cập vào các tệp nguồn trang web cần thiết.

Tôi đã triển khai một trang web HTML rất đơn giản để chứng minh điều này vì vậy tôi sẽ chỉ thực hiện bước 1 và 3 trên mỗi máy chủ vì tôi chưa cài đặt chứng chỉ SSL. Trang web demo mà tôi đang sử dụng có thể được tải xuống từ [Initializr](http://www.initializr.com/). Nếu bạn chưa sử dụng tài nguyên này trước đây, hãy kiểm tra nó vì nó là một cách tuyệt vời để bắt đầu một trang web nhanh chóng và hiệu quả.

## Sao lưu và khôi phục các tệp của trang web ##

- Mở thư mục chứa các tệp của trang web
- Nén tất cả các tệp để di chuyển dễ dàng hơn
- Sao chép và giải nén tất cả các tệp trên máy chủ đích.

## Sao lưu và khôi phục cấu hình IIS ##

- Di chuyển đến thư mục C:\Windows\System32\inetsrv trên máy chủ nguồn và chạy lệnh sau để tạo bản sao lưu. Hãy nhớ chạy với tư quyền Administrator.

```
cd C:\Windows\System32\inetsrv
appcmd thêm sao lưu “<backup name>”
```

<div class="imgcap">
<div >
    <img src="/assets/backup-and-restore-iis-on-windows-server/B1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Mở thư mụcC:\Windows\System32\inetsrv\backup sẽ tìm thấy các tệp sao lưu được lưu trữ trong một thư mục. Nén thư mục để di chuyển dễ dàng hơn và sao chép nó vào cùng một vị trí trên máy chủ đích.

<div class="imgcap">
<div >
    <img src="/assets/backup-and-restore-iis-on-windows-server/B2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
 
- Trên máy chủ đích, giải nén các tệp cấu hình đã sao lưu.

<div class="imgcap">
<div >
    <img src="/assets/backup-and-restore-iis-on-windows-server/B3.png" width = "800">
</div>
<div class="thecap"></div>
</div>
 
- Mở command prompt với quyền Administrator và di chuyển đến đến C:\Windows\System32\inetsrv và chạy lệnh sau:

```
cd C:\Windows\System32\inetsrv
appcmd list backup
```

<div class="imgcap">
<div >
    <img src="/assets/backup-and-restore-iis-on-windows-server/B4.png" width = "800">
</div>
<div class="thecap"></div>
</div>
 
Bản sao lưu của bạn sẽ được liệt kê. Bạn cũng có thể thấy một số bản sao lưu hệ thống trong danh sách của mình như tôi có trong ví dụ bên dưới.

Lưu ý rằng bạn có thể tìm thấy các tệp sao lưu hệ thống tại C:\inetpub\history

Bây giờ, tất cả những gì chúng ta cần làm là chạy lệnh khôi phục để hoàn tất và khởi động trang web đã di chuyển của chúng ta.

- Lệnh khôi phục là:

```
appcmd restore backup “<backup name>”
```

Nếu mọi việc suôn sẻ, bạn sẽ thấy một thông báo cho biết máy chủ đã khôi phục thành công "Restored configuration from backup "<backup name>"". 

<div class="imgcap">
<div >
    <img src="/assets/backup-and-restore-iis-on-windows-server/B5.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**

- [chrislazari](https://chrislazari.com/backup-and-restore-iis-on-windows-server-2016/)

---
