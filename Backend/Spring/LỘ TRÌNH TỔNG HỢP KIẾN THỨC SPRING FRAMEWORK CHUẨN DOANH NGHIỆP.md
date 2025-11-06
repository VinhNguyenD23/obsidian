## 1. Spring Boot: Nền tảng Xây dựng Ứng dụng

### 1.1 Cấp độ Dễ (Junior Foundations)

- **Khái niệm & Mục đích:**
    
    - **Khái niệm:** Spring Boot là một framework được xây dựng _trên nền_ Spring Framework truyền thống. Nó không phải là một thứ gì đó hoàn toàn mới, mà là một lớp giúp đơn giản hóa và tăng tốc quá trình thiết lập, cấu hình và chạy các ứng dụng Spring.   
        
    - Triết lý cốt lõi của Spring Boot là "Convention over Configuration" (Ưu tiên quy ước hơn cấu hình). Thay vì bắt lập trình viên phải định nghĩa hàng loạt cấu hình (ví dụ: file XML), Spring Boot đưa ra các "quy ước" (assumptions) hợp lý về những gì bạn có thể cần, dựa trên các thư viện mà nó tìm thấy trên classpath.   
        
    - **Mục đích sử dụng:** Mục tiêu chính là cho phép lập trình viên "tập trung nhiều hơn vào tính năng nghiệp vụ và ít hơn vào cơ sở hạ tầng".   
        
        1. **Đơn giản hóa cấu hình:** Giảm thiểu hoặc loại bỏ hoàn toàn nhu cầu cấu hình XML phức tạp, thay bằng các chú thích (annotations) Java và file `.properties` hoặc `.yml`.   
            
        2. **Tăng tốc phát triển:** Cung cấp các "starters" (bộ khởi động) để nhanh chóng thêm và cấu hình các dependency (thư viện phụ thuộc).   
            
        3. **Dễ dàng triển khai:** Cung cấp máy chủ web (ví dụ: Tomcat, Jetty) nhúng sẵn. Ứng dụng có thể được đóng gói thành một file `.jar` duy nhất và tự chạy (standalone), giúp việc triển khai trở nên cực kỳ đơn giản.   
            
- **Các thành phần/khái niệm quan trọng:**
    
    - **Spring Initializr (`start.spring.io`):** Đây là công cụ web chính thức và là điểm khởi đầu cho mọi dự án Spring Boot. Nó cho phép bạn chọn công cụ xây dựng (Maven hoặc Gradle), ngôn ngữ (Java), phiên bản Spring Boot, và quan trọng nhất là các "Dependencies" (ví dụ: `Spring Web`, `Spring Data JPA`, `Lombok`) mà dự án của bạn cần.   
        
    - **Spring Boot Starters:** Đây là các gói mô tả dependency (dependency descriptors) tiện lợi. Về bản chất, chúng là các file `pom.xml` (với Maven) được định nghĩa sẵn, giúp gộp một nhóm các thư viện phổ biến lại với nhau.   
        
        - _Ví dụ:_ Thay vì tự thêm `spring-core`, `spring-web`, `spring-webmvc`, `jackson-databind`, `tomcat-embed`,... bạn chỉ cần thêm một dependency duy nhất: `spring-boot-starter-web`.   
            
        - _Lợi ích:_ Spring Boot quản lý phiên bản (version) và đảm bảo tất cả các thư viện con này tương thích với nhau, loại bỏ xung đột phiên bản (version conflicts).   
            
    - **Annotation `@SpringBootApplication`:** Đây là annotation chính, thường được đặt tại class `main` của ứng dụng. Nó là một "meta-annotation" (annotation gộp), kết hợp 3 annotation quan trọng khác:   
        
        1. **`@Configuration` (hoặc `@SpringBootConfiguration`):** Đánh dấu class này là một nguồn định nghĩa Bean. Spring sẽ tìm các phương thức được đánh dấu `@Bean` bên trong class này để đưa vào Application Context.   
            
        2. **`@EnableAutoConfiguration`:** Đây là "phép thuật" cốt lõi của Spring Boot. Nó ra lệnh cho Spring Boot "đoán" và tự động cấu hình các bean mà bạn _có thể_ cần, dựa trên các file `.jar` (dependency) mà nó tìm thấy trong classpath. Ví dụ: Nếu Spring Boot thấy `spring-boot-starter-data-jpa` và `mysql-connector-j` trên classpath, nó sẽ tự động cố gắng cấu hình một `DataSource` và `EntityManagerFactory` để kết nối tới MySQL.   
            
        3. **`@ComponentScan`:** Tự động quét (scan) các package con để tìm các Spring Component khác (`@Component`, `@Service`, `@Repository`, `@RestController`) và đăng ký chúng dưới dạng Bean.   
            
    - **`application.properties` (hoặc `application.yml`):** Đây là file cấu hình trung tâm. Bạn không cần cấu hình những thứ mặc định, nhưng nếu muốn _thay đổi_ (override) quy ước của Spring Boot, bạn sẽ định nghĩa chúng ở đây.   
        
        - _Ví dụ:_ Spring Boot mặc định chạy Tomcat ở cổng 8080. Để đổi sang 8081, bạn chỉ cần thêm dòng: `server.port=8081`.   
            
