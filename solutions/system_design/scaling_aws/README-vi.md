*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Hệ thống Mở rộng lên Hàng Triệu Người Dùng trên AWS (Scaling AWS)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

---

## Bước 1: Xác định rõ Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

Trong môi trường phỏng vấn thực tế, bạn sẽ hỏi người phỏng vấn để làm rõ. Ở đây, chúng ta sẽ tự đặt ra các use case và ràng buộc cụ thể cho bài toán.

### Use Cases (Các trường hợp sử dụng)

Để giải quyết bài toán này, chúng ta áp dụng quy trình lặp 4 bước: 
1. **Benchmark / Load Test** (Đo kiểm hiệu năng / Thử tải)
2. **Profile** (Phân tích điểm nghẽn cổ chai)
3. **Giải quyết điểm nghẽn** bằng cách đánh giá các giải pháp thay thế và sự đánh đổi (Trade-offs)
4. **Lặp lại chu trình trên**

Đây là khuôn mẫu chuẩn để nâng cấp một kiến trúc cơ bản thành một kiến trúc có khả năng chịu tải hàng chục triệu người dùng.

*Lưu ý:* Trừ khi bạn ứng tuyển vào vị trí đòi hỏi kiến thức chuyên sâu về AWS, còn lại các chi tiết riêng của AWS không phải là bắt buộc. **Hầu hết các nguyên lý trong bài tập này đều áp dụng rộng rãi cho bất kỳ nền tảng đám mây nào khác (GCP, Azure, hoặc On-premise).**

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Người dùng (User)** gửi request Đọc (Read) hoặc Ghi (Write):
  * **Hệ thống (Service)** tiếp nhận, xử lý logic, lưu trữ dữ liệu người dùng và trả về kết quả.
* **Hệ thống** cần tiến hóa dần từ lúc chỉ phục vụ một lượng nhỏ người dùng cho tới hàng chục triệu người dùng.
* **Hệ thống** phải đảm bảo tính sẵn sàng cao (High Availability).

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng truy cập không phân bổ đều (có giờ cao điểm, giờ thấp điểm).
* Dữ liệu có tính quan hệ (Relational data).
* Mở rộng từ **1 người dùng** lên **hơn 10 triệu người dùng**:
  * $10$ triệu người dùng hoạt động.
  * $1$ tỷ thao tác Ghi (Writes) mỗi tháng.
  * $100$ tỷ thao tác Đọc (Reads) mỗi tháng.
  * Tỉ lệ Đọc / Ghi: **100:1** (Hệ thống thiên về Đọc - Read-heavy).
  * Kích thước nội dung mỗi lần ghi: $1\text{ KB}$.

#### Tính toán ước lượng sơ bộ (Back-of-the-envelope calculations):
* **Dung lượng lưu trữ nội dung mới mỗi tháng:**
  * $1\text{ KB} \times 1\text{ tỷ writes} = 1\text{ TB}$ nội dung mới mỗi tháng.
  * $36\text{ TB}$ dữ liệu mới sau 3 năm.
  * Giả định hầu hết các thao tác ghi là tạo mới nội dung thay vì cập nhật nội dung cũ.
* **Tốc độ xử lý trung bình mỗi giây:**
  * Ghi trung bình: $\approx 400\text{ writes/giây}$.
  * Đọc trung bình: $\approx 40.000\text{ reads/giây}$.

*(Quy đổi nhanh: 1 tháng có khoảng 2.5 triệu giây $\rightarrow$ 1 request/giây $\approx$ 2.5 triệu requests/tháng).*

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/B8LDKD7.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Use case: Người dùng gửi request Đọc hoặc Ghi

#### Mục tiêu ban đầu (1 - 2 Users):
* Khi chỉ có vài người dùng, bạn chỉ cần một thiết lập đơn giản nhất có thể:
  * Tất cả gói gọn trong một máy chủ duy nhất (Single box).
  * Áp dụng Scale dọc (Vertical Scaling) khi cần.
  * Cài đặt giám sát (Monitoring) để phát hiện điểm nghẽn.

#### Bắt đầu với một máy chủ duy nhất:
* **Web Server** chạy trên một máy ảo EC2.
* Lưu trữ dữ liệu người dùng trực tiếp trên cùng máy: **MySQL Database**.

