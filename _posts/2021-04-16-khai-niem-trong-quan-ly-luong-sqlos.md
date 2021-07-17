---
layout: post
comments: true
title:  "SQL SERVER - Một vài khái niệm trong quản lý luồng (thread management) của SQLOS"
title2:  "Một vài khái niệm trong quản lý luồng (thread management) của SQLOS"
date:   2021-04-16 15:10:00
permalink: 2021/04/16/khai-niem-trong-quan-ly-luong-sqlos
mathjax: false
tags: SQL_Server Database Restore
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/khai-niem-trong-quan-ly-luong-sqlos/Hinh1.png
summary: Một vài khái niệm trong quản lý luồng (thread management) của SQLOS

---

Bắt đầu từ version 2005 trở đi, kiến trúc của SQL Server có thêm một lớp chịu trách nhiệm cho việc lập lịch và quản lý luồng (thread management and scheduling) độc lập với hệ điều hành của máy chủ và thường được gọi là SQLOS. Hôm nay chúng ta sẽ làm quen với một vài khái niệm liên quan như Tasks, Workers, Threads, Scheduler, Sessions, Connections, Requests.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/khai-niem-trong-quan-ly-luong-sqlos/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Thể hiện schedulers nằm ở SQLOS layer.</div>
</div>
<hr>

## **1. SQLOS Scheduler – Bộ Lập Lịch**

Khi SQL Server khởi động nó sẽ tạo một **scheduler** tương ứng với mỗi logical processor mà nó thấy trên máy chủ. Tùy theo số lượng CPU được cấp phát cho SQL Server (Processor Affinity setting) mà những schedulers này có trạng thái enabled hoặc disabled. Các bạn có thể kiểm tra trạng thái ở DMV [sys.dm_os_schedulers](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-schedulers-transact-sql?view=sql-server-ver15). Scheduler là đối tượng sẽ lập lịch và cho phép các worker thread tiếp xúc với CPU để được xử lý.

## **2. Tasks – Tác vụ**

Một tác vụ ([sys.dm_os_tasks](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-tasks-transact-sql?view=sql-server-ver15)) đại diện cho một đơn vị công việc cần được thực hiện. Một request đến SQL Server có thể được chia thành một hoặc nhiều tasks và đây chỉ là một đơn vị logic mà bản thân task sau đó được gán cho worker thread mới có thể hoàn thành request của client.

## **3. Workers (Worker Threads)**

Worker threads chính là luồng của SQLOS tạo ra và lập lịch để thực hiện request từ clients. Worker thread sẽ làm việc trực tiếp với scheduler và hoàn thành các đơn vị công việc từ các tasks cung cấp. Số lượng worker threads trong một instance là [có hạn](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-the-max-worker-threads-server-configuration-option?view=sql-server-ver15) và nếu kiểm soát không bạn sẽ gặp tình trạng chờ [THREADPOOL](https://www.sqlskills.com/help/waits/threadpool/). Các worker threads đã được tạo ra trên SQL Server instance cụ thể ở DMV [sys.dm_os_workers](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-workers-transact-sql?view=sql-server-ver15).

## **4. Thread – Luồng**

Đây là những thread chạy bên dưới (thuộc) process của SQL Server được tạo và quản lý bởi hệ điều hành máy chủ và được lưu trữ trong SQL Server thông qua DMV [sys.dm_os_threads](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-threads-transact-sql?view=sql-server-ver15). Những threads này ánh xạ 1-1 với worker thread (SQLOS).

## **5. Request – Yêu cầu**

Mỗi khi chúng ta gửi một batch (ngăn cách bằng lệnh GO trong SSMS) lên SQL Server thì đó là một request. Kế đến request này sẽ được gán cho một task và task này sẽ gắn vào một worker thread để có thể làm việc với scheduler. Bạn có thể khảo sát những request này thông qua DMV [sys.dm_exec_requests](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver15).

## **6. Session – Phiên Làm Việc**

khi client kết nối với SQL Server, hai bên thiết lập một “phiên” để trao đổi thông tin. Nói một cách chính xác thì một phiên không giống như một kết nối vật lý bên dưới, nó là một thể hiện logic của một kết nối đến SQL Server. Mỗi một [phiên](/2021/03/12/SQL-Server-Activity-Monitor-p2) được gán một số định danh thường được gọi là session_id. Bạn có thể thấy tất cả những phiên hiện có trong DMV [sys.dm_exec_sessions](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql?view=sql-server-ver15) và Microsoft phân định những session_id nhỏ hơn 50 là của hệ thống được dùng để chạy các tác vụ nội bộ như LazyWriter, Checkpoint, Log Writer, Service Broker,…và những phiên này không có connection tương ứng.

## **7. Connections – Kết nối**

Đây là kết nối vật lý thực tế theo đúng nghĩa đen của nó. Mỗi một kết nối từ client đến SQL Server được thể hiện trong DMV [sys.dm_exec_connections](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-connections-transact-sql?view=sql-server-ver15). Ứng với mỗi connection này sẽ có một phiên (và tất nhiên session_id) được tạo ra tương ứng. Một connection_id trong sys.dm_exec_connections là một GUID và được sử dụng để xác định duy nhất kết nối vật lý đó. Chúng ta không sử dụng connection_id để xác định phiên nào đang thực hiện truy vấn mà thường là một session_id.

## **Nguồn tham khảo:**

- [Tasks, Workers, Threads, Scheduler, Sessions, Connections, Requests ; what does it all mean?](https://techcommunity.microsoft.com/t5/sql-server-support/tasks-workers-threads-scheduler-sessions-connections-requests/ba-p/333990)
- [Microsoft SQL Server 2012 Internals](https://www.microsoftpressstore.com/store/microsoft-sql-server-2012-internals-9780735658561)
- [SQL Server 2005 Practical Troubleshooting: The Database Engine](https://www.amazon.com/SQL-Server-2005-Practical-Troubleshooting/dp/0321447743)

Nguồn: [quantricsdulieu](https://quantricsdulieu.com/2021/04/09/mot-vai-khai-niem-trong-quan-ly-luong-thread-management-sqlos/)

---
