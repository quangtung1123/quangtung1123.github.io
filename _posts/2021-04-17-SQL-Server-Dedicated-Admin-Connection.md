---
layout: post
comments: true
title:  "SQL SERVER - SQL Server Dedicated Admin Connection (DAC) dùng làm gì?"
title2:  "SQL Server Dedicated Admin Connection (DAC) dùng làm gì?"
date:   2021-04-17 15:10:00
permalink: 2021/04/17/SQL-Server-Dedicated-Admin-Connection
mathjax: false
tags: SQL_Server Database Restore
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/SQL-Server-Dedicated-Admin-Connection/Hinh1.png
summary: SQL Server Dedicated Admin Connection (DAC) dùng làm gì?
---

## **Dedicated Admin Connection là gì?**

[**Dedicated Admin Connection**](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/diagnostic-connection-for-database-administrators?view=sql-server-ver15) viết tắt là **DAC** theo đúng nghĩa đen là **kết nối dành riêng cho quản trị viên**. Trong một số tình huống khi SQL Server gặp vấn đề nghiêm trọng về hiệu năng việc thiết lập một kết nối tới SQL Server sẽ không dễ dàng như lúc bình thường. Vì khi đó một số tài nguyên sẽ bị hạn chế như [worker threads](/2021/04/16/khai-niem-trong-quan-ly-luong-sqlos) đã hết, tài nguyên CPU đang bị tranh chấp và % Processor Time có thể lên đến 100% nên dù có kết nối được cũng sẽ gặp vấn đề hiệu năng khi thao tác trên kết nối đó.

Những lúc như thế này DAC sẽ cho phép quản trị viên truy cập vào SQL Server bằng con đường dành riêng cho quản trị viên và thực hiện các câu truy vấn cơ bản để xử lý sự cố mà hạn chế được việc tranh chấp tài nguyên.

[Mặc định](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/remote-admin-connections-server-configuration-option?view=sql-server-ver15) sau khi cài đặt, SQL Server chỉ cho phép thực hiện kết nối sử dụng DAC khi đứng trên chính server đó. Để có thể thực hiện kết nối từ xa, chúng ta cần kích hoạt tính năng “remote admin connection” trong sys.configurations.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Dedicated-Admin-Connection/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1: mặc định giá trị của remote admin connection trong sys.configurations</div>
</div>
<hr>

SQL Server dành riêng resource như [scheduler](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-schedulers-transact-sql?view=sql-server-ver15), worker threads cho connection dạng Dedicated Admin Connection (DAC) để đảm bảo luôn sẵn sàng khi cần.

## **Kích hoạt remote dedicated admin connection như thế nào?**

Có hai cách để cho phép kết nối sử dụng dedicated admin connection (DAC) từ xa. Sử dụng T-SQL hoặc sử dụng giao diện Surface Area Configuration trên SSMS.

### **Sử dụng T-SQL để kích hoạt remote DAC**

Chúng ta có thể sử dụng đoạn T-SQL bên dưới để kích hoạt remote dedicated admin connection.

```sql
Use master
GO
sp_configure 'remote admin connections', 1 
GO
RECONFIGURE
GO
```

### **Sử dụng giao diện từ SSMS để kích hoạt remote DAC**

Từ cửa sổ **Object Explorer** trên SSMS, click chuột phải trên **instance name** chọn **Facets**. Trong cửa sổ **view Facets**, tại mục **Facet** chọn **Surface Area Configuration**. Dưới phần **Facet Properties**, chọn giá trị “True” cho **RemoteDacEnabled** như hình bên dưới. Sau đó click **OK** để lưu giá trị thay đổi.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Dedicated-Admin-Connection/Hinh2.png" width = "800">
</div>
<div class="thecap">Hình 2: kích hoạt remote DAC bằng giao diện trên SSMS.</div>
</div>
<hr>

Bạn có thể kiểm tra giá trị “remote admin connection” đã thay đổi chưa bằng cách chạy lại câu truy vấn trong hình 1.

## **Sử dụng remote DAC như thế nào?**

Để sử dụng được tính năng này bạn cần là member của sysadmin role. Có hai cách thực hiện kết nối sử dụng DAC là thông qua SQL Server command line sqlcmd hoặc thông qua SSMS.

Cú pháp khi sử dụng **sqlcmd** như sau:

```sql
SQLCMD -S [SQL Server Name] -U [User Name] -P [Password] -A 
```

Bởi vì tại một thời điểm chỉ có một connection được kết nối sử dụng DAC nên bạn không thể từ cửa sổ Object Explorer trên SSMS connect vô SQL Server database engine, thay vào đó bạn mở một cửa sổ query bất kì và chuột phải chọn **Connection > Change connection…** và nhập từ "**ADMIN:**" trước server name như hình dưới đây, sau đó nhập username và password như thường và click connect.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Dedicated-Admin-Connection/Hinh3.png" width = "800">
</div>
<div class="thecap">Hình 3: chuẩn bị kết nối tới SQL Server sử dụng DAC.</div>
</div>
<hr>
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Dedicated-Admin-Connection/Hinh4.png" width = "800">
</div>
<div class="thecap">Hình 4: sử dụng từ khóa ADMIN: để login bằng DAC.</div>
</div>
<hr>
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Dedicated-Admin-Connection/Hinh5.png" width = "800">
</div>
<div class="thecap">Hình 5: Đăng nhập thành công.</div>
</div>
<hr>

Mặc dù bạn đăng nhập thành công nhưng error message sẽ xuất hiện như hình 5. Đặc điểm để bạn nhận dạng đã thành công là thông tin server của connection hiện tại có thêm từ ADMIN phía trước server name ở thanh status (status bar) như hình trên.

Đứng ở cửa sổ query này, bạn click **New Query** để mở cửa sổ mới với connection như vậy thì sẽ nhận được thông báo như hình sau, và tất nhiên không thành công vì tại một thời điểm chỉ có duy nhất một DAC connection như mình đề cập ở trên.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/SQL-Server-Dedicated-Admin-Connection/Hinh6.png" width = "800">
</div>
<div class="thecap">Hình 6: Lỗi nhận được khi new query từ một cửa sổ DAC.</div>
</div>
<hr>

Như vậy là bạn đã đăng nhập thành công với dedicated admin connection, bây giờ bạn có thể thực hiện công việc troubleshooting của mình như thường rồi.

## **Tài liệu tham khảo** ##

- [quantricsdulieu](https://quantricsdulieu.com/2021/04/15/sql-server-dedicated-admin-connection-dac-dung-lam-gi/)
---
