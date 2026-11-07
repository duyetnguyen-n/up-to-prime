*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Cấu Trúc Dữ Liệu Cho Mạng Xã Hội (Social Graph)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Người dùng (User)** tìm kiếm một người khác và nhìn thấy **đường đi ngắn nhất** kết nối giữa 2 người (ví dụ: Bạn $\rightarrow$ Người A $\rightarrow$ Người B $\rightarrow$ Người cần tìm - Mối quan hệ qua mấy bậc bạn bè / Degree of Separation).
* **Hệ thống (Service)** phải đảm bảo tính sẵn sàng cao (High Availability).

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng tìm kiếm không phân bổ đều theo thời gian.
* **Dữ liệu đồ thị quan hệ bạn bè không thể nhét vừa trên một máy chủ duy nhất.**
* Các cạnh trong đồ thị là không trọng số (Unweighted edges - chỉ quan tâm có là bạn bè hay không).
* $100$ triệu người dùng (Vertices / Nodes).
* Trung bình mỗi người dùng có 50 bạn bè.
* $\rightarrow$ Khoảng **$5$ tỷ mối quan hệ bạn bè** (Edges).
* $1$ tỷ lượt tìm kiếm bạn bè mỗi tháng.
* $\approx 400\text{ search requests/giây}$.

*(Lưu ý của bài toán: Yêu cầu áp dụng các hệ thống phân tán truyền thống thay vì dùng thẳng các CSDL đồ thị chuyên dụng như Neo4j).*

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/wxXyq2J.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Thuật toán tìm đường đi ngắn nhất (BFS):

Nếu toàn bộ dữ liệu nằm trên 1 máy, chúng ta chỉ cần chạy thuật toán **Tìm kiếm theo chiều rộng (BFS - Breadth-First Search)**:
* Bắt đầu từ node gốc (Source) $\rightarrow$ Duyệt qua tất cả bạn bè bậc 1 $\rightarrow$ Duyệt tiếp bạn của bạn (bậc 2)... cho tới khi gặp node đích (Destination).

Tuy nhiên, với $100$ triệu người và $5$ tỷ quan hệ, dữ liệu bắt buộc phải được **Sharding (phân mảnh)** ra nhiều cụm máy chủ **Person Servers**.

#### Kiến trúc phân tán:
1. **Lookup Service:** Đóng vai trò danh bạ, cho biết thông tin của `person_id` cụ thể đang nằm ở máy chủ **Person Server** nào.
2. **User Graph Service:** Nhận lệnh tìm kiếm từ API, phối hợp với Lookup Service để điều phối quá trình BFS phân tán qua mạng.
3. Khi duyệt từ người A sang người B, nếu người B nằm ở server khác, hệ thống phải thực hiện gọi mạng (RPC call) sang server đó.

---

## Bước 4: Mở rộng thiết kế (Scale the Design & Tối ưu hóa)

<p align="center">
  <img src="http://i.imgur.com/cdCv5g7.png">
  <br/>
</p>

### Các kỹ thuật tối ưu hóa then chốt cho Social Graph:

1. **Tìm kiếm 2 chiều đồng thời (Bidirectional BFS):**
   * Thay vì chỉ chạy BFS 1 chiều từ Người A $\rightarrow$ Người B (số lượng node tăng theo cấp số nhân theo từng bậc), ta **chạy 2 luồng BFS cùng lúc**:
     * Một luồng chạy xuôi từ Người A.
     * Một luồng chạy ngược từ Người B.
     * Khi hai luồng chạm nhau ở một node trung gian $\rightarrow$ Ghép 2 nửa đường đi lại.
   * **Hiệu quả:** Giảm độ phức tạp từ $O(b^d)$ xuống còn $O(b^{d/2})$, nhanh hơn hàng nghìn lần!
2. **Bộ nhớ đệm đường đi (Memory Cache - Redis):**
   * Lưu trữ các đường đi BFS đã từng tính toán vào Redis Cache để các lần tìm kiếm sau có kết quả ngay lập tức $O(1)$.
3. **Sharding theo Vị trí Địa lý (Location-based Sharding):**
   * Trong thế giới thực, bạn bè thường sống gần nhau hoặc cùng thành phố/quốc gia.
   * Phân vùng dữ liệu máy chủ theo khu vực địa lý giúp đa số các truy vấn kết bạn nằm gọn trong cùng một Data Center / cụm server, giảm thiểu tối đa các bước nhảy mạng (Network hops) giữa các máy chủ.
4. **Ưu tiên duyệt qua các "Node siêu kết nối" (Well-connected nodes):**
   * Ưu tiên mở rộng qua các tài khoản có lượng bạn bè lớn, vì họ đóng vai trò là "cầu nối" giúp rút ngắn số bậc cách biệt (Degrees of separation) cực kỳ nhanh chóng.
