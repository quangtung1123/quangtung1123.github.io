---
layout: post
comments: true
title:  "DDOS - Chống ddos bằng Nginx"
title2:  "Chống ddos bằng Nginx"
date:   2021-07-08 21:45:00
permalink: 2021/07/08/Chong-ddos-bang-Nginx
mathjax: false
tags: Security DDOS Linux Nginx
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Chong-ddos-bang-Nginx/Hinh0.jpg
summary: Hướng dẫn chống ddos trên Linux
---

Ở bài Truy tìm nguyên nhân lỗi "[Error establishing a database connection](/2021/07/07/Tim-nguyen-nhan-loi-database-cua-WP)" của WordPress, chúng ta đã xác định chính xác nguyên nhân MySQL bị tắt do VPS hết RAM và OOM Killer đã kill dịch vụ MySQL. Đồng thời ta cũng xác định được website đang bị tấn công DDoS và lấy được log mẫu, bước tiếp theo sẽ tiến hành dùng nginx để chống tấn công DDoS này.

## **PHÂN TÍCH LOG TẤN CÔNG DDoS**
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Chong-ddos-bang-Nginx/Hinh1.png" width = "800">
</div>
<div class="thecap">Log tấn công ddos</div>
</div>
<hr>
Các điểm bất thường trong log:
- Các truy cập vào chung 1 URL có pattern là “/?add_to_wishlist=xxxx“
- Truy cập xuất phát từ nhiều IP khác nhau nhưng lại chung 1 User-Agent
- Cùng sử dụng HTTP/1.0
- Các IP trên khi truy cập URL KHÔNG load kèm theo các static resources như css, js

Có thể kết luận:
- Các truy cập trên được sinh ra từ cùng 1 công cụ tự động, được cài đặt ở nhiều máy tính khác nhau.
- Các truy cập trên không phải của người dùng hợp lệ và không sử dụng browser để truy cập (do sử dụng HTTP/1.0 và không load các static resource kèm theo)
- Đây là kiểu tấn công DDoS bằng botnet

## **CHỐNG TẤN CÔNG DDoS bằng Nginx**

Việc cần làm tiếp theo là chặn các truy cập không hợp lệ mang các đặc điểm trên. Để giảm thiểu tối đa ảnh hưởng đến người dùng hợp lệ khác, ta cần tạo ra một chữ ký nhận diện sao cho càng sát với đặc điểm của botnet càng tốt. Tôi chọn chữ ký bao gồm TẤT CẢ các đặc điểm sau:
- 1. Truy cập vào đúng URL /
- 2. Request có argument là  “add_to_wishlist” và theo sau đó là 1 dãy số.
- 3. Sử dụng User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36
- 4. Sử dụng HTTP/1.0
- 5. Sử dụng method: GET

Ta cần viết rule chặn các điều kiện riêng lẻ trước, đảm bảo chúng hoạt động đúng trước khi kết hợp toàn bộ lại với nhau. Hiện tại website đang sử dụng mô hình [Reverse Proxy](/2021/07/08/Chong-ddos-bang-Nginx) nên tôi sẽ tiến hành cấu hình chống tấn cống DDoS trên Nginx

### **1. Truy cập vào đúng URL /**

Để chỉ áp dụng filter lên riêng các truy cập chính xác đến URL /, ta khai báo location sử dụng exact match (dấu "="):
```
location = / { 
...
}
```

Các điều kiện số 2-5 sẽ được config bên trong location này.

### **2. Request có argument là  “add_to_wishlist” và theo sau là 1 dãy số**

Để kiểm tra argument “add_to_wishlist” có tồn tại hay không, và theo sau có phải là 1 dãy số, ta sử dụng regular expression như sau:
```
if ($arg_add_to_wishlist ~ "[0-9]+") {
    return 406;
}
```

Kiểm tra thử và đảm bảo điều kiện trên hoạt động đúng: Điều chỉnh file config virtualhost của website, thêm vào như sau:
```
location = / { 
    if ($arg_add_to_wishlist ~ "[0-9]+") {
        return 406;
    }
}
```
Dùng curl để kiểm tra, nếu filter hoạt động đúng sẽ trả về status code là 406 với các request dạng "*/?add_to_wishlis=xxxx*", với các request khác thì status code vẫn là 200 như bình thường:

```
# Request tấn công, sẽ trả vể status code 406
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/?add_to_wishlist=123' -I
HTTP/1.1 406 Not Acceptable
Server: nginx/1.16.1
Date: Wed, 29 Apr 2020 03:39:28 GMT
Content-Type: text/html
Content-Length: 163
Connection: keep-alive

# Request bình thường khác, trả về status code 200
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/' -I
HTTP/1.1 200 OK
Server: nginx/1.16.1
Date: Wed, 29 Apr 2020 04:48:53 GMT
Content-Type: text/html
Content-Length: 4833
Last-Modified: Fri, 16 May 2014 15:12:48 GMT
Connection: keep-alive
ETag: "53762af0-12e1"
Accept-Ranges: bytes
```

### **3. Request sử dụng User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36**

Để kiểm tra User-Agent ta sử dụng biến *$http_user_agent*. Điều chỉnh file config Virtual Host, thêm vào như sau:
```
location = / {
    if ($http_user_agent = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36") {
	    return 406;
    }
}
```