#### Mở rộng theo chiều dọc (Vertical Scaling):
* Chỉ cần chọn một cỗ máy to hơn, cấu hình mạnh hơn (nhiều CPU, RAM, ổ SSD nhanh hơn).
* Theo dõi các chỉ số tài nguyên: CPU, RAM, Disk I/O, Network để biết khi nào cần nâng cấp máy (dùng CloudWatch, `top`, StatsD...).
* *Nhược điểm:* Rất đắt đỏ khi lên cấu hình cao, và không có cơ chế dự phòng (nếu máy sập là toàn bộ dịch vụ chết hoàn toàn - Single Point of Failure).

#### Dùng SQL và cân nhắc NoSQL:
* Bài toán yêu cầu dữ liệu quan hệ, do đó bắt đầu bằng **MySQL Database** là lựa chọn hợp lý.

#### Gán IP tĩnh và cấu hình DNS:
* Sử dụng **Elastic IP**: Cung cấp địa chỉ IP tĩnh không đổi ngay cả khi khởi động lại server.
* Dùng dịch vụ **DNS (Route 53)** để trỏ tên miền về IP của máy chủ.

#### Bảo mật máy chủ:
* Chỉ mở các cổng mạng tối thiểu cần thiết:
  * Cổng 80 cho HTTP.
  * Cổng 443 cho HTTPS.
  * Cổng 22 cho SSH (chỉ cho phép kết nối từ dải IP tin cậy được đưa vào Whitelist).
* Ngăn chặn Web Server tự ý tạo kết nối chiều ra ngoài không kiểm soát.

---

## Bước 4: Mở rộng thiết kế theo từng giai đoạn (Scale the Design)

---

### Giai đoạn 1: Users+ (Vài nghìn người dùng)

<p align="center">
  <img src="http://i.imgur.com/rrfjMXB.png">
  <br/>
</p>

#### Bối cảnh:
Lượng người dùng bắt đầu tăng lên, tải đè nặng lên máy chủ duy nhất. Qua công cụ phân tích (Profiling), ta nhận thấy **MySQL Database** ngốn sạch RAM và CPU, trong khi ổ cứng sắp đầy do chứa ảnh và file người dùng tải lên. Việc nâng cấp máy chủ (Vertical Scaling) bắt đầu trở nên quá tốn kém và không thể mở rộng riêng rẽ Web Server và Database được nữa.

#### Giải pháp:
1. **Tách dữ liệu tĩnh ra Object Store riêng biệt (AWS S3):**
   * Đưa toàn bộ file tĩnh (ảnh, video, file tải lên của user, CSS, JS) sang **S3**.
   * S3 có độ bền cực cao (99.999999999%), dung lượng vô hạn và tự động scale.
2. **Tách MySQL Database sang một máy chủ riêng (AWS RDS):**
   * Chuyển MySQL sang dịch vụ **Amazon RDS** chạy trên máy chủ độc lập.
   * Giúp Web Server và Database không còn tranh chấp RAM và CPU với nhau.
   * Dễ dàng cấu hình sao lưu tự động và dự phòng đa vùng sẵn sàng (Multi-AZ).
3. **Bảo mật mạng với VPC (Virtual Private Cloud):**
   * **Public Subnet:** Đặt Web Server ở đây để nhận traffic từ internet.
   * **Private Subnet:** Đặt Database và các tài nguyên nội bộ ở đây, tuyệt đối không mở IP công khai ra internet. Chỉ Web Server trong nội bộ mới có quyền truy cập vào Database.

---

### Giai đoạn 2: Users++ (Hàng chục nghìn đến 100.000 người dùng)

<p align="center">
  <img src="http://i.imgur.com/raoFTXM.png">
  <br/>
</p>

#### Bối cảnh:
Vào các khung giờ cao điểm, một chiếc Web Server duy nhất bị nghẽn (bottleneck), dẫn đến phản hồi chậm hoặc thậm chí sập web. Cần tăng độ sẵn sàng (High Availability) và khả năng dự phòng (Redundancy).

#### Giải pháp:
1. **Chuyển sang Scale ngang (Horizontal Scaling) với Load Balancer:**
   * Đặt một bộ cân bằng tải **Load Balancer (ELB / HAProxy)** ở phía trước.
   * Nhân bản ra **nhiều Web Servers** đặt rải rác trên nhiều vùng khả dụng khác nhau (Availability Zones - AZ) để nếu một trung tâm dữ liệu gặp sự cố, các server ở vùng khác vẫn chạy bình thường.
   * **SSL Termination:** Thực hiện giải mã HTTPS ngay tại Load Balancer để giải phóng CPU cho các Web Server.
