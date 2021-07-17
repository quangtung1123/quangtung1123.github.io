---
layout: post
comments: true
title:  "SQL SERVER - Cấu tạo database trong SQL Server"
title2:  "Cấu tạo database trong SQL Server"
date:   2021-03-10 15:10:00
permalink: 2021/03/10/Cau-tao-database-SQL-Server
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/Cau-tao-database-SQL-Server/Hinh0.png
summary: Tìm hiểu SQL Server Activity Monitor
---

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Cau-tao-database-SQL-Server/Hinh0.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## SQL database trong SQL Server

Dưới góc nhìn của SSMS thì database có hình dạng như thế này
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Cau-tao-database-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: danh sách databases view từ SSMS</div>
</div>
<hr>

Nếu bạn click chuột phải view properties và chọn page File thì đằng sau cái icon đó là tập hợp các files trên các thiết bị lưu trữ (local disk hoặc network storage) và những database files này trông như hình bên dưới
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Cau-tao-database-SQL-Server/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: cấu tạo của database qtcsdl</div>
</div>
<hr>

Mỗi SQL Server instance có thể chứa tối đa 32767 databases, mỗi database chứa tối đa 32767 files. Khi bạn tạo database trong SQL Server sẽ có tối thiểu hai files trên hệ điều hành, một data file và một log file. **Data file** chứa data và các đối tượng như bảng, indexes, stored procedure và views. **Log file** ghi lại những thao tác thay đổi database với mục đích hỗ trợ re-do và undo trong bước recovery.

Về cơ bản SQL Server database chỉ có bao nhiêu đó, **tất cả những gì thuộc về một database đều gói gọn trong các files này và bạn có thể mang database từ nơi này sang nơi khác chỉ đơn giản bằng cách copy tất cả những files này sang máy khác** (tất nhiên còn có những cách khác như backup/restore) và attach chúng vào SQL Server trên server đó.

## Câu lệnh T-SQL tạo database

Câu lệnh tạo database đơn giản nhất có cú pháp như sau

```sql
IF DATABASEPROPERTYEX (N'qtcsdl', N'Version') > 0
BEGIN
    ALTER DATABASE qtcsdl SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    DROP DATABASE qtcsdl
END
GO
 
CREATE DATABASE qtcsdl
GO
```

Database trên sẽ được tạo ra với hai files tối thiểu như đề cập ở trên cùng các thông số mặc định từ dường dẫn chứa data và log files, kích thước file ban đầu, autogrowth size,… Bạn có thể view properties để xét các giá trị này như hướng dẫn ở hình 2 hoặc sử dụng DMVs **sys.database_files** như script sau:

```sql
CREATE DATABASE qtcsdl
GO
 
USE qtcsdl
GO
  
SELECT *
FROM sys.database_files
GO
```

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Cau-tao-database-SQL-Server/Hinh3.png" width = "800">
</div>
<div class="thecap">Hình 3: tạo database với các giá trị mặc định</div>
</div>
<hr>

Câu lệnh tạo database với các thông số tùy chọn cho **qtcsdl** trong hình 2 như sau:

```sql
IF DATABASEPROPERTYEX (N'qtcsdl', N'Version') > 0
BEGIN
    ALTER DATABASE qtcsdl SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    DROP DATABASE qtcsdl
END
GO
  
CREATE DATABASE [qtcsdl]  
ON PRIMARY
    ( NAME = N'qtcsdl', FILENAME = N'C:\user_db\qtcsdl\qtcsdl.mdf' , SIZE = 8192KB , MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB ), 
FILEGROUP [DATA] 
    ( NAME = N'data01', FILENAME = N'C:\user_db\qtcsdl\data01.ndf' , SIZE = 8192KB , MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB ),
    ( NAME = N'data02', FILENAME = N'C:\user_db\qtcsdl\data02.ndf' , SIZE = 8192KB , MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB )
LOG ON
    ( NAME = N'qtcsdl_log', FILENAME = N'C:\user_db\qtcsdl\qtcsdl_log.ldf' , SIZE = 262144KB , MAXSIZE = 2048GB , FILEGROWTH = 262144KB )
GO 
```

Trong câu lệnh trên, bạn tạo mới một database với các chỉ định về đường dẫn chứa database files, tạo thêm filegroup, kích thước ban đầu của file, kích thước mỗi khi grow và giới hạn của mỗi file.

