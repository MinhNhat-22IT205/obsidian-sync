---

# 🟦 TASK 1 — Tái xây dựng AWS môi trường (manual)

## Nhóm 1. Phân tích & thiết kế chi tiết

---

### **T1-01 — Phân tích Basic Design và xác định phạm vi triển khai**

**Mô tả công việc**

- Đọc tài liệu Network方式設計書 (bản cơ bản)
    
- Trích xuất các thành phần hạ tầng cần có trong môi trường dev
    
- Phân loại thành phần: bắt buộc / tùy chọn / có thể giản lược theo budget
    
- Ghi rõ các điểm chưa có trong tài liệu (cần quyết định ở bước thiết kế chi tiết)
    

**Đầu ra**

- Danh sách resource cần triển khai (scope list)
    
- Danh sách giả định (assumptions)
    
- Danh sách điểm chưa rõ (open points)
    

**Done khi**

- Có tài liệu/bảng scope được cả nhóm dùng để chia việc tiếp theo
    
- Mỗi resource có trạng thái “Build / Skip / Pending decision”
    

---

### **T1-02 — Thiết kế IP Plan (VPC / Subnet)**

**Mô tả công việc**

- Chốt CIDR cho VPC (theo tài liệu: /16)
    
- Thiết kế subnet /24 cho 2AZ
    
- Phân bổ public/private subnet theo AZ
    
- Reserve khoảng IP dự phòng cho mở rộng sau
    

**Đầu ra**

- Bảng IP plan (VPC CIDR, subnet CIDR, purpose, AZ)
    
- Sơ đồ logic subnet đơn giản
    

**Done khi**

- Không trùng CIDR
    
- Đủ subnet để triển khai các thành phần chính (ALB/NAT/Bastion/App/DB/Redis)
    

---

### **T1-03 — Xác định cấu hình tối thiểu trong budget**

**Mô tả công việc**

- Đề xuất cấu hình tối thiểu cho EC2, RDS, Redis, NAT, ALB/NLB
    
- So sánh phương án “đúng theo basic design” và “dev tối ưu chi phí”
    
- Ước lượng chi phí tháng (mục tiêu ~10万円)
    

**Đầu ra**

- Bảng cấu hình đề xuất
    
- Bảng estimate cost sơ bộ theo từng dịch vụ
    
- Đề xuất thành phần nào bật/tắt để không vượt budget
    

**Done khi**

- Có phương án khả thi trong phạm vi ngân sách
    
- Ghi rõ rủi ro nếu giản lược (ví dụ bỏ Multi-AZ cho dev)
    

---

### **T1-04 — Thiết kế Security Group matrix**

**Mô tả công việc**

- Liệt kê luồng giao tiếp cần thiết: Internet→ALB, ALB→App, App→DB, App→Redis, Bastion→App/DB, App→Proxy/NAT...
    
- Xác định port/protocol tối thiểu
    
- Áp dụng whitelist principle
    

**Đầu ra**

- Bảng SG matrix (source, destination, port, protocol, purpose)
    
- Danh sách SG cần tạo
    

**Done khi**

- Mỗi luồng được mô tả rõ
    
- Không mở port dư thừa
    

---

## Nhóm 2. Network Foundation

---

### **T1-05 — Tạo VPC và subnet cho 2AZ**

**Mô tả công việc**

- Tạo VPC theo IP plan
    
- Tạo public/private subnet cho AZ1 và AZ2
    
- Gắn tag theo quy ước (env, role, az)
    

**Đầu ra**

- VPC + 4 subnet (hoặc nhiều hơn nếu tách app/db/cache)
    
- Screenshot/record cấu hình
    

**Done khi**

- Subnet tạo đủ và đúng CIDR/AZ
    
- Có thể dùng cho bước routing tiếp theo
    

---

### **T1-06 — Cấu hình Internet Gateway và route table public**

**Mô tả công việc**

- Tạo và attach IGW vào VPC
    
- Tạo route table public
    
- Thêm default route ra IGW
    
- Associate route table với public subnet
    

**Đầu ra**

- IGW hoạt động
    
- Public route table cấu hình xong
    

**Done khi**

- Public subnet có route 0.0.0.0/0 → IGW
    
- Bastion/ALB có thể dùng public path (ở bước sau)
    

---

### **T1-07 — Tạo NAT Gateway và route table private**

**Mô tả công việc**

- Tạo NAT Gateway (ưu tiên cấu hình tối thiểu cho dev)
    
- Tạo/điều chỉnh route table private
    
- Route outbound từ private subnet đi qua NAT
    

**Đầu ra**

- NAT Gateway
    
- Private route table
    

**Done khi**

