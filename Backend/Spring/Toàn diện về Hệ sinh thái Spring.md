## Phần I: Nền tảng Spring Core: Inversion of Control (IoC) và Dependency Injection (DI)

Trước khi tìm hiểu về Spring Boot, điều bắt buộc là phải hiểu các nguyên tắc cốt lõi mà nó được xây dựng trên đó.

### 1.1 Inversion of Control (IoC) là gì?

Inversion of Control (IoC) là một nguyên tắc thiết kế, nơi quyền kiểm soát việc tạo đối tượng và quản lý vòng đời của chúng được "đảo ngược" (inverted) từ lập trình viên sang một framework. Trong lập trình Java truyền thống, bạn tự tạo các đối tượng: `MyService service = new MyServiceImpl();`. Với IoC, bạn chỉ định nghĩa các đối tượng, và framework (container) sẽ chịu trách nhiệm khởi tạo và "kết nối" (wire) chúng lại với nhau.   

### 1.2 IoC Container (Spring Container)

IoC Container là thành phần trung tâm của Spring Framework, chịu trách nhiệm triển khai IoC. Nó đọc siêu dữ liệu cấu hình (configuration metadata) — dù là từ file XML, Java Config (sử dụng `@Configuration`) hay các Annotation (chú thích) — để biết phải tạo, cấu hình và lắp ráp đối tượng nào.   

Spring cung cấp hai loại container chính :   

1. **BeanFactory:** Container cơ bản, cung cấp hỗ trợ Dependency Injection. Nó khởi tạo các bean một cách "lười biếng" (lazily), tức là chỉ khi được yêu cầu.   
    
2. **ApplicationContext:** Là một sub-interface của `BeanFactory` và là loại được sử dụng phổ biến nhất hiện nay. Nó cung cấp thêm nhiều chức năng cấp doanh nghiệp như tích hợp Spring AOP, xử lý message (cho quốc tế hóa), và tự động khởi tạo tất cả các bean "singleton" ngay khi khởi động (eagerly).   
    

### 1.3 Spring Beans là gì?

Trong Spring, các đối tượng được quản lý bởi IoC container (tức là được khởi tạo, lắp ráp và quản lý vòng đời) được gọi là **Beans**. Chúng là xương sống của ứng dụng của bạn.   

### 1.4 Dependency Injection (DI)

Dependency Injection (DI) là một _pattern_ (mẫu thiết kế) cụ thể được sử dụng để triển khai nguyên tắc IoC. Thay vì một đối tượng tự tìm kiếm hoặc tạo ra các phụ thuộc (dependencies) của nó, các phụ thuộc này được _tiêm_ (inject) vào đối tượng bởi container.   

Hãy tưởng tượng một lớp `Car`. Lớp `Car` cần các đối tượng `Wheels` và `Battery`.

- **Không có DI:** Lớp `Car` sẽ tự tạo các phụ thuộc:
    
    Java
    
    ```
    public class Car {
        private Wheels wheels = new Wheels(); // Car tự tạo
        private Battery battery = new Battery(); // Car tự tạo
    }
    ```
    
- **Có DI:** Lớp `Car` nhận các phụ thuộc từ bên ngoài:
    
    Java
    
    ```
    public class Car {
        private Wheels wheels;
        private Battery battery;
        // Phụ thuộc được "tiêm" qua hàm khởi tạo (constructor)
        public Car(Wheels wheels, Battery battery) {
            this.wheels = wheels;
            this.battery = battery;
        }
    }
    ```
    

Lợi ích là lớp `Car` không còn bị phụ thuộc cứng vào một triển khai `Wheels` cụ thể, giúp mã nguồn trở nên linh hoạt và dễ kiểm thử (testing) hơn.   

### 1.5 Các loại DI và Best Practice

Có ba cách chính để tiêm phụ thuộc trong Spring:

1. **Field Injection (Tiêm qua trường - KHÔNG NÊN DÙNG):**
    
    Java
    
    ```
    @Autowired
    private MyService myService;
    ```
    
    Đây là cách phổ biến trong các hướng dẫn cũ nhưng được coi là một anti-pattern trong thực tế doanh nghiệp.
    
    - **Tại sao tệ:** Nó che giấu các phụ thuộc của lớp. Nó làm cho việc kiểm thử (unit test) trở nên khó khăn, vì bạn phải sử dụng reflection để gán các mock object. Nó không cho phép bạn khai báo trường là `final`, vi phạm tính bất biến (immutability). Bạn có nguy cơ `NullPointerException` nếu cố gắng sử dụng đối tượng bên ngoài Spring context.   
        
2. **Setter Injection (Tiêm qua Setter):**
    
    Java
    
    ```
    private MyService myService;
    
    @Autowired
    public void setMyService(MyService myService) {
        this.myService = myService;
    }
    ```
    
    Cách này tốt hơn Field Injection và hữu ích cho các phụ thuộc _không bắt buộc_ (optional), cho phép thay đổi phụ thuộc trong quá trình chạy. Tuy nhiên, nó vẫn cho phép đối tượng ở trạng thái không hoàn chỉnh (các phụ thuộc có thể là `null`).   
    
3. **Constructor Injection (Tiêm qua Hàm khởi tạo - BEST PRACTICE):**
    
    Java
    
    ```
    private final MyService myService;
    
    @Autowired // (Không bắt buộc nếu chỉ có 1 constructor)
    public MyController(MyService myService) {
        this.myService = myService;
    }
    ```
    
    Đây là phương pháp được nhóm Spring _chính thức khuyến nghị_.   
    
    - **Tại sao tốt:**
        
        - **Immutability (Bất biến):** Các phụ thuộc có thể được khai báo là `final`, đảm bảo chúng không bao giờ bị thay đổi sau khi đối tượng được tạo.   
            
        - **Null Safety:** Đảm bảo đối tượng luôn được khởi tạo với tất cả các phụ thuộc _bắt buộc_. Không thể có một đối tượng `MyController` tồn tại mà không có `MyService`.   
            
        - **Testability (Khả năng Kiểm thử):** Cực kỳ dễ kiểm thử. Bạn không cần Spring context. Chỉ cần gọi `new MyController(mockMyService)` trong unit test.   
            
        - **Thiết kế Tốt:** Nó buộc bạn phải suy nghĩ về các phụ thuộc của lớp. Một hàm khởi tạo có 10 tham số là một "code smell" (dấu hiệu mã xấu) rõ ràng, cho thấy lớp của bạn đang làm quá nhiều việc (vi phạm Single Responsibility Principle).   
            

## Phần II: Spring Boot: Tăng tốc Phát triển Ứng dụng

Spring Boot không phải là một framework mới. Nó là một "lớp" (layer) nằm trên Spring Core, giúp giải quyết các vấn đề về cấu hình và thiết lập dự án.

### 2.1 Spring Boot Starters: Quản lý Phụ thuộc

- **Vấn đề:** Để xây dựng một ứng dụng web với Spring MVC, bạn cần `spring-webmvc`, `jackson-databind` (cho JSON), `tomcat-embed` (cho server), `hibernate-validator` (cho validation), và phải đảm bảo tất cả các phiên bản này _tương thích_ với nhau.
    
- **Giải pháp (Starters):** Starters là các tệp POM (Project Object Model) mô tả phụ thuộc tiện lợi (convenient dependency descriptors). Chúng _không_ phải là code, mà là một "gói" các phụ thuộc thường được sử dụng cùng nhau.   
    
