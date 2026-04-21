Dưới đây là **Chương 2** được viết lại chi tiết hơn, theo văn phong học thuật, có cấu trúc rõ ràng, phù hợp để đưa trực tiếp vào báo cáo:

---

# **CHƯƠNG 2. THIẾT KẾ KIẾN TRÚC HỆ THỐNG (SYSTEM ARCHITECTURE DESIGN)**

Chương này trình bày một cách có hệ thống các yêu cầu, nguyên tắc và quy trình thiết kế kiến trúc cho hệ thống quản lý nhân sự trên nền tảng điện toán đám mây AWS. Mục tiêu của chương là xây dựng một kiến trúc hệ thống đảm bảo các tiêu chí về bảo mật, tính sẵn sàng, hiệu năng và khả năng mở rộng, đồng thời giải quyết các rủi ro an ninh thường gặp trong môi trường Cloud.

---

## **2.1. Phân tích yêu cầu hệ thống**

### **2.1.1. Yêu cầu chức năng**

Hệ thống quản lý nhân sự (HRM) cần đáp ứng các chức năng cốt lõi sau:

- **Quản lý thông tin nhân sự:**  
    Cho phép lưu trữ, cập nhật và truy xuất thông tin nhân viên bao gồm thông tin cá nhân, hợp đồng lao động, lịch sử công tác và dữ liệu lương thưởng.
    
- **Quản lý tài khoản và phân quyền:**  
    Hệ thống hỗ trợ nhiều vai trò người dùng (quản trị viên, nhân viên, bộ phận nhân sự), mỗi vai trò có quyền truy cập và thao tác khác nhau.
    
- **Quản lý tài liệu:**  
    Cho phép tải lên, lưu trữ và truy xuất các tài liệu liên quan như CV, hợp đồng, quyết định bổ nhiệm.
    
- **Giao diện truy cập web:**  
    Người dùng truy cập hệ thống thông qua trình duyệt web với giao thức HTTP/HTTPS.
    

---

### **2.1.2. Yêu cầu phi chức năng**

Bên cạnh các chức năng chính, hệ thống cần đáp ứng các yêu cầu phi chức năng mang tính quyết định đến chất lượng hệ thống:

- **Bảo mật (Security):**  
    Bảo vệ dữ liệu nhạy cảm khỏi truy cập trái phép thông qua các cơ chế xác thực, phân quyền và mã hóa.
    
- **Hiệu năng (Performance):**  
    Đảm bảo thời gian phản hồi nhanh, xử lý hiệu quả khi có nhiều người dùng đồng thời.
    
- **Khả năng mở rộng (Scalability):**  
    Hệ thống có thể mở rộng linh hoạt theo nhu cầu sử dụng mà không ảnh hưởng đến hiệu năng.
    
- **Tính sẵn sàng (Availability):**  
    Đảm bảo hệ thống hoạt động liên tục với thời gian downtime tối thiểu, hỗ trợ cơ chế dự phòng và khôi phục.
    
- **Khả năng kiểm toán (Auditability):**  
    Ghi log và theo dõi các hoạt động truy cập, phục vụ cho việc kiểm tra và phát hiện sự cố.
    

---

## **2.2. Nguyên tắc và phương pháp thiết kế**

Để đảm bảo hệ thống đáp ứng các yêu cầu đề ra, quá trình thiết kế được xây dựng dựa trên các nguyên tắc sau:

### **2.2.1. Nguyên tắc bảo mật đa lớp (Defense in Depth)**

Hệ thống được thiết kế theo mô hình nhiều lớp bảo vệ, trong đó mỗi lớp đóng vai trò kiểm soát và giảm thiểu rủi ro. Nếu một lớp bị xâm nhập, các lớp còn lại vẫn duy trì khả năng bảo vệ hệ thống.

### **2.2.2. Nguyên tắc đặc quyền tối thiểu (Least Privilege)**

Mỗi người dùng hoặc dịch vụ chỉ được cấp quyền tối thiểu cần thiết để thực hiện nhiệm vụ, hạn chế tối đa khả năng lạm dụng quyền.

### **2.2.3. Phân vùng và cô lập hệ thống (Segmentation & Isolation)**

Hệ thống được chia thành nhiều vùng mạng độc lập nhằm hạn chế phạm vi ảnh hưởng khi xảy ra sự cố.

