---
layout: post
comments: true
title:  "Cassandra - Các tham số chính trong cassandra yaml"
title2:  "Các tham số chính trong cassandra yaml"
date:   2021-04-20 10:15:00
permalink: 2021/04/20/cac-tham-so-chinh-trong-cassandra-yaml
mathjax: false
tags: Cassandra Database
category: Cassandra Database
sc_project: 12494739
sc_security: d785a534
img: /assets/cac-tham-so-chinh-trong-cassandra-yaml/Hinh1.jpg
summary: Các tham số chính trong file cấu hình cassandra-yaml

---

File thiết lập cassandra.yaml là file chứa các thiết lập được sử dụng khi khởi tạo một cluster mới hoặc khi đưa một node mới vào cluster có sẵn. File này cần được xem xét và thay đổi phù hợp mỗi khi khởi động node lần đầu. Các thiết lập này điều khiển cách một node hoạt động trong cluster ví dụ như giao tiếp giữa các node, phân vùng dữ liệu (data partitioning), vị trí đặt bản sao dữ liệu.

**Cluster_name**: là tên của cluster. Giá trị này phải giống nhau đối với mọi node nằm trong cluster.

```
cluster_name: 'Test Cluster'
```

**num_tokens**: số lượng nút ảo trong Cassandra, được sử dụng phân vùng dữ liệu, giúp cho dữ liệu được cân bằng trong các máy chủ vật lý. Theo kinh nghiệm thực tế thì tham số này không nên đặt quá cao, nên đặt là 8.


```
num_tokens: 8
```

**hints_directory**: vị trí lưu các file hints. Thư mục mặc định là $CASSANDRA_HOME/data/hints
```
hints_directory: /cassandra_data/hints
```

**authenticator **: dùng để kích hoạt việc xác thực tài khoản đăng nhập. Các tham số này không kích hoạt theo mặc định, do đó bạn nên kích hoạt tham số này để đảm bảo tính bảo mật của hệ thống.
- AllowAllAuthenticator nghĩa là không cần xác thực tài khoản;
- PasswordAuthenticator nghĩa là xác thực tài khoản bằng tên đăng nhập và mật khẩu. Tên đăng nhập và mật khẩu được lưu trong bảng *system_auth.credentials*. Nên thay đổi hệ số sao chép của bảng này để đảm bảo việc truy cập không bị gián đoạn. Nếu sử dụng tham số này thì *CassandraRoleManager* cũng phải được sử dụng

```
authenticator: PasswordAuthenticator
```

**authorizer**: dùng để kích hoạt việc phân quyền đăng nhập. Các tham số này không kích hoạt theo mặc định, do đó bạn nên kích hoạt tham số này để đảm bảo tính bảo mật của hệ thống.
- AllowAllAuthorizer nghĩa là cho phép bất kỳ hành động gì đối với bất kỳ tài khoản này;
- CassandraAuthorizer nghĩa là các tài khoản được phân quyền khác nhau, các quyền được lưu trong bảng *system_auth.permissions*. Nên thay đổi hệ số sao chép của bảng này để đảm bảo việc truy cập không bị gián đoạn.

```
authorizer: CassandraAuthorizer
```

**role_manager**: dùng để quản lý việc phân quyền đăng nhập. Nên thay đổi hệ số sao chép của keyspace *system_auth* để đảm bảo việc truy cập không bị gián đoạn.

```
role_manager: CassandraRoleManager
```

**Partitioner**: (Phân vùng dữ liệu) là việc bạn quyết định dữ liệu được phân tán như thế nào trên các node trong cluster (bao gồm cả các bản sao).
- RandomPartitioner: phân vùng ngẫu nhiên, phân phối dữ liệu đồng nhất trên toàn bộ cụm dựa trên các giá trị băm MD5
- ByteOrderedPartitioner: phân vùng theo thứ tự theo từng key byte
- Murmur3Partitioner: là phân vùng mặc định, phân phối đồng nhất dữ liệu trên toàn bộ cụm dựa trên các giá trị băm MurmurHash, cung cấp hiệu suất băm nhanh hơn và cải thiện hơn RandomPartitioner

**data_file_directories**: vị trí lưu các dữ liệu về column family (SSTables). Thư mục mặc định là $CASSANDRA_HOME/data/data
```
data_file_directories:
    - /cassandra_data/data
```

**commitlog_directory**: vị trí lưu các file commitlog. Để tối ưu hoá tốc độ ghi, DataStax khuyên bạn nên để commit log ở một ổ đĩa khác với ổ lưu trữ file dữ liệu ( lý tưởng là khác một cách vật lý, không phải là khách theo kiểu logic). Thư mục mặc định là $CASSANDRA_HOME/data/commitlog
```
commitlog_directory: /cassandra_data/commitlog
```


