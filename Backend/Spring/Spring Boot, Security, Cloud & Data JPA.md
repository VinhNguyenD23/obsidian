## 1. Spring Boot

### 1.1 Cấp độ Dễ (Foundation: The "What" and "Why")

Phần này giới thiệu các khái niệm cốt lõi tuyệt đối của Spring Boot, giúp lập trình viên hiểu _tại sao_ Spring Boot tồn tại và các thành phần cơ bản nhất của nó.

#### 1.1.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Spring Boot không phải là một framework mới. Nó là một _phần mở rộng_ (extension) của Spring Framework, được thiết kế để _tăng tốc_ (accelerate) quá trình xây dựng ứng dụng. Nó cung cấp một bộ "ý kiến" (opinionated) mạnh mẽ về cách cấu hình, giúp giảm thiểu công sức thiết lập ban đầu.   
    
- **Mục đích:** Mục tiêu chính là loại bỏ "địa ngục cấu hình" (configuration hell), đặc biệt là các tệp XML phức tạp của Spring truyền thống. Spring Boot giúp lập trình viên tập trung vào _logic nghiệp vụ_ (business logic) thay vì _cấu hình cơ sở hạ tầng_ (infrastructure configuration).   
    

#### 1.1.2 Các thành phần/khái niệm quan trọng

- **Starter Dependencies:** Đây là các tệp `pom.xml` (hoặc `build.gradle`) được đóng gói sẵn, hoạt động như các "bộ dụng cụ" (toolkit).   
    
    - Ví dụ: `spring-boot-starter-web`  không chỉ thêm Spring MVC, mà còn _tự động_ thêm Tomcat  làm máy chủ nhúng và thư viện Jackson (JSON) để xử lý REST.   
        
    - Lợi ích: Chúng quản lý "dependency management" , đảm bảo các phiên bản thư viện _tương thích_ (compatible) với nhau.   
        
- **Auto-Configuration (Tự động cấu hình):** Đây là "phép thuật" chính của Spring Boot. Spring Boot "đoán" (intelligently guesses) các cấu hình (Beans) bạn cần dựa trên các `jar` file (dependencies) bạn đã thêm vào classpath.   
    
    - Ví dụ: Nếu Spring Boot thấy `spring-boot-starter-data-jpa` và `h2.jar` trên classpath, nó _tự động_ cấu hình một `DataSource` và một `EntityManagerFactory` kết nối tới H2 in-memory mà không cần bạn viết bất kỳ dòng code cấu hình nào.
        
- **Embedded Servers (Máy chủ nhúng):** Ứng dụng Spring Boot không cần cài đặt và cấu hình Tomcat/Jetty/Undertow riêng biệt. Ứng dụng Spring Boot tự nó là một tệp `.jar` có thể chạy được (runnable JAR / "fat JAR"), bên trong đã chứa máy chủ.   
    

#### 1.1.3 Best-practice / Lưu ý doanh nghiệp

- **Clean Code (Cấu trúc):** Doanh nghiệp _phải_ tuân thủ cấu trúc package chuẩn: `com.company.project.controller`, `com.company.project.service`, `com.company.project.repository`, `com.company.project.config`, `com.company.project.dto` để dễ dàng bảo trì.
    
- **Clean Code (Annotation):** Đặt annotation `@SpringBootApplication` ở package gốc (root package). Mọi thứ sẽ tự động được quét (component scan) từ đó trở xuống.
    
- **Vận hành:** Quản lý tất cả cấu hình ứng dụng trong `application.properties` hoặc (ưu tiên hơn) `application.yml`. `application.yml` được ưu tiên vì nó có cấu trúc (hierarchical) và ít lặp lại (less verbose).   
    
- **Bảo mật:** Không bao giờ hardcode-credentials (mật khẩu, API keys) trong code. Sử dụng `application.properties`  (và ở cấp độ cao hơn là biến môi trường).   
    

#### 1.1.4 Ví dụ ngắn

- **Code mẫu (Hello World REST Controller):**
    

Java

```JAVA
@RestController // Kết hợp @Controller và @ResponseBody
public class HelloController {
    @GetMapping("/hello") // Xử lý GET request
    public String hello() {
        return "Hello, World!";
    }
}
```

- **Cấu hình (`application.yml`):**
    

YAML

```yaml
server:
  port: 8080 # Cấu hình cổng máy chủ nhúng 
```

Việc Spring Boot sử dụng "opinionated defaults" (các mặc định có chủ ý)  là một công cụ mạnh mẽ nhưng cũng là con dao hai lưỡi. Nó giúp 80% các trường hợp thông thường được triển khai gần như ngay lập tức, tăng năng suất cho Junior developer. Tuy nhiên, đối với 20% trường hợp doanh nghiệp cần tùy biến (ví dụ: đổi Tomcat sang Undertow, hoặc tắt một Auto-Configuration  cụ thể), developer cần phải hiểu rõ cách "ghi đè" (override) và "loại trừ" (exclude) các cấu hình tự động đó.   

Sự trỗi dậy của "Runnable JAR"  không chỉ là về sự tiện lợi. Nó là _yếu tố nền tảng_ (enabler) cho kiến trúc Microservices và Cloud-Native. Một "runnable JAR" là một đơn vị độc lập, chứa mọi thứ nó cần (code + dependencies + server). Điều này làm cho việc đóng gói vào một Docker container trở nên tầm thường (chỉ cần `java -jar app.jar`). Kết quả là, Spring Boot đã thúc đẩy mạnh mẽ việc áp dụng Docker và Kubernetes, thay đổi văn hóa từ Ops-centric sang DevOps bằng cách trao cho Developer khả năng kiểm soát môi trường runtime.   

### 1.2 Cấp độ Trung (Configuration & Web)

Phần này tập trung vào việc quản lý cấu hình ứng dụng một cách chuyên nghiệp theo môi trường và xây dựng các Web API (REST) bài bản, bao gồm cả xử lý lỗi.

#### 1.2.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Quản lý các cấu hình thay đổi theo môi trường (ví dụ: CSDL ở `dev` khác `prod`) và xây dựng các Web API (REST) một cách chuyên nghiệp.
    
- **Mục đích:** Tách biệt code (thứ không thay đổi) ra khỏi cấu hình (thứ thay đổi liên tục). Xử lý lỗi và binding dữ liệu một cách nhất quán.   
    

#### 1.2.2 Các thành phần/khái niệm quan trọng

- **Spring Boot Profiles:** Cho phép định nghĩa các tệp cấu hình riêng biệt (`application-dev.yml`, `application-prod.yml`). Ứng dụng sẽ nạp tệp `application.yml` chung, sau đó nạp tệp profile cụ thể (được kích hoạt qua `spring.profiles.active`) và ghi đè các giá trị.   
    
- **`@ConfigurationProperties`:** Một cách "type-safe" (an toàn kiểu) để bind các thuộc tính trong `application.yml` vào một Java class (POJO).   
    
- **Exception Handling (Xử lý ngoại lệ):** Sử dụng `@RestControllerAdvice` và `@ExceptionHandler` để bắt (intercept) các ngoại lệ (ví dụ: `UserNotFoundException`) và trả về một JSON response (ví dụ: `ErrorMessage`) nhất quán.   
    
- **Logging:** Spring Boot dùng Logback làm default. Cấu hình logging level (INFO, DEBUG, TRACE)  cho các môi trường khác nhau trong tệp `application-{profile}.yml`.   
    

#### 1.2.3 Best-practice / Lưu ý doanh nghiệp

- **Bảo mật:** _Không bao giờ_ (NEVER) hardcode-credentials (database password, API keys) vào `application.yml`, kể cả `application-prod.yml`. Tệp này nằm trong Git.   
    
- **Bảo mật (Tiếp theo):** Sử dụng biến môi trường (Environment Variables) hoặc Docker/Kubernetes Secrets để inject các giá trị nhạy cảm này lúc runtime (ví dụ: `spring.datasource.password=${DB_PASSWORD}`). Spring Boot sẽ tự động đọc biến môi trường và ghi đè lên tệp `yml`.
    
- **Vận hành:** Profile `prod` _phải_ set logging level là `INFO` hoặc `WARN`. `DEBUG` hoặc `TRACE` ở production sẽ giết chết hiệu năng I/O và làm tốn chi phí logging.   
    
- **Clean Code:** Ưu tiên `@ConfigurationProperties`  hơn là dùng `@Value` lẻ tẻ. `@ConfigurationProperties` nhóm các thuộc tính liên quan lại (ví dụ: `app.jwt.secret`, `app.jwt.expiration`), hỗ trợ validation, và dễ dàng cho IDE auto-complete.   
    
- **Clean Code (API):** Luôn sử dụng DTOs (Data Transfer Objects) cho input (ví dụ: `@RequestBody UserDTO`) và output (ví dụ: `ResponseEntity<UserDTO>`). _Không_ bao giờ expose Entity (xem 4.1) ra API.
    

#### 1.2.4 Ví dụ ngắn

- Code mẫu (@ConfigurationProperties - ):   
    

Java

```java
@ConfigurationProperties(prefix = "app.jwt") // Bind với các key "app.jwt.*"
@Configuration
@Validated // Bật validation
public class JwtConfig {
    @NotBlank
    private String secret;
    @Min(300000) // Tối thiểu 5 phút
    private long expirationMs;
    
    // Getters and setters
}
```

- Code mẫu (@RestControllerAdvice - ):   
    

Java

```java
@RestControllerAdvice // 
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(ex.getMessage(), "NOT_FOUND");
    }
}
```

- Cấu hình (Profiles - ):   
    

YAML

```yaml
# application.yml (chung)
spring:
  application:
    name: "my-app"

--- # Phân tách profile
# application-dev.yml (hoặc đặt trong cùng tệp)
spring:
  config:
    activate:
      on-profile: "dev" # 
  datasource:
    url: "jdbc:h2:mem:testdb"
  logging:
    level:
      org.springframework: "DEBUG" # 

---
# application-prod.yml
spring:
  config:
    activate:
      on-profile: "prod"
  datasource:
    url: "${DB_PROD_URL}" # Lấy từ biến môi trường
  logging:
    level:
      org.springframework: "INFO" # 
```

Sự phổ biến của `@RestControllerAdvice`  là hệ quả trực tiếp của việc Spring Boot được dùng để xây dựng REST API. Trong Spring MVC truyền thống (phục vụ web page), việc xử lý lỗi là trả về một trang 500.html. Trong Spring Boot (phục vụ REST API ), client (React/Angular) cần một JSON response có cấu trúc để biết lỗi là gì. `@RestControllerAdvice`  ra đời để "dịch" (translate) tất cả exception sang một DTO (ví dụ: `ErrorResponse`) chung, thúc đẩy một chuẩn về Error Response trong doanh nghiệp, giúp frontend và các service khác xử lý lỗi nhất quán.   

### 1.3 Cấp độ Khó (Operations & Customization)

