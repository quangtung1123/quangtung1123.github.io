---
layout: post
title:  "Cài đặt Restyaboard trên Windows sử dụng IIS"
permalink: 2021/10/11/install-restyaboard-in-windows-using-iis
tags: Restyaboard
category: Restyaboard
img: /assets/install-restyaboard-in-windows-using-iis/Hinh1.png
summary: Cài đặt Restyaboard trên Windows sử dụng IIS

---

## Giới thiệu

Bạn đang quản lý một nhóm nhỏ và bạn không biết họ đã làm gì và tiến độ đến đâu? Do đó, bạn cần một phần mềm quản lý công việc để có thể quản lý những vấn đề đó. Hôm nay, mình sẽ giới thiệu cho các bạn phần mềm quản lý công việc Restyaboard để các bạn tham khảo.

Restyaboard là một phần mềm mã nguồn mở, sẽ giúp bạn dễ dàng quản lý công việc của mình và công việc của các thành viên trong nhóm, cộng thêm với các tính năng bổ sung thông minh như đồng bộ hóa ngoại tuyến, khác biệt/sửa đổi, nhận xét lồng nhau, nhiều bố cục xem, trò chuyện và hơn thế nữa. Và vì nó được cài đặt trên hệ thống của bạn nên dữ liệu, quyền riêng tư và bảo mật có thể được đảm bảo.

Restyaboard giống như một ghi chú dán điện tử để sắp xếp các công việc và việc cần làm. Ngoài ra, nó còn lý tưởng cho bảng Kanban, Agile, Gemba và quản lý quy trình kinh doanh/quy trình làm việc.

## Các bước để cài đặt

