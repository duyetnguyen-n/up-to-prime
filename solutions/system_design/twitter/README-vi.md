*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Mạng Xã Hội Twitter: Dòng Thời Gian (Timeline) & Tìm Kiếm (Search)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

**Thiết kế Bảng tin Facebook (Facebook News Feed) & Facebook Search** là những câu hỏi tương tự với cùng bản chất kiến trúc.

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Người dùng (User)** đăng một tweet (bài viết):
  * **Hệ thống (Service)** đẩy tweet tới những người theo dõi (Followers), đồng thời gửi Push Notifications và Email.
* **Người dùng** xem Dòng thời gian cá nhân (**User Timeline** - toàn bộ bài viết do chính user đó đăng).
* **Người dùng** xem Bảng tin trang chủ (**Home Timeline** - bài viết tổng hợp từ những người mà user đang theo dõi).
* **Người dùng** tìm kiếm từ khóa (**Search Keywords**).
* **Hệ thống** phải đảm bảo tính sẵn sàng cao (High Availability).

#### Nằm ngoài phạm vi bài toán (Out of Scope):
* Đẩy tweet vào Twitter Firehose hoặc các luồng dữ liệu đối tác thứ ba.
* Bộ lọc ẩn bài viết nâng cao (ẩn reply nếu không follow cả hai bên, ẩn retweet).
* Hệ thống phân tích số liệu thống kê (Analytics).

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng truy cập không phân bổ đều theo thời gian.
* Đăng tweet phải diễn ra cực nhanh.
* Việc phân phối tweet tới tất cả người theo dõi (**Fanout**) phải diễn ra nhanh chóng (trừ trường hợp tài khoản có hàng triệu followers - người nổi tiếng).
* $100$ triệu người dùng hoạt động hàng ngày (Active Users).
* $500$ triệu tweet mỗi ngày $\approx 15$ tỷ tweet mỗi tháng.
  * Trung bình mỗi tweet được phân phối tới 10 người theo dõi (Fanout factor = 10).
  * $5$ tỷ tweet được phân phối (Fanout deliveries) mỗi ngày $\approx 150$ tỷ tweet/tháng.
* $250$ tỷ lượt xem timeline mỗi tháng (Đọc - Reads).
* $10$ tỷ lượt tìm kiếm mỗi tháng (Search).
* **Tỉ lệ đọc/ghi:** Hệ thống thiên về Đọc rất lớn (Read-heavy) $\rightarrow$ Cần tối ưu tối đa cho tốc độ Đọc tweet.

#### Tính toán ước lượng sơ bộ (Back-of-the-envelope calculations):
* **Kích thước mỗi tweet:**
  * `tweet_id`: 8 bytes
  * `user_id`: 32 bytes
  * `text`: 140 bytes
  * `media` (ảnh/video đính kèm): Trung bình 10 KB
  * **Tổng cộng:** $\approx 10\text{ KB}$ / tweet.
* **Lưu lượng nội dung mới:**
  * $10\text{ KB} \times 500\text{ triệu tweets/ngày} \times 30\text{ ngày} = 150\text{ TB}$ dữ liệu mới mỗi tháng.
  * Trong 3 năm: $\approx 5.4\text{ PB}$ (Petabytes) dữ liệu tweet mới.
* **Tốc độ xử lý trung bình mỗi giây (RPS):**
  * Đọc Timeline: $\approx 100.000\text{ reads/giây}$.
  * Đăng Tweet: $\approx 6.000\text{ tweets/giây}$.
  * Phân phối bài viết (Fanout): $\approx 60.000\text{ deliveries/giây}$.
  * Tìm kiếm (Search): $\approx 4.000\text{ searches/giây}$.

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/48tEA2j.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Use case 1: Người dùng đăng một tweet