- **Ví dụ:** Thay vì thêm 5-10 dependency, bạn chỉ cần thêm _một_:
    
    XML
    
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    ```
    
    Starter này sẽ tự động kéo `spring-webmvc`, `jackson`, `tomcat` (mặc định), và `spring-boot-starter-logging` (với Logback) vào dự án của bạn với các phiên bản đã được kiểm thử và tương thích. Chúng là "one-stop-shop" (cửa hàng một điểm dừng) cho các nhu cầu của bạn.   
    

### 2.2 Spring Boot Auto-Configuration: Cấu hình Tự động

Đây là "phép thuật" chính của Spring Boot, tuân theo triết lý "convention over configuration" (ưu tiên quy ước hơn cấu hình).   

- **Vấn đề:** Bạn đã thêm `spring-boot-starter-data-jpa` và `mysql-connector-j`. Bạn _biết_ mình sẽ cần một `DataSource` bean và một `EntityManagerFactory` bean. Theo truyền thống, bạn phải viết một lớp `@Configuration` dài dòng để định nghĩa các bean này.
    
- **Giải pháp (Auto-Configuration):** Spring Boot tự động cấu hình ứng dụng của bạn dựa trên các JAR có trong classpath.   
    

**Luồng hoạt động (Chain of Thought):**

1. Bạn thêm annotation `@SpringBootApplication` vào lớp chính. Annotation này thực chất bao gồm `@EnableAutoConfiguration`.   
    
2. Khi khởi động, Spring Boot quét classpath để tìm tệp `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (trong các phiên bản cũ hơn là `META-INF/spring.factories`).   
    
3. Tệp này chứa danh sách hàng trăm lớp cấu hình tự động, ví dụ `DataSourceAutoConfiguration`.
    
4. Spring Boot duyệt qua danh sách này và đánh giá từng lớp. Mỗi lớp sử dụng các annotation `@Conditional`.   
    
5. _Ví dụ cho `DataSourceAutoConfiguration`_:
    
    - `@ConditionalOnClass(DataSource.class)`: Nó kiểm tra: "Lớp `DataSource` có tồn tại trong classpath không?" (Có, vì bạn đã thêm `starter-data-jpa`).   
        
    - `@ConditionalOnMissingBean(DataSource.class)`: Nó kiểm tra: "Lập trình viên (bạn) đã _tự mình_ định nghĩa một bean `DataSource` nào chưa?".   
        
6. **Kết quả:** Nếu câu trả lời cho (a) là CÓ và (b) là KHÔNG, Spring Boot sẽ tự động tạo một `DataSource` bean mặc định cho bạn, sử dụng các thuộc tính trong `application.properties` (như `spring.datasource.url`). Nếu bạn tự định nghĩa bean `DataSource` của mình, cấu hình tự động sẽ "lùi lại" (backs off) và không làm gì cả.   
    

### 2.3 Gỡ lỗi (Debug) Auto-Configuration

Khi cấu hình tự động không hoạt động như mong đợi, có hai cách chính để "nhìn" vào bên trong:

1. **Báo cáo Điều kiện (Conditions Report):** Chạy ứng dụng với cờ `debug=true` trong `application.properties`  hoặc bật "Enable debug output" trong Run Configuration của IDE. Khi khởi động, Spring Boot sẽ in ra một báo cáo chi tiết về tất cả các lớp auto-configuration, cho biết lớp nào _đã khớp_ (matched) và lớp nào _không khớp_ (did not match) và lý do tại sao.   
    
2. **Actuator `/conditions` Endpoint:** Nếu bạn đã bao gồm `spring-boot-starter-actuator`, bạn có thể kích hoạt endpoint `/conditions` (bằng cách thêm `management.endpoints.web.exposure.include=conditions` vào `application.properties` ). Khi truy cập endpoint này, nó sẽ hiển thị một JSON chi tiết về trạng thái của tất cả các điều kiện, tương tự như báo cáo debug, nhưng ở thời gian chạy.   
    

### 2.4 (Nâng cao) Tạo Custom Spring Boot Starter

Kết hợp hai khái niệm trên, bạn có thể tạo Starter của riêng mình cho thư viện nội bộ của công ty.   

1. **Module Auto-Configure:** Tạo một dự án (ví dụ: `my-service-autoconfigure`) chứa lớp `MyServiceAutoConfiguration` (được chú thích bằng `@Configuration` và `@ConditionalOn...`).   
    
2. **Đăng ký:** Trong module này, tạo tệp `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` và thêm tên đầy đủ của lớp cấu hình của bạn vào đó (ví dụ: `com.mycompany.MyServiceAutoConfiguration`).   
    
3. **Module Starter:** Tạo một dự án thứ hai (ví dụ: `my-service-spring-boot-starter`). Dự án này _không_ chứa code Java, chỉ có một tệp `pom.xml`. Tệp POM này khai báo các dependency:   
    
    - `my-service-autoconfigure` (bạn vừa tạo)
        
    - Thư viện lõi (ví dụ: `my-service-lib`)
        
    - Bất kỳ dependency nào khác mà `my-service` cần.   
        
4. **Sử dụng:** Các ứng dụng khác bây giờ chỉ cần thêm một dependency duy nhất là `my-service-spring-boot-starter`, và mọi thứ sẽ được tự động cấu hình.
    

## Phần III: Tương tác Dữ liệu với Spring Data JPA và Hibernate

Đây là một trong những phần quan trọng nhất của một ứng dụng doanh nghiệp.

### 3.1 JPA, Hibernate, và Spring Data JPA

Việc phân biệt rõ ràng các công nghệ này là rất quan trọng :   

- **JPA (Java Persistence API):** Là một **đặc tả (specification)** của Java (nay là Jakarta EE). Nó chỉ là một bộ các interface (như `EntityManager`, `Entity`) và các annotation (`@Entity`, `@Id`, `@OneToMany`). Nó định nghĩa một API chuẩn cho Object-Relational Mapping (ORM).   
    
- **Hibernate:** Là một **triển khai (implementation)** của đặc tả JPA. Nó là _công cụ_ (engine) ORM thực sự, đọc các annotation JPA của bạn và thực hiện các lệnh SQL tương ứng.   
    
- **Spring Data JPA:** Là một **lớp trừu tượng (abstraction)** nằm _phía trên_ JPA. Mục tiêu của nó là giảm thiểu tối đa lượng code boilerplate (mã lặp đi lặp lại) cần thiết để triển khai tầng truy cập dữ liệu (Data Access Layer - DAL).   
    

### 3.2 Spring Data Repository

Đây là tính năng cốt lõi của Spring Data JPA. Nó cho phép bạn tạo ra các DAL mà không cần viết bất kỳ một lớp triển khai nào.

**Cơ chế hoạt động:**

1. **Định nghĩa Interface:** Bạn chỉ cần định nghĩa một interface kế thừa từ một trong các interface cơ sở của Spring Data, ví dụ: `JpaRepository`.   
    
    Java
    
    ```
    public interface UserRepository extends JpaRepository<User, Long> { }
    ```
    
2. **Tạo Proxy tại Runtime:** Khi ứng dụng khởi động, Spring Data JPA sẽ phát hiện interface này. Nó _không_ tạo code. Thay vào đó, nó sử dụng một `RepositoryFactory` (cụ thể là `JpaRepositoryFactory`) để tạo ra một **JDK Proxy** động.   
    
3. **Chặn Lệnh gọi (Method Interception):** Khi bạn gọi `userRepository.save(user)`, lời gọi này được chuyển đến proxy. Proxy này ủy quyền cho một `MethodInterceptor` (như `QueryExecutorMethodInterceptor`). Interceptor này sẽ xem xét phương thức được gọi và quyết định thực thi logic nào.   
    
4. **Triển khai Chung (Base Implementation):** Đối với các phương thức CRUD tiêu chuẩn (như `save`, `findById`), interceptor sẽ ủy quyền cho một bean triển khai cơ sở gọi là `SimpleJpaRepository`, nơi chứa logic `EntityManager` thực sự (ví dụ: `entityManager.persist(user)`).   
    

**Các Interface Repository chính:**

- `Repository<T, ID>`: Interface đánh dấu.
    
- `CrudRepository<T, ID>`: Cung cấp các phương thức CRUD cơ bản (Create, Read, Update, Delete).   
    
- `PagingAndSortingRepository<T, ID>`: Kế thừa `CrudRepository` và thêm các phương thức để phân trang (`Pageable`) và sắp xếp (`Sort`).   
    
- `JpaRepository<T, ID>`: Kế thừa `PagingAndSortingRepository` và thêm các tính năng dành riêng cho JPA như `flush()` (đẩy các thay đổi vào DB) và `saveAll()` (lưu theo batch).   
    

