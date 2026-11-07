*[English](README.md) ∙ [Tiếng Việt](README-vi.md) ∙ [日本語](README-ja.md) ∙ [简体中文](README-zh-Hans.md) ∙ [繁體中文](README-zh-TW.md)*

# Giáo trình Thiết kế Hệ thống (The System Design Primer)

<p align="center">
  <img src="images/jj3A5N8.png">
  <br/>
</p>

## Động lực

> Học cách thiết kế các hệ thống quy mô lớn (Large-Scale Systems).
>
> Chuẩn bị cho các buổi phỏng vấn thiết kế hệ thống (System Design Interview).

### Học cách thiết kế hệ thống quy mô lớn

Học cách thiết kế các hệ thống có khả năng mở rộng (Scalable systems) sẽ giúp bạn trở thành một kỹ sư giỏi hơn.

Thiết kế hệ thống là một chủ đề rất rộng lớn. Có **vô số tài nguyên rải rác khắp nơi trên internet** về các nguyên lý thiết kế hệ thống.

Kho lưu trữ (repository) này là một **tập hợp tài nguyên có tổ chức và hệ thống** nhằm giúp bạn học cách xây dựng các hệ thống ở quy mô lớn.

### Học từ cộng đồng mã nguồn mở

Đây là một dự án mã nguồn mở được cập nhật liên tục. Mọi đóng góp (Contributions) đều được hoan nghênh!

### Chuẩn bị cho buổi phỏng vấn Thiết kế Hệ thống (System Design Interview)

Bên cạnh các vòng phỏng vấn Coding thuật toán, thiết kế hệ thống là một **phần bắt buộc** trong **quy trình phỏng vấn kỹ thuật** tại rất nhiều công ty công nghệ lớn.

**Thực hành các câu hỏi phỏng vấn thiết kế hệ thống phổ biến** và **so sánh** kết quả của bạn với **các giải pháp mẫu**: bao gồm thảo luận kiến trúc, code và sơ đồ thiết kế.

