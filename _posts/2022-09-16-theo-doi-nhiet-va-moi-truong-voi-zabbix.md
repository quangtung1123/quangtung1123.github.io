---
layout: post
title:  "Theo dõi nhiệt độ và độ ẩm môi trường với Zabbix"
date:   2022-09-16 10:48:00
permalink: 2022/09/16/theo-doi-nhiet-va-moi-truong-voi-zabbix
tags: Monitoring Zabbix
category: Monitoring
img: /assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh1.jpg
summary: Theo dõi nhiệt độ và độ ẩm môi trường với Zabbix

---

Nhiệt độ và hơi ẩm là kẻ thù của mọi loại thiết bị điện tử, trong các phòng server thì 2 yếu tố này luôn cần phải được kiểm soát chặt chẽ để đảm bảo sự hoạt động ổn định của hệ thống. Trong bài viết này tôi xin giới thiệu phương án giám sát nhiệt độ và độ ẩm trong phòng thông qua cảm biến nhiệt độ và độ ẩm DHT22 bằng Zabbix agent chạy trên Raspberry PI.

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh1.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

## Theo dõi nhiệt độ và độ ẩm môi trường với Zabbix ##

Nội dung thực hiện:

1 – Kết nối module DHT22 với PI 3.

2 – Cài Raspbian lên thẻ nhớ và boot vào OS trên PI 3.

3 – Cài Zabbix Agent để trao đổi dữ liệu với server giám sát, Agent sẽ lấy dữ liệu thông qua python script (có sẵn trên github.com).

## Phần cứng cần thiết ##
Các linh kiện cần sử dụng hoàn toàn có thể mua trên các trang bán linh kiện điện tử ở Việt nam (ex: banlinhkien.vn…) Tổng chi phí khoảng 1,5 tr.

- Raspberry Pi 3
- Nguồn cho Raspberry Pi 3
- Cảm biến DHT22
- Thẻ nhớ Micro SD 4-8G
- Đầu đọc thẻ nhớ
(Thích thì mua case cho Pi3, có thể bắt vít lên tấm gỗ chạy ok :D)
OS: Raspbian Jessie Lite, Python 2.7 và zabbix agent 2.x

## Các bước thực hiện ##
**Bước 1: Tải hệ điều hành cho Raspbian Pi:**
Tải bản lite tại: https://www.raspberrypi.org/downloads/raspbian/

**Bước 2: Ghi hệ điều hành vào thẻ nhớ.**
Có thể dùng UtraIso, Win32DiskImager, để ghi vào thẻ nhớ.

Trên linux thì dùng lệnh dd để ghi.
```
unzip -p 2017-11-29-raspbian-stretch.zip | sudo dd of=/dev/sdX bs=4M conv=fsync
```

Trong đó sdX là thẻ nhớ được mount trong linux.

Sau ghi ghi xong, tháo thẻ nhớ và cắm vào Raspberry. Cắm nguồn máy tính, dây mạng vào Pi3 và bắt đầu chiến!

**Bước 3: Kết nối với Pi 3 qua SSH.**

Thời gian khởi động của Pi 3 khoảng 60s tùy tốc độ đọc của thẻ nhớ. Để kết nối với Pi 3 ta cần xác định IP cảu Pi 3 sau khi boot (có thể xem trên modem và so sách MAC, hoặc dùng nmap hoặc 1 phần mềm scan Ip nào đó) và kết nối qua SSH (trên windows dùng PUTTY), m ặc định username/password kết nối là username=pi password=raspberry

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Bước 4: Kết nối module cảm biến DHT22 với Pi 3**

Sau khi kết nối SSH thành công với Pi 3, cần kết nối module cảm biến với nó để hoạt động.

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh4.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

Cảm biến gồm 3 chân theo thứ tự như hình vẽ:

- Chân bên trái : + 3,3v => dây màu vàng
- Chân giữa: GPIO => dây màu dây màu nâu
- Chân bên phải: Mass => dây màu đen 

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh5.jpg" width = "800">
</div>
<div class="thecap"></div>
</div>

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh6.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Cách nối:
- Chân + 3,3v => dây vàng => Chân màu vàng bất kỳ trên PI 3.
- Chân GPIO => dây nâu => Chân số 2 trên PI 3.
- Chân Mass => dây đen =>  Chân màu đen trên PI 3.

**Bước 5: Cài phần mềm cần thiết trên PI 3.**

Thông qua SSH ta tiến hành update và cài các ứng dụng cần thiết như sau:
```
sudo apt-get update && sudo apt-get -y install git python-dev

git clone https://github.com/adafruit/Adafruit_Python_DHT.git

cd Adafruit_Python_DHT/

sudo python setup.py install
```

Các bước trên sẽ cài đặt đầy đủ phần mềm để kết nối PI với module cảm biến và nhận dữ liệu.

**Bước 6: Test cảm biến.**
Để kiểm tra cảm biến, sử dụng lệnh:

```
sudo /home/pi/Adafruit_Python_DHT/examples/AdafruitDHT.py 22 2
```

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh7.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Cấu trúc của lệnh và các tham số: tham số đầu là mã cảm biến (DHT22 => 22) tham số thứ 2 là thứ tự chân cắm GPIO trên board (số 2 như đã đề cập ở trên).

Vậy là cảm biến đã hoạt động tốt. Tiếp theo ta sẽ cấu hình zabbix agent đọc các tham số từ kết quả trên ra biến để truyền tới server. Để đơn giản tôi chỉ sử dụng zabbix_get để kiểm tra sự trả kết quả của agent. Nếu bạn muốn tham khảo cách tạo keyitem trên server và kết nối với Zabbix agent trên Pi3 xin đọc các bài viết trước.

**Bước 7: Cấu hình zabbix agent**

```
sudo apt-get -y install zabbix-agent
```

Sửa file config:

```
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Sửa giá trị Server=127.0.0.1 thành IP của zabbix server đồng thời thêm dòng sau vào file:

```
UserParameter=dht.pull[*],sudo /home/pi/Adafruit_Python_DHT/examples/AdafruitDHT.py 22 2 | awk -F[=*%] '{print '$'"$1"}'
```

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh8.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Save file lại và chạy zabbix agent:

```
sudo service zabbix-agent restart
```

Chú ý: Để zabbix có thể chạy lệnh lấy dữ liệu, ta phải cấp quyền execute cho zabbix với tập tin AdafruitDHT.py, có 2 cách:

Cách 1: cho phép zabbix user chạy script với quyền root: bằng cách sửa file sudo.conf:

```
sudo visudo
```

Sau đó thêm dòng này vào cuối file

```
zabbix ALL=(ALL) NOPASSWD: /home/pi/Adafruit_Python_DHT/examples/AdafruitDHT.py
```

Cách  2: Cấp thẳng quyền execute cho file bằng lệnh

```
sudo chmod + x/home/pi/Adafruit_Python_DHT/examples/AdafruitDHT.py
```

Bước 8: Kiểm tra zabbix thực hiện việc lấy dữ liệu từ cảm biến:
Như đã đề cập ở trên, tôi sẽ dùng zabbix_get để test.

```
sudo apt-get -y install zabbix-proxy-sqlite3
```

Kiểm tra với item key dht.pull đã cài đặt trong agent ở bước 7:

```
zabbix_get -s 127.0.0.1 -k dht.pull[2]
zabbix_get -s 127.0.0.1 -k dht.pull[4]
```

Tham số truyền vào là 2 => nhiệt độ, tham số truyền vào là 4 => độ ẩm.

<div class="imgcap">
<div >
    <img src="/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/Hinh9.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Vậy là xong, bạn có thể kết nối với server và tiến hành theo dõi thông qua zabbix, có thể lập graph để theo dõi cho đẹp.

## Tải template cho zabbix server##

[TẢI TEMPLATE DHT22](/assets/theo-doi-nhiet-va-moi-truong-voi-zabbix/template_dht22_module.zip)