- **Best-practice / Lưu ý doanh nghiệp:**
    
    - **Clean Code (Cấu trúc dự án):** Đây là một quy tắc cơ bản nhưng cực kỳ quan trọng, và là lỗi phổ biến nhất của lập trình viên Junior.
        
        - **Quy tắc:** Luôn đặt class chứa `@SpringBootApplication` (class `main`) ở package gốc (root package), ví dụ: `com.congty.duan`.
            
        - **Giải thích (Tại sao):** Như đã nêu, `@ComponentScan` (bên trong `@SpringBootApplication`) chỉ quét các package _bên dưới_ nó. Nếu bạn đặt class `main` trong `com.congty.duan.config` và đặt `Controller` trong `com.congty.duan.controller`, Spring sẽ _không bao giờ_ tìm thấy `Controller` của bạn (vì `controller` không phải là package con của `config`), dẫn đến ứng dụng chạy nhưng mọi API đều trả về 404 Not Found.   
            
        - Cấu trúc chuẩn :   
            
            ```
            com.congty.duan
            ├── MainApplication.java  <-- (@SpringBootApplication nằm ở đây)
            ├── controller/
            │   └── NguoiDungController.java
            ├── service/
            │   └── NguoiDungService.java
            ├── repository/
            │   └── NguoiDungRepository.java
            └── domain/
                └── NguoiDung.java
            ```
            
    - **Vận hành:** Cần nắm rõ 2 cách chạy ứng dụng :   
        
        1. **Môi trường phát triển (DEV):** Chạy phương thức `main()` trực tiếp từ IDE (IntelliJ, STS) để phát triển và gỡ lỗi (debug).   
            
        2. **Môi trường Production (PROD):** Chạy lệnh build (ví dụ: `mvn clean package`) để đóng gói ứng dụng thành một file `.jar` (đã chứa Tomcat). Tải file `.jar` này lên máy chủ và chạy bằng lệnh `java -jar ten-ung-dung.jar`.   
            
- **Ví dụ ngắn (Code mẫu):**
    
    - Tạo một REST API "Hello World" đơn giản.
        
    - `@RestController`  là một annotation kết hợp. Nó bao gồm `@Controller` (đánh dấu đây là một bean xử lý request) và `@ResponseBody` (ra lệnh cho Spring tự động chuyển đổi đối tượng Java trả về thành JSON, thay vì cố gắng tìm một file view/HTML).   
        
    
    package com.congty.duan.controller;
    
    import org.springframework.web.bind.annotation.GetMapping; import org.springframework.web.bind.annotation.RestController;
    
    //  Đánh dấu đây là một REST controller @RestController public class HelloController {   
    
    ```java
      //  Ánh xạ (map) request HTTP GET tới URL "/"
      @GetMapping("/") 
      public String index() {
          // Trả về chuỗi String (sẽ được hiển thị dạng text/plain)
          return "Greetings from Spring Boot!";
      }
    
      //  Ánh xạ tới URL "/api/hello"
      @GetMapping("/api/hello")
      public Greeting sayHello() {
          // Trả về một đối tượng Java. Spring (với Jackson) 
          // sẽ tự động chuyển đổi nó thành JSON: {"message":"Hello, World!"}
          return new Greeting("Hello, World!");
      }
    
      // Sử dụng Java Record (từ Java 16+) để làm DTO (Data Transfer Object)
      record Greeting(String message) {}
    ```
    
    }
    

### 1.2 Cấp độ Trung (Mid-level Developer)

- **Khái niệm & Mục đích:** "Giải mã" các cơ chế tự động của Spring Boot. Ở cấp độ này, lập trình viên không chỉ _sử dụng_ framework mà còn phải _hiểu_ nó đang làm gì. Mục tiêu là làm chủ việc quản lý cấu hình cho nhiều môi trường một cách linh hoạt và an toàn.
    
