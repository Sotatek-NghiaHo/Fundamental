# Redis Sentinel and PostgreSQL 14 Deployment Guide

This document outlines the prerequisites, deployment steps, and verification procedures for setting up Redis (with Sentinel high availability) and PostgreSQL 14, drawing exclusively from the provided technical excerpts.

## 1. Prerequisite

### Compute (OS, Hardware, Storage)

| Component | Requirement | Details | Citation |
| :--- | :--- | :--- | :--- |
| **Operating System** | RHEL 9.x (x86_64) | Red Hat Enterprise Linux 9.x 64-bit architecture | [1] |
| **CPU** | Minimum 2 cores | Recommended 4 cores or more | [1] |
| **Memory (Redis)** | 2 GB RAM or more | Required for Redis operations | [1] |
| **Memory (PostgreSQL)** | 4 GB RAM or more | Required for PostgreSQL operations | [1] |
| **Storage (Redis)** | 10 GB SSD | Must be I/O optimized | [1] |
| **Storage (PostgreSQL)** | 50 GB SSD or more | Required storage capacity | [1] |
| **Disk Format** | XFS or EXT4 | Acceptable file system formats | [1] |

### Network (Firewall, Security, Topology)

| Component | Requirement | Details | Citation |
| :--- | :--- | :--- | :--- |
| **Firewall** | Open required ports | Services require specific ports open for communication. | [1] |
| **Redis Client Port** | 6379 (TCP) | Client connection port | [1] |
| **Redis Sentinel Port** | 26379 (TCP) | Sentinel monitoring and communication port | [1] |
| **PostgreSQL Port** | 5432 (TCP) | PostgreSQL database connection port | [1] |
| **Security (SELinux)** | Enforcing or Permissive | Custom policy should be added for Redis/PostgreSQL if necessary | [1] |
| **Security (Admin)** | SSH key-based access | Recommended for administrative access | [1] |
| **Cluster Network** | VPN or private network | Recommended for cluster nodes [2] |

### Software and Tools

| Component | Version | Role | Citation |
| :--- | :--- | :--- | :--- |
| **Redis** | 6.2.16 | Database software | [1, 3, 4] |
| **PostgreSQL** | 14 | Database software | [1, 5] |
| **Tools** | psql, redis-cli, redis-sentinel, systemctl | Required utilities for management and operation | [2] |

## 2. Deployment: Steps

### 2.1. Installation

The installation process primarily utilizes `yum` for package management, targeting RHEL 9.x [1, 3, 5].

1.  **Install Redis and PostgreSQL:**
    ```bash
    yum install redis.x86_64
    yum install postgresql14.x86_64
    ```
    (Note: Installed versions are Redis 6.2.16-1.el9 and PostgreSQL 14.0-1) [3, 5].

2.  **Configure PostgreSQL Library Path (libpq):**
    Ensure the `libpq` library (e.g., `libpq.so.5`) is visible to the system by adding the path `/usr/local/postgresql14/lib` to the system configuration:
    ```bash
    echo "/usr/local/postgresql14/lib" > /etc/ld.so.conf.d/postgresql14.conf
    ldconfig
    ```
    [4]

3.  **Optional: Add psql to PATH:**
    For direct command line usage of `psql`:
    ```bash
    echo 'export PATH=/usr/local/postgresql14/bin:$PATH' >> ~/.bashrc
    source ~/.bashrc
    ```
    [6]

### 2.2. Redis Master-Slave Replication Setup

The recommended topology for Redis Sentinel includes 3 servers [2]:

| Server | IP Address | Role |
| :--- | :--- | :--- |
| `servera` | 192.168.38.130 | Master + Sentinel (Initial Master) | [2] |
| `serverb` | 192.168.38.132 | Slave + Sentinel (New Master after Failover tests) | [2, 7, 8] |
| `serverc` | 192.168.38.133 | Sentinel Only | [2] |

#### A. Master Configuration (e.g., `servera` 192.168.38.130):

