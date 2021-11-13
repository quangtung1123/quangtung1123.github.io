---
layout: post
title:  "Cài đặt phiên bản VMware Tanzu Community miễn phí"
permalink: 2021/11/12/install-the-free-new-vmware-tanzu-community-edition
tags: VMware Tanzu Community
category: VMware
img: /assets/install-the-free-new-vmware-tanzu-community-edition/Hinh1.png
summary: Cài đặt phiên bản VMware Tanzu Community miễn phí

---

VMware đã phát hành một phiên bản Tanzu mới được gọi là Tanzu Community Edition. Phiên bản này là bản tải xuống miễn phí mà bạn có thể cài đặt và sử dụng trên một số máy Windows, Linux và macOS.


Sau khi cài đặt môi trường cơ sở (Tanzu CLI), bạn có thể triển khai Tanzu Community Edition từ xa cho môi trường thử nghiệm, có thể là VMware vSphere, Amazon, Azure hoặc Docker.

Các tùy chọn triển khai được trình bày thông qua giao diện người dùng (UI), nơi bạn sẽ theo dõi một trợ lý sẽ tạo tệp cấu hình. Việc cài đặt từ xa chỉ đơn giản dựa trên tệp cấu hình đó.

## VMware Tanzu Community Edition là gì? ##

Đây là một nền tảng đầy đủ tính năng cho những người dùng muốn học cách quản lý nền tảng Kubernetes. Đây là phiên bản miễn phí của Tanzu, mã nguồn mở và được cộng đồng hỗ trợ mà bạn có thể cài đặt trên máy tính xách tay hoặc máy trạm của mình và sau đó triển khai đến một vị trí từ xa.

Bạn có thể tạo các cụm Kubernetes thông qua Cluster API, là một dự án con của Kubernetes cung cấp các API khai báo và các công cụ để đơn giản hóa việc cung cấp, nâng cấp và vận hành nhiều cụm Kubernetes. Hệ thống cho phép bạn sắp xếp khối lượng công việc và cài đặt các gói nền tảng hỗ trợ các ứng dụng chạy theo cụm.

