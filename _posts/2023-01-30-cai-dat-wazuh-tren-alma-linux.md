---
layout: post
title:  "Cài đặt Wazuh trên Alma Linux"
date:   2023-01-30 10:48:00
permalink: 2023/01/30/cai-dat-wazuh-tren-alma-linux
tags: Log Wazuh
category: Log
img: /assets/cai-dat-wazuh-tren-alma-linux/Hinh1.jpg
summary: Cài đặt phần mềm giám sát log tập trung Wazuh trên Alma Linux

---

Cài đặt Wazuh lần lượt theo quy trình như sau:
- Wazuh Indexer
- Wazuh Server
- Wazuh Dashboard

## Wazuh Indexer ##

Wazuh Indexer có thể được cài đặt dưới dạng một nút đơn hoặc dưới dạng cụm nhiều nút.

**Đề xuất phần cứng cho mỗi nút**

|---------------+----------+---------+---------+-----------|
|  | Tối thiểu |  | Khuyến khích |  |
|:----------------:|:----------------:|:---------:|:---------:|:---------:|
| Thành phần | RAM (GB) | CPU (lõi) | RAM (GB) | CPU (lõi) |
| Wazuh Indexer | 4 | 2 | 16 | 8 |

**Yêu cầu về dung lượng ổ đĩa**

- Lượng dữ liệu phụ thuộc vào các cảnh báo được tạo mỗi giây (APS). Bảng này nêu chi tiết dung lượng ổ đĩa ước tính cần thiết cho mỗi tác nhân để lưu trữ 90 ngày cảnh báo trên máy chủ bộ chỉ mục Wazuh, tùy thuộc vào loại thiết bị được giám sát.

|---------------+----------+--------------------|
| Tối thiểu | APS | Storage in Wazuh indexer (GB/90 days) |
|:----------------:|:----------------:|:------------------:|
| Servers | 0.25 | 3.7 |
| Workstations | 0.1 | 1.5 |
| Network devices | 0.5 | 7.4 |

- Ví dụ: đối với môi trường có 80 máy trạm, 10 máy chủ và 10 thiết bị mạng, dung lượng lưu trữ cần thiết trên máy chủ bộ chỉ mục Wazuh trong 90 ngày có cảnh báo là 230 GB.

**Cài đặt cụm Wazuh Indexer**

Quá trình cài đặt được chia thành ba giai đoạn.
- Cấu hình ban đầu
- Cài đặt nút chỉ mục Wazuh
- Khởi tạo cụm

**Cấu hình ban đầu**

- Tạo chứng chỉ SSL để mã hóa thông tin liên lạc giữa các thành phần Wazuh và tạo mật khẩu ngẫu nhiên để bảo mật cài đặt của bạn.
- Tải xuống trợ lý cài đặt Wazuh và tệp cấu hình.
```
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.3/config.yml
```

- Chỉnh sửa file ./config.yml và thay thế tên nút và giá trị IP bằng tên và địa chỉ IP tương ứng. Bạn cần làm điều này cho tất cả các nút máy chủ Wazuh, bộ chỉ mục Wazuh và bảng điều khiển Wazuh. Thêm bao nhiêu trường nút nếu cần.

```
nodes:
  # Wazuh indexer nodes
  indexer:
    - name: node-1
      ip: <indexer-node-ip>
    #- name: node-2
    #  ip: <indexer-node-ip>
    #- name: node-3
    #  ip: <indexer-node-ip>

  # Wazuh server nodes
  # If there is more than one Wazuh server
  # node, each one must have a node_type
  server:
    - name: wazuh-1
      ip: <wazuh-manager-ip>
    #  node_type: master
    #- name: wazuh-2
    #  ip: <wazuh-manager-ip>
    #  node_type: worker
    #- name: wazuh-3
    #  ip: <wazuh-manager-ip>
    #  node_type: worker

  # Wazuh dashboard nodes
  dashboard:
    - name: dashboard
      ip: <dashboard-node-ip>
```

- Chạy file *wazuh-install.sh* với tùy chọn ```--generate-config-files``` để tạo khóa cụm Wazuh, chứng chỉ và mật khẩu cần thiết để cài đặt. Bạn có thể tìm thấy các tệp này ở trong ./wazuh-install-files.tar.

```
bash wazuh-install.sh --generate-config-files
```

- Sao chép file *wazuh-install-files.tar* vào tất cả các máy chủ triển khai phân tán, bao gồm máy chủ Wazuh, bộ chỉ mục Wazuh và các nút bảng điều khiển Wazuh. Điều này có thể được thực hiện bằng cách sử dụng tiện ích scp.

**Cài đặt các nút chỉ mục Wazuh**

- Tải xuống file cài đặt Wazuh.

```
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
```

- Chạy trợ lý với tùy chọn *--wazuh-indexer* và tên nút để cài đặt và định cấu hình bộ chỉ mục Wazuh. Tên nút phải giống với tên được sử dụng config.yml cho cấu hình ban đầu, ví dụ: node-1.

