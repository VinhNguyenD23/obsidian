

## Lời Mở Đầu: Từ Người Hướng Dẫn (Mentor)

Chào mừng bạn đến với Lộ trình Học SQL Chuyên sâu. Bạn đang bắt đầu một hành trình không chỉ là học một ngôn ngữ; bạn đang học cách giao tiếp với dữ liệu—tài sản quý giá nhất của thế kỷ 21. SQL (Structured Query Language) là ngôn ngữ phổ quát của dữ liệu. Dù bạn muốn trở thành Kỹ sư Dữ liệu, Nhà khoa học Dữ liệu, Nhà phân tích Kinh doanh, hay Lập trình viên Backend, việc thành thạo SQL là một kỹ năng không thể thương lượng.

Lộ trình này được thiết kế theo phương pháp "từ dưới lên". Chúng ta sẽ bắt đầu từ những viên gạch cơ bản nhất và xây dựng một tòa nhà kiến thức vững chắc. Không có đường tắt. Mỗi khái niệm được xây dựng dựa trên khái niệm trước đó.

Cam kết của tôi với bạn: Nếu bạn đi theo lộ trình này một cách có kỷ luật, hoàn thành mọi bài tập, và thực sự _hiểu_ (không chỉ sao chép) các truy vấn, bạn sẽ kết thúc với một bộ kỹ năng SQL vượt trội so với 90% các chuyên gia trong ngành.

Hãy bắt đầu.

## Phần 0: Thiết Lập Môi Trường Học Tập (Setup)

- **Thời lượng ước tính:** 2 giờ
    
- **Mục tiêu:** Cài đặt một cơ sở dữ liệu (PostgreSQL) và một công cụ (Client) để tương tác với nó. Đây là bước đầu tiên và là nền tảng cho toàn bộ quá trình học.
    

### 0.1. PostgreSQL là gì?

- **Lý thuyết cốt lõi:** PostgreSQL là một Hệ Quản trị Cơ sở Dữ liệu Quan hệ (RDBMS) mã nguồn mở, cực kỳ mạnh mẽ, tuân thủ nghiêm ngặt chuẩn SQL.1 Nó hoạt động theo mô hình client-server: "server" (máy chủ) là nơi quản lý và lưu trữ dữ liệu, còn "client" (máy khách) là công cụ bạn dùng để gửi yêu cầu (truy vấn SQL) đến server và nhận kết quả. Chúng ta chọn PostgreSQL vì nó miễn phí, mạnh mẽ và có các tính năng nâng cao (như Window Functions, CTEs) mà chúng ta sẽ học ở các phần sau.
    

### 0.2. Cài đặt PostgreSQL Server

- **Lý thuyết:** Đây là "bộ não" lưu trữ dữ liệu. Bạn cần cài đặt nó trên máy tính của mình (local server).
    
- **Hướng dẫn cài đặt (Ví dụ trên Ubuntu/Debian):**
    
    - Cách dễ nhất là dùng trình quản lý gói của hệ điều hành.2
        
    - `sudo apt update`
        
    - `sudo apt install postgresql postgresql-contrib` 2
        
