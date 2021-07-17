---
layout: post
comments: true
title:  "Cassandra - Cấu hình trong Cassandra (Phần 2)"
title2:  "Cấu hình trong Cassandra (Phần 2)"
date:   2021-06-02 10:15:00
permalink: 2021/06/02/Cau-hinh-CSDL-Cassandra-p2
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/Cau-hinh-CSDL-Cassandra/Hinh1.jpg
summary: Cấu hình trong Cassandra
---

## 2. Cấu hình file cassandra-rackdc.properties

Một số tùy chọn snitch sử dụng cấu hình file <span style="color:#c7254e">cassandra-rackdc.properties</span> để xác định các datacenter và các nút cụm thuộc về rack. Thông tin về cấu trúc liên kết mạng cho phép các yêu cầu được định tuyến một cách hiệu quả và phân phối các bản sao đồng đều. Các snitches sau có thể được cấu hình tại đây:
- GossipingPropertyFileSnitch
- AWS EC2 snitch một vùng
- AWS EC2 snitch đa vùng 

GossipingPropertyFileSnitch được khuyên dùng để môi trường sản phẩm. Snitch này sử dụng datacenter và thông tin rack được cấu hình trong tệp <span style="color:#c7254e">cassandra-rackdc.properties</span> của nút cục bộ và truyền thông tin đến các nút khác bằng cách sử dụng gossip. Đây là snitch mặc định và các cài đặt trong file cấu hình này được bật.

Các snitches AWS EC2 được cấu hình cho các cụm trong AWS. Snitch này sử dụng các tùy chọn <span style="color:#c7254e">cassandra-rackdc.properties</span> để chỉ định một trong hai quy ước đặt tên datacenter AWS EC2 và rack:
- Kế thừa: Tên datacenter là một phần của tên vùng khả dụng đứng trước dấu "-" cuối cùng khi vùng kết thúc bằng -1 và bao gồm số nếu không phải -1. Tên rack là một phần của tên vùng khả dụng theo sau dấu "-" cuối cùng.
```Ví dụ: us-west-1a => dc: us-west, rack: 1a; us-west-2b => dc: us-west-2, rack: 2b;```

- Tiêu chuẩn: Tên datacenter là tên vùng AWS tiêu chuẩn, bao gồm cả số. Tên rack là khu vực cộng với ký tự khu vực khả dụng.
```Ví dụ: us-west-1a => dc: us-west-1, rack: us-west-1a; us-west-2b => dc: us-west-2, rack: us-west-2b;```

Snitch có thể đặt để sử dụng địa chỉ IP cục bộ hoặc nội bộ khi nhiều datacenter không giao tiếp với nhau.

### a. GossipingPropertyFileSnitch

<span style="color:#c7254e">dc</span>
- Tên của datacenter. Giá trị có phân biệt chữ hoa chữ thường.
- **Giá trị mặc định**: DC1

<span style="color:#c7254e">rack</span>
- Chỉ định rack. Giá trị có phân biệt chữ hoa chữ thường.
- **Giá trị mặc định**: RAC1

### b. Snitch AWS EC2

<span style="color:#c7254e">ec2_naming_scheme</span>
- Quy ước đặt tên datacenter và rack. Tùy chọn là <span style="color:#c7254e">legacy</span> hoặc <span style="color:#c7254e">standard</span> (mặc định). Tùy chọn này được nhận xét theo mặc định.
- **Giá trị mặc định**: standard

*Ghi chú: BẠN PHẢI SỬ DỤNG GIÁ TRỊ* <span style="color:#c7254e">*legacy*</span> *NẾU BẠN ĐANG NÂNG CẤP MỘT CỤM TRƯỚC PHIÊN BẢN 4.0.*

### c. Snitch

<span style="color:#c7254e">prefer_local</span>
- Tùy chọn sử dụng địa chỉ IP cục bộ hoặc nội bộ khi giao tiếp không qua các datacenter khác nhau. Tùy chọn này được nhận xét theo mặc định.
- Giá trị mặc định: true

## 3. Cấu hình file cassandra-env.sh