**Primary data file**: là data file có đuôi .mdf, mỗi database chỉ có duy nhất một primary data file chứa thông tin khởi tạo cho database và những system objects.

**Secondary data file**: cũng là data file, nhưng do người dùng chỉ định, không bắt buộc. Một database có thể có nhiều secondary data files và đặt trên những thư mục khác nhau hoặc ổ đĩa vật lý khác nhau.

**Transaction log file**: lưu thông tin dùng để recover database, mỗi database phải có tối thiểu một file log.

**Filegroup**: là đơn vị logic dùng để gom nhóm các data files cho mục đích cấp phát space và quản trị của SQL Server. Mỗi data files (secondary file) chỉ thuộc về duy nhất một filegroup, mỗi filegroup có thể chứa nhiều data files. Transaction log file không thuộc bất kì filegroup nào. Mặc định sẽ luôn có default filegroup tên PRIMARY khi bạn tạo một database, giá trị default này ảnh hưởng khi bạn tạo bảng mà không chỉ định trên filegroup nào thì SQL Server sẽ lưu vào PRIMARY.

**File size**: kích thước ban đầu khi tạo file, giá trị này khá là quan trọng đặc biệt đối với transaction log file.

**FILEGROWTH**: mỗi khi data file không còn space để insert data mới SQL Server sẽ request OS tăng kích thước file lên theo giá trị chỉ định ở thuộc tính này.

**Maxsize**: kích thước tối đa mà một file có thể tăng trưởng.

## Tạo thêm data files và filegroups

Data sẽ tăng trưởng theo thời gian và những data files này ngày càng chiếm không gian nhiều hơn trên đĩa cứng lưu trữ. Sẽ đến lúc bạn cần gắn thêm ổ đĩa (LUNs ) để có thể chứa database ngày càng lớn này. Hoặc khi bạn cần sắp xếp và phân bổ các data files trên những ổ đĩa khác nhau nhằm mục đích tăng tốc độ đọc file của SQL Server hay là cân bằng không gian lưu trữ thì việc tạo thêm files/filegroup và xóa những files/filegroups không cần dùng đến là những công việc thường thấy của DBA. Cú pháp câu lệnh T-SQL để thêm mới file và filegroup như sau:

```sql
USE master
GO
ALTER DATABASE [qtcsdl]
ADD FILE 
    ( NAME = N'data03', FILENAME = N'D:\user_data\data03.ndf' , SIZE = 8192KB , MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB )
TO FILEGROUP [DATA]
GO
 
GO
ALTER DATABASE [qtcsdl] ADD FILEGROUP Test1FG1
GO
ALTER DATABASE [qtcsdl] 
ADD FILE 
    ( NAME = N'testfile01', FILENAME = N'D:\user_data\testfile01.ndf' , SIZE = 8192KB , MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB ),
    ( NAME = N'testfile02', FILENAME = N'D:\user_data\testfile02.ndf' , SIZE = 8192KB , MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB )
TO FILEGROUP Test1FG1
GO
```

Đoạn code T-SQL trên tạo mới file data03 trên cùng filegroup DATA với hai files đang có là data01 và data02 nhưng trên ổ đĩa D$ . Bên cạnh đó, script cũng tạo mới filegroup test1FG1 và tạo hai data file testfile01 và testfile02 trên filegroup mới này. Chúng ta có thể kiểm tra danh sách các files, kích thước, vị trí của chúng bằng DMV sys.database_files như sau

```sql
USE qtcsdl
GO
SELECT DB_NAME() AS dbname,fg.name AS filegroup, f.name AS filename, file_id, physical_name,
(size * 8.0/1024) AS size_MB,
((size * 8.0/1024) - (FILEPROPERTY(f.name, 'SpaceUsed') * 8.0/1024)) AS FreeSpace_MB
FROM sys.database_files f
    LEFT JOIN sys.filegroups fg ON f.data_space_id = fg.data_space_id
```
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Cau-tao-database-SQL-Server/Hinh4.png" width = "800">
</div>
<div class="thecap">Hình 4: Tạo thêm data files và filegroups</div>
</div>
<hr>

