---
layout: post
comments: true
title:  "SnipeIT – Hướng dẫn sử dụng (Phần 1)"
title2:  "Hướng dẫn sử dụng Snipe-IT (Phần 1)"
date:   2021-04-11 15:10:00
permalink: 2021/04/11/Huong-dan-su-dung-SnipeIT-p1
mathjax: false
tags: SnipeIT ITTools
category: SnipeIT
sc_project: 12494739
sc_security: d785a534
img: /assets/Huong-dan-su-dung-SnipeIT-p1/Hinh0.png
summary: Hướng dẫn sử dụng Snipe-IT
---

Chào các bạn hiện mình đang setup để quản lý tài sản cho công ty nên mình sẽ kết hợp hướng dẫn sử dụng SnipeIT cơ bản cho các bạn trả lời câu hỏi: Phải làm gì với cái này sau khi cài lên đây? (Thấy khá nhiều bạn sau khi cài lên thì ngồi nhìn thôi không biết bắt đầu từ đâu cả).

Bài hướng dẫn sử dụng SnipeIT này dưa theo phiên bản sử dụng **tiếng việt** nha (nên đừng hỏi sao có mấy chữ ghi khó hiểu quá, do cộng đồng dịch chỉ hỗ trợ đến thế thôi, bạn có thể chuyển qua tiếng anh đọc cho dễ hiểu nghĩa, do mình bàn giao cho hành chính công ty toàn mấy chị em xài tiếng việt là chính nên ko để tiếng anh được).

Đầu tiên chúng ta cần xem các trường bắt buộc và nên có của 1 tài sản là những gì:
- Công Ty
- Thẻ tài sản
- Kiểu tài sản
- Tình trạng
- Tên tài sản
- Ngày mua
- Nhà cung cấp
- Vị trí mặc định
- Hình
- Người đang sử dụng

Mình sẽ demo 1 tài sản với các thông tin như sau: Công ty TCX, văn phòng tại HCM – Võ Văn Tần, Màn hình DELL 21.5″, mua tại Phong Vũ, nhân viên Annlt đang sử dụng, mua hôm 1/1/2021.

Sau đây bắt đầu tạo các thông tin cần thiết để đưa được tài sản vào phần mềm.

## **1- Công ty**

Các bạn có thể tạo nhiều công ty dựa theo quy mô và cách tổ chức của công ty cần sử dụng. Như ở đây mình tạo 1 công ty để làm mẫu – Công ty TCX
- Cài đặt -> Các Công ty – Tạo mới

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh1.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

- Nhập tên công ty, thêm logo nếu có (nên có để nhìn cho chuyên nghiệp xíu) -> Lưu

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh2.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr> 

## **2- Danh mục tài sản**

Chúng ta xác định màn hình này thuộc về đống tài sản CNTT – tùy vào tài sản mà bạn cần quản lý thuộc danh mục nào mà tạo (vd nội ngoại thất, dụng cụ văn phòng, vật liệu xây dựng…), nên sẽ tạo danh mục như sau:
- Cài đặt -> Danh mục -> Tạo mới

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh3.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr> 

- Nhập tên danh mục (**Thiết bị CNTT**) -> loại (ở đây có nhiều loại tùy theo mà các bạn chọn, ở đây mình chọn nó là asset) -> Lưu (phần này không cần có hình cũng được)
- "Loại" ở đây các bạn có thể thay đổi ví như là bản quyền, phụ kiện hay vật tư, tùy theo các bạn muốn quản lý cái gì, ở đây chủ yếu mình chỉ quản lý tài sản (asset).

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh4.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## **3- Nhà sản xuất**

- Cài đặt -> Nhà sản xuất -> Tạo mới

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh5.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr> 

Điền tên hãng thiết bị (**DELL**) -> Lưu

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh6.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## **4- Kiểu tài sản**

Phần này thì tùy các bạn xác định tùy theo các loại thiết bị trong công ty bạn– ví dụ công ty mình chỉ có màn 21.5″ thôi nhưng sẽ có nhiều hãng
- Cài đặt -> Kiểu tài sản -> Tạo mới

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh7.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

- Điền tên kiểu tài sản (**Màn hình 21.5**) -> chọn nhà sản xuất (DELL – đã tạo phía trên) -> Chọn tên hạng mục (Thiết bị CNTT – đã tạo ở phần danh mục) -> Lưu

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh8.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## **5- Nhà cung cấp**

Tạo đơn vị cung cấp thiết bị – ở đây sẽ là Phong Vũ
- Cài đặt -> Nhà cung cấp -> Tạo mới

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh9.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

- Điền tên nhà cung cấp (Phong Vũ) -> Lưu

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh10.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## **6- Địa Phương**

Tạo đia chỉ công ty bạn, nếu có chi nhánh khác thì tạo thêm, hoặc dạng tập đoàn đứng tên nhiều công ty thì bạn tạo thêm công ty ở phần 1
- Cài đặt -> địa phương -> tạo mới

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh11.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

- Điền tên địa phương (Võ Văn Tần) -> Lưu

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh12.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr> 

## **7- Thành viên**

Về cơ bản thì tạo người dùng để gán tài sản là chính, các tính năng khác mình sẽ giới thiệu thêm sau

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh13.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

- Những thông tin có dấu cam phía sau là bắt buộc

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh14.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr>

## **8- Tạo tài sản**

Bắt đầu tạo tài sản dựa theo thông tin đã tạo.

Lưu ý:
- Thẻ tài sản bạn phải tư duy hoặc thống nhất theo chuẩn bên kế toán đã lưu để sau này kiểm kê trích xuất dữ liệu cho đồng bộ 2 bên dễ dàng kiểm tra.
- Kiểu tài sản bạn sẽ thấy có tùy chọn theo các thông tin đã tạo bên trên với logic: danh mục – hãng / kiểu tài sản (còn ở phần cài đặt – kiểu tài sản thì hiển thị sẽ không giống).
- Tình trạng: thì thường sẽ là ready to deploy do hiện tại chúng ta kiểm kê và nhập liệu cho tài sản đang sử dụng được, khác cái là bạn có check out vào đối tượng nào hay chưa – nếu chưa check out thì xem như đó là hàng tồn kho chờ được cấp, phần này bạn nhập chừng 10 20 thiết bị thì sẽ thấy được. Và bạn có thể tạo thêm nhiều tình trạng tài sản khác tùy theo nhu cầu của bạn tại Cài đặt -> tình trạng nhãn.
- Check out đến: thì thường bạn sẽ tạo danh sách thành viên cho toàn bộ nhân viên trong công ty rồi check out tài sản theo người dùng, hoặc tùy nhu cầu mà check out theo địa phương.
- Tên tài sản: ở đây bạn có thể ghi cho đầy đủ thông tin cơ bản của tài sản để nhìn vào biết cơ bản đó là món gì, thông tin này có thể giống nhau giữa các tài sản.
- Nên có hình ảnh cho từng tài sản để kiểm kê dễ hơn.

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh15.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr> 

Về cơ bản thì sau phần này thì các bạn đã nhập liệu cho tài sản, sau khi nhập thủ công để nắm cơ chế hoạt động thì bạn có thể đến bược tiếp theo xịn xò hơn, là import file vào hàng loạt với sample csv tải [tại đây](https://snipe-it.readme.io/docs/importing), hay xem report các kiểu.

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Huong-dan-su-dung-SnipeIT-p1/Hinh16.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>
<hr> 

Hướng dẫn sử dụng SnipeIT cơ bản đến đây gần như là tạm đủ để các bạn nhập liệu được tài sản của công ty để đưa vào thực tế.

Các thông tin trên mình điền là cơ bản, các bạn có thể điền thêm thông tin đầy đủ vào.

Phần sau mình sẽ giới thiệu thêm các tính năng advance hơn.

Nguồn: [ITFORVN](https://itforvn.com/snipeit-huong-dan-su-dung-p1-co-ban/)

---
