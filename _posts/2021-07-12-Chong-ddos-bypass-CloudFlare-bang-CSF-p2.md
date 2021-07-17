---
layout: post
comments: true
title:  "DDOS - Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall (phần 2)"
title2:  "Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall (phần 2)"
date:   2021-07-12 21:45:00
permalink: 2021/07/12/Chong-ddos-bypass-CloudFlare-bang-CSF-p2
mathjax: false
tags: Security DDOS CloudFlare
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p2/Hinh3.jpg
summary: Hướng dẫn chống DDoS bypass CloudFlare
---

Ở phần 1 của bài Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall, mình đã giải thích rõ về:
- Mô hình hoạt động của Cloudflare
- Cơ chế cache của CloudFlare, đặc điểm về cache key
- Đặc điểm về source IP nhận được ở phía Backend Server
- Các cách DDoS bypass CloudFlare phổ biến.

Xem lại: [Hướng dẫn chống DDoS bypass CloudFlare bằng CSF Firewall – Phần 1](/2021/07/11/Chong-ddos-bypass-CloudFlare-bang-CSF-p1)

Phần 2 này mình sẽ hướng dẫn cách cấu hình kết hợp NGINX với CSF để chống DDoS cho server nằm sau CloudFlare. Bao gồm các bước:
- Cấu hình NGINX ghi log IP thực của người dùng khi nằm sau CloudFlare (để lấy được IP botnet, sau đó push lên CF chặn)
- Sử dụng NGINX rate limit để giới hạn tần số truy cập (phát hiện đâu là botnet, đâu là người dùng hợp lệ)
- Kết nối CSF với CloudFlare để chặn các IP botnet (push IP botnet lên CloudFlare, không cho đẩy kết nối về Backend)

## **Cấu hình NGINX ghi log IP thật của User khi nằm sau CloudFlare**

Cloudflare là 1 Reverse Proxy hoạt động ở Layer 7 và CloudFlare thay mặt client để kết nối đến backend server lấy data. Do đó:
- Phía backend, source IP (giá trị của biến **$remote_addr** trong NGINX) là IP của CloudFlare.
- IP thực của người dùng (Client IP) được CloudFlare chèn vào header HTTP "**X-Forwarded-For**" hoặc "**CF-Connecting-IP**".