- Private subnet có outbound path hợp lệ
    
- Không có inbound public trực tiếp vào private subnet
    

---

### **T1-08 — Triển khai Proxy Server cho outbound control**

**Mô tả công việc**

- Tạo EC2 làm proxy (spec tối thiểu)
    
- Cài proxy software (ví dụ Squid hoặc tương đương)
    
- Cấu hình outbound test từ private/app qua proxy
    
- Chuẩn bị rule blacklist URL cơ bản (nếu áp dụng ngay)
    

**Đầu ra**

- Proxy server chạy được
    
- Tài liệu cấu hình proxy cơ bản
    

**Done khi**

- Có thể xác nhận request outbound đi qua proxy
    
- App/private instance có thể dùng proxy để ra ngoài (test đơn giản)
    

---

### **T1-09 — Bật VPC Flow Logs**

**Mô tả công việc**

- Enable VPC Flow Logs cho VPC (hoặc subnet)
    
- Chọn destination (CloudWatch Logs/S3 tùy phương án)
    
- Kiểm tra log phát sinh
    

**Đầu ra**

- Flow logs được bật
    
- Log destination cấu hình xong
    

**Done khi**

- Có bản ghi flow log xuất hiện
    
- Có thể truy vết lưu lượng cơ bản
    

---

## Nhóm 3. Security Layer

---

### **T1-10 — Tạo Security Group cho ALB và App**

**Mô tả công việc**

- Tạo SG cho ALB (chỉ HTTP/HTTPS từ ngoài theo yêu cầu)
    
- Tạo SG cho App
    
- Cấu hình inbound từ ALB → App theo port ứng dụng
    
- Cấu hình outbound tối thiểu cần thiết cho App
    

**Đầu ra**

- 2 SG (ALB, App) + rule đúng theo matrix
    

**Done khi**

- Rule đúng chiều và đúng port
    
- Không mở SSH từ Internet vào App
    

---

### **T1-11 — Tạo Security Group cho DB và Redis**

**Mô tả công việc**

- Tạo SG cho RDS (chỉ cho phép từ App/Bastion nếu cần)
    
- Tạo SG cho Redis (chỉ TCP/6379 từ App)
    
- Chặn truy cập từ public subnet/internet
    

**Đầu ra**

- 2 SG (DB, Redis) + rule tối thiểu
    

**Done khi**

- App có thể kết nối DB/Redis (ở bước test sau)
    
- Không có rule mở công khai
    

---

### **T1-12 — Tạo Security Group cho Bastion**

**Mô tả công việc**

- Tạo SG cho Bastion
    
- Chỉ cho phép SSH (22) từ IP whitelist admin
    
- Outbound cho Bastion đủ để quản trị (SSH tới private, update nếu cần)
    

**Đầu ra**

- SG Bastion
    

**Done khi**

- Chỉ IP whitelist mới vào được port 22
    
- Không có 0.0.0.0/0 vào SSH
    

---

### **T1-13 — Cấu hình SSH whitelist cho Bastion**

**Mô tả công việc**

- Áp dụng IP cố định quản trị viên vào SG Bastion
    
- Test từ IP hợp lệ / không hợp lệ
    
- Ghi nhận cách cập nhật whitelist khi đổi IP
    

**Đầu ra**

- Cấu hình SSH whitelist đã xác nhận
    
- Biên bản test đơn giản
    

**Done khi**

- Truy cập SSH chỉ thành công từ IP được phép
    

---

### **T1-14 — Cấu hình Network ACL cho Public Subnet**

**Mô tả công việc**

- Thiết kế NACL mức tối thiểu cho public subnet (theo tài liệu: nghiêm hơn private)
    
- Tạo inbound/outbound rule rõ ràng (vì NACL stateless)
    
- Gắn NACL vào public subnet
    

**Đầu ra**

- NACL public subnet
    
- Bảng rule NACL
    

**Done khi**

- Public subnet hoạt động bình thường cho traffic cần thiết
    
- Không chặn nhầm lưu lượng hợp lệ (ALB/Bastion)
    

---

### **T1-15 — Triển khai AWS WAF (Managed Rules)**

**Mô tả công việc**

- Tạo Web ACL
    
- Áp dụng AWS Managed Rules cơ bản
    
- Associate WAF với ALB (hoặc entry point phù hợp)
    
- Kiểm tra association thành công
    

**Đầu ra**

- WAF Web ACL + ruleset cơ bản
    
- Tài liệu rule đang bật
    

**Done khi**

- WAF attach thành công vào ALB
    
- Có log/metric hoặc xác nhận hoạt động cơ bản
    

---

## Nhóm 4. Compute & Load Balancing

