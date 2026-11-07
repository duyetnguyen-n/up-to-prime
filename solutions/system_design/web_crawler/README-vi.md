*[English](README.md) ∙ [Tiếng Việt](README-vi.md)*

# Thiết kế Hệ Thống Thu Thập Dữ Liệu Web (Web Crawler)

*Ghi chú: Tài liệu này liên kết trực tiếp tới các phần tương ứng trong [Giáo trình Thiết kế Hệ thống](../../../README-vi.md) để tránh trùng lặp nội dung. Hãy tham khảo các liên kết đó để xem chi tiết các luận điểm, sự đánh đổi (trade-offs) và các giải pháp thay thế.*

---

## Bước 1: Xác định Use Cases và Ràng buộc (Constraints)

> Thu thập yêu cầu và khoanh vùng bài toán.
> Đặt câu hỏi để làm rõ các use case và ràng buộc.
> Thảo luận về các giả định.

### Use Cases (Các trường hợp sử dụng)

#### Chúng ta khoanh vùng bài toán xử lý các use case sau:
* **Hệ thống (Service)** thu thập dữ liệu (crawl) từ danh sách các đường dẫn URL:
  * Tạo chỉ mục đảo (**Reverse Index / Inverted Index**) ánh xạ từ các từ khóa tới các trang web chứa từ khóa đó.
  * Trích xuất tiêu đề (**Title**) và đoạn trích tóm tắt (**Snippet**) cho từng trang.
    * Tiêu đề và đoạn trích là cố định (Static), không thay đổi theo từng truy vấn tìm kiếm.
* **Người dùng (User)** nhập từ khóa tìm kiếm và nhận về danh sách các trang web liên quan kèm tiêu đề và snippet mà crawler đã tạo ra.
  * Chỉ phác thảo sơ đồ mức cao cho use case này, không cần đi quá sâu vào thuật toán ranking của search engine.
* **Hệ thống** phải đảm bảo tính sẵn sàng cao (High Availability).

#### Nằm ngoài phạm vi bài toán (Out of Scope):
* Phân tích hành vi tìm kiếm (Search Analytics).
* Cá nhân hóa kết quả tìm kiếm (Personalized Search).
* Thuật toán tính điểm uy tín trang web (PageRank).

---

### Ràng buộc và Giả định (Constraints and Assumptions)

#### Các giả định đặt ra:
* Lưu lượng tìm kiếm không phân bổ đều (một số từ khóa cực kỳ phổ biến, một số chỉ được tìm kiếm một lần duy nhất).
* Chỉ hỗ trợ người dùng ẩn danh.
* Tốc độ trả về kết quả tìm kiếm phải cực nhanh.
* **Bộ thu thập web (Web Crawler) không được rơi vào vòng lặp vô tận (Infinite loop):** Vòng lặp xảy ra khi đồ thị liên kết giữa các website tạo thành chu trình (Cycles: Trang A trỏ sang B, B trỏ sang C, C trỏ ngược lại A).
* Cần thu thập **$1$ tỷ liên kết (links)**:
  * Các trang web cần được crawl lại định kỳ để đảm bảo tính tươi mới (Freshness) của dữ liệu.
  * Chu kỳ cập nhật trung bình: khoảng $1\text{ lần/tuần}$ (với các trang tin tức hot thì thường xuyên hơn).
  * $\rightarrow$ Khoảng **$4$ tỷ lượt crawl mỗi tháng**.
  * Dung lượng lưu trữ trung bình mỗi trang web: $500\text{ KB}$.
* **$100$ tỷ lượt tìm kiếm mỗi tháng**.

*(Lưu ý: Để rèn luyện tư duy thiết kế hệ thống phân tán, bài toán này yêu cầu tự thiết kế các thành phần thay vì chỉ đơn giản gọi các công cụ có sẵn như Apache Solr hay Apache Nutch).*

#### Tính toán ước lượng sơ bộ (Back-of-the-envelope calculations):
* **Dung lượng lưu trữ nội dung web mỗi tháng:**
  * $500\text{ KB} \times 4\text{ tỷ links/tháng} = 2\text{ PB}$ (Petabytes) nội dung mới mỗi tháng.
  * Sau 3 năm: $\approx 72\text{ PB}$ dữ liệu web được lưu trữ.
* **Tốc độ xử lý trung bình mỗi giây:**
  * Thu thập dữ liệu (Writes): $\approx 1.600\text{ requests/giây}$.
  * Truy vấn tìm kiếm (Reads): $\approx 40.000\text{ search requests/giây}$.

---

## Bước 2: Tạo thiết kế tổng quan (High-Level Design)

<p align="center">
  <img src="http://i.imgur.com/xjdAAUv.png">
  <br/>
</p>

---

## Bước 3: Thiết kế các thành phần cốt lõi (Core Components)

### Use case 1: Hệ thống thu thập dữ liệu từ danh sách URLs

Chúng ta có 2 danh sách dữ liệu chính:
1. `links_to_crawl`: Danh sách các URL đang chờ được crawl, được xếp thứ tự ưu tiên (Priority) dựa trên độ uy tín của website.
2. `crawled_links`: Danh sách các URL đã được cào dữ liệu xong kèm chữ ký nội dung trang (**Page Signature**) để chống trùng lặp.