Các chủ đề bổ sung để chuẩn bị phỏng vấn:
* [Lộ trình học tập (Study guide)](#lộ-trình-học-tập-study-guide)
* [Cách tiếp cận một câu hỏi phỏng vấn thiết kế hệ thống](#cách-tiếp-cận-một-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống)
* [Các câu hỏi phỏng vấn thiết kế hệ thống kèm lời giải](#các-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống-kèm-lời-giải)
* [Các câu hỏi thiết kế hướng đối tượng (OOD) kèm lời giải](#các-câu-hỏi-thiết-kế-hướng-đối-tượng-ood-kèm-lời-giải)
* [Các câu hỏi phỏng vấn thiết kế hệ thống bổ sung](#các-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống-bổ-sung)

---

## Flashcards Anki

<p align="center">
  <img src="images/zdCAkB3.png">
  <br/>
</p>

Các bộ [thẻ ghi nhớ Anki (Anki flashcards)](https://apps.ankiweb.net/) đi kèm sử dụng phương pháp lặp lại ngắt quãng (Spaced Repetition) để giúp bạn ghi nhớ sâu các khái niệm thiết kế hệ thống quan trọng.

* [Bộ thẻ System Design](https://github.com/donnemartin/system-design-primer/tree/master/resources/flash_cards/System%20Design.apkg)
* [Bộ thẻ bài tập System Design](https://github.com/donnemartin/system-design-primer/tree/master/resources/flash_cards/System%20Design%20Exercises.apkg)
* [Bộ thẻ bài tập Object-Oriented Design](https://github.com/donnemartin/system-design-primer/tree/master/resources/flash_cards/OO%20Design.apkg)

Rất tiện lợi để ôn tập mọi lúc mọi nơi trên điện thoại hoặc máy tính.

---

## Mục lục các chủ đề Thiết kế Hệ thống

> Tóm tắt các chủ đề thiết kế hệ thống khác nhau, bao gồm ưu và nhược điểm. **Mọi thứ đều là sự đánh đổi (Everything is a trade-off)**.
> Mỗi phần đều chứa liên kết đến các tài liệu nghiên cứu chuyên sâu hơn.

<p align="center">
  <img src="images/jrUBAF7.png">
  <br/>
</p>

* [Các chủ đề thiết kế hệ thống: Bắt đầu từ đây](#các-chủ-đề-thiết-kế-hệ-thống-bắt-đầu-từ-đây)
    * [Bước 1: Xem video bài giảng về Scalability](#bước-1-xem-video-bài-giảng-về-scalability)
    * [Bước 2: Đọc bài viết tổng quan về Scalability](#bước-2-đọc-bài-viết-tổng-quan-về-scalability)
    * [Các bước tiếp theo](#các-bước-tiếp-theo)
* [Performance vs Scalability (Hiệu năng vs Khả năng mở rộng)](#performance-vs-scalability)
* [Latency vs Throughput (Độ trễ vs Thông lượng)](#latency-vs-throughput)
* [Availability vs Consistency (Độ sẵn sàng vs Tính nhất quán)](#availability-vs-consistency)
    * [Định lý CAP (CAP Theorem)](#định-lý-cap-cap-theorem)
        * [CP - Consistency và Partition Tolerance](#cp---consistency-và-partition-tolerance)
        * [AP - Availability và Partition Tolerance](#ap---availability-và-partition-tolerance)
* [Các mô hình nhất quán (Consistency Patterns)](#các-mô-hình-nhất-quán-consistency-patterns)
    * [Weak consistency (Nhất quán yếu)](#weak-consistency)
    * [Eventual consistency (Nhất quán cuối cùng)](#eventual-consistency)
    * [Strong consistency (Nhất quán mạnh)](#strong-consistency)
* [Các mô hình sẵn sàng cao (Availability Patterns)](#các-mô-hình-sẵn-sàng-cao-availability-patterns)
    * [Fail-over (Chuyển đổi dự phòng)](#fail-over)
    * [Replication (Nhân bản dữ liệu)](#replication)
    * [Đo lường độ sẵn sàng bằng những con số (Availability in numbers)](#đo-lường-độ-sẵn-sàng-bằng-những-con-số)
* [Domain Name System (DNS)](#domain-name-system-dns)
* [Content Delivery Network (CDN)](#content-delivery-network-cdn)
    * [Push CDN](#push-cdn)
    * [Pull CDN](#pull-cdn)
* [Load Balancer (Bộ cân bằng tải)](#load-balancer-bộ-cân-bằng-tải)
    * [Active-passive](#active-passive)
    * [Active-active](#active-active)
    * [Layer 4 Load Balancing](#layer-4-load-balancing)
    * [Layer 7 Load Balancing](#layer-7-load-balancing)
    * [Horizontal scaling (Mở rộng theo chiều ngang)](#horizontal-scaling)
* [Reverse Proxy (Web Server)](#reverse-proxy-web-server)
    * [Load Balancer vs Reverse Proxy](#load-balancer-vs-reverse-proxy)
* [Tầng ứng dụng (Application Layer)](#tầng-ứng-dụng-application-layer)
    * [Microservices](#microservices)
    * [Service Discovery](#service-discovery)
* [Cơ sở dữ liệu (Database)](#cơ-sở-dữ-liệu-database)
    * [Hệ quản trị cơ sở dữ liệu quan hệ (RDBMS)](#hệ-quản-trị-cơ-sở-dữ-liệu-quan-hệ-rdbms)
        * [Master-slave replication](#master-slave-replication)
        * [Master-master replication](#master-master-replication)
        * [Federation (Phân vùng theo chức năng)](#federation)
        * [Sharding (Phân mảnh dữ liệu)](#sharding)
        * [Denormalization (Phi chuẩn hóa)](#denormalization)
        * [SQL Tuning (Tối ưu hóa SQL)](#sql-tuning)
    * [NoSQL](#nosql)
        * [Key-value store](#key-value-store)
        * [Document store](#document-store)
        * [Wide column store](#wide-column-store)
        * [Graph database](#graph-database)
    * [Nên chọn SQL hay NoSQL?](#nên-chọn-sql-hay-nosql)
* [Cache (Bộ nhớ đệm)](#cache-bộ-nhớ-đệm)
    * [Client caching](#client-caching)
    * [CDN caching](#cdn-caching)
    * [Web server caching](#web-server-caching)
    * [Database caching](#database-caching)
    * [Application caching](#application-caching)
    * [Caching ở cấp độ database query](#caching-ở-cấp-độ-database-query)
    * [Caching ở cấp độ đối tượng (Object level)](#caching-ở-cấp-độ-đối-tượng-object-level)
    * [Khi nào cần cập nhật Cache?](#khi-nào-cần-cập-nhật-cache)
        * [Cache-aside](#cache-aside)
        * [Write-through](#write-through)
        * [Write-behind (Write-back)](#write-behind-write-back)
        * [Refresh-ahead](#refresh-ahead)
* [Xử lý bất đồng bộ (Asynchronism)](#xử-lý-bất-đồng-bộ-asynchronism)
    * [Message Queues](#message-queues)
    * [Task Queues](#task-queues)
    * [Back Pressure](#back-pressure)
* [Giao tiếp mạng (Communication)](#giao-tiếp-mạng-communication)
    * [Hypertext Transfer Protocol (HTTP)](#hypertext-transfer-protocol-http)
    * [Transmission Control Protocol (TCP)](#transmission-control-protocol-tcp)
    * [User Datagram Protocol (UDP)](#user-datagram-protocol-udp)
    * [Remote Procedure Call (RPC)](#remote-procedure-call-rpc)
    * [Representational State Transfer (REST)](#representational-state-transfer-rest)
* [Bảo mật (Security)](#bảo-mật-security)
* [Phụ lục (Appendix)](#phụ-lục-appendix)
    * [Bảng lũy thừa của 2 (Powers of two table)](#bảng-lũy-thừa-của-2)
    * [Các con số về độ trễ mà mọi lập trình viên cần biết (Latency numbers)](#các-con-số-về-độ-trễ-mà-mọi-lập-trình-viên-cần-biết)
    * [Các câu hỏi phỏng vấn thiết kế hệ thống bổ sung](#các-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống-bổ-sung)
    * [Kiến trúc các hệ thống trong thực tế (Real world architectures)](#kiến-trúc-các-hệ-thống-trong-thực-tế)
    * [Kiến trúc kỹ thuật của các công ty công nghệ lớn](#kiến-trúc-kỹ-thuật-của-các-công-ty-công-nghệ-lớn)
    * [Engineering Blogs của các công ty công nghệ](#engineering-blogs-của-các-công-ty-công-nghệ)

---

## Lộ trình học tập (Study guide)

> Đề xuất các chủ đề cần ôn luyện dựa trên quỹ thời gian trước buổi phỏng vấn của bạn (ngắn hạn, trung hạn, dài hạn).

![Imgur](images/OfVllex.png)

**Hỏi: Đi phỏng vấn có bắt buộc phải biết tất cả mọi thứ ở đây không?**

**Đáp: Không, bạn không cần phải biết tất cả mọi thứ ở đây để chuẩn bị cho buổi phỏng vấn**.

Những gì bạn được hỏi trong buổi phỏng vấn phụ thuộc vào nhiều yếu tố:
* Kinh nghiệm thực tế của bạn
* Nền tảng kỹ thuật của bạn
* Vị trí bạn đang ứng tuyển (Junior, Mid, Senior, Tech Lead, Architect)
* Công ty bạn đang phỏng vấn
* Yếu tố may mắn

Ứng viên có nhiều kinh nghiệm hơn thường được kỳ vọng sẽ hiểu sâu hơn về thiết kế hệ thống. Vị trí Kiến trúc sư (Architect) hoặc Trưởng nhóm kỹ thuật (Team Lead) sẽ được hỏi sâu hơn so với lập trình viên thông thường. Các công ty công nghệ hàng đầu (FAANG/Big Tech) thường có từ 1 đến nhiều vòng phỏng vấn thiết kế hệ thống.

Hãy bắt đầu từ diện rộng (Broad) rồi đi sâu (Deep) vào một số lĩnh vực trọng điểm. Hãy điều chỉnh lộ trình học theo bảng sau:

* **Thời gian ngắn (Short timeline):** Tập trung vào **chiều rộng (breadth)** của các chủ đề. Luyện tập giải **một vài** câu hỏi phỏng vấn cốt lõi.
* **Thời gian trung bình (Medium timeline):** Nắm chắc **chiều rộng** và **một phần chiều sâu (some depth)**. Luyện tập giải **nhiều** câu hỏi phỏng vấn.
* **Thời gian dài hạn (Long timeline):** Nắm vững **chiều rộng** và **đào sâu tối đa (more depth)**. Luyện tập giải **hầu hết** các câu hỏi trong repo.

| Nhiệm vụ | Ngắn hạn | Trung hạn | Dài hạn |
|---|---|---|---|
| Đọc qua [Mục lục các chủ đề](#mục-lục-các-chủ-đề-thiết-kế-hệ-thống) để nắm tổng quan cách hệ thống vận hành | :+1: | :+1: | :+1: |
| Đọc một vài bài viết trên [Engineering Blogs](#engineering-blogs-của-các-công-ty-công-nghệ) của công ty bạn sắp phỏng vấn | :+1: | :+1: | :+1: |
| Đọc một vài [Kiến trúc hệ thống thực tế](#kiến-trúc-các-hệ-thống-trong-thực-tế) | :+1: | :+1: | :+1: |
| Nắm vững [Cách tiếp cận câu hỏi phỏng vấn thiết kế hệ thống](#cách-tiếp-cận-một-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống) | :+1: | :+1: | :+1: |
| Thực hành [Các câu hỏi thiết kế hệ thống có lời giải](#các-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống-kèm-lời-giải) | Một vài bài | Nhiều bài | Hầu hết |
| Thực hành [Các câu hỏi OOD có lời giải](#các-câu-hỏi-thiết-kế-hướng-đối-tượng-ood-kèm-lời-giải) | Một vài bài | Nhiều bài | Hầu hết |
| Xem qua [Các câu hỏi phỏng vấn bổ sung](#các-câu-hỏi-phỏng-vấn-thiết-kế-hệ-thống-bổ-sung) | Một vài bài | Nhiều bài | Hầu hết |

---

## Cách tiếp cận một câu hỏi phỏng vấn thiết kế hệ thống

> Cách giải quyết một câu hỏi System Design Interview một cách bài bản.

Phỏng vấn thiết kế hệ thống là một **cuộc đối thoại mở (open-ended conversation)**. Người phỏng vấn kỳ vọng **bạn là người chủ động dẫn dắt cuộc thảo luận**.

Bạn có thể áp dụng quy trình 4 bước chuẩn sau đây:

### Bước 1: Xác định rõ Use Cases, Ràng buộc (Constraints) và Giả định (Assumptions)

Thu thập yêu cầu và khoanh vùng bài toán. Đặt câu hỏi cho người phỏng vấn để làm rõ các use case và ràng buộc. Thảo luận về các giả định.

* Ai sẽ sử dụng hệ thống này?
* Họ sử dụng hệ thống này như thế nào?
* Có bao nhiêu người dùng (DAU / MAU)?
* Hệ thống thực hiện những chức năng chính nào?
* Đầu vào (Inputs) và đầu ra (Outputs) của hệ thống là gì?
* Dự kiến lượng dữ liệu cần xử lý là bao nhiêu (Dung lượng lưu trữ)?
* Dự kiến có bao nhiêu request mỗi giây (RPS / QPS)?
* Tỉ lệ giữa Đọc và Ghi dự kiến là bao nhiêu (Read to Write ratio)?

### Bước 2: Tạo thiết kế tổng quan (High-Level Design)

Phác thảo sơ đồ kiến trúc mức cao với tất cả các thành phần quan trọng.

* Vẽ các khối thành phần chính và kết nối giữa chúng (Client $\rightarrow$ Load Balancer $\rightarrow$ Web App Servers $\rightarrow$ Database).
* Đưa ra lý do xác đáng cho lựa chọn của bạn.

### Bước 3: Thiết kế chi tiết các thành phần cốt lõi (Core Components)

Đi sâu vào chi tiết của từng thành phần cốt lõi. Ví dụ, nếu bạn được yêu cầu [thiết kế dịch vụ rút gọn link như TinyURL/Pastebin](solutions/system_design/pastebin/README.md), hãy thảo luận:

* Cách sinh và lưu trữ chuỗi hash của URL đầy đủ:
    * Thuật toán băm [MD5](solutions/system_design/pastebin/README.md) và mã hóa [Base62](solutions/system_design/pastebin/README.md).
    * Xử lý xung đột chuỗi hash (Hash collisions).
    * Lựa chọn SQL hay NoSQL.
    * Database schema (thiết kế cấu trúc bảng).
* Dịch ngược URL rút gọn về URL gốc:
    * Truy vấn Database (Database lookup).
* Thiết kế API Endpoints và thiết kế hướng đối tượng (Object-Oriented Design).

### Bước 4: Mở rộng thiết kế (Scale the Design)

Xác định và xử lý các điểm nghẽn (bottlenecks) dựa trên các ràng buộc đã làm rõ ở Bước 1. Ví dụ, bạn có cần những giải pháp sau để giải quyết bài toán scale không:

* Load Balancer (Bộ cân bằng tải)
* Horizontal Scaling (Mở rộng theo chiều ngang)
* Caching (Bộ nhớ đệm Redis / Memcached)
* Database Sharding (Phân mảnh dữ liệu)

Thảo luận về các giải pháp tiềm năng và **sự đánh đổi (trade-offs)**. Mọi thứ trong thiết kế hệ thống đều là sự đánh đổi. Giải quyết điểm nghẽn bằng [các nguyên lý thiết kế hệ thống có khả năng mở rộng](#mục-lục-các-chủ-đề-thiết-kế-hệ-thống).

### Ước lượng sơ bộ trên phong bì (Back-of-the-envelope calculations)

Đôi khi bạn sẽ được yêu cầu tính toán nhẩm nhanh một số con số ước lượng. Hãy tham khảo phần [Phụ lục](#phụ-lục-appendix) cho các tài nguyên sau:
* [Bảng lũy thừa của 2](#bảng-lũy-thừa-của-2)
* [Các con số về độ trễ mà mọi lập trình viên cần biết](#các-con-số-về-độ-trễ-mà-mọi-lập-trình-viên-cần-biết)

---

## Các câu hỏi phỏng vấn thiết kế hệ thống kèm lời giải

> Các câu hỏi thiết kế hệ thống kinh điển kèm theo thảo luận mẫu, code và sơ đồ minh họa nằm trong thư mục `solutions/`.

| Câu hỏi | Lời giải chi tiết (Tiếng Việt) | Bản gốc (English) |
|---|---|---|
| Thiết kế Pastebin.com (hoặc Bit.ly - Dịch vụ rút gọn link) | [Xem Lời giải Tiếng Việt](solutions/system_design/pastebin/README-vi.md) | [English](solutions/system_design/pastebin/README.md) |
| Thiết kế Twitter Timeline và Search (hoặc Facebook Feed) | [Xem Lời giải Tiếng Việt](solutions/system_design/twitter/README-vi.md) | [English](solutions/system_design/twitter/README.md) |
| Thiết kế Web Crawler (Bot thu thập dữ liệu web quy mô lớn) | [Xem Lời giải Tiếng Việt](solutions/system_design/web_crawler/README-vi.md) | [English](solutions/system_design/web_crawler/README.md) |
| Thiết kế Mint.com (Ứng dụng quản lý tài chính cá nhân) | [Xem Lời giải Tiếng Việt](solutions/system_design/mint/README-vi.md) | [English](solutions/system_design/mint/README.md) |
| Thiết kế Cấu trúc dữ liệu cho Mạng xã hội (Social Graph) | [Xem Lời giải Tiếng Việt](solutions/system_design/social_graph/README-vi.md) | [English](solutions/system_design/social_graph/README.md) |
| Thiết kế Key-Value Store cho Search Engine (Query Cache) | [Xem Lời giải Tiếng Việt](solutions/system_design/query_cache/README-vi.md) | [English](solutions/system_design/query_cache/README.md) |
| Thiết kế tính năng Bảng xếp hạng doanh số của Amazon (Sales Rank) | [Xem Lời giải Tiếng Việt](solutions/system_design/sales_rank/README-vi.md) | [English](solutions/system_design/sales_rank/README.md) |
| Thiết kế hệ thống mở rộng lên hàng triệu người dùng trên AWS (Scaling AWS) | [Xem Lời giải Tiếng Việt](solutions/system_design/scaling_aws/README-vi.md) | [English](solutions/system_design/scaling_aws/README.md) |

---

## Các chủ đề thiết kế hệ thống: Bắt đầu từ đây

Bạn là người mới bắt đầu học thiết kế hệ thống?

Trước tiên, bạn cần hiểu những nguyên lý cơ bản: chúng là gì, được dùng như thế nào, ưu điểm và nhược điểm ra sao.

### Bước 1: Xem video bài giảng về Scalability
* [Scalability Lecture tại Đại học Harvard](https://www.youtube.com/watch?v=-W9F__D3oY4)
* Các chủ đề trong bài giảng: Vertical scaling, Horizontal scaling, Caching, Load balancing, Database replication, Database partitioning.

### Bước 2: Đọc bài viết tổng quan về Scalability
* [Scalability for Dummies](https://web.archive.org/web/20221030091841/http://www.lecloud.net/tagged/scalability/chrono) (Bản lưu trữ web).

### Các bước tiếp theo
Chúng ta sẽ cùng tìm hiểu những sự đánh đổi ở mức cao (High-level trade-offs):
* **Performance** vs **Scalability**
* **Latency** vs **Throughput**
* **Availability** vs **Consistency**

Hãy luôn nhớ rằng: **Mọi thứ đều là sự đánh đổi (Everything is a trade-off)**. Sau đó chúng ta sẽ đi sâu vào từng công nghệ cụ thể như DNS, CDN, Load Balancer, Database, Caching và Message Queues.

---

## Performance vs Scalability

Một dịch vụ được gọi là **Scalable (có khả năng mở rộng)** nếu hiệu năng của nó tăng tỉ lệ thuận với lượng tài nguyên được bổ sung. Thông thường, tăng hiệu năng có nghĩa là xử lý được nhiều tác vụ hơn, hoặc xử lý được khối lượng dữ liệu lớn hơn khi tập dữ liệu phình to.

> **Cách phân biệt đơn giản:**
> * Nếu bạn gặp vấn đề về **Hiệu năng (Performance)**: Hệ thống của bạn **chậm đối với một người dùng duy nhất**.
> * Nếu bạn gặp vấn đề về **Khả năng mở rộng (Scalability)**: Hệ thống **chạy rất nhanh với một người dùng, nhưng bị chậm hoặc sập khi chịu tải nặng (nhiều người dùng)**.

---

## Latency vs Throughput

* **Độ trễ (Latency):** Là khoảng thời gian cần thiết để thực hiện một hành động hoặc tạo ra một kết quả (đơn vị: ms, giây).
* **Thông lượng (Throughput):** Là số lượng hành động hoặc kết quả được thực hiện trong một đơn vị thời gian (đơn vị: RPS - Requests Per Second, QPS - Queries Per Second).

> Mục tiêu chung trong thiết kế hệ thống: Hướng tới **Thông lượng tối đa (Maximal Throughput)** với **Độ trễ chấp nhận được (Acceptable Latency)**.

---

## Availability vs Consistency

### Định lý CAP (CAP Theorem)

<p align="center">
  <img src="images/bgLMI2u.png">
  <br/>
  <i><a href="https://robertgreiner.com/cap-theorem-revisited">Nguồn: CAP theorem revisited</a></i>
</p>

Trong một hệ thống máy tính phân tán, bạn chỉ có thể đảm bảo được tối đa **hai trong ba** yếu tố sau:

* **Tính nhất quán (Consistency):** Mọi thao tác đọc đều nhận được dữ liệu được ghi mới nhất hoặc trả về lỗi.
* **Độ sẵn sàng (Availability):** Mọi request gửi lên đều nhận được phản hồi (không bị lỗi), nhưng không đảm bảo phản hồi đó chứa phiên bản dữ liệu mới nhất.
* **Khả năng chịu lỗi phân vùng (Partition Tolerance):** Hệ thống vẫn tiếp tục vận hành bình thường dù mạng giữa các máy chủ bị đứt quãng hoặc mất gói tin.

*Vì mạng kết nối giữa các máy tính vật lý không bao giờ đáng tin cậy 100%, nên bạn bắt buộc phải hỗ trợ Partition Tolerance (P). Do đó, bạn buộc phải đưa ra sự đánh đổi giữa Consistency (C) và Availability (A).*

#### CP - Consistency và Partition Tolerance
Chờ đợi phản hồi từ node bị ngắt kết nối mạng có thể dẫn đến lỗi hết thời gian chờ (timeout). CP là sự lựa chọn phù hợp nếu nghiệp vụ của bạn bắt buộc phải có các thao tác đọc/ghi mang tính nguyên tử (Atomic reads and writes - ví dụ: giao dịch tài chính ngân hàng).

#### AP - Availability và Partition Tolerance
Hệ thống luôn trả về phiên bản dữ liệu sẵn có nhất trên bất kỳ node nào, cho dù phiên bản đó có thể chưa phải là mới nhất. Thao tác ghi có thể mất một khoảng thời gian để lan truyền (propagate) sang các node khác sau khi sự cố mạng được giải quyết.

AP là sự lựa chọn tuyệt vời nếu nghiệp vụ cho phép **Nhất quán cuối cùng (Eventual Consistency)**, hoặc khi hệ thống bắt buộc phải tiếp tục phục vụ người dùng bất chấp sự cố ngoại cảnh (ví dụ: mạng xã hội, đếm view/like).

---

## Các mô hình nhất quán (Consistency Patterns)

Khi lưu trữ nhiều bản sao chép (replicas) của cùng một dữ liệu trên nhiều máy chủ, chúng ta cần chọn cách đồng bộ chúng để client có cái nhìn nhất quán:

### Weak Consistency (Nhất quán yếu)
Sau khi ghi dữ liệu, các thao tác đọc sau đó có thể thấy hoặc không thấy dữ liệu mới. Hệ thống áp dụng cách tiếp cận "cố gắng hết sức" (best effort).
* Mô hình này thường thấy trong Memcached.
* Rất phù hợp cho các use case thời gian thực như gọi điện thoại qua internet (VoIP), video chat, hoặc game nhiều người chơi thời gian thực (realtime multiplayer games). Ví dụ: Khi bạn đang gọi điện mà bị mất sóng 2 giây, khi kết nối trở lại bạn sẽ không nghe thấy những gì người kia đã nói trong 2 giây mất sóng đó.

### Eventual Consistency (Nhất quán cuối cùng)
Sau khi ghi dữ liệu, các thao tác đọc sau đó **chắc chắn sẽ thấy dữ liệu mới** (thường trong vòng vài mili-giây đến vài giây). Dữ liệu được nhân bản bất đồng bộ (asynchronously).
* Mô hình này thấy rõ trong hệ thống DNS và dịch vụ Email. Hoạt động cực kỳ hiệu quả trong các hệ thống đòi hỏi độ sẵn sàng cao (High Availability).

### Strong Consistency (Nhất quán mạnh)
Sau khi ghi dữ liệu thành công, mọi thao tác đọc sau đó **chắc chắn thấy ngay dữ liệu mới nhất**. Dữ liệu được nhân bản đồng bộ (synchronously).
* Mô hình này thấy trong các hệ thống tệp (File Systems) và Hệ quản trị CSDL quan hệ (RDBMS). Rất phù hợp cho các hệ thống cần tính toàn vẹn giao dịch (Transactions).

---

## Các mô hình sẵn sàng cao (Availability Patterns)

Có 2 mô hình bổ trợ lẫn nhau để hỗ trợ tính sẵn sàng cao: **Fail-over (Chuyển đổi dự phòng)** và **Replication (Nhân bản dữ liệu)**.

### Fail-over

#### Active-passive
Trong mô hình Active-Passive, tín hiệu nhịp tim (heartbeat) được gửi liên tục giữa server đang hoạt động (Active) và server dự phòng (Passive). Nếu tín hiệu heartbeat bị gián đoạn (Server Active bị sập), server Passive sẽ lập tức tiếp quản địa chỉ IP của Active và tiếp tục phục vụ người dùng.
* Thời gian gián đoạn (downtime) phụ thuộc vào việc server Passive đã chạy sẵn ở chế độ "hot standby" hay phải khởi động từ đầu ("cold standby").
* Bình thường, chỉ có server Active xử lý lưu lượng truy cập. Còn được gọi là Master-Slave failover.

#### Active-active
Trong mô hình Active-Active, cả hai server đều cùng lúc nhận và xử lý traffic, chia đều tải cho nhau.
* Nếu là server công khai ra ngoài internet, DNS cần biết IP công khai của cả hai server.
* Nếu là server nội bộ, logic ứng dụng cần biết địa chỉ của cả hai server. Còn được gọi là Master-Master failover.

#### Nhược điểm của Fail-over:
* Cần thêm phần cứng và làm tăng độ phức tạp trong quản trị hệ thống.
* Có nguy cơ mất mát dữ liệu nếu hệ thống Active bị sập đột ngột trước khi kịp đồng bộ dữ liệu mới ghi sang Passive.

### Đo lường độ sẵn sàng bằng những con số (Availability in numbers)

Độ sẵn sàng thường được định lượng bằng tỉ lệ phần trăm thời gian hệ thống hoạt động bình thường (Uptime). Thường được đo bằng "số lượng chữ số 9":

#### 99.9% Uptime (Ba số 9)
* Thời gian chết tối đa cho phép mỗi năm: **8 giờ 45 phút 57 giây**
* Mỗi tháng: 43 phút 49 giây
* Mỗi ngày: 1 phút 26 giây

#### 99.99% Uptime (Bốn số 9)
* Thời gian chết tối đa cho phép mỗi năm: **52 phút 35 giây**
* Mỗi tháng: 4 phút 23 giây
* Mỗi ngày: 8.6 giây

#### Ghép nối tiếp (In sequence) vs Ghép song song (In parallel)
* **Ghép nối tiếp:** Độ sẵn sàng tổng thể sẽ **giảm đi**.
  $$\text{Availability}_{\text{Total}} = \text{Availability}(A) \times \text{Availability}(B)$$
  Nếu A và B đều đạt 99.9%, ghép nối tiếp sẽ chỉ còn **99.8%**.
* **Ghép song song (Dự phòng):** Độ sẵn sàng tổng thể sẽ **tăng lên vượt bậc**.
  $$\text{Availability}_{\text{Total}} = 1 - (1 - \text{Availability}(A)) \times (1 - \text{Availability}(B))$$
  Nếu A và B đều đạt 99.9%, ghép song song dự phòng sẽ nâng độ sẵn sàng lên **99.9999%** (Sáu số 9).

---

## Domain Name System (DNS)

<p align="center">
  <img src="images/IOyLj4i.jpg">
  <br/>
</p>

Hệ thống tên miền (DNS) có nhiệm vụ dịch một tên miền như `www.example.com` thành một địa chỉ IP.

DNS hoạt động theo cấu trúc phân cấp (Hierarchical). Kết quả phân giải DNS được cache ở nhiều tầng (Trình duyệt, Hệ điều hành, ISP) trong một khoảng thời gian xác định bởi **TTL (Time to Live)**.

* **NS record (Name Server):** Chỉ định DNS server quản lý tên miền.
* **MX record (Mail Exchange):** Chỉ định máy chủ nhận email.
* **A record (Address):** Trỏ tên miền trực tiếp tới một địa chỉ IPv4.
* **CNAME (Canonical):** Trỏ tên miền tới một tên miền khác (ví dụ `example.com` trỏ sang `www.example.com`).

Các dịch vụ DNS hiện đại (Cloudflare, AWS Route 53) hỗ trợ các thuật toán điều hướng lưu lượng:
* **Weighted Round Robin:** Điều hướng theo trọng số (dành cho bảo trì cụm máy chủ, chia tỉ lệ cho A/B testing).
* **Latency-based:** Điều hướng người dùng tới server có độ trễ mạng thấp nhất.
* **Geolocation-based:** Điều hướng dựa trên vị trí địa lý của người dùng.

---

## Content Delivery Network (CDN)

<p align="center">
  <img src="images/h9TAuGI.jpg">
  <br/>
</p>

CDN là một mạng lưới các máy chủ proxy phân tán trên toàn cầu, phục vụ nội dung từ các vị trí gần với người dùng nhất. Thông thường, các tệp tĩnh như HTML/CSS/JS, hình ảnh và video được phân phối qua CDN.

Sử dụng CDN mang lại 2 lợi ích hiệu năng vượt trội:
1. Người dùng nhận nội dung từ trung tâm dữ liệu gần nhất, giảm tối đa độ trễ.
2. Máy chủ gốc (Origin Server) của bạn được giải phóng, không phải gánh các request tải file tĩnh.

### Push CDN
Mỗi khi có nội dung mới hoặc thay đổi trên server, ứng dụng của bạn sẽ **chủ động đẩy (Push)** file trực tiếp lên CDN.
* Thích hợp cho: Website có ít traffic, nội dung hiếm khi thay đổi. Tối ưu băng thông mạng nhưng tốn dung lượng lưu trữ trên CDN.

### Pull CDN
CDN sẽ tự động **kéo (Pull)** nội dung mới từ server của bạn khi có **người dùng đầu tiên** yêu cầu file đó.
* File được lưu cache trên CDN dựa theo TTL.
* Thích hợp cho: Website có lượng truy cập lớn (Heavy traffic), nội dung mới liên tục vì chỉ những file được người dùng thực sự xem mới được lưu trên CDN.

---

## Load Balancer (Bộ cân bằng tải)

<p align="center">
  <img src="images/h81n9iK.png">
  <br/>
</p>

Bộ cân bằng tải (Load Balancer) phân phối các request gửi đến các máy chủ ứng dụng hoặc cơ sở dữ liệu. Lợi ích:
* Ngăn chặn việc gửi request đến các server đang gặp sự cố (Unhealthy).
* Ngăn chặn tình trạng một máy chủ bị quá tải tài nguyên.
* Loại bỏ điểm lỗi đơn lẻ (Single Point of Failure - SPOF).

Các tính năng mở rộng:
* **SSL Termination:** Giải mã SSL/HTTPS ngay tại Load Balancer, giúp các server ứng dụng phía sau không tốn tài nguyên xử lý mã hóa.
* **Session Persistence (Sticky Sessions):** Gắn cookie để giữ một người dùng luôn kết nối tới cùng một máy chủ nếu ứng dụng chưa chuyển sang kiến trúc Stateless.

### Layer 4 Load Balancing (Cấp độ mạng)
Chỉ kiểm tra thông tin ở tầng Giao vận (Transport Layer) như địa chỉ IP nguồn/đích và Port, không can thiệp vào nội dung gói tin. Thực hiện chuyển tiếp gói tin cực nhanh bằng NAT (Network Address Translation).

### Layer 7 Load Balancing (Cấp độ ứng dụng)
Đọc được toàn bộ dữ liệu ở tầng Ứng dụng (Application Layer): HTTP Header, Cookie, URL. Cho phép điều hướng thông minh (ví dụ: request `/video` đẩy sang cụm server xử lý video, request `/checkout` đẩy sang cụm server thanh toán). Tốn tài nguyên CPU hơn Layer 4 một chút nhưng linh hoạt hơn rất nhiều.

### Horizontal Scaling (Mở rộng theo chiều ngang)
Load Balancer là chìa khóa để Scale ngang:
* Cho phép mở rộng bằng cách thêm các máy chủ thông thường (Commodity machines), tiết kiệm chi phí hơn rất nhiều so với nâng cấp một cỗ máy khổng lồ (Vertical Scaling).
* **Yêu cầu quan trọng:** Máy chủ ứng dụng phải là **Stateless (phi trạng thái)** — không lưu session hay dữ liệu riêng của người dùng trong bộ nhớ cục bộ, mà phải lưu session tập trung ở Database hoặc Redis Cache.

---

## Reverse Proxy (Web Server)

<p align="center">
  <img src="images/n41Azff.png">
  <br/>
</p>

Reverse Proxy (như Nginx, HAProxy) là máy chủ web tập trung hóa các dịch vụ nội bộ và cung cấp một giao diện thống nhất ra bên ngoài.

Lợi ích:
* **Tăng cường bảo mật:** Che giấu địa chỉ IP thật của các máy chủ nội bộ phía sau, chặn IP xấu, giới hạn tốc độ (Rate limiting).
* **Nén dữ liệu:** Tự động nén gzip/brotli phản hồi trả về cho client để tiết kiệm băng thông.
* **SSL Termination & Caching tĩnh:** Tự động cache và phục vụ các file tĩnh trực tiếp mà không cần gọi vào ứng dụng.

### So sánh Load Balancer vs Reverse Proxy
* **Load Balancer:** Hữu ích khi bạn có **nhiều server** làm cùng một nhiệm vụ và cần chia đều tải.
* **Reverse Proxy:** Hữu ích ngay cả khi bạn **chỉ có 1 server duy nhất**, giúp bảo mật, nén dữ liệu và quản lý chứng chỉ SSL.
* Các phần mềm như NGINX và HAProxy có thể đóng cả hai vai trò cùng lúc.

---

## Tầng ứng dụng (Application Layer)

<p align="center">
  <img src="images/yB5SYwm.png">
  <br/>
</p>

Tách biệt tầng Web (Web Layer) ra khỏi tầng Ứng dụng/Nền tảng (Application/Platform Layer) giúp bạn có thể mở rộng (scale) độc lập từng tầng.

### Microservices
Microservices là tập hợp các dịch vụ nhỏ, độc lập, có thể triển khai riêng biệt (independently deployable). Mỗi dịch vụ chạy trong một tiến trình riêng và giao tiếp qua các cơ chế gọn nhẹ (như REST API hoặc gRPC) để phục vụ một mục tiêu nghiệp vụ cụ thể.

### Service Discovery
Khi có hàng trăm microservices chạy động, các công cụ như **Consul**, **Etcd**, **Zookeeper** đóng vai trò là "danh bạ điện thoại", ghi nhớ tên dịch vụ, địa chỉ IP, cổng và liên tục kiểm tra sức khỏe (Health checks) để các dịch vụ tự tìm thấy nhau.

---

## Cơ sở dữ liệu (Database)

<p align="center">
  <img src="images/Xkm5CXz.png">
  <br/>
</p>

### Hệ quản trị cơ sở dữ liệu quan hệ (RDBMS)

Một cơ sở dữ liệu quan hệ (như PostgreSQL, MySQL) lưu trữ dữ liệu dưới dạng các bảng có quan hệ chặt chẽ. Đảm bảo tính chất **ACID**:
* **Atomicity (Tính nguyên tử):** Mọi giao dịch hoặc là thành công trọn vẹn, hoặc là hủy bỏ hoàn toàn (không có trạng thái lỡ dở).
* **Consistency (Tính nhất quán):** Mọi giao dịch đều đưa cơ sở dữ liệu từ trạng thái hợp lệ này sang trạng thái hợp lệ khác.
* **Isolation (Tính cô lập):** Các giao dịch chạy đồng thời không can thiệp hay nhìn thấy dữ liệu trung gian của nhau.
* **Durability (Tính bền vững):** Một khi giao dịch đã commit thành công, dữ liệu chắc chắn được lưu trữ vĩnh viễn dù hệ thống bị mất điện đột ngột.

Các kỹ thuật mở rộng quy mô RDBMS:

#### 1. Master-Slave Replication (Một Master - Nhiều Slave)
<p align="center">
  <img src="images/C9ioGtn.png">
  <br/>
</p>

* **Master:** Tiếp nhận toàn bộ thao tác **Ghi (Writes)** và Đọc. Sau đó Master đồng bộ dữ liệu sang các Slave.
* **Slaves:** Chỉ phục vụ thao tác **Đọc (Reads)**.
* Nếu Master bị sập, hệ thống vẫn tiếp tục đọc bình thường trong khi một Slave được chỉ định bầu làm Master mới.
* *Nhược điểm:* Có độ trễ sao chép (Replication Lag), nếu ghi quá nhiều thì các Slave có thể bị chậm đọc do mải replay dữ liệu ghi.

#### 2. Master-Master Replication
<p align="center">
  <img src="images/krAHLGg.png">
  <br/>
</p>

* Cả hai Master đều tiếp nhận cả Đọc và Ghi, phối hợp với nhau khi có thao tác ghi.
* Nếu một bên sập, bên kia vẫn gánh toàn bộ Đọc & Ghi bình thường.
* *Nhược điểm:* Xung đột dữ liệu (Conflict resolution) khi cùng sửa một bản ghi, tính nhất quán lỏng lẻo hơn.

#### 3. Federation (Phân vùng theo chức năng)
<p align="center">
  <img src="images/U3qV33e.png">
  <br/>
</p>

Tách một CSDL lớn thành các CSDL nhỏ theo chức năng nghiệp vụ. Ví dụ thay vì 1 DB duy nhất, ta tách thành 3 DB: **Users**, **Products**, và **Orders**.
* *Ưu điểm:* Giảm tải lưu lượng đọc/ghi cho từng DB, dữ liệu nhỏ hơn dễ nhét vừa bộ nhớ RAM.
* *Nhược điểm:* Khó thực hiện các câu lệnh `JOIN` giữa 2 bảng nằm ở 2 DB khác nhau.

#### 4. Sharding (Phân mảnh dữ liệu)
<p align="center">
  <img src="images/wU8x5Id.png">
  <br/>
</p>

Sharding chia nhỏ dữ liệu của cùng một bảng ra nhiều server DB khác nhau, mỗi server chỉ quản lý một tập con (subset).
* Ví dụ: Bảng Users 100 triệu người: Shard 1 chứa User có ID từ 1-25 triệu, Shard 2 chứa từ 25-50 triệu...
* Kỹ thuật sharding thông dụng: Dựa trên chữ cái đầu của tên, vị trí địa lý, hoặc dùng thuật toán **Consistent Hashing**.
* *Nhược điểm:* Xử lý logic phức tạp ở code ứng dụng, việc `JOIN` dữ liệu giữa các Shard rất tốn kém, hiện tượng lệch tải nếu một Shard chứa nhiều "khách hàng VIP" (Power users).

#### 5. Denormalization (Phi chuẩn hóa)
Chấp nhận lưu dữ liệu trùng lặp ở nhiều bảng để tránh các thao tác `JOIN` phức tạp ngốn CPU và Disk I/O. Rất phù hợp cho các hệ thống có tỉ lệ Đọc gấp 100 đến 1.000 lần tỉ lệ Ghi.

#### 6. SQL Tuning (Tối ưu hóa SQL)
* Sử dụng đúng kiểu dữ liệu: Dùng `CHAR` cho dữ liệu độ dài cố định, dùng `DECIMAL` cho tiền tệ (tránh sai số dấu phẩy động), không lưu file nhị phân lớn (`BLOB`) vào DB mà lưu URL trỏ tới S3/Object Storage.
* Đánh `INDEX` chuẩn xác (sử dụng cấu trúc B-tree): Đánh index cho các cột hay dùng trong `WHERE`, `JOIN`, `ORDER BY`. Chú ý index làm nhanh thao tác Đọc nhưng làm chậm thao tác Ghi.

---

### NoSQL

NoSQL lưu trữ dữ liệu phi cấu trúc hoặc bán cấu trúc, không bắt buộc tuân thủ quan hệ bảng chặt chẽ và thường tuân theo nguyên lý **BASE**:
* **Basically Available:** Đảm bảo hệ thống luôn sẵn sàng phản hồi.
* **Soft State:** Trạng thái hệ thống có thể thay đổi theo thời gian mà chưa cần có input mới.
* **Eventual Consistency:** Dữ liệu sẽ đạt tới sự nhất quán sau một khoảng thời gian nhất định.

4 loại CSDL NoSQL phổ biến:

#### 1. Key-Value Store (Redis, Memcached)
* Bản chất là một bảng băm (Hash Table) khổng lồ, tốc độ đọc/ghi $O(1)$, lưu trên RAM hoặc SSD. Thích hợp cho lưu Session, Token đăng nhập, Caching.

#### 2. Document Store (MongoDB, CouchDB, Elasticsearch)
* Dữ liệu lưu dưới dạng các tài liệu (JSON, BSON, XML). Mỗi document có thể có cấu trúc linh hoạt khác nhau. Thích hợp cho danh mục sản phẩm, bài viết blog, hồ sơ người dùng.

#### 3. Wide Column Store (Cassandra, HBase, Google Bigtable)
<p align="center">
  <img src="images/n16iOGk.png">
  <br/>
</p>

* Lưu trữ dữ liệu theo cột thay vì theo dòng. Khả năng scale cực kỳ mạnh mẽ lên tới hàng Terabyte/Petabyte dữ liệu. Rất phù hợp cho dữ liệu chuỗi thời gian (Time-series), dữ liệu log, lịch sử giao dịch.

#### 4. Graph Database (Neo4j)
<p align="center">
  <img src="images/fNcl65g.png">
  <br/>
</p>

* Tối ưu hóa cho các mối quan hệ phức tạp hình mạng nhện (Nodes và Edges). Cực kỳ mạnh trong việc tìm kiếm bạn bè chung, gợi ý mạng xã hội, phát hiện gian lận tài chính.

---

## Cache (Bộ nhớ đệm)

<p align="center">
  <img src="images/Q6z24La.png">
  <br/>
</p>

Cache lưu trữ dữ liệu tạm thời trên bộ nhớ RAM siêu tốc (như Redis, Memcached) để giảm tải cho Database và tăng tốc độ phản hồi cho người dùng.

### Các chiến lược cập nhật Cache (Cache Invalidation Patterns):

#### 1. Cache-Aside (Lazy Loading - Phổ biến nhất)
<p align="center">
  <img src="images/ONjORqk.png">
  <br/>
</p>

* **Luồng xử lý:** Ứng dụng kiểm tra Cache trước $\rightarrow$ Nếu thấy (Cache Hit) thì trả về ngay $\rightarrow$ Nếu không thấy (Cache Miss), ứng dụng truy vấn Database $\rightarrow$ Ghi kết quả vào Cache $\rightarrow$ Trả về cho client.
* *Ưu điểm:* Chỉ cache những dữ liệu người dùng thực sự cần đọc.
* *Nhược điểm:* Khi cache miss sẽ tốn 3 lần di chuyển dữ liệu (App $\rightarrow$ Cache $\rightarrow$ DB $\rightarrow$ Cache).

#### 2. Write-Through (Ghi đồng thời)
<p align="center">
  <img src="images/0vBc0hN.png">
  <br/>
</p>

* Ứng dụng ghi dữ liệu vào Cache $\rightarrow$ Cache tự động ghi đồng bộ xuống Database ngay lập tức.
* Dữ liệu trong Cache luôn mới nhất, nhưng tốc độ ghi sẽ chậm hơn vì phải chờ cả 2 ghi xong.

#### 3. Write-Behind / Write-Back (Ghi hoãn lại)
<p align="center">
  <img src="images/rgSrvjG.png">
  <br/>
</p>

* Ứng dụng ghi vào Cache xong báo thành công ngay lập tức $\rightarrow$ Cache từ từ gom lại rồi ghi bất đồng bộ xuống Database sau.
* Tốc độ ghi siêu nhanh, nhưng có nguy cơ mất dữ liệu nếu server Cache bị mất điện/sập trước khi kịp lưu xuống Database.

#### 4. Refresh-Ahead
* Cache tự động dự đoán và làm mới các bản ghi sắp hết hạn trước khi người dùng yêu cầu.

---

## Xử lý bất đồng bộ (Asynchronism)

<p align="center">
  <img src="images/54GYsSx.png">
  <br/>
</p>

Giúp đưa các tác vụ tốn nhiều thời gian (gửi email, render video, xuất file excel, phân tích dữ liệu) ra khỏi luồng request chính để người dùng không phải chờ đợi.

### Message Queues (Hàng đợi thông điệp)
* **Quy trình:** Ứng dụng đẩy một công việc (job) vào hàng đợi (Queue) rồi trả về cho người dùng "Đã tiếp nhận" $\rightarrow$ Các Worker chạy ngầm sẽ lấy job từ Queue ra để xử lý từ từ.
* **Công nghệ phổ biến:**
  * **Redis:** Dùng làm message broker đơn giản, cực nhanh nhưng có rủi ro mất tin nhắn nếu sập.
  * **RabbitMQ:** Hỗ trợ định tuyến phức tạp (AMQP), đảm bảo độ tin cậy cao.
  * **Apache Kafka:** Xử lý luồng dữ liệu khổng lồ (Streaming data), throughput cực lớn.
  * **Amazon SQS:** Dịch vụ hàng đợi được quản lý hoàn toàn trên AWS.

### Back Pressure
Khi lượng công việc đổ vào Queue quá nhanh mà Worker xử lý không kịp, hàng đợi sẽ phình to quá dung lượng RAM. Cơ chế **Back Pressure** giúp giới hạn kích thước hàng đợi, trả về mã HTTP 503 (Server Busy) yêu cầu client thử lại sau với thuật toán giãn cách lũy thừa (**Exponential Backoff**).

---

## Giao tiếp mạng (Communication)

<p align="center">
  <img src="images/5KeocQs.jpg">
  <br/>
</p>

### HTTP
Giao thức hướng request/response ở tầng ứng dụng. Các phương thức phổ biến:
* `GET`: Đọc tài nguyên (Idempotent - gọi nhiều lần kết quả không đổi, Safe, có thể Cache).
* `POST`: Tạo mới tài nguyên.
* `PUT`: Tạo hoặc thay thế toàn bộ tài nguyên (Idempotent).
* `PATCH`: Cập nhật một phần tài nguyên.
* `DELETE`: Xóa tài nguyên (Idempotent).

### So sánh TCP vs UDP

| Tiêu chí | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---|---|---|
| Kết nối | Cần bắt tay thiết lập kết nối (3-way Handshake) | Không cần kết nối (Connectionless) |
| Độ tin cậy | Đảm bảo 100% gói tin đến đúng thứ tự, tự gửi lại nếu mất gói | Không đảm bảo gói tin đến đích, có thể mất gói hoặc sai thứ tự |
| Kiểm soát tắc nghẽn | Có Flow Control & Congestion Control | Không có |
| Tốc độ & Độ trễ | Chậm hơn, tốn overhead bộ nhớ | Siêu nhanh, độ trễ tối thiểu |
| Use cases | Web (HTTP/HTTPS), Database, Email, File transfer (FTP) | Video livestream, Voice chat (VoIP), Game thời gian thực |

### So sánh REST vs RPC (gRPC)

* **REST (Representational State Transfer):** Tập trung vào **Tài nguyên (Resources/Data)**. Sử dụng HTTP verbs (`GET`, `POST`, `PUT`...) và định dạng JSON. Dễ đọc, linh hoạt, chuẩn mực cho các Public API ra bên ngoài internet.
* **RPC (Remote Procedure Call - ví dụ gRPC):** Tập trung vào **Hành động (Actions/Behaviors)**. Gọi một hàm trên server từ xa giống hệt như gọi hàm local trong code. Sử dụng binary serialization (Protobuf) cho tốc độ truyền tải cực nhanh, nén dữ liệu tối đa. Rất lý tưởng cho giao tiếp nội bộ giữa các microservices (Internal communication).

---

## Bảo mật (Security)

4 nguyên tắc bảo mật tối thiểu trong thiết kế hệ thống:
1. **Mã hóa:** Mã hóa dữ liệu khi truyền trên đường truyền (In Transit - HTTPS/TLS) và khi lưu trữ trong ổ cứng (At Rest - AES-256).
2. **Khử trùng dữ liệu đầu vào (Input Sanitization):** Kiểm tra chặt chẽ mọi dữ liệu do người dùng gửi lên để ngăn chặn tấn công **XSS** và **SQL Injection**.
3. **Parameterized Queries:** Luôn sử dụng Prepared Statements trong truy vấn Database để triệt tiêu lỗi SQL Injection.
4. **Nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege):** Mỗi dịch vụ hay tài khoản chỉ được cấp đúng những quyền tối thiểu cần thiết để làm việc.

---

## Phụ lục: Các con số mọi Kỹ sư cần nắm vững

### Bảng lũy thừa của 2
* $2^{10} \approx 1.000$ (1 KB - Kilo)
* $2^{20} \approx 1.000.000$ (1 MB - Mega)
* $2^{30} \approx 1.000.000.000$ (1 GB - Giga)
* $2^{40} \approx 1.000.000.000.000$ (1 TB - Tera)

### Các con số về độ trễ (Latency numbers every programmer should know)
* Truy xuất L1 Cache: **0.5 ns**
* Truy xuất RAM: **100 ns** (chậm gấp 200 lần L1 Cache)
* Đọc ngẫu nhiên từ ổ SSD: **150.000 ns (150 $\mu$s)**
* Đọc 1 MB tuần tự từ RAM: **250.000 ns (250 $\mu$s)**
* Một vòng Round-trip trong cùng Datacenter: **500.000 ns (0.5 ms)**
* Đọc 1 MB tuần tự từ SSD: **1.000.000 ns (1 ms)**
* Tìm kiếm trên ổ cơ HDD (Seek): **10.000.000 ns (10 ms)** (chậm gấp 10 lần SSD)
* Gửi gói tin khứ hồi Mỹ $\rightarrow$ Hà Lan $\rightarrow$ Mỹ: **150.000.000 ns (150 ms)**