### 3.3 Vòng đời Entity (Hibernate Entity Lifecycle)

Không hiểu vòng đời entity là nguyên nhân phổ biến nhất gây ra các bug liên quan đến JPA. Mọi entity object tồn tại ở một trong 4 trạng thái, liên quan đến một "Persistence Context" (do `EntityManager` quản lý).   

1. **Transient (hoặc New):** Đối tượng vừa được tạo bằng `new User()`. Nó chưa được liên kết với Persistence Context và không có bản ghi tương ứng trong CSDL.   
    
2. **Managed (hoặc Persistent):** Đối tượng đang được _quản lý_ bởi Persistence Context. Điều này xảy ra khi bạn tải nó từ CSDL (`findById`) hoặc khi bạn vừa lưu nó (`persist`).   
    
    - **Tính năng quan trọng:** "Automatic Dirty Checking". Bất kỳ thay đổi nào bạn thực hiện trên một đối tượng _Managed_ (ví dụ: `user.setName("new name")`) sẽ được Hibernate _tự động_ phát hiện và đồng bộ hóa với CSDL khi transaction kết thúc. Bạn _không_ cần phải gọi `userRepository.save(user)` lần nữa.   
        
3. **Detached:** Đối tượng _đã từng_ được quản lý (Managed), nhưng Persistence Context (phiên) đã bị đóng (ví dụ: request web kết thúc, transaction hoàn thành). Hibernate không còn theo dõi các thay đổi của nó nữa.   
    
4. **Removed:** Đối tượng đã được đưa vào Persistence Context và được đánh dấu để xóa (`entityManager.remove()`). Lệnh `DELETE` thực tế sẽ được thực thi khi transaction kết thúc.   
    

**Best Practice: `persist()` vs `merge()` (Theo Vlad Mihalcea)**

Các phương thức `save`, `update`, `saveOrUpdate` là các API của Hibernate. Best practice là tuân thủ API của JPA chuẩn:

- **`persist(entity)`:** Chỉ dùng cho các entity ở trạng thái **Transient** (mới). Nó chuyển entity sang trạng thái **Managed**. Đây là phương thức đúng để tạo một bản ghi mới.   
    
- **`merge(entity)`:** Dùng cho các entity ở trạng thái **Detached**. Nó _không_ làm cho entity detached trở thành managed. Thay vào đó, nó:   
    
    1. Tìm entity tương ứng trong CSDL (hoặc Persistence Context).
        
    2. _Sao chép_ (copy) các trường từ entity detached của bạn sang entity managed tương ứng.
        
    3. Trả về bản sao _managed_ đó. **Cảnh báo:** Luôn sử dụng đối tượng được trả về từ `merge`. Ví dụ: `ManagedUser = entityManager.merge(detachedUser)`.   
        

### 3.4 Các mối quan hệ (Relationships) trong JPA

JPA cho phép bạn mô hình hóa các mối quan hệ CSDL (`@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`).   

- **Bên Sở hữu (Owning Side) vs. Bên Đảo ngược (Inverse Side):**
    
    - Trong một mối quan hệ, chỉ có _một_ bên chịu trách nhiệm quản lý khóa ngoại (foreign key). Đây là bên sở hữu.   
        
    - **`@JoinColumn`:** Được dùng ở bên sở hữu. Nó chỉ định cột khóa ngoại trong bảng của entity đó.   
        
    - **`mappedBy`:** Được dùng ở bên đảo ngược (inverse side) (thường là bên `@OneToMany` hoặc `@OneToOne` không sở hữu). Giá trị của `mappedBy` là tên của _trường_ (field) ở bên sở hữu.   
        
- **FetchType: LAZY vs EAGER**
    
    - `FetchType.EAGER` (Háo hức): Tải tất cả dữ liệu liên quan (ví dụ: các `Posts` của `User`) _ngay lập tức_ trong cùng một truy vấn. Đây là mặc định cho các quan hệ `...@ToOne`.   
        
    - `FetchType.LAZY` (Lười biếng): Chỉ tải entity chính (`User`). Đối với trường `posts`, Hibernate sẽ tạo một "proxy" (đối tượng giả). Dữ liệu `posts` thực sự chỉ được tải từ CSDL khi bạn _truy cập_ nó lần đầu tiên (ví dụ: `user.getPosts().size()`). Đây là mặc định cho các quan hệ `...@ToMany`.   
        
- **Best Practice (Theo Thorben Janssen & Vlad Mihalcea):**
    
    - **LUÔN LUÔN sử dụng `FetchType.LAZY` cho TẤT CẢ các mối quan hệ**.   
        
    - **Tại sao `EAGER` tệ:** `FetchType.EAGER` là một anti-pattern. Nó có vẻ tiện lợi nhưng là nguồn gốc của các vấn đề hiệu suất nghiêm trọng. Khi bạn tải một entity, nó sẽ tải tất cả các entity liên quan EAGER, và các entity đó lại tải các entity liên quan EAGER của chúng, dẫn đến việc tải _nửa cơ sở dữ liệu_ chỉ cho một truy vấn đơn giản.   
        
    - `LAZY` cho bạn quyền kiểm soát. Khi bạn _thực sự_ cần dữ liệu liên quan, bạn có thể yêu cầu tải nó một cách rõ ràng _trên cơ sở từng truy vấn_ (per-query basis) bằng cách sử dụng `JOIN FETCH` hoặc `@EntityGraph`.   
        

### 3.5 Phương thức Truy vấn (Query Methods)

Spring Data JPA cung cấp nhiều cách để truy vấn dữ liệu:

1. **Query Derivation (Tạo truy vấn từ tên phương thức):** Spring Data tự động phân tích tên phương thức của bạn trong interface Repository.   
    
    - Ví dụ: `List<User> findByEmailAddressAndLastname(String email, String lastname);`
        
    - Spring sẽ tự động tạo ra câu lệnh JPQL: `select u from User u where u.emailAddress =?1 and u.lastname =?2`.   
        
2. **`@Query` (JPQL):** Cho các truy vấn phức tạp hơn mà không thể biểu diễn qua tên phương thức.
    
    - Java
        
        ```
        @Query("SELECT u FROM User u WHERE u.username LIKE CONCAT('%', :username, '%')")
        List<User> findByUsernameContaining(@Param("username") String username);
        ```
        
    
    .   
    
    - Sử dụng `@Modifying` cùng với `@Query` cho các hoạt động `UPDATE` hoặc `DELETE`.   
        
3. **`@Query(nativeQuery = true)`:** Cho phép bạn viết SQL thuần túy (plain SQL).   
    
4. **Phân trang (Pagination):** Chỉ cần thêm một tham số kiểu `Pageable` vào bất kỳ phương thức nào.
    
    - `Page<User> findByStatus(Status status, Pageable pageable);`
        
    - Spring Data sẽ tự động thêm logic `LIMIT` và `OFFSET` (hoặc tương đương) vào truy vấn và trả về một đối tượng `Page`, đối tượng này chứa cả danh sách kết quả cho trang hiện tại và thông tin về tổng số trang, tổng số bản ghi.   
        

### 3.6 Vấn đề N+1 Query và Cách giải quyết (Rất quan trọng)

Đây là vấn đề hiệu suất phổ biến và nghiêm trọng nhất trong các ứng dụng ORM.

- **Định nghĩa:** Vấn đề N+1 xảy ra khi ứng dụng thực hiện **1** truy vấn để lấy danh sách các entity cha, và sau đó thực hiện **N** truy vấn bổ sung để lấy dữ liệu liên quan cho mỗi entity cha trong số đó.   
    