Cả 2 bảng này được lưu trong **NoSQL Key-Value Database**. Đối với `links_to_crawl`, chúng ta sử dụng **Redis Sorted Sets (ZSET)** để duy trì hàng đợi ưu tiên theo điểm số (Score/Rank).

#### Vòng lặp xử lý của Crawler Service:
1. Lấy URL có độ ưu tiên cao nhất từ `links_to_crawl`.
2. Kiểm tra `crawled_links` trong **NoSQL Database** xem chữ ký trang này đã từng được crawl gần đây hay chưa:
   * Nếu nội dung trang tương tự đã tồn tại: Hạ độ ưu tiên của link đó xuống và bỏ qua (để tránh bị mắc kẹt trong chu trình lặp).
   * Nếu chưa: Tiến hành tải nội dung trang web về.
     * Đẩy một công việc (job) vào hàng đợi của **Reverse Index Service** để phân tích từ khóa và tạo chỉ mục đảo.
     * Đẩy job vào hàng đợi của **Document Service** để bóc tách tiêu đề (Title) và đoạn trích (Snippet).
     * Sinh chữ ký nội dung trang (**Page Signature**).
     * Xóa link khỏi `links_to_crawl` và thêm vào bảng `crawled_links`.

#### Xử lý chống trùng lặp nội dung (Duplicate Content Handling):
* **Trùng lặp URL:** Dùng giải thuật **MapReduce** lọc qua danh sách hàng tỷ URL để loại bỏ các URL trùng nhau.
* **Trùng lặp nội dung (khác URL nhưng nội dung giống nhau):** Tính chữ ký trang (Page Signature) và so sánh độ tương đồng bằng các thuật toán như **Jaccard Index** hoặc **Cosine Similarity** (hoặc SimHash).

#### Xác định chu kỳ cập nhật lại (Re-crawl Frequency):
* Mỗi bản ghi lưu trường `timestamp` lần cào gần nhất.
* Mặc định sau 7 ngày sẽ refresh lại.
* Hỗ trợ đọc file `robots.txt` của từng website để tuân thủ quy định về tần suất cào và không gây quá tải cho máy chủ đối tác.

---

### Use case 2: Người dùng tìm kiếm từ khóa

1. **Client** gửi request tìm kiếm tới **Web Server** (Reverse Proxy).
2. **Web Server** chuyển tiếp tới máy chủ **Query API**.
3. Máy chủ **Query API** thực hiện:
   * **Phân tích truy vấn (Query Parser):** Tách từ, chuyển về chữ thường, xử lý lỗi chính tả (Spell correction), chuyển thành biểu thức logic Boolean (AND, OR).
   * Gọi tới **Reverse Index Service** để tra cứu chỉ mục đảo: Tìm tất cả tài liệu chứa các từ khóa đó $\rightarrow$ Xếp hạng (Rank) và lấy ra danh sách các trang phù hợp nhất.
   * Gọi tới **Document Service** để lấy tiêu đề và đoạn trích ngắn (Snippet) tương ứng.
   * Trả kết quả JSON về cho Client.

#### Thiết kế REST API:
* **Endpoint:** `GET https://search.com/api/v1/search?query=hello+world`
* **Response (HTTP 200):**
  ```json
  [
    {
      "title": "Trang chủ Hello World",
      "snippet": "Chào mừng bạn đến với thế giới lập trình...",
      "link": "https://example.com/hello"
    }
  ]
  ```

---

## Bước 4: Mở rộng thiết kế (Scale the Design & Xử lý Điểm nghẽn)

<p align="center">
  <img src="http://i.imgur.com/bWxPtQA.png">
  <br/>
</p>

### Các giải pháp mở rộng quy mô lớn:

1. **Bộ nhớ đệm cho Tìm kiếm (Search Memory Cache):**
   * Trong thực tế, các truy vấn tìm kiếm tuân theo phân phối Pareto (quy luật 80/20): Một số từ khóa hot được tìm kiếm hàng triệu lần, trong khi nhiều từ khóa chỉ được tìm 1 lần.
   * Đặt **Redis / Memcached** trước cụm Query API để lưu cache kết quả của các từ khóa phổ biến. Giảm thời gian phản hồi xuống còn vài mili-giây và giải phóng hoàn toàn cho cụm Index Service.
2. **Phân vùng chỉ mục đảo (Reverse Index Sharding & Federation):**
   * Với hàng chục Petabyte dữ liệu, toàn bộ chỉ mục đảo bắt buộc phải được **Sharding** ra hàng trăm máy chủ (theo chữ cái đầu của từ khóa hoặc theo Document ID) để chạy song song.
3. **Tối ưu hóa tầng mạng của Crawler Service:**
   * **DNS Lookup Bottleneck:** Cào hàng triệu tên miền khác nhau mỗi giờ sẽ khiến việc phân giải DNS trở thành nút thắt cổ chai. Bộ Crawler cần duy trì một **DNS Cache cục bộ riêng** được làm mới định kỳ.
   * **Connection Pooling:** Duy trì sẵn các kết nối TCP mở (Keep-Alive) để không tốn thời gian bắt tay 3 bước (3-way Handshake) cho mỗi trang web.
   * **Băng thông mạng (Network Bandwidth):** Hệ thống cào web ngốn băng thông cực lớn, cần đặt các Crawler tại nhiều Data Centers trên khắp thế giới để tối ưu đường truyền.