- **Các thành phần/khái niệm quan trọng:**
    
    - **Cơ chế Auto-Configuration (Chi tiết):**
        
        - Nó không phải là phép thuật. Khi khởi động, Spring Boot đọc file `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (từ Spring Boot 3) hoặc `META-INF/spring.factories` (trước 3.0) từ tất cả các file `.jar` trên classpath.   
            
        - Các file này chứa danh sách hàng trăm class `@AutoConfiguration`. Mỗi class này chứa các định nghĩa `@Bean` (ví dụ: `DataSourceAutoConfiguration` chứa bean `DataSource`).
            
        - Các bean này được bảo vệ bởi các annotation điều kiện (`@Conditional...`).   
            
        - Các điều kiện quan trọng nhất :   
            
            1. `@ConditionalOnClass`: Chỉ tạo bean nếu một class cụ thể (ví dụ: `com.mysql.cj.jdbc.Driver`) tồn tại trên classpath.
                
            2. `@ConditionalOnProperty`: Chỉ tạo bean nếu một thuộc tính (property) trong file `.yml` được thiết lập (ví dụ: `feature.toggle.enabled=true`).   
                
            3. `@ConditionalOnMissingBean`: Chỉ tạo bean nếu lập trình viên _chưa_ tự định nghĩa một bean cùng loại (đây là cơ chế cho phép bạn "ghi đè" cấu hình mặc định).   
                
    - **`application.yml` (YAML) vs `.properties`:**
        
        - `.properties` là định dạng "phẳng" (key=value) truyền thống.   
            
        - `.yml` (YAML) được ưa chuộng hơn vì nó hỗ trợ cấu trúc phân cấp (hierarchical/nested), giúp tổ chức các cấu hình phức tạp (ví dụ: `spring.datasource...`) dễ đọc và dễ quản lý hơn.   
            
        - YAML cũng cho phép định nghĩa nhiều profile trong cùng một file, phân tách bởi dấu `---`.   
            
    - **Spring Profiles (`@Profile`):**
        
        - **Mục đích:** Cho phép định nghĩa các nhóm cấu hình (hoặc bean) khác nhau và chỉ kích hoạt chúng trong các môi trường cụ thể (ví dụ: `dev` để phát triển, `staging` để kiểm thử, `prod` cho sản phẩm).   
            
        - **Cách đặt tên file:** Spring Boot tự động nạp file `application-{profile}.yml` (ví dụ: `application-dev.yml`, `application-prod.yml`) khi profile tương ứng được kích hoạt.   
            
        - Cách kích hoạt (3 cách phổ biến) :   
            
            1. Trong `application.yml`: `spring.profiles.active=dev` (chỉ dùng cho phát triển).
                
            2. Biến môi trường (Cách chuẩn cho production/staging): `export SPRING_PROFILES_ACTIVE=prod`
                
            3. Tham số dòng lệnh (Command-line): `java -jar app.jar --spring.profiles.active=prod`.   
                
    - **`@ConfigurationProperties` (Type-safe Configuration):**
        
        - **Mục đích:** Thay vì dùng `@Value` để tiêm (inject) từng thuộc tính, `@ConfigurationProperties` cho phép liên kết (bind) một nhóm thuộc tính có cấu trúc trong file `.yml` vào một Java Object (POJO hoặc Java Record) một cách an toàn về kiểu dữ liệu.   
            
        - Nó hỗ trợ hoàn hảo cấu trúc lồng nhau (nested) của YAML và tự động chuyển đổi kiểu dữ liệu (ví dụ: `30s` thành `Duration`, `512MB` thành `DataSize`).   
            
    - **Quản lý Logging (`logback-spring.xml`):**
        
        - Spring Boot mặc định sử dụng Logback. Để tùy chỉnh nâng cao (ví dụ: log ra file, thay đổi log pattern), bạn chỉ cần tạo file `logback-spring.xml` trong `src/main/resources`. Spring Boot sẽ tự động nhận diện và sử dụng nó.   
            
- **Best-practice / Lưu ý doanh nghiệp:**
    
    - **Clean Code (Insight #1):** _Luôn luôn_ ưu tiên sử dụng `@ConfigurationProperties`  thay vì rải rác `@Value("${some.property}")`  ở khắp mọi nơi trong code.   
        
        - _Lý do:_ `@Value` vi phạm Nguyên tắc Đơn trách nhiệm (Single Responsibility Principle - SRP) vì logic cấu hình bị phân tán. `@ConfigurationProperties` gom nhóm tất cả các thuộc tính liên quan (ví dụ: `AppConfig`, `MailConfig`) vào một class duy nhất. Điều này giúp code dễ đọc, dễ quản lý, dễ validate (có thể dùng annotation `@Validated`), và dễ dàng viết unit test cho cấu hình.   
            
    - **Bảo mật (Quan trọng):** _Tuyệt đối không_ commit (đẩy lên Git) bất kỳ thông tin nhạy cảm nào (mật khẩu database, API keys, secret keys) vào file `application.yml`.   
        
        - _Cách làm (Best Practice):_ Sử dụng placeholder (biến giữ chỗ) trong `application.yml` (ví dụ: `password: ${DB_PASSWORD}`). Giá trị thực sẽ được nạp từ **Biến môi trường (Environment Variables)**  hoặc các hệ thống quản lý bí mật (Secrets Management) như HashiCorp Vault, AWS Secrets Manager, hay Google Secret Manager. Spring Boot tự động đọc biến môi trường và ghi đè lên giá trị trong file.   
            
    - **Vận hành (Insight #2):** Sử dụng `@Profile`  không chỉ để thay đổi _giá trị_ (như URL của database) mà còn để thay đổi _hành vi_ (kích hoạt các bean khác nhau).   
        
        - _Ví dụ:_ Tạo một bean `MockSmsService` (chỉ in ra console) với `@Profile("dev")`. Tạo một bean `RealSmsService` (gọi API thật) với `@Profile("prod")`. Điều này giúp môi trường `dev` chạy độc lập, nhanh chóng và không tốn chi phí API bên ngoài.   
            
    - **Clean Code (Logging):** Khi tạo file `logback-spring.xml` tùy chỉnh, _luôn_ thêm dòng `<include resource="org/springframework/boot/logging/logback/defaults.xml"/>` ở đầu file.   
        
        - _Lý do:_ Điều này đảm bảo bạn kế thừa các thiết lập mặc định thông minh của Spring Boot (như logging màu cho console, các pattern chuẩn) và chỉ ghi đè những phần cần thiết.   
            
    - **Vận hành (Logging):** Sử dụng thẻ `<springProfile>` _bên trong_ file `logback-spring.xml`. Điều này cho phép bạn tự động thay đổi mức log (ví dụ: `DEBUG` ở môi trường `dev` và `INFO` ở `prod`) dựa trên profile đang active, mà không cần thay đổi file cấu hình `application.yml`.   
        
    - **Insight #3 (Ghi đè Auto-Configuration):** Cách "chuẩn" để ghi đè một bean tự động cấu hình của Spring Boot (ví dụ: `DataSource`, `RestTemplate`) là tận dụng `@ConditionalOnMissingBean`.   
        
        - _Cách làm:_ Bạn chỉ cần _định nghĩa_ `@Bean` của riêng bạn (ví dụ: `public DataSource myDataSource() {...}`). Spring Boot, khi chạy auto-configuration, sẽ thấy bean `DataSource` của bạn đã tồn tại và _tự động rút lui_ (không tạo bean mặc định của nó nữa). Đây là cách làm đúng, thay vì cố gắng tìm cách "tắt" (disable) auto-configuration.
            
- **Ví dụ ngắn (Code mẫu):**
    
    - Sử dụng Spring Profiles và `@ConfigurationProperties`.
        
    - `pom.xml` (cần thêm dependency này để IDE hỗ trợ auto-complete cho `.yml` và `@ConfigurationProperties`):
        
        XML
        
        ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        ```
        
    - `AppConfig.java` (Sử dụng Java Record để đảm bảo tính bất biến - immutable) :   
        
        Java
        
        ```java
        import org.springframework.boot.context.properties.ConfigurationProperties;
        import java.util.List;
        
        // Sẽ bind với các key bắt đầu bằng "app.notification"
        @ConfigurationProperties(prefix = "app.notification") 
        public record AppConfig(String serviceName, List<String> adminEmails, Smtp smtp) {
        
            // Hỗ trợ nested object (đối tượng lồng nhau) 
            public record Smtp(String host, int port) {}
        }
        ```
        
    - `application.yml` (Sử dụng cấu trúc phân cấp và profiles) :   
        
        YAML
        
        ```yaml
        # Cấu hình mặc định (profile 'default')
        app:
          notification:
            service-name: "My App (Dev)"
            admin-emails:
              - "dev1@example.com"
              - "dev2@example.com"
            smtp:
              host: "dev-smtp.example.com"
              port: 587
        
        ---
        # Dấu '---' bắt đầu một document mới (profile mới) 
        # Profile "prod" 
        spring:
          config:
            activate:
              on-profile: prod
        
        app:
          notification:
            service-name: "My App (Production)"
            admin-emails:
              - "admin@example.com"
            smtp:
              host: "prod-smtp.example.com"
              port: 465
              # Mật khẩu được nạp từ biến môi trường, không hardcode
              password: ${SMTP_PASSWORD} 
        ```
        
    - Sử dụng trong Service (chỉ cần inject `AppConfig`):
        
        Java
        
        ```java
        @Service
        public class NotificationService {
            private final AppConfig appConfig;
        
            // Tiêm (inject) toàn bộ config object thay vì từng @Value
            public NotificationService(AppConfig appConfig) {
                this.appConfig = appConfig;
            }
        
            public void sendAdminNotification() {
                // Sử dụng config một cách type-safe
                System.out.println("Sending from: " + appConfig.serviceName());
                System.out.println("Using SMTP host: " + appConfig.smtp().host());
                System.out.println("Notifying admins: " + appConfig.adminEmails());
            }
        }
        ```
        

