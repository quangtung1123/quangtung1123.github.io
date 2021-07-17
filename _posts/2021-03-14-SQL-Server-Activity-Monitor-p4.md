---
layout: post
comments: true
title:  "SQL SERVER - SQL Server Activity Monitor (phần 4) – Disk I/O latency"
title2:  "SQL Server Activity Monitor (phần 4) – Disk I/O latency"
date:   2021-03-14 15:10:00
permalink: 2021/03/14/SQL-Server-Activity-Monitor-p4
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Activity-Monitor-p4/Hinh1.png
summary: Tìm hiểu SQL Server Activity Monitor

---

Disk I/O latency là một thông số quan trọng cần theo dõi để thấy hiệu năng của I/O subsystem trong một hệ thống. SQL Server cần đọc dữ liệu từ disk lên memory để phục vụ các nhu cầu truy vấn và ngược lại, cần ghi dữ liệu từ memory xuống disk cho những thay đổi mà user thực hiện. Bảng Data file I/O này giúp quản trị viên theo dõi dung lượng dữ liệu được đọc lên và ghi xuống, cùng với độ trễ của mỗi hành động (I/O request), sử dụng kết hợp với những lời khuyên từ các chuyên gia về SQL Server hoặc chính từ Microsoft sẽ giúp bạn có nhận định đúng về hiện trạng I/O subsystem của mình và có hành động phù hợp.

## Bảng Data file I/O

Nếu xét việc theo dõi hiệu năng hoạt động của các [data và log files](/2021/03/10/Cau-tao-database-SQL-Server) trong các databases, theo các bạn những thông số nào thể hiện được điều này? Data luôn nằm trên memory là tình huống tốt nhất vì khi câu truy vấn cần dùng là có ngay, nhưng nếu phải đọc từ file ta cần biết mất bao lâu để data đó load lên memory, tương tự như vậy cho việc ghi xuống. Bảng Data file I/O cung cấp cho quản trị viên hai thước đo cơ bản nhất để đánh giá tình hình hoạt động của các files này đó là lưu lượng data đọc/ghi trên file và độ trễ (latency) của các lần thực hiện. Hình bên dưới thể hiện những thông tin Activity Monitor cung cấp cho chúng ta theo các tiêu chí này.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p4/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: minh họa disk I/O latency trên data file .mdf của StackOverflow database.</div>
</div>
<hr>

Chỉ có mỗi data file .mdf của stackoverflow database là có hoạt động với lưu lượng hơn 1GB/sec và độ trễ 39 ms, những data file khác hầu như không có tương tác gì. Hình trên là kết quả khi sử dụng phần demo [bài viết trước](/2021/03/11/SQL-Server-Activity-Monitor-p1) (Database I/O), cũng với thiết lập 3GB buffer pool và các câu truy vấn đọc tất cả các bảng trong database stackoverflow. Chúng ta cùng xem qua ý nghĩa các cột trên bảng này.

- **Database** – Tên database.
- **File Name** – Tên file và đường dẫn đến thư mục chứa file trên host.
- **MB/sec Read** – giá trị trung bình lượng data đọc từ file này trong 1 giây (theo đơn vị tính MB).
- **MB/sec Written** – giá trị trung bình lượng data ghi xuống file này trong 1 giây (theo đơn vị MB).
- **Response Time (ms)** – độ trễ trung bình trên 1 giây ( theo đơn vị milliseconds) của cả hai hành động đọc lên và ghi xuống file trong khoảng “Refresh Interval”.

Cứ mỗi một khoảng thời gian “Refresh Interval”, giả dụ chúng ta chọn là 10 giây. Activity Monitor lại truy vấn các thông tin này từ SQL Server, thực hiện các thao tác tính toán để lấy ra tổng dung lượng đọc/ghi, số lần request I/O, và độ trễ phát sinh trong 10 giây vừa qua. Từ các giá trị này chúng ta xác định được các thông tin như trên bảng Data file I/O.

## SQL Server lấy Disk I/O latency từ đâu?

