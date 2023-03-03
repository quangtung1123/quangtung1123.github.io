---
layout: post
title:  "Wazuh cơ bản"
date:   2023-02-08 10:48:00
permalink: 2023/02/08/wazuh-co-ban
tags: Log Wazuh
category: Log
img: /assets/wazuh-co-ban/Hinh1.jpg
summary: Wazuh cơ bản

---

Wazuh là một phần mềm bảo mật mã nguồn mở, cung cấp các chức năng sau:

- Giám sát hệ thống: Giám sát các tập tin và thông tin hệ thống để phát hiện bất thường.
- Phát hiện tấn công: Sử dụng các quy tắc và cảnh báo để phát hiện các hoạt động tấn công.
- Bảo vệ dữ liệu: Giám sát và bảo vệ dữ liệu quan trọng trên hệ thống.
- Giám sát mạng: Giám sát hoạt động mạng để phát hiện các hoạt động tấn công và vi phạm an ninh.
- Bảo mật cục bộ: Giám sát các hoạt động trên hệ thống và các dịch vụ cục bộ để phát hiện bất thường.
- Tích hợp với các công cụ bảo mật khác: Tích hợp với các công cụ bảo mật khác như Snort, Suricata, v.v. để cung cấp một giải pháp bảo mật toàn diện.
- Quản lý ghi nhận sự cố: Lưu trữ và quản lý các ghi nhận sự cố bảo mật.
- Thống kê và báo cáo: Tạo ra các báo cáo và thống kê về hoạt động bảo mật.
- Tích hợp với ELK Stack: Tích hợp với ELK Stack để cung cấp một giải pháp tổng quan và đồng bộ hóa cho việc giám sát và báo cáo sự cố bảo mật.
- Hỗ trợ cho các hệ điều hành và môi trường mạng khác nhau: Wazuh hỗ trợ cho các hệ điều hành Linux, Windows và các môi trường mạng như cloud, IoT v.v.
- Tự động hóa quy trình bảo mật: Tự động hóa các quy trình bảo mật và tự động gửi cảnh báo khi phát hiện sự cố.
- Hỗ trợ cho các nền tảng bảo mật mã nguồn mở: Wazuh hỗ trợ cho các nền tảng bảo mật mã nguồn mở như OSSEC, OpenSCAP và SELinux.
- Cung cấp API: Cung cấp một API mạnh mẽ để tích hợp với các hệ thống bảo mật khác.
- Hỗ trợ cho các nền tảng cloud: Wazuh hỗ trợ cho các nền tảng cloud như AWS, Azure và Google Cloud.
- Hỗ trợ cho các nền tảng container: Wazuh hỗ trợ cho các nền tảng container như Docker và Kubernetes.


Wazuh có thể sử dụng IIS (Internet Information Services) logs để phát hiện các hoạt động bất thường trên hệ thống. Để cấu hình việc giám sát IIS logs, bạn cần:

- Cấu hình Agent:
	- Cài đặt Wazuh Agent trên máy chủ chạy IIS.
	- Cấu hình Agent để giám sát các tệp logs của IIS.
- Cấu hình Manager:
	- Cấu hình Manager để nhận các logs từ Agent.
	- Sử dụng các rule để phát hiện các hoạt động bất thường trong IIS logs. Bạn có thể tùy chỉnh các rule để phù hợp với nhu cầu của mình và cấu hình các alert để nhận được thông báo khi phát hiện bất thường.

Lưu ý: Bạn cần có kiến thức về IIS, Wazuh và cách cấu hình các rule trong Wazuh để cấu hình giám sát IIS logs chính xác.


Wazuh cung cấp một số rule mặc định để phát hiện các hoạt động bất thường trong IIS logs. Bạn có thể sử dụng các rule sau để phát hiện các hoạt động cụ thể:

- Phát hiện lạc lối của máy khách: Rule này sẽ phát hiện các hoạt động lạc lối của máy khách, chẳng hạn như truy cập vào một địa chỉ IP không hợp lệ hoặc gửi yêu cầu tới một địa chỉ IP sai.

- Phát hiện tấn công SQL Injection: Rule này sẽ phát hiện các hoạt động tấn công SQL Injection, chẳng hạn như sử dụng các ký tự đặc biệt trong các yêu cầu.

