---
layout: post
comments: true
title:  "SQL SERVER - Sao lưu và khôi phục"
title2:  "Sao lưu và khôi phục SQL Server"
date:   2021-04-12 15:10:00
permalink: 2021/04/12/Sao-luu-va-khoi-phuc-SQL-Server
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh0.png
summary: Các chế độ sao lưu và khôi phục cơ sở dữ liệu trong SQL Server
---

## **Chiến lược phục hồi dữ liệu (Data Restoration Strategy)**

Khi nào ta cần khôi phục lại dữ liệu? Có rất nhiều lý do mà người Quản trị CSDL (Database Administrator) phải đảm bảo để dữ liệu của hệ thống được chính xác, không bị sai lệch vàn cần phải giảm tối đa số lần phải phục hồi dữ liệu, luôn theo dõi, kiểm tra thường xuyên để phát hiện các trục trặc trước khi nó xảy ra. Phải dự phòng các biến cố có thể xảy ra và bảo đảm rằng có thể nhanh chóng phục hồi dữ liệu trong thời gian sớm nhất có thể được.

Các dạng biến cố hay tai họa có thể xảy ra là:
- Ðĩa chứa file dữ liệu hoặc file Transaction Log hay file hệ thống bị mất
- Server bị hư hỏng
- Những thảm họa tự nhiên như bão lụt, động đất, hỏa hoạn
- Toàn bộ server bị đánh cắp hoặc phá hủy
- Các thiết bị dùng để backup – restore bị đánh cắp hay hư hỏng
- Những lỗi do vô ý của người sử dụng như lỡ tay xoá dữ liệu chẳng hạn
- Những hành vi mang tính phá hoại của nhân viên như cố ý đưa vào những thông tin sai lạc.
- Bị hack (nếu server có kết nối với internet).

Bạn phải tự hỏi khi các vấn đề trên xảy ra thì bạn sẽ làm gì và phải luôn có biện pháp đề phòng cụ thể cho từng trường hợp cụ thể. Ngoài ra bạn phải xác định thời gian tối thiểu cần phục hồi dữ liệu và đưa server trở lại hoạt động bình thường.

## **Các loại sao lưu (Backup)**

Ðể có thể hiểu các kiểu phục hồi dữ liệu khác nhau bạn phải biết qua các loại sao lưu trong SQL Server
- **Full Database Backups** : Copy tất cả các file dữ liệu trong một database . Tất cả những user data và database objects như system tables, indexes, user-defined tables đều được backup.
- **Differential Database Backups** : Copy những thay đổi trong tất cả các file dữ liệu kể từ lần backup gần nhất.
File or File Group Backups : Copy một data file đơn hay một nhóm file.
- **Differential File or File Group Backups** : Tương tự như differential database backup nhưng chỉ copy những thay đổi trong data file đơn hay một file group.
- **Transaction Log Backups** : Ghi nhận một cách thứ tự tất cả các giao dịch (transaction) chứa trong file transaction log kể từ lần transaction log backup gần nhất. Loại sao lưu này cho phép ta phục hồi dữ liệu trở ngược lại vào một thời điểm nào đó trong quá khứ mà vẫn đảm bảo tính đồng nhất (consistent).

Trong lúc backup SQL Server cũng copy tất cả các hoạt động của database kể cả hoạt động xảy ra trong quá trình backup cho nên ta có thể backup trong khi SQL đang chạy mà không cần phải dừng lại.

## **Các chế độ khôi phục (Recovery Models)**

- **Chế độ Full Recovery**: Ðây là chế độ cho phép phục hồi dữ liệu với ít rủi ro nhất. Nếu một database ở trong chế độ này thì tất cả các hoạt động không chỉ insert, update, delete mà kể cả insert bằng **Bulk Insert**, hay **bcp** đều được log vào file transaction log. Khi có sự cố thì ta có thể phục hồi lại dữ liệu ngược trở lại tới một thời điểm trong quá khứ. Khi file dữ liệu bị hư nếu ta có thể sao lưu được file transaction log thì ta có thể phục hồi CSDL đến thời điểm transaction gần nhất đã được xác nhận (commited).
- **Chế độ Bulk-Logged Recovery**: Ở chế độ này các hoạt động mang tính hàng loạt như Bulk Insert, bcp, Create Index, WriteText, UpdateText chỉ được log minimum vào File Transaction Log đủ để cho biết là các hoạt động này có diễn ra mà không log toàn bộ chi tiết như trong chế độ Full Recovery. Các hoạt động khác như Insert, Update, Delete vẫn được log đầy đủ để dùng cho việc phục hồi sau này.
- **Chế độ Simple Recovery**: Ở chế độ này thì File Transaction Log được cắt xén thường xuyên và không cần sao lưu. Với chế độ này bạn chỉ có thể phục hồi tới thời điểm backup gần nhất mà không thể phục hồi tới một thời điểm trong quá khứ.

