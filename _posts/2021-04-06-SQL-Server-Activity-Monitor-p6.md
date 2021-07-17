---
layout: post
comments: true
title:  "SQL SERVER - Tìm hiểu SQL Server Activity Monitor (phần 6) – Câu truy vấn đang diễn ra"
title2:  "SQL Server Activity Monitor (phần 6) – Câu truy vấn đang diễn ra"
date:   2021-04-06 15:10:00
permalink: 2021/04/06/SQL-Server-Activity-Monitor-p6
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Activity-Monitor-p6/Hinh1.png
summary: Tìm hiểu SQL Server Activity Monitor
---

Tại sao Activity Monitor đã có [bảng Processes](/2021/03/12/SQL-Server-Activity-Monitor-p2) thể hiện những connection hiện có đến SQL Server lại cần thêm bảng Active Expensive Queries này làm gì? Đúng như tiêu đề bài viết đề cập, chỉ những câu truy vấn đang diễn ra mới lọt vào bảng theo dõi này. Session vẫn còn giữ kết nối mặc dù không còn request nào từ client và khi đó nó có status là **sleeping**, nhưng những sessions này sẽ không xuất hiện trong bảng Active Expensive Queries. Chỉ khi người dùng gửi request truy vấn data và SQL Server đang thực thi request đó thì chúng mới xuất hiện trong bảng này.

Bây giờ bạn có thể hình dung sự hữu ích của bảng theo dõi này rồi chứ? Bảng Processes liệt kê tất cả những sessions kết nối tới SQL Server và chúng có thể gây nhiễu thông tin theo dõi cho quản trị viên vì không chắc chúng thật sự có request từ user. [Bảng Recent Expensive Queries](/2021/04/05/SQL-Server-Activity-Monitor-p5) giúp bạn xác định những câu truy vấn sử dụng nhiều tài nguyên đã diễn ra, chúng thuộc về quá khứ.

## **Bảng Active Expensive Queries**

Đối với bảng Active Expensive Queries, những câu truy vấn trong này là nguyên nhân gây ra hiện trạng hiện tại của SQL Server. Nếu CPU đang lên cao, chính là những câu truy vấn trong danh sách này gây nên. Nếu SQL Server của bạn ì ạch, cũng từ danh sách này. Bạn nghe user báo có report chạy chậm, bạn sẽ thấy nó ở trong này. Vậy khi xử lý các sự cố của SQL Server, bạn nhìn vào danh sách những câu truy vấn đang diễn ra này và xác định xem ai là thủ phạm.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p6/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Nhiều processes/sessions nhưng không thật sự có request.</div>
</div>
<hr>
Mình đã mở nhiều New Query trên SSMS và chúng được thể hiện trên bảng Processes như hình trên. Nhưng vì không có request nào đang được thực thi nên bảng Active Expensive Queries trống trơn. Dưới đây là demo tình huống một câu truy vấn dài dằng dặc xuất hiện trong bảng trông như thế nào.

```sql
USE  StackOverflow2010
GO
 
SELECT COUNT(1)
FROM Posts p1 
    CROSS JOIN Posts p2
    CROSS JOIN Posts p3
GO
```

