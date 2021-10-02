---
layout: post
title:  "Các lệnh trong linux"
permalink: 2021/10/02/linux-commands-cheat-sheet
tags: Linux Command
category: Linux
img: /assets/linux-commands-cheat-sheet/Hinh1.png
summary: Các lệnh trong linux

---

## Các lệnh Linux cơ bản ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| ls | Liệt kê tất cả các tệp và thư mục trong thư mục làm việc hiện tại |
| ls-R | Liệt kê các tệp trong thư mục con |
| ls-a | Liệt kê các tệp và các tệp ẩn |
| ls-al | Liệt kê các tệp và thư mục với thông tin chi tiết như quyền, kích thước, chủ sở hữu, v.v. |
| cd or cd ~ | Điều hướng đến thư mục HOME |
| cd .. | Di chuyển lên một cấp độ |
| cd | Để thay đổi một thư mục cụ thể |
| cd / | Di chuyển đến thư mục gốc |
| cat > filename | Tạo một tệp mới |
| cat filename | Hiển thị nội dung tệp |
| cat file1 file2 > file3 | Kết hợp hai tệp (file1, file2) và lưu trữ kết quả đầu ra trong tệp mới (file3) |
| mv file "new file path" | Di chuyển các tệp đến vị trí mới |
| mv filename new_file_name | Đổi tên tệp thành tên tệp mới |
| sudo | Cho phép người dùng thông thường chạy các chương trình có đặc quyền bảo mật của người dùng cấp cao hoặc người chủ |
| rm filename | Xóa một tệp |
| man | Cung cấp thông tin trợ giúp về một lệnh |
| history | Cung cấp danh sách tất cả các lệnh trước đây được nhập trong phiên hiện tại |
| clear | Xóa các lệnh đã thao tác |
| mkdir directoryname | Tạo một thư mục mới trong thư mục làm việc hiện tại hoặc một tại đường dẫn được chỉ định |
| rmdir | Xóa một thư mục |
| mv | Đổi tên một thư mục |
| pr -x | Chia tệp thành x cột |
| pr -h | Gán tiêu đề cho tệp |
| pr -n | Biểu thị tệp bằng Số dòng |
| lp -nc , lpr c | In “c” bản sao của tệp |
|  lp-d lp-P | Chỉ định tên của máy in |
| apt-get | Lệnh được sử dụng để cài đặt và cập nhật các gói |
| mail -s 'subject' -c 'cc-address' -b 'bcc-address' 'to-address' | Lệnh gửi email |
| mail -s "Subject" to-address < Filename | Lệnh gửi email có tệp đính kèm |


## Lệnh với quyền truy cập tệp ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| ls-l | để hiển thị loại tệp và quyền truy cập |
| r | quyền đọc |
| w | quyền ghi |
| x | quyền thực thi |
| -= | không cho phép |
| chown user | Để thay đổi quyền sở hữu tệp / thư mục |
| chown user:group filename | thay đổi người dùng cũng như nhóm cho một tệp hoặc thư mục |


## Lệnh với biến môi trường ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| echo $VARIABLE | Để hiển thị giá trị của một biến |
| env | Hiển thị tất cả các biến môi trường |
| VARIABLE_NAME= variable_value | Tạo một biến mới |
| unset | Xóa một biến |
| export Variable=value | Để đặt giá trị của một biến môi trường |

## Thông tin và quản lý người dùng ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| id | Hiển thị id người dùng và id nhóm của người dùng hiện tại của bạn. |
| who | Hiển thị những người dùng cuối cùng đã đăng nhập vào hệ thống. |
| w | Hiển thị ai đang trực tuyến |
| whoami | Bạn đăng nhập bằng tài khoản nào |
| sudo adduser username | Để thêm người dùng mới |
| sudo passwd -l 'username' | Để thay đổi mật khẩu của người dùng |
| sudo userdel -r 'username' | Để xóa người dùng mới được tạo |
| sudo usermod -a -G GROUPNAME USERNAME | Để thêm người dùng vào một nhóm |
| sudo deluser USER GROUPNAME | Để xóa một người dùng khỏi một nhóm |
| groupadd test | Tạo một nhóm có tên là "test" |
| finger | Hiển thị thông tin của tất cả người dùng đã đăng nhập |
| finger username | Cung cấp thông tin của một người dùng cụ thể |

## Lệnh với mạng ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| ssh username@ip-address or hostname | đăng nhập vào một máy Linux từ xa bằng SSH |
| ping hostname="" or ="" | Để ping và phân tích kết nối mạng và máy chủ |
| dir | Hiển thị các tệp trong thư mục hiện tại của máy tính từ xa |
| cd "dirname" | thay đổi thư mục thành “dirname” trên máy tính từ xa |
| put file | tải 'tệp' từ cục bộ lên máy tính từ xa |
| get file | Tải xuống 'tệp' từ điều khiển từ xa đến máy tính cục bộ |
| quit | Đăng xuất |

