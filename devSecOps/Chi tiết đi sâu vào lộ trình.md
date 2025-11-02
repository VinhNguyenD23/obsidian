Previous: [[lộ trình chi tiết nhất học devSecOps]]
### Giai đoạn 1: Nền tảng & Monolith

#### 1. Core Principles (Yêu cầu phi chức năng)

- Cách tư duy (Thinking Process):
    
    Đây là bước quan trọng nhất. Một thiết kế "tốt" là thiết kế đáp ứng đúng yêu cầu. Trước khi vẽ một cái hộp nào, tôi phải hỏi: Hệ thống này cần ưu tiên gì nhất?
    
    - **Availability (Tính sẵn sàng):** Hệ thống có cần online 99.999% không (vd: hệ thống thanh toán)? Hay 99.9% là đủ (vd: trang blog nội bộ)?
        
    - **Performance (Hiệu năng):** Response time P99 (percentile 99) phải dưới 200ms (vd: trang TMĐT) hay 1 giây là chấp nhận được (vd: trang báo cáo)?
        
    - **Consistency (Tính nhất quán):** Dữ liệu có phải _luôn luôn_ chính xác ngay lập tức không (vd: chuyển tiền ngân hàng - Strong Consistency)? Hay có thể "trễ" vài giây (vd: số lượt "like" - Eventual Consistency)?
        
    - **Scalability (Khả năng mở rộng):** Hệ thống cần phục vụ 1,000 users/ngày hay 1,000,000 users/ngày?
        
    - **Trade-off (Đánh đổi):** Theo định lý CAP, không thể có cả 3 (Consistency, Availability, Partition Tolerance). Với hệ thống phân tán, P (lỗi mạng) luôn xảy ra. Vậy tôi phải chọn giữa C và A. Hệ thống TMĐT có thể ưu tiên A (luôn cho đặt hàng) hơn C (lý tưởng nhất là check kho chính xác, nhưng thà cho đặt rồi báo hết hàng sau còn hơn sập web).
        
- Cách trả lời (How to Answer):
    
    "Để bắt đầu, chúng ta cần làm rõ các yêu cầu phi chức năng (NFRs). Dựa trên mô tả, tôi giả định đây là một hệ thống TMĐT, vì vậy Availability và Performance (latency thấp) là ưu tiên hàng đầu. Chúng ta sẽ chấp nhận Eventual Consistency cho các tính năng như đếm 'view' hoặc 'review', nhưng đảm bảo Strong Consistency cho các nghiệp vụ lõi như 'thanh toán' và 'kho hàng' (theo tiêu chí của bạn: ưu tiên transaction). Mục tiêu của chúng ta là P99 latency dưới 300ms và uptime 99.95%."
    
- Góc nhìn DevSecOps:
    
    Bảo mật là một yêu cầu phi chức năng. Ngay từ bước này, tôi phải thực hiện Threat Modeling (Mô hình hóa mối đe dọa) cơ bản.
    
    - _Tôi đang bảo vệ cái gì?_ (Dữ liệu PII của user, thông tin thẻ).
        
    - _Ai đang tấn công?_ (User thông thường, hacker mũ đen, botnet).
        
    - _Họ tấn công thế nào?_ (SQL Injection, XSS, DDoS, tấn công logic nghiệp vụ).
        
    - Câu trả lời sẽ là: "Thiết kế phải tuân thủ nguyên tắc **Least Privilege**, **Defense in Depth**, và mọi dữ liệu PII _phải_ được mã hóa at-rest và in-transit."
        

---

#### 2. Thiết kế Monolith (Layered)

- Cách tư duy (Thinking Process):
    
    Với một team nhỏ hoặc sản phẩm mới (MVP), Monolith là lựa chọn đúng đắn. Nó đơn giản để phát triển, debug và deploy. Tuy nhiên, tôi phải cấu trúc nó thật sạch sẽ để có thể tách ra sau này. Cấu trúc yêu thích của tôi là Layered Architecture (Controller → Service → Repository), tuân thủ SOLID, đặc biệt là chữ D (Dependency Inversion). Service không nên biết về request hay response của Express/NestJS, nó chỉ nên nhận DTO (theo tiêu chí của bạn: controller mỏng).
    
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ bắt đầu với kiến trúc Monolith để tối ưu tốc độ phát triển và đơn giản hóa việc deploy. Để đảm bảo 'maintainability', chúng ta sẽ áp dụng chặt chẽ kiến trúc 3-lớp:
    
    1. **Controller:** Chỉ chịu trách nhiệm routing, validate DTO (dùng `class-validator`) và gọi Service.
        
    2. **Service:** Chứa toàn bộ business logic. Đây là nơi thực hiện transaction (user memory).
        
    3. Repository: (Ví dụ dùng Prisma Client/TypeORM Repo) Chỉ chịu trách nhiệm giao tiếp với DB.
        
        Điều này giúp code base sạch sẽ và sẵn sàng cho việc 'refactor' sang microservices sau này."
        
- Góc nhìn DevSecOps:
    
    Một khối Monolith là một 'attack surface' (bề mặt tấn công) lớn và tập trung.
    
    1. **Input Validation:** Phải validate _mọi_ dữ liệu đầu vào tại lớp Controller (DTO) để chặn Mass Assignment, SQLi, XSS.
        
    2. **Security Headers:** Dùng `helmet` (user memory) ngay lập tức để vá các lỗ hổng cơ bản của HTTP.
        
    3. **Logging:** Dùng `pino` (user memory) để có structured logging, và _phải_ có `correlation-id` (user memory) để theo dõi request.
        

