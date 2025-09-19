# Install mongodb

```bash
vi /etc/yum.repos.d/mongodb-org-8.2.repo
<
[mongodb-org-8.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/8.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://pgp.mongodb.com/server-8.2.asc
>
```
Install MongoDB Community Server.
To install the latest stable version of MongoDB, issue the following command:
```
sudo yum install -y mongodb-org
```
Alternatively, to install a specific release of MongoDB, specify each component package individually and append the version number to the package name, as in the following example:
```bash
sudo yum install -y mongodb-org-8.2.0 mongodb-org-database-8.2.0 mongodb-org-server-8.2.0 mongodb-mongosh mongodb-org-mongos-8.2.0 mongodb-org-tools-8.2.0
```
> Note: mongodb version 8 need avx cpu → Use vmware (virtual box error)
---

## Directory Paths
To Use Default Directories
By default, MongoDB runs using the mongod user account and uses the following default directories:

- `/var/lib/mongo` (the data directory)

- `/var/log/mongodb` (the log directory)

The package manager creates the default directories during installation. The owner and group name are mongod.

---

Install the SELinux Policy
1. Ensure you have the following packages installed:
- git
- make
- checkpolicy
- policycoreutils
- selinux-policy-devel
```
sudo yum install git make checkpolicy policycoreutils selinux-policy-devel
```
2. Download the policy repository.
```
git clone https://github.com/mongodb/mongodb-selinux
```
3. Build the policy.
```
cd mongodb-selinux
make
```
4. Apply the policy.
```
sudo make install
```
---

1. Start MongoDB.
```
sudo systemctl start mongod
```
If you receive an error similar to the following when starting mongod:

Failed to start mongod.service: Unit mongod.service not found.

Run the following command first:
```
sudo systemctl daemon-reload
```
Then run the start command above again.

2. Verify that MongoDB has started successfully.
```
sudo systemctl status mongod
sudo systemctl enable mongod
```
3. Stop MongoDB.
```
sudo systemctl stop mongod
```
4. Restart MongoDB.
```
sudo systemctl restart mongod
```
5. Begin using MongoDB.
```
mongosh
```

---
## Uninstall MongoDB Community Edition
1. Stop MongoDB.
Stop the mongod process by issuing the following command:
```
sudo service mongod stop
```
2. Remove Packages.
Remove any MongoDB packages that you had previously installed.
```
sudo yum erase $(rpm -qa | grep mongodb-org)
```
3. Remove Data Directories.
Remove MongoDB databases and log files.
```
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongo
```


---

## 1. **Kết nối Mongo Shell**

```bash
mongosh
```

> Lệnh này khởi động trình shell để làm việc với MongoDB (nếu bạn dùng bản cũ thì là mongo).
> 

---

## 2. **Danh sách database**

```jsx
show dbs
```

## 3. **Tạo / chuyển database**

```jsx
use myDatabase
```

> Nếu myDatabase chưa tồn tại thì MongoDB sẽ tạo khi bạn lưu dữ liệu đầu tiên vào đó.
> 

---

## 4. **Tạo collection (giống như table)**

```jsx
db.createCollection("myCollection")
# hoac
db.myCollection.insertOne({ name: "Nghia"})
```

> Trong MongoDB, bạn không cần tạo trước collection. Bạn có thể insert dữ liệu vào collection chưa tồn tại, MongoDB sẽ tự tạo.
> 

---

## 5. **Thêm dữ liệu (document) vào collection**

```jsx
db.myCollection.insertOne({ name: "Nghia", age: 25 })
```

Thêm nhiều document:

```jsx
db.myCollection.insertMany([
  { name: "Lan", age: 22 },
  { name: "Tuan", age: 30 }
])
```

---

## 6. **Truy vấn dữ liệu**

```jsx
db.myCollection.find()                 // Hiển thị tất cả
db.myCollection.find({ name: "Lan" })  // Truy vấn theo điều kiện
```

Hiển thị đẹp:

```jsx
db.myCollection.find().pretty()
```

---

## 7. **Cập nhật dữ liệu**

```jsx
db.myCollection.updateOne(
  { name: "Lan" },
  { $set: { age: 23 } }
)
```

---

## 8. **Xoá dữ liệu**

```jsx
db.myCollection.deleteOne({ name: "Tuan" })
```

---

## 9. **Danh sách collection**

```jsx
show collections
```

---

## 10. **Xoá collection hoặc database**

```jsx
db.myCollection.drop()  // Xoá collection
db.dropDatabase()       // Xoá database hiện tại
```

---

## 11. **Tạo user mới**

Kích hoạt xác thực (nếu chưa): trong file cấu hình `mongod.conf`, bật:

`vi /etc/mongod.conf`

```yaml
security:
  authorization: enabled
```

Tạo user:

```jsx
use admin
db.createUser({
  user: "adminUser",
  pwd: "adminPass",
  roles: [ { role: "root", db: "admin" } ]
})
```

Có các role phổ biến như:

- `read`: chỉ đọc
- `readWrite`: đọc & ghi
- `dbAdmin`: quản lý DB
- `userAdmin`: quản lý user
- `root`: toàn quyền

Tạo user cho database cụ thể:

```jsx
use myDatabase
db.createUser({
  user: "myUser",
  pwd: "myPass",
  roles: [ { role: "readWrite", db: "myDatabase" } ]
})
```

---

## 12. **Đăng nhập với user**

```bash
mongosh -u "myUser" -p "myPass" --authenticationDatabase "myDatabase"
```

## 13. **Hiển thị danh sách user trong database hiện tại**

```jsx
use mydb
show users
```

Hoặc truy vấn chi tiết:

```jsx
js
CopyEdit
db.getUsers()

```

---

## 14. Xem tất cả user trên MongoDB

Chuyển về database `admin`:

```jsx
js
CopyEdit
use admin
db.system.users.find().pretty()

```

---

## 15. **Xóa user**

```jsx
js
CopyEdit
db.dropUser("user1")

```

---

[Replication](https://www.notion.so/Replication-247f893014c080269deee819a88152c9?pvs=21)

---

Xoa het mongodb

```bash
sudo systemctl stop mongod
sudo apt-get purge -y mongodb-org*
sudo rm -rf /var/log/mongodb
sudo rm -rf /var/lib/mongodb
sudo rm -rf /etc/mongod.conf
sudo rm -rf /etc/mongod.conf.orig
sudo rm -rf /etc/systemd/system/mongod.service

```

---

check log

`cat /var/log/mongodb/mongod.log`

---

setParameter: enableLocalhostAuthBypass: false

---
## Docs: 
https://www.mongodb.com/docs/manual/administration/install-community/?linux-distribution=red-hat&linux-package=default&operating-system=linux&search-linux=with-search-linux