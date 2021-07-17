---
layout: post
comments: true
title:  "DDOS - Hướng dẫn chống ddos trên Linux (phần 3) - iptables connlimit"
title2:  "Hướng dẫn chống ddos trên Linux (phần 3) - iptables connlimit"
date:   2021-07-03 21:45:00
permalink: 2021/07/03/Huong-dan-chong-ddos-tren-linux-p3
mathjax: false
tags: Security DDOS Linux
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh0.jpg
summary: Hướng dẫn chống ddos trên Linux
---

**iptables connlimit - Giới hạn số lượng kết nối**

Module **connlimit** cho phép quản lí số lượng kết nối đồng thời (concurrent connections) đến server hoặc rời đi từ server.

Để xem các options có thể sử dụng:

```
# iptables -m connlimit --help
...
connlimit match options:
  --connlimit-upto n     match if the number of existing connections is 0..n
  --connlimit-above n    match if the number of existing connections is >n
  --connlimit-mask n     group hosts using prefix length (default: max len)
  --connlimit-saddr      select source address for grouping
  --connlimit-daddr      select destination addresses for grouping
```

## **I. Mô hình demo**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh1.jpg" width = "800">
</div>
<div class="thecap">Hình 1: Mô hình Demo</div>
</div>
<hr>
- Server: có IP 10.10.10.100
- client1: có IP 10.10.10.200
- client2: có IP 10.10.10.250

Để demo, tại Server tôi sử dụng socat để tạo listening socket đóng vai trò như một ứng dụng chấp nhận kết nối trên port 2001:

```socat TCP-LISTEN:2001,bind=10.10.10.100,fork,reuseaddr SYSTEM:'while true; do echo port 2001 - connected...; sleep 1 ; done'```

Từ client sử dụng nc (netcat) hoặc telnet kết nối đến port 2001:

```telnet 10.10.10.100 2001```

Kết quả khi client có thể mở kết nối đến server:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh2.jpg" width = "800">
</div>
<div class="thecap">Hình 2</div>
</div>
<hr>

Sau đó, có thể Ctrl + c để ngắt kết nối, hoàn tất việc chuẩn bị.

## **II. Giới hạn số lượng kết nối đồng thời**

Trên Server, mở 3 ports 2001-2003:

socat TCP-LISTEN:2001,bind=10.10.10.100,fork,reuseaddr SYSTEM:'while true; do echo port 2001 - connected...; sleep 1 ; done' &
socat TCP-LISTEN:2002,bind=10.10.10.100,fork,reuseaddr SYSTEM:'while true; do echo port 2002 - connected...; sleep 1 ; done' &
socat TCP-LISTEN:2003,bind=10.10.10.100,fork,reuseaddr SYSTEM:'while true; do echo port 2003 - connected...; sleep 1 ; done' &

Server đã sẵn sàng kết nối trên 3 ports:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh3.jpg" width = "800">
</div>
<div class="thecap">Hình 3</div>
</div>
<hr>
Giả sử tôi phải thực hiện các yêu cầu sau:
- Yêu cầu 1: Port 2001, mỗi client được phép mở tối đa 1 kết nối cùng lúc
- Yêu cầu 2: Port 2002, mỗi client được phép mở tối đa 2 kết nối cùng lúc
- Yêu cầu 3: Port 2003, tại một thời điểm server chấp nhận 3 kết nối từ clients

Tôi bắt đầu phần tích và viết iptabes rule cho từng yêu cầu một.

**Yêu cầu 1: Port 2001, mỗi client được phép mở tối đa 1 kết nối cùng lúc, ta cần phân tích:**

Xét những gói tin mở kết nối tcp đến server: -t raw PREROUTING -d 10.10.10.100/32 -p tcp –syn
Xét trên port 2001: –dport 2001
Xét những gói tin đến Server, mỗi source IP có tối đa 1 kết nối: -m connlimit –connlimit-above 1 –connlimit-mask 32 –connlimit-saddr -j DROP
Chạy lệnh trên Server:

```
iptables -t raw -A PREROUTING -d 10.10.10.100/32 -p tcp --syn --dport 2001 -m connlimit --connlimit-above 1 --connlimit-mask 32 --connlimit-saddr -j DROP
```