---

### **T1-16 — Tạo ALB (Public)**

**Mô tả công việc**

- Tạo Application Load Balancer ở public subnet
    
- Cấu hình listener (HTTP/HTTPS theo phạm vi hiện tại)
    
- Tạo target group cho App
    

**Đầu ra**

- ALB + listener + target group
    

**Done khi**

- ALB active
    
- Target group sẵn sàng nhận EC2 App
    

---

### **T1-17 — Tạo NLB (Private)**

**Mô tả công việc**

- Tạo Network Load Balancer trong private subnet (theo thiết kế)
    
- Cấu hình listener/target group nội bộ cho app server (nếu cần cho internal flow)
    
- Xác nhận internal-only
    

**Đầu ra**

- Internal NLB
    

**Done khi**

- NLB được tạo trong private context
    
- Có cấu hình target group phù hợp
    

---

### **T1-18 — Tạo Bastion Host**

**Mô tả công việc**

- Launch EC2 Bastion tại public subnet
    
- Gắn SG Bastion
    
- Cấu hình key pair / session access
    
- Gắn tag chuẩn
    

**Đầu ra**

- Bastion Host hoạt động
    

**Done khi**

- SSH vào Bastion từ IP whitelist thành công
    
- Bastion có thể reach private subnet theo policy
    

---

### **T1-19 — Tạo EC2 App Server**

**Mô tả công việc**

- Launch EC2 App Server tại private subnet
    
- Gắn SG App
    
- Không gán public IP
    
- Xác nhận truy cập quản trị qua Bastion
    

**Đầu ra**

- App Server EC2 chạy ở private subnet
    

**Done khi**

- App Server reachable từ Bastion
    
- Không truy cập trực tiếp từ Internet
    

---

### **T1-20 — Cài middleware trên App Server**

**Mô tả công việc**

- Cài runtime/web server/middleware cần thiết cho app (theo dự án)
    
- Cấu hình service khởi động
    
- Kiểm tra version và service status
    

**Đầu ra**

- Middleware đã cài xong
    
- Tài liệu cài đặt ngắn (commands/config chính)
    

**Done khi**

- Service chạy ổn định
    
- Sẵn sàng deploy ứng dụng test
    

---

### **T1-21 — Tạo Auto Scaling Group cơ bản**

**Mô tả công việc**

- Tạo Launch Template (hoặc tương đương)
    
- Tạo ASG cho App
    
- Gắn target group (ALB hoặc internal flow theo thiết kế)
    
- Thiết lập min/max/desired cơ bản (dev-friendly)
    

**Đầu ra**

- ASG hoạt động với App instance
    

**Done khi**

- ASG tạo instance thành công
    
- Instance join target group
    

---

### **T1-22 — Test scale-out hoạt động**

**Mô tả công việc**

- Tạo điều kiện scale-out (manual/metric test)
    
- Xác nhận instance mới được tạo và health check pass
    
- Xác nhận traffic routing vẫn ổn
    

**Đầu ra**

- Kết quả kiểm tra scale-out
    
- Ghi chú issue nếu có
    

**Done khi**

- Scale-out thành công ít nhất 1 lần
    
- Không ảnh hưởng traffic cơ bản
    

---

## Nhóm 5. Data Layer

---

### **T1-23 — Triển khai RDS**

**Mô tả công việc**

- Tạo RDS instance theo cấu hình đã chọn (ưu tiên tối ưu chi phí dev)
    
- Cấu hình subnet group / SG / parameter cơ bản
    
- Bật Multi-AZ nếu phương án budget cho phép (hoặc ghi rõ ngoại lệ dev)
    

**Đầu ra**

- RDS instance chạy được
    
- Thông tin endpoint / cấu hình kết nối
    

**Done khi**

- RDS available
    
- Chỉ private access theo SG
    

---

### **T1-24 — Triển khai Redis (ElastiCache)**

**Mô tả công việc**

- Tạo ElastiCache Redis trong private subnet
    
- Cấu hình SG Redis
    
- Thiết lập replication / Multi-AZ nếu áp dụng trong phạm vi dev
    

**Đầu ra**

- Redis endpoint hoạt động
    
- Cấu hình cluster/replication
    

**Done khi**

- Redis available
    
- App SG có thể truy cập TCP/6379
    

---

### **T1-25 — Test kết nối App ↔ DB**

**Mô tả công việc**

- Từ App Server test kết nối đến RDS
    
- Xác nhận network path + credential/config cơ bản
    
- Ghi lại kết quả và lỗi (nếu có)
    

**Đầu ra**

- Kết quả test DB connectivity
    

**Done khi**

- Kết nối DB thành công từ App
    
