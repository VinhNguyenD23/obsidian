## 🗺️ Giai đoạn 1 (Chi tiết): Nền tảng, Cài đặt & Job "Freestyle"

### 🎯 Mục tiêu

Thực sự _cài đặt_ được Jenkins Controller, hiểu rõ các thành phần cấu hình cốt lõi, và _tự tay_ tạo ra một quy trình build đơn giản (Freestyle Job) để hiểu các khối xây dựng cơ bản: **Get Code** -> **Build Code** -> **Save Artifacts**.

---

### 1.1. Cài đặt Jenkins Controller

Đây là bước đầu tiên. Trong doanh nghiệp, bạn sẽ cài Jenkins trên một máy chủ Linux chuyên dụng. Nhưng để học, **Docker là cách nhanh nhất và sạch nhất**.

#### 🔑 Keyword: Docker, Volume, Port Mapping

#### ⌨️ Ví dụ: Cài đặt bằng Docker (Khuyến nghị)

Chúng ta sẽ dùng `docker-compose.yml` vì nó dễ quản lý hơn `docker run`.

1. Tạo một file tên `docker-compose.yml`:
    

YAML

```
# docker-compose.yml
version: '3.8'

services:
  jenkins:
    # 'lts' = Long-Term Support, phiên bản ổn định cho doanh nghiệp
    image: jenkins/jenkins:lts-jdk11 
    privileged: true # Cần thiết nếu bạn muốn chạy Docker-in-Docker sau này
    user: root # Chạy với quyền root để có thể cài đặt thêm tool (như docker client)
    ports:
      # Cổng 8080 trên máy thật (host) sẽ trỏ vào cổng 8080 của container (Jenkins UI)
      - "8080:8080"
      # Cổng 50000 dùng cho các agent kết nối vào (sẽ học ở Giai đoạn 4)
      - "50000:50000"
    container_name: jenkins-lts
    volumes:
      # Đây là phần quan trọng nhất: 'jenkins_home' là nơi lưu trữ toàn bộ dữ liệu 
      # (cấu hình, job, plugin). Dùng 'named volume' giúp dữ liệu tồn tại
      # ngay cả khi bạn xóa container.
      - jenkins_home:/var/jenkins_home

volumes:
  # Khai báo 'named volume'
  jenkins_home:
```

2. Trong thư mục chứa file đó, chạy lệnh:
    
    Bash
    
    ```
    docker-compose up -d
    ```
    
3. Sau khi chạy, Jenkins cần vài phút để khởi động. Bạn cần lấy mật khẩu admin ban đầu:
    
    Bash
    
    ```
    docker logs jenkins-lts
    ```
    
    Bạn sẽ thấy một đoạn log giống như sau (copy đoạn mật khẩu dài):
    
    ```
    *************************************************************
    
    Jenkins initial admin password: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    
    *************************************************************
    ```
    

---

### 1.2. Cài đặt sau khi cài (Post-installation Wizard)

Đây là trình hướng dẫn bạn thấy khi truy cập `http://localhost:8080` lần đầu tiên.

1. **Unlock Jenkins:** Dán (paste) mật khẩu admin bạn vừa copy từ log vào.
    
2. **Customize Jenkins:** Màn hình này hỏi bạn cài plugin.
    
    - **Luôn chọn: "Install suggested plugins" (Cài đặt plugin được đề xuất).**
        
    - Tại sao? Nó sẽ tự động cài các plugin _thiết yếu_ mà chúng ta sẽ cần ở các giai đoạn sau, ví dụ: `pipeline`, `git`, `credentials`, `workflow-aggregator`.
        
3. **Create First Admin User:** Tạo tài khoản admin của riêng bạn (ví dụ: user `vinh`, password `123456`). **Đừng bỏ qua bước này!** Sau bước này, bạn sẽ đăng nhập bằng tài khoản này.
    
4. **Instance Configuration:** Xác nhận URL của Jenkins (thường là `http://localhost:8080/` là đủ).
    

---

### 1.3. Khám phá Giao diện (UI Tour)

Sau khi đăng nhập, giao diện Jenkins (còn gọi là "Blue Ocean" hoặc "Classic UI") có thể hơi rối. Hãy tập trung vào mục quan trọng nhất ở cột bên trái: **Manage Jenkins** (Quản lý Jenkins).

Đây là "phòng điều khiển" của bạn. Các mục quan trọng nhất cần biết:

|**Mục trong Manage Jenkins**|**Giải thích chi tiết**|
|---|---|
|**System Configuration**|(Cấu hình Hệ thống) Nơi bạn cài đặt các biến toàn cục (global), URL của Jenkins, cấu hình email server để gửi thông báo.|
|**Security**|(Bảo mật) Quản lý ai có quyền làm gì (ví dụ: Matrix Security), ai được đăng nhập (ví dụ: dùng user/pass của Jenkins, hoặc LDAP/AD của công ty).|
|**Manage Plugins**|(Quản lý Plugin) "Cửa hàng ứng dụng" của Jenkins. Đây là nơi bạn cài, gỡ, cập nhật plugin. **Bạn sẽ vào đây rất thường xuyên.**|
|**Manage Nodes and Clouds**|(Quản lý Node/Cloud) Nơi bạn thêm/cấu hình các **Agents** (máy thực thi build). Đây là trái tim của Giai đoạn 4.|
|**Manage Credentials**|(Quản lý Credentials) Nơi bạn lưu trữ an toàn các "bí mật" (mật khẩu, API key, SSH key). Đây là trái tim của Giai đoạn 5.|
|**Tools**|(Cấu hình Công cụ) Nơi bạn chỉ định đường dẫn của các công cụ (ví dụ: JDK 11, Node.js 16, Maven) nếu bạn cài chúng _trực tiếp_ trên agent.|

---

### 1.4. Tạo Job đầu tiên (Freestyle Project)

Chúng ta sẽ tạo một job "cổ điển" (Freestyle) để build một dự án Node.js đơn giản.

1. Từ trang chủ Jenkins, nhấn **New Item** (Tạo mục mới).
    
2. Đặt tên: `my-first-node-job`.
    
3. Chọn **Freestyle project** và nhấn OK.
    

Bạn sẽ được đưa đến trang cấu hình job. Đây là các tab quan trọng:

#### Tab 1: Source Code Management (SCM)

Đây là nơi bạn nói Jenkins lấy code từ đâu.

- Chọn **Git**.
    
- **Repository URL:** Dán một URL dự án public, ví dụ: `https://github.com/jenkins-docs/simple-node-js-react-app.git`
    
- **Branch Specifier:** Để `*/main` hoặc `*/master` (tùy vào nhánh chính của repo).
    

#### Tab 2: Build Triggers (Trình kích hoạt Build)

Đây là nơi bạn quyết định _khi nào_ chạy job này.

- **Poll SCM:** (Cách cũ) Jenkins tự hỏi Git "có gì mới không?" mỗi X phút.
    
    - Ví dụ: `H/5 * * * *` (kiểm tra mỗi 5 phút).
        
- **GitHub hook trigger...:** (Cách chuẩn) GitHub _chủ động_ báo cho Jenkins "có code mới, build đi!" ngay lập tức. (Cách này cần cấu hình Webhook bên GitHub, phức tạp hơn cho lần đầu).
    
- _Bây giờ, hãy để trống để chúng ta tự build bằng tay._
    

#### Tab 3: Build Steps (Các bước Build)

Đây là phần "thịt" của job. Bạn ra lệnh cho Jenkins làm gì _sau khi_ lấy code.

- Nhấn **Add build step**.
    
- Chọn **Execute shell** (vì chúng ta đang chạy trên container Linux).
    
- Trong ô lệnh, gõ:
    

Bash

```
# In ra các biến môi trường để debug
echo "Build đang chạy tại thư mục: $WORKSPACE"
echo "Build ID là: $BUILD_NUMBER"

# Hiển thị phiên bản các công cụ (giả sử đã cài sẵn trong image)
# Image 'jenkins/jenkins:lts-jdk11' đã có sẵn node
node -v
npm -v

# Các lệnh build thực tế
echo "--- BẮT ĐẦU CÀI ĐẶT DEPENDENCIES ---"
npm install

echo "--- BẮT ĐẦU CHẠY UNIT TEST ---"
npm test
```

#### Tab 4: Post-build Actions (Hành động sau Build)

Đây là những việc cần làm _sau khi_ các bước build chạy xong (dù thành công hay thất bại).

- Nhấn **Add post-build action**.
    
- Chọn **Archive the artifacts** (Lưu trữ artifacts).
    
- **Files to archive:** Đây là nơi bạn chỉ định file/folder "thành phẩm" muốn Jenkins lưu lại.
    
    - Ví dụ: Nếu `npm test` tạo ra 1 file báo cáo `test-report.xml`, bạn sẽ điền `test-report.xml`.
        
    - Để ví dụ, hãy gõ `package-lock.json` (chỉ để xem nó hoạt động).
        

Nhấn **Save** (Lưu) để hoàn tất.

---

### 1.5. Chạy & Phân tích Build

1. Sau khi lưu, bạn sẽ ở trang của Job. Nhấn vào **Build Now** (Build Ngay) ở cột bên trái.
    
2. Một build mới sẽ xuất hiện ở mục **Build History** (ví dụ: `#1`).
    
3. Nhấn vào build `#1` đó.
    
4. Bạn sẽ thấy 2 mục quan trọng nhất:
    
    - **Console Output (Đầu ra Console):** Đây là **file log** của build. **Bạn phải học cách đọc file này.** Đây là nơi bạn xem các lệnh `echo` của mình, và quan trọng nhất là xem _lỗi_ nếu build thất bại (failed).
        
    - **Workspace (Không gian làm việc):** Đây là "thư mục" trên agent nơi Jenkins clone code và chạy build. Bạn có thể duyệt file ở đây.
        
    - **Artifacts:** Nếu build thành công, bạn sẽ thấy file `package-lock.json` (hoặc file bạn đã cấu hình) xuất hiện ở đây.
    