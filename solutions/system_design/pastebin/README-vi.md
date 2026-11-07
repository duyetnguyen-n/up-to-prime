*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Dịch vụ Pastebin.com (hoặc Bit.ly - Dịch vụ rút gọn link)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

**Thiết kế Bit.ly:** Là một bài toán hoàn toàn tương tự, chỉ khác là Pastebin yêu cầu lưu trữ nội dung văn bản (paste content), còn Bit.ly lưu trữ URL gốc dài ban đầu.

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Người dùng (User)** nhập vào một đoạn văn bản và nhận về một đường link ngẫu nhiên được sinh ra (`shortlink`):
  * **Hết hạn (Expiration):**
    * Mặc định: Không bao giờ hết hạn.
    * Tùy chọn: Có thể thiết lập thời gian tự động hết hạn (ví dụ sau 1 giờ, 1 ngày, 1 tháng).
* **Người dùng** truy cập vào đường link rút gọn và xem được nội dung văn bản gốc.
* **Người dùng** hoàn toàn ẩn danh (Anonymous - không cần đăng nhập).
* **Hệ thống (Service)** theo dõi thống kê số liệu truy cập (Analytics):
  * Thống kê lượt truy cập theo từng tháng.
* **Hệ thống** tự động xóa các nội dung đã hết hạn.
* **Hệ thống** phải đảm bảo tính sẵn sàng cao (High Availability).

#### Nằm ngoài phạm vi bài toán (Out of Scope):
* Đăng ký tài khoản người dùng, xác thực email, đăng nhập.
* Chỉnh sửa lại nội dung đã paste.
* Phân quyền xem (công khai / riêng tư).
* Tự đặt link theo ý thích (Custom shortlink).

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng truy cập không phân bổ đều theo thời gian.
* Tốc độ mở link rút gọn phải cực nhanh (độ trễ thấp).
* Nội dung paste chỉ là văn bản thuần túy (Plain text).
* Phân tích số liệu truy cập (Analytics) không cần phải là thời gian thực (Real-time).
* $10$ triệu người dùng.
* $10$ triệu lượt tạo paste mới mỗi tháng (Writes).
* $100$ triệu lượt truy cập xem paste mỗi tháng (Reads).
* Tỉ lệ Đọc / Ghi: **10:1**.

#### Tính toán ước lượng sơ bộ (Back-of-the-envelope calculations):
* **Kích thước trung bình mỗi bản ghi:**
  * Nội dung paste: $1\text{ KB}$.
  * `shortlink`: $7\text{ bytes}$.
  * `expiration_length_in_minutes`: $4\text{ bytes}$.
  * `created_at`: $5\text{ bytes}$.
  * `paste_path` (đường dẫn lưu file): $255\text{ bytes}$.
  * **Tổng cộng:** $\approx 1.27\text{ KB}$ / bản ghi.
* **Dung lượng lưu trữ:**
  * $1.27\text{ KB} \times 10\text{ triệu pastes} \approx 12.7\text{ GB}$ dữ liệu mới mỗi tháng.
  * Trong 3 năm: $\approx 450\text{ GB}$ dữ liệu mới.
  * Tổng cộng sinh ra $360$ triệu đường link rút gọn sau 3 năm.
* **Tốc độ xử lý trung bình mỗi giây:**
  * Ghi trung bình: $\approx 4\text{ writes/giây}$.
  * Đọc trung bình: $\approx 40\text{ reads/giây}$.

*(Quy đổi: 1 tháng $\approx$ 2.5 triệu giây $\rightarrow$ 4 writes/giây $\approx$ 10 triệu writes/tháng).*

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/BKsBnmG.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Use case 1: Người dùng nhập văn bản và nhận link rút gọn

Chúng ta có thể dùng một **Cơ sở dữ liệu quan hệ (RDBMS)** như một bảng băm khổng lồ, ánh xạ chuỗi link rút gọn (`shortlink`) tới đường dẫn file trên máy chủ lưu trữ.

Thay vì tự quản lý máy chủ file, chúng ta nên dùng một **Object Store (AWS S3)** hoặc **NoSQL Document Store** để lưu nội dung file văn bản.

#### Luồng dữ liệu (Data Flow):
1. **Client** gửi request tạo paste tới **Web Server** (đóng vai trò Reverse Proxy).
2. **Web Server** chuyển tiếp request đến máy chủ **Write API**.
3. Máy chủ **Write API** thực hiện các bước sau:
   * Sinh ra một chuỗi URL định danh duy nhất (`shortlink`).
   * Kiểm tra trong **SQL Database** xem chuỗi này đã bị trùng lặp hay chưa (Duplicate check).
   * Lưu metadata vào bảng `pastes` trong **SQL Database**.
   * Lưu nội dung file văn bản thực tế vào **Object Store (S3)**.
   * Trả URL rút gọn về cho Client.

#### Thiết kế Database Schema (Bảng `pastes`):
```sql
CREATE TABLE pastes (
    shortlink VARCHAR(7) NOT NULL,
    expiration_length_in_minutes INT NOT NULL,
    created_at DATETIME NOT NULL,
    paste_path VARCHAR(255) NOT NULL,
    PRIMARY KEY (shortlink)
);
CREATE INDEX idx_created_at ON pastes(created_at);
```
* Cột `shortlink` làm Khóa chính (`PRIMARY KEY`) sẽ tự động được đánh Index để đảm bảo tính duy nhất và truy vấn $O(1)$.
* Đánh thêm Index trên cột `created_at` để tối ưu hóa việc tìm kiếm và xóa các bản ghi hết hạn.