Kiểm chứng:
Client1 chỉ có thể mở một kết nối, kết nối thứ hai không thành công:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh4.jpg" width = "800">
</div>
<div class="thecap">Hình 4</div>
</div>
<hr>
Client2, tương tự chỉ có thể mở một kết nối:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh5.jpg" width = "800">
</div>
<div class="thecap">Hình 5</div>
</div>
<hr>
Giới hạn 1 kết nối chỉ áp dụng với port 2001 và không ảnh hưởng các port khác. Ví dụ client2 có thể mở thêm kết nối đến port 2002:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh6.jpg" width = "800">
</div>
<div class="thecap">Hình 6</div>
</div>
<hr>

**Yêu cầu 2: Port 2002, mỗi client được phép mở tối đa 2 kết nối cùng lúc**

Xét những gói tin mở kết nối tcp đến server: -t raw PREROUTING -d 10.10.10.100/32 -p tcp –syn
Chỉ áp dụng đối với port 2002: –dport 2001
Xét những gói tin đến Server, mỗi source IP có tối đa 1 kết nối: -m connlimit –connlimit-above 2 –connlimit-mask 32 –connlimit-saddr -j DROP
Chạy lệnh trên Server:

```
iptables -t raw -A PREROUTING -d 10.10.10.100/32 -p tcp --syn --dport 2002 -m connlimit --connlimit-above 2 --connlimit-mask 32 --connlimit-saddr -j DROP
```

Kiểm chứng:
Một client chỉ được mở tối đa 2 kết nối cùng lúc:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh7.jpg" width = "800">
</div>
<div class="thecap">Hình 7</div>
</div>
<hr>
Giới hạn 2 kết nối chỉ áp dụng với port 2002.

**Yêu cầu 3: Port 2003, tại một thời điểm server chấp nhận 3 kết nối từ clients**

Chỉ áp dụng đối với port 2003: –dport 2001
Xét những gói tin đến Server, server chỉ chấp nhận 2 kết nối từ client bất kì, vì thế tôi cần giới hạn trên destination IP sử dụng –connlimit-daddr mode : -m connlimit –connlimit-above 3 –connlimit-mask 32 –connlimit-daddr -j DROP

Chạy lệnh trên Server:

```
iptables -t raw -A PREROUTING -d 10.10.10.100/32 -p tcp --syn --dport 2003 -m connlimit --connlimit-above 3 --connlimit-mask 32 --connlimit-daddr -j DROP
```

Kiểm chứng:
Client1 mở 2 kết nối thành công, Client2 mở thêm kết nối thứ thứ 3 thành công. Sau đó, Server bắt đầu từ chối kết nối thứ tư:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p3/Hinh8.jpg" width = "800">
</div>
<div class="thecap">Hình 8</div>
</div>
<hr>

## **III Ứng dụng khác**

Vì lí do độ dài, một số lưu ý và kinh nghiệm khác không được đề cập. Đọc giả có thể tham khảo thêm và vận dụng:

Tôi chỉ demo giới hạn các gói tin đến Server, trường hợp khác đọc giả vẫn có thể sử dụng module connlimit để giới hạn kết nối chiều OUT từ Server ra ngoài.
Tôi dùng –connlimit-above và target -j DROP, đọc giả có thể sử dụng –connlimit-upto tùy vào bối cảnh. Ví dụ, yêu cầu số 2 tôi viết lại với policy DROP:

```
iptables -t raw -A PREROUTING -d 10.10.10.100/32 -p tcp --syn --dport 2002 -m connlimit --connlimit-upto 2 --connlimit-mask 32 --connlimit-saddr -j ACCEPT
iptables -t raw -A PREROUTING -d 10.10.10.100/32 -p tcp --dport 2002 -j DROP
```

Mặc định, option –connlimit-mask 32 sẽ áp dụng giới hạn cho một IP, bạn có thể áp dụng giới hạn cho cả một network. Ví dụ, yêu cầu số 1 được viết lại:

```
iptables -t raw -A PREROUTING -d 10.10.10.100/32 -p tcp --syn --dport 2001 -m connlimit --connlimit-above 1 --connlimit-mask 24 --connlimit-daddr -j DROP
```

Khi IP bất kì 192.168.1.5 kết nối đến Server, giới hạn sẽ apply cho cả network 192.168.1.0/24, các IP trong range này sẽ không thể mở kết nối thứ hai.

Thông qua các ví dụ, hi vọng đọc giả cảm thấy thoải mái hơn khi ứng dụng của module connlimit để đáp ứng các nhu cầu riêng trong quản trị.

Nguồn: [Vietnix](https://blog.vietnix.vn/iptables-connlimit-gioi-han-so-luong-ket-noi.html)

---