---

#### 3. Database (Monolith)

- Cách tư duy (Thinking Process):
    
    Trong Monolith, 99% trường hợp tôi sẽ dùng CSDL quan hệ (SQL). Nó mạnh mẽ, hỗ trợ ACID transaction toàn diện, điều mà nghiệp vụ (như đặt hàng, thanh toán) rất cần. User của tôi ưu tiên Postgres (user memory), đây là lựa chọn tuyệt vời. Tôi sẽ thiết kế schema cẩn thận, chuẩn hóa (normalization) ở mức 3NF, và sau đó 'de-normalize' (phản chuẩn hóa) một vài chỗ nếu performance yêu cầu. Phải nghĩ về indexing ngay từ đầu cho các cột WHERE và JOIN phổ biến.
    
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ sử dụng PostgreSQL (user memory) làm CSDL chính vì khả năng hỗ trợ transaction ACID mạnh mẽ, phù hợp với các nghiệp vụ phức tạp. Schema sẽ được thiết kế chuẩn hóa (normalized) để đảm bảo tính toàn vẹn dữ liệu. Chúng ta sẽ dùng ORM (Prisma/TypeORM) (user memory) để tương tác và quản lý migration (user memory). Các nghiệp vụ quan trọng (ví dụ: createOrder) sẽ luôn được bọc trong một database transaction tại Service layer."
    
- Góc nhìn DevSecOps:
    
    Bảo vệ "kho báu".
    
    1. **Password Hashing:** _Không bao giờ_ lưu password plaintext. Dùng **Argon2** (user memory) hoặc bcrypt.
        
    2. **Data Encryption:** Dữ liệu nhạy cảm (PII, số thẻ) phải được mã hóa _trong DB_ (encryption at-rest), có thể ở mức application-level (dùng AES) hoặc DB-level (TDE của Postgres).
        
    3. **Access Control:** User của ứng dụng kết nối tới DB phải có quyền giới hạn (vd: chỉ `SELECT`, `INSERT`, `UPDATE` trên các bảng cụ thể), _không_ dùng user `postgres` (superuser).
        
    4. **Migration Security:** (user memory) Mọi script migration phải được review (code review) để tránh các thay đổi nguy hiểm (vd: `DROP TABLE`).
        

---

#### 4. Caching (Bộ đệm)

- Cách tư duy (Thinking Process):
    
    Sau khi DB được thiết kế, tôi thấy có những query rất tốn kém (vd: SELECT * FROM products JOIN ... WHERE ... ORDER BY ...) hoặc dữ liệu được đọc rất nhiều nhưng ít thay đổi (vd: profile user, danh mục sản phẩm). Đây là lúc cần Caching.
    
    - _Chiến lược là gì?_ Tôi thường dùng **Cache-Aside Pattern**.
        
    - _Logic:_ App -> Hỏi Cache (Redis) -> Cache có? (Cache Hit) -> Trả về.
        
    - _Logic:_ App -> Hỏi Cache (Redis) -> Cache không có? (Cache Miss) -> App hỏi DB -> App lấy data -> App ghi data vào Cache -> Trả về.
        
    - _Vấn đề:_ Dữ liệu bị "trôi" (stale) thì sao? Cần chiến lược **Cache Invalidation**. Cách đơn giản nhất là set **TTL** (Time-to-Live), vd: 5 phút. Cách phức tạp hơn là "write-through" hoặc "write-around" (khi update DB, chủ động xóa key trong cache).
        
- Cách trả lời (How to Answer):
    
    "Để cải thiện performance cho các tác vụ đọc, chúng ta sẽ implement một lớp caching dùng Redis. Chúng ta sẽ áp dụng 'Cache-Aside Pattern' để cache các dữ liệu hay được truy cập, ví dụ như thông tin sản phẩm hoặc session user. Chúng ta sẽ set TTL (ví dụ: 10 phút) cho các key này để cân bằng giữa performance và tính 'freshness' của dữ liệu."
    
- Góc nhìn DevSecOps:
    
    Cache cũng là một "data store" và có thể bị tấn công.
    
    1. **Secure Connection:** Redis phải được đặt trong mạng nội bộ (private network), có đặt mật khẩu ( `requirepass` ), và giao tiếp qua TLS nếu có thể.
        
    2. **Cache Poisoning:** Nếu tôi cache một trang dựa trên input của user (vd: `GET /search?q=abc`), kẻ tấn công có thể chèn script vào `q` (`<script>...`) và nó sẽ bị cache lại, tấn công mọi user khác. _Giải pháp:_ Luôn sanitize input _trước khi_ dùng nó làm key hoặc cache content.
        
    3. **Data Sensitivity:** _Không_ cache dữ liệu quá nhạy cảm (vd: raw token, chi tiết thẻ ngân hàng) trừ khi nó được mã hóa cẩn thận.
        

---

### Giai đoạn 2: Mở rộng Monolith

#### 5. Vertical vs. Horizontal Scaling

- Cách tư duy (Thinking Process):
    
    App Monolith của tôi bắt đầu chậm. Lựa chọn đầu tiên là Vertical Scaling (Scale Up): nâng cấp server (thêm CPU, RAM). Nó rất đơn giản (chỉ cần restart), nhưng đắt đỏ và có giới hạn (không thể có server 1000-core).
    
    Lựa chọn tốt hơn (và bắt buộc cho microservices sau này) là Horizontal Scaling (Scale Out): chạy nhiều bản sao (instance) của app. Nó rẻ hơn, linh hoạt vô hạn, nhưng có một yêu cầu: Ứng dụng phải Stateless.
    
    - _Stateless nghĩa là gì?_ App không được lưu trạng thái (state) gì trong bộ nhớ của nó. Ví dụ: không lưu session user trong RAM, mà phải đẩy ra ngoài (Redis/DB).
        