Tập tin bash script <span style="color:#c7254e">cassandra-env.sh</span> có thể được sử dụng để vượt qua tùy chọn bổ sung cho máy ảo Java (JVM), chẳng hạn như tối đa và tối thiểu kích thước heap, chứ không phải là thiết lập chúng trong môi trường. Nếu cài đặt JVM là tĩnh và không cần tính toán từ các đặc điểm của nút, thì các tệp tệp jvm-\* sẽ được sử dụng thay thế. Ví dụ: các giá trị thường được tính toán là kích thước heap, sử dụng các giá trị hệ thống.

Ví dụ: thêm <span style="color:#c7254e">JVM_OPTS="$JVM_OPTS -D cassandra.load_ring_state=false"</span> vào tệp <span style="color:#c7254e">cassandra-env.sh</span> và chạy dòng lệnh <span style="color:#c7254e">cassandra</span> để bắt đầu. Tùy chọn được đặt từ tệp <span style="color:#c7254e">cassandra-env.sh</span> và tương đương với việc khởi động Cassandra bằng tùy chọn dòng lệnh <span style="color:#c7254e">cassandra -D cassandra.load_ring_state=false</span>

Tùy chọn <span style="color:#c7254e">-D</span> xác định instance các thông số khởi động trong cả hai dòng lệnh và tập tin <span style="color:#c7254e">cassandra-env.sh</span>. Lựa chọn tiếp theo đã khả thi:

<span style="color:#c7254e">cassandra.auto_bootstrap=false</span>
- Tạo điều kiện thuận lợi cho việc đặt auto_bootstrap thành false khi thiết lập cụm ban đầu. Lần sau khi bạn khởi động cụm, bạn không cần phải thay đổi tệp <span style="color:#c7254e">cassandra.yaml</span> trên mỗi nút để hoàn nguyên về "true", giá trị mặc định.

<span style="color:#c7254e">cassandra.available_processors=&lt;number_of_processors&gt;</span>
- Khi triển khai nhiều trường hợp, nhiều trường hợp Cassandra sẽ tự giả định rằng tất cả các bộ xử lý CPU đều có sẵn cho nó. Cài đặt này cho phép bạn chỉ định một bộ vi xử lý nhỏ hơn.

<span style="color:#c7254e">cassandra.config=&lt;directory&gt</span>
- Vị trí thư mục chứa tệp <span style="color:#c7254e">cassandra.yaml</span>. Vị trí mặc định phụ thuộc vào loại cài đặt.

<span style="color:#c7254e">cassandra.ignore_dynamic_snitch_severity=true|false</span>
- Việc đặt thuộc tính này thành true sẽ khiến dynamic snitch bỏ qua chỉ báo mức độ nghiêm trọng từ các gossip khi tính điểm các nút. Xem thêm tính năng phát hiện và khôi phục lỗi cũng như tính năng tìm kiếm động để biết thêm thông tin.
- **Mặc định**: false

<span style="color:#c7254e">cassandra.initial_token=&lt;token&gt;</span>
- Sử dụng khi các nút ảo (vnodes) không được sử dụng. Đặt mã token phân vùng ban đầu cho một nút lần đầu tiên được khởi động. Lưu ý: Vnodes rất được khuyến khích vì chúng tự động chọn mã token.
- **Mặc định**: disabled

<span style="color:#c7254e">cassandra.join_ring=true|false</span>
- Đặt thành false để khởi động Cassandra trên một nút nhưng không để nút đó tham gia cụm. Bạn có thể sử dụng <span style="color:#c7254e">nodetool join</span> và gọi JMX để tham gia vòng sau đó.
- **Mặc định**: true

<span style="color:#c7254e">cassandra.load_ring_state=true|false</span>
- Đặt thành false để xóa tất cả trạng thái gossip cho nút khi khởi động lại.
- **Mặc định**: true

<span style="color:#c7254e">cassandra.metricsReporterConfigFile=&lt;filename&gt;</span>
- Bật báo cáo chỉ số có thể cắm vào được. Tìm hiểu báo cáo chỉ số có thể cắm thêm để biết thêm thông tin.

<span style="color:#c7254e">cassandra.partitioner=&lt;partitioner&gt;</span>
- Đặt bộ phân vùng.
- **Mặc định**: org.apache.cassandra.dht.Murmur3Partitioner

<span style="color:#c7254e">cassandra.prepared_statements_cache_size_in_bytes=&lt;cache_size&gt;</span>
- Đặt kích thước bộ nhớ cache cho các câu lệnh đã chuẩn bị.

