---
layout: post
comments: true
title:  "Security - Bảo vệ hệ thống trước kiểu tấn công Man in the Middle (MITM)"
title2:  "Bảo vệ hệ thống trước kiểu tấn công Man in the Middle (MITM)"
date:   2021-04-09 15:10:00
permalink: 2021/04/09/Bao-ve-he-thong-truoc-tan-cong-MITM
mathjax: false
tags: Security
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Bao-ve-he-thong-truoc-tan-cong-MITM/Hinh1.jpg
summary: Bảo vệ hệ thống trước kiểu tấn công Man in the Middle (MITM)
---

Bảo vệ hệ thống trước kiểu tấn công Man in the Middle (MITM)
Man in the Middle (MITM) là một trong những kiểu tấn công mạng thường thấy nhất được sử dụng để đánh cấp những thông tin cá nhân và các tổ chức chính thông qua kết nối đầu cuối. Có thể rằng MITM giống như một kẻ nghe trộm. MITM hoạt động bằng cách thiết lập các kết nối đến máy tính nạn nhân (victim) và chuyển tiếp dữ liệu giữa chúng.

Trong trường hợp bị tấn công, nạn nhân (victim) cứ tin tưởng là họ đang truyền thông một cách trực tiếp với đầu bên kia có thể là máy chủ hoặc một hệ thống, nhưng sự thực thì các luồng truyền thông lại bị thông qua Host (thiết bị PC, Laptop, Server) của kẻ thực hiện tấn công MITM. Và kết quả là các host này không chỉ có thể thu thập những dữ liệu thông tin nhạy cảm mà nó còn có thể gửi xen vào cũng như thay đổi nội dung hoặc luồng dữ liệu để kiểm soát sâu hơn thông tin của nạn nhân.

Ví dụ minh hoạ thực tế:
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Bao-ve-he-thong-truoc-tan-cong-MITM/Hinh1.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

Trong cuộc đấu giá gói thầu xây dựng cho công trình ABC. Có rất nhiều công ty tham gia nhận gói thầu thông qua việc gửi hồ sơ đấu thầu online. Công ty XYZ đưa ra gói thầu là 5 tỷ và thông tin chi tiết giá đấu này là tuyệt đối bí mật.

Vì không quan tâm đến bảo mật và an ninh mạng, cũng như công ty XYZ không có nhân sự chuyên nghiệp về hệ thống và bảo mật. Công ty XYZ đã bị hacker tấn công hệ thống liên lạc nội bộ theo hình thức MITM các thông tin nội bộ giữa lãnh đạo trong Hội đồng quản trị trao đổi với nhau đã bị Hacker đọc lén.

Thay vào đó Hacker đã bán thông tin gói thầu của công ty XYZ này cho các đối thủ cạnh tranh khác và bên cạnh Hacker còn thay đổi thông tin giá thầu trong nội bộ từ 5 tỷ thành 1 tỷ. Làm sai lệnh thông tin trao đổi trong nội giữa các lãnh đạo trong Hội đồng quản trị công ty XYZ và một số thông tin khác giữa các phòng ban, nhân viên hay các chi nhánh trong hệ thống.

## **Các phương pháp tấn công MITM**

Kẻ tấn có thể sử dụng các cuộc tấn công MITM để giành quyền kiểm soát các thiết bị theo nhiều cách khác nhau.

### **1. Sniffing – Thăm dò**

Sniffing hoặc Packet Sniffing là một kỹ thuật được sử dụng để nắm bắt các gói dữ liệu chảy vào và ra khỏi một hệ thống/mạng. Packet Sniffing trong mạng tương đương với việc nghe trộm trong điện thoại. Hãy nhớ rằng Sniffing là hợp pháp nếu được sử dụng đúng cách và nhiều doanh nghiệp làm điều đó vì mục đích bảo mật.

### **2. Packet Injection – Tiêm mã độc vào gói tin**