- Cách trả lời (How to Answer):
    
    "Hệ thống đang gặp bottleneck. Scale dọc (Vertical) chỉ là giải pháp tạm thời. Để chuẩn bị cho tăng trưởng, chúng ta sẽ thiết kế ứng dụng theo hướng Stateless, tức là mọi trạng thái (như user session) sẽ được lưu trữ bên ngoài (vd: Redis). Khi đã stateless, chúng ta có thể scale ngang (Horizontal) bằng cách chạy nhiều instance của ứng dụng."
    
- Góc nhìn DevSecOps:
    
    Scale ngang làm tăng số lượng "danh tính" (identities).
    
    1. **Secret Management:** Giờ tôi có 3 instance, vậy 3 instance này lấy DB password ở đâu? Không thể hardcode hay để trong `.env`. Chúng phải được inject lúc runtime một cách an toàn. Đây là lúc cần các công cụ như **HashiCorp Vault**, AWS Secrets Manager, hoặc ít nhất là **env schema-validated** (user memory) để đảm bảo các biến môi trường cần thiết được cung cấp an toàn.
        
    2. **Consistency:** Đảm bảo mọi instance được build từ _cùng một_ artifact, cùng một config (ngoại trừ secrets).
        

---

#### 6. Load Balancer (LB)

- Cách tư duy (Thinking Process):
    
    Tôi đã có 3 instance (từ bước 5). Làm sao client (trình duyệt, mobile app) biết phải gọi vào instance nào? Tôi cần một "cảnh sát giao thông" đứng trước: Load Balancer.
    
    - _Nhiệm vụ 1:_ Phân phối traffic (Round Robin, Least Connections...).
        
    - _Nhiệm vụ 2:_ **Health Checks**. LB sẽ liên tục "ping" các instance (`/health` endpoint). Nếu instance 1 bị sập, LB sẽ tự động ngừng gửi traffic đến nó, đảm bảo tính sẵn sàng (HA).
        
- Cách trả lời (How to Answer):
    
    "Khi đã scale ngang, chúng ta sẽ đặt một Load Balancer (ví dụ: Nginx, HAProxy, hoặc ALB của AWS) ở phía trước các instance. LB sẽ chịu trách nhiệm phân phối traffic và thực hiện Health Checks tới endpoint /health của từng instance. Nếu một instance lỗi, LB sẽ tự động loại nó ra khỏi pool, giúp hệ thống đạt High Availability."
    
- Góc nhìn DevSecOps:
    
    LB là tuyến phòng thủ đầu tiên (first line of defense) từ Internet.
    
    1. **TLS/SSL Termination:** LB là nơi "giải mã" HTTPS. Nó xử lý toàn bộ gánh nặng mã hóa, các server bên trong chỉ cần nói chuyện HTTP (trong mạng nội bộ an toàn).
        
    2. **Rate Limiting:** (user memory) Đây là nơi _tốt nhất_ để implement rate limiting, chặn các cuộc tấn công DDoS layer 7 hoặc brute-force login.
        
    3. **WAF (Web Application Firewall):** Nhiều LB hiện đại tích hợp WAF, có thể tự động lọc các request có dấu hiệu SQLi, XSS...
        

---

#### 7. Database Scaling (Read Replicas)

- Cách tư duy (Thinking Process):
    
    App chạy nhanh, nhưng giờ DB đang là bottleneck. Tôi phân tích (vd: dùng pg_stat_statements) và thấy 90% query là SELECT, 10% là INSERT/UPDATE.
    
    Giải pháp: Read Replicas (Bản sao chỉ đọc).
    
    - Tôi sẽ tạo 1 DB Master (nhận 10% ghi) và 1 hoặc nhiều DB Replica (nhận 90% đọc).
        
    - Dữ liệu từ Master sẽ tự động đồng bộ (replicate) sang Replicas.
        
    - _Trade-off:_ **Replication Lag** (Độ trễ đồng bộ). Dữ liệu ở Replica có thể bị trễ vài mili-giây (hoặc vài giây nếu tải nặng) so với Master. App phải chấp nhận điều này.
        
- Cách trả lời (How to Answer):
    
    "DB đang quá tải vì các tác vụ đọc. Chúng ta sẽ implement mô hình Master-Replica. Ứng dụng sẽ được cấu hình với 2 connection string:
    
    1. **Master Connection:** Cho mọi tác vụ CUD (Create, Update, Delete).
        
    2. Replica Connection: Cho tất cả các tác vụ Read (SELECT).
        
        (Trong TypeORM/Prisma, điều này có thể được cấu hình). Chúng ta cần nhận thức và xử lý vấn đề Replication Lag, ví dụ: user vừa đổi avatar (WRITE vào Master), ngay lập tức refresh trang (READ từ Replica), có thể họ sẽ thấy avatar cũ."
        
- Góc nhìn DevSecOps:
    
    Phân tách quyền hạn rõ ràng.
    
    1. **Least Privilege:** DB user mà ứng dụng dùng để kết nối vào **Replica** _chỉ được có quyền `SELECT`_. Tuyệt đối không có quyền `INSERT` hay `DROP`.
        
    2. **Network Isolation:** Đảm bảo traffic đồng bộ (replication traffic) giữa Master và Replica diễn ra trên một kênh mã hóa (SSL) hoặc một mạng nội bộ cực kỳ an toàn.
        