- **Tình huống điển hình (Gây ra bởi `FetchType.LAZY`):**
    
    1. Bạn có entity `User` và `List<Post> posts` (LAZY).
        
    2. Bạn gọi: `List<User> users = userRepository.findAll();`
        
        - **SQL (1 query):** `SELECT * FROM users;`
            
    3. Bạn lặp qua danh sách users trong tầng service để xây dựng DTO:
        
        Java
        
        ```
        for (User user : users) {
            dto.setPostCount(user.getPosts().size()); // Truy cập LAZY
        }
        ```
        
    4. **SQL (N queries):** Khi `user.getPosts()` được gọi, Hibernate phải thực hiện một truy vấn _riêng biệt_ cho _mỗi_ user.
        
        - `SELECT * FROM posts WHERE user_id = 1;`
            
        - `SELECT * FROM posts WHERE user_id = 2;`
            
        - `SELECT * FROM posts WHERE user_id = 3;`
            
        - ... (N lần)
            
    
    - Tổng cộng: 1 + N truy vấn.   
        

**Giải pháp Chuẩn Doanh nghiệp:**

1. **Giải pháp 1: `JOIN FETCH` (trong JPQL)**
    
    - Bạn viết lại truy vấn `findAll` bằng `@Query` và thêm `JOIN FETCH`.
        
    - Java
        
        ```
        @Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.posts")
        List<User> findAllWithPosts();
        ```
        
    - **Cách hoạt động:** Từ khóa `JOIN FETCH` yêu cầu Hibernate không chỉ thực hiện `JOIN` (nối bảng) trong SQL, mà còn _khởi tạo_ (initialize) collection `u.posts` ngay trong truy vấn đầu tiên. Kết quả là 1 truy vấn SQL duy nhất trả về một "siêu" tập kết quả (Cartesian product), và Hibernate sẽ "gỡ rối" nó thành các đối tượng `User` với `posts` đã được điền sẵn.   
        
2. **Giải pháp 2: `@EntityGraph` (Cách sạch hơn)**
    
    - Thay vì viết JPQL, bạn chỉ cần chú thích phương thức Repository.
        
    - Java
        
        ```
        @EntityGraph(attributePaths = { "posts" })
        @Override
        List<User> findAll();
        ```
        
    - **Cách hoạt động:** `@EntityGraph` là một cách khai báo (declarative) để nói với Spring Data JPA: "Khi thực hiện phương thức này, hãy tự động tạo một truy vấn `JOIN FETCH` cho các thuộc tính (attributePaths) được chỉ định".   
        
    - Đây là cách ưa thích hơn vì nó giữ cho tên phương thức (như `findAll`) sạch sẽ và có thể tái sử dụng, đồng thời tách biệt định nghĩa truy vấn (logic) khỏi cách thức tải dữ liệu (hiệu suất).   
        

### 3.7 DTOs và JPA Projections

- **Vấn đề:** Trả về entity JPA (`User`) trực tiếp từ API (Controller) là một _best practice rất tệ_.
    
    - **Bảo mật:** Bạn có thể vô tình làm lộ các trường nhạy cảm (ví dụ: `passwordHash`, `salary`).   
        
    - **Dư thừa dữ liệu:** Client có thể chỉ cần 3 trường, nhưng bạn gửi 30 trường.
        
    - **Nguy cơ `LazyInitializationException`:** Đây là rủi ro lớn nhất.
        
- **`LazyInitializationException` được giải thích:**
    
    1. Tầng `@Service` của bạn (chú thích bằng `@Transactional`) gọi `userRepository.findById(1)`. Bạn nhận được một entity `User` (trạng thái **Managed**).
        
    2. Phương thức service trả về entity `User` cho `@Controller`.
        
    3. Ngay khi phương thức service kết thúc, transaction (`@Transactional`) cũng kết thúc. Persistence Context (phiên Hibernate) bị **đóng**.   
        
    4. Entity `User` bây giờ ở trạng thái **Detached**.   
        
    5. Trong `@Controller`, Jackson (bộ tuần tự hóa JSON) cố gắng chuyển `User` thành JSON. Nó gọi `user.getPosts()` (trường này là LAZY).
        
    6. Vì entity đã bị _detached_, proxy LAZY không thể tải dữ liệu vì phiên đã đóng.
        
    7. **Exception:** `org.hibernate.LazyInitializationException: could not initialize proxy - no Session`.   
        
- **Giải pháp 1: DTO (Data Transfer Object) Pattern**
    
    - DTO là một lớp POJO (hoặc Java Record) đơn giản, "ngu ngốc", chỉ chứa dữ liệu bạn muốn trả về cho client.   
        
    - Cách DTO giải quyết `LazyInitializationException` :   
        
        1. Trong tầng `@Service` (khi transaction _vẫn còn mở_), bạn tải entity `User`.
            
        2. Bạn _truy cập một cách tường minh_ vào các trường lazy cần thiết (ví dụ: `user.getPosts().size()`).
            
        3. Bạn _ánh xạ_ (map) dữ liệu từ entity (Managed) sang một `UserDTO` mới.
            
            Java
            
            ```
            // Bên trong @Service @Transactional
            User user = userRepository.findById(1);
            List<String> postTitles = user.getPosts().stream().map(Post::getTitle).toList();
            return new UserDTO(user.getName(), postTitles); // Trả về DTO, không phải Entity
            ```
            
        4. Transaction đóng lại.
            
        5. `Controller` trả về `UserDTO`. Jackson chỉ thấy một POJO đơn giản và tuần tự hóa nó. Không có proxy, không có lazy loading, không có exception.   
            
    - DTOs cũng giải quyết vấn đề bảo mật (chỉ chọn trường) và dư thừa dữ liệu.   
        
- **Giải pháp 2: JPA Projections**
    
    - Spring Data JPA cho phép bạn trả về DTOs hoặc Interfaces trực tiếp từ Repository, thay vì mapping thủ công trong Service.
        
    - **Interface-Based Projections:** Định nghĩa một interface chỉ với các getter bạn cần.   
        
        Java
        
        ```
        public interface UserSummary {
            String getUsername();
            int getPostCount(); // Spring Data hiểu điều này
        }
        // Trong Repository:
        UserSummary findByUsername(String username);
        ```
        
    - **Class-Based (DTO) Projections:** Sử dụng DTO (hoặc Record) làm kiểu trả về. Spring Data JPA đủ thông minh để chỉ `SELECT` các cột cần thiết cho constructor của DTO.   
        
        Java
        
        ```
        public record UserSummary(String username, int postCount) {}
        // Trong Repository:
        @Query("SELECT new com.example.UserSummary(u.username, size(u.posts)) FROM User u WHERE u.id = :id")
        UserSummary findSummaryById(Long id);
        ```
        
    - Đây là cách tối ưu nhất về hiệu suất, vì nó chỉ lấy chính xác dữ liệu cần thiết từ CSDL.
        

## Phần IV: Spring Security: Bảo mật Ứng dụng

Spring Security là một framework cực kỳ mạnh mẽ (và phức tạp) để xử lý bảo mật.

### 4.1 Authentication (Xác thực) vs. Authorization (Ủy quyền)

Đây là hai khái niệm cơ bản nhất :   

1. **Authentication (Xác thực):** Trả lời câu hỏi "Bạn là ai?". Đây là quá trình xác minh danh tính của người dùng (ví dụ: kiểm tra username và password, xác thực token).   
    
2. **Authorization (Ủy quyền):** Trả lời câu hỏi "Bạn được phép làm gì?". Quá trình này diễn ra _sau khi_ xác thực thành công, để quyết định liệu người dùng đã được xác thực có quyền truy cập vào một tài nguyên cụ thể hay không (ví dụ: `ROLE_ADMIN` có thể truy cập `/admin`, nhưng `ROLE_USER` thì không).   
    

### 4.2 Kiến trúc Cốt lõi: Security Filter Chain (Chuỗi Filter Bảo mật)

Kiến trúc của Spring Security dựa hoàn toàn vào một chuỗi các Servlet Filter. Khi một `HttpServletRequest` đến, nó phải đi qua một chuỗi các filter _trước khi_ nó đến được `DispatcherServlet` (và `Controller` của bạn).   

Một chuỗi filter điển hình có thể trông như sau: `Request` -> `CsrfFilter` -> `UsernamePasswordAuthenticationFilter` -> `BasicAuthenticationFilter` -> `AuthorizationFilter` -> `Controller`.   