### **2.2.4. Thiết kế hướng dịch vụ (Service-Oriented Architecture)**

Tận dụng các dịch vụ managed của AWS nhằm giảm thiểu rủi ro vận hành và nâng cao độ tin cậy.

---

## **2.3. Tổng quan kiến trúc hệ thống**

Hệ thống được thiết kế theo mô hình kiến trúc nhiều lớp (multi-tier architecture), bao gồm bốn lớp chính:

1. **Lớp biên (Edge Layer)**
    
2. **Lớp mạng (Network Layer)**
    
3. **Lớp ứng dụng (Application Layer)**
    
4. **Lớp dữ liệu (Data Layer)**
    

Các lớp này phối hợp với nhau tạo thành một kiến trúc bảo mật toàn diện, đảm bảo luồng dữ liệu được kiểm soát chặt chẽ từ người dùng đến hệ thống backend.

---

## **2.4. Thiết kế chi tiết kiến trúc hệ thống**

### **2.4.1. Lớp biên (Edge/Perimeter Security Layer)**

Lớp biên đóng vai trò là tuyến phòng thủ đầu tiên của hệ thống trước các truy cập từ Internet.

- **Dịch vụ phân phối nội dung và DNS:**  
    Sử dụng hệ thống phân giải tên miền và CDN giúp tăng tốc độ truy cập và giảm tải cho máy chủ gốc. Đồng thời, địa chỉ IP thực của hệ thống backend được ẩn đi, hạn chế khả năng bị tấn công trực tiếp.
    
- **Tường lửa ứng dụng web (WAF):**  
    Được triển khai để kiểm tra và lọc các request HTTP/HTTPS. WAF cho phép:
    
    - Phát hiện các mẫu tấn công phổ biến như SQL Injection và XSS
        
    - Chặn các request bất thường
        
    - Giới hạn tốc độ truy cập nhằm giảm thiểu tấn công DDoS ở tầng ứng dụng
        

**So với mô hình truyền thống**, trong đó server thường được expose trực tiếp ra Internet, kiến trúc này bổ sung lớp bảo vệ trung gian giúp giảm đáng kể bề mặt tấn công.

---

### **2.4.2. Lớp mạng (Network Security Layer)**

Lớp mạng được thiết kế nhằm kiểm soát và phân tách lưu lượng giữa các thành phần trong hệ thống.

- **Virtual Private Cloud (VPC):**  
    Toàn bộ hệ thống được triển khai trong một mạng ảo riêng biệt, đảm bảo tính cô lập với các hệ thống khác.
    
- **Phân chia Subnet:**
    
    - **Public Subnet:** chứa các thành phần cần giao tiếp với Internet như Load Balancer và Bastion Host
        
    - **Private Subnet:** chứa Application Server và Database, không cho phép truy cập trực tiếp từ bên ngoài
        
- **Cơ chế kiểm soát truy cập mạng:**
    
    - Security Groups: kiểm soát lưu lượng ở cấp instance
        
    - Network ACLs: kiểm soát lưu lượng ở cấp subnet
        

Ví dụ:

- Database chỉ cho phép nhận kết nối từ Application Server
    
- Không cho phép truy cập trực tiếp từ Internet vào tài nguyên trong Private Subnet
    

**Khác với kiến trúc không phân tách mạng**, cách tiếp cận này giúp giảm thiểu nguy cơ lây lan khi một thành phần bị xâm nhập.

---

### **2.4.3. Lớp ứng dụng và cân bằng tải (Application Layer)**

Lớp ứng dụng chịu trách nhiệm xử lý logic nghiệp vụ và giao tiếp với người dùng.

- **Load Balancer:**  
    Phân phối lưu lượng truy cập đến nhiều Application Server, giúp:
    
    - Tăng khả năng chịu tải
        
    - Đảm bảo tính sẵn sàng cao
        
    - Hỗ trợ mã hóa dữ liệu qua giao thức SSL/TLS
        
- **Application Server:**  
    Xử lý các yêu cầu từ người dùng và thực hiện logic nghiệp vụ của hệ thống HRM.
    
