---
layout: post
title:  "Chặn các cuộc tấn công brute force Remote Desktop bằng Windows PowerShell"
date:   2022-02-15 15:10:00
permalink: 2022/02/15/block-brute-force-remote-desktop-attacks-with-windows-powershell
tags: Security Windows
category: Security
img: /assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh1.png
summary: Chặn các cuộc tấn công brute force Remote Desktop bằng Windows PowerShell

---

Vào đỉnh điểm của đại dịch Covid-19, nhiều công ty đã chuyển sang môi trường làm việc từ xa. Điều này dẫn đến sự gia tăng các cuộc tấn công brute force RDC, cái mà có thể ngăn chặn quyền truy cập vào các phiên máy tính từ xa và cuối cùng dẫn đến một mạng bị xâm phạm. Mặc dù hầu hết các tổ chức sử dụng thiết bị bảo mật để phát hiện và chặn các cuộc tấn công này, nhưng không phải tổ chức nào cũng đủ khả năng mua giấy phép do chi phí đắt đỏ của các thiết bị bảo mật.

Tập lệnh PowerShell này đếm số lượng địa chỉ IP trong nhật ký tường lửa Windows Defender mà đang cố gắng kết nối qua Remote Desktop. Nếu có hơn 10 lần thử cùng địa chỉ IP trong một khoảng thời gian nhất định (5 phút), thì tập lệnh PowerShell sẽ ghi lại địa chỉ IP đó, biến nó thành mạng con /16 và thêm (các) mạng con vào quy tắc tường lửa để chặn các nỗ lực kết nối từ các mạng con đó.

Thông thường, các tác nhân đe dọa bắt đầu các loại tấn công brute force này sẽ thay đổi địa chỉ IP nhiều lần, vì vậy điều quan trọng là toàn bộ mạng con bị từ chối truy cập đến.

Hãy xem tập lệnh PowerShell:

```powershell
# RDC Brute Force Prevention
# .
# .

# Copy firewall log file
Copy-Item -LiteralPath C:\Windows\System32\LogFiles\Firewall\pfirewall.log -Destination C:\Windows\System32\LogFiles\Firewall\pfirewall-source.log

# Declare variables
$FirewallLog = 'C:\Windows\System32\LogFiles\Firewall\pfirewall-source.log'
$FirewallRule = Get-NetFirewallRule -DisplayName "RDC Brute Force Prevention"
$CSVHeader = 'Date', 'Time', 'Action', 'Protocol', 'SourceIP', 'DestIP', 'SourcePort', 'DestPort'
$CSVFile = Import-Csv -Delimiter ' ' -Path $FirewallLog -Header $CSVHeader
$Date = Get-Date -DisplayHint Date -Format "yyyy-MM-dd"
$Time = Get-Date -DisplayHint Time -Format "HH:mm:ss"
$TimeSub5 = (Get-Date -DisplayHint Time).AddMinutes(-5)
$Obj = New-Object PSObject
$Subnets = @()
$NewScope = @()

# Add firewall rule (if it doesn't already exist)
If ($FirewallRule -EQ $null) {
    # Firewall rule does not exist
    New-NetFirewallRule -DisplayName "RDC Brute Force Prevention" -Direction Inbound -Action Block -RemoteAddress '255.255.255.254'
} else {
    # Do nothing; firewall rule exists
}

# Clear existing subnets from firewall rule after 24 hours
If ($Time -LIKE "00:00:**") {
    # It's midnight
    Set-NetFirewallRule -DisplayName "RDC Brute Force Prevention" -RemoteAddress '255.255.255.254'
} else {
    # Do nothing
}

# Import CSV | Skip the first 5 lines | Criteria: 
## - Date is today
## - Time is between the last 5 minutes and now
## - Source IP is not the host
## - Destination port is 3389
## - Count the number of IPs trying to connect to 3389 in the last 5 minutes
$CSVFile | Select -Skip 5 | Where {$_.Date -EQ $Date -and $_.Time -GE $TimeSub5.ToString("HH:mm:ss") -and $_.SourceIP -NE '127.0.0.1' -and $_.DestPort -EQ '3389'} | Group-Object -Property SourceIP -NoElement | Where {$_.Count -GT '5'} | Sort-Object -Descending | % {
    # Convert counted IPs to subnets and add to $Subnets array
    ForEach-Object {
        $IPAddress = "$($_.Name)".ToString()
        $Byte = $IPAddress.Split(".")
        $Subnet = ($Byte[0] + "." + $Byte[1] + ".0.0/255.255.0.0")
        $Subnets += $Subnet
        }
    }

# Remove duplicate subnets from Subnets array
$Subnets = ($Subnets | Sort -Unique)

# Add new subnets to NewScope array
$ExistingScope = (Get-NetFirewallRule -DisplayName "RDC Brute Force Prevention" | Get-NetFirewallAddressFilter).RemoteAddress
$NewScope += $Subnets + $ExistingScope

# Remove duplicate subnets from NewScope array
$NewScope = ($NewScope | Sort -Unique)
$NewScope

# Add new subnets to firewall rule
Set-NetFirewallRule -DisplayName "RDC Brute Force Prevention" -RemoteAddress $NewScope

# Write to event log
New-EventLog -LogName Application -Source "RDC Brute Force Prevention Script" -ErrorAction SilentlyContinue
Write-EventLog -LogName Application -Source "RDC Brute Force Prevention Script" -EntryType Information -EventId 1 -Message "The following subnets have been blocked for 24 hours:`n$NewScope"

