---
layout: post
comments: true
title:  "DDOS - Bảo mật và phòng chống Ddos cho website WordPress (phần 2)"
title2:  "Bảo mật và phòng chống Ddos cho website WordPress (phần 2)"
date:   2021-05-02 15:10:00
permalink: 2021/05/02/Phong-chong-Ddos-website-WP-p2
mathjax: false
tags: Security DDOS WordPress
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Phong-chong-Ddos-website-WP-p2/Hinh1.jpg
summary: Bảo mật và phòng chống Ddos cho website WordPress
---

# Thiết lập Rulers Chống DDOS với Cloudflare

Như đã chia sẻ trong bài viết trước đó, Website của Tôi bị tấn công DDOS nặng nề khiến cho hầu hết các nhà cung cấp dịch vụ tại Việt Nam phải từ chối phục vụ và hoàn tiền.
Hôm nay, Sau hơn 30 ngày và Website vẫn sống khoẻ cùng với việc theo dõi khối lượng tấn công tăng vọt trên Cloudflare nhưng Website vẫn ổn định và CPU chỉ ở mức dưới 5%.
Tôi viết bài này chia sẻ với các bạn chút hiểu biết nông cạn mà tôi đã ứng dụng để chống lại đợt tấn công quy mô lớn này với gần 7 tỷ request trong chưa tới 30 ngày và trong 2 ngày gần nhất, mỗi ngày lượng request đã tăng gấp đôi, 500tr request mỗi ngày.
Tổng dữ liệu qua Cloudflare gánh là 14TB.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh1.jpg" width = "800">
</div>
<div class="thecap">Hình 1</div>
</div>
<hr>

## 1. CLOUDFLARE PRO
Tôi phải mua Cloudflare Pro $20/tháng để có thêm các tính năng chống DDOS và Web Application Firewall với nhiều Ruler mà Cloudflare thiết lập cho WordPress và các kiểu tấn công DDOS khác.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh2.jpg" width = "800">
</div>
<div class="thecap">Hình 2</div>
</div>
<hr>

### Các Ruler riêng

Tôi sử dụng tổng cộng 9 Ruler như hình ảnh số 03 các bạn có thể xem qua sau đó chúng ta tiếp tục với các Ruler Firewall cụ thể.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh3.jpg" width = "800">
</div>
<div class="thecap">Hình 3</div>
</div>
<hr>

## 2. CHẶN TẤN CÔNG, KHÔNG ĐƯỢC CHẶN BOT TÌM KIẾM VÀ CÁC DỊCH VỤ BỔ TRỢ.

Tôi sử dụng WordPress vì vậy Jetpack là một phần không thể thiếu cùng với Sucuri, Wordfence và đặc biệt là các Bot tìm kiếm vì Website của tôi có thứ hạng rất cao trên Google. Chính vì vậy tôi cho phép các request từ dải IP của jetpack và các dịch vụ cần thiết được gửi thẳng đến Server mà không qua tường lửa để không làm gián đoạn dịch vụ. Đặc biệt, Cloudflare có một Option là Know Bot, bạn phải luôn bật tính năng này để không chặn các công cụ tìm kiếm Index website làm giảm thứ hạng hoặc không xuất hiện trên kết quả tìm kiếm. Để tránh mất thời gian thêm từng dải IP, Tôi chọn cách Allow các ASNs của các bên thứ ba.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh4.jpg" width = "800">
</div>
<div class="thecap">Hình 4</div>
</div>
<hr>

## 3. CHẶN CÁC ASNS NGUY HIỂM

Khi phân tích các Request, Tôi nhận thấy các đợt tấn công đến từ 05 ASNs chủ lực bao gồm:
- AS4134 - CHINANET-BACKBONE - Trung Quốc
- AS14061 - DIGITALOCEAN-ASN Nhà cung cấp dịch vụ VPS lớn nhất nhì thế giới.
- AS53667 - PONYNET mạng Botnet nằm trong lòng nước Mỹ thiên đường
- AS396507 - EMERALD-ONION
- AS22773 - ASN-CXA-ALL-CCI-22773-RDC

Tại Việt Nam, Tôi thấy có 1 lượng tấn công rất lớn từ WEBICO VỚI ASNS LÀ AS135951.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh5.jpg" width = "800">
</div>
<div class="thecap">Hình 5</div>
</div>
<hr>

## 4. CHẶN CÁC REQUEST VỚI RANDOM QUERY STRING TỪ VIỆT NAM

Khi quan sát, Tôi thấy các Ruler có thể sẽ bị bỏ qua nên quyết định tách Vietnam và các quốc gia ngoài Vietnam để thêm vào Ruler cho chắc cú. Chỗ này tôi chưa có thời gian để Logic thêm nên sẽ cải thiện sau. Tôi phân tích và nhận thấy các tấn công có một đặc điểm chung là sau đường dẫn tĩnh, thường được thêm vào một loạt các ký tự đặc biệt với đặc trưng:
- ?1
- ?2
- ?A
- ?a
- ?b
- ?B
...

Nên tôi đã thêm nó vào thành một Ruler và dùng điều kiện hoặc - Và trong Cloudflare để setup như hình số 06
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh6.jpg" width = "800">
</div>
<div class="thecap">Hình 6</div>
</div>
<hr>

