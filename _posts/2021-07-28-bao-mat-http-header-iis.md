---
layout: post
title:  "IIS - Bảo mật HTTP Headers để ngăn chặn lỗ hổng"
date:   2021-07-28 15:10:00
permalink: 2021/07/28/bao-mat-http-header-iis
tags: Security HTTP Header IIS
category: Security
img: /assets/bao-mat-http-header/Hinh1.png
summary: Bảo mật HTTP Headers để ngăn chặn lỗ hổng

---

> Bạn có biết hầu hết các lỗ hổng bảo mật có thể được khắc phục bằng cách triển khai các tiêu đề cần thiết trong tiêu đề phản hồi không?

Bảo mật cũng cần thiết như nội dung và SEO của trang web của bạn, và hàng nghìn trang web bị tấn công do lỗi cấu hình sai hoặc thiếu bảo vệ. Nếu bạn là chủ sở hữu trang web hoặc kỹ sư bảo mật và đang tìm cách bảo vệ trang web của mình khỏi các cuộc tấn công **Clickjacking**, **code injection**, **MIME**, **XSS**,... thì hướng dẫn này sẽ giúp bạn.

Trong bài viết này, tôi sẽ nói về các HTTP Headers khác nhau (được [OWASP](https://owasp.org/www-project-secure-headers/) khuyến nghị) để triển khai trong nhiều máy chủ web, nhà cung cấp mạng & CDN để bảo vệ trang web tốt hơn.

**Ghi chú:**

- Bạn nên sao lưu tệp cấu hình trước khi thực hiện thay đổi
- Một số header có thể không được hỗ trợ trên tất cả các trình duyệt, vì vậy hãy [kiểm tra tính tương thích](https://caniuse.com/) trước khi triển khai

Điều đầu tiên chúng ta nên làm là kiểm tra trang web của mình trước khi thực hiện bất kỳ thay đổi nào, để nắm rõ mọi thứ hiện tại như thế nào. Dưới đây là một số trang web mà chúng tôi có thể sử dụng để quét trang web của mình:

- [securityheaders](https://securityheaders.com/) của Scott Helme ([blog](https://scotthelme.co.uk/), [twitter](https://twitter.com/Scott_Helme)).
- [HTTP Security Report](https://httpsecurityreport.com/) của Stefán Orri Stefánsson ([twitter](https://twitter.com/stefan_orri)).
- [Headers Security Test ](https://gf.dev/secure-headers-test) của Geek Flare Tools ([website](https://tools.geekflare.com/)).

Nếu trang web của bạn không có bảo mật header, rất có thể bạn sẽ bị xếp hạng F nghiêm trọng, giống như ảnh chụp màn hình sau:

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/rating-F.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

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

- Mở IIS Manager và thêm header bằng cách đi tới “HTTP Response Headers” cho trang web tương ứng.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-hsts.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Hoặc thêm dòng sau vào trong file web.config:

```config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- Protects against Clickjacking attacks. ref.: https://www.owasp.org/index.php/HTTP_Strict_Transport_Security_Cheat_Sheet -->
      <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains"/>
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

- Khởi động lại trang web

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

- Thêm header bằng cách đi tới “HTTP Response Headers” cho trang web tương ứng.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-x-frame-options.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Hoặc thêm dòng sau vào trong file web.config:

```config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- Protects against Clickjacking attacks. ref.: http://stackoverflow.com/a/22105445/1233379 -->
      <add name="X-Frame-Options" value="SAMEORIGIN" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

- Khởi động lại trang web để xem kết quả.

## 3. X-Content-Type-Options ##

Ngăn chặn các loại rủi ro bảo mật MIME bằng cách thêm header này vào phản hồi HTTP của trang web của bạn. Việc có header này hướng dẫn trình duyệt xem xét các loại tệp như được xác định và không cho phép dò tìm nội dung. Chỉ có một tham số bạn phải thêm "nosniff".

- Mở IIS và chuyển đến HTTP Response Headers
- Nhấp vào Add và nhập vào Name và Value

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-mime-types.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Hoặc thêm dòng sau vào trong file web.config:

```config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- Protects against MIME-type confusion attack. ref.: https://www.veracode.com/blog/2014/03/guidelines-for-setting-security-headers/ -->
      <add name="X-Content-Type-Options" value="nosniff" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

- Khởi động lại IIS để xác minh kết quả.

## 4. Content Security Policy ##

Ngăn chặn các cuộc tấn công XSS, clickjacking, **code injection** bằng cách triển khai Content Security Policy (CSP) header trong phản hồi HTTP trang web của bạn. CSP hướng dẫn trình duyệt tải nội dung được phép tải trên trang web.

Điều này sử dụng phương pháp danh sách trắng cho trình duyệt biết nơi tìm nạp hình ảnh, scripts, CSS, v.v.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/blog_diagram.png" width = "800">
</div>
<div class="thecap"></div>
</div>

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
| img-src | Chỉ định nguồn mà hình ảnh có thể được truy xuất. |
| media-src | xác định các vị trí mà từ đó đa phương tiện như video có thể được truy xuất. |
| object-src | để xác định các vị trí mà từ đó các plugin có thể được truy xuất. |
| font-src | Chỉ định các nguồn được phép tải phông chữ. |

Ví dụ:

    Content-Security-Policy: default-src 'self'; media-src beagle.com beaglesecurity.com; script-src beagle.com; img-src *;

Điều này được trình duyệt hiểu là:

- default-src 'self'; - tải mọi thứ từ miền hiện tại
- media-src beagle.com beaglesecurity.com; - chỉ tải đa phương tiện từ beagle.com và beaglesecurity.com
- script-src beagle.com; - chỉ tải tập lệnh từ beagle.com
- img-src \*; - tải hình ảnh từ mọi nơi

Triển khai trên máy chủ:
Ví dụ sau về tải mọi thứ từ cùng một nguồn gốc trong các máy chủ web khác nhau.

- Đi tới HTTP Response Headers cho trang web tương ứng của bạn trong IIS Manager và thêm phần sau

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-csp.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Hoặc thêm dòng sau vào trong file web.config:

```config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- CSP modern XSS directive-based defence, used since 2014. ref.: http://content-security-policy.com/ -->
      <add name="Content-Security-Policy" value="default-src 'self'; font-src *;img-src * data:; script-src *; style-src *;" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

- Khởi động lại IIS để xác minh kết quả.

## 5. Referrer-Policy ##

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

- Thêm dòng sau vào trong file web.config:

```config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- Prevents from leaking referrer data over insecure connections. ref.: https://scotthelme.co.uk/a-new-security-header-referrer-policy/ -->
      <add name="Referrer-Policy" value="strict-origin" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

- Khởi động lại IIS để xác minh kết quả.

## 6. X-Xss-Protection ##

Header này được sử dụng để định cấu hình bảo vệ XSS phản chiếu được tích hợp sẵn trong Internet Explorer, Chrome và Safari (Webkit). Cài đặt hợp lệ cho header là 0 - vô hiệu hóa bảo vệ, 1 - cho phép bảo vệ và 1; mode=block - yêu cầu trình duyệt chặn phản hồi nếu nó phát hiện ra một cuộc tấn công thay vì làm sạch tập lệnh.

- Đi tới HTTP Response Headers cho trang web tương ứng của bạn trong IIS Manager và thêm phần sau:

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-xxss-header.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Hoặc thêm dòng sau vào trong file web.config:

```config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- Protects against XSS injections. ref.: https://www.veracode.com/blog/2014/03/guidelines-for-setting-security-headers/ -->
      <add name="X-XSS-Protection" value="1; mode=block" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

- Khởi động lại IIS để xác minh kết quả.

## 7. HTTP Public Key Pinning ##

Điều tuyệt vời về chứng chỉ SSL/TLS là bạn có thể mua chứng chỉ từ bất kỳ Tổ chức phát hành chứng chỉ đáng tin cậy nào và trình duyệt sẽ vui vẻ chấp nhận nó. Vấn đề với điều này là khi Tổ chức phát hành chứng chỉ bị xâm phạm, kẻ tấn công có thể tự cấp chứng chỉ SSL / TLS cho trang web của bạn và trình duyệt sẽ chấp nhận chứng chỉ giả mạo này vì nó đến từ Tổ chức phát hành chứng chỉ đáng tin cậy. HPKP cho phép bạn tự bảo vệ mình bằng cách cung cấp danh sách trắng các nhận dạng mật mã mà trình duyệt nên tin tưởng. Trong khi HSTS nói rằng trình duyệt phải luôn sử dụng HTTPS, HPKP cho biết trình duyệt chỉ nên chấp nhận một bộ chứng chỉ cụ thể. Đọc thêm [Ghim khóa công khai HTTP](https://scotthelme.co.uk/hpkp-http-public-key-pinning/).

- Đi tới HTTP Response Headers cho trang web tương ứng của bạn trong IIS Manager và thêm phần sau:

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-hpkp-header.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Khởi động lại IIS để xác minh kết quả.

## 8. Xóa bỏ Headers ##

Bước tiếp theo trong việc tăng cường các header phản hồi HTTP của bạn là xem xét các header mà bạn có thể xóa để giảm lượng thông tin bạn tiết lộ về máy chủ của mình và những gì đang chạy trên đó. Máy chủ thường sẽ tiết lộ phần mềm nào đang chạy trên chúng, phiên bản phần mềm nào trên đó và những frameworks nào đang cung cấp cho nó. Giảm lượng thông tin bạn tiết lộ luôn là một lợi ích. Tôi sẽ xem xét một số tiêu đề phổ biến nhất nhưng bạn luôn có thể kiểm tra các tiêu đề phản hồi HTTP trên trang web của riêng mình để xem liệu có thêm tiêu đề nào mà bạn có thể xóa hay không bằng cách sử dụng các công cụ ở phần đầu bài viết này.

### Server header ###

Server header là header phổ biến nhất mà bạn có thể sẽ thấy trên một trang web. Trường này chứa thông tin về
phần mềm được máy chủ gốc sử dụng để xử lý yêu cầu. Việc tiết lộ phiên bản phần mềm cụ thể của máy chủ có thể cho phép máy chủ trở nên dễ bị tấn công hơn trước các cuộc tấn công chống lại phần mềm được biết là có chứa các lỗ hổng bảo mật. Người triển khai máy chủ được khuyến khích đặt trường này thành một tùy chọn có thể định cấu hình.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-server-header.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Đối với IIS, để loại bỏ header này có vẻ hơi dài dòng. Điều đầu tiên bạn cần là module [URL-Rewrite](http://www.iis.net/downloads/microsoft/url-rewrite). Sau khi cài đặt, hãy đi tới IIS Manager và chọn trang web của bạn, sau đó chọn tiếp URL Rewrite.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Chọn "View Server Variables", rồi thêm mới Server Variable có tên là RESPONSE_SERVER.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-view-server-variables.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-view-server-variables-add.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-view-server-variables-add-value.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Sau khi bạn có server variable mới, hãy quay lại trang Rules, thêm Rule mới và chọn blank outbound rule.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-view-server-variables-back-to-rules.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-add-rules.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-add-rules-blank.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Đặt tên cho rule, chọn Matching Scope là Server Variable, Variable name là RESPONSE_SERVER và đặt Pattern là .* để khớp với bất kỳ nội dung nào. Click Apply để lưu rule.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-add-rule-content.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Những thay đổi này bây giờ sẽ xóa nội dung của Server header, vì vậy nó sẽ trông giống như thế này:

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-blank-server-header.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Nếu muốn, bạn có thể chỉnh sửa rule và cuộn xuống sâu hơn để cung cấp cho header một số nội dung.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-add-rule-content-value.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-server-header-with-value.png" width = "800">
</div>
<div class="thecap"></div>
</div>

### X-Powered-By ###

Header X-Powered-By cung cấp thông tin về công nghệ hỗ trợ máy chủ Web.

Có 2 cách khả thi để bạn có thể xóa hoặc thay đổi header X-Powered-By trong IIS. Cách đầu tiên và dễ nhất là kiểm tra phần HTTP Response Headers.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-response-headers-1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Nếu header X-Powered-By xuất hiện ở đây, bạn có thể chỉ cần sửa đổi giá trị của nó hoặc xóa nó.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-remove-x-powered-by.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Phương pháp thứ hai, giống như Server header, là sử dụng mô-đun URL-Rewrite để xóa hoặc thay đổi giá trị. Thực hiện theo các bước tương tự đối với Server header, nhưng thay thế tên của server variable và các chi tiết khi tạo rule.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-add-variable-x-powered-by.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/iis-url-rewrite-add-rule-content-x-powered-by.png" width = "800">
</div>
<div class="thecap"></div>
</div>

### X-AspNet-Version ###

Nó tiết lộ phiên bản cụ thể của Asp.NET mà bạn đang chạy, vì vậy nó phải hoạt động. Thực sự dễ dàng để loại bỏ, nó chỉ yêu cầu một thay đổi nhỏ trong tệp web.config của bạn.

```
<system.web>
<httpRuntime enableVersionHeader="false" />
</system.web>
```

Sau khi bạn đã lưu các thay đổi, hãy khởi động lại trang web của bạn để chúng có hiệu lực.

## 9. Kiểm tra ##

Sử dụng các công cụ ở đầu bài viết này, kết quả kiểm tra lại các header trên như sau:

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/rating-A.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

## 10. Kết luận ##

Bảo mật một trang web là một thách thức và tôi hy vọng bằng cách triển khai các header trên giúp bạn thêm một lớp bảo mật. Nếu bạn đang điều hành một trang web kinh doanh, thì bạn cũng có thể cân nhắc sử dụng cloud-WAF như [SUCURI](https://www.anrdoezrs.net/links/8092889/type/dlg/https://sucuri.net/website-firewall/) để bảo vệ hoạt động kinh doanh trực tuyến của mình. Điều tốt về SUCURI là nó cung cấp cả bảo mật và hiệu suất.

Nếu bạn truy cập SUCURI WAF, bạn sẽ tìm thấy phần header bổ sung trong tab Firewall >>Security.

<div class="imgcap">
<div >
    <img src="/assets/bao-mat-http-header/sucuri-secure-headers.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**

- [geekflare](https://geekflare.com/http-header-implementation/)
- [ryadel](https://www.ryadel.com/en/iis-web-config-secure-http-response-headers-pass-securityheaders-io-scan/)
- [scotthelme](https://scotthelme.co.uk/hardening-your-http-response-headers/)

---