- Không cần mở thêm rule ngoài thiết kế (hoặc có ghi nhận nếu cần điều chỉnh)
    

---

### **T1-26 — Test kết nối App ↔ Redis**

**Mô tả công việc**

- Từ App Server test TCP/6379 và thao tác Redis cơ bản (PING/SET/GET)
    
- Xác nhận SG/routing đúng
    

**Đầu ra**

- Kết quả test Redis connectivity
    

**Done khi**

- App truy cập Redis thành công
    
- Không có truy cập từ public path
    

---

### **T1-27 — Test failover RDS (hoặc xác nhận giới hạn dev)**

**Mô tả công việc**

- Nếu dùng Multi-AZ: thực hiện/chuẩn bị test failover ở mức phù hợp
    
- Nếu dev không bật Multi-AZ vì cost: ghi rõ “skip with reason” + tác động
    
- Ghi lại thời gian khôi phục / ảnh hưởng (nếu test được)
    

**Đầu ra**

- Biên bản failover test hoặc biên bản miễn trừ (waiver) cho dev
    

**Done khi**

- Có bằng chứng xác nhận khả năng/giới hạn failover của môi trường hiện tại
    

---

## Nhóm 6. DNS & Domain

---

### **T1-28 — Tạo Route53 Public Hosted Zone**

**Mô tả công việc**

- Tạo public hosted zone (nếu domain đã có)
    
- Chuẩn bị record cho ALB
    
- Ghi nhận trạng thái domain thật/chưa có domain thật
    

**Đầu ra**

- Public hosted zone hoặc tài liệu giả lập cấu hình
    

**Done khi**

- Zone được tạo thành công (hoặc có phương án thay thế cho dev)
    

---

### **T1-29 — Tạo Route53 Private Hosted Zone**

**Mô tả công việc**

- Tạo private hosted zone cho domain nội bộ (ví dụ *.internal)
    
- Associate vào VPC
    
- Chuẩn bị record cho internal service
    

**Đầu ra**

- Private hosted zone
    

**Done khi**

- Zone được associate đúng VPC
    
- Có thể thêm record nội bộ
    

---

### **T1-30 — Cấu hình DNS record cho ALB**

**Mô tả công việc**

- Tạo record trỏ đến ALB (A/AAAA alias nếu phù hợp)
    
- Xác nhận resolve từ phía client bên ngoài (hoặc test nội bộ nếu chưa có public domain)
    

**Đầu ra**

- Record DNS cho entry point
    

**Done khi**

- Resolve đúng đến ALB
    
- Truy cập endpoint hoạt động ở mức cơ bản
    

---

### **T1-31 — Kiểm tra resolve nội bộ (Private DNS)**

**Mô tả công việc**

- Tạo record nội bộ cho DB/Redis/app nội bộ (nếu áp dụng)
    
- Test resolve từ EC2 trong VPC
    
- Xác nhận không resolve được từ ngoài VPC
    

**Đầu ra**

- Kết quả test private DNS
    

**Done khi**

- Resolve nội bộ thành công
    
- Hành vi đúng với private zone policy
    

---

## Nhóm 7. Validation & Hoàn tất môi trường

---

### **T1-32 — Deploy ứng dụng test**

**Mô tả công việc**

- Triển khai ứng dụng mẫu hoặc bản build đơn giản
    
- Cấu hình môi trường (DB endpoint, Redis endpoint, biến môi trường)
    
- Khởi động service
    

**Đầu ra**

- Ứng dụng test chạy trên App Server
    

**Done khi**

- Có endpoint truy cập được qua kiến trúc đã dựng
    

---

### **T1-33 — Kiểm tra full traffic flow**

**Mô tả công việc**

- Xác minh các luồng chính:
    
    - User → ALB → App
        
    - App → DB
        
    - App → Redis
        
    - Private outbound → Proxy/NAT
        
- Kiểm tra health check/load balancer routing
    

**Đầu ra**

- Checklist traffic flow + kết quả pass/fail
    

**Done khi**

- Các luồng chính pass
    
- Lỗi còn lại (nếu có) được ghi thành issue riêng
    

---

### **T1-34 — Security check cơ bản**

**Mô tả công việc**

- Kiểm tra nhanh các điểm security theo basic design:
    
    - App/DB không có public IP
        
    - SSH chỉ qua Bastion
        
    - Redis không public
        
    - SG/NACL theo nguyên tắc tối thiểu
        
- Ghi nhận sai lệch
    

**Đầu ra**

- Checklist security baseline
    
- Danh sách gap
    

**Done khi**

- Có kết quả xác nhận mức độ tuân thủ basic design cho môi trường dev
    

