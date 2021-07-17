---
layout: post
comments: true
title:  "DDOS - Hướng dẫn chống ddos trên Linux (phần 2) - IP Header"
title2:  "Hướng dẫn chống ddos trên Linux (phần 2) - IP Header"
date:   2021-07-02 21:45:00
permalink: 2021/07/02/Huong-dan-chong-ddos-tren-linux-p2
mathjax: false
tags: Security DDOS Linux
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh0.jpg
summary: Hướng dẫn chống ddos trên Linux
---

**Kỹ thuật chống DDoS - Phân tích ddos bằng IP Header**

Nắm vững kỹ năng phân tích IP header sẽ giúp bạn phân tích DDoS một cách nhanh chóng, chính xác và hiệu quả.

## **Lấy mẫu gói tin cần phân tích ddos**

Đơn giản nhất, ta có thể mở Terminal với lệnh ping liên tục **ping 8.8.8.8**. Song song, mở một Terminal thứ hai dùng **tcpdump** capture 2 gói tin *ICMP request* và *ICMP response* để làm mẫu.

Kết quả có thể như sau:

```
root@livebox:/tmp# tcpdump -nni any icmp -vv -X -c2
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
23:45:19.121629 IP (tos 0x0, ttl 64, id 27680, offset 0, flags [DF], proto ICMP (1), length 84)
    192.168.1.29 > 8.8.8.8: ICMP echo request, id 26786, seq 1, length 64
    0x0000:  4500 0054 6c20 4000 4001 fcb3 c0a8 011d  E..Tl.@.@.......
    0x0010:  0808 0808 0800 e7e8 68a2 0001 9f68 445d  ........h....hD]
    0x0020:  0000 0000 03db 0100 0000 0000 1011 1213  ................
    0x0030:  1415 1617 1819 1a1b 1c1d 1e1f 2021 2223  .............!"#
    0x0040:  2425 2627 2829 2a2b 2c2d 2e2f 3031 3233  $%&'()*+,-./0123
    0x0050:  3435 3637                                4567
23:45:19.152894 IP (tos 0x20, ttl 55, id 0, offset 0, flags [none], proto ICMP (1), length 84)
    8.8.8.8 > 192.168.1.29: ICMP echo reply, id 26786, seq 1, length 64
    0x0000:  4520 0054 0000 0000 3701 b1b4 0808 0808  E..T....7.......
    0x0010:  c0a8 011d 0000 efe8 68a2 0001 9f68 445d  ........h....hD]
    0x0020:  0000 0000 03db 0100 0000 0000 1011 1213  ................
    0x0030:  1415 1617 1819 1a1b 1c1d 1e1f 2021 2223  .............!"#
    0x0040:  2425 2627 2829 2a2b 2c2d 2e2f 3031 3233  $%&'()*+,-./0123
    0x0050:  3435 3637                                4567
2 packets captured
2 packets received by filter
0 packets dropped by kernel
```

Ta “hít” được 2 packets, thông tin về packet đầu tiên được bắt đầu và kết thúc:

```
23:45:19.121629 IP (tos 0x0, ttl 64, id 27680, offset 0, flags [DF], proto ICMP (1), length 84)
    192.168.1.29 > 8.8.8.8: ICMP echo request, id 26786, seq 1, length 64
    0x0000:  4500 0054 6c20 4000 4001 fcb3 c0a8 011d  E..Tl.@.@.......
    0x0010:  0808 0808 0800 e7e8 68a2 0001 9f68 445d  ........h....hD]
    0x0020:  0000 0000 03db 0100 0000 0000 1011 1213  ................
    0x0030:  1415 1617 1819 1a1b 1c1d 1e1f 2021 2223  .............!"#
    0x0040:  2425 2627 2829 2a2b 2c2d 2e2f 3031 3233  $%&'()*+,-./0123
    0x0050:  3435 3637                                4567
```

, trong đó:

Với tcpdump option **-vv**, 2 dòng đầu tiên hiển thị tóm tắt các thông tin về IP header và ICMP header mà gói tin mang theo:

```
23:45:19.121629 IP (tos 0x0, ttl 64, id 27680, offset 0, flags [DF], proto ICMP (1), length 84)
192.168.1.29 > 8.8.8.8: ICMP echo request, id 26786, seq 1, length 64
```

Tham số **-X** giúp hiển thị data “hít” được dưới dạng số **hex** và **ASCII**. Những giá trị này giúp ích khi sử dụng một số iptables modules như string, u32, bpf,…

