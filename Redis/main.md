# Triển khai Redis Sentinel với 3 Server

## Mô hình

| Server  | IP             | Role             |
| ------- | -------------- | ---------------- |
| servera | 192.168.38.130 | master + sentinel |
| serverb | 192.168.38.132 | slave + sentinel  |
| serverc | 192.168.38.133 | sentinel          |

---

## 1. Cài đặt Redis trên 3 server
Docs: https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/rpm/

```
[root@servere ~]# yum list redis
 
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use "rhc" or "subscription-manager" to register.

Last metadata expiration check: 1:37:59 ago on Wed 24 Sep 2025 06:54:07 PM +07.
Installed Packages
redis.x86_64                    6.2.16-1.el9                    @local-redis
```

```
[root@servere ~]# yum list postgresql14
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use "rhc" or "subscription-manager" to register.

Last metadata expiration check: 1:38:44 ago on Wed 24 Sep 2025 06:54:07 PM +07.
Installed Packages
postgresql14.x86_64                 14.0-1                 @local-postgresql
[root@servere ~]# 

```

Install
```
yum install redis.x86_64 
yum install postgresql14.x86_64 
redis-server --version
Redis server v=6.2.16 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=85a2e85bf3a77649

```
1. Kiểm tra thư viện libpq
ls -l /usr/local/postgresql14/lib/libpq.so*


Bạn sẽ thấy file libpq.so.5 hoặc symlink trỏ đến nó.

2. Thêm thư viện vào hệ thống

Có nhiều cách, thường dùng nhất là cập nhật ld.so.conf:

echo "/usr/local/postgresql14/lib" > /etc/ld.so.conf.d/postgresql14.conf
ldconfig

3. Kiểm tra lại
/usr/local/postgresql14/bin/psql --version


Kết quả mong muốn:

psql (PostgreSQL) 14.0

4. (Tùy chọn) Thêm vào PATH

Để dùng psql trực tiếp thay vì phải gõ đường dẫn dài:
```
echo 'export PATH=/usr/local/postgresql14/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
```
[root@servere ~]# psql --version
psql (PostgreSQL) 14.0
[root@servere ~]# 

```


•	Kiểm tra phiên bản Redis:  
```
[root@serverb ~]# redis-cli --version
redis-cli 8.2.1
```

### Master (servera - 192.168.38.130)
Chỉnh sửa cấu hình: `vi /etc/redis/redis.conf`
```bash
bind 192.168.38.130
requirepass passwd
masterauth passwd
```
Khởi động lại dịch vụ:
```
systemctl restart redis
systemctl enable redis
```

Kiểm tra:
```
redis-cli
AUTH passwd
INFO REPLICATION
```

### Slave (serverb - 192.168.38.132)
Chỉnh sửa cấu hình: `vi /etc/redis/redis.conf`
```
bind 192.168.38.132
replicaof 192.168.38.130 6379
protected-mode no
daemonize no
masterauth passwd
requirepass passwd
```

Khởi động lại dịch vụ:
```
systemctl restart redis
systemctl enable redis
```

**Kiểm tra vai trò:**
```
[root@serverf ~]# redis-cli -h 192.168.38.130
192.168.38.130:6379> auth passwd
OK
192.168.38.130:6379> info replication
# Replication
role:slave
master_host:192.168.38.132
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:365096
slave_repl_offset:365096
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4df14148c852fe71713120516d0d42f790566727
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:365096
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:361185
repl_backlog_histlen:3912
192.168.38.130:6379> 

[root@serverf ~]# redis-cli -h 192.168.38.132
192.168.38.132:6379> auth passwd
OK
192.168.38.132:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.38.130,port=6379,state=online,offset=451313,lag=1
master_failover_state:no-failover
master_replid:4df14148c852fe71713120516d0d42f790566727
master_replid2:04350abba74a68813d30d88e9b3828d3abdb484a
master_repl_offset:451456
second_repl_offset:53522
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:451456
192.168.38.132:6379> 


```

Firewall  
Master-slave
```bash
[root@servere ~]# systemctl status firewalld.service 
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; pr>
     Active: active (running) since Wed 2025-09-24 19:54:52 +07; 23min ago
       Docs: man:firewalld(1)
   Main PID: 2743 (firewalld)
      Tasks: 4 (limit: 10706)
     Memory: 29.7M
        CPU: 1.086s
     CGroup: /system.slice/firewalld.service
             └─2743 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

Sep 24 19:54:52 servere systemd[1]: Starting firewalld - dynamic firewall d>
Sep 24 19:54:52 servere systemd[1]: Started firewalld - dynamic firewall da>
lines 1-13/13 (END)

firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.38.0/24" port port=6379 protocol=tcp accept'

firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.38.0/24" port port=26379 protocol=tcp accept'

firewall-cmd --reload
```


### Kiểm tra đồng bộ hóa
Trên máy Master, thêm 1 key value
```
redis-cli -h 192.168.38.130
SET key1 "Hello from Master"
``` 
Trên máy Slave, kiểm tra dữ liệu: 
```
redis-cli
GET key1
```
Nếu kết quả trả về "Hello from Master", đồng bộ hóa hoạt động bình thường


---
## 2. Cấu hình Sentinel trên 3 server 
```bash
port 26379
daemonize no
protected-mode no
bind 0.0.0.0

sentinel monitor mymaster 192.168.38.132 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster passwd

```
Để chạy Sentinel dưới dạng dịch vụ hệ thống: 
Tạo file dịch vụ (ví dụ: /etc/systemd/system/redis-sentinel.service): 
```bash

[root@servere ~]# which redis-server
/usr/bin/redis-server
[root@servere ~]# 


[root@servere ~]# cat /etc/systemd/system/redis-sentinel.service
[Unit]
Description=Redis Sentinel
After=network.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis/sentinel.conf --sentinel
User=redis
Group=redis

[Install]
WantedBy=multi-user.target

```
Kiem tra Sentinel
o	Kích hoạt và khởi động dịch vụ: 
```
systemctl daemon-reload
sudo systemctl enable redis-sentinel
sudo systemctl start redis-sentinel
```

•	Kết nối đến một Sentinel: 
```
redis-cli -h <sentinel_ip> -p 26379
```
•	Kiểm tra trạng thái master: 
```
SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
```
Kết quả sẽ trả về IP và port của master hiện tại (ví dụ: 192.168.1.10 6379).
•	Kiểm tra danh sách Sentinel: 
```
SENTINEL SENTINELS mymaster
```
Or
```
[root@serverf ~]# redis-cli -h 192.168.38.132 -p 26379 SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
1) "192.168.38.132"
2) "6379"
[root@serverf ~]# redis-cli -h 192.168.38.132 -p 26379 SENTINEL SENTINELS mymaster
1)  1) "name"
    2) "97e1c87dbbeec2322dd5ce2a62ef2cc30757a4e7"
    3) "ip"
    4) "192.168.38.130"
    5) "port"
    6) "26379"
    7) "runid"
    8) "97e1c87dbbeec2322dd5ce2a62ef2cc30757a4e7"
    9) "flags"
   10) "sentinel"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "168"
   19) "last-ping-reply"
   20) "168"
   21) "down-after-milliseconds"
   22) "30000"
   23) "last-hello-message"
   24) "257"
   25) "voted-leader"
   26) "?"
   27) "voted-leader-epoch"
   28) "1"
2)  1) "name"
    2) "78358f926f1cebdcd07f43f4bc7e1dfb10f65f43"
    3) "ip"
    4) "192.168.38.133"
    5) "port"
    6) "26379"
    7) "runid"
    8) "78358f926f1cebdcd07f43f4bc7e1dfb10f65f43"
    9) "flags"
   10) "sentinel"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "168"
   19) "last-ping-reply"
   20) "168"
   21) "down-after-milliseconds"
   22) "30000"
   23) "last-hello-message"
   24) "1185"
   25) "voted-leader"
   26) "?"
   27) "voted-leader-epoch"
   28) "1"
[root@serverf ~]# 


```

Server Seninel
```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.38.0/24" port port=26379 protocol=tcp accept'

firewall-cmd --reload

```

## Test Failover

```
[root@serverf ~]# tail /var/log/redis/sentinel.log 
1992:X 24 Sep 2025 19:53:15.945 # +failover-state-reconf-slaves master mymaster 192.168.38.130 6379
1992:X 24 Sep 2025 19:53:15.996 # +failover-end master mymaster 192.168.38.130 6379
1992:X 24 Sep 2025 19:53:15.996 # +switch-master mymaster 192.168.38.130 6379 192.168.38.132 6379
1992:X 24 Sep 2025 19:53:15.996 * +slave slave 192.168.38.130:6379 192.168.38.130 6379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:53:46.014 # +sdown slave 192.168.38.130:6379 192.168.38.130 6379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:55:24.093 # +sdown sentinel 97e1c87dbbeec2322dd5ce2a62ef2cc30757a4e7 192.168.38.130 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:55:40.752 # +sdown sentinel 78358f926f1cebdcd07f43f4bc7e1dfb10f65f43 192.168.38.133 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:59:51.101 # -sdown sentinel 97e1c87dbbeec2322dd5ce2a62ef2cc30757a4e7 192.168.38.130 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 20:00:06.972 # +new-epoch 2
1992:X 24 Sep 2025 20:00:07.500 # -sdown sentinel 78358f926f1cebdcd07f43f4bc7e1dfb10f65f43 192.168.38.133 26379 @ mymaster 192.168.38.132 6379

```
```bash
# Server master
[root@servera ~]# systemctl stop redis

# Server bat ky - server slave
[root@serverb ~]# redis-cli -h 192.168.38.130 -p 26379
192.168.38.130:26379> SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
1) "192.168.38.130"
2) "6379"
192.168.38.130:26379> SENTINEL SENTINELS mymaster
1)  1) "name"
    2) "2ffdb2d0ebde9d1e5b87c92cb3ace6801a28bdca"
    3) "ip"
    4) "192.168.38.133"
    5) "port"
    6) "26379"
    ...
2)  1) "name"
    2) "ef4e3230f8b417b417da1147caa40aba230b269a"
    3) "ip"
    4) "192.168.38.132"
    5) "port"
    6) "26379"
    ...
 

```
```
[root@serverf ~]# tail /var/log/redis/sentinel.log 
1992:X 24 Sep 2025 19:53:15.996 # +failover-end master mymaster 192.168.38.130 6379
1992:X 24 Sep 2025 19:53:15.996 # +switch-master mymaster 192.168.38.130 6379 192.168.38.132 6379
1992:X 24 Sep 2025 19:53:15.996 * +slave slave 192.168.38.130:6379 192.168.38.130 6379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:53:46.014 # +sdown slave 192.168.38.130:6379 192.168.38.130 6379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:55:24.093 # +sdown sentinel 97e1c87dbbeec2322dd5ce2a62ef2cc30757a4e7 192.168.38.130 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:55:40.752 # +sdown sentinel 78358f926f1cebdcd07f43f4bc7e1dfb10f65f43 192.168.38.133 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 19:59:51.101 # -sdown sentinel 97e1c87dbbeec2322dd5ce2a62ef2cc30757a4e7 192.168.38.130 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 20:00:06.972 # +new-epoch 2
1992:X 24 Sep 2025 20:00:07.500 # -sdown sentinel 78358f926f1cebdcd07f43f4bc7e1dfb10f65f43 192.168.38.133 26379 @ mymaster 192.168.38.132 6379
1992:X 24 Sep 2025 20:20:35.976 # -sdown slave 192.168.38.130:6379 192.168.38.130 6379 @ mymaster 192.168.38.132 6379
```

```
[root@serverf ~]# redis-cli -h 192.168.38.130
192.168.38.130:6379> auth passwd
OK
192.168.38.130:6379> info replication
# Replication
role:slave
master_host:192.168.38.132
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:365096
slave_repl_offset:365096
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4df14148c852fe71713120516d0d42f790566727
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:365096
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:361185
repl_backlog_histlen:3912
192.168.38.130:6379> 
[root@serverf ~]# redis-cli -h 192.168.38.132
192.168.38.132:6379> auth paswwd
(error) WRONGPASS invalid username-password pair or user is disabled.
192.168.38.132:6379> auth passwd
OK
192.168.38.132:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.38.130,port=6379,state=online,offset=451313,lag=1
master_failover_state:no-failover
master_replid:4df14148c852fe71713120516d0d42f790566727
master_replid2:04350abba74a68813d30d88e9b3828d3abdb484a
master_repl_offset:451456
second_repl_offset:53522
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:451456
192.168.38.132:6379> 
```


