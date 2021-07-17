---
layout: post
comments: true
title:  "DDOS - Chống DDoS SYN Flood bằng module u32 iptables (phần 1)"
title2:  "Chống DDoS SYN Flood bằng module u32 iptables (phần 1)"
date:   2021-07-05 21:45:00
permalink: 2021/07/05/Chong-ddos-synflood-bang-u32-iptables-p1
mathjax: false
tags: Security DDOS Linux iptables synflood u32
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Chong-ddos-synflood-bang-u32-iptables-p1/Hinh0.jpg
summary: Hướng dẫn chống ddos trên Linux
---

## **1. Tình huống thực tế**

Thời gian gần đây, hệ thống bên mình liên tục nhận được các cuộc tấn công SYN Flood Spoofed Source IP (Fake – giả mạo địa chỉ IP) với tần số hơn 1 triệu SYN packet mỗi giây. Điểm đặc biệt trong đợt tấn công lần này các gói SYN có “Total Length” không cố định, mang theo Payload Random. Việc gói SYN mang theo Payload là dấu hiệu bất thường, có thể dùng làm chữ ký để nhận diện, tuy nhiên việc Length không cố định gây khá nhiều khó khăn trong việc viết rule. Lúc này, mình quyết định sử dụng module u32 để giải quyết vấn đề trên.

## **2. Giới thiệu về module u32**

### **2.1. Chức năng**

- u32 là module cho phép chúng ta có thể lấy ra byte/bit nào ở bất kỳ vị trí nào trong gói tin để tính toán và kiểm tra giá trị. Sở dĩ có tên là u32 vì mỗi lần lấy data, u32 sẽ lấy 4 byte, tương đương 32bit. Cú pháp của u32 như sau:

Start&Mask=Value\|RangeValue

Điều này có nghĩa:
- Lấy 4 byte dữ liệu từ vị trí “Start”, thực hiện phép toán “AND” với một bit “Mask” (một dãy bit) và so sánh giá trị của kết quả vừa tính toán có bằng với “Value” hoặc nằm trong “RangeValue” hay không.
- Nếu bằng thì rule match, ngược lại rule được xem là không match.
- Nếu vị trí “Start” nằm cuối hoặc gần cuối gói tin khiến cho việc lấy 4 byte tiếp theo vượt ra khỏi phạm vi gói tin thì mặc định kết quả sẽ là False.

### **2.2. Các toán tử hỗ trợ**

- &   	: phép toán AND hai dãy bit với nhau
- \>>   	: Shift Left (dịch bit sang trái)
- @		: jump (Nhảy đến vị trí của kết quả vừa tính toán nằm phía trước dấu @)
- &&  	: Combining tests - AND nhiều điều kiện test lại với nhau

Với các phiên bản cũ, u32 hỗ trợ phép toán "-" để lùi ngược về vị trí byte trước đó (negative offset), tuy nhiên các phiên bản sau này không còn hỗ trợ phép toán này nữa. 

## **3. Cách sử dụng**

### **3.1. Xác định vị trí Start**

Do đặc điểm mỗi lần lấy dữ liệu, u32 sẽ lấy 4 byte tính từ vị trí “Start” nên vị trí “Start” của chúng ta sẽ bằng vị trí cần lấy trừ đi 3.

**Ví dụ:**
- Bạn muốn lấy giá trị của byte 13, vị trí "**Start**" sẽ là: 13-3=10. Lúc này, tính từ vị trí "**Start**" (byte 10), u32 sẽ lấy 4 byte bao gồm: byte 10 + byte 11 + byte 12 + byte 13 (byte chúng ta cần lấy)
- Nếu bạn muốn lấy các byte ở vị trí đầu của gói tin như byte 1, byte 2 thì kết quả trừ đi 3 sẽ ra số âm, đây là vị trí không hợp lệ và chúng ta sẽ "**Start**" ở vị trí 0 và kết hợp với dịch bit (>>) để tính toán giá trị.

### **3.2. Tính toán giá trị**