```
0x0000:  4500 0054 6c20 4000 4001 fcb3 c0a8 011d  E..Tl.@.@.......
0x0010:  0808 0808 0800 e7e8 68a2 0001 9f68 445d  ........h....hD]
0x0020:  0000 0000 03db 0100 0000 0000 1011 1213  ................
0x0030:  1415 1617 1819 1a1b 1c1d 1e1f 2021 2223  .............!"#
0x0040:  2425 2627 2829 2a2b 2c2d 2e2f 3031 3233  $%&'()\*+,-./0123
0x0050:  3435 3637                                4567
```

Tương tự, gói tin thứ 2 chứa tin nhắn “ICMP echo reply”.

## **PHÂN TÍCH DDoS BẰNG IP HEADER**

Cấu trúc một IP header:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh1.png" width = "800">
</div>
<div class="thecap">Hình 1</div>
</div>
<hr>

*Nếu đọc giả chưa có kiến thức hoặc không thoải mái với các định nghĩa trên IP Header, bạn có thể tạm dừng và tham khảo lại tài liệu về IP header trước khi tiếp tục.*

Chúng ta bắt đầu tập phân tích từ Field đầu tiên của IP Header:

**Version (4 bits đầu)**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh2.jpg" width = "800">
</div>
<div class="thecap">Hình 2</div>
</div>
<hr>

Giá trị **0x4** cho biết đây là một gói tin IPv4. Nếu là IPv6 sẽ có giá trị **0x6**.

*Liệu có bất thường nếu một ngày server nhận hàng loạt packets có giá trị khác?*

**IHL (Internet Header Length). 4 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh3.jpg" width = "800">
</div>
<div class="thecap">Hình 3</div>
</div>
<hr>

Giá trị **0x5** cho biết độ dài của IP Header là 5 ***words*** (Mỗi word 32 bits), do đó:

IP header có độ dài 20 bytes, bắt đầu và kết thúc “**4500 0054 6c20 4000 4001 fcb3 c0a8 011d 0808 0808**“. Phần còn lại là data (gói ICMP).

20 bytes là độ dài tối thiểu của một IP header, vậy packet này không có IP options.

*Dịch vụ của bạn có sử dụng thêm IP options?*

**TOS (Type of Service). 8 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh4.jpg" width = "800">
</div>
<div class="thecap">Hình 4</div>
</div>
<hr>

Field TOS của gói echo-request bằng 0x00, gói tin echo-reply từ Google 8.8.8.8 bằng “0x20“.

*“0x20” tương ứng |0010 0000|. Field TOS được bật bit thứ 5 có ý nghĩa gì?
Ứng dụng của bạn khi trao đổi dữ liệu có sử dụng bit đặc biệt nào không? Giá trị thường thấy là bao nhiêu?*

**Total length. 16 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh5.jpg" width = "800">
</div>
<div class="thecap">Hình 5</div>
</div>
<hr>

Tổng độ dài packet (Tính cả IP Header và Data mang theo) là 0x0054 hay 84 bytes. Ta biết IP Header dài 20 byptes, vậy Data phía sau dài 84 – 20 = 64 bytes.

**Identification. 16 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh6.jpg" width = "800">
</div>
<div class="thecap">Hình 6</div>
</div>
<hr>

Định danh gói tin, mỗi gói tin được đánh số duy nhất. Gói một có id 0x6c20 hay id 27608. Mặt định khi build gói tin, HĐH đánh số packet sau tăng một đơn vị so với packet trước đó.

*Trường hợp gói tin reply từ Google có id cố định bằng 0x0000 (không) là một giá trị đặc biệt của máy chủ Google. Một số công cụ (hoặc lập trình) crafted packets tạo ra traffic DoS với các gói tin có id cố định, đây cũng là một dấu hiệu nhận biết để phân biệt traffic tấn công trong quá trình phân tích DDoS.*

**IP fragmentation fieds. 16 bit.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh7.jpg" width = "800">
</div>
<div class="thecap">Hình 7</div>
</div>
<hr>

4 bytes tiếp theo cung cấp thông tin: liệu gói tin có phân mảnh (fragment). Nếu có phân mảnh xảy ra, HĐH sẽ thực hiện quá trình hợp nhất (reassembly) các gói tin bị phân mảnh lại với nhau, dựa theo giá trị offset trên từng gói.

Theo qui định, 1 bit unused đầu tiên không được sử dụng và thường set về 0.
1 bit flag DF (Don’t fragment).
1 bit fag MF (More fragments).
13 bits còn lại, giá trị offset. Offset thường chỉ có giá trị nếu Datagram có phân mảnh.

Trong hai gói tin mẫu:

