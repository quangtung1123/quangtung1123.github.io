---
layout: post
comments: true
title:  "SQL SERVER - SQL Server Activity Monitor (phần 2) – Blocking"
title2:  "SQL Server Activity Monitor (phần 2) – Blocking"
date:   2021-03-12 15:10:00
permalink: 2021/03/12/SQL-Server-Activity-Monitor-p2
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Activity-Monitor-p2/Hinh1.png
summary: Tìm hiểu SQL Server Activity Monitor

---

Sau khi bạn nắm được [tình trạng hiện tại](/2021/03/11/SQL-Server-Activity-Monitor-p1) của SQL Server, có thể nó đang có lượng request nhiều hơn bình thường hoặc database I/O tăng cao bất thường, hoặc cũng có thể số lượng process trong biểu đồ waiting tasks nhiều quá. Bước tiếp theo bạn nên làm là nhìn vào bảng Processes để tìm xem thành phần nào có khả năng gây ra những hiện tượng này.

## Bảng Processes

Chức năng của bảng Processes này là cung cấp chi tiết về những processes đang chạy dưới SQL Server. Nó sẽ cho bạn biết có bao nhiêu requests? Những requests này đến từ client nào? Connect vào database gì? Thực hiện việc UPDATE hay SELECT? Trạng thái hiện tại như thế nào? Hiệu suất truy vấn có hiệu quả hay không (thông qua thời gian chờ)? Có tranh chấp tài nguyên lẫn nhau không? Và sử dụng bao nhiêu memory? Bạn có thể lấy được nhiều thông tin từ bảng này để giải đáp cho kết quả bạn thấy trên các biểu đồ của [bảng Overview](/2021/03/11/SQL-Server-Activity-Monitor-p1).

Giả sử bạn thấy Waiting tasks tăng cao và muốn biết nó là do tình huống **blocking** hay là wait type (sẽ đề cập bên dưới) khác. Nếu bạn chưa quen thuộc với thuật ngữ blocking thì hãy xét ví dụ sau. Process A cần tài nguyên là data của pageID 178 và yêu cầu SQL Server cấp phát lock (ví dụ lock X) trên page này để truy cập, nhưng SQL Server không thể cấp phát vì process B đã giữ lock X (xung đột với request của A) nên process A phải vào hàng đợi, chờ process B thực hiện xong, ta nói **process A bị blocked bởi process B hoặc process B block process A**.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p2/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Thông tin các process đang chạy dưới SQL Server.</div>
</div>
<hr>

- Session ID – Mỗi một connection được khởi tạo đến SQL Server sẽ được gán một giá trị gọi là [session_id](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql?view=sql-server-ver15).
- User Process – Chỉ định process này là của user hay của system. Có giá trị là 0 nếu là process của system process. Mặc định khi bạn mở Activity Monitor bảng Processes chỉ lấy những process của user với giá trị bằng 1.
- Login – Tên của login đang sử dụng session này.
- Database – Tên của database mà connection trỏ tới.
- Task State – Trạng thái của [tasks](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-tasks-transact-sql?view=sql-server-ver15), có các giá trị như PENDING, RUNABLE, RUNNING, SUSPENDED, DONE, SPINLOOP.
- Command – Loại [command](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver15) mà task đang được thực thi. Sẽ là một trong các giá trị SELECT, INSERT, UPDATE, DELETE, BACKUP DATABASE.
- Application – Tên ứng dụng connect tới SQL Server trên session này. Trên máy lab của mình thường thấy SSMS, SQLQueryStress.
- Wait Time (ms) – Tổng thời gian chờ ([wait_duration_ms](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql?view=sql-server-ver15) ) tài nguyên mà task này đã thực hiện, theo đơn vị milliseconds.
- Wait Type – Mỗi tình huống chờ sẽ được SQL Server gán cho một [tên](https://www.sqlskills.com/help/waits/) để phân biệt, có thể là WRITE_LOG, PAGEIOLATCH_SH
- Wait Resource – Mô tả về tài nguyên mà session này đang chờ ([resource_description](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql?view=sql-server-ver15) ).
- Blocked By – Proccess này có bị blocked bởi process nào hay không, nếu có thì session_id của process đó sẽ hiển thị ở cột này.
- Head Blocker – Nếu process A này block process B và không bị blocked bởi một process nào khác thì giá trị này là 1.
- Memory Use (KB) – Lượng memory cấp phát cho câu lệnh T-SQL của process này.
- Host Name – Tên machine (client) thực hiện connection này.
- Workload Group – Tên workload group của [Resource Governor](https://docs.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver15) mà session này được gán vào.

## Theo dõi tình huống blocking trên Activity Monitor

Bây giờ chúng ta hãy tạo ra một tình huống blocking như sau. Để giống với ví dụ ở trên ta có process B thực hiện update data của bảng Users nhưng chưa commit transaction. Tiếp đó process A truy vấn dòng dữ liệu này nhưng lock không tương thích nên chưa được cấp phát và phải đợi process B. Ta sẽ thấy process B block process A. Tiếp nữa process C nhảy vào thực hiện việc update trên cùng dòng dữ liệu này nhưng khác cột, ta hãy xem Activity Monitor sẽ thể hiện như thế nào. Các bạn chạy ba đoạn code dưới đây trên ba session khác nhau trong SSMS, theo đúng thứ tự đề cập trong ví dụ.

Process B update tuổi của user có Id bằng 1.

```sql
USE StackOverflow2010
GO
 
BEGIN TRANSACTION
 
UPDATE users 
SET Age = 18 
WHERE Id = 1 
 
-- COMMIT
-- ROLLBACK
```

Process A truy vấn thông tin của user có Id bằng 1.

```sql
USE StackOverflow2010
GO
 
SELECT * 
FROM users WHERE Id = 1
GO
```

Process C tăng Reputation của user Id 1 lên một đơn vị.

```sql
USE StackOverflow2010
GO
 
UPDATE users 
SET Reputation = Reputation + 1 
WHERE id = 1 
```

Theo các bạn process C có bị blocked hay không? Nếu có thì bị blocked bởi process nào?
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p2/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: Phân tích blocking issue trên Activity Monitor</div>
</div>
<hr>

Kết quả trên máy lab của mình cho thấy process C bị blocked bởi process A, mặc dù A chưa được cấp phát lock để thực thi. Trong hình trên session_id 64 là process B, chạy update bảng Users và vì mình không chạy lệnh **COMMIT** nên nó chưa thật sự kết thúc câu lệnh update. Kế tiếp session_id 57 là process A truy vấn data của user có Id bằng 1, nhưng process này phải đợi lock **LCK_M_S** (cột Wait Type) và đánh dấu là bị blocked bởi session_id 64. Kế tiếp session_id 55 là process C vào update giá trị Reputation cho user có Id bằng 1, nhưng kết quả lại bị block bởi session_id 57 (process A) và có Wait Type là **LCK_M_X**. Trong chuỗi blocking này 64–>57–>55 thì session_id 64 dẫn đầu nên các bạn thấy cột Head Blocker của session_id 64 bật lên 1.

Đây là một tình huống blocking thường thấy trong các hệ thống OLTP, trong thực tế chuỗi blocking có thể lên đến hàng chục thậm chí hàng trăm sessions tham gia. Từ kết quả phân tích như trên các bạn phát hiện session nào là header và tập trung phân tích vào nó để tìm giải pháp xử lý.

Nguồn: https://quantricsdulieu.com/2021/03/06/tim-hieu-sql-server-activity-monitor-phan-2-blocking/

---