- Phát hiện tấn công Cross-Site Scripting (XSS): Rule này sẽ phát hiện các hoạt động tấn công Cross-Site Scripting (XSS), chẳng hạn như sử dụng các mã độc trong các yêu cầu.

Phát hiện tấn công Remote File Inclusion (RFI): Rule này sẽ phát hiện các hoạt động tấn công Remote File Inclusion (RFI), chẳng hạn như sử dụng các đường dẫn URL sai trong các yêu cầu.


**Các bước cấu hình Forward log IIS vào Wazuh 4.3 như sau:**

- Mở tệp cấu hình "ossec.conf" và thêm các dòng sau để cấu hình Forward IIS Logs:

```
<localfile>
  <log_format>iis</log_format>
  <location>C:\inetpub\logs\LogFiles\W3SVC1\u_ex*.log</location>
</localfile>
```

Lưu và khởi động lại dịch vụ Wazuh để áp dụng thay đổi.
Lưu ý: Đường dẫn đến tệp log IIS trong cấu hình có thể khác nhau tùy vào phiên bản và cấu hình của IIS mà bạn đang sử dụng.

Trong Wazuh, mức của một rule định nghĩa mức độ nghiêm trọng của một sự kiện được ghi nhận. Wazuh hỗ trợ một số mức rule, mỗi mức có một mức độ nghiêm trọng khác nhau:

- Low: Mức độ thấp nhất, dùng cho thông tin chi tiết và đầy đủ, có thể giúp trong việc gỡ lỗi.

- Medium: Thông tin hữu ích, dùng để theo dõi hoạt động hệ thống thông thường.

- High: Sự kiện không quan trọng nhưng vẫn cần được ghi nhận, nhưng không cần được quan tâm ngay lập tức.

- Critical: Sự kiện nghiêm trọng cho thấy có vấn đề với hệ thống, nhưng hệ thống vẫn có thể hoạt động.

- Emergency: Sự kiện nghiêm trọng cho thấy một sự cố lớn, và hệ thống có thể không hoạt động đúng.

Trong Wazuh, các mức của rule được xác định bằng các số từ 0 đến 15, với mức cao nhất là 15 và mức thấp nhất là 0.

Số lượng của mức của rule có thể được chỉ định trong tệp cấu hình Wazuh, /var/ossec/etc/ossec.conf, như sau:

```
<rules>
  <level>2</level>
</rules>
```

Trong ví dụ này, mức của rule được thiết lập là 2, tương đương với mức "medium".

Các mức của rule trong Wazuh được liệt kê dưới đây:

0 - None
1 - Low
2 - Medium
3 - High
4 - Critical
5 - Emergency
6-15 - Custom levels (Tùy chỉnh)

Wazuh cung cấp một công cụ quản lý và giám sát bảo mật mã nguồn mở cho hệ thống và mạng. Để cập nhật các quy tắc mới trong Wazuh, bạn cần thực hiện các bước sau:

- Tải xuống các quy tắc mới từ trang web của Wazuh hoặc từ nguồn khác.

- Chỉnh sửa các tệp quy tắc để phù hợp với môi trường của bạn.

- Sao lưu các tệp quy tắc cũ để tránh mất dữ liệu trong trường hợp có lỗi xảy ra.

- Đặt các tệp quy tắc mới vào thư mục quy tắc của Wazuh và khởi động lại dịch vụ Wazuh.

- Kiểm tra xem các quy tắc mới đã được kích hoạt và hoạt động như mong đợi bằng cách sử dụng công cụ giám sát và quản lý của Wazuh.

Lưu ý: Trước khi cập nhật các quy tắc, hãy chắc chắn rằng bạn đã hiểu rõ về những thay đổi mà những quy tắc mới sẽ gây ra và các hậu quả của nó đối với môi trường bảo mật của bạn.

**Cơ sở dữ liệu GeoIP**
Wazuh sử dụng cơ sở dữ liệu GeoIP của MaxMind để cung cấp thông tin địa lý cho địa chỉ IP. Cơ sở dữ liệu này có thể được cập nhật bằng cách tải về phiên bản mới từ trang web của MaxMind và cấu hình Wazuh để sử dụng cơ sở dữ liệu đã cập nhật.