---

#### 8. Message Queue (MQ) / Async

- Cách tư duy (Thinking Process):
    
    User nhấn nút "Đặt hàng". API của tôi phải: 1. Lưu Order vào DB. 2. Gửi email xác nhận. 3. Thông báo cho kho. 4. Thông báo cho Giao vận.
    
    Nếu làm đồng bộ (synchronous), tác vụ 2, 3, 4 có thể chậm (vd: server email chậm 3 giây). User phải chờ 3 giây + thời gian của 1.
    
    Giải pháp: Asynchronous (Bất đồng bộ) dùng Message Queue (RabbitMQ, Kafka).
    
    - _Logic mới:_ API "Đặt hàng" -> 1. Lưu Order vào DB (Sync, < 50ms) -> 2. Đẩy 3 message ("Email", "Kho", "Giaoận") vào Queue -> Trả về 200 OK cho user ngay lập tức.
        
    - Ở một nơi khác, các **Workers** (tiến trình riêng) sẽ lắng nghe Queue và từ từ xử lý 3 message kia.
        
    - Lợi ích: API siêu nhanh (tăng performance) và Kiên cường (resilience) (nếu server email sập, message vẫn nằm trong queue, worker sẽ thử lại sau).
        
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ tách các tác vụ không yêu cầu phản hồi ngay lập tức (như gửi email, xử lý ảnh) ra khỏi luồng request chính. Chúng ta sẽ dùng RabbitMQ. Khi user đặt hàng, service chỉ ghi vào DB và 'publish' một message (ví dụ: OrderCreatedEvent) vào queue, sau đó trả về 202 Accepted cho user. Các 'worker' riêng biệt sẽ 'subscribe' vào queue đó để xử lý bất đồng bộ."
    
- Góc nhìn DevSecOps:
    
    Queue cũng là một hạ tầng cần bảo mật.
    
    1. **Data in Message:** Message có chứa PII không? (vd: email của user). Nếu có, message _phải được mã hóa_ nội dung, vì admin của RabbitMQ có thể đọc được.
        
    2. **Access Control (ACLs):** Service nào được publish vào queue nào? Service nào được subscribe? Phải cấu hình ACLs chặt chẽ. `PaymentService` không nên được phép đọc message của `UserService`.
        
    3. **Dead Letter Queue (DLQ):** Khi một message xử lý lỗi 10 lần (vd: user đăng ký email rác), nó nên được chuyển vào DLQ để phân tích thủ công, tránh vòng lặp vô tận làm sập worker.
        

---

### Giai đoạn 3: Chuyển đổi (Microservices)

#### 9. Decomposition (Chia tách)

- Cách tư duy (Thinking Process):
    
    Đây là bước khó nhất. Chia Monolith như thế nào? Cách tốt nhất là theo Domain-Driven Design (DDD). Tôi phải xác định các Bounded Contexts (Ngữ cảnh bị ràng buộc).
    
    - Một "Bounded Context" là một khu vực nghiệp vụ độc lập.
        
    - Ví dụ: Nghiệp vụ "Users" (quản lý profile, auth) là một context. "Products" (quản lý catalog, giá) là một context. "Orders" (quản lý giỏ hàng, đặt hàng) là một context.
        
    - Mỗi Bounded Context này sẽ trở thành một **Microservice** (`UserService`, `ProductService`, `OrderService`).
        
    - _Quy tắc vàng:_ Mỗi service _sở hữu_ database của riêng nó. `OrderService` _không bao giờ_ được phép query thẳng vào DB của `UserService`. (User memory: không trộn read/write).
        
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ chia Monolith dựa trên các Bounded Contexts theo DDD. Phân tích nghiệp vụ cho thấy chúng ta có 3 context chính: UserContext, ProductContext, và OrderContext. Chúng sẽ trở thành 3 microservice đầu tiên: UserService, ProductService, OrderService. Quan trọng nhất, mỗi service sẽ có một database riêng để đảm bảo tính tự chủ (autonomy) và 'loose coupling'."
    
- Góc nhìn DevSecOps:
    
    Bề mặt tấn công (attack surface) vừa tăng lên N lần.
    
    1. **Zero Trust:** (Không tin tưởng). `OrderService` _không được_ tin tưởng `UserService`. Mọi request đi từ `UserService` đến `OrderService` (nếu có) _vẫn phải_ được xác thực (vd: dùng JWT nội bộ, mTLS).
        
    2. **Data Ownership:** Vì DB bị tách ra, việc xác định dữ liệu nhạy cảm (PII) nằm ở đâu và ai chịu trách nhiệm bảo vệ nó trở nên phức tạp hơn. Cần một "data catalog" rõ ràng.
        

---

#### 10. API Gateway

- Cách tư duy (Thinking Process):
    
    Giờ tôi có 3 service (Users, Products, Orders) chạy ở 3 port/IP khác nhau. Client (Mobile App) không thể biết và gọi cả 3.
    
    Tôi cần một "mặt tiền" (Facade) duy nhất: API Gateway.
    
    - Client chỉ gọi `api.example.com`.
        
    - Gateway sẽ lo:
        
        1. **Routing:** `POST /orders` -> chuyển đến `OrderService`. `GET /users/me` -> chuyển đến `UserService`.
            
        2. **AuthN/AuthZ:** (user memory) Kiểm tra JWT token _một lần_ tại đây.
            
        3. **Rate Limiting, Logging, Caching** (chung): Xử lý tập trung.
            