<span style="color:#c7254e">cassandra.replace_address=&lt;listen_address of dead node&lt;|&lt;broadcast_address of dead node&gt;</span>
- Để thay thế một nút đã chết, hãy khởi động lại một nút mới ở vị trí của nó chỉ định <span style="color:#c7254e">listen_address</span> hoặc <span style="color:#c7254e">broadcast_address</span> mà nút mới đang giả định. Nút mới không được có bất kỳ dữ liệu nào trong thư mục dữ liệu của nó, trạng thái giống như trước khi khởi động. Lưu ý: Giá trị <span style="color:#c7254e">broadcast_address</span> mặc định thành <span style="color:#c7254e">listen_address</span> ngoại trừ khi sử dụng Ec2MultiRegionSnitch.

<span style="color:#c7254e">cassandra.replayList=&lt;table&gt;</span>
- Cho phép khôi phục các bảng cụ thể từ nhật ký cam kết đã lưu trữ.

<span style="color:#c7254e">cassandra.ring_delay_ms=&lt;number_of_ms&gt;</span>
- Xác định khoảng thời gian một nút chờ nghe từ các nút khác trước khi chính thức tham gia vòng.
- **Mặc định**: 1000ms

<span style="color:#c7254e">cassandra.native_transport_port=&lt;port&gt;</span>
- Đặt cổng mà phương tiện truyền tải gốc CQL (CQL native transport) lắng nghe máy khách.
- **Mặc định**: 9042

<span style="color:#c7254e">cassandra.rpc_port=&lt;port&gt;</span>
- Đặt cổng cho dịch vụ Thrift RPC, được sử dụng cho các kết nối máy khách.
- **Mặc định**: 9160

<span style="color:#c7254e">cassandra.storage_port=&lt;port&gt;</span>
- Đặt cổng cho giao tiếp giữa các nút.
- **Mặc định**: 7000

<span style="color:#c7254e">cassandra.ssl_storage_port=&lt;port&gt;</span>
- Đặt cổng SSL cho giao tiếp được mã hóa.
- **Mặc định**: 7001

<span style="color:#c7254e">cassandra.start_native_transport=true|false</span>
- Bật hoặc tắt máy chủ truyền tải gốc. Xem <span style="color:#c7254e">start_native_transport </span> trong <span style="color:#c7254e">cassandra.yaml</span>.
- **Mặc định**: true

<span style="color:#c7254e">cassandra.start_rpc=true|false</span>
- Bật hoặc tắt máy chủ Thrift RPC.
- **Mặc định**: true

<span style="color:#c7254e">cassandra.triggers_dir=&lt;directory&gt;</span>
- Đặt vị trí mặc định cho các trigger JAR.
- **Mặc định**: conf/trigger

<span style="color:#c7254e">cassandra.write_survey=true</span>
- Để thử nghiệm các chiến lược nén và nén mới. Nó cho phép bạn thử nghiệm với các chiến lược khác nhau và sự khác biệt về hiệu suất ghi điểm chuẩn mà không ảnh hưởng đến khối lượng công việc sản xuất.

<span style="color:#c7254e">consistent.rangemovement=true|false</span>
- Đặt thành true giúp Cassandra thực hiện bootstrap một cách an toàn mà không vi phạm tính nhất quán. False vô hiệu hóa điều này.

## 4. Cấu hình file cassandra-topologies.properties

Tùy chọn <span style="color:#c7254e">PropertyFileSnitch</span> snitch sử dụng cấu hình tập tin <span style="color:#c7254e">cassandra-topologies.properties</span> để xác định datacenter và rack thuộc về cụm nút. Nếu sử dụng các snitches khác, thì phải sử dụng: ref:cassandra_rackdc. Snitch xác định cấu trúc liên kết mạng (mức độ gần nhau của rack và datacenter) để các yêu cầu được định tuyến một cách hiệu quả và cho phép cơ sở dữ liệu phân phối các bản sao đồng đều.

Bao gồm mọi nút trong cụm trong tập tin thuộc tính, xác định tên datacenter của bạn như trong định nghĩa keyspace. Tên d785a534 và rack phân biệt chữ hoa chữ thường.

Tập tin <span style="color:#c7254e">cassandra-topologies.properties</span> phải được sao chép y hệt trên mỗi nút trong cluster.

