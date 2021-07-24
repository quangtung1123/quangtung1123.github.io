---
layout: post
comments: true
title:  "SQL SERVER - 7 cấu hình cài đặt SQL Server để có hiệu năng cao"
date:   2021-07-22 15:10:00
permalink: 2021/07/22/7-cau-hinh-cai-dat-sql-server-de-co-hieu-nang-cao
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/7-cau-hinh-cai-dat-sql-server-de-co-hieu-nang-cao/Hinh1.png
summary: 7 cấu hình cài đặt SQL Server để có hiệu năng cao

---

Bạn đang là quản trị viên với hàng tá SQL Server instances hoặc bạn vừa tiếp quản những instances này từ người khác, hoặc bạn cài mới một instance,…thì 7 cấu hình cài đặt SQL Server phổ biến dưới đây sẽ giúp bạn hiểu và quản trị instances tốt hơn. Ở bài này mình chỉ nêu khái niệm cơ bản hoặc mô tả ngắn gọn công dụng từng cấu hình, cùng với đó là đường dẫn tới document hoặc bài viết trên mạng để các bạn tham khảo thêm, mục tiêu của mình là sẽ có từng bài chuyên sâu cho mỗi cấu hình.

## 1. SQL Server Service và SQL Agent Account ##

Bạn có thể sử dụng bất kì loại account nào để đăng ký cho hai services trên, chúng có thể là domain user, local user, virtual account, built-in system account. Tùy theo nhu cầu và mục đích mà bạn chọn loại account phù hợp ví dụ như sử dụng domain account để chạy service cho SQL Server failover cluster, và chỉ nên cấp phát permission vừa đủ. Cụ thể chúng ta chỉ cần “full control” trên các thư mục chứa data files, log files và trên thư mục chứa backup files. Bạn có thể tham khảo thêm ở [link này](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15).

## 2. Instant File Initialization ##

Khi bạn tạo mới database, hoặc add thêm data files log file vào database có sẵn, hoặc khi bạn tăng kích thước data files của database nào đó SQL Server sẽ yêu cầu OS cấp phát vùng không gian mới này. Để đảm bảo nội dung của vùng sắp được cấp phát trên đĩa không bị lộ với đối tượng sắp được cấp phát OS sẽ reset các bit trên này về 0. Với kích thước càng lớn thì thời gian reset về 0 này càng dài, do vậy ảnh hưởng đến hiệu năng của SQL Server.

