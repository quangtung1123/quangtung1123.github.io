---
layout: post
comments: true
title:  "SQL SERVER - SQL Server Activity Monitor (phần 1)"
title2:  "SQL Server Activity Monitor (phần 1)"
date:   2021-03-11 15:10:00
permalink: 2021/03/11/SQL-Server-Activity-Monitor-p1
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Activity-Monitor-p1/Hinh3.png
summary: Tìm hiểu SQL Server Activity Monitor

---

Để biết được SQL Server đang làm gì sẽ là một thách thức không hề nhẹ, đặc biệt nếu bạn không phải là quản trị viên cơ sở dữ liệu. Bên cạnh việc xác định những câu truy vấn nào đang thực thi, chúng ta còn có thể biết được mức độ ảnh hưởng của chúng lên hiệu năng hiện tại của server. [SQL Server Activity Monitor](https://docs.microsoft.com/en-us/sql/relational-databases/performance-monitor/activity-monitor?redirectedfrom=MSDN&view=sql-server-ver15) là một chức năng trong SQL Server Management Studio, nó giúp theo dõi những tiến trình đang diễn ra như thế nào và tài nguyên hệ thống được sử dụng ra sao. Bài viết hôm nay chúng ta sẽ cùng tìm hiểu về công cụ Activity Monitor này.

## Bảng điều khiển SQL Server Activity Monitor
Có hai cách để mở bản điều khiển này, hoặc là bạn click vào icon trên tool bar như hình 1. Hoặc bạn click chuột phải vào instance name và chọn Activity Monitor trong cửa sổ Objects Explorer như trong hình 2.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Mở Activity Monitor từ icon trên tool bar.</div>
</div>
<hr>

<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: Mở Activity Monitor từ cửa sổ Object Explorer.</div>
</div>
<hr>

Sau khi chọn một trong hai cách trên, bạn sẽ thấy bảng điều khiển chính của Activity Monitor hiện lên và bắt đầu tải các dữ liệu liên quan. Có 6 bảng con lần lượt là Overview, Processes, Resource Waits, Data File I/O, Recent Expensive Queries, và Active Expensive Queries. Các bảng con này có thể mở rộng hoặc thu gọn cho tiện việc theo dõi và chỉ những bảng nào được chọn (mở rộng) SQL Server Activity Monitor mới tải data. Mặc định là bảng điều khiển con Overview được mở như hình dưới.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh3.png" width = "800">
</div>
<div class="thecap">Hình 3: Giao diện chính của Activity Monitor.</div>
</div>
<hr>

SQL Server Activity Monitor thu thập thông tin từ nhiều nguồn nhưng chủ yếu là truy vấn trực tiếp trên instance mà bạn đang theo dõi. Bạn có thể click chuột phải vào vùng trống trong bảng con Overview để thiết lập giá trị “Refresh interval” từ 1 giây đến 1 giờ cho cả bảng điều khiển chính. Mình chọn 1 giây để dễ nhận thấy sự thay đổi khi demo, Activity Monitor sẽ cập nhật thông tin trên dashboard sau mỗi giây.

## Quyền hạn cần thiết để sử dụng SQL Server Activity Monitor
Vì Activity Monitor truy vấn các data tổng quan ở cấp độ instance nên login cần có quyền VIEW SERVER STATE. Bên cạnh đó, các bảng điều khiển còn lại thu thập các thông tin về database I/O, nội dung các câu truy vấn nên sẽ cần thêm quyền CREATE DATABASE, ALTER ANY DATABASE, và VIEW ANY DEFINITION. Vì một số bảng hỗ trợ thêm các chức năng như trace process cụ thể bằng profiler, kill process nào đó nên sẽ cần có thêm các quyền tương ứng như ALTER TRACE và sysadmin role để kill process.

Để việc sử dụng dashboard này dễ dàng và hiệu quả, mình sẽ giải thích ý nghĩa từng thông số cùng với ví dụ minh họa để các bạn thấy được những tình huống, hành động nào tác động làm tăng hoặc giảm các thông số tương ứng để sau này chỉ nhìn vào SQL Server Activity Monitor các bạn có thể mường tượng được SQL Server bạn theo dõi đang chịu tải như thế nào.

## Bảng Overview
Bảng này gồm các đồ thị thể hiện những thông số tổng quan nhưng cũng cực kỳ quan trọng giúp chúng ta hình dung được trạng thái của SQL Server hiện tại thế nào? đang hoạt động ra sao?
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh4.png" width = "800">
</div>
<div class="thecap">Hình 4: Bảng Overview.</div>
</div>
<hr>

- % Processor Time – Phần trăm thời gian CPUs đang được dùng để thực thi các tiến trình. Số này càng cao hệ thống càng bận rộn.
- Waiting Tasks – Số lượng các tasks (có thể xem là các worker threads) đang phải chờ tài nguyên như locks, memory, I/O hoặc chờ CPU. Đối với bản thân mỗi worker thread, thời gian chờ giống như thời gian chết. Nếu có nhiều waiting tasks và kéo dài chứng tỏ hệ thống hoạt động chưa hiệu quả.
- Database I/O – Lượng data được transfer qua lại mỗi giây (tính theo đơn vị MB) giữa memory và disk hoặc giữa các disk với nhau.
- Batch Requests/sec – Số lượng requests SQL Server nhận được mỗi giây. Đây là thông số quan trọng để đo mức độ hoạt động của một instance. Con số này càng lớn đòi hỏi quản trị viên càng chú ý theo dõi, vận hành instance đó kĩ hơn.

## % Processor Time
Hẳn các bạn quá quen thuộc với thông số này rồi, % Processor Time là tỷ lệ phần trăm CPUs được sử dụng để thực thi các tác vụ trong server, giá trị này là của cả OS chứ không phải riêng SQL Server. Các request đến SQL Server phần lớn bị giới hạn bởi thời gian xử lý I/O ( I/O inbound) hơn là CPU, tuy nhiên trong các hệ thống OLTP với lượng memory đủ lớn để cache data và số lượng requests đến SQL Server cũng đủ lớn thì tài nguyên CPUs có thể trở thành điểm nghẽn (CPU bottleneck).

Database mình dùng demo là stackoverflow2010 với kích thước sau khi restore xấp xỉ 10GB, con lab mình sử dụng có cấu hình 4 cores (8 threads), 16GB memory và ổ cứng SSD. SQL Server maximum server memory được thiết lập 12GB, đủ để chứa cả database trong bộ nhớ cache. Câu truy vấn được dùng cho demo như sau

```sql
USE StackOverflow2010
GO
 
SELECT COUNT(1) FROM    Comments
SELECT COUNT(1) FROM    Votes
SELECT COUNT(1) FROM    Comments
SELECT COUNT(1) FROM    Posts
SELECT COUNT(1) FROM    Badges
SELECT COUNT(1) FROM    Users
SELECT COUNT(1) FROM    PostLinks
GO 10
```

Lệnh GO 10 ở trên sẽ giúp cả batch được thực thi 10 lần. Kết quả sau 2 lần F5 trên lab của mình như bên dưới. Nếu bạn chú ý sẽ thấy chỉ có thông tin về % Processor Time là nhảy vọt lên cao khoản 40%, các thông số khác hầu như rất thấp ( < 10).
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh5.png" width = "800">
</div>
<div class="thecap">Hình 5: % Processor Time.</div>
</div>
<hr>

## Waiting Tasks
Waiting là việc một process chờ một tài nguyên nào đó để có thể tiếp tục thực thi, và tất nhiên trong khi chờ tài nguyên process này sẽ có trạng thái là suspended (không dùng CPU time). Ở trên mình có đề cập các tài nguyên như CPU, I/O, memory và locks. Đây chỉ là những tài nguyên nói chung, thực chất SQL Server chia nhỏ các tài nguyên trên thành rất nhiều thành phần nhỏ hơn mà bạn có thể tìm hiểu ở phần [wait statistics](https://www.sqlskills.com/blogs/paul/wait-statistics-or-please-tell-me-where-it-hurts/). Phần demo này mình sẽ cho các bạn thấy một số process phải chờ khóa (lock) và chúng sẽ xuất hiện ở danh sách các waiting tasks.

Mình sẽ sử dụng công cụ SQLQueryStress để giả lập nhiều request truy vấn data bảng Users, trong khi đó một process ở SSMS đang update một record của bảng và chưa kết thúc transaction của nó thành ra các process trên phải chờ. Bạn thực thi đoạn code sau trên SSMS:

```sql
USE StackOverflow2010
GO
 
BEGIN TRANSACTION
 
UPDATE users SET age = 30 WHERE id = -1 
 
--- ROLLBACK
```

Sau đó bạn hãy chạy SQLQueryStress với cấu hình như sau
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh6.png" width = "800">
</div>
<div class="thecap">Hình 6: SQLQueryStress với 7 threads giả lập.</div>
</div>
<hr>

Và kết quả là có 7 processes đang chờ tài nguyên lock trên bảng Users. Lần này chỉ có thông số waiting tasks tăng cao, các thông số khác đều thấp tè sát đáy.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh7.png" width = "800">
</div>
<div class="thecap">Hình 7: Kết quả của những processes đang chờ locks.</div>
</div>
<hr>

<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh8.png" width = "800">
</div>
<div class="thecap">Hình 8: Kết quả từ stored procedure sp_whoisactive.</div>
</div>
<hr>

Các bạn hãy nhớ rollback transaction trên SSMS trước khi chúng ta qua thông số tiếp theo.

## Database I/O
Vì SQL Server buffer pool (data cache) trên lab của mình đang được cấu hình 12GB, đủ để chứa nguyên database stackoverflow2010 (10GB) nên chúng ta sẽ không thấy data transfer giữa disk và memory. Hãy chỉnh giá trị này về 3GB và chúng ta truy vấn data của tất cả các bảng trong database để xem việc transfer data diễn ra thế nào nhé.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh9.png" width = "800">
</div>
<div class="thecap">Hình 9: Set maximum server memory về 3GB.</div>
</div>
<hr>

Sau đó mình dùng lại câu truy vấn ở demo đầu tiên, lần này mình mở 2 cửa sổ query trên SSMS nhằm giả lập 2 connections để tăng áp lực cho việc load data lên memory. Các bạn chú ý, vì script demo của mình đọc data từ tất cả các bảng trong database (10GB) và dung lượng data cache chỉ gần 3GB (vì phải dùng cho một số thành phần khác như plan cache) nên SQL Server buộc phải clean data đã đọc và load data mới lên cache liên tục để phục vụ các câu truy vấn.

```sql
USE StackOverflow2010
GO
 
SELECT COUNT(1) FROM    Comments
SELECT COUNT(1) FROM    Votes
SELECT COUNT(1) FROM    Comments
SELECT COUNT(1) FROM    Posts
SELECT COUNT(1) FROM    Badges
SELECT COUNT(1) FROM    Users
SELECT COUNT(1) FROM    PostLinks
GO 10
```

<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh10.png" width = "800">
</div>
<div class="thecap">Hình 10: Database I/O thể hiện việc transfer data giữa memory và disk, hơn 2GB/sec.</div>
</div>
<hr>

Chúng ta thấy ngoài thông số Database I/O thì % Processor Time cũng nhảy lên cao. Waiting tasks xảy ra là những lúc hai process này phải chờ data load từ disk lên memory.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh11.png" width = "800">
</div>
<div class="thecap">Hình 11: Các processes chờ tài nguyên I/O.</div>
</div>
<hr>

## Batch Requests/sec
Số lượng requests SQL Server nhận trong 1 giây. Để giả lập được nhiều requests nhất có thể ( với sức mạnh có hạn của máy lab) mình sẽ chỉ dùng câu lệnh đơn giản là SELECT @@SERVERNAME. Và chúng ta lại cần sự hỗ trợ của SQLQueryStress để có thể giả lập lượng request này.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh12.png" width = "800">
</div>
<div class="thecap">Hình 12: SQLQueryStress giả lập 8 threads lặp 1000 lần.</div>
</div>
<hr>

<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p1/Hinh13.png" width = "800">
</div>
<div class="thecap">Hình 13: Batch Requests/sec tăng cao.</div>
</div>
<hr>

Các bạn thấy với sự hỗ trợ của SQLQueryStress chúng ta đã giả lập được hơn 200 requests gửi đến SQL Server trong mỗi giây. Với những hệ thống lớn thực tế, lượng request này có thể lên đến vài chục ngàn.

Phần 1 của loạt bài tìm hiểu SQL Server Activity Monitor này mình đã giới thiệu cho các bạn khái niệm, ý nghĩa và những tình huống cụ thể tác động đến một vài thông số quan trọng trong việc xác định tải của SQL Server. Hiểu được khi nào những giá trị này cao, khi nào chúng thấp sẽ giúp bạn giới hạn phạm vi tìm hiểu trong các tình huống xử lý sự cố về hiệu năng SQL Server.

Nguồn: https://quantricsdulieu.com/2021/03/05/tim-hieu-sql-server-activity-monitor/

---