**Ví dụ**
Ví dụ này sử dụng ba trung tâm dữ liệu:

```
# datacenter 1

175.56.12.105=DC1:RAC1
175.50.13.200=DC1:RAC1
175.54.35.197=DC1:RAC1

120.53.24.101=DC1:RAC2
120.55.16.200=DC1:RAC2
120.57.102.103=DC1:RAC2

# datacenter 2

110.56.12.120=DC2:RAC1
110.50.13.201=DC2:RAC1
110.54.35.184=DC2:RAC1

50.33.23.120=DC2:RAC2
50.45.14.220=DC2:RAC2
50.17.10.203=DC2:RAC2

# datacenter 3

172.106.12.120=DC3:RAC1
172.106.12.121=DC3:RAC1
172.106.12.122=DC3:RAC1

# mặc định cho nút chưa biết
default =DC3:RAC1
```

## 5. Cấu hình file commitlog-archiving.properties-file

Tệp cấu hình <span style="color:#c7254e">commitlog-archiving.properties</span> có thể tùy chọn đặt các lệnh được thực thi khi lưu trữ hoặc khôi phục một đoạn commitlog.

**Tùy chọn **
<span style="color:#c7254e">archive_command==&lt;command&gt;</span>
- Một lệnh có thể được chèn với các đối số %path và %name. %path là đường dẫn đủ điều kiện của phân đoạn commitlog để lưu trữ. %name là tên tệp của commitlog. Không thể thực hiện STDOUT, STDIN hoặc nhiều lệnh. Nếu cần nhiều lệnh, hãy thêm con trỏ vào một tập lệnh trong tùy chọn này.
- **Ví dụ**: archive_command=/bin/ln %path /backup/%name
- **Giá trị mặc định**: trống

<span style="color:#c7254e">restore_command==&lt;command&gt;</span>
- Một lệnh có thể được chèn với các đối số %from và %to. %from là đường dẫn đủ điều kiện đến một phân đoạn commitlog đã lưu trữ bằng cách sử dụng các thư mục khôi phục được chỉ định. %to xác định thư mục cho vị trí commitlog trực tiếp.
- **Ví dụ**: restore_command=/bin/cp -f %from %to
- **Giá trị mặc định**: trống

<span style="color:#c7254e">restore_directories==&lt;directory&gt;</span>
- Xác định thư mục để quét các tệp khôi phục vào.
- **Giá trị mặc định**: trống

<span style="color:#c7254e">restore_point_in_time==&lt;timestamp&gt;</span>
- Khôi phục đột biến được tạo lên đến và bao gồm dấu thời gian này theo GMT ở định dạng <span style="color:#c7254e">yyyy:MM:dd HH:mm:ss</span>. Quá trình khôi phục sẽ tiếp tục qua phân đoạn khi gặp phải dấu thời gian đầu tiên do máy khách cung cấp lớn hơn thời gian này, nhưng chỉ những đột biến nhỏ hơn hoặc bằng dấu thời gian này mới được áp dụng.
- **Ví dụ**: 2020:04:31 20:43:12
- **Giá trị mặc định**: trống

<span style="color:#c7254e">precision==&lt;timestamp_precision&gt;</span>
- Độ chính xác của thời gian được sử dụng trong các phần thêm dữ liệu. Thường lựa chọn là MILLISECONDS hoặc MICROSECONDS.
- **Giá trị mặc định**: MICROSECONDS

## 6. Cấu hình file logback.xml

Tệp cấu hình <span style="color:#c7254e">logback.xml</span> có thể tùy chọn đặt cấp độ ghi nhật ký cho các nhật ký được ghi vào <span style="color:#c7254e">system.log</span> và <span style="color:#c7254e">debug.log</span>. Các cấp độ ghi nhật ký cũng có thể được thiết lập bằng cách sử dụng <span style="color:#c7254e">nodetool setlogginglevels</span>