Phần này đi sâu vào "bên trong" Spring Boot, dành cho các developer (Mid/Senior) cần _vận hành_ (operate) ứng dụng và _tùy biến_ (customize) các hành vi mặc định.

#### 1.3.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Giám sát (monitor) , quản lý (manage) trạng thái ứng dụng lúc đang chạy, và tùy biến sâu các "ý kiến" (opinions)  của Spring Boot.   
    
- **Mục đích:** (1) Trả lời câu hỏi: "Ứng dụng của tôi có đang 'khỏe' không?". (2) Đóng gói các cấu hình chung (cross-cutting concerns) của doanh nghiệp (ví dụ: logging, security) để tái sử dụng.   
    

#### 1.3.2 Các thành phần/khái niệm quan trọng

- **Spring Boot Actuator:** Một "starter" (`spring-boot-starter-actuator`)  cung cấp các endpoint "production-ready" (sẵn sàng cho sản xuất)  để giám sát và quản lý ứng dụng.   
    
- **Actuator Endpoints (Các endpoint chính):**    
    
    - `/actuator/health`:  Endpoint quan trọng nhất. Cung cấp trạng thái `UP`/`DOWN`. Tích hợp với nhiều thành phần (DB, Disk, RabbitMQ) để cung cấp "health" tổng thể.   
        
    - `/actuator/metrics`:  Cung cấp danh sách các metric chi tiết (ví dụ: `jvm.memory.used`, `http.server.requests`, `system.cpu.usage`).   
        
    - `/actuator/prometheus`:  Expose metrics theo định dạng mà Prometheus (xem 1.4) có thể đọc (scrape).   
        
    - Các endpoint nhạy cảm: `/actuator/env` (xem cấu hình), `/actuator/heapdump` (dump bộ nhớ).
        
- **Custom Auto-Configuration:** Khả năng tự viết các class `@Configuration` và đăng ký chúng qua `META-INF/spring.factories` (Boot 2)  hoặc `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 3+).   
    
- **Custom Starters:** Đóng gói một (hoặc nhiều) auto-configuration  và các dependency liên quan vào một tệp `.jar` duy nhất.   
    

#### 1.3.3 Best-practice / Lưu ý doanh nghiệp

- **Bảo mật:** _Cực kỳ quan trọng._ Các endpoint của Actuator  phơi bày thông tin _cực kỳ_ nhạy cảm. _Luôn luôn_ bảo vệ chúng.   
    
    1. Thêm `spring-boot-starter-security` vào `pom.xml`.
        
    2. Trong `application.yml`, _chỉ_ expose các endpoint cần thiết công khai (ví dụ: `/health`).   
        
    3. Yêu cầu xác thực (ví dụ: role `ACTUATOR_ADMIN`) cho tất cả các endpoint còn lại (đặc biệt là `env` và `heapdump`).
        
- **Vận hành:** Trong Kubernetes, `/actuator/health`  là _cách thức tiêu chuẩn_ (standard way) để cấu hình `livenessProbe` và `readinessProbe`.   
    
    - `livenessProbe` (dùng `/actuator/health`): Nếu fail (trả về `DOWN`), Kubernetes sẽ _restart_ container (ứng dụng chết).
        
    - `readinessProbe` (dùng `/actuator/health/readiness`): Nếu fail, Kubernetes sẽ _ngừng gửi traffic_ mới đến (ứng dụng đang bận, ví dụ: khởi động, đang xử lý tác vụ nặng).
        
- **Clean Code:** Nếu doanh nghiệp của bạn có 10+ microservices, _hãy_ (DO) tạo custom starter. Đóng gói các cấu hình chung (ví dụ: cấu hình logging chuẩn, cấu hình tracing, cấu hình security , cấu hình `@RestControllerAdvice`  chung) vào `my-company-starter`. Điều này đảm bảo tính nhất quán và giảm lặp lại code.   
    

#### 1.3.4 Ví dụ ngắn

- Cấu hình (Actuator Security - ):   
    

YAML

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,metrics,prometheus" # Chỉ expose 3 endpoint này 
        exclude: "env,heapdump,beans" # Không bao giờ expose env, heapdump
  endpoint:
    health:
      show-details: "when-authorized" # Chỉ show chi tiết (DB status) khi đã xác thực
```

- Code mẫu (Custom Auto-Configuration - ):   
    

Java

```java
// Trong tệp: src/main/resources/META-INF/spring.factories 
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.mycompany.config.MyCustomAutoConfiguration

// Tệp Java
@Configuration
@ConditionalOnClass(MyService.class) // Chỉ kích hoạt khi có class MyService trên classpath
public class MyCustomAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean // Chỉ tạo bean nếu chưa tồn tại
    public MyService myService() {
        // Bean này sẽ được tự động cấu hình cho bất kỳ ai
        // thêm "my-company-starter" 
        return new MyService();
    }
}
```

Actuator  chính là _cầu nối_ (bridge) giữa Developer (người viết code) và DevOps/SRE (người vận hành). Nó cung cấp một API (hợp đồng) (ví dụ: `/health`, `/metrics`)  mà cả hai bên đều hiểu. DevOps có thể tích hợp API này vào hệ thống của họ (Kubernetes , Prometheus ) mà không cần hiểu code bên trong. Điều này thúc đẩy văn hóa DevOps, phá vỡ bức tường "it works on my machine" bằng cách cung cấp một "hợp đồng" chuẩn về sức khỏe ứng dụng, cho phép tự động hóa vận hành.   

### 1.4 Cấp độ Chuyên sâu (Cloud-Native Monitoring & Observability)

Phần này xây dựng trên Cấp độ 1.3, đi sâu vào việc _thu thập, lưu trữ_ và _trực quan hóa_ các metrics  mà Actuator  cung cấp, trong bối cảnh Cloud-Native.   

#### 1.4.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Chuyển từ "Monitoring" (Giám sát - Tôi biết hệ thống down khi nó báo lỗi) sang "Observability" (Khả năng quan sát - Tôi có thể hỏi _bất kỳ câu hỏi nào_ về hệ thống và có câu trả lời, ví dụ: _tại sao_ nó chậm?).   
    
- **Mục đích:** Cung cấp khả năng quan sát sâu  vào các hệ thống phân tán (microservices), nơi lỗi có thể xảy ra ở bất kỳ đâu và khó chẩn đoán.   
    

#### 1.4.2 Các thành phần/khái niệm quan trọng

- **Ba trụ cột của Observability (The 3 Pillars):**    
    
    1. **Metrics (Số liệu):**  Các con số theo thời gian (ví dụ: CPU, memory, request/giây). Được cung cấp bởi Micrometer.   
        
    2. **Logging (Nhật ký):**  Các sự kiện (event) rời rạc (ví dụ: "User X logged in"). (Ví dụ: ELK Stack, Splunk ).   
        
    3. **Tracing (Truy vết):**  Theo dõi một request qua nhiều microservices  (Xem 3.4).   
        
- **Micrometer:**  Một _facade_ (lớp giao diện) cho metrics trong Spring Boot. Giống như SLF4J cho logging, Micrometer  cho phép bạn _viết code_ metrics một lần , và _xuất_ (export) ra nhiều hệ thống: Prometheus, Datadog, New Relic....   
    
- **Prometheus:**  Hệ thống Time-Series Database (TSDB) và hệ thống alerting (cảnh báo)  hàng đầu. Nó hoạt động theo mô hình "pull" (chủ động "cào" - scrape - metrics  từ endpoint `/actuator/prometheus`  của các ứng dụng).   
    
- **Grafana:**  Công cụ trực quan hóa (visualization). Nó truy vấn dữ liệu từ Prometheus  (hoặc nguồn khác) và vẽ các dashboard đẹp, tương tác.   
    

#### 1.4.3 Best-practice / Lưu ý doanh nghiệp

- **Hiệu năng:**  Không chỉ giám sát _System Metrics_ (CPU, Disk)  và _JVM Metrics_ (Heap, GC, Threads) , mà quan trọng nhất là: _Application-Level/Business Metrics_ (Số liệu cấp ứng dụng/nghiệp vụ)  (ví dụ: số lượng đơn hàng đã tạo, tỷ lệ lỗi thanh toán). Hãy tạo custom metrics  bằng `MeterRegistry` của Micrometer.   
    
- **Vận hành:**  Thiết lập _Alerting_ (ví dụ: dùng AlertManager  của Prometheus). Dashboard đẹp  là vô dụng nếu không ai nhìn. Alerting sẽ chủ động thông báo (ví dụ: qua Slack, PagerDuty) khi có sự cố (ví dụ: error rate > 5%, p95 latency > 1s).   
    
- **Vận hành:**  Xác định các _Key Metrics_ (Chỉ số quan trọng)  cho từng service, ví dụ:   
    
    - `jvm.memory.max` / `jvm.memory.used`  (Phát hiện Memory Leaks).   
        
    - `process.cpu.usage`  (Phát hiện bottleneck CPU).   
        
    - `http.server.requests` (count, p95, max)  (Đo lường throughput và latency).   
        
    - `logback.events` (count by level=ERROR)  (Đếm số lỗi).   
        
- **Bảo mật:**  Endpoint `/actuator/prometheus`  phải được bảo vệ. Trong Kubernetes, thường chỉ cho phép "scrape" từ namespace/IP của Prometheus server, không public ra ngoài.   
    

#### 1.4.4 Khi triển khai thực tế (Microservices/Cloud-Native)

- **Monitoring Stack:** Bộ ba "Prometheus + Grafana + AlertManager"  là stack giám sát tiêu chuẩn trong Kubernetes. Thường được cài đặt qua Helm Chart.   
    
- **Service Discovery:** Prometheus được cấu hình để _tự động phát hiện_ (auto-discover)  các Spring Boot application (thông qua Kubernetes Service/Pod annotations) và tự động "scrape" metrics từ chúng. Không cần cấu hình IP bằng tay.   
    
- **Visualization:** Xây dựng (hoặc import) các dashboard Grafana  chuẩn cho JVM (Micrometer cung cấp sẵn), và các dashboard riêng cho từng service, hiển thị các "Golden Signals" (Latency, Traffic, Errors, Saturation).   
    

#### 1.4.5 Ví dụ ngắn

- Cấu hình (application.yml - ):   
    

YAML

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,prometheus" # 
  metrics:
    tags:
      # Gắn tag này vào TẤT CẢ metrics, giúp lọc trên Grafana
      application: "${spring.application.name}" 
```

- Code mẫu (Custom Metric - Micrometer - ):   
    

Java

```java
@Service
public class OrderService {
    private final Counter orderCounter;
    private final Timer orderProcessTimer; // 

    public OrderService(MeterRegistry registry) { // 
        // Định nghĩa 1 custom metric 
        this.orderCounter = registry.counter("orders.placed", "type", "online");
        this.orderProcessTimer = registry.timer("orders.processing.time"); // 
    }

