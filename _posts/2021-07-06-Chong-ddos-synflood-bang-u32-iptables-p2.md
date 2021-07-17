---
layout: post
comments: true
title:  "DDOS - Chống DDoS SYN Flood bằng module u32 iptables (phần 2)"
title2:  "Chống DDoS SYN Flood bằng module u32 iptables (phần 2)"
date:   2021-07-06 21:45:00
permalink: 2021/07/06/Chong-ddos-synflood-bang-u32-iptables-p2
mathjax: false
tags: Security DDOS Linux iptables synflood u32
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Chong-ddos-synflood-bang-u32-iptables-p2/Hinh0.jpg
summary: Hướng dẫn chống ddos trên Linux
---

Ở bài Hướng dẫn chống DDoS SYN FLood bằng module u32 iptables Phần 1 , chúng ta đã nắm được cách sử dụng module u32, bây giờ sẽ tiến hành phân tích cuộc tấn công và áp dụng vào thực tế.

Xem lại: [Chống DDoS SYN Flood bằng module u32 iptables (phần 1)](/2021/07/05/Chong-ddos-synflood-bang-u32-iptables-p1)

## **1. Giả lập lại cuộc tấn công**

Trước tiên, mình giả lập lại cuộc tấn công để có traffic, tiện cho việc kiểm tra lại xem các rule mình viết đã hoạt động đúng yêu cầu hay chưa.

Nhắc lại 1 vài đặc điểm của cuộc tấn công:
- Kiểu tấn công: SYN Flood
- Source IP: giả mạo và Random
- Đặc biệt, các gói SYN có mang theo Payload từ 4-22 bytes
- Tần số tấn công: > 1 triệu packet/s

Với các đặc điểm trên, mình sử dụng công cụ ***hping3*** để giả lập lại traffic tấn công, dựng mô hình LAB ở local sử dụng card Host Only với topo  như sau:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p2/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
- Máy tạo traffic tấn công (attacker – nguồn tấn công): **192.168.56.1**
- Server: **192.168.56.20**

Thử nghiệm tấn công:
- Trên máy attacker, chạy lệnh:
```
$ hping3 --flood --rand-source -S -p 80 --data 4 192.168.56.20
```

- Trên server, kiểm tra traffic bằng vnstat, thấy server đang nhận vào 200,000 packet/s. Traffic tấn công đã gửi thành công đến mục tiêu: 
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p2/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
- Kểm tra các gói tin xem có giống với đặc điểm của cuộc tấn công hay chưa:
```
$ tcpdump -i eth0 -nnn -vvv -X -c 10 -P in port 80
```
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p2/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
=> Các điều kiện đã được đáp ứng, bắt tay vào xử lý.

Xem lại cách capture và phân tích gói tin: [Hướng dẫn chống ddos trên Linux (phần 2) - IP Header](/2021/07/02/Huong-dan-chong-ddos-tren-linux-p2)

## **2. Phân tích**

Bắt 1000 gói tin để phân tích source IP xem đây là kiểu tấn công SYN FLood bằng Botnet hay bằng tool Fake Random IP:
```
$ tcpdump -i eth0 -nnn -c 1000 -P in port 80 | awk '{print $3}' | cut -d"." -f-4 | sort | uniq -c | sort -rn | more
```
- Địa chỉ Source có prefix trải đều một cách tuần tự (93.x.x.x, 94.x.x.x, 95.x.x.x, 96.x.x.x, 97.x.x.x, 98.x.x.x, 99.x.x.x …)
- Mỗi IP gửi gói tin đến server với tần số thấp
- Cấu trúc gói tin gần như là giống nhau

Từ các dữ kiện trên, có thể xác định đây là kiểu tấn công Random Source IP với các gói tin được tạo ra từ cùng một nguồn.

Với đặc điểm trên:
- Số lượng Source IP là **vô hạn**, vì tool có thể tạo gói tin SYN với source IP bất kỳ => Việc chặn IP sẽ không giải quyết được vấn đề.
- Tần số từ mỗi IP sẽ thấp => Đặt rate limiting cũng sẽ không có tác dụng
- Dấu hiệu bất thường là các gói tin SYN có mang theo Payload, ta sẽ dựa vào đây và viết rule DROP bỏ các gói tin có đặc điểm này.

Kết luận: điều kiện để các tin bị xem là không hợp lệ là gói tin mang **tất cả** các đặc điểm sau:
- Gói tin sử dụng giao thức TCP
- Gói tin có cờ SYN được bật
- Gói tin có mang theo payload

## **3. Tiến hành viết rule sử dụng module u32**

### **3.1. Kiểm tra gói tin có phải giao thức là TCP hay không**