Dùng curl kiểm tra lại:
- Nếu request sử dụng User-Agent “Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36” sẽ trả về status code 406.
- Các User-Agent khác trả về status code 200 như bình thường
```
# Request sử dụng User-Agent phải trả về 406
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/' -I -H "User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36"
HTTP/1.1 406 Not Acceptable
Server: nginx/1.16.1
Date: Wed, 29 Apr 2020 03:46:05 GMT
Content-Type: text/html
Content-Length: 565
Connection: keep-alive

# Request sử dụng User-Agent khác: trả về 200
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/' -I -H "User-Agent: Iphone"
HTTP/1.1 200 OK
Server: nginx/1.16.1
Date: Wed, 29 Apr 2020 03:51:31 GMT
Content-Type: text/html
Content-Length: 4833
Last-Modified: Fri, 16 May 2014 15:12:48 GMT
Connection: keep-alive
ETag: "53762af0-12e1"
Accept-Ranges: bytes
```

Đã hoạt động chính xác như mong muốn.

### **4. Request sử dụng HTTP1/0:**

Để kiểm tra version HTTP, ta sử dụng biến "*$server_protocol*". Điều chỉnh file config Virtual Host và thêm vào như sau:
```
location = / { 
    if ($server_protocol = "HTTP/1.0") {
    	return 406;
    }
}
```

Kiểm tra, nếu request sử dụng HTTP/1.0 thì trả về status code 406, ngược lại trả về status code 200 như bình thường:
```
# Request sử dụng HTTP/1.0: trả về 406
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/' -I --http1.0
HTTP/1.1 406 Not Acceptable
Server: nginx/1.16.1
Date: Wed, 29 Apr 2020 03:52:12 GMT
Content-Type: text/html
Content-Length: 163
Connection: close

# Reques sử dụng HTTP/1.1: trả về 200
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/' -I --http1.1
HTTP/1.1 200 OK
Server: nginx/1.16.1
Date: Wed, 29 Apr 2020 04:57:35 GMT
Content-Type: text/html
Content-Length: 4833
Last-Modified: Fri, 16 May 2014 15:12:48 GMT
Connection: keep-alive
ETag: "53762af0-12e1"
Accept-Ranges: bytes
```

Đã hoạt động đúng như mong muốn.

### **5. Sử dụng method: GET**

Để kiểm tra request method, ta sử dụng biến “$request_method”, điều chỉnh file config thêm vào dòng sau:
```
location = / { 
    if ($request_method = "GET") {
    	return 406;
    }
}
```

### **6. Tổng hợp tất cả các điều kiện**

Các điều kiện riêng lẻ đã được kiểm tra và đảm bảo hoạt động đúng như mong muốn, ta cần kết hợp các điều kiện lại với nhau sao cho khi request chứa TẤT CẢ các đặc điểm trên sẽ bị DROP. 

Tuy nhiên, nginx không hỗ trợ toán tử AND để giúp ta AND các điều kiện lại với nhau, ta cần sử dụng trick như sau:
- Khai báo một biến $BLOCK với giá trị rỗng.
- Mỗi điều kiện match sẽ Append một giá trị vào biến $BLOCK
- Nếu kết quả cuối cùng của biến $BLOCK chứa đầy đủ các giá trị mong muốn thì DROP request đấy.

Cụ thể:
```
   location = / {
                set $BLOCK "";
                if ($arg_add_to_wishlist ~ "[0-9]+") {
                        set $BLOCK "${BLOCK}T";
                }

                if ($http_user_agent = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36") {
                        set $BLOCK "${BLOCK}R";
                }

                if ($server_protocol = "HTTP/1.0") {
                        set $BLOCK "${BLOCK}U";
                }

                if ($request_method = "GET") {
                        set $BLOCK "${BLOCK}E";
                }

                if ($BLOCK = "TRUE") {
                        return 444;
                }
                [...]
        }
```

Kiểm tra lại: 
```
[zero2hero@example.vn ~]$ curl 'https://blog.example.vn/?add_to_wishlist=123' -H "User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36" --http1.0
curl: (52) Empty reply from server
```

Khi sử dụng return 444, kết quả nhận được khi curl sẽ là "*curl: (52) Empty reply from server*", kết quả hoàn toàn đúng như ta mong đợi.

## **TỔNG KẾT**

1. Đến đây, ngoài việc xử lý vấn đề khách hàng đang gặp là lỗi “Error Establishing database connection”, ta cũng đã xác định được nguyên chính dẫn đến lỗi này và khắc phục để lỗi không lặp lại.

2. Cách chống tấn công DDoS hiện tại đang rất chuyên biệt cho riêng trường hợp này, nếu kẻ tấn công thay đổi một chút thì sẽ bypass được. Tuy nhiên, đó cũng là chủ ý của mình: khi chống tấn công DDoS chặn càng sát thì càng dễ bị bypass, đổi lại sẽ càng ít khách hàng hợp lệ bị chặn nhầm, đây là trade off tùy mỗi người lựa chọn. Vẫn có những cách chống tấn công DDoS khác tổng quan hơn, mình sẽ chia sẻ với các bạn trong một dịp khác.

Nguồn: [Vietnix](https://blog.vietnix.vn/chong-tan-cong-ddos-bang-nginx.html)

---
