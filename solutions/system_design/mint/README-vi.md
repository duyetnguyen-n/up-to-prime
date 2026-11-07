*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Hệ Thống Quản Lý Tài Chính Cá Nhân (Mint.com)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Người dùng (User)** liên kết tài khoản tài chính / ngân hàng.
* **Hệ thống (Service)** trích xuất các giao dịch (Transactions) từ tài khoản:
  * Tự động cập nhật giao dịch hàng ngày.
  * Tự động phân loại danh mục chi tiêu (Categorization) cho từng giao dịch:
    * Cho phép người dùng tự điều chỉnh lại danh mục thủ công (Manual override).
    * Không tự động phân loại lại các giao dịch cũ.
  * Phân tích tổng hợp chi tiêu hàng tháng theo từng danh mục.
* **Hệ thống** đề xuất hạn mức chi tiêu (Budget):
  * Cho phép người dùng tự đặt ngân sách chi tiêu.
  * Gửi thông báo cảnh báo khi chi tiêu sắp chạm ngưỡng hoặc vượt quá ngân sách.
* **Hệ thống** phải đảm bảo tính sẵn sàng cao (High Availability).

#### Nằm ngoài phạm vi bài toán (Out of Scope):
* Hệ thống ghi log và phân tích kinh doanh mở rộng.

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng truy cập không phân bổ đều theo thời gian.
* Tự động đồng bộ giao dịch hàng ngày chỉ áp dụng cho những người dùng có hoạt động trong vòng 30 ngày gần nhất (Active Users).
* Thao tác thêm hoặc xóa tài khoản ngân hàng diễn ra rất ít.
* Thông báo ngân sách không nhất thiết phải gửi theo thời gian thực (Real-time).
* $10$ triệu người dùng:
  * Mỗi người dùng có khoảng 10 danh mục chi tiêu (Nhà ở: \$1.000, Ăn uống: \$200, Xăng xe: \$100...) $\rightarrow$ Tổng cộng $100$ triệu hạn mức ngân sách.
  * Phân loại giao dịch dựa trên tên người bán (Seller/Merchant): Có khoảng $50.000$ bên bán phổ biến.
* $30$ triệu tài khoản tài chính được liên kết.
* $5$ tỷ giao dịch phát sinh mỗi tháng.
* $500$ triệu lượt truy vấn đọc dữ liệu mỗi tháng.
* **Tỉ lệ Ghi / Đọc: 10:1** (Hệ thống thiên về Ghi - Write-heavy, vì giao dịch phát sinh liên tục mỗi ngày trong khi người dùng chỉ thỉnh thoảng mới vào app xem).

#### Tính toán ước lượng sơ bộ (Back-of-the-envelope calculations):
* **Kích thước mỗi bản ghi giao dịch:**
  * `user_id`: 8 bytes
  * `created_at`: 5 bytes
  * `seller`: 32 bytes
  * `amount`: 5 bytes
  * **Tổng cộng:** $\approx 50\text{ bytes}$ / giao dịch.
* **Dung lượng lưu trữ:**
  * $50\text{ bytes} \times 5\text{ tỷ transactions/tháng} = 250\text{ GB}$ dữ liệu mới mỗi tháng.
  * Sau 3 năm: $\approx 9\text{ TB}$ dữ liệu giao dịch mới.
* **Tốc độ xử lý trung bình mỗi giây:**
  * Ghi giao dịch: $\approx 2.000\text{ transactions/giây}$.
  * Truy vấn đọc: $\approx 200\text{ reads/giây}$.

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/E8klrBh.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Use case 1: Người dùng liên kết tài khoản tài chính

1. **Client** gửi thông tin đăng nhập ngân hàng tới **Web Server** (Reverse Proxy).
2. **Web Server** chuyển tiếp tới máy chủ **Accounts API**.
3. **Accounts API** mã hóa thông tin xác thực và lưu vào bảng `accounts` trong **SQL Database**:
   ```sql
   CREATE TABLE accounts (
       id INT NOT NULL AUTO_INCREMENT,
       created_at DATETIME NOT NULL,
       last_update DATETIME NOT NULL,
       account_url VARCHAR(255) NOT NULL,
       account_login VARCHAR(32) NOT NULL,
       account_password_hash CHAR(64) NOT NULL,
       user_id INT NOT NULL,
       PRIMARY KEY (id),
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   CREATE INDEX idx_accounts_user ON accounts(user_id, created_at);
   ```

---

