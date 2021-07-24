---
layout: post
comments: true
title:  "SQL SERVER - Câu lệnh DML và DDL trong SQL Server"
date:   2021-07-23 15:10:00
permalink: 2021/07/23/cau-lenh-dml-va-ddl-trong-sql-server
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/cau-lenh-dml-va-ddl-trong-sql-server/Hinh1.png
summary: Câu lệnh DML và DDL trong SQL Server

---

Bài hôm nay sẽ làm quen với khái niệm Data Manipulation Language (DML) và Data Definition Language (DDL) trong SQL Server.

## DML và DDL là gì? ##

[DDL](https://docs.microsoft.com/en-us/sql/t-sql/statements/statements?view=sql-server-ver15) là viết tắt của Ngôn Ngữ Định Nghĩa Dữ Liệu và do vậy có thể nói câu lệnh DDL được dùng để định hình dữ liệu của bạn trông như thế nào, tổ chức ra sao. Một số câu lệnh DDL phổ biến mà chúng ta dễ bắt gặp nhất chính là:

- **CREATE** – được dùng để tạo mới các đối tượng trong cơ sở dữ liệu như database, table, function, stored procedure, trigger.
- **ALTER** – được dùng để sửa đổi các đối tượng như table (thêm cột), column (sửa đổi kiểu dữ liệu), trigger (sửa đổi nội dung).
- **DROP** – dùng để xóa các đối tượng trong cơ sở dữ liệu.
- **TRUNCATE** – dùng để xóa tất cả dữ liệu của bảng một cách nhanh chóng.

[DML](https://docs.microsoft.com/en-us/sql/t-sql/statements/statements?view=sql-server-ver15) là viết tắt của Ngôn Ngữ Thao Tác Dữ liệu, chính là những câu lệnh truy vấn, thêm xóa sửa mà chúng ta thường dùng khi làm việc với dữ liệu lưu trữ trong SQL Server. Một số câu lệnh phổ biến như:

- **INSERT** – thêm dữ liệu vào một bảng trong cơ sở dữ liệu.
- **UPDATE** – sửa đổi dữ liệu trong một bảng.
- **DELETE** – xóa dòng dữ liệu trong bảng.
- **SELECT** – truy vấn dữ liệu.

## Một vài ví dụ của câu lệnh DDL và DML ##

Sử dụng DDL để tạo mới bảng, sau đó thêm cột, sử đổi kiểu dữ liệu, xóa cột.

```sql
CREATE DATABASE labdb03
GO
USE labdb03
GO
 
-- 1. tạo bảng
CREATE TABLE T1(
    Id INT,
    data1 SMALLDATETIME,
    data2 VARCHAR(16)
    )
GO
-- 2. sửa đổi bảng, thêm cột
ALTER TABLE T1 ADD data3 TINYINT
 
-- 3. sửa đổi kiểu dữ liệu của cột data3, đổi sang kiểu bigint
ALTER TABLE T1 ALTER COLUMN data3 BIGINT
 
-- 4. xóa cột
ALTER TABLE T1 DROP COLUMN data3
 
-- 5. xóa tất cả dữ liệu của bảng
TRUNCATE TABLE T1
```

Thêm dữ liệu vào bảng T1 ở trên, cập nhật thay đổi, xóa dữ liệu.

```
USE labdb03
GO
 
-- 1. thêm mới các dòng dữ liệu vào bảng T1
INSERT INTO T1(Id,data1, data2)
SELECT 1, GETDATE(), 'row 1'
UNION ALL
SELECT 2, GETDATE(), 'row 2'
UNION ALL
SELECT 3, GETDATE(), 'row 3'
 
-- 2. cập nhật các thay đổi
UPDATE T1 
SET data2 = ''
WHERE Id = 2
 
-- 3. xóa dòng dữ liệu của bảng T1
DELETE FROM T1 
WHERE Id = 2 
```

## Một vài khác biệt cơ bản giữa hai câu lệnh DDL và DML ##

- Câu lệnh DDL tác động lên các đối tượng của cơ sở dữ liệu như bảng, cột, view, trigger, stored procedure. Trong khi đó DML tác động lên đối tượng là các dòng dữ liệu trong bảng.
- Các câu lệnh DDL không có mệnh đề WHERE vì chúng sẽ ảnh hưởng đến cả đối tượng bị tác động. Câu lệnh DML có thể sử dụng mệnh đề WHERE để giới hạn số lượng dòng dữ liệu bị ảnh hưởng.

**Tài liệu tham khảo**

- [quantricsdulieu](https://quantricsdulieu.com/cau-lenh-dml-va-ddl-trong-sql-server/)

---
