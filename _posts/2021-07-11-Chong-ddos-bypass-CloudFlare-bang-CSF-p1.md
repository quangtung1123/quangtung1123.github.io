---
layout: post
comments: true
title:  "DDOS - Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall (phần 1)"
title2:  "Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall (phần 1)"
date:   2021-07-11 21:45:00
permalink: 2021/07/11/Chong-ddos-bypass-CloudFlare-bang-CSF-p1
mathjax: false
tags: Security DDOS CloudFlare
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p1/Hinh0.png
summary: Hướng dẫn chống DDoS bypass CloudFlare
---

**Với sự phát triển của các dịch vụ tấn công DDoS (DDoS for Hire) như hiện nay, chỉ với một chi  phí nhỏ, bất kỳ ai cũng có thể triển khai một cuộc tấn công DDoS để triệt hạ mục tiêu với các phương thức tấn công được weaponized mà không cần hiểu biết nhiều về kỹ thuật.**

Tấn công thì đơn giản nhưng việc chống tấn công DDoS thì hết sức phức tạp, đỏi hỏi rất nhiều kiến thức và kinh nghiệm. Phần lớn khi bị tấn công đều được khuyên đưa website chạy qua CloudFlare để chặn DDoS. Tuy nhiên, không phải trường hợp nào đưa qua CloudFlare cũng hiệu quả, và các công cụ tấn công DDoS hiện nay đều được tích hợp phương thức để bypass CloudFlare. Trong loạt bài viết này, Vietnix sẽ chia sẻ về:
- Cách thức hoạt động của CloudFlare
- Vì sao CloudFlare bị bypass?
- Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall

## **Cơ chế hoạt động của CloudFlare**

CloudFlare là một Reverse Proxy đứng giữa người dùng (User) và Server chạy website của bạn (được gọi là Backend hoặc Origin Server), hoạt động ở Layer 7 (Application Layer). Các request của User sẽ được web server của CloudFlare tiếp nhận, phân tích và xử lý. Nếu CloudFlare cho rằng request đó là hợp lệ thì CloudFlare sẽ kết nối về Backend để lấy data và trả kết quả (response) về cho người dùng. Do đó:
- Chỉ các HTTP request hợp lệ mới được đẩy về backend, nghĩa là User (hoặc botnet) phải tạo được HTTP request (hoàn thành được 3 Way Handshake, tạo được ESTABLISHED socket) gửi đến CloudFlare.
- Do CloudFlare là đối tượng thực hiện kết nối với Backend để lấy data nên ở phía Backend, **source IP** của các kết nối là **IP của CloudFlare** chứ không phải IP của User (hoặc botnet). Vậy nên, các network firewall hoạt động ở Layer 3 & Layer 4 (ví dụ iptables) ở phía backend không có tác dụng trong quá trình chống DDoS nếu server của bạn nằm sau CloudFlare.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p1/Hinh1.jpg" width = "800">
</div>
<div class="thecap">IP nhận được phía Backend là IP của CloudFlare</div>
</div>
<hr>
- Nếu bạn đang bị tấn công DDoS ở tầng Network (UDP Flood, SYN Flood …) **trực tiếp** vào IP của Backend thì việc chuyển domain qua CloudFlare không giải quyết được vấn đề, trừ khi bạn đổi IP mới cho Backend và giữ bí mật được IP này.
- Các tấn công DDoS ở tầng Network vào IP của CloudFlare hiển nhiên sẽ không ảnh hưởng đến Backend vì chúng chưa lên đến tầng Application và chưa tạo được HTTP request hợp lệ.

Xem thêm: CloudFlare là gì & Cách hoạt động của CloudFlare

## **Cách DDoS bypass CloudFlare**