- Cách trả lời (How to Answer):
    
    "Client sẽ không gọi trực tiếp các microservice. Chúng ta sẽ triển khai một API Gateway (dùng Kong, Ocelot, hoặc Spring Cloud Gateway) làm điểm vào duy nhất (single entry point). Gateway sẽ chịu trách nhiệm routing request, xác thực JWT token, và áp dụng các chính sách chung như rate limiting."
    
- Góc nhìn DevSecOps:
    
    Gateway là một điểm chốt bảo mật (security chokepoint) cực kỳ quan trọng.
    
    1. **Token Validation:** Nó _phải_ validate chữ ký, thời hạn (exp), và "issuer" (iss) của _mọi_ JWT token.
        
    2. **Scope-based AuthZ:** (user memory) Gateway không chỉ xác thực (AuthN), mà còn nên phân quyền (AuthZ) cơ bản. Ví dụ: Token này có scope `order:create` thì mới được gọi `POST /orders`.
        
    3. **Single Point of Failure (SPOF):** Nếu Gateway sập, toàn bộ hệ thống sập. Nó phải được làm HA (chạy nhiều instance + LB).
        

---

#### 11. Inter-Service Communication (Giao tiếp liên service)

- Cách tư duy (Thinking Process):
    
    Làm sao OrderService (cần tạo đơn hàng) lấy được giá sản phẩm từ ProductService?
    
    - **Lựa chọn 1: Synchronous (Đồng bộ) - REST/gRPC.**
        
        - `OrderService` gọi API (`GET /products/123`) của `ProductService` và _chờ_ kết quả trả về.
            
        - _Ưu điểm:_ Đơn giản, dễ hiểu. gRPC (dùng Protobuf) hiệu năng cao.
            
        - _Nhược điểm:_ **Khớp nối chặt (Tight Coupling)**. Nếu `ProductService` sập hoặc chậm, `OrderService` cũng sập hoặc chậm theo (cascading failure).
            
    - **Lựa chọn 2: Asynchronous (Bất đồng bộ) - Message Bus (Kafka/RabbitMQ).**
        
        - `ProductService` _phát_ (publish) mọi thay đổi về giá lên một "topic" Kafka (vd: `product-price-changes`).
            
        - `OrderService` _lắng nghe_ (subscribe) topic đó và _lưu một bản sao_ (copy) giá sản phẩm vào chính DB của nó.
            
        - Khi tạo Order, `OrderService` chỉ cần đọc DB của _chính nó_.
            
        - _Ưu điểm:_ **Khớp nối lỏng (Loose Coupling)**. `OrderService` chạy được ngay cả khi `ProductService` sập.
            
        - _Nhược điểm:_ Phức tạp (Eventual Consistency), dữ liệu bị nhân bản.
            
    - _Quy tắc của tôi:_ Ưu tiên Async. Chỉ dùng Sync khi nghiệp vụ _bắt buộc_ phải có dữ liệu real-time.
        
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ ưu tiên giao tiếp Asynchronous (Event-Driven) dùng Kafka/RabbitMQ để đảm bảo 'loose coupling' và 'resilience'. Ví dụ, ProductService sẽ phát event ProductPriceChanged và OrderService sẽ lắng nghe để cập nhật bản sao dữ liệu của mình. Chúng ta chỉ dùng giao tiếp Synchronous (ưu tiên gRPC vì hiệu năng) cho các trường hợp bắt buộc, ví dụ: gọi PaymentService để trừ tiền ngay lập tức."
    
- Góc nhìn DevSecOps:
    
    Giao tiếp nội bộ cũng phải an toàn.
    
    1. **Sync (gRPC/REST):** Phải dùng **mTLS (Mutual TLS)**. Không chỉ client xác thực server, mà _server cũng phải xác thực client_. `OrderService` phải "chứng minh" nó là `OrderService` khi gọi `ProductService`.
        
    2. **Async (Kafka):** Phải dùng ACLs của Kafka. Topic `product-price-changes` _chỉ_ `ProductService` được ghi vào, và _chỉ_ `OrderService` được đọc.
        

---

### Giai đoạn 4: Vận hành Microservices

#### 12. Data Consistency (Saga Pattern)

- Cách tư duy (Thinking Process):
    
    Vấn đề kinh điển: User đặt hàng.
    
    - `OrderService`: Tạo order (pending).
        
    - `PaymentService`: Trừ tiền.
        
    - StockService: Trừ kho.
        
        Trong Monolith, tôi bọc cả 3 bước này trong 1 DB Transaction. Nếu bước 3 (trừ kho) lỗi, cả 1 và 2 sẽ ROLLBACK.
        
        Trong Microservices, 3 service này có 3 DB riêng -> Không thể dùng DB transaction.
        
        Nếu PaymentService (bước 2) lỗi thì sao? Order đã lỡ tạo ở bước 1.
        
        Giải pháp: Saga Pattern.
        
    - Một "Saga" là một chuỗi các "local transaction" (giao dịch nội bộ).
        
    - Nếu một bước thất bại, Saga sẽ chạy các **Compensating Transactions** (giao dịch bù trừ) để "undo" các bước đã thành công.
        
    - Ví dụ: `PaymentService` lỗi -> Saga kích hoạt compensating transaction của `OrderService` -> `OrderService` cập nhật trạng thái order thành "CANCELLED".
        