    public void placeOrder() {
        //  Bắt đầu bấm giờ
        this.orderProcessTimer.record(() -> {
            //... logic xử lý đơn hàng
            this.orderCounter.increment(); // Tăng giá trị metric
        });
    }
}
```

Micrometer  là một mảnh ghép chiến lược, đảm bảo Spring _không bị trói buộc_ (vendor lock-in) vào một nhà cung cấp monitoring nào. Đây là một "vendor-neutral approach". Nó cho phép doanh nghiệp tự do chuyển đổi hệ thống monitoring (ví dụ: từ Prometheus  sang Datadog ) mà _không cần thay đổi một dòng code nghiệp vụ nào_.   

Hơn nữa, xu hướng mới nổi của OpenTelemetry (OTel)  đang thống nhất "3 trụ cột"  (Metrics, Logs, Traces) vốn trước đây rời rạc. Spring Boot 3+ (thông qua Micrometer Tracing ) đang dịch chuyển mạnh mẽ sang OTel , cho phép sự _liên kết_ (correlation) mạnh mẽ giữa metrics  và traces  trong tương lai.   

---

## 2. Spring Security

### 2.1 Cấp độ Dễ (Core Concepts)

Phần này giới thiệu các khái niệm _cơ bản_ và _bắt buộc_ về bảo mật, giúp Junior developer hiểu các khối xây dựng (building blocks) của Spring Security.

#### 2.1.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Một framework mạnh mẽ và có khả năng tùy biến cao, cung cấp cả hai tính năng _xác thực_ (Authentication) và _phân quyền_ (Authorization) cho ứng dụng Java.   
    
- **Mục đích:** Bảo vệ tài nguyên (endpoints, methods) của ứng dụng, đảm bảo chỉ "đúng người" (xác thực) mới được vào, và họ chỉ được "làm đúng việc" (phân quyền).
    

#### 2.1.2 Các thành phần/khái niệm quan trọng

- **Authentication (Xác thực):** "Bạn là ai?". Quá trình xác minh danh tính, thường qua username/password , token, hoặc SSO.   
    
- **Authorization (Phân quyền):** "Bạn được phép làm gì?". Quá trình kiểm tra xem một người dùng (đã được xác thực) có quyền (ví dụ: `ROLE_ADMIN` hay `ROLE_USER`)  để truy cập một tài nguyên hay không.   
    
- **`SecurityFilterChain`:** Khái niệm _trung tâm_ trong Spring Security hiện đại (thay thế `WebSecurityConfigurerAdapter` đã deprecated). Đây là một chuỗi các filter (giống như các chốt chặn). Mọi request _phải_ đi qua chuỗi này. Ta cấu hình chuỗi này để bảo vệ các endpoint.   
    
- **`UserDetailsService`:** Một interface. Nhiệm vụ của nó rất đơn giản: nhận vào một `username` và trả về một đối tượng `UserDetails` (chứa password, roles...). Đây là cầu nối giữa Spring Security và nơi lưu trữ user (ví dụ: In-memory , Database ).   
    
- **`PasswordEncoder`:** Một interface. Nhiệm vụ: (1) Băm (hash) mật khẩu khi đăng ký (2) So sánh mật khẩu (plain text) người dùng nhập với mật khẩu đã băm.   
    

#### 2.1.3 Best-practice / Lưu ý doanh nghiệp

- **Bảo mật:** _Bắt buộc_ (Mandatory). Luôn luôn định nghĩa một `@Bean PasswordEncoder`.   
    
- **Bảo mật:** _Không bao giờ_ (NEVER) dùng `{noop}`  (no-op, không mã hóa)  trong production. `{noop}`  chỉ để test. _Không bao giờ_ lưu mật khẩu dạng plain text.   
    
- **Bảo mật:** Sử dụng các thuật toán hashing mạnh và thích ứng (adaptive) như `BCryptPasswordEncoder`. Tránh xa MD5, SHA-1 (đã lỗi thời).   
    
- **Clean Code:** Spring Security đã _loại bỏ_ (deprecated) `WebSecurityConfigurerAdapter`. Doanh nghiệp _phải_ chuyển sang cấu hình dựa trên Java Configuration và `@Bean SecurityFilterChain`.   
    

#### 2.1.4 Ví dụ ngắn

- Code mẫu (SecurityConfig cơ bản - ):   
    

Java

```java
@Configuration
@EnableWebSecurity // Bật Spring Security
public class SecurityConfig {

    //  Bắt buộc phải có Bean này
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); 
    }

    //  Cung cấp user từ Database
    @Bean
    public UserDetailsService userDetailsService(UserRepository userRepository) {
        //  Đây là implementation của UserRepoUserDetailsService
        return username -> {
            User user = userRepository.findByUsername(username); // 
            if (user == null) {
                throw new UsernameNotFoundException("User not found"); // 
            }
            return user; // User entity phải implement UserDetails
        };
    }
    
    //  Cấu hình chuỗi filter
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
           .authorizeHttpRequests(authz -> authz
               .requestMatchers("/public/**", "/login").permitAll() // Cho phép /public
               .requestMatchers("/admin/**").hasRole("ADMIN") //  Chỉ ADMIN
               .anyRequest().authenticated() // Mọi request khác phải xác thực
            )
           .formLogin(withDefaults()); // Bật form login mặc định
        return http.build();
    }
}
```

Việc Spring Security _bắt buộc_ (default expects) có `PasswordEncoder`  là một _bài học đắt giá_ (hard-learned lesson) từ quá khứ của ngành phần mềm. Spring Security (từ version 5) đã đưa ra một "ý kiến" mạnh mẽ: "Nếu bạn không cung cấp một `@Bean PasswordEncoder` , ứng dụng sẽ crash". Điều này _ép buộc_ (forces) best practice về bảo mật  ngay từ Cấp độ Dễ, ngăn chặn lỗ hổng lưu mật khẩu plain text ngay cả khi developer không hiểu rõ về hashing.   

### 2.2 Cấp độ Trung (Web Security)

Phần này tập trung vào các tính năng bảo mật _cụ thể_ cho ứng dụng web (cả truyền thống và REST API) và cách bảo vệ ở cấp độ method.

#### 2.2.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Bảo vệ các ứng dụng web truyền thống (dùng session)  và các REST API hiện đại (stateless).   
    
- **Mục đích:** Xử lý các hình thức tấn công web phổ biến (CSRF , XSS) và quản lý cách các service (từ các domain khác nhau) giao tiếp với nhau (CORS ).   
    

#### 2.2.2 Các thành phần/khái niệm quan trọng

- **Session Management (Quản lý phiên):**  Cách Spring Security tạo và quản lý session.   
    
    - `STATEFUL` (default): Tạo session (JSESSIONID cookie) cho mỗi user. Dùng cho web truyền thống.
        
    - `STATELESS`:  Không tạo hoặc sử dụng session. _Bắt buộc_ cho REST API dùng JWT (xem 2.3).   
        
- **CSRF (Cross-Site Request Forgery):** Một loại tấn công mà kẻ xấu "lừa" trình duyệt của người dùng (đã đăng nhập) gửi một request (ví dụ: `POST /transfer?amount=1000`) đến ứng dụng mà người dùng không biết. Spring Security bật tính năng này (yêu cầu CSRF token ) mặc định để chống lại điều này.   
    
- **CORS (Cross-Origin Resource Sharing):** Một cơ chế _của trình duyệt_, ngăn chặn trang web `a.com` gọi API ở `b.com`. Ta _phải_ cấu hình CORS ở phía backend (`b.com`) để "cho phép" (allow) `a.com`.   
    
- **Method-Level Security:** Bảo vệ ở tầng service, _sau khi_ đã qua tầng web.   
    
    - `@EnableMethodSecurity` (thay thế `@EnableGlobalMethodSecurity` ).   
        
    - `@PreAuthorize("hasRole('ADMIN')")`:  Kiểm tra quyền _trước khi_ chạy method.   
        
    - `@PostAuthorize`: Kiểm tra quyền _sau khi_ chạy (ít dùng).
        

#### 2.2.3 Best-practice / Lưu ý doanh nghiệp

- **Bảo mật (Kiến trúc):** Phân biệt rõ hai luồng cấu hình:
    
    - **Web App truyền thống (Stateful):** Bật `formLogin()`, bật `csrf()` , dùng `sessionManagement(STATEFUL)`.   
        
    - **REST API (Stateless - phục vụ SPA/Mobile):** Tắt `formLogin()`, _bắt buộc_ `csrf(csrf -> csrf.disable())` , _bắt buộc_ dùng `sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))`.   
        
- **Bảo mật (CSRF):** Tại sao tắt CSRF  cho REST API? Vì REST API không dùng session/cookie (JSESSIONID) để xác thực, mà dùng `Authorization: Bearer` header. Tấn công CSRF  dựa trên việc trình duyệt _tự động_ gửi cookie. Nếu API của bạn không dựa vào cookie (stateless), tấn công CSRF là vô hiệu.   
    
- **Bảo mật (CORS):**  _Không bao giờ_ (NEVER) dùng `allow-origins="*"`. Đây là một lỗ hổng bảo mật. Hãy chỉ định rõ ràng các domain frontend (ví dụ: `https://my-app.com`).   
    
- **Clean Code (Defense-in-Depth):**  Luôn kết hợp bảo vệ ở 2 lớp:   
    
    1. Web Layer (`SecurityFilterChain`): Bảo vệ "thô" (ví dụ: `requestMatchers("/admin/**").hasRole("ADMIN")`).   
        
    2. Service Layer (`@PreAuthorize`):  Bảo vệ "tinh" (fine-grained) (ví dụ: `hasRole('USER') and @bean.isOwner(authentication, #invoiceId)`).   
        

#### 2.2.4 Ví dụ ngắn

- Code mẫu (Cấu hình cho REST API STATELESS - ):   
    

Java

```java
@Configuration
@EnableMethodSecurity //  Bật @PreAuthorize
public class RestApiSecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
           .cors(cors -> cors.configurationSource(myCorsConfigSource())) // 
           .csrf(csrf -> csrf.disable()) //  Tắt CSRF cho REST API
           .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS) //  Không tạo session
            )
           .authorizeHttpRequests(authz -> authz
               .anyRequest().authenticated()
            );
        //... Thêm filter JWT (xem mục 2.3)
        return http.build();
    }

    //  Cấu hình CORS chặt chẽ
    CorsConfigurationSource myCorsConfigSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(Arrays.asList("https://my-frontend.com"));
        config.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(Arrays.asList("Authorization", "Content-Type"));
        config.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}

@Service
public class MyService {
    @PreAuthorize("hasRole('ADMIN')") // 
    public void doAdminTask() {... }
}
```

Sự tồn tại của `@PreAuthorize`  là sự thừa nhận rằng bảo vệ ở tầng Web (`requestMatchers`) là _không đủ_ (insufficient). Nếu một `ReportService` (mà ai cũng gọi được) lại `@Autowired` và gọi nội bộ `UserService.deleteUser()`, thì cuộc gọi này _bỏ qua_ (bypass) `SecurityFilterChain`. `@PreAuthorize`  bảo vệ _trực tiếp_ method, bất kể ai gọi nó (từ web hay từ service khác).   