Bạn có thể tải xuống bộ tệp từ VMware [tại đây](https://tanzucommunityedition.io/download/) . Chỉ cần chọn phiên bản bạn muốn, tùy thuộc vào hệ điều hành bạn đang làm việc. Bạn có thể sử dụng máy tính xách tay Linux / MAC hoặc máy trạm Windows của mình; có các tệp khác nhau cho mỗi nền tảng.

Trang GitHub có thể được tìm thấy [ở đây](https://github.com/vmware-tanzu/community-edition).

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh1.png" width = "800">
</div>
<div class="thecap">Các phiên bản VMware Tanzu Community</div>
</div>

## Tôi có thể cài đặt VMware Tanzu Community Edition ở đâu? ##

Tanzu Community Edition bao gồm Tanzu CLI và một tập hợp các plugin chọn lọc được sử dụng để triển khai từ xa giải pháp cho nền tảng bạn chọn.

Sau khi tải xuống, bạn sẽ cài đặt Tanzu Community Edition trên máy trạm cục bộ của mình và sau đó sử dụng Tanzu CLI để triển khai một cụm cho nền tảng mục tiêu đã chọn của bạn, có thể là VMware vSphere, Amazon AWS, Microsoft Azure hoặc Docker.

Máy cục bộ của bạn thường được gọi là máy bootstrap của bạn và quá trình triển khai một cụm được gọi là bootstrapping.

## Làm cách nào để cài đặt VMware Tanzu Community Edition? ##

Tùy thuộc vào môi trường cục bộ của bạn, bạn có thể muốn làm theo [quy trình cài đặt](https://tanzucommunityedition.io/docs/latest/cli-installation/) cho nền tảng của mình. Trong bài đăng này, chúng tôi sẽ tập trung vào môi trường Windows.

Đảm bảo đọc các hướng dẫn và yêu cầu hệ thống. Máy bên dưới sẽ sử dụng WSL2 hoặc Hyper-V, tùy thuộc vào hệ thống của bạn.

Đối với máy tính để bàn Windows, hãy đảm bảo rằng nó đáp ứng các yêu cầu sau:
- Windows 11 64-bit: Home hoặc Pro phiên bản 21H2 trở lên hoặc Enterprise hoặc Education phiên bản 21H2 trở lên.
- Windows 10 64-bit: Home hoặc Pro 2004 (bản dựng 19041) trở lên hoặc Enterprise hoặc Education 1909 (bản dựng 18363) trở lên.
- Bật tính năng **WSL 2** trên Windows. Để biết hướng dẫn chi tiết, hãy tham khảo tài liệu của Microsoft.
- Các điều kiện tiên quyết về phần cứng sau đây là bắt buộc để chạy thành công WSL 2 trên Windows 10 hoặc Windows 11:
  - Bộ xử lý 64-bit với Dịch địa chỉ mức thứ hai (SLAT)
  - RAM hệ thống 4 GB
  - Hỗ trợ ảo hóa phần cứng cấp BIOS phải được bật trong cài đặt BIOS. Để biết thêm thông tin, hãy xem phần Ảo hóa.
- Tải xuống và cài đặt [gói cập nhật nhân Linux](https://docs.microsoft.com/windows/wsl/wsl2-kernel).

Lưu ý: Hỗ trợ ảo hóa phần cứng cấp BIOS phải được bật trong cài đặt BIOS. Nếu bạn đang triển khai từ một máy ảo (như tôi, từ máy ảo W10), bạn sẽ cần phải đi tới cài đặt máy ảo và kích hoạt công cụ ảo hóa **Virtualize Intel VT-x / EPT hoặc AMD-V / RVI** . Ảnh chụp màn hình từ phần mềm VMware Workstation.

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh2.png" width = "800">
</div>
<div class="thecap">Cài đặt CPU VMware Workstation VM</div>
</div>

Chúng tôi sẽ sử dụng trình quản lý gói Chocolatey để triển khai Tanzu Community Edition. Tuy nhiên, bạn không cần phải sử dụng Chocolatey. Bạn chỉ có thể sử dụng tệp install.bat để cài đặt cục bộ. Trong trường hợp này, vui lòng sử dụng bảng điều khiển PowerShell để cài đặt.

Lệnh dưới đây sẽ tải xuống, cài đặt và đặt đường dẫn Windows cho Chocolatey.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Sau đó, sử dụng lệnh này để cài đặt Tanzu Community Edition.

```powershell
choco install tanzu-community-edition --version=0.9.1
```

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh3.png" width = "800">
</div>
<div class="thecap">Sử dụng Chocolatey để cài đặt Tanzu Community Edition</div>
</div>


Như bạn có thể thấy, chúng tôi thiếu hai điều kiện tiên quyết khác:
- **Docker** (tải Docker desktop cho Windows [tại đây](https://docs.docker.com/desktop/windows/install/) ) —Xem yêu cầu hệ thống [tại đây](https://docs.docker.com/desktop/windows/install/).
- **Kubectl** (xem hướng dẫn về kubectl [tại đây](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/) ) — Kubectl kiểm soát trình quản lý cụm Kubernetes.

Cài đặt Kubectl qua lệnh này thông qua Chocolatey:

```powershell
choco install kubernetes-cli
```

Trình cài đặt sẽ tải xuống các gói từ vị trí từ xa.

Theo tài liệu, kubectl cần được thêm vào đường dẫn người dùng để hoạt động.

Trong trường hợp của tôi, tôi chỉ cần tải xuống tệp kubectl và đặt nó trong một thư mục có tên kubectl trên ổ đĩa c: của tôi. (Tải xuống bản phát hành mới nhất v1.22.0 [tại đây](https://dl.k8s.io/release/v1.22.0/bin/windows/amd64/kubectl.exe).)

Sau đó, tôi tạo một biến môi trường. Chỉ cần đi tới **Advance System settings > Environment Variables** và chỉnh sửa path bằng cách thêm "C:\kubectl".

Mở một Windows command prompt và nhập lệnh ***kubectl*** để xem liệu môi trường có phản hồi chính xác hay không.

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh4.png" width = "800">
</div>
<div class="thecap">Cấu hình KubeCTL trên Windows</div>
</div>

Hãy theo dõi sát trang tài liệu vì có các bước khác và mẹo khắc phục sự cố không được trình bày chi tiết ở đây.

Sau đó, bạn sẽ cài đặt Docker desktop. Đảm bảo đọc các hướng dẫn và yêu cầu cài đặt chi tiết. Trong trường hợp của tôi, mất khá nhiều thời gian để khởi tạo.

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh5.png" width = "800">
</div>
<div class="thecap">Cài đặt Docker Desktop</div>
</div>

Có một yêu cầu cuối cùng mà chúng tôi cần thực hiện vì nếu không, chúng tôi sẽ nhận được thông báo lỗi cho biết rằng chứng chỉ x509 được ký bởi một cơ quan không xác định khi sử dụng Tanzu Community Edition (TCE) trên Windows. Đó là một lỗi.

Đây là thông báo lỗi:

*Downloading TKG compatibility file from 'projects.registry.vmware.com/tkg/framework-zshippable/tkg-compatibility'*

*Error: unable to create Tanzu Standalone Cluster client*

*Cause: unable to ensure prerequisites: unable to ensure tkg BOM file: failed to download TKG compatibility file from the registry: failed to list TKG compatibility image tags: Get "https://projects.registry.vmware.com/v2/": x509: certificate signed by unknown authority.*

Chúng tôi cần thêm một đoạn mã vào tệp cấu hình YAML, bạn có thể tìm thấy tệp này tại đây:

*%USERPROFILE%\.config\tanzu\tkg\config.yaml*

Mở tệp YAML và thêm thông tin này:

```
release:
version: ""
TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY: true
```

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh6.png" width = "800">
</div>
<div class="thecap">Ví dụ về cấu hình tệp YAML</div>
</div>

Sau khi hoàn thành, chúng ta đã hoàn thành tất cả các yêu cầu và có thể bắt đầu.

Mở cửa sổ PowerShell với quyền Admin của bạn và nhập lệnh để triển khai cụm Tanzu cục bộ:

```powershell
tanzu standalone-cluster create --ui
```

Hệ thống nên tạo một cụm độc lập và mở trang khởi chạy chính thông qua trình duyệt web, hiển thị cho bạn các tùy chọn triển khai. Như bạn có thể thấy, Docker, vSphere Amazon EC2 và Microsoft Azure hiện được hỗ trợ dưới dạng vị trí.

<div class="imgcap">
<div >
    <img src="/assets/install-the-free-new-vmware-tanzu-community-edition/Hinh7.png" width = "800">
</div>
<div class="thecap">Tùy chọn triển khai VMware Tanzu</div>
</div>

VMware nói rằng cách dễ nhất để triển khai môi trường phát triển là cung cấp một cụm "độc lập", nhưng bạn cũng có thể tạo một cụm quản lý.

Tanzu Community Edition sẽ yêu cầu bạn cung cấp một số thông tin đầu vào, chẳng hạn như số lượng nút, thông tin xác thực và ủy quyền cung cấp các nút đó. Cuối cùng, bạn sẽ kết thúc với cụm Kubernetes được triển khai.

Chúng tôi sẽ không đi sâu vào những chi tiết đó trong bài đăng này mà chỉ nêu bật trường hợp sử dụng quan trọng nhất cho Tanzu Community Edition. Trong nhiều trường hợp, nhu cầu về môi trường học tập và nhà phát triển là lý do số một cho việc triển khai Kubernetes. Đây cũng là mục tiêu của VMware - phổ biến Tanzu trong cơ sở người dùng của họ. Môi trường phát triển và thử nghiệm rất có thể là trường hợp sử dụng đầu tiên.

## Lời cuối ##

[VMware Tanzu Community Edition](https://tanzu.vmware.com/content/blog/vmware-tanzu-community-edition-announcement) là một sản phẩm đóng gói sẵn rất đẹp cho phép bạn tìm hiểu hoặc thiết lập môi trường phát triển. Nó chủ yếu dành cho các môi trường quy mô nhỏ hoặc tiền sản xuất.

Sản phẩm được tải về và sử dụng miễn phí; tuy nhiên, nó không thích hợp để sử dụng trong môi trường sản xuất. Bạn có thể triển khai nó từ bất kỳ hệ điều hành nào đến một vị trí từ xa, có thể là VMware vSphere, Amazon AWS, Microsoft Azure hoặc Docker.

VMware cung cấp tài liệu tốt cho các nền tảng khác nhau. Nếu bạn gặp lỗi, luôn có một diễn đàn hoặc bài đăng trên blog để giúp bạn.

**Tài liệu tham khảo**

- [4sysops](https://4sysops.com/archives/install-the-free-new-vmware-tanzu-community-edition/)

---