- **Bastion Host:**  
    Là điểm truy cập duy nhất cho quản trị viên khi cần kết nối vào hệ thống nội bộ. Việc sử dụng Bastion Host giúp:
    
    - Kiểm soát truy cập quản trị
        
    - Ghi log hoạt động
        
    - Hạn chế truy cập trực tiếp vào tài nguyên quan trọng
        

**So với phương pháp truy cập trực tiếp vào server**, mô hình này giúp tăng cường khả năng kiểm soát và giảm thiểu rủi ro bảo mật.

---

### **2.4.4. Lớp dữ liệu (Data Security Layer)**

Lớp dữ liệu chịu trách nhiệm lưu trữ và bảo vệ dữ liệu quan trọng của hệ thống.

- **Cơ sở dữ liệu:**  
    Dữ liệu nhân sự được lưu trữ trong hệ quản trị cơ sở dữ liệu có hỗ trợ mã hóa và sao lưu tự động.
    
- **Lưu trữ đối tượng:**  
    Các file tài liệu được lưu trữ trong hệ thống object storage với chính sách truy cập nghiêm ngặt.
    
- **Mã hóa dữ liệu:**  
    Dữ liệu được mã hóa ở cả hai trạng thái:
    
    - **At rest:** khi lưu trữ
        
    - **In transit:** khi truyền tải
        
- **Quản lý khóa mã hóa:**  
    Sử dụng dịch vụ quản lý khóa để đảm bảo an toàn và kiểm soát việc sử dụng khóa.
    

**Khác với hệ thống không áp dụng mã hóa**, giải pháp này đảm bảo dữ liệu vẫn an toàn ngay cả khi bị truy cập trái phép ở mức hạ tầng.

---

## **2.5. Quy trình thiết kế hệ thống**

Quy trình thiết kế kiến trúc hệ thống được thực hiện theo các bước tuần tự như sau:

1. **Thu thập và phân tích yêu cầu**  
    Xác định rõ nhu cầu chức năng và phi chức năng của hệ thống.
    
2. **Phân tích rủi ro và mối đe dọa**  
    Xác định các điểm yếu tiềm ẩn và các kịch bản tấn công có thể xảy ra.
    
3. **Lựa chọn mô hình kiến trúc phù hợp**  
    Áp dụng mô hình đa lớp nhằm đảm bảo bảo mật toàn diện.
    
4. **Thiết kế chi tiết từng lớp**  
    Xây dựng kiến trúc cụ thể cho từng lớp: Edge, Network, Application, Data.
    
5. **Đánh giá và tối ưu thiết kế**  
    Kiểm tra khả năng đáp ứng yêu cầu và tối ưu hiệu năng.
    
6. **Chuẩn bị triển khai**  
    Hoàn thiện tài liệu thiết kế và sẵn sàng cho giai đoạn triển khai thực tế.
    

---

## **Kết chương 2**

Chương 2 đã trình bày chi tiết quá trình phân tích yêu cầu và thiết kế kiến trúc cho hệ thống quản lý nhân sự trên nền tảng AWS. Kiến trúc được xây dựng theo mô hình đa lớp, kết hợp các nguyên tắc bảo mật hiện đại nhằm đảm bảo tính bảo mật, toàn vẹn và sẵn sàng của hệ thống.

Quy trình thiết kế được thực hiện một cách có hệ thống, từ việc phân tích yêu cầu đến xây dựng kiến trúc và tối ưu giải pháp. Đây là cơ sở quan trọng để triển khai hệ thống trong thực tế và kiểm chứng hiệu quả thông qua các kịch bản thử nghiệm trong chương tiếp theo.

---

Nếu bạn cần, mình có thể:

- Viết tiếp **Chương 3 (triển khai + demo chi tiết từng bước trên AWS)**
    
- Hoặc chỉnh lại **ngôn ngữ cho đúng format luận văn (có trích dẫn, numbering chuẩn IEEE/APA)**



---

# **CHƯƠNG 3. XÂY DỰNG VÀ TRIỂN KHAI HỆ THỐNG (SYSTEM IMPLEMENTATION AND DEMONSTRATION)**

Chương này trình bày quá trình triển khai thực tế hệ thống quản lý nhân sự trên nền tảng AWS dựa trên kiến trúc đã thiết kế ở Chương 2. Đồng thời, chương này cũng xây dựng các kịch bản thử nghiệm (Proof of Concept) nhằm đánh giá khả năng bảo mật và hiệu quả vận hành của hệ thống.