---

### **T1-35 — Review chi phí thực tế sau dựng môi trường**

**Mô tả công việc**

- Kiểm tra cost phát sinh thực tế sau khi dựng
    
- So sánh với estimate ở T1-03
    
- Đề xuất cắt giảm nếu có nguy cơ vượt budget
    

**Đầu ra**

- Báo cáo cost review ngắn
    
- Danh sách action tối ưu (nếu cần)
    

**Done khi**

- Có đánh giá “Trong/Nguy cơ vượt budget”
    
- Có đề xuất xử lý rõ ràng
    

---

# 🟦 TASK 2 — Cost Alert (50% / 75% / 100% gửi Teams)

---

### **T2-01 — Tạo AWS Budget cho môi trường**

**Mô tả công việc**

- Tạo budget theo ngưỡng tháng (mục tiêu ~10万円)
    
- Chọn loại budget (cost budget)
    
- Thiết lập thời gian theo dõi
    

**Đầu ra**

- AWS Budget được tạo
    

**Done khi**

- Budget active và hiển thị đúng ngưỡng theo tháng
    

---

### **T2-02 — Thiết kế luồng cảnh báo SNS → Lambda → Teams**

**Mô tả công việc**

- Xác định cách Budget trigger SNS
    
- SNS gọi Lambda
    
- Lambda gửi message đến Teams webhook
    
- Xác định format message (mức %, budget hiện tại, thời gian)
    

**Đầu ra**

- Sơ đồ luồng + mô tả payload/message format
    

**Done khi**

- Cả nhóm thống nhất flow triển khai
    
- Có mẫu message chuẩn
    

---

### **T2-03 — Tạo SNS Topic cho budget alert**

**Mô tả công việc**

- Tạo SNS topic nhận cảnh báo budget
    
- Thiết lập subscription phù hợp (Lambda)
    
- Gắn naming/tag theo môi trường
    

**Đầu ra**

- SNS Topic + subscription
    

**Done khi**

- SNS topic tạo xong và ready để nhận event
    

---

### **T2-04 — Tạo Lambda gửi thông báo sang Teams**

**Mô tả công việc**

- Viết Lambda nhận event từ SNS/Budget
    
- Parse nội dung alert
    
- Gửi message đến Teams qua incoming webhook
    
- Xử lý lỗi cơ bản (HTTP error / invalid payload)
    

**Đầu ra**

- Lambda function chạy được
    
- Source code + cấu hình biến môi trường (webhook URL, v.v.)
    

**Done khi**

- Test invoke Lambda gửi Teams thành công
    

---

### **T2-05 — Cấu hình budget alert mức 50%**

**Mô tả công việc**

- Tạo threshold 50%
    
- Liên kết notification tới SNS topic
    
- Kiểm tra cấu hình đúng channel
    

**Đầu ra**

- Rule alert 50%
    

**Done khi**

- Rule hiển thị active trong Budget settings
    

---

### **T2-06 — Cấu hình budget alert mức 75%**

**Mô tả công việc**

- Tạo threshold 75%
    
- Liên kết SNS topic như thiết kế
    

**Đầu ra**

- Rule alert 75%
    

**Done khi**

- Rule active và đúng ngưỡng
    

---

### **T2-07 — Cấu hình budget alert mức 100%**

**Mô tả công việc**

- Tạo threshold 100%
    
- Liên kết SNS topic như thiết kế
    

**Đầu ra**

- Rule alert 100%
    

**Done khi**

- Rule active và đúng ngưỡng
    

---

### **T2-08 — Test end-to-end cảnh báo cost lên Teams**

**Mô tả công việc**

- Thực hiện test bằng event giả lập (không chờ cost thật)
    
- Xác nhận SNS → Lambda → Teams hoạt động
    
- Kiểm tra nội dung message đủ thông tin
    

**Đầu ra**

- Kết quả test E2E + screenshot Teams message
    

**Done khi**

- Teams nhận đúng message mẫu từ hệ thống
    

---

# 🟦 TASK 3 — Tự động dừng RDS / EC2 theo giờ (Lambda / SSM / EventBridge)

---

### **T3-01 — Phân tích điều kiện dừng EC2/RDS và ràng buộc môi trường**

**Mô tả công việc**

- Kiểm tra loại RDS đang dùng có thể stop/start theo yêu cầu không
    
- Xác định tác động nếu App dùng ASG (ASG có thể tự tạo lại EC2)
    
- Chốt phạm vi tài nguyên sẽ tự động stop (tag-based hay fixed list)
    

**Đầu ra**

- Tài liệu quyết định phạm vi stop/start + lưu ý kỹ thuật
    

**Done khi**