### 2.3 Cấp độ Khó (Token-Based: JWT)

Phần này tập trung vào _cách thức_ (how-to) triển khai xác thực _phổ biến nhất_ cho microservices và SPA: JSON Web Tokens (JWT).

#### 2.3.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** JSON Web Token (JWT)  là một tiêu chuẩn mở (RFC 7519) định nghĩa một cách "nhỏ gọn" (compact) và "tự chứa" (self-contained) để truyền thông tin (dưới dạng JSON). Nó được "ký" (signed) bằng một `secret`  để đảm bảo tính toàn vẹn (không bị sửa đổi).   
    
- **Mục đích:** Sử dụng làm "vé" (access token) cho các ứng dụng `STATELESS`  (xem 2.2). Client (SPA, mobile) gửi JWT  này trong mỗi request (thường ở `Authorization: Bearer <token>` header ) để chứng minh danh tính. Server _không cần_ lưu trữ session , chỉ cần xác thực chữ ký (signature) của token.   
    

#### 2.3.2 Các thành phần/khái niệm quan trọng

- **JWT Structure (Header, Payload, Signature):**
    
    - Payload (Claims): Thông tin về user (ví dụ: `sub` (subject/username) , `roles`, `iat` (issued at) , `exp` (expiration) ).   
        
    - Signature: Chữ ký điện tử. Đây là phần quan trọng nhất để xác thực.   
        
- **`JwtTokenProvider` / `JwtUtils`:**  Một class "helper" (thường là tự viết) sử dụng thư viện (ví dụ: `io.jsonwebtoken/jjwt` ) để: (1) Tạo (generate) token , (2) Xác thực (validate) token , (3) Trích xuất (parse) thông tin (ví dụ: username) từ token.   
    
- **Custom Filter (`OncePerRequestFilter`):**  Một filter tùy chỉnh, được chèn vào `SecurityFilterChain`. Nhiệm vụ: (1) Đọc `Authorization: Bearer <token>`  từ header, (2) Dùng `JwtUtils`  để xác thực token , (3) Nếu token hợp lệ, lấy `UserDetails`  và set vào `SecurityContextHolder.getContext().setAuthentication(...)`.   
    

#### 2.3.3 Best-practice / Lưu ý doanh nghiệp

- **Bảo mật (Storage):** _Không_ (DO NOT) lưu trữ JWT trong `localStorage`. `localStorage` dễ bị tấn công XSS (Cross-Site Scripting).   
    
- **Bảo mật (Storage):**  Cách an toàn hơn (và là best practice) là lưu JWT trong **`HttpOnly` Cookie**. Cookie `HttpOnly` _không thể_ bị truy cập bởi JavaScript , do đó (phần lớn) miễn nhiễm với XSS.   
    
- **Bảo mật (Token):** Access Token (JWT)  phải có thời gian hết hạn _ngắn_ (short-lived), ví dụ: 5-15 phút.   
    
- **Vận hành (Refresh Token):**  Vì Access Token hết hạn nhanh, ta cần "Refresh Token".   
    
    - Refresh Token là một token (thường là một UUID, không phải JWT) có hạn _dài_ (ví dụ: 7 ngày).   
        
    - Nó được lưu trữ an toàn (trong DB phía server  và `HttpOnly` cookie phía client).   
        
    - Khi Access Token hết hạn, client gửi Refresh Token đến một endpoint `/api/auth/refreshtoken`  để lấy Access Token mới.   
        
    - Lợi ích: (1) User không phải đăng nhập lại. (2) Nếu Refresh Token bị trộm, ta có thể _thu hồi_ (revoke) nó ở phía server (xóa khỏi DB).   
        
- **Hiệu năng:**  Không nhồi nhét _quá nhiều_ dữ liệu (claims) vào payload. JWT được gửi trong _mỗi_ request, nếu nó quá lớn, nó sẽ làm tăng network latency.   
    

#### 2.3.4 Ví dụ ngắn

- Cấu hình (Properties - ):   
    

YAML

```yaml
# application.properties 
app.jwt-secret=daf66e01593f61a15b857cf433aae03a005812b31234e149036bcc8dee755dbb
app.jwt-expiration-milliseconds=600000 # 10 phút 
app.jwt.refresh-expiration-ms=604800000 # 7 ngày 
```

- Code mẫu (JwtTokenProvider - ):   
    

Java

```java
@Component
public class JwtTokenProvider {
    @Value("${app.jwt-secret}") // 
    private String jwtSecret;
    @Value("${app-jwt-expiration-milliseconds}")
    private long jwtExpirationDate; // 

    public String generateToken(Authentication auth) {
        String username = auth.getName(); // 
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationDate);

        return Jwts.builder()
               .subject(username) // 
               .issuedAt(now) // 
               .expiration(expiryDate) // 
               .signWith(key()) // 
               .compact();
    }

    public String getUsername(String token) { // 
        return Jwts.parser()
               .verifyWith((SecretKey) key())
               .build()
               .parseSignedClaims(token) // 
               .getPayload()
               .getSubject();
    }

    public boolean validateToken(String token) { // 
        try {
            Jwts.parser().verifyWith((SecretKey) key()).build().parse(token); // 
            return true;
        } catch (ExpiredJwtException ex) { // 
            log.error("Expired JWT token");
        } catch (MalformedJwtException ex) { // 
            log.error("Invalid JWT token");
        }
        return false;
    }

    private Key key() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(jwtSecret)); // 
    }
}
```

JWT là một _sự đánh đổi_ (trade-off) giữa _Khả năng mở rộng_ (Scalability) và _Khả năng thu hồi_ (Revocability). Session (STATEFUL)  truyền thống yêu cầu server phải lưu trữ session (khó scale nhưng dễ thu hồi). JWT (STATELESS)  thì ngược lại: scale vô hạn (vì server không cần lưu gì) nhưng _không thể_ "thu hồi" (revoke) một JWT đã cấp. Một khi đã cấp, nó có hiệu lực cho đến khi _hết hạn_. Đây chính là lý do tại sao Access Token _phải_ có thời gian hết hạn _siêu ngắn_ (5-15 phút)  và _phải_ dùng pattern Refresh Token. Doanh nghiệp chấp nhận thu hồi Refresh Token (được lưu trong DB ) và chấp nhận Access Token "mồ côi" tồn tại trong 5-15 phút.   

### 2.4 Cấp độ Chuyên sâu (Federation: OAuth2/OIDC)

Phần này nói về việc _không_ tự làm (build) hệ thống xác thực, mà _tích hợp_ (integrate) với một hệ thống chuyên dụng (Identity Provider - IdP).

#### 2.4.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** OAuth2  là một framework về _ủy quyền_ (Authorization). Nó _không_ phải là xác thực. Nó cho phép ứng dụng (Client)  truy cập tài nguyên (Resource Server)  thay mặt cho chủ sở hữu (Resource Owner). OpenID Connect (OIDC) là một lớp (layer) nằm _trên_ OAuth2, cung cấp tính năng _xác thực_ (Authentication).   
    
- **Mục đích:**  Cho phép "Đăng nhập bằng Google/Facebook"  hoặc (quan trọng hơn trong doanh nghiệp) tạo ra một _Identity Provider (IdP)_ trung tâm (ví dụ: Keycloak , Okta, ADFS ) để quản lý _tất cả_ user và _tất cả_ ứng dụng một cách tập trung.   
    

#### 2.4.2 Các thành phần/khái niệm quan trọng

- **Các vai trò (Roles) trong OAuth2:**    
    
    1. `Resource Owner`: Người dùng (User).   
        
    2. `Client`: Ứng dụng (ví dụ: React App, Spring Boot App).   
        
    3. `Resource Server`: API backend (ví dụ: Microservice của bạn).   
        
    4. `Authorization Server (AS)`: Máy chủ cấp token (ví dụ: Keycloak , Google ).   
        
- **Các luồng (Grant Types):**    
    
    - `Authorization Code`:  Luồng an toàn nhất, dùng cho web/mobile app.   
        
    - `Client Credentials`:  Dùng cho giao tiếp "máy-đến-máy" (machine-to-machine), ví dụ: service A gọi service B.   
        

#### 2.4.3 Best-practice / Lưu ý doanh nghiệp

- **Bảo mật:** _Không tự viết_ (DO NOT BUILD) Authorization Server (AS)  của riêng bạn. Việc này _cực kỳ_ phức tạp và rủi ro.   
    
- **Bảo mật:**  Sử dụng các giải pháp đã được kiểm chứng (battle-tested) như Keycloak  (open-source) hoặc các dịch vụ (SaaS) như Okta, Auth0, AWS Cognito , ADFS.   
    
- **Bảo mật:** Luôn dùng luồng `Authorization Code` + `PKCE` cho SPA (React, Angular). _Không bao giờ_ dùng luồng `Implicit` (đã deprecated) hoặc `Resource Owner Password Credentials`  (rò rỉ mật khẩu user).   
    
- **Kiến trúc:**  Trong kiến trúc microservices:   
    
    - API Gateway (Spring Cloud Gateway) (xem 3.3) đóng vai trò là `Client`.   
        
    - Các Microservices (Order, Payment...) đóng vai trò là `Resource Server`.   
        

#### 2.4.4 Khi triển khai thực tế (Microservices)

- **Centralized Auth:**  Tất cả các microservices (Resource Servers) được cấu hình trỏ đến _một_ IdP (Keycloak)  duy nhất.   
    
- **Cấu hình (Auto-Configuration):** Chỉ cần thêm `spring-boot-starter-oauth2-resource-server`  và cấu hình `issuer-uri`  trong `application.yml`. Spring Boot sẽ _tự động_:   
    
    1. Tải public key (JWK Set URI) từ IdP.
        
    2. Xác thực chữ ký (signature) của mọi JWT đến.
        
    3. Xác thực thời gian hết hạn (`exp`) và nhà phát hành (`iss`).
        
    4. "Dịch" các `roles` và `scopes` trong JWT  thành `GrantedAuthority` của Spring Security (giúp `@PreAuthorize`  hoạt động).   
        
- **RBAC (Role-Based Access Control):**  Các vai trò (ví dụ: `app-admin`, `app-user`) được quản lý _tập trung_ trên Keycloak , không phải trong DB của từng service. Developer chỉ việc dùng `@PreAuthorize("hasRole('app-admin')")`.   
    

#### 2.4.5 Ví dụ ngắn

- Cấu hình (application.yml cho Resource Server - ):   
    

YAML

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          #  Trỏ đến Keycloak (hoặc Okta, ADFS...)
          # Spring Boot sẽ tự động tìm.well-known/openid-configuration
          issuer-uri: "http://localhost:8080/realms/SpringBootKeycloak" # 
```

- Code mẫu (Bảo vệ endpoint - ):   
    

Java

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    @GetMapping("/{id}")
    @PreAuthorize("hasAuthority('SCOPE_read:orders')") // Kiểm tra scope (cho app)
    public Order getOrder(@PathVariable Long id) {
        //...
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')") //  Kiểm tra role (cho user)
    public void deleteOrder(@PathVariable Long id) {
        //...
    }
}
```

