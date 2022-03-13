---
layout: post
title:  "Một số lưu ý khi cài đặt Linux"
date:   2022-03-13 10:10:00
permalink: 2022/03/13/mot-so-luu-y-khi-cai-dat-linux
tags: Linux
category: Linux
img: /assets/mot-so-luu-y-khi-cai-dat-linux/Hinh1.png
summary: Một số lưu ý khi cài đặt Linux

---

Khi cài mới đặt hệ điều hành Linux, chúng ta cần chú ý một số phần sau để tránh việc phải cài đặt lại sau này.

## Installation Destination: phân vùng ##

Trong phần phân vùng, để tùy chỉnh sâu hơn ta sẽ chọn Custom thay vì Automatic ở mục Storage Configuration:

<div class="imgcap">
<div >
    <img src="/assets/mot-so-luu-y-khi-cai-dat-linux/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Tại giao diện Manual Partitioning, chúng ta có thể lựa chọn theo nhu cầu sử dụng như sau: 
- Standard Partition (Phân vùng tiêu chuẩn): có thể chứa một hệ thống tệp hoặc trao đổi không gian hoặc cung cấp một thùng chứa cho RAID phần mềm hoặc khối lượng vật lý LVM.
- LVM (Phân vùng Khối lượng logic): tạo ra một khối logic LVM rất hữu ích vì nó cải thiện hiệu suất khi sử dụng đĩa vật lý. Điều này rất thiết thực vì khi sử dụng nó, bạn có thể dễ dàng thay đổi kích thước các phân vùng của mình bằng cách thêm một ổ cứng mới.
- LVM Thin Provisioning (Cung cấp mỏng LVM): giúp quản lý một kho lưu trữ không gian trống thường được gọi là một hồ mỏng. Nhóm mỏng là hữu ích vì nó có thể được mở rộng linh hoạt khi cần thiết để phân bổ không gian lưu trữ hiệu quả.

<div class="imgcap">
<div >
    <img src="/assets/mot-so-luu-y-khi-cai-dat-linux/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Tiếp đó chọn button + để thêm các partition tùy ý.
Theo mặc định chúng ta nên khởi tạo các partition bao gồm:
- /boot với dung lượng từ 200MB trở lên, phân vùng này sẽ chứa các kernel dùng để boot hệ điều hành lên. Với 200MB có thể chưa được khoảng 4 phiên bản kernel.
- swap: Tuy nhiên phần này tuy thuộc vào nhu cầu sử dụng và có thể mở rộng.
- / là phần vùng cuối cùng cần được tạo, phân vùng này sẽ là nơi chứa toàn bộ dữ liệu máy chủ của bạn, trong một số trường hợp người quản trị sẽ chia từng phân vùng cụ thể như /home /var nhằm chia tách các dữ liệu với nhau. Dung lượng phân vùng / sẽ chọn toàn bộ phần AVAILABLE SPACE còn lại.

<div class="imgcap">
<div >
    <img src="/assets/mot-so-luu-y-khi-cai-dat-linux/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Điều quan trọng cần lưu ý là khi chọn phân vùng, bạn cũng cần chọn hệ thống tệp phù hợp tùy thuộc vào những gì bạn cần. Về các hệ thống tập tin có sẵn trong quá trình cài đặt, bạn có:
- BIOS Boot: cần thiết để khởi động thiết bị trên hệ thống BIOS
- EFI System Partition: cần thiết để khởi động thiết bị trên hệ thống UEFI
- vfat: đó là một hệ thống tệp Linux tương thích với các tên tệp dài của Microsoft Windows trên hệ thống tệp FAT
- Ext – Extended file system: là định dạng file hệ thống đầu tiên được thiết kế dành riêng cho Linux. Có tổng cộng 4 phiên bản Ext1, Ext2, Ext3, Ext4. Hiện nay đa phần người dùng sử dụng định dạng Ext4 vì nó có thể giảm bớt hiện tượng phân mảnh dữ liệu trong ổ cứng, hỗ trợ các file và phân vùng có dung lượng lớn.
- XFS: Khá tương đồng với Ext4 về một số mặt nào đó, chẳng hạn như hạn chế được tình trạng phân mảnh dữ liệu, không cho phép các snapshot tự động kết hợp với nhau, hỗ trợ nhiều file dung lượng lớn lên tới 16 EiB, có thể thay đổi kích thước file dữ liệu. XFS khá phù hợp với việc áp dụng vào mô hình server media vì khả năng truyền tải file video rất tốt. Tuy nhiên, nhiều phiên bản distributor yêu cầu phân vùng /boot riêng biệt, hiệu suất hoạt động với các file dung lượng nhỏ không bằng được khi so với các định dạng file hệ thống khác, do vậy sẽ không thể áp dụng với mô hình database, email và một vài loại server có nhiều file log. Nếu dùng với máy tính cá nhân, thì đây cũng không phải là sự lựa chọn tốt nên so sánh với Ext, vì hiệu suất hoạt động không khả thi, ngoài ra cũng không có gì nổi trội về hiệu năng, quản lý so với Ext3/4.

**Tài liệu tham khảo**

- [7host](https://www.7host.vn/huong-dan-cai-dat-centos-8-toan-tap/)
- [cloudviet](https://cloudviet.com.vn/huong-dan-cach-cai-dat-cau-hinh-he-dieu-hanh-linux-centos-8/)

---