- Cách trả lời (How to Answer):
    
    "Chúng ta không thể dùng 2-phase-commit (transaction phân tán) vì nó chậm và phức tạp. Thay vào đó, chúng ta sẽ quản lý tính nhất quán nghiệp vụ bằng Saga Pattern. Ví dụ, với quy trình đặt hàng, nếu PaymentService thất bại, Saga sẽ kích hoạt một 'compensating transaction' trên OrderService để hủy đơn hàng đó. Chúng ta có thể implement Saga bằng 2 cách: Choreography (dùng message bus, service tự lắng nghe) hoặc Orchestration (có 1 service 'nhạc trưởng' điều phối)."
    
- Góc nhìn DevSecOps:
    
    Saga rất khó debug và dễ bị tấn công logic.
    
    1. **Correlation ID:** (user memory) _Bắt buộc_ phải có 1 `correlation-id` duy nhất cho _toàn bộ_ chuỗi Saga (từ tạo order -> thanh toán -> kho), và ID này phải có trong _mọi_ log.
        
    2. **Idempotency:** (user memory) Worker xử lý "Hủy Order" có thể chạy 2 lần (do retry). Hành động "Hủy" phải **idempotent** (làm 1 lần hay 10 lần kết quả vẫn như nhau).
        
    3. **Security State:** Phải log _rất_ cẩn thận trạng thái của Saga. Kẻ tấn công có thể cố gắng làm cho Saga bị "kẹt" ở trạng thái có lợi (vd: đã trừ kho nhưng chưa thanh toán).
        

---

#### 13. Distributed Tracing / Logging

- Cách tư duy (Thinking Process):
    
    Một request của user (vd: GET /homepage) đi qua 5 service: API Gateway -> HomepageService -> UserService (lấy tên) -> ProductService (lấy sp hot) -> AdsService.
    
    Request này bị chậm (5 giây). Nó chậm ở đâu?
    
    - **Logging:** Tôi cần _tất cả_ log từ 5 service đó phải được đẩy về _một nơi_ (Centralized Logging), vd: **ELK Stack** (Elasticsearch, Logstash, Kibana) hoặc **Loki**. (User memory: Log cấu trúc, corr-id).
        
    - **Tracing:** Tôi cần một "dấu vết" (trace) cho thấy: Request A mất 50ms ở Gateway, 3000ms ở `HomepageService`, 200ms ở `UserService`... (Distributed Tracing), vd: **Jaeger** hoặc **Zipkin**.
        
- Cách trả lời (How to Answer):
    
    "Để có khả năng quan sát (Observability) hệ thống phân tán, chúng ta sẽ implement 'The Three Pillars':
    
    1. **Centralized Logging:** Dùng ELK hoặc Loki/Grafana. Mọi service sẽ log ở định dạng JSON (structured logging) và _phải_ chứa **Correlation ID**.
        
    2. **Distributed Tracing:** Tích hợp OpenTelemetry (hoặc Jaeger) vào mọi service để theo dõi một request đi qua các service.
        
    3. **(Metrics):** (Sẽ nói ở bước 18)."
        
- Góc nhìn DevSecOps:
    
    Log là con dao hai lưỡi.
    
    1. **Log Sanitization:** (user memory) _Tuyệt đối không log secrets_ (API keys, passwords, raw JWT tokens).
        
    2. **Masking PII:** (user memory) Log phải tự động "che" (mask) các dữ liệu nhạy cảm (vd: `email: "user@..."`, `phone: "...1234"`).
        
    3. **Log Access:** Ai được phép xem log? Cần có RBAC trên hệ thống log (Kibana/Grafana). Dev team A không được xem log của Dev team B.
        

---

#### 14. Service Discovery & Config

- Cách tư duy (Thinking Process):
    
    (Đã nói ở bước 11) OrderService muốn gọi ProductService. ProductService đang chạy 3 instance ở IP: 10.0.0.5, 10.0.0.6, 10.0.0.7.
    
    Làm sao OrderService biết các IP này? IP này có thể thay đổi bất cứ lúc nào (do K8s scale).
    
    Tôi không thể "hardcode" IP.
    
    Tôi cần một "danh bạ điện thoại" (Service Discovery) tự động cập nhật.
    
    - _Logic:_ `ProductService` instance 4 (mới) khởi động -> Nó tự "đăng ký" với "Danh bạ" (vd: **Consul**, **Eureka**, hoặc **DNS nội bộ của K8s**): "Tôi là ProductService, IP của tôi là 10.0.0.8".
        
    - `OrderService` muốn gọi -> Nó hỏi "Danh bạ": "Cho tôi 1 IP của ProductService" -> Danh bạ trả về 1 IP.
        
- Cách trả lời (How to Answer):
    
    "Chúng ta không thể hardcode địa chỉ IP. Các service sẽ tìm thấy nhau bằng Service Discovery. Nếu dùng Kubernetes, chúng ta sẽ dùng DNS nội bộ (ví dụ: http://product-service.default.svc.cluster.local). Nếu không, chúng ta có thể dùng Consul. Tương tự, Config (DB url, settings...) sẽ được quản lý tập trung tại Config Server (Spring Cloud Config) hoặc ConfigMaps/Secrets (K8s)."
    
- Góc nhìn DevSecOps:
    
    Nơi chứa "chìa khóa vương quốc".
    
    1. **Config Server Security:** Config Server (hoặc K8s Secrets) chứa _toàn bộ_ secrets (DB passwords, API keys). Nó phải được mã hóa, kiểm soát truy cập nghiêm ngặt nhất, và có **audit log** (ghi lại ai đã đọc secret nào, khi nào).
        
    2. **Service Mesh (Nâng cao):** Các công cụ như Istio/Linkerd có thể xử lý Service Discovery và mTLS (bước 11) một cách tự động, trong suốt.
        

