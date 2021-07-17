---
layout: post
comments: true
title:  "Cassandra - Cấu hình trong Cassandra (Phần 1)"
title2:  "Cấu hình trong Cassandra (Phần 1)"
date:   2021-06-01 10:15:00
permalink: 2021/06/01/Cau-hinh-CSDL-Cassandra-p1
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/Cau-hinh-CSDL-Cassandra/Hinh1.jpg
summary: Cấu hình trong Cassandra
---

## 1. Cấu hình file cassandra.yaml

<span style="color:#c7254e">cluster_name</span>
- Tham số khai báo tên của cluster. Điều này chủ yếu được sử dụng để ngăn các máy trong một cluster logic này tham gia vào một cluster logic khác.
- *Giá trị mặc định cluster_name*: 'Test Cluster'

<span style="color:#c7254e">num_tokens</span>
- Tham số xác định số lượng mã token được gán ngẫu nhiên cho node này trên vòng token ring. Giá trị mã token càng lớn so với các node khác, tỷ lệ dữ liệu mà node này sẽ lưu trữ càng cao. Cassandra khuyến nghị thiết lập tất cả các node trong cùng cluster với cùng số lượng mã token như nhau (giả thiết các node có cấu hình phần cứng như nhau). 
- Nếu không xác định tham số này, Cassandra sẽ sử dụng 1 mã token theo cơ chế kế thừa và sẽ sử dụng tham số initial_token như được mô tả dưới đây.
- Khi được thiết lập, Cassandra sẽ sử dụng tham số initial_token vào lần khởi động ban đầu của node để ấn định (nhân công) mã token của node trên vòng token ring. Việc thay đổi initial_token trong những lần khởi động tiếp theo, cũng không có ý nghĩa.
- Cassandra khuyến nghị nên thiết lập  <span style="color:#c7254e">allocate_tokens_for_local_replication_factor</span> kết hợp với initial_token để đảm bảo phân bổ đồng đều dữ liệu trên các node.
- Giá trị mặc định initial_token: 256

<span style="color:#c7254e">allocate_tokens_for_keyspace</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tham số này sẽ kích hoạt phân bổ tự động các mã token dựa theo num_tokens cho node. Thuật toán phân bổ cố gắng chọn mã token theo cách tối ưu hóa dữ liệu sao chép qua các node trong datacenter để đảm bảo dữ liệu được gán cho mỗi node sẽ gần tỷ lệ với số vnodes của node đó so với các node khác trong datacenter.
- Các bản sao dữ liệu được xác định thông qua có chế sao chép dữ liệu được sử dụng trong keyspace. Việc tối ưu dữ liệu chỉ được hỗ trợ với Murmur3Partitioner, do vậy, Cassandra khuyến nghị sử dụng <span style="color:#c7254e">allocate_tokens_for_local_replication_factor</span> thay thế để đơn giản hóa quá trình hoạt động.
- *Giá trị mặc định*: KEYSPACE

<span style="color:#c7254e">allocate_tokens_for_local_replication_factor</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Mã token sẽ được phân bổ dựa trên yếu tố sao chép trong keyspace hoặc datacenter.
- *Giá trị mặc định*: 3

<span style="color:#c7254e">initial_token</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- initial_token cho phép ấn định nhân công vị trí mã token của node trên vòng token ring. Tham số này vẫn có thể được sử dụng với vnodes (num_tokens > 1, ở trên) – khi đó, cần cung cấp danh sách vị trí mã token tương ứng với mỗi vnode được phân tách bằng dấu phẩy. Điều này chủ yếu được sử dụng khi thêm các node vào các cluster kế thừa chưa bật vnodes.

<span style="color:#c7254e">hinted_handoff_enabled</span>
- Tham số này là một tính năng của Cassandra tối ưu hóa quy trình nhất quán của cluster và chống hỗn loạn khi một node sở hữu một bản sao dữ liệu không sẵn sàng do sự cố mạng hoặc các vấn đề khác. Tham số này có thể được đặt là "true" hoặc "false". 
- *Giá trị mặc định*: true

<span style="color:#c7254e">hinted_handoff_disabled_datacenters</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
Khi tham số hinted_handoff_enabled được đặt là true, danh sách đen các datacenter sẽ không thực hiện chuyển giao các gợi ý.
- *Giá trị mặc định (tùy chọn phức tạp)*:

```
#    - DC1
#    - DC2
```
<span style="color:#c7254e">max_hint_window_in_ms</span>
- Tham số này xác định khoảng thời gian tối đa mà một máy chủ bị mất kết nối với các thành viên còn lại sẽ được gợi ý ghi dữ liệu từ các máy chủ khác. Sau khoảng thời gian đó, các máy chủ khác sẽ ngừng tạo gợi ý ghi dữ liệu mới cho nó cho đến khi kết nối được khôi phục trở lại.
- *Giá trị mặc định*: 10800000 (tương ứng với 3 giờ)

<span style="color:#c7254e">hinted_handoff_throttle_in_kb</span>
- Tham số này thiết lập lưu lượng băng thông (KBps) trên mỗi luồng phân bổ dữ liệu tùy thuộc vào số node trong cluster: nếu có hai node trong clulster, mỗi luồng phân phối sẽ sử dụng lưu lượng băng thông; nếu có ba node, mỗi node sử dụng một nửa lưu lượng băng thông, vì về lý thuyết, sẽ có hai node cung cấp gợi ý đồng thời.
- *Giá trị mặc định*: 1024

<span style="color:#c7254e">max_hints_delivery_threads</span>
- Số lượng luồng phân bổ dữ liệu; Khi triển khai nhiều datacenter, cần cân nhắc tăng con số này vì quá trình xử lý chéo giữa các datacenter có xu hướng chậm hơn.
- *Giá trị mặc định*: 2

<span style="color:#c7254e">hints_directory</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Thư mục nơi Cassandra nên lưu trữ các gợi ý. Nếu không được thiết lập, Cassandra sẽ lưu mặc định tại thư mục $CASSANDRA_HOME/data/hints.
- *Giá trị mặc định*: /var/lib/cassandra/hints

<span style="color:#c7254e">hints_flush_period_in_ms</span>
- Tần suất các gợi ý ghi dữ liệu được chuyển từ bộ đệm bên trong vào ổ đĩa vật lý. Sẽ không kích hoạt fsync.
- *Giá trị mặc định*: 10000

<span style="color:#c7254e">max_hints_file_size_in_mb</span>
- Kích thước tối đa cho một tệp gợi ý, tính bằng megabyte.
- *Giá trị mặc định*: 128

<span style="color:#c7254e">hints_compression</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tham số này cho phép chọn cơ chế nén dữ liệu cho các file gợi ý. Nếu bỏ qua, các tệp gợi ý sẽ được ghi không nén. Cassandra hỗ trợ các chuẩn nén LZ4, Snappy và Deflate.
- *Giá trị mặc định (tùy chọn phức tạp)*:

```
#   - class_name: LZ4Compressor
#     parameters:
#     
```

<span style="color:#c7254e">batchlog_replay_throttle_in_kb</span>
- Tổng băng thông tối đa tính bằng KB trên giây. Điều này sẽ được giảm tương ứng với số lượng node trong cluster.
- *Giá trị mặc định*: 1024

<span style="color:#c7254e">authenticator</span>
- Tham số xác định tính năng xác thực người dùng, tính năng này được cung cấp bởi thư viện org.apache.cassandra.auth với 2 tham số:
- AllowAllAuthenticator không thực hiện kiểm tra - đặt nó để vô hiệu hóa xác thực.
-	PasswordAuthenticator dựa vào cặp tên người dùng / mật khẩu để xác thực người dùng. Cassandra lưu tên người dùng và mật khẩu được băm trong bảng system_auth.roles. Cassandra khuyến nghị tăng hệ số sao chép keyspace system_auth nếu sử dụng trình xác thực này. Ngoài ra cũng cần kết hợp với tham số CassandraRoleManager (xem bên dưới)
- *Giá trị mặc định*: AllowAllAuthenticator

<span style="color:#c7254e">authorizer</span>
- Tham số được sử dụng để giới hạn và phân quyền cấp quyền người dùng theo tài khoản đăng nhập, tính năng này được cung cấp bởi thư viện org.apache.cassandra.auth với 2 tham số:
-	AllowAllAuthorizer cho phép người dùng có toàn quyền truy cập hệ thống.
-	CassandraAuthorizer: Cho phếp cấu hình phân quyền người dùng được lưu trữ trong bảng system_auth.role_permissions. Cassandra khuyến nghị tăng hệ số sao chép keyspace system_auth sử dụng trình xác thực này.
- *Giá trị mặc định*: AllowAllAuthorizer

<span style="color:#c7254e">role_manager</span>
- Tham số xác thực & phân quyền; được sử dụng để duy trì các khoản quyền và mối quan hệ giữa các phân quyền. Tính năng này được cung cấp bởi thư viện org.apache.cassandra.auth.CassandraRoleManager lưu trữ thông tin các phân quyền trong keyspace system_auth. Hầu hết các chức năng của quản lý phân quyền đều yêu cầu xác thực đăng nhập, do đó tham số này chỉ có tác dụng khi cấu hình tham số Authenticator.
-	CassandraRoleManager lưu trữ dữ liệu các phân quyền trong keyspace system_auth. Cassandra khuyến nghị tăng hệ số sao chép keyspace system_auth sử dụng trình xác thực này.
- *Giá trị mặc định*: CassandraRoleManager

<span style="color:#c7254e">network_authorizer</span>
- Tham số xác thực mạng, được sử dụng để hạn chế quyền truy cập của người dùng vào các Datacenter nhất định. tính năng này được cung cấp bởi thư viện org.apache.cassandra.auth. với 2 tham số như sau:
-	AllowAllNetworkAuthorizer cho phép bất kỳ người dùng nào truy cập vào datacenter.
-	CassandraNetworkAuthorizer giới hạn phân quyền các lớp mạng được phép truy cập hệ thống. Các quyền được lưu trữ trong bảng system_auth.network_permissions. Cassandra khuyến nghị tăng hệ số sao chép keyspace system_auth sử dụng trình xác thực này.
- *Giá trị mặc định*: AllowAllNetworkAuthorizer

<span style="color:#c7254e">roles_validity_in_ms</span>
- Thời gian hiệu lực để lưu trữ các phân quyền người dùng trong bộ nhớ đệm. Các quyền truy cập sẽ được lưu vào bộ đệm cho các phiên được xác thực trong AuthenticatedUser và sau khoảng thời gian được chỉ định ở đây, sẽ được tải lại. Mặc định là 2000. Nếu được đặt là 0 sẽ tắt hoàn toàn bộ nhớ đệm. Chức năng này tự động tắt nếu chọn AllowAllAuthenticator.
- *Giá trị mặc định*: 2000

<span style="color:#c7254e">roles_update_interval_in_ms</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Khoảng thời gian tải lại các phân quyền người dùng được lưu bộ nhớ đệm (nếu được kích hoạt). Sau khoảng thời gian này, bộ nhớ cache sẽ đủ điều kiện để tải lại. Trong lần truy cập tiếp theo, quá trình tải lại không đồng bộ được lên lịch và giá trị cũ được trả về cho đến khi hoàn tất. Nếu role_validity_in_ms khác 0 thì giá trị này cũng phải thiết lập khác 0. Mặc định có cùng giá trị với role_validity_in_ms.
- *Giá trị mặc định*: 2000

<span style="color:#c7254e">permissions_validity_in_ms</span>
- Khoảng thời gian hiệu lực cho các quyền được lưu trong bộ nhớ đệm. Nếu đặt là 0 để tắt chức năng này. Chức năng này tự động tắt nếu chọn AllowAllAuthenticator.
- *Giá trị mặc định*: 2000

<span style="color:#c7254e">permissions_update_interval_in_ms</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Khoảng thời gian tải lại các quyền được lưu trong bộ nhớ đệm (nếu được kích hoạt). Sau khoảng thời gian này, bộ nhớ cache sẽ đủ điều kiện để tải lại. Trong lần truy cập tiếp theo, quá trình tải lại không đồng bộ được lên lịch và giá trị cũ được trả về cho đến khi hoàn tất. Nếu permissions_validity_in_ms khác 0, thì giá trị này cũng phải thiết lập khác 0. Mặc định có cùng giá trị với permissions_validity_in_ms.
- *Giá trị mặc định*: 2000