### Use case 2: Trích xuất và phân loại giao dịch

Việc kết nối tới API ngân hàng đối tác để cào giao dịch có thể mất nhiều thời gian và dễ bị timeout $\rightarrow$ Bắt buộc phải xử lý **Bất đồng bộ qua Hàng đợi (Message Queue)**.

#### Luồng dữ liệu:
1. Khi user bấm đồng bộ hoặc đến lịch chạy ngầm hàng đêm: **Accounts API** đẩy công việc vào **Message Queue (Amazon SQS / RabbitMQ)**.
2. **Transaction Extraction Service (Worker)** lấy job từ Queue:
   * Kết nối tới ngân hàng để lấy danh sách giao dịch mới, lưu log thô vào **Object Store (S3)**.
   * Gọi **Category Service** để tự động gán danh mục cho từng giao dịch (dựa trên tên Seller).
   * Gọi **Budget Service** để tính tổng chi tiêu lũy kế trong tháng và so sánh với ngân sách.
     * Nếu chi tiêu vượt ngưỡng: Gửi tín hiệu sang **Notification Service** để gửi email/thông báo cho user.
   * Ghi các giao dịch đã phân loại vào bảng `transactions` trong **SQL Database**.
   * Cập nhật số liệu tổng hợp vào bảng `monthly_spending`.

#### Thuật toán phân loại danh mục (Category Service):
* Duy trì một bảng tra cứu (Dictionary / Hash Map) chứa $50.000$ Seller phổ biến nhất (chỉ tốn khoảng $12\text{ MB}$ RAM):
  * `Exxon` $\rightarrow$ `GAS (Xăng xe)`
  * `Target / Walmart` $\rightarrow$ `SHOPPING (Mua sắm)`
* **Học từ cộng đồng (Crowdsourcing):** Nếu gặp Seller lạ chưa có trong từ điển, hệ thống sẽ kiểm tra xem người dùng khác có từng sửa tay (manual override) Seller này sang danh mục nào nhiều nhất hay không (dùng cấu trúc Heap / Min-Heap để tìm lựa chọn phổ biến nhất trong thời gian $O(1)$).

---

### Use case 3: Đề xuất và theo dõi ngân sách (Budget Service)

* Áp dụng quy tắc quản lý tài chính chuẩn (ví dụ quy tắc 50/30/20 hoặc theo mức thu nhập) để tự tạo mẫu ngân sách gợi ý:
  * Nhà ở: 40% thu nhập
  * Ăn uống: 20% thu nhập
  * Xăng xe: 10% thu nhập
  * Mua sắm: 20% thu nhập
* Nếu người dùng chỉnh sửa ngân sách cho danh mục nào, ta lưu bản ghi đè vào bảng `budget_overrides`.
* **Tính toán số liệu chi tiêu tháng:** Thay vì chạy câu lệnh `SUM()` trực tiếp trên bảng 5 tỷ giao dịch (gây nghẽn Database), ta chạy các job **MapReduce** định kỳ trên file log giao dịch để tính toán tổng chi tiêu theo tháng cho từng user.

---

## Bước 4: Mở rộng thiết kế (Scale the Design & Xử lý Điểm nghẽn)

<p align="center">
  <img src="http://i.imgur.com/V5q57vU.png">
  <br/>
</p>

### Các giải pháp mở rộng:

1. **Bộ nhớ đệm (Memory Cache - Redis):**
   * Đặt **Redis** trước Database theo mô hình **Cache-Aside**.
   * Cache session người dùng, bảng phân loại người bán (Seller Map), số liệu tổng kết chi tiêu tháng và các giao dịch gần nhất.
2. **Tách riêng Database phân tích (Data Warehouse):**
   * Thay vì lưu toàn bộ lịch sử giao dịch trong CSDL giao dịch chính (SQL), ta chỉ giữ lại giao dịch của 1-3 tháng gần nhất.
   * Toàn bộ lịch sử giao dịch cũ hơn được đẩy vào **Kho dữ liệu (Amazon Redshift / Google BigQuery)** để phục vụ truy vấn phân tích dài hạn.
3. **Mở rộng cho tải ghi cao (2.000 writes/giây):**
   * Sử dụng **Database Sharding** phân mảnh bảng `transactions` theo `user_id` để chia đều tải ghi cho nhiều cụm server Database.
   * Cân nhắc chuyển các giao dịch sang **NoSQL Database (như DynamoDB)** để đạt thông lượng ghi cao mà không bị nghẽn khóa bảng.
