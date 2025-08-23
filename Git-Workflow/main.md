# Những câu lệnh Git thông dụng
Cấu hình tên user,email (toàn cục). Thông tin sẽ hiển thị trong commit history.
```bash
git config --global user.name "Ho Van Nghia"
git config --global user.email "nghia.ho@sotatek.com"
```

Pull toàn bộ repository từ server về máy local.
```bash
git clone http://git.nghiahv.tech/shoeshop/shoeshop.git
---
Username:
Password:
```

Thêm tất cả file mới hoặc thay đổi vào staging area (khu vực chuẩn bị commit).
```bash
git add .
```
Note: Có thể thay `.` bằng tên file cụ thể, ví dụ `git add index.js`.

Tạo một commit (một điểm lưu thay đổi trong lịch sử Git).
```
git commit -m "feat(project): create base project"
```

Đẩy (push) commit từ local lên remote repo (origin), nhánh (main).
```bash
git push origin main
```

Lấy (fetch) và đồng bộ (merge) code mới nhất từ remote repository về local.
```
git pull 
```
Note: 
`git pull` = `git fetch` + `git merge`
- `git fetch`: tải thay đổi từ remote về, nhưng chưa áp dụng vào code local.
- `git merge`: gộp những thay đổi vừa fetch vào nhánh hiện tại của bạn.


Git command:  
Git command | Cong dung
--- | ---
git init | Khởi tạo repo Git mới trong thư mục hiện tại
git status | Xem trạng thái file (modified, staged, untracked)
git log | Xem lịch sử commit
git branch	| Liệt kê các nhánh local
git checkout `branch`	|Chuyển sang nhánh khác
git merge `branch` |Gộp nhánh `branch` vào nhánh hiện tại

---
# Git workflow   
Git workflow là một quy trình làm việc hoặc phương pháp tổ chức việc sử dụng Git trong quản lý mã nguồn  
![](pic/2.png)

Một workflow thường quy định:
- Các nhánh: main, develop, feature, hotfix, release,…
- Quy trình tách và hợp nhất nhánh: Ví dụ như nhánh feature phải được tách ra từ develop, hay mã nguồn từ develop mới được phép hợp nhất vào main
- Quy tắc cộng tác: Đưa ra quy định về việc tạo Pull request, review code trước khi hợp nhất,…

Y nghia cac nhanh tren git workflow
- nhánh "main" chứa code môi trường người dùng.
- nhánh "develop" chứa code môi trường phát triển.
- nhánh "feature" được tạo từ nhánh develop chứa code các chức năng.
- nhánh "release" chứa code môi trường thử nghiệm.
- nhánh "hotfix" được tạ ra từ nhánh main.

# Các Thành Phần Chính Của Git Workflow

**1. Quy Tắc Commit**  
Nguyên tắc cơ bản:
- Viết commit message ngắn gọn, rõ ràng
- Tuân theo format thống nhất trong team
- Mỗi commit chỉ giải quyết một vấn đề cụ thể
```bash
# Format chuẩn
<type>[optional scope]: <description>

# Ví dụ thực tế
feat: thêm tính năng đăng nhập bằng Google
fix: khắc phục lỗi hiển thị trên Safari
docs: cập nhật README.md
refactor: tái cấu trúc module thanh toán
test: thêm unit test cho chức năng giỏ hàng
```

Các Type Commit Phổ Biến:
Type	|Mục Đích Sử Dụng
---|---
feat	|Thêm tính năng mới
fix	|Sửa lỗi
docs	|Cập nhật tài liệu
style	|Thay đổi format, không ảnh hưởng code
refactor	|Tái cấu trúc code
test	|Thêm hoặc sửa test
chore	|Công việc bảo trì

**2. Pull Request Workflow - Quy Trình Review Code**  
Pull Request (PR) là cơ chế chính để review và merge code trong team. Một quy trình PR chuẩn bao gồm:

Các Bước Thực Hiện:

2.1 Tạo Branch Mới:
- Branch cho feature: feature/ten-tinh-nang
- Branch cho bugfix: bugfix/mo-ta-loi
- Branch cho hotfix: hotfix/van-de-khan

2.2 Phát Triển Code:
- Thực hiện các thay đổi nhỏ, dễ review
- Commit thường xuyên theo quy tắc
- Test kỹ trước khi tạo PR

2.3 Tạo Pull Request:
- Viết mô tả chi tiết về các thay đổi
- Liên kết với issue/task liên quan
- Thêm reviewer phù hợp

2.4 Review Process:
- Code review từ ít nhất một thành viên
- Phản hồi và thảo luận trên PR
- Chỉnh sửa theo góp ý

2.5 Kiểm Tra Tự Động:
- CI/CD pipeline tự động chạy
- Kiểm tra coding standard
- Chạy automated tests

2.6 Merge Code:
- Merge khi đã được approve
- Xử lý conflict nếu có
- Delete branch sau khi merge

**3. Tích Hợp Liên Tục (CI/CD) - Tự Động Hóa Quy Trình**  
CI/CD là phần không thể thiếu trong modern Git workflow, giúp đảm bảo chất lượng code và tự động hóa quy trình.

***Các Thành Phần Chính***:  
*Continuous Integration (CI):*
- Kiểm tra mã tự động (linting)
- Code formatting check
- Unit tests
- Integration tests
- Code coverage check  

*Continuous Deployment (CD):*  
- Build tự động
- Deploy đến môi trường test
- Deploy đến staging
- Deploy đến production

**Best Practices**  
*Branch Strategy:*
- Main/Master: code production
- Develop: code đang phát triển
- Feature branches: tính năng mới
- Release branches: chuẩn bị release

*Code Review:*
- Review code thường xuyên
- Tập trung vào logic và structure
- Góp ý mang tính xây dựng
- Sử dụng code review checklist

*Documentation:*
- README.md cập nhật
- Comment code khi cần thiết
- Tài liệu API và architecture
- Changelog cho mỗi version

---
**Tài Liệu Tham Khảo**  
Docs: https://www.ducxinh.com/techblog/git-workflow-cho-team:-huong-dan-toan-dien-ve-quy-trinh-lam-viec
Docs git commit: https://www.conventionalcommits.org/en/v1.0.0/  
GitHub Flow: https://guides.github.com/introduction/flow  
GitLab CI/CD: https://docs.gitlab.com/ee/ci  
Pull Request Best Practices: https://github.blog/2015-01-21-how-to-write-the-perfect-pull-request