---
layout: post
comments: true
title:  "SQL SERVER - Tổ chức lưu trữ trong SQL Server - data pages và data rows"
title2:  "Tổ chức lưu trữ trong SQL Server - data pages và data rows"
date:   2021-04-07 15:10:00
permalink: 2021/04/07/To-chuc-luu-tru-trong-SQL-Server
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/To-chuc-luu-tru-trong-SQL-Server/Hinh1.png
summary: Tổ chức lưu trữ trong SQL Server - data pages và data rows
---

Database gồm nhiều filegroup, mỗi filegroup sẽ có một hoặc nhiều data files, mỗi data file gồm nhiều data pages, mỗi data page sẽ có một hoặc nhiều rows tùy theo kích thước của mỗi row.

Chúng ta biết rằng cơ sở dữ liệu quan hệ (relational database) lưu trữ dữ liệu của người dùng dưới dạng hàng cột – hay gọi là bảng. Mỗi bảng có thể có một hoặc nhiều cột và mỗi cột phải thuộc về một kiểu dữ liệu nào đó như số nguyên (integer), ngày tháng năm (date), chuỗi các ký tự (varchar)… Mỗi dòng (row) sẽ có giá trị cho từng cột. Kích thước của một row chính là tổng kích thước các kiểu dữ liệu của các cột cộng với một số bytes phát sinh của việc tổ chức lưu trữ trong SQL Server.

Những rows này sẽ được gom lại thành các đơn vị lớn hơn gọi là page. Mỗi page có kích thước cố định là 8KB ( 8192 bytes) và các page này nằm liên tiếp trên các [data files](/2021/03/10/Cau-tao-database-SQL-Server). Nhằm mục đích hỗ trợ việc cấp phát không gian lưu trữ cho các bảng hiệu quả hơn SQL Server sử dụng đơn vị extent – một extent gồm 8 data pages nằm kế nhau hình thành một khối 64KB. Vậy có thể hình dung các cấp trong việc tổ lưu trữ SQL Server gồm **database -> filegroups -> files -> extents -> pages -> rows**.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/To-chuc-luu-tru-trong-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: Data file gồm các 8KB data pages liên tiếp nhau.</div>
</div>
<hr>

## **Data pages**

Mỗi page được định vị bằng địa chỉ FileID:PageID và hoặc là thuộc về một bảng nào đó (có thể là user hoặc system tables) hoặc là page của hệ thống, như dùng để quản lý việc cấp phát không gian lưu trữ. Page đã có bao nhiêu rows, còn trống bao nhiêu bytes,..tất cả những thông tin cần thiết để việc quản lý và truy cập data pages hiệu quả hơn.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/To-chuc-luu-tru-trong-SQL-Server/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: Các thành phần trong cấu trúc một data page.</div>
</div>
<hr>
Page header là nơi chứa thông tin quản lý như đề cập ở trên, vùng này chiếm cố định 96 bytes. Tiếp đến là payload – vùng chứa data rows và phía dưới cùng là slot array – mảng các phần tử 2 bytes chỉ vị trí bắt đầu các rows trong page. Số lượng phần tử này tương ứng với số lượng rows và thứ tự của chúng phản ánh thứ tự các rows trong index theo key. Các bạn chú ý, vị trí các data rows trong payload không quan trọng, thứ tự của slot mới quyết định vị trí của chúng trong index.