<span style="color:#c7254e">credentials_validity_in_ms</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Khoảng thời gian hiệu lực cho bộ nhớ đệm lưu thông tin xác thực. Nếu một triển khai xác thực khác được cấu hình, bộ đệm này sẽ không được tự động sử dụng và do đó các cài đặt tiếp theo dưới sẽ không còn hiệu lực. Xin lưu ý, thông tin xác thực được lưu trong bộ nhớ đêmk dưới dạng mã hóa, vì vậy trong khi kích hoạt bộ nhớ đệm này có thể làm giảm số lượng truy vấn được thực hiện cho bảng bên dưới, nó có thể không làm giảm đáng kể độ trễ của các lần xác thực riêng lẻ. Mặc định là 2000, đặt thành 0 để tắt bộ nhớ đệm thông tin xác thực.
- *Giá trị mặc định*: 2000

<span style="color:#c7254e">credentials_update_interval_in_ms</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Khoảng thời gian tải lại bộ nhớ đệm thông tin xác thực (nếu được kích hoạt). Sau khoảng thời gian này, bộ nhớ đệm sẽ đủ điều kiện để tải lại. Trong lần truy cập tiếp theo, quá trình tải lại không đồng bộ được lên lịch và giá trị cũ được trả về cho đến khi hoàn tất. Nếu thông tin xác thực_validity_in_ms khác 0 thì giá trị này cũng phải thiết lập khác 0. Mặc định có cùng giá trị với credentials_validity_in_ms.
- *Giá trị mặc định*: 2000

<span style="color:#c7254e">partitioner</span>
- Tham số xác định cơ chế phân phối dữ liệu theo nhóm các hàng dữ liệu (dựa trên khóa phân vùng) qua các node trong cluster. KHÔNG thể thay đổi cơ chế phân phối dữ liệu mà không cần tải lại tất cả dữ liệu. Do đó khi thêm node hoặc nâng cấp hệ thống, Cassandra khuyến nghị cần cài đặt tham số này giống với các node hiện đang sử dụng.
- Trình phân vùng mặc định là Murmur3Partitioner. Các trình phân vùng cũ hơn như RandomPartitioner, ByteOrderedPartitioner và OrderPreservingPartitioner chỉ được đưa vào để tương thích ngược với các phiên bản cũ hơn. Đối với các cluster mới, tuyệt đối không nên thay đổi giá trị này.
- *Giá trị mặc định*: org.apache.cassandra.dht.Murmur3Partitioner

<span style="color:#c7254e">data_file_directories</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tham số lutu thông tin thư thư mục lưu trữ dữ liệu trên ổ đĩa vật lý của một node. Nếu nhiều thư mục được chỉ định, Cassandra sẽ trải đều dữ liệu trên chúng bằng cách phân vùng các phạm vi mã token. Nếu không được thiết lập, dữ liệu sẽ được lưu vào thư mục mặc định là $CASSANDRA_HOME/data/data.
- *Giá trị mặc định (tùy chọn phức tạp)*:

```
#     - /var/lib/cassandra/data
```

<span style="color:#c7254e">local_system_data_file_directory</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tham số lưu thông tin thư mục lưu trữ dữ liệu của các system keyspaces trong nội bộ một node. Theo mặc định, Cassandra sẽ lưu trữ dữ liệu của các system keyspaces hệ (ngoại trừ system.batches, system.paxos, system.compaction_history, system.prepared_statements và các bảng system.repair) trong thư mục dữ liệu đầu tiên được chỉ định bởi data_file_directories. Điều này đảm bảo rằng nếu một trong các ổ đĩa lưu trữ dữ liệu khác bị mất, Cassandra vẫn có thể tiếp tục hoạt động. Để tăng cường bảo mật, tham số này cho phép thiết lập thêm các thư mục khác lưu trữ dự phòng.

<span style="color:#c7254e">commitlog_directory</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tham số lưu thông tin thư mục lưu trữ file Commitlog, về nguyên tắc, đây phải là thư mục lưu trữ tách biệt so với các thư mục dữ liệu. Nếu không được thiết lập, Cassandra sẽ lưu Commitlog vào thư mục mặc định là $CASSANDRA_HOME/data/commitlog.
- *Giá trị mặc định*: /var/lib/cassandra/commitlog

<span style="color:#c7254e">cdc_enabled</span>
- Tham số sử dụng để bật / tắt chức năng  CDC trên mỗi node. Điều này giúp thay đổi một cách logic để từ chối cấp phát đường dẫn ghi (mô hình chuẩn: không bao giờ từ chối. CDC: từ chối các thay đổi chứa trong bảng hỗ trợ CDC nếu đạt tới hạn dung lượng thư mục lưu trữ cdc_raw_directory).
- *Giá trị mặc định*: false

<span style="color:#c7254e">cdc_raw_directory</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tham số lưu thông tin thư mục lưu trữ dữ liệu khi các phân đoạn commit đẩy sang ổ đĩa vật lý nếu thiết lập cdc_enabled: true và lưu phân đoạn chứa các đột biến cho bảng hỗ trợ CDC. Thư mục này này nên được đặt trong một đường dẫn riêng biệt so với các thư mục dữ liệu khác. Nếu không được thiết lập, Cassandra sẽ lưu vào thư mục mặc định là $CASSANDRA_HOME/data/cdc_raw.
- *Giá trị mặc định*: /var/lib/cassandra/cdc_raw

<span style="color:#c7254e">disk_failure_policy</span>
- Tham số lưu chính sách khi lỗi ổ đĩa lưu dữ liệu. Tham  số này có các tùy chọn sau:
- **die**: tắt giao thức gossip và ngừng chuyển tải dữ liệu client, đồng thời dừng (kill) tiến trình JVM khi node gặp bất kỳ lỗi fs hoặc lỗi single-sstable, vì vậy node có thể được thay thế.
- **stop_paranoid**: tắt giao thức gossip và ngừng chuyển tải dữ liệu client ngay cả đối với các lỗi lỗi single-sstable, đồng thời dừng tiến trình JVM đối với các lỗi trong quá trình khởi động.
- **stop**: tắt giao thức gossip và ngừng chuyển tải dữ liệu client, như vậy sẽ không ảnh hưởng đến hoạt động chung của các node khác, nhưng vẫn có thể được kiểm tra thông qua tiến trình JMX, dừng tiến trình JVM để tìm lỗi trong quá trình khởi động.
- **best_effort**: ngừng sử dụng đĩa bị lỗi và phản hồi các yêu cầu dựa trên các bảng SStable còn lại. Điều này có nghĩa là client có thể sẽ thấy dữ liệu cũ tại CL.ONE!
- **ignore**: bỏ qua các lỗi nghiêm trọng và phản hồi yêu cầu không thành công, như trong các phiên bản Cassandra trước 1.2
- *Giá trị mặc định*: stop

<span style="color:#c7254e">commit_failure_policy</span>
- Tham số lưu chính sách khi lỗi ổ đĩa lưu file commitlog. Tham  số này có các tùy chọn sau:
- **die**: tắt node và dừng tiến trình JVM, vì vậy node có thể được thay thế.
- **stop**: tắt node, như vậy sẽ không ảnh hưởng đến hoạt động chung của các node khác, nhưng vẫn có thể được kiểm tra thông qua tiến trình JMX.
- **stop_commit**: tắt cập nhật thông tin vào commitlog, cho phép thu thập các yêu cầu ghi nhưng vẫn tiếp tục dịch vụ đọc (phản hồi các yêu cầu đọc), như trong các phiên bản Cassandra trước 2.0.5
- **ignore**: bỏ qua các lỗi nghiêm trọng và để các lô không thành công
- *Giá trị mặc định*: stop

<span style="color:#c7254e">prepared_statements_cache_size_mb</span>
- Kích thước tối đa của bộ nhớ đệm native protocol prepared statement
- Giá trị hợp lệ là "auto" (bỏ qua giá trị) hoặc giá trị lớn hơn 0.
- Lưu ý rằng việc chỉ định một giá trị quá lớn sẽ dẫn đến các tiến trình GC chạy với thời gian dài và có thể xảy ra lỗi tràn bộ nhớ. Giữ giá trị ở một phần nhỏ của bộ nhớ động (heap).
- Nếu bạn liên tục thấy thông báo "prepared statements discarded in the last minute because cache limit reached", bước đầu tiên là điều tra nguyên nhân chính của những thông báo này và kiểm tra xem câu lệnh đã chuẩn bị có được sử dụng đúng cách hay không - tức là sử dụng dấu liên kết cho các phần biến.
- Chỉ thay đổi giá trị mặc định, nếu bạn thực sự có nhiều câu lệnh chuẩn bị hơn là vừa với bộ nhớ cache. Trong hầu hết các trường hợp, không cần thiết phải thay đổi giá trị này. Liên tục chuẩn bị lại các tuyên bố là một hình phạt về hiệu suất.
- Giá trị mặc định ("tự động") là 1/256 của heap hoặc 10MB, tùy theo giá trị nào lớn hơn

<span style="color:#c7254e">key_cache_size_in_mb</span>
- Kích thước tối đa của bộ đệm theo key trong bộ nhớ.
- Mỗi lần truy cập bộ nhớ cache chính giúp tiết kiệm 1 lần tìm kiếm và mỗi lần truy cập bộ nhớ cache hàng sẽ tiết kiệm tối thiểu 2 lần tìm kiếm, đôi khi nhiều hơn. Bộ nhớ đệm cache chính khá nhỏ so với lượng thời gian mà nó tiết kiệm được, vì vậy rất đáng để sử dụng với số lượng lớn. Bộ nhớ đệm cache hàng thậm chí còn tiết kiệm thời gian hơn, nhưng phải chứa toàn bộ hàng, vì vậy nó cực kỳ tốn dung lượng. Tốt nhất chỉ nên sử dụng bộ đệm theo hàng nếu bạn có hàng nóng hoặc hàng tĩnh.
- LƯU Ý: nếu bạn giảm kích thước, bạn có thể không tải được các từ khóa nóng nhất khi khởi động.
- Giá trị mặc định để trống để làm cho nó "auto" (tối thiểu (5% Heap (tính bằng MB), 100MB)). Đặt thành 0 để tắt bộ nhớ cache của khóa.

<span style="color:#c7254e">key_cache_save_period</span>
- Khoảng thời gian tính bằng giây sau đó Cassandra sẽ lưu bộ nhớ cache theo key. Bộ nhớ cache được lưu vào save_caches_directory như được chỉ định trong tệp cấu hình này.
- Bộ nhớ đệm đã lưu giúp cải thiện đáng kể tốc độ khởi động nguội và tương đối rẻ về I/O cho bộ nhớ đệm theo key. Tiết kiệm bộ nhớ đệm theo hàng đắt hơn nhiều và bị hạn chế sử dụng.
Mặc định là 14400 hoặc 4 giờ.
- *Giá trị mặc định*: 14400

<span style="color:#c7254e">key_cache_keys_to_save</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Số lượng keys từ bộ nhớ đệm theo key để lưu Bị vô hiệu hóa theo mặc định, có nghĩa là tất cả các keys sẽ được lưu
- *Giá trị mặc định*: 100

<span style="color:#c7254e">row_cache_class_name</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Tên lớp bộ đệm theo hàng được triển khai. Các triển khai có sẵn:
  - **org.apache.cassandra.cache.OHCProvider**: Triển khai bộ đệm ẩn hàng hoàn toàn ngoài heap (mặc định).
  - **org.apache.cassandra.cache.SerializingCacheProvider**: Đây là triển khai bộ đệm hàng có sẵn trong các bản phát hành trước của Cassandra.
- *Giá trị mặc định*: org.apache.cassandra.cache.OHCProvider

