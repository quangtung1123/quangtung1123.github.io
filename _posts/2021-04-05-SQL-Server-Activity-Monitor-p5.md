---
layout: post
comments: true
title:  "SQL SERVER - SQL Server Activity Monitor (phần 5) – Câu truy vấn tốn tài nguyên nhất"
title2:  "SQL Server Activity Monitor (phần 5) – Câu truy vấn tốn tài nguyên nhất"
date:   2021-04-05 15:10:00
permalink: 2021/04/05/SQL-Server-Activity-Monitor-p5
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Activity-Monitor-p5/Hinh1.png
summary: Tìm hiểu SQL Server Activity Monitor

---

Như thế nào là một câu truy vấn tốn tài nguyên? Nếu bạn còn phân vân tài nguyên truy vấn là gì, các tiêu chí đánh giá và cách đo lường khi thực thi một câu truy vấn T-SQL thì hãy đọc "[đo lường hiệu suất truy vấn](https://quantricsdulieu.com/2021/01/06/hanh-trinh-dem-sao-4-do-luong-hieu-suat-truy-van-t-sql/)" trước khi tiếp tục bài viết này. Chỉ khi bạn hiểu thế nào là physical read, logical read/write chúng ta mới cùng xem Activity Monitor cung cấp những truy vấn tốn tài nguyên theo tiêu chí nào.

Các bạn chú ý, câu truy vấn tốn tài nguyên nhất khác với câu truy vấn hiệu năng thấp. Có thể trong SQL server của bạn câu truy vấn nào cũng có hiệu năng cao nhưng sẽ có câu truy vấn tốn tài nguyên nhất. Bản thân Activity Monitor không giúp chúng ta xác định những câu truy vấn tệ hại mà chỉ liệt kê những câu truy vấn nào tốn kém nhất trong khoảng thời gian theo dõi. Và danh sách này chính là những câu truy vấn bạn có thể hiệu chỉnh để cải thiện hiệu năng chung của SQL Server.

## **Bảng Recent Expensive Queries**

Mặc định bảng này sẽ liệt kê những câu truy vấn tốn tài nguyên nhất trong 20 giây vừa qua (mặc dù giá trị Refresh Interval mình set là 10). Từ giao diện chính bạn có thể click chuột phải lên bất kì câu truy vấn nào và chọn “Edit Query Text” để xem đầy đủ nội dung câu truy vấn. Hoặc chọn “Show Execution Plan” để xem [cách SQL Server thực thi câu truy vấn](https://quantricsdulieu.com/2021/01/27/hanh-trinh-dem-sao-6-execution-plan-la-gi/) này. Các cột cho phần này là:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p5/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Danh sách các câu truy vấn tốn tài nguyên CPU nhất trong 10 giây vừa qua.</div>
</div>
<hr>
Mình sử dụng script đếm số row từng bảng trong database stackoverflow cho demo để có kết quả như hình trên.

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

Click trên tiêu đề (header) của cột nào sẽ sắp xếp theo tiêu chí của cột đó, các cột trong bảng này có ý nghĩa như sau:
- **Query** – Câu lệnh truy vấn đang bị theo dõi.
- **Executions/min** – Số lần thực thi trong 1 phút của câu truy vấn này.
- **CPU (ms/sec**) – Giá trị trung bình tài nguyên CPU dùng bởi câu truy vấn này trong 1 giây.
- **Physical Reads/sec** – Giá trị trung bình lượng physcial read trong 1 giây.
- **Logical Writes/sec** – Giá trị trung bình lượng logical write trong 1 giây.
- **Logical Reads/sec** – Giá trị trung bình lượng logical read trong 1 giây.
- **Average Duration (ms)** – Giá trị trung bình thời gian mỗi lần thực thi.
- **Plan Count** – Số lượng query plan của câu truy vấn dạng này trong plan cache.

## **Activity Monitor xác định câu truy vấn tốn tài nguyên nhất như thế nào?**

Chúng ta lại sử dụng SQL Profiler như [bài trước](/2021/03/13/SQL-Server-Activity-Monitor-p3) sẽ thấy cách tổ chức và thu thập thông tin giống như Wait Statistics. Activity Monitor sẽ tạo bảng tạm #am_get_querystats và stored procedure để mỗi lần thực thi lấy data từ hai DMVs [sys.dm_exec_query_stats](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql?view=sql-server-ver15) và [sys.dm_exec_requests](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver15) tính toán rồi đổ vào bạn tạm đó. Lần sau thực thi sẽ lấy data mới trừ data cũ sẽ ra data phát sinh trong khoảng interval 10 giây. DMV sys.dm_exec_query_stats lưu lại các thông số về hiệu suất các lần thực thi trước của một câu truy vấn, còn sys.dm_exec_requests chứa thông tin về request đang được thực thi.

Sử dụng Activity Monitor và hiểu được cách nó thu thập danh sách những câu truy vấn tốn tài nguyên này giúp chúng ta hiểu hơn về nguyên nhân tạo ra [workload](/2021/03/11/SQL-Server-Activity-Monitor-p1) tương ứng (trong 10 giây vừa rồi) trong SQL Server. Sắp xếp theo tiêu chí tài nguyên mà bạn quan tâm (CPU, logical read, logical write, duration) rồi chọn một trong những câu truy vấn và áp dụng các kĩ thuật tối ưu truy vấn sau đó quan sát workload để thấy sự khác biệt. Thông thường top 3-5 câu truy vấn tốn kém nhất sử dụng phần lớn tài nguyên của server.

Bài sau chúng ta sẽ tìm hiểu bảng cuối cùng trong Activity Monitor – Active Expensive queries. Đây là nơi cho bạn biết những câu truy vấn nào thật sự đang được thực thi. Hẹn gặp lại các bạn.

Nguồn: [quantricsdulieu](https://quantricsdulieu.com/2021/03/20/tim-hieu-sql-server-activity-monitor-p5-truy-van-ton-tai-nguyen/)

---