2. **Dự phòng cho Database (Master - Slave Failover):**
   * Cấu hình MySQL RDS chạy ở chế độ **Multi-AZ**: Một Master nhận ghi/đọc chính, một Standby Slave ở vùng khác sẵn sàng tiếp quản nếu Master gặp sự cố.
3. **Tách tầng Web Server và Tầng Ứng dụng (Application Server):**
   * Web Server đóng vai trò **Reverse Proxy**, điều phối request.
   * Application Server tập trung xử lý logic nghiệp vụ. Có thể chia cụm server chuyên xử lý API Đọc và cụm server chuyên xử lý API Ghi.
4. **Đưa nội dung tĩnh lên mạng phân phối CDN (CloudFront):**
   * Đặt **CDN (CloudFront / Cloudflare)** trước S3.
   * Nội dung tĩnh được phân phối từ các Edge Server gần người dùng nhất, giảm tải 80-90% băng thông cho server gốc và giảm độ trễ từ hàng trăm ms xuống còn 10-20ms.

---

### Giai đoạn 3: Users+++ (Hàng trăm nghìn đến 1.000.000 người dùng)

<p align="center">
  <img src="http://i.imgur.com/OZCxJr0.png">
  <br/>
</p>

*(Lưu ý: Các Internal Load Balancers được ẩn bớt trên sơ đồ để hình vẽ gọn gàng).*

#### Bối cảnh:
Tỉ lệ truy cập là **100 Đọc : 1 Ghi** (Read-heavy). Với hàng triệu lượt xem mỗi ngày, Database bắt đầu quá tải vì phải liên tục xử lý các câu lệnh `SELECT` lặp đi lặp lại.

#### Giải pháp:
1. **Thêm tầng In-Memory Cache (Redis / Memcached - AWS ElastiCache):**
   * Đặt Cache phía trước Database theo mô hình **Cache-Aside**.
   * Cache các dữ liệu được đọc thường xuyên nhất.
   * *So sánh tốc độ:* Đọc 1 MB tuần tự từ RAM chỉ mất khoảng **250 micro-giây**, nhanh hơn 4 lần so với SSD và nhanh hơn **80 lần** so với ổ đĩa cơ.
2. **Lưu trữ Session người dùng tập trung:**
   * Chuyển toàn bộ Session đăng nhập của người dùng vào **Redis Cache** thay vì lưu trong Web Server.
   * Web Server trở thành hoàn toàn **Stateless (phi trạng thái)** $\rightarrow$ Giúp việc thêm bớt máy chủ trở nên cực kỳ dễ dàng.
3. **Thêm các bản sao chỉ đọc của Database (MySQL Read Replicas):**
   * Cấu hình **Read Replicas**: Mọi thao tác Ghi (`INSERT`, `UPDATE`, `DELETE`) được gửi tới **Master Database**; mọi thao tác Đọc (`SELECT`) được phân phối đều tới các **Read Replicas**.
   * Đặt Load Balancer phía trước các bản sao Read Replicas.

---

### Giai đoạn 4: Users++++ (Vài triệu người dùng)

<p align="center">
  <img src="http://i.imgur.com/3X8nmdL.png">
  <br/>
</p>

#### Bối cảnh:
Lượng truy cập tăng vọt vào ban ngày và giảm mạnh vào ban đêm. Việc duy trì một lượng lớn máy chủ cố định 24/7 gây lãng phí chi phí khổng lồ. Đồng thời, việc quản lý hạ tầng bằng tay bắt đầu vượt ngoài tầm kiểm soát của đội ngũ kỹ sư nhỏ.

#### Giải pháp:
1. **Tự động co giãn máy chủ (Autoscaling):**
   * Tạo các nhóm **Auto Scaling Groups (ASG)** cho Web Servers và Application Servers.
   * Thiết lập ngưỡng co giãn (Scaling Policies) dựa trên các chỉ số giám sát từ CloudWatch:
     * Tự động bật thêm máy chủ khi CPU vượt quá 70% hoặc lượng Request tăng vọt.
     * Tự động tắt bớt máy chủ vào ban đêm khi nhu cầu giảm để tiết kiệm tiền.
2. **Tự động hóa DevOps & Quản lý cấu hình:**
   * Sử dụng các công cụ Infrastructure as Code (Terraform, Ansible, Chef, Puppet) để việc tạo mới môi trường diễn ra hoàn toàn tự động chỉ với 1 cú click.
