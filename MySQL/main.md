# -------------MYSQL | MYSQL SQL REPLICATION-------------
---
Required:  
- MySQL installation and configuration
- Study configurations then setup successfully
- Configure systems to boot into a specific target automatically
- Setup Master-Slave replication
- Monitor and troubleshoot replication
---
**3.2. Installing MySQL**  
1. Install MySQL server packages:
```
dnf install mysql-server -y
```
2. Start the mysqld service:
```
systemctl start mysqld.service
```
3. Enable the mysqld service to start at boot:
```
systemctl enable mysqld.service
```
4. Recommended: To improve security when installing MySQL, run the following command:
```
mysql_secure_installation
```
Lệnh này sẽ khởi chạy một tập lệnh tương tác hoàn toàn, nhắc nhở từng bước trong quy trình. Tập lệnh này cho phép bạn cải thiện bảo mật theo những cách sau:
- Đặt mật khẩu cho tài khoản `root`
- Xóa người dùng ẩn danh
- Không cho phép đăng nhập `root` từ xa (bên ngoài máy chủ cục bộ)

**3.3. Configuring MySQL**  
1. Edit the [mysqld] section of the `/etc/my.cnf.d/mysql-server.cnf` file.  
- bind-address - is the address on which the server listens. Possible options are:
  - a host name
  - an IPv4 address
  - an IPv6 address
```bash
[mysqld]
bind-address = 0.0.0.0      # Lắng nghe trên tất cả địa chỉ IPv4
# bind-address = 127.0.0.1  # Chỉ cho phép kết nối từ localhost
# bind-address = ::         # Cho phép IPv6
```
- skip-networking - controls whether the server listens for TCP/IP connections. Possible values are:
  - 0 - to listen for all clients
  - 1 - to listen for local clients only
```bash
skip-networking = 0   # Cho phép kết nối TCP/IP (mặc định)
# skip-networking = 1 # Chỉ cho kết nối local socket (unix domain socket)
```
- port - the port on which MySQL listens for TCP/IP connections.
```
port = 3306
```
2. Restart the mysqld service:
```
systemctl restart mysqld.service
```

**3.6. Replicating MySQL**  
Để thiết lập sao chép trong MySQL, bạn phải: 
- Cấu hình máy chủ nguồn 
- Cấu hình máy chủ bản sao 
- Tạo người dùng sao chép trên máy chủ nguồn 
- Kết nối máy chủ bản sao với máy chủ nguồn

Trong mô hình MySQL Replication:  
- Source server (Master): Là máy chủ chính, nơi dữ liệu được ghi đầu tiên.
- Replica server (Slave): Là máy chủ phụ, tự động đồng bộ dữ liệu từ Source qua cơ chế replication.

***3.6.1. Configuring a MySQL source server*** 
1. Để Source server hoạt động đúng cho replication, ta cần chỉnh trong file cấu hình `/etc/my.cnf.d/mysql-server.cnf` (trong phần [mysqld]).

- `bind-address=source_ip_address`
  - Cho phép các Replica kết nối đến Source.
  - source_ip_address thường là IP của Source server trong mạng LAN/WAN.
  - Ví dụ: `bind-address = 192.168.1.100`
- `server-id=id`
  - Mỗi server trong hệ thống replication cần ID duy nhất (số nguyên, thường > 1).
  - Ví dụ: `server-id = 1`
- `log_bin=path_to_source_server_log`
  - Bật Binary Log, nơi lưu lại toàn bộ các thay đổi (INSERT, UPDATE, DELETE).
  - Các Replica sẽ đọc log này để đồng bộ dữ liệu.
  - Ví dụ: `log_bin = /var/log/mysql/mysql-bin.log`
- `gtid_mode=ON`
  - Bật `GTID` (Global Transaction Identifier) – giúp MySQL theo dõi transaction một cách duy nhất trên toàn hệ thống.
  - Thuận tiện khi Replica cần đồng bộ, tránh lặp lại hoặc bỏ sót.
- `enforce-gtid-consistency=ON`
  - Ép buộc mọi câu lệnh phải tuân thủ tính nhất quán GTID, nghĩa là chỉ cho phép chạy những câu lệnh có thể được log bằng GTID.
  - Nhờ đó tránh lỗi khi replicate.

*Các tùy chọn bổ sung (không bắt buộc)*
- `binlog_do_db=db_name`
  - Chỉ replicate một số database được chọn.
  - Nếu muốn nhiều DB thì lặp lại option:
```
binlog_do_db = sales
binlog_do_db = hr
binlog_do_db = accounting
```
- `binlog_ignore_db=db_name`
  - Ngược lại, bỏ qua một số database khi replicate.
  - Ví dụ: `binlog_ignore_db = test`

2. Restart the mysqld service:
```
systemctl restart mysqld.service
```

***3.6.2 Configuring a MySQL replica server***
1. Bao gồm các tùy chọn sau trong tệp /etc/my.cnf.d/mysql-server.cnf trong phần [mysqld]:  
- `server-id=id`
  - Giống như Source server, mỗi Replica cũng cần một ID duy nhất trong hệ thống replication.
  - Ví dụ: `server-id = 2`
- `relay-log=path_to_replica_server_log`
  - Relay log là file log trung gian trên Replica.
  - Replica sẽ nhận dữ liệu từ Source → ghi vào relay log → apply vào DB local.
  - Ví dụ: `relay-log = /var/log/mysql/mysql-relay-bin.log`
- `log_bin=path_to_replica_server_log`
  - Bật binary log trên Replica.
  - Không bắt buộc, nhưng nên bật nếu bạn muốn Replica này có thể làm Source cho Replica khác (chuỗi replication).
  - Ví dụ: `log_bin = /var/log/mysql/mysql-bin.log`
- `gtid_mode=ON`
  - Bật GTID (Global Transaction Identifier).
  - Đảm bảo Replica đồng bộ chính xác transaction với Source.
- `enforce-gtid-consistency=ON`
  - Ép các lệnh phải tương thích với GTID, tránh lỗi trong replication.
- `log-replica-updates=ON`
  - Khi Replica nhận transaction từ Source và áp dụng, nó cũng sẽ ghi transaction đó vào binary log của mình.
  - Cần thiết nếu Replica đóng vai trò trung gian trong replication chain.
- `skip-replica-start=ON`
  - Khi MySQL khởi động lại, Replica không tự động chạy replication threads.
  - Điều này giúp bạn kiểm soát việc start/stop replication thủ công (qua lệnh `START REPLICA;`).

Các cấu hình tùy chọn
- binlog_do_db=db_name
  - Chỉ replicate các DB được chọn.
  - Ví dụ:
```
binlog_do_db = sales
binlog_do_db = hr
```
- `binlog_ignore_db=db_name`
  - Bỏ qua một số DB không muốn replicate.
  - Ví dụ: `binlog_ignore_db = test`


2. Restart the mysqld service:
```
systemctl restart mysqld.service
```

***3.6.3. Creating a replication user on the MySQL source server***
- Replica cần tài khoản MySQL đặc biệt để kết nối đến Source.
- User này chỉ có quyền REPLICATION SLAVE (trong MySQL 8.0+ gọi là REPLICATION REPLICA).
- Nhờ đó Replica có thể đọc binary log từ Source nhưng không có quyền thay đổi dữ liệu trên Source.

1. Tạo user replication
```bash
CREATE USER 'replication_user'@'replica_server_ip'
IDENTIFIED WITH mysql_native_password BY 'password';
```
- `replication_user`: tên tài khoản replication.
- `replica_server_ip`: IP của Replica server.
   - Nếu muốn cho phép mọi Replica kết nối → dùng `%`.  
    `CREATE USER 'replication_user'@'%' IDENTIFIED BY 'password';`
- `mysql_native_password`: đảm bảo tương thích (đặc biệt khi Replica dùng MySQL < 8).

2. Cấp quyền replication
```bash
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'replica_server_ip';
```
- Cho phép Replica đọc binary log.
- Lưu ý: từ MySQL 8.0.23 trở đi, nên dùng:
    ```
    GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'replication_user'@'replica_server_ip';
    ```
3. Tải lại bảng quyền (Reload the grant tables in the MySQL database)
```
FLUSH PRIVILEGES;
```
- Đảm bảo quyền vừa cấp được áp dụng ngay.

4. (Tùy chọn) Đặt Source về chế độ read-only
```
SET @@GLOBAL.read_only = ON;
```
- Thường dùng khi muốn đảm bảo Source không bị ghi thêm dữ liệu trong quá trình setup ban đầu.
- Nhưng sau khi thiết lập replication xong, nên để Source ở chế độ bình thường (không read-only), vì Source chính là nơi ghi dữ liệu.

***3.6.4. Connecting the replica server to the source server***
1. Đặt Replica ở trạng thái chỉ đọc (read-only)
```
SET @@GLOBAL.read_only = ON;
```
- Mục đích: ngăn ghi dữ liệu trực tiếp vào Replica trong lúc setup replication, tránh lệch dữ liệu.
- Sau khi kết nối thành công có thể tắt đi.
2. Cấu hình thông tin kết nối đến Source server
```bash
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='source_ip_address',
    SOURCE_USER='replication_user',
    SOURCE_PASSWORD='password',
    SOURCE_AUTO_POSITION=1;
```
- `SOURCE_HOST`: IP hoặc hostname của Source server.
- `SOURCE_USER`: tài khoản replication bạn đã tạo bên Source.
- `SOURCE_PASSWORD`: mật khẩu của user đó.
- `SOURCE_AUTO_POSITION=1`: bật GTID-based replication (thay vì chỉ định log file + vị trí).
  - GTID giúp Replica tự động đồng bộ đúng transaction mà không cần quan tâm đến file log nào.

3. Bắt đầu luồng replication
```
START REPLICA;
```
- Lệnh này sẽ khởi động các I/O thread (nhận dữ liệu từ Source) và SQL thread (ghi dữ liệu vào Replica).
4. Tắt chế độ read-only (nếu muốn)
```
SET @@GLOBAL.read_only = OFF;
```
- Thông thường:
  - Source: OFF (để có thể ghi dữ liệu).
  - Replica: có thể giữ ON để tránh ai vô tình ghi vào, trừ khi Replica cũng cần phục vụ ghi dữ liệu (ít khi dùng).
5. Kiểm tra trạng thái Replica
```
SHOW REPLICA STATUS\G
```
- Thông tin quan trọng:
  - Replica_IO_Running: YES (thread đang chạy).
  - Replica_SQL_Running: YES (thread đang áp dụng log).
  - Seconds_Behind_Source: 0 (Replica đang đồng bộ kịp thời).

*Debug khi có lỗi*
- Nếu Replica không start được hoặc báo lỗi về binary log, có thể bỏ qua một số sự kiện để tiếp tục:
```
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
START REPLICA;
```
⚠️ Cẩn thận: Bỏ qua sự kiện có thể làm Replica mất đồng bộ dữ liệu → chỉ nên dùng trong trường hợp test hoặc biết chắc event bị lỗi không quan trọng.

6. Dừng replication (nếu cần)
```
STOP REPLICA;
```

***3.6.5. Verification steps***
1. Tạo database test trên Source server  

Trên `Source` (Master), chạy:
```
CREATE DATABASE test_db_name;
```
- Ví dụ: `CREATE DATABASE demo_test;`

Nếu replication hoạt động bình thường → database `demo_test` sẽ tự động xuất hiện trên Replica server.

2. Kiểm tra trạng thái binary log
Trên `Source` hoặc `Replica`, chạy lệnh:
```
SHOW MASTER STATUS;
```
Kết quả sẽ có các cột như:

File	|Position	|Binlog_Do_DB|	Binlog_Ignore_DB	|Executed_Gtid_Set
---|---|---|---|---
mysql-bin.000003	|157			| | |5a1f9c1d-...:1-10

- File: Tên file binary log hiện tại.
- Position: Vị trí (offset) trong file log.
- Executed_Gtid_Set: Danh sách các GTID đã được thực thi trên server.

👉 Quan trọng: `Executed_Gtid_Set` KHÔNG được rỗng → nghĩa là GTID replication đang bật và các transaction đã được ghi nhận.

3. Kiểm tra trên Replica
Trên Replica, có thể dùng:
```
SHOW REPLICA STATUS\G 
```
Trong đó cần chú ý:
- `Replica_IO_Running`: Yes
- `Replica_SQL_Running`: Yes
- `Retrieved_Gtid_Set`: GTID nhận được từ Source.
- `Executed_Gtid_Set`: GTID đã áp dụng trên Replica.

👉 Giá trị `Executed_Gtid_Set` trên Replica phải khớp với Source (ít nhất là không thiếu transaction nào).

---
Docs: https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/9/html/configuring_and_using_database_servers/installing-mysql_assembly_using-mysql