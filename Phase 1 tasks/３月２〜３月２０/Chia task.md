# 🔵 TASK 1 – AWS環境 再構築（Manual Build）

---

## 🔹 Nhóm A – Phân tích & Detailed Design

### T1-01 Phân tích Basic Design & xác định phạm vi build

**Mô tả:**

- Đọc tài liệu Network方式設計書
- Liệt kê toàn bộ AWS resource cần triển khai
- Xác định phần nào áp dụng cho môi trường dev hiện tại  
    **Output:** Danh sách resource + scope xác nhận
    

---

### T1-02 Thiết kế IP Plan (VPC & Subnet)

**Mô tả:**

- Chọn CIDR /16
    
- Thiết kế subnet /24 cho 2AZ
    
- Public / Private tách biệt  
    **Output:** Bảng phân bổ IP
    

---

### T1-03 Xác định cấu hình tối thiểu trong budget

**Mô tả:**

- Chọn EC2 instance type
    
- Chọn RDS instance type (Multi-AZ?)
    
- Chọn Redis node type
    
- Ước tính cost sơ bộ  
    **Output:** Bảng cost estimate
    

---

### T1-04 Thiết kế Security Group matrix

**Mô tả:**

- Vẽ luồng ALB → App → DB → Redis
    
- Xác định port được phép
    
- Áp dụng whitelist principle  
    **Output:** Bảng rule SG
    

---

## 🔹 Nhóm B – Network Foundation

### T1-05 Tạo VPC & Subnet (2AZ)

**Mô tả:**

- Tạo VPC
    
- Tạo Public/Private subnet cho AZ1, AZ2
    

---

### T1-06 Cấu hình IGW & Route Public

**Mô tả:**

- Attach IGW
    
- Cấu hình route table public
    

---

### T1-07 Tạo NAT Gateway & Route Private

**Mô tả:**

- Tạo NAT
    
- Cấu hình route private subnet
    

---

### T1-08 Triển khai Proxy Server

**Mô tả:**

- Tạo EC2 Proxy
    
- Kiểm tra outbound traffic qua proxy
    

---

### T1-09 Enable VPC Flow Logs

**Mô tả:**

- Bật Flow Logs
    
- Xác nhận log được ghi
    

---

## 🔹 Nhóm C – Security

### T1-10 Tạo Security Group cho ALB & App

### T1-11 Tạo Security Group cho DB & Redis

### T1-12 Tạo Security Group cho Bastion

### T1-13 Cấu hình SSH whitelist

### T1-14 Cấu hình NACL Public Subnet

### T1-15 Triển khai AWS WAF (Managed Rules)

(Mỗi ticket: cấu hình + test rule hoạt động)

---

## 🔹 Nhóm D – Compute & Load Balancing

### T1-16 Tạo ALB (Public)

### T1-17 Tạo NLB (Private)

### T1-18 Tạo Bastion Host

### T1-19 Tạo EC2 App Server

### T1-20 Cài Middleware trên App Server

### T1-21 Tạo Auto Scaling Group

### T1-22 Test scale-out hoạt động

---

## 🔹 Nhóm E – Data Layer

### T1-23 Triển khai RDS

### T1-24 Triển khai Redis Multi-AZ

### T1-25 Test kết nối App ↔ DB

### T1-26 Test kết nối App ↔ Redis

### T1-27 Test failover RDS (nếu Multi-AZ)

---

## 🔹 Nhóm F – DNS

### T1-28 Tạo Route53 Public Zone

### T1-29 Tạo Route53 Private Zone

### T1-30 Cấu hình record ALB

### T1-31 Kiểm tra resolve nội bộ

---

## 🔹 Nhóm G – Validation

### T1-32 Deploy ứng dụng test

### T1-33 Kiểm tra full traffic flow

### T1-34 Security check cơ bản

### T1-35 Review chi phí thực tế

---

# 🔵 TASK 2 – Cost Alert

---

### T2-01 Tạo AWS Budget

### T2-02 Tạo SNS Topic

### T2-03 Tạo Lambda gửi Teams

### T2-04 Cấu hình alert 50%

### T2-05 Cấu hình alert 75%

### T2-06 Cấu hình alert 100%

### T2-07 Test end-to-end

---

# 🔵 TASK 3 – Auto Stop

---

### T3-01 Quyết định phương án (Lambda/SSM)

### T3-02 Tạo IAM Role automation

### T3-03 Tạo Lambda stop EC2

### T3-04 Tạo Lambda stop RDS

### T3-05 Tạo EventBridge schedule

### T3-06 Test stop/start

### T3-07 Kiểm tra ảnh hưởng Auto Scaling

---

# 🔵 TASK 4 – CloudFormation

---

### T4-01 Thiết kế cấu trúc template

### T4-02 Template Network

### T4-03 Template Security

### T4-04 Template Compute

### T4-05 Template RDS

### T4-06 Template Redis

### T4-07 Template ALB/NLB

### T4-08 Deploy test stack

### T4-09 Validate recreate thành công

---

# 🔵 TASK 5 – CI/CD

---

### T5-01 Quyết định công cụ

### T5-02 Thiết kế pipeline

### T5-03 Cấu hình build stage

### T5-04 Cấu hình deploy EC2

### T5-05 Test auto deploy

### T5-06 Thiết lập rollback cơ bản