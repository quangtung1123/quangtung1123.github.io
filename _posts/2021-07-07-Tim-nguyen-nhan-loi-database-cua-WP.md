---
layout: post
comments: true
title:  "DDOS - Tìm nguyên nhân lỗi Error establishing a database connection của Wordpress"
title2:  "Tìm nguyên nhân lỗi Error establishing a database connection của Wordpress"
date:   2021-07-07 21:45:00
permalink: 2021/07/07/Tim-nguyen-nhan-loi-database-cua-WP
mathjax: false
tags: Security DDOS Linux WordPress
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Tim-nguyen-nhan-loi-database-cua-WP/Hinh0.jpg
summary: Tìm nguyên nhân lỗi Error establishing a database connection của Wordpress
---

Website đang chạy bình thường, bỗng nhiên xuất hiện lỗi “Error establishing a database connection”, lỗi này cho thấy WordPress không thể kết nối được Database, trong nhiều trường hợp, chỉ cần restart lại service MySQL sẽ hoạt động lại bình thường.

Có nhiều nguyên nhân dẫn đến lỗi này, hôm nay mình chia sẻ với các bạn 1 case thực tế mình vừa gặp sáng nay và cách xác định nguyên nhân chính dẫn đến lỗi để có hướng khắc phục triệt để.

Khi bị lỗi, vào kiểm tra thấy dịch vụ MySQL đã bị stop, phóng ngay lệnh “dmesg” để xem log của hệ thống, phát hiện OOM Killer (Out Of Memory Killer) đã kill process MySQL:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Tim-nguyen-nhan-loi-database-cua-WP/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Trước mắt, đã xác định được nguyên nhân MySQL bị stop là do OOM Killer. Hãy nhìn kỹ vào thông báo trên, một số thông tin quan trọng cần phải để ý:
- Out of memory: Kill process 27657 (mysqld): Confirm lại chính OOM đã kill mysqld khi hệ thống bị hết RAM.
- Số lượng process httpd được sinh ra rất nhiều: đây chính là nguyên nhân gây hết RAM.
- Process httpd được sinh ra bởi user có UID là 501, đây là thông tin cực kỳ quan trọng

Tìm user có UID 501:
```
$ grep 501 /etc/passwd
```
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Tim-nguyen-nhan-loi-database-cua-WP/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Khi đã xác định được user, việc tiếp theo là cần check httpd log của user trên (do số lượng process httpd của user trên vào thời điểm lỗi nhiều bất thường). Tuy nhiên, do dmesg log không ghi lại thời gian cụ thể, cần check tiếp message log
```
$ tail -n 200 /var/log/message
```
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Tim-nguyen-nhan-loi-database-cua-WP/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Thời gian được xác định là “05:08:08”, đây là thời điểm OOM được kích hoạt, vậy cần phải khoanh vùng các truy cập vào Website của user admin trong khoảng thời gian từ “05:08:08” trở về trước

Sau khi check log cách đó khoảng 1p30s thì phát hiện:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Tim-nguyen-nhan-loi-database-cua-WP/Hinh4.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Vào thời điểm trên, có 1 loạt log nhìn rất bất thường, vì sao?
- Các truy cập vào chung 1 URL có pattern là "*/?add_to_wishlist=xxxx*"
- Truy cập xuất phát từ nhiều IP khác nhau nhưng lại chung 1 User-Agent
- Sử dụng cùng HTTP 1.0
- Các IP trên khi load URL KHÔNG load kèm theo các static resources như css, js

Nhiêu đây thông tin đã giúp tôi hình dung ra được cách thức cũng như mục tiêu của sự cố, theo bạn thì website này đang bị gì? Mục tiêu và cách thức ra sao?

Hãy cho tôi biết kết quả của bạn nhé, giờ tôi phải start lại dịch vụ cho khách hàng và điều chỉnh config để giải quyết cho họ rồi :D.

Nguồn: [Vietnix](https://blog.vietnix.vn/tip-19-truy-tim-nguyen-nhan-loi-error-establishing-a-database-connection-cua-wordpress.html)

---