**Tùy chọn**
<span style="color:#c7254e">appender name="&lt;appender_choice&gt;"...&lt;/appender&gt;</span>
- Chỉ định loại nhật ký và cài đặt. Tên appender có thể là: <span style="color:#c7254e">SYSTEMLOG</span>, <span style="color:#c7254e">DEBUGLOG</span>, <span style="color:#c7254e">ASYNCDEBUGLOG</span>, và <span style="color:#c7254e">STDOUT</span>. <span style="color:#c7254e">SYSTEMLOG</span> đảm bảo rằng thông báo WARN và ERROR được ghi đồng bộ vào tệp được chỉ định. <span style="color:#c7254e">DEBUGLOG</span> và <span style="color:#c7254e">ASYNCDEBUGLOG</span> đảm bảo rằng các thông báo DEBUG được ghi đồng bộ hoặc không đồng bộ tương ứng vào tệp được chỉ định. <span style="color:#c7254e">STDOUT</span> ghi tất cả bản tin vào bảng điều khiển ở định dạng mà con người có thể đọc được.
- **Ví dụ**: <appender name="SYSTEMLOG" class="ch.qos.logback.core.rolling.RollingFileAppender">

<span style="color:#c7254e">&lt;file&gt; &lt;filename&gt; &lt;/file&gt;</span>
- Chỉ định tên tệp cho nhật ký.
- **Ví dụ**: <file>${cassandra.logdir}/system.log</file>

<span style="color:#c7254e">&lt;level&gt; &lt;log_level&gt; &lt;/level&gt;</span>
- Chỉ định mức độ cho một nhật ký. Một phần của bộ lọc. Các mức là: <span style="color:#c7254e">ALL, TRACE, DEBUG, INFO, WARN, ERROR, OFF</span>. <span style="color:#c7254e">TRACE</span> tạo ra nhật ký dài nhất, <span style="color:#c7254e">ERROR</span> ít nhất.
- Lưu ý: Việc tăng mức độ ghi nhật ký có thể tạo ra sản lượng ghi nhật ký lớn trên một cụm được quản lý vừa phải. Bạn có thể sử dụng lệnh <span style="color:#c7254e">nodetool getlogginglevels</span> để xem cấu hình ghi nhật ký hiện tại.
- **Mặc định**: INFO
- **Ví dụ**: <level>INFO</level>

<span style="color:#c7254e">&lt;rollingPolicy class="&lt;rolling_policy_choice&gt;" &lt;fileNamePattern&gt;&lt;pattern_info&gt;&lt;/fileNamePattern&gt; ... &lt;/rollingPolicy&gt;</span>
- Chỉ định chính sách để chuyển nhật ký vào lưu trữ.
- **Ví dụ**:  <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">

<span style="color:#c7254e">&lt;fileNamePattern&gt; &lt;pattern_info&gt; &lt;/fileNamePattern&gt;</span>
- Chỉ định thông tin mẫu để cuộn qua nhật ký để lưu trữ. Một phần của chính sách cuốn chiếu.
- **Ví dụ**: <fileNamePattern>${cassandra.logdir}/system.log.%d{yyyy-MM-dd}.%i.zip</fileNamePattern>

<span style="color:#c7254e">&lt;maxFileSize&gt; &lt;size&gt; &lt;/maxFileSize&gt;</span>
- Chỉ định kích thước tệp tối đa để kích hoạt cuộn nhật ký. Một phần của chính sách cuốn chiếu.
- **Ví dụ**: <maxFileSize>50MB</maxFileSize>

<span style="color:#c7254e">&lt;maxHistory&gt; &lt;number_of_days&gt; &lt;/maxHistory&gt;</span>
- Chỉ định lịch sử tối đa tính theo ngày để kích hoạt ghi nhật ký. Một phần của chính sách cuốn chiếu.
- **Ví dụ**: <maxHistory>7</maxHistory>

<span style="color:#c7254e">&lt;encoder&gt; &lt;pattern&gt;...&lt;/pattern&gt; &lt;/encoder&gt;</span>
- Chỉ định định dạng của bản tin. Một phần của chính sách cuốn chiếu.
- **Ví dụ**: <encoder> <pattern>%-5level [%thread] %date{ISO8601} %F:%L - %msg%n</pattern> </encoder>

Nội dung mặc định của <span style="color:#c7254e">logback.xml</span>