Sử dụng database [StackOverflow2010](https://quantricsdulieu.com/2020/08/10/hanh-trinh-dem-sao-2-cai-dat-moi-truong-sql-server/) và câu truy vấn là cross join bảng Posts với chính nó, chạy hơn 10 phút vẫn chưa xong.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p6/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: Câu truy vấn đang diễn ra – long running query.</div>
</div>
<hr>
Bảng Active Expensive Queries trên mấy mình cho kết quả như hình trên, một câu truy vấn là của SQL Profiler, câu còn lại là CROSS JOIN bảng Post của chúng ta. Click chuột phải vào câu truy vấn bất kì chúng ta có thể xem chi tiết nội dung câu lệnh T-SQL hoặc xem [execution plan](https://quantricsdulieu.com/2021/01/27/hanh-trinh-dem-sao-6-execution-plan-la-gi/) của nó. Ngoài ra bạn còn có thể xem [Live Execution plan](https://docs.microsoft.com/en-us/sql/relational-databases/performance/live-query-statistics?view=sql-server-ver15) để thấy data luân chuyển trong execution plan như thế nào. Nếu mục này trên SSMS của bạn bị disable thì hãy dùng [enable trace flag 7412](https://www.scarydba.com/2018/06/11/plan-metrics-without-the-plan-trace-flag-7412/) để nó hiện lên.

```sql
DBCC TRACEON (7412, -1);
```
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p6/Hinh3.png" width = "800">
</div>
<div class="thecap">Hình 3: Live query statistics.</div>
</div>
<hr>
Nếu các bạn xem trực tiếp trên SSMS sẽ thấy Live Execution Plan này cung cấp các thông tin cực kì hữu ích như số lượng dòng đang xử lý, chiều di chuyển data, thời gian đã thực hiện,..cho việc xử lý các sự cố hiệu năng truy vấn. Các cột trong bảng Active Expensive Queries phần lớn giống các bảng chúng ta đã tìm hiểu, có 4 cột khác biệt rõ rệt là:
- **Row Count** – Số lượng dòng đã trả về client trong request này.
- **Required Memory** – Lượng KB memory tối thiểu cần có để thực thi request.
- **Allocated Memory** – Lượng KB memory SQL Server đã cấp phát cho request.
- **Used Memory** – Lượng KB memory request đã dùng tại thời điểm quan sát.

## **Các câu truy vấn đang diễn ra lưu ở đâu**

Sử dụng SQL Profiler như [bài trước](/2021/03/13/SQL-Server-Activity-Monitor-p3) chúng ta thấy Activity Monitor truy vấn data từ hai DMVs chính để lấy thông tin về các câu truy vấn đang diễn ra. [sys.dm_exec_requests](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver15) chứa các request đang được thực thi bởi SQL Server, [sys.dm_exec_query_memory_grants](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-memory-grants-transact-sql?view=sql-server-ver15) cung cấp thông tin về memory đã cấp phát cho truy vấn đó. Câu lệnh T-SQL đầy đủ mà Activity Monitor dùng như sau.

```sql
WITH profiled_sessions as (
    SELECT DISTINCT session_id profiled_session_id from sys.dm_exec_query_profiles
)
SELECT TOP 10 SUBSTRING(qt.TEXT, (er.statement_start_offset/2)+1,
((CASE er.statement_end_offset
WHEN -1 THEN DATALENGTH(qt.TEXT)
ELSE er.statement_end_offset
END - er.statement_start_offset)/2)+1) as [Query],
er.session_id as [Session Id],
er.cpu_time as [CPU (ms/sec)],
db.name as [Database Name],
er.total_elapsed_time as [Elapsed Time],
er.reads as [Reads],
er.writes as [Writes],
er.logical_reads as [Logical Reads],
er.row_count as [Row Count],
mg.granted_memory_kb as [Allocated Memory],
mg.used_memory_kb as [Used Memory],
mg.required_memory_kb as [Required Memory],
/* We must convert these to a hex string representation because they will be stored in a DataGridView, which can't handle binary cell values (assumes anything binary is an image) */
master.dbo.fn_varbintohexstr(er.plan_handle) AS [sample_plan_handle], 
er.statement_start_offset as [sample_statement_start_offset],
er.statement_end_offset as [sample_statement_end_offset],
profiled_session_id as [Profiled Session Id]
FROM
sys.dm_exec_requests er
LEFT OUTER JOIN sys.dm_exec_query_memory_grants mg 
    ON er.session_id = mg.session_id
LEFT OUTER JOIN profiled_sessions
    ON profiled_session_id = er.session_id
CROSS APPLY sys.dm_exec_sql_text(er.sql_handle) qt,
sys.databases db
WHERE db.database_id = er.database_id
AND er.session_id  <> @@spid
```

Như vậy chúng ta đã tìm hiểu hết các tính năng theo dõi có trên Activity Monitor. Bằng việc quan sát hoạt động các data files trong database, các câu truy vấn, cũng như các thông số khác giúp quản trị viên hiểu được điều gì đang xảy ra với SQL Server. Activity Monitor cũng cung cấp chi tiết tình trạng thực thi của các câu truy vấn giúp ta đào sâu hơn nguyên nhân gây ra các tình huống nghẽn tài nguyên, hiệu năng kém… Sử dụng Activity Monitor là bước tiếp cận đầu tiên để có thể giải quyết các sự cố trong SQL Server.

Hi vọng loạt bài viết này giúp các bạn hiểu rõ hơn hệ thống mình đang quản trị.

Nguồn: [quantricsdulieu](https://quantricsdulieu.com/2021/03/26/tim-hieu-sql-server-activity-monitor-p6-cau-truy-van-dang-dien-ra/)

---