- **Hướng dẫn cài đặt (Windows/macOS):**
    
    - Cách đơn giản nhất là tải trình cài đặt EDB tại [https://www.enterprisedb.com/downloads/postgres-postgresql-downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads).3 Trình cài đặt này thường sẽ cài đặt cả server PostgreSQL và công cụ pgAdmin.3
        
- **Tác vụ sau cài đặt (Quan trọng):**
    
    - Sau khi cài đặt, dịch vụ PostgreSQL sẽ tự động chạy.2
        
    - Bạn cần đặt mật khẩu cho người dùng quản trị mặc định tên là `postgres`.
        
    - Trên Linux/macOS, mở Terminal và chạy: `sudo -u postgres psql`.2
        
    - Bên trong `psql`, gõ: `\password postgres` và nhập mật khẩu mới của bạn. Hãy ghi nhớ mật khẩu này.
        
    - Trên Windows, trình cài đặt EDB sẽ hỏi bạn mật khẩu ngay trong lúc cài đặt.
        

### 0.3. Cài đặt Giao diện (Database Client)

- **Lý thuyết:** Bạn cần một công cụ (client) với giao diện đồ họa (GUI) để viết truy vấn và xem kết quả. Hai lựa chọn phổ biến nhất cho PostgreSQL là pgAdmin và DBeaver.
    
    - **pgAdmin:** Được thiết kế riêng cho PostgreSQL.4 Đây là công cụ chuyên dụng, rất tốt nếu bạn _chỉ_ làm việc với PostgreSQL.6
        
    - **DBeaver:** Là một công cụ cơ sở dữ liệu "phổ quát" (universal).4 Nó hỗ trợ hơn 80 loại cơ sở dữ liệu khác nhau (như MySQL, SQLite, SQL Server...).4
        
- **Gợi ý công cụ (Lời khuyên của Mentor):**
    
    - Trong thế giới thực, các nhà phân tích và kỹ sư dữ liệu hiếm khi chỉ làm việc với một loại cơ sở dữ liệu.6 Họ có thể cần kết nối với PostgreSQL, MySQL, và Redshift trong cùng một ngày.
        
    - Việc học **DBeaver** ngay từ đầu sẽ trang bị cho bạn một kỹ năng làm việc thực tế và linh hoạt hơn. pgAdmin rất tốt, nhưng DBeaver sẽ phục vụ bạn tốt hơn trong sự nghiệp lâu dài.
        
- **Thực hành cài đặt:**
    
    1. **Cài đặt DBeaver:** Tải phiên bản Community (miễn phí) tại [dbeaver.io](https://dbeaver.io/download/).3
        
    2. **Kết nối đến Server:**
        
        - Mở DBeaver và nhấp vào biểu tượng "New Database Connection" (hình ổ cắm điện).
            
        - Chọn "PostgreSQL".
            
        - Trong tab "Main", nhập thông tin kết nối:
            
            - `Host`: $localhost$ (nghĩa là máy của bạn)
                
            - `Database`: $postgres$ (database mặc định)
                
            - `Username`: $postgres$
                
            - `Password`: $[mật khẩu bạn đã tạo ở bước 0.2]$
                
        - Nhấp "Test Connection...". Nếu báo thành công, bạn đã sẵn sàng.
            

---

## Phần 1: Cấp Độ Mới Bắt Đầu (Beginner Level)

- **Tổng thời lượng:** 15-20 giờ
    
- **Mục tiêu:** Học cách "đọc" (truy xuất), lọc và sắp xếp dữ liệu. Đây là nền tảng của mọi thứ trong SQL.
    

### 1.1. Ngôn Ngữ SQL và Cú Pháp Cơ Bản

- **Thời lượng:** 2 giờ
    
- **Lý thuyết cốt lõi:**
    
    - SQL là ngôn ngữ _khai báo_ (declarative), không phải _mệnh lệnh_ (imperative). Bạn nói _cái gì_ bạn muốn (ví dụ: "lấy cho tôi tên sinh viên"), không phải _làm thế nào_ để lấy nó (ví dụ: "mở file, lặp qua từng dòng...").
        
    - Các nhóm lệnh chính:
        
        - **DQL (Data Query Language):** `SELECT` (để truy vấn dữ liệu).
            
        - **DDL (Data Definition Language):** `CREATE`, `ALTER`, `DROP` (để định nghĩa cấu trúc).7
            
        - **DML (Data Manipulation Language):** `INSERT`, `UPDATE`, `DELETE` (để thao tác dữ liệu).8
            
        - **TCL (Transaction Control Language):** `BEGIN`, `COMMIT`, `ROLLBACK` (để quản lý giao dịch).
            
        - **DCL (Data Control Language):** `GRANT`, `REVOKE` (để quản lý phân quyền).7
            
    - Các khái niệm cốt lõi: **Table** (Bảng, ví dụ: `students`), **Column** (Cột/Trường, ví dụ: `name`), và **Row** (Hàng/Bản ghi, ví dụ: sinh viên 'Nguyễn Văn A').
        
- **Database Mẫu (Giới thiệu):**
    
    - Chúng ta sẽ sử dụng một CSDL đơn giản về quản lý sinh viên.9
        
    - Trong DBeaver, hãy mở "SQL Editor" (Ctrl+Enter hoặc F3) và chạy lệnh sau để tạo bảng và chèn dữ liệu.
        
- **Schema (PostgreSQL):**
    
    SQL
    
    ```sql
    -- Xóa bảng nếu đã tồn tại để làm sạch
    DROP TABLE IF EXISTS students;
    
    -- Tạo bảng mới
    CREATE TABLE students (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        age INT,
        major VARCHAR(50),
        gpa NUMERIC(3, 2) -- Ví dụ: 3.45
    );
    
    -- Chèn dữ liệu mẫu
    INSERT INTO students (name, age, major, gpa) VALUES
    ('An Nguyen', 21, 'CS', 3.50),
    ('Binh Tran', 20, 'Economics', 3.10),
    ('Chi Le', 22, 'CS', 3.85),
    ('Dung Pham', 23, 'Physics', 3.20),
    ('Anh Vu', 20, 'Economics', 2.90),
    ('Linh Hoang', 21, 'Math', 3.90),
    ('Minh Trieu', 22, 'CS', 3.60),
    ('Nhat Bui', 20, NULL, 2.50); -- Chú ý: major là NULL
    ```
    

### 1.2. Truy Vấn Cơ Bản (SELECT, FROM, WHERE)

- **Thời lượng:** 4 giờ
    
- **Lý thuyết cốt lõi:**
    
    - `SELECT`: Chỉ định các cột (columns) bạn muốn xem.14
        
    - `FROM`: Chỉ định bảng (table) bạn muốn lấy dữ liệu.
        
    - `WHERE`: Lọc các hàng (rows) dựa trên một điều kiện.15
        
- **Database:** `students`
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: Lấy tất cả thông tin từ bảng_
        
        SQL
        
        ```sql
        SELECT * FROM students;
        ```
        
        Kết quả minh họa:
        
        (Hiển thị tất cả 8 hàng và 5 cột như đã chèn ở trên)
        
    - _Ví dụ 2: Lấy các cột cụ thể_
        
        SQL
        
        ```sql
        SELECT name, major FROM students;
        ```
        
        _Kết quả minh họa:_
        

|**name**|**major**|
|---|---|
|An Nguyen|CS|
|Binh Tran|Economics|
|Chi Le|CS|
|...|...|


*   *Ví dụ 3: Lọc cơ bản với `WHERE`*
    ```sql
    SELECT name, age, gpa FROM students WHERE major = 'CS';
    ```
    *Kết quả minh họa:*

|**name**|**age**|**gpa**|
|---|---|---|
|An Nguyen|21|3.50|
|Chi Le|22|3.85|
|Minh Trieu|22|3.60|

- **Thực hành:**
    
    - **Yêu cầu 1:** Tìm tất cả sinh viên (chỉ hiển thị tên và gpa) có GPA lớn hơn 3.5.
        
    - _Output mong đợi:_
        

|**name**|**gpa**|
|---|---|
|Chi Le|3.85|
|Linh Hoang|3.90|
|Minh Trieu|3.60|

*   **Yêu cầu 2:** Tìm tên và chuyên ngành của các sinh viên 20 tuổi.
*   *Output mong đợi:*

|**name**|**major**|
|---|---|
|Binh Tran|Economics|
|Anh Vu|Economics|
|Nhat Bui|NULL|

### 1.3. Lọc Nâng Cao và Sắp Xếp

- **Thời lượng:** 5 giờ
    
- **Lý thuyết cốt lõi:**
    
    - `ORDER BY`: Sắp xếp kết quả trả về. Mặc định là `ASC` (tăng dần). Sử dụng `DESC` (giảm dần).16
        
    - `DISTINCT`: Chỉ trả về các giá trị duy nhất (unique), loại bỏ trùng lặp.14
        
    - Toán tử logic: `AND` (cả hai đều đúng), `OR` (một trong hai đúng), `NOT` (phủ định).
        
    - Toán tử so sánh:
        
        - `IN`: Khớp với một giá trị bất kỳ trong danh sách (ví dụ: `major IN ('CS', 'Math')`).18
            
        - `BETWEEN`: Nằm trong một khoảng (ví dụ: `age BETWEEN 20 AND 22`).18
            
        - `LIKE`: Khớp với một mẫu (ví dụ: `name LIKE 'A%'` nghĩa là tên bắt đầu bằng 'A').18
            
    - `NULL`: Biểu thị giá trị bị thiếu. Để so sánh `NULL`, bạn _phải_ dùng `IS NULL` hoặc `IS NOT NULL` (không thể dùng $= NULL$).
        
- **Database:** `students`
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: `ORDER BY`_
        
        SQL
        
        ```sql
        -- Lấy tên và GPA, sắp xếp theo GPA giảm dần
        SELECT name, gpa FROM students ORDER BY gpa DESC;
        ```
        
        _Kết quả minh họa:_ (Sinh viên có GPA cao nhất lên đầu)
        

|**name**|**gpa**|
|---|---|
|Linh Hoang|3.90|
|Chi Le|3.85|
|Minh Trieu|3.60|
|...|...|

*   *Ví dụ 2: `AND`, `BETWEEN`, `IN`* 
    ```sql
    -- Tìm sinh viên CS hoặc Economics có tuổi từ 20 ĐẾN 22 (bao gồm cả 2 đầu)
    SELECT name, age, major
    FROM students
    WHERE major IN ('CS', 'Economics')
      AND age BETWEEN 20 AND 22;
    ```
    *Kết quả minh họa:*

|**name**|**age**|**major**|
|---|---|---|
|An Nguyen|21|CS|
|Binh Tran|20|Economics|
|Chi Le|22|CS|
|Anh Vu|20|Economics|
|Minh Trieu|22|CS|


*   *Ví dụ 3: `LIKE`*
    ```sql
    -- Tìm sinh viên có tên (name) bắt đầu bằng 'An'
    SELECT name FROM students WHERE name LIKE 'An%';
    ```
    *Kết quả minh họa:*

|**name**|
|---|
|An Nguyen|
|Anh Vu|


*   *Ví dụ 4: Xử lý `NULL`*
    ```sql
    -- Tìm sinh viên chưa đăng ký chuyên ngành
    SELECT name, major FROM students WHERE major IS NULL;
    ```
    *Kết quả minh họa:*

|**name**|**major**|
|---|---|
|Nhat Bui|NULL|

- **Thực hành:**
    
    - **Yêu cầu 1:** Tìm tên và GPA của tất cả sinh viên _không_ học 'CS', có GPA dưới 3.0, và sắp xếp theo tên (A-Z).
        
    - _Output mong đợi:_
        

|**name**|**gpa**|
|---|---|
|Anh Vu|2.90|
|Nhat Bui|2.50|


*   **Yêu cầu 2:** Tìm tên và tuổi của sinh viên có tên chứa 'inh' (ví dụ: Binh, Linh, Minh).
*   *Output mong đợi:*


|**name**|**age**|
|---|---|
|Binh Tran|20|
|Linh Hoang|21|
|Minh Trieu|22|

### 1.4. Hàm Tổng Hợp (Aggregate Functions)

- **Thời lượng:** 4 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Hàm tổng hợp thực hiện một phép tính trên một tập hợp các hàng và trả về _một giá trị duy nhất_.20
        
    - `COUNT()`: Đếm số lượng hàng.20
        
    - `SUM()`: Tính tổng các giá trị (chỉ dùng cho số).20
        
    - `AVG()`: Tính trung bình (chỉ dùng cho số).20
        
    - `MIN()`: Tìm giá trị nhỏ nhất.20
        
    - `MAX()`: Tìm giá trị lớn nhất.20
        
- **Lưu ý quan trọng: `COUNT(*)` vs `COUNT(column)`**
    
    - `COUNT(*)`: Đếm _tất cả các hàng_ trong nhóm, bất kể giá trị.
        
    - `COUNT(column_name)`: Chỉ đếm các hàng mà `column_name` _không phải là NULL_.
        
    - Đây là một "cạm bẫy" gỡ lỗi phổ biến. `COUNT(*)` và `COUNT(major)` trong bảng `students` của chúng ta sẽ cho ra hai kết quả khác nhau!
        
- **Database:** `students`
    
- **Query Ví dụ:**
    
    SQL
    
    ```SQL
    SELECT
        COUNT(*) AS total_students,
        COUNT(major) AS students_with_major,
        AVG(gpa) AS average_gpa,
        MAX(gpa) AS highest_gpa,
        MIN(age) AS youngest_age
    FROM students;
    ```
    
    _Kết quả minh họa:_
    
|**total_students**|**students_with_major**|**average_gpa**|**highest_gpa**|**youngest_age**|
|---|---|---|---|---|
|8|7|3.318750|3.90|20|

*(Ghi chú: `total_students` là 8, nhưng `students_with_major` là 7 vì 'Nhat Bui' có `major` là NULL)*


- **Thực hành:**
    
    - **Yêu cầu:** Tính GPA trung bình của riêng các sinh viên học 'CS'.
        
    - _Gợi ý:_ Kết hợp `WHERE` và `AVG()`.
        
    - _Output mong đợi:_
        

|**avg_gpa_cs**|
|---|
|3.650000|

### Bảng Tổng Kết Cấp Độ 1 (Beginner)

|**Chủ đề**|**Lệnh SQL chính**|**Mục tiêu kỹ năng**|
|---|---|---|
|**Cài đặt**|`psql`, `CREATE DATABASE`|Thiết lập môi trường làm việc PostgreSQL và DBeaver.2|
|**Truy vấn cơ bản**|`SELECT`, `FROM`|Biết cách lấy dữ liệu thô từ một bảng cụ thể.14|
|**Lọc dữ liệu**|`WHERE`, `AND`, `OR`, `NOT`|Lọc các hàng dựa trên các điều kiện logic đơn giản.15|
|**Lọc nâng cao**|`IN`, `BETWEEN`, `LIKE`, `IS NULL`|Sử dụng các toán tử mạnh mẽ để lọc các tập dữ liệu phức tạp.18|
|**Sắp xếp**|`ORDER BY (ASC/DESC)`|Sắp xếp kết quả đầu ra để phân tích dễ dàng hơn.16|
|**Hàm tổng hợp**|`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`|Thực hiện các phép tính thống kê cơ bản trên toàn bộ bảng.20|

---

## Phần 2: Cấp Độ Trung Cấp (Intermediate Level)

- **Tổng thời lượng:** 25-30 giờ
    
- **Mục tiêu:** Vượt ra ngoài một bảng duy nhất. Học cách _kết nối_, _biến đổi_, và _quản lý_ dữ liệu. Đây là lúc bạn bắt đầu "nhào nặn" dữ liệu để tạo ra thông tin có giá trị.
    

### 2.1. Phân Nhóm Dữ Liệu (GROUP BY & HAVING)

- **Thời lượng:** 5 giờ
    
- **Lý thuyết cốt lõi:**
    
    - `GROUP BY` (S28): Thường được sử dụng với các hàm tổng hợp. Nó chia các hàng thành các nhóm nhỏ hơn (ví dụ: nhóm tất cả sinh viên 'CS', nhóm tất cả sinh viên 'Economics') và sau đó chạy hàm tổng hợp trên _từng nhóm_.23
        
    - `HAVING`: Lọc các nhóm _sau khi_ `GROUP BY` đã thực hiện.24
        
- **Phân biệt `WHERE` vs. `HAVING` (Rất quan trọng):**
    
    - `WHERE` lọc _hàng_ (rows) _trước khi_ chúng được nhóm lại (ví dụ: `WHERE age > 20`).
        
    - `HAVING` lọc _nhóm_ (groups) _sau khi_ chúng được tính toán (ví dụ: `HAVING COUNT(*) > 5`).24
        
    - Bạn không thể dùng `HAVING AVG(gpa) > 3.0` nếu không có `GROUP BY`. Bạn không thể dùng `WHERE COUNT(*) > 5`.
        
- **Database Mẫu (Chuyển đổi):** Chúng ta cần một CSDL phức tạp hơn. Giới thiệu CSDL `cong_ty`.24
    
- **Schema:**
    
    SQL
    
    ```sql
    DROP TABLE IF EXISTS employees, departments CASCADE;
    
    CREATE TABLE departments (
        id SERIAL PRIMARY KEY,
        dept_name VARCHAR(100) NOT NULL
    );
    CREATE TABLE employees (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        salary NUMERIC(10, 2),
        dept_id INT REFERENCES departments(id)
    );
    
    INSERT INTO departments (dept_name) VALUES ('Sales'), ('Engineering'), ('Marketing'), ('HR');
    
    INSERT INTO employees (name, salary, dept_id) VALUES
    ('Alice', 70000, 1),
    ('Bob', 80000, 2),
    ('Charlie', 120000, 2),
    ('David', 65000, 1),
    ('Eva', 90000, 3),
    ('Frank', 50000, 4),
    ('Grace', 85000, 2);
    ```
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: `GROUP BY` cơ bản_ 23
        
        SQL
        
        ```sql
        -- Đếm số lượng nhân viên trong mỗi phòng ban
        SELECT dept_id, COUNT(*) AS num_employees
        FROM employees
        GROUP BY dept_id;
        ```
        
        _Kết quả minh họa:_
        

|**dept_id**|**num_employees**|
|---|---|
|1|2|
|2|3|
|3|1|
|4|1|


*   *Ví dụ 2: `GROUP BY` và `HAVING`* 
    ```sql
    -- Tìm các phòng ban có TỔNG lương > 150,000
    SELECT dept_id, SUM(salary) AS total_salary
    FROM employees
    GROUP BY dept_id
    HAVING SUM(salary) > 150000;
    ```
    *Kết quả minh họa:* (Chỉ phòng Engineering (80k+120k+85k=285k) mới xuất hiện)

|**dept_id**|**total_salary**|
|---|---|
|2|285000.00|

- **Thực hành:**
    
    - **Yêu cầu:** Tính lương trung bình cho mỗi phòng ban, nhưng chỉ hiển thị các phòng ban có lương trung bình dưới 70.000.
        
    - _Output mong đợi:_
        

|**dept_id**|**avg_salary**|
|---|---|
|1|67500.00|
|4|50000.00|

### 2.2. Kết Nối Bảng (JOINs)

- **Thời lượng:** 6 giờ
    
- **Lý thuyết cốt lõi:**
    
    - `JOIN` được sử dụng để kết hợp dữ liệu từ hai hoặc nhiều bảng dựa trên một cột liên quan (thường là Khóa Chính và Khóa Ngoại).25
        
    - **`INNER JOIN` (Mặc định):** Chỉ trả về các hàng có _khớp_ ở cả hai bảng.25
        
    - **`LEFT JOIN` (hoặc `LEFT OUTER JOIN`):** Trả về _tất cả_ các hàng từ bảng bên trái, và các hàng khớp từ bảng bên phải. Nếu không khớp, cột của bảng bên phải sẽ là `NULL`.25
        
    - **`RIGHT JOIN`:** Tương tự `LEFT JOIN`, nhưng ưu tiên bảng bên phải (ít dùng hơn, vì luôn có thể viết lại thành `LEFT JOIN`).
        
    - **`FULL OUTER JOIN`:** Trả về tất cả các hàng khi có một kết quả khớp ở một trong hai bảng.26
        
- **Lựa chọn JOIN (INNER vs LEFT):**
    
    - Đây không phải là một lựa chọn cú pháp, mà là một _lựa chọn nghiệp vụ_.
        
    - Câu hỏi: "Cho tôi danh sách nhân viên và tên phòng ban của họ." -> `INNER JOIN`. (Bạn không quan tâm đến nhân viên chưa có phòng ban, hoặc phòng ban không có nhân viên).
        
    - Câu hỏi: "Cho tôi _tất cả_ nhân viên, và nếu họ có phòng ban, hãy cho tôi biết tên." -> `LEFT JOIN` (bảng `employees` bên trái). (Bạn muốn tìm cả những nhân viên _chưa_ được gán phòng ban - một tác vụ kiểm toán dữ liệu quan trọng).26
        
- **Database:** `cong_ty` (`employees`, `departments`)
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: `INNER JOIN`_ 25
        
        SQL
        
        ```sql
        -- Lấy tên nhân viên và tên phòng ban của họ
        SELECT e.name AS employee_name, d.dept_name AS department_name
        FROM employees e
        INNER JOIN departments d ON e.dept_id = d.id;
        ```
        
        _Kết quả minh họa:_ (Hiển thị 7 hàng, không có `NULL`)
        

|**employee_name**|**department_name**|
|---|---|
|Alice|Sales|
|Bob|Engineering|
|Charlie|Engineering|
|David|Sales|
|Eva|Marketing|
|Frank|HR|
|Grace|Engineering|

*   *Ví dụ 2: `LEFT JOIN`* 
    ```sql
    -- Thêm một nhân viên mới chưa có phòng ban
    INSERT INTO employees (name, salary, dept_id) VALUES ('Helen', 75000, NULL);
    
    -- Query INNER JOIN sẽ "mất" Helen. LEFT JOIN sẽ giữ Helen lại.
    SELECT e.name AS employee_name, d.dept_name AS department_name
    FROM employees e
    LEFT JOIN departments d ON e.dept_id = d.id;
    ```
    *Kết quả minh họa:* (Bây giờ có 8 hàng, Helen xuất hiện với `NULL`)


|**employee_name**|**department_name**|
|---|---|
|Alice|Sales|
|...|...|
|Grace|Engineering|
|Helen|NULL|

- **Thực hành:**
    
    - **Yêu cầu 1:** Hiển thị tên các phòng ban _không có_ nhân viên nào.
        
    - _Gợi ý:_ `INSERT` một phòng ban mới (ví dụ: 'Finance') và sử dụng `LEFT JOIN` (bảng `departments` bên trái) và `WHERE e.id IS NULL`.
        
    - `INSERT INTO departments (dept_name) VALUES ('Finance');`
        
    - _Output mong đợi:_
        

|**dept_name**|
|---|
|Finance|

*   **Yêu cầu 2:** Tính tổng lương cho mỗi *tên* phòng ban (dept_name).
*   *Gợi ý:* Kết hợp `JOIN` và `GROUP BY d.dept_name`.
*   *Output mong đợi:*

|**department_name**|**total_salary_by_dept**|
|---|---|
|Sales|135000.00|
|Engineering|285000.00|
|Marketing|90000.00|
|HR|50000.00|
|Finance|NULL|

### 2.3. Truy Vấn Con (Subqueries)

- **Thời lượng:** 5 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Truy vấn con (Subquery) là một câu `SELECT` được lồng bên trong một câu `SELECT`, `INSERT`, `UPDATE`, hoặc `DELETE` khác.27
        
    - Chúng có thể được sử dụng ở nhiều nơi:
        
        - Trong `WHERE` (thường với `IN`, `NOT IN`, `EXISTS`).27
            
        - Trong `FROM` (còn gọi là Bảng Phái sinh - Derived Table).29
            
        - Trong `SELECT` (còn gọi là Subquery Tương Quan - Correlated Subquery).
            
- **Database:** `cong_ty`
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: Subquery trong `WHERE`_ 27
        
        SQL
        
        ```sql
        -- Tìm tên các nhân viên làm việc trong phòng 'Engineering'
        -- (Cách này thay thế cho JOIN)
        SELECT name
        FROM employees
        WHERE dept_id IN (SELECT id FROM departments WHERE dept_name = 'Engineering');
        ```
        
        _Kết quả minh họa:_
        

|**name**|
|---|
|Bob|
|Charlie|
|Grace|


*   *Ví dụ 2: Subquery trong `FROM`* 
    ```sql
    -- Lấy kết quả tổng hợp của phòng ban và tham gia lại
    -- Bảng "d_stats" là một bảng tạm chỉ tồn tại trong truy vấn này
    SELECT e.name, d_stats.avg_salary
    FROM employees e
    JOIN (
        SELECT dept_id, AVG(salary) AS avg_salary
        FROM employees
        GROUP BY dept_id
    ) AS d_stats ON e.dept_id = d_stats.dept_id;
    ```
    *Kết quả minh họa:*

|**name**|**avg_salary**|
|---|---|
|Alice|67500.00|
|David|67500.00|
|Bob|95000.00|
|Charlie|95000.00|
|Grace|95000.00|
|...|...|

- **Thực hành:**
    
    - **Yêu cầu:** Tìm tên và lương của tất cả nhân viên có lương cao hơn mức lương trung bình của _toàn bộ_ công ty.
        
    - _Gợi ý:_ Bạn sẽ cần một subquery trong `WHERE` để tính lương trung bình (`SELECT AVG(salary) FROM employees`).
        
    - _Output mong đợi:_ (Giả sử lương trung bình là ~78.125)
        

|**name**|**salary**|
|---|---|
|Bob|80000.00|
|Charlie|120000.00|
|Eva|90000.00|
|Grace|85000.00|

### 2.4. Logic Điều Kiện (CASE WHEN)

- **Thời lượng:** 3 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Mệnh đề `CASE` là cách để thêm logic "if-then-else" vào bên trong câu `SELECT` của bạn.30
        
    - Nó cho phép bạn tạo ra các cột "phái sinh" (derived columns) dựa trên điều kiện, mà không thay đổi dữ liệu gốc.
        
    - Cú pháp: `CASE WHEN [condition1] THEN [result1] WHEN [condition2] THEN [result2]... ELSE [result_default] END`.30
        
- **Database:** `cong_ty`
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: Phân loại lương_ 30
        
        SQL
        
        ```sql
        SELECT
            name,
            salary,
            CASE
                WHEN salary < 60000 THEN 'Junior'
                WHEN salary >= 60000 AND salary < 90000 THEN 'Mid'
                ELSE 'Senior'
            END AS seniority_level
        FROM employees;
        ```
        
        _Kết quả minh họa:_
        

|**name**|**salary**|**seniority_level**|
|---|---|---|
|Alice|70000.00|Mid|
|Bob|80000.00|Mid|
|Charlie|120000.00|Senior|
|Frank|50000.00|Junior|
|...|...|...|

*   *Ví dụ 2: `CASE` bên trong hàm tổng hợp (Pivoting)*
    ```sql
    -- Đếm số lượng nhân viên 'Senior' (>=90k) và 'Other' trong phòng Engineering
    SELECT
        COUNT(CASE WHEN salary >= 90000 THEN 1 END) AS senior_count,
        COUNT(CASE WHEN salary < 90000 THEN 1 END) AS other_count
    FROM employees
    WHERE dept_id = 2; -- Engineering
    ```
    *Kết quả minh họa:*


|**senior_count**|**other_count**|
|---|---|
|1|2|

- **Thực hành:**
    
    - **Yêu cầu:** Viết truy vấn tăng lương giả định (chỉ hiển thị, không `UPDATE`). Tăng 10% cho 'Sales', 5% cho 'Engineering', và 3% cho các phòng ban khác. Hiển thị tên, tên phòng ban, lương cũ, và lương mới.
        
    - _Gợi ý:_ Bạn sẽ cần `JOIN` và `CASE` trên `dept_name`.
        
    - _Output mong đợi:_
        

|**name**|**dept_name**|**old_salary**|**new_salary**|
|---|---|---|---|
|Alice|Sales|70000.00|77000.000|
|Bob|Engineering|80000.00|84000.000|
|Eva|Marketing|90000.00|92700.000|
|...|...|...|...|

### 2.5. Thao Tác Dữ Liệu (DML)

- **Thời lượng:** 4 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Đây là các lệnh để _thay đổi_ dữ liệu vĩnh viễn: `INSERT`, `UPDATE`, `DELETE`.8
        
    - `INSERT INTO table (col1, col2) VALUES (val1, val2);`: Thêm một hàng mới.8
        
    - `UPDATE table SET col1 = val1 WHERE [condition];`: Cập nhật các hàng hiện có.8
        
    - `DELETE FROM table WHERE [condition];`: Xóa các hàng hiện có.8
        
    - `TRUNCATE TABLE table;`: Xóa _tất cả_ dữ liệu (nhanh hơn `DELETE` không `WHERE`, nhưng ít an toàn hơn).
        
- **Tính năng PostgreSQL: `RETURNING`**
    
    - Bạn có thể thêm `RETURNING *` (hoặc `RETURNING id`) vào cuối `INSERT`, `UPDATE`, `DELETE` để trả về các hàng vừa bị ảnh hưởng. Cực kỳ hữu ích! 34
        
- **Lưu ý AN TOÀN (Lời khuyên của Mentor):**
    
    - Nguy hiểm lớn nhất là quên mệnh đề `WHERE` khi chạy `UPDATE` hoặc `DELETE`.
        
    - `UPDATE employees SET salary = 10000;` (không có `WHERE`) sẽ đặt lương của _MỌI NGƯỜI_ thành 10000. Đây là một thảm họa.
        
    - **Quy tắc vàng:** _Luôn luôn_ viết câu `SELECT *` với mệnh đề `WHERE` của bạn trước. Ví dụ: `SELECT * FROM employees WHERE name = 'Frank';`. Nếu nó trả về đúng hàng bạn muốn, _hãy_ thay `SELECT *` bằng `DELETE` hoặc `UPDATE`.
        
- **Database:** `cong_ty`
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: INSERT với `RETURNING`_ 8
        
        SQL
        
        ```sql
        INSERT INTO employees (name, salary, dept_id)
        VALUES ('Ivy', 55000, 4)
        RETURNING id, name; -- Trả về ID và tên của nhân viên mới
        ```
        
        _Kết quả minh họa:_
        

|**id**|**name**|
|---|---|
|9|Ivy|

*   *Ví dụ 2: UPDATE* 
    ```sql
    -- Tăng lương 10% cho Alice
    UPDATE employees
    SET salary = salary * 1.10
    WHERE name = 'Alice'
    RETURNING name, salary;
    ```
    *Kết quả minh họa:*

|**name**|**salary**|
|---|---|
|Alice|77000.00|

*   *Ví dụ 3: DELETE* 
    ```sql
    -- Xóa nhân viên 'Frank'
    DELETE FROM employees
    WHERE name = 'Frank'
    RETURNING *; -- Trả về toàn bộ hàng đã bị xóa
```
- **Thực hành:**
    
    - **Yêu cầu:** Phòng Marketing (id=3) được sáp nhập vào Sales (id=1).
        
    - 1. Cập nhật (UPDATE) tất cả nhân viên có `dept_id = 3` thành `dept_id = 1`.
            
    - 2. Xóa (DELETE) phòng ban 'Marketing' khỏi bảng `departments`. (Lưu ý: Bạn có thể cần xử lý Foreign Key nếu có).
            
    - _Output mong đợi (sau bước 1):_ 'Eva' bây giờ nên có `dept_id = 1`.
        
    - _Output mong đợi (sau bước 2):_ Hàng 'Marketing' biến mất khỏi `departments`.
        
### 2.6. Định Nghĩa Dữ Liệu (DDL) & Tính Toàn Vẹn

- **Thời lượng:** 6 giờ
    
- **Lý thuyết cốt lõi:**
    
    - DDL là các lệnh định nghĩa _cấu trúc_ của CSDL.7
        
    - `CREATE TABLE table (...);`: Tạo bảng mới.37
        
    - `ALTER TABLE table...;`: Sửa đổi bảng hiện có.38
        
        - `ADD COLUMN col_name type;` 38
            
        - `DROP COLUMN col_name;` 39
            
        - `ALTER COLUMN col_name TYPE new_type;`
            
    - `DROP TABLE table;`: Xóa bảng (và tất cả dữ liệu của nó!)
        
    - **Constraints (Ràng buộc):** Đây là các _quy tắc_ bạn đặt trên dữ liệu để đảm bảo tính toàn vẹn.40
        
        - `PRIMARY KEY`: Bộ nhận diện duy nhất cho mỗi hàng (không `NULL`, không trùng lặp).40
            
        - `FOREIGN KEY`: Một cột tham chiếu đến `PRIMARY KEY` của bảng khác. Đây là cách bạn tạo mối quan hệ.40 Nó _thực thi_ tính toàn vẹn tham chiếu (bạn không thể thêm một `employee` với `dept_id = 99` nếu không có phòng ban `id = 99` tồn tại).43
            
        - `UNIQUE`: Đảm bảo tất cả các giá trị trong cột là duy nhất (ví dụ: `email`).42
            
        - `NOT NULL`: Cột này không được phép có giá trị `NULL`.42
            
        - `CHECK`: Đảm bảo giá trị tuân theo một điều kiện (ví dụ: `CHECK (salary > 0)`).42
            
- **Database Mẫu:** Tự tạo CSDL `thu_vien`.
    
- **Query Ví dụ:**
    
    SQL
    
    ```sql
    DROP TABLE IF EXISTS books, authors CASCADE;
    
    CREATE TABLE authors (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL
    );
    
    CREATE TABLE books (
        id SERIAL PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        author_id INT REFERENCES authors(id) ON DELETE SET NULL, -- Nếu tác giả bị xóa, set author_id thành NULL
        isbn VARCHAR(13) UNIQUE,
        published_year INT CHECK (published_year > 1500)
    );
    
    -- Thêm một cột mới vào bảng 'books'
    ALTER TABLE books
    ADD COLUMN page_count INT;
    
    -- Xóa cột vừa thêm
    ALTER TABLE books
    DROP COLUMN page_count;
    ```
    
- **Thực hành:**
    
    - **Yêu cầu:** Tạo một bảng `loans` (lượt mượn sách) với các cột và ràng buộc sau:
        
        - `id` (primary key, tự động tăng)
            
        - `book_id` (foreign key tham chiếu đến `books(id)`)
            
        - `borrower_name` (không được `NULL`)
            
        - `loan_date` (kiểu DATE, mặc định là ngày hôm nay: `DEFAULT CURRENT_DATE`)
            
        - `due_date` (kiểu DATE)
            
        - Thêm một ràng buộc `CHECK` ở cấp độ bảng để đảm bảo `due_date` luôn sau `loan_date`. (Gợi ý: `CONSTRAINT chk_dates CHECK (due_date > loan_date)`)
            
    - _Output mong đợi:_ Một câu `CREATE TABLE` thành công.
        

### 2.7. Các Toán Tử Tập Hợp (Set Operators)

- **Thời lượng:** 2 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Các toán tử này kết hợp kết quả của hai câu `SELECT` (mà phải có cùng số lượng cột và kiểu dữ liệu tương thích) thành một tập kết quả duy nhất.
        
    - `UNION`: Gộp hai tập kết quả và _loại bỏ các hàng trùng lặp_.44
        
    - `UNION ALL`: Gộp hai tập kết quả và _giữ tất cả các hàng_, bao gồm cả hàng trùng lặp.44
        
    - `INTERSECT`: Trả về các hàng chỉ xuất hiện ở _cả hai_ tập kết quả.
        
    - `EXCEPT`: Trả về các hàng xuất hiện ở tập đầu tiên mà _không_ có ở tập thứ hai.
        
- **Lưu ý về hiệu suất:**
    
    - `UNION` cần thực hiện một bước sắp xếp (sort) và loại bỏ trùng lặp (de-duplicate), điều này tốn tài nguyên.45
        
    - `UNION ALL` chỉ đơn giản là nối hai kết quả lại, vì vậy nó _nhanh hơn nhiều_.46
        
    - **Lời khuyên:** Luôn sử dụng `UNION ALL` làm mặc định. Chỉ sử dụng `UNION` khi bạn _chắc chắn_ rằng bạn cần loại bỏ trùng lặp.
        
- **Database:** `students` & một bảng `alumni` (cựu sinh viên) mới.
    
    SQL
    
    ```sql
    DROP TABLE IF EXISTS alumni;
    CREATE TABLE alumni (id SERIAL PRIMARY KEY, name VARCHAR(100), graduation_year INT);
    INSERT INTO alumni (name, graduation_year) VALUES ('Binh Tran', 2023), ('Anh Vu', 2024);
    ```
    
- **Query Ví dụ:**
    
    SQL
    
    ```sql
    -- Tất cả tên (sinh viên VÀ cựu sinh viên), không trùng lặp
    SELECT name FROM students
    UNION
    SELECT name FROM alumni;
    -- Kết quả: 'Binh Tran' và 'Anh Vu' chỉ xuất hiện 1 lần (tổng 8 hàng)
    
    -- Tất cả tên, giữ trùng lặp
    SELECT name FROM students
    UNION ALL
    SELECT name FROM alumni;
    -- Kết quả: 'Binh Tran' và 'Anh Vu' xuất hiện 2 lần (tổng 10 hàng)
    
    -- Sinh viên hiện tại mà KHÔNG PHẢI là cựu sinh viên
    SELECT name FROM students
    EXCEPT
    SELECT name FROM alumni;
    -- Kết quả: Trả về 6 sinh viên (An, Chi, Dung, Linh, Minh, Nhat)
    ```
    
- **Thực hành:**
    
    - **Yêu cầu:** Viết truy vấn tìm những người vừa là sinh viên hiện tại vừa là cựu sinh viên (có thể họ đang học bằng 2).
        
    - _Gợi ý:_ Sử dụng `INTERSECT`.
        
    - _Output mong đợi:_
        

|**name**|
|---|
|Binh Tran|
|Anh Vu|

### Bảng Tổng Kết Cấp Độ 2 (Intermediate)

|**Chủ đề**|**Lệnh SQL chính**|**Mục tiêu kỹ năng**|
|---|---|---|
|**Phân nhóm**|`GROUP BY`, `HAVING`|Tổng hợp dữ liệu thành các nhóm (ví dụ: theo phòng ban) và lọc các nhóm đó.24|
|**Kết nối**|`INNER JOIN`, `LEFT JOIN`|Kết hợp dữ liệu từ nhiều bảng. Hiểu khi nào dùng `INNER` vs `LEFT`.25|
|**Truy vấn con**|`SELECT... WHERE IN (SELECT...)`|Sử dụng một truy vấn bên trong một truy vấn khác để lọc dữ liệu phức tạp.27|
|**Logic điều kiện**|`CASE WHEN... THEN... END`|Tạo các cột mới "phái sinh" và áp dụng logic nghiệp vụ trong SQL.30|
|**Thao tác dữ liệu**|`INSERT`, `UPDATE`, `DELETE`|Sửa đổi, thêm và xóa dữ liệu một cách an toàn.8|
|**Định nghĩa CSDL**|`CREATE/ALTER/DROP TABLE`|Thiết kế và sửa đổi cấu trúc CSDL.7|
|**Tính toàn vẹn**|`PRIMARY KEY`, `FOREIGN KEY`, `CHECK`|Sử dụng ràng buộc để bảo vệ dữ liệu và thực thi các quy tắc nghiệp vụ.40|
|**Toán tử tập hợp**|`UNION`, `UNION ALL`, `EXCEPT`|Kết hợp các tập kết quả từ nhiều truy vấn.44|

---

## Phần 3: Cấp Độ Nâng Cao (Advanced Level)

- **Tổng thời lượng:** 20 giờ
    
- **Mục tiêu:** Viết các truy vấn hiệu quả, dễ đọc và mạnh mẽ cho các tác vụ phân tích phức tạp. Đây là cấp độ phân biệt giữa "người dùng SQL" và "nhà phát triển SQL".
    

### 3.1. Biểu Thức Bảng Chung (Common Table Expressions - CTEs)

- **Thời lượng:** 5 giờ
    
- **Lý thuyết cốt lõi:**
    
    - CTE là một "bảng tạm" (temporary result set) mà bạn định nghĩa ở đầu truy vấn bằng mệnh đề `WITH`.47
        
    - Chúng không được lưu trữ và chỉ tồn tại trong suốt thời gian của một truy vấn duy nhất.
        
    - Mục đích chính: Làm cho các truy vấn phức tạp (với nhiều subquery lồng nhau) trở nên _dễ đọc hơn_ và có cấu trúc hơn.49
        
- **CTE vs. Subquery:**
    
    - Thay vì lồng các subquery trong `FROM` như `SELECT... FROM (SELECT... FROM (SELECT...))` rất khó đọc, CTE cho phép bạn viết truy vấn theo kiểu "tuần tự" hoặc "kể chuyện".
        
    - 1. `WITH buoc_1 AS (... )` -- "Đầu tiên, tôi làm việc này"
            
    - 2. `, buoc_2 AS ( SELECT... FROM buoc_1... )` -- "Tiếp theo, tôi dùng kết quả bước 1 để làm việc này".47
            
    - 3. `SELECT... FROM buoc_2;` -- "Cuối cùng, tôi chọn từ bước 2"
            
- **Database:** `cong_ty` (`employees`, `departments`)
    
- **Query Ví dụ:**
    
    - _Bài toán:_ Tìm tất cả các nhân viên có lương cao hơn lương trung bình của phòng ban _họ_.
        
    - _Cách 1: Dùng Subquery Tương Quan (Khó đọc)_
        
        SQL
        
        ```sql
        SELECT name, salary
        FROM employees e
        WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e.dept_id);
        ```
        
    - _Cách 2: Dùng CTE (Dễ đọc)_ 50
        
        SQL
        
        ```sql
        WITH department_averages (dept_id, avg_salary) AS (
            -- CTE này tính lương trung bình cho mỗi phòng ban
            SELECT dept_id, AVG(salary)
            FROM employees
            GROUP BY dept_id
        )
        -- Truy vấn chính JOIN với CTE
        SELECT e.name, e.salary, da.avg_salary
        FROM employees e
        JOIN department_averages da ON e.dept_id = da.dept_id
        WHERE e.salary > da.avg_salary;
        ```
        
    - _Kết quả minh họa:_ (Charlie (120k) và Grace (85k) có lương cao hơn mức trung bình 95k của phòng Engineering)
        

|**name**|**salary**|**avg_salary**|
|---|---|---|
|Alice|77000.00|67500.00|
|Charlie|120000.00|95000.00|
|Eva|90000.00|90000.00|
|Grace|85000.00|95000.00|

```sql
    -- Chỉnh sửa dữ liệu để ví dụ có ý nghĩa hơn:
    -- INSERT INTO employees (name, salary, dept_id) VALUES ('Grace', 100000, 2);
    -- Giả sử lương Grace là 100k, Bob là 80k, Charlie là 120k -> AVG là 100k
    -- Kết quả sẽ là Charlie.
    -- Cập nhật lại: salary của Alice là 70k, David là 65k -> AVG 67.5k. Alice (70k) xuất hiện.
    -- salary của Bob 80k, Charlie 120k, Grace 85k -> AVG 95k. Charlie (120k) xuất hiện.
```

|**name**|**salary**|**avg_salary**|
|---|---|---|
|Alice|70000.00|67500.00|
|Charlie|120000.00|95000.00|

- **Thực hành:**
    
    - **Yêu cầu:** Sử dụng CTE, tìm phòng ban (tên) có tổng lương cao nhất.
        
    - _Gợi ý:_
        
        1. Tạo CTE `dept_salary` (JOIN `employees` và `departments`, `GROUP BY` `dept_name` để `SUM(salary)`).
            
        2. Truy vấn chính: `SELECT * FROM dept_salary ORDER BY total_salary DESC LIMIT 1;`.
            
    - _Output mong đợi:_
        

|**dept_name**|**total_salary**|
|---|---|
|Engineering|285000.00|

### 3.2. Hàm Cửa Sổ (Window Functions)

- **Thời lượng:** 8 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Đây là tính năng mạnh mẽ nhất cho phân tích trong SQL.
        
    - Giống như hàm tổng hợp (`GROUP BY`), chúng thực hiện các phép tính trên một tập hợp các hàng (gọi là "cửa sổ" - window).51
        
    - **Sự khác biệt cốt lõi:** `GROUP BY` _thu gọn_ (collapses) nhiều hàng thành một hàng duy nhất. Hàm Cửa Sổ _giữ nguyên_ các hàng và trả về kết quả tính toán _bên cạnh_ mỗi hàng đó.51
        
    - Cú pháp: `function() OVER (PARTITION BY... ORDER BY...)`
        
        - `PARTITION BY...`: Chia cửa sổ thành các nhóm (giống `GROUP BY`).53
            
        - `ORDER BY...`: Xác định thứ tự trong mỗi cửa sổ (rất quan trọng cho các hàm xếp hạng).
            
- **Database:** `cong_ty`
    
- **Query Ví dụ:**
    
    - _Ví dụ 1: Tổng hợp cửa sổ (Window Aggregate)_ 51
        
        SQL
        
        ```sql
        -- Lấy lương của mỗi nhân viên VÀ lương trung bình của phòng ban họ
        -- (Bài toán ở mục 3.1, giải quyết bằng Window Function)
        SELECT
            name,
            dept_id,
            salary,
            AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg_salary
        FROM employees;
        ```
        
        _Kết quả minh họa:_ (Chú ý `dept_avg_salary` lặp lại cho mỗi nhân viên trong cùng phòng ban, hàng không bị thu gọn)
        

|**name**|**dept_id**|**salary**|**dept_avg_salary**|
|---|---|---|---|
|Alice|1|70000.00|67500.00|
|David|1|65000.00|67500.00|
|Bob|2|80000.00|95000.00|
|Charlie|2|120000.00|95000.00|
|Grace|2|85000.00|95000.00|
|...|...|...|...|

*   *Ví dụ 2: Xếp hạng (Ranking)* [52, 53, 55]
    *   `ROW_NUMBER()`: Gán số thứ tự duy nhất (1, 2, 3, 4).
    *   `RANK()`: Xếp hạng, có thể có "lỗ hổng" (1, 2, 2, 4).
    *   `DENSE_RANK()`: Xếp hạng, không có "lỗ hổng" (1, 2, 2, 3).
    ```sql
    -- Xếp hạng lương nhân viên trong mỗi phòng ban
    SELECT
        name,
        dept_id,
        salary,
        RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS salary_rank
    FROM employees;
    ```
    *Kết quả minh họa:*


|**name**|**dept_id**|**salary**|**salary_rank**|
|---|---|---|---|
|Alice|1|70000.00|1|
|David|1|65000.00|2|
|Charlie|2|120000.00|1|
|Grace|2|85000.00|2|
|Bob|2|80000.00|3|
|...|...|...|...|

- **Thực hành:**
    
    - **Yêu cầu:** Chỉ lấy 2 nhân viên có lương cao nhất từ _mỗi_ phòng ban.
        
    - _Gợi ý:_ Bạn không thể `WHERE salary_rank <= 2` trong cùng một truy vấn. Bạn phải dùng truy vấn Xếp hạng (Ví dụ 2) làm CTE, sau đó `SELECT * FROM ten_cte WHERE salary_rank <= 2;`.
        
    - _Output mong đợi:_
        

|**name**|**dept_id**|**salary**|**salary_rank**|
|---|---|---|---|
|Alice|1|70000.00|1|
|David|1|65000.00|2|
|Charlie|2|120000.00|1|
|Grace|2|85000.00|2|
|Eva|3|90000.00|1|
|Frank|4|50000.00|1|

### 3.3. Quản Lý Giao Dịch (Transactions)

- **Thời lượng:** 4 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Giao dịch (Transaction) là một nhóm các câu lệnh SQL được coi là _một đơn vị công việc duy nhất_.
        
    - Nó tuân theo tính chất **ACID**:
        
        - **A**tomicity (Nguyên tử): Hoặc _tất cả_ các lệnh thành công, hoặc _không lệnh nào_ thành công (tất cả sẽ bị `ROLLBACK`).
            
        - **C**onsistency (Nhất quán): CSDL luôn ở trạng thái hợp lệ.
            
        - **I**solation (Cô lập): Các giao dịch đồng thời không ảnh hưởng lẫn nhau (xem Phần 4.3).
            
        - **D**urability (Bền vững): Một khi đã `COMMIT`, dữ liệu không bị mất.
            
    - Các lệnh TCL (Transaction Control Language):
        
        - `BEGIN;`: Bắt đầu một giao dịch.
            
        - `COMMIT;`: Lưu vĩnh viễn các thay đổi.
            
        - `ROLLBACK;`: Hủy bỏ tất cả các thay đổi kể từ `BEGIN;`.
            
- **Database Mẫu:** `banking`
    
    SQL
    
    ```sql
    DROP TABLE IF EXISTS accounts;
    CREATE TABLE accounts (id INT PRIMARY KEY, balance NUMERIC(10, 2) NOT NULL);
    INSERT INTO accounts VALUES (1, 1000.00), (2, 500.00);
    ```
    
- **Query Ví dụ (Kịch bản chuyển tiền):**
    
    - _Bài toán:_ Chuyển $200 từ tài khoản 1 sang tài khoản 2. Cả hai lệnh `UPDATE` phải cùng thành công hoặc cùng thất bại (ví dụ: nếu tài khoản 1 không đủ tiền, cả giao dịch phải bị hủy).
        
    
    SQL
    
    ```sql
    BEGIN; -- Bắt đầu giao dịch
    
    -- Trừ tiền tài khoản 1
    UPDATE accounts SET balance = balance - 200 WHERE id = 1;
    
    -- Thêm tiền tài khoản 2
    UPDATE accounts SET balance = balance + 200 WHERE id = 2;
    
    -- (Giả sử có một bước kiểm tra, nếu thất bại, chúng ta chạy ROLLBACK)
    -- IF (SELECT balance FROM accounts WHERE id = 1) < 0 THEN
    --     ROLLBACK;
    -- ELSE
    --     COMMIT;
    -- END IF;
    
    COMMIT; -- Nếu mọi thứ ổn, xác nhận thay đổi
    
    -- Thử chạy lại kịch bản, nhưng thay COMMIT bằng ROLLBACK
    -- bạn sẽ thấy số dư quay trở lại 1000 và 500.
    ```
    
- **Thực hành:**
    
    - **Yêu cầu:** Mở hai cửa sổ SQL Editor trong DBeaver (để mô phỏng hai kết nối).
        
    - _Editor 1:_ Chạy `BEGIN;`.
        
    - _Editor 1:_ Chạy `UPDATE accounts SET balance = 9999 WHERE id = 1;` (Không COMMIT).
        
    - _Editor 2:_ Chạy `SELECT * FROM accounts WHERE id = 1;`. Bạn thấy gì? (Bạn sẽ thấy `balance = 1000` (hoặc 800 từ ví dụ trên) vì Editor 1 chưa `COMMIT`).
        
    - _Editor 1:_ Chạy `COMMIT;`.
        
    - _Editor 2:_ Chạy lại `SELECT * FROM accounts WHERE id = 1;`. Bây giờ bạn thấy gì? (Bây giờ bạn sẽ thấy `balance = 9999`).
        
    - -> Bạn vừa chứng kiến cấp độ cô lập `Read Committed` (sẽ học ở Phần 4.3).
        

---

## Phần 4: Cấp Độ Chuyên Gia (Expert Level - Tối Ưu & Quản Trị)

- **Tổng thời lượng:** 25-30 giờ
    
- **Mục tiêu:** Chuyển từ "viết query chạy được" sang "viết query chạy nhanh". Đây là nơi DBA và Kỹ sư Dữ liệu tỏa sáng. Chúng ta sẽ làm việc với một CSDL lớn.
    

### 4.1. Đánh Chỉ Mục (Indexing)

- **Thời lượng:** 6 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Chỉ mục (Index) giống như mục lục của một cuốn sách.56 Thay vì lật từng trang (Full Table Scan) để tìm một chủ đề, bạn xem mục lục và đi thẳng đến trang đó (Index Scan).
        
    - Nó tăng tốc độ `SELECT` (đọc) một cách đáng kể, đặc biệt là với `WHERE`, `JOIN`, và `ORDER BY`.57
        
    - **Nhược điểm (Trade-off):** Nó làm chậm các thao tác `INSERT`, `UPDATE`, `DELETE` (ghi) vì CSDL phải cập nhật cả bảng và chỉ mục.57
        
- **Khi nào (và không) dùng Index:**
    
    - **NÊN:** Index các cột Khóa ngoại (Foreign Key), các cột thường dùng trong `WHERE`, và các cột dùng trong `ORDER BY`.57
        
    - **KHÔNG NÊN:** Index mọi cột (Over-indexing).58
        
    - **KHÔNG NÊN:** Index các cột có "tính chọn lọc thấp" (low cardinality), ví dụ: cột "giới tính" (chỉ có 2-3 giá trị). Index trên cột này gần như vô dụng.57
        
- **Database Mẫu:** `ecommerce_products` (Bảng này cần lớn để thấy sự khác biệt).
    
    SQL
    
    ```sql
    DROP TABLE IF EXISTS ecommerce_products;
    CREATE TABLE ecommerce_products (
        id SERIAL PRIMARY KEY,
        product_name VARCHAR(255),
        category_id INT,
        price NUMERIC(10, 2),
        in_stock BOOLEAN DEFAULT true
    );
    -- Chèn 1 triệu hàng giả lập (có thể mất 1-2 phút)
    INSERT INTO ecommerce_products (product_name, category_id, price, in_stock)
    SELECT
        'Product ' |
    
    ```

| s.n,
```sql
(random() * 50)::INT + 1,

(random() * 1000)::NUMERIC(10, 2),

(random() > 0.1)

FROM generate_series(1, 1000000) s(n);

```

- **Query Ví dụ (Tạo Index):**
    
    - _Ví dụ 1: Index B-Tree (mặc định)_
        
        SQL
        
        ```sql
        -- Tạo index trên cột hay được tìm kiếm
        CREATE INDEX idx_products_category_id ON ecommerce_products(category_id);
        ```
        
    - _Ví dụ 2: Index kết hợp (Composite Index)_
        
        SQL
        
        ```sql
        -- Tối ưu cho query lọc theo cả category và price
        CREATE INDEX idx_products_category_price ON ecommerce_products(category_id, price);
        ```
        
    - _Ví dụ 3: Index một phần (Partial Index)_ 58
        
        SQL
        
        ```sql
        -- Rất hiệu quả nếu bạn chỉ tìm các sản phẩm hết hàng (một phần nhỏ)
        -- Index này sẽ RẤT NHỎ và nhanh
        CREATE INDEX idx_products_out_of_stock ON ecommerce_products(id) WHERE in_stock = false;
        ```
        
- **Thực hành:** (Kết hợp với mục 4.2)
    

### 4.2. Phân Tích Kế Hoạch Truy Vấn (Query Plan Analysis)

- **Thời lượng:** 8 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Làm thế nào bạn biết query của mình chậm, và _tại sao_?
        
    - `EXPLAIN`: Hiển thị kế hoạch _ước tính_ mà bộ tối ưu của CSDL sẽ sử dụng.
        
    - `EXPLAIN ANALYZE`: _Thực thi_ truy vấn và hiển thị kế hoạch _thực tế_ đã chạy, kèm theo thời gian thực thi (actual time) và số hàng thực tế (actual rows).60 Đây là công cụ chẩn đoán quan trọng nhất của bạn.
        
- **Cách Đọc `EXPLAIN ANALYZE`:** 61
    
    - **Tìm điểm nghẽn:** Tìm các nút (nodes) có `actual time` cao nhất.61
        
    - **Tìm `Seq Scan`:** Tìm kiếm `Sequential Scan` (Quét toàn bộ bảng) trên một bảng lớn. Nếu có `Filter` trên `Seq Scan` đó, đây là dấu hiệu rõ ràng nhất cho thấy bạn _thiếu index_.61
        
    - **So sánh Ước tính vs. Thực tế:** So sánh `rows=...` (ước tính) và `actual rows=...` (thực tế). Nếu chúng chênh lệch _rất lớn_ (ví dụ: `rows=1` nhưng `actual rows=500000`), điều này có nghĩa là thống kê của CSDL bị cũ, dẫn đến việc chọn kế hoạch truy vấn sai. Bạn có thể sửa bằng cách chạy `ANALYZE ten_bang;`.61
        
- **Database:** `ecommerce_products` (từ 4.1)
    
- **Query Ví dụ / Thực hành (So sánh A/B):**
    
    - **Bước 1 (Trước khi Index):** Chạy `EXPLAIN ANALYZE`
        
        SQL
        
        ```sql
        EXPLAIN ANALYZE
        SELECT * FROM ecommerce_products WHERE category_id = 25 AND price > 900;
        ```
        
        _Kết quả minh họa (Trước Index):_
        
        ```sql
        Gather  (cost=1000.00..21448.88) (rows=198) (actual time=1.859..183.170)
          Workers Planned: 2
          ->  Parallel Seq Scan on ecommerce_products  (cost=0.00..20429.08) (rows=82)
                Filter: ((category_id = 25) AND (price > 900::numeric))
        ```
        
    
    ...
    
    Planning Time:... ms
    
    Execution Time: 183.500 ms <--- Rất chậm
        
    - **Bước 2: Tạo Index (từ 4.1):**
        
        SQL
        
        ```sql
        CREATE INDEX idx_products_category_price ON ecommerce_products(category_id, price);
        ```
        
    - **Bước 3 (Sau khi Index):** Chạy lại `EXPLAIN ANALYZE`
        
        SQL
        
        ```sql
        EXPLAIN ANALYZE
        SELECT * FROM ecommerce_products WHERE category_id = 25 AND price > 900;
        ```
        
        _Kết quả minh họa (Sau Index):_
        
        ```sql
        Bitmap Heap Scan on ecommerce_products  (cost=17.20..271.74) (rows=198) (actual time=0.076..0.211)
          Recheck Cond: ((category_id = 25) AND (price > 900::numeric))
          ->  Bitmap Index Scan on idx_products_category_price  (cost=0.00..17.15) (rows=198)
                Index Cond: ((category_id = 25) AND (price > 900::numeric))
        ```
        
    
    ...
    
    Planning Time:... ms
    
    Execution Time: 0.250 ms <--- Cực kỳ nhanh (nhanh hơn ~700 lần)
        
    - **Kết luận:** Bạn đã chứng minh được rằng Index đã thay đổi `Parallel Seq Scan` (quét toàn bộ 1 triệu hàng) thành `Bitmap Index Scan` (chỉ đọc index), giúp giảm thời gian từ 183ms xuống 0.25ms.
        

### 4.3. Các Cấp Độ Cô Lập Giao Dịch (Transaction Isolation)

- **Thời lượng:** 4 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Phần 3.3 là về ACID. Phần này là về chữ 'I' (Isolation).
        
    - Khi nhiều giao dịch chạy cùng lúc (concurrency), các vấn đề có thể xảy ra 62:
        
        - **Dirty Read:** Đọc dữ liệu chưa `COMMIT` (PostgreSQL không cho phép điều này).
            
        - **Non-Repeatable Read:** Đọc cùng một hàng 2 lần và thấy giá trị khác nhau (vì ai đó đã `UPDATE` và `COMMIT` ở giữa).62
            
        - **Phantom Read:** Đọc một tập hợp hàng 2 lần và thấy _nhiều hàng hơn_ (vì ai đó đã `INSERT` và `COMMIT` ở giữa).62
            
    - Bốn cấp độ cô lập 77:
        
        - `Read Uncommitted`: (PostgreSQL không hỗ trợ, nó tự nâng lên `Read Committed`).62
            
        - `Read Committed` (Mặc định của PG) 63: Chỉ thấy dữ liệu đã commit. _Cho phép_ Non-Repeatable và Phantom Reads. Nhanh, tốt cho hầu hết ứng dụng web.
            
        - `Repeatable Read`: Đảm bảo bạn thấy cùng một "ảnh chụp" (snapshot) dữ liệu trong suốt giao dịch. _Ngăn_ Non-Repeatable Read, nhưng _vẫn cho phép_ Phantom Read (trong một số trường hợp).63
            
        - `Serializable` (S92): Cấp độ nghiêm ngặt nhất. Ngăn cả Phantom Read. Đảm bảo các giao dịch như thể chúng chạy _tuần tự_. Rất chậm và có thể gây lỗi serialization.
            
- **Database:** `banking` (từ 3.3)
    
- **Query Ví dụ (Mô phỏng Non-Repeatable Read):**
    
    - _Editor 1 (Session A):_
        
        SQL
        
        ```sql
        BEGIN;
        SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
        SELECT * FROM accounts WHERE id = 1; -- Thấy balance = 1000
        -- (chờ...)
        SELECT * FROM accounts WHERE id = 1; -- Thấy balance = 800 (BỊ LỖI NON-REPEATABLE READ)
        COMMIT;
        ```
        
    - _Editor 2 (Session B):_
        
        SQL
        
        ```sql
        -- (sau khi Session A chạy SELECT lần 1)
        UPDATE accounts SET balance = 800 WHERE id = 1;
        COMMIT; -- Thay đổi này được commit
        ```
        
- **Thực hành:**
    
    - **Yêu cầu:** Lặp lại kịch bản trên, nhưng lần này Session A đặt `ISOLATION LEVEL REPEATABLE READ`.
        
    - _Editor 1 (Session A):_
        
        SQL
        
        ```sql
        BEGIN;
        SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
        SELECT * FROM accounts WHERE id = 1; -- Thấy balance = 800
        -- (chờ Session B chạy và COMMIT)
        SELECT * FROM accounts WHERE id = 1; -- Vẫn thấy balance = 800 (ĐÃ NGĂN ĐƯỢC LỖI)
        COMMIT;
        ```
        
    - _Editor 2 (Session B):_ (Chạy `UPDATE accounts SET balance = 500 WHERE id = 1; COMMIT;`)
        
    - _Output mong đợi:_ Session A sẽ thấy $800 ở _cả hai_ lần `SELECT`, vì nó đang nhìn vào một "ảnh chụp" (snapshot) tại thời điểm `BEGIN;`. Nó được bảo vệ khỏi thay đổi của Session B.
        

### 4.4. Bảo Mật và Phân Quyền (RBAC)

- **Thời lượng:** 3 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Không ai (ngoại trừ DBA) nên dùng tài khoản `postgres` (superuser) trong ứng dụng hàng ngày.
        
    - PostgreSQL sử dụng Role-Based Access Control (RBAC). Bạn tạo `ROLE` (vai trò) và cấp quyền cho vai trò đó.64
        
    - `CREATE ROLE role_name;`: Tạo một vai trò. `LOGIN` có nghĩa là vai trò này có thể đăng nhập.
        
    - `GRANT [privilege] ON [object] TO [role_name];` (S99, S125): Cấp quyền.
        
    - `REVOKE [privilege] ON [object] FROM [role_name];` (S98): Thu hồi quyền.
        
    - Quyền (Privilege) bao gồm: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `USAGE` (trên schema), `CONNECT` (trên database).65
        
- **Database:** `cong_ty`
    
- **Query Ví dụ:**
    
    SQL
    
    ```sql
    -- Tạo một vai trò (người dùng) chỉ có thể đăng nhập
    CREATE ROLE data_analyst LOGIN PASSWORD 'SecurePass123';
    
    -- Cấp cho họ quyền kết nối vào CSDL
    GRANT CONNECT ON DATABASE postgres TO data_analyst; -- (Giả sử tên DB là postgres)
    
    -- Cấp cho họ quyền sử dụng schema 'public'
    GRANT USAGE ON SCHEMA public TO data_analyst;
    
    -- Cấp cho họ quyền CHỈ ĐỌC (SELECT) trên TẤT CẢ các bảng trong schema
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO data_analyst; [67]
    
    -- QUAN TRỌNG: Cấp quyền cho các bảng được tạo TRONG TƯƠNG LAI
    ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO data_analyst; [67]
    
    -- Tạo vai trò HR, chỉ có thể XEM và CẬP NHẬT cột 'salary'
    CREATE ROLE hr_manager LOGIN PASSWORD 'SuperSecurePass';
    GRANT CONNECT ON DATABASE postgres TO hr_manager;
    GRANT USAGE ON SCHEMA public TO hr_manager;
    GRANT SELECT ON employees TO hr_manager;
    GRANT UPDATE(salary) ON employees TO hr_manager; -- Chỉ UPDATE cột 'salary'
    GRANT SELECT ON departments TO hr_manager;
    
    -- Thu hồi quyền
    REVOKE UPDATE(salary) ON employees FROM hr_manager;
    ```
    
- **Thực hành:**
    
    - **Yêu cầu:** Tạo một vai trò `intern` (thực tập sinh) chỉ có thể `SELECT` cột `name` và `dept_id` (không phải `salary`) từ bảng `employees`.
        
    - _Gợi ý:_ Bạn không thể `GRANT SELECT(name, dept_id)` (quyền `SELECT` chỉ ở cấp độ bảng). Bạn phải tạo một `VIEW` (đối tượng ảo) và cấp quyền trên `VIEW` đó.
        
    - `CREATE VIEW employee_public_view AS SELECT name, dept_id FROM employees;`
        
    - `CREATE ROLE intern LOGIN PASSWORD '...';`
        
    - `GRANT USAGE ON SCHEMA public TO intern;`
        
    - `GRANT SELECT ON employee_public_view TO intern;`
        
    - _Output mong đợi:_ Khi `intern` đăng nhập, họ không thể `SELECT * FROM employees` (permission denied), nhưng có thể `SELECT * FROM employee_public_view`.
        

### 4.5. Sao Lưu và Phục Hồi (Backup & Restore)

- **Thời lượng:** 3 giờ
    
- **Lý thuyết cốt lõi:**
    
    - Nếu dữ liệu của bạn không được sao lưu, nó không tồn tại.
        
    - **Backup logic (`pg_dump`):** Xuất CSDL thành một tệp `.sql` (plain text) hoặc `.dump` (custom format).68 Tệp này chứa các lệnh SQL để tạo lại CSDL.
        
    - **Restore logic (`psql` hoặc `pg_restore`):**
        
        - `psql`: Dùng để khôi phục tệp `.sql` (plain text).69
            
        - `pg_restore`: Dùng để khôi phục tệp `.dump` (custom/directory format).71 `pg_restore` mạnh mẽ hơn, hỗ trợ song song và khôi phục có chọn lọc.68
            
- **Lưu ý: Plain vs. Custom format**
    
    - `pg_dump -f backup.sql` (Plain text): Dễ đọc, nhưng chậm và không linh hoạt.
        
    - `pg_dump -Fc -f backup.dump` (Custom format): _Luôn được khuyên dùng_. Tệp này được nén (nhỏ hơn) và chỉ có thể được đọc bởi `pg_restore`, cho phép bạn khôi phục song song (nhanh hơn).68
        
- **Query Ví dụ (Thực thi trong Terminal/Command Prompt, _không phải_ SQL Editor):**
    
    - **Bước 1: Backup (Sử dụng Custom Format)** 70
        
        Bash
        
        ```bash
        # -U: User (Tên người dùng)
        # -W: Hỏi mật khẩu (Password)
        # -F c: Định dạng Custom (tốt nhất)
        # -d: Tên database
        # >: Ghi ra tệp (hoặc -f <filename>)
        pg_dump -U postgres -W -F c -d cong_ty > D:\Backups\cong_ty_backup.dump
        ```
        
    - **Bước 2: Restore (vào một CSDL MỚI)** 70
        
        Bash
        
        ```bash
        # Tạo một CSDL mới để chứa bản khôi phục
        createdb -U postgres -W cong_ty_new
        
        # Khôi phục bằng pg_restore
        # -d: Tên database ĐÍCH
        pg_restore -U postgres -W -d cong_ty_new D:\Backups\cong_ty_backup.dump
        ```
        
- **Thực hành:**
    
    - **Yêu cầu:** Thực hiện các bước trên: Sao lưu CSDL `cong_ty` của bạn ra tệp `.dump`. Xóa (DROP) CSDL `cong_ty` (cẩn thận!). Tạo CSDL `cong_ty_restored`. Khôi phục bản sao lưu vào đó.
        
    - _Output mong đợi:_ Bạn có thể kết nối vào `cong_ty_restored` và `SELECT * FROM employees` thấy tất cả dữ liệu của mình.
        

### Bảng Tổng Kết Cấp Độ 3 & 4 (Advanced/Expert)

|**Chủ đề**|**Lệnh SQL chính**|**Mục tiêu kỹ năng**|
|---|---|---|
|**CTEs**|`WITH... AS (SELECT...)`|Viết các truy vấn phân tích phức tạp, nhiều bước một cách rõ ràng, dễ đọc.47|
|**Window Functions**|`OVER (PARTITION BY...)`|Thực hiện các phép tính tinh vi (xếp hạng, tổng dồn) mà không thu gọn hàng.51|
|**Transactions**|`BEGIN`, `COMMIT`, `ROLLBACK`|Đảm bảo tính toàn vẹn dữ liệu (ACID) khi thực hiện nhiều lệnh liên quan.|
|**Indexing**|`CREATE INDEX`|Hiểu sự đánh đổi của Index và biết cách tạo index để tăng tốc `SELECT`.56|
|**Query Tuning**|`EXPLAIN ANALYZE`|Đọc và diễn giải kế hoạch thực thi để tìm ra lý do query bị chậm.60|
|**Isolation**|`SET TRANSACTION...`|Hiểu các rủi ro về đồng thời (concurrency) và chọn cấp độ cô lập phù hợp.62|
|**Security**|`CREATE ROLE`, `GRANT`, `REVOKE`|Tạo người dùng và quản lý quyền truy cập một cách chi tiết (RBAC).65|
|**Backup/Restore**|`pg_dump`, `pg_restore`|Thực hiện các tác vụ quản trị thiết yếu để bảo vệ dữ liệu.68|

---

## Phần 5: Tổng Kết & Đồ Án Cuối Khóa (Capstone)

### 5.1. Bảng Tổng Kết Toàn Bộ Lộ Trình (Tóm tắt ở trên)

### 5.2. Đồ Án Cuối Khóa: Phân Tích CSDL E-commerce

- **Thời lượng ước tính:** 10-15 giờ
    
- **Mô Tả:** Bạn được cung cấp một CSDL E-commerce.75 Nhiệm vụ của bạn là thực hiện một loạt các tác vụ "full-stack" (từ DDL đến Phân tích) để chứng minh sự thành thạo của mình với toàn bộ kiến thức đã học.
    
- **Schema Đồ Án:**
    
    SQL
    
    ```sql
    DROP TABLE IF EXISTS order_items, orders, products, categories, customers CASCADE;
    
    CREATE TABLE customers (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        email VARCHAR(100) UNIQUE
    );
    CREATE TABLE categories (
        id SERIAL PRIMARY KEY,
        category_name VARCHAR(50) NOT NULL
    );
    CREATE TABLE products (
        id SERIAL PRIMARY KEY,
        product_name VARCHAR(255),
        category_id INT REFERENCES categories(id),
        price NUMERIC(10, 2) CHECK (price > 0)
    );
    CREATE TABLE orders (
        id SERIAL PRIMARY KEY,
        customer_id INT REFERENCES customers(id),
        order_date DATE DEFAULT CURRENT_DATE,
        status VARCHAR(20) DEFAULT 'Pending' CHECK (status IN ('Pending', 'Shipped', 'Cancelled'))
    );
    CREATE TABLE order_items (
        order_id INT REFERENCES orders(id),
        product_id INT REFERENCES products(id),
        quantity INT CHECK (quantity > 0),
        PRIMARY KEY (order_id, product_id) -- Composite Primary Key
    );
    
    -- (Tự chèn dữ liệu mẫu cho mỗi bảng)
    ```
    
- **Yêu Cầu Nhiệm Vụ:**
    
    1. **DDL & DML:** Tạo schema trên và tự chèn dữ liệu (ít nhất 5-10 hàng cho mỗi bảng).
        
    2. **Truy Vấn Phân Tích (DQL):**
        
        - **Q1 (JOIN, GROUP BY):** Tìm 5 khách hàng chi tiêu nhiều nhất (Tổng tiền = `SUM(oi.quantity * p.price)`).
            
        - **Q2 (CTE, Date):** Tính tổng doanh thu theo tháng (sử dụng hàm `DATE_TRUNC('month', order_date)`).
            
        - **Q3 (Window Function):** Tìm 3 sản phẩm bán chạy nhất (theo số lượng) trong _mỗi_ danh mục. (Bài toán "Top N per Group").
            
        - **Q4 (Subquery, EXCEPT):** Tìm các sản phẩm _chưa bao giờ_ được bán.
            
        - **Q5 (Tự JOIN):** Tìm các khách hàng đã mua sản phẩm 'A' và 'B' trong _cùng một_ đơn hàng.
            
    3. **Quản Trị (TCL & Expert):**
        
        - **Q6 (Transaction):** Viết một `Transaction` để xử lý một đơn hàng mới. Nó phải bao gồm:
            
            - `INSERT` vào `orders`.
                
            - `INSERT` vào `order_items`.
                
            - (Giả định có bảng `inventory`): `UPDATE inventory SET stock = stock - quantity`.
                
            - Nếu bất cứ điều gì thất bại (ví dụ: `stock < quantity`), toàn bộ giao dịch phải `ROLLBACK`.
                
        - **Q7 (Tối ưu):** Chạy `EXPLAIN ANALYZE` trên Q1. Đề xuất và tạo `INDEX` (ví dụ: trên `customer_id` của `orders` và `product_id` của `order_items`). Chạy lại `EXPLAIN ANALYZE` để chứng minh sự cải thiện.
            
        - **Q8 (Bảo mật):** Tạo một `ROLE` tên là `sales_analytics` chỉ có quyền `SELECT` trên tất cả các bảng.
            

## Lời Kết

Nếu bạn đã hoàn thành đồ án cuối khóa, bạn không chỉ học được SQL. Bạn đã học cách suy nghĩ như một Kỹ sư Dữ liệu. Bạn đã học cách đặt câu hỏi, cách tìm câu trả lời, cách bảo vệ dữ liệu và cách làm cho nó hoạt động nhanh hơn. Chào mừng đến với thế giới của dữ liệu. Giờ thì hãy thêm "SQL (PostgreSQL)" vào CV của bạn với sự tự tin.