## Lệnh với process ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|
| bg | Để gửi một process ra nền |
| fg | Để chạy một process đã dừng ở phía trước |
| top | Thông tin chi tiết về tất cả các process đang hoạt động |
| ps | Cung cấp trạng thái của các process đang chạy cho người dùng |
| ps PID | Cung cấp trạng thái của một process cụ thể |
| pidof | Cung cấp process ID (PID) của một process |
| kill PID | Kết thúc một process |
| nice | Bắt đầu một process với một mức độ ưu tiên nhất định |
| renice | Thay đổi mức độ ưu tiên của một process đã chạy |
| df | Cung cấp thông tin dung lượng ổ cứng còn trống trên hệ thống |
| free | Cung cấp thông tin dung lượng RAM còn trống trên hệ thống |

## Lệnh trong trình soạn thảo VI ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|	
| i | Chèn tại con trỏ (chuyển sang chế độ chèn) |
| a | Viết sau con trỏ (chuyển sang chế độ chèn) |
| A | Viết ở cuối dòng (chuyển sang chế độ chèn) |
| ESC | Chấm dứt chế độ chèn |
| u | Hoàn tác thay đổi cuối cùng |
| U | Hoàn tác tất cả các thay đổi đối với toàn bộ dòng |
| o | Mở một dòng mới (chuyển sang chế độ chèn) |
| dd | Xóa dòng |
| 3dd | Xóa 3 dòng |
| D | Xóa nội dung của dòng sau con trỏ |
| C | Xóa nội dung của một dòng sau con trỏ và chèn văn bản mới. Nhấn phím ESC để kết thúc quá trình chèn. |
| dw | Xóa từ |
| 4dw | Xóa 4 từ |
| cw | Thay đổi từ |
| x | Xóa ký tự tại con trỏ |
| r | Thay thế ký tự |
| R | Ghi đè các ký tự từ con trỏ trở đi |
| s | Thay thế một ký tự dưới con trỏ tiếp tục chèn |
| S | Thay thế toàn bộ dòng và bắt đầu chèn vào đầu dòng |
| ~ | Thay đổi trường hợp của từng ký tự |

## Thông tin hệ thống ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|	
| uname -a | Hiển thị thông tin hệ thống Linux |
| uname -r | Hiển thị thông tin phát hành kernel |
| cat /etc/redhat-release | Hiển thị phiên bản Red Hat đã được cài đặt |
| uptime | Hiển thị thời gian hệ thống đã chạy + tải |
| hostname | Hiển thị tên máy chủ hệ thống |
| hostname -I | Hiển thị tất cả các địa chỉ IP cục bộ của máy chủ. |
| last reboot | Hiển thị lịch sử khởi động lại hệ thống |
| date | Hiển thị ngày và giờ hiện tại |
| cal | Hiển thị lịch của tháng này |

## Thông tin phần cứng ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|	
| dmesg | Hiển thị thông báo trong bộ đệm vòng kernel |
| cat /proc/cpuinfo | Hiển thị thông tin CPU |
| cat /proc/meminfo | Hiển thị thông tin bộ nhớ |
| free -h | Hiển thị bộ nhớ trống và đã sử dụng (-h cho con người có thể đọc được, -m cho MB, -g cho GB.) |
| lspci -tv | Hiển thị thiết bị PCI |
| lsusb -tv | Hiển thị thiết bị USB |
| dmidecode | Hiển thị DMI / SMBIOS (thông tin phần cứng) từ BIOS |
| hdparm -i /dev/sda | Hiển thị thông tin về đĩa sda |
| hdparm -tT /dev/sda | Thực hiện kiểm tra tốc độ đọc trên đĩa sda |
| hdparm -tT /dev/sda | Kiểm tra các khối không đọc được trên đĩa sda |

## Theo dõi và thống kê hiệu suất ##

|------------------+---------|
| Lệnh | Ý nghĩa |
|:----------------:|:---------|	
| top | Hiển thị và quản lý các process hàng đầu |
| htop | Trình xem process tương tác (thay thế hàng đầu) |
| mpstat 1 | Hiển thị thống kê liên quan đến bộ xử lý |
| vmstat 1 | Hiển thị thống kê bộ nhớ ảo |
| iostat 1 | Hiển thị thống kê I/O |
| tail -100 /var/log/messages | Hiển thị 100 thông báo nhật ký hệ thống gần đây nhất (Sử dụng /var/log/syslog cho hệ thống dựa trên Debian.) |
| tcpdump -i eth0 | Chụp và hiển thị tất cả các gói trên giao diện eth0 |
| tcpdump -i eth0 'port 80' | Giám sát tất cả lưu lượng trên cổng 80 (HTTP) |
| lsof | Liệt kê tất cả các tệp đang mở trên hệ thống |
| lsof -u user | Danh sách tệp do người dùng mở |
| free -h | Hiển thị bộ nhớ trống và đã sử dụng (-h cho con người có thể đọc được, -m cho MB, -g cho GB.) |
| watch df -h | Thực thi "df -h", hiển thị các bản cập nhật định kỳ |

**Tài liệu tham khảo**

- [guru99](https://www.guru99.com/linux-commands-cheat-sheet.html)
- [linuxtrainingacademy](https://www.linuxtrainingacademy.com/linux-commands-cheat-sheet/)

---