Để kiểm tra gói tin có phải là TCP hay không, ta cần kiểm tra trường ***Protocol*** trong IP Header (byte thứ 9), nếu giá trị của trường này bằng 6 thì là TCP:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p2/Hinh4.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
- Start Value: 9 -3 = 6
- Mask: 0xFF
- Value: 6
- Biểu thức cuối cùng: 6&0xFF=6

### **3.2. Kiểm tra có phải gói SYN hay không**

Để kiểm tra xem có phải là gói SYN hay không, cần kiểm tra cờ SYN trong TCP mang giá trị 0 (cờ SYN không bật) hay 1 (cờ SYN được bật)
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p2/Hinh4.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Tương tự "Ví dụ 3" trong [Hướng dẫn chống DDoS SYN Flood bằng module u32 iptables (Phần 1)](/2021/07/05/Chong-ddos-synflood-bang-u32-iptables-p1):
- Cờ SYN trong byte thứ 13 của TCP header. Muốn lấy được byte này, việc đầu tiên ta cần làm là nhảy qua khỏi chiều dài của IP Header và đến byte đầu tiên của TCP Header, tiếp theo chọn byte thứ 10 (13-3=10) từ vị trí vừa nhảy tới. Thực hiện việc này bằng cách sử dụng toán tử “@“
- 0>>22&0x3C@10
- Trong byte 13, cờ SYN nằm ở bit thứ 2 từ phải sang, do đó dịch sang phải 1 bit để đưa bit của cờ SYN vào đúng bị trí right most
- 0>>22&0x3C@10>>1
- zero out tất cả những giá trị ta không qua tâm, chính là 31 bit nằm phía trước. Sử dụng Mask:
- 0000 0000 \| 0000 0000 \| 0000 0000 \| 0000 0001
- Tương đương: 0x00000001
- Rút gọn: 0x01
- Kiểm tra xem giá trị có bằng 1 hay không (cờ SYN đang được bật)
- 0>>22&0x3C@10>>1&0x1=0x1

Trong đó:
- 0>>22&0x3C@ : Tính chiều dài của IP Header và nhảy đến vị trí byte vừa tính toán
- 10>>1&0x1=0x1 : Lấy byte số 10-13, dịch qua phải 1 bit để đưa cờ SYN vào vị trí Right Most, loại bỏ 31 bit phía trước và kiểm tra giá trị cuối cùng có bằng 1 hay không.

### **3.3. Kiểm tra gói tin có Payload hay không**

Để kiểm tra gói tin có Payload hay không, ta cần “nhảy” qua khỏi IP Header, TCP Header và đến byte đầu tiên của phần Data, kiểm tra xem có mang giá trị hay không
- Biểu thức: 0>>22&0x3C@12>>26&0x3C@0>>24=0:255

Trong đó:
- 0>>22&0x3C@: Tính toán chiều dài của IP Header, nhảy đến vị trí của byte vừa tính toán (chính là vị trí bắt đầu của TCP Header)
- 12>>26&0x3C@: Tính toán chiểu dài của TCP Header và nhảy đến vị trí của byte vừa tính toán (chính là vị trí bắt đầu của Data)
- 0>>24=0:255: Từ vị trí đầu tiên của Data, lấy 4 byte bắt đầu từ vị trí số 0, dịch qua phải 24 bit để đưa byte đầu tiên vào vị trí right most, kiểm tra giá trị của byte này có nằm trong khoảng từ 0:255 hay không.

### **3.4. Kết quả cuối cùng**

Kết hợp các điều kiện lại sử dụng toán tử "**&&**", ta có rule như sau:
```
$ iptables -I INPUT -m u32 --u32 "6&0xFF=6 && 0>>22&0x3C@10&0x02=2 && 0>>22&0x3C@12>>26&0x3C@0>>24=0:255" -j DROP
```

## **4. Tổng kết**

Có nhiều cách để chặn gói SYN mang payload, hôm nay chúng ta đã ứng dụng thành công u32 để chặn kiểu tấn công này. Tuy nhiên, trong quá trình làm việc với u32, mình phát hiện u32 vẫn có 1 hạn chế là không thể chặn được gói tin SYN có payload nếu payload < 4 byte. Trong bài tiếp theo, mình sẽ hướng dẫn các bạn cách khác để chặn SYN Flood có payload < 4 byte.

Video chi tiết về lý thuyết, thực hành cũng như kết quả của việc dùng u32 chống DDoS SYN Flood Fake Random Source IP các bạn có thể xem video bên dưới.
<iframe width="560" height="315" src="https://www.youtube.com/embed/zosEUSetAAY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Nguồn: [Vietnix](https://blog.vietnix.vn/huong-dan-chong-ddos-syn-flood-bang-module-u32-iptables-p2.html)

---
