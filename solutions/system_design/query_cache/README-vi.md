*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Key-Value Cache Lưu Kết Quả Tìm Kiếm (Query Cache)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Người dùng (User)** gửi request tìm kiếm và gặp **Cache Hit** (dữ liệu đã có sẵn trong bộ nhớ đệm).
* **Người dùng** gửi request tìm kiếm và gặp **Cache Miss** (dữ liệu chưa có, cần truy vấn hệ thống gốc rồi lưu vào cache).
* **Hệ thống (Service)** phải đảm bảo tính sẵn sàng cao (High Availability).

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng truy cập không phân bổ đều (các truy vấn phổ biến phải hầu như luôn nằm trong Cache).
* Cần xác định chiến lược hết hạn và làm mới dữ liệu trong Cache (Expiration / Refresh).
* Phục vụ từ Cache đòi hỏi tốc độ tìm kiếm cực nhanh ($O(1)$).
* Dung lượng bộ nhớ RAM trong Cache là có hạn $\rightarrow$ Cần cơ chế dọn dẹp các bản ghi cũ khi đầy bộ nhớ (Eviction Policy).
* $10$ triệu người dùng.
* $10$ tỷ lượt truy vấn tìm kiếm mỗi tháng.
* $\approx 4.000\text{ search requests/giây}$.

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/KqZ3dSx.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Giải thuật thay thế Cache: LRU (Least Recently Used)

Vì dung lượng RAM có hạn, chúng ta áp dụng thuật toán **LRU (Giải phóng bản ghi ít được dùng nhất gần đây)**.

#### Cấu trúc dữ liệu xây dựng LRU Cache:
Để đạt độ phức tạp thời gian **$O(1)$ cho cả thao tác Đọc (`get`) và Ghi (`set`)**:
* **Bảng băm (Hash Map / Hash Table):** Cho phép tra cứu nhanh một từ khóa tìm kiếm (`query`) tới Node tương ứng trong bộ nhớ với thời gian $O(1)$.
* **Danh sách liên kết kép (Doubly-Linked List):**
  * Phần tử mới truy cập sẽ được đẩy lên đầu danh sách (**Head**).
  * Phần tử lâu nhất không ai đụng tới sẽ bị đẩy dần về cuối danh sách (**Tail**).
  * Khi Cache đầy bộ nhớ (`MAX_SIZE`), ta chỉ việc xóa phần tử ở đuôi (`tail`) trong thời gian $O(1)$.

```
[Head - Mới nhất] <---> [Node 1] <---> [Node 2] <---> [Tail - Cũ nhất, sẽ bị xóa khi đầy]
```

#### Luồng xử lý khi nhận request (Cache-Aside Pattern):
1. **Client** gửi truy vấn tìm kiếm tới **Query API**.
2. **Query API** chuẩn hóa câu truy vấn (bỏ khoảng trắng, sửa lỗi chính tả) và kiểm tra trong **Memory Cache**:
   * **Nếu Cache Hit:** Cập nhật vị trí của bản ghi đó lên đầu danh sách LRU và trả kết quả ngay cho client.
   * **Nếu Cache Miss:** Gọi tới cụm **Reverse Index Service & Document Service** để lấy kết quả $\rightarrow$ Ghi kết quả mới vào đầu danh sách Cache (nếu Cache đầy thì xóa bản ghi ở đuôi Tail) $\rightarrow$ Trả kết quả cho client.

---

## Bước 4: Mở rộng thiết kế (Scale the Design)

<p align="center">
  <img src="http://i.imgur.com/4j99mhe.png">
  <br/>
</p>

### Mở rộng Cache ra nhiều máy chủ (Distributed Cache Cluster):

Với hàng tỷ truy vấn, dữ liệu cache không thể chứa vừa trên 1 server RAM duy nhất. Có 3 phương án mở rộng cụm Cache:
1. **Mỗi máy chủ cache độc lập:** Dễ triển khai nhưng tỉ lệ Cache Hit rất thấp (người dùng hỏi máy A rồi lần sau lại hỏi máy B).
2. **Mỗi máy lưu một bản sao giống hệt nhau (Replication):** Lãng phí RAM nghiêm trọng và tốn tài nguyên đồng bộ.
3. **Phân mảnh Cache (Cache Sharding với Consistent Hashing - Giải pháp tối ưu):**
   * Dùng giải thuật **Consistent Hashing** để xác định một truy vấn sẽ do máy chủ Cache nào chịu trách nhiệm lưu trữ:
     $$\text{Server} = \text{Hash}(\text{Query}) \pmod N$$
   * Khi thêm hoặc bớt một máy chủ trong cụm, thuật toán Consistent Hashing chỉ làm di chuyển một lượng rất nhỏ dữ liệu, tránh làm sập toàn bộ hệ thống Cache (Cache Stampede).
