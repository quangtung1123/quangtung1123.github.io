---
layout: post
comments: true
title:  "SQL SERVER - So sánh câu lệnh DELETE với TRUNCATE"
date:   2021-07-24 15:10:00
permalink: 2021/07/24/so-sanh-cau-lenh-delete-voi-truncate
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/so-sanh-cau-lenh-delete-voi-truncate/Hinh1.png
summary: So sánh câu lệnh DELETE với TRUNCATE

---

Cả hai câu lệnh DELETE với TRUNCATE đều mang lại kết quả là xóa data của một bảng, tuy nhiên nếu đào sâu vào kết quả chúng lại có rất nhiều điểm khác nhau.

## DML và DDL ##

TRUNCATE là câu lệnh [DDL](/2021/07/23/cau-lenh-dml-va-ddl-trong-sql-server), vậy nên đối tượng của nó là bảng. DELETE là câu lệnh [DML](/2021/07/23/cau-lenh-dml-va-ddl-trong-sql-server) nên đối tượng của nó là các dòng dữ liệu trong bảng.

Cú pháp của câu lệnh TRUNCATE:

```sql
TRUNCATE TABLE T1;
```

Cú pháp của câu lệnh DELETE:

```
DELETE FROM T1;
```

## Mệnh đề WHERE ##

Câu lệnh TRUNCATE không sử dụng được mệnh đề WHERE, và nó sẽ xóa tất cả dữ liệu của bảng. Còn với câu lệnh DELETE, bạn có thể chỉ định xóa một hoặc một vài dòng hoặc tất cả với mệnh đề WHERE.

```sql
DELETE FROM T1
WHERE data1 >= 3;
```

## Sử dụng khóa (locking) ##

Câu lệnh TRUNCATE sẽ yêu cầu khóa **Sch-M** (Schema modification lock), đây là khóa tương tự dùng để sửa đổi bảng (ALTER TABLE) bởi vì câu lệnh TRUNCATE sửa đổi định nghĩa không gian lưu trữ của bảng. Cần thận trọng khi thực hiện các câu lệnh sử dụng khóa **Sch-M**, vì chúng có thể gây ra blocking dây chuyền.

Câu lệnh DELETE sẽ yêu cầu khóa **X** (exclusive lock) trên ROW hoặc PAGE hoặc TABLE tùy theo ngữ cảnh.

## Tốc độ thực hiện ##

Câu lệnh TRUNCATE chỉ [ghi log](https://sqlperformance.com/2013/05/sql-performance/drop-truncate-log-myth) (transaction log) cho hành động sửa đổi không gian lưu trữ nên sinh ra ít log records hơn nên có **tốc độ nhanh hơn**.

Câu lệnh DELETE phải ghi lại từng dòng bị xóa vào transaction log nên đòi sẽ tốn thời gian hơn, do đó sẽ chạy chậm hơn.

## Cột IDENTITY ##

Câu lệnh TRUNCATE sẽ reset (thiết lập về giá trị bắt đầu) giá trị cột IDENTITY trong bảng. Do đó sau khi bạn truncate bảng thì giá trị cột này chạy từ đầu.

Câu lệnh DELETE không làm thay đổi giá trị IDENTITY của bảng, nên dòng dữ liệu thêm mới sẽ có giá trị tiếp theo trên cột IDENTITY.

## Quyền hạn cần thiết (permission) ##
Để chạy được câu lệnh TRUNCATE bạn cần quyền ALTER trên bảng đó.

Để chạy được câu lệnh DELETE bạn cần quyền delete trên bảng đó.

**Tài liệu tham khảo**

- [quantricsdulieu](https://quantricsdulieu.com/so-sanh-cau-lenh-delete-voi-truncate/)

---
