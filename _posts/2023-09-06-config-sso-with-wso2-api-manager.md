---
layout: post
title:  "Cấu hình Single Sign On với WS02 API Manager"
date:   2023-09-06 08:00:00
permalink: 2023/09/06/config-sso-with-wso2-api-manager
tags: SSO WS02
category: SSO
img: /assets/config-sso-with-wso2-api-manager/saml-sso-scenario-diagram.png
summary: Cấu hình Single Sign On với WS02 API Manager

---

## Cấu hình Single Sign On WSO2 cho ứng dụng SAML ##
Có hai ứng dụng web SAML được gọi là pickup-dispatch và pickup-manager. Khi SSO được cấu hình cho cả hai ứng dụng này, nhân viên chỉ cần yêu cầu cung cấp thông tin xác thực của họ cho ứng dụng đầu tiên và người dùng sẽ tự động đăng nhập vào ứng dụng thứ hai theo mô hình sau:

<div class="imgcap">
<div >
    <img src="/assets/config-sso-with-wso2-api-manager/saml-sso-scenario-diagram.png" width = "800">
</div>
<div class="thecap"></div>
</div>

### Khai báo service providers ###
- Đăng nhập vào Management Console (https://<IS_HOST>:<PORT>/carbon);
- Vào menu Main > Identity > Service Providers và click Add;
- Nhập saml2-web-app-pickup-dispatch trong ô Service Provider Name và click Register.
- Trong Inbound Authentication Configuration, click Configure ở dưới SAML2 Web SSO Configuration;
- Nhập các trường thông tin sau:
	- Issuer: saml2-web-app-pickup-dispatch.com
	- Assertion Consumer URL: http://localhost.com:8080/saml2-web-app-pickup-dispatch.com/home.jsp
- Click Yes.
- Chọn các Checkbox sau:
	- Enable Response Signing
	- Enable Single Logout
	- Enable Attribute Profile
	- Include Attributes in the Response Always
	- Enable Signature Validation in Authentication Requests and Logout Requests

- Click Register để lưu lại.
- Tạo thêm service Provider mới cho Pickup Manager tương tự như trên với các thông tin như sau:
	- Name: saml2-web-app-pickup-manager
	- Issuer : saml2-web-app-pickup-manager.com
	- Assertion Consumer URL : http://localhost.com:8080/saml2-web-app-pickup-manager.com/home.jsp

### Triển khai các ứng dụng mẫu ###
- Cài đặt Apache Tomcat 8.x hoặc XAMPP để sử dụng Tomcat;
- Tải về các mẫu saml2-web-app-pickup-manager.com.war và saml2-web-app-pickup-dispatch.com.war tại https://github.com/wso2/samples-is/releases
- Giải nén file \*.war phía trên vào thư mục webapps của Tomcat
- Vào thư mục .../WEB-INF/classes và thay đổi các thuộc tính từ 'https://localhost:9443' sang link thực tế của WSO2 trong file sso.properties
- Khởi động lại Tomcat và truy cập ứng dụng http://localhost:8080/saml2-web-app-pickup-dispatch.com/index.jsp

### Cấu hình xác nhận quyền truy cập (claims) ###
Có thể cấu hình thêm xác nhận quyền truy cập như sau:
- Trong Menu chính của management console, click Service Providers>List, và Edit "pickup-dispatch" service provider;
- Mở rộng khối Claim Configuration trong service provider form.
- Lựa chọn claims muốn gửi đến service provider. Chọn **Use Local Claim Dialect** và click Add Claim URI.
- Thêm các thông tin Requested Claims như sau:
	- http://wso2.org/claims/fullname
	- http://wso2.org/claims/emailaddress
- Chọn http://wso2.org/claims/fullname trong Subject claim URI và click Update để lưu cấu hình service provider.

<div class="imgcap">
<div >
    <img src="/assets/config-sso-with-wso2-api-manager/dispatch-configure-claims.png" width = "800">
</div>
<div class="thecap"></div>
</div>

- Truy cập http://localhost:8080/saml2-web-app-pickup-dispatch.com trên trình duyệt và click Login.

<div class="imgcap">
<div >
    <img src="/assets/config-sso-with-wso2-api-manager/dispatch-email-consent.png" width = "800">
</div>
<div class="thecap"></div>
</div>

Nếu hiện ra bảng xác nhận như trên thì đã cấu hình thành công các yêu cầu bổ sung cho nhà cung cấp dịch vụ của mình.


## Tham khảo ##
- https://dzone.com/articles/configuring-sso-using-wso2-identity-server
