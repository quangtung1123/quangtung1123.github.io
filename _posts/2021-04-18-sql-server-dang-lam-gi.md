---
layout: post
comments: true
title:  "SQL SERVER - SQL Server đang làm gì?"
title2:  "SQL Server đang làm gì?"
date:   2021-04-18 15:10:00
permalink: 2021/04/18/sql-server-dang-lam-gi
mathjax: false
tags: SQL_Server Database Restore
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/sql-server-dang-lam-gi/Hinh1.png
summary: SQL Server đang làm gì?

---

Công việc quản trị SQL Server mang đến cho tôi rất nhiều trải nghiệm về thời gian, những khi hệ thống chạy trơn tru chúng ta tha hồ rê chuột dạo quanh blog của các chuyên gia về lĩnh vực này. Có đôi khi một vài câu truy vấn block lẫn nhau liền kéo chúng ta trở về với thực tại, hoặc việc tranh chấp tài nguyên làm lượng requests đến server giảm đột ngột, hoặc SQL Server đột nhiên trở nên ì ạch mặc dù các thông số tài nguyên đều bình thường, hoặc processor time của server lên 100%… Tất cả những thứ này đặt chúng ta vào tình huống khẩn cấp và việc tìm ra nguyên nhân để xử lý tình huống giống như bạn phải bê những tảng đá lớn ra khỏi đường ray khi tàu hỏa phía sau đang chạy đến, lúc này không còn khái niệm thời gian. Trong những lúc thế này việc đầu tiên bạn cần xác định là điều gì đang diễn ra, SQL Server đang làm gì, có admin nào đang thay đổi thiết lập gì trên production không? Hầu hết các tình huống phản ứng đầu tiên bạn nên làm là tìm xem SQL Server đang chạy những câu lệnh nào.

Giống như lần trước khi chúng ta muốn xem lịch sử các lần chạy của một câu truy vấn sẽ truy cập dynamic management view (DMV) sys.dm_exec_query_stats. Đối với những câu lệnh đang thực thi của người dùng từ SSMS hoặc từ các ứng dụng và thậm chí các system tasks của SQL Server cũng được lưu trong DMV sys.dm_exec_requests. Vì chỉ chứa những requests đang được thực thi nên những tình huống đang xảy ra đối với SQL Server phần lớn đều liên quan đến các câu lệnh trong này, do đó bạn có thể khảo sát từng đứa một để tìm xem ai làm mất khái niệm thời gian của bạn. Câu lệnh sau chỉ lấy ra những câu truy vấn của người dùng (các system process của SQL Server có session_id từ 1 đến 50).

```sql
SELECT r.session_id, r.status,r.command,r.cpu_time,r.total_elapsed_time,
r.wait_type, r.wait_resource, r.wait_time 
FROM sys.dm_exec_requests r
WHERE session_id > 50
```

Kết quả của câu lệnh trên chỉ có mỗi session mình đang mở từ SSMS (bản thân câu lệnh đó).
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Mỗi request đến SQL Server sẽ có một session_id, status là trạng thái của câu lệnh đó có thể là running hoặc runable hoặc sleeping. CPU time tính bằng đơn vị milliseconds, wait_time là thời gian đã chờ resource nào đó, wait_type và wait_resource mô tả thông tin về resource mà session này đang chờ. Các bạn có thể tìm hiểu ý nghĩa các cột của DMV này [ở đây](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver15). Để thấy rõ hơn session_id 53 này có nội dung là gì chúng ta sử dụng kết hợp với DMF sys.dm_exec_sql_text như sau.

```sql
SELECT r.session_id,t.text, r.status,r.command, r.cpu_time,r.total_elapsed_time,
r.wait_type, r.wait_resource, r.wait_time 
FROM sys.dm_exec_requests r
    CROSS APPLY sys.dm_exec_sql_text(sql_handle) AS t
WHERE session_id > 50
```
    