Muốn biết CSDL của bạn đang ở mode nào bạn có thể **Right-click lên một database nào đó** trong SQL Server Management Studio chọn **Properties->Options->Recovery model**

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh1.png" width = "800">
</div>
<div class="thecap">Chế độ khôi phục dữ liệu</div>
</div>
<hr>

Tuy nhiên có thể tới đây bạn cảm thấy rất khó hiểu về những điều trình bày ở trên. Chúng ta hãy dùng một ví dụ sau để làm rõ vấn đề.

**Ví dụ:**

Chúng ta có một database được áp dụng chiến lược sao lưu như hình vẽ sau:

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh2.gif" width = "800">
</div>
<div class="thecap">Chiến lược sao lưu một Database</div>
</div>
<hr>

Trong ví dụ này chúng ta lập lịch ở chế độ *Full Database Backup* vào ngày Chủ Nhật và *Differential Backup* vào các ngày thứ Ba và Thứ Năm. *Transaction Log Backup* được lập lịch hằng ngày. Vào một ngày Thứ Sáu "đen tối" một sự cố xảy ra đó là đĩa chứa file dữ liệu của CSDL bị hỏng và bạn là một  quản trị CSDL, bạn được yêu cầu phải phục hồi dữ liệu và đưa CSDL hoạt động trở lại  bình thường. Bạn phải làm sao?

Trước hết bạn phải sao lưu ngay File Transaction Log (trong ví dụ này File Transaction Log được chứa trong một đĩa khác với đĩa chứa File dữ liệu nên không bị hỏng và vẫn còn hoạt động). Người ta còn gọi file sao lưu trong trường hợp này là "**the tail of the log**" (phần đuôi). Nếu Log File được chứa trên cùng một đĩa với File dữ liệu thì bạn có thể sẽ không sao lưu được "phần đuôi" và như vậy bạn phải dùng đến file sao lưu log gần nhất. Khi sao lưu "phần đuôi" này bạn cần phải dùng tuỳ chọn **NO_TRUNCATE** bởi vì thông thường các Transaction Log Backup sẽ cắt xén những phần không cần dùng đến trong transaction log file, đó là những transaction đã được chấp nhận và đã được viết vào CSDL (còn gọi là inactive portion of the transaction log) để giảm kích thước của log file. Tuy nhiên khi sao lưu phần đuôi không được cắt xén để đảm bảo tính nhất quán (consistent) của database.

Kế đến bạn phải khôi phục CSDL (restore database) từ File Full Backup của ngày Chủ Nhật. Nó sẽ làm 2 chuyện: copy dữ liệu, log, index, … từ đĩa sao lưu vào các file dữ liệu và sau đó sẽ lần lượt thực thi các transaction trong transaction log. Lưu ý ta phải dùng tuỳ chọn **WITH NORECOVERY** trong trường hợp này (tức là tuỳ chọn thứ 2 "**Leave database nonoperational but able to restore additional transaction logs**" trong SQL Server Management Studio). Nghĩa là các transaction chưa hoàn tất (incomplete transaction) sẽ **không được roll back**. Như vậy CDSL lúc này sẽ ở trong tình trạng **inconsistent** và không thể dùng được. Nếu ta chọn **WITH RECOVERY** (hay "**Leave database operational. No additional transaction logs can be restored**" trong SQL Server Management Studio) thì các incomplete transaction sẽ được roll back và CSDL ở trạng thái **consistent nhưng ta không thể nào khôi phục các transaction log backup được nữa**.

Tiếp theo bạn phải khôi phục Differential Backup của ngày Thứ Năm. Sau đó lần lượt khôi phục các Transaction Log Backup kể từ sau lần Differential Backup cuối cùng nghĩa là restore Transaction Log Backup của ngày Thứ Năm và "Phần Ðuôi". Như vậy ta có thể phục hồi data trở về trạng thái trước khi biến cố xảy ra. Quá trình này gọi là **phục hồi CSDL (Database Recovery)**.

Cũng xin làm rõ cách dùng từ **Database Restoration** và **Database Recovery** trong SQL Server. Hai từ này nếu dịch ra tiếng Việt đều có nghĩa là phục hồi cơ sở dữ liệu nhưng khi đọc sách tiếng Anh phải cẩn thận vì nó có nghĩa hơi khác nhau.