Hình 4 chỉ ra filegroup mới và vị trí các data files của nó trên server, cùng với đó là kích thước của các file và dung lượng còn trống của mỗi data file. Bạn có thể tạo data file với kích thước 10GB và OS sẽ cấp phát ngay lúc đó, nghĩa là OS sẽ ghi nhận vị trí 10GB này trên disk đã được dùng cho SQL Server. Đối với bảng hoặc index trên file này, SQL Server sẽ cấp phát các data pages (với kích thước 8K mỗi page) theo nhu cầu thực tế. Vậy nên bạn sẽ thấy SQL Server báo freespace của file khá lớn khi chưa có objects nào trên đó. Tương tự như hình trên, các secondary data file có kích thước 8MB và freespace cũng gần 8MB.

## Tạo thêm log file

Việc có hai hay nhiều log files không giúp cho SQL Server ghi log nhanh hơn, bởi vì SQL Server không thể ghi các log records vào nhiều file cùng một lúc mà phải ghi log tuần tự từng file. Tuy nhiên không phải vì vậy mà bạn không cần câu lệnh tạo thêm file log vì đôi khi bạn cần thêm space cho log file khi ổ đĩa hiện tại đã đầy.

```sql
USE master 
GO
ALTER DATABASE qtcsdl ADD LOG FILE 
( NAME = N'qtcsdl_log02', FILENAME = N'D:\user_data\qtcsdl_log02.ldf' , SIZE = 262144KB , MAXSIZE = 2048GB , FILEGROWTH = 262144KB )
 
GO
```

Câu lệnh trên tạo thêm một log file qtcsdl_log02 trên ổ đĩa D$ với các kích thước tương tự như file log đầu tiên. Chạy lại câu lệnh kiểm tra danh sách các files bạn sẽ thấy có thêm log file mới như hình dưới.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Cau-tao-database-SQL-Server/Hinh5.png" width = "800">
</div>
<div class="thecap">Hình 5: Tạo log files mới</div>
</div>
<hr>

## Xóa data file, filegroup và log file trong SQL Server

Bạn cần phải đảm bảo rằng không có data trong các files này mới có thể thực hiện việc xóa file khỏi database. Cú pháp của câu lệnh xóa data file và log file là giống nhau, bạn có thể xem ở demo dưới đây.

```sql
USE master;
GO
ALTER DATABASE qtcsdl REMOVE FILE qtcsdl_log02 
GO
ALTER DATABASE qtcsdl REMOVE FILE testfile01 
ALTER DATABASE qtcsdl REMOVE FILE testfile02
GO
ALTER DATABASE qtcsdl REMOVE FILEGROUP Test1FG1
```

## Một số chú ý khi làm việc với File và Filegroup trong SQL Server

1. Hầu hết các database với một data file và một log file như mặc định [đều ổn](2021/03/09/Tim-hieu-cac-database-he-thong), tuy nhiên bạn nên lưu những bảng người dùng (user tables) trên filegroup khác thay vì PRIMARY.
2. Nên chọn default filegroup khác PRIMARY.
3. Để đạt được hiệu năng tốt nhất, bạn nên tạo các data files và filegroup trên những ổ đĩa vật lý khác nhau (nếu có thể) và đặt những bảng có cường độ insert/update/delete cao trên những filegroup khác nhau.
4. Không nên đặt transaction log file trên cùng ổ đĩa với data files.
5. Những bảng có cường độ insert/update cao nên xem xét đặt các non-clustered indexes trên ổ đĩa khác với data (clustered index/heap)
6. Những bảng cùng trong mệnh đề JOIN của một câu query có thể đặt trên những filegroup khác nhau (ổ đĩa khác nhau) nhằm tối đa hiệu năng truy vấn.

Nguồn tham khảo:
1. [Database files and filegroups](https://docs.microsoft.com/en-us/sql/relational-databases/databases/database-files-and-filegroups?view=sql-server-ver15)
2. [Benchmarking: do multiple data files make a difference?](https://www.sqlskills.com/blogs/paul/benchmarking-do-multiple-data-files-make-a-difference/)
3. [Khảo sát cách tổ chức files và filegroups trong SQL Server database](https://www.sqlskills.com/blogs/paul/files-and-filegroups-survey-results/)
4. [Thêm mới data files](https://docs.microsoft.com/en-us/sql/relational-databases/databases/add-data-or-log-files-to-a-database?view=sql-server-ver15)
5. [8 Steps to better transaction log throughput](https://www.sqlskills.com/blogs/kimberly/8-steps-to-better-transaction-log-throughput/)

Nguồn: https://quantricsdulieu.com/2021/03/16/tim-hieu-sql-server-activity-monitor-p4-disk-i-o-latency/

---