Lưu ý trong phần này, bạn không được chặn một số Random Query String bắt đầu với các tham số sau:
- ?a : Nếu dùng AMP của Google thì cuối url có ?amp
- ?f vì các truy cập từ Facebook cũng sinh một loạt các query string ngẫu nhiên bắt đầu bằng ?fbid
- ?u: Vì các campain quảng cáo bắt đầu với ?utm
- ?s vì khi tìm kiếm, WordPress thường bắt đầu với ?s=
- ?p vì trong các url admin của Wordpress thường có tham số ?p như ?page hoặc ?post ...

P/S: Hôm qua sau khi đăng tải, Tôi có nhận được các Inbox góp ý thêm về việc sử dụng URI Path nhưng Tôi chưa thử áp dụng.

## 5. CHẶN REFERER TỪ CÁC NGUỒN LẠ

Khi phân tích Referer, Tôi nhận thấy rất lạ khi có những truy cập với Referer từ FBI, CIA, Whitehouse... Và nhiều nguồn vớ vẩn khác, nên Tôi chặn Referer Vietnam từ các nguồn này như hình số 07
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh7.jpg" width = "800">
</div>
<div class="thecap">Hình 7</div>
</div>
<hr>

Một số kiểu Referer điển hình các bạn sẽ thấy như hình 08
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh8.jpg" width = "800">
</div>
<div class="thecap">Hình 8</div>
</div>
<hr>

**Lưu ý**: Không được chặn Bing và Google vì đó là 2 nguồn chủ lực. Nhưng nếu referer từ các domain từ các quốc gia khác như google.com.ai, google.ai, google.ae... Thì đó chắc chắn là tấn công chứ không phải Bot. Tôi không chặn Referer từ Google.com và google.com.vn còn lại cho nó vào chặn hết.

## 6. CHẶN REFERER TƯƠNG TỰ RULER 05 NHƯNG CHO CÁC QUỐC GIA KHÁC

Chắc bạn sẽ khó hiểu chỗ này, nhưng Tôi không thể chặn Việt Nam và không thể để ruler Challenge vì nó ảnh hưởng tới bạn đọc. Nên Tôi Allow Vietnam và Challenge các quốc gia khác, thành ra nếu Truy cập từ Vietnam có Referer bẩn như trên Cloudflare nó vẫn allow. Nên Tôi phải cho 1 Ruler riêng đặt lên trên Ruler này. Bạn xem hình 09, Referer tương tự nhưng loại bỏ yếu tố quốc gia.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh9.jpg" width = "800">
</div>
<div class="thecap">Hình 9</div>
</div>
<hr>

Trong phần này, ngoài các Referer, Tôi cho luôn cả các Random Query String tương tự mục 04.

## 7. BLOCK CÁC QUỐC GIA TẤN CÔNG MẠNH VÀ CHỦ YẾU LÀ SPAM

Các quốc gia nguy hiểm bị tôi chặn vì thực tế cũng chẳng có truy cập từ đó mà nó tấn công thì ác quá nên Block cho chắc cú.
- Đứng đầu là China
- Ấn Độ
- Afghanistan
- Indonesia
- Ecuado
- Iran
- Iraq
- Israel

Ảnh 10 bạn sẽ thấy cách Setup Ruler chính xác.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh10.jpg" width = "800">
</div>
<div class="thecap">Hình 10</div>
</div>
<hr>

## 8. CHALLENGE VỚI CÁC QUỐC GIA KHÔNG PHẢI VIỆT NAM VÀ KHÔNG BỊ BLOCK

Tôi phải bật tường lửa và sẽ có 1 trang yêu cầu capcha với các quốc gia không phải Vietnam và không bị Block vì Tôi không thể rảnh mà thêm toàn bộ các quốc gia đó vào danh sách Block. Ngoài ra, Tôi vẫn có truy cập tốt từ các quốc gia như Nhật, Hàn, Pháp, Úc nên lựa chọn Challenge là tốt nhất. Sau khi vượt qua Challenge, User có khoảng 4 tiếng mới bị lại là một giải pháp tốt. Bạn có thể xem Ruler ở ảnh 11.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh11.jpg" width = "800">
</div>
<div class="thecap">Hình 11</div>
</div>
<hr>

## 9. CHỈ CHO PHÉP KẾT NỐI TCP QUA CỔNG 80 VÀ 443 THÔNG QUA CÁC IP ĐẾN TỪ CLOUDFLARE

Tôi đang sử dụng VPS tại Vultr và có một tính năng thú vị là Firewall với Option chỉ cho phép kết nối, request từ Cloudflare.
Tôi đã bật tính năng này và link Droplet với ruler Firewall. Bạn có thể xem hình 12
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh12.jpg" width = "800">
</div>
<div class="thecap">Hình 12</div>
</div>
<hr>

Trên đây là toàn bộ chia sẻ của Tôi về việc sử dụng Cloudflare để phòng chống tấn công DDOS mà không nhất thiết phải bật chế độ Under Attack Mode của Cloudflare gây khó chịu cho người sử dụng.
Tính đến hôm nay, sau hơn 30 ngày dồn dập tấn công, thống kê 24 giờ qua, các đợt tấn công dường như đã dừng hẳn và không còn ghi nhận hàng trăm triệu Request nữa.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p2/Hinh13.jpg" width = "800">
</div>
<div class="thecap">Hình 13</div>
</div>
<hr>

Trong các bài viết sau, Tôi sẽ chia sẻ thêm các kiến thức có giới hạn trong tầm hiểu biết của Tôi về việc dò tìm và phát hiện Website WordPress bị hack một cách cơ bản.

Nguồn: [Tô Triều](https://www.facebook.com/groups/hieupcwithfriends/permalink/2863925930552449/)

---
