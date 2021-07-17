---
layout: post
comments: true
title:  "Cassandra - Giám sát Cassandra (Phần 3) - Cách thu thập các thông số (JConsole)"
title2:  "Giám sát Cassandra (Phần 3) - Cách thu thập các thông số (JConsole)"
date:   2021-06-13 10:15:00
permalink: 2021/06/13/Giam-sat-Cassandra-p3
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/Giam-sat-Cassandra-p3/Hinh1.jpg
summary: Cách xem các thông số trong hoạt động của Cassandra bằng nodetool
---

Phần trước, chúng ta đã tìm hiểu cách thu thập số liệu bằng nodetool, phần này sẽ tiếp tục thu thập số liệu sử dụng JConsole

## **2. Thu thập số liệu sử dụng JConsole** ##

JConsole là một Java GUI đơn giản đi kèm với bộ phát triển Java (JDK). Nó cung cấp một giao diện để khám phá đầy đủ các số liệu mà Cassandra cung cấp thông qua JMX. Nếu JDK đã được cài đặt vào một thư mục trong đường dẫn hệ thống của bạn, bạn có thể khởi động JConsole đơn giản bằng cách chạy lệnh:

```
jconsole
```

Nếu không, nó có thể được tìm thấy trong thư mục 'your_JDK_install_dir/bin'

Để lấy số liệu trong JConsole, bạn có thể chọn tiến trình cục bộ có liên quan hoặc theo dõi tiến trình từ xa bằng địa chỉ IP của nút (Cassandra sử dụng cổng 7199 cho JMX theo mặc định):
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Giam-sat-Cassandra-p2/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Tab MBeans hiển thị tất cả các đường dẫn JMX có sẵn:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Giam-sat-Cassandra-p2/Hinh2.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Ngoài ra, org.apache.cassandra.metrics (dựa trên [thư viện Metrics](https://github.com/dropwizard/metrics)) cung cấp gần như tất cả các số liệu mà bạn cần để theo dõi một cụm Cassandra. (Xem chú thích đầu tiên trên bảng dưới đây cho trường hợp ngoại lệ.) Trước khi Cassandra 2.2, nhiều số liệu giống hệt hoặc tương tự cũng có sẵn thông qua đường dẫn JMX thay thế (org.apache.cassandra.db, org.apache.cassandra.internal, vv), trong đó, trong khi vẫn có thể sử dụng trong một số phiên bản, phản ánh một cấu trúc cũ đã không được dùng nữa. Dưới đây là các đường dẫn JMX hiện đại, phản ánh cấu trúc thư mục của giao diện JConsole, cho các số liệu chính được mô tả trong bài viết này:

|  Metric  | JMX path  |
|  :----:  |  :----:  |
|  Throughput (writes\|reads)  |  org.apache.cassandra.metrics: type=ClientRequest,scope=(Write\|Read),name=Latency Attribute: OneMinuteRate  |
|  Latency (writes\|reads)*  |  org.apache.cassandra.metrics: type=ClientRequest,scope=(Write\|Read),name=TotalLatency Attribute: Count org.apache.cassandra.metrics: type=ClientRequest,scope=(Write\|Read),name=Latency Attribute: Count  |
|  Key cache hit rate*  |  org.apache.cassandra.metrics: type=Cache,scope=KeyCache,name=Hits Attribute: Count org.apache.cassandra.metrics: type=Cache,scope=KeyCache,name=Requests Attribute: Count  |
|  Load  |  org.apache.cassandra.metrics: type=Storage,name=Load Attribute: Count  |
|  Total disk space used  |  org.apache.cassandra.metrics: type=ColumnFamily,keyspace=KeyspaceName),scope=(ColumnFamilyName),name=TotalDiskSpaceUsed Attribute: Count  |
|  Completed compaction tasks  |  org.apache.cassandra.metrics: type=Compaction,name=CompletedTasks Attribute: Value  |
|  Pending compaction tasks  |  org.apache.cassandra.metrics: type=Compaction,name=PendingTasks Attribute: Value  |
|  ParNew garbage collections (count\|time)  |  java.lang: type=GarbageCollector,name=ParNew Attribute: (CollectionCount\|CollectionTime)  |
|  CMS garbage collections (count\|time)  |  java.lang: type=GarbageCollector,name=ConcurrentMarkSweep Attribute: (CollectionCount\|CollectionTime)  |
|  Exceptions  |  org.apache.cassandra.metrics: type=Storage,name=Exceptions Attribute: Count  |
|  Timeout exceptions (writes\|reads)  |  org.apache.cassandra.metrics: type=ClientRequest,scope=(Write\|Read),name=Timeouts Attribute: Count  |
|  Unavailable exceptions (writes\|reads)  |  org.apache.cassandra.metrics: type=ClientRequest,scope=(Write\|Read),name=Unavailables Attribute: Count  |
|  Pending tasks (per stage)**  |  org.apache.cassandra.metrics: type=ThreadPools,path=request,scope=CounterMutationStage\|MutationStage\|ReadRepairStage\|ReadStage\|RequestResponseStage), name=PendingTasks Attribute: Value  |
|  Currently blocked tasks**  |  org.apache.cassandra.metrics: type=ThreadPools,path=request,scope=(CounterMutationStage\|MutationStage\|ReadRepairStage\|ReadStage\|RequestResponseStage), name=CurrentlyBlockedTasks Attribute name: Count  |


/* Các số liệu cần thiết để theo dõi độ trễ gần đây và tỷ lệ truy cập bộ nhớ đệm chính có sẵn trong JConsole, nhưng phải được tính toán từ hai số liệu riêng biệt. Ví dụ về độ trễ đọc, các chỉ số liên quan là ReadTotalLatency (tổng độ trễ đọc tích lũy, tính bằng micro giây) và thuộc tính "Count" của ReadLatency (số lượng sự kiện đọc). Đối với hai lần đọc tại thời điểm 0 và 1, độ trễ đọc gần đây sẽ được tính toán từ delta của hai chỉ số đó:

```
(ReadTotalLatency1−ReadTotalLatency0)/(ReadLatency1−ReadLatency0)
```

/** Có năm giai đoạn yêu cầu khác nhau trong Cassandra, cộng với khoảng một chục giai đoạn nội bộ, mỗi giai đoạn có chỉ số nhóm luồng riêng.


## Tài liệu tham khảo
- [datadoghq](https://www.datadoghq.com/blog/how-to-collect-cassandra-metrics/#collecting-metrics-with-nodetool)
- Mastering Apache Cassandra 3.x - Third Edition Ebook

---