<span style="color:#c7254e">row_cache_size_in_mb</span>
- Kích thước tối đa của bộ đệm theo hàng trong bộ nhớ. Xin lưu ý rằng việc triển khai bộ nhớ cache OHC yêu cầu một số bộ nhớ ngoài heap bổ sung để quản lý cấu trúc bản đồ và một số bộ nhớ trong trong quá trình hoạt động trước/sau các mục trong bộ nhớ cache có thể được tính vào dung lượng bộ nhớ cache. Chi phí chung này thường nhỏ so với toàn bộ công suất. Không chỉ định thêm bộ nhớ mà hệ thống có thể cung cấp trong tình huống xấu nhất thông thường và để lại khoảng trống cho bộ nhớ đệm cấp khối hệ điều hành. Không bao giờ cho phép hệ thống của bạn hoán đổi.
Giá trị mặc định là 0, để tắt bộ nhớ đệm theo hàng.

<span style="color:#c7254e">row_cache_save_period</span>
- Khoảng thời gian tính bằng giây sau đó Cassandra sẽ lưu vào bộ nhớ cache theo hàng. Bộ nhớ cache được lưu vào save_caches_directory như được chỉ định trong tệp cấu hình này.
- Bộ nhớ đệm đã lưu giúp cải thiện đáng kể tốc độ khởi động nguội và tương đối rẻ về I/O cho bộ nhớ đệm theo key. Tiết kiệm bộ nhớ cache theo hàng đắt hơn nhiều và bị hạn chế sử dụng.
- Mặc định là 0 để tắt lưu bộ đệm theo hàng.

<span style="color:#c7254e">row_cache_keys_to_save</span>
- *Tùy chọn này được cho vào phần ghi chú theo mặc định*
- Số lượng khóa từ bộ nhớ cache của hàng để lưu. Chỉ định 0 (là mặc định), nghĩa là tất cả các khóa sẽ được lưu
- *Giá trị mặc định*: 100

<span style="color:#c7254e">counter_cache_size_in_mb</span>
- Kích thước tối đa của bộ đệm truy cập trong bộ nhớ.
- Bộ nhớ đệm của bộ đếm giúp giảm sự tranh giành của các khóa bộ đếm đối với các ô của bộ đếm nóng. Trong trường hợp RF = 1, một lần truy cập vào bộ nhớ đệm bộ đếm sẽ khiến Cassandra bỏ qua quá trình đọc trước khi ghi hoàn toàn. Với RF> 1, một lần truy cập bộ nhớ đệm bộ đếm sẽ vẫn giúp giảm thời gian giữ khóa, giúp cập nhật ô bộ đếm nóng, nhưng sẽ không cho phép bỏ qua hoàn toàn quá trình đọc. Chỉ bộ đếm cục bộ (đồng hồ, số đếm) của ô bộ đếm được lưu trong bộ nhớ, không phải toàn bộ bộ đếm, vì vậy nó tương đối rẻ.
LƯU Ý: nếu bạn giảm kích thước, bạn có thể không tải được các phím nóng nhất khi khởi động.
Giá trị mặc định trống để làm cho nó "auto" (tối thiểu (2,5% Heap (tính bằng MB), 50MB)). Đặt thành 0 để tắt bộ nhớ cache của bộ đếm. LƯU Ý: nếu bạn thực hiện xóa bộ đếm và dựa trên gcgs thấp, bạn nên tắt bộ nhớ cache của bộ đếm.

15.	counter_cache_save_period
Khoảng thời gian tính bằng giây sau đó Cassandra sẽ lưu vào bộ nhớ đệm của bộ đếm (chỉ dành cho các phím). Bộ nhớ cache được lưu vào save_caches_directory như được chỉ định trong tệp cấu hình này.
Mặc định là 7200 hoặc 2 giờ.
Giá trị mặc định: 7200
16.	counter_cache_keys_to_save
Tùy chọn này được nhận xét theo mặc định.
Số lượng khóa từ bộ nhớ đệm của bộ đếm để lưu Bị vô hiệu hóa theo mặc định, có nghĩa là tất cả các khóa sẽ được lưu
Giá trị mặc định: 100
17.	saved_caches_directory
Tùy chọn này được nhận xét theo mặc định.
bộ nhớ đệm đã lưu Nếu không được đặt, thư mục mặc định là $ CASSANDRA_HOME / data / save_caches.
Giá trị mặc định: / var / lib / cassandra / save_caches
18.	commitlog_sync_batch_window_in_ms
Tùy chọn này được nhận xét theo mặc định.
commitlog_sync có thể là “định kỳ”, “nhóm” hoặc “lô”.
Khi ở chế độ hàng loạt, Cassandra sẽ không ghi cho đến khi nhật ký cam kết được chuyển vào đĩa. Mỗi lần ghi đến sẽ kích hoạt tác vụ tuôn ra. commitlog_sync_batch_window_in_ms là một giá trị không được dùng nữa. Trước đây nó hầu như không có giá trị và đang bị loại bỏ.
Giá trị mặc định: 2
19.	commitlog_sync_group_window_in_ms
Tùy chọn này được nhận xét theo mặc định.
chế độ nhóm tương tự như chế độ hàng loạt, nơi Cassandra sẽ không ghi cho đến khi nhật ký cam kết được chuyển vào đĩa. Sự khác biệt là chế độ nhóm sẽ đợi đến commitlog_sync_group_window_in_ms giữa các lần xả.
Giá trị mặc định: 1000
20.	commitlog_sync
tùy chọn mặc định là “định kỳ” trong đó việc ghi có thể được thực hiện ngay lập tức và commitLog được đồng bộ hóa đơn giản sau mỗi mili giây commitlog_sync_period_in_ms.
Giá trị mặc định: định kỳ
21.	commitlog_sync_period_in_ms
Giá trị mặc định: 10000
22.	periodic_commitlog_sync_lag_block_in_ms
Tùy chọn này được nhận xét theo mặc định.
Khi ở chế độ cam kết định kỳ, số mili giây để chặn ghi trong khi chờ quá trình xả đĩa chậm hoàn tất.
23.	commitlog_segment_size_in_mb
Kích thước của các phân đoạn tệp cam kết riêng lẻ. Một đoạn commitlog có thể được lưu trữ, xóa hoặc tái chế sau khi tất cả dữ liệu trong đó (có thể từ mỗi họ cột trong hệ thống) đã được chuyển sang sstables.
Kích thước mặc định là 32, hầu như luôn ổn, nhưng nếu bạn đang lưu trữ các phân đoạn commitlog (xem commitlog_archiving.properties), thì bạn có thể muốn lưu trữ chi tiết hơn; 8 hoặc 16 MB là hợp lý. Kích thước đột biến tối đa cũng có thể được định cấu hình thông qua cài đặt max_mutation_size_in_kb trong cassandra.yaml. Giá trị mặc định là một nửa kích thước commitlog_segment_size_in_mb * 1024. Giá trị này phải là số dương và nhỏ hơn 2048.
LƯU Ý: Nếu max_mutation_size_in_kb được đặt rõ ràng thì commitlog_segment_size_in_mb phải được đặt thành ít nhất gấp đôi kích thước của max_mutation_size_in_kb / 1024
Giá trị mặc định: 32
24.	commitlog_compression
Tùy chọn này được nhận xét theo mặc định.
Nén để áp dụng cho nhật ký cam kết. Nếu bị bỏ qua, bản ghi cam kết sẽ được ghi không nén. Máy nén LZ4, Snappy và Deflate được hỗ trợ.
Giá trị mặc định (tùy chọn phức hợp) :
#   - class_name: LZ4Compressor
#     parameters:
#         -
25.	table
Tùy chọn này được nhận xét theo mặc định. Nén để áp dụng cho SSTables khi chúng tuôn ra cho các bảng đã nén. Lưu ý rằng các bảng không được bật tính năng nén sẽ không tôn trọng cờ này.
Vì các máy nén tỷ lệ cao như LZ4HC, Zstd và Deflate có thể có khả năng gây tắc nghẽn quá lâu, nên mặc định là xả bằng máy nén nhanh đã biết trong những trường hợp đó. Các tùy chọn là:
none: Xả mà không cần nén khối nhưng vẫn thực hiện tổng kiểm tra. nhanh: Xả bằng máy nén nhanh. Nếu bảng đã sử dụng
máy nén nhanh mà máy nén được sử dụng.
Giá trị mặc định: Luôn xả bằng cùng một máy nén mà bảng sử dụng. Điều này
26.	flush_compression
Tùy chọn này được nhận xét theo mặc định.
là hành vi trước 4.0.
Giá trị mặc định: nhanh
27.	seed_provider
bất kỳ lớp nào triển khai giao diện SeedProvider và có một phương thức khởi tạo nhận Map <String, String> của các tham số sẽ làm được.
Giá trị mặc định (tùy chọn phức hợp) :
# Addresses of hosts that are deemed contact points.
# Cassandra nodes use this list of hosts to find each other and learn
# the topology of the ring.  You must change this if you are running
# multiple nodes!
- class_name: org.apache.cassandra.locator.SimpleSeedProvider
  parameters:
      # seeds is actually a comma-delimited list of addresses.
      # Ex: "<ip1>,<ip2>,<ip3>"
      - seeds: "127.0.0.1:7000"