3. **Hệ thống giám sát và cảnh báo toàn diện:**
   * **Host level:** Giám sát từng cá thể EC2.
   * **Log Analysis:** Thu thập và phân tích log tập trung (CloudWatch Logs, Elasticsearch/OpenSearch, Splunk).
   * **APM & Performance:** Theo dõi hiệu năng ứng dụng (New Relic, Datadog).
   * **Báo lỗi & Sự cố:** Sentry (báo lỗi code real-time), PagerDuty (gọi điện cảnh báo cho kỹ sư trực khi hệ thống gặp sự cố).

---

### Giai đoạn 5: Users+++++ (Trên 10 triệu người dùng, 100 tỷ lượt đọc/tháng, 40.000 RPS)

<p align="center">
  <img src="http://i.imgur.com/jj3A5N8.png">
  <br/>
</p>

#### Bối cảnh:
Hệ thống đạt đến quy mô khổng lồ: 
* $40.000$ lượt đọc mỗi giây.
* $400$ lượt ghi mỗi giây (đỉnh điểm có thể lên tới hàng nghìn ghi/giây).
* $1\text{ TB}$ dữ liệu mới mỗi tháng.
Lúc này, một cụm Master-Slave MySQL đơn thuần không thể chịu nổi nữa.

#### Giải pháp toàn diện:

1. **Lưu trữ dữ liệu lịch sử vào Kho dữ liệu (Data Warehouse):**
   * Database chính (MySQL) chỉ lưu dữ liệu gần đây (ví dụ trong vòng 3-6 tháng).
   * Dữ liệu cũ hơn được đẩy sang Data Warehouse như **Amazon Redshift** hoặc Snowflake để phục vụ phân tích báo cáo (dễ dàng chứa hàng chục đến hàng trăm Terabyte dữ liệu).
2. **Các kỹ thuật mở rộng SQL chuyên sâu:**
   * **Federation (Phân vùng theo chức năng):** Tách riêng Database Người dùng, Database Đơn hàng, Database Sản phẩm.
   * **Sharding (Phân mảnh dữ liệu):** Chia nhỏ dữ liệu người dùng ra nhiều cụm máy chủ DB khác nhau (dùng thuật toán Consistent Hashing).
   * **Denormalization (Phi chuẩn hóa):** Chấp nhận lưu trùng lặp một số trường dữ liệu để tránh lệnh `JOIN` phức tạp.
3. **Chuyển các tập dữ liệu phù hợp sang NoSQL (DynamoDB / Cassandra):**
   * Những dữ liệu có khối lượng ghi cực lớn, không cần ràng buộc quan hệ phức tạp (như log hành vi, giỏ hàng, bảng xếp hạng điểm, dữ liệu tạm) được chuyển sang **DynamoDB** để đạt tốc độ ghi gần như không giới hạn.
4. **Xử lý bất đồng bộ với Message Queue & Background Workers:**
   * Đưa các tác vụ nặng ra khỏi luồng xử lý chính bằng hàng đợi thông điệp:
     * Ví dụ khi người dùng tải ảnh lên: Client gửi ảnh lên $\rightarrow$ App Server đẩy thông điệp vào **Amazon SQS** rồi báo thành công ngay cho user $\rightarrow$ Các **Worker chạy ngầm (EC2 hoặc AWS Lambda)** sẽ lấy việc từ SQS ra để nén ảnh, tạo ảnh thumbnail, lưu vào S3 và cập nhật Database.
   * Giúp hệ thống không bao giờ bị nghẽn (choke) khi có đột biến tải đột ngột.

---

## Tóm tắt các chủ đề cần nhớ

1. **Mở rộng hệ thống là một quy trình lặp:** Luôn bắt đầu từ Benchmark $\rightarrow$ Tìm điểm nghẽn $\rightarrow$ Tối ưu điểm nghẽn $\rightarrow$ Tiếp tục đo kiểm.
2. **Đừng tối ưu hóa quá sớm (Avoid premature optimization):** Ở giai đoạn đầu (vài người dùng), một máy chủ duy nhất là đủ. Chỉ mở rộng khi dữ liệu thực tế cho thấy hệ thống cần mở rộng.
3. **Nguyên tắc phi trạng thái (Stateless):** Giữ cho tầng Web Server không lưu trữ trạng thái người dùng (đẩy session vào Redis) là chìa khóa để có thể scale ngang vô hạn.
4. **Bộ nhớ đệm (Cache) và Hàng đợi (Queue):** Là hai vũ khí mạnh nhất để giải quyết bài toán tải cao (Cache để giảm tải đọc, Queue để san phẳng đỉnh tải ghi).