Việc lưu tweet của chính người dùng để phục vụ trang cá nhân (**User Timeline**) có thể dùng **SQL Database** hoặc **NoSQL**. 
Tuy nhiên, việc phân phối tweet đó tới Home Timeline của tất cả những người theo dõi ($60.000$ phân phối/giây) sẽ ngay lập tức làm tê liệt một Database quan hệ truyền thống. Do đó, chúng ta cần dùng **In-Memory Cache (Redis)** hoặc **NoSQL Database** có tốc độ ghi cực nhanh.

Ảnh và video đính kèm sẽ được lưu trữ trực tiếp trên **Object Store (AWS S3)**.

#### Luồng dữ liệu (Data Flow):
1. **Client** gửi request đăng tweet tới **Web Server** (Reverse Proxy).
2. **Web Server** chuyển tiếp tới máy chủ **Write API**.
3. Máy chủ **Write API** lưu tweet vào **SQL Database** (bảng chứa bài viết của user).
4. Máy chủ **Write API** gọi tới **Dịch vụ phân phối (Fanout Service)**:
   * Truy vấn **User Graph Service** để lấy danh sách Followers của user (danh sách này được lưu sẵn trong **Redis Cache**).
   * Ghi ID của tweet mới vào **Home Timeline Cache** của tất cả những người theo dõi đó (Mô hình Fan-out-on-write).
   * Đẩy tweet vào **Search Index Service** để lập chỉ mục tìm kiếm ngay lập tức.
   * Đưa ảnh/video vào **Object Store (S3)**.
   * Đẩy sự kiện vào **Message Queue** để **Notification Service** chạy nền gửi thông báo (Push Notification / Email) cho người theo dõi mà không làm nghẽn luồng request chính.

#### Cấu trúc lưu trữ Home Timeline trong Redis Cache:
Sử dụng kiểu dữ liệu danh sách liên kết kép (**Redis List**) cho mỗi người dùng:
```
           tweet n+2                   tweet n+1                   tweet n
| 8 bytes   8 bytes  1 byte | 8 bytes   8 bytes  1 byte | 8 bytes   8 bytes  1 byte |
| tweet_id  user_id  meta   | tweet_id  user_id  meta   | tweet_id  user_id  meta   |
```
Mỗi khi có tweet mới, ta chỉ việc `LPUSH` bản ghi gồm `tweet_id` + `user_id` vào danh sách của follower. Khi follower mở app, việc lấy 20 tweet mới nhất chỉ tốn độ phức tạp $O(1)$.

---

### Use case 2: Người dùng xem Home Timeline (Bảng tin bạn bè)

1. **Client** gửi request lấy Home Timeline tới **Web Server**.
2. **Web Server** chuyển tiếp tới **Read API Server**.
3. **Read API Server** gọi tới **Timeline Service**:
   * Kéo danh sách `tweet_id` và `user_id` gần nhất từ **Redis Cache** $\rightarrow$ Thao tác này cực nhanh: $O(1)$.
   * Gọi song song tới **Tweet Info Service** bằng lệnh đa truy vấn (`multiget`) để lấy nội dung text, ảnh của các tweet đó.
   * Gọi tới **User Info Service** (`multiget`) để lấy thông tin tác giả (avatar, tên hiển thị).
   * Ghép dữ liệu và trả về cho Client.

---

### Use case 3: Người dùng xem User Timeline (Trang cá nhân)

* Client gửi request xem trang cá nhân của một người dùng cụ thể.
* Request được chuyển tới **Read API Server**.
* Khác với Home Timeline (tổng hợp từ nhiều người), User Timeline chỉ chứa bài viết của 1 người duy nhất $\rightarrow$ Kéo trực tiếp từ **SQL Database** (đã được đánh index theo `user_id` và `created_at`) hoặc tầng Cache riêng của user đó.

---

### Use case 4: Người dùng tìm kiếm từ khóa (Search)