Trong kỹ thuật này, kẻ tấn công đưa các gói dữ liệu độc hại vào với dữ liệu thông thường. Bằng cách này, người dùng thậm chí không nhận thấy tệp / phần mềm độc hại vì chúng đến như một phần của luồng truyền thông hợp pháp. Những tập tin này rất phổ biến trong các cuộc tấn công trung gian cũng như các cuộc tấn công từ chối dịch vụ.

### **3. IP spoofing – Giả mạo địa chỉ IP**

Mỗi thiết bị có khả năng kết nối với internet đều có internet protocolt address (IP), tương tự như địa chỉ cho nhà bạn. Với IP spoofing, kẻ tấn công có thể thay thế bạn hoặc đối tượng tương tác với bạn và lừa bạn rằng bạn đang liên hệ trực tiếp với bên kia, kẻ tấn công có thể truy cập vào thông tin mà bạn đang trao đổi.

### **4. DNS spoofing – Giả mạo thông tin DNS**

Domain Name Server (DNS) spoofing là một kỹ thuật buộc người dùng vào một website giả mạo chứ không phải trang mà người dùng dự định truy cập. Nếu bạn là nạn nhân của DNS spoofing, bạn sẽ nghĩ rằng bạn đang truy cập một website đáng tin khi bạn thực sự tương tác với một kẻ lừa đảo. Mục tiêu của thủ phạm là tăng lượng truy cập website giả mạo hoặc đánh cắp thông tin đăng nhập của người dùng.

Kẻ tấn công giả mạo DNS bằng cách thay đổi địa chỉ của website trong máy chủ DNS. Nạn nhân vô tình truy cập website giả mạo và kẻ tấn công sẽ cố gắng đánh cắp thông tin của họ.

### **5. HTTPS spoofing – Giả mạo HTTPS**

Khi truy cập website, HTTPS trong URL, chứ không phải là HTTP là dấu hiệu cho thấy website này an toàn. Kẻ tấn công có thể đánh lừa trình duyệt của bạn rằng đang truy cập một website đáng tin cậy bằng cách chuyển hướng trình duyệt của bạn đến một website không an toàn sau khi truy cập, kẻ tấn công có thể theo dõi các tương tác của bạn với website đó và có thể đánh cắp thông tin cá nhân bạn đang chia sẻ.

### **6. SSL hijacking – Đánh cắp SSL**

Khi thiết bị của bạn kết nối với máy chủ không bảo mật (HTTP) máy chủ thường có thể tự động chuyển hướng bạn đến phiên bản bảo mật (HTTPS). Kết nối đến một máy chủ an toàn có nghĩa là các giao thức bảo mật tiêu chuẩn được đặt ra, bảo vệ dữ liệu bạn chia sẻ với máy chủ đó. Secure Sockets Layer (SSL), một giao thức thiết lập các liên kết được mã hóa giữa trình duyệt và máy chủ web.

Tấn công SSL, kẻ tấn công sử dụng một máy tính và máy chủ bảo mật khác và chặn tất cả thông tin truyền qua giữa máy chủ và máy tính của người dùng.

### **7. Email hijacking – Đánh cắp Email**

Một cuộc tấn công trung gian phổ biến khác là Email hijacking.

Giả sử bạn đã nhận được một email có vẻ là từ ngân hàng của bạn, yêu cầu bạn đăng nhập vào tài khoản để xác nhận thông tin liên hệ. Bạn nhấp vào một liên kết trong email và được đưa đến trang đăng nhập và thực hiện nhiệm vụ được yêu cầu.

Trong kịch bản này, MITM đã gửi cho bạn email, làm cho nó có vẻ hợp pháp. Nhưng khi bạn làm điều đó, bạn không đăng nhập vào tài khoản ngân hàng của mình, bạn đang bàn giao thông tin đăng nhập cho kẻ tấn công.

