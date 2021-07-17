---
layout: post
comments: true
title:  "Cassandra - Giám sát Cassandra (Phần 2) - Cách thu thập các thông số (nodetool)"
title2:  "Giám sát Cassandra (Phần 2) - Cách thu thập các thông số (nodetool)"
date:   2021-06-12 10:15:00
permalink: 2021/06/12/Giam-sat-Cassandra-p2
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/Giam-sat-Cassandra-p2/Hinh1.jpg
summary: Cách xem các thông số trong hoạt động của Cassandra bằng nodetool
---

Trong [Phần 1](/2021/06/11/Giam-sat-Cassandra-p1), chúng ta đã biết về các thông số hiệu suất chính có sẵn trong Cassandra và trong phần này, chúng ta tiếp tục tìm hiểu cách thu thập các thông số này như thế nào?

Giống như Solr, Tomcat và các ứng dụng Java khác, Cassandra đưa ra các số liệu về tính khả dụng và hiệu suất thông qua JMX (Java Management Extensions). Kể từ phiên bản 1.1, các số liệu của Cassandra đã dựa trên [thư viện Metrics](https://github.com/dropwizard/metrics) phổ biến của Coda Hale, trong đó có rất nhiều tích hợp với các công cụ theo dõi và vẽ đồ thị. Có ít nhất ba cách để xem và theo dõi các số liệu của Cassandra, từ các tiện ích nhẹ nhưng hạn chế đến các dịch vụ được lưu trữ, đầy đủ tính năng:
- nodetool, một giao diện dòng lệnh đi kèm với Cassandra
- JConsole, một GUI đi kèm với Bộ phát triển Java (JDK)
- JMXTerm (CLI): giao diện dòng lệnh
- JMX/Metrics tích hợp với các công cụ và dịch vụ giám sát và vẽ đồ thị bên ngoài

## **1. Thu thập các chỉ số bằng nodetool** ##

Nodetool là một tiện ích dòng lệnh để quản lý và giám sát một cụm Cassandra. Nó có thể được sử dụng để kích hoạt các giao dịch theo cách thủ công, để chuyển dữ liệu trong bộ nhớ vào đĩa hoặc để đặt các thông số như kích thước bộ nhớ cache và ngưỡng nén. Nó cũng có một số lệnh trả về các chỉ số nút và cụm đơn giản có thể cung cấp thông tin nhanh về tình trạng cụm của bạn. Nodetool đi cùng với Cassandra và có trong thư mục 'bin' của Cassandra.

Chạy lệnh 'bin/nodetool status' từ thư mục mà bạn đã cài đặt Cassandra sẽ đưa ra một cái nhìn tổng quan về cụm, bao gồm cả **tải** hiện tại trên mỗi nút và trạng thái các nút riêng lẻ là up hay down:

```
$ bin/nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Owns    Host ID   Token                  Rack
UN  127.0.0.1  14.76 MB   66.7%   9e524995  -9223372036854775808   rack1
UN  127.0.0.1  14.03 MB   66.7%   12e12ead  -3074457345618258603   rack1
UN  127.0.0.1  13.92 MB   66.7%   44387d08   3074457345618258602   rack1
```

'nodetool info' đưa ra số liệu thống kê chi tiết hơn một chút cho một nút riêng lẻ trong cụm, bao gồm cả thời gian hoạt động, **tải**, **tỷ lệ truy cập bộ nhớ cache chính** và tổng số tất cả **ngoại lệ**. Bạn có thể chỉ định nút nào bạn muốn kiểm tra bằng cách sử dụng '--host' đối số với địa chỉ IP hoặc tên máy chủ:

```
$ bin/nodetool --host 127.0.0.1 info

ID                     : 9aa4fe41-c9a8-43bb-990a-4a6192b3b46d
Gossip active          : true
Thrift active          : false
Native Transport active: true
Load                   : 14.76 MB
Generation No          : 1449113333
Uptime (seconds)       : 527
Heap Memory (MB)       : 158.50 / 495.00
Off Heap Memory (MB)   : 0.07
Data Center            : datacenter1
Rack                   : rack1
Exceptions             : 0
Key Cache              : entries 26, size 2.08 KB, capacity 24 MB, 87 hits, 122 requests, 0.713 recent hit rate, 14400 save period in seconds
Row Cache              : entries 0, size 0 bytes, capacity 0 bytes, 0 hits, 0 requests, NaN recent hit rate, 0 save period in seconds
Counter Cache          : entries 0, size 0 bytes, capacity 12 MB, 0 hits, 0 requests, NaN recent hit rate, 7200 save period in seconds
Token                  : -9223372036854775808
```

'nodetool cfstats' cung cấp số liệu thống kê về từng keyspace và column family (tương tự như cơ sở dữ liệu và bảng cơ sở dữ liệu), bao gồm **độ trễ đọc**, **độ trễ ghi**, và **tổng dung lượng ổ đĩa được sử dụng**. Theo mặc định, nodetool in số liệu thống kê trên tất cả các keyspace và column families, nhưng bạn có thể giới hạn truy vấn trong một keyspace duy nhất bằng cách thêm tên của keyspace vào lệnh:

```
$ bin/nodetool cfstats demo

Keyspace: demo
    Read Count: 4
    Read Latency: 1.386 ms.
    Write Count: 4
    Write Latency: 0.71675 ms.
    Pending Flushes: 0
        Table: users
        SSTable count: 3
        Space used (live), bytes: 16178
        Space used (total), bytes: 16261
        ...
        Local read count: 4
        Local read latency: 1.153 ms
        Local write count: 4
        Local write latency: 0.224 ms
        Pending flushes: 0
        ...
```

'nodetool compactionstats' hiển thị các tác vụ nén đang xử lý cũng như số lượng tác vụ nén đang chờ xử lý (**pending compaction tasks**).

```
$ bin/nodetool compactionstats

pending tasks: 5
          compaction type        keyspace           table       completed           total      unit  progress
               Compaction       Keyspace1       Standard1       282310680       302170540     bytes    93.43%
               Compaction       Keyspace1       Standard1        58457931       307520780     bytes    19.01%
Active compaction remaining time :   0h00m16s
```

'nodetool gcstats' trả về thống kê về bộ thu gom rác, bao gồm tổng số số lượng bộ thu gom và thời gian trôi qua(cả tổng và thời gian đã trôi qua tối đa). Các bộ đếm được đặt lại mỗi khi lệnh được đưa ra, vì vậy số liệu thống kê chỉ tương ứng với khoảng thời gian giữa các lệnh 'gcstats'.

```
$ bin/nodetool gcstats

Interval (ms)  Max GC Elapsed (ms)  Total GC Elapsed (ms)  Stdev GC Elapsed (ms)  GC Reclaimed (MB)  Collections  Direct Memory Bytes
     73540574                   64                    595                      7         3467143560           83             67661338
```

'nodetool tpstats' cung cấp số liệu thống kê sử dụng luồng của Cassandra, bao gồm các tác vụ đang chờ xử lý cũng như các tác vụ hiện tại và lịch sử bị chặn.

```
$ bin/nodetool tpstats

Pool Name                    Active   Pending      Completed   Blocked  All time blocked
ReadStage                         0         0          11801         0                 0
MutationStage                     0         0         125405         0                 0
CounterMutationStage              0         0              0         0                 0
GossipStage                       0         0              0         0                 0
RequestResponseStage              0         0              0         0                 0
AntiEntropyStage                  0         0              0         0                 0
MigrationStage                    0         0             10         0                 0
MiscStage                         0         0              0         0                 0
InternalResponseStage             0         0              0         0                 0
ReadRepairStage                   0         0              0         0                 0
```

## Tài liệu tham khảo
- [datadoghq](https://www.datadoghq.com/blog/how-to-collect-cassandra-metrics/#collecting-metrics-with-nodetool)
- Mastering Apache Cassandra 3.x - Third Edition Ebook

---