Chúng ta cũng sử dụng Profiler giống [bài viết trước](/2021/03/13/SQL-Server-Activity-Monitor-p3) để theo dõi cách Activity Monitor query data như thế nào? Thực hiện các bước tương tự cho Profiler chúng ta thấy lúc khởi tạo Activity Monitor tạo bảng tạm tên #am_dbfileio, sau đó định kì thực thi một đoạn code T-SQL để thu thập và tính toán ra kết quả mong muốn. Các bạn có thể xem đoạn code sau để thấy rõ điều này.

```sql
-- bước 1: tạo bảng tạm
IF OBJECT_ID('tempdb..#am_dbfileio', 'U') IS NULL
BEGIN
    CREATE TABLE #am_dbfileio (collection_time datetime PRIMARY KEY, total_io_bytes numeric (28, 1));
END;
 
IF OBJECT_ID ('tempdb..#am_dbfilestats', 'U') IS NULL
BEGIN
    CREATE TABLE #am_dbfilestats (
        [collection_time] datetime, 
        [Database] sysname, 
        [File] nvarchar(1024), 
        [Total MB Read] numeric (28,1), 
        [Total MB Written] numeric (28,1), 
        [Total I/O Count] bigint, 
        [Total I/O Wait Time (ms)] bigint, 
        [Size (MB)] bigint   
    );
    CREATE CLUSTERED INDEX cidx ON #am_dbfilestats ([collection_time]);
END;
 
--- bước 2: truy vấn data định kì mỗi "Refresh Interval"
DECLARE @current_collection_time datetime;
SET @current_collection_time = GETDATE();
 
-- Grab a snapshot
INSERT INTO #am_dbfilestats
SELECT
    @current_collection_time AS collection_time, 
    d.name AS [Database], 
    f.physical_name AS [File], 
    (fs.num_of_bytes_read / 1024.0 / 1024.0) [Total MB Read], 
    (fs.num_of_bytes_written / 1024.0 / 1024.0) AS [Total MB Written], 
    (fs.num_of_reads + fs.num_of_writes) AS [Total I/O Count], 
    fs.io_stall AS [Total I/O Wait Time (ms)], 
    fs.size_on_disk_bytes / 1024 / 1024 AS [Size (MB)]
FROM sys.dm_io_virtual_file_stats(default, default) AS fs
INNER JOIN sys.master_files f ON fs.database_id = f.database_id AND fs.file_id = f.file_id
INNER JOIN sys.databases d ON d.database_id = fs.database_id; 
 
-- Get the timestamp of the previous collection time
DECLARE @previous_collection_time datetime;
SELECT TOP 1 @previous_collection_time = collection_time 
FROM #am_dbfilestats 
WHERE collection_time < @current_collection_time
ORDER BY collection_time DESC;
 
DECLARE @interval_ms int;
SET @interval_ms = DATEDIFF (millisecond, @previous_collection_time, @current_collection_time); 
 
-- Return the diff of this snapshot and last
SELECT
    cur.[Database], 
    cur.[File] AS [File Name], 
    CONVERT (numeric(28,1), (cur.[Total MB Read] - prev.[Total MB Read]) * 1000 / @interval_ms) AS [MB/sec Read], 
    CONVERT (numeric(28,1), (cur.[Total MB Written] - prev.[Total MB Written]) * 1000 / @interval_ms) AS [MB/sec Written], 
    -- protect from div-by-zero
    CASE
        WHEN (cur.[Total I/O Count] - prev.[Total I/O Count]) = 0 THEN 0
        ELSE
            (cur.[Total I/O Wait Time (ms)] - prev.[Total I/O Wait Time (ms)]) 
                / (cur.[Total I/O Count] - prev.[Total I/O Count])
    END AS [Response Time (ms)]
FROM #am_dbfilestats AS cur
INNER JOIN #am_dbfilestats AS prev ON prev.[Database] = cur.[Database] AND prev.[File] = cur.[File]
WHERE cur.collection_time = @current_collection_time 
    AND prev.collection_time = @previous_collection_time;
 
-- Delete the older snapshot
DELETE FROM #am_dbfilestats
WHERE collection_time != @current_collection_time;
```

