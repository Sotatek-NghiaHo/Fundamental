# Triển khai Redis Sentinel với 3 Server

## Mô hình

| Server  | IP             | Role             |
| ------- | -------------- | ---------------- |
| servera | 192.168.38.127 | master + sentinel |
| serverb | 192.168.38.128 | slave + sentinel  |
| serverc | 192.168.38.129 | sentinel          |

---

## 1. Cài đặt Redis trên 3 server
Docs: https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/rpm/

•	Kiểm tra phiên bản Redis:  
```
[root@serverb ~]# redis-cli --version
redis-cli 8.2.1
```

### Master (servera - 192.168.38.127)
Chỉnh sửa cấu hình: `vi /etc/redis/redis.conf`
```bash
bind 192.168.38.127
requirepass passwd
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

### Slave (serverb - 192.168.38.128)
Chỉnh sửa cấu hình: `vi /etc/redis/redis.conf`
```
bind 192.168.38.128
replicaof 192.168.38.127 6379
protected-mode no
daemonize no
masterauth passwd
```

Khởi động lại dịch vụ:
```
systemctl restart redis
systemctl enable redis
```

**Kiểm tra vai trò:**
```
[root@servera ~]# redis-cli -h 192.168.38.127 -p 6379  info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.38.128,port=6379,state=online,offset=2188498,lag=1
master_failover_state:no-failover
master_replid:d13044520141abe80859d696de59d79026a8dfec
master_replid2:336cd9de9912fa8b8a61c35bcfd8453eaa9225ca
master_repl_offset:2188498
second_repl_offset:2135696
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2135696
repl_backlog_histlen:52803

---

[root@servera ~]# redis-cli -h 192.168.38.128 -p 6379  info replication
# Replication
role:slave
master_host:192.168.38.127
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:2242148
slave_repl_offset:2242148
replica_full_sync_buffer_size:0
replica_full_sync_buffer_peak:0
master_current_sync_attempts:1
master_total_sync_attempts:1
master_link_up_since_seconds:742
total_disconnect_time_sec:0
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:d13044520141abe80859d696de59d79026a8dfec
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:2242148
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2136589
repl_backlog_histlen:105560

```

Firewall
```


```

### Kiểm tra đồng bộ hóa
Trên máy Master, thêm 1 key value
```
redis-cli -h 192.168.38.127
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
bind <ip-server>

sentinel monitor mymaster 192.168.38.127 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1

 
# Ap dung
redis-sentinel /etc/redis/sentinel/sentinel.conf
```
## Test Failover
```bash
# Server master
[root@servera ~]# systemctl stop redis

# Server bat ky - server slave
[root@serverb ~]# redis-cli -h 192.168.38.127 -p 26379
192.168.38.127:26379> SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
1) "192.168.38.127"
2) "6379"
192.168.38.127:26379> SENTINEL SENTINELS mymaster
1)  1) "name"
    2) "2ffdb2d0ebde9d1e5b87c92cb3ace6801a28bdca"
    3) "ip"
    4) "192.168.38.129"
    5) "port"
    6) "26379"
    ...
2)  1) "name"
    2) "ef4e3230f8b417b417da1147caa40aba230b269a"
    3) "ip"
    4) "192.168.38.128"
    5) "port"
    6) "26379"
    ...
 

```
