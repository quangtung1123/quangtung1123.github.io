---
layout: post
title:  "Tản mạn về một cuộc điều tra tấn công DDoS"
date:   2021-08-24 15:10:00
permalink: 2021/08/24/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos
tags: Security DDOS
category: Security
img: /assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/Hinh1.png
summary: Tản mạn về một cuộc điều tra tấn công DDoS

---

Xin chào, sau một thời gian chống chọi Covid, Zepto team chúng tôi đã quay trở lại rồi đây =]]

Hôm nay chúng tôi sẽ chia sẻ về quá trình điều tra, truy vết nguồn gốc của một chiến dịch tấn công DDoS mà hệ thống giám sát ghi nhận được. Qua điều tra, chúng tôi đã phát hiện đây là một cuộc tấn công DDoS đến từ một hệ thống botnet của một tổ chức chuyên bán các dịch vụ tấn công từ chối dịch vụ. Quá trình điều tra và truy vết như thế nào sẽ được chúng tôi chia sẻ cụ thể ở bên dưới, hi vọng mọi người sẽ cảm thấy hữu ích với bài viết này.

## Let’s go ##

Sau khi nhận được thông tin từ hệ thống giám sát rằng hệ thống đang có hiện tượng bị tấn công từ chối dịch vụ, chúng tôi đã ngay lập tức cử người đến để phối hợp ứng cứu sự cố và điều tra về tấn công này. Việc xử lý và ngăn chặn tấn công DDoS đã được thực hiện ngay sau đó và hệ thống đã quay trở lại hoạt động bình thường. Tuy nhiên mục tiêu chúng tôi đặt ra là làm sao để biết được tấn công này đến từ đâu và hệ thống nào điều khiển cuộc tấn công này thì vẫn còn là một dấu hỏi lớn cần được giải đáp.

Mục tiêu chúng tôi đặt ra trong trường hợp này như sau:

- Phát hiện được cách thức tấn công
- Phát hiện hệ thống điều khiển tấn công
- Phát hiện kẻ đứng sau thực hiện tấn công.
- 
## Điều tra cách thức tấn công ##

Sau khi phân tích các dữ liệu được cung cấp từ đội ngũ quản trị hệ thống, chúng tôi đã đưa ra kết luận đây là một cuộc tấn công SYN Flood nhằm vào hệ thống máy chủ ứng dụng. Đây là một hình thức tấn công từ chối dịch vụ thông qua việc gửi liên tục các gói tin khởi tạo kết nối SYN đến máy chủ dẫn đến vượt quá khả năng xử lý của máy chủ làm cho hệ thống không kịp phản hồi các kết nối hợp lệ từ người dùng thông thường.

Ngay sau khi phát hiện hình thức tấn công thì các biện pháp khắc phục, xử lý đã được thực hiện để đưa hệ thống trở lại trạng thái hoạt động bình thường.

## Điều tra hệ thống điều khiển tấn công ##

Ghi nhận thông tin hệ thống bị tấn công đồng loạt từ nhiều ip khác nhau (tấn công từ nhiều máy trạm), để làm được điều này kẻ tấn công không thể đến trực tiếp từng máy trạm để thực hiện hành động này. Do đó chắc chắn phải có một hệ thống đứng sau điều khiển tất cả máy trạm thực hiện tấn công đồng loạt vào một thời điểm nhất định.

Chúng tôi thực hiện lọc trên hệ thống bảo vệ và có kết quả nhận về một số lượng lớn ip vẫn đang có các hành vi thực hiện tấn công vào hệ thống nhưng đã bị chặn. Để tìm được kẻ đứng sau thì chúng tôi quyết định tiến hành điều tra truy vết thông tin từ những địa chỉ ip này.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image1.png" width = "800">
</div>
<div class="thecap">Danh sách ip bị chặn khi phát hiện tấn công SYN Flood</div>
</div>

Qua kiểm tra một số ip thuộc danh sách trên chúng tôi đưa ra hai giả thuyết như sau:

- Tấn công đến từ hệ thống máy tính bị nhiễm mã độc
- Tấn công đến từ một hệ thống các thiết bị IoT bị kiểm soát

Sử dụng hệ thống tra cứu thông tin của NCSC Việt Nam, chúng tôi phát hiện số lượng lớn ip bị chặn thuộc các hệ thống botnet.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image2.png" width = "800">
</div>
<div class="thecap">Các ip bị chặn nằm trong các hệ thống botnet</div>
</div>

Như vậy có thể nhận định rằng đây là một cuộc tấn công đến từ một hệ thống botnet. Kiểm tra các ip này phần lớn đang mở các dịch vụ quản trị từ xa của các thiết bị IoT. Có thể nhận định các thiết bị IoT này tồn tại các lỗ hổng bảo mật dẫn đến bị khai thác chiếm quyền điều khiển và đưa vào hệ thống botnet. Tuy nhiên, làm sao để biết được hệ thống botnet này hoạt động như nào và được điều khiển từ đâu lại là một vấn đề phức tạp.

Để giải quyết vấn đề này tôi có đưa ra hai cách khả thi như sau:

1. Liên hệ nhà mạng (bởi các ip này có nhiều ip thuộc các nhà mạng Việt Nam) để biết được ip này của khách hàng nào. Sau đó liên hệ khách hàng liên quan để lấy thiết bị, tiến hành phân tích thiết bị để tìm kiếm mã độc.
2. Tìm cách truy cập vào thiết bị từ xa để phân tích, tìm kiếm mã độc.

Có thể dễ thấy cách đầu tiên nêu ra tốn rất nhiều thời gian và nguồn lực, do đó bằng kỹ năng nghiệp vụ và những hệ thống thông tin có sẵn của NCSC, chúng tôi đã quyết định tìm phương án sử dụng cách thứ 2.

Sau khi truy cập thành công vào thiết bị nghi ngờ nằm trong hệ thống botnet, chúng tôi đã phát hiện được một số tiến trình lạ đang chạy trong thiết bị.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image3.png" width = "800">
</div>
<div class="thecap">Một số tiến trình lạ đang chạy trên thiết bị</div>
</div>

Truy cập vào dịch vụ ftp của ip tìm thấy trong command khởi chạy của một tiến trình bất thường chúng tôi phát hiện nhiều mẫu mã độc IoT được lưu tại đó.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image4.png" width = "800">
</div>
<div class="thecap">Mẫu mã độc IoT được lưu trên server nghi là c&c</div>
</div>

Các mẫu tìm thấy trùng khớp với các tập tin đang được chạy trên thiết bị, từ đó khẳng định một cách chính xác được thiết bị đang bị điều khiển bởi máy chủ mạng botnet.

Thực hiện phân tích mẫu mã độc thu được chúng tôi đã thấy được đây là một mẫu mã độc được sử dụng cho mục đích tấn công DDoS với nhiều chức năng khác nhau được điều khiển thông qua các lệnh được gửi về từ máy chủ c&c. Tìm kiếm trên mạng internet chúng tôi thấy được một số mã nguồn tương tự với mẫu này đã được công bố trên internet:

- https://pastebin.com/tZ4dLq2P
- https://gist.github.com/CanadianJeff/62cd5bdcec930e64b115

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image5.png" width = "800">
</div>
<div class="thecap">Phân tích mẫu mã độc tìm thấy trên thiết bị</div>
</div>

Mã độc nhận lệnh từ máy chủ c&c (dịch vụ chạy tại port 1111) và thực hiện hành động tấn công DDoS tương ứng.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image6.png" width = "800">
</div>
<div class="thecap">Thông tin dịch vụ c&c được tìm thấy trong mẫu mã độc</div>
</div>

