---
layout: post
comments: true
title:  "Cassandra - Kiến trúc cơ sở dữ liệu (Phần 3)"
title2:  "Kiến trúc cơ sở dữ liệu Cassandra (Phần 3)"
date:   2021-03-22 10:15:00
permalink: 2021/03/22/Kien-truc-CSDL-Cassandra-p3
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/Kien-truc-CSDL-Cassandra-p3/Hinh1.PNG
summary: Kiến trúc cơ bản của cơ sở dữ liệu Cassandra
---

<!-- MarkdownTOC -->

- [Mức độ nhất quán](#-Consistency Levels)
- [Truy vấn và nút điều phối](#-Queries-and-Coordinator-Nodes)
- [Gợi ý bàn giao](#-Hinted Handoff)
- [Chiến lược nhân rộng](#-Replication-Strategies)
- [Tài liệu tham khảo](#-Tai-lieu-tham-khao)

<!-- /MarkdownTOC -->

<a name="-Consistency-Levels"></a>
## Mức độ nhất quán (Consistency Levels)

Trong phần trước, chúng ta đã thảo luận về định lý CAP của Brewer, trong đó tính nhất quán, tính khả dụng và dung sai phân vùng được trao đổi với nhau. Cassandra cung cấp các mức độ nhất quán có thể điều chỉnh cho phép bạn thực hiện các đánh đổi này ở mức chi tiết. Bạn chỉ định mức nhất quán trên mỗi truy vấn đọc hoặc ghi cho biết mức độ nhất quán mà bạn yêu cầu. Mức độ nhất quán cao hơn có nghĩa là nhiều nút hơn cần phản hồi truy vấn đọc hoặc ghi, giúp bạn đảm bảo hơn rằng các giá trị hiện có trên mỗi bản sao là như nhau. 

Đối với truy vấn đọc, mức nhất quán chỉ định số lượng nút bản sao phải đáp ứng yêu cầu đọc trước khi trả lại dữ liệu. Đối với hoạt động ghi, mức nhất quán chỉ định số lượng nút bản sao phải phản hồi để ghi được báo cáo là thành công cho máy khách. Vì Cassandra cuối cùng đã nhất quán, các bản cập nhật cho các nút bản sao khác có thể tiếp tục trong nền.

Các cấp độ nhất quán có sẵn bao gồm MỘT, HAI và BA, mỗi cấp chỉ định số lượng tuyệt đối các nút bản sao phải phản hồi một yêu cầu. Mức nhất quán QUORUM yêu cầu phản hồi từ phần lớn các nút bản sao. Điều này đôi khi được biểu thị là:

	*Q* = *floor*(*RF*/2 + 1)

Trong phương trình này, *Q* đại diện cho số lượng nút cần thiết để đạt được định số tối thiểu cho hệ số nhân bản RF. Có thể đơn giản hơn để minh họa điều này bằng một vài ví dụ: nếu *RF* là 3 thì *Q* là 2; nếu *RF* là 4 thì *Q* là 3; nếu *RF* là 5 thì *Q* là 3, v.v.

Mức độ nhất quán TẤT CẢ yêu cầu phản hồi từ tất cả các bản sao. Chúng ta sẽ xem xét các mức nhất quán này và các mức khác chi tiết hơn trong phần sau.

Tính nhất quán có thể điều chỉnh được trong Cassandra vì máy khách có thể chỉ định mức độ nhất quán mong muốn trên cả đọc và ghi. Có một phương trình được sử dụng phổ biến để biểu thị cách đạt được tính nhất quán mạnh mẽ trong Cassandra: R + W > RF = tính nhất quán mạnh mẽ (strong consistency). Trong phương trình này, *R*, *W* và *RF* lần lượt là số bản sao đọc, số bản sao ghi và hệ số sao chép; tất cả các lần đọc của máy khách sẽ thấy lần ghi gần đây nhất trong trường hợp này và bạn sẽ có tính nhất quán cao. Như chúng ta sẽ thảo luận chi tiết hơn trong phần sau, cách được khuyến nghị để đạt được tính nhất quán mạnh mẽ trong Cassandra là ghi và đọc bằng cách sử dụng mức độ nhất quán QUORUM hoặc LOCAL_QUORUM.

---

***Phân biệt các mức độ nhất quán và các yếu tố nhân bản**

*Nếu bạn chưa quen với Cassandra, bạn có thể dễ nhầm lẫn giữa các khái niệm về hệ số nhân bản và mức độ nhất quán. Hệ số nhân bản được đặt trên mỗi keyspace. Mức độ nhất quán được chỉ định cho mỗi truy vấn, bởi ứng dụng máy khách. Yếu tố nhân bản cho biết bạn muốn sử dụng bao nhiêu nút để lưu trữ một giá trị trong mỗi thao tác ghi. Mức độ nhất quán chỉ định số lượng nút mà máy khách đã quyết định phải đáp ứng để cảm thấy tin tưởng về một hoạt động đọc hoặc ghi thành công. Sự nhầm lẫn nảy sinh bởi vì mức độ nhất quán dựa trên yếu tố nhân bản, không dựa trên số lượng các nút trong hệ thống.*

---

<a name="-Queries-and-Coordinator-Nodes"></a>
## Truy vấn và nút điều phối (Queries and Coordinator Nodes)

Hãy tập hợp các khái niệm này lại với nhau để thảo luận về cách các nút Cassandra tương tác để hỗ trợ đọc và ghi từ các ứng dụng máy khách. Hình dưới cho thấy đường tương tác điển hình với Cassandra. 
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Kien-truc-CSDL-Cassandra-p3/Hinh1.PNG" width = "800">
</div>
<div class="thecap">Hình 1: Máy khách, nút điều phối và bản sao</div>
</div>
<hr>

Máy khách có thể kết nối với bất kỳ nút nào trong cụm để bắt đầu truy vấn đọc hoặc ghi. Nút này được gọi là nút điều phối (coordinator node). Bộ điều phối xác định các nút nào là bản sao cho dữ liệu đang được ghi hoặc đọc và chuyển tiếp các truy vấn tới chúng.

Đối với việc ghi, nút điều phối liên hệ với tất cả các bản sao, được xác định bởi mức độ nhất quán và yếu tố sao chép, và coi việc ghi thành công khi một số bản sao tương ứng với mức độ nhất quán xác nhận việc ghi.

Đối với một lần đọc, điều phối viên liên hệ với đủ bản sao để đảm bảo đáp ứng mức độ nhất quán cần thiết và trả lại dữ liệu cho máy khách.

Tất nhiên, đây là những mô tả "con đường hạnh phúc" về cách hoạt động của Cassandra. Để có được bức tranh toàn cảnh về kiến trúc của Cassandra, bây giờ chúng ta sẽ thảo luận về một số cơ chế sẵn có cao của Cassandra mà nó sử dụng để giảm thiểu các lỗi, bao gồm cả việc sửa chữa và chuyển giao gợi ý. 

Gợi ý Handoff

Hãy xem xét tình huống sau: một yêu cầu ghi được gửi đến Cassandra, nhưng một nút bản sao nơi ghi đúng cách không khả dụng do phân vùng mạng, lỗi phần cứng hoặc một số lý do khác. Để đảm bảo tính khả dụng chung của chiếc nhẫn trong tình huống như vậy, Cassandra triển khai một tính năng được gọi là bàn giao gợi ý. Bạn có thể nghĩ về một gợi ý là một Ghi chú sau đó chứa thông tin từ yêu cầu viết. Nếu nút bản sao nơi ghi thuộc về không thành công, bộ điều phối sẽ tạo một gợi ý, đó là một lời nhắc nhỏ có nội dung “Tôi có thông tin ghi dành cho nút B. Tôi sẽ tiếp tục việc ghi này, và tôi sẽ nhận thấy khi nút B trực tuyến trở lại; khi nó xảy ra, tôi sẽ gửi cho nó yêu cầu viết thư. " Có nghĩa là, một khi nó phát hiện qua tin đồn rằng nút B đã trực tuyến trở lại, nút A sẽ “chuyển giao” cho nút B “gợi ý” về việc ghi. Cassandra có một gợi ý riêng cho từng phân vùng sẽ được viết.

Điều này cho phép Cassandra luôn sẵn sàng để ghi và thường cho phép một cụm duy trì cùng một tải ghi ngay cả khi một số nút bị hỏng. Nó cũng làm giảm thời gian mà một nút bị lỗi sẽ không nhất quán sau khi nó trực tuyến trở lại.

Nói chung, các gợi ý không được tính là viết cho các mục đích về mức độ nhất quán. Ngoại lệ là mức nhất quán BẤT KỲ, được thêm vào 0,6. Mức độ nhất quán này có nghĩa là chỉ một giao dịch gợi ý sẽ được coi là đủ để đạt được thành công của một thao tác ghi. Có nghĩa là, ngay cả khi chỉ có một gợi ý có thể được ghi lại, việc ghi vẫn được coi là thành công. Lưu ý rằng việc ghi được coi là bền, nhưng dữ liệu có thể không đọc được cho đến khi gợi ý được chuyển đến bản sao đích.

<a name="-Hinted-Handoff"></a>
## Gợi ý bàn giao (Hinted Handoff)

Hãy xem xét tình huống sau: một yêu cầu ghi được gửi đến Cassandra, nhưng một nút bản sao nơi ghi đúng cách không khả dụng do phân vùng mạng, lỗi phần cứng hoặc một số lý do khác. Để đảm bảo tính khả dụng chung của vòng trong tình huống như vậy, Cassandra triển khai một tính năng được gọi là bàn giao gợi ý. Bạn có thể nghĩ về một gợi ý là một Ghi chú sau đó chứa thông tin từ yêu cầu ghi. Nếu nút bản sao nơi ghi thuộc về không thành công, bộ điều phối sẽ tạo một gợi ý, đó là một lời nhắc nhỏ có nội dung "Tôi có thông tin ghi dành cho nút B. Tôi sẽ tiếp tục việc ghi này, và tôi sẽ nhận thấy khi nút B trực tuyến trở lại; khi nó xảy ra, tôi sẽ gửi cho nó yêu cầu ghi". Có nghĩa là, một khi nó phát hiện qua tin đồn rằng nút B đã trực tuyến trở lại, nút A sẽ "chuyển giao (hint)" cho nút B "gợi ý (hand off)" về việc ghi. Cassandra có một gợi ý riêng cho từng phân vùng sẽ được ghi.

Điều này cho phép Cassandra luôn sẵn sàng để ghi và thường cho phép một cụm duy trì cùng một tải ghi ngay cả khi một số nút bị hỏng. Nó cũng làm giảm thời gian mà một nút bị lỗi sẽ không nhất quán sau khi nó trực tuyến trở lại.

Nói chung, các gợi ý không được tính là ghi cho các mục đích về mức độ nhất quán. Ngoại lệ là mức nhất quán BẤT KỲ, được thêm vào 0,6. Mức độ nhất quán này có nghĩa là chỉ một giao dịch gợi ý sẽ được coi là đủ để đạt được thành công của một thao tác ghi. Có nghĩa là, ngay cả khi chỉ có một gợi ý có thể được ghi lại, việc ghi vẫn được coi là thành công. Lưu ý rằng việc ghi được coi là bền vững, nhưng dữ liệu có thể không đọc được cho đến khi gợi ý được chuyển đến bản sao đích.

---

***Chuyển giao Gợi ý và phân phối Đảm bảo (Hinted Handoff and Guaranteed Delivery)***

Chuyển giao Gợi ý được sử dụng trong Amazon’s Dynamo, điều này đã truyền cảm hứng cho việc thiết kế cơ sở dữ liệu, bao gồm Cassandra và Amazon’s DynamoDB. Nó cũng quen thuộc với những người biết về khái niệm phân phối đảm bảo trong các hệ thống nhắn tin như Java Message Service (JMS). Trong hàng đợi JMS phân phối đảm bảo bền vững, nếu không thể gửi một thông điệp đến người nhận, JMS sẽ đợi trong một khoảng thời gian nhất định và sau đó gửi lại yêu cầu cho đến khi nhận được tin nhắn. 

---

<div class="row">
  <div class="col-md-8" markdown="1">
  Some text.
  </div>
  <div class="col-md-4" markdown="1">
  <!-- ![Alt Text](../img/folder/blah.jpg) -->
  <img height="600px" class="center-block" src="/assets/Kien-truc-CSDL-Cassandra-p2/Hinh1.PNG">
  </div>
</div>



<hr>
<div class="imgcap">
<div >
    <img src="/assets/Kien-truc-CSDL-Cassandra-p2/Hinh2.PNG" width = "800">
</div>
<div class="thecap">Hình 2: Vai trò của trình phân vùng</div>
</div>
<hr>



<a name="-Tai-lieu-tham-khao"></a>
## Tài liệu tham khảo
- Ebook Cassandra: The Definitive Guide by Jeff Carpenter and Eben Hewitt


---