```config
  <configuration scan="true" scanPeriod="60 seconds">
    <jmxConfigurator />

    <!-- No shutdown hook; we run it ourselves in StorageService after shutdown -->

    <!-- SYSTEMLOG rolling file appender to system.log (INFO level) -->

    <appender name="SYSTEMLOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
<level>INFO</level>
      </filter>
      <file>${cassandra.logdir}/system.log</file>
      <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!-- rollover daily -->
        <fileNamePattern>${cassandra.logdir}/system.log.%d{yyyy-MM-dd}.%i.zip</fileNamePattern>
        <!-- each file should be at most 50MB, keep 7 days worth of history, but at most 5GB -->
        <maxFileSize>50MB</maxFileSize>
        <maxHistory>7</maxHistory>
        <totalSizeCap>5GB</totalSizeCap>
      </rollingPolicy>
      <encoder>
        <pattern>%-5level [%thread] %date{ISO8601} %F:%L - %msg%n</pattern>
      </encoder>
    </appender>

    <!-- DEBUGLOG rolling file appender to debug.log (all levels) -->

    <appender name="DEBUGLOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>${cassandra.logdir}/debug.log</file>
      <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!-- rollover daily -->
        <fileNamePattern>${cassandra.logdir}/debug.log.%d{yyyy-MM-dd}.%i.zip</fileNamePattern>
        <!-- each file should be at most 50MB, keep 7 days worth of history, but at most 5GB -->
        <maxFileSize>50MB</maxFileSize>
        <maxHistory>7</maxHistory>
        <totalSizeCap>5GB</totalSizeCap>
      </rollingPolicy>
      <encoder>
        <pattern>%-5level [%thread] %date{ISO8601} %F:%L - %msg%n</pattern>
      </encoder>
    </appender>

    <!-- ASYNCLOG assynchronous appender to debug.log (all levels) -->

    <appender name="ASYNCDEBUGLOG" class="ch.qos.logback.classic.AsyncAppender">
      <queueSize>1024</queueSize>
      <discardingThreshold>0</discardingThreshold>
      <includeCallerData>true</includeCallerData>
      <appender-ref ref="DEBUGLOG" />
    </appender>

    <!-- STDOUT console appender to stdout (INFO level) -->

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>INFO</level>
      </filter>
      <encoder>
        <pattern>%-5level [%thread] %date{ISO8601} %F:%L - %msg%n</pattern>
      </encoder>
    </appender>

    <!-- Uncomment bellow and corresponding appender-ref to activate logback metrics
    <appender name="LogbackMetrics" class="com.codahale.metrics.logback.InstrumentedAppender" />
     -->

    <root level="INFO">
      <appender-ref ref="SYSTEMLOG" />
      <appender-ref ref="STDOUT" />
      <appender-ref ref="ASYNCDEBUGLOG" /> <!-- Comment this line to disable debug.log -->
      <!--
      <appender-ref ref="LogbackMetrics" />
      -->
    </root>

    <logger name="org.apache.cassandra" level="DEBUG"/>
    <logger name="com.thinkaurelius.thrift" level="ERROR"/>
  </configuration>
```

## 7. Cấu hình file jvm-*

Một số tệp cho cấu hình JVM được bao gồm trong Cassandra. Tập tin <span style="color:#c7254e">jvm-server.options</span>, và các tập tin tương ứng <span style="color:#c7254e">jvm8-server.options</span> và <span style="color:#c7254e">jvm11-server.options</span> là những tập tin chính cho các thiết lập có ảnh hưởng đến hoạt động của Cassandra JVM trên các cụm nút. Tệp bao gồm các thông số khởi động, cài đặt JVM chung như thu gom rác và cài đặt heap. Các tệp <span style="color:#c7254e">jvm-clients.options</span> và <span style="color:#c7254e">vm8-clients.options</span>, <span style="color:#c7254e">jvm11-clients.options</span> tương ứng có thể được sử dụng để định cấu hình cài đặt JVM cho các ứng dụng máy khách như các công cụ <span style="color:#c7254e">nodetool</span> và <span style="color:#c7254e">sstable</span>.

Xem từng tệp để biết ví dụ về cài đặt.

**Ghi chú**
Các tệp jvm-\* thay thế tệp cassandra-env.sh được sử dụng trong các phiên bản Cassandra trước Cassandra 3.0. Các tập tin bash script cassandra-env.sh vẫn còn hữu ích nếu cài đặt JVM phải được tính toán động dựa trên các thiết lập hệ thống. Các tệp jvm-\* chỉ lưu trữ các cài đặt JVM tĩnh.

## Tài liệu tham khảo
- [Cassandra Documentation](https://cassandra.apache.org/doc/latest/configuration/index.html)


---