### 1.3 Cấp độ Khó (Senior Developer)

- **Khái niệm & Mục đích:** Làm chủ các kỹ thuật testing nâng cao (testing "lát cắt" - slice testing) để đảm bảo chất lượng mà không làm chậm CI/CD. Hiểu rõ các cơ chế vận hành (Actuator) để giám sát và quản lý ứng dụng. Nắm vững khả năng tùy biến sâu để tạo các "starter" (bộ khởi động) tùy chỉnh cho doanh nghiệp.
    
- **Các thành phần/khái niệm quan trọng:**
    
    - **Externalized Configuration Precedence (Thứ tự ưu tiên cấu hình):**
        
        - Đây là kiến thức bắt buộc khi vận hành. Cấu hình được định nghĩa _bên ngoài_ ứng dụng (external) _luôn_ ghi đè (override) cấu hình được đóng gói _bên trong_ (internal).   
            
        - **Hợp đồng Ops/Dev:** Đây là tính năng vận hành quan trọng, cho phép team Ops/DevOps thay đổi cấu hình (ví dụ: URL database, kích thước pool) tại môi trường production mà _không cần build và triển khai lại code_.   
            
        - Thứ tự ưu tiên (từ cao đến thấp) :   
            
            1. **Command-line arguments** (ví dụ: `java -jar app.jar --server.port=9090`).
                
            2. **OS Environment Variables** (biến môi trường, ví dụ: `SERVER_PORT=9090`).
                
            3. File config **bên ngoài** JAR (ví dụ: file `application.yml` đặt cùng thư mục với file `.jar`).   
                
            4. File config **bên trong** JAR (đóng gói trong `src/main/resources/`).   
                
        - _Lưu ý:_ Cấu hình theo profile (ví dụ: `application-prod.yml`) luôn ưu tiên hơn cấu hình mặc định (ví dụ: `application.yml`) ở _cùng một vị trí_ (ví dụ: bên ngoài JAR, `prod` > `default`).   
            
    - **Spring Boot Testing Framework:**
        
        - **`@SpringBootTest` (Integration Testing):** Khởi động _toàn bộ_ Application Context của Spring (tải tất cả bean, kết nối CSDL,...).   
            
            - _Sử dụng khi:_ Cần test sự tương tác của _nhiều_ layer (Controller -> Service -> Repository).   
                
            - Thường dùng `webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT` và `TestRestTemplate` để gọi API như một client thực thụ.   
                
        - **Test Slices (Testing "Lát cắt"):** Nhanh hơn rất nhiều vì chỉ load _một phần_ (một "lát") của context mà bạn cần test.   
            
            - `@WebMvcTest(MyController.class)`: Chỉ load web layer (chỉ `MyController`, các `Filter`, `WebSecurityConfig`, `JsonConverter`). Nó _không_ load các bean `@Service` hay `@Repository`.   
                
            - `@MockBean`: Bắt buộc sử dụng trong `@WebMvcTest` để "giả lập" (mock) các dependency (như `UserService`) mà `MyController` phụ thuộc vào.   
                
            - `@DataJpaTest`: Chỉ load persistence layer (chỉ các `@Repository`). Nó tự động cấu hình một database H2 (in-memory) và tự động rollback transaction sau mỗi test, giữ cho test độc lập.   
                
            - `@JsonTest`: Chỉ test việc (de)serialization JSON của các DTO.   
                
    - **Spring Boot Actuator:** Cung cấp các endpoint (API) sẵn sàng cho production để giám sát (monitoring) và quản lý (management) ứng dụng.   
        
        - Cần thêm `spring-boot-starter-actuator` vào `pom.xml`.   
            
        - **Expose Endpoints:** Mặc định (từ Spring Boot 2+), chỉ `/health` và `/info` được expose (lộ ra) qua web. Để mở tất cả (cho môi trường `dev`): `management.endpoints.web.exposure.include=*`.   
            
        - Các endpoints quan trọng :   
            
            - `/actuator/health`: Kiểm tra sức khỏe tổng quan của ứng dụng (ví dụ: DB còn sống, disk còn dung lượng).   
                
            - `/actuator/metrics`: Hiển thị các chỉ số chi tiết (ví dụ: `jvm.memory.used`, `http.server.requests`).   
                
            - `/actuator/loggers`: Xem và thay đổi log level (ví dụ: đổi `com.mycompany` sang `DEBUG`) mà không cần restart app.   
                
            - `/actuator/env`: Hiển thị tất cả các biến môi trường và properties đang được nạp.   
                
            - `/actuator/beans`: Liệt kê tất cả các bean đã được Spring tạo ra.   
                
            - `/actuator/heapdump`: Tải về một file "snapshot" bộ nhớ (heap dump) để phân tích memory leak.   
                
    - **Tạo Custom Auto-Configuration (Starter):**
        
        - **Mục đích:** Khi bạn muốn đóng gói một thư viện (ví dụ: thư viện logging, tracing, hoặc kết nối nội bộ) cho toàn bộ công ty sử dụng, bạn nên tạo một "starter" tùy chỉnh.   
            
        - "Hiểu cách build một starter là cách nhanh nhất để hiểu Spring Boot thực sự làm gì.".   
            
        - Cấu trúc : Một starter chuẩn doanh nghiệp thường có 2 module:   
            
            1. `my-library-autoconfigure`: Chứa code logic, các class `@AutoConfiguration` (sử dụng `@Conditional...`), và file `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (để đăng ký auto-configuration của bạn).
                
            2. `my-library-spring-boot-starter`: Một file `pom.xml` gần như rỗng, chỉ khai báo dependency đến `my-library-autoconfigure` và các thư viện bên thứ ba (ví dụ: `okhttp`) mà starter này cần.
                
- **Best-practice / Lưu ý doanh nghiệp:**
    
    - **Testing (Insight #1 - Testing Pyramid):** Lạm dụng `@SpringBootTest` là một anti-pattern trong doanh nghiệp. Nó khởi động toàn bộ context cho _mỗi_ class test, làm cho CI/CD pipeline chạy _rất chậm_.   
        
        - Quy tắc (Kim tự tháp Test) :   
            
            - **Nền (70-80%):** Unit Test (thuần JUnit/Mockito, không cần Spring) cho business logic bên trong Service.
                
            - **Giữa (15-20%):** Integration Test (dùng Test Slices). Dùng `@WebMvcTest`  để test Controller (đảm bảo validation, HTTP status, (de)serialization JSON đúng). Dùng `@DataJpaTest`  để test Repository (đảm bảo các câu query JPQL/SQL tùy chỉnh viết đúng).   
                
            - **Đỉnh (5-10%):** End-to-End Test (dùng `@SpringBootTest` ) chỉ cho các luồng nghiệp vụ quan trọng nhất (critical paths), ví dụ: luồng thanh toán.   
                
    - **Bảo mật (Actuator):** _Không bao giờ_ expose tất cả Actuator endpoints (`*`) ra public trong môi trường production.   
        
        - Rủi ro : `/actuator/env` làm lộ toàn bộ bí mật (credentials). `/actuator/heapdump` làm lộ dữ liệu nhạy cảm trong bộ nhớ. `/actuator/loggers` cho phép kẻ tấn công tắt logging. `/actuator/shutdown` cho phép tắt ứng dụng.   
            
        - **Cách làm (Best Practice):** Tích hợp Spring Security để yêu cầu `ROLE_ADMIN` mới được truy cập các endpoint này. Trong môi trường Kubernetes, chỉ expose `/actuator/health` và `/actuator/prometheus` ra bên ngoài, các endpoint nhạy cảm khác chỉ nên truy cập nội bộ (hoặc tắt).   
            
    - **Testing (Testcontainers):** Khi dùng `@DataJpaTest`, bạn bị giới hạn bởi H2 in-memory, vốn không tương thích 100% với cú pháp SQL của (ví dụ) PostgreSQL hoặc Oracle (native queries, JSON functions,...).
        
        - **Best Practice (Doanh nghiệp):** Sử dụng **Testcontainers**  để khởi động một container Docker (ví dụ: PostgreSQL, Kafka, Redis) _thật_ trong quá trình integration test. Điều này đảm bảo 100% rằng test của bạn chạy trên cùng một công nghệ với môi trường production.   
            
- **Ví dụ ngắn (Code mẫu):**
    
    - Viết một `@WebMvcTest` để test Controller và mock Service.   
        
    - `UserController.java` :   
        
        Java
        
        ```java
        @RestController
        @RequestMapping("/api/users")
        public class UserController {
            private final UserService userService;
        
            public UserController(UserService userService) { 
                this.userService = userService; 
            }
        
            @GetMapping("/{id}")
            public User getUser(@PathVariable Long id) {
                // Giả sử findById ném exception nếu không tìm thấy
                return userService.findById(id); 
            }
        }
        ```
        
    - `UserControllerTest.java` :   
        
        Java
        
        ```java
        import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
        import org.springframework.boot.test.mock.mockito.MockBean;
        import org.springframework.test.web.servlet.MockMvc;
        import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
        import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
        import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
        import static org.mockito.Mockito.when;
        
        //  Chỉ load layer web và cụ thể là UserController
        @WebMvcTest(UserController.class) 
        class UserControllerTest {
        
            @Autowired
            private MockMvc mockMvc; //  Spring Boot tự cấu hình MockMvc để giả lập request
        
            // [49, 50] Mock Service, vì @WebMvcTest không load @Service
            @MockBean 
            private UserService userService;
        
            @Test
            void getUser_ReturnsUser_WhenFound() throws Exception {
                // 1. Arrange: Dạy cho mock service
                User user = new User(1L, "Alice");
                when(userService.findById(1L)).thenReturn(user);
        
                // 2. Act & 3. Assert: Thực thi request và kiểm tra kết quả JSON
                mockMvc.perform(get("/api/users/1")
                      .accept(MediaType.APPLICATION_JSON))
                      .andExpect(status().isOk()) // Kiểm tra HTTP 200
                      .andExpect(jsonPath("$.name").value("Alice")); // Kiểm tra nội dung JSON
            }
        
            @Test
            void getUser_ReturnsNotFound_WhenMissing() throws Exception {
                // 1. Arrange: Dạy mock service ném exception
                when(userService.findById(99L)).thenThrow(new UserNotFoundException());
        
                // 2. Act & 3. Assert
                mockMvc.perform(get("/api/users/99")
                      .accept(MediaType.APPLICATION_JSON))
                      .andExpect(status().isNotFound()); // Kiểm tra HTTP 404
            }
        }
        ```
        

### 1.4 Cấp độ Chuyên sâu (Architect/Principal)

- **Khái niệm & Mục đích:** Tối ưu hóa ứng dụng cho môi trường Cloud-Native (đặc biệt là Kubernetes), tập trung vào hiệu năng (startup, memory), khả năng phục hồi (resilience), và khả năng quan sát (observability). Đây là cấp độ của Architect và Principal Engineer, nơi các quyết định ảnh hưởng trực tiếp đến chi phí và độ ổn định vận hành.
    
- **Các thành phần/khái niệm quan trọng:**
    
    - **Custom Health Indicators (Chỉ báo Sức khỏe Tùy chỉnh):**
        
        - **Mục đích:** Endpoint `/actuator/health` mặc định chỉ kiểm tra các thành phần cơ bản (ví dụ: `DataSource`, `DiskSpace`). Trong microservices, "sức khỏe" còn phụ thuộc vào các dịch vụ bên ngoài (API bên thứ ba, Message Queue, cache).   
            
        - **Cách làm:** Tạo một `@Component` implement `HealthIndicator` và override phương thức `health()`. Trả về `Health.up()` hoặc `Health.down()` kèm chi tiết.   
            
        - **Health Groups (Nhóm Sức khỏe):** Gán các `HealthIndicator` tùy chỉnh vào các "group" (nhóm) do Kubernetes định nghĩa, như `readiness` hoặc `liveness`. Đây là chìa khóa để tích hợp với K8s.   
            
    - **Tối ưu hóa hiệu năng (Startup & Memory):**
        
        - **Lazy Initialization:** Kích hoạt bằng `spring.main.lazy-initialization=true`.   
            
        - _Trade-off (Đánh đổi):_ Giúp ứng dụng khởi động _nhanh hơn_ (tốt cho dev, test, và các môi trường FaaS/Serverless). Tuy nhiên, nó làm cho request _đầu tiên_ (khi bean được tạo) _chậm hơn_ đáng kể và có thể ẩn các lỗi (ví dụ: `ClassNotFoundException`, lỗi cấu hình) cho đến khi runtime thay vì báo lỗi ngay khi khởi động.   
            
        - **JVM Tuning (cho Container):** Mặc định, JVM không nhận biết giới hạn memory của container (do Docker/K8s đặt ra).   
            
            - Best Practice : Luôn set `XX:MaxRAMPercentage` (ví dụ: `-XX:MaxRAMPercentage=80.0`) để JVM biết nó chỉ được dùng 80% memory của container, 20% còn lại cho HĐH và các tiến trình khác.   
                
            - Best Practice : Giảm bộ nhớ stack cho mỗi thread (ví dụ: `-Xss512k`) nếu ứng dụng của bạn không có đệ quy sâu.   
                
            - Best Practice : Rà soát và loại bỏ các `starter` không cần thiết. Mỗi `starter` mang vào nhiều dependency và auto-configuration, làm tăng dấu chân bộ nhớ (memory footprint).   
                
    - **Graceful Shutdown (Tắt ứng dụng an toàn):**
        
        - **Mục đích:** Khi Kubernetes muốn "giết" một Pod (gửi tín hiệu `SIGTERM`), ứng dụng cần phải: 1) Ngừng nhận request mới, và 2) Xử lý nốt các request đang chạy (in-flight) để tránh mất dữ liệu hoặc trả về lỗi 503 cho client.   
            
        - **Cấu hình:** Bật bằng `server.shutdown=graceful` (đây là mặc định từ Spring Boot 2.3+). Quan trọng là đặt thời gian chờ: `spring.lifecycle.timeout-per-shutdown-phase=20s`.   
            
        - _Lưu ý (Quan trọng):_ Giá trị `timeout-per-shutdown-phase` (ví dụ: 20s) phải _nhỏ hơn_ `terminationGracePeriodSeconds` (ví dụ: 30s) trong cấu hình Pod của Kubernetes.   
            
    - **Nguyên tắc 12-Factor App (chọn lọc):**
        
        - Đây là kim chỉ nam cho việc xây dựng các ứng dụng cloud-native, SaaS. Spring Boot được thiết kế để tuân thủ các nguyên tắc này.   
            
        - **III. Config:** Lưu cấu hình trong biến môi trường (Environment Variables). Spring Boot tự động làm điều này.   
            
        - **VI. Processes:** Ứng dụng phải _stateless_ (phi trạng thái). Không lưu trữ bất cứ state nào (session, file) trên local filesystem của container (vì container là "ephemeral" - có thể bị xóa bất cứ lúc nào). State phải được lưu trữ ở backing service (như DB, Redis).   
            
        - **IX. Disposability:** Tối ưu khả năng "dùng một lần" (khởi động nhanh và tắt an toàn). Spring Boot hỗ trợ điều này qua Lazy Initialization và Graceful Shutdown.   
            
- **Khi triển khai thực tế (Microservices & Cloud-Native):**
    
    - **Containerization (Đóng gói Docker Image):** So sánh 3 cách:
        
        - `Dockerfile`: Cung cấp kiểm soát tuyệt đối, nhưng thủ công, dễ sai sót và tạo ra các image "béo" (không tối ưu layer).   
            
        - `Jib` (Google): Tối ưu cho Java. Không cần Docker daemon (build thẳng từ Maven/Gradle). Tự động chia layer thông minh (dependencies, resources, classes).   
            
        - `Cloud Native Buildpacks` (lệnh `mvn spring-boot:build-image`): Tự động hóa cao, hỗ trợ đa ngôn ngữ, cũng chia layer thông minh.   
            
        - **Thực tiễn Doanh nghiệp:** Jib  và Buildpacks  _vượt trội_ hơn `Dockerfile` cho các ứng dụng Spring Boot. Lý do: Chúng tự động chia image thành các layer (lớp). Điều này cho phép CI/CD build nhanh hơn (chỉ build lại layer code đã thay đổi) và cho phép _vá lỗi bảo mật_ cho OS/JVM (base layer) mà _không cần build lại_ code của ứng dụng (application layer).   
            
    - **Kubernetes (K8s) Integration:**
        
        - **Cấu hình (ConfigMaps & Secrets):** Có 2 cách :   
            
            1. **Cách đơn giản:** Mount Secret/ConfigMap vào biến môi trường (ENV) của Pod. `application.yml` đọc bằng placeholder (ví dụ: `password: ${DB_PASSWORD}`).   
                
            2. **Cách Cloud-native:** Dùng dependency `spring-cloud-kubernetes-config`. Ứng dụng Spring Boot sẽ tự động nạp ConfigMap/Secret làm `PropertySource` (giống như Spring Cloud Config).   
                
        - **Health Probes (Liveness & Readiness):**
            
            - **Bối cảnh:** Đây là cách ứng dụng "nói chuyện" với K8s về trạng thái của nó. Cấu hình sai sẽ gây ra "vòng xoáy chết chóc" (crash loop) hoặc rớt traffic.   
                
            - **`livenessProbe`** (trỏ đến `/actuator/health/liveness`): "Tôi còn sống không?" Nếu endpoint này trả về lỗi, K8s sẽ _giết và khởi động lại_ (restart) Pod. Dùng cho các lỗi không thể phục hồi (ví dụ: deadlock, hết bộ nhớ).   
                
            - **`readinessProbe`** (trỏ đến `/actuator/health/readiness`): "Tôi sẵn sàng nhận traffic không?" Nếu endpoint này trả về lỗi, K8s sẽ _giữ Pod sống_ nhưng _cắt traffic_ (gỡ khỏi Load Balancer). Dùng cho các trạng thái bận tạm thời (ví dụ: đang khởi động, service phụ thuộc đang bị chậm).   
                
            - _Best Practice:_ Phải gán các `HealthIndicator` tùy chỉnh (S105) vào đúng group (S103). Ví dụ: Mất kết nối DB có thể làm `liveness` fail, nhưng API bên thứ 3 bị chậm chỉ nên làm `readiness` fail (để K8s không restart app một cách vô ích).   
                
    - **Monitoring (Metrics) với Prometheus/Grafana:**
        
        - **Luồng hoạt động (Data Flow):**
            
            1. (App) Thêm `spring-boot-starter-actuator` + `micrometer-registry-prometheus`.   
                
            2. (App) Cấu hình để expose (lộ ra) endpoint `/actuator/prometheus`.   
                
            3. (Prometheus Server) Cấu hình để định kỳ _thu thập_ (scrape) metrics từ endpoint đó.   
                
            4. (Grafana) Cấu hình để kết nối tới Prometheus, dùng PromQL để truy vấn và _vẽ biểu đồ_ (dashboards).   
                
    - **Distributed Tracing (Tracing) với Micrometer/Zipkin:**
        
        - **Mục đích:** Theo dõi một request khi nó đi qua nhiều microservice (ví dụ: Request A -> Service 1 -> Service 2 -> Service 3).   
            
        - **Luồng hoạt động:** Spring Boot 3 dùng **Micrometer Tracing** (đã thay thế Spring Cloud Sleuth).   
            
        - Cần 2 loại dependency:
            
            1. Bridge (Cầu nối API): `micrometer-tracing-bridge-brave` (hoặc `-otel`).   
                
            2. Exporter (Trình gửi): `zipkin-reporter-brave` (hoặc `opentelemetry-exporter-zipkin`) để gửi dữ liệu đến Zipkin Server.   
                
        - **Cấu hình:** `management.tracing.sampling.probability=1.0` (để trace mọi request, mặc định là 0.1 - 10%)  và `spring.zipkin.base-url=http://zipkin-server:9411`.   
            
    - **Observability (Khả năng quan sát):**
        
        - Ba trụ cột của Observability là **Metrics** (chỉ số), **Tracing** (truy vết), và **Logging** (logs).   
            
        - Khi Micrometer Tracing được kích hoạt, nó tự động tạo ra `traceId` (định danh cho toàn bộ request) và `spanId` (định danh cho từng bước).   
            
        - **Best Practice (Doanh nghiệp):** Cấu hình Logback (sử dụng MDC) để _tự động thêm_ `traceId` và `spanId` vào _tất cả_ các dòng log mà ứng dụng ghi ra.   
            
        - **Hệ quả (MTTR):** Khi có sự cố, SRE (Site Reliability Engineer) sẽ:
            
            1. Nhận cảnh báo (Alert) từ **Metrics** (Prometheus) (ví dụ: tỷ lệ lỗi HTTP 500 tăng).
                
            2. Xem **Tracing** (Zipkin) để xác định `traceId` của các request lỗi và thấy request đó thất bại ở service nào.   
                
            3. Lấy `traceId` đó và tìm kiếm trong hệ thống **Logging** (ví dụ: ELK Stack hoặc Loki) để xem _chính xác_ log lỗi chi tiết (bao gồm cả stack trace) từ tất cả các service liên quan.   
                
        - Việc liên kết được 3 trụ cột này giúp giảm MTTR (Mean Time to Resolution - Thời gian trung bình để khắc phục) từ "hàng giờ" xuống "hàng phút".   
            
