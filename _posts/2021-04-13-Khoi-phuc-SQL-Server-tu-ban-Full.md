---
layout: post
comments: true
title:  "SQL SERVER - Phục hồi database SQL Server từ bản sao lưu Full"
title2:  "Phục hồi database SQL Server từ bản sao lưu Full"
date:   2021-04-13 15:10:00
permalink: 2021/04/13/Khoi-phuc-SQL-Server-tu-ban-Full
mathjax: false
tags: SQL_Server Database Restore
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/Khoi-phuc-SQL-Server-tu-ban-Full/Hinh1.png
summary: Phục hồi database SQL Server từ bản sao lưu Full
---

## **Tình huống** ##

Bạn sử dụng bản sao lưu Full để phục hồi database nếu rơi vào một trong hai trường hợp sau:
- Bản sao lưu Full là bản sao lưu gần thời điểm xảy ra sự cố nhất. Nghĩa là từ lúc tiến hành sao lưu Full đến lúc bị sự cố, bạn không có thêm bất kỳ bản sao lưu Differential hoặc Transaction log nào khác.
- Database chỉ được sao lưu Full mà không sao lưu Differential hoặc Transaction log. Lý do có thể vì Recovery model của database là SIMPLE khiến bạn không thể sao lưu Transaction log, hoặc đơn giản vì bạn không tiến hành sao lưu Differential hoặc Transaction log.

**Lưu ý**: Nếu muốn phục hồi database từ bản sao lưu Differential hay Transaction log, bạn vẫn phải phục hồi từ bản sao lưu Full ngay trước đó.

## **Các bước tiến hành** ##

Quá trình phục hồi database từ bản sao lưu Full được tiến hành theo 2 bước sau:
- Phục hồi bản sao lưu Full từ Disk/Tape/Cloud
- Khôi phục bản sao lưu Full vào database bằng lệnh của SQL Server

## **1. Phục hồi bản sao lưu Full từ Disk/Tape/Cloud** ##

Trước khi tiến hành bất kỳ thao tác khôi phục nào với SQL Server, bạn cần phải có bản sao lưu Full và tiến hành phục hồi dữ liệu từ bản sao lưu này vào máy tính. Tùy phần mềm/công cụ sao lưu trước đây, bạn tiến hành phục hồi theo hướng dẫn tương ứng.

**Lưu ý:** Không có bản sao lưu thì không thể nói đến chuyện phục hồi hay bất kỳ thao tác nào khác để khôi phục lại database. Do đó, bạn cần có phương án sao lưu và lưu trữ an toàn các bản sao lưu.

## **2. Khôi phục bản sao lưu Full vào database** ##

Có thể thực hiện khôi phục bằng 1 trong 2 cách:
- Sử dụng T-SQL
- Sử dụng SQL Server Management Studio (SSMS)

## **2.1. Sử dụng T-SQL** ##

Sau khi có được bản sao lưu Full, bạn tiến hành khôi phục database vào SQL Server bằng lệnh RESTORE DATABASE.

```sql
RESTORE DATABASE ERP
FROM DISK = ‘E:\SQLRestoreData\DATABASE_ERP.bak’
GO
```

Mặc định, lệnh RESTORE DATABASE sẽ khôi phục Data file và Log file vào thư mục trước đây của database (giá trị PhysicalName của lệnh RESTORE FILELISTONLY). Nếu thư mục này không tồn tại hoặc đã có file khác trùng tên thì lỗi sau xuất hiện:

```
File ‘ERP’ cannot be restored to ‘C:\Program Files\Microsoft SQL Server\MSSQL10.MSSQLSERVER\MSSQL\DATA\ERP_Data.mdf’. Use WITH MOVE to identify a valid location for the file
```

Để khắc phục lỗi trên, bạn cần sử dụng tùy chọn MOVE để chỉ định tên thư mục sẽ chứa Data file và Log file được phục hồi. Nhưng trước hết, bạn cần truy xuất các thông tin dữ liệu của database bằng lệnh RESORE FILELISTONLY.

```sql
RESTORE FILELISTONLY ERP
FROM DISK = ‘E:\SQLRestoreData\DATABASE_ERP.bak’
GO
```