- Có kết luận rõ tài nguyên nào được auto stop
    
- Có note các ngoại lệ/rủi ro
    

---

### **T3-02 — Quyết định phương án triển khai (Lambda / SSM / EventBridge)**

**Mô tả công việc**

- So sánh phương án triển khai theo độ đơn giản và vận hành
    
- Chốt phương án chính (thường EventBridge + Lambda)
    
- Xác định lịch chạy “n giờ mỗi ngày”
    

**Đầu ra**

- Quyết định giải pháp + mô tả kiến trúc
    

**Done khi**

- Cả nhóm thống nhất một phương án triển khai
    

---

### **T3-03 — Tạo IAM Role cho automation**

**Mô tả công việc**

- Tạo role/policy cho Lambda hoặc SSM để stop/start EC2/RDS
    
- Giới hạn quyền theo resource/tag nếu có thể
    
- Kiểm tra principle of least privilege ở mức hợp lý
    

**Đầu ra**

- IAM Role + policy cho automation
    

**Done khi**

- Function/service có đủ quyền chạy, không dùng quyền quá rộng nếu tránh được
    

---

### **T3-04 — Tạo automation stop EC2**

**Mô tả công việc**

- Tạo Lambda/SSM logic dừng EC2 theo tag/instance list
    
- Ghi log kết quả xử lý từng instance
    
- Bỏ qua instance không thuộc phạm vi
    

**Đầu ra**

- Automation stop EC2
    

**Done khi**

- Test chạy tay (manual invoke) dừng EC2 thành công
    

---

### **T3-05 — Tạo automation stop RDS**

**Mô tả công việc**

- Tạo Lambda/SSM logic dừng RDS theo tag/DB list
    
- Xử lý trường hợp không stop được (unsupported state)
    
- Ghi log kết quả
    

**Đầu ra**

- Automation stop RDS
    

**Done khi**

- Manual test dừng RDS thành công (hoặc log đúng lý do fail)
    

---

### **T3-06 — Tạo EventBridge schedule chạy hàng ngày**

**Mô tả công việc**

- Tạo schedule rule theo giờ yêu cầu (n giờ)
    
- Gắn target automation (EC2/RDS)
    
- Xác định timezone/cron expression đúng
    

**Đầu ra**

- EventBridge rule(s) active
    

**Done khi**

- Schedule được tạo đúng giờ mong muốn
    
- Có thể trigger thử
    

---

### **T3-07 — Test stop/start và kiểm tra vận hành**

**Mô tả công việc**

- Test end-to-end lịch stop
    
- Test start lại thủ công (hoặc automation nếu có)
    
- Kiểm tra impact đến ASG/app service
    
- Ghi hướng dẫn vận hành khi cần bảo trì ngoài giờ
    

**Đầu ra**

- Biên bản test + hướng dẫn vận hành cơ bản
    

**Done khi**

- Xác nhận cơ chế auto stop dùng được trong môi trường dev
    

---

# 🟦 TASK 4 — Tạo template CloudFormation để dựng cùng môi trường

---

### **T4-01 — Phân tích phạm vi IaC và thứ tự phụ thuộc resource**

**Mô tả công việc**

- Liệt kê resource của môi trường manual cần chuyển sang CloudFormation
    
- Xác định dependency (VPC → subnet → SG → EC2/RDS/ALB...)
    
- Chốt những phần đưa vào phase này và phần deferred (nếu có)
    

**Đầu ra**

- Danh sách resource IaC hóa + dependency map
    

**Done khi**

- Có scope rõ ràng cho template implementation
    

---

### **T4-02 — Thiết kế cấu trúc template (single / nested)**

**Mô tả công việc**

- Chọn cấu trúc template (single stack hay nested)
    
- Thiết kế parameter/output chuẩn (CIDR, env, instance type, subnet ids...)
    
- Thiết kế naming convention / tagging trong template
    

**Đầu ra**

- Thiết kế cấu trúc template + danh sách parameter/output
    

**Done khi**

- Có blueprint đủ để tách ticket code template tiếp theo
    

---

### **T4-03 — Tạo template Network (VPC/Subnet/Route cơ bản)**

**Mô tả công việc**

- Viết CloudFormation cho VPC, subnet, route table (phạm vi network base)
    
- Thêm tags/parameters
    
- Validate syntax template
    

**Đầu ra**

- File template network
    

**Done khi**

- Template validate pass
    
- Có thể deploy phần network cơ bản
    

---

### **T4-04 — Tạo template Internet/NAT (IGW, NAT, route liên quan)**

**Mô tả công việc**

- Viết template cho IGW, attachment, route public
    
- Viết template NAT gateway + route private
    