---

## **3.1. Môi trường triển khai**

Hệ thống được triển khai trên nền tảng AWS với các dịch vụ chính:

- Dịch vụ mạng: VPC, Subnet, Internet Gateway, NAT Gateway
    
- Dịch vụ bảo mật: Security Groups, Network ACLs, WAF
    
- Dịch vụ tính toán: EC2 (Application Server, Bastion Host)
    
- Dịch vụ cân bằng tải: Application Load Balancer (ALB)
    
- Dịch vụ lưu trữ: RDS (Database), S3 (Object Storage)
    
- Dịch vụ CDN: CloudFront
    
- Dịch vụ mã hóa: KMS
    

Hệ thống được triển khai theo mô hình nhiều lớp nhằm đảm bảo tính bảo mật và khả năng mở rộng.

---

## **3.2. Quy trình triển khai hệ thống**

Quá trình triển khai được thực hiện theo các bước chính sau:

### **3.2.1. Thiết lập hạ tầng mạng**

- Tạo VPC với dải địa chỉ IP riêng
    
- Tạo các Public Subnet và Private Subnet
    
- Cấu hình Internet Gateway cho Public Subnet
    
- Cấu hình NAT Gateway để Private Subnet có thể truy cập Internet khi cần
    

### **3.2.2. Cấu hình bảo mật mạng**

- Thiết lập Security Groups cho từng thành phần:
    
    - ALB cho phép HTTP/HTTPS từ Internet
        
    - Application Server chỉ nhận traffic từ ALB
        
    - Database chỉ nhận kết nối từ Application Server
        
- Cấu hình Network ACL để tăng cường kiểm soát lưu lượng
    

### **3.2.3. Triển khai máy chủ và dịch vụ**

- Khởi tạo EC2 Instance cho Application Server trong Private Subnet
    
- Thiết lập Bastion Host trong Public Subnet để truy cập quản trị
    
- Cài đặt và cấu hình ứng dụng HRM trên Application Server
    

### **3.2.4. Cấu hình Load Balancer và CDN**

- Tạo Application Load Balancer để phân phối lưu lượng
    
- Kết nối ALB với các Application Server
    
- Triển khai CloudFront để phân phối nội dung và tăng cường bảo mật
    

### **3.2.5. Triển khai hệ thống dữ liệu**

- Tạo cơ sở dữ liệu RDS trong Private Subnet
    
- Cấu hình backup tự động và Multi-AZ
    
- Tạo S3 Bucket để lưu trữ tài liệu và cấu hình chặn truy cập public
    

### **3.2.6. Cấu hình mã hóa và quản lý khóa**

- Sử dụng KMS để quản lý khóa mã hóa
    
- Áp dụng mã hóa cho RDS và S3
    
- Bật HTTPS cho toàn bộ hệ thống
    

---

## **3.3. Kịch bản kiểm thử (Proof of Concept)**

### **3.3.1. Demo 1: Kiểm tra truy cập hợp lệ và không hợp lệ**

**Mục tiêu:** Kiểm tra khả năng kiểm soát truy cập của hệ thống

- Thực hiện ping hoặc SSH trực tiếp vào Application Server từ Internet  
    → Kết quả: Không thể truy cập (bị chặn)
    
- Thực hiện SSH thông qua Bastion Host  
    → Kết quả: Truy cập thành công
    

**Nhận xét:**  
Hệ thống đã đảm bảo nguyên tắc không cho phép truy cập trực tiếp vào tài nguyên nội bộ.

---

### **3.3.2. Demo 2: Kiểm tra chống tấn công Web**

**Mục tiêu:** Đánh giá khả năng phát hiện và ngăn chặn tấn công ứng dụng

- Gửi request chứa mã SQL Injection vào form đăng nhập
    
- Hệ thống WAF phát hiện và chặn request
    

→ Kết quả: Trả về mã lỗi **403 Forbidden**

**Nhận xét:**  
WAF hoạt động hiệu quả trong việc bảo vệ hệ thống trước các tấn công phổ biến.

---

### **3.3.3. Demo 3: Kiểm tra bảo mật truyền tải dữ liệu**

