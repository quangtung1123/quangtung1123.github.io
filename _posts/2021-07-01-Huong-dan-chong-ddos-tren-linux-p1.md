---
layout: post
comments: true
title:  "DDOS - Hướng dẫn chống ddos trên Linux (phần 1) - Nhập môn"
title2:  "Hướng dẫn chống ddos trên Linux (phần 1) - Nhập môn"
date:   2021-07-01 21:45:00
permalink: 2021/07/01/Huong-dan-chong-ddos-tren-linux-p1
mathjax: false
tags: Security DDOS Linux
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Huong-dan-chong-ddos-tren-linux-p1/Hinh1.jpg
summary: Hướng dẫn chống ddos trên Linux
---

## **Tại sao tôi viết series này?**

Ở môi trường làm việc, chúng tôi – System Administrators – đã có kinh nghiệm nhiều năm quản trị HĐH Linux và chống DDoS hàng trăm trận với quy mô khác nhau. Tôi muốn chia sẻ kinh nghiệm với bạn đọc và cũng đáp ứng nhu cầu xây dựng document cho những đồng nghiệp tương lai, giúp họ nhanh chóng tiếp cận những kỹ năng thiết yếu thúc đẩy công việc quản trị đạt hiệu quả tại tổ chức. Hy vọng series Hướng dẫn chống DDoS sẽ mang lại giá trị thiết thực cho các bạn.

## **Series “Hướng dẫn chống DDoS trên Linux” sẽ giúp bạn:**

Trang bị những kiến thức nền tảng và kỹ năng/công cụ cần thiết để phân tích – nhận dạng – phòng thủ trước tấn công DoS/DDoS.

Bạn sẽ tự mình build được bộ firewall rules cho application. Không cầu toàn trong “sự nghiệp” chống DDoS, không khái niệm “one-size-fits-all“, không tồn tại một bộ core-rule-set bảo vệ tất cả application.
Tùy thuộc vào đối tượng dịch vụ bạn đang quản trị, những kiến thức và kỹ năng hữu ích là chìa khóa giúp bạn build được firewall cho nhu cầu riêng mình.

Tham khảo:

- [Phân tích IP Header để chống tấn công DDoS](/2021/07/01/Huong-dan-chong-ddos-tren-linux-p1)
- [Hướng dẫn chống SYN Flood bằng iptables u32 module (Phần 1)](/2021/07/01/Huong-dan-chong-ddos-tren-linux-p1)
- [Hướng dẫn chống SYN Flood bằng iptables u32 module (Phần 2)](/2021/07/01/Huong-dan-chong-ddos-tren-linux-p1)


## **Tại sao chúng tôi chọn iptables/ipset ?**

Theo chiều dài series, tôi và bạn đặt mình trong tình huống quản trị một server Linux, với nhu cầu chống tấn công DDoS quy mô trong phạm vi phần cứng server có thể chịu được (NICs, bandwidth, CPU, interrupts,…).

Series chỉ tập trung công cụ có sẵn iptables/ipset của Netfilter (Linux kernel). iptables/ipset là công cụ mạnh mẽ, gọn nhẹ, có sẵn, linh hoạt và hiệu suất tuyệt đối để build firewall.

Vì thế, bài viết chỉ tập trung xoay quanh packet filter ở layer 3 (tầng network). Các công cụ và khái niệm khác như IDS, IPS, HA, LB,… và cả WAF nằm ngoài phạm vi bài viết này.

## **Đọc series ” Hướng dẫn chống DDoS trên Linux” như thế nào?**

Giá trị của bài viết đánh giá bởi sự đồng hành của các đọc giả, chúng tôi khuyến khích bạn đọc trang bị kỹ những kiến thức nền tảng được nhắc tới, và cảm phiền bỏ qua nếu chúng tôi lặp lại những kiến thức bạn đã biết.

Series sẽ được chia thành 5 **PHẦN** với vai trò cụ thể từng PHẦN:

| **Phần**	| **Vai trò** |
|-----------|-------------|
| **CÔNG CỤ** |	**Nội dung**: giới thiệu công cụ(tools) sẽ sử dụng xuyên suốt series: tcpdump, ss/netstat, vnstat, iftop, top, socat, ngrep, /proc/interrupts,… Phạm vi dừng lại ở những tutorials phổ dụng trong thực tế. Đọc giả cần tham khảo nguồn khác hoặc manual page, thông thạo network tools sẽ giúp ích phân tích network. |
| **KIẾN THỨC NỀN TẢNG** |	Trang bị kiến thức nền về giao thức (tcp/ip) trong mô hình OSI, và bất cứ kiến thức bạn cần để đồng hành cùng series. **Mục tiêu của tôi**: luôn đi kèm demo để giúp bạn thấy tầm quan trọng của những kiến thức nền tảng trong phân tích tấn công. |
| **IPTABLES MODULES** |	**Nội dung**: Giới thiệu từng iptables modules(phần) riêng lẻ: iptables flow chains/TRACE target, conntrack/state machine, connlimit, hashlimit, recent, string, u32, bpf, synproxy/CT, set/match-set, mark,… **Vai trò của tôi**: Tập trung cách hoạt động, ứng dụng, làm demo để mô phỏng ứng dụng của module. **Mục tiêu của bạn đọc**: thực hành để cảm nhận thực tiễn. Kết thúc PHẦN này bạn phải có khả năng: đứng trước một nhu cầu chặn/limit có thể chọn được module phù hợp để sử dụng. |
| **CASE STUDY** |	Tập trung những write-up khi xử lý chống tấn công DDoS thực tế. **Nội dung**: sử dụng kinh nghiệm bản thân để phân tích cuộc tấn công, get signature, viết firewall rules, (test rules), ...|
| **NÂNG CAO** |	Những bài viết/lab/benchmark nâng cao nếu có thể: – Hardware/Kernel tuning, – Kernel bypass/offload, ... |

## **Lời kết:**

System Admin không thể viết firewall rule chống DDoS nếu không thể phân tích cuộc tấn công. Sử dụng thông thạo **CÔNG CỤ** và vững **KIẾN THỨC NỀN TẢNG** là yêu cầu bắt buộc.

Avoid-a-boring-life, trong series chúng tôi chỉ tập trung nhiều vào tấn công giao thức TCP. Những kỹ năng bạn được trang bị sẽ dễ dàng mở rộng sang bất kì giao thức khác.

**Get-your-hands-dirty**

Mục tiêu cuối cùng của bạn đọc là KỸ NĂNG, được hình thành bởi rèn luyện **IPTABLES MODULES** và trau dồi ở **CASE STUDY**. Thói quen monitor/quan sát traffic vào ra server giúp bạn có khả năng phân biệt giữa yếu tố bình thường/bất thường trong network.

Nguồn: [Vietnix](https://blog.vietnix.vn/huong-dan-chong-ddos-phan-1.html)

---