Thực hiện kết nối đến c&c của botnet và nhận các lệnh từ trên c&c gửi về các máy trạm thông qua việc mô phỏng lại một thiết bị IoT nhiễm độc. Chúng tôi thấy rằng hệ thống này vẫn hoạt động bình thường, và các lệnh tấn công vẫn đang được gửi về từ c&c tới mạng botnet.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image7.png" width = "800">
</div>
<div class="thecap">Các lệnh được gửi về từ c&c</div>
</div>

## Truy tìm kẻ đứng sau thực hiện tấn công ##

Ngoài ra chúng tôi còn phát hiện trên thiết bị chạy một mẫu mã độc thực hiện các hành động tấn công dò quét lỗ hổng trên các thiết bị IoT khác nhằm mục đích mở rộng hệ thống botnet.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image8.png" width = "800">
</div>
<div class="thecap">Mẫu mã độc sử dụng để mở rộng hệ thống botnet</div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image9.png" width = "800">
</div>
<div class="thecap">Một trong số các mã khai thác được sử dụng để tấn công các thiết bị khác (CVE-2017–17215)</div>
</div>

Qua quá trình điều tra và phân tích các mẫu tìm được, chúng tôi đã phát hiện được 3 c&c:

- 163[.]172[.]40[.]236 có địa chỉ IP tại Pháp
- 51[.]77[.]220[.]127 có địa chỉ IP tại Pháp
- 185[.]132[.]53[.]194 có địa chỉ IP tại Đức

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image10.png" width = "800">
</div>
<div class="thecap">Thông tin liên quan đến máy chủ c&c</div>
</div>

Trong số 3 c&c ở trên, có một c&c có chứa nhiều thông tin hơn 2 c&c còn lại, kiểm tra thông tin máy chủ c&c này chúng tôi phát hiện một nhóm chuyên bán các dịch vụ DDoS nextstress[.]pw. Tuy nhiên tại thời điểm điều tra, trang web này không còn hoạt động. Có thể nhóm này đã chuyển sang sử dụng một hệ thống web khác tương tự.

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image11.png" width = "800">
</div>
<div class="thecap">Thông tin liên quan đến máy chủ c&c</div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/image12.png" width = "800">
</div>
<div class="thecap">Nhóm hacker quảng cáo dịch vụ DDoS trên twitter https://twitter.com/httpg0d1</div>
</div>

## Kết luận ##

Từ những kết quả trên chúng ta có thể kết luận rằng kẻ xấu đã thuê dịch vụ tấn công DDoS thuộc một trong các hệ thống tương tự nextstress[.]pw để thực hiện các hành vi tấn công vào hệ thống.

Tấn công DDoS là một dạng tấn công gây tác động lớn đến các hệ thống, và có thể thấy rằng việc thực hiện các hành vi xấu thông qua tấn công DDoS ngày càng dễ dàng thực hiện thông qua việc thuê các dịch vụ từ bên thứ ba. Các dịch vụ cung cấp các tấn công này thì có hệ thống botnet ngày càng phức tạp, và quy mô ngày càng lớn.

Có thể nói an toàn thông tin cho các thiết bị IoT là rất quan trọng, nó vừa đảm bảo an toàn cho hệ thống, vừa ngăn chặn được việc tận dụng thiết bị IoT cho các chiến dịch tấn công DDoS. Có thể nhiều người sẽ thắc mắc những vấn đề sau:

- Tại sao các thiết bị IoT lại là đối tượng để kẻ xấu tận dụng để xây dựng các hệ thống mạng botnet phục vụ cho các hình thức tấn công DDoS?
- Tại sao một thiết bị IoT lại có thể trở thành một máy trạm trong hệ thống mạng botnet?
- Làm sao để đảm bảo an toàn cho các thiết bị IoT?

Có thể những câu hỏi trên sẽ được giải đáp trong những blog tiếp theo của chúng tôi.

Cuối cùng, xin cảm ơn các bạn đã đọc bài viết này và hẹn gặp lại trong blog tiếp theo!

**Tài liệu tham khảo**

- [khonggianmang](https://blog.khonggianmang.vn/tan-man-ve-mot-cuoc-dieu-tra-tan-cong-ddos/)

---