Kẻ tấn công nhắm vào email khách hàng của các ngân hàng và các tổ chức tài chính khác. Khi họ có quyền truy cập, họ có thể giám sát các giao dịch giữa tổ chức và khách hàng của mình. Những kẻ tấn công sau đó có thể giả mạo địa chỉ email của ngân hàng và gửi email có chứa một vài hướng dẫn cho khách hàng. Điều này khiến cho khách hàng làm theo hướng dẫn của kẻ tấn công chứ không phải ngân hàng. Kết quả tồi tệ là khách hàng đặt tiền vào tay kẻ tấn công.

Email sẽ có vẻ hợp pháp và vô hại đối với người nhận làm cho cuộc tấn công này rất hiệu quả và tàn phá về tài chính.

### **8. WiFi eavesdropping – Nghe lén Wi-Fi**

WiFi Eavesdropping – một cách thụ động để triển khai các cuộc tấn công MITM.

Tấn công dạng MITM thường xuyên xảy ra trên các mạng WiFi.

Với hình thức MITM truyền thống, kẻ tấn công cần có quyền truy cập vào bộ định tuyến WiFi (Có thể do không được bảo mật hoặc bảo mật kém). Các loại kết nối này thường là các kết nối công cộng (Các điểm truy cập Wi-Fi miễn phí), hoặc cũng có thể là WiFi cá nhân nếu người dùng không bảo vệ tốt cho chúng.

Tội phạm mạng có thể thiết lập kết nối WiFi với các tên nghe có vẻ rất hợp pháp. Khi người dùng kết nối với WiFi của kẻ tấn công, họ có thể theo dõi hoạt động trực tuyến của người dùng và có thể chặn thông tin đăng nhập, thông tin thẻ thanh toán…

Khi kẻ tấn công tìm thấy bộ định tuyến dễ bị tấn công, chúng có thể triển khai các công cụ để chặn và đọc dữ liệu truyền của nạn nhân. Kẻ tấn công sau đó cũng có thể chèn các công cụ của chúng vào giữa máy tính của nạn nhân và các website mà nạn nhân truy cập để ghi lại thông tin đăng nhập, thông tin ngân hàng và thông tin cá nhân khác.

### **9. Stealing browser cookies – Đánh cắp Cookie trình duyệt**

Để hiểu Stealing browser cookies, bạn cần biết: Cookie trình duyệt là một phần thông tin nhỏ mà một website lưu trữ trên máy tính của bạn.

Một tội phạm mạng có thể chiếm quyền điều khiển các cookie trình duyệt. Vì cookie lưu trữ thông tin từ phiên duyệt web của bạn, kẻ tấn công có thể truy cập vào mật khẩu, địa chỉ và thông tin nhạy cảm khác của bạn.

## **Một số cách tránh các cuộc tấn công man-in-the-middle ?**

- Đảm bảo rằng các trang web bạn truy cập đã được cài SSL (tức là truy cập bằng HTTPS thay vì HTTP)
- Trước khi nhấp vào email, hãy kiểm tra người gửi Email
- Nếu bạn là quản trị viên trang web, bạn nên triển khai HTST
- Hạn chế kết nối trực tiếp với bộ định tuyến WiFi công cộng. Virtual Private Network (VNP) mã hóa kết nối internet của bạn trên các điểm truy cập công cộng để bảo vệ dữ liệu riêng tư bạn gửi và nhận trong khi sử dụng WiFi công cộng, như mật khẩu hoặc thông tin thẻ tín dụng.
- Hãy tăng tính bảo mật cho mạng WiFi tại nhà của bạn bằng cách thay đổi tên người dùng và mật khẩu mặc định trên bộ định tuyến và tất cả các thiết bị được kết nối thành mật khẩu mạnh và duy nhất.
- Người dùng có thể tự bảo vệ mình khỏi MITM bằng cách tránh gửi bất kỳ thông tin cá nhân nào qua mạng WiFi công cộng trừ khi chúng được bảo vệ bởi VPN.

Nguồn: [tranhieuit](https://tranhieuit.com/tinh-nang-noi-bat-windows-server-2019-download-server-2019/)

---