- Từ vị trí "**Start**", u32 sẽ lấy 4 byte và trả về cho chúng ta kết quả là một dãy bit gồm 32bit, chúng ta cần điều chỉnh (Mask/Shift) 32bit này và chuyển đối giá trị cuối cùng qua dạng thập phân hoặc hex để so sánh với "**Value**".
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p1/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
**Ví dụ 1**: Giá trị cần giữ lại kiểm tra có chiều dài 2 byte: kiểm tra xem giá trị của trường IPID có nằm trong khoảng từ 2 đến 256:
- Chọn vị trí Start: IPID nằm ở byte 4 và byte 5 của IP Header, vậy nên vị trí start của chúng ta sẽ là 5-3=2
- Lúc này kết quả trả về sẽ là 4 byte, bao gồm byte 2 + byte 3 + byte 4 + byte 5, trong đó ta chỉ quan tâm byte 4 + byte 5, các byte còn lại ta cần “zero out” để không còn ảnh hưởng đến giá trị tính toán cuối cùng. 
- Để "zero out" các giá trị không mong muốn, ta thực hiện phép toán AND các vị trí đó với 0 (vì bit 0 hay bit 1 khi AND với 0 đều bằng 0), các vị trí cần giữ lại sẽ AND với 1 (vì nếu giá trị của bit hiện tại là 1 thì AND với 1 sẽ là 1, giá trị của bit hiện tại là 0 khi AND với 1 cũng vẫn là 0)
- Lúc này, “Mask” chúng ta sẽ sử dụng: 
- 0000 0000 \| 0000 0000 \| 1111 1111 \| 1111 1111
- Tương ứng dạng hex: 0x0000FFFF
- Viết gọn lại: 0xFFFF
- Zero out 2 byte đầu trong 4 byte kết quả lấy được, giữ nguyên gía trị của 2 byte cuối (trường IPID)
- Kết quả cuối cùng chúng ta có: 
- iptables -m u32 --u32 "2&0xFFFF=0x02:0x100"
- Áp dụng tương tự với trường hợp byte cần lấy có chiều dài 1 byte, 3 byte hay 4 byte.

**Ví dụ 2**: byte cần lấy nằm ở đầu header: lấy giá trị của trường IP Header Length, trường này gồm 4 bit cuối của byte đầu tiên (byte số 0)
- Do vị trí byte cần lấy là byte 0, nếu trừ 3 sẽ ra số âm nên vị trí Start sẽ là 0
- u32 sẽ lấy 4 byte từ vị trí bắt đầu bao gồm byte 0 + byte 1 + byte 2 + byte 3
- Kết quả trả về 1 là dãy gồm 32 bit, trong đó có chứa 4 bit ta cần quan tâm nằm ở byte số 0.
- Giả sử dãy bit được trả về có giá trị như sau: 
- 0100 **0101** \| 0000 1101 \| 0010 1111 \| 0010 1101
- Để tính toán, ta cần đưa 4 bit mình quan tâm về vị trí sát góc bên phải (right most) bằng phép toán dịch bit. Ta cần dịch qua bên phải 3 byte (24 bit): 0>>24
- Kết quả: 
- 0000 0000 \| 0000 0000 \| 0000 0000 \| 0100 **0101**
- Tuy nhiên, theo RFC của TCP/IP, muốn tính chiều dài của IP Header, ta cần lấy giá trị này nhân với 4, tương đương với việc dịch giá trị hiện tại qua trái 2 bit. Vậy nên thay vì dịch 24 bit qua phải rồi sau đó dịch lại 2 bit qua trái, ta chỉ cần dịch giá trị ban đầu 22 bit qua phải.
- Kết quả: 0000 0000 \| 0000 0000 \| 0000 0010 \| 00**01** **01**00
- Bước tiếp theo, "zero out" các byte ta không quan tâm, sử dụng Mask: 
- 0000 0000 \| 0000 0000 \| 0000 0000 \| 00**11** **11**00
- Tương đương: 0x0000003C
- Rút gọn: 0x3C
- Kết quả cuối cùng nhận được dãy bit có giá trị: 
- 0000 0000 \| 0000 0000 \| 0000 0010 \| 00**01** **01**00
- Quy đổi ra thập phân = 20
- Đây chính là chiều dài của IP Header.
- Biểu thức cuối cùng: 
- iptables -m u32 --u32 "0>>22&0x3C=20"
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-synflood-bang-u32-iptables-p1/Hinh2.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
**Ví dụ 3**: Giá trị cần kiểm tra là 1 bit: Kiểm tra giá trị của cờ SYN có được bật hay không.
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
- Biểu thức cuối cùng
- iptables -m u32 --u32 "0>>22&0x3C@10>>1&0x1=0x1"

## **4. Tổng kết**

Đến đây, ta đã xong phần lý thuyết của u32, tiếp theo sẽ đến phần thực hành áp dụng vào cuộc tấn công SYN Flood Spoofed Source IP với gói SYN có payload.

Với cơ chế hoạt động linh hoạt, cho phép ta lấy bất kỳ byte/bit dữ liệu nào trong gói tin ra để kiểm tra, u32 giúp ta linh hoạt trong việc viết rule, có thể kiểm tra những gía trị mà các module hiện tại của iptables không hỗ trợ.

Nguồn: [Vietnix](https://blog.vietnix.vn/huong-dan-chong-ddos-syn-flood-bang-module-u32-iptabes-p1.html)

---
