---
layout: post
comments: true
title:  "DDOS - Bảo mật và phòng chống Ddos cho website WordPress (phần 4)"
title2:  "Bảo mật và phòng chống Ddos cho website WordPress (phần 4)"
date:   2021-05-04 15:10:00
permalink: 2021/05/04/Phong-chong-Ddos-website-WP-p4
mathjax: false
tags: Security DDOS WordPress
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Phong-chong-Ddos-website-WP-p4/Hinh1.jpg
summary: Bảo mật và phòng chống Ddos cho website WordPress
---

## Bảo Mật và phòng chống DDOS cho Website WordPress: Dùng Wordfence

Tôi muốn chia sẻ thêm với các anh em về vấn đề bảo mật cho Website WordPress khi Cloudflare bị đục nhằm hạn chế mức độ của các cuộc tấn công. Tất nhiên, tôi không thể chặn toàn bộ nhưng giảm rất nhiều. Ở phần 01, Tôi chia sẻ cách dùng plugin bảo mật cơ bản cho WordPress với Wordfence.

## 1. Wordfence

Wordfence là một Plugin cung cấp một giải pháp cơ bản và nâng cao hỗ trợ bảo mật cho WordPress. Cái mà mình thích khi sử dụng WordFence là nó cho phép thiết lập các quy tắc nâng cao để ngăn chặn các cuộc tấn công DDOS như:
- Chặn IP
- Chặn nếu IP tấn công vào 1 URL theo một quy tắc nào đó
- Chặn truy cập vào 1 URL bất kỳ
- Chặn nếu cố gáng đăng nhập
- Chặn các cuộc tấn công cơ bản thông qua các kiểu tấn công như xss, sqli, xxe, ...
- Yêu cầu đăng nhập 2 bước dùng Google Authenticator
- Yêu cầu đặt mật khẩu khó ...

Một cái nữa mà Wordfence cung cấp là tính năng Live Traffic giúp tôi có thể phân tích trực tiếp những truy cập vào Website mà đôi khi không cần đọc Log Server. Tôi phải dùng thêm Wordfence vì Cloudflare vẫn để lọt các truy vấn tấn công.

## 2. Thiết lập Firewall và chặn các đợt tấn công nếu lọt qua Cloudflare Firewall

### 2.1 Chặn các truy cập với URL String Random

Ở bài viết trước, tôi đã thiết lập các ruler để chặn các truy vấn với các URL String Random mà Tôi nghi ngờ khi kết thúc các URL thường có các ký tự ngẫu nhiên kiểu như: ?B, ?C, ?1, ?2, ... Trong mục Firewall (Ảnh 1) tôi chọn All Firewall Option:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh1.jpg" width = "800">
</div>
<div class="thecap">Hình 1</div>
</div>
<hr>

Và trong phần Immediately block IPs that access these URLs tôi sẽ nhập vào các ruler sau đây:

```
/?A*
/?B*
/?C*
/?D*
/?E*
/?G*
/?H*
/?I*
/?J*
/?K*
/?L*
/?M*
/?N*
/?O*
/?Q*
/?R*
/?T*
/?V*
/?X*
/?Y*
/?Z*
/?0*
/?1*
/?2*
/?3*
/?4*
/?5*
/?6*
/?7*
/?8*
/?9*
/?b*
/?c*
/?e*
/?g*
/?h*
/?j*
/?l*
/?m*
/?n*
/?o*
/?q*
/?v*
/?x*
/?y*
/?z*
```

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh2.jpg" width = "800">
</div>
<div class="thecap">Hình 2</div>
</div>
<hr>

### 2.2 Block nếu cố login vào các username admin không tồn tại.

Tôi xem xét log truy cập và thấy hay bị các truy cập thử login trái phép vào các username như Admin, Administrator, login, mod, moderator... Nên thêm các user này vào và lock ngay lập tức vì các user đó không tồn tại trên Website.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh3.jpg" width = "800">
</div>
<div class="thecap">Hình 3</div>
</div>
<hr>

### 2.3 Thiết lập khác

Bạn nên thiết lập bổ sung các mục mà tôi check trong ảnh 3 như yêu cầu user khi đăng ký phải chọn mật khẩu mạnh, ... và tránh password admin bị leak.

### 2.4. Block các Referer giả mạo