Với tính năng [Instant File Initialization](https://docs.microsoft.com/en-us/sql/relational-databases/databases/database-instant-file-initialization?view=sql-server-ver15) SQL Server sẽ yêu cầu OS sẽ bỏ qua bước reset về 0 cho vùng không gian cấp phát. Kích hoạt tính năng này bằng cách [cấp phát permission](https://www.sqlshack.com/sql-server-setup-instant-file-initialization-ifi/) Perform Volume Maintenance Tasks cho SQL Server service account trên windows và khi đó việc add thêm files hoặc tăng trưởng sẽ nhanh hơn nhiều. Tuy nhiên, tính năng này không thể áp dụng cho [transaction log file](https://www.sqlskills.com/blogs/paul/search-engine-qa-24-why-cant-the-transaction-log-use-instant-initialization/) hoặc cho database có mã hóa Transparent Data Encryption (TDE).

## 3. DAC connection ##

[Dedicated Admin Connection](/2021/04/17/SQL-Server-Dedicated-Admin-Connection) là phương thức kết nối dành riêng cho quản trị viên thuộc sysadmin role. Sử dụng phương thức này để tránh việc dùng chung tài nguyên (như CPU, worker threads ,…) với các kết nối thông thường khác do đó không bị tranh chấp khi server đang trong tình trạng quá tải. Điều này rất hữu ích trong các tình huống xử lý sự cố khi CPU của server lên 100% và tất cả các requests hầu như đang bị treo không thể làm ăn được gì nhưng kết nối bằng DAC vẫn có thể thực hiện các request thành công. Mặc định sau khi cài đặt SQL Server disable tính năng này do vậy bạn cần bật lên để có thể sử dụng khi cần.

## 4. MIN/MAX server memory ##

[Max Server Memory](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/server-memory-server-configuration-options?view=sql-server-ver15) là lượng memory tối đa tính theo đơn vị MB mà một instance được phép dùng. Mặc định sau khi cài đặt SQL Server được cấp phát 2.147.483.647 MB, có nghĩa là SQL Server không được phép đòi hỏi memory vượt qua con số đó. Bạn cần thiết lập lại con số này cho phù hợp với nhu cầu sử dụng thực tế. Nếu giữ nguyên giá trị này và lượng physical memory trên server nhỏ hơn (thường như vậy) có thể gây ra tình trạng tranh chấp memory với OS, do đó sẽ ảnh hưởng ngược lại SQL Server. Còn nếu thiết lập giá trị này quá thấp so với workload của hệ thống sẽ gây tình trạng quá tải memory nội bộ SQL Server, và do đó ảnh hưởng đến hiệu năng chung.

[Min Server Memory](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/server-memory-server-configuration-options?view=sql-server-ver15) là lượng memory tối thiểu tính theo đơn vị MB mà SQL Server sẽ cố gắng giữ lại trong bất kì tình huống nào. Khi SQL Server service khởi động, nó không đòi OS lượng memory tối thiểu này liền mà sẽ tăng dần theo nhu cầu sử dụng. Một khi lượng memory chiếm giữ vượt qua con số Min Server Memory này thì SQL Server sẽ không để tuột memory đã cấp phát xuống dưới giá trị này.

Bạn có thể tham khảo các giá trị Max Server Memory phù hợp với lượng physical memory mà server mình có ở [link này](https://www.brentozar.com/blitz/max-memory/). Hãy tính toán thật kĩ khi bạn chạy hai hoặc nhiều instances trên cùng machine.

## 5. Maximum Degree of Parallelism ##

MAXDOP – Maximum Degree of Parallelism – số lượng cores (CPUs) mà một câu truy vấn song song có thể dùng. Giá trị mặc định Microsoft thiết lập là 0 – không giới hạn, có nghĩa rằng câu truy vấn có thể dùng tất cả các CPUs mà server đó có. Do vậy, bạn cần giới hạn số lượng cores có thể dùng cho một câu truy vấn song song để tránh tình trạng tranh chấp CPUs và sử dụng hiệu quả hơn. Vậy nên thiết lập giá trị MAXDOP này bao nhiêu là phù hợp? Nó tùy vào cấu hình server của bạn, hãy tham khảo ở [link này](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option?view=sql-server-ver15) để chọn con số phù hợp.

## 6. Cost Threshold For Parallelism ##

Mỗi query plan sẽ được Query Optimizer tính toán và gán cho một chi phí (trong quá trình optimize), nếu chi phí này vượt qua giá trị **cost threshold for parallelism** SQL Server sẽ xem xét áp dụng truy vấn song song cho query plan đó. Giá trị này [mặc định](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-the-cost-threshold-for-parallelism-server-configuration-option?view=sql-server-ver15) là 5, một con số khá thấp. Câu truy vấn càng phức tạp hoặc xử lý data càng nhiều thì chi phí càng cao tức là dễ vượt qua con số 5 này. Do đó bạn cần thiết lập lại giá trị này cho phù hợp với workload (OLTP hay là OLAP) của hệ thống mình.

## 7. Model database ##

Đây có thể xem là một database mẫu, mỗi khi bạn tạo mới một database SQL Server sẽ copy nội dung từ model qua database mới, ví dụ như bảng, stored procedure, users và permission gán cho users đó. Hoặc kể cả các thuộc tính như recovery model, kích thước tăng data files,… Điều này có nghĩa là bạn có thể tạo những đối tượng cần thiết, add user vào database model, gán permission và chỉnh recovery model để các databases được tao ra luôn tuân theo những tiêu chuẩn bạn thiết lập.

**Tài liệu tham khảo**

- [quantricsdulieu](https://quantricsdulieu.com/7-cau-hinh-cai-dat-sql-server-de-co-hieu-nang-cao/)

---
