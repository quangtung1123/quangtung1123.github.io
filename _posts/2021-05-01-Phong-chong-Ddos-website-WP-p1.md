---
layout: post
comments: true
title:  "DDOS - Bảo mật và phòng chống Ddos cho website WordPress (phần 1)"
title2:  "Bảo mật và phòng chống Ddos cho website WordPress (phần 1)"
date:   2021-05-01 15:10:00
permalink: 2021/05/01/Phong-chong-Ddos-website-WP-p1
mathjax: false
tags: Security DDOS WordPress
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Phong-chong-Ddos-website-WP-p1/Hinh1.jpg
summary: Bảo mật và phòng chống Ddos cho website WordPress
---

Chào các bạn, mình muốn chia sẻ về việc Website cá nhân đang bị tấn công DDOS dữ dội với nửa tỷ kết nối trong chưa đầy 7 ngày và hôm nay gánh chịu gần 300 triệu Request.

## Thực trạng hiện tại.

Website vẫn đang bị tấn công dồn dập, dù tạm thời thì đã không còn lọt vào được đến Server nữa.
Tổng số request tới Website trong 7 ngày qua là khoảng NỬA TỶ REQUEST.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh1.jpg" width = "800">
</div>
<div class="thecap">Hình 1: Nửa tỉ requests trong 7 ngày</div>
</div>
<hr>

Hệ thống đang sử dụng: WordPress + DirectAdmin + Cloudflare + Sucuri + CSF
## Cách thức tấn công: 

### Gửi các Request tới các URL hợp lệ được sinh các query string ngẫu nhiên.

Đây là kiểu tấn công mạnh nhất trong đợt tấn công này khi chúng liên tục gửi Request tới các url hợp lệ và đằng sau đó sinh ra các query ngẫu nhiên nên gần như lọt qua được cả Sucuri và Cloudflare.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh2.jpg" width = "800">
</div>
<div class="thecap">Hình 2: Request tới các url hợp lệ</div>
</div>
<hr>

Với các truy vấn này, theo mình thấy chúng lọt qua được là do chúng có http referer đến từ các nguồn cực kỳ uy tín như Youtube, Google, FBI, CIA, ...
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh3.jpg" width = "800">
</div>
<div class="thecap">Hình 3: Request có http referer đến từ các nguồn cực kỳ uy tín</div>
</div>
<hr>

### Tấn công qua XML-RPC của WordPress

Đây là cách thức thứ hai và với kiểu tấn công này, chúng cũng liên tục gửi request thông qua file xml-rpc.php của WordPress.

### Tấn công qua FEED WordPress

Đặc điểm của WordPress là có thể lấy FEED cho từng bài, từng chuyên mục, từng Tag nên chúng đã liên tục gửi hơn 100 Request kết hợp với random query string vào FEED URL gây quá tải hệ thống
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh4.jpg" width = "800">
</div>
<div class="thecap">Hình 4</div>
</div>
<hr>

### Tấn công qua 404 URL

HÌnh thức tiếp theo là chúng gửi các Request qua 404 URL tới các URL không tồn tại và để lọt qua được các hệ thống chống DDOS của CF và Sucuri, chúng cũng referer từ các nguồn có độ Trust cao cùng với việc sinh các Random Query String.

Nếu bạn từng truy cập từ Facebook vào Website thông qua link, bạn sẽ thấy ở đằng sau có thêm ?fb=....... Thì viẹc chúng sinh random query string cũng tương tự. Nghĩa là nếu bạn viết ruler cho các String này thì có thể bạn cũng sẽ block luôn các nguồn đáng tin thậm chí các campain chạy Ads.

### Tấn công qua việc tải các hình ảnh và File tĩnh

Cách tiếp theo mà chúng tấn công là qua các File tĩnh. Bằng cách nào đó, chúng liệt kê được hầu hết các File ảnh có trên thư mục uploads của Wordpress và các File tĩnh sau đó liên tục gửi Request để tải về.