28.	concurrent_reads
Đối với khối lượng công việc có nhiều dữ liệu hơn mức có thể chứa trong bộ nhớ, node thắt cổ chai của Cassandra sẽ là các lần đọc cần tìm nạp dữ liệu từ đĩa. “Concurrent_reads” nên được đặt thành (16 * number_of_drives) để cho phép các hoạt động xếp hàng đủ thấp trong ngăn xếp mà hệ điều hành và ổ đĩa có thể sắp xếp lại chúng. Tương tự cũng áp dụng cho “concurrent_counter_writes”, vì bộ đếm ghi đọc các giá trị hiện tại trước khi tăng dần và ghi lại chúng.
Mặt khác, vì các lần ghi hầu như không bao giờ bị ràng buộc IO, nên số lượng lý tưởng của “concurrent_writes” phụ thuộc vào số lượng lõi trong hệ thống của bạn; (8 * number_of_cores) là một nguyên tắc nhỏ.
Giá trị mặc định: 32
29.	concurrent_writes
Giá trị mặc định: 32
30.	concurrent_counter_writes
Giá trị mặc định: 32
31.	concurrent_materialized_view_writes
Đối với các lần ghi dạng xem cụ thể hóa, vì có liên quan đến việc đọc, vì vậy điều này nên được hạn chế bằng cách ít đọc đồng thời hoặc ghi đồng thời.
Giá trị mặc định: 32
32.	file_cache_size_in_mb
Tùy chọn này được nhận xét theo mặc định.
Bộ nhớ tối đa để sử dụng cho bộ nhớ đệm sstable chunk và tổng hợp bộ đệm. 32MB trong số này được dành cho bộ đệm gộp, phần còn lại được sử dụng làm bộ nhớ đệm chứa các khối có thể nén không nén. Mặc định là nhỏ hơn 1/4 heap hoặc 512MB. Nhóm này được cấp phát off-heap, ngoài bộ nhớ được cấp phát cho heap cũng vậy. Bộ nhớ đệm cũng có chi phí trên heap khoảng 128 byte cho mỗi chunk (tức là 0,2% kích thước dành riêng nếu kích thước chunk 64k mặc định được sử dụng). Bộ nhớ chỉ được cấp phát khi cần thiết.
Giá trị mặc định: 512
33.	buffer_pool_use_heap_if_exhausted
Tùy chọn này được nhận xét theo mặc định.
Cờ cho biết có nên phân bổ bật hay tắt heap khi vùng đệm sstable cạn kiệt, đó là khi nó đã vượt quá bộ nhớ tối đa file_cache_size_in_mb, vượt quá mức này, nó sẽ không lưu bộ đệm mà phân bổ theo yêu cầu.
Giá trị mặc định: true
34.	disk_optimization_strategy
Tùy chọn này được nhận xét theo mặc định.
Chiến lược để tối ưu hóa việc đọc đĩa Các giá trị có thể là: ssd (đối với đĩa trạng thái rắn, mặc định) quay (đối với đĩa quay)
Giá trị mặc định: ssd
35.	memtable_heap_space_in_mb
Tùy chọn này được nhận xét theo mặc định.
Tổng bộ nhớ được phép sử dụng cho các bảng ghi nhớ. Cassandra sẽ ngừng chấp nhận ghi khi vượt quá giới hạn cho đến khi một lần xả hoàn tất và sẽ kích hoạt một lần xả dựa trên memtable_cleanup_threshold Nếu bỏ qua, Cassandra sẽ đặt cả hai thành 1/4 kích thước của đống.
Giá trị mặc định: 2048
36.	memtable_offheap_space_in_mb
Tùy chọn này được nhận xét theo mặc định.
Giá trị mặc định: 2048
37.	memtable_cleanup_threshold
Tùy chọn này được nhận xét theo mặc định.
memtable_cleanup_threshold không được dùng nữa. Tính toán mặc định là sự lựa chọn hợp lý duy nhất. Xem các bình luận trên memtable_flush_writers để biết thêm thông tin.
Tỷ lệ giữa kích thước bảng ghi nhớ không xả đã chiếm dụng so với tổng kích thước cho phép sẽ kích hoạt một lần xả bảng ghi nhớ lớn nhất. Mct lớn hơn sẽ có nghĩa là xả lớn hơn và do đó ít nén hơn, nhưng hoạt động xả đồng thời cũng ít hơn, điều này có thể gây khó khăn cho việc giữ cho đĩa của bạn được nạp dưới tải ghi nặng.
memtable_cleanup_threshold mặc định là 1 / (memtable_flush_writers + 1)
Giá trị mặc định: 0,11
38.	memtable_allocation_type
Chỉ định cách Cassandra phân bổ và quản lý bộ nhớ có thể ghi nhớ. Các tùy chọn là:
heap_buffers
trên bộ đệm heap nio
offheap_buffers
tắt bộ đệm nio heap (trực tiếp)
offheap_objects
off đống đối tượng
Giá trị mặc định: heap_buffers
39.	repair_session_space_in_mb
Tùy chọn này được nhận xét theo mặc định.
Hạn chế sử dụng bộ nhớ cho các tính toán cây Merkle trong quá trình sửa chữa. Giá trị mặc định là 1/16 của heap có sẵn. Sự cân bằng chính là các cây nhỏ hơn có độ phân giải kém hơn, điều này có thể dẫn đến việc truyền dữ liệu quá mức. Nếu bạn thấy áp suất đống trong quá trình sửa chữa, hãy xem xét giảm mức này xuống, nhưng bạn không thể xuống dưới một megabyte. Nếu bạn thấy nhiều luồng quá mức, hãy cân nhắc việc tăng cường mức này hoặc sử dụng sửa chữa dải phụ.
Để biết thêm chi tiết, hãy xem https://issues.apache.org/jira/browse/CASSANDRA-14096 .
40.	commitlog_total_space_in_mb
Tùy chọn này được nhận xét theo mặc định.
Tổng dung lượng để sử dụng cho các bản ghi cam kết trên đĩa.
Nếu không gian vượt quá giá trị này, Cassandra sẽ xóa mọi CF bẩn trong phân đoạn cũ nhất và xóa nó. Vì vậy, tổng không gian cam kết nhỏ sẽ có xu hướng gây ra nhiều hoạt động tuôn ra hơn trên các họ cột ít hoạt động hơn.
Giá trị mặc định là nhỏ hơn 8192 và 1/4 tổng không gian của khối lượng cam kết.
Giá trị mặc định: 8192
41.	memtable_flush_writers
Tùy chọn này được nhận xét theo mặc định.
Điều này đặt số lượng chủ đề trình ghi ghi nhớ xóa trên mỗi đĩa cũng như tổng số các ghi nhớ có thể được xóa đồng thời. Đây thường là sự kết hợp giữa tính toán và IO ràng buộc.
Xả ghi nhớ hiệu quả hơn CPU so với nhập ghi nhớ và một luồng duy nhất có thể theo kịp tốc độ nhập của toàn bộ máy chủ trên một đĩa nhanh duy nhất cho đến khi nó tạm thời bị ràng buộc IO dưới sự tranh chấp thường là với nén. Tại thời điểm đó, bạn cần nhiều chủ đề tuôn ra. Tại một số thời điểm trong tương lai, nó có thể trở thành CPU bị ràng buộc mọi lúc.
Bạn có thể biết liệu quá trình xả có tụt hậu hay không bằng cách sử dụng chỉ số MemtablePool.BlockedOnAllocation phải bằng 0, nhưng sẽ khác 0 nếu các luồng bị chặn đang chờ xả để giải phóng bộ nhớ.
memtable_flush_writers mặc định là hai cho một thư mục dữ liệu. Điều này có nghĩa là hai bảng ghi nhớ có thể được chuyển đồng thời vào một thư mục dữ liệu duy nhất. Nếu bạn có nhiều thư mục dữ liệu, mặc định là một bản ghi nhớ tuôn ra tại một thời điểm nhưng quá trình tuôn ra sẽ sử dụng một chuỗi trên mỗi thư mục dữ liệu, do đó bạn sẽ có hai hoặc nhiều người ghi.
Nói chung, hai là đủ để xóa trên đĩa nhanh [mảng] được gắn kết dưới dạng một thư mục dữ liệu duy nhất. Việc bổ sung thêm nhiều trình viết flush sẽ dẫn đến các flush nhỏ hơn thường xuyên hơn, tạo ra nhiều chi phí nén hơn.
Có một sự cân bằng trực tiếp giữa số lượng bảng ghi nhớ có thể được xóa đồng thời với kích thước và tần suất xả. Nhiều hơn không phải là tốt hơn, bạn chỉ cần đủ trình viết xả để không bao giờ ngừng chờ xả để giải phóng bộ nhớ.
Giá trị mặc định: 2
42.	cdc_total_space_in_mb
Tùy chọn này được nhận xét theo mặc định.
Tổng dung lượng để sử dụng cho các nhật ký ghi lại dữ liệu thay đổi trên đĩa.
Nếu không gian vượt quá giá trị này, Cassandra sẽ ném WriteTimeoutException trên Mutations bao gồm các bảng có bật CDC. Một CDCCompactor chịu trách nhiệm phân tích cú pháp các bản ghi CDC thô và xóa chúng khi quá trình phân tích cú pháp hoàn tất.
Giá trị mặc định là tối thiểu 4096 mb và 1/8 tổng dung lượng ổ đĩa nơi cdc_raw_directory cư trú.
Giá trị mặc định: 4096
43.	cdc_free_space_check_interval_ms
Tùy chọn này được nhận xét theo mặc định.
Khi chúng tôi đạt đến giới hạn cdc_raw của mình và CDCCompactor đang chạy chậm hoặc gặp áp suất ngược, chúng tôi kiểm tra vào khoảng thời gian sau để xem liệu có sẵn không gian mới nào cho các bảng được theo dõi cdc hay không. Mặc định là 250ms
Giá trị mặc định: 250
44.	index_summary_capacity_in_mb
Kích thước nhóm bộ nhớ cố định tính bằng MB cho tóm tắt chỉ mục SSTable. Nếu để trống, điều này sẽ mặc định là 5% kích thước heap. Nếu việc sử dụng bộ nhớ của tất cả các tóm tắt chỉ mục vượt quá giới hạn này, các SSTables có tốc độ đọc thấp sẽ thu nhỏ các tóm tắt chỉ mục của chúng để đáp ứng giới hạn này. Tuy nhiên, đây là một quá trình nỗ lực hết mình. Trong điều kiện khắc nghiệt, Cassandra có thể cần sử dụng nhiều hơn số lượng bộ nhớ này.
45.	index_summary_resize_interval_in_minutes
Tần suất tóm tắt chỉ mục nên được lấy mẫu lại. Điều này được thực hiện định kỳ để phân phối lại bộ nhớ từ nhóm có kích thước cố định thành các chuỗi tương ứng với tỷ lệ đọc gần đây của chúng. Đặt thành -1 sẽ vô hiệu hóa quá trình này, để lại các tóm tắt chỉ mục hiện có ở mức lấy mẫu hiện tại của chúng.
Giá trị mặc định: 60
46.	trickle_fsync
Có nên, khi thực hiện ghi tuần tự, fsync () tại các khoảng thời gian để buộc hệ điều hành xả các bộ đệm bẩn. Bật tính năng này để tránh xả bộ đệm bẩn đột ngột do ảnh hưởng đến độ trễ đọc. Hầu như luôn luôn là một ý tưởng hay trên SSD; không nhất thiết phải trên đĩa.
Giá trị mặc định: false
47.	trickle_fsync_interval_in_kb
Giá trị mặc định: 10240
48.	storage_port
Cổng TCP, dành cho lệnh và dữ liệu Vì lý do bảo mật, bạn không nên để cổng này lên internet. Tường lửa cho nó nếu cần.
Giá trị mặc định: 7000
49.	ssl_storage_port
Cổng SSL, dành cho giao tiếp được mã hóa kế thừa. Thuộc tính này không được sử dụng trừ khi được bật trong server_encryption_options (xem bên dưới). Kể từ cassandra 4.0, thuộc tính này không được dùng nữa vì một cổng duy nhất có thể được sử dụng cho một trong hai / cả các kết nối an toàn và không an toàn. Vì lý do bảo mật, bạn không nên để cổng này tiếp xúc với internet. Tường lửa cho nó nếu cần.
Giá trị mặc định: 7001
50.	listen_address
Địa chỉ hoặc giao diện để liên kết và yêu cầu các node Cassandra khác kết nối với. Bạn _must_ thay đổi điều này nếu bạn muốn nhiều node có thể giao tiếp!
Đặt nghe_address HOẶC listening_interface, không đặt cả hai.
Để trống nó sẽ chuyển sang InetAddress.getLocalHost (). Điều này sẽ luôn thực hiện Điều đúng _if_ node được định cấu hình đúng cách (tên máy chủ, độ phân giải tên, v.v.) và Điều đúng là sử dụng địa chỉ được liên kết với tên máy chủ (có thể không).
Đặt Listen_address thành 0.0.0.0 luôn luôn sai.
Giá trị mặc định: localhost
51.	listen_interface
Tùy chọn này được nhận xét theo mặc định.
Đặt nghe_address HOẶC listening_interface, không đặt cả hai. Các giao diện phải tương ứng với một địa chỉ duy nhất, bí danh IP không được hỗ trợ.
Giá trị mặc định: eth0
52.	listen_interface_prefer_ipv6
Tùy chọn này được nhận xét theo mặc định.
Nếu bạn chọn chỉ định giao diện theo tên và giao diện có ipv4 và địa chỉ ipv6, bạn có thể chỉ định địa chỉ nào nên được chọn bằng cách sử dụng nghe_interface_prefer_ipv6. Nếu sai, địa chỉ ipv4 đầu tiên sẽ được sử dụng. Nếu đúng, địa chỉ ipv6 đầu tiên sẽ được sử dụng. Mặc định là ipv4 ưu tiên sai. Nếu chỉ có một địa chỉ, nó sẽ được chọn bất kể ipv4 / ipv6.
Giá trị mặc định: false
53.	broadcast_address
Tùy chọn này được nhận xét theo mặc định.
Địa chỉ để phát tới các node Cassandra khác Để trống vùng này sẽ đặt nó thành cùng một giá trị với listening_address
Giá trị mặc định: 1.2.3.4
54.	listen_on_broadcast_address
Tùy chọn này được nhận xét theo mặc định.
Khi sử dụng nhiều giao diện mạng vật lý, hãy đặt giá trị này thành true để lắng nghe trên broadcast_address bên cạnh địa chỉ listening_address, cho phép các node giao tiếp trong cả hai giao diện. Bỏ qua thuộc tính này nếu cấu hình mạng tự động định tuyến giữa mạng công cộng và mạng riêng, chẳng hạn như EC2.
Giá trị mặc định: false
55.	internode_authenticator
Tùy chọn này được nhận xét theo mặc định.
Phụ trợ xác thực Internode, triển khai IInternodeAuthenticator; được sử dụng để cho phép / không cho phép kết nối từ các node ngang hàng.
Giá trị mặc định: org.apache.cassandra.auth.AllowAllInternodeAuthenticator
56.	start_native_transport
Có khởi động máy chủ truyền tải gốc hay không. Địa chỉ mà phương tiện truyền tải gốc bị ràng buộc được xác định bởi rpc_address.
Giá trị mặc định: true
57.	native_transport_port
cổng cho phương tiện truyền tải gốc CQL để lắng nghe khách hàng trên Vì lý do bảo mật, bạn không nên để lộ cổng này trên internet. Tường lửa cho nó nếu cần.
Giá trị mặc định: 9042
58.	native_transport_port_ssl
Tùy chọn này được nhận xét theo mặc định. Bật mã hóa truyền tải gốc trong client_encryption_options cho phép bạn sử dụng mã hóa cho cổng tiêu chuẩn hoặc sử dụng một cổng bổ sung, chuyên dụng cùng với native_transport_port tiêu chuẩn không được mã hóa. Bật mã hóa ứng dụng khách và tắt native_transport_port_ssl sẽ sử dụng mã hóa cho native_transport_port. Đặt native_transport_port_ssl thành một giá trị khác với native_transport_port sẽ sử dụng mã hóa cho native_transport_port_ssl trong khi vẫn giữ nguyên native_transport_port không được mã hóa.
Giá trị mặc định: 9142
59.	native_transport_max_threads
Tùy chọn này được nhận xét theo mặc định. Các luồng tối đa để xử lý các yêu cầu (lưu ý rằng các luồng không hoạt động bị dừng sau 30 giây nên không có cài đặt tối thiểu tương ứng).
Giá trị mặc định: 128
60.	native_transport_max_frame_size_in_mb
Tùy chọn này được nhận xét theo mặc định.
Kích thước tối đa của khung cho phép. Khung (yêu cầu) lớn hơn khung này sẽ bị từ chối vì không hợp lệ. Mặc định là 256MB. Nếu bạn đang thay đổi thông số này, bạn có thể muốn điều chỉnh max_value_size_in_mb cho phù hợp. Giá trị này phải là dương và nhỏ hơn 2048.
Giá trị mặc định: 256
61.	native_transport_frame_block_size_in_kb
Tùy chọn này được nhận xét theo mặc định.
Nếu tổng kiểm tra được bật như một tùy chọn giao thức, biểu thị kích thước của các phần mà khung là phần thân sẽ bị phá vỡ và tổng kiểm tra.
Giá trị mặc định: 32
62.	native_transport_max_concurrent_connections
Tùy chọn này được nhận xét theo mặc định.
Số lượng tối đa các kết nối ứng dụng khách đồng thời. Mặc định là -1, có nghĩa là không giới hạn.
Giá trị mặc định: -1
63.	native_transport_max_concurrent_connections_per_ip
Tùy chọn này được nhận xét theo mặc định.
Số lượng tối đa các kết nối máy khách đồng thời trên mỗi ip nguồn. Mặc định là -1, có nghĩa là không giới hạn.
Giá trị mặc định: -1
64.	native_transport_allow_older_protocols
Kiểm soát xem liệu Cassandra có tôn vinh các phiên bản giao thức cũ hơn, nhưng hiện được hỗ trợ hay không. Giá trị mặc định là đúng, có nghĩa là tất cả các giao thức được hỗ trợ sẽ được sử dụng.
Giá trị mặc định: true
65.	native_transport_idle_timeout_in_ms
Tùy chọn này được nhận xét theo mặc định.
Điều khiển khi các kết nối khách không hoạt động bị đóng. Các kết nối không hoạt động là những kết nối không đọc hoặc ghi trong một khoảng thời gian.
Khách hàng có thể thực hiện nhịp tim bằng cách gửi tin nhắn giao thức gốc OPTIONS sau một khoảng thời gian chờ, điều này sẽ đặt lại bộ đếm thời gian chờ nhàn rỗi ở phía máy chủ. Để đóng các kết nối máy khách không hoạt động, các giá trị tương ứng cho khoảng nhịp tim phải được đặt ở phía máy khách.
Thời gian chờ kết nối không hoạt động bị tắt theo mặc định.
Giá trị mặc định: 60000
66.	rpc_address
Địa chỉ hoặc giao diện để liên kết máy chủ truyền tải gốc.
Đặt rpc_address HOẶC rpc_interface, không phải cả hai.
Để trống rpc_address có tác dụng tương tự như trên listening_address (tức là nó sẽ dựa trên tên máy chủ được cấu hình của node).
Lưu ý rằng không giống như listening_address, bạn có thể chỉ định 0.0.0.0, nhưng bạn cũng phải đặt broadcast_rpc_address thành một giá trị khác 0.0.0.0.
Vì lý do bảo mật, bạn không nên để cổng này tiếp xúc với internet. Tường lửa cho nó nếu cần.
Giá trị mặc định: localhost
67.	rpc_interface
Tùy chọn này được nhận xét theo mặc định.
Đặt rpc_address HOẶC rpc_interface, không phải cả hai. Các giao diện phải tương ứng với một địa chỉ duy nhất, bí danh IP không được hỗ trợ.
Giá trị mặc định: eth1
68.	rpc_interface_prefer_ipv6
Tùy chọn này được nhận xét theo mặc định.
Nếu bạn chọn chỉ định giao diện theo tên và giao diện có ipv4 và địa chỉ ipv6, bạn có thể chỉ định địa chỉ nào nên được chọn bằng cách sử dụng rpc_interface_prefer_ipv6. Nếu sai, địa chỉ ipv4 đầu tiên sẽ được sử dụng. Nếu đúng, địa chỉ ipv6 đầu tiên sẽ được sử dụng. Mặc định là ipv4 ưu tiên sai. Nếu chỉ có một địa chỉ, nó sẽ được chọn bất kể ipv4 / ipv6.
Giá trị mặc định: false
69.	broadcast_rpc_address
Tùy chọn này được nhận xét theo mặc định.
Địa chỉ RPC để phát tới trình điều khiển và các node Cassandra khác. Điều này không thể được đặt thành 0.0.0.0. Nếu để trống, giá trị này sẽ được đặt thành giá trị của rpc_address. Nếu rpc_address được đặt thành 0.0.0.0, thì broadcast_rpc_address phải được đặt.
Giá trị mặc định: 1.2.3.4
70.	rpc_keepalive
bật hoặc tắt keepalive trên các kết nối rpc / native
Giá trị mặc định: true
71.	internode_send_buff_size_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Bỏ ghi chú để đặt kích thước bộ đệm ổ cắm cho giao tiếp lóng Lưu ý rằng khi thiết lập điều này, kích thước bộ đệm bị giới hạn bởi net.core.wmem_max và khi không đặt nó được xác định bởi net.ipv4.tcp_wmem Xem thêm: / proc / sys / net / core / wmem_max / proc / sys / net / core / rmem_max / proc / sys / net / ipv4 / tcp_wmem / proc / sys / net / ipv4 / tcp_wmem and 'man tcp'
72.	internode_recv_buff_size_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Bỏ ghi chú để đặt kích thước bộ đệm ổ cắm cho giao tiếp lóng Lưu ý rằng khi thiết lập điều này, kích thước bộ đệm bị giới hạn bởi net.core.wmem_max và khi không đặt nó được xác định bởi net.ipv4.tcp_wmem
73.	incremental_backups
Đặt thành true để yêu cầu Cassandra tạo liên kết cứng tới từng sstable được gửi hoặc phát trực tuyến cục bộ trong bản sao lưu / thư mục con của dữ liệu keyspace. Việc loại bỏ các liên kết này là trách nhiệm của nhà điều hành.
Giá trị mặc định: false
74.	snapshot_before_compaction
Có hay không chụp nhanh trước mỗi lần đầm. Hãy cẩn thận khi sử dụng tùy chọn này, vì Cassandra sẽ không xóa các ảnh chụp nhanh cho bạn. Hầu hết hữu ích nếu bạn hoang tưởng khi có sự thay đổi định dạng dữ liệu.
Giá trị mặc định: false
75.	auto_snapshot
Liệu ảnh chụp nhanh có được lấy dữ liệu hay không trước khi cắt hoặc giảm không gian phím của các họ cột. Nên sử dụng giá trị mặc định được khuyến nghị MẠNH MẼ là true để đảm bảo an toàn cho dữ liệu. Nếu bạn đặt cờ này thành false, bạn sẽ mất dữ liệu khi cắt hoặc giảm.
Giá trị mặc định: true
76.	column_index_size_in_kb
Mức độ chi tiết của chỉ số đối chiếu của các hàng trong một phân vùng. Tăng nếu các hàng của bạn lớn hoặc nếu bạn có số lượng hàng rất lớn trên mỗi phân vùng. Các mục tiêu cạnh tranh là:
•	mức độ chi tiết nhỏ hơn có nghĩa là tạo ra nhiều mục nhập chỉ mục hơn và việc tra cứu các hàng trong phân vùng theo cột đối chiếu sẽ nhanh hơn
•	nhưng, Cassandra sẽ giữ chỉ mục đối chiếu trong bộ nhớ cho các hàng nóng (như một phần của bộ đệm ẩn khóa), do đó, mức độ chi tiết lớn hơn có nghĩa là bạn có thể lưu vào bộ nhớ cache nhiều hàng nóng hơn
Giá trị mặc định: 64
77.	column_index_cache_size_in_kb
Mỗi mục nhập bộ đệm ẩn khóa được lập chỉ mục sstable (chỉ mục đối chiếu trong bộ nhớ được đề cập ở trên) vượt quá kích thước này sẽ không được lưu giữ trên heap. Điều này có nghĩa là chỉ thông tin phân vùng được giữ trên heap và các mục chỉ mục được đọc từ đĩa.
Lưu ý rằng kích thước này đề cập đến kích thước của thông tin chỉ mục được tuần tự hóa chứ không phải kích thước của phân vùng.
Giá trị mặc định: 2
78.	concurrent_compactors
Tùy chọn này được nhận xét theo mặc định.
Số lượng giao dịch đồng thời cho phép, KHÔNG bao gồm "giao dịch" xác thực để sửa chữa chống entropy. Các giao dịch đồng thời có thể giúp duy trì hiệu suất đọc trong một khối lượng công việc đọc / ghi hỗn hợp, bằng cách giảm thiểu xu hướng tích lũy các chuỗi nhỏ trong một giao dịch chạy dài. Mặc định thường ổn và nếu bạn gặp vấn đề với việc nén chạy quá chậm hoặc quá nhanh, bạn nên xem lại compaction_throughput_mb_per_sec trước.
concurrent_compactors mặc định là nhỏ hơn (số đĩa, số lõi), với tối thiểu là 2 và tối đa là 8.
Nếu các thư mục dữ liệu của bạn được hỗ trợ bởi SSD, bạn nên tăng số lõi này lên.
Giá trị mặc định: 1
79.	concurrent_validations
Tùy chọn này được nhận xét theo mặc định.
Số lần xác nhận sửa chữa đồng thời cho phép. Mặc định là không bị ràng buộc Các giá trị nhỏ hơn một được hiểu là không bị ràng buộc (mặc định)
Giá trị mặc định: 0
80.	concurrent_materialized_view_builders
Số lượng tác vụ trình tạo chế độ xem cụ thể hóa đồng thời được phép.
Giá trị mặc định: 1
81.	compaction_throughput_mb_per_sec
Nén tiết lưu thành tổng thông lượng đã cho trên toàn bộ hệ thống. Bạn chèn dữ liệu càng nhanh, bạn cần phải thu gọn nhanh hơn để giữ cho số đếm ngược, nhưng nói chung, đặt điều này thành 16 đến 32 lần tốc độ bạn đang chèn dữ liệu là quá đủ. Đặt giá trị này thành 0 sẽ tắt điều chỉnh. Lưu ý rằng tài khoản này dành cho tất cả các loại nén, bao gồm đầm nén xác nhận.
Giá trị mặc định: 16
82.	sstable_preemptive_open_interval_in_mb
Khi nén, (các) sstable thay thế có thể được mở ra trước khi chúng được viết hoàn toàn và được sử dụng thay cho các sstable trước đó cho bất kỳ phạm vi nào đã được viết. Điều này giúp chuyển các lần đọc giữa các bảng một cách trơn tru, giảm sự gián đoạn bộ nhớ cache của trang và giữ cho các hàng nóng luôn nóng
Giá trị mặc định: 50
83.	stream_entire_sstables
Tùy chọn này được nhận xét theo mặc định.
Khi được bật, cho phép Cassandra không sao chép luồng toàn bộ SSTables đủ điều kiện giữa các node, bao gồm mọi thành phần. Điều này làm tăng tốc độ truyền mạng đáng kể tùy thuộc vào điều chỉnh được chỉ định bởi stream_throughput_outbound_megabits_per_sec. Bật điều này sẽ giảm áp lực GC lên node gửi và nhận. Khi không được đặt, giá trị mặc định sẽ được bật. Mặc dù tính năng này cố gắng giữ cân bằng các đĩa, nhưng nó không thể đảm bảo. Tính năng này sẽ tự động bị vô hiệu hóa nếu mã hóa lóng được bật. Hiện tại điều này có thể được sử dụng với Leveled Compaction. Sau khi CASSANDRA-14586 được sửa chữa, các chiến lược đầm nén khác cũng sẽ được hưởng lợi khi được sử dụng kết hợp với CASSANDRA-6696.
Giá trị mặc định: true
84.	stream_throughput_outbound_megabits_per_sec
Tùy chọn này được nhận xét theo mặc định.
Điều chỉnh tất cả các quá trình truyền tệp phát trực tuyến ra ngoài trên node này thành tổng thông lượng nhất định tính bằng Mbps. Điều này là cần thiết vì Cassandra chủ yếu thực hiện IO tuần tự khi truyền dữ liệu trong quá trình khởi động hoặc sửa chữa, điều này có thể dẫn đến bão hòa kết nối mạng và làm giảm hiệu suất rpc. Khi không được đặt, giá trị mặc định là 200 Mbps hoặc 25 MB / s.
Giá trị mặc định: 200
85.	inter_dc_stream_throughput_outbound_megabits_per_sec
Tùy chọn này được nhận xét theo mặc định.
Điều chỉnh tất cả quá trình truyền tệp trực tuyến giữa các trung tâm dữ liệu, cài đặt này cho phép người dùng điều chỉnh lưu lượng luồng liên một chiều ngoài việc điều chỉnh tất cả lưu lượng luồng mạng như được định cấu hình với stream_throughput_outbound_megabits_per_sec Khi không được đặt, mặc định là 200 Mbps hoặc 25 MB / s
Giá trị mặc định: 200
86.	read_request_timeout_in_ms
Điều phối viên sẽ đợi bao lâu để hoàn tất các thao tác đọc. Giá trị thấp nhất có thể chấp nhận được là 10 ms.
Giá trị mặc định: 5000
87.	range_request_timeout_in_ms
Điều phối viên sẽ đợi bao lâu để quá trình quét seq hoặc chỉ mục hoàn tất. Giá trị thấp nhất có thể chấp nhận được là 10 ms.
Giá trị mặc định: 10000
88.	write_request_timeout_in_ms
Điều phối viên nên đợi bao lâu để hoàn tất quá trình viết. Giá trị thấp nhất có thể chấp nhận được là 10 ms.
Giá trị mặc định: 2000
89.	counter_write_request_timeout_in_ms
Điều phối viên nên đợi bao lâu để hoàn thành việc ghi vào bộ đếm. Giá trị thấp nhất có thể chấp nhận được là 10 ms.
Giá trị mặc định: 5000
90.	cas_contention_timeout_in_ms
Điều phối viên nên tiếp tục thử lại thao tác CAS trong thời gian bao lâu, cạnh tranh với các đề xuất khác cho cùng một hàng. Giá trị thấp nhất có thể chấp nhận được là 10 ms.
Giá trị mặc định: 1000
91.	truncate_request_timeout_in_ms
Điều phối viên sẽ đợi bao lâu để hoàn thành việc cắt ngắn (Điều này có thể lâu hơn nhiều, bởi vì trừ khi auto_snapshot bị tắt, chúng tôi cần xóa trước để chúng tôi có thể chụp nhanh trước khi xóa dữ liệu.) Giá trị thấp nhất có thể chấp nhận được là 10 mili giây.
Giá trị mặc định: 60000
92.	request_timeout_in_ms
Thời gian chờ mặc định cho các hoạt động khác, linh tinh. Giá trị thấp nhất có thể chấp nhận được là 10 ms.
Giá trị mặc định: 10000
93.	internode_application_send_queue_capacity_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Cài đặt phòng thủ để bảo vệ Cassandra khỏi các phân vùng mạng thực. Xem (CASSANDRA-14358) để biết thêm chi tiết.
94.	internode_tcp_connect_timeout_in_ms
Khoảng thời gian chờ thiết lập kết nối tcp lóng.
Giá trị mặc định: 2000
95.	internode_tcp_user_timeout_in_ms
Khoảng thời gian dữ liệu chưa được xác nhận được phép trên một kết nối trước khi chúng tôi loại bỏ kết nối Lưu ý rằng điều này chỉ được hỗ trợ trên Linux + epoll và nó dường như hoạt động kỳ lạ trên cài đặt 30000 (mất nhiều hơn 30 giây) kể từ Linux 4.12 . Nếu bạn muốn một cái gì đó cao, hãy đặt giá trị này thành 0, chọn mặc định của hệ điều hành và định cấu hình net.ipv4.tcp_retries2 sysctl thành ~ 8.
Giá trị mặc định: 30000
96.	internode_streaming_tcp_user_timeout_in_ms
Khoảng thời gian dữ liệu chưa được xác nhận được cho phép trên kết nối trực tuyến trước khi chúng tôi đóng kết nối.
Giá trị mặc định: 300000 (5 phút)
97.	internode_application_timeout_in_ms
Khoảng thời gian liên tục tối đa mà kết nối có thể không được thực hiện trong không gian ứng dụng.
Giá trị mặc định: 30000
Giới hạn toàn cầu, mỗi điểm cuối và mỗi kết nối được áp dụng đối với các thông báo được xếp hàng đợi để gửi đến các node khác và chờ được xử lý khi đến từ các node khác trong cluster. Các giới hạn này được áp dụng cho kích thước trực tuyến của tin nhắn được gửi hoặc nhận.
Giới hạn cơ bản cho mỗi liên kết được sử dụng riêng lẻ trước khi áp dụng bất kỳ điểm cuối hoặc giới hạn toàn cầu nào. Mỗi cặp node có ba liên kết: khẩn cấp, nhỏ và lớn. Vì vậy, bất kỳ node nhất định nào có thể có tối đa N * 3 * (internode_application_send_queue_capacity_in_bytes + internode_application_receive_queue_capacity_in_bytes) thông báo được xếp hàng đợi mà không có bất kỳ sự phối hợp nào giữa chúng mặc dù trên thực tế, với định tuyến nhận dạng mã thông báo, chỉ các node mã thông báo RF * mới cần giao tiếp với băng thông đáng kể.
Giới hạn mỗi điểm cuối được áp dụng cho tất cả các thông báo vượt quá giới hạn mỗi liên kết, đồng thời với giới hạn toàn cục, trên tất cả các liên kết đến hoặc từ một node duy nhất trong cluster. Giới hạn chung được áp dụng cho tất cả các thông báo vượt quá giới hạn mỗi liên kết, đồng thời với giới hạn mỗi điểm cuối, trên tất cả các liên kết đến hoặc từ bất kỳ node nào trong cluster.
Giá trị mặc định: 4194304 # 4MiB
98.	internode_application_send_queue_reserve_endpoint_capacity_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Giá trị mặc định: 134217728 # 128MiB
99.	internode_application_send_queue_reserve_global_capacity_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Giá trị mặc định: 536870912 # 512MiB
100.	internode_application_receive_queue_capacity_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Giá trị mặc định: 4194304 # 4MiB
101.	internode_application_receive_queue_reserve_endpoint_capacity_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Giá trị mặc định: 134217728 # 128MiB
102.	internode_application_receive_queue_reserve_global_capacity_in_bytes
Tùy chọn này được nhận xét theo mặc định.
Giá trị mặc định: 536870912 # 512MiB
103.	slow_query_log_timeout_in_ms
Bao lâu trước khi một node ghi lại các truy vấn chậm. Chọn các truy vấn mất nhiều thời gian hơn thời gian chờ này để thực thi, sẽ tạo ra một thông báo nhật ký tổng hợp, để có thể xác định các truy vấn chậm. Đặt giá trị này thành 0 để tắt ghi nhật ký truy vấn chậm.
Giá trị mặc định: 500
104.	cross_node_timeout
Tùy chọn này được nhận xét theo mặc định.
Cho phép trao đổi thông tin thời gian chờ hoạt động giữa các node để đo lường chính xác thời gian chờ yêu cầu. Nếu bị vô hiệu hóa, các bản sao sẽ cho rằng các yêu cầu đã được điều phối viên chuyển tiếp đến họ ngay lập tức, có nghĩa là trong điều kiện quá tải, chúng tôi sẽ lãng phí thêm nhiều thời gian để xử lý các yêu cầu đã quá thời hạn.
Cảnh báo: Người ta thường cho rằng người dùng đã thiết lập NTP trên các cluster của họ và đồng hồ được đồng bộ một cách khiêm tốn, vì đây là yêu cầu về tính đúng đắn chung của các lần ghi cuối cùng.
Giá trị mặc định: true
105.	streaming_keep_alive_period_in_secs
Tùy chọn này được nhận xét theo mặc định.
Đặt khoảng thời gian duy trì để phát trực tuyến Node này sẽ gửi một thông báo duy trì hoạt động theo định kỳ với khoảng thời gian này. Nếu node không nhận được thông báo duy trì hoạt động từ đồng đẳng trong 2 chu kỳ duy trì, phiên phát trực tiếp sẽ hết thời gian chờ và không thành công Giá trị mặc định là 300 giây (5 phút), có nghĩa là luồng bị dừng sẽ hết thời gian chờ trong 10 phút theo mặc định
Giá trị mặc định: 300
106.	streaming_connections_per_host
Tùy chọn này được nhận xét theo mặc định.
Giới hạn số lượng kết nối trên mỗi máy chủ lưu trữ để phát trực tuyến Tăng điều này khi bạn nhận thấy rằng các liên kết bị ràng buộc bởi CPU chứ không phải là mạng bị ràng buộc (ví dụ: một vài node có tệp lớn).
Giá trị mặc định: 1
107.	phi_convict_threshold
Tùy chọn này được nhận xét theo mặc định.
giá trị phi phải đạt được để máy chủ lưu trữ được đánh dấu xuống. hầu hết người dùng không bao giờ cần phải điều chỉnh điều này.
Giá trị mặc định: 8
108.	endpoint_snitch
endpoint_snitch - Đặt điều này thành một lớp triển khai IEndpointSnitch. Snitch có hai chức năng:
•	nó dạy cho Cassandra đủ về cấu trúc liên kết mạng của bạn để định tuyến các yêu cầu một cách hiệu quả
•	nó cho phép Cassandra phát tán các bản sao xung quanh cluster của bạn để tránh các lỗi tương quan. Nó thực hiện điều này bằng cách nhóm các máy thành “trung tâm dữ liệu” và “giá đỡ”. Cassandra sẽ cố gắng hết sức để không có nhiều hơn một bản sao trên cùng một “giá đỡ” (thực tế có thể không phải là một vị trí thực tế)
CASSANDRA SẼ KHÔNG CHO PHÉP BẠN CHUYỂN ĐỔI ĐẾN MỘT DỮ LIỆU TUYỆT ĐỐI KHÔNG THỂ PHẢN ỨNG ĐƯỢC CHÈN VÀO BỘ ĐIỀU CHỈNH. Điều này sẽ gây ra mất dữ liệu. Điều này có nghĩa là nếu bạn bắt đầu với SimpleSnitch mặc định, định vị mọi node trên “rack1” trong “datacenter1”, tùy chọn duy nhất của bạn nếu bạn cần thêm một trung tâm dữ liệu khác là GossipingPropertyFileSnitch (và PFS cũ hơn). Từ đó, nếu bạn muốn chuyển sang một snitch không tương thích như Ec2Snitch, bạn có thể thực hiện bằng cách thêm các node mới trong Ec2Snitch (sẽ định vị chúng trong một “trung tâm dữ liệu” mới) và ngừng hoạt động các node cũ.
Ngoài ra, Cassandra cung cấp:
SimpleSnitch:
Coi thứ tự Chiến lược là sự gần gũi. Điều này có thể cải thiện vị trí bộ nhớ cache khi tắt sửa chữa đọc. Chỉ thích hợp cho việc triển khai một trung tâm dữ liệu.
GossipingPropertyFileSnitch
Đây sẽ là snitch bạn nên sử dụng trong sản xuất. Giá đỡ và trung tâm dữ liệu cho node cục bộ được định nghĩa trong cassandra-rackdc.properties và được truyền đến các node khác thông qua gossip. Nếu cassandra-topology.properties tồn tại, nó được sử dụng như một dự phòng, cho phép di chuyển từ PropertyFileSnitch.
PropertyFileSnitch:
Vùng lân cận được xác định bởi giá đỡ và trung tâm dữ liệu, được định cấu hình rõ ràng trong cassandra-topology.properties.
Ec2Snitch:
Thích hợp cho việc triển khai EC2 trong một Khu vực. Tải thông tin Vùng và Vùng khả dụng từ API EC2. Vùng được coi là trung tâm dữ liệu và Vùng khả dụng là giá đỡ. Chỉ các IP riêng được sử dụng, vì vậy điều này sẽ không hoạt động trên nhiều Khu vực.
Ec2MultiRegionSnitch:
Sử dụng IP công cộng làm địa chỉ quảng bá để cho phép kết nối giữa các vùng. (Do đó, bạn cũng nên đặt địa chỉ gốc thành IP công cộng.) Bạn sẽ cần mở kho lưu trữ hoặc ssl_storage_port trên tường lửa IP công cộng. (Đối với lưu lượng nội bộ, Cassandra sẽ chuyển sang IP riêng sau khi thiết lập kết nối.)
RackInferringSnitch:
Vùng lân cận được xác định bởi giá đỡ và trung tâm dữ liệu, được giả định là tương ứng với octet thứ 3 và thứ 2 của địa chỉ IP của mỗi node, tương ứng. Trừ khi điều này xảy ra phù hợp với các quy ước triển khai của bạn, điều này tốt nhất được sử dụng làm ví dụ về việc viết một lớp Snitch tùy chỉnh và được cung cấp theo tinh thần đó.
Bạn có thể sử dụng Snitch tùy chỉnh bằng cách đặt tên này thành tên lớp đầy đủ của snitch, tên này sẽ được giả định là trên classpath của bạn.
Giá trị mặc định: SimpleSnitch
109.	dynamic_snitch_update_interval_in_ms
kiểm soát tần suất thực hiện phần tốn kém hơn trong tính toán điểm máy chủ
Giá trị mặc định: 100
110.	dynamic_snitch_reset_interval_in_ms
kiểm soát tần suất đặt lại tất cả các điểm số của máy chủ, cho phép máy chủ không tốt có thể khôi phục
Giá trị mặc định: 600000
111.	dynamic_snitch_badness_threshold
nếu được đặt lớn hơn 0, điều này sẽ cho phép 'ghim' các bản sao vào máy chủ để tăng dung lượng bộ nhớ cache. Ngưỡng độ xấu sẽ kiểm soát mức độ tồi tệ hơn của máy chủ được ghim trước khi snitch động sẽ thích các bản sao khác hơn nó. Giá trị này được biểu thị dưới dạng kép đại diện cho tỷ lệ phần trăm. Do đó, giá trị 0,2 có nghĩa là Cassandra sẽ tiếp tục thích các giá trị snitch tĩnh cho đến khi máy chủ được ghim kém hơn 20% so với tốc độ nhanh nhất.
Giá trị mặc định: 0,1
112.	server_encryption_options
Bật hoặc tắt mã hóa liên node JVM và mặc định netty cho các giao thức ổ cắm SSL được hỗ trợ và bộ mật mã có thể được thay thế bằng cách sử dụng các tùy chọn mã hóa tùy chỉnh. Điều này không được khuyến nghị trừ khi bạn có các chính sách quy định một số cài đặt nhất định hoặc cần phải tắt mật mã hoặc giao thức dễ bị tấn công trong trường hợp không thể cập nhật JVM. Các cài đặt tuân thủ FIPS có thể được định cấu hình ở cấp JVM và không liên quan đến việc thay đổi cài đặt mã hóa tại đây: https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/FIPS.html
LƯU Ý Hiện tại, không có tùy chọn mã hóa tùy chỉnh nào được bật Các tùy chọn lóng có sẵn là: all, none, dc, rack Nếu được đặt thành dc cassandra sẽ mã hóa lưu lượng giữa các DC Nếu được đặt thành rack cassandra sẽ mã hóa lưu lượng giữa các giá
Mật khẩu được sử dụng trong các tùy chọn này phải khớp với mật khẩu được sử dụng khi tạo kho khóa và kho tin cậy. Để biết hướng dẫn về cách tạo các tệp này, hãy xem: http://download.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#CreateKeystore
Giá trị mặc định (tùy chọn phức hợp) :
# set to true for allowing secure incoming connections
enabled: false
# If enabled and optional are both set to true, encrypted and unencrypted connections are handled on the storage_port
optional: false
# if enabled, will open up an encrypted listening socket on ssl_storage_port. Should be used
# during upgrade to 4.0; otherwise, set to false.
enable_legacy_ssl_storage_port: false
# on outbound connections, determine which type of peers to securely connect to. 'enabled' must be set to true.
internode_encryption: none
keystore: conf/.keystore
keystore_password: cassandra
truststore: conf/.truststore
truststore_password: cassandra
# More advanced defaults below:
# protocol: TLS
# store_type: JKS
# cipher_suites: [TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_DHE_RSA_WITH_AES_128_CBC_SHA,TLS_DHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA]
# require_client_auth: false
# require_endpoint_verification: false
113.	client_encryption_options
bật hoặc tắt mã hóa từ máy khách đến máy chủ.
Giá trị mặc định (tùy chọn phức hợp) :
enabled: false
# If enabled and optional is set to true encrypted and unencrypted connections are handled.
optional: false
keystore: conf/.keystore
keystore_password: cassandra
# require_client_auth: false
# Set trustore and truststore_password if require_client_auth is true
# truststore: conf/.truststore
# truststore_password: cassandra
# More advanced defaults below:
# protocol: TLS
# store_type: JKS
# cipher_suites: [TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_DHE_RSA_WITH_AES_128_CBC_SHA,TLS_DHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA]
114.	internode_compression
internode_compression kiểm soát xem lưu lượng giữa các node có được nén hay không. Có thể:
tất cả
tất cả lưu lượng được nén
dc
lưu lượng truy cập giữa các trung tâm dữ liệu khác nhau được nén
không ai
không có gì được nén.
Giá trị mặc định: dc
115.	inter_dc_tcp_nodelay
Bật hoặc tắt tcp_nodelay cho giao tiếp liên dc. Việc vô hiệu hóa nó sẽ dẫn đến việc gửi các gói mạng lớn hơn (nhưng ít hơn), giảm chi phí từ chính giao thức TCP, với cái giá là tăng độ trễ nếu bạn chặn phản hồi giữa các trung tâm dữ liệu.
Giá trị mặc định: false
116.	tracetype_query_ttl
TTL cho các loại dấu vết khác nhau được sử dụng trong quá trình ghi nhật ký của quá trình sửa chữa.
Giá trị mặc định: 86400
117.	tracetype_repair_ttl
Giá trị mặc định: 604800
118.	enable_user_defined_functions
Nếu không được đặt, tất cả các Tạm dừng GC lớn hơn gc_log_threshold_in_ms sẽ ghi nhật ký ở cấp INFO UDF (chức năng do người dùng xác định) bị tắt theo mặc định. Kể từ Cassandra 3.0, có một hộp cát để ngăn chặn việc thực thi mã độc.
Giá trị mặc định: false
119.	enable_scripted_user_defined_functions
Bật các UDF có tập lệnh (JavaScript UDF). Các UDF của Java luôn được bật, nếu enable_user_defined_functions là true. Bật tùy chọn này để có thể sử dụng UDF với “ngôn ngữ javascript” hoặc bất kỳ nhà cung cấp JSR-223 tùy chỉnh nào. Tùy chọn này không có hiệu lực, nếu enable_user_defined_functions là false.
Giá trị mặc định: false
120.	windows_timer_interval
Độ phân giải lập lịch và hẹn giờ hạt nhân Windows mặc định là 15,6ms để tiết kiệm năng lượng. Giảm giá trị này trên Windows có thể cung cấp độ trễ chặt chẽ hơn và thông lượng tốt hơn, tuy nhiên, một số môi trường ảo hóa có thể thấy tác động tiêu cực đến hiệu suất từ việc thay đổi cài đặt này bên dưới mặc định hệ thống của chúng. Công cụ sysinternals 'clockres' có thể xác nhận cài đặt mặc định của hệ thống của bạn.
Giá trị mặc định: 1
121.	transparent_data_encryption_options
Cho phép mã hóa dữ liệu ở trạng thái nghỉ (trên đĩa). Các nhà cung cấp khóa khác nhau có thể được cắm vào, nhưng mặc định đọc từ kho khóa kiểu JCE. Một kho khóa duy nhất có thể chứa nhiều khóa, nhưng khóa được tham chiếu bởi “key_alias” là khóa duy nhất sẽ được sử dụng để mã hóa opertaion; các khóa đã sử dụng trước đó vẫn có thể (và nên!) trong kho khóa và sẽ được sử dụng trong các hoạt động giải mã (để xử lý trường hợp xoay khóa).
Chúng tôi đặc biệt khuyên bạn nên tải xuống và cài đặt Tệp Chính sách Quyền hạn Sức mạnh Không giới hạn của Phần mở rộng Mã hóa Java (JCE) cho phiên bản JDK của bạn. (liên kết hiện tại: http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html )
Hiện tại, chỉ những loại tệp sau được hỗ trợ để mã hóa dữ liệu minh bạch, mặc dù sẽ có nhiều loại tệp hơn trong các bản phát hành cassandra trong tương lai: commitlog, gợi ý
Giá trị mặc định (tùy chọn phức hợp) :
enabled: false
chunk_length_kb: 64
cipher: AES/CBC/PKCS5Padding
key_alias: testing:1
# CBC IV length for AES needs to be 16 bytes (which is also the default size)
# iv_length: 16
key_provider:
  - class_name: org.apache.cassandra.security.JKSKeyProvider
    parameters:
      - keystore: conf/.keystore
        keystore_password: cassandra
        store_type: JCEKS
        key_password: cassandra
