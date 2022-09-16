---
layout: post
title:  "Cài đặt WAF ModSecurity với Nginx"
date:   2022-06-24 15:10:00
permalink: 2022/06/24/how-to-install-modsecurity-with-nginx-on-rocky-linux-8
tags: WAF Nginx ModSecurity
category: WAF
img: /assets/how-to-install-modsecurity-with-nginx-on-rocky-linux-8/Hinh1.png
summary: Cài đặt WAF ModSecurity với Nginx

---

ModSecurity, thường gọi là Modsec, là một tường lửa ứng dụng web mã nguồn mở miễn phí (WAF). ModSecurity được tạo như một mô-đun cho Máy chủ Apache HTTP. Tuy nhiên, kể từ những ngày đầu thành lập, WAF đã phát triển và hiện bao gồm một loạt các khả năng lọc phản hồi và yêu cầu Giao thức truyền siêu văn bản cho các nền tảng khác nhau như Microsoft IIS, Nginx và Apache.

Cách WAF hoạt động, công cụ ModSecurity được triển khai trước ứng dụng web, cho phép công cụ quét các kết nối HTTP đến và đi. ModSecurity được sử dụng phổ biến nhất cùng với Bộ quy tắc cốt lõi OWASP (CRS), một bộ quy tắc mã nguồn mở được viết bằng ngôn ngữ SecRules của ModSecurity và được đánh giá cao trong ngành bảo mật.

Bộ quy tắc OWASP với ModSecurity gần như ngay lập tức có thể giúp bảo vệ máy chủ của bạn chống lại:
- Tác nhân người dùng xấu
- DDOS
- Tập lệnh trang web chéo
- SQL injection
- Chiếm quyền điều khiển phiên
- Các mối đe dọa khác

Trong hướng dẫn sau, bạn sẽ học cách cài đặt ModSecurity với Nginx trên Rocky Linux 8.

## Cập nhật hệ điều hành
Cập nhật hệ điều hành Rocky linux của bạn để đảm bảo tất cả các gói hiện có đều được cập nhật:

```
sudo dnf upgrade --refresh -y
```

## Cài đặt kho lưu trữ EPEL
Để cài đặt thành công ModSecurity với Rocky Linux 8, bạn sẽ cần kích hoạt kho EPEL để hoàn tất quá trình cài đặt Modsecurity.

Cài đặt Kho lưu trữ EPEL:

```
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y
```

## Cài đặt Nginx mới nhất
Nếu bạn đã cài Nginx thì có thể bỏ qua bước này.
Để kiểm tra phiên bản Nginx của bạn, hãy sử dụng lệnh sau:

```
nginx -v
```

## Tải xuống mã nguồn Nginx
Bạn sẽ cần tải xuống mã nguồn Nginx để biên dịch ModSecurity mô-đun động. Bạn sẽ cần tải xuống và lưu trữ gói nguồn trong vị trí thư mục /etc/local/src/nginx.

```
sudo mkdir /usr/local/src/nginx && cd /usr/local/src/nginx
```

Tùy chọn - Gán quyền cho thư mục nếu cần như sau:
```
sudo chown username:username /usr/local/src/ -R
```

Tải xuống nguồn bằng cách sử dụng wget lệnh như sau (chỉ ví dụ):

```
wget http://nginx.org/download/nginx-1.21.1.tar.gz
```

Tiếp theo, giải nén kho lưu trữ như sau:

````
tar -xvzf nginx-1.21.1.tar.gz
```

Tiếp theo, xác nhận rằng gói nguồn giống với phiên bản Nginx được cài đặt trên hệ điều hành Rocky Linux của bạn.

Để làm điều này, hãy sử dụng nginx -v lệnh như sau:
```
nginx -v
```

## Cài đặt libmodsecurity3 cho ModSecurity ##
Gói libmodsecurity3 là phần cơ bản của WAF thực hiện Lọc HTTP cho các ứng dụng web của bạn. Bạn sẽ biên dịch từ nguồn.

** Clone ModSecurity Repsoitory từ Github **
Bước đầu tiên là sao chép từ Github và nếu bạn chưa cài đặt git, bạn sẽ cần thực hiện lệnh sau:

```
sudo dnf install git -y
```

Tiếp theo, sao chép libmodsecurity3 GIT kho lưu trữ như sau:

```
sudo git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity /usr/local/src/ModSecurity/
```

Sau khi nhân bản, bạn sẽ cần phải CD vào thư mục:

```
cd /usr/local/src/ModSecurity/
```

** Cài đặt các phụ thuộc libmodsecurity3 **
Để biên dịch, bạn sẽ cần cài đặt các phụ thuộc sau như sau:

```
sudo dnf install gcc-c++ flex bison yajl curl-devel zlib-devel pcre-devel autoconf automake git curl make libxml2-devel pkgconfig libtool httpd-devel redhat-rpm-config wget openssl openssl-devel nano -y
```

Tiếp theo, cài đặt các phần phụ thuộc khác bằng cách sử dụng kho PowerTool:

```
sudo dnf --enablerepo=powertools install doxygen yajl-devel -y
```

Bây giờ cài đặt GeoIP bằng cách sử dụng kho lưu trữ REMI:

```
dnf install epel-release https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
sudo dnf --enablerepo=remi install GeoIP-devel -y
```

Bây giờ để kết thúc, hãy cài đặt các mô-đun con GIT sau như sau:

```
git submodule init
```

Sau đó cập nhật các mô-đun con:

```
git submodule update
```

** Xây dựng Môi trường ModSecurity **
Bước tiếp theo bây giờ thực sự là xây dựng môi trường trước tiên. Sử dụng lệnh sau:

```
./build.sh
```

Tiếp theo, chạy lệnh cấu hình:

```
./configure
```

Lưu ý, bạn có thể sẽ gặp lỗi sau

```
fatal: No names found, cannot describe anything.
```

Bạn có thể bỏ qua điều này một cách an toàn và chuyển sang bước tiếp theo.

** Biên dịch mã nguồn ModSecurity **
Bây giờ bạn đã xây dựng và định cấu hình môi trường cho libmodsecurity3, đã đến lúc biên dịch nó bằng lệnh làm cho.

```
make
```

Một thủ thuật hữu ích là chỉ định -NS vì điều này có thể tăng đáng kể tốc độ biên dịch nếu bạn có một máy chủ mạnh. Ví dụ, LinuxCapable máy chủ có 6 CPU, và tôi có thể sử dụng cả 6 hoặc ít nhất là sử dụng 4 đến 5 để tăng tốc độ.

```
make -j 6
```

Sau khi biên dịch mã nguồn, bây giờ hãy chạy lệnh cài đặt trong thiết bị đầu cuối của bạn:

```
sudo make install
```

Lưu ý, cài đặt được thực hiện trong /usr/local/modsecurity/, mà bạn sẽ tham khảo sau trong hướng dẫn.

## Cài đặt ModSecurity-nginx Connector ##

Mô hình Đầu nối ModSecurity-nginx là điểm kết nối giữa Nginx và libmodsecurity. Nó là thành phần giao tiếp giữa Nginx và ModSecurity (libmodsecurity3).

** Clone ModSecurity-nginx Repsoitory từ Github **
Tương tự như bước trước đó sao chép kho lưu trữ libmodsecurity3, bạn sẽ cần sao chép lại kho lưu trữ trình kết nối bằng cách sử dụng lệnh sau:

```
sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git /usr/local/src/ModSecurity-nginx/
```

** Cài đặt ModSecurity-nginx **

Tiếp theo, đưa thư mục CD vào thư mục nguồn Nginx như sau:

```
cd /usr/local/src/nginx/nginx-1.21.1
```

Lưu ý, thay thế phiên bản của hướng dẫn bằng phiên bản Nginx hiện tại trong hệ thống của bạn.

Tiếp theo, bạn sẽ biên dịch Trình kết nối ModSecurity-nginx mô-đun chỉ với –With-compat cờ như sau:

```
sudo ./configure --with-compat --add-dynamic-module=/usr/local/src/ModSecurity-nginx
```


./configure --user=nginx --group=nginx --with-pcre-jit --with-debug --with-http_ssl_module --with-http_realip_module --add-module=/usr/local/src/ModSecurity-nginx

Hiện nay làm cho (tạo) các mô-đun động bằng lệnh sau:

```
sudo make modules
```

Tiếp theo, khi ở trong thư mục nguồn Nginx, hãy sử dụng lệnh sau để di chuyển mô-đun động bạn đã tạo đã được lưu tại vị trí objs/ngx_http_modsecurity_module.so và sao chép nó vào /usr/share/nginx/modules/ thư mục.

```
sudo cp objs/ngx_http_modsecurity_module.so /usr/share/nginx/modules/
```

## Tải và định cấu hình Trình kết nối ModSecurity-nginx với Nginx ##

Bây giờ bạn đã biên dịch mô-đun động và định vị nó cho phù hợp, bạn cần chỉnh sửa /etc/nginx/nginx.conf tệp cấu hình để ModSecurity hoạt động với máy chủ web Nginx của bạn.

** Bật ModSecurity trong nginx.conf **
Trước tiên, bạn cần xác định load_module và đường dẫn đến mô-đun bảo mật của bạn.


Mở ra nginx.conf với bất kỳ trình soạn thảo văn bản nào.

```
sudo nano /etc/nginx/nginx.conf
```

Tiếp theo, thêm dòng sau vào tệp gần đầu:
```
load_module modules/ngx_http_modsecurity_module.so;
```

Nếu bạn đã đặt mô-đun ở nơi khác, hãy bao gồm đường dẫn đầy đủ.

Bây giờ hãy thêm mã sau vào HTTP {} phần như sau:

```
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/modsec-config.conf;
```

Chỉ ví dụ:
<div class="imgcap">
<div >
    <img src="/assets/how-to-install-modsecurity-with-nginx-on-rocky-linux-8/H1.png" width = "800">
</div>
<div class="thecap">Cách cài đặt ModSecurity với Nginx trên Rocky Linux 8</div>
</div>

Nếu bạn đã đặt mô-đun ở nơi khác, hãy bao gồm đường dẫn đầy đủ.

Lưu nginx.conf hồ sơ (CTRL + O), sau đó thoát ra (CTRL + X).

** Tạo và cấu hình thư mục và tệp cho ModSecurity **

Bạn sẽ cần tạo một thư mục để lưu trữ các tệp cấu hình và các quy tắc trong tương lai, OWASP CRS cho phần hướng dẫn.

Sử dụng lệnh sau để tạo / etc / nginx / modsec thư mục như sau:

```
sudo mkdir -p /etc/nginx/modsec/
```

Bây giờ, bạn cần sao chép lại tệp cấu hình ModSecurity mẫu từ bản sao của chúng tôi GIT danh mục:

```
sudo cp /usr/local/src/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
```

Sử dụng trình soạn thảo văn bản yêu thích của bạn trong Rocky Linux, mở modsecurity.conf tập tin như sau:

```
sudo nano /etc/nginx/modsec/modsecurity.conf
```

Theo mặc định, cấu hình ModSecurity có công cụ quy tắc được chỉ định là (DetectionOnly), nói cách khác, chạy ModSecurity và phát hiện tất cả các hành vi độc hại nhưng không chặn hoặc cấm hành động và ghi lại tất cả các giao dịch HTTP mà nó gắn cờ. Điều này chỉ nên được sử dụng nếu bạn có nhiều kết quả dương tính giả hoặc đã tăng cài đặt cấp độ bảo mật lên mức cực đoan và thử nghiệm để xem có bất kỳ kết quả dương tính giả nào xảy ra hay không.

Để thay đổi hành vi này thành (trên), tìm phần sau trên dòng 7:

```
SecRuleEngine DetectionOnly
```

Thay đổi dòng này để kích hoạt ModSecurity:

```
SecRuleEngine On
```

Bây giờ, bạn cần xác định vị trí sau, nằm trên dòng 224:

```
# Log everything we know about a transaction.
SecAuditLogParts ABIJDEFHZ
```

Điều này không đúng và cần được thay đổi. Sửa đổi dòng thành sau:

```
SecAuditLogParts ABCEFHJKZ
```

Bây giờ lưu modsecurity.conf tập tin sử dụng (CTRL + O) sau đó thoát ra (CTRL + X).

Phần tiếp theo là tạo tệp sau modsec-config.conf. Ở đây bạn sẽ thêm modsecurity.conf nộp cùng và sau đó về các quy tắc khác, chẳng hạn như OWASP CRS và nếu bạn đang sử dụng WordPress, WPRS CRS bộ quy tắc.

Sử dụng lệnh sau để tạo và mở tệp:

```
sudo nano /etc/nginx/modsec/modsec-config.conf
```

Khi vào bên trong tệp, hãy thêm dòng sau:

```
Include /etc/nginx/modsec/modsecurity.conf
```

Lưu modsec-config.conf nộp hồ sơ với (CTRL + O) sau đó (CTRL + X) lối thoát.

Cuối cùng, sao chép ModSecurity's unicode.mapping tập tin với CP lệnh như sau:

```
sudo cp /usr/local/src/ModSecurity/unicode.mapping /etc/nginx/modsec/
```

Bây giờ trước khi tiếp tục, bạn nên chạy thử dịch vụ Nginx của mình bằng lệnh terminal sau:

```
sudo nginx -t
```

Nếu bạn đã thiết lập mọi thứ một cách chính xác, bạn sẽ nhận được kết quả sau:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Để thực hiện các thay đổi trực tiếp, hãy khởi động lại dịch vụ Nginx của bạn bằng cách sử dụng systemctl chỉ huy:
```
sudo systemctl restart nginx
```

## Cài đặt Bộ quy tắc cốt lõi OWASP cho ModSecurity ##

Bản thân ModSecurity không bảo vệ máy chủ web của bạn và bạn cần phải có các quy tắc. Một trong những quy tắc nổi tiếng nhất, được tôn trọng và được nhiều người biết đến là bộ quy tắc OWASP CRS. Các quy tắc được sử dụng rộng rãi nhất trong số các máy chủ web và các WAF khác, và hầu hết các hệ thống tương tự khác dựa trên hầu hết các quy tắc của chúng dựa trên CRS này. Việc cài đặt bộ quy tắc này sẽ tự động cung cấp cho bạn một nguồn bảo vệ tuyệt vời chống lại hầu hết các mối đe dọa mới xuất hiện trên Internet bằng cách phát hiện các tác nhân độc hại và ngăn chặn chúng.

Sử dụng lệnh wget, tải về Kho lưu trữ OWASP CRS 3.3.2 như sau:

```
wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.2.zip
```

Đối với những người muốn sống vượt trội, bạn có thể tải xuống bản dựng hàng đêm. Chỉ sử dụng hàng đêm nếu bạn chuẩn bị tiếp tục biên dịch lại và kiểm tra CoreRuleSet Github thường xuyên để cập nhật và tự tin hơn trong việc tìm ra các vấn đề. Về mặt kỹ thuật, hàng đêm có thể an toàn hơn nhưng có thể tạo ra các vấn đề.

Đối với người mới sử dụng, hãy sử dụng phiên bản ổn định và không sử dụng phiên bản dưới đây.

```
wget https://github.com/coreruleset/coreruleset/archive/refs/tags/nightly.zip
```

cài đặt Giải nén gói nếu bạn chưa cài đặt cái này trên máy chủ của mình:

```
sudo dnf install unzip -y
```

Hiện nay giải nén các master.zip lưu trữ như sau:

```
sudo unzip v3.3.2.zip -d /etc/nginx/modsec
```

Như trước đây, như modsecurity.conf cấu hình mẫu, OWASP CRS đi kèm với một tệp cấu hình mẫu mà bạn cần đổi tên. Tốt nhất là sử dụng CP lệnh và giữ một bản sao lưu cho tương lai trong trường hợp bạn cần khởi động lại.

```
sudo cp /etc/nginx/modsec/coreruleset-3.3.2/crs-setup.conf.example /etc/nginx/modsec/coreruleset-3.3.2/crs-setup.conf
```

Để kích hoạt các quy tắc, hãy mở /etc/nginx/modsec/modsec-config.conf sử dụng lại bất kỳ trình soạn thảo văn bản nào:

```
sudo nano /etc/nginx/modsec/modsec-config.conf
```

Khi vào bên trong tệp một lần nữa, hãy thêm hai dòng bổ sung sau:

```
Include /etc/nginx/modsec/coreruleset-3.3.2/crs-setup.conf
Include /etc/nginx/modsec/coreruleset-3.3.2/rules/*.conf
```

Lưu các tập tin (CTRL + O) và thoát ra (CTRL + T).

Như trước đây, bạn cần kiểm tra mọi bổ sung mới cho dịch vụ Nginx của mình trước khi đưa nó vào hoạt động:

```
sudo nginx -t
```

Bạn sẽ nhận được kết quả sau có nghĩa là mọi thứ đang hoạt động chính xác:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Khởi động lại dịch vụ Nginx của bạn để thực hiện các thay đổi như sau:

```
sudo systemctl restart nginx
```

## Sử dụng và hiểu OWASP CRS ##

Quy tắc cốt lõi của OWASP có khá nhiều tùy chọn, tuy nhiên, cài đặt mặc định sẽ bảo vệ hầu hết các máy chủ ngay lập tức mà không làm tổn hại đến khách truy cập tự nhiên và bot SEO tốt của bạn. Dưới đây, một số lĩnh vực sẽ được đề cập để giúp giải thích. Đọc thêm sẽ là tốt nhất để điều tra tất cả các tùy chọn trong các tệp cấu hình vì chúng có khá nhiều dữ liệu văn bản để giải thích chúng là gì.

Mở CRS-setup.conf tập tin như sau:
```
sudo nano /etc/nginx/modsec/coreruleset-3.4-dev/crs-setup.conf
```

Lưu ý, đây là cấu hình phiên bản dev với các mục bổ sung so với phiên bản 3.3.

Từ đây, bạn có thể sửa đổi hầu hết các cài đặt OWASP CRS của mình.

** Chấm điểm CRS OWASP **

Để chia nhỏ nó, ModSecurity có hai chế độ:

Chế độ chấm điểm bất thường

```
# -- [[ Anomaly Scoring Mode (default) ]] --
# In CRS3, anomaly mode is the default and recommended mode, since it gives the
# most accurate log information and offers the most flexibility in setting your
# blocking policies. It is also called "collaborative detection mode".
# In this mode, each matching rule increases an 'anomaly score'.
# At the conclusion of the inbound rules, and again at the conclusion of the
# outbound rules, the anomaly score is checked, and the blocking evaluation
# rules apply a disruptive action, by default returning an error 403.
```

Chế độ khép kín

```
# -- [[ Self-Contained Mode ]] --
# In this mode, rules apply an action instantly. This was the CRS2 default.
# It can lower resource usage, at the cost of less flexibility in blocking policy
# and less informative audit logs (only the first detected threat is logged).
# Rules inherit the disruptive action that you specify (i.e. deny, drop, etc).
# The first rule that matches will execute this action. In most cases this will
# cause evaluation to stop after the first rule has matched, similar to how many
# IDSs function.
```

Tính điểm bất thường thường dành cho hầu hết người dùng chế độ tốt nhất để sử dụng.

Có bốn mức độ hoang tưởng:

- Hoang tưởng cấp độ 1 - Mức mặc định và được khuyến nghị cho hầu hết người dùng.
 Hoang tưởng cấp độ 2 - Chỉ người dùng nâng cao.
 Hoang tưởng cấp độ 3 - Chỉ người dùng thành thạo.
 Hoang tưởng cấp độ 4 - Không được khuyến khích ở tất cả, ngoại trừ các trường hợp đặc biệt.

```
# -- [[ Paranoia Level Initialization ]] ---------------------------------------
#
# The Paranoia Level (PL) setting allows you to choose the desired level
# of rule checks that will add to your anomaly scores.
#
# With each paranoia level increase, the CRS enables additional rules
# giving you a higher level of security. However, higher paranoia levels
# also increase the possibility of blocking some legitimate traffic due to
# false alarms (also named false positives or FPs). If you use higher
# paranoia levels, it is likely that you will need to add some exclusion
# rules for certain requests and applications receiving complex input.
#
# - A paranoia level of 1 is default. In this level, most core rules
#   are enabled. PL1 is advised for beginners, installations
#   covering many different sites and applications, and for setups
#   with standard security requirements.
#   At PL1 you should face FPs rarely. If you encounter FPs, please
#   open an issue on the CRS GitHub site and don't forget to attach your
#   complete Audit Log record for the request with the issue.
# - Paranoia level 2 includes many extra rules, for instance enabling
#   many regexp-based SQL and XSS injection protections, and adding
#   extra keywords checked for code injections. PL2 is advised
#   for moderate to experienced users desiring more complete coverage
#   and for installations with elevated security requirements.
#   PL2 comes with some FPs which you need to handle.
# - Paranoia level 3 enables more rules and keyword lists, and tweaks
#   limits on special characters used. PL3 is aimed at users experienced
#   at the handling of FPs and at installations with a high security
#   requirement.
# - Paranoia level 4 further restricts special characters.
#   The highest level is advised for experienced users protecting
#   installations with very high security requirements. Running PL4 will
#   likely produce a very high number of FPs which have to be
#   treated before the site can go productive.
#
# All rules will log their PL to the audit log;
# example: [tag "paranoia-level/2"]. This allows you to deduct from the
# audit log how the WAF behavior is affected by paranoia level.
#
# It is important to also look into the variable
# tx.enforce_bodyproc_urlencoded (Enforce Body Processor URLENCODED)
# defined below. Enabling it closes a possible bypass of CRS.
```

** Kiểm tra CRS OWASP trên Máy chủ của bạn **

Để kiểm tra xem các quy tắc OWASP có đang hoạt động trên máy chủ của bạn hay không, hãy mở Trình duyệt Internet của bạn và sử dụng các thao tác sau:

```
https://www.yourdomain.com/index.html?exec=/bin/bash
```

Bạn sẽ nhận được một Lỗi 403 bị cấm. Nếu không, thì một bước đã bị bỏ lỡ.

Vấn đề phổ biến nhất là thiếu để thay đổi DetectionOnly đến Trên, như đã đề cập trước đó trong hướng dẫn.


Đối phó với các khẳng định sai và loại trừ các quy tắc tùy chỉnh
Một trong những nhiệm vụ thường không bao giờ kết thúc là xử lý các lỗi dương tính giả, ModSecurity và OWASP CRS cùng nhau thực hiện một công việc tuyệt vời, nhưng điều đó phải trả giá bằng thời gian của bạn, nhưng với sự bảo vệ, bạn sẽ nhận được nó rất xứng đáng. Đối với người mới bắt đầu, không bao giờ đặt mức độ hoang tưởng lên cao để bắt đầu là quy tắc vàng.

Một nguyên tắc nhỏ là chạy quy tắc được đặt ra trong vài tuần đến vài tháng mà hầu như không có bất kỳ trường hợp dương tính giả nào, sau đó tăng lên, chẳng hạn như hoang tưởng cấp độ 1 đến hoang tưởng cấp độ 2, vì vậy bạn không bị ngập trong cả tấn đồng thời.

Loại trừ các ứng dụng đã biết về khẳng định sai khẳng định
Modsecurity, theo mặc định, có thể đưa vào danh sách trắng các hành động hàng ngày dẫn đến kết quả dương tính giả như sau:

```
#SecAction \
# "id:900130,\
#  phase:1,\
#  nolog,\
#  pass,\
#  t:none,\
#  setvar:tx.crs_exclusions_cpanel=1,\
#  setvar:tx.crs_exclusions_dokuwiki=1,\
#  setvar:tx.crs_exclusions_drupal=1,\
#  setvar:tx.crs_exclusions_nextcloud=1,\
#  setvar:tx.crs_exclusions_phpbb=1,\
#  setvar:tx.crs_exclusions_phpmyadmin=1,\
#  setvar:tx.crs_exclusions_wordpress=1,\
#  setvar:tx.crs_exclusions_xenforo=1"
```

Ví dụ: để bật, WordPress, phpBB và phpMyAdmin khi bạn sử dụng cả ba, bỏ ghi chú các dòng và rời khỏi (1) số nguyên vẹn, thay đổi các dịch vụ khác mà bạn không sử dụng, ví dụ: Xenforo thành (0) vì bạn không muốn đưa các quy tắc này vào danh sách trắng. Ví dụ bên dưới:

```
SecAction \
"id:900130,\
phase:1,\
nolog,\
pass,\
t:none,\
setvar:tx.crs_exclusions_cpanel=0,\
setvar:tx.crs_exclusions_dokuwiki=0,\
setvar:tx.crs_exclusions_drupal=0,\
setvar:tx.crs_exclusions_nextcloud=0,\
setvar:tx.crs_exclusions_phpbb=1,\
setvar:tx.crs_exclusions_phpmyadmin=1,\
setvar:tx.crs_exclusions_wordpress=1,\
setvar:tx.crs_exclusions_xenforo=0"
```

Bạn cũng có thể sửa đổi cú pháp, cú pháp sẽ rõ ràng hơn. Ví dụ:

```
SecAction \
"id:900130,\
phase:1,\
nolog,\
pass,\
t:none,\
setvar:tx.crs_exclusions_phpbb=1,\
setvar:tx.crs_exclusions_phpmyadmin=1,\
setvar:tx.crs_exclusions_wordpress=1"
```

Như bạn có thể thấy, loại bỏ là các tùy chọn không cần thiết và được thêm vào (“) ở cuối WordPress để biết cú pháp chính xác.

** Loại trừ các quy tắc trước CRS **
Để đối phó với các loại trừ tùy chỉnh, trước tiên, bạn cần thay đổi tên từ YÊU CẦU-900-LOẠI TRỪ-QUY TẮC-TRƯỚC-CRS-SAMPLE.conf tập tin với lệnh cp như sau:

```
sudo cp /etc/nginx/modsec/coreruleset-3.4-dev/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example /etc/nginx/modsec/coreruleset-3.4-dev/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
```

Một điểm cần nhớ, khi tạo quy tắc loại trừ, mỗi quy tắc phải có id: và là duy nhất, nếu không khi bạn kiểm tra dịch vụ Nginx của mình, bạn sẽ gặp lỗi. Thí dụ “Id: 1544, phase: 1, log, allow, ctl: ruleEngine = off”, id 1544 không thể được sử dụng cho quy tắc thứ hai.

Ví dụ: một số REQUEST_URI sẽ làm tăng kết quả dương tính giả. Ví dụ dưới đây là hai với đèn hiệu tốc độ trang của Google và plugin WMUDEV dành cho WordPress:

```
SecRule REQUEST_URI "@beginsWith /wp-load.php?wpmudev" "id:1544,phase:1,log,allow,ctl:ruleEngine=off"
SecRule REQUEST_URI "@beginsWith /ngx_pagespeed_beacon" "id:1554,phase:1,log,allow,ctl:ruleEngine=off"
```

Như bạn có thể thấy, bất kỳ URL nào bắt đầu bằng đường dẫn sẽ được tự động cho phép.


Một tùy chọn khác là đưa các địa chỉ IP vào danh sách trắng, bạn có thể thực hiện một số cách sau:

```
SecRule REMOTE_ADDR "^195\.151\.128\.96" "id:1004,phase:1,nolog,allow,ctl:ruleEngine=off"
## or ###
SecRule REMOTE_ADDR "@ipMatch 127.0.0.1/8, 195.151.0.0/24, 196.159.11.13" "phase:1,id:1313413,allow,ctl:ruleEngine=off"
```

Mô hình @ipMatch có thể được sử dụng rộng rãi hơn cho các mạng con. Nếu bạn muốn từ chối một subnet or Địa chỉ IP thay đổi, cho phép từ chối. Với một số bí quyết, bạn cũng có thể tạo danh sách đen và danh sách trắng và định cấu hình điều này với fail2ban. Các khả năng thường có thể là vô tận.

Một ví dụ cuối cùng là chỉ tắt các quy tắc kích hoạt xác thực sai, không liệt kê toàn bộ đường dẫn vào danh sách trắng, như bạn đã thấy với ví dụ REQUEST_URI đầu tiên. Tuy nhiên, điều này cần nhiều thời gian và thử nghiệm hơn. Ví dụ: bạn muốn xóa các quy tắc 941000 và 942999 từ / admin / area của bạn vì nó tiếp tục kích hoạt các lệnh cấm và chặn sai cho nhóm của bạn, hãy tìm trong tệp nhật ký bảo mật modsecurity của bạn ID quy tắc và sau đó chỉ vô hiệu hóa ID đó với RemoveByID như ví dụ bên dưới:

```
SecRule REQUEST_FILENAME "@beginsWith /admin" "id:1004,phase:1,pass,nolog,ctl:ruleRemoveById=941000-942999"
```

Có thể tìm thấy các ví dụ trên ModSecurity GIT trang wiki; LinuxCapable trong tương lai sẽ tạo một bài hướng dẫn về phần này vì có khá nhiều thứ cần trình bày.

** Tùy chọn - Bao gồm Dự án Honeypot **

Dự án Honey Pot là hệ thống phân tán đầu tiên và duy nhất để xác định những kẻ gửi thư rác và các rô bốt gửi thư rác mà họ sử dụng để thu thập địa chỉ khỏi trang web của bạn. Sử dụng hệ thống Project Honey Pot, bạn có thể cài đặt các địa chỉ được gắn thẻ tùy chỉnh theo thời gian và địa chỉ IP của khách truy cập vào trang web của bạn. Nếu một trong những địa chỉ này bắt đầu nhận email, chúng tôi có thể biết rằng các thư đó là spam, thời điểm chính xác khi địa chỉ đó được thu thập và địa chỉ IP đã thu thập nó.

ModSecurity có thể có tùy chọn tích hợp Project Honeypot, nó sẽ truy vấn cơ sở dữ liệu và chặn bất kỳ địa chỉ nào nằm trong danh sách đen của HoneyPot. Lưu ý, sử dụng điều này có thể dẫn đến kết quả dương tính giả, nhưng nhìn chung nó được coi là rất đáng tin cậy so với các lựa chọn thay thế tương tự.

Bước 1. Tạo một tài khoản Tài khoản miễn phí.

Bước 2. Khi bạn đã đăng ký và đăng nhập, trên trang tổng quan, hãy tìm dòng (Khóa API http: BL của bạn) và nhấp lấy một cái.

Cách cài đặt ModSecurity với Nginx trên Rocky Linux 8
Bước 3. Quay lại tệp CRS-setup.conf bằng cách mở nó bằng trình soạn thảo văn bản:

```
sudo nano /etc/nginx/modsec/coreruleset-3.4-dev/crs-setup.conf
```

Bước 4. Tìm dòng bắt đầu bằng #SecHttpBlKey, nằm trên dòng 629.
```
#SecHttpBlKey XXXXXXXXXXXXXXXXX
#SecAction "id:900500,\
#  phase:1,\
#  nolog,\
#  pass,\
#  t:none,\
#  setvar:tx.block_search_ip=1,\
#  setvar:tx.block_suspicious_ip=1,\
#  setvar:tx.block_harvester_ip=1,\
#  setvar:tx.block_spammer_ip=1"
```

Bước 5. Thay đổi SecHttpBlKey XXXXXXXXXXXXXXXXXXX bằng khóa của bạn từ Project HoneyPot.

Ví dụ:
```
SecHttpBlKey amhektvkkupe
```

Bước 6. Tiếp theo, bỏ ghi chú tất cả các dòng để kích hoạt quy tắc. Nếu bạn muốn hủy kích hoạt một quy tắc, thay vì (1), hãy đặt (0) để tắt tùy chọn quét Project Honeypot.

Theo mặc định, block_search_ip = 0 là dành cho bot của công cụ tìm kiếm, không bật tính năng này trừ khi bạn muốn Bing, Google và các bot tốt khác đến với trang web của bạn.

```
SecHttpBlKey amhektvkkupe
SecAction "id:900500,\
phase:1,\
nolog,\
pass,\
t:none,\
setvar:tx.block_search_ip=0,\
setvar:tx.block_suspicious_ip=1,\
setvar:tx.block_harvester_ip=1,\
setvar:tx.block_spammer_ip=1"
```

Lưu ý, không sử dụng amhektvkkupe. Sử dụng chìa khóa của bạn để thay thế!

Bước 7. Kiểm tra Nginx để đảm bảo không có lỗi nào xảy ra với những điều sau:

```
sudo nginx -t
```

Ví dụ đầu ra nếu tất cả đều đúng:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Bây giờ hãy khởi động lại dịch vụ Nginx của bạn:

```
sudo systemctl restart nginx
```

## Bộ quy tắc WPRS của WordPress cho ModSecurity ##
Một lựa chọn khác cho WordPress người dùng phải cài đặt và chạy cùng với bộ quy tắc OWASP CRS của bạn, một dự án nổi tiếng có tên bộ quy tắc WPRS. Vì điều này là tùy chọn và không dành cho tất cả mọi người, nên hướng dẫn sẽ không đề cập đến nó trong phần này.

Tuy nhiên, nếu bạn muốn cài đặt điều này để bảo vệ thêm nếu bạn sử dụng WordPress trên máy chủ của mình, vui lòng truy cập hướng dẫn của chúng tôi về Cài đặt Bộ quy tắc bảo mật ModSecurity WordPress (WPRS).


Tạo tệp ModSecurity LogRotate
Với số lượng dòng và thông tin nó có thể ghi lại, ModSecurity sẽ phát triển khá nhanh chóng. Khi bạn đang biên dịch mô-đun và nó không được cài đặt thông qua bất kỳ kho lưu trữ chính thức nào từ Rocky Linux, bạn sẽ cần tạo tệp xoay nhật ký của riêng mình.

Đầu tiên, tạo và mở tệp xoay ModSecurity của bạn modsec:

Click To Copy!
sudo nano /etc/logrotate.d/modsec
Thêm mã sau:

Click To Copy!
/var/log/modsec_audit.log
{
        rotate 31
        daily
        missingok
        compress
        delaycompress
        notifempty
}
Điều này sẽ giữ nhật ký cho 31 ngày. Nếu bạn muốn có ít hơn, hãy thay đổi 31 để nói 7 số ngày bằng nhau của các bản ghi giá trị trong tuần. Bạn nên luân phiên hàng ngày cho ModSecurity. Nếu bạn cần xem lại các tệp nhật ký, thì một tệp hàng tuần sẽ là một thảm họa để sàng lọc, vì nó sẽ lớn như thế nào.

Tùy chọn - Bảo mật Nginx với Chứng chỉ miễn phí Let's Encrypt SSL
Lý tưởng nhất là bạn muốn chạy Nginx của mình trên HTTPS sử dụng chứng chỉ SSL. Cách tốt nhất để làm điều này là sử dụng Hãy mã hóa, một tổ chức phát hành chứng chỉ mở, tự động và miễn phí được điều hành bởi Nhóm nghiên cứu bảo mật Internet phi lợi nhuận (ISRG).

Đầu tiên, cài đặt EPEL kho lưu trữ và mod_ssl gói cho các gói được cập nhật tốt hơn và bảo mật.

Click To Copy!
sudo dnf install epel-release mod_ssl -y
Tiếp theo, cài đặt gói certbot như sau:

Click To Copy!
sudo dnf install python3-certbot-nginx -y
Sau khi cài đặt, hãy chạy lệnh sau để bắt đầu tạo chứng chỉ của bạn:

Click To Copy!
sudo certbot --nginx --agree-tos --redirect --hsts --staple-ocsp --email you@example.com -d www.example.com
Thiết lập lý tưởng này bao gồm chuyển hướng HTTPS 301 cưỡng bức, tiêu đề Nghiêm ngặt-Vận chuyển-Bảo mật và OCSP Stapling. Chỉ cần đảm bảo điều chỉnh e-mail và tên miền theo yêu cầu của bạn.

Bây giờ URL của bạn sẽ là HTTPS://www.example.com thay vì HTTP://www.example.com.


Nếu bạn sử dụng cái cũ URL HTTP, nó sẽ tự động chuyển hướng đến HTTPS.

Theo tùy chọn, bạn có thể đặt công việc cron để tự động gia hạn chứng chỉ. Certbot cung cấp một tập lệnh thực hiện điều này tự động và trước tiên bạn có thể kiểm tra để đảm bảo mọi thứ đang hoạt động bằng cách thực hiện chạy khô.

Click To Copy!
sudo certbot renew --dry-run
Nếu mọi thứ đang hoạt động, hãy mở cửa sổ crontab của bạn bằng lệnh terminal sau.

Click To Copy!
sudo crontab -e
Tiếp theo, chỉ định thời gian nó sẽ tự động gia hạn. Điều này phải được kiểm tra hàng ngày ở mức tối thiểu và nếu chứng chỉ cần được gia hạn, tập lệnh sẽ không cập nhật chứng chỉ. Sử dụng crontab.guru nếu bạn cần trợ giúp để tìm một thời điểm thích hợp để đặt.

Click To Copy!
00 00 */1 * * /usr/sbin/certbot-auto renew
Lưu (CTRL + O) sau đó thoát ra (CTRL + X), và cronjob sẽ tự động được kích hoạt.

Nhận xét và kết luận
Trong hướng dẫn này, bạn đã nắm được cách cài đặt nguồn Nginx, biên dịch ModSecurity và thiết lập Quy tắc OWASP giữa các phần trên cùng. Nhìn chung, việc triển khai ModSecurity cho máy chủ của bạn sẽ cung cấp khả năng bảo vệ tức thì.

Tuy nhiên, sự kiên nhẫn, thời gian và sự tận tâm trong học tập sẽ là một tính năng tuyệt vời. Điều cuối cùng bạn muốn là chặn bot SEO hoặc quan trọng hơn là người dùng thực có thể là khách hàng tiềm năng.


**Tài liệu tham khảo**
- [linuxcapable](https://www.linuxcapable.com/vi/c%C3%A1ch-c%C3%A0i-%C4%91%E1%BA%B7t-modsecurity-v%E1%BB%9Bi-nginx-tr%C3%AAn-Rock-linux-8/)