Để lấy danh sách các pages mà một bảng (clustered/heap table) hoặc một index có thể có, ta sử dụng câu lệnh [DBCC IND](https://www.sqlskills.com/blogs/paul/inside-the-storage-engine-using-dbcc-page-and-dbcc-ind-to-find-out-if-page-splits-ever-roll-back/) hoặc undocumented DMF [sys.dm_db_database_page_allocations](https://sqlity.net/en/2471/sys-dm_db_database_page_allocations/) (từ SQL Server 2012 trở lên) với cú pháp như sau.

DBCC IND(<database_name>,<table_name>,<index_id>)

```sql
CREATE DATABASE quantricsdulieu
GO
USE quantricsdulieu
GO
 
CREATE TABLE tblTest(
    Id INT IDENTITY PRIMARY KEY CLUSTERED,
    NgayNhap DATETIME,
    MoTa1 VARCHAR(64),
    MoTa2 CHAR(1024)
    )
 
DECLARE @i INT = 1
BEGIN TRANSACTION
WHILE @i <= 100
BEGIN
    INSERT INTO tblTest(NgayNhap,MoTa1,MoTa2)
    SELECT GETDATE(),NEWID(),CAST(@i AS CHAR(8))
    SET @i = @i + 1
END
COMMIT
 
GO
 
DBCC IND('quantricsdulieu','tblTest',1)
 
GO
```
<hr>
<div class="imgcap">
<div >
    <img src="/assets/To-chuc-luu-tru-trong-SQL-Server/Hinh3.png" width = "800">
</div>
<div class="thecap">Hình 3: các data pages của bảng tblTest.</div>
</div>
<hr>
Đoạn code T-SQL trên chỉ đơn giản tạo database quantricsdulieu sau đó tạo bảng tblTest với primary key clustered trên cột Id và đổ data vào. Hình 2 cho thấy phần payload chỉ có tối đa 8096 bytes nên với kích thước row của bảng tblTest mỗi page chỉ có thể chứa tối đa 7 rows, chúng ta sẽ kiểm tra điều này sớm thôi. Ý nghĩa các cột trong kết quả của câu lệnh **DBCC IND** như sau:

- **PageFID** – File ID chứa data page này.
- **PagePID** – page ID, đánh số thứ tự từ đầu file là 0,1,2,..
- **IAMFID** – Tất cả pages của một bảng/index đều được theo dõi thông qua các IAM pages (Index Allocation Map), và IAM page này cũng nằm trên file, cột này thể hiện file ID mà IAM đang map page (dòng) này thuộc về. Nếu dòng này là IAM page thì cột này sẽ NULL.
- **IAMPID** – ID của page IAM đang map page (dòng) này.
- **ObjectID** – ID của bảng mà page này thuộc về.
- **IndexID** – ID của index mà page này thuộc về.
- **PartitionNumber** – chính là giá trị partition_number trong DMV [sys.partitions](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-partitions-transact-sql?view=sql-server-ver15).
- **PartitionID** – Ià giá trị partitionid trong DMV sys.partitions.
- **iam_chain_type** – là loại [allocation unit](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-allocation-units-transact-sql?view=sql-server-ver15) mà IAM sử dụng.
- **PageType** – loại page, có các loại phổ biến như
  - 1 – page chứa data của bảng (data pages)
  - 2 – page chứa các index record
  - 3 và 4 – chứa text data
  - 8 – GAM page (global allocation map)
  - 9 – SGAM page (shared global allocation map)
  - 10 – IAM page (index allocation map)
  - 11 – PFS page (page free space)
- **IndexLevel** – cấp của page này trong cây b-tree, 0 là node lá và cao nhất là node root
- **NextPageFID và NextPagePID** – địa chỉ page kế tiếp của page này, sử dụng cặp FileID:PageID. Trong B-tree các page liên kết nhau theo hai chiều.
- **PrevPageFID và PrevPagePID** – địa chỉ page trước của page này.

Kết quả trong hình 3 cho ta thấy clustered index b-tree của bảng tblTest này node root là level 1, các node lá có level 0. Page root ko có page liền trước và liền sau. Chúng ta hãy xét xem PageID 312 chứa những thông tin gì từ DMV [sys.dm_db_page_info](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-db-page-info-transact-sql?view=sql-server-ver15) hoặc bằng câu lệnh **DBCC PAGE** với cú pháp:

DBCC PAGE( {‘dbname’ | dbid}, filenum, pagenum [, printopt={0|1|2|3} ])

```sql
DBCC TRACEON(3604)
DBCC PAGE('quantricsdulieu',1,312,1)
DBCC TRACEOFF(3604)
```

Chúng ta sử dụng thêm câu lệnh **DBCC TRACEON(3604)** để kết quả output ra màn hình SSMS. Các bạn có thể chọn print option (tham số thứ 3) khác nhau để khảo sát page này. Với print option là 1 thì kết quả page header như hình dưới đây:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/To-chuc-luu-tru-trong-SQL-Server/Hinh4.png" width = "800">
</div>
<div class="thecap">Hình 4: Thông tin chứa trong page header của PageID 312.</div>
</div>
<hr>

Các bạn có thể đoán được ý nghĩa của các values mình đánh dấu ở trên? m_type chính là loại page, m_slotCnt là số dòng trong page, ObjectID và IndexID là ám chỉ bảng clustered index của bảng tblTest. Ngoài ra còn nhiều thông tin khác chúng ta chưa vội khám phá lúc này.

## **Data rows**

Một data row (record) thông thường (không compress) có cấu trúc như hình bên dưới (trích từ ebook [Microsoft SQL Server 2008 Internal](https://www.oreilly.com/library/view/microsoft-sql-server/9780735634787/), vẫn còn đúng cho các version sau này)
<hr>
<div class="imgcap">
<div >
    <img src="/assets/To-chuc-luu-tru-trong-SQL-Server/Hinh4.png" width = "800">
</div>
<div class="thecap">Hình 5: cấu trúc của một data row.</div>
</div>
<hr>

Có nhiều loại record như data, index, text, ghost (sau khi delete),forwarding/forwarded (trong bảng heap) và bạn có thể đọc thêm [ở trang này](https://www.sqlskills.com/blogs/paul/inside-the-storage-engine-anatomy-of-a-record/) . Tất cả records có cấu trúc giống nhau, chỉ khác về số cột và data type các cột. Một record thông thường (chưa compress) có cấu trúc như hình trên và có thể hiểu như sau:
- Record header
  - có kích thước 4 bytes
  - hai bytes đầu giữ thông tin về loại record
  - hai bytes sau lưu chiều dài (n bytes) của fixed-length data type
- Vùng lưu trữ data của fixed-length data type (n bytes). Những data type như INT, DATETIME, CHAR của bảng tblTest được lưu trong này, có chiều dài cố định.
- Hai bytes tiếp theo giữ số lượng cột trong bảng.
- Tùy vào số lượng cột mà chiều dài NULL bitmap có thể khác nhau, công thức chính là \[số lượng cột\]/8 làm tròn thành byte. Ví dụ 9 cột sẽ có 2 bytes cho NULL bitmap.
- Hai bytes tiếp theo lưu số lượng cột kiểu variable-length data type (varchar, nvarchar)
- Hai bytes cho mỗi cột kiểu variable-length data type, trỏ đến vùng lưu data phía sau.
- Tiếp đến là vùng lưu data của variable-length.

Hiểu được các thành phần tham gia vào lưu trữ và cấu trúc của chúng sẽ giúp rất nhiều trong việc thiết kế database hiệu quả, thiết kế, sử dụng và bảo trì index. Ngoài ra nó là tiền đề để hiểu và phân tích các vấn đề về locking và blocking.

## **Tài liệu tham khảo:**

- [Inside the Storage Engine: Anatomy of a page](https://www.sqlskills.com/blogs/paul/inside-the-storage-engine-anatomy-of-a-page/)
- [Inside the Storage Engine: Anatomy of a record](https://www.sqlskills.com/blogs/paul/inside-the-storage-engine-anatomy-of-a-record/)
- [Inside the Storage Engine: Anatomy of an extent](https://www.sqlskills.com/blogs/paul/inside-the-storage-engine-anatomy-of-an-extent/)
- [Inside The Storage Engine: GAM, SGAM, PFS and other](https://www.sqlskills.com/blogs/paul/inside-the-storage-engine-gam-sgam-pfs-and-other-allocation-maps/)

Nguồn: [quantricsdulieu](https://quantricsdulieu.com/2021/04/03/to-chuc-luu-tru-trong-sql-server-data-pages-data-rows/)

---