**Luồng Xác thực (Authentication Flow) chi tiết (ví dụ: Form Login):**

Đây là luồng "stateful" truyền thống, là nền tảng cho mọi thứ khác.   

1. Người dùng gửi `POST` đến `/login` với username/password.
    
2. Request bị chặn bởi `UsernamePasswordAuthenticationFilter`.   
    
3. Filter trích xuất username/password và tạo một đối tượng `Authentication` (cụ thể là `UsernamePasswordAuthenticationToken`) với trạng thái `authenticated = false`.   
    
4. Filter gọi `AuthenticationManager.authenticate(token)`.   
    
5. `AuthenticationManager` là một interface. Triển khai phổ biến nhất là `ProviderManager`.   
    
6. `ProviderManager` duyệt qua một danh sách các `AuthenticationProvider` đã được cấu hình.   
    
7. Nó tìm một `Provider` hỗ trợ loại token này (ví dụ: `DaoAuthenticationProvider` cho username/password).   
    
8. `DaoAuthenticationProvider` gọi `UserDetailsService` (đây là interface _bạn_ phải triển khai).   
    
9. Phương thức `loadUserByUsername(username)` của bạn được gọi. Bạn truy cập CSDL để tìm user và trả về một đối tượng `UserDetails` (thường là entity `User` của bạn implement `UserDetails`).   
    
10. `DaoAuthenticationProvider` nhận `UserDetails`, sau đó so sánh password (sử dụng `PasswordEncoder` đã cấu hình).   
    
11. Nếu khớp, `DaoAuthenticationProvider` trả về một đối tượng `Authentication` _mới_ với `authenticated = true` và danh sách các `GrantedAuthority` (roles).   
    
12. `UsernamePasswordAuthenticationFilter` nhận được `Authentication` đã được xác thực này và đặt nó vào `SecurityContextHolder`.   
    
13. Chuỗi filter tiếp tục.
    

**`SecurityContextHolder` là gì?** Nó là một đối tượng `ThreadLocal`. Nó giữ `SecurityContext`, và `SecurityContext` giữ đối tượng `Authentication` (chứa thông tin về người dùng đang đăng nhập). Bất cứ đâu trong ứng dụng, bạn cũng có thể lấy thông tin người dùng hiện tại bằng cách gọi `SecurityContextHolder.getContext().getAuthentication()`.   

### 4.3 Xác thực JWT (Stateless Authentication)

Trong các API hiện đại, chúng ta không sử dụng session (stateless). JWT (JSON Web Token) là tiêu chuẩn.   

**Luồng hoạt động (JWT Flow):**

1. **Login (`/api/login`):** Người dùng gửi username/password. Luồng xác thực ở mục 4.2 diễn ra _một lần_.
    
2. **Tạo Token:** Nếu xác thực thành công, thay vì lưu vào session, server _tạo_ ra một chuỗi JWT. Chuỗi này chứa thông tin người dùng (ví dụ: `username`, `roles`) và được ký (sign) bằng một khóa bí mật (secret key).   
    
3. **Trả về Token:** Server trả về chuỗi JWT này cho client.   
    
4. **Các yêu cầu tiếp theo:** Client _phải_ đính kèm JWT này trong _mọi_ yêu cầu tiếp theo, thường là trong header `Authorization: Bearer <token>`.   
    

**Triển khai Filter JWT (Best Practice):**

Vì server không nhớ bất cứ điều gì (stateless), nó phải xác thực lại token trong _mọi_ yêu cầu. Điều này được thực hiện bằng một filter tùy chỉnh.

1. Tạo một lớp (ví dụ: `JwtTokenFilter`) kế thừa từ **`OncePerRequestFilter`**.   
    
    - _Tại sao là `OncePerRequestFilter`?_ Spring đảm bảo filter này chỉ chạy _một lần cho mỗi request_, ngay cả trong các kịch bản phức tạp như forward hoặc include nội bộ.   
        
2. Ghi đè phương thức `doFilterInternal`.
    
3. **Logic bên trong `doFilterInternal`:** a. Đọc header `Authorization` từ `request`. b. Nếu header tồn tại và bắt đầu bằng "Bearer ", trích xuất token. c. Xác thực token (kiểm tra chữ ký, ngày hết hạn). d. Nếu token hợp lệ, trích xuất `username` và `authorities` (roles) từ token. e. _Bước quan trọng:_ Tạo một đối tượng `UsernamePasswordAuthenticationToken` _mới_ với `username`, `null` (cho password), và `authorities`. f. _Bước quan trọng nhất:_ Đặt đối tượng `Authentication` này vào `SecurityContextHolder`: `SecurityContextHolder.getContext().setAuthentication(authentication);`.   
    
4. Thêm filter này vào `SecurityFilterChain` (thường là _trước_ `UsernamePasswordAuthenticationFilter`).
    

_Kết quả:_ Đối với mọi yêu cầu có token hợp lệ, `SecurityContextHolder` sẽ được điền thông tin người dùng. Các phần còn lại của Spring Security (ví dụ: `AuthorizationFilter`, `@PreAuthorize`) sẽ hoạt động bình thường như thể đây là một phiên đăng nhập stateful.

### 4.4 OAuth2 và OpenID Connect (OIDC)

- **OAuth2:** Là một framework cho **ủy quyền** (delegated authorization). Nó _không_ phải là xác thực. Nó nói về việc cho phép ứng dụng A (Client) truy cập tài nguyên của bạn trên ứng dụng B (Resource Server) mà không cần đưa mật khẩu của bạn cho ứng dụng A (ví dụ: "Cho phép ứng dụng XYZ truy cập danh bạ Google của bạn").   
    
- **OIDC (OpenID Connect):** Là một lớp (layer) mỏng nằm _trên_ OAuth2. Nó thêm phần **xác thực** (authentication) còn thiếu. OIDC cung cấp một "ID Token" (một JWT) để xác minh "Bạn là ai".   
    

**Các vai trò trong Spring:**

- **Authorization Server (AS):** Máy chủ cấp phát token (ví dụ: Google, GitHub, Keycloak, hoặc `spring-authorization-server`). Chịu trách nhiệm xác thực người dùng và cấp `access_token` và `id_token`.   
    
- **Resource Server (RS):** Là **backend API (microservice) của bạn**. Nó _bảo vệ_ tài nguyên. Nó _không_ xác thực người dùng, nó chỉ _xác thực token_.   
    
    - **Dependency:** `spring-boot-starter-oauth2-resource-server`.   
        
    - **Cấu hình:** Bạn chỉ cần cung cấp `issuer-uri` (ví dụ: `spring.security.oauth2.resourceserver.jwt.issuer-uri`). Resource Server sẽ tự động tải các khóa công khai (public keys) từ AS để xác thực chữ ký của JWT.   
        
- **Client:** Là ứng dụng (ví dụ: một ứng dụng Spring Boot khác, hoặc một Backend-for-Frontend) _thay mặt_ người dùng đi lấy token.   
    
    - **Dependency:** `spring-boot-starter-oauth2-client`.   
        
    - **Cấu hình:** Cung cấp `client-id`, `client-secret`, `redirect-uri`. Framework này sẽ xử lý tất cả các bước chuyển hướng (redirect) phức tạp của luồng OAuth2 (ví dụ: chuyển hướng đến Google, nhận lại authorization code, đổi code lấy token).   
        

## Phần V: Spring Cloud: Kiến trúc Microservices

Khi ứng dụng của bạn phát triển thành nhiều dịch vụ nhỏ (microservices), Spring Boot là không đủ. Bạn cần Spring Cloud để quản lý sự phức tạp của một hệ thống phân tán. Spring Cloud là một "hệ thống của các hệ thống", cung cấp các pattern (mẫu) phổ biến.   

### 5.1 Service Discovery (Phát hiện Dịch vụ) với Eureka

- **Vấn đề:** Trong kiến trúc microservices, Service A cần gọi Service B. Trong môi trường đám mây, các instance của Service B được tạo ra và mất đi liên tục, địa chỉ IP của chúng thay đổi. Bạn không thể hard-code địa chỉ IP.   
    