#### Thuật toán sinh chuỗi URL rút gọn độc nhất:
1. Lấy mã băm **MD5** của: `ip_address + timestamp`.
   * MD5 sinh ra giá trị băm 128-bit, phân bổ rất đồng đều.
2. Mã hóa chuỗi băm đó bằng **Base 62**:
   * Base 62 sử dụng bảng chữ cái gồm 62 ký tự: `[a-zA-Z0-9]`, rất an toàn cho URL trên trình duyệt (không bị dính ký tự đặc biệt như `+` hay `/` của Base 64).
   * Pseudocode thuật toán Base 62 (chạy trong thời gian $O(k)$ với $k=7$):
     ```python
     def base_encode(num, base=62):
         digits = []
         while num > 0:
             remainder = num % base
             digits.append(remainder)
             num = num // base
         return digits[::-1]
     ```
3. **Lấy 7 ký tự đầu tiên** của kết quả:
   $$62^7 \approx 3.5\text{ nghìn tỷ khả năng}$$
   Con số này hoàn toàn dư sức đáp ứng yêu cầu 360 triệu liên kết sau 3 năm mà hầu như không sợ cạn kiệt.

#### Thiết kế Public REST API:
* **Endpoint:** `POST https://pastebin.com/api/v1/paste`
* **Request Body:**
  ```json
  {
    "expiration_length_in_minutes": 60,
    "paste_contents": "Hello World! Đây là văn bản mẫu."
  }
  ```
* **Response (HTTP 200/201):**
  ```json
  {
    "shortlink": "aB3dE7x"
  }
  ```

---

### Use case 2: Người dùng nhập URL rút gọn để xem nội dung

1. **Client** gửi request xem paste tới **Web Server**.
2. **Web Server** chuyển tiếp request đến máy chủ **Read API**.
3. Máy chủ **Read API** thực hiện:
   * Tìm kiếm mã `shortlink` trong **SQL Database**.
   * Nếu thấy: Lấy đường dẫn file `paste_path` và kéo nội dung từ **Object Store (S3)** về.
   * Nếu không thấy hoặc đã hết hạn: Trả về lỗi 404 Not Found.

#### Thiết kế REST API:
* **Endpoint:** `GET https://pastebin.com/api/v1/paste?shortlink=aB3dE7x`
* **Response (HTTP 200):**
  ```json
  {
    "paste_contents": "Hello World! Đây là văn bản mẫu.",
    "created_at": "2026-11-07 20:00:00",
    "expiration_length_in_minutes": 60
  }
  ```

---

### Use case 3: Thống kê số liệu truy cập (Analytics)

Vì không cần hiển thị số liệu theo thời gian thực (Real-time), chúng ta có thể áp dụng mô hình **MapReduce** (chạy bằng Hadoop / Spark định kỳ hàng đêm) để quét qua toàn bộ file Web Server Access Logs và tổng hợp số lượt truy cập theo tháng cho từng URL.

---

### Use case 4: Xóa các nội dung đã hết hạn

Một tiến trình chạy nền (Cron Job / Background Worker) sẽ định kỳ quét **SQL Database** để tìm các bản ghi có:
$$\text{created\_at} + \text{expiration\_length\_in\_minutes} < \text{current\_timestamp}$$
Sau đó tiến hành xóa metadata trong Database và xóa file tương ứng trên S3 (hoặc áp dụng cơ chế tự hủy file **S3 Lifecycle Rules**).

---

## Bước 4: Mở rộng thiết kế (Scale the Design)

<p align="center">
  <img src="http://i.imgur.com/4edXG0T.png">
  <br/>
</p>

> **Lưu ý quan trọng:** Trong phỏng vấn, đừng bao giờ nhảy ngay vào sơ đồ hoàn chỉnh cuối cùng! Hãy luôn trình bày theo từng bước lặp: Đo kiểm $\rightarrow$ Phát hiện điểm nghẽn $\rightarrow$ Đưa ra giải pháp và phân tích đánh đổi (Trade-offs).

### Các điểm nghẽn và giải pháp xử lý:

1. **Tầng phân phối nội dung tĩnh (CDN):**
   * Đặt **CDN (CloudFront)** trước Object Store S3 để người dùng trên toàn cầu tải nội dung paste nhanh nhất.
2. **Cân bằng tải & Mở rộng ngang (Load Balancer & Stateless Web Servers):**
   * Thêm **Load Balancer (ELB)** chia đều tải cho cụm Web Server.
   * Tách riêng cụm **Read API Servers** và **Write API Servers** để có thể scale độc lập.
3. **Tầng bộ nhớ đệm (Memory Cache - Redis):**
   * Xử lý 40 reads/giây (và cao điểm hàng trăm/nghìn reads/giây) bằng cách đặt **Redis** trước Database theo mô hình **Cache-Aside**.
   * Các bài paste "hot" (được chia sẻ nhiều) sẽ được phục vụ trực tiếp từ RAM, không chạm vào Database.
4. **Cơ sở dữ liệu (Database Replication):**
   * Cấu hình mô hình **SQL Write Master - Read Slaves**: Ghi vào Master, đọc từ các Slave.
5. **Mở rộng khi dữ liệu phình to:**
   * Lưu trữ phân tích số liệu (Analytics) trong **Data Warehouse** như Amazon Redshift hoặc BigQuery.
   * Nếu sau này lượng ghi tăng cao đột biến, có thể áp dụng thêm **Consistent Hashing** để Sharding Database.