**Mục tiêu:** Đảm bảo dữ liệu được mã hóa khi truyền tải

- Truy cập hệ thống bằng HTTP
    
- Hệ thống tự động chuyển hướng sang HTTPS
    

**Nhận xét:**  
Toàn bộ dữ liệu được bảo vệ bằng SSL/TLS, đảm bảo an toàn khi truyền tải.

---

## **3.4. Đánh giá kết quả triển khai**

Sau khi triển khai và kiểm thử, hệ thống đạt được các kết quả:

- Đảm bảo kiểm soát truy cập chặt chẽ
    
- Ngăn chặn hiệu quả các tấn công phổ biến
    
- Đảm bảo mã hóa dữ liệu toàn diện
    
- Hệ thống hoạt động ổn định và có khả năng mở rộng
    

Tuy nhiên, hệ thống vẫn có thể được cải thiện thêm về tự động hóa và giám sát.

---

## **Kết chương 3**

Chương 3 đã trình bày quá trình triển khai thực tế hệ thống quản lý nhân sự trên AWS và các kịch bản kiểm thử nhằm đánh giá hiệu quả của kiến trúc đã thiết kế. Kết quả cho thấy hệ thống đáp ứng tốt các yêu cầu về bảo mật, hiệu năng và tính sẵn sàng, đồng thời chứng minh tính khả thi của giải pháp đề xuất.

---

# **KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN**

## **1. Kết luận**

Đề tài đã thực hiện việc thiết kế và triển khai một hệ thống quản lý nhân sự an toàn trên nền tảng AWS dựa trên mô hình bảo mật đa lớp. Hệ thống đã áp dụng các nguyên tắc bảo mật hiện đại như Defense in Depth và Principle of Least Privilege, đảm bảo các tiêu chí của mô hình CIA.

Thông qua quá trình triển khai và kiểm thử, hệ thống đã chứng minh khả năng:

- Bảo vệ dữ liệu nhạy cảm
    
- Ngăn chặn các tấn công phổ biến
    
- Đảm bảo tính sẵn sàng và khả năng mở rộng
    

Kết quả đạt được cho thấy giải pháp đề xuất là khả thi và phù hợp với các hệ thống thực tế.


## **2. Hướng phát triển**

Trong tương lai, hệ thống có thể được mở rộng và nâng cấp theo các hướng sau:

- **Triển khai Zero Trust Architecture:**  
    Tăng cường xác thực và kiểm soát truy cập ở mọi cấp độ
    
- **Tích hợp hệ thống giám sát và phát hiện xâm nhập:**  
    Sử dụng các công cụ như hệ thống phân tích log và phát hiện bất thường
    
- **Tự động hóa bảo mật:**  
    Áp dụng Infrastructure as Code (IaC) và các công cụ kiểm tra cấu hình
    
- **Cải thiện kế hoạch khôi phục thảm họa (Disaster Recovery):**
    
    - Triển khai sao lưu đa vùng (Cross-Region)
        
    - Tối ưu RTO và RPO
        
    - Xây dựng kịch bản failover tự động
        
- **Tích hợp AI/ML trong bảo mật:**  
    Phát hiện sớm các hành vi bất thường và tấn công nâng cao
    


Nếu bạn muốn, mình có thể:

- Format lại toàn bộ báo cáo theo **chuẩn Word (heading, numbering, mục lục tự động)**
    
- Viết thêm **Tài liệu tham khảo (APA/IEEE)**
    
- Hoặc làm luôn **slide thuyết trình 10–15 trang** 🚀


--- 
---
---
Dưới đây là phiên bản **viết lại hoàn toàn theo văn phong học thuật, dạng đoạn văn (không bullet points)** cho **Chương 3 và Kết luận**:

---

# **CHƯƠNG 3. XÂY DỰNG VÀ TRIỂN KHAI HỆ THỐNG (SYSTEM IMPLEMENTATION AND DEMONSTRATION)**

Chương này trình bày quá trình triển khai thực tế hệ thống quản lý nhân sự trên nền tảng điện toán đám mây AWS dựa trên kiến trúc đã được đề xuất trong Chương 2. Đồng thời, chương này xây dựng các kịch bản thử nghiệm nhằm đánh giá hiệu quả của các cơ chế bảo mật được tích hợp trong hệ thống, qua đó kiểm chứng tính khả thi và mức độ đáp ứng các yêu cầu đặt ra.