- **Giải pháp:** Một "Danh bạ điện thoại" (Phone Book) cho các service, được gọi là **Service Registry**.   
    
- **Kiến trúc (Netflix Eureka):**
    
    1. **Eureka Server:** Bạn tạo một Spring Boot app duy nhất, thêm `@EnableEurekaServer` vào đó. Đây là sổ đăng ký trung tâm.   
        
    2. **Eureka Client:** _Tất cả_ các microservice khác (Service A, Service B...) thêm `@EnableEurekaClient` (hoặc `@EnableDiscoveryClient`).   
        
    3. **Hoạt động:** Khi Service B (ví dụ: `user-service`) khởi động, nó sẽ "gọi điện" cho Eureka Server và nói: "Chào, tôi là `user-service`, tôi đang chạy ở `10.1.2.5:8080`". Eureka Server lưu thông tin này. Service A, khi muốn gọi `user-service`, sẽ hỏi Eureka Server: "Cho tôi địa chỉ của `user-service`".   
        

### 5.2 API Gateway (Cổng API) với Spring Cloud Gateway

- **Vấn đề:** Ứng dụng di động của bạn cần lấy thông tin từ 5 microservice khác nhau (users, products, orders...). Nó không nên phải biết địa chỉ của cả 5 service và thực hiện 5 lệnh gọi mạng riêng biệt.   
    
- **Giải pháp:** Một **API Gateway** đóng vai trò là "cửa ngõ" (entry point) duy nhất cho tất cả các traffic từ bên ngoài. Nó nhận tất cả các yêu cầu, sau đó _định tuyến_ (route) chúng đến các microservice nội bộ thích hợp.   
    
- **Spring Cloud Gateway vs. Zuul:**
    
    - **Zuul 1 (Netflix):** Công nghệ cũ, dựa trên Servlet (blocking I/O). Nó không hỗ trợ các kết nối lâu dài như WebSockets.   
        
    - **Spring Cloud Gateway (Hiện đại):** Là giải pháp thay thế. Nó được xây dựng trên **Spring WebFlux / Project Reactor**, nghĩa là nó hoàn toàn _non-blocking_ và _reactive_. Điều này mang lại hiệu suất vượt trội cho các tác vụ I/O nặng và hỗ trợ WebSockets.   
        
- **Tích hợp Gateway + Eureka (Rất quan trọng):**
    
    - API Gateway cũng là một Eureka Client. Nó sử dụng Service Registry để tìm các service khác.
        
    - **Cấu hình Gateway (ví dụ `application.yml`):**
        
        YAML
        
        ```
        spring:
          cloud:
            gateway:
              routes:
              - id: user_service_route
                uri: lb://USER-SERVICE
                predicates:
                - Path=/api/users/**
        ```
        
    - **Luồng hoạt động (Chain of Thought):**
        
        1. Một yêu cầu từ bên ngoài đến Gateway: `GET /api/users/123`.
            
        2. Gateway kiểm tra `predicates`: Request khớp với `Path=/api/users/**`.   
            
        3. Gateway nhìn vào `uri: lb://USER-SERVICE`.   
            
        4. Tiền tố **`lb://`** ("load-balanced") là chỉ thị đặc biệt. Nó yêu cầu Gateway _không_ coi "USER-SERVICE" là một hostname. Thay vào đó, nó yêu cầu `DiscoveryClient` (được cung cấp bởi Eureka) tra cứu dịch vụ có tên logic là "USER-SERVICE".   
            
        5. Eureka trả về một danh sách các địa chỉ IP:PORT đang hoạt động cho "USER-SERVICE" (ví dụ: `[10.1.2.5:8080, 10.1.2.6:8080]`).
            
        6. Gateway (sử dụng bộ cân bằng tải tích hợp) chọn một trong các địa chỉ đó và chuyển tiếp (forward) yêu cầu đến `http://10.1.2.5:8080/api/users/123`.   
            

### 5.3 Centralized Configuration (Cấu hình Tập trung) với Spring Cloud Config

- **Vấn đề:** Bạn có 50 microservice. Tất cả chúng đều cần chuỗi kết nối CSDL. Mật khẩu CSDL thay đổi. Bạn không muốn phải build và deploy lại 50 service chỉ để thay đổi một thuộc tính (property).
    
- **Kiến trúc:**
    
    1. **Config Server:** Một Spring Boot app riêng biệt, thêm `@EnableConfigServer`.   
        
    2. **Backend:** Config Server này được cấu hình để trỏ đến một kho lưu trữ (thường là **Git repository**) nơi bạn lưu trữ tất cả các tệp `application.yml` hoặc `.properties` của mình.   
        
    3. **Config Client:** Tất cả các microservice khác của bạn (users, products...) là các client.   
        
- Quá trình Bootstrap (Khởi động) :   
    
    - Đây là một quá trình khởi động 2 giai đoạn.
        
    - 1. Khi một Config Client (ví dụ: `user-service`) khởi động, _trước khi_ `ApplicationContext` chính được tạo, nó sẽ tạo một `BootstrapContext` nhỏ.   
            
    - 2. Context này đọc một tệp cấu hình đặc biệt. Trong các phiên bản Spring Cloud cũ, đây là `bootstrap.yml`. Trong các phiên bản hiện đại (từ Spring Boot 2.4+), nó được định nghĩa trong `application.properties` bằng cách sử dụng `spring.config.import`.   
            
        
        - Ví dụ (Cách mới): `spring.config.import=configserver:http://localhost:8888`
            
    - 3. `BootstrapContext` kết nối đến Config Server (tại `http://localhost:8888`), gửi tên (`spring.application.name`) và profile (`spring.profiles.active`) của nó.   
            
    - 4. Config Server tìm trong Git repo các tệp phù hợp (ví dụ: `user-service.yml` và `user-service-dev.yml`) và gửi các thuộc tính đó trở lại.
            
    - 5. Các thuộc tính này được thêm vào `Environment` của Spring với độ ưu tiên cao.   
            
    - 6. `BootstrapContext` bị hủy.
            
    - 7. _Bây giờ_, `ApplicationContext` chính mới được tạo, và nó đã có tất cả các cấu hình (như `server.port`, `spring.datasource.url`) được tải từ Git.
            
- **Cập nhật cấu hình động (Dynamic Refresh):**
    
    - **Vấn đề:** Bạn vừa đẩy (push) một thay đổi cấu hình lên Git. Làm thế nào để các service đang chạy nhận được thay đổi này _mà không cần khởi động lại_?
        
    - **Cách làm:**
        
        1. Trên bất kỳ bean nào (`@Component`, `@Service`...) mà bạn muốn nhận cấu hình mới, hãy thêm annotation **`@RefreshScope`**.   
            
        2. Thực hiện một lệnh `POST` (thường là qua webhook hoặc lệnh thủ công) đến endpoint actuator của service đó: `POST /actuator/refresh`. (Điều này yêu cầu `spring-boot-starter-actuator` ).   
            
    - **Hoạt động:** Khi endpoint `/refresh` được gọi, service client sẽ lặp lại quá trình bootstrap (kéo cấu hình mới từ Config Server). Nó phát hiện ra sự thay đổi, và sau đó _hủy_ (destroy) tất cả các bean được đánh dấu `@RefreshScope` và xóa chúng khỏi cache. Lần tiếp theo bean đó được yêu cầu (ví dụ: trong một request mới), Spring sẽ _tạo lại_ bean đó từ đầu, và lần này nó sẽ được tiêm các giá trị thuộc tính mới.   
        

### 5.4 Resilience Pattern (Mẫu Tăng cường): Circuit Breaker (Bộ ngắt mạch)

- **Vấn đề:** Service A gọi Service B. Service B bị lỗi hoặc phản hồi rất chậm. Service A có một thread pool. Các thread của Service A bắt đầu bị "treo" khi chờ Service B phản hồi. Khi tất cả các thread bị treo, Service A cũng ngừng hoạt động. Lỗi của B đã lan truyền (cascading failure) sang A.   
    