Chúng ta sẽ cấu hình NGINX sử dụng [module http_realip](http://nginx.org/en/docs/http/ngx_http_realip_module.html) để thay thế giá trị của header "**CF-Connecting-IP**" vào giá trị của biến **$remote_addr**. Qua đó, giúp các module hoạt động dựa giá trị của biến **$remote_addr** (ví dụ module log, rate limit) hoạt động dựa trên giá trị IP thực của người dùng.

Xem thêm: Hướng dẫn cài đặt module real ip cho NGINX

Để NGINX nhận diện IP thực của người dùng, thêm danh sách [ipv4](https://www.cloudflare.com/ips-v4) và [ipv6](https://www.cloudflare.com/ips-v6) của CloudFlare vào phần **set_real_ip_from**. Tạo file **/etc/nginx/conf/cloudflare.conf** với nội dung như sau:

```
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 104.16.0.0/12;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 131.0.72.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2c0f:f248::/32;
set_real_ip_from 2a06:98c0::/29;
real_ip_header CF-Connecting-IP;
```

include file "**cloudflare.conf**" vào **nginx.conf** ở context **http**:
```
http {
...
  include "/etc/nginx/conf/cloudflare.conf";
...
}
```

Kiểm tra lại config NGINX:
```
nginx -t
```

Restart nginx
```
systemctl restart nginx
```

Log mặc định của NGINX sử dụng format combined, được định nghĩa như sau: 
```
log_format combined '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';
```

Format này sử biến **$remote_addr** làm IP của client. Sau khi config sử dụng module Real IP, hãy tiến hành kiểm tra access log xem đã ghi log được IP thật của người dùng hay chưa:
```
112.197.118.134 - - [21/Mar/2021:13:17:57 +0700] "GET / HTTP/1.1" 200 10179 "https://www.google.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:86.0) Gecko/20100101 Firefox/86.0"
```

Whois IP để xem IP trên thuộc nhà mạng nào, nếu không phải của CloudFlare là thành công:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p2/Hinh1.png" width = "800">
</div>
<div class="thecap">Thông tin cho thấy log ghi nhận IP của User Việt Nam</div>
</div>
<hr>
NGINX đã nhận biết được IP thực của người dùng thay vì IP của CloudFlare. Các module khác như allow, deny IP, rate limiting đã có thể sử dụng.

## **NGINX chống DDoS bằng cách giới hạn tần số truy cập**

Khi load một URL, ngoài việc load nội dung (content) của URL, browser còn phải load các static content khác để có thể render thành một web page hoàn chỉnh như: image, javascript, css, font, icon … Điều này có nghĩa khi load một URL, client không chỉ tạo một request đến server mà là nhiều request. Do đó, để giới hạn tần số hiệu quả, ta cần tối ưu như sau:
- Để location chứa static content như: image, javascript, css, icon … nằm riêng
- Tách location cho trang chủ (trang /) vì đây là location bị đánh phổ biến nhất.
- Tách location cho các URL bị đánh trong trường hợp khẩn cấp để apply limit phù hợp.

Ví dụ:
```
server {
...
    location ~* \.(swf|ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|css|rss|atom|js|jpg|jpeg|gif|png|ico|tgz|gz|rar|bz2|tar|mid|midi|wav|bmp|rtf|woff|woff2|mo|zip|txt)$ {
        expires max;
    }

    location = / {
        proxy_pass https://127.0.0.1:8080;
    }
﻿
    location / {
        proxy_pass https://127.0.0.1:8080;
    }
}
```

Để cấu hình rate limit, cần thực hiện 2 bước: khai báo **limit_req_zone** và khai báo **limit_req**.

Chi tiết về thuật toán, cách cấu hình, ý nghĩa các tham số tham khảo tại: [Hướng dẫn cấu hình giới hạn tần số truy cập với NGINX](https://vietnix.vn/huong-dan-cau-hinh-nginx-rate-limit/)
- **location static content**: thường thì không cần limit hoặc limit ở rate cao nếu bạn muốn. Vì việc phục vụ các request ở khu vực này thường ko tốn quá nhiều resource (ngoại trừ băng thông)
- **location = /** : location này chỉ phục vụ riêng cho trang index, có thể đặt tần số thấp.
- **location /** : catch all location, phục vụ các request không match các location trước đó, cần để limit tương đối thoáng.

Ở bước 1, ta đã config NGINX nhận được real IP của client, ta có thể dùng biến **$binary_remote_addr** để làm key. 

**Ví dụ**
Giới hạn tần số truy cập như sau:
- Không limit với **location static**
- Limit **location = /** – mỗi IP không được phép thực hiện quá 3 request mỗi giây.
- **location /** : limit mỗi IP không được phép thực hiện quá 10 request mỗi giây.

File virtual host **blog.example.vn.conf** sẽ có nội dung:
```
limit_req_zone $binary_remote_addr zone=location_index:10m rate=3r/s; # khai báo zone cho page index

limit_req_zone $binary_remote_addr zone=location_catchall:10m rate=5r/s; # khai báo zone choh catch all location
﻿
server {
 ...
    server_name blog.example.vn;
    index index.php;
    access_log /var/log/nginx/access_log combined;
    error_log /var/log/nginx/error_log error;
    root /var/www/example.vn/

    location ~* \.(swf|ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|css|rss|atom|js|jpg|jpeg|gif|png|ico|tgz|gz|rar|bz2|tar|mid|midi|wav|bmp|rtf|woff|woff2|mo|zip|txt)$ {
        expires max;
    }
﻿
    location = / {
        limit_req zone=location_index burst=3 nodelay;
        proxy_pass https://127.0.0.1:8080;
    }
﻿
    location / {
        limit_req zone=location_catchall burst=10 delay=7;
        proxy_pass https://127.0.0.1:8080;
    }
}
```

Tác dụng:
- Giới hạn tần số truy cập vào home page (trang index) là 3 request/s. Việc set “burst=3 nodelay” cho phép 3 request đầu tiên được phục vụ ngay lập tức mà không cần chờ. Đồng thời set hàng đợi tối đa là 3 request, nếu vượt qua số lượng này sẽ bị reject.
- Các truy cập vào static resources (file tĩnh) không bị giới hạn.
- Truy cập vào các khu vực còn lại bị giới hạn tần số 5 request/s với 7 request đầu tiên được phục vụ ngay lập tức (delay = 7), các request sau đó (request 8, 9, 10) bị delay để đảm bảo tần số 5 request/s. Hàng đợi chứa được tối đa 10 request, các request vượt quá số lượng này sẽ bị reject.

## **Theo dõi các IP vi phạm**

Các IP truy cập quá tần số quy định sẽ bị ghi log vào file error_log với nội dung như sau:
```
2021/03/29 15:53:24 [error] 8564#0: *24818 limiting requests, excess: 4.544 by zone "location_index", client: 3.140.197.140, server: blog.example.vn, request: "GET / HTTP/1.1", host: "blog.example.vn"
```
Trong đó:
- zone "location_index" – zone name được hai báo ở limit_req_zone
- client: 3.140.197.140 – IP tấn công
- request – URL bị tấn công
- host: domain bị tấn công

Để tránh request của các IP này không truy cập được về backend nữa, ta cần tiến hành block IP của chúng trên CloudFlare. Để tất cả IP tấn công tự động bằng cách theo dõi file error log và tự động add chúng vào Firewall trên CloudFlare, ta sử dụng CSF Firewall

## **Cài đặt CSF Firewall và cấu hình chống DDoS bypass CloudFlare**

Hướng dẫn cài đặt CSF tham khảo tại đây: [Hướng dẫn cài đặt CSF](https://vietnix.vn/cai-dat-csf-centos-7/)

CSF có khá nhiều tính năng hay, mình sẽ làm 1 series bài chi tiết về CSF sau. Trong phạm vi bài viết này, các bạn chỉ cần quan tâm điều chỉnh các thông số được liệt kê bên dưới.

Mở file cấu hình **/etc/csf/csf.conf** và điều chỉnh thành giá trị như sau:
```
# Enable Firewall
TESTING = 1

# Khai báo các TCP Port cho phép client kết nối đến Server
TCP_IN = "22,80,443"

# Khai báo các TCP port cho phép server kết nối ra ngoài
TCP_OUT = "1:65535"

# Khai báo các UDP port cho phép client kết nối đến server
UDP_IN = ""

# Khai bác các UDP port cho phép server kết nối ra ngoài
UDP_OUT = "1:65535"

# Nâng giới hạn Deny IP
DENY_IP_LIMIT = "500"

# Nâng giới hạn số lượng IP bị Temporary Deny
DENY_TEMP_IP_LIMIT = "1000"

# Khai báo sử dụng ipset
LF_IPSET = "1"

# Enable tính năng CloudFlare
CF_ENABLE = "1"

# Khi add IP lên CloudFlare, add vào Block List
CF_BLOCK = "block"

# Thời gian add tạm là 3600s
CF_TEMP = "3600"

# Thay bằng đường dẫn chứa error_log của domain của bạn
CUSTOM1_LOG = "/var/log/nginx/error_log"
```

Cấu hình CSF theo dõi log tấn công và tự động đẩy IP tấn công lên CloudFlare

Thêm dòng sau vào file **/etc/csf/regex.custom.pm**, nằm trước dòng "return 0". CSF sẽ dựa theo regex này để parse file log, lấy ra các IP vượt quá limit bị ghi log:
```
        if (($globlogs{CUSTOM1_LOG}{$lgfile}) and ($line =~ /^\d{4}\/\d{2}\/\d{2} \d{2}:\d{2}:\d{2} \[.+\] .+?: \S+ limiting requests, excess: \S+ by zone \"[^"]+", client: (\S+), server: .*/)) {
                return ("DDoS Attack Detected", $1, "ratelimit", "2", "", "1800", "1")
        }
```

Trong đó:
- DDoS Attack Detected – Nội dung thông báo
- $1 – sẽ được thay thế bằng IP của botnet
- ratelimit – tên của rule (cần phải đặt unique)
- 2 – số lần vi phạm tối đa cho phép, nếu vi phạm vượt quá số lần quy định (IP xuất hiện ở error log nhiều lần) sẽ bị trigger block IP. Ở đây mình để là 2, quá limit rate 2 lần mới bị block
- 1800 – thời gian block (tính bằng giây)
- 1 – Enable việc đẩy IP vi phạm lên block trên firewall của CloudFlare.

## **Cấu hình CSF kết nối API với CloudFlare**

Đăng nhập vào CloudFlare, chọn domain cần cấu hình và chọn phần "**Get API Token**"

Lấy Token: **API Tokens > Global API Key > View**

Copy Token và edit file **/etc/csf/csf.cloudflare**, chèn thêm dòng sau vào:
```
DOMAIN:any:USER:example:CFACCOUNT:info@example.vn:CFAPIKEY:123456789abcdefghijklmnopqr
```

Trong đó,
- USER:example – Đặt tên sao cho dễ nhớ là được.
- CFACCOUNT:info@example.vn – Thay bằng email login CloudFlare
- CFAPIKEY:123456789\[…\] – Thay bằng Token lấy ở trước trên.

Restart csf & lfd để config có hiệu lực
```
 csf -r
 systemctl restart lfd
```

Kiểm tra xem CSF đẩy được IP lên CloudFlare hay chưa:
```
# Add IP 99.99.99.99 lên FW CF
csf --cloudflare add block 99.99.99.99 example

# Liệt kê danh sách IP trên FW CF
csf --cloudflare list all example
```

## **Kiểm tra việc chống DDoS bypass CloudFlare bằng CSF**

Chạy lệnh sau (hoặc dùng các tool benchmark/DDoS) để gửi 20 request lên server và kiểm tra kết quả:
```
for i in $(seq 1 20) ; do curl https://blog.example.vn/ ; done
```

Thay domain bằng domain của bạn.

Theo dõi output hoặc **access_log** trên server. Nếu cấu hình đúng thì các request vượt quá limit sẽ nhận được status code **503**.

Theo dõi **error_log**, IP vi phạm sẽ được log lại
```
2021/03/30 23:03:47 [error] 20800#20800: *1754 limiting requests, excess: 3.318 by zone "location_index", client: 171.252.190.190, server: blog.example.vn, request: "GET / HTTP/1.1", host: "blog.example.vn"
```

Kiểm tra xem CSF đã parse được file error_log và block IP vi phạm hay chưa
```
[root@blog.example.vn nginx]# csf -t 
A/D   IP address                               Port   Dir   Time To Live     Comment
DENY  171.252.190.190                            *    inout 29m 42s          lfd - (ratelimit) DDoS Attack Detected 171.252.190.190 (VN/Vietnam/-): 2 in the last 3600 secs
```

Kiểm tra Firewall trên CloudFlare xem đã có danh sách IP do CSF đẩy lên chưa:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p2/Hinh2.png" width = "800">
</div>
<div class="thecap">Các IP tấn công DDoS được add vào Block list trên CloudFlare</div>
</div>
<hr>
Đến đây là hoàn tất, các IP khi có tần số truy cập cao sẽ được phát hiện và giới hạn truy cập bằng NGINX. IP vi phạm được ghi vào error_log, file này được monitor bởi LFD của CSF và định kỳ lấy các IP vi phạm thỏa điều kiện (vi phạm từ 2 lần trở lên) đẩy lên Firewall trên CloudFlare.

Việc IP được đưa vào Firewall ở CloudFlare sẽ ngăn traffic tấn công đổ về backend, giúp giải tải cho server. Số lượng botnet luôn là con số hữu hạn. Chỉ cần chờ 1 thời gian, Firewall CloudFlare sẽ có đủ list IP của botnet!
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bypass-CloudFlare-bang-CSF-p2/Hinh3.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## **Tổng kết phần 2 Chống DDoS bypass CloudFlare bằng CSF**

Các cấu hình trên chỉ mang tính chất tham khảo. Tùy theo mỗi website mà bạn cần điều chỉnh cấu hình cho phù hợp: các location & rate tương ứng.

Các cấu hình hiện tại chưa tối ưu và đang là mô hình sơ khai nhất để kết hợp CloudFlare và CSF. Phần 3 mình sẽ đi tiếp các nội dung giúp hoàn hiện và giúp Backend chống DDoS tốt hơn:
- Cấu hình mirco cache cho website chạy WordPress để tăng khả năng chịu tải khi bị tấn công.
- Các cấu hình tối ưu thêm trên CloudFlare, NGINX
- Notify telegram các IP botnet bị chặn
- Script thay thế khi không sử dụng CSF

Nguồn: [Vietnix](https://blog.vietnix.vn/chong-ddos-bypass-cloudflare-bang-csf-p2.html)

---