## **3.1. Môi trường và kiến trúc triển khai**

Hệ thống được triển khai trên nền tảng AWS với việc sử dụng kết hợp nhiều dịch vụ nhằm đảm bảo tính bảo mật và khả năng mở rộng. Hạ tầng mạng được thiết lập thông qua một Virtual Private Cloud (VPC), trong đó các tài nguyên được phân bố vào các subnet công khai và riêng tư nhằm đảm bảo tính cô lập và kiểm soát truy cập.

Các thành phần chính của hệ thống bao gồm máy chủ ứng dụng được triển khai trên các instance trong private subnet, cơ sở dữ liệu được quản lý thông qua dịch vụ RDS, và hệ thống lưu trữ file được xây dựng trên S3. Bên cạnh đó, Application Load Balancer được sử dụng để phân phối lưu lượng truy cập, trong khi CloudFront đóng vai trò như một lớp phân phối nội dung trung gian, giúp cải thiện hiệu năng và tăng cường bảo mật. Hệ thống cũng sử dụng các cơ chế bảo mật như Security Groups, Network ACLs và Web Application Firewall nhằm kiểm soát và giám sát lưu lượng truy cập.

## **3.2. Quy trình triển khai hệ thống**

Quá trình triển khai hệ thống được thực hiện theo một quy trình có cấu trúc nhằm đảm bảo tính nhất quán và hiệu quả. Trước hết, hạ tầng mạng được thiết lập bằng cách tạo VPC với dải địa chỉ IP riêng, sau đó phân chia thành các subnet công khai và riêng tư. Internet Gateway được cấu hình để cho phép các thành phần trong public subnet giao tiếp với Internet, trong khi NAT Gateway được sử dụng để cho phép các tài nguyên trong private subnet truy cập Internet một cách gián tiếp khi cần thiết.

Tiếp theo, các cơ chế bảo mật mạng được cấu hình thông qua việc thiết lập Security Groups và Network ACLs. Các quy tắc truy cập được xây dựng theo nguyên tắc đặc quyền tối thiểu, trong đó mỗi thành phần chỉ được phép giao tiếp với các thành phần cần thiết. Sau đó, các máy chủ ứng dụng được triển khai trong private subnet và được cấu hình để xử lý các yêu cầu từ người dùng thông qua Load Balancer.

Hệ thống cũng thiết lập một Bastion Host trong public subnet để phục vụ cho việc quản trị, đảm bảo rằng các truy cập quản trị không được thực hiện trực tiếp vào các tài nguyên nội bộ. Đồng thời, cơ sở dữ liệu được triển khai trên RDS với các cơ chế sao lưu tự động và cấu hình Multi-AZ nhằm đảm bảo tính sẵn sàng cao.

Cuối cùng, các cơ chế mã hóa được áp dụng nhằm bảo vệ dữ liệu ở cả trạng thái lưu trữ và truyền tải. Việc sử dụng HTTPS được bắt buộc đối với toàn bộ hệ thống, trong khi các dịch vụ như RDS và S3 được cấu hình để mã hóa dữ liệu thông qua hệ thống quản lý khóa.

## **3.3. Kịch bản kiểm thử và đánh giá**

Để đánh giá hiệu quả của hệ thống, một số kịch bản thử nghiệm đã được xây dựng nhằm mô phỏng các tình huống tấn công và truy cập thực tế. Trong kịch bản đầu tiên, hệ thống được kiểm tra khả năng kiểm soát truy cập bằng cách thử kết nối trực tiếp đến các máy chủ ứng dụng trong private subnet từ Internet. Kết quả cho thấy các kết nối này đều bị từ chối, trong khi việc truy cập thông qua Bastion Host vẫn được thực hiện thành công. Điều này chứng minh rằng hệ thống đã triển khai hiệu quả cơ chế cô lập mạng.

Trong kịch bản thứ hai, hệ thống được kiểm tra khả năng phòng chống các cuộc tấn công ứng dụng web. Một yêu cầu chứa mã SQL Injection được gửi đến hệ thống thông qua giao diện đăng nhập. Kết quả cho thấy yêu cầu này đã bị chặn bởi Web Application Firewall và trả về mã lỗi phù hợp, đồng thời được ghi nhận trong hệ thống log. Điều này chứng minh khả năng phát hiện và ngăn chặn các tấn công phổ biến của hệ thống.