- **Giải pháp:** **Circuit Breaker Pattern** (Mẫu Bộ ngắt mạch), được triển khai bởi thư viện như **Resilience4j** (Hystrix của Netflix đã cũ và không còn được duy trì). Nó hoạt động giống như cầu dao điện trong nhà bạn.   
    

Các trạng thái của Circuit Breaker :   

1. **CLOSED (Đóng):** Trạng thái bình thường. Các yêu cầu (call) được phép đi qua. Resilience4j theo dõi tỷ lệ lỗi.   
    
2. **OPEN (Mở):** Nếu tỷ lệ lỗi vượt quá một ngưỡng (ví dụ: 50% trong 100 call gần nhất) , mạch sẽ "nhảy" sang trạng thái OPEN. Trong trạng thái này, _tất cả_ các yêu cầu tiếp theo đến Service B sẽ bị _chặn ngay lập tức_ (fail-fast) mà không cần thực hiện cuộc gọi mạng. Điều này bảo vệ Service A khỏi bị cạn kiệt thread.   
    
3. **HALF-OPEN (Nửa mở):** Sau một thời gian chờ (ví dụ: 60 giây), mạch chuyển sang HALF-OPEN. Nó cho phép một _số lượng giới hạn_ các yêu cầu "thử nghiệm" đi qua.   
    
    - Nếu các yêu cầu thử nghiệm này thành công -> Mạch quay về `CLOSED`.
        
    - Nếu chúng thất bại -> Mạch quay lại `OPEN` trong 60 giây nữa.   
        

**Triển khai (Best Practice: `fallbackMethod`):**

- Bạn thêm dependency `spring-cloud-starter-circuitbreaker-resilience4j`  và cấu hình các ngưỡng trong `application.yml`.   
    
- Trên phương thức gọi service ngoài, bạn thêm annotation `@CircuitBreaker` và chỉ định một **`fallbackMethod`**.   
    
    Java
    
    ```
    @Service
    public class ExternalCallService {
    
        @CircuitBreaker(name = "myExternalService", fallbackMethod = "getFallbackData")
        public String callExternalApi() {
            //... thực hiện lệnh gọi RestTemplate (có thể thất bại)
        }
    
        // Đây là phương thức dự phòng
        public String getFallbackData(Throwable t) {
            // Được gọi khi mạch ở trạng thái OPEN, hoặc khi call thất bại
            log.warn("External API is down, returning fallback data.", t.getMessage());
            return "Cached or Default Data";
        }
    }
    ```
    
- **Quy tắc:** Phương thức fallback phải nằm trong cùng một lớp, có cùng kiểu trả về và các tham số (có thể thêm một tham số `Throwable` ở cuối để bắt lỗi).   
    

## Phần VI: Hoạt động Chuẩn Doanh nghiệp (Operations & Best Practices)

Đây là những kỹ thuật cần thiết để chạy các ứng dụng Spring Boot trong môi trường production một cách đáng tin cậy.

### 6.1 Global Exception Handling (Xử lý Lỗi Toàn cục)