---

#### 15. Resiliency (Circuit Breaker)

- Cách tư duy (Thinking Process):
    
    (Mở rộng của bước 11) OrderService gọi PaymentService (sync).
    
    PaymentService bị lỗi, response chậm (vd: 30 giây timeout).
    
    100 user cùng lúc đặt hàng -> 100 thread của OrderService bị "treo" để chờ PaymentService.
    
    Hậu quả: OrderService hết thread pool -> Sập.
    
    Đây là Cascading Failure (Lỗi dây chuyền).
    
    Giải pháp: Circuit Breaker (Cầu dao tự động).
    
    - _Logic:_ `OrderService` theo dõi `PaymentService`.
        
    - Nếu tỷ lệ lỗi > 50% trong 1 phút -> "Cầu dao" **MỞ (OPEN)**.
        
    - Khi MỞ: Mọi request (mới) đến `PaymentService` sẽ bị _chặn ngay lập tức_ (fail fast) mà _không cần gọi_, trả về lỗi cho user (vd: "Hệ thống thanh toán tạm thời gián đoạn, vui lòng thử lại sau 5 phút").
        
    - Sau 5 phút (ví dụ), cầu dao "NỬA MỞ" (HALF-OPEN): Cho 1 request đi qua. Nếu thành công -> ĐÓNG (CLOSE) lại (hệ thống bình thường). Nếu thất bại -> MỞ tiếp.
        
- Cách trả lời (How to Answer):
    
    "Để ngăn chặn lỗi dây chuyền (cascading failure), chúng ta sẽ implement Circuit Breaker Pattern (ví dụ: dùng thư viện Resilience4j, Hystrix). Ví dụ, khi OrderService gọi PaymentService, nếu tỷ lệ lỗi vượt ngưỡng (vd: 50%), 'cầu dao' sẽ mở. Mọi request sau đó sẽ 'fail fast' trong 5 phút, cho phép PaymentService có thời gian phục hồi và bảo vệ OrderService khỏi sập."
    
- Góc nhìn DevSecOps:
    
    Circuit Breaker là một công cụ vận hành (Ops) quan trọng.
    
    1. **Monitoring & Alerting:** Phải có dashboard (Grafana) theo dõi trạng thái của _mọi_ cầu dao. Một cầu dao MỞ (OPEN) phải là một **Cảnh báo (Alert) P1/P0** gửi ngay cho team Ops/Dev.
        
    2. **Graceful Degradation:** Khi cầu dao mở, hệ thống nên "xuống cấp" một cách duyên dáng. Thay vì trả lỗi 500, trả về một thông báo thân thiện, hoặc dữ liệu cache.
        

---

### Giai đoạn 5: DevSecOps Toàn diện

#### 16. CI/CD Pipeline & Security

- Cách tư duy (Thinking Process):
    
    (User memory: Migration qua CI, Test pass).
    
    Tôi muốn tự động hóa toàn bộ quy trình từ code -> deploy. Nhưng không chỉ deploy, mà phải an toàn.
    
    Đây là lúc "Sec" được nhúng vào "DevOps": Shift-Left Security (Dịch chuyển bảo mật sang trái). Thay vì đợi đến cuối mới test bảo mật, tôi test trong lúc build.
    
    - Pipeline (Đường ống) lý tưởng:
        
        1. Developer push code.
            
        2. CI Server (Jenkins/GitLab CI) trigger:
            
        3. `Build` (npm run build)
            
        4. `Unit Test` (jest) -> Phải pass (user memory)
            
        5. **SAST (Static Analysis):** (vd: SonarQube) Quét code tĩnh để tìm lỗi (security hotspots, bugs, code smells).
            
        6. **SCA (Software Composition):** (vd: Snyk, OWASP Dependency-Check) Quét `package.json` để tìm thư viện (npm) có lỗ hổng (vulnerabilities).
            
        7. _Gate:_ Nếu SAST/SCA tìm thấy lỗi "Critical" -> **FAIL PIPELINE**.
            
        8. `Build Docker Image`
            
        9. `Push Image` (lên Artifactory/ECR)
            
        10. `Deploy` (lên Staging env)
            
        11. **DAST (Dynamic Analysis):** (vd: OWASP ZAP) Chạy tool tự động "tấn công" (scan) app đang chạy trên Staging.
            
        12. `Deploy` (lên Production) (có thể cần duyệt thủ công).
            
        13. (User memory: `Run Migration` -> `Run Rollback script` nếu deploy fail).
            
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ xây dựng một CI/CD pipeline hoàn chỉnh, tích hợp bảo mật theo triết lý 'Shift-Left'. Pipeline sẽ bao gồm các bước:
    
    1. Build & Unit Test.
        
    2. **SAST (SonarQube)** để quét code tĩnh.
        
    3. **SCA (Snyk)** để quét lỗ hổng thư viện.
        
    4. Pipeline sẽ 'fail' nếu phát hiện lỗ hổng nghiêm trọng.
        
    5. Build Docker Image và đẩy lên registry.
        
    6. Deploy lên Staging, chạy **DAST (OWASP ZAP)**.
        
    7. Sau khi duyệt, deploy Production và chạy script migration (có cơ chế rollback)."
        
