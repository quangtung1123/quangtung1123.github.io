---
layout: post
title:  "Giám sát Cassandra sử dụng Zabbix"
date:   2023-03-07 10:48:00
permalink: 2023/03/07/giam-sat-cassandra-su-dung-zabbix
tags: Monitoring Zabbix Cassandra
category: Monitoring
img: /assets/giam-sat-cassandra-su-dung-zabbix/Hinh1.jpg
summary: Giám sát Cassandra sử dụng Zabbix

---

Các bước cụ thể như sau:

## Cài đặt Zabbix Java Gateway ##

```
sudo dnf install zabbix-java-gateway
```

- Cấu hình Zabbix Java Gateway (sửa file /etc/zabbix/zabbix_java_gateway.conf) theo một số tham số sau:

```
sudo vi /etc/zabbix/zabbix_java_gateway.conf

# IP address that Zabbix Java Gateway listens on (for Zabbix Server connection)
LISTEN_IP="127.0.0.1"

# Port number that Zabbix Java Gateway listens on
LISTEN_PORT=10052

# Number of worker threads that Zabbix Java Gateway starts
START_POLLERS=5

# uncomment to enable remote monitoring of the standard JMX objects on the Zabbix Java Gateway itself
JAVA_OPTIONS="$JAVA_OPTIONS -Dcom.sun.management.jmxremote 				-Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.registry.ssl=false"

```

- Cấu hình Zabbix server (sửa file /etc/zabbix/zabbix_server.conf) để kết nối đến Zabbix Java Gateway:

```
sudo vi /etc/zabbix/zabbix_server.conf

# IP address or host name where Zabbix Java Gateway is installed
JavaGateway=127.0.0.1

# port number that Zabbix Java Gateway listens on
JavaGatewayPort=10052

# the number of process handlers within Zabbix server that are
# started to poll from Zabbix Java Gateway threads
StartJavaPollers=5

```

Lưu ý: cấu hình ở trên chỉ dành cho việc định cấu hình kết nối giữa Zabbix Java Gateway và máy chủ Zabbix. Kết nối giữa các nguồn JMX metrics và Zabbix Java Gateway được định cấu hình thông qua giao diện người dùng (UI) Web Zabbix.

Sau khi thực hiện thay đổi cấu hình, hãy khởi động lại Zabbix Server và Zabbix Java Gateway:

```
sudo systemctl restart zabbix-java-gateway
sudo systemctl restart zabbix-server
```

## Import template giám sát ##
- Thực hiện tài template giám sát Cassandra trên zabbix và import vào zabbix để giám sát.


**Tài liệu tham khảo**
- [pythian](https://blog.pythian.com/monitor-cassandra-using-zabbix/)
