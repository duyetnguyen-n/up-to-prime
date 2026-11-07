*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Tính Năng Xếp Hạng Bán Chạy Theo Danh Mục Của Amazon (Sales Rank)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Hệ thống (Service)** tính toán danh sách các sản phẩm bán chạy nhất trong tuần qua theo từng danh mục hàng hóa (Category).
* **Người dùng (User)** xem bảng xếp hạng sản phẩm bán chạy nhất trong tuần theo danh mục.
* **Hệ thống** phải đảm bảo tính sẵn sàng cao (High Availability).

#### Nằm ngoài phạm vi bài toán (Out of Scope):
* Toàn bộ website thương mại điện tử tổng thể (chỉ tập trung thiết kế tính năng tính toán bảng xếp hạng doanh số Sales Rank).

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng truy cập không phân bổ đều.
* Một sản phẩm có thể thuộc nhiều danh mục khác nhau.
* Bảng xếp hạng phải được **cập nhật định kỳ mỗi giờ một lần** (Hourly updates).
* $10$ triệu sản phẩm.
* $1.000$ danh mục hàng hóa.
* $1$ tỷ giao dịch mua hàng mỗi tháng.
* $100$ tỷ lượt xem bảng xếp hạng mỗi tháng.
* **Tỉ lệ Đọc / Ghi: 100:1** (Hệ thống thiên về Đọc rất lớn).
* $\approx 400\text{ transactions/giây}$ và $\approx 40.000\text{ reads/giây}$.

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/vwMa1Qu.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Xử lý dữ liệu lớn với MapReduce 2 bước (MapReduce Multi-step):

Mỗi khi người dùng mua hàng, log giao dịch thô được ghi nhận và đẩy lên **Object Store (AWS S3)**:
```
timestamp   product_id  category_id    qty     total_price   seller_id    buyer_id
```

Để tính toán bảng xếp hạng hàng giờ cho 1 tỷ giao dịch mà không làm quá tải Database, hệ thống chạy **MapReduce**:

* **Bước 1 — Tính tổng số lượng bán:**
  * **Mapper:** Đọc log giao dịch trong 7 ngày qua, trích xuất cặp key-value: `((category_id, product_id), quantity)`.
  * **Reducer:** Gom nhóm theo `(category_id, product_id)` và tính tổng số lượng bán: `((category_id, product_id), sum(quantity))`.
* **Bước 2 — Sắp xếp phân tán (Distributed Sort):**
  * Đảo ngược cấu trúc thành: `((category_id, total_sold), product_id)`.
  * Cơ chế Shuffle & Sort của MapReduce sẽ tự động sắp xếp theo thứ tự giảm dần của số lượng bán trong từng danh mục.
* Kết quả sau đó được nạp vào bảng `sales_rank` trong **SQL Database**:
  ```sql
  CREATE TABLE sales_rank (
      id INT NOT NULL AUTO_INCREMENT,
      category_id INT NOT NULL,
      total_sold INT NOT NULL,
      product_id INT NOT NULL,
      PRIMARY KEY (id)
  );
  CREATE INDEX idx_sales_rank ON sales_rank(category_id, total_sold DESC);
  ```

---

## Bước 4: Mở rộng thiết kế (Scale the Design)

<p align="center">
  <img src="http://i.imgur.com/MzExP06.png">
  <br/>
</p>

### Các giải pháp mở rộng:
1. **Bộ nhớ đệm (Memory Cache - Redis):**
   * Với $40.000$ lượt đọc mỗi giây, kết quả bảng xếp hạng của 1.000 danh mục được lưu trực tiếp trên **Redis Cache** theo dạng `sales_rank:category_id` $\rightarrow$ Trả về kết quả trong $1-2\text{ ms}$.
   * Vì bảng xếp hạng chỉ cập nhật 1 giờ/lần, TTL của Cache được đặt là 1 giờ.
2. **Cơ sở dữ liệu (Master - Read Replicas):**
   * Nếu có Cache Miss, các lượt đọc được chuyển sang cụm **SQL Read Replicas** để giảm tải cho máy chủ chính.