- Góc nhìn DevSecOps:
    
    Đây chính là DevSecOps. Tự động hóa việc tuân thủ bảo mật.
    
    1. **Policy as Code:** Các "gate" (ngưỡng) bảo mật (vd: "không cho phép lỗi Critical") phải được định nghĩa bằng code (vd: trong file config của SonarQube, GitLab CI).
        
    2. **Immutable Artifacts:** Docker image đã được build và scan là "bất biến". _Không_ được SSH vào container trên production để "vá" code. Muốn vá? Quay lại bước 1, sửa code, đẩy qua pipeline.
        

---

#### 17. Container & Orchestration (K8s)

- Cách tư duy (Thinking Process):
    
    Tôi có 50 microservices. Mỗi cái cần 3 instance. Làm sao tôi quản lý 150 tiến trình (process) này? Deploy thủ công? Chết.
    
    Tôi cần 2 thứ:
    
    1. **Container (Docker):** (User memory: Container hóa) Một cách "đóng gói" (package) ứng dụng (code + runtime + thư viện) thành một "cái hộp" tiêu chuẩn. "Chạy trên máy tôi" -> "Chạy trên mọi máy". (User memory: Dockerfile best practice, tối ưu và nhẹ).
        
    2. **Orchestration (Kubernetes - K8s):** Một "người điều phối" để chạy 150 cái hộp đó.
        
    
    - Tôi chỉ cần _khai báo_ (declarative) với K8s: "Tôi muốn 3 instance của `UserService` chạy".
        
    - K8s sẽ tự lo: Tìm server còn trống, deploy, load balancing (Service Discovery nội bộ), **Self-Healing** (nếu 1 instance sập, K8s tự bật cái mới), **Scaling** (tự tăng/giảm instance theo CPU/RAM).
        
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ container hóa (Docker) mọi microservice. Các Dockerfile sẽ được tối ưu (multi-stage builds) để 'nhẹ' (user memory). Toàn bộ hệ thống sẽ được deploy và điều phối (orchestrate) bằng Kubernetes (K8s). K8s sẽ giúp chúng ta tự động hóa việc deploy, scaling (dùng HPA), và self-healing, giảm tải vận hành."
    
- Góc nhìn DevSecOps:
    
    Bảo mật ở "Layer" mới: Layer hạ tầng container.
    
    1. **Image Security:** Scan Docker image (vd: Trivy) để tìm lỗ hổng OS (vd: OpenSSL cũ). _Không_ chạy container với quyền `root` (dùng `USER nonroot`).
        
    2. **K8s RBAC:** Phân quyền ai được làm gì trên K8s cluster. Dev team A chỉ được deploy vào namespace "A".
        
    3. **Network Policies:** (Rất quan trọng) Đây là "firewall" bên trong K8s. Tôi sẽ tạo Policy quy định: `OrderService` _chỉ_ được phép nói chuyện với `PaymentService`, _cấm_ nói chuyện với `UserService`. Đây là cách thực thi **Zero Trust** ở mức mạng.
        

---

#### 18. Monitoring & Alerting

- Cách tư duy (Thinking Process):
    
    (Nối tiếp bước 13). Hệ thống đang chạy trên K8s. Làm sao tôi biết nó "khỏe"?
    
    Tôi cần "The Three Pillars of Observability" (3 trụ cột của khả năng quan sát):
    
    1. **Logs:** (Đã nói ở bước 13 - Loki) Cho biết _cái gì_ đã xảy ra.
        
    2. **Traces:** (Đã nói ở bước 13 - Jaeger) Cho biết _ở đâu_ và _tại sao_ (nó chậm).
        
    3. **Metrics:** (Trụ cột mới) Các con số theo thời gian.
        
    
    - Tôi sẽ dùng **Prometheus** (chuẩn de-facto). Mọi service sẽ "phơi" (expose) ra metrics (vd: `/metrics`) như: số request/giây, tỷ lệ lỗi 5xx, latency P99, CPU/RAM usage.
        
    - Prometheus sẽ "cào" (scrape) các metrics này mỗi 15 giây.
        
    - **Grafana** sẽ đọc từ Prometheus/Loki/Jaeger để vẽ dashboard đẹp đẽ.
        
    - **AlertManager** (đi chung với Prometheus) sẽ gửi cảnh báo (qua Slack/PagerDuty) khi metrics vượt ngưỡng (vd: `tỷ lệ lỗi 5xx > 1% trong 5 phút`).
        
- Cách trả lời (How to Answer):
    
    "Chúng ta sẽ hoàn thiện 'Observability Stack' bằng cách thêm Prometheus cho
    
    Metrics và Grafana cho Dashboard. Mọi service sẽ expose metrics (theo chuẩn OpenTelemetry) về latency, error rate, và saturation. AlertManager sẽ được cấu hình để gửi cảnh báo P1 (vd: site down) hoặc P2 (vd: latency tăng) cho team trực."
    
- Góc nhìn DevSecOps:
    
    Đây cũng là Security Monitoring.
    
    1. **Security Metrics:** Tôi sẽ tạo metrics/alerts cho các sự kiện bảo mật. Ví dụ: Alert nếu `số lỗi 401/403 (Unauthorized) > 100/phút` (dấu hiệu brute-force). Alert nếu `số tài khoản đăng ký mới > 1000/giờ` (dấu hiệu spam bot).
        
    2. **Audit Logs:** K8s API server, DB... tất cả phải có audit log (ai đã làm gì) và các log này cũng phải được đẩy về hệ thống logging tập trung để phân tích (vd: phát hiện admin cố tình xóa log).
        