Ở phần trước, tôi cũng setup block các Referer từ FBI, CIA, ... Nhưng cloudflare không hiểu lý do gì vẫn để lọt được các truy cập vào Server nên tôi áp dụng cả các quy tắc này vào trong Wordfence. Tôi chặn các Referer nguy hiểm trong hình 04 bằng cách vào Firewall -> Blocking, Chọn Custom pattern và nhập vào như sau:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh4.jpg" width = "800">
</div>
<div class="thecap">Hình 4</div>
</div>
<hr>

Mục Referer nhập vào các Url chỉ bao gồm domain và các chuỗi liên quan tới Website của Tôi với 2 dấu \* ở hai đầu để chặn triệt để. 

Ví dụ với baidu, full url referer của nó có https://help thì tôi bỏ cái đó đo và thay bằng dấu \* vì nó có thể thay cái help bằng bất cứ cái gì.

Ở đằng sau có keywords=www.example.net/ nó có thể là domain của tôi hoặc thay bằng một url khác có trên Website của tôi, nên đăng sau dấu / Tôi thêm dấu \* để chặn luôn nếu là 1 URL khác có trên Website của tôi.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh5.jpg" width = "800">
</div>
<div class="thecap">Hình 5</div>
</div>
<hr>

### 2.5 Block Browser User Agent

Cũng trong mục Blocking, tôi Block 15 Browser User Agent tấn công mạnh nhất vào Website của tôi như ảnh 06.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh6.jpg" width = "800">
</div>
<div class="thecap">Hình 6</div>
</div>
<hr>

Có 2 Browser User Agent tấn công hàng trăm triệu Request vào Website của tôi là:
- Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36
- Ubuntu Chromium/34.0.1847.116 Chrome/34.0.1847.116 Safari/537.36

Bạn có thể xem cách tạo ruler block ở ảnh 07
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh7.jpg" width = "800">
</div>
<div class="thecap">Hình 7</div>
</div>
<hr>

### 2.6 Scan Website

Wordfence có một tính năng hữu ích khác là Scan Website. Nhờ việc Scan và xem lại các thư mục thủ công mà tôi phát hiện được một số Backdoor trên Website. Wordfence sẽ báo cho bạn các vấn đề như ở ảnh 08.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh8.jpg" width = "800">
</div>
<div class="thecap">Hình 8</div>
</div>
<hr>

## 3. Live Traffic

Đây là phần hỗ trợ tôi thêm trong việc phân tích trực quan các nguồn truy cập mà không cần thông qua việc xem log Server. Trong phần này, tôi lựa chọn All Traffic như ảnh 09 vì nếu chỉ chọn Security Only, các truy cập vào các URL với Random Query String có thể sẽ bị bỏ qua.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh9.jpg" width = "800">
</div>
<div class="thecap">Hình 9</div>
</div>
<hr>

Các truy vấn với Random Query String đã lọt qua được cloudflare và bị chặn bởi Wordfence chính xác bạn có thể xem ở ảnh 10.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh10.jpg" width = "800">
</div>
<div class="thecap">Hình 10</div>
</div>
<hr>

## 4. Lời kết

Sau hơn 1 tháng bị tấn công dữ dội và cương độ ngày càng tăng, Website của tôi vẫn có thể trụ vững với CPU ổn định ở dưới 5%. Hiện tại, chúng đã tăng cường độ tấn công từ 200 triệu Request mỗi ngày lên gần 700 triệu Request mỗi ngày, nhưng Web vẫn có thể truy cập với tốc độ dưới 1 giây. Bạn có thể xem Request trong 24 giờ qua ở ảnh 10.1.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh10.1.jpg" width = "800">
</div>
<div class="thecap">Hình 10.1</div>
</div>
<hr>

Với mức độ tấn công cao như vậy, nên tôi phải Buy Wordfence với giá $99/month và thấy là nó đáng giá. Trong vòng 30 ngày qua, Website của tôi đã chịu khoảng hơn 9 tỷ Request với khoảng 17TB dữ liệu truyền tải bạn có thể xem ở ảnh 11.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p4/Hinh11.jpg" width = "800">
</div>
<div class="thecap">Hình 11</div>
</div>
<hr>

Nguồn: [Tô Triều](https://www.facebook.com/groups/hieupcwithfriends/permalink/2868244380120604/)

---