- **Vấn đề:** Bạn không muốn lặp lại logic `try-catch` và tạo `ResponseEntity` trong mọi phương thức `@Controller`. Điều này vi phạm nguyên tắc DRY (Don't Repeat Yourself) và dẫn đến các phản hồi lỗi không nhất quán.
    
- **Giải pháp (Best Practice):** Sử dụng `@RestControllerAdvice` (hoặc `@ControllerAdvice` cho các ứng dụng MVC truyền thống).   
    
- **Cách làm:**
    
    1. Tạo một lớp duy nhất được chú thích bằng `@RestControllerAdvice`. Lớp này sẽ "chặn" (intercept) các exception được ném ra từ _tất cả_ các `@RestController` trong ứng dụng.   
        
    2. Bên trong lớp này, định nghĩa các phương thức được chú thích bằng `@ExceptionHandler` cho từng loại exception cụ thể mà bạn muốn xử lý (ví dụ: `ProductNotFoundException`).
        
    3. **Best Practice:** Cho lớp này kế thừa từ `ResponseEntityExceptionHandler`.   
        
        - _Tại sao?_ Lớp cơ sở này đã cung cấp các phương thức để xử lý các exception _nội bộ_ của Spring MVC (ví dụ: `MethodArgumentNotValidException` - xảy ra khi validation `@Valid` thất bại).
            
        - Bạn có thể `@Override` các phương thức này để tùy chỉnh thông báo lỗi validation, đảm bảo _tất cả_ các lỗi (cả lỗi nghiệp vụ và lỗi framework) đều trả về cùng một định dạng JSON `ApiError` nhất quán.
            
    
    Java
    
    ```
    @RestControllerAdvice
    public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        // Xử lý lỗi nghiệp vụ tùy chỉnh
        @ExceptionHandler(ProductNotFoundException.class)
        public ResponseEntity<Object> handleProductNotFound(ProductNotFoundException ex) {
            ApiError apiError = new ApiError(HttpStatus.NOT_FOUND, "Product not found", ex.getMessage());
            return new ResponseEntity<>(apiError, HttpStatus.NOT_FOUND);
        }
    
        // Ghi đè xử lý lỗi validation mặc định của Spring
        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {
    
            List<String> errors = ex.getBindingResult().getFieldErrors().stream()
               .map(error -> error.getField() + ": " + error.getDefaultMessage())
               .collect(Collectors.toList());
    
            ApiError apiError = new ApiError(HttpStatus.BAD_REQUEST, "Validation Failed", errors);
            return new ResponseEntity<>(apiError, HttpStatus.BAD_REQUEST);
        }
    }
    ```
    

### 6.2 Logging (Ghi nhật ký) Chuẩn Doanh nghiệp

- **Hệ sinh thái:**
    
    - **SLF4J (Simple Logging Facade for Java):** Là một **Facade** (bề mặt). Code của bạn _chỉ_ nên tương tác với API của SLF4J (ví dụ: `org.slf4j.Logger`). Điều này giúp tách rời code của bạn khỏi một triển khai logging cụ thể.   
        
    - **Logback:** Là **Implementation** (triển khai). `spring-boot-starter-logging` (được bao gồm mặc định trong `starter-web`) sử dụng Logback.   
        
- **Best Practice (Levels):** Sử dụng các cấp độ log một cách có chủ đích.   
    
    - `ERROR`: Lỗi nghiêm trọng, cần hành động ngay lập tức.
        
    - `WARN`: Tình huống không mong muốn, có thể là dấu hiệu của vấn đề.
        
    - `INFO`: Các sự kiện quan trọng, mang tính nghiệp vụ (ví dụ: "Application started", "User ID 5 logged in").
        
    - `DEBUG`: Thông tin chi tiết cho lập trình viên để gỡ lỗi (ví dụ: "Entering method X", "Query parameters: [...]").
        
    - `TRACE`: Cực kỳ chi tiết (ví dụ: giá trị biến từng bước).
        
- **Best Practice (Performance):** _Luôn_ sử dụng placeholders (`{}`) thay vì nối chuỗi.
    
    - **Tệ:** `logger.debug("Processing user: " + user.getId());`    
        
        - _Tại sao tệ:_ Câu lệnh `"Processing user: " + user.getId()` _luôn được thực thi_ (tốn chi phí tạo String) ngay cả khi log level của bạn là `INFO` (và `DEBUG` bị tắt).
            
    - **Tốt:** `logger.debug("Processing user: {}", user.getId());`    
        
        - _Tại sao tốt:_ Việc tạo String chỉ xảy ra _sau khi_ Logback kiểm tra và thấy rằng `DEBUG` đang được bật. Nếu `DEBUG` bị tắt, hàm này gần như không tốn chi phí.
            
- **Best Practice (Configuration):** Sử dụng `logback-spring.xml`.   
    
    - Spring Boot ưu tiên tệp `logback-spring.xml` hơn `logback.xml`. Lý do là tệp `-spring` cho phép bạn sử dụng các tag đặc biệt của Spring, quan trọng nhất là `<springProfile>`.   
        
    - **Cấu hình theo Profile (Rất quan trọng):** Bạn có thể định nghĩa các cấu hình log khác nhau cho các môi trường khác nhau (dev, staging, prod) trong _cùng một tệp_.   
        
    
    XML
    
    ```
    <configuration>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">...</appender>
    
        <springProfile name="dev">
            <root level="DEBUG">
                <appender-ref ref="STDOUT" />
            </root>
            <logger name="org.hibernate" level="DEBUG" />
        </springProfile>
    
        <springProfile name="prod">
            <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">...</appender>
            <root level="INFO">
                <appender-ref ref="STDOUT" />
                <appender-ref ref="FILE" />
            </root>
            <logger name="org.hibernate" level="WARN" />
        </springProfile>
    </configuration>
    ```
    
- **Tối ưu hóa (Async Logging):** Ghi log ra file là một hoạt động I/O và có thể làm chậm luồng chính của ứng dụng. Logback cung cấp `AsyncAppender`. Nó nhận log message, đặt vào một hàng đợi (queue), và trả về quyền kiểm soát cho luồng của bạn ngay lập tức. Một luồng riêng biệt (worker thread) sẽ lấy log từ hàng đợi và thực hiện việc ghi I/O, giúp cải thiện đáng kể thông lượng (throughput) của ứng dụng.   
    

### 6.3 Kim tự tháp Kiểm thử (Testing Pyramid) Chuẩn Doanh nghiệp

Một chiến lược kiểm thử tốt là nền tảng của các ứng dụng đáng tin cậy.

- **Cấp 1: Unit Tests (Kiểm thử Đơn vị) (Nhanh, Rẻ, Số lượng nhiều)**
    
    - **Mục đích:** Kiểm tra một lớp (ví dụ: `ProductService`) một cách _biệt lập_ (isolation).   
        
    - **Công nghệ:** JUnit 5 + Mockito.   
        
    - **Cách làm:** _Không_ khởi động Spring context. Sử dụng `@Mock` để tạo các phụ thuộc giả (ví dụ: `ProductRepository`). Sử dụng `@InjectMocks` để tiêm các mock đó vào lớp `ProductService` đang được kiểm thử. Bạn chỉ kiểm tra logic nghiệp vụ _bên trong_ `ProductService`.   
        
- **Cấp 2: Integration Tests (Kiểm thử Tích hợp) (Chậm, Đắt, Số lượng ít)**
    
    - **Mục đích:** Kiểm tra sự _tương tác_ giữa các tầng. Ví dụ: "Một API call đến `@Controller` có kích hoạt đúng logic `@Service` và lưu vào CSDL (`@Repository`) không?"   
        
    - **Công nghệ:** `@SpringBootTest` (để tải toàn bộ `ApplicationContext`)  hoặc các "slice test" (như `@DataJpaTest` để chỉ kiểm tra tầng JPA , `@WebMvcTest` để chỉ kiểm tra tầng Controller ).   
        
    - **Vấn đề:** Theo truyền thống, các bài kiểm thử tích hợp `@DataJpaTest` sử dụng CSDL H2 (in-memory).   
        
- **Cấp 3 (Best Practice): Integration Tests với Testcontainers**
    
    - **Vấn đề của H2:** H2 _không phải_ là PostgreSQL, MySQL, hay Oracle. Nó có các kiểu dữ liệu khác, các hàm SQL khác, và các hành vi transaction khác. Một truy vấn `@Query(nativeQuery=true)` phức tạp có thể chạy hoàn hảo trên H2 (trong khi test) nhưng _thất bại_ thảm hại trên PostgreSQL (trong production).   
        
    - **Giải pháp (Testcontainers):** Là một thư viện Java cho phép bạn khởi chạy các **Docker container** (ví dụ: một CSDL PostgreSQL, Kafka, hay Redis _thật_) một cách có lập trình như một phần của bài kiểm thử JUnit.   
        
    - **Luồng hoạt động (với Spring Boot 3.1+):**
        
        1. Thêm dependency `org.testcontainers:postgresql`.
            
        2. Trong lớp test (`@SpringBootTest`), bạn chú thích container:
            
        
        Java
        
        ```
        @SpringBootTest
        @Testcontainers // Báo cho JUnit 5 biết về Testcontainers
        class UserRepositoryIntegrationTest {
        
            @Container
            @ServiceConnection // << "Phép thuật" của Spring Boot 3.1
            static PostgreSQLContainer<?> postgres = 
                new PostgreSQLContainer<>("postgres:15-alpine");
        
            @Autowired
            private UserRepository userRepository;
        
            @Test
            void testCustomNativeQuery() {
                // Bài test này đang chạy với MỘT CƠ SỞ DỮ LIỆU POSTGRESQL THẬT
                // Spring Boot tự động cấu hình DataSource để trỏ đến 
                // container đang chạy này.
                // Độ tin cậy của bài test này là 100%.
            }
        }
        ```
        
    - Annotation `@ServiceConnection`  là một tính năng mới mạnh mẽ của Spring Boot, nó tự động phát hiện container và cấu hình `DataSource` của Spring Boot để trỏ đến nó, loại bỏ hoàn toàn nhu cầu cấu hình thủ công.   
        
    - **Kết luận về Testcontainers:** Đây là "tiêu chuẩn vàng" (gold standard) của doanh nghiệp hiện đại cho kiểm thử tích hợp. Nó làm cho các bài kiểm thử tích hợp trở nên cực kỳ đáng tin cậy. Một số chuyên gia cho rằng Testcontainers đang thay đổi "Kim tự tháp" (Pyramid) thành "Tổ ong" (Honeycomb), nơi mà các bài kiểm thử tích hợp (đáng tin cậy) có giá trị cao trở nên phổ biến hơn.   
        

## Phần VII: Kết luận

Hệ sinh thái Spring, bắt đầu từ các nguyên tắc cơ bản là IoC và DI, đã phát triển thành một bộ công cụ toàn diện cho phát triển ứng dụng doanh nghiệp.

1. **Spring Core** cung cấp nền tảng linh hoạt với `Constructor Injection` là best practice cho code dễ bảo trì và kiểm thử.
    
2. **Spring Boot** loại bỏ sự phức tạp về cấu hình thông qua `Starters` và `Auto-Configuration`, cho phép lập trình viên tập trung vào logic nghiệp vụ.
    
3. **Spring Data JPA** cung cấp một lớp trừu tượng mạnh mẽ cho việc truy cập dữ liệu, nhưng đòi hỏi sự hiểu biết sâu sắc về các khái niệm cốt lõi của Hibernate (như vòng đời entity, `FetchType.LAZY`, và vấn đề N+1) để tránh các bẫy về hiệu suất. Việc sử dụng DTO hoặc Projections là bắt buộc trong môi trường production.
    
4. **Spring Security** cung cấp một kiến trúc bảo mật mạnh mẽ, phân lớp (dựa trên Filter Chain). Việc nắm vững luồng xác thực (Authentication) và sự khác biệt giữa các kiến trúc (Stateful, JWT, OAuth2 Resource Server) là chìa khóa để bảo vệ API.
    
5. **Spring Cloud** giải quyết các vấn đề của hệ thống phân tán. Việc kết hợp `Gateway` (cửa ngõ), `Eureka` (danh bạ), `Config Server` (cấu hình ngoài) và `Resilience4j` (bộ ngắt mạch) tạo thành một bộ khung sườn (scaffolding) vững chắc cho kiến trúc microservices.
    
6. **Operations & Best Practices** (Logging, Exception Handling, và Testcontainers) là những kỹ năng phân biệt giữa một ứng dụng hoạt động được và một ứng dụng _có thể bảo trì và đáng tin cậy_ trong môi trường production.
    

Việc thành thạo hệ sinh thái này không chỉ là học các annotation, mà là hiểu rõ _cách thức hoạt động_ (how) và _lý do tại sao_ (why) đằng sau mỗi lớp trừu tượng, từ đó đưa ra các quyết định kiến trúc đúng đắn, phù hợp với tiêu chuẩn doanh nghiệp.