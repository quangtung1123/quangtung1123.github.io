---
layout: post
comments: true
title:  "SQL SERVER DBA - Công việc hằng ngày của quản trị viên cơ sở dữ liệu"
title2:  "Công việc hằng ngày của DBA"
date:   2021-03-10 15:10:00
permalink: 2021/03/10/vanhanhsqlserver
mathjax: false
tags: SQL_Server Database
category: SQL Server
sc_project: 12494739
sc_security: d785a534
img: /assets/vanhanhsqlserver/profile.png
summary: Công việc hằng ngày của quản trị viên cơ sở dữ liệu
---

Nếu như bạn thắc mắc về công việc hằng ngày của một quản trị viên (database administrator – DBA) là những gì, hoặc sắp tới bạn được giao nhiệm vụ quản trị một hoặc vài máy chủ cơ sở sở dữ liệu – SQL Server và muốn tìm hiểu những gì cần phải làm với nó hoàn thành trách nhiệm được giao thì hi vọng bài viết này giúp bạn phần nào hình dung được những công việc đang chờ đón. Bất cứ một vị trí nào công việc và trách nhiệm đều không giới hạn, và với vai trò là quản trị viên cơ sở dữ liệu thì công việc cũng đa dạng và khác biệt giữa các công ty, tổ chức khác nhau, nó phụ thuộc vào số lượng cũng như kích thước của dữ liệu, lượng người dùng, hoạt động của doanh nghiệp và đặc biệt hơn nữa mức độ chủ động của người quản trị.

## Công việc hằng ngày của quản trị viên cơ sở dữ liệu
- Cài đặt, cấu hình SQL Server, cập nhật các bản build (setup and configuration)
- Thực hiện sao lưu dự phòng và khôi phục dữ liệu ( backup & recovery)
- Thiết kế và cài đặt tính sẵn sàng cho hệ thống (high availability)
- Theo dõi và kiểm tra trạng thái hoạt động hệ thống (system health check)
- Thực hiện các công việc bảo trì (maintenance tasks)
- Cải thiện hiệu năng hệ thống (performance tuning)
- Quản trị các SQL users và logins, thu cấp quyền (security management)
- Khắc phục các sự cố phát sinh (troubleshooting)
- Triển khai những thay đổi, thêm mới các database objects trên production (Deployment)
- Luôn luôn sẵn sàng 24/7
- Trau dồi kiến thức chuyên môn (self-improvement)

## Cài đặt, cấu hình SQL Server, cập nhật các bản build
Đây là công việc không thể thiếu với vai trò của một quản trị viên, cài đặt SQL Server là việc đầu tiên bạn phải thực hiện để xây dựng các ứng dụng cơ sở dữ liệu. Tiếp sau cài đặt là việc cấu hình cho SQL Server instance với những thông số phù hợp để đạt được hiệu năng cao, đây là vấn đề không đơn giản như chúng ta tưởng vì thông thường chúng ta giữ nguyên các giá trị mặc định từ Microsoft mà không hẳn phù hợp cho tình huống cụ thể của mình. Ví dụ như việc cấp phát CPU, memory, các cấu hình liên quan đến xử lý truy vấn song song (parallelism), cấu hình tempdb,..đều cần các thiết lập phù hợp cho tình huống cụ thể. Bên cạnh đó quản trị viên cũng cần phải theo dõi các bản cập nhật của SQL Server ( cumulative update hoặc service pack) nếu thấy cần thiết cho hệ thống của mình.

## Thực hiện sao lưu dự phòng và khôi phục dữ liệu
Sao lưu dự phòng là việc tạo ra các bản copy nhằm đảm tăng khả năng tồn tại của dữ liệu doanh nghiệp. Các sự cố như ổ cứng gặp vấn đề không truy xuất được hoặc xóa nhầm dữ liệu có thể xảy ra bất cứ lúc nào và yêu cầu đặt ra cho quản trị viên là có thể khôi phục dữ liệu nhiều nhất có thể trong những tình huống như vậy. Quản trị viên cần kiểm tra các bản sao lưu thường xuyên để chắc chắn chúng có khả năng khôi phục dữ liệu khi cần.

## Thiết kế và cài đặt tính sẵn sàng cho hệ thống
Bên cạnh việc bảo vệ dữ liệu doanh nghiệp còn đòi hỏi khả năng khôi phục lại hệ thống để tiếp tục phục vụ việc kinh doanh càng nhanh càng tốt. Chẳng hạn một hệ thống gặp sự cố máy chủ cơ sở dữ liệu bị chết không truy cập được buộc phải dừng mọi hoạt động, mong muốn của doanh nghiệp là trong một hoặc hai tiếng đồng hồ hệ thống phải hoạt động như cũ thì đó chính là thước đo tính sẵn sàng của một hệ thống. Thời gian khôi phục càng thấp tính sẵn sàng cao. Hai thông thông số điển hình để xác định mức độ sẵn sàng của hệ thống là RTO và RPO – cần bao lâu để khôi phục và khôi phục tới thời điểm nào.