- Xử lý dependency và parameter cho subnet public/private
    

**Đầu ra**

- File template internet/nat (hoặc module tương ứng)
    

**Done khi**

- Deploy test phần routing internet/private thành công
    

---

### **T4-05 — Tạo template Security (SG/NACL/IAM cơ bản)**

**Mô tả công việc**

- Viết template tạo SG cho ALB/App/DB/Redis/Bastion
    
- Viết NACL public subnet tối thiểu
    
- Tạo IAM role cơ bản cần cho hạ tầng (nếu nằm trong scope)
    

**Đầu ra**

- File template security
    

**Done khi**

- Rule tạo đúng như matrix đã thiết kế
    
- Template validate/deploy pass
    

---

### **T4-06 — Tạo template Compute (Bastion/App/ASG)**

**Mô tả công việc**

- Viết template cho Bastion EC2
    
- Viết template cho App (Launch Template/ASG nếu áp dụng)
    
- Gắn SG/subnet/tags/user-data cơ bản
    

**Đầu ra**

- File template compute
    

**Done khi**

- Deploy tạo được Bastion/App/ASG theo cấu hình dev
    

---

### **T4-07 — Tạo template Data (RDS/Redis)**

**Mô tả công việc**

- Viết template cho DB subnet group + RDS
    
- Viết template cho Redis subnet group + ElastiCache
    
- Parameter hóa instance type/node type theo env
    

**Đầu ra**

- File template data
    

**Done khi**

- Deploy test tạo được data resources ở mức cơ bản
    

---

### **T4-08 — Tạo template Load Balancer (ALB/NLB)**

**Mô tả công việc**

- Viết template cho ALB, listener, target group
    
- Viết template cho NLB nội bộ (nếu trong scope)
    
- Kết nối với compute resources qua parameter/output
    

**Đầu ra**

- File template lb
    

**Done khi**

- Deploy test tạo LB thành công, target group hoạt động
    

---

### **T4-09 — Tạo template DNS/WAF (phần trong scope dev)**

**Mô tả công việc**

- Viết template Route53 zone/record (nếu domain và quyền cho phép)
    
- Viết template WAF managed rules + association (nếu đưa vào phase này)
    
- Nếu chưa thể fully automate, ghi rõ phần manual còn lại
    

**Đầu ra**

- File template dns/waf hoặc tài liệu phạm vi ngoại lệ
    

**Done khi**

- Các phần trong scope được IaC hóa hoặc có ghi chú loại trừ rõ ràng
    

---

### **T4-10 — Deploy test stack và xử lý lỗi dependency**

**Mô tả công việc**

- Deploy stack từ template đã tạo
    
- Ghi nhận lỗi dependency / parameter / naming
    
- Sửa template để deploy pass
    

**Đầu ra**

- Kết quả deploy test + danh sách lỗi/sửa
    

**Done khi**

- Stack deploy thành công ở mức tối thiểu
    

---

### **T4-11 — Validate tái tạo môi trường (recreate)**

**Mô tả công việc**

- Kiểm tra khả năng recreate (xóa/tạo lại theo phạm vi an toàn)
    
- Xác nhận template có thể dùng để dựng lại môi trường nhất quán
    
- Ghi giới hạn nếu còn bước manual
    

**Đầu ra**

- Biên bản validate recreate
    
- Danh sách manual steps còn lại
    

**Done khi**

- Chứng minh được khả năng dựng lại cùng môi trường bằng CloudFormation (đúng yêu cầu task)
    

---

# 🟦 TASK 5 — Xây dựng môi trường CI/CD

---

### **T5-01 — Quyết định công cụ CI/CD**

**Mô tả công việc**

- So sánh GitHub Actions / CodePipeline (hoặc công cụ đang dùng)
    
- Xét tiêu chí: nhanh triển khai, chi phí, tích hợp AWS, dễ vận hành
    
- Chốt phương án cho môi trường hiện tại
    

**Đầu ra**

- Quyết định công cụ + Điểm mạnh/ yếu + lý do chọn
    

**Done khi**

- Cả nhóm thống nhất công cụ triển khai
    

---

### **T5-02 — Thiết kế pipeline flow (Build/Test/Deploy)**

**Mô tả công việc**

- Xác định trigger (push/merge/manual)
    
- Xác định bước build, test, package, deploy
    
- Xác định target deploy (EC2/ASG) và cơ chế cập nhật
    

**Đầu ra**

- Sơ đồ pipeline + mô tả từng stage
    

**Done khi**

- Có thiết kế đủ chi tiết để cấu hình pipeline thực tế
    

---

### **T5-03 — Thiết lập repository và cấu hình CI cơ bản**