Edit the Redis configuration file (`vi /etc/redis/redis.conf`) with essential security and binding settings:
```bash
bind 192.168.38.130
requirepass passwd
masterauth passwd
 Restart and enable the Redis service:
systemctl restart redis
systemctl enable redis
B. Slave Configuration (e.g.,  192.168.38.132):
Edit the configuration file (vi /etc/redis/redis.conf) to configure replication pointing to the initial master (192.168.38.130):
bind 192.168.38.132
replicaof 192.168.38.130 6379
protected-mode no
daemonize no
masterauth passwd
requirepass passwd
 Restart and enable the Redis service.
C. Firewall Configuration (Master-Slave Communication):
Ensure the firewall allows traffic on ports 6379 (Redis data) and 26379 (Sentinel) from the local network subnet (192.168.38.0/24):
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.38.0/24" port port=6379 protocol=tcp accept'
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.38.0/24" port port=26379 protocol=tcp accept'
firewall-cmd --reload
2.3. Sentinel Configuration
Sentinel must be configured on all three servers.
1. Sentinel Configuration File: The configuration defines the monitored master, named mymaster, which is initially located at 192.168.38.132 (based on the observation that the initial master role switched shortly after configuration). A quorum of 2 is required for failover.
3. Sentinel Service Creation: To run Sentinel as a system service, create a service file (e.g., /etc/systemd/system/redis-sentinel.service) using the path to redis-server (/usr/bin/redis-server):
5. Start Sentinel Service:
3. Key Takeaways
• Version Specificity: The deployment requires specific versions of Redis (6.2.16) and PostgreSQL (14), deployed on RHEL 9.x.
• Security Configuration: SSH key-based access is recommended for administration, and SELinux must be properly configured (enforcing or permissive). Access control relies on requirepass and masterauth using the password "passwd".
• High Availability Topology: The Redis setup uses a 3-node Sentinel cluster to monitor the master/slave configuration.
• Dynamic Role Switch: Although servera (192.168.38.130) was initially configured as the master, verification steps show that the Redis cluster switched its roles, with serverb (192.168.38.132) acting as the Master and servera acting as the Slave. Sentinel configuration also points to 192.168.38.132 as the monitored master.
• Failover Mechanism: The logs confirm Sentinel successfully initiates a failover (+switch-master), moving the master role from 192.168.38.130 to 192.168.38.132, and then reconfiguring 192.168.38.130 as the slave under the new master.
• Configuration Review: Key points requiring attention/clarification include the detailed interaction between the bind directive, service management, and firewall rules; explaining the role of configuration parameters; and establishing the absolute minimum required topology [Note after 26].
4. Verification (Using Tools)
Verification confirms correct installation, service roles, replication status, and failover capability.
Verification Step
Command(s)
Expected Outcome/Observation
Citation
PostgreSQL Version
/usr/local/postgresql14/bin/psql --version
psql (PostgreSQL) 14.0
Redis Server Version
redis-server --version
Redis server v=6.2.16
Role Check (Master)
redis-cli -h 192.168.38.132, auth passwd, info replication
role:master, connected_slaves:1 (slave 192.168.38.130 online)
Role Check (Slave)
redis-cli -h 192.168.38.130, auth passwd, info replication
role:slave, master_host:192.168.38.132, master_link_status:up
Synchronization Test
Master: SET key1 "Hello from Master"; Slave: GET key1
Slave returns "Hello from Master" (indicating successful replication)
Sentinel Master Check
redis-cli -h <sentinel_ip> -p 26379 SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
Returns the current master's IP and port (e.g., 192.168.38.132 6379)
Sentinel Peers Check
SENTINEL SENTINELS mymaster
Lists all monitored Sentinel instances (e.g., 192.168.38.130 and 192.168.38.133 on 26379)
Failover Demonstration
Stop Master service (systemctl stop redis on initial master)
Sentinel logs show +switch-master followed by the new epoch and the role change.
5. References
This guide is based on excerpts from "Psql_version14.0.pdf" detailing the deployment of the following major software components:
• Redis version 6.2.16
• PostgreSQL version 14.0
• RHEL 9.x Operating System
• Tools: psql, redis-cli, redis-sentinel, systemctl