**Saved_caches_directory**: Là đường dẫn tới nơi column family key và row cache được lưu trữ. Thư mục mặc định là $CASSANDRA_HOME/data/saved_caches
```
saved_caches_directory: /cassandra_data/saved_caches
```

**Seed_provider**: là một pluggable interface để cung cấp một danh sách các seed node.
**Seeds**: Khi một node tham gia vào cluster, nó sẽ liên lạc tới các seed node để quyết định cấu trúc hình học của ring và lấy thông tin gossip từ các node khác trong cluster. Mọi node trong cluster phải có cùng một danh sách các seed node. Trong một cluster trải trên nhiều data center, mỗi data center phải bao gồm ít nhất một seed node. Giá trị của tham số này sẽ giống với tham số **listen_address** và **rpc_address**. list IP trong seeds càng nhiều thì tính fault tolerance càng cao, tuy nhiên lại làm giảm gossipping performance. thông thường 3 seed node trên một data center

```
seed_provider:
    - class_name: SEED_PROVIDER
      parameters:
          - seeds: "192.168.1.10", "192.168.1.11","192.168.1.12"
```

**Listen_address**: Là địa chỉ IP hay hostname mà các Cassandra node khác sẽ sử dụng nó để kết nối tới node này. Nếu bạn để trống, bạn cần có một dải hostname được thiết lập chính xác trên tất cả các node trong cluster để các hostname đó có thể chuyển tới địa chỉ IP chính xác của node này (sử dụng /etc/hostname, /etc/hosts hoặc là DNS).

```
listen_address: 192.168.0.101
```

**Rpc_address**: Địa chỉ nghe của remote procedure call ( các kết nối tới client). Để nghe được tất cả các giao interface đã được thiết lập, đặt giá trị là 0.0.0.0. Nếu bạn để trống, bạn cần có một dải hostname được thiết lập chính xác trên tất cả các node trong cluster để các hostname đó có thể chuyển tới địa chỉ IP chính xác của node này (sử dụng /etc/hostname, /etc/hosts hoặc là DNS). Giá trị mặc định : localhost. Giá trị được cho phép: một địa chỉ IP, một hostname, hoặc để trông để dẫn tới một địa chỉ sử dụng hostname được thiết lập ở node.

```
rpc_address: 192.168.0.101
```

**Broadcast_address**: Nếu cluster của bạn được deploy trên nhiều vùng của Amazon EC2 (và bạn sử dụng EC2MultiRegionSnitch), bạn nên đặt broadcast_address là địa chỉ IP public của node (và địa chỉ IP private cho listen_address). Nếu bạn không thiết lập broadcast_address, mặc định nó sẽ là địa chỉ giống listen_address.

```
broadcast_address: 192.168.0.101
```

**Initial_token**: Giá trị này gán vị trị token của node trong ring và gán một khoảng dữ liệu cho node khi nó khởi động lần đầu. Initial_token cũng có thể không cần thiết lập khi đưa một node mới vào cluster sẵn có. Nếu không, giá trị token phụ thuộc vào partitioner mà bạn đang sử dụng. Với một partitioner ngẫu nhiên, giá trị này sẽ nằm trong khoảng từ 0 tới 2 mũ 127. Với ByteOrderPreservingPartitioner, giá trị này sẽ là một mảng byte của các giá trị hex nằm trong giá trị row key thực sự của bạn. Với OrderPreservingPartitioner và CollatedOrderPreservingPartitioner, giá trị này sẽ là một chuỗi ký tự UTF-8 dựa trên giá trị row key thực sự của bạn.

**Lưu ý**: Nếu sử dụng các nút ảo (vnodes), bạn không cần phải tính toán các mã thông báo. Nếu không sử dụng vnodes, bạn phải tính toán các mã thông báo để gán cho tham số *initial_token*.

**Endpoint_snitch**: Đặt snitch để thiết lập vị trí node và tìm đường cho request. Snitch trong Cassandra gồm có các kiểu:

```
org.apache.cassandra.locator.SimpleSnitch

org.apache.cassandra.locator.RackInferringSnitch

org.apache.cassandra.locator.PropertyFileSnitch

org.apache.cassandra.locator.Ec2Snitch
```



## Tài liệu tham khảo
- [Cassandra Documentation](https://cassandra.apache.org/doc/latest/configuration/cass_yaml_file.html)
- [datastax](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/configuration/configCassandra_yaml.html)
- [itfromzero](https://itfromzero.com/database/cassandra/cassandra-yaml-va-nodetool-utility-trong-cassandra.html)


---