kết quả trả về sẽ có thêm cột text là nội dung câu lệnh T-SQL của chúng ta.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Để các bạn thấy rõ hơn tác dụng của câu lệnh này chúng ta hãy giả lập một lượng workloads từ một ứng dụng cụ thể. SQLQueryStress là một công cụ miễn phí có thể giúp chúng ta trong việc này. Công cụ này giúp cho việc chạy một câu truy vấn hàng ngàn lần từ nhiều sessions khác nhau trở nên dễ dàng hơn, giống như là một ứng dụng thật sự. Bạn có thể download bản mới nhất ở [trang gốc](https://github.com/ErikEJ/SqlQueryStress) cùng với [hướng dẫn](https://github.com/ErikEJ/SqlQueryStress/wiki) cụ thể từ họ. Cách sử dụng khá là đơn giản, sau khi load về và extract bạn sẽ thấy file SQLQueryStress.exe, click chạy sẽ ra giao diện như hình bên dưới.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh3.png" width = "800">
</div>
<div class="thecap">Giao diện chính của SQLQueryStress</div>
</div>
<hr>

Chúng ta sẽ dần tìm hiểu các giá trị biểu hiện trên màn hình này, các thiết lập cơ bản nhất là chọn server và database để kết nối, nhập script T-SQL bạn muốn thực thi, nhập số lượng threads cần giả lập và số lần lặp, cuối cùng click GO để ứng dụng thực hiện. Mình kết nối tới SQL Server trên máy local và database như hình bên dưới.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh4.png" width = "800">
</div>
<div class="thecap">Chọn kết nối cơ sở dữ liệu</div>
</div>
<hr>

Các bạn chú ý đến phương thức authentication, ở hình trên mình sử dụng windows authentication vì account đang login windows này có thể truy cập SQL Server, hoặc các bạn có thể sử dụng SQL Server authentication như thường. Sau khi kết nối cơ sở dữ liệu xong chúng ta sẽ giả lập một ứng dụng có 4 threads truy vấn đồng thời tới server và mỗi thread lặp 10 lần. Chúng ta thử chạy câu truy vấn có điệu kiện tìm kiếm ở mệnh đề having và check xem nó được thực hiện như thế nào.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh5.png" width = "800">
</div>
<div class="thecap">Giả lập 4 threads, mỗi thread lặp 10 lần.</div>
</div>
<hr>

Sau khi thiết lập mọi thứ như hình trên chúng ta click vào button GO. Trong lúc này chúng ta hãy chạy lại câu lệnh kiểm tra ở trên để thấy có những session_id nào đang thực thi và nội dung của chúng là gì.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh6.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Ngoài session_id 53 ra còn có 4 sessions khác với nội dung câu truy vấn chính là câu lệnh SELECT có điều kiện tìm kiếm ở mệnh đề HAVING của chúng ta. Đây chính là một công cụ hữu ích để giúp công việc quản trị của chúng ta trở nên dễ dàng hơn, bằng việc xác định được những hoạt động nào đang diễn tra trên SQL Server và trạng thái của chúng là gì là bước đầu tiên để chúng ta truy tìm nguồn gốc của các sự cố phát sinh như CPU cao, server bị chậm, ổ đĩa cứng bị đầy,… Với 4 core 8 CPUs trên máy của mình phải mất gần 58 giây để chạy xong workload trên.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh7.png" width = "800">
</div>
<div class="thecap">Kết quả sau khi chạy xong.</div>
</div>
<hr>

Tổng số lần lặp là 40 (4 threads lặp 10 lần), giá trị trung bình CPU time, elapsed time và logical reads của mỗi lần lặp cũng được thống kê để các bạn tham khảo so sánh giữa các lần chạy.

Trong tình huống câu lệnh thực thi là một stored procedure thì thế nào? Chúng ta biết rằng một stored procedure có thể chứa nhiều câu lệnh SELECT và chúng được thực thi tuần tự theo thứ tự logic trong procedure đó, tức tại một thời điểm chỉ có một statement đang thực thi vậy cột text kia sẽ có nội dung gì? Hãy tạo một stored procedure với tham số truyền vào là một số nguyên để chọn câu truy vấn cần thực thi. Mình sẽ đặt hai câu truy vấn ở bài trước vào trong stored procedure này với ID lần lượt là 1 và 2 như đoạn code bên dưới.

```sql
CREATE PROCEDURE sp_app_query_data
(
    @queryID INT = 1
)
AS
BEGIN
    IF @queryID = 1
    BEGIN
        --- điều kiện tìm kiếm ở mệnh đề HAVING
        SELECT location, COUNT(*) AS cnt
        FROM Users u
            INNER JOIN Posts p ON u.Id = p.OwnerUserId
        WHERE PostTypeId = 2 
        GROUP BY location
        HAVING location LIKE '%vietnam%'
            AND COUNT(*) > 10
        ORDER BY cnt
        OPTION (MAXDOP 1)
    END
    ELSE IF @queryID = 2
    BEGIN
        --- điều kiện tìm kiếm ở mệnh đề WHERE
        SELECT location, COUNT(*) AS cnt
        FROM Users u
            INNER JOIN Posts p ON u.Id = p.OwnerUserId
        WHERE PostTypeId = 2 AND location LIKE '%vietnam%'
        GROUP BY location
        HAVING COUNT(*) > 10
        ORDER BY cnt
        OPTION (MAXDOP 1)
    END
END
```

Và giờ chúng ta hãy chạy lại workload như trên nhưng lần này với câu lệnh execute stored procedure chúng ta mới tạo với tham số là 1 (chú ý: kết quả trong hình của lần chạy trước).
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh8.png" width = "800">
</div>
<div class="thecap">Thay thế adhoc query bằng stored procedure.</div>
</div>
<hr>

Click GO và chạy lại câu lệnh kiểm tra dưới đây để xem kết quả thế nào.

```sql
SELECT r.session_id,t.text,
[current_stmt] = SUBSTRING (t.text, r.statement_start_offset/2,
                    (CASE WHEN r.statement_end_offset = -1 THEN DATALENGTH(t.text)  
                        ELSE r.statement_end_offset END - r.statement_start_offset)/2),
r.status,r.command, r.cpu_time,r.total_elapsed_time,
r.wait_type, r.wait_resource, r.wait_time 
FROM sys.dm_exec_requests r
    CROSS APPLY sys.dm_exec_sql_text(sql_handle) AS t
WHERE session_id > 50
```
    
Mỗi khi SQL Server chạy tới statement nào trong stored procedure sẽ cập nhật lại các cột statement_start_offset và statement_end_offset trong DMV sys.dm_exec_requests để phản ánh đúng câu lệnh SELECT mà nó đang thực thi. Chính vì vậy mà ta có thể dựa vào thông tin này để biết câu lệnh hiện tại là gì. Kết quả như hình dưới, đó là câu lệnh SELECT có điều kiện tìm kiếm ở mệnh đề HAVING.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/sql-server-dang-lam-gi/Hinh9.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Sẽ là một thiếu xót nếu muốn kiểm tra xem SQL Server đang làm gì mà không nhắc đến stored procedure sp_whoisactive. SP này được viết bởi [Adam Machanic](http://dataeducation.com/about/), một SQL Server MVP. sp_whoisactive cung cấp rất nhiều thông tin cần thiết để bạn troubleshoot mỗi khi gặp sự cố về performance. Các bạn có thể [download](http://whoisactive.com/downloads/) và tìm hiểu thêm ở [link này](http://whoisactive.com/docs/). Nếu các bạn gặp vấn đề gì ở việc download và cài đặt SP trên thì hãy comment để mình hướng dẫn thêm. Các bạn chú ý, nó cần quyền system admin để chạy sp_whoisactive nên xem xét việc test thử ở môi trường dev.

Vậy là chúng ta đã cùng tìm hiểu cơ bản cách xác định những câu truy vấn nào đang hoạt động trên SQL Server. Bằng cách sử dụng hai DMVs sys.dm_exec_requests và sys.dm_exec_sql_text chúng ta có thể biết được chính xác statement nào đang được thực thi và trạng thái của chúng là gì, lượng CPU đã dùng và còn nhiều thông tin khác nữa mà mình chưa đề cập trong bài viết này. Tất cả những thông tin này bạn đều có thể lấy được thông qua việc tìm hiểu các cột trong hai DMVs trên. Chúc các bạn quản trị thật tốt SQL Server của mình.

**Tài liệu tham khảo**

- [quantricsdulieu](https://quantricsdulieu.com/hanh-trinh-dem-sao-5-sql-server-dang-lam-gi/)

---
