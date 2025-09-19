Docs: https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/

Install mongodb

```markdown
apt-get install gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc |    sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg    --dearmor
cat /etc/os-release 
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
ps --no-headers -o comm 1
systemctl start mongod
netstat -tulpn

```

Note: mongodb version 8 need avx cpu → Use vmware (virtual box error)

---

## 🧱 1. **Kết nối Mongo Shell**

```bash
mongosh
```

> Lệnh này khởi động trình shell để làm việc với MongoDB (nếu bạn dùng bản cũ thì là mongo).
> 

---

## 📂 2. **Danh sách database**

```jsx
show dbs
```

## 📁 3. **Tạo / chuyển database**

```jsx
use myDatabase
```

> Nếu myDatabase chưa tồn tại thì MongoDB sẽ tạo khi bạn lưu dữ liệu đầu tiên vào đó.
> 

---

## 📦 4. **Tạo collection (giống như table)**

```jsx
db.createCollection("myCollection")
# hoac
db.myCollection.insertOne({ name: "Nghia"})
```

> Trong MongoDB, bạn không cần tạo trước collection. Bạn có thể insert dữ liệu vào collection chưa tồn tại, MongoDB sẽ tự tạo.
> 

---

## 🧾 5. **Thêm dữ liệu (document) vào collection**

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

## 🔎 6. **Truy vấn dữ liệu**

```jsx
db.myCollection.find()                 // Hiển thị tất cả
db.myCollection.find({ name: "Lan" })  // Truy vấn theo điều kiện
```

Hiển thị đẹp:

```jsx
db.myCollection.find().pretty()
```

---

## ✏️ 7. **Cập nhật dữ liệu**

```jsx
db.myCollection.updateOne(
  { name: "Lan" },
  { $set: { age: 23 } }
)
```

---

## ❌ 8. **Xoá dữ liệu**

```jsx
db.myCollection.deleteOne({ name: "Tuan" })
```

---

## 🧾 9. **Danh sách collection**

```jsx
show collections
```

---

## 🗑️ 10. **Xoá collection hoặc database**

```jsx
db.myCollection.drop()  // Xoá collection
db.dropDatabase()       // Xoá database hiện tại
```

---

## 🔐 11. **Tạo user mới**

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

## 🔐 12. **Đăng nhập với user**

```bash
mongosh -u "myUser" -p "myPass" --authenticationDatabase "myDatabase"
```

## 🧾 5. **Hiển thị danh sách user trong database hiện tại**

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

## 🔍 6. **Xem tất cả user trên MongoDB**

Chuyển về database `admin`:

```jsx
js
CopyEdit
use admin
db.system.users.find().pretty()

```

---

## 🧹 7. **Xóa user**

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