# Remove old source firewall log files
Remove-Item -Force C:\Windows\System32\LogFiles\Firewall\pfirewall-source.log 
```

Tập lệnh này thực hiện các hành động sau:

- Tạo bản sao của tệp nhật ký tường lửa. **Đảm bảo rằng bạn đã bật tính năng ghi nhật ký trong Windows Defender Firewall!**
- Thêm quy tắc tường lửa mặc định được gọi là RDC Brute Force Prevention.
- Phân tích cú pháp tệp nhật ký tường lửa đã sao chép và đếm số IP đang cố gắng kết nối qua cổng 3389 trong vòng 5 phút qua. Nếu một địa chỉ IP thực hiện hơn 10 lần thử kết nối trong vòng 5 phút cuối cùng đó, thì địa chỉ đó sẽ bị tập lệnh gắn cờ là kết nối độc hại. (Điều này có thể được điều chỉnh trong kịch bản.)
- Chuyển đổi từng IP thành mạng con /16.
- Loại bỏ trùng lặp mạng con /16.
- Thêm các mạng con mới được phát hiện vào quy tắc tường lửa RDC Brute Force Prevention.
- Ghi nhật ký các mạng con mới được thêm vào Event Viewer.
- Xóa tệp nhật ký tường lửa đã sao chép.

Nếu bạn muốn tăng khoảng thời gian mà PowerShell sử dụng để đếm số IP nguồn đang cố gắng kết nối qua cổng 3389, bạn sẽ cần thay đổi giá trị được đặt cho biến $TimeSub5. Ví dụ: để tăng khoảng thời gian lên 10 phút, bạn sẽ thay đổi giá trị của .AddMinutes(-5) thành .AddMinutes(-10). Bạn cũng nên thay đổi tên biến để phản ánh khoảng thời gian. Vì vậy, trong ví dụ này, bạn sẽ thay đổi tên biến thành $TimeSub10.

Nếu bạn muốn thay đổi cổng đích mặc định từ 3389 thành một cổng tùy chỉnh, bạn sẽ cần chỉnh sửa tiêu chí cho cổng đích trong lệnh để phân tích cú pháp tệp nhật ký tường lửa. Ví dụ: để thay đổi cổng đích mặc định từ 3389 thành 1234, bạn sẽ thay đổi tiêu chí của $_.DestPort -EQ '3389' thành $_.DestPort -EQ '1234'.

Cuối cùng, nếu bạn muốn thay đổi số lượng IP nguồn được coi là brute force, bạn sẽ cần phải chỉnh sửa tiêu chí cho số lượng IP nguồn trong lệnh để phân tích cú pháp tệp nhật ký tường lửa. Ví dụ: để thay đổi số lượng IP nguồn từ 5 thành 10, bạn sẽ thay đổi tiêu chí của $_.Count -GT '5' thành $_.Count -GT '10'.

Khi bạn đã thực hiện các tùy chỉnh cần thiết, hãy lưu tập lệnh vào một tệp chia sẻ có thể truy cập được cho tất cả các máy chủ mà bạn muốn tập lệnh này chạy trên đó. Thông thường, tôi lưu các tập lệnh này vào phần chia sẻ tệp NETLOGON.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh1.png" width = "800">
</div>
<div class="thecap">NETLOGON chia sẻ với script</div>
</div>

Khởi chạy phần Group Policy MMC snap-in và mở đối tượng Group Policy có chứa cài đặt máy tính cho máy chủ mà bạn muốn áp dụng hướng dẫn này. Tại thời điểm này, chúng tôi cần chỉ định cài đặt chính sách sẽ sao chép tập lệnh RDC Brute Force Prevention.

Điều hướng đến **Computer Configuration\Preferences\Windows Settings\Files** và thêm một tệp mới. Đối với tệp nguồn, hãy nhập đường dẫn tệp tới RDC Brute Force Prevention Script mà bạn đã lưu trên mạng chia sẻ có thể truy cập được bằng Chính sách nhóm. Đối với hướng dẫn này, tôi sẽ lưu trữ tập lệnh từ xa trong **\\\perigon-networks.local\NETLOGON\Scripts\RDC_Brute_Force_Prevention.ps1**.

Đối với tệp đích, hãy nhập đường dẫn tệp tới tập lệnh RDC Brute Force, nơi nó sẽ được lưu trữ trên mỗi máy chủ cục bộ. Đối với hướng dẫn này, tôi sẽ lưu trữ tập lệnh cục bộ trong **C:\Scripts\**.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh2.png" width = "800">
</div>
<div class="thecap">Bản sao tệp GPO của tập lệnh RDC Brute Force Prevention</div>
</div>

Điều hướng đến **Computer Configuration\Preferences\Control Panel Settings\Scheduled Tasks** và thêm **New Scheduled Task** (ít nhất là Windows 7).

Đối với Tên, hãy nhập **Block Brute Force RDP Attacks**.

Đối với tài khoản Run as, chọn **NT AUTHORITY\System**.

Chọn tùy chọn để **Run whether user is logged on or not**.

Chọn tùy chọn để **Run with highest privileges**.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh3.png" width = "800">
</div>
<div class="thecap">Scheduled Task General Tab</div>
</div>

Đối với trình kích hoạt, hãy tạo một lịch biểu mà nhiệm vụ bắt đầu vào ngày nó được thực hiện , lúc 12 giờ đêm , lặp lại mỗi 1 ngày , cứ 5 phút một lần và dừng sau 30 phút.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh4.png" width = "800">
</div>
<div class="thecap">Scheduled Task Trigger Tab</div>
</div>

Đối với hành động, hãy tạo một hành động trong đó một chương trình được khởi động, chương trình là powershell.exe và các đối số là **-file C:\Path\To\Script**. Đối với hướng dẫn này, tôi sẽ sử dụng các đối số **-file C:\Path\To\Script**.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh5.png" width = "800">
</div>
<div class="thecap">Scheduled Task Action Tab</div>
</div>

Đóng trình New Task wizard. Đăng nhập vào một trong các máy chủ trong môi trường của bạn và chạy lệnh **gpupdate /force**. Mở **Task Scheduler** với tư cách Administrator và bạn sẽ thấy tác vụ đã lên lịch.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh6.png" width = "800">
</div>
<div class="thecap">Task Scheduler</div>
</div>

Sau 5 phút, nhiệm vụ theo lịch trình **Block Brute Force RDP Attacks** sẽ chạy. Sau khi hoàn tất, hãy khởi chạy Windows Defender Firewall với Advanced Security MMC snap-in. Trong Quy tắc đến, bạn sẽ thấy quy tắc từ chối được gọi là **RDC Brute Force Prevention**.

<div class="imgcap">
<div >
    <img src="/assets/block-brute-force-remote-desktop-attacks-with-windows-powershell/Hinh6.png" width = "800">
</div>
<div class="thecap">Quy tắc gửi đến của RDC Brute Force Connection</div>
</div>

Ngoài ra, sẽ có một mục nhập trong Event Viewer với danh sách các mạng con được thêm vào quy tắc **RDC Brute Force Prevention**, nếu có.

**Tài liệu tham khảo**

- [4sysops](https://4sysops.com/archives/block-brute-force-remote-desktop-attacks-with-windows-powershell/)

---