**Mô tả công việc**

- Chuẩn hóa repo structure (nếu cần)
    
- Thêm file workflow/pipeline config
    
- Cấu hình secrets/credentials cần thiết (theo best practice)
    
- Thiết lập build stage cơ bản
    

**Đầu ra**

- Pipeline config trong repo
    
- Build stage chạy được
    

**Done khi**

- Trigger CI và build pass ở mức cơ bản
    

---

### **T5-04 — Cấu hình deploy lên EC2/App environment**

**Mô tả công việc**

- Thiết lập cơ chế deploy artifact lên App Server/ASG
    
- Cấu hình script deploy (copy/restart/check service)
    
- Bảo đảm deploy không cần truy cập thủ công quá nhiều
    

**Đầu ra**

- Deploy stage hoạt động
    
- Script deploy
    

**Done khi**

- Có thể deploy app từ pipeline xuống môi trường dev
    

---

### **T5-05 — Test auto deploy end-to-end**

**Mô tả công việc**

- Thực hiện thay đổi nhỏ trong code
    
- Trigger pipeline
    
- Xác nhận build → deploy → app cập nhật thành công
    
- Kiểm tra endpoint sau deploy
    

**Đầu ra**

- Kết quả test E2E CI/CD + log/screenshot
    

**Done khi**

- Một vòng auto deploy hoàn tất thành công
    

---

### **T5-06 — Thiết lập rollback cơ bản và hướng dẫn vận hành**

**Mô tả công việc**

- Thiết lập cơ chế rollback tối thiểu (re-deploy bản trước / script revert)
    
- Xác định điều kiện rollback
    
- Viết hướng dẫn vận hành ngắn cho team
    

**Đầu ra**

- Phương án rollback cơ bản
    
- Tài liệu thao tác rollback
    

**Done khi**

- Team có thể thực hiện rollback theo hướng dẫn khi deploy lỗi


===========
=================
## 🟦 TASK 1 — Xây dựng lại môi trường AWS theo thiết kế chuẩn

### Mục tiêu

Tạo một môi trường AWS:

- Đúng theo tài liệu thiết kế mạng
    
- Có tính bảo mật
    
- Có tính sẵn sàng cao
    
- Ứng dụng có thể chạy được
    
- Trong ngân sách ~10万円/tháng
    

### Nội dung chính gồm 5 nhóm việc

1️⃣ Thiết kế chi tiết (IP, Security, Cost)  
2️⃣ Dựng nền tảng mạng (VPC, Subnet, NAT, Proxy)  
3️⃣ Thiết lập bảo mật (Security Group, WAF, Bastion)  
4️⃣ Triển khai server và database (EC2, RDS, Redis, Load Balancer)  
5️⃣ Kiểm tra toàn bộ luồng hoạt động và chi phí

👉 Kết quả:  
Môi trường AWS hoàn chỉnh, ứng dụng chạy được, đúng thiết kế.

---

## 🟦 TASK 2 — Cảnh báo chi phí tự động

### Mục tiêu

Tránh vượt ngân sách.

### Làm gì?

- Tạo cơ chế theo dõi chi phí hàng tháng
    
- Khi đạt 50% / 75% / 100% ngân sách
    
- Hệ thống tự gửi cảnh báo lên Teams
    

👉 Kết quả:  
Team biết ngay khi chi phí tăng bất thường.

---

## 🟦 TASK 3 — Tự động tắt tài nguyên theo giờ

### Mục tiêu

Giảm chi phí môi trường dev.

### Làm gì?

- Hàng ngày đến giờ cố định
    
- EC2 và RDS tự động dừng
    
- Khi cần có thể bật lại
    

👉 Kết quả:  
Tiết kiệm chi phí vận hành ngoài giờ làm việc.

---

## 🟦 TASK 4 — Tạo template CloudFormation

### Mục tiêu

Có thể dựng lại cùng một môi trường bằng 1 lần chạy lệnh.

### Làm gì?

- Chuyển toàn bộ hạ tầng sang dạng Infrastructure as Code
    
- Viết template tạo VPC, server, database, security…
    
- Test khả năng dựng lại từ đầu
    

👉 Kết quả:  
Có thể recreate môi trường bất kỳ lúc nào.

---

## 🟦 TASK 5 — Xây dựng CI/CD

### Mục tiêu

Không deploy thủ công nữa.

### Làm gì?

- Tạo pipeline tự động:
    
    - Build
        
    - Test
        
    - Deploy
        
- Khi push code → hệ thống tự cập nhật lên server
    

👉 Kết quả:  
Quy trình phát triển chuyên nghiệp và ổn định hơn.