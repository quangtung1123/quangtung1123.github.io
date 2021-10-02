---
layout: post
title:  "Các lệnh trong linux"
permalink: 2021/10/02/linux-commands-cheat-sheet
tags: WAF Nginx ModSecurity
category: WAF
img: /assets/linux-commands-cheat-sheet/Hinh1.png
summary: Các lệnh trong linux

---

## Các lệnh Linux cơ bản ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| max-age | Khoảng thời gian (tính bằng giây) để thông báo cho trình duyệt biết rằng các yêu cầu chỉ khả dụng qua HTTPS. |
| ls | Liệt kê tất cả các tệp và thư mục trong thư mục làm việc hiện tại |
| ls-R | Liệt kê các tệp trong thư mục con |
| ls-a | Liệt kê các tệp ẩn cũng như |
| ls-al | Liệt kê các tệp và thư mục với thông tin chi tiết như quyền, kích thước, chủ sở hữu, v.v. |
| cd or cd ~ | Điều hướng đến thư mục HOME |
| cd .. | Di chuyển lên một cấp độ |
| cd | Để thay đổi một thư mục cụ thể |
| cd / | Di chuyển đến thư mục gốc |
| cat > filename | Tạo một tệp mới |
| cat filename | Hiển thị nội dung tệp |
| cat file1 file2 > file3 | Kết hợp hai tệp (tệp1, tệp2) và lưu trữ kết quả đầu ra trong tệp mới (tệp3) |
| mv file "new file path" | Di chuyển các tệp đến vị trí mới |
| mv filename new_file_name | Đổi tên tệp thành tên tệp mới |
| sudo | Cho phép người dùng thông thường chạy các chương trình có đặc quyền bảo mật của người dùng cấp cao hoặc người chủ |
| rm filename | Xóa một tệp |
| man | Cung cấp thông tin trợ giúp về một lệnh |
history	Cung cấp danh sách tất cả các lệnh trước đây được nhập trong phiên đầu cuối hiện tại |
| clear | Xóa thiết bị đầu cuối |
| mkdir directoryname | Tạo một thư mục mới trong thư mục làm việc hiện tại hoặc một tại đường dẫn được chỉ định |
| rmdir | Xóa một thư mục |
| mv | Đổi tên một thư mục |
| pr -x | Chia tệp thành x cột |
| pr -h | Gán tiêu đề cho tệp |
| pr -n | Biểu thị tệp bằng Số dòng |
| lp -nc , lpr c | In bản sao “c” của Tệp |
|  lp-d lp-P | Chỉ định tên của máy in |
| apt-get | Lệnh được sử dụng để cài đặt và cập nhật các gói |
| mail -s 'subject' -c 'cc-address' -b 'bcc-address' 'to-address' | Lệnh gửi email |
| mail -s "Subject" to-address < Filename | Lệnh gửi email có tệp đính kèm |


## Lệnh cho phép tệp ##

Chỉ huy	Sự miêu tả	
ls-l	để hiển thị loại tệp và quyền truy cập
r	quyền đọc
w	viết quyền
x	thực thi quyền
-=	không cho phép
Chown user	Để thay đổi quyền sở hữu tệp / thư mục
Chown user:group filename	thay đổi người dùng cũng như nhóm cho một tệp hoặc thư mục
Lệnh biến môi trường
Chỉ huy	Sự miêu tả	
echo $VARIABLE	Để hiển thị giá trị của một biến
env	Hiển thị tất cả các biến môi trường
VARIABLE_NAME= variable_value	Tạo một biến mới
Unset	Xóa một biến
export Variable=value	Để đặt giá trị của một biến môi trường
Các lệnh quản lý người dùng của linux
Chỉ huy	Sự miêu tả	
sudo adduser username	Để thêm người dùng mới
sudo passwd -l 'username'	Để thay đổi mật khẩu của người dùng
sudo userdel -r 'username'	Để xóa người dùng mới được tạo
sudo usermod -a -G GROUPNAME USERNAME	Để thêm người dùng vào một nhóm
sudo deluser USER GROUPNAME	Để xóa một người dùng khỏi một nhóm
finger	Hiển thị thông tin của tất cả người dùng đã đăng nhập
finger username	Cung cấp thông tin của một người dùng cụ thể
Lệnh mạng
Chỉ huy	Sự miêu tả	
SSH username@ip-address or hostname	đăng nhập vào một máy Linux từ xa bằng SSH
Ping hostname="" or =""	Để ping và phân tích kết nối mạng và máy chủ lưu trữ
dir	Hiển thị các tệp trong thư mục hiện tại của máy tính từ xa
cd "dirname"	thay đổi thư mục thành “dirname” trên máy tính từ xa
put file	tải 'tệp' từ cục bộ lên máy tính từ xa
get file	Tải xuống 'tệp' từ điều khiển từ xa đến máy tính cục bộ
quit	Đăng xuất
Xử lý lệnh
Chỉ huy	Sự miêu tả	
bg	Để gửi một quy trình đến nền
fg	Để chạy một quá trình đã dừng ở phía trước
top	Thông tin chi tiết về tất cả các Quy trình đang hoạt động
ps	Cung cấp trạng thái của các quy trình đang chạy cho người dùng
ps PID	Cung cấp trạng thái của một quá trình cụ thể
pidof	Cung cấp ID quy trình (PID) của một quy trình
kill PID	Giết một quá trình
nice	Bắt đầu một quy trình với một mức độ ưu tiên nhất định
renice	Thay đổi mức độ ưu tiên của một quy trình đã chạy
df	Cung cấp dung lượng đĩa cứng trống trên hệ thống của bạn
free	Cung cấp RAM miễn phí trên hệ thống của bạn
VI Chỉnh sửa lệnh
Chỉ huy	Sự miêu tả	
i	Chèn tại con trỏ (chuyển sang chế độ chèn)
a	Viết sau con trỏ (chuyển sang chế độ chèn)
A	Viết ở cuối dòng (chuyển sang chế độ chèn)
ESC	Chấm dứt chế độ chèn
u	Hoàn tác thay đổi cuối cùng
U	Hoàn tác tất cả các thay đổi đối với toàn bộ dòng
o	Mở một dòng mới (chuyển sang chế độ chèn)
dd	Xóa dòng
3dd	Xóa 3 dòng
D	Xóa nội dung của dòng sau con trỏ
C	Xóa nội dung của một dòng sau con trỏ và chèn văn bản mới. Nhấn phím ESC để kết thúc quá trình chèn.
dw	Xóa từ
4dw	Xóa 4 từ
cw	Thay đổi từ
x	Xóa ký tự tại con trỏ
r	Thay thế ký tự
R	Ghi đè các ký tự từ con trỏ trở đi
s	Thay thế một ký tự dưới con trỏ tiếp tục chèn
S	Thay thế toàn bộ dòng và bắt đầu chèn vào đầu dòng
~	Thay đổi trường hợp của từng ký tự

**Tài liệu tham khảo**

- [guru99](https://www.guru99.com/linux-commands-cheat-sheet.html)

---