Gói tin thứ nhất: 0x4000 tương đương \|0100 0000 0000 0000\|, cho biết bit “DF” được bật, không có More fragments và offset bằng không.
Gói tin thứ hai: 0x0000 \|0000 0000 0000 0000\| không có bit flag nào bật, offset bằng không.

*IP fragmentation là một chủ đề rộng và có nhiều dạng tấn công khai thác xoay quanh(Teardrop, TCP Header Fragments…).*

*Kiến thức fragmentation cũng giúp ta phân biệt được những crafted packets không được build theo đúng chuẩn, gói không phân mảnh nhưng offset khác 0 là một ví dụ.*

**TTL (Time to Live). 8 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh8.jpg" width = "800">
</div>
<div class="thecap">Hình 8</div>
</div>
<hr>

Mặc định khi build gói tin, giá trị TTL khởi tạo của HĐH Linux là 64, HĐH Windows là 128, các thiết bị Cisco là 254,… Đồng thời, khi truyền đi trên Internet, giá trị TTL được giảm đi 1 khi đi qua một hop (router, modem, host,.. các thiết bị layer 3).

Gói tin thứ nhất có TTL 0x40 hay 64, được tạo bởi máy tính cá nhân HĐH Linux, được captured trên card mạng khi chưa đi qua hop nào. Do đó, giá trị TTL 64 và chưa giảm.

Gói tin thứ hai có TTL 0x37 hay 55, bạn có thể dự đoán tại server 8.8.8.8 gói tin được khởi tạo với TTL 64, vào internet về đến máy tính của tôi đã truyền qua qua 9 hops khác nhau.

**Protocol. 8 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh9.jpg" width = "800">
</div>
<div class="thecap">Hình 9</div>
</div>
<hr>

Giá trị 0x01 cho biết giao thức lớp trên là ICMP – hay gói tin mang theo một tin nhắn ICMP phía sau.

*Giao thức lớp trên: TCP có giá trị 0x06, UDP có giá trị 0x11.*

**Header checksum. 16 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh10.jpg" width = "800">
</div>
<div class="thecap">Hình 10</div>
</div>
<hr>

2 bytes tiếp theo là giá trị checksum của IP header và IP options.

**Source IP address. 32 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh11.jpg" width = "800">
</div>
<div class="thecap">Hình 11</div>
</div>
<hr>

4 bytes 0xc0a8011d lưu địa chỉ source IP của gói tin.

Cụ thể, 0xc0a8011d chia thành 4 octecs \| c8 \| a9 \| 01 \| 1d \| tương đương thập phân \|192 \| 168 \| 1 \| 29\|.

Vậy gói tin này được tạo từ máy tính của tôi với source IP 192.168.1.29.

**Destination IP address. 32 bits.**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh12.jpg" width = "800">
</div>
<div class="thecap">Hình 12</div>
</div>
<hr>

4 bytes lưu địa chỉ đích của gói tin. Với giá trị 0x08080808, ta dễ dàng nhận biết gói tin được gửi đến địa chỉ 8.8.8.8 của Google.

**IP data**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p2/Hinh13.jpg" width = "800">
</div>
<div class="thecap">Hình 13</div>
</div>
<hr>

Thông qua IP Header Length, chúng ta đã biết phần Header của gói tin IP đang xét dài 20 bytes và không có IP Options. Chúng ta đã xét đủ từ byte đầu tiên đến byte thứ 20 của IP Header.

Phần còn lại dài 64 bytes là data mà gói IP đang mang. Thông qua field Protocol bằng 0x01 ta cũng biết được thêm rằng data phía sau là một gói ICMP.

Phân tích chi tiết về gói tin ICMP nằm ngoài phạm vi bài viết, đọc giả có thể tự mình tìm hiểu và phân tích ICMP header một cách tương tự.

## **Kết thúc phân tích ddos bằng IP Header**

Hầu hết các fields của IP header chứa những thông tin hữu ích khi phân tích DDoS. Với từng field cụ thể sẽ có những kiến thức về tiêu chuẩn đóng gói, giá trị mặt định, giá trị bất thường, kỹ thuật khai thác,… đa dạng và là chủ đề mở rộng so với nội dung bài viết này.

Thường xuyên theo dõi traffic ra vào Server giúp bạn quen thuộc với traffic của ứng dụng đang quản trị, giúp nhận diện được những điểm dị biệt của traffic tấn công.

Nguồn: [Vietnix](https://blog.vietnix.vn/ky-thuat-chong-ddos-phan-2-phan-tich-ip-header.html)

---