Đối với các doanh nghiệp, việc _tự xây dựng_ hệ thống xác thực (Cấp độ 2.3)  đang trở thành một "anti-pattern". Lý do là việc triển khai Cấp độ 2.3 một cách _an toàn_ (quản lý Refresh Token , HttpOnly Cookie , thu hồi , quên mật khẩu, MFA...) là _cực kỳ_ phức tạp và dễ sai lầm. Các IdP như Keycloak  cung cấp _miễn phí_ tất cả các tính năng này. Với Spring Boot , việc _tích hợp_ (Cấp độ 2.4) giờ đây _dễ hơn_ (chỉ cần 1 dòng `issuer-uri` ) so với việc _tự làm_ (Cấp độ 2.3).   

---

## 3. Spring Cloud

### 3.1 Cấp độ Dễ (Microservice Concepts)

Phần này giới thiệu _lý do_ (why) Spring Cloud tồn tại và các _vấn đề_ (problems) mà nó giải quyết.

#### 3.1.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Spring Cloud _không_ phải là một thứ. Nó là một _bộ sưu tập_ (umbrella project)  gồm nhiều dự án (projects). Mỗi dự án cung cấp một _giải pháp_ (implementation) cho một _vấn đề_ (pattern) phổ biến trong kiến trúc hệ thống phân tán (microservices).   
    
- **Mục đích:** Giúp developer của Spring xây dựng các ứng dụng microservices một cách _nhất quán_ theo "phong cách Spring" (Spring idioms and best practices) , cung cấp các công cụ cho service discovery, configuration, resilience....   
    

#### 3.1.2 Các thành phần/khái niệm quan trọng

- **Kiến trúc Microservices:** Chia một ứng dụng "to" (monolith) thành nhiều dịch vụ "nhỏ" (microservices), độc lập (về code, deploy, scale), giao tiếp với nhau qua mạng (thường là HTTP/REST hoặc Messaging).   
    
- **Vấn đề của hệ thống phân tán (Distributed Systems Problems):**    
    
    1. **Fault Tolerance:**  Service B có thể "chết" bất cứ lúc nào, Service A phải xử lý thế nào?   
        
    2. **Service Discovery:**  Service A làm sao tìm được địa chỉ IP của Service B (khi IP thay đổi liên tục trong Cloud/Kubernetes)?.   
        
    3. **Configuration:** Làm sao quản lý cấu hình cho 50 service?.   
        
    4. **Logging & Tracing:**  Một request đi qua 5 service, làm sao biết nó lỗi ở đâu?.   
        

#### 3.1.3 Best-practice / Lưu ý doanh nghiệp

- **Kiến trúc:** _Không lạm dụng_ (Do Not Overuse) microservices. Microservices _tăng_ (increase) sự phức tạp về _vận hành_ (operations).   
    
- **Kiến trúc (Cân nhắc):** Bắt đầu với "Modular Monolith" (monolith nhưng code được tổ chức rõ ràng theo module/domain). Chỉ _tách_ (extract) một module ra thành microservice khi có lý do chính đáng (ví dụ: cần scale riêng module đó, hoặc 2 team khác nhau làm).
    
- **Vận hành:**  Chi phí vận hành của microservices _cao hơn_ monolith rất nhiều. Doanh nghiệp _phải_ đầu tư mạnh vào CI/CD, Centralized Logging (Tổng hợp log tập trung) , Monitoring (Giám sát) , và Tracing (Truy vết)  _trước khi_ áp dụng microservices.   
    

Spring Cloud  ra đời vì Spring Boot  _tạo điều kiện_ (enabled) cho microservices. Khi developer bắt đầu tạo ra _nhiều_ "runnable JAR" , họ ngay lập tức gặp phải các vấn đề của hệ thống phân tán  (Làm sao chúng nói chuyện với nhau?  Làm sao quản lý cấu hình? ). Spring Cloud  ra đời _trực tiếp_ để giải quyết các vấn đề _do chính_ sự thành công của Spring Boot gây ra.   

### 3.2 Cấp độ Trung (Service Discovery & Load Balancing)

Phần này giải quyết vấn đề cơ bản nhất của microservices: "Làm sao Service A tìm và gọi Service B?".

#### 3.2.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm (Service Discovery):**  Trong một môi trường động (cloud, K8s), IP của service thay đổi liên tục. Service Discovery (Phát hiện dịch vụ)  là một "cuốn danh bạ" (phone book) tự động cập nhật. Các service (Client)  khi khởi động sẽ tự "đăng ký" (Register)  tên và địa chỉ của mình vào "danh bạ" (Registry).   
    
- **Khái niệm (Load Balancing):**  Khi Service A muốn gọi Service B, nó hỏi "danh bạ" và thấy Service B có 3 "thể hiện" (instances) đang chạy (ở 3 IP khác nhau). Load Balancer (Cân bằng tải)  sẽ giúp Service A chọn _một_ trong 3 thể hiện đó (ví dụ: theo kiểu round-robin) để phân phối tải.   
    

#### 3.2.2 Các thành phần/khái niệm quan trọng

- **Service Registry (Danh bạ):**    
    
    - **Netflix Eureka:**  Một dự án của Spring Cloud. Bao gồm `Eureka Server` (nơi đăng ký, "danh bạ") và `Eureka Client` (thư viện trong microservice để "tự đăng ký").   
        
    - **Consul / Zookeeper:**  Các lựa chọn thay thế Eureka.   
        
- **Load Balancing (Client-Side):**  Cân bằng tải phía Client.   
    
    - **Spring Cloud LoadBalancer:**  (Thay thế cho Netflix Ribbon đã deprecated). Đây là một thư viện phía _client_ (Service A).   
        
    - **`@LoadBalanced`:**  Một annotation. Khi bạn thêm nó vào một `RestTemplate`  (hoặc `WebClient.Builder`), nó "dạy" `RestTemplate`  cách sử dụng "danh bạ" (Eureka)  để phân giải tên (ví dụ: "user-service") thành IP.   
        

#### 3.2.3 Best-practice / Lưu ý doanh nghiệp

- **Vận hành:**  Eureka Server là _trái tim_ của hệ thống. Nếu nó chết, các service mới không thể đăng ký. _Luôn luôn_ chạy Eureka Server ở chế độ High Availability (HA) (cụm 2 hoặc 3 node, tự sao chép lẫn nhau).   
    
- **Vận hành:**  Tích hợp Health Checks  của Spring Boot Actuator (`/actuator/health`)  với Eureka. Điều này cho phép Eureka biết service của bạn vẫn "sống" (`UP`) nhưng không "khỏe" (ví dụ: `OUT_OF_SERVICE` vì mất kết nối DB). Eureka sẽ _tạm thời_ ngừng gửi traffic đến nó.   
    
- **Kiến trúc (K8s):**  Nếu bạn đang chạy 100% trên Kubernetes , bạn _có thể không cần_ Eureka. Kubernetes đã cung cấp cơ chế Service Discovery  (thông qua DNS nội bộ) và Load Balancing (thông qua `Service` object). Spring Cloud Kubernetes  là dự án giúp Spring Cloud "nói chuyện" trực tiếp với API của K8s.   
    

#### 3.2.4 Ví dụ ngắn

- **Cấu hình (Microservice - Eureka Client - `application.yml`):**
    

YAML

```yaml
spring:
  application:
    name: "order-service" # Tên sẽ đăng ký với Eureka 
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/ # 
  instance:
    health-check-url-path: "/actuator/health" #  Dùng health check của Actuator
```

- Code mẫu (Sử dụng `@LoadBalanced` - ):   
    

Java

```java
@Configuration
public class AppConfig {
    @Bean
    @LoadBalanced // 
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    @Autowired
    private RestTemplate restTemplate; // 

    public String getUserInfo(Long userId) {
        //  Dùng "tên" (application name), không dùng IP:PORT
        // Spring Cloud LoadBalancer sẽ tự động dịch "user-service"
        // thành một IP:PORT cụ thể từ Eureka.
        String url = "http://user-service/api/users/" + userId;
        return restTemplate.getForObject(url, String.class);
    }
}
```

`@LoadBalanced`  là một ví dụ tuyệt vời về "ma thuật" của Spring. Khi được thêm vào, Spring Boot tự động thêm một `ClientHttpRequestInterceptor` vào `RestTemplate`. Interceptor này chặn request, lấy "host" (`user-service`), hỏi Eureka  để lấy danh sách IP, chọn một IP bằng logic Load Balancing , và _viết lại_ (rewrite) URL trước khi gửi đi. Điều này hoàn toàn minh bạch với developer, cho phép họ trừu tượng hóa vị trí của service.   

### 3.3 Cấp độ Khó (Gateway & Config)

Phần này giải quyết 2 vấn đề vận hành lớn: "Ai được vào?" (Gateway) và "Làm sao quản lý 50 cái `application.yml`?" (Config Server).

#### 3.3.1 Giải thích khái niệm + Mục đích sử dụng

- **API Gateway (Spring Cloud Gateway):**  Là một service _duy nhất_ đóng vai trò "cổng vào" (entrypoint)  cho _tất cả_ các request từ bên ngoài (client, mobile app). Nó giống như "lễ tân" của tòa nhà microservices.   
    
    - **Mục đích:**  (1) **Routing (Định tuyến):**  Hướng request `.../api/orders/**` đến `order-service`, `.../api/users/**` đến `user-service`. (2) **Cross-Cutting Concerns (Các vấn đề chung):**  Xử lý các nghiệp vụ _chung_ tại một nơi: Bảo mật (xác thực token - xem 2.4), Rate Limiting (chống DDoS), Logging, Metrics.   
        
- **Config Server (Spring Cloud Config):**  Là một service _duy nhất_ cung cấp cấu hình (properties) cho _tất cả_ các microservices khác.   
    
    - **Mục đích:**  Quản lý cấu hình tập trung. Thay vì mỗi service có 1 tệp `application.yml` , tất cả `yml` được lưu ở một nơi (thường là Git repo).   
        

#### 3.3.2 Các thành phần/khái niệm quan trọng

- **Spring Cloud Gateway:**  (Dự án thay thế Zuul 1).   
    
    - **Route:**  Cấu hình (ID, URI, Predicates, Filters).   
        
    - **Predicate:**  Điều kiện để _khớp_ (match) request (ví dụ: `Path=/api/**`, `Method=GET`).   
        
    - **Filter:**  Xử lý request _trước_ (pre) hoặc _sau_ (post) khi gửi đến microservice (ví dụ: `AddRequestHeader`, `RewritePath`).   
        
- **Spring Cloud Config:**    
    
    - **Config Server:**  Service Spring Boot (`@EnableConfigServer`) đọc từ Git repo.   
        
    - **Config Client:**  Logic bên trong microservice. Khi khởi động, nó "gọi" Config Server để _tải_ (fetch) cấu hình của mình.   
        
    - **`bootstrap.yml`:**  Tệp cấu hình _đặc biệt_ được nạp _trước cả_ `application.yml`. Nó chứa thông tin để _tìm_ Config Server. (Trong Spring Boot 3, điều này được thay bằng `spring.config.import`).   
        