Như trong ví dụ trên khi ta **restore** database từ một file backup nghĩa là chỉ đơn giản tái tạo lại database từ những file backup và thực thi lại những transaction đã được commit nhưng database có thể ở trong trạng thái inconsistent và không sử dụng được. Nhưng khi nói đến **recover** nghĩa là ta không chỉ phục hồi lại data mà còn bảo đảm cho nó ở trạng thái consistent và sử dụng được (usable).

Có thể bạn sẽ hỏi **consistent** là thế nào? Phần này sẽ được trình bày trong bài về Tính toàn ven dữ liệu (Data Integrity). Ví dụ : Giả sử số tiền VND 500000 được trừ khỏi tài khoản A nhưng lại không được cộng vào tài khoản B và nếu CSDL không được quá trình khôi phục dữ liệu tự động (automatic recovery process) của SQL rollback thì nó sẽ ở trạng thái không đồng nhất. Nếu CSDL ở trạng thái giống như trước khi trừ tiền hoặc sau khi đã cộng VND 500000 thành công vào tài khoản B thì gọi là đồng nhất.

Cho nên việc sao lưu File Transaction Log sẽ giúp cho việc recovery dữ liệu tới bất kỳ thời điểm nào trong quá khứ. Ðối với chế độ Simple Recovery ta chỉ có thể recover tới lần sao lưu gần nhất mà thôi.

Như vậy khi restore database ta có thể chọn tuỳ chọn **WITH RECOVERY** để roll back các transaction chưa được commited  và CSDL có thể hoạt động bình thường nhưng ta không thể restore thêm backup file nào nữa, thường tuỳ chọn này được chọn khi restore file backup cuối cùng trong chuỗi backup. Nếu chọn tuỳ chọn **WITH NORECOVERY** các transaction chưa được commited sẽ không được roll back do đó SQL Server sẽ không cho phép ta sử dụng database nhưng ta có thể tiếp tục restore các file backup kế tiếp, thường option này được chọn khi sau đó ta còn phải restore các file backup khác.

Không lẽ chỉ có thể chọn một trong hai option trên mà thôi hay sao? Không hoàn toàn như vậy ta có thể chọn một tuỳ chọn trung lập hơn là tuỳ chọn **WITH STANDBY** (tức là tuỳ chọn 3 “**Leave database read-only and able to restore additional transaction logs**” trong SQL Server Management Studio). Với tuỳ chọn này ta sẽ có luôn đặc tính của hai tuỳ chọn trên: các incomplete transaction sẽ được roll back để đảm bảo sự đồng nhất của CSDLvà có thể sử dụng được nhưng chỉ dưới dạng Read-only mà thôi, đồng thời sau đó ta có thể tiếp tục restore các file backup còn lại (SQL Server sẽ log các transaction được roll back trong undo log file và khi ta restore backup file kế tiếp SQL Server sẽ trả lại trạng thái no recovery từ những gì ghi trên undo file). Người ta dùng option này khi muốn restore database trở lại một thời điểm nào đó (a point in time) nhưng không rõ là đó có phải là thời điểm mà họ muốn không, cho nên họ sẽ restore từng backup file ở dạng **Standby** và kiểm chứng một số data xem đó có phải là thời điểm mà họ muốn restore hay không (chẳng hạn như trước khi bị delete hay trước khi một transaction nào đó được thực thi) trước khi chuyển sang Recovery option.

## **Sao lưu CSDL – Backup Database** ##

Trong phần này chúng ta sẽ bàn về cách sao lưu CSDL. Nhưng trước hết chúng ta hãy làm quen với một số thuật ngữ dùng trong quá trình sao lưu và phục hồi. Có những từ ta sẽ để nguyên tiếng Anh mà không dịch.

| **Thuật Ngữ** | 	**Giải Thích** |
| Backup | 	Quá trình copy toàn bộ hay một phần của database, transaction log, file hay file group hình thành một backup set. Backup set được chứa trên backup media (tape or disk) bằng cách sử dụng một backup device (tape drive name hoặc physical filename) |
| Backup Device | 	Định nghĩa việc sao lưu một Device logic tới một file trên ổ đĩa.  Một device logic là tên người dùng tự định nghĩa mà trỏ tới một device sao lưu vật lý  (như C:\SQLBackups\Full.bak) hoặc tape drive (như \\.\Tape0). |
| Backup File	File |  chứa một backup set |
| Backup Media | 	Disk hay tape được sử dụng để chứa một backup set. Backup media có thể chứa nhiều backup sets (ví dụ như từ nhiều SQL Server 2000 backups và từ nhiều Windows 2000 backups). |
| Backup Set | 	Một bộ backup từ một lần backup đơn được chứa trên backup media. |