Bạn cần phải tải và cài đặt các phần mềm phụ trợ sau:
- [Microsoft Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx)
- [NodeJS / NPM](https://nodejs.org)
- [PostgreSQL](https://www.postgresql.org/download)

Các bước cài đặt bao gồm:
- Tải Restyaboard
- Kiểm tra phân quyền thư mục IIS
- Cài đặt các phần mềm phụ thuộc và biên dịch less/js (npm/grunt)
- Cài đặt PHP và PostgreSQL
- Kiểm tra các tiện ích mở rộng của PHP
- Thiết lập Cơ sở dữ liệu Restyaboard
- Thiết lập một trang web mới trong IIS
- Cấu hình file web.config

Nào chúng ta bắt đầu thực hiện cài đặt Restyaboard trên Windows với IIS:

## Tải Restyaboard

- Tạo thư mục mới để lưu trữ phần mềm Restyaboard trên hệ thống của bạn. Ví dụ:
	```C:\inetpub\wwwroot\restyaboard```
- Tải phiên bản mới nhất board-vX.X.X.zip [tại đây](https://github.com/RestyaPlatform/board/releases)
- Giải nén file ZIP vào thư mục tạo phía trên


## Kiểm tra phân quyền thư mục IIS

- Trong Windows Explorer, nhấp chuột phải vào thư mục cài đặt phía trên của bạn và chọn  **Properties**. 
- Chọn ***Security Tab***
- Trong hộp *Group or user names*, chọn **Edit**
- Nếu **IIS_IUSRS** và **IUSR** không có trong danh sách, hãy thêm chúng.
- Chọn **Add**
- Nhập tên tài khoản (như đã viết ở trên), chọn **Check Names**, chọn **OK**.
- Đánh dấu vào ô *Allow* với **full control.** và chọn **OK**.

## Cài đặt các phần mềm phụ thuộc và biên dịch less/js

Mở Command Prompt:
- Di chuyển đến thư mục cài đặt: ```cd C:\inetpub\wwwroot\restyaboard``` 
- Nếu bạn chưa cài đặt ``grunt-cli`` hoặc bạn không chắc chắn, hãy nhập như sau:
   ```npm install -g grunt-cli```
- Gõ các lệnh sau, nhấn enter sau mỗi lệnh:
    ``npm install``
    ``grunt less``
    ``grunt jst``
> **Lưu ý:** chỉ sử dụng lệnh 'grunt' là đủ, nhưng tôi đã gặp sự cố với nó khi cài đặt các gói liên quan đến eslint, hãy làm như trên nếu bạn gặp phải vấn đề tương tự.
 
## Cài đặt PHP và PostgreSQL

> Nếu PHP và PostgreSQL đã được cài đặt trên hệ thống của bạn thì bạn có thể bỏ qua bước này.

#### PHP v5.6+
- Tải xuống và cài đặt NodeJS nếu bạn chưa có
- Sử dụng Web Platform Installer, hãy cài đặt những gói sau:
   - PHP 5.6.31
- Nó sẽ tự động đề nghị cài đặt những thứ sau:
   - Windows Cache Extension 1.3 for PHP 5.3
   - Microsoft Drivers 3.2 for PHP v5.6 for SQL Server in IIS 
   
#### PostgreSQL 10
- Tải xuống và cài đặt PostgreSQL bằng trình cài đặt được EnterpriseDB chứng nhận [tại đây](https://www.postgresql.org/download/windows/).

## Kiểm tra các tiện ích mở rộng của PHP

Restyaboard sử dụng một vài phần mở rộng PHP quan trọng để giao tiếp với cơ sở dữ liệu PSQL. Di chuyển đến thư mục cài đặt PHP của bạn (có thể ``C:\Program Files (x86)\PHP\v5.6``) và chỉnh sửa tệp cấu hình php.ini của bạn. Bạn có thể cần mở Notepad hoặc một trình soạn thảo khác với tư cách Quản trị viên. Ở cuối tệp, hãy đảm bảo bạn có phần mở rộng  được in đậm dưới đây:

\[ExtensionList\]

extension=php_mysql.dll

extension=php_mysqli.dll

extension=php_mbstring.dll

**extension=php_gd2.dll

**extension=php_ldap.dll**

extension=php_gettext.dll

**extension=php_curl.dll**

extension=php_exif.dll

extension=php_xmlrpc.dll

extension=php_openssl.dll

extension=php_soap.dll

extension=php_pdo_mysql.dll

extension=php_pdo_sqlite.dll

**extension=php_imap.dll**

extension=php_tidy.dll

**extension=php_pgsql.dll**

**extension=php_pdo_pgsql.dll**

## Thiết lập Cơ sở dữ liệu Restyaboard
- Mở pgAdmin (đã cài đặt với PostgreSQL phía trên)
  > Bạn sẽ được nhắc đăng nhập bằng mật khẩu bạn đã đặt trong khi cài đặt
- Mở rộng nút PostgreSQL 10
- Nhấp chuột phải vào **Databases**, di chuột qua *Create* và chọn '**Database**'
- Nhập tên của cơ sở dữ liệu là '*restyaboard*' và nhấp vào Save
- Nhấp chuột phải vào **Login/Group Roles**, di chuột qua *Create* và chọn "**Login/Group Role**"
- Trong tab *general*, hãy cung cấp tên ``restya``
- Trong tab *definition*, hãy cung cấp mật khẩu ``hjVl2!rGd``
- Nhấp chuột phải vào cơ sở dữ liệu '*restyaboard*' đã tạo, và chọn '**Query Tool**'
- Dán những dòng sau:

 ```
CREATE USER restya WITH PASSWORD 'hjVl2!rGd';
ALTER DATABASE restyaboard OWNER TO restya;
GRANT ALL PRIVILEGES ON DATABASE restyaboard TO restya;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO restya;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO restya;
```

> Sau khi dán ở trên, nhấp vào biểu tưởng tam giác sáng đậm hoặc nhấn F5 để thực hiện.
- Chọn mở biểu tượng (thư mục) trong *Query Tool* và nhấp đúp qua các thư mục hệ thống tệp của bạn để tìm tệp SQL từ bản cài đặt Restyaboard đã tải xuống của bạn. Ví dụ như:
``c:\inetpub\wwwroot\restyaboard\sql\restyaboard_with_empty_data.sql``
Nhấp vào biểu tưởng tam giác sáng đậm hoặc nhấn F5 để thực hiện.


## Thiết lập một trang web mới trong IIS

- Mở **IIS Manager**
- Chuột phải vào *Sites* và chọn **Add Web Site**
- Đặt tên theo ý muốn của bạn
- Đối với đường dẫn vật lý, hãy điều hướng đến vị trí thư mục chứa phần mềm Restyaboard.
     
## Cấu hình file web.config

Thêm tệp web.config với nội dung phía dưới vào thư mục chứa phần mềm Restyaboard:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <clear />
                <rule name="iCAL URLs" stopProcessing="true">
                    <match url="^/?ical/([0-9]*)/([0-9]*)/([a-zA-Z0-9]*).ics$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/server/php/ical.php?id={R:1}&amp;user_id={R:2}&amp;hash={R:2}" appendQueryString="false" />
                </rule>
                <rule name="Download URLs" stopProcessing="true">
                    <match url="^/?download/([0-9]*)/([a-zA-Z0-9_\.]*)$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
                    <action type="Rewrite" url="/server/php/download.php?id={R:1}&amp;hash={R:2}" />
                </rule>
                <rule name="oAuth Callback" stopProcessing="true">
                    <match url="^/?oauth_callback/([a-zA-Z0-9_\.]*)/([a-zA-Z0-9_\.]*)$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/server/php/oauth_callback.php?plugin={R:1}&amp;code={R:2}" appendQueryString="false" />
                </rule>
                <rule name="oAuth Authorize">
                    <match url="^/?oauth/authorize$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/server/php/authorize.php" />
                </rule>
                <rule name="API URLs" stopProcessing="true">
                    <match url="^/?api/(.*)$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
                    <action type="Rewrite" url="/server/php/R/r.php?_url={R:1}&amp;" />
                </rule>
                <rule name="Site Root to Client" stopProcessing="true">
                    <match url="^$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
                    <action type="Rewrite" url="client/" />
                </rule>
                <rule name="CSS, JS, IMG, Font, Apps, Locales" stopProcessing="false">
                    <match url="^/?(css|js|img|font|apps|locales)/(.*)$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/client/{R:1}/{R:2}" />
                </rule>
                <rule name="Image URLs" stopProcessing="true">
                    <match url="^/?client/img/([a-zA-Z_]*)/([a-zA-Z_]*)/([a-zA-Z0-9_\.]*)\??(.*)?$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/server/php/image.php?size={R:1}&amp;model={R:2}&amp;filename={R:3}" appendQueryString="false" />
                </rule>
                <rule name="Favicon" stopProcessing="true">
                    <match url="^/?favicon.ico$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/client/favicon.ico" appendQueryString="false" />
                </rule>
                <rule name="Manifest" stopProcessing="true">
                    <match url="^/?manifest.json$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/client/manifest.json" appendQueryString="false" />
                </rule>
                <rule name="Apple Touch Icon" stopProcessing="true">
                    <match url="^/?apple-touch-icon(.*)$" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
                    <action type="Rewrite" url="/client/apple-touch-icon{R:1}" appendQueryString="false" />
                </rule>
            </rules>
        </rewrite>
        <httpProtocol>
            <customHeaders>
                <add name="Cache-Control" value="public, must-revalidate" />
            </customHeaders>
        </httpProtocol>
        <staticContent>
            <mimeMap fileExtension=".json" mimeType="application/json" />
        </staticContent>
        <handlers>
            <remove name="PHP_via_FastCGI" />
            <remove name="WebDAV" />
            <remove name="ExtensionlessUrlHandler-Integrated-4.0" />
            <remove name="OPTIONSVerbHandler" />
            <remove name="TRACEVerbHandler" />
            <add name="ExtensionlessUrlHandler-Integrated-4.0" path="*." verb="*" type="System.Web.Handlers.TransferRequestHandler" preCondition="integratedMode,runtimeVersionv4.0" />
            <add name="PHP_via_FastCGI" path="*.php" verb="GET,HEAD,POST,PUT,DELETE" modules="FastCgiModule" scriptProcessor="C:\Program Files (x86)\PHP\v5.6\php-cgi.exe" resourceType="Either" requireAccess="Script" />
        </handlers>
        <modules>
            <remove name="WebDAVModule" />
        </modules>
    </system.webServer>
</configuration>
```

Vui lòng xem nhanh các thẻ thêm và xóa cho PHP bên dưới nút trình xử lý trước khi tiếp tục. Mô-đun PHP của bạn CÓ THỂ được gọi một cái gì đó khác. Tùy thuộc vào hệ thống của bạn, vị trí cài đặt của php-cgi.exe cũng có thể khác nhau. Chỉ cần kiểm tra xem những điều trên có khớp với những gì có trên hệ thống của bạn hay không trước khi tiếp tục.

> Bạn có thể kiểm tra tên của trình xử lý PHP của mình bằng cách điều hướng đến trang web của bạn trong IIS Manager và nhấp đúp vào Handler Mappings. Bạn cũng có thể chỉnh sửa mục nhập cho PHP bằng IIS Manager tại đây nếu bạn muốn. Về cơ bản, bạn cần đảm bảo rằng nó đang cho phép các hành động PUT và DELETE.
 
Nói một cách rộng rãi, tệp web.config ở trên sao chép những gì tệp .htaccess thực hiện được cho restyaboard trong cài đặt apache. Nhưng nó cũng giảm thiểu một số vấn đề đã biết với cài đặt IIS.
+ WebDAV đôi khi ngăn các API RESTful nhận các yêu cầu PUT và DELETE
+ Các hành động PUT và DELETE cần được cho phép trên một vài trình xử lý có liên quan để API của Restyaboard hoạt động. API được sử dụng cho gần như mọi tác vụ không đồng bộ trên giao diện người dùng, vì vậy điều quan trọng là nó phải hoạt động.

## Kiểm tra Restyaboard

Sử dụng trình duyệt web của bạn, điều hướng đến vị trí thích hợp. Hy vọng rằng bạn sẽ thấy một màn hình đăng nhập Restyaboard. Thông tin đăng nhập quản trị viên mặc định là ``admin`` và ``restya``.

## *Lưu ý:* Nâng cấp

Nâng cấp có thể được thực hiện bằng cách ghi đè thư mục cài đặt bằng các tệp từ bản phát hành mới nhất và cập nhật cơ sở dữ liệu bằng cách sử dụng tập lệnh SQL thích hợp. Sau khi sao chép các tệp mới, hãy chạy lại ``npm install`` và ``grunt`` và sử dụng pgAdmin để mở tập lệnh nâng cấp cơ sở dữ liệu. Các tập lệnh này nằm trong ``restyaboard/sql``

**Tài liệu tham khảo**

- [restya](https://restya.com/board/docs/windows-iis/)

---