#### 3.3.3 Best-practice / Lưu ý doanh nghiệp

- **Kiến trúc:**  API Gateway  là _lớp_ (layer) lý tưởng để thực thi bảo mật (xem 2.4). Client (React, Mobile) chỉ cần biết đến Gateway. Gateway sẽ xác thực token (ví dụ: OAuth2 ). Các microservices bên trong có thể nằm trong "mạng riêng" (private network).   
    
- **Vận hành (Config):**  Quyết định chiến lược Bootstrap: "Config First" (Default) (service phải biết IP của Config Server)  hoặc "Discovery First" (service hỏi Eureka  để tìm Config Server). "Discovery First" linh hoạt hơn nhưng phức tạp hơn (vấn đề con gà-quả trứng ).   
    
- **Bảo mật (Config):**  Git repo  chứa cấu hình là _cực kỳ nhạy cảm_. Phải bảo vệ nó. Spring Cloud Config cung cấp cơ chế `encrypt` (mã hóa) các secrets ngay cả khi chúng nằm trong Git, và Config Server sẽ `decrypt` khi phục vụ client.   
    

#### 3.3.4 Ví dụ ngắn

- Cấu hình (Spring Cloud Gateway - `application.yml` - ):   
    

YAML

```yaml
spring:
  cloud:
    gateway:
      routes:
        #  Cấu hình thủ công
        - id: order_service_route
          uri: "lb://order-service" # "lb://" -> Dùng Eureka/LoadBalancer 
          predicates:
            - Path=/api/orders/** # [42, 43]
          filters:
            #  Bỏ tiền tố /api/orders
            - RewritePath=/api/orders/(?<segment>.*), /${segment} 

        - id: user_service_route
          uri: "lb://user-service"
          predicates:
            - Path=/api/users/**
```

- Cấu hình (Config Client - `bootstrap.yml` - ):   
    

YAML

```yaml
spring:
  application:
    name: order-service # Tên của tệp config: order-service.yml 
  cloud:
    config:
      uri: http://config-server:8888 #  Địa chỉ Config Server
      profile: "prod" # Lấy config của profile "prod"
```

Spring Cloud Gateway  (dùng Project Reactor/WebFlux) có hiệu năng vượt trội so với Zuul 1 (dùng Servlet/Blocking). Zuul 1 dùng mô hình "một request, một thread". Nếu 200 request đồng thời gọi một service chậm (5s), 200 thread của Zuul sẽ bị "cạn kiệt" (exhausted), dẫn đến sập Gateway. Spring Cloud Gateway  dùng non-blocking I/O, sử dụng _ít_ thread hơn (Event Loop) để xử lý _nhiều_ request. Nếu service chậm, Gateway _không_ bị block, nó chỉ "đặt lịch" (callback) và đi xử lý request khác.   

### 3.4 Cấp độ Chuyên sâu (Resilience & Observability)

Phần này giải quyết 2 vấn đề "ác mộng" nhất của microservices: "Service B chết thì sao?" (Resilience) và "Làm sao tìm lỗi khi request đi qua 5 service?" (Observability/Tracing).

#### 3.4.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm (Resilience):**  Khả năng _phục hồi_. Trong hệ thống phân tán, _lỗi là bình thường_ (failure is normal). Resilience (Khả năng phục hồi)  là các pattern  giúp hệ thống vẫn _hoạt động_ (hoặc "fail" một cách duyên dáng) khi một phần của nó bị lỗi.   
    
- **Khái niệm (Distributed Tracing):**  Khả năng _theo dõi_ (trace) một request  khi nó đi qua nhiều microservices.   
    
- **Khái niệm (Event-Driven):**  Giao tiếp _bất đồng bộ_ (asynchronous)  qua Message Broker (Kafka , RabbitMQ ) thay vì gọi HTTP/REST đồng bộ.   
    

#### 3.4.2 Các thành phần/khái niệm quan trọng

- **Resilience (Resilience4j):**  (Thay thế Hystrix).   
    
    - **Circuit Breaker (Ngắt mạch):**  Pattern quan trọng nhất. Nếu Service B bị lỗi liên tục (ví dụ: 50% lỗi trong 10s), Service A sẽ "ngắt mạch" (mở cầu dao) , ngừng gọi Service B trong 1 phút. Tránh bão "retry" (retry storm) và cho Service B thời gian phục hồi.   
        
    - **Fallback:**  Khi mạch "ngắt", thay vì báo lỗi, Service A trả về một giá trị dự phòng (ví dụ: data từ cache).   
        
    - **Bulkhead:**  Phân vùng tài nguyên (thread pool hoặc semaphore). Ngăn Service B (chậm) làm _cạn kiệt_ toàn bộ thread pool của Service A.   
        
    - **Retry:**  Tự động gọi lại (retry) nếu request thất bại.   
        
- **Distributed Tracing (Micrometer Tracing / OTel):**    
    
    - `Trace ID`:  Một ID _duy nhất_ cho _toàn bộ_ request.   
        
    - `Span ID`:  Một ID cho _từng bước_ (operation) trong request.   
        
    - **Spring Cloud Sleuth:**  (Cũ, Boot 2) Tự động thêm (auto-instruments) Trace/Span ID vào log (MDC)  và truyền qua header.   
        
    - **OpenTelemetry (OTel):**  (Mới, Boot 3+) Tiêu chuẩn _mới_ và _thống nhất_  cho cả Tracing, Metrics, Logs.   
        
    - **Backend:** Zipkin , Jaeger  là các DB/UI để _lưu trữ_ và _trực quan hóa_ các trace.   
        
- **Event-Driven (Spring Cloud Stream):**    
    
    - Trừu tượng hóa (abstraction)  việc giao tiếp với Kafka  / RabbitMQ.   
        
    - Cung cấp các khái niệm: `Supplier` (Producer) , `Function` (Processor), `Consumer`.   
        

#### 3.4.3 Best-practice / Lưu ý doanh nghiệp

- **Kiến trúc:** Phân biệt rõ khi nào dùng _đồng bộ_ (Synchronous - REST) và _bất đồng bộ_ (Asynchronous - Event-Driven ).   
    
    - Dùng REST khi client _cần_ câu trả lời _ngay lập tức_ (ví dụ: `GET /order/{id}`).
        
    - Dùng Event-Driven  khi client _không cần_ đợi (ví dụ: `POST /order` -> `OrderPlacedEvent` ). `EmailService`, `InventoryService` sẽ _lắng nghe_ (subscribe)  event đó và tự xử lý.   
        
- **Kiến trúc (Resilience):**  Sử dụng Circuit Breaker  cho _mọi_ cuộc gọi mạng (network call) ra bên ngoài.   
    
- **Vận hành (Tracing):**  Tracing là _bắt buộc_ (must-have) trong microservices. Không có Tracing , việc debug một lỗi (ví dụ: "request này chậm ở đâu?" ) là _không thể_.   
    
- **Vận hành (Tracing):**  Cách nhanh nhất để bắt đầu là dùng OTel Java Agent. Nó "tự động" (auto-instruments)  thêm tracing vào code (Spring MVC, JDBC, RestTemplate ) mà _không cần thay đổi code_.   
    

#### 3.4.4 Khi triển khai thực tế (Cloud-Native)

- Như đã đề cập (3.1.5), trong Kubernetes, Service Mesh (Istio)  có thể cung cấp các tính năng này (Circuit Breaker , Retry , Tracing ) ở _cấp độ nền tảng_ (platform), bên ngoài ứng dụng.   
    
- **Lựa chọn chiến lược:** Doanh nghiệp phải lựa chọn: (a) App-level resilience (dùng Resilience4j  - developer kiểm soát) hay (b) Platform-level resilience (dùng Istio  - DevOps kiểm soát).   
    
- **Observability Stack:**  Stack hiện đại là: Spring Boot App -> (Micrometer Tracing) -> OTel Collector (Sử dụng OTLP protocol ) -> (Gửi đến nhiều backend: Jaeger  cho Tracing, Prometheus  cho Metrics).   
    

#### 3.4.5 Ví dụ ngắn

- Code mẫu (Resilience4j - ):   
    

Java

```java
@Service
public class PaymentService {
    @Autowired
    private RestTemplate restTemplate;
    
    //  Áp dụng nhiều pattern
    @CircuitBreaker(name = "payment-gateway", fallbackMethod = "fallbackPayment")
    @Bulkhead(name = "payment-gateway", type = Bulkhead.Type.THREADPOOL)
    @Retry(name = "payment-gateway", fallbackMethod = "fallbackPayment")
    public String callPaymentGateway(Order order) {
        //... restTemplate.postForObject(...) 
    }

    //  Fallback method phải có cùng (hoặc tương thích)
    // signature + 1 tham số Throwable
    public String fallbackPayment(Order order, Throwable t) {
        log.warn("Payment fallback: " + t.getMessage());
        return "PAYMENT_PENDING"; // Trả về trạng thái "đang chờ"
    }
}
```

- Cấu hình (Resilience4j - `application.yml` - ):   
    

YAML

```yaml
resilience4j:
  circuitbreaker: # 
    instances:
      payment-gateway:
        sliding-window-size: 10
        failure-rate-threshold: 50 # Ngắt mạch nếu 50% lỗi 
        wait-duration-in-open-state: "10s" # 10 giây
  bulkhead: # 
    instances:
      payment-gateway:
        max-concurrent-calls: 10 # Chỉ cho phép 10 cuộc gọi đồng thời
```

Giao tiếp Event-Driven  là giải pháp cho các vấn đề do giao tiếp _đồng bộ_ (sync) (HTTP/REST) gây ra. Nếu `OrderService` (đồng bộ) gọi `InventoryService` VÀ `EmailService`, và `EmailService` (không quan trọng) bị _chậm_ hoặc _lỗi_, thì `OrderService` sẽ bị _treo_ (blocked) và _báo lỗi_ (fail) cho người dùng. Đây gọi là "cascading failure" (lỗi dây chuyền). Bằng cách chuyển sang Event-Driven , `OrderService` chỉ cần _bắn_ (fire-and-forget) một event `OrderPlaced`  lên Kafka. Nếu `EmailService` chết , nó _không ảnh hưởng_ đến `OrderService`. Hệ thống trở nên _linh hoạt_ (resilient)  và _ít coupling_ (loosely coupled) hơn.   

---

## 4. Spring Data JPA / Hibernate

### 4.1 Cấp độ Dễ (Core Concepts & CRUD)

Phần này giới thiệu cách _tương tác_ với CSDL quan hệ (MySQL, Postgre...) bằng Java `class` thay vì viết SQL.

#### 4.1.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm (ORM):** Object-Relational Mapping (Ánh xạ Đối tượng Quan hệ). Là một kỹ thuật "dịch" (mapping) giữa các Java `class` (Object) và các `bảng` (Table) trong CSDL quan hệ (Relational).
    
