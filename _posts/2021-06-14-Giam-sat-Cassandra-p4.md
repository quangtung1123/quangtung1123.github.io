---
layout: post
comments: true
title:  "Cassandra - Giám sát Cassandra (Phần 4) - Cách thu thập các thông số (JMXTerm)"
title2:  "Giám sát Cassandra (Phần 4) - Cách thu thập các thông số (JMXTerm)"
date:   2021-06-14 10:15:00
permalink: 2021/06/14/Giam-sat-Cassandra-p4
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/Giam-sat-Cassandra-p4/Hinh1.png
summary: Cách xem các thông số trong hoạt động của Cassandra bằng JMXTerm
---

Phần trước, chúng ta đã tìm hiểu cách thu thập số liệu bằng JConsole, phần này sẽ tiếp tục thu thập số liệu sử dụng JMXTerm

## **3. Thu thập số liệu sử dụng JMXTerm (CLI)** ##

Không giống JConsole, JMXTerm là một tiện ích dòng lệnh cho bất kỳ tiến trình Java nào với giao diện JMX để xem các số liệu và thực hiện các thao tác để quản lý ứng dụng. Nó chỉ là một công cụ dòng lệnh, tương tự như JConsole, mà không có giao diện đồ họa. Nói cách khác, nó là một dòng lệnh thay thế cho jconsole. JMXTerm dựa vào thư viện jconsole trong thời gian chạy. JMXTerm cũng có thể được nhúng với ngôn ngữ lập trình không phải Java như Perl, Shell và Python, đồng thời cho phép các ngôn ngữ này truy cập máy chủ Java MBean theo lập trình. Tham khảo tài liệu JMXTerm để biết thêm thông tin tại [Apache-Confluence: JMXTerm Quick start](https://cwiki.apache.org/confluence/display/KAFKA/jmxterm+quickstart).

**Thực thi Uberjar có thể được sử dụng như**:
- Bảng điều khiển tương tác: nếu không có bất kỳ tùy chọn nào, nó sẽ mở ra một bảng điều khiển dòng lệnh tương tác.
- Trình thông dịch tệp: với tùy chọn -i, nó xử lý một tệp kịch bản (script ) nhất định và cuối cùng sẽ thoát ra.
- Nhúng: nó có thể được gọi từ ngôn ngữ kịch bản như SHELL hoặc PERL.

Tải [JMXTerm jar](https://docs.cyclopsgroup.org/jmxterm/) sau đó chạy lệnh để kết nối vào máy chủ tương ứng:
```
java -jar <absolutePath>/jmxterm-<VERSION>-uber.jar
open <IP>:7199
```

Ví dụ:
```
jiaqi@happycow:~$ java -jar jmxterm-1.0.2-uber.jar

Welcome to JMX terminal. Type "help" for available commands.

?$

Once you are in the console, help command shows all available commands and help with command name or command with -h option shows the usage of each command. For example



> help get

usage: get [-b <val>] [-d <val>] [-h] [-i] [-q] [-s]

Get value of MBean attribute(s)

-b,--bean <val> MBean name where the attribute is. Optional if bean has

been set

-d,--domain <val> Domain of bean, optional

-h,--help Display usage

-i,--info Show detail information of each attribute

-q,--quots Quotation marks around value

-s,--simple Print simple expression of value without full expression

* stands for all attributes. eg. get Attribute1 Attribute2 or get *
```

Sử dụng lệnh domains để lấy danh sách các gói MBean được Cassandra hiển thị:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Giam-sat-Cassandra-p4/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

**Lấy giá trị thông số**

Để lấy một số liệu, chẳng hạn như DownEnpointCount trong gói org.apache.Cassandra.net, lệnh *get* có thể được sử dụng như dưới. Thông số này thường được sử dụng để đếm số lượng nút bị ngắt và đưa vào cảnh báo.
```
get -b org.apache.cassandra.net:type=FailureDetector DownEndpointCount
```

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Giam-sat-Cassandra-p4/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

**Thực hiện một hành động**

Chúng ta có thể thực hiện một hành động thông qua JMXTerm. Ví dụ: bạn có thể gọi hoạt động stopGossiping trong gói org.apache.Cassandra.db bằng cách sử dụng lệnh chạy dưới, điều này sẽ ngừng nói chuyện phiếm với các nút khác.
```
run -b org.apache.cassandra.db: type = StorageService stopGossiping
```
Kết quả như sau:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Giam-sat-Cassandra-p4/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Thay vì tương tác bằng CLI, bạn có thể chạy danh sách lệnh bằng tệp, như sau:
```
echo -e "open <IP>: 7199 \ nrun -b org.apache.cassandra.db: type = StorageService forceTerminaAllRepairSessions \ nclose"> jmxcommands
java -jar <absolutePath> /jmxterm-1.0.0.-uber.jar -v im lặng -n <jmxcommands
```

Việc dừng quá trình sửa chữa đang chạy cũng tương tự trên bất kỳ nút sản xuất nào. Nhưng việc gọi loại thao tác trước đó sẽ dừng quá trình sửa chữa một cách mềm mại mà không cần khởi động lại Cassandra. Điều này có thể được sử dụng vào đầu giờ làm việc để giảm tác động của quá trình sửa chữa.


**Ví dụ nhúng trong Script**

***PERL***

```perl
# This Perl script open connection and call domains

# $jar is the path of jmxterm uber jar file

open JMX, "| java -jar $jar -n";

print JMX "help \n";

my $host = "localhost";

my $port = 9991;

print JMX "open $host:$port\n";

print JMX "domains\n";

print JMX "close\n";

close JMX;
```

***SHELL***

Ví dụ sau là gọi Jmxterm từ shell. Lệnh cụ thể này hữu ích khi bạn muốn lấy PID của tiến trình JVM lắng nghe trên cổng JMX đã biết. 

```shell
$ echo get -s -b java.lang:type=Runtime Name | \

java -jar target/jmxterm-1.0.2-uber.jar \

    -l localhost:9991 -v silent -n

11383@happycow
```

## **4. Thu thập chỉ số thông qua tích hợp JMX/Metrics** ##

Nodetool và JConsole đều nhẹ và có thể cung cấp cái nhìn nhanh các chỉ số rất nhanh chóng, nhưng cả hai đều không phù hợp với các loại câu hỏi lớn nảy sinh trong môi trường sản xuất: Xu hướng dài hạn cho các chỉ số là gì? Có bất kỳ mẫu quy mô lớn nào mà tôi nên biết không? Các thay đổi trong số liệu hiệu suất có xu hướng tương quan với các hành động hoặc sự kiện ở nơi khác trong môi trường của tôi không?

Để trả lời những câu hỏi kiểu này, bạn cần một hệ thống giám sát phức tạp hơn. Tin tốt là hầu như mọi dịch vụ và công cụ giám sát chính đều hỗ trợ giám sát Cassandra, cho dù thông qua plugin JMX ; thông qua các thư viện báo cáo Metrics có thể cắm được ; hoặc thông qua các trình kết nối ghi số liệu JMX ra StatsD, Graphite hoặc các hệ thống khác.

Các bước cấu hình phụ thuộc rất nhiều vào các công cụ giám sát cụ thể mà bạn chọn, nhưng cả JMX và Metrics đều hiển thị các chỉ số Cassandra bằng cách sử dụng phân loại được nêu trong bảng đường dẫn JMX ở trên.

## **Phần kết luận** ##

Trong bài đăng này, chúng tôi đã đề cập đến một số cách để truy cập số liệu của Cassandra bằng các công cụ đơn giản, nhẹ. Để giám sát sẵn sàng sản xuất, bạn có thể sẽ muốn có một hệ thống giám sát mạnh mẽ hơn để nhập các chỉ số Cassandra cũng như các chỉ số chính từ tất cả các công nghệ khác trong ngăn xếp của bạn.

## Tài liệu tham khảo
- [datadoghq](https://www.datadoghq.com/blog/how-to-collect-cassandra-metrics/#collecting-metrics-with-nodetool)
- Mastering Apache Cassandra 3.x - Third Edition Ebook

---