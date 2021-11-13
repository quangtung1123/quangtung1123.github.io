---
layout: post
title:  "Máy chủ bị kiểm soát và tống tiền như thế nào?"
date:   2021-08-26 15:10:00
permalink: 2021/08/26/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien
tags: Security Malwares
category: Malwares
img: /assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/Hinh1.png
summary: Máy chủ bị kiểm soát và tống tiền như thế nào?

---

## HIỆN TRẠNG BAN ĐẦU ##

Chúng tôi nhận được yêu cầu hỗ trợ xử lý, điều tra máy chủ website bị tấn công Ransomware. Kẻ tấn công yêu cầu chuyển tiền vào ví BITCOIN của hắn để có thể chuộc lại dữ liệu đã bị mã hóa nhưng liệu sau khi chuyển tiền thì kẻ tấn công liệu có giải mã dữ liệu không? Chúng tôi đã xử lý nhiều trường hợp trước đây, có trường hợp chính kẻ tấn công cũng không thể giải mã được dữ liệu sau khi mã hóa mặc dù nạn nhân đã trả tiền. Vậy nếu dữ liệu thực sự rất quan trọng, không có dữ liệu backup thì bạn hãy yêu cầu kẻ tấn công giải mã trước một vài file trước xem có thực sự giải mã được không thì mới tính đến chuyện trả tiền.

Máy chủ bị nhiễm là Windows Server 2016 – RAM 64G, chạy dịch vụ web với IIS.

Khi tiếp nhận máy chủ thì tất cả các file dữ liệu (tài liệu, mã nguồn website, database,…) đều bị mã hóa. Mã độc không còn trên máy chủ, có thể kẻ tấn công đã xóa sau khi thực hiện mã hóa xong.

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W01.png" width = "800">
</div>
<div class="thecap"></div>
</div> 

## THU THẬP VÀ ĐIỀU TRA ##

Vậy kẻ tấn công đã xâm nhập máy chủ và phát tán mã độc ransomware như thế nào? Đây là câu hỏi mà bất cứ nạn nhân nào cũng muốn biết để có thể khắc phục sự cố, triển khai các giải pháp an toàn dữ liệu sau này.

***Khoanh vùng thời điểm tấn công***

Tất cả các file bị mã hóa ngày 14-03-2020 đây là mốc thời gian quan trọng để khoanh vùng thời gian xâm nhập của mã độc. Chúng tôi tiếp tục xem tất cả các sự kiện của máy chủ thông qua logs của Windows và IIS với mốc thời gian trên trở về trước.

***Thu thập logs máy chủ***

Chúng tôi thu thập được một vài logs powershell quan trọng theo mốc thời gian khoanh vùng như sau:

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W02.png" width = "800">
</div>
<div class="thecap">Kẻ tấn công tải xuống file ngrok.exe và lưu vào thư mục “C:\Users\Public\”</div>
</div> 

Đây là một file “sạch”, đây là công cụ mà kẻ tấn công sử dụng tạo ra secure tunnel để remote remote desktop, tại sao chúng tôi lại kết luận như vậy thì sẽ giải thích thêm ở phần dưới, tìm hiểu thêm công cụ này tại đây (https://ngrok.com/docs).

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W03.png" width = "800">
</div>
<div class="thecap">Kẻ tấn công tải xuống file Encrypt.exe và lưu vào thư mục “C:\Users\Public\”</div>
</div> 

Theo như chính tên của nó, đây là mã độc ransomware có hành vi mã hóa dữ liệu.

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W13.png" width = "800">
</div>
<div class="thecap"></div>
</div> 

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W04.png" width = "800">
</div>
<div class="thecap"></div>
</div> 

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W05.png" width = "800">
</div>
<div class="thecap">Điều tra thêm thì trong khoảng thời gian lây nhiễm thì có một user “Mamad” được tạo mới</div>
</div> 

Sau đó là event Remote Deskop với user Mamad, để ý thời gian tạo account là “11:13:59 AM 13-03-2020” ngay sau khi tải file Ngrok.exe, Có thể thấy kẻ tấn công sử dụng Ngrok.exe để Remote Desktop. Đây là chiêu để truy cập vào máy chủ khi dịch vụ Remote Desktop trên máy chủ Web đã bị Firewall chặn.

Tiếp tục đào thêm dữ liệu thì thấy dịch vụ web đã bị hacker dò tìm lỗi để tấn công từ ngày 5-03-2020

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W06.png" width = "800">
</div>
<div class="thecap"></div>
</div> 

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W07.png" width = "800">
</div>
<div class="thecap"></div>
</div> 

Dựa vào request thì có thể thấy kẻ tấn công đang chạy công cụ rò quét lỗ hổng tự động, chúng tôi tìm thêm logs IIS xem có bị mã hóa không, rất may logs IIS vẫn còn nguyên, đây là bước tiến rất quan trọng trong việc xác định xem kẻ tấn công đã khai thác lỗ hổng nào để xâm nhập vào máy chủ.

***Phân tích logs IIS***

Dựa vào các mốc thời gian đã thu thập ở trên thì chúng tôi phát hiện rất nhiều request/payload tấn công được gửi lên máy chủ web như sau

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W08.png" width = "800">
</div>
<div class="thecap">Các request này thường được các công cụ tìm kiếm lỗ hổng tự động sinh ra, có thể thấy ngày 05-03-2020 chính là ngày đầu tiên kẻ tấn công thu thập thông tin web server</div>
</div> 

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W09.png" width = "800">
</div>
<div class="thecap">Các ngày sau đó cũng ghi nhận rất nhiều payload tấn công khác đa dạng khác (trong hình là payload SQLi)</div>
</div> 

Sau một thời gian tìm kiếm thì chúng tôi phát hiện kẻ tấn công đã upload được webshell lên máy chủ web thông qua lỗ hổng của **Telerik Web UI**.

**Telerik Web UI** ứng dụng web trên Framework ASP.NET rất phổ biến và được triển khai nhiều ở cơ quan chính phủ, doanh nghiệp.

Trên ứng dụng tồn tại 2 lỗ hổng nghiêm trọng cho phép kẻ tấn công tải xuống, upload tệp bất kỳ hoặc có thể thực thi mã từ xa trên máy chủ web, cụ thể đó là 2 CVE: CVE-2017-9248, CVE-2019-18935, cả 2 CVE đều đã public mã khai thác.

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W11.png" width = "800">
</div>
<div class="thecap">Kẻ tấn công chạy rất nhiều payload khai thác lỗ hổng của Telerik, dựa vào đặc điểm của payload thì đây là payload khai thác CVE-2017-9248, CVE-2019-18935. Sau khi khai thác thành công thì kẻ tấn công có thể thực thi shellcode, upload webshell (Goz.aspx à trả về mã 200 cho thấy kẻ tấn công đã upload thành công webshell).</div>
</div> 

<div class="imgcap">
<div >
    <img src="/assets/dieu-tra-may-chu-bi-kiem-soat-va-tong-tien/W12.png" width = "800">
</div>
<div class="thecap">Một vài request dùng Webshell mà kẻ tấn công để lại</div>
</div> 

## TIMELINE TẤN CÔNG ##

05-03-2020: Bắt đầu dò quét lỗ hổng trên dịch vụ web.

12-03-2020: Khai thác thành công lỗ hổng Telerik và upload được webshell.

13-03-2020:

- Thực hiện chạy powershell để download file Ngrok.exe.
- Thực hiên tạo account “Mamad”.
- Thực hiện hành vi Remote Desktop với user vừa tạo.

14-03-2020:

- Thực hiện chạy powershell để download file Encrypt.exe để mã hóa dữ liệu.
- Mã hóa xong, kẻ tấn công tiến hành xóa dấu vết (webshell, ransomware).
 

## KẾT LUẬN ##

Máy chủ web bị kẻ tấn công phát tán ransomware thông qua lỗ hổng web, cụ thể là Telerik Web UI (CVE-2017-9248, CVE-2019-18935) cho phép kẻ tấn công chạy shellcode, upload webshell nếu khai thác thành công.

## KHUYẾN CÁO ##

- Cập nhật các bản vá Hệ điều hành và dịch vụ web thường xuyên.

- Triển khai các giải pháp an toàn, an ninh dữ liệu

- Backup dữ liệu thường xuyên.

**Tài liệu tham khảo**

- [cyradar](https://cyradar.com/2020/03/30/dieu-tra-may-chu-1-tap-doan-bi-kiem-soat-va-tong-tien-nhu-the-nao/)

---