- **JPA (Java Persistence API):**  Là một _tiêu chuẩn_ (specification) (một bộ interface/quy tắc) của Java, định nghĩa cách làm ORM.   
    
- **Hibernate:**  Là _triển khai_ (implementation) _phổ biến nhất_ của JPA. Nó là "cỗ máy" (engine) thực sự làm công việc "dịch".   
    
- **Spring Data JPA:**  _Không phải_ là JPA hay Hibernate. Nó là một dự án của Spring, nằm _trên cùng_ (on top). Nó cung cấp một lớp _trừu tượng hóa_ (abstraction)  nữa, giúp việc viết code `Repository` (lớp truy cập dữ liệu) _cực kỳ_ đơn giản, giảm code boilerplate (code lặp đi lặp lại).   
    

#### 4.1.2 Các thành phần/khái niệm quan trọng

- **`@Entity`:**  Annotation. Đánh dấu một Java class là một "thực thể" (entity), tương ứng với một bảng trong DB.   
    
- **`@Id`, `@GeneratedValue`:**  Đánh dấu trường (field) là khóa chính (Primary Key) và cách nó được tạo (ví dụ: auto-increment).   
    
- **`@Column`:**  Tùy chỉnh (tên, độ dài...) cột.   
    
- **`CrudRepository`:**  Interface cơ bản, cung cấp các hàm: `save()`, `findById()`, `findAll()`, `delete()`.   
    
- **`JpaRepository`:**  Interface _kế thừa_ (extends) `CrudRepository`. Nó cung cấp _thêm_ các tính năng của JPA (ví dụ: `flush()`, `saveAndFlush()`) và (quan trọng nhất) Paging & Sorting.   
    

#### 4.1.3 Best-practice / Lưu ý doanh nghiệp

- **Clean Code:** _Luôn luôn_ kế thừa (extends) từ `JpaRepository`  thay vì `CrudRepository`. `JpaRepository`  cung cấp Paging/Sorting, thứ mà _mọi_ ứng dụng doanh nghiệp _cuối cùng_ đều cần.   
    
- **Clean Code (DTO):** Entity  là để _ánh xạ_ (mapping), _không phải_ để truyền dữ liệu (DTO). _Không_ (DO NOT) trả về `@Entity`  trực tiếp từ Controller/API (làm lộ cấu trúc DB, dễ gây lỗi bảo mật và N+1). Hãy tạo các class DTO (Data Transfer Object) và "map" dữ liệu từ Entity  sang DTO.   
    
- **Hiệu năng:**  _Không_ nghĩ rằng dùng JPA/Hibernate  nghĩa là bạn _không cần_ biết SQL. Bạn _phải_ bật logging SQL (ví dụ: `spring.jpa.show-sql=true` trong `dev`) để _xem_ (inspect) các câu SQL mà Hibernate  tạo ra.   
    

#### 4.1.4 Ví dụ ngắn

- Code mẫu (Entity - ):   
    

Java

```java
@Entity
@Table(name = "users") // 
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false, unique = true)
    private String username;
    private String password;
    // Getters, Setters... 
}
```

- Code mẫu (Repository - ):   
    

Java

```java
//  Kế thừa JpaRepository, Spring sẽ tự động implement
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring Data JPA sẽ tự động tạo query cho bạn (xem 4.2)
}
```

- Code mẫu (Service - ):   
    

Java

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User createUser(UserDTO dto) {
        User newUser = new User();
        newUser.setUsername(dto.getUsername());
        newUser.setPassword(dto.getPassword());
        return userRepository.save(newUser); // 
    }

    public Optional<User> findUserById(Long id) {
        return userRepository.findById(id); // 
    }
}
```

Sự kết hợp của Spring Data JPA (Cấp 4.1) và Spring Security (Cấp 2.1) là một ví dụ điển hình về sức mạnh của hệ sinh thái Spring. Spring Security cần một `UserDetailsService` , và `UserDetailsService`  cần một cách để `findByUsername`. Spring Data JPA cung cấp chính xác `Query Method` đó (`User findByUsername(String username)`)  _mà không cần viết một dòng code_ implementation nào. Sự kết hợp này cho phép developer implement một luồng xác thực (authn)  kết nối CSDL  _chỉ trong vài phút_.   

### 4.2 Cấp độ Trung (Querying & Relationships)

Phần này tập trung vào cách _lấy_ (query) dữ liệu phức tạp hơn CRUD  và cách _mô hình hóa_ (model) các mối quan hệ.   

#### 4.2.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm:** Định nghĩa các cách phức tạp hơn để _lấy_ (fetch) dữ liệu  và _mô hình hóa_ (model) các mối quan hệ  giữa các bảng (ví dụ: User và Order).   
    
- **Mục đích:** Trả lời các câu hỏi nghiệp vụ (ví dụ: "Tìm tất cả đơn hàng của User X", "Tìm 5 sản phẩm mới nhất").
    

#### 4.2.2 Các thành phần/khái niệm quan trọng

- **Query Methods (Derived Queries):**  Spring Data JPA "suy luận" (derive) câu lệnh SQL/JPQL từ _tên_ (name) của method  trong interface Repository.   
    
    - Ví dụ: `findByUsername(String username)`  ->   `... WHERE username =?`
        
    - Ví dụ: `findByLastNameAndPhoneType(...)`  ->   `... WHERE lastName =? AND phoneType =?`
        
- **`@Query` (JPQL):**  Khi Query Method  trở nên quá dài hoặc phức tạp, ta dùng `@Query`  để _tự viết_ câu truy vấn bằng JPQL (Java Persistence Query Language).   
    
    - JPQL  giống SQL, nhưng làm việc với _Entity_ (class) thay vì _bảng_ (table). (Ví dụ: `SELECT u FROM User u WHERE u.username = :name`).   
        
- **Relationships (Quan hệ):**    
    
    - **`@ManyToOne`:**  Nhiều (Many) `Order` thuộc về Một (One) `User`. Đây là "phía con" (owning side), chứa khóa ngoại (foreign key).   
        
    - **`@OneToMany`:**  Một (One) `User` có Nhiều (Many) `Order`. Thường có một `List<Order>`.   
        
    - **`@ManyToMany`:** Một (Many) `Post` có Nhiều (Many) `Tag`.
        

#### 4.2.3 Best-practice / Lưu ý doanh nghiệp

- **Clean Code:**  Ưu tiên Query Methods  cho các query đơn giản (tìm theo 1-2 trường). Dùng `@Query`  cho mọi thứ phức tạp hơn (JOIN, DTO projections).   
    
- **Hiệu năng:** _Tránh_ `@ManyToMany` bằng mọi giá. Nó che giấu "bảng trung gian" (join table) và _cực kỳ_ khó để tối ưu query, dễ gây N+1 (xem 4.3).
    
    - _Giải pháp:_ _Luôn_ tạo "bảng trung gian" (ví dụ: `PostTag`) thành một `@Entity`  riêng. Sau đó bạn có 2 quan hệ `@ManyToOne`.   
        
- **Hiệu năng (Relationship):**  _Luôn luôn_ (ALWAYS) làm chủ (own) quan hệ `@ManyToOne`  từ "phía con". (Ví dụ: `Order` class _phải_ có trường `private User user;`).   
    
- **Clean Code (Bidirectional):**  Khi dùng quan hệ 2 chiều (bidirectional), bạn _phải_ (MUST) viết các "helper method" (hàm trợ giúp)  để giữ 2 đầu _đồng bộ_ (synchronize).   
    

#### 4.2.4 Ví dụ ngắn

- Code mẫu (Query Methods & @Query - ):   
    

Java

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    // 1. Query Method  (Tự động JOIN bảng User)
    List<Order> findByCustomer_Username(String username);

    // 2. @Query (JPQL) 
    @Query("SELECT o FROM Order o WHERE o.amount > :minAmount")
    List<Order> findOrdersWithMinAmount(@Param("minAmount") double minAmount);

    // 3. @Query (Native SQL) - 
    @Query(value = "SELECT * FROM orders WHERE amount >?1", nativeQuery = true)
    List<Order> findOrdersWithMinAmountNative(double minAmount);
}
```

- Code mẫu (Helper method cho quan hệ 2 chiều - ):   
    

Java

```java
@Entity
public class User {
    //  "mappedBy" = "user" (tên trường bên phía Order)
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY) // 
    private List<Order> orders = new ArrayList<>();

    // Helper method 
    public void addOrder(Order order) {
        this.orders.add(order);
        order.setUser(this); //  Giữ đồng bộ 2 chiều
    }
}

@Entity
@Table(name = "orders")
public class Order {
    @Id private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY) //  (xem 4.3)
    @JoinColumn(name = "user_id") // Khóa ngoại
    private User user;
    //...
}
```

### 4.3 Cấp độ Khó (Transactions & Performance)

Phần này nói về 2 thứ: (1) Đảm bảo dữ liệu _đúng_ (Transaction ) và (2) Đảm bảo ứng dụng _nhanh_ (Performance ).   

#### 4.3.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm (Transaction):**  Một "giao dịch" (ACID). Một nhóm các thao tác (ví dụ: 1. Trừ tiền tài khoản A; 2. Cộng tiền tài khoản B) được coi là _một_ đơn vị (unit). Hoặc _tất cả_ thành công (commit), hoặc _tất cả_ thất bại (rollback).   
    
- **Khái niệm (Performance):**  Tối ưu hóa các " узкие места" (bottlenecks) phổ biến nhất do ORM gây ra.   
    
- **Mục đích:** (1) Đảm bảo _tính toàn vẹn dữ liệu_ (Data Integrity). (2) Ngăn chặn ứng dụng "sập" vì các truy vấn CSDL chậm.   
    

#### 4.3.2 Các thành phần/khái niệm quan trọng

- **`@Transactional`:**  Một annotation. Thêm nó vào một method , Spring sẽ _tự động_ "bọc" (wrap) method đó trong một proxy: bắt đầu transaction _trước khi_ chạy, và `commit` (nếu thành công) hoặc `rollback` (nếu có `RuntimeException`)  _sau khi_ chạy.   
    
- **Propagation (Lan truyền):**  Điều gì xảy ra nếu một method `@Transactional` gọi một method `@Transactional` khác?   
    
    - `REQUIRED` (Default):  Tham gia (join) vào transaction _hiện có_. (Phổ biến nhất).   
        
    - `REQUIRES_NEW`:  _Luôn_ tạo một transaction _mới_ (tạm dừng transaction cũ). (Dùng để log, audit).   
        
- **Isolation (Cô lập):**  Mức độ "nhìn thấy" dữ liệu chưa commit của các transaction khác. (ví dụ: `READ_COMMITTED` , `REPEATABLE_READ` , `SERIALIZABLE` ).   
    