- **Ví dụ ngắn (Cấu hình K8s Probes):**
    
    - Cấu hình file `deployment.yaml` của Kubernetes để trỏ vào các endpoint của Actuator Health Group.
        
        YAML
        
        ```yaml
        # Trích đoạn file deployment.yaml của Kubernetes
        spec:
          containers:
            - name: my-spring-boot-app
              image: my-app:latest
              ports:
                - containerPort: 8080 # Cổng ứng dụng
        
              #  Liveness: Nếu fail, K8s sẽ restart container
              # Dùng để phát hiện các lỗi nghiêm trọng, không thể phục hồi
              livenessProbe:
                httpGet:
                  path: /actuator/health/liveness
                  port: 8080
                initialDelaySeconds: 15 # Chờ 15s sau khi container start
                periodSeconds: 20       # Kiểm tra mỗi 20s
                failureThreshold: 3     # Restart sau 3 lần thất bại
        
              #  Readiness: Nếu fail, K8s sẽ ngừng gửi traffic
              # Dùng để báo hiệu app tạm thời bận hoặc chưa sẵn sàng
              readinessProbe:
                httpGet:
                  path: /actuator/health/readiness
                  port: 8080
                initialDelaySeconds: 5  # Check sớm hơn liveness
                periodSeconds: 10       # Kiểm tra mỗi 10s
        ```