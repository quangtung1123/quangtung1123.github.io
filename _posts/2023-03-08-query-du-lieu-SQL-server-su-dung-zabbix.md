---
layout: post
title:  "Query dữ liệu của SQL Server sử dụng Zabbix"
date:   2023-03-08 10:48:00
permalink: 2023/03/08/query-du-lieu-SQL-server-su-dung-zabbix
tags: Monitoring Zabbix SQL
category: Monitoring
img: /assets/query-du-lieu-SQL-server-su-dung-zabbix/Hinh1.jpg
summary: Query dữ liệu của SQL Server sử dụng Zabbix

---

Các bước cụ thể như sau:

## Cài đặt Microsoft ODBC driver cho SQL Server trên Linux ##

- Thực hiện chạy lệnh tương ứng với hệ điều hành và phiên bản SQL server:

```

#Download appropriate package for the OS version
#Choose only ONE of the following, corresponding to your OS version

#RHEL 7 and Oracle Linux 7
sudo curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo

#RHEL 8 and Oracle Linux 8
sudo curl https://packages.microsoft.com/config/rhel/8/prod.repo > /etc/yum.repos.d/mssql-release.repo

#RHEL 9
sudo curl https://packages.microsoft.com/config/rhel/9.0/prod.repo > /etc/yum.repos.d/mssql-release.repo

sudo dnf remove unixODBC-utf16 unixODBC-utf16-devel #to avoid conflicts
sudo ACCEPT_EULA=Y dnf install -y msodbcsql18
# optional: for bcp and sqlcmd
sudo ACCEPT_EULA=Y dnf install -y mssql-tools18
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bashrc
source ~/.bashrc
# optional: for unixODBC development headers
sudo dnf install -y unixODBC unixODBC-devel
```


## Tạo kết nối tới cơ sở dữ liệu SQL Server ##

- Chỉnh sửa file ```/etc/odbc.ini``` và tạo DSN với các tham số tương ứng với SQL Server của bạn:

```
sudo vi /etc/odbc.ini

# [DSN name]
[MSSQLTest]  
Driver = ODBC Driver 18 for SQL Server  
# Server = [protocol:]server[,port]  
Server = tcp:localhost,1433
Encrypt = no
#
# Note:  
# Port isn't a valid keyword in the odbc.ini file  
# for the Microsoft ODBC driver on Linux or macOS
#
```

- Để xác minh xem kết nối ODBC có hoạt động thành công hay không, nên kiểm tra kết nối với cơ sở dữ liệu. Điều đó có thể được thực hiện với tiện ích ```isql``` (có trong gói unixODBC):

```
isql DSN_Name user_sql pass_sql


+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| echo [string]                         |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>

```

## Cấu hình Item trên Zabbix ##

- Sử dụng db.odbc.get\[<unique short description>,<dsn>,<connection string>\] để biến đổi kết quả truy vấn SQL thành một mảng JSON:

<div class="imgcap">
<div >
    <img src="/assets/query-du-lieu-SQL-server-su-dung-zabbix/Hinh1.png" width = "800">
</div>
<div class="thecap"></div>
</div>


- Sử dụng JSONPath để tách giá trị cần kiểm tra:

<div class="imgcap">
<div >
    <img src="/assets/query-du-lieu-SQL-server-su-dung-zabbix/Hinh2.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Chạy thử để kiểm tra kết quả:

<div class="imgcap">
<div >
    <img src="/assets/query-du-lieu-SQL-server-su-dung-zabbix/Hinh3.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Tạo trigger để cảnh báo khi vượt ngưỡng:

<div class="imgcap">
<div >
    <img src="/assets/query-du-lieu-SQL-server-su-dung-zabbix/Hinh4.png" width = "800">
</div>
<div class="thecap"></div>
</div>

**Tài liệu tham khảo**
- [microsoft](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver16&tabs=redhat18-install%2Calpine17-install%2Cdebian8-install%2Credhat7-13-install%2Crhel7-offline)
