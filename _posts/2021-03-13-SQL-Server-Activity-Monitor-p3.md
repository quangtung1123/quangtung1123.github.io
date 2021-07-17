---
layout: post
comments: true
title:  "SQL SERVER - SQL Server Activity Monitor (phần 3) – Wait statistics"
title2:  "SQL Server Activity Monitor (phần 3) – Wait statistics"
date:   2021-03-13 15:10:00
permalink: 2021/03/13/SQL-Server-Activity-Monitor-p3
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Activity-Monitor-p3/Hinh1.png
summary: Tìm hiểu SQL Server Activity Monitor

---

Nếu [Waiting Tasks](/2021/03/11/SQL-Server-Activity-Monitor-p1) thể hiện số lượng processes đang chờ tài nguyên và [bảng Processes](2021/03/12/SQL-Server-Activity-Monitor-p2) hữu ích trong việc chỉ ra các session block nhau (cũng là một dạng chờ tài nguyên) thì bảng Resource Waits lại cho ta biết tài nguyên nào đang bị nghẽn, hay nói cách khác các processes đang chờ tài nguyên gì nhiều nhất. [Wait statistics](https://sqlperformance.com/2019/05/sql-performance/introduction-to-wait-statistics) là thông tin SQL Server thống kê các request đang chờ tài nguyên gì và chờ trong bao lâu. Bảng Resource Waits giúp chúng ta xác định các tài nguyên như Memory, CPU, Network I/O hay Disk I/O đâu là điểm nghẽn ảnh hưởng hiệu năng của SQL Server.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p3/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Bảng Resource Waits trong Activity Monitor.</div>
</div>
<hr>

## Đôi điều về SQL Server Wait Statistics

SQL Server thực thi một request bằng scheduler và thuật toán điều phối (scheduling ) của riêng nó, hay nói cách khác là nó có OS riêng gọi là SQLOS. Khi ứng dụng yêu cầu một kết nối (**connection**), SQL Server sẽ xác thực người dùng và tạo cho connection này một **session** và gán cho một id tương ứng là **session_id**. Khi user gửi một **request** thông qua session này, SQL Server sẽ tạo ra một hoặc nhiều **tasks** tương ứng. Mỗi task sẽ lại được gán một **worker thread** để có thể sử dụng tài nguyên CPU thực hiện công việc trong task đó. Không phải các worker thread đều có thể sử dụng CPU thẳng cho đến khi thực hiện xong công việc của mình. Trong quá trình thực thi request, vì SQL Server sử dụng thuật toán điều phối **cooperative** nên mỗi worker thread sẽ có quota cho mỗi lần dùng CPU là **4ms** (mặc định) gọi là **quantum**. Hết lượng quota này worker thread sẽ nhảy vào hàng đợi, nhường CPU cho worker threads khác và chờ đến lượt mình lần nữa, cứ như vậy cho đến lúc thực hiện xong công việc.

Nhưng không phải request chỉ cần dùng mỗi CPU time, khi cần tài nguyên khác như truy cập data page mà chưa có trên buffer pool hoặc network I/O chưa sẵn sàng để transfer data về client hoặc memory không đủ để thực hiện công việc thì các request đều chuyển sang trạng thái chờ tài nguyên. Đến khi các tài nguyên này sẵn sàng, worker threads sẽ chuyển sang trạng thái chuẩn bị để có thể sử dụng CPU. Các trạng thái của một worker thread có thể mô tả như sau

- **RUNNING** đang ở trên CPU
- **SUSPENDED** bất cứ khi nào worker thread cần resource mà không thể có ngay, nó sẽ rời CPU và chuyển vào hàng đợi (không thứ tự) resource với status là suspended. Đến khi resource này sẵn sàng cho thread thì nó sẽ nhảy sang hàng đợi (có thứ tự) runable
- **RUNABLE** là status của các threads không phải đợi resource mà cũng chưa thể dùng CPU. Đây là trạng thái chờ đợi thể hiện áp lực lên CPU thường được gọi là signal wait.

Trong quá trình thực hiện các request SQL Server sẽ ghi nhận từng loại tài nguyên mà các threads đã đợi cũng như tổng thời gian chờ. Thời gian thực thi của một request: **duration = CPU time + wait time**.

## Bảng Resource Waits

Có hàng tá các loại [wait](https://www.sqlskills.com/help/waits/) có thể có cho một resource nào đó, ví như lock thì ta có LCK_M_X, LCK_M_IX, LCK_M_S hoặc buffer I/O thì ta có PAGEIO_LATCH_EX, PAGEIO_LATCH_SH, PAGEIO_LATCH_UP,… Activity Monitor gom nhóm các wait type mô tả cùng một loại tài nguyên để quản trị viên dễ dàng phân biệt thành phần nào của SQL Server đang gặp sự cố về hiệu năng và dự đoán được chức năng nào của ứng dụng hay câu truy vấn nào của người dùng có khả năng gây ra. Từ đó giới hạn vùng tìm kiếm để nhanh chóng tìm hiểu rõ vấn đề. Ý nghĩa các cột trong bảng Resource Wait như sau:

- **Wait Category** – Nhóm các wait type trên cùng tài nguyên, chúng ta sẽ cùng tìm hiểu Activity Monitor gom nhóm như thế nào ở phía dưới.
- **Wait Time (ms/sec)** – Thời gian đợi trung bình (đơn vị millisecond) trên giây theo mỗi chu kỳ “Refresh Interval”. con số này được tính trên tất cả các wait type trong category.
- **Recent Wait Time (ms/sec)** – Trong trường hợp “Refresh Interval” nhỏ hoặc bạn nhấn F5 liên tục, giá trị wait time trong khoảng thời gian ngắn này thể hiện rõ resource nào đang nghẽn nên Activity Monitor sử dụng công thức tính trung bình có trọng số nhằm tăng mức độ tin cậy của giá trị này.
- **Average Waiter Count** – Số lượng xuất hiện các lần đợi tài nguyên trong khoảng “refresh interval”. Một task có thể trải qua nhiều trạng thái trong quá trình thực thi chạy->chờ->chờ cpu->chạy->chờ…mỗi một lần một task nhảy vào hàng đợi để chờ tài nguyên là giá trị này tăng lên 1.
- **Cumulative Wait Time (sec)** – Tổng thời gian chờ (đơn vị giây) của các wait type tính từ lúc SQL Server start.

## Activity Monitor lấy thông tin wait statistics từ đâu?

Để hiểu rõ hơn cách Activity Monitor thu thập các thông tin về wait statistics này chúng ta có thể sử dụng SQL Profiler để bắt xem nó gửi gì xuống SQL Server từ khi quản trị viên mở dashboard cho đến lúc mỗi refresh interval thực thi. Vì profiler ảnh hưởng đến hiệu năng chung của SQL Server nên chỉ thực hiện điều này trên môi trường test.

Nếu bạn đã cài SSMS 18.0 trở lên bạn có thể vào windows run command (phím Window + R) và gõ "SQL Server Profiler 18" rồi enter, cửa sổ SQL Profiler sẽ hiện lên. Tiếp đó từ menu bạn chọn **File** > **New trace**.. và **đăng nhập** vào SQL instance bạn đang dùng. Ở cửa sổ **Trace Properties** chọn tab **Events Selections** và bỏ chọn các events khác chỉ chừa lại **RPC:Completed** và **SQL:BatchCompleted** như hình bên dưới, sau đó click **Run**.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Activity-Monitor-p3/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: Sử dụng SQL Profiler để tìm hiểu Activity Monitor lấy wait statistics từ đâu.</div>
</div>
<hr>

Vì hiện tại SQL Server không có request nào nên mình có thể capture tất cả, nhưng nếu máy bạn có những requests khác thì hãy filter chúng ra để kết quả trace của chúng ta dễ xem hơn. Sau khi Profiler đã sẵn sàng bạn hãy mở [Activity Monitor](2021/03/11/SQL-Server-Activity-Monitor-p1) lên, ngay lập tức bạn sẽ thấy những objects, stored procedures mà Activity Monitor tạo ra dùng để theo dõi wait statistics và tính toán để đưa lên giao diện. Dưới đây là đoạn code tạo các bảng tạm và phân bố các wait type vào category để hỗ trợ theo dõi wait statistics theo nhóm tài nguyên mà SQL Profiler đã bắt được trên máy của mình.

```sql
SET NOCOUNT ON;
 
-- Table [#am_wait_types]
-- This table contains the relation between SQL wait types and wait categories 
IF (OBJECT_ID(N'tempdb.dbo.#am_wait_types', 'U') IS NULL) 
BEGIN
    CREATE TABLE #am_wait_types(
            [category_name] [nvarchar](40) NOT NULL,
            [wait_type] [nvarchar](45) PRIMARY KEY NOT NULL,
            [ignore] [bit] NOT NULL
    ) ON [PRIMARY];
 
    -- Insert wait types
    BEGIN TRAN am_wait_categories;
 
    INSERT INTO #am_wait_types VALUES (N'Backup', N'BACKUP', 0);
    INSERT INTO #am_wait_types VALUES (N'Backup', N'BACKUP_CLIENTLOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Backup', N'BACKUP_OPERATOR', 0);
    INSERT INTO #am_wait_types VALUES (N'Backup', N'BACKUPBUFFER', 0);
    INSERT INTO #am_wait_types VALUES (N'Backup', N'BACKUPIO', 0);
    INSERT INTO #am_wait_types VALUES (N'Backup', N'BACKUPTHREAD', 0);
    INSERT INTO #am_wait_types VALUES (N'Backup', N'DISKIO_SUSPEND', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'ASYNC_DISKPOOL_LOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'ASYNC_IO_COMPLETION', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'FCB_REPLICA_READ', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'FCB_REPLICA_WRITE', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'IO_COMPLETION', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'PAGEIOLATCH_DT', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'PAGEIOLATCH_EX', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'PAGEIOLATCH_KP', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'PAGEIOLATCH_NL', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'PAGEIOLATCH_SH', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'PAGEIOLATCH_UP', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer I/O', N'REPLICA_WRITES', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer Latch', N'PAGELATCH_DT', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer Latch', N'PAGELATCH_EX', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer Latch', N'PAGELATCH_KP', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer Latch', N'PAGELATCH_NL', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer Latch', N'PAGELATCH_SH', 0);
    INSERT INTO #am_wait_types VALUES (N'Buffer Latch', N'PAGELATCH_UP', 0);
    INSERT INTO #am_wait_types VALUES (N'Compilation', N'RESOURCE_SEMAPHORE_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Compilation', N'RESOURCE_SEMAPHORE_QUERY_COMPILE', 0);
    INSERT INTO #am_wait_types VALUES (N'Compilation', N'RESOURCE_SEMAPHORE_SMALL_QUERY', 0);
    INSERT INTO #am_wait_types VALUES (N'Full Text Search', N'MSSEARCH', 0);
    INSERT INTO #am_wait_types VALUES (N'Full Text Search', N'SOAP_READ', 0);
    INSERT INTO #am_wait_types VALUES (N'Full Text Search', N'SOAP_WRITE', 0);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SERVER_IDLE_CHECK', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'ONDEMAND_TASK_QUEUE', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SNI_HTTP_ACCEPT', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_BPOOL_FLUSH', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_DBSTARTUP', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_DCOMSTARTUP', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_MSDBSTARTUP', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_SYSTEMTASK', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_TASK', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SLEEP_TEMPDBSTARTUP', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'WAIT_FOR_RESULTS', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'WAITFOR_TASKSHUTDOWN', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'SQLTRACE_BUFFER_FLUSH', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'TRACEWRITE', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'XE_DISPATCHER_WAIT', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'XE_TIMER_EVENT', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'REQUEST_FOR_DEADLOCK_SEARCH', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'RESOURCE_QUEUE', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'LOGMGR_QUEUE', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'KSOURCE_WAKEUP', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'LAZYWRITER_SLEEP', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'BROKER_EVENTHANDLER', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'BROKER_TRANSMITTER', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'CHECKPOINT_QUEUE', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'CHKPT', 1);
    INSERT INTO #am_wait_types VALUES (N'Idle', N'BROKER_RECEIVE_WAITFOR', 1);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'DEADLOCK_ENUM_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'LATCH_DT', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'LATCH_EX', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'LATCH_KP', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'LATCH_NL', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'LATCH_SH', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'LATCH_UP', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'INDEX_USAGE_STATS_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Latch', N'VIEW_DEFINITION_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_BU', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_IS', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_IU', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_IX', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RIn_NL', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RIn_S', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RIn_U', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RIn_X', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RS_S', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RS_U', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RX_S', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RX_U', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_RX_X', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_S', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_SCH_M', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_SCH_S', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_SIU', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_SIX', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_U', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_UIX', 0);
    INSERT INTO #am_wait_types VALUES (N'Lock', N'LCK_M_X', 0);
    INSERT INTO #am_wait_types VALUES (N'Logging', N'LOGBUFFER', 0);
    INSERT INTO #am_wait_types VALUES (N'Logging', N'LOGMGR', 0);
    INSERT INTO #am_wait_types VALUES (N'Logging', N'LOGMGR_FLUSH', 0);
    INSERT INTO #am_wait_types VALUES (N'Logging', N'LOGMGR_RESERVE_APPEND', 0);
    INSERT INTO #am_wait_types VALUES (N'Logging', N'WRITELOG', 0);
    INSERT INTO #am_wait_types VALUES (N'Memory', N'UTIL_PAGE_ALLOC', 0);
    INSERT INTO #am_wait_types VALUES (N'Memory', N'SOS_RESERVEDMEMBLOCKLIST', 0);
    INSERT INTO #am_wait_types VALUES (N'Memory', N'SOS_VIRTUALMEMORY_LOW', 0);
    INSERT INTO #am_wait_types VALUES (N'Memory', N'LOWFAIL_MEMMGR_QUEUE', 0);
    INSERT INTO #am_wait_types VALUES (N'Memory', N'RESOURCE_SEMAPHORE', 0);
    INSERT INTO #am_wait_types VALUES (N'Memory', N'CMEMTHREAD', 0);
    INSERT INTO #am_wait_types VALUES (N'Network I/O', N'NET_WAITFOR_PACKET', 0);
    INSERT INTO #am_wait_types VALUES (N'Network I/O', N'OLEDB', 0);
    INSERT INTO #am_wait_types VALUES (N'Network I/O', N'MSQL_DQ', 0);
    INSERT INTO #am_wait_types VALUES (N'Network I/O', N'DTC_STATE', 0);
    INSERT INTO #am_wait_types VALUES (N'Network I/O', N'DBMIRROR_SEND', 0);
    INSERT INTO #am_wait_types VALUES (N'Network I/O', N'ASYNC_NETWORK_IO', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'ABR', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_REGISTERALLENDPOINTS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_SHUTDOWN', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_TASK_STOP', 1);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BAD_PAGE_PROCESS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_CONNECTION_RECEIVE_TASK', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_ENDPOINT_STATE_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BUILTIN_HASHKEY_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'CHECK_PRINT_RECORD', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_INIT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'BROKER_MASTERSTART', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'CURSOR', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'CURSOR_ASYNC', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBMIRROR_WORKER_QUEUE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBMIRRORING_CMD', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBTABLE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DAC_INIT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBCC_COLUMN_TRANSLATION_CACHE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBMIRROR_DBM_EVENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBMIRROR_DBM_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DBMIRROR_EVENTS_QUEUE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DEADLOCK_TASK_SEARCH', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DEBUG', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DISABLE_VERSIONING', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DLL_LOADING_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DROPTEMP', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DUMP_LOG_COORDINATOR', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DUMP_LOG_COORDINATOR_QUEUE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'DUMPTRIGGER', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'EC', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'EE_PMOLOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'EE_SPECPROC_MAP_INIT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'ENABLE_VERSIONING', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'ERROR_REPORTING_MANAGER', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'FSAGENT', 1);
    INSERT INTO #am_wait_types VALUES (N'Other', N'FT_RESTART_CRAWL', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'FT_RESUME_CRAWL', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'FULLTEXT GATHERER', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'GUARDIAN', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'HTTP_ENDPOINT_COLLCREATE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'HTTP_ENUMERATION', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'HTTP_START', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'IMP_IMPORT_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'IMPPROV_IOWAIT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'EXECUTION_PIPE_EVENT_INTERNAL', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'FAILPOINT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'INTERNAL_TESTING', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'IO_AUDIT_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'KTM_ENLISTMENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'KTM_RECOVERY_MANAGER', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'KTM_RECOVERY_RESOLUTION', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'MSQL_SYNC_PIPE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'MIRROR_SEND_MESSAGE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'MISCELLANEOUS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'MSQL_XP', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'REQUEST_DISPENSER_PAUSE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'PARALLEL_BACKUP_QUEUE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'PRINT_ROLLBACK_PROGRESS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QNMANAGER_ACQUIRE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QPJOB_KILL', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QPJOB_WAITFOR_ABORT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QRY_MEM_GRANT_INFO_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_ERRHDL_SERVICE_DONE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_EXECUTION_INDEX_SORT_EVENT_OPEN', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_NOTIFICATION_MGR_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_NOTIFICATION_SUBSCRIPTION_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_NOTIFICATION_TABLE_MGR_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_NOTIFICATION_UNITTEST_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_OPTIMIZER_PRINT_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_REMOTE_BRICKS_DONE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'QUERY_TRACEOUT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'RECOVER_CHANGEDB', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'REPL_CACHE_ACCESS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'REPL_SCHEMA_ACCESS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_EVENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_INTERNAL', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_RWLOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_SEMAPHORE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_SLEEP', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_TRACELOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOSHOST_WAITFORDONE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SHUTDOWN', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_CALLBACK_REMOVAL', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_DISPATCHER_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_LOCALALLOCATORLIST', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_OBJECT_STORE_DESTROY_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_PROCESS_AFFINITY_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SNI_CRITICAL_SECTION', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SNI_HTTP_WAITFOR_0_DISCON', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SNI_LISTENER_ACCESS', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SNI_TASK_COMPLETION', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SEC_DROP_TEMP_KEY', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SEQUENTIAL_GUID', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'VIA_ACCEPT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_STACKSTORE_INIT_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SOS_SYNC_TASK_ENQUEUE_EVENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SQLSORT_NORMMUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SQLSORT_SORTMUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'WAITSTAT_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'WCC', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'WORKTBL_DROP', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SQLTRACE_LOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SQLTRACE_SHUTDOWN', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SQLTRACE_WAIT_ENTRIES', 1);
    INSERT INTO #am_wait_types VALUES (N'Other', N'SRVPROC_SHUTDOWN', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'TEMPOBJ', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'THREADPOOL', 1);
    INSERT INTO #am_wait_types VALUES (N'Other', N'TIMEPRIV_TIMEPERIOD', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_TIMER_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_TIMER_TASK_DONE', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_BUFFERMGR_ALLPROCECESSED_EVENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_BUFFERMGR_FREEBUF_EVENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_DISPATCHER_JOIN', 1);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_MODULEMGR_SYNC', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_OLS_LOCK', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_SERVICES_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_SESSION_CREATE_SYNC', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_SESSION_SYNC', 0);
    INSERT INTO #am_wait_types VALUES (N'Other', N'XE_STM_CREATE', 0);
    INSERT INTO #am_wait_types VALUES (N'Parallelism', N'EXCHANGE', 1);
    INSERT INTO #am_wait_types VALUES (N'Parallelism', N'EXECSYNC', 1);
    INSERT INTO #am_wait_types VALUES (N'Parallelism', N'CXPACKET', 1);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_AUTO_EVENT', 1);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_CRST', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_JOIN', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_MANUAL_EVENT', 1);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_MEMORY_SPY', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_MONITOR', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_RWLOCK_READER', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_RWLOCK_WRITER', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_SEMAPHORE', 1);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLR_TASK_START', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'CLRHOST_STATE_ACCESS', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'ASSEMBLY_LOAD', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'FS_GARBAGE_COLLECTOR_SHUTDOWN', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'SQLCLR_APPDOMAIN', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'SQLCLR_ASSEMBLY', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'SQLCLR_DEADLOCK_DETECTION', 0);
    INSERT INTO #am_wait_types VALUES (N'SQLCLR', N'SQLCLR_QUANTUM_PUNISHMENT', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRAN_MARKLATCH_DT', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRAN_MARKLATCH_EX', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRAN_MARKLATCH_KP', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRAN_MARKLATCH_NL', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRAN_MARKLATCH_SH', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRAN_MARKLATCH_UP', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'TRANSACTION_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'XACT_OWN_TRANSACTION', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'XACT_RECLAIM_SESSION', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'XACTLOCKINFO', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'XACTWORKSPACE_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'DTC_TMDOWN_REQUEST', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'DTC_WAITFOR_OUTCOME', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'MSQL_XACT_MGR_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'MSQL_XACT_MUTEX', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'DTC', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'DTC_ABORT_REQUEST', 0);
    INSERT INTO #am_wait_types VALUES (N'Transaction', N'DTC_RESOLVE', 0);
    INSERT INTO #am_wait_types VALUES (N'User Waits', N'WAITFOR', 1);
    COMMIT TRAN;
END;
 
-- Table [#am_wait_stats_snapshots]
-- This table contains two snapshots of wait stats (the current snapshot, and the immediately 
-- prior snapshot), to use to calculate wait time for the just-completed time interval. 
IF OBJECT_ID ('tempdb.dbo.#am_wait_stats_snapshots') IS NULL
BEGIN
    CREATE TABLE #am_wait_stats_snapshots (
        snapshot_id bigint, 
        collection_time datetime, 
        wait_type nvarchar(120), 
        waiting_tasks_count bigint, 
        signal_wait_time_ms bigint, 
        wait_time_ms bigint, 
        raw_wait_time_ms bigint
    );
    CREATE CLUSTERED INDEX cidx ON #am_wait_stats_snapshots (snapshot_id, wait_type);
END;
 
-- Table [#am_resource_mon_snap]
-- This table contains the stats for the two most recently completed time intervals.  
IF OBJECT_ID ('tempdb.dbo.#am_resource_mon_snap') IS NULL
BEGIN
    CREATE TABLE #am_resource_mon_snap (
        previous_snapshot_id bigint, 
        current_snapshot_id bigint, 
        previous_collection_time datetime, 
        current_collection_time datetime, 
        interval_sec numeric (20,4), 
        category_name nvarchar(40), 
        wait_type nvarchar(45), 
        interval_waiting_tasks_count int, 
        interval_resource_wait_time bigint, 
        interval_resource_signal_time bigint,         
        interval_wait_time_per_sec bigint, 
        interval_avg_waiter_count numeric (10,1), 
        resource_wait_time_cumulative bigint, 
        weighted_average_wait_time_per_sec bigint
    );
    CREATE CLUSTERED INDEX cidx ON #am_resource_mon_snap (current_snapshot_id, wait_type);
END;
```

Bạn có thể chọn ra vài [wait type](https://www.sqlskills.com/help/waits/) và tìm hiểu ý nghĩa của chúng để làm quen với khái niệm waits statistics cũng như cách hoạt động. Bạn sẽ hiểu tại sao đây là phương pháp tiếp cận hiệu quả trong các tình huống xử lý sự cố hiệu năng hệ thống.

SQL Server lưu giữ thông tin về wait statistics trong hai DMVs [sys.dm_os_wait_stats](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql?view=sql-server-ver15) & [sys.dm_os_waiting_tasks](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql?view=sql-server-ver15) mà bạn có thể kiểm tra bằng việc đọc qua đoạn code tạo stored procedure #am_generate_waitstats trong kết quả Profiler trên. Mình sẽ có bài viết về hai DMVs này ở những lượt bài tới, rất mong các bạn đón xem.

Nguồn: https://quantricsdulieu.com/2021/03/11/tim-hieu-sql-server-activity-monitor-phan-3-wait-statistics/

---
