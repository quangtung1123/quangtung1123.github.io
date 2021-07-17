---
layout: post
comments: true
title:  "DDOS - Hướng dẫn chống ddos trên Linux (phần 4) - iptables connlimit để lọt traffic"
title2:  "Hướng dẫn chống ddos trên Linux (phần 4) - iptables connlimit để lọt traffic"
date:   2021-07-04 21:45:00
permalink: 2021/07/04/Huong-dan-chong-ddos-tren-linux-p4
mathjax: false
tags: Security DDOS Linux
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Huong-dan-chong-ddos-tren-linux-p4/Hinh0.jpg
summary: Hướng dẫn chống ddos trên Linux
---

**LỖI THÚ VỊ KHI CHỐNG DDOS BẰNG IPTABLES**

Trong quá trình phát triển Proxy chống DDoS bằng iptables, chúng tôi vô tình phát hiện lỗi thú vị liên quan đến 2 module hashlimit và [connlimit](/2021/07/03/Huong-dan-chong-ddos-tren-linux-p3) khiến cho traffic tấn công DDoS bị lọt qua khỏi firewall. Chúng tôi vẫn chưa xác định được nguyên nhân lỗi chính xác do kernel hay do iptables version vì search không thấy thông tin nào liên quan đến lỗi tính tới thời điểm hiện tại. Chúng tôi thấy mình có trách nhiệm chia sẻ thông tin lỗi khi chống DDoS bằng iptables này với các bạn, hãy đọc và kiểm tra xem hệ thống của bạn có đang gặp tình trạng này không nhé!

## **IPTABLES CONNLIMIT ĐỂ LỌT TRAFFIC TẤN CÔNG DDOS**

Module connlimit của iptables có tác dụng giới hạn số lượng kết nối đồng thời (concurrent connection) từ một địa chỉ IP đến server (connection ở trạng thái ESTABLISHED). Đây cũng là một trong những module chủ chốt sử dụng trong quá trình chống DDoS.

Connlimit hoạt động rất chính xác theo mục đích thiết kế, tuy nhiên, trường hợp sau lại là một ngoại lệ:
- Firewall có 1 rule connlimit đang hoạt động từ trước và đang hoạt động rất chính xác.
- 1 thời gian sau, ta thêm 1 rule connlimit khác vào firewall (khác hoàn toàn so với rule cũ), lúc này, bộ counter của các rule có sẵn sẽ bị reset và đếm lại từ đầu, dẫn đến việc client mở được nhiều kết nối hơn quy định ở rule cũ.

Trong đa số trường hợp, lỗi trên cũng không ảnh hưởng nhiều, nhưng với mô hình thu thập, phân tích và chống tấn công tự động như chúng tôi đang dùng thì lại ảnh hưởng nặng nề. Hệ thống chống DDoS chúng tôi hoạt động dựa trên cơ chế Dynamic Learning, hệ thống sensor sẽ thu thập traffic và generate rule chống DDoS tự động sau đó apply trực tiếp vào firewall (không xóa các rule cũ). Do đó, mỗi khi hệ thống chèn thêm 1 rule mới liên quan connlimit thì tất cả các rule connlimit hiện có lại cho phép user mở thêm kết nối mới dù user đó đang đạt số lượng kết nối tối đa mà firewall cho phép.

Cụ thể, đây là trường hợp iptables connlimit hoạt động bình thường:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p4/Hinh1.png" width = "800">
</div>
<div class="thecap">Lỗi khi chống ddos bằng iptables connlimit</div>
</div>
<hr>
Giải thích:
- Hình 1: Ta giới hạn mỗi IP (connlimit-saddr + connlimit-mask /32) chỉ được phép mở 1 kết nối SSH (tcp port 22) đến server, nếu có nhiều hơn 1 kết nối thì sẽ DROP bỏ.
- Hình 2: Sau khi apply, ta có thể SSH đến server bình thường.
- Hình 3: Tiếp tục mở thêm 1 kết nối SSH nữa đến server (sau khi đã SSH thành công ở trên), kết quả là bị chặn, không thể SSH được

Sau đó, ta tiếp tục chèn thêm 1 rule mới khác hoàn toàn so với rule cũ nhưng có sử dụng module iptables connlimit:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p4/Hinh2.png" width = "800">
</div>
<div class="thecap">Lỗi khi chống ddos bằng iptables connlimit</div>
</div>
<hr>
Giải thích:
- Hình 1: Chèn rule mới để giới hạn kết nối lên port 9999, giới hạn là 10 kết nối và subnet /24, action là ACCEPT (mục đích để khác với rule giới hạn SSH cũ)
- Hình 2: Kết nối cũ ở bước trên vẫn được giữ.
- Hình 3: Sau khi apply rule mới, ta lại có thể mở thêm kết nối SSH thành công tới server, dù limit đang áp dụng chỉ cho phép mở 1 kết nối SSH mà thôi!

## **IPTABLES HASHLIMIT ĐỂ LỌT TRAFFIC TẤN CÔNG DDOS**