Trong kịch bản thứ ba, hệ thống được kiểm tra về bảo mật truyền tải dữ liệu. Khi người dùng truy cập hệ thống thông qua giao thức HTTP, hệ thống tự động chuyển hướng sang HTTPS, đảm bảo rằng toàn bộ dữ liệu được mã hóa trong quá trình truyền tải. Kết quả này cho thấy hệ thống đã đáp ứng tốt yêu cầu về bảo mật dữ liệu trong môi trường mạng.

## **3.4. Đánh giá tổng thể**

Kết quả triển khai và kiểm thử cho thấy hệ thống đã đáp ứng tốt các yêu cầu đề ra về bảo mật, hiệu năng và tính sẵn sàng. Kiến trúc đa lớp giúp giảm thiểu rủi ro từ các mối đe dọa phổ biến, trong khi việc sử dụng các dịch vụ quản lý của AWS giúp đơn giản hóa quá trình vận hành và nâng cao độ tin cậy của hệ thống. Tuy nhiên, hệ thống vẫn còn tiềm năng để cải thiện, đặc biệt trong việc tự động hóa và giám sát an ninh.

## **Kết chương 3**

Chương 3 đã trình bày quá trình triển khai thực tế hệ thống và các kịch bản kiểm thử nhằm đánh giá hiệu quả của kiến trúc đề xuất. Các kết quả thu được cho thấy hệ thống hoạt động ổn định và đáp ứng tốt các yêu cầu bảo mật, qua đó khẳng định tính khả thi của giải pháp trong thực tế.

---

# **KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN**

## **1. Kết luận**

Đề tài đã thực hiện việc nghiên cứu, thiết kế và triển khai một hệ thống quản lý nhân sự trên nền tảng AWS với trọng tâm là đảm bảo an toàn thông tin. Thông qua việc áp dụng mô hình bảo mật đa lớp và các nguyên tắc bảo mật hiện đại, hệ thống đã đáp ứng được các tiêu chí của mô hình CIA, bao gồm bảo mật, toàn vẹn và sẵn sàng của dữ liệu.

Quá trình triển khai và kiểm thử đã chứng minh rằng hệ thống có khả năng ngăn chặn các truy cập trái phép, bảo vệ dữ liệu trước các tấn công phổ biến và đảm bảo hoạt động ổn định trong môi trường thực tế. Kết quả đạt được cho thấy giải pháp đề xuất không chỉ mang tính lý thuyết mà còn có thể áp dụng hiệu quả trong các hệ thống thực tiễn.

## **2. Hướng phát triển**

Trong tương lai, hệ thống có thể được mở rộng theo nhiều hướng nhằm nâng cao hơn nữa mức độ bảo mật và khả năng vận hành. Một trong những hướng phát triển quan trọng là triển khai mô hình Zero Trust, trong đó mọi truy cập đều phải được xác thực và kiểm tra nghiêm ngặt. Bên cạnh đó, việc tích hợp các hệ thống giám sát và phát hiện xâm nhập sẽ giúp phát hiện sớm các hành vi bất thường.

Ngoài ra, hệ thống có thể được cải tiến bằng cách áp dụng các công cụ tự động hóa trong quản lý hạ tầng và bảo mật, giúp giảm thiểu sai sót cấu hình. Đặc biệt, việc xây dựng kế hoạch khôi phục thảm họa với cơ chế sao lưu đa vùng và tối ưu các chỉ số RTO, RPO sẽ giúp hệ thống duy trì hoạt động ngay cả trong các tình huống sự cố nghiêm trọng.

Việc kết hợp các công nghệ mới như trí tuệ nhân tạo và học máy trong phân tích hành vi cũng là một hướng đi tiềm năng nhằm nâng cao khả năng phát hiện và phòng chống các mối đe dọa an ninh trong tương lai.

---

Nếu bạn cần, mình có thể giúp bạn **ghép toàn bộ thành 1 file Word chuẩn luận văn (có mục lục tự động, numbering chuẩn)** hoặc chỉnh lại đúng format của trường bạn.