Mặc định, CloudFlare [không cache HTML](https://support.cloudflare.com/hc/en-us/articles/200172516-Understanding-Cloudflare-s-CDN) mà chỉ cache các [static content](https://support.cloudflare.com/hc/en-us/articles/200172516-Understanding-Cloudflare-s-CDN#h_a01982d4-d5b6-4744-bb9b-a71da62c160a), trừ khi chúng ta setup Page Rule để "Cache Everything". Nếu chưa setup page rule, các request DDoS do botnet gửi lên sẽ được CloudFlare đẩy toàn bộ về Backend.

Cache Everything kết hợp các thông số như: zone_id, scheme, hostname, **request_uri** làm [cache_key](https://support.cloudflare.com/hc/en-us/articles/115004290387-Creating-Cache-Keys).

Ví dụ: có ta có URL "https://example.vn/?fbclid=IwAR0nKKu3", giá trị tương ứng của các biến sẽ như sau:
- $scheme – https
- $hostname – example.vn
- $uri – /
- $is_args – ?
- $args – fbclid=IwAR0nKKu3
- $request_uri – /?fbclid=IwAR0nKKu3
- zone_id là 1 giá do CloudFlare định danh cho mỗi domain, giá trị này được lấy trên CloudFlare (thông qua web hoặc API).

```
 => cache key = md5sum($zone_id:$scheme://$hostname$request_uri) = md5sum(123abc:https://example.vn/?fbclid=IwAR0nKKu3) = 258580a80ba7e32b9c46dad524877118
```

CloudFlare sử dụng Nginx làm nền tảng để phát triển web server của họ, do đó, ta có thể xem cơ chế xử lý cache của CloudFlare tương tự như Nginx. cache_key sẽ được hash (md5sum) thành một string dùng để phân biệt các cache entry với nhau. Nếu 2 request có cùng 1 kết quả hash thì sẽ được phục vụ từ cùng 1 file cache. Trường hợp file cache chưa tồn tại (cache MISS) thì CloudFlare sẽ request về Backend để lấy data.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p1/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Xem thêm: Cơ chế hoạt động & Hướng dẫn cấu hình NGINX cache

Mục tiêu DDoS bypass CloudFlare là đẩy càng nhiều request về Backend càng tốt để làm quá tải nó. Bằng cách làm cho kết quả hash giữa các request khác nhau sẽ gây ra tình trạng **cache MISS** và các request sẽ được đẩy hết về backend.

Nhìn lại cấu trúc cache key mặc định của CloudFlare, các tham số như zone_id, hostname là cố định, scheme thì chỉ có 2 lựa chọn http hoặc https, biến còn lại [request_uri](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_uri) bao gồm URI + Query String, cả 2 tham số này đều dễ dàng bị thay đổi bởi User. Bằng việc Random URI hoặc Query String, hệ thống cache của CloudFlare sẽ bị bypass. CloudFlare cho phép chúng ta custom cache key (sử dụng $uri làm key, bỏ qua $query_string) nhưng tính năng này chỉ available cho các khách hàng sử dụng gói Enterprise (giá 200$/tháng).

## **Các phương thức bypass CloudFlare được sử dụng phổ biến hiện nay**

- **Random URL**: các attacker sẽ gửi lượng lớn request với các URL random để bypass cache. Về phía backend, tuy các URL này sẽ không hợp lệ và trả về 404, nhưng để xác định URL không tồn tại, mã nguồn cũng tốn tài nguyên để xử lý, truy vấn database (đặc biệt với các mã nguồn sử dụng route controller như wordpress…). Hơn nữa, nếu giao diện trang 404 của mã nguồn có liên quan đến các thông tin cần truy vấn trong database sẽ nhanh chóng làm máy chủ quá tải.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p1/Hinh3.png" width = "800">
</div>
<div class="thecap">DDoS bypass CloudFlare – Random URL</div>
</div>
<hr>
- **Random Query String**: cách thường được dùng nhất do độ hiệu quả cao. Attacker sẽ gửi request đến các URL hợp lệ và kèm theo random Query String. Việc random Query String thường không ảnh hưởng đến kết quả load trang (nhất là trang chủ), nhưng nó giúp bypass hoàn toàn hệ thống cache của CF, kèm theo việc Backend phải xử lý một page hợp lệ.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p1/Hinh4.png" width = "800">
</div>
<div class="thecap">DDoS bypass CloudFlare – Random Query String</div>
</div>
<hr>
- **HTTP POST method**: các POST request sẽ được đẩy về backend hoàn toàn vì mặc định không cache POST method. Có thể kết hợp thêm random URL + random Query String
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p1/Hinh4.png" width = "800">
</div>
<div class="thecap">DDoS bypass CloudFlare – POST method</div>
</div>
<hr>
- Botnet có khả năng **solve js challange**: cách này phụ thuộc vào tính năng của botnet hoặc công cụ/dịch vụ DDoS nơi attacker sử dụng. Các botnet này có khả năng xử lý JS challenge của CloudFlare, qua đó, CloudFlare xem chúng là những người dùng hợp lệ và cho vào hệ thống.
- DDoS bypass CloudFlare **low rate**: sử dụng botnet với tần số trên mỗi botnet thấp để bypass WAF và rate limit.
- DDoS bằng cách **GET static content** có dung lượng lớn kết hợp random query string để bypass cache CF, mục tiêu làm tiêu hao Bandwidth phía Backend Server.

Tuy nhiên, điều quan trọng nhất, để tạo được request, các botnet phải thành công trong quá trình thiết lập kết nối ở tầng Network (qua được quá trình bắt tay 3 bước), và chỉ có những client thực sự – có real IP – không phải spoofed IP mới qua được bước này (mình sẽ có 1 bài chi tiết nói về vấn đề này, nó quan trọng vì sẽ quyết định cách tiếp cận chống DDoS của chúng ta). Vậy nên, số lượng IP của botnet là **hữu hạn**, hướng tiếp cận của chúng ta là xác định các IP của botnet đưa chúng vào Firewall. Khi DDoS bypass CloudFlare, muốn đánh sập backend, các botnet cần tạo ra tần số đủ lớn, vượt qua khỏi khả năng chịu tải của backend:
- Nếu **botnet đủ lớn**, để đạt được tần số mong muốn, kẻ tấn công sẽ chỉ cần rate (tần số) request trên mỗi IP botnet **thấp**.
- Nếu **botnet ít**, chúng sẽ cần tần số request trên mỗi IP cao.

Đây là cơ sở quan trọng nhất để chúng ta xác định đâu là botnet và đâu là người dùng hợp lệ: rate limit

Một mindset cực kỳ quan trọng mà Hưng đã đúc kết qua gần 10 năm chống lại hàng ngàn cuộc tấn công DDoS mà các bạn khi mới tiếp cận thường mắc phải: Chúng ta **không cần chặn 100% botnet**, mục tiêu cuối cùng là giữ server **hoạt động ổn** định và phục vụ người dùng hợp lệ một cách **bình thường**. Do đó, không cần phải siết quá chặt để dẫn đến **chặn nhầm** người dùng thật, ảnh hưởng đến trải nghiệm. Nếu để lọt một vài botnet có tần số thấp và truy cập của chúng **không đủ sức gây ảnh hưởng** đến server thì không nhất thiết phải tìm cách chặn, hãy kệ chúng!

## **Rate Limit cho server nằm sau CloudFlare**

Rate limit giúp kiểm soát tần số truy cập của mỗi IP vào server, nếu tần số truy cập lớn hơn quy định thì truy cập đó sẽ bị từ chối và bị ghi log lại. Do web server sẽ tính toán tần số truy cập dựa trên IP nên yêu cầu web server phải nhận biết được IP thật của người dùng. 

**Quan trọng**: đọc kỹ bài viết này để nắm rõ cơ chế hoạt động của Nginx Rate Limiting, chuẩn bị sẵn sàng cho phần cấu hình backend ở Phần 2 của bài viết: Cơ chế hoạt động & Hướng dẫn cấu hình NGINX rate limit

Với trường hợp web server nằm sau CloudFlare, các kết nối backend nhận thấy đều mang source IP của CloudFlare nên không thể áp dụng rate limit lên các IP này. Vị trí tuyệt vời để triển khai rate limit trong mô hình này tất nhiên là CloudFlare, và tin buồn là tính năng này [không miễn phí](https://www.cloudflare.com/rate-limiting/) (of course :D)

Nhưng đừng lo, nếu backend sử dụng Nginx ta sẽ tự triển khai tính năng này. [module http_realip](http://nginx.org/en/docs/http/ngx_http_realip_module.html) sẽ thay thế IP của CloudFlare ở web server (biến **$remote_addr**) thành IP của client và giúp các module khác hoạt động dựa trên biến **$remote_addr** (module allow/deny ip, log …) sẽ hoạt động chính xác như thể server đang kết nối trực tiếp với client, bao gồm module rate_limit.

**Lưu ý**: module trên chỉ có tác dụng đối với logic của Nginx, đối với các ứng dụng khác hoặc ở tầng network thì kết nối vẫn mang source IP của CloudFlare. Vậy làm thế nào để kết hợp CSF – một network Firewall hoạt động ở Layer 3, 4 với CloudFlare để chống tấn công DDoS? Đáp án sẽ có ở phần 2 của bài viết.

## **Lời kết**

Phần 1 chủ yếu tập trung về lý thuyết và hướng tiếp cận để chống DDoS bypass CloudFlare. Phần 2 của bài viết mình sẽ hướng dẫn chi tiết cách cấu hình ở backend. Phần 3 sẽ là những kỹ thuật nâng cao, tool tự động hóa và các cấu hình tối ưu cần thực hiện trên CloudFlare.

Nguồn: [Vietnix](https://blog.vietnix.vn/chong-ddos-bypass-cloudflare-bang-csf-p1.html)

---
