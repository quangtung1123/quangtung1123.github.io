---
layout: post
title:  "Hướng dẫn rà soát phần mềm độc hại trên Linux"
date:   2022-02-20 08:10:00
permalink: 2022/02/20/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux
tags: Security Linux
category: Security
img: /assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh1.png
summary: Hướng dẫn rà soát phần mềm độc hại trên Linux

---

Trên Windows, có rất nhiều phần mềm cũng như giải pháp để phát hiện và ngăn chặn phần mềm độc hại. Tuy nhiên, trên các phiên bản phân phối của Linux, dường như có ít các giải pháp để phát hiện và ngăn chặn chúng. Vì vậy trong bài viết này, mình sẽ hướng dẫn các bạn cách rà soát phần mềm độc hại bằng các công cụ có sẵn trên Linux.

Các chương trình độc hại thường có mục đích cụ thể. Để đạt được mục đích này, malware cần thực hiện một số hoạt động cụ thể, chính những hoạt động này để lại dấu vết trên hệ thống, từ đó giúp chúng ta có thể điều tra và phát hiện chúng.

Các biểu hiện thường thấy ở phần mềm độc hại là:
- nghe trên một hoặc một số cổng xác định hoặc giao tiếp với máy chủ kiểm soát và ra lệnh (C2)
- tăng mức tiêu thụ tài nguyên máy tính như RAM, CPU, băng thông mạng...

Rà soát phần mềm độc hại

**1. Kiểm tra mức độ tiêu tốn tài nguyên**

Để kiểm tra tiến trình nào tiêu tốn tài nguyên hệ thống như CPU, RAM, chúng ta sử dụng lệnh: 

```bash
top
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Ở đây, chúng ta kiểm tra xem tiến trình nào sử dụng nhiều CPU, RAM và các tên các tiến trình đáng ngờ.

Khi tìm thấy tiến trình đáng ngờ, bạn có thể xem thông tin chi tiết bằng lệnh:

```bash
ps -f --forest -C tên_tiến_trình
```

Để kết thúc tiến trình đáng ngờ, bạn dùng lệnh:

```bash
kill -9 pid_tiến trình
```

**2. Kiểm tra mức sử dụng ổ đĩa**

Để kiểm tra mức độ sử dụng ổ đĩa, chúng ta dùng công cụ **iotop**.

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Để xem tiến trình nào đọc và ghi từ ổ đĩa trong mỗi 25 giây, ta dùng lệnh:

```bash
sudo pidstat -dl 25
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**3. Kiểm tra các cổng mở**

Phần mềm độc hại thường sẽ chạy trên một cổng nhất định, cố gắng mở cổng và thiết lập kết nối. Để xem các kết nối được thiết lập, ta dùng lệnh:

```bash
sudo ss -tupn
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh4.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Khi thấy một kết nối đáng ngờ, chúng ta có thể chặn chúng bằng cách sử dụng iptables.

Để xem trạng thái của các cổng đang mở, chúng ta dùng lệnh:

```bash
sudo ss -tulpn
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh5.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**4. Phân tích cách tệp được mở**

Để xem các tệp nào được mở có liên kết với một tiến trình cụ thể, chúng ta dùng lệnh:

```bash
sudo lsof | grep tên_tiến_trình
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh6.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**5. Kiểm tra các dịch vụ đang chạy**

Để kiểm tra các dịch vụ đang chạy, chúng ta dùng lệnh:

```bash
systemctl list-unit-files | grep active
```

**6. Tìm các tệp được tạo gần đây**

Phần mềm độc hại thường tạo ra một số tệp trên hệ thống cho một mục đích nhất định, chúng ta có thể tìm các tệp được tạo gần đây với lệnh **find**.

Ví dụ, để tìm các tệp được tạo trong 50 ngày trong thư mục, ta dùng lệnh:

```bash
find /bin/ -mtime -50
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh7.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Để xem tất cả các tệp được truy cập trong 50 ngày, ta dùng lệnh:

```bash
find / -atime 50
```

Để tìm các thuộc tính (permission, owner, group) đã bị thay đổi trong 50 phút trước đó, ta dùng lệnh:

```bash
find / -cmin -50
```

Để tìm tất cả các tệp đã được truy cập trong 60 phút trước đó, ta dùng lệnh:

```bash
find / -amin -60
```

**7. Kiểm tra các dịch vụ chạy cùng với hệ thống khi khởi động**

Phần mềm độc hại sẽ cố gắng tồn tại bền bỉ trên hệ thống, do đó, nó thường tìm cách khởi chạy cùng với hệ thống khi khởi động. Để kiểm tra các dịch vụ tự động chạy khi hệ thống khởi động, ta dùng lệnh:

```bash
systemctl list-unit-files | grep enabled
```

<div class="imgcap">
<div >
    <img src="/assets/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux/Hinh8.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**8. Kiểm tra Cron job.**

Để tránh sự phát hiện, phần mềm độc hại thường lập lịch để chạy một tác vụ nào đó trong khoảng thời gian xác định. Để đạt được điều này, malware thường sử dụng cron. Để kiểm tra tất cả các tác vụ đã được lập lịch cho tất cả user trên hệ thống, ta dùng lệnh:

```bash
for user in (cut -f1 -d: /etc/passwd); do sudo crontab -u(cut−f1−d:/etc/passwd);dosudocrontab−uuser -l 2>/dev/null | grep -v '^#'; done
```

**9. Kiểm tra các tập lệnh được thực thi tự động**

Linux có nhiều tập lệnh chạy tự động khi user đăng nhập vào hệ thống. Phần mềm độc hại lợi dụng tính năng này để tự động khởi chạy. Do đó, chúng ta cần rà soát các tệp và thư mục như
- /etc/profile
- /etc/profile.d/*
- ~/.bash_profile
- ~/.bashrc
- /etc/bashrc

**Tài liệu tham khảo**

- [whitehat](https://whitehat.vn/threads/huong-dan-ra-soat-phan-mem-doc-hai-tren-linux.16284/)

---