1. **Client** gửi request tìm kiếm từ khóa tới **Search API Server**.
2. **Search API** gọi tới **Search Service**:
   * Phân tích cú pháp (Tokenize & Parse): Tách từ, loại bỏ khoảng trắng, chuẩn hóa chữ hoa/thường, xử lý từ đồng nghĩa.
   * Truy vấn cụm **Search Cluster (Elasticsearch / Apache Lucene)**:
     * Sử dụng mô hình **Scatter-Gather**: Gửi câu lệnh tìm kiếm phân tán tới tất cả các node trong cụm để tìm kiếm trên chỉ mục đảo (Inverted Index).
     * Thu thập kết quả, tính điểm xếp hạng (Ranking), sắp xếp theo độ liên quan/thời gian và trả về cho người dùng.

---

## Bước 4: Mở rộng thiết kế (Scale the Design & Xử lý Điểm nghẽn)

<p align="center">
  <img src="http://i.imgur.com/jrUBAF7.png">
  <br/>
</p>

### Điểm nghẽn nghiêm trọng nhất: "Vấn đề người nổi tiếng" (The Celebrity Problem)

* **Vấn đề:** Nếu một tài khoản có **50 triệu người theo dõi** (như Cristiano Ronaldo, Taylor Swift hay Elon Musk) đăng 1 bài viết:
  * Theo cơ chế **Fan-out-on-write** (ghi trước vào timeline của từng người follow), hệ thống sẽ phải thực hiện **50 triệu thao tác ghi vào Redis**!
  * Quá trình này có thể mất tới vài phút mới xong, gây nghẽn hàng đợi (Queue), tốn tài nguyên và khiến người bình luận trước còn xuất hiện trước cả khi bài viết kịp tới người khác.

* **Giải pháp kết hợp (Hybrid Approach):**
  * **Với người dùng bình thường (< 20.000 followers):** Tiếp tục dùng **Fan-out-on-write** (đẩy sẵn vào Redis của followers lúc đăng bài).
  * **Với người nổi tiếng (VIP / Celebrities - hàng triệu followers):** **KHÔNG dùng Fan-out-on-write**. Tweet của người nổi tiếng chỉ được lưu tại một nơi duy nhất.
  * **Khi người dùng mở Home Timeline (Lúc Đọc - Fan-out-on-read):**
    * Hệ thống lấy timeline từ Redis của người dùng đó.
    * Đồng thời kiểm tra xem người dùng có đang theo dõi người nổi tiếng nào không $\rightarrow$ Kéo tweet mới nhất của người nổi tiếng đó về.
    * Hợp nhất (Merge) 2 danh sách lại theo thời gian thực trước khi hiển thị cho người dùng.

### Các tối ưu hóa bộ nhớ đệm (Cache Optimizations):
* **Chỉ cache vài trăm tweet gần nhất** cho mỗi Home Timeline (người dùng hầu như không cuộn xem quá 500 bài viết cũ).
* **Chỉ cache timeline của người dùng tích cực (Active Users):** Nếu một tài khoản không mở app trong vòng 30 ngày qua, hệ thống sẽ xóa timeline của họ khỏi Redis để tiết kiệm RAM. Khi nào họ đăng nhập trở lại, hệ thống sẽ tái tạo lại timeline từ SQL Database.
* Tầng tìm kiếm (**Search Cluster**) giữ các chỉ mục "hot" trên bộ nhớ RAM để đảm bảo tốc độ phản hồi dưới $50\text{ ms}$.

### Mở rộng Cơ sở dữ liệu:
* Vì lượng ghi lên tới $6.000\text{ tweets/giây}$, một cụm Master-Slave đơn thuần không thể chịu nổi. Cần áp dụng:
  * **Database Sharding:** Phân vùng Database dựa trên `user_id` bằng giải thuật **Consistent Hashing**.
  * **Tách bảng (Federation):** Tách riêng DB Lưu trữ bài viết, DB Đồ thị quan hệ bạn bè (Followers graph), và DB Hồ sơ người dùng.