## Theo dõi và kiểm tra trạng thái hoạt động hệ thống
Bạn cần phải rõ hơn ai hết những “tâm sinh lý” của hệ thống mình đang quản trị nên bạn cần có những tiêu chí đo lường để theo dõi và đánh giá theo thời gian. Để dễ dàng nhận biết sự biến động khác thường của hệ thống bạn nên có thông số tiêu chuẩn của các tiêu chí theo dõi (baseline counters). Việc theo dõi sát sao các error logs (SQL Server logs & Windows logs) cũng góp phần xác định tình trạng sức khỏe của hệ thống để có hành động kịp thời.

## Thực hiện các công việc bảo trì
Để đảm bảo mọi thứ luôn ở trạng thái tốt nhất có thể. Những công việc có thể kể đến như là chống phân mảnh indexes (indexes defragmentation), kiểm tra toàn vẹn cơ sở dữ liệu (database integrity checks), cập nhật statistics,…sẽ giúp SQL Server kéo dài khả năng phục vụ các truy vấn dữ liệu.

## Cải thiện hiệu năng hệ thống
Theo mình đây là công việc thách thức và đầy thú vị khi là một quản trị viên. Tùy vào tình huống bạn gặp phải mà bạn cần tác động vào đúng thành phần gây nên các điểm nghẽn. Để cái thiện hiệu năng có thể phải hiệu chỉnh các câu truy vấn, sửa đổi hay tạo thêm các index, thay đổi cấu hình SQL Server instance hoặc thậm chí là nâng cấp phần cứng,… Những việc này đòi hỏi bạn phải có kiến thức vững chắc để có thể xác định chính xác vấn đề nằm ở đâu và khi sửa đổi sẽ không gây ra những điểm nghẽn khác.

## Quản trị các SQL users và logins, thu cấp quyền
Luôn có một nguyên tắc đó là cấp phát vừa đủ quyền hạn sử dụng cho bất kì một user nào.

## Khắc phục các sự cố phát sinh
Là công việc có thể xem là thường xuyên nhưng không thể biết trước được. Có hàng tá thứ có thể xảy ra ngay cả khi mọi thứ đang yên ổn cả thời gian dài trước đó như dữ liệu bị không toàn vẹn, SQL Server Replication chậm truyền tải data, những errors lạ xuất hiện, hoặc lượng dữ liệu tăng nhiều bất thường,…Tùy vào kích thước cơ sở dữ liệu, lượng người dùng và công nghệ, kĩ thuật bạn sử dụng tương ứng những sự cố này sẽ đa dạng và có độ phức tạp khác nhau. Với vai trò là một quản trị viên công việc của bạn là tìm ra nguyên nhân và khắc phục chúng trong thời gian ngắn nhất có thể, đặc biệt hơn là không nên để nó xảy ra lần nữa.

## Triển khai những thay đổi, thêm mới các database objects trên production
Đây là loại công việc đòi hỏi sự tập trung và chính xác cao độ vì bất cứ một thiếu xót nhầm lẫn nào đều có thể gây hậu quả nghiêm trọng. Cùng một hành động nhưng thực hiện trên hai đối tượng khác nhau có thể mang lại kết quả khác nhau một trời một vực. Ví dụ bạn muốn mở rộng kiểu dữ liệu của một cột nào đó từ varchar(20) lên varchar(50) chẳng hạn, chạy trên bảng có dữ liệu lớn sẽ lâu hơn nhiều so với bảng nhỏ và nếu thời gian thực thi càng lớn mức độ ảnh hưởng càng rộng chứ không chỉ trên mỗi bảng đó. Do vậy mọi hành động sửa đổi nên được chạy thử ở môi trường phát triển trước khi áp dụng cho production.

## Luôn luôn sẵn sàng 24/7
Như đã đề cập ở trên các sự cố phát sinh bất cứ lúc nào và khi quản trị viên ở bất cứ nơi đâu. Bạn hãy luôn sẵn sàng để nhận các tín hiệu theo dõi tự động hoặc nhận cuộc gọi từ các thành viên khác trong công ty.

## Trau dồi kiến thức chuyên môn
Không riêng gì nghề này, dù bạn có làm việc trong lĩnh vực nào đi chăng nữa thì việc liên tục nâng cao năng lực chuyên môn, trau dồi các kĩ năng hỗ trợ cho công việc là luôn cần thiết. Mọi vấn đề bạn gặp phải hầu như đều có sẵn đâu đó trên google, nhưng bạn cũng cần thời gian và trải nghiệm và có hiểu biết nhất định để có thể tự tin áp dụng các giải pháp tìm thấy trên internet mà không gây ra thêm hậu quả gì khác. Khi làm việc với SQL Server bạn sẽ cần có thêm kiến thức về hệ điệu hành Windows Server, hiểu biết về phần cứng, thiết bị, công nghệ lưu trữ (storage), mạng máy tính để có thể tìm tòi và khắc phục các sự cố xảy ra trong hệ thống.

Trên đây là những công việc mà hầu hết đều xuất hiện trong danh sách thường ngày của quản trị viên cơ sở dữ liệu. Tùy vào công ty khác nhau mà số lượng có thể nhiều hơn hoặc khác đi chứ không bao giờ ít hơn, đặc biệt là khi bạn phải quản trị nhiều hơn một hệ quản trị cơ sở dữ liệu như MySQL, Oracle, PostgreSQL.

Nguồn: https://quantricsdulieu.com/2021/03/12/cong-viec-hang-ngay-cua-quan-tri-vien-csdl-sql-server-dba/

---
