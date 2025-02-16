---
layout: post
title:  "Bảo mật HTTP Headers để ngăn chặn lỗ hổng Apache, Nginx"
date:   2021-07-29 15:10:00
permalink: 2021/07/29/bao-mat-http-header
tags: Security HTTP Header Apache Nginx
category: Security
img: /assets/bao-mat-http-header/Hinh1.png
summary: Bảo mật HTTP Headers để ngăn chặn lỗ hổng Apache, Nginx

---

> Bạn có biết hầu hết các lỗ hổng bảo mật có thể được khắc phục bằng cách triển khai các tiêu đề cần thiết trong tiêu đề phản hồi không?

Bảo mật cũng cần thiết như nội dung và SEO của trang web của bạn, và hàng nghìn trang web bị tấn công do lỗi cấu hình sai hoặc thiếu bảo vệ. Nếu bạn là chủ sở hữu trang web hoặc kỹ sư bảo mật và đang tìm cách bảo vệ trang web của mình khỏi các cuộc tấn công **Clickjacking**, **code injection**, **MIME**, **XSS**,... thì hướng dẫn này sẽ giúp bạn.

Trong bài viết này, tôi sẽ nói về các HTTP Headers khác nhau (được [OWASP](https://owasp.org/www-project-secure-headers/) khuyến nghị) để triển khai trong nhiều máy chủ web, nhà cung cấp mạng & CDN để bảo vệ trang web tốt hơn.

**Ghi chú:**

- Bạn nên sao lưu tệp cấu hình trước khi thực hiện thay đổi
- Một số header có thể không được hỗ trợ trên tất cả các trình duyệt, vì vậy hãy [kiểm tra tính tương thích](https://caniuse.com/) trước khi triển khai
- Mod_headers phải được kích hoạt trong Apache để triển khai các header này. Đảm bảo dòng sau không nằm trong ghi chú trong tệp httpd.conf:

```
LoadModule headers_module modules/mod_headers.so
```

- Sau khi thực hiện, bạn có thể sử dụng [công cụ trực tuyến](https://gf.dev/secure-headers-test) để xác minh kết quả.

## 1. HTTP Strict Transport Security ##

HSTS header (HTTP Strict Transport Security) để đảm bảo tất cả thông tin liên lạc từ trình duyệt được gửi qua HTTPS (Bảo mật HTTP). Điều này ngăn chặn HTTPS click-through prompts và chuyển hướng các yêu cầu HTTP đến HTTPS.

Trước khi triển khai header này, bạn phải đảm bảo rằng tất cả các trang web của bạn đều có thể truy cập được qua HTTPS nếu không chúng sẽ bị chặn.

HSTS header được hỗ trợ trên tất cả các phiên bản mới nhất của trình duyệt như IE, Firefox, Opera, Safari và Chrome. Có ba tham số cấu hình:

|------------------+---------|
| Giá trị tham số | Ý nghĩa |
|:----------------:|:---------|
| max-age | Khoảng thời gian (tính bằng giây) để thông báo cho trình duyệt biết rằng các yêu cầu chỉ khả dụng qua HTTPS. |
| includeSubDomains | Cấu hình cũng có hiệu lực cho tên miền phụ. |
| preload | Sử dụng nếu bạn muốn tên miền của mình được đưa vào [danh sách tải trước HSTS](https://hstspreload.appspot.com/) |

Vì vậy, hãy lấy một ví dụ về việc cấu hình HSTS trong một năm, bao gồm preload cho tên miền và tên miền phụ.

### Máy chủ Apache HTTP ###

Bạn có thể triển khai HSTS trong Apache bằng cách thêm mục sau vào tệp httpd.conf:

```
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

Khởi động lại apache để xem kết quả

### Nginx ###

Để cấu hình HSTS trong Nginx, hãy thêm mục tiếp theo vào trong nginx.conf dưới phần điều khiển máy chủ (SSL)

```
add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains; preload';
```

Như thường lệ, bạn sẽ cần khởi động lại Nginx để kiểm tra.

### Cloudflare ###

Nếu bạn đang sử dụng Cloudflare, thì bạn có thể bật HSTS chỉ trong vài cú nhấp chuột.

- Đăng nhập vào Cloudflare và chọn trang web
- Chuyển đến tab “Crypto” và nhấp vào “Enable HSTS”.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/cloudflare-hsts-config.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Chọn cài đặt bạn cần và các thay đổi sẽ được áp dụng nhanh chóng.

## 2. X-Frame-Options ##

Sử dụng header X-Frame-Options để ngăn chặn lỗ hổng **Clickjacking** trên trang web của bạn. Bằng cách triển khai header này, bạn hướng dẫn trình duyệt không nhúng trang web của bạn vào frame/iframe. Điều này có một số hạn chế trong trình duyệt hỗ trợ, vì vậy bạn phải kiểm tra trước khi triển khai nó.

Bạn có thể cấu hình ba thông số sau.

|------------------+---------|
| Giá trị tham số | Ý nghĩa |
|:----------------:|:---------|
| SAMEORIGIN | Frame/iframe của nội dung chỉ được phép từ cùng một nguồn gốc trang web. |
| DENY | Ngăn bất kỳ tên miền nào nhúng nội dung của bạn bằng frame/iframe. |
| ALLOW-FROM | Chỉ cho phép đóng khung nội dung trên một URI cụ thể. |

Hãy xem cách triển khai “DENY” để không có miền nào nhúng trang web.

### Apache ###

Thêm dòng sau vào httpd.conf và khởi động lại máy chủ web để xác minh kết quả.

```
Header always append X-Frame-Options DENY
```

### Nginx ###

Thêm phần sau vào nginx.conf dưới khối lệnh server directive/.

    add_header X-Frame-Options “DENY”;

Khởi động lại để xác minh kết quả

### F5 LTM ###

Tạo iRule với phần sau và liên kết với máy chủ ảo tương ứng.

    when HTTP_RESPONSE {
    
    HTTP::header insert "X-FRAME-OPTIONS" "DENY"
    
    }

Bạn không cần phải khởi động lại bất cứ điều gì, các thay đổi được áp dụng ngay.

### WordPress ###

Bạn cũng có thể triển khai header này thông qua WordPress. Thêm phần sau vào tệp wp-config.php

    header('X-Frame-Options: DENY);

Nếu bạn không thoải mái khi chỉnh sửa tệp, thì bạn có thể sử dụng một plugin như được [giải thích ở đây](https://geekflare.com/wordpress-x-frame-options-httponly-cookie/) hoặc đã đề cập [ở đây](https://wordpress.org/plugins/http-headers/).

## 3. X-Content-Type-Options ##

Ngăn chặn các loại rủi ro bảo mật MIME bằng cách thêm header này vào phản hồi HTTP của trang web của bạn. Việc có header này hướng dẫn trình duyệt xem xét các loại tệp như được xác định và không cho phép dò tìm nội dung. Chỉ có một tham số bạn phải thêm "nosniff".

### Apache ###

- Bạn có thể thực hiện việc này bằng cách thêm dòng dưới đây vào tệp httpd.conf

    Header set X-Content-Type-Options nosniff

- Đừng quên khởi động lại máy chủ web Apache để cấu hình hoạt động.

### Nginx ###

- Thêm dòng sau vào tệp nginx.conf dưới khối server.

    add_header X-Content-Type-Options nosniff;

- Như thường lệ, bạn phải khởi động lại Nginx để kiểm tra kết quả.

## 4. Content Security Policy ##

Ngăn chặn các cuộc tấn công XSS, clickjacking, **code injection** bằng cách triển khai Content Security Policy (CSP) header trong phản hồi HTTP trang web của bạn. CSP hướng dẫn trình duyệt tải nội dung được phép tải trên trang web.

Tất cả các trình duyệt không hỗ trợ CSP, vì vậy bạn phải xác minh trước khi triển khai nó. Có ba cách để bạn có thể đạt được tiêu đề CSP.

- Content-Security-Policy – Mức 2/1.0
- X-Content-Security-Policy – Không dùng nữa
- X-Webkit-CSP – Không dùng nữa

Nếu bạn vẫn đang sử dụng phiên bản không dùng nữa, thì bạn có thể xem xét nâng cấp lên phiên bản mới nhất.

Có thể có nhiều tham số để triển khai CSP và bạn có thể tham khảo [OWASP](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#Content-Security-Policy) để biết ý tưởng. Tuy nhiên, hãy cùng điểm qua hai tham số được sử dụng nhiều nhất.

|------------------+---------|
| Giá trị tham số | Ý nghĩa |
|:----------------:|:---------|
| default-src | Tải mọi thứ từ một nguồn đã xác định |
| script-src | Chỉ tải các tập lệnh từ một nguồn đã xác định |

Ví dụ sau về tải mọi thứ từ cùng một nguồn gốc trong các máy chủ web khác nhau.

### Apache ###

- Thêm phần sau vào tệp httpd.conf và khởi động lại máy chủ web để có hiệu lực.

    Header set Content-Security-Policy "default-src 'self';"

### Nginx ###

- Thêm phần sau vào khối server trong tệp nginx.conf

    add_header Content-Security-Policy "default-src 'self';";

Kiểm tra điều này để triển khai [frame-ancestors](https://geekflare.com/csp-frame-ancestors-configuration/) bằng CSP. Đây là phiên bản nâng cao của X-Frame-Options.

## 5. X-Permitted-Cross-Domain-Policies ##

Sử dụng các sản phẩm của Adobe như PDF, Flash, v.v.?

Bạn có thể triển khai header này để hướng dẫn trình duyệt cách xử lý các yêu cầu qua miền chéo. Bằng cách triển khai header này, bạn hạn chế tải nội dung trang web của mình từ các miền khác để tránh lạm dụng tài nguyên.

Có một số tùy chọn có sẵn.

|---------+---------|
| Giá trị | Ý nghĩa |
|:--------:|:---------|
| none | không có chính sách nào được cho phép |
| master-only | chỉ cho phép chính sách chính |
| all | mọi thứ đều được phép |
| by-content-only | Chỉ cho phép một loại nội dung nhất định. Ví dụ - XML |
| by-ftp-only | chỉ áp dụng cho máy chủ FTP |

### Apache ###

- Nếu bạn không muốn cho phép bất kỳ chính sách nào.

    Header set X-Permitted-Cross-Domain-Policies "none"

Bạn sẽ thấy header như sau.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/permitted-cross-domain.png" width = "800">
</div>
<div class="thecap"></div>
</div>

### Nginx ###

Và, giả sử bạn cần triển khai master-only, thì thêm phần sau vào tệp nginx.conf dưới khối server.

    add_header X-Permitted-Cross-Domain-Policies master-only;

Và kết quả.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/nginx-permitted-cross.png" width = "800">
</div>
<div class="thecap"></div>
</div>

## 6. Referrer-Policy ##

Tìm cách kiểm soát chính sách liên kết giới thiệu của trang web của bạn? Có một số lợi ích về quyền riêng tư và bảo mật. Tuy nhiên, không phải tất cả các tùy chọn đều được tất cả các trình duyệt hỗ trợ, vì vậy hãy xem xét các yêu cầu của bạn trước khi triển khai.

Referrer-Policy hỗ trợ cú pháp sau:

|---------+---------|
| Giá trị | Ý nghĩa |
|:-------:|:--------|
| no-referrer | Thông tin người giới thiệu sẽ không được gửi cùng với yêu cầu. |
| no-referrer-when-downgrade | Cài đặt mặc định trong đó liên kết giới thiệu được gửi đến cùng một giao thức như HTTP tới HTTP, HTTPS tới HTTPS. |
| unsafe-url | URL đầy đủ sẽ được gửi cùng với yêu cầu. |
| same-origin | Người giới thiệu sẽ chỉ được gửi cho cùng một trang web gốc. |
| strict-origin | chỉ gửi khi giao thức là HTTPS |
| strict-origin-when-cross-origin | URL đầy đủ sẽ được gửi qua một giao thức nghiêm ngặt như HTTPS |
| origin | gửi URL gốc trong tất cả các yêu cầu |
| origin-when-cross-origin | gửi URL đầy đủ trên cùng một nguồn gốc. Tuy nhiên, chỉ gửi URL gốc trong các trường hợp khác. |

### Apache ###

Bạn có thể thêm phần sau nếu bạn muốn đặt liên kết không giới thiệu.

    Header set Referrer-Policy "no-referrer"

Và sau khi khởi động lại, bạn sẽ có trong header phản hồi.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/referrer-policy-apache.png" width = "800">
</div>
<div class="thecap"></div>
</div>

### Nginx ###

Giả sử bạn cần triển khai cùng một nguồn gốc, vì vậy bạn phải thêm phần sau.

    add_header Referrer-Policy same-origin;

Sau khi cấu hình, bạn sẽ có kết quả bên dưới.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/referrer-nginx-same-origin.png" width = "800">
</div>
<div class="thecap"></div>
</div>

## 7. Expect-CT ##

Một header mới vẫn ở trạng thái thử nghiệm để hướng dẫn trình duyệt xác thực kết nối với máy chủ web về tính minh bạch của chứng chỉ (CT). Dự án này của Google nhằm khắc phục một số [lỗi trong hệ thống chứng chỉ SSL/TLS](https://geekflare.com/ssl-test-certificate/).

Ba biến sau có sẵn cho header Expect-CT.

|---------+---------|
| Giá trị | Ý nghĩa |
|:-------:|:--------|
| max-age | Tính bằng giây, trình duyệt sẽ lưu chính sách trong bộ nhớ cache trong bao lâu. |
| enforce | Một chỉ thị tùy chọn để thực thi chính sách. |
| report-uri | Trình duyệt để gửi báo cáo đến URL được chỉ định khi không nhận được tính minh bạch của chứng chỉ hợp lệ. |

### Apache ###

Giả sử bạn muốn thực thi chính sách, báo cáo và bộ nhớ cache này trong 12 giờ thì bạn phải thêm những thứ sau.

    Header set Expect-CT 'enforce, max-age=43200, report-uri="https://somedomain.com/report"'

Và đây là kết quả.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/expect-ct-apache-http.png" width = "800">
</div>
<div class="thecap"></div>
</div>

### Nginx ###

Nếu bạn muốn báo cáo và lưu vào bộ nhớ cache trong 1 giờ thì sao?

    add_header Expect-CT 'max-age=60, report-uri="https://mydomain.com/report"';

Đầu ra sẽ là.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/expect-ct-nginx.png" width = "800">
</div>
<div class="thecap"></div>
</div>

## 8. Permissions-Policy ##

Trước đó được gọi là Feature-Policy, nó được đổi tên thành Permissions-Policy với các tính năng nâng cao. Bạn có thể xem [phần này](https://github.com/w3c/webappsec-permissions-policy/blob/main/permissions-policy-explainer.md#appendix-big-changes-since-this-was-called-feature-policy) để hiểu những thay đổi lớn giữa Feature-Policy đối với Permissions-Policy.

Với Permissions-Policy, bạn có thể kiểm soát các tính năng của trình duyệt như định vị địa lý, toàn màn hình, loa, USB, tự động phát, tai nghe, micrô, thanh toán, trạng thái pin, v.v. để bật hoặc tắt trong ứng dụng web. Bằng cách triển khai chính sách này, bạn cho phép máy chủ của mình hướng dẫn ứng dụng khách (trình duyệt) tuân theo chức năng của ứng dụng web.

### Apache ###

Giả sử bạn cần phải tắt tính năng toàn màn hình và để làm như vậy, bạn có thể thêm tệp sau vào httpd.conf hoặc tệp apache2.conf tùy thuộc vào phiên bản của máy chủ Apache HTTP mà bạn sử dụng.

    Header always set Permissions-Policy "fullscreen 'none' "

Làm thế nào về việc thêm nhiều tính năng trong một dòng?

Điều đó cũng có thể xảy ra!

    Header always set Permissions-Policy "fullscreen 'none'; microphone 'none'"

Khởi động lại Apache HTTP để xem kết quả.

    HTTP/1.1 200 OK
    Date: Thu, 29 Apr 2021 06:40:43 GMT
    Server: Apache/2.4.37 (centos)
    Permissions-Policy: fullscreen 'none'; microphone 'none'
    Last-Modified: Thu, 29 Apr 2021 06:40:41 GMT
    ETag: "3-5c116c620a6f1"
    Accept-Ranges: bytes
    Content-Length: 3
    Keep-Alive: timeout=5, max=100
    Connection: Keep-Alive
    Content-Type: text/html; charset=UTF-8
    
Đoạn mã trên sẽ hướng dẫn trình duyệt tắt chế độ toàn màn hình và micrô.

Bạn cũng có thể tắt hoàn toàn tính năng này bằng cách để trống danh sách cho phép.

Ví dụ: bạn có thể thêm phần sau để tắt tính năng định vị.

    Header always set Permissions-Policy "geolocation=()"

Điều này sẽ xuất ra trên trình duyệt như bên dưới.

    HTTP/1.1 200 OK
    Date: Thu, 29 Apr 2021 06:44:19 GMT
    Server: Apache/2.4.37 (centos)
    Permissions-Policy: geolocation=()
    Last-Modified: Thu, 29 Apr 2021 06:40:41 GMT
    ETag: "3-5c116c620a6f1"
    Accept-Ranges: bytes
    Content-Length: 3
    Keep-Alive: timeout=5, max=100
    Connection: Keep-Alive
    Content-Type: text/html; charset=UTF-8

### Nginx ###

Hãy lấy một ví dụ khác - tắt tính năng rung.

    Header always set Permissions-Policy "geolocation=()"

Hoặc tắt định vị địa lý, máy ảnh và loa.

    add_header Permissions-Policy "geolocation 'none'; camera 'none'; speaker 'none';";

Đây là kết quả sau khi khởi động lại Nginx.

    HTTP/1.1 200 OK
    Server: nginx/1.14.1
    Date: Thu, 29 Apr 2021 06:48:35 GMT
    Content-Type: text/html
    Content-Length: 4057
    Last-Modified: Mon, 07 Oct 2019 21:16:24 GMT
    Connection: keep-alive
    ETag: "5d9bab28-fd9"
    Permissions-Policy: geolocation 'none'; camera 'none'; speaker 'none';
    Accept-Ranges: bytes

Tất cả cấu hình Nginx đều bị http chặn trong nginx.conf hoặc bất kỳ tệp tùy chỉnh nào bạn sử dụng.

## 9. Xóa dữ liệu trang web ##

Như bạn có thể đoán bằng tên, việc triển khai header Xóa dữ liệu trang web là một cách tuyệt vời để yêu cầu khách hàng xóa dữ liệu duyệt web như bộ nhớ cache, bộ nhớ, cookie hoặc mọi thứ. Điều này cho phép bạn kiểm soát nhiều hơn cách bạn muốn lưu trữ dữ liệu của trang web trong trình duyệt.

## Apache ##

Giả sử bạn muốn xóa bộ nhớ cache gốc, bạn có thể thêm vào bên dưới.

    Header always set Clear-Site-Data "cache"

Nó sẽ xuất ra phản hồi HTTP như bên dưới.

    HTTP/1.1 200 OK
    Date: Thu, 29 Apr 2021 07:52:14 GMT
    Server: Apache/2.4.37 (centos)
    Clear-Site-Data: cache
    Last-Modified: Thu, 29 Apr 2021 06:40:41 GMT
    ETag: "3-5c116c620a6f1"
    Accept-Ranges: bytes
    Content-Length: 3
    Keep-Alive: timeout=5, max=100
    Connection: Keep-Alive
    Content-Type: text/html; charset=UTF-8

hoặc, để xóa mọi thứ.

    Header always set Clear-Site-Data "*"

### Nginx ###

Hãy đặt Nginx để xóa cookie.

    add_header Clear-Site-Data "cookies";

Và, bạn sẽ thấy kết quả bên dưới.

    HTTP/1.1 200 OK
    Server: nginx/1.14.1
    Date: Thu, 29 Apr 2021 07:55:58 GMT
    Content-Type: text/html
    Content-Length: 4057
    Last-Modified: Mon, 07 Oct 2019 21:16:24 GMT
    Connection: keep-alive
    ETag: "5d9bab28-fd9"
    Clear-Site-Data: cookies
    Accept-Ranges: bytes

## Kết luận ##

Bảo mật một trang web là một thách thức và tôi hy vọng bằng cách triển khai các header trên giúp bạn thêm một lớp bảo mật. Nếu bạn đang điều hành một trang web kinh doanh, thì bạn cũng có thể cân nhắc sử dụng cloud-WAF như [SUCURI](https://www.anrdoezrs.net/links/8092889/type/dlg/https://sucuri.net/website-firewall/) để bảo vệ hoạt động kinh doanh trực tuyến của mình. Điều tốt về SUCURI là nó cung cấp cả bảo mật và hiệu suất.

Nếu bạn truy cập SUCURI WAF, bạn sẽ tìm thấy phần header bổ sung trong tab Firewall >> Security.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/sucuri-secure-headers.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**

- [geekflare](https://geekflare.com/http-header-implementation/)

---