- **Fetch Types:**    
    
    - `EAGER`:  Tải _ngay lập tức_. Khi bạn `findById(userId)`, Hibernate _cũng_ tải _tất cả_ `Order` của user đó (ngay cả khi bạn không cần).   
        
    - `LAZY`:  Tải _khi cần_. Khi bạn `findById(userId)`, Hibernate _chỉ_ tải `User`. Nó _chỉ_ tải `List<Order>`  nếu bạn gọi `user.getOrders()` (và transaction  vẫn còn mở).   
        
- **Vấn đề N+1 Query:**  _Kẻ giết hiệu năng_ số 1 của Hibernate.   
    
    - Ví dụ: (1) `SELECT * FROM users` (1 query). (2) Lặp (loop) qua 100 User, (3) Với _mỗi_ User, gọi `user.getOrders()` (vì `getOrders` là `LAZY` ), (4) Gây ra _thêm_ 100 query `SELECT * FROM orders WHERE user_id =?`.   
        
    - Tổng cộng: 1 + N (100) = 101 query.   
        

#### 4.3.3 Best-practice / Lưu ý doanh nghiệp

- **Hiệu năng:**  _Quy tắc vàng:_   
    
    - `@ManyToOne`, `@OneToOne`: Mặc định là `EAGER`. OK.
        
    - `@OneToMany`, `@ManyToMany`: Mặc định là `LAZY`. _Luôn giữ_ `LAZY`. _Không bao giờ_ đổi  thành `EAGER`. `EAGER`  ở collection là "code smell" (dấu hiệu code có vấn đề).   
        
- **Hiệu năng (Giải quyết N+1):**  Dùng "JOIN FETCH" trong JPQL.   
    
    - Thay vì `userRepository.findAll()` (gây N+1).
        
    - Hãy viết: `@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")`.
        
    - Hibernate sẽ tạo ra _một_ (1) câu query SQL (dùng `LEFT JOIN`)  và "điền" (populate) cả `User` và `List<Order>` trong một lần.   
        
- **Clean Code:**  `@Transactional`  _chỉ_ nên đặt ở **public method**  tại **Service Layer** (lớp Service). Nó _không_ hoạt động trên `private` method  hoặc "self-invocation" (tự gọi trong cùng 1 class)  do cơ chế proxy.   
    
- **Hiệu năng:**  _Luôn_ dùng `@Transactional(readOnly = true)`  cho _tất cả_ các method `SELECT` (ví dụ: `find`, `get`, `search`). Nó "báo" cho Hibernate  (1) "Không cần 'dirty check' (tăng tốc)", (2) "Có thể dùng CSDL replica (read-only)"  (xem 4.4).   
    
- **Bảo mật:**  Hiểu rõ `rollbackFor`. Mặc định, Spring _chỉ_ rollback  khi có `RuntimeException` (unchecked). Nó _không_ rollback nếu có `Exception` (checked). Best practice: `@Transactional(rollbackFor = {Exception.class})`  để _luôn_ rollback (kể cả checked exception).   
    

#### 4.3.4 Ví dụ ngắn

- Code mẫu (Tối ưu read/write - ):   
    

Java

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional(readOnly = true) //  Tối ưu cho "đọc"
    public User findUser(String username) {
        return userRepository.findByUsername(username);
    }

    @Transactional(rollbackFor = {Exception.class}) //  Đảm bảo rollback
    public User createUser(UserDTO dto) throws IOException {
        User user = new User();
        userRepository.save(user);
        // if (true) throw new IOException(); //  Sẽ rollback vì có rollbackFor
        return user;
    }

    @Transactional(readOnly = true)
    public List<User> findAllUsersAndOrders() {
        //  Dùng JOIN FETCH để giải quyết N+1
        return userRepository.findAllWithOrders(); 
    }
}

public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders") // 
    List<User> findAllWithOrders();
    
    User findByUsername(String username); // 
}
```

### 4.4 Cấp độ Chuyên sâu (Caching, Replication & Scaling)

Phần này nói về các kỹ thuật _cực kỳ_ quan trọng để CSDL có thể chịu được tải (load) cao trong môi trường doanh nghiệp.

#### 4.4.1 Giải thích khái niệm + Mục đích sử dụng

- **Khái niệm (Caching):** Giảm tải cho CSDL bằng cách lưu trữ dữ liệu _thường xuyên đọc_ (hot data) vào bộ nhớ (RAM) nhanh hơn.
    
- **Khái niệm (Replication):** Sao chép (Replicate) CSDL. Tạo ra một (hoặc nhiều) bản sao (replica) chỉ-đọc (read-only) của CSDL chính (primary, read-write).
    
- **Mục đích:** (1) Tăng _tốc độ_ (latency) bằng Caching. (2) Tăng _khả năng chịu tải_ (throughput) và _độ tin cậy_ (availability) bằng Replication.
    

#### 4.4.2 Các thành phần/khái niệm quan trọng

- **L1 Cache (Persistence Context):** (Xem 4.1.5) Đây là cache _bên trong_ Hibernate. Nó tồn tại trong phạm vi _một_ transaction (`@Transactional`). Nó _luôn_ được bật và không thể tắt.
    
- **L2 Cache (Second-Level Cache):**  Một cache _bên ngoài_ (ví dụ: EhCache , Hazelcast, Infinispan , Redis). Nó được _chia sẻ_ (shared) giữa _tất cả_ các transaction và (nếu cấu hình) giữa _nhiều_ node (server).   
    
    - Đây là cache cho _Entities_  (ví dụ: `cache.find(User.class, 1L)`).   
        
- **Query Cache:** Một cache (cũng nằm trong L2) dùng để cache _kết quả_ (results) của các _câu truy vấn_ (queries).
    
    - Ví dụ: Cache kết quả của `userRepository.findActiveUsers()`.
        
- **Read/Write Replication (Tách Đọc/Ghi):**
    
    - Một pattern kiến trúc. Cấu hình ứng dụng để:
        
    - Tất cả các lệnh `INSERT`, `UPDATE`, `DELETE` (lệnh "ghi") đi đến CSDL _Primary_ (Master).
        
    - Tất cả các lệnh `SELECT` (lệnh "đọc") đi đến các CSDL _Replica_ (Slave).
        

#### 4.4.3 Best-practice / Lưu ý doanh nghiệp

- **Hiệu năng:** _Không_ lạm dụng L2 Cache. L2 Cache _cực kỳ_ phức tạp để cấu hình đúng (ví dụ: chiến lược `READ_WRITE`, `NONSTRICT_READ_WRITE` và xử lý cache invalidation (làm mất hiệu lực cache) ).   
    
- **Hiệu năng:** L2 Cache  chỉ hiệu quả cho các _dữ liệu "gần như tĩnh"_ (ví dụ: danh sách `Permission`, `Role`, `Category`, `Country`). _Không bao giờ_ cache các entity (thực thể) thay đổi liên tục (ví dụ: `Order`, `StockPrice`).   
    
- **Hiệu năng:** _Luôn_ ưu tiên Caching ở _Application Level_ (ví dụ: dùng `@Cacheable` của Spring Cache với Redis) hơn là L2 Cache  của Hibernate. Cache ở Application Level (thường là cache DTOs) dễ kiểm soát, dễ debug và linh hoạt hơn (vì nó không bị trói buộc vào Hibernate).   
    
- **Kiến trúc:**  Pattern Read/Write Replication là một trong những cách _hiệu quả nhất_ để scale CSDL quan hệ.   
    
    - `@Transactional(readOnly = false)` (mặc định) -> Spring sẽ route đến _Primary_ `DataSource`.
        
    - `@Transactional(readOnly = true)`  (xem 4.3) -> Spring (với cấu hình đúng, ví dụ: dùng `AbstractRoutingDataSource`) sẽ route đến _Replica_ `DataSource`.   
        
    - Điều này làm cho `@Transactional(readOnly = true)`  _cực kỳ_ quan trọng, vì nó giúp _giảm tải_ (offload) toàn bộ traffic "đọc" (thường là 80-90% traffic) khỏi CSDL chính.   
        

#### 4.4.4 Khi triển khai thực tế (Scalability)

- **Vấn đề (L2 Cache):**  Caching L2 (ví dụ: EhCache ) trong một môi trường microservices (nhiều "thể hiện" - instances) rất phức tạp. Nếu Instance A cập nhật `User 1`, làm sao nó báo cho L2 Cache của Instance B, C, D biết rằng `User 1` đã cũ (stale)? Điều này đòi hỏi cơ chế "cache replication" (ví dụ: JGroups, Redis Pub/Sub), làm tăng độ phức tạp.   
    
- **Giải pháp (Ưu tiên):**
    
    1. **Tối ưu Query:** (Cấp 4.3) Dùng `JOIN FETCH` , `readOnly = true` , và tạo Index (chỉ mục) CSDL. (Luôn là bước đầu tiên).   
        
    2. **Application Cache (Spring Cache):** (Cấp 4.5 - ngoài lề) Dùng `@Cacheable` để cache DTOs trong Redis. (Hiệu quả nhất cho 90% trường hợp).
        
    3. **Read/Write Replication:** (Cấp 4.4) Tách `DataSource` cho `readOnly = true`.   
        
    4. **L2 Cache:**  Chỉ dùng làm _phương án cuối cùng_ cho các "reference data" (dữ liệu tham chiếu) thực sự tĩnh.   
        

#### 4.4.5 Ví dụ ngắn

- Cấu hình (L2 Cache & Query Cache - `application.yml` - ):   
    

YAML

```yaml
spring:
  jpa:
    properties:
      hibernate:
        cache:
          use_second_level_cache: true #  Bật L2
          use_query_cache: true         # Bật Query Cache
          region:
            factory_class: "org.hibernate.cache.ehcache.EhCacheRegionFactory" # 
#... và cần tệp ehcache.xml
```

- Code mẫu (Đánh dấu Entity là @Cacheable - ):   
    

Java

```java
@Entity
@Cacheable // Báo cho Hibernate 
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // Chiến lược
public class Country {
    @Id
    private String code;
    private String name;
    //...
}
```

- Code mẫu (Sử dụng Query Cache - ):   
    

Java

```java
public interface CountryRepository extends JpaRepository<Country, String> {
    
    //  Phải bật cả 2 (Query Cache + L2)
    @QueryHints({@QueryHint(name = "org.hibernate.cacheable", value = "true")})
    List<Country> findAll();
}
```

Trong thực tế, L1 Cache (Persistence Context) (xem 4.1.5) là thứ _gây nhầm lẫn_ nhiều nhất. Junior dev không hiểu tại sao họ _chỉ_ gọi `setter` (ví dụ: `user.setName("A")`) mà _không_ gọi `save()`, nhưng CSDL _vẫn_ bị `UPDATE`. Đó là vì trong một `@Transactional` , entity `user` đang được "quản lý" (managed) bởi L1 Cache. Hibernate (qua cơ chế "dirty checking") _tự động_ phát hiện sự thay đổi và tạo lệnh `UPDATE` lúc transaction `commit`. Hiểu rõ "Entity Lifecycle" (Transient, Managed, Detached) là _bắt buộc_ để làm chủ Hibernate.