Dưới đây là các bước để cập nhật cơ sở dữ liệu GeoIP trong Wazuh:

- Tải cơ sở dữ liệu GeoIP: Bạn có thể tải về phiên bản mới của cơ sở dữ liệu GeoIP từ trang web của MaxMind tại https://www.maxmind.com/.

- Sao chép cơ sở dữ liệu đến máy chủ Wazuh: Sau khi tải về cơ sở dữ liệu GeoIP, hãy sao chép nó đến máy chủ Wazuh. Đề nghị đặt cơ sở dữ liệu trong thư mục "/var/ossec/geoip".

- Cấu hình Wazuh để sử dụng cơ sở dữ liệu đã cập nhật: Bạn cần chỉnh sửa tệp cấu hình của Wazuh để cho Wazuh biết sử dụng cơ sở dữ liệu GeoIP đã cập nhật. Tệp cấu hình nằm ở "/var/ossec/etc/ossec.conf". Tìm phần "geoip" trong tệp và cập nhật đường dẫn đến cơ sở dữ liệu đến đúng vị trí.

- Khởi động lại dịch vụ Wazuh: Sau khi cập nhật tệp cấu hình, bạn cần khởi động lại dịch vụ Wazuh để áp dụng các thay đổi. Bạn có thể thực hiện việc này bằng cách chạy lệnh "systemctl restart wazuh-manager".

**Định nghĩa ngưỡng cảnh báo.**
Mỗi event trên Wazuh Agent được đặt severity level mặc định là 1. Tất cả các event level này tăng lên sẽ tạo ra cảnh báo trên Wazuh Manager.
Ngưỡng cảnh báo được cấu hình trên ossec.conf file như sau:

```
<ossec_config>
  <alerts>
      <log_alert_level>6</log_alert_level>
  </alerts>
</ossec_config>
```

Cấu hình này sẽ đặt severity level nhỏ nhất để tạo cảnh báo, mà sẽ được lưu trữ tại alerts.log và/hoặc trên các file alerts.json

**Tự động tạo báo cáo**
Bạn có thể cấu hình để tự động tạo báo cáo hàng ngày với option report trong osssec.conf. Để cấu hình gửi email thông báo, tham khảo tại cấu hình email và cấu hình SMTP server

```
<ossec_config>
  <reports>
      <category>syscheck</category>
      <title>Daily report: File changes</title>
      <email_to>example@test.com</email_to>
  </reports>
</ossec_config>
```

Cấu hình trên sẽ gửi báo cáo hàng ngày của tất cả các syscheck alert tới email example@test.com

Các rule có thể được lọc bằng level, source, username, rule id... :

```
<ossec_config>
  <reports>
      <level>10</level>
      <title>Daily report: Alerts with level higher than 10</title>
      <email_to>example@test.com</email_to>
  </reports>
</ossec_config>
```

Cấu hình trên sẽ gửi báo cáo với tất cả các rule với level cảnh báo lớn hơn 10.

**Cấu hình gửi email**
Để cấu hình Wazuh gửi email cảnh báo, các thiết lập email phải được cấu hình trong section global ở osssec.conf file :

```
<ossec_config>
    <global>
        <email_notification>yes</email_notification>
        <email_to>me@test.com</email_to>
        <smtp_server>mail.test.com..</smtp_server>
        <email_from>wazuh@test.com</email_from>
    </global>
    ...
</ossec_config>
```

Khi các giá trị trên được cấu hình, email_alert_level cần được đặt với level cảnh báo nhỏ nhất để tạo email. Mặc định, level là 12:

```
<ossec_config>
  <alerts>
      <email_alert_level>10</email_alert_level>
  </alerts>
  ...
</ossec_config>
```

Ví dụ này sẽ đặt level nhỏ nhất tới 10.

**Cập nhật lỗ hổng**
Wazuh cập nhật thông tin về lỗ hổng từ các nguồn tin cậy và chuyên nghiệp như National Vulnerability Database (NVD) và Common Vulnerabilities and Exposures (CVE). Wazuh sử dụng công cụ tự động để theo dõi các bản cập nhật mới nhất về lỗ hổng và tự động cập nhật vào cơ sở dữ liệu của hệ thống.

Việc cập nhật thông tin về lỗ hổng là rất quan trọng để đảm bảo rằng hệ thống của bạn được bảo vệ chặt chẽ với các lỗ hổng mới nhất và được cập nhật theo thời gian thực.

Bạn có thể kiểm tra cơ sở dữ liệu lỗ hổng đã được cập nhật bản mới nhất hay chưa bằng cách sử dụng lệnh sau trong Wazuh Manager:

```
$ sudo wazuh-manager check_update
```

Lệnh này sẽ kiểm tra các bản cập nhật mới nhất và liệt kê các lỗ hổng đã được cập nhật và các lỗ hổng mới. Nếu có bản cập nhật mới, hệ thống sẽ tự động cập nhật vào cơ sở dữ liệu.

Lưu ý: Bạn cần có quyền root hoặc cấp quyền sudo để chạy lệnh này.

**Bổ sung hệ điều hành để quét lỗ hổng**
Để bổ sung hệ điều hành mà không có trong danh sách mặc định của Wazuh ta làm như sau:
- Mở file cấu hình "/var/ossec/etc/ossec.conf" và thêm ten hệ điều hành vào nhóm phân phối của hệ điều hành. Ví dụ: thêm AlmaLinux 9.1 vào nhóm phân phối redhat với tham số allow.

```
 <provider name="redhat">
      <enabled>yes</enabled>
      <os>5</os>
      <os>6</os>
      <os>7</os>
      <os>8</os>
      <os allow="AlmaLinux-9">9</os>
      <update_interval>1h</update_interval>
    </provider>
````

- Khởi động lại dịch vụ Wazuh bằng cách chạy lệnh "systemctl restart wazuh-manager".

Lưu ý: nếu có nhiều hệ điều hành thì danh sách các hệ điều hành được ngăn cách nhau bằng dấu "," (comma).

**Tích hợp với VirusTotal**
- Đăng ký một tài khoản VirusTotal: Bạn cần đăng ký một tài khoản VirusTotal để có thể sử dụng API của VirusTotal.
- Cấu hình trên Wazuh Manager: Integrator được cấu hình trong file ossec.conf. Thêm các cấu hình sau bên trong section <ossec_config> </ossec_config> (phía trên của dòng <!-- Osquery integration -->):

```
<integration>
  <name>virustotal</name>
  <api_key>VirusTotal_API_Key</api_key>
  <group>syscheck,</group>
</integration>
```
- Khởi động lại dịch vụ Wazuh bằng cách chạy lệnh "systemctl restart wazuh-manager".
- Kích hoạt module qua giao diện web: Vào menu Settings/Modules, tìm đến VirusTotal và enable lên.

Lưu ý: với cấu hình trên thì Wazuh sẽ quét toàn bộ file theo group syscheck. Nếu muốn quét file theo rule_id thì thay như sau:
```
  <!-- virustotal integration -->
  <integration>
    <name>virustotal</name>
    <api_key>e621d288327fe01699b1719d741e27b3abb935ee8122dc70fb68af6ac1a522ad</api_key> <!-- Replace with your VirusTotal API key -->
    <!-- <group>syscheck</group>-->
    <rule_id>752</rule_id>
    <alert_format>json</alert_format>
  </integration>
```

Ví dụ trên sẽ quét toàn bộ file mới được thêm vào server (rule_id: 752). Các file và rule_id sẽ tương ứng trong Modules Integrity monitoring.

Một số bộ lọc như sau:

```
<!-- Optional filters -->
  <rule_id> </rule_id>
  <level> </level>
  <group> </group>
  <event_location> </event_location>
```

**MITRE ATT&CK** là một bộ công cụ phân tích bảo mật mã nguồn mở do MITRE Corporation phát triển. Nó cung cấp một bộ sưu tập các kỹ thuật tấn công và các thủ tục bất thường được sử dụng trong các cuộc tấn công và các hoạt động bất thường trên mạng. Nó cũng cung cấp các công cụ phân tích và cảnh báo để giúp bạn phát hiện các cuộc tấn công và các thủ tục bất thường trên mạng của bạn.


**Tài liệu tham khảo**
- [ChatGPT]()