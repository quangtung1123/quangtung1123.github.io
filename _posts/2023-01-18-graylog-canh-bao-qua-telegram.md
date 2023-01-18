---
layout: post
title:  "Graylog cảnh báo qua Telegram"
date:   2023-01-18 10:48:00
permalink: 2023/01/18/graylog-canh-bao-qua-telegram
tags: Log Graylog
category: Log
img: /assets/graylog-canh-bao-qua-telegram/Hinh1.jpg
summary: Graylog cảnh báo qua Telegram

---

## Cài đặt ##
- Tải file plugin mới nhất tại: https://github.com/irgendwr/TelegramAlert/releases/latest
- Copy file telegram-notification-x.x.x.jar vừa tải được vào thư mục /usr/share/graylog-server/plugin/ trên máy chủ Graylog
- Khởi động lại dịch vụ graylog server
```
systemctl restart graylog-server
```

## Tạo Bot trên Telegram ##

- Truy cập vào link dưới để tạo Bot bằng BotFather
```
    https://t.me/BotFather
```
- Lần lượt nhập 2 lệnh sau:
```
    /start
    /newbot
```

<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_1.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Đặt tên cho Bot và username cho Bot. Sau đó bạn sẽ nhận được token API. Hãy lưu lại token này để sử dụng cho bước sau này.
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_3-2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Tạo một Channel mới trên Telegram
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_4-2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Đặt tên cho Channel và viết mô tả. Sau đó click **CREATE**
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_5-1.png" width = "800">
</div>
<div class="thecap"></div>

- Add Bot vào Channel để test. Chọn mục 'Add members'
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_6-2.png" width = "800">
</div>
<div class="thecap"></div>

- Tìm kiếm tên Bot và add.
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_7-2.png" width = "800">
</div>
<div class="thecap"></div>

- Truy cập vào URL bên dưới để test bot có hoạt động hay không. Chú ý: bạn cần thay thế `<BOT_TOKEN>` bằng token mà bạn nhận được ở bước tạo bot lúc đầu
```
    https://api.telegram.org/bot<BOT_TOKEN>/getUpdates
```
- Nếu trình duyệt trả về kết quả như dưới là đã thành công
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_8-3.png" width = "800">
</div>
<div class="thecap"></div>

- Quay lại Channel trên Telegram nhập thử một vài dòng.
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_10-2.png" width = "800">
</div>
<div class="thecap"></div>

- Xem trên trình duyệt và kiểm tra kết quả trả về đã OK.
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_11-2-1024x124.png" width = "800">
</div>
<div class="thecap"></div>

## Cấu hình trên Web interface của graylog server ##

- Vào mục **Alerts** -> **Notifications** và chọn **Create Notifications**
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_12-2.png" width = "800">
</div>
<div class="thecap"></div>

- Điền thông tin cho Notification
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/H1.png" width = "800">
</div>
<div class="thecap"></div>

<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/H2.png" width = "800">
</div>
<div class="thecap"></div>

- Tiếp tục sử dụng thông tin đã được cấp để điền vào các trường. Lưu ý: – **Chat IDs** bạn lấy từ url test lúc trước (https://api.telegram.org/bot/getUpdates).
- Sau đó bạn có thể test Notification bằng cách click vào **Execute Test Notification**. Nếu kết quả trả về là Success thì ta đã thành công.

- Sau khi chọn **Create** ta sẽ nhận được một Notification mới
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_15-2.png" width = "800">
</div>
<div class="thecap"></div>

***Lưu ý: nếu trong Notification type không tìm thấy Telegram Notification thì thực hiện khởi động lại máy chủ Graylog***

## Cấu hình thử event SSH và gửi qua Telegram ##
- Tiếp tục cấu hình bên tab **Event Definitions** và click vào **Edit** để chỉnh sửa
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_20.png" width = "800">
</div>
<div class="thecap"></div>

- Chuyển đến mục **Notifications** để thêm Noti vừa tạo lúc trước
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_17-2.png" width = "800">
</div>
<div class="thecap"></div>

- Sau đó chọn **Next** để chuyển sang mục **Summary**. Cuối cùng là chọn **Done** để kết thúc quá trình cài đặt.
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_19-2.png" width = "800">
</div>
<div class="thecap"></div>

## Test kết quả ##

Thử đăng nhập sai 5 lần trong 5 phút và check tin nhắn gửi về trên Telegram. Nếu kết quả giống như bên dưới thì tức là ta đã cấu hình thành công.
<div class="imgcap">
<div >
    <img src="/assets/graylog-canh-bao-qua-telegram/Screenshot_22-2.png" width = "800">
</div>
<div class="thecap"></div>

## Cập nhật plugin ##
Khi có phiên bản mới của plugin, ta thực hiện cập nhật như sau:
- Xóa file plugin cũ graylog-plugin-telegram-notification-x.x.x.jar (hoặc telegram-alert-x.x.x.jar ) khỏi máy chủ graylog
- Cái đặt plugin mới tương tự hướng dẫn phía trên
- Xóa Telegram Notifications cũ và tạo lại trên web quản trị graylog.

**Tài liệu tham khảo**
- [graylog](https://community.graylog.org/t/telegram-notification/22693)
- [cloud365](https://news.cloud365.vn/graylog-lab-phan-10-cau-hinh-graylog-server-tich-hop-canh-bao-qua-telegram/)