Sau đó, sử dụng lệnh RESTORE DATABASE với tùy chọn MOVE để khôi phục database vào thư mục chỉ định.

```
RESTORE DATABASE ERP
FROM DISK = ‘E:\SQLRestoreData\DATABASE_ERP.bak’
MOVE ‘ERP_Data’
TO ‘E:\SQLData\ERP_Data.mdf’
MOVE ‘ERP_Log’
TO ‘E:\SQLData\ERP_Log.ldf’
GO
```

Sau khi lệnh RESTORE DATABASE được thực thi, database sẽ được khôi phục hoàn toàn và ở tình trạng có thể sử dụng được. Do đó, bạn không thể tiến hành phục hồi thêm từ bản sao lưu Differential hay Transaction log.

**Lưu ý:** Nếu database cần khôi phục bị trùng database đã có trong SQL Server, lỗi sau sẽ xuất hiện:

```
Msg 3159, Level 16, State 1, Line 1
The tail of the log for the database “ERP” has not been backed up. Use BACKUP LOG WITH NORECOVERY to backup the log if it contains work you do not want to lose. Use the WITH REPLACE or WITH STOPAT clause of the RESTORE statement to just overwrite the contents of the log.
Msg 3013, Level 16, State 1, Line 1
RESTORE DATABASE is terminating abnormally.
```

Với lỗi này, bạn sử dụng tùy chọn REPLACE để ghi đè database đang có.

```sql
RESTORE DATABASE ERP
FROM DISK = ‘E:\SQLRestoreData\DATABASE_ERP.bak’
WITH REPLACE
GO
```

Sau khi lệnh RESTORE DATABASE được thực thi xong, bạn refresh lại mục Databases trong SSMS và thấy database ERP vừa được phục hồi.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

**Một số tùy chọn khác bao gồm:**
- Tùy chọn **WITH**:
  - Ghi đè vào database đã có (**WITH REPLACE**)
  - Duy trì các thiết lập sao chép (**WITH KEEP_REPLICATION**)
  - Hạn chế truy cập vào CSDL đã khôi phục (**WITH RESTRICTED_USER**)
- Tùy chọn trạng thái **Recovery**, xác định trạng thái của CSDL sau khi khôi phục:
  - **RESTORE WITH RECOVERY** là tuỳ chọn mặc định, giúp CSDL sẵn sàng sử dụng bằng cách khôi phục các transaction chưa được commit. Chọn tùy chọn này nếu chỉ có 1 bản backup hoặc restore bản backup cuối cùng. Nếu chọn tùy chọn này thì sẽ không restore tiêp

## **2.2. Sử dụng SQL Server Management Studio (SSMS)** ##

Bước 1: Mở MSSQL Server Management Studio và kết nối đến SQL Server bằng tài khoản "sa" hoặc tài khoản có quyền restore database

Bước 2: Chuột phải vào mục Databases, chọn Restore Database…
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Bước 3: Chọn **Device**:, rồi chọn dấu ba chấm (...) để đến vị trí backup file.
Bước 4: Chọn **Add** di chuyển đến vị trí file .bak. Chọn .bak file và chọn **OK**.
Bước 5: Chọn **OK**  để đóng cửa sổ **Select backup devices**.
Bước 6: Chọn **OK** để băt đầu khôi phục

Bước 2: Gõ tên database ở mục To Database, sau đó chọn nơi chứa Database backup bằng cách click chọn From device và click ... :
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>


Bước 3: Click Add , sau đó tìm đến nới chứa và chọn file backup (*.bak). Sau đó bấm OK.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>


Bước 4: Click chọn database tương ứng bằng cách tích vào như hình dưới. Sau đó bấm OK để hoàn tất restore.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Trường hợp nhận thông báo lỗi sau khi nhấn nút OK, vào ngay mục Options bên trái ở bước 4, chọn Overwrite the existing database (WITH REPLACE) sau đó nhấn nút OK.

Vui lòng chờ trong giây lát để restore database.

## **Tài liệu tham khảo** ##

- [backupacademy](http://backupacademy.zbackup.vn/sql-server/phuc-hoi-sql-server-tu-ban-sao-luu-full/)
- https://docs.mendix.com/developerportal/deploy/restoring-a-sql-server-database
- 
---