Activity Monitor sử dụng DMV [sys.master_files](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-master-files-transact-sql?view=sql-server-ver15) để lấy ra tất cả các data files cần theo dõi trên SQL Server và DMF [sys.dm_io_virtual_file_stats](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql?view=sql-server-ver15) cung cấp thông tin hoạt động của từng file. Mỗi khi bước 2 được thực thi Activity Monitor sẽ lấy giá trị mới lưu vào bảng tạm, tiếp đó so sánh giá trị mới này với giá trị đã lưu lần trước và tính ra kết quả chênh lệch, đây chính là thông số phát sinh trong khoảng thời gian “Refresh Interval”. Function sys.dm_io_virtual_file_stats này nhận hai tham số đầu vào là database_id và file_id, nếu bạn để mặc định nó sẽ trả ra tất cả các files của các database.

```sql
select f.name, f.physical_name,f.type_desc,f.state_desc, f.size/128 as inMB, 
    sample_ms, num_of_reads, num_of_bytes_read, Io_stall_read_ms, num_of_writes, num_of_bytes_written, 
io_stall_write_ms, io_stall, size_on_disk_bytes
FROM sys.dm_io_virtual_file_stats(default, default) AS fs
    INNER JOIN sys.master_files f ON fs.database_id = f.database_id 
        AND fs.file_id = f.file_id
```

<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p4/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: các column chính của sys.dm_io_virtual_file_stats</div>
</div>
<hr>

Có thể hiểu ý nghĩa các cột trong câu truy vấn trên như sau:

- **database_id** – mỗi database trong instance có một giá trị id và duy nhất.
- **file_id** – mỗi database sẽ có danh sách các files đánh id từ 1. file_id là duy nhất trong một database.
- **sample_ms** – thời gian tính bằng đơn vị millisecond kể từ khi SQL Server start.
- **num_of_reads** – số lần đọc data trên file này, tính theo giá trị sample_ms (từ lúc SQL Server start). Con số này tính theo physical read, chứ không phải [logical read](/2021/03/Do-hieu-suat-truy-van).
- **num_of_bytes_read** – dung lượng bytes đã đọc từ file này kể từ khi SQL Server start, lưu ý rằng kích thước bytes mỗi lần đọc có thể khác nhau.
- **Io_stall_read_ms** – tổng thời gian các processes đã chờ I/O request.
- **num_of_writes** – số lần ghi data trên file này kể tức lúc start.
- **num_of_bytes_written** – dung lượng bytes đã ghi lên file này.
- **io_stall_write_ms** – tổng thời gian chờ hành động ghi của các processes.
- **io_stall** – tổng thời gian chờ của cả request đọc + ghi
- **size_on_disk_bytes** – kích thước file trên đĩa, theo đơn vị byte.

Hiểu được cách tổ chức lưu trữ và hoạt động của DMF này giúp cho việc đọc và phân tích kết quả từ bảng Data file I/O hiệu quả hơn. Bạn nhận ra được những file nào đọc ghi thường xuyên và khối lượng là bao nhiêu, chúng có phù hợp với mong đợi của hệ thống không hay là do hiệu suất câu truy vấn không tốt gây nên. Giá trị disk I/O latency (tương tự response time hoặc io_stall) như thế nào là ổn? Dưới đây là giá trị tham khảo từ trang [SQLskills.com](https://www.sqlskills.com/blogs/paul/are-io-latencies-killing-your-performance/), từ những chuyên gia hàng đầu về SQL Server

- Excellent: < 1ms
- Very good: < 5ms
- Good: 5 – 10ms
- Poor: 10 – 20ms
- Bad: 20 – 100ms
- Shockingly bad: 100 – 500ms
- WOW!: > 500ms

Disk I/O latency trên hệ thống của bạn đang là bao nhiêu? Hi vọng bài viết này giúp bạn hiểu rõ hơn về hệ thống của mình.


Nguồn: [quantricsdulieu](https://quantricsdulieu.com/2021/03/16/tim-hieu-sql-server-activity-monitor-p4-disk-i-o-latency/)

---