Bạn có thể xem thiệt hại qua file tĩnh chúng request qua CDN trong chưa đầy 9h lên tới 405GB.

## GIẢI PHÁP PHÒNG CHỐNG LÀ GÌ?

Giải pháp ở đây là mình đang sử dụng bộ 3: Tối ưu hoá Website + Cloudflare + Sucuri + CSF

### Cloudflare

Cloudflare được sử dụng là lớp đầu tiên vì mình đang sử dụng Gói Pro $20/month nên có thêm nhiều tính năng.

- Bật WAF: bạn bắt buộc phải bật WAF vì trong đó có nhiều ruler và có 1 Option riêng hỗ trợ WordPress chống DDOS. 
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh4.1.jpg" width = "800">
</div>
<div class="thecap">Hình 4.1</div>
</div>
<hr>

- Thiết lập các Rulers
- Challenge toàn bộ các kết nối đến từ các quốc gia ngoài Việt Nam. Nghĩa là nếu kết nối đến từ bên ngoài sẽ có một phần check capcha trước khi lọt vào được Website.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh4.2.jpg" width = "800">
</div>
<div class="thecap">Hình 4.2</div>
</div>
<hr>

- Block toàn bộ request đến từ các quốc gia nguy hiểm bao gồm: Trung Quốc, Ấn Độ, Indonesia, Nga, Mexico, Iran, Iraq
- Block toàn bộ các ASNs nguy hiểm như Ponnynet, Chinanet Backbone
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh6.jpg" width = "800">
</div>
<div class="thecap">Hình 6</div>
</div>
<hr>

- Cho phép các Bot quan trọng truy cập nếu bạn không muốn rớt thứ hạng SEO trên Google vì chặn các Bot khiến chúng không thể truy cập được.
- Lọc và chặn các IP tấn công trực tiếp vào hệ thống thông qua Access Log trên Server.

## Sucuri

Sucuri có Option hoạt động phía sau Cloudflare như lớp phòng chống thứ hai. Gói trả phí $10/tháng. Sucuri cho phép bật chế độ chống DDOS khẩn cấp giống Cloudflare. Hiểu đơn giản là các kết nối sẽ được thực thi qua Cloudflare, sau đó xuống lớp lọc của Sucuri cuối cùng mới lọt vào Server. Sucuri có realtime trên hệ thống của họ cho biết map + nguồn đang truy cập và chọn Block luôn hay không. 
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh6.1.jpg" width = "800">
</div>
<div class="thecap">Hình 6.1</div>
</div>
<hr>

Sucuri đồng thời cũng hỗ trợ chặn kết nối qua XMLRPC của WordPress và nhiều thứ hay ho khác nữa. Sucuri cũng hỗ trợ Block thông qua http referer nên mình block luôn các referer qua FBI, CIA, WhiteHouse, ...

### CSF Firewall

Mình chọn thiết lập tiêu chuẩn và chặn các kết nối từ các quốc gia nguy hiểm như Trung Quốc, Ấn Độ, Indonesia, Nga, Mexico, Iran, Iraq, .. mà chúng lọt qua được Cloudflare và Sucuri.

### Tối ưu hoá Website.

Đây là bước quan trọng, mình đã tối ưu hoá Website cực kỹ. Hình ảnh dùng định dạng Webp rất nhẹ. Tổng load Website chỉ tầm 300kb. Số Request trực tiếp tới Server cực ít (Tầm 19-21 Request)

**Kết quả**: Trong tình trạng Server luôn bị quá tải ở mức 203% (Hình 7) thì giờ đây, khi đang gánh chịu gần 300 triệu Request, CPU đã ổn đỉnh ở mức dưới 10% (Hình 8) và Website chạy phà phà.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh7.jpg" width = "800">
</div>
<div class="thecap">Hình 7</div>
</div>
<hr>

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p1/Hinh8.jpg" width = "800">
</div>
<div class="thecap">Hình 8</div>
</div>
<hr>

Nguồn: [Tô Triều](https://www.facebook.com/groups/hieupcwithfriends/permalink/2845799252365117/)

---