122.	tombstone_warn_threshold
AN TOÀN THRESHOLDS # 
Khi thực hiện quét, trong hoặc trên một phân vùng, chúng ta cần lưu giữ các bia mộ được nhìn thấy trong bộ nhớ để có thể trả lại cho điều phối viên, điều phối viên sẽ sử dụng chúng để đảm bảo các bản sao khác cũng biết về các hàng đã bị xóa. Với khối lượng công việc tạo ra nhiều bia mộ, điều này có thể gây ra các vấn đề về hiệu suất và thậm chí làm cạn kiệt hệ thống máy chủ. ( http://www.datastax.com/dev/blog/cassandra-anti-patterns-queues-and-queue-like-datasets ) Điều chỉnh các ngưỡng tại đây nếu bạn hiểu những nguy hiểm và muốn quét thêm bia mộ. Các ngưỡng này cũng có thể được điều chỉnh trong thời gian chạy bằng cách sử dụng StorageService mbean.
Giá trị mặc định: 1000
123.	tombstone_failure_threshold
Giá trị mặc định: 100000
124.	batch_size_warn_threshold_in_kb
Ghi nhật ký CẢNH BÁO trên bất kỳ kích thước lô nhiều phân vùng nào vượt quá giá trị này. 5kb mỗi đợt theo mặc định. Cần thận trọng khi tăng kích thước của ngưỡng này vì nó có thể dẫn đến sự không ổn định của node.
Giá trị mặc định: 5
125.	batch_size_fail_threshold_in_kb
Không thành công bất kỳ lô nhiều phân vùng nào vượt quá giá trị này. 50kb (ngưỡng cảnh báo 10x) theo mặc định.
Giá trị mặc định: 50
126.	unlogged_batch_across_partitions_warn_threshold
Ghi nhật ký CẢNH BÁO trên bất kỳ lô nào không thuộc loại LOGGED hơn khoảng trên nhiều phân vùng hơn giới hạn này
Giá trị mặc định: 10
127.	compaction_large_partition_warning_threshold_mb
Ghi cảnh báo khi nén các phân vùng lớn hơn giá trị này
Giá trị mặc định: 100
128.	gc_log_threshold_in_ms
Tùy chọn này được nhận xét theo mặc định.
GC Tạm dừng lớn hơn 200 ms sẽ được ghi ở mức INFO Ngưỡng này có thể được điều chỉnh để giảm thiểu việc ghi nhật ký nếu cần
Giá trị mặc định: 200
129.	gc_warn_threshold_in_ms
Tùy chọn này được nhận xét theo mặc định.
GC Tạm dừng lớn hơn gc_warn_threshold_in_ms sẽ được ghi ở mức WARN Điều chỉnh ngưỡng dựa trên yêu cầu thông lượng ứng dụng của bạn. Đặt thành 0 sẽ tắt tính năng này.
Giá trị mặc định: 1000
130.	max_value_size_in_mb
Tùy chọn này được nhận xét theo mặc định.
Kích thước tối đa của bất kỳ giá trị nào trong SSTables. Biện pháp an toàn để phát hiện sớm tham nhũng SSTable. Bất kỳ kích thước giá trị nào lớn hơn ngưỡng này sẽ dẫn đến việc đánh dấu SSTable là bị hỏng. Giá trị này phải là dương và nhỏ hơn 2048.
Giá trị mặc định: 256
131.	back_pressure_enabled
Cài đặt áp suất ngược # Nếu được bật, điều phối viên sẽ áp dụng chiến lược áp suất ngược được chỉ định bên dưới cho mỗi đột biến được gửi đến các bản sao, với mục đích giảm áp lực lên các bản sao quá tải.
Giá trị mặc định: false
132.	back_pressure_strategy
Đã áp dụng chiến lược gây áp lực ngược. Việc triển khai mặc định, RateBasedBackPressure, nhận ba đối số: tỷ lệ cao, yếu tố và loại luồng và sử dụng tỷ lệ giữa phản hồi đột biến đến và yêu cầu đột biến gửi đi. Nếu tỷ lệ dưới cao, tỷ lệ đột biến đi ra bị giới hạn theo tỷ lệ đến giảm bởi hệ số đã cho; nếu tỷ lệ trên cao, giới hạn tỷ lệ được tăng lên theo hệ số đã cho; yếu tố như vậy thường được định cấu hình tốt nhất trong khoảng từ 1 đến 10, sử dụng các giá trị lớn hơn để phục hồi nhanh hơn với chi phí là các đột biến có khả năng bị giảm nhiều hơn; giới hạn tốc độ được áp dụng theo loại luồng: nếu NHANH, tốc độ đó được giới hạn ở tốc độ của bản sao nhanh nhất, nếu CHẬM ở tốc độ của bản sao chậm nhất. Các chiến lược mới có thể được thêm vào. Người triển khai cần triển khai org.apache.cassandra.net.
133.	otc_coalescing_strategy
Tùy chọn này được nhận xét theo mặc định.
Chiến lược hợp tác # Hợp tác nhiều thông điệp hóa ra lại tăng đáng kể thông lượng xử lý thông báo (nghĩ rằng tăng gấp đôi hoặc hơn). Trên kim loại trần, tầng cho thông lượng xử lý gói đủ cao mà nhiều ứng dụng sẽ không nhận thấy, nhưng trong môi trường ảo hóa, điểm mà ứng dụng có thể bị ràng buộc bởi quá trình xử lý gói mạng có thể thấp một cách đáng ngạc nhiên so với thông lượng xử lý tác vụ điều đó có thể thực hiện được bên trong máy ảo. Không phải là kim loại trần không được hưởng lợi từ các thông điệp liên kết, mà là số lượng gói tin mà giao diện mạng kim loại trần có thể xử lý đủ cho nhiều ứng dụng mà không xảy ra tình trạng đói tải ngay cả khi không liên kết. Có những lợi ích khác khi liên kết các thông điệp mạng khó tách biệt hơn với một số liệu đơn giản như tin nhắn trên giây. Bằng cách kết hợp nhiều tác vụ lại với nhau, một chuỗi mạng có thể xử lý nhiều thông báo với chi phí cho một chuyến đi để đọc từ một ổ cắm và tất cả các công việc gửi tác vụ có thể được thực hiện đồng thời giúp giảm chuyển đổi ngữ cảnh và tăng tính thân thiện với bộ đệm của quá trình xử lý thông báo mạng. Xem CASSANDRA-8692 để biết thêm chi tiết.
Chiến lược sử dụng để kết hợp các thông điệp trong OutboundTcpConnection. Có thể được cố định, trung bình di chuyển, timehorizon, vô hiệu hóa (mặc định). Bạn cũng có thể chỉ định một lớp con của CoalescingStrategies.CoalescingStrategy theo tên.
Giá trị mặc định: ĐÃ TẮT
134.	otc_coalescing_window_us
Tùy chọn này được nhận xét theo mặc định.
Cần bao nhiêu micro giây để chờ kết hợp lại. Đối với chiến lược cố định, đây là khoảng thời gian sau khi nhận được tin nhắn đầu tiên trước khi nó được gửi cùng với bất kỳ tin nhắn đi kèm nào. Đối với đường trung bình, đây là khoảng thời gian tối đa sẽ được chờ đợi cũng như khoảng thời gian mà các thông báo phải đến trung bình để kích hoạt liên kết.
Giá trị mặc định: 200
135.	otc_coalescing_enough_coalesced_messages
Tùy chọn này được nhận xét theo mặc định.
Đừng cố gắng kết hợp các tin nhắn nếu chúng ta đã nhận được nhiều tin nhắn đó. Giá trị này phải lớn hơn 2 và nhỏ hơn 128.
Giá trị mặc định: 8

<span style="color:#c7254e">otc_backlog_expiration_interval_ms</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định
- Bao nhiêu mili giây phải chờ giữa hai lần hết hạn chạy trên phần tồn đọng (hàng đợi) của OutboundTcpConnection. Hết hạn được thực hiện nếu bản tin đang chất đống trong phần tồn đọng. Các bản tin có thể được cho vào hết hạn để giải phóng bộ nhớ do các bản tin hết hạn sử dụng. Khoảng thời gian phải nằm trong khoảng từ 0 đến 1000 và trong hầu hết các cài đặt, giá trị mặc định sẽ phù hợp. Giá trị nhỏ hơn có thể hết hạn thông báo sớm hơn một chút với chi phí tốn nhiều thời gian hơn của CPU và tranh chấp hàng đợi trong khi lặp lại các thông báo tồn đọng. Khoảng 0 vô hiệu hóa mọi thời gian chờ, đó là hành vi của các phiên bản Cassandra trước đây.
- *Giá trị mặc định*: 200

<span style="color:#c7254e">ideal_consistency_level</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định
- Theo dõi số liệu trên mỗi keyspace cho biết liệu bản sao có đạt được mức nhất quán lý tưởng để ghi mà không bị hết thời gian hay không. Điều này khác với mức độ nhất quán được yêu cầu bởi mỗi lần ghi có thể thấp hơn để tạo điều kiện cho tính khả dụng.
- *Giá trị mặc định*: EACH_QUORUM

<span style="color:#c7254e">automatic_sstable_upgrade</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định
- Tự động nâng cấp sstable sau khi nâng cấp - nếu không có việc nén thông thường để thực hiện, sstable cũ nhất không được nâng cấp sẽ được nâng cấp lên phiên bản mới nhất
- *Giá trị mặc định*: false

<span style="color:#c7254e">max_concurrent_automatic_sstable_upgrades</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định. Giới hạn số lượng nâng cấp sstable đồng thời
- *Giá trị mặc định*: 1

<span style="color:#c7254e">audit_logging_options</span>
- Ghi nhật ký kiểm tra - Ghi lại mọi yêu cầu lệnh CQL, xác thực cho một node. Xem tài liệu trên audit_logging để biết chi tiết đầy đủ về các tùy chọn cấu hình khác nhau.

<span style="color:#c7254e">full_query_logging_options</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định.
- Các tùy chọn mặc định cho ghi nhật ký truy vấn đầy đủ - những tùy chọn này có thể thay đổi bằng dòng lệnh nodetool enablefullquerylog

<span style="color:#c7254e">corrupted_tombstone_strategy</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định.
- Xác thực đánh dấu khi đọc và nén có thể là "disabled", "warn" hoặc "exception"
- *Giá trị mặc định*: disabled

<span style="color:#c7254e">diagnostic_events_enabled</span>
- Nếu được enabled, sự kiện chẩn đoán có thể hữu ích để khắc phục sự cố vận hành. Các sự kiện phát ra chứa thông tin chi tiết về trạng thái nội bộ và các mối quan hệ thời gian giữa các sự kiện, có thể truy cập từ máy khách thông qua JMX.
- *Giá trị mặc định*: false

<span style="color:#c7254e">native_transport_flush_in_batches_legacy</span>
- Tùy chọn này được cho vào phần ghi chú theo mặc định.
- Sử dụng kết hợp bản tin TCP truyền tải gốc. Nếu khi nâng cấp lên 4.0, bạn thấy thông lượng của mình giảm và đặc biệt là bạn chạy một nhân cũ hoặc có rất ít kết nối máy khách thì tùy chọn này có thể đáng để đánh giá.
- *Giá trị mặc định*: false

<span style="color:#c7254e">repaired_data_tracking_for_range_reads_enabled</span>
- Cho phép theo dõi trạng thái dữ liệu đã sửa chữa trong quá trình đọc và so sánh giữa các bản sao sự không khớp giữa các tập bản sao đã sửa chữa có thể được mô tả là đã được xác nhận hoặc chưa được xác nhận. Trong ngữ cảnh này, chưa được xác nhận chỉ ra rằng sự hiện diện của các phiên sửa chữa đang chờ xử lý, điểm đánh dấu phân vùng chưa được sửa chữa hoặc một số điều kiện khác có nghĩa là sự chênh lệch không thể được coi là kết thúc. Sự không phù hợp đã được xác nhận phải là nguyên nhân dẫn đến việc điều tra vì chúng có thể là dấu hiệu của việc hư hỏng hoặc mất dữ liệu. Có các cờ riêng biệt cho các lần đọc phạm vi so với phân vùng vì các lần đọc phân vùng đơn chỉ được theo dõi khi CL > 1 và thông báo không khớp xảy ra. Hiện tại, các truy vấn phạm vi không sử dụng thông báo vì vậy nếu được bật để đọc phạm vi, tất cả các lần đọc phạm vi sẽ bao gồm theo dõi dữ liệu đã sửa chữa. Vì điều này thêm một số chi phí, các nhà khai thác có thể muốn vô hiệu hóa nó trong khi vẫn bật nó để đọc phân vùng.
- *Giá trị mặc định*: false

<span style="color:#c7254e">repaired_data_tracking_for_partition_reads_enabled</span>
- *Giá trị mặc định*: false

<span style="color:#c7254e">report_unconfirmed_repaired_data_mismatches</span>
- Nếu false, chỉ những điểm không phù hợp đã được xác nhận mới được báo cáo. Nếu true, một chỉ số riêng cho các điểm không khớp chưa được xác nhận cũng sẽ được ghi lại. Điều này là để tránh tín hiệu tiềm ẩn: các vấn đề nhiễu chưa được xác nhận, các vấn đề không phù hợp sẽ ít bị xử lý hơn các vấn đề đã được xác nhận.
- *Giá trị mặc định*: false

<span style="color:#c7254e">enable_materialized_views</span>
- TÍNH NĂNG THỬ NGHIỆM
- Cho phép tạo chế độ xem cụ thể hóa trên node này. Chế độ xem cụ thể hóa được coi là thử nghiệm và không được khuyến khích sử dụng trong sản xuất.
- *Giá trị mặc định*: false

<span style="color:#c7254e">enable_sasi_indexes</span>
- Cho phép tạo chỉ mục SASI trên node này. Các chỉ số SASI được coi là thử nghiệm và không được khuyến khích sử dụng trong sản xuất.
- *Giá trị mặc định*: false

<span style="color:#c7254e">enable_transient_replication</span>
- Cho phép tạo keyspaces được sao chép tạm thời trên node này. Sao chép nhất thời là thử nghiệm và không được khuyến khích sử dụng trong sản xuất.
- *Giá trị mặc định*: false


## Tài liệu tham khảo
- [Cassandra Documentation](https://cassandra.apache.org/doc/latest/configuration/index.html)


---