Module hashlimit của iptables có tác dụng giới hạn tần số các gói tin một cách linh hoạt, điểm khác biệt giữa module hashlimit và module limit là hashlimit cho phép áp giới hạn tần số theo từng source IP hoặc subnet, trong khi module limit không giới hạn được theo từng địa chỉ source IP. Hashlimit là một module rất hữu dụng và được sử dụng rất nhiều trong kỹ thuật chống DDoS.

Khi sử dụng module hashlimit, nếu name khai báo ở phần *"–hashlimit-name"* dài hơn 15 ký tự thì khi gõ lệnh vẫn thành công bình thường, tuy nhiên, hashtable name lưu trong kernel sẽ bị **cắt lại chỉ còn 15 ký tự**. Điều này khiến cho hoạt động của hashlimit bị **sai lệch** hoàn toàn:
- Các traffic đi vào, dù thỏa các điều kiện của rule nhưng entry của chúng không được update vào hashtable.
- Các counter của hashtable (với name đã bị cắt chỉ còn 15 ký tự) không được update.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-chong-ddos-tren-linux-p4/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>
Ta khai báo name của hashtable là *"name_dai_hon_15_ky_tu"* nhưng khi check trong hệ thống, hashtable được tạo ra chỉ có name là *"name_dai_hon_15"*

## **LỖI DO ĐÂU?**

Lỗi của module hashlimit, chúng tôi đã xác định được nguyên nhân là do kernel version: "Bug kinh điển của module hashlimit ở kernel centos bản 2018 2019 trở về trước. 3.10.0-957.27.2.el7.x86_64 lỗi, 3.10.0-1062.18.1.el7.x86_64 không lỗi"

Lỗi của module connlimit, chúng tôi vẫn chưa xác định được chính xác nguyên nhân lỗi do kernel hay do iptables version, bởi vì:
- Kernel: chúng tôi đã thử upgrade lên kernel 4.x, 5.x … đều bị lỗi tương tự.
- iptables: chúng tôi đã upgrade lên iptables version mới nhất v1.4.21 vẫn bị lỗi

Tuy nhiên, với CentOS 8 thì lỗi này đã được khắc phục. Kiểm tra kỹ một chút thì thấy CentOS 8 đã chuyển sang sử dụng nftables thay cho iptables, chúng tội dự đoán đây có thể là nguyên nhân chính!

Vậy nftables, netfilter là gì và chúng với iptables liên quan nhau thế nào?

## **NFTABLES vs NETFILTER**

***netfilter***: *There may be some confusion about the difference between Netfilter and iptables. Netfilter is an infrastructure; it is the basic API that the Linux 2.4 kernel offers for applications that want to view and manipulate network packets. Iptables is an interface that uses Netfilter to classify and act on packets.*

Theo định nghĩa trên, netfilter là một framework của kernel cho phép các ứng dụng tương tác với packet, còn iptables chỉ là công cụ cho phép tương tác với netfilter. Trong quá trình phát triển ip_tables, đội ngũ của netfilter nhận thấy nó có nhiều bất cập về tính năng, hiệu năng nên đã phát triển phiên bản mới để thay thế: đó chính là nftables

Lưu ý:
- Để iptables hoạt động được: "**iptables** requires a kernel that features the **ip_tables** packet filter. This includes all 2.4.x and later kernel releases".
- iptables dùng để chỉ userspace tool – cái mà chúng ta hay dùng để tương tác, còn ip_tables là kernel module)

**nftables**: is a subsystem of the Linux kernel providing filtering and classification of network packets/datagrams/frames. It has been available since Linux kernel 3.13 released on 19 January 2014. nftables replaces the legacy iptables portions of Netfilter. Among the advantages of nftables over iptables is less code duplication and easier extension to new protocols. nftables is configured via the user-space utility nft, while legacy tools are configured via the utilities iptables, ip6tables, arptables and ebtables frameworks.

nftables cũng là một framework của kernel cho phép tương tác với packet, nó được add vào kernel từ version 3.13, công cụ để tương tác với nftables là nft. nftables thay thế *"features ip_tables packet filter"* của netfilter, khắc phục các yếu điểm của ip_tables.

Theo trang chủ của netfilter:

"**nftables** replaces the popular **{ip,ip6,arp,eb}tables**. This software provides a new in-kernel packet classification framework that is based on a network-specific Virtual Machine (VM) and a new **nft** userspace command line tool. **nftables** reuses the existing Netfilter subsystems such as the existing hook infrastructure, the connection tracking system, NAT, userspace queueing and logging subsystem." 

Từ đây, ta thấy ở CentOS 8, phần backend filter packet bên dưới đã không còn là ip_tables mà đã chuyển sang sử dụng nftables, do đó, rất có thể 2 lỗi kể trên xuất phát từ iptables và được khắc phục ở nftables.

## **Các tài liệu tham khảo:**
- https://ungleich.ch/en-us/cms/blog/2018/08/18/iptables-vs-nftables/
- https://en.wikipedia.org/wiki/Nftables
- https://www.netfilter.org/projects/iptables/index.html
- https://www.netfilter.org/projects/nftables/index.html
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-nftables_configuring-and-managing-networking

Nguồn: [Vietnix](https://blog.vietnix.vn/loi-khi-chong-ddos-bang-iptables.html)

---