### **1. Backup Device** ###

Chúng ta có thể tạo một backup device cố định (permanent) hay tạo ra một backup file mới cho mỗi lần backup. Thông thường chúng ta sẽ tạo một backup device cố định để có thể dùng đi dùng lại đặc biệt cho việc tự động hóa công việc backup. Ðể tạo một backup device dùng SQL Management Studio bạn làm như sau:

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh3.png" width = "800">
</div>
<div class="thecap">Cách sao lưu backup device</div>
</div>
<hr>

- Sau khi kết nối tới một instance thích hợp của Microsoft SQL Server Database Engine, trong cửa sổ **Object Explorer**, kích vào tên Server hiển thị cây các đối tượng trong mục này.
- Mở  Server Objects, và kích chuột phải Backup Devices.
- Kích New Backup Device. Hộp thoại Backup Device mở ra.
- Nhập vào Device Name.
- Phần Destination, kích File và chỉ định đường dẫn đầy đủ của file.
- Hoàn thành bằng cách kích nút OK.
- 
Ngoài ra bạn có thể dùng một stored procedure có tên sp_addumpdevice  như ví dụ sau:
- Kết nối tới Microsoft SQL Server Database Engine.
- Từ trên thanh Toolbar chuẩn, kích New Query.
- Copy và paste ví dụ sau vào cửa sổ query và kích Execute. Ví dụ này giới thiệu cách sử dụng [sp_addumpdevice](https://msdn.microsoft.com/en-us/library/ms188409.aspx) để định nghĩa một logical backup device cho một file của đĩa. Ví dụ này thêm thiết bị sao lưu đĩa có tên mydiskdump tới một tên vật lý C:\SQLBackups\Full.bak.

```sql
USE Master
Go
Sp_addumpdevice 'disk' , 'mydiskdump' , 'C:\SQLBackups\Full.bak'
```

### **2. Backup database** ###

Ðể backup database bạn có thể dùng Backup Wizard hoặc kích lên trên tên database muốn backup sau đó kích chuột phải chọn Tasks->Back Up … sẽ hiện ra cửa sổ như sau:

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh4.png" width = "800">
</div>
<div class="thecap">Cách sao lưu Database</div>
</div>
<hr>

Sau đó dựa tùy theo yêu cầu của database mà chọn các option thích hợp. Ta có thể schedule cho SQL Server backup định kỳ.

## **Khôi phục CSDL – Restore Database** ##

Trước khi khôi phục CSDL ta phải xác định được thứ tự file cần khôi phục. Các thông tin này được SQL Server chứa trong CSDL msdb và sẽ cho ta biết backup device nào, ai backup vào thời điểm nào. Sau đó ta tiến hành restore. Ðể khôi phục CSDL  kích chuột phải ->Tasks ->Restore -> Database… sẽ thấy cửa sổ như sau:

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh5.png" width = "800">
</div>
<div class="thecap">Khôi phục cơ sở dữ liệu</div>
</div>
<hr>

Chú ý: Nếu bạn khôi phục CSDL ***từ một instance khác của SQL Server hay từ một server khác*** bạn nên lựa chọn  **From device** và chọn file backup tương ứng.

Nếu bạn muốn ghi đè cơ sở dữ liệu có sẵn với dữ liệu được sao lưu bạn có thể chọn tuỳ chọn **Overwrite the existing database** như hình:

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Sao-luu-va-khoi-phuc-SQL-Server/Hinh6.png" width = "800">
</div>
<div class="thecap">Các tuỳ chọn khôi phục CSDL</div>
</div>
<hr>

Bạn có thể chọn **leave database operational** hoặc **nonoperational** tùy theo trường hợp như đã giải thích ở trên.

## **Kết luận** ##

Trong bài này chúng ta đã tìm hiểu một về cách sao lưu và phục hồi một CSDL (backup và restore database) trong SQL Server. Ðể có thể hiểu rõ hơn bạn cần phải thực tập hay làm thử để có thêm kinh nghiệm. Trong bài sau chúng ta sẽ bàn về chủ đề toàn vẹn dữ liệu (Data Integrity) nghĩa là làm sao để đảm bảo dữ liệu chứa trong CSDL là đáng tin cậy và không bị dư thừa dữ liệu.

Nguồn: [timoday](https://timoday.edu.vn/bai-4-sao-luu-va-khoi-phuc-database-trong-sql-server/)

---