*Ghi chú Đảm bảo rằng một bản sao của wazuh-install-files.tar, được tạo trong bước cấu hình ban đầu, được đặt trong thư mục làm việc của bạn.*
```
bash wazuh-install.sh --wazuh-indexer node-1
```

Lặp lại giai đoạn này của quá trình cài đặt cho mọi nút bộ chỉ mục Wazuh trong cụm của bạn. Sau đó, tiến hành khởi tạo cụm một nút hoặc nhiều nút của bạn trong bước tiếp theo.

**Khởi tạo cụm**

- Giai đoạn cuối cùng của việc cài đặt cụm nút đơn hoặc đa nút của trình chỉ mục Wazuh bao gồm chạy tập lệnh quản trị bảo mật.
- Chạy trợ lý cài đặt Wazuh với tùy chọn *--start-cluster* trên bất kỳ nút bộ chỉ mục Wazuh nào để tải thông tin chứng chỉ mới và khởi động cụm.
```
bash wazuh-install.sh --start-cluster
```

*Ghi chú Bạn chỉ phải khởi tạo cụm một lần , không cần chạy lệnh này trên mọi nút.*

**Kiểm tra cài đặt cụm**

- Chạy lệnh sau để lấy mật khẩu quản trị viên:
```
tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "\'admin\'" -A 1
```

- Chạy lệnh sau để xác nhận rằng quá trình cài đặt thành công. Thay thế <ADMIN_PASSWORD>bằng mật khẩu nhận được từ đầu ra của lệnh trước đó. Thay thế <WAZUH_INDEXER_IP>bằng địa chỉ IP của trình chỉ mục Wazuh đã định cấu hình:

```
curl -k -u admin:<ADMIN_PASSWORD> https://<WAZUH_INDEXER_IP>:9200

Output
{
  "name" : "node-1",
  "cluster_name" : "wazuh-cluster",
  "cluster_uuid" : "cMeWTEWxQWeIPDaf1Wx4jw",
  "version" : {
    "number" : "7.10.2",
    "build_type" : "rpm",
    "build_hash" : "e505b10357c03ae8d26d675172402f2f2144ef0f",
    "build_date" : "2022-01-14T03:38:06.881862Z",
    "build_snapshot" : false,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

- Thay thế <WAZUH_INDEXER_IP> và <ADMIN_PASSWORD>, và chạy lệnh sau để kiểm tra xem cụm có hoạt động bình thường không:

```
curl -k -u admin:<ADMIN_PASSWORD> https://<WAZUH_INDEXER_IP>:9200/_cat/nodes?v
```

Trình chỉ mục Wazuh hiện đã được cài đặt thành công và tiếp theo bạn có thể tiến hành cài đặt máy chủ Wazuh.

## Wazuh Server ##

Bạn có thể cài đặt Wazuh Server trên một máy chủ duy nhất. Ngoài ra, bạn có thể cài đặt nó được phân phối trong nhiều nút trong cấu hình cụm. Cấu hình nhiều nút cung cấp tính khả dụng cao và hiệu suất được cải thiện. Và nếu được kết hợp với bộ cân bằng tải mạng thì có thể sử dụng hiệu quả dung lượng của nó.

**Đề xuất phần cứng cho mỗi nút**

|---------------+----------+---------+---------+-----------|
|  | Tối thiểu |  | Khuyến khích |  |
|:----------------:|:----------------:|:---------:|:---------:|:---------:|
| Thành phần | RAM (GB) | CPU (lõi) | RAM (GB) | CPU (lõi) |
| Wazuh Server | 2 | 2 | 4 | 8 |

**Yêu cầu về dung lượng ổ đĩa**

- Lượng dữ liệu phụ thuộc vào các cảnh báo được tạo mỗi giây (APS). Bảng này nêu chi tiết dung lượng ổ đĩa ước tính cần thiết cho mỗi tác nhân để lưu trữ 90 ngày cảnh báo trên máy chủ bộ chỉ mục Wazuh, tùy thuộc vào loại thiết bị được giám sát.

|---------------+----------+--------------------|
| Tối thiểu | APS | Storage in Wazuh indexer (GB/90 days) |
|:----------------:|:----------------:|:------------------:|
| Servers | 0.25 | 0.1 |
| Workstations | 0.1 | 0.04 |
| Network devices | 0.5 | 0.2 |

- Ví dụ: đối với môi trường có 80 máy trạm, 10 máy chủ và 10 thiết bị mạng, dung lượng lưu trữ cần thiết trên máy chủ bộ chỉ mục Wazuh trong 90 ngày có cảnh báo là 6GB.

Để xác định xem máy chủ Wazuh có cần thêm tài nguyên hay không, hãy theo dõi các tệp sau:

- /var/ossec/var/run/wazuh-analysisd.state: biến events_droppedcho biết liệu các sự kiện có bị hủy do thiếu tài nguyên hay không.

- /var/ossec/var/run/wazuh-remoted.state: biến discarded_countcho biết liệu tin nhắn từ các tác nhân có bị loại bỏ hay không.

Hai biến này phải bằng 0 nếu môi trường hoạt động bình thường. Nếu không, các nút bổ sung có thể được thêm vào cụm.

**Cài đặt cụm Wazuh Server**

- Tải xuống trợ lý cài đặt Wazuh.
```
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
```

- Chạy trợ lý với tùy chọn *--wazuh-server* theo sau là tên nút để cài đặt máy chủ Wazuh. Tên nút phải giống với tên được sử dụng config.yml cho cấu hình ban đầu, ví dụ: wazuh-1.

*Ghi chú Đảm bảo rằng một bản sao của tệp wazuh-install-files.tar, được tạo trong bước cấu hình ban đầu, được đặt trong thư mục làm việc của bạn.*

```
bash wazuh-install.sh --wazuh-server wazuh-1
```

- Máy chủ Wazuh của bạn hiện đã được cài đặt thành công.

- Nếu bạn muốn có một cụm nút đơn máy chủ Wazuh, mọi thứ đã được đặt và bạn có thể tiến hành trực tiếp với Cài đặt bảng điều khiển Wazuh bằng trợ lý .

- Nếu bạn muốn có một cụm nhiều nút máy chủ Wazuh, hãy lặp lại quy trình này trên mọi nút máy chủ Wazuh.

Quá trình cài đặt máy chủ Wazuh hiện đã hoàn tất và tiếp theo bạn có thể tiến hành cài đặt bảng điều khiển Wazuh. 

## Wazuh Dashboard ##

Thành phần trung tâm này là một giao diện web linh hoạt và trực quan để khai thác, phân tích và trực quan hóa dữ liệu bảo mật. Nó cung cấp bảng điều khiển vượt trội, cho phép bạn điều hướng liền mạch qua giao diện người dùng.

Với bảng điều khiển Wazuh, người dùng có thể trực quan hóa các sự kiện bảo mật, lỗ hổng được phát hiện, dữ liệu giám sát tính toàn vẹn của tệp, kết quả đánh giá cấu hình, sự kiện giám sát cơ sở hạ tầng đám mây và các tiêu chuẩn tuân thủ quy định.

**Đề xuất phần cứng cho mỗi nút**

|  | Tối thiểu |  | Khuyến khích |  |
| Thành phần | RAM (GB) | CPU (lõi) | RAM (GB) | CPU (lõi) |
| Wazuh Server | 4 | 2 | 8 | 4 |

**Cài đặt bảng điều khiển Wazuh**

- Tải xuống trợ lý cài đặt Wazuh. Có thể bỏ qua bước này nếu bạn đã cài đặt bộ chỉ mục Wazuh trên cùng một máy chủ.
```
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
```
- Chạy trợ lý với tùy chọn *--wazuh-dashboard* và tên nút để cài đặt và định cấu hình bảng điều khiển Wazuh. Tên nút phải giống với tên được sử dụng config.yml cho cấu hình ban đầu, ví dụ: dashboard.

*Ghi chú Đảm bảo rằng một bản sao của wazuh-install-files.tartệp, được tạo trong bước cấu hình ban đầu, được đặt trong thư mục làm việc của bạn.*

```
bash wazuh-install.sh --wazuh-dashboard dashboard
```

Sau khi trợ lý hoàn tất quá trình cài đặt, đầu ra sẽ hiển thị thông tin đăng nhập truy cập và thông báo xác nhận rằng quá trình cài đặt đã thành công.

```
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
   User: admin
   Password: <ADMIN_PASSWORD>

INFO: Installation finished.
```

- Bây giờ bạn đã cài đặt và định cấu hình Wazuh. Tất cả các mật khẩu được tạo bởi trợ lý cài đặt Wazuh có thể được tìm thấy trong file *wazuh-passwords.txt* bên trong kho lưu trữ wazuh-install-files.tar. Để in chúng, hãy chạy lệnh sau:

```
tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

- Truy cập giao diện web Wazuh bằng thông tin đăng nhập của bạn.
```
URL: https://<wazuh-dashboard-ip>

Tên người dùng : quản trị viên

Mật khẩu : <ADMIN_PASSWORD>
```

- Khi bạn truy cập bảng điều khiển Wazuh lần đầu tiên, trình duyệt sẽ hiển thị thông báo cảnh báo cho biết chứng chỉ không được cấp bởi cơ quan đáng tin cậy. Một ngoại lệ có thể được thêm vào trong các tùy chọn nâng cao của trình duyệt web. Để tăng cường bảo mật, thay vào đó, tệp root-ca.pem được tạo trước đó có thể được nhập vào trình quản lý chứng chỉ của trình duyệt. Ngoài ra, có thể định cấu hình chứng chỉ từ cơ quan đáng tin cậy.

Tất cả các thành phần trung tâm Wazuh đã được cài đặt thành công.

**Tài liệu tham khảo**
- [wazuh](https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html)