| Branch        | Mục đích                         | Ai sử dụng         | Nguồn tạo (từ đâu)  | Merge vào đâu                     | MR (Pull Request)           | Hoạt động chính                 | Ghi chú                              |
| ------------- | -------------------------------- | ------------------ | ------------------- | --------------------------------- | --------------------------- | ------------------------------- | ------------------------------------ |
| **main**      | Code production (đã release)     | DevOps / Tech Lead | Từ `release`        | ❌ Không merge đi đâu (nguồn cuối) | Bắt buộc review nghiêm ngặt | Deploy production               | Luôn ổn định, không commit trực tiếp |
| **develop**   | Code tích hợp cho môi trường dev | Tất cả developer   | Từ `main` (ban đầu) | `release`                         | Bắt buộc MR                 | Tích hợp feature                | Không deploy production              |
| **feature/*** | Phát triển tính năng riêng lẻ    | Developer          | Từ `develop`        | `develop`                         | Bắt buộc MR                 | Code, test unit                 | 1 feature = 1 branch                 |
| **release/*** | Chuẩn bị release                 | Dev + QA + DevOps  | Từ `develop`        | `main` + `develop`                | Bắt buộc MR                 | Test, fix bug, chuẩn bị release | Không thêm feature mới               |
# 🔄 Flow hoạt động (dễ hiểu)

### 1. Phát triển

- Tạo `feature/xxx` từ `develop`
- Code xong → tạo MR → merge vào `develop`

---

### 2. Chuẩn bị release

- Tạo `release/x.x.x` từ `develop`
- QA test trên branch này

---

### 3. Trong lúc release

- Nếu có bug → fix trực tiếp trên `release`
- Không thêm feature mới
- ---
### 4. Release

- Merge `release` → `main` (deploy production 🚀)
- Đồng thời merge `release` → `develop` (đồng bộ code)

---
### ❌ Không được làm

- Không commit trực tiếp vào `main`
- Không commit trực tiếp vào `develop`
- Không thêm feature vào `release`