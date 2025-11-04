## 📚 1. Shared Library (Thư viện dùng chung)

### 💡 Nó là gì?

**Shared Library** là một **kho code (repository) Git riêng**, chứa các đoạn mã `Jenkinsfile` (viết bằng Groovy) mà bạn muốn **tái sử dụng** ở nhiều dự án khác nhau.

Hãy nghĩ nó như một "hàm" (function) hoặc một "bộ công cụ" (toolkit) cho Jenkins Pipeline của bạn.

### ❓ Vấn đề nó giải quyết?

Giả sử công ty bạn có 50 dự án (microservices) khác nhau, nhưng tất cả đều:

1.  Build bằng Node.js.
2.  Chạy `npm test`.
3.  Quét SonarQube.
4.  Build Docker image.
5.  Đẩy lên Docker Registry.

Nếu không có Shared Library, bạn sẽ phải **copy-paste** cái `Jenkinsfile` (dài 80 dòng) vào **50 dự án khác nhau**.

**Vấn đề:** Khi sếp yêu cầu thêm một bước (ví dụ: "Thêm 1 stage quét bảo mật"), bạn sẽ phải đi sửa **50** file `Jenkinsfile` đó. Rất tốn thời gian và dễ sai sót.

### ⚙️ Nó hoạt động như thế nào?

1.  **Tạo một Repo Git mới:** Bạn tạo một repo Git riêng, ví dụ tên là `my-jenkins-library`.

2.  **Tạo file hàm (Groovy):**

      * Trong repo đó, bạn tạo thư mục `vars/`.
      * Trong `vars/`, bạn tạo file `standardNodeBuild.groovy`. (Tên file `standardNodeBuild` sẽ là **tên hàm** bạn gọi).
      * Nội dung file `standardNodeBuild.groovy`:
        ```groovy
        // vars/standardNodeBuild.groovy
        // 'call' là hàm đặc biệt sẽ được chạy khi bạn gọi tên file
        def call(Map config) {
            // 'config' là các tham số bạn truyền vào
            
            // BẠN DÁN TOÀN BỘ LOGIC PIPELINE 80 DÒNG VÀO ĐÂY
            pipeline {
                agent none
                stages {
                    stage('Build & Test') {
                        agent { docker { image "node:${config.nodeVersion}-alpine" } }
                        steps {
                            sh 'npm install'
                            sh 'npm test'
                        }
                    }
                    stage('SonarQube') {
                        // ...
                    }
                    stage('Build Docker') {
                        // ... sử dụng config.dockerImageName
                    }
                    // ... và các stage khác ...
                }
            }
        }
        ```

3.  **Đăng ký Thư viện trong Jenkins:**

      * Bạn vào `Manage Jenkins` -\> `Configure System` -\> `Global Pipeline Libraries`.
      * Bạn "Add" (Thêm) thư viện mới, đặt tên là `company-library` và trỏ URL Git tới repo `my-jenkins-library`.

4.  **Sử dụng trong `Jenkinsfile` của dự án:**
    Bây giờ, `Jenkinsfile` của 50 dự án kia sẽ trở nên *siêu ngắn gọn*:

    ```groovy
    // Jenkinsfile của dự án 'user-service'

    // 1. "Import" thư viện dùng chung
    @Library('company-library') _

    // 2. "Gọi" hàm (tên file .groovy) và truyền tham số
    standardNodeBuild(
        nodeVersion: 18,
        sonarProjectKey: 'user-service-project',
        dockerImageName: 'my-registry/user-service'
    )
    ```

### ✅ Lợi ích

  * **Tái sử dụng (Reusable):** Viết code một lần, dùng cho 100 dự án.
  * **Tập trung (Centralized):** Khi sếp yêu cầu thêm stage "Quét bảo mật", bạn chỉ cần **sửa 1 file** (`standardNodeBuild.groovy`) trong thư viện. Tất cả 50 dự án sẽ tự động được cập nhật ở lần build tiếp theo.
  * **Dễ đọc (Readable):** `Jenkinsfile` của dự án chỉ còn vài dòng, rất sạch sẽ.

-----

## ⚙️ 2. JCasC (Jenkins Configuration as Code)

### 💡 Nó là gì?

**JCasC** (viết tắt của **Jenkins Configuration as Code**) là một plugin/triết lý cho phép bạn quản lý **toàn bộ cấu hình hệ thống của Jenkins** (những thứ trong `Manage Jenkins`) bằng các file **YAML**, thay vì click chuột.

  * **Pipeline as Code (PaC):** Quản lý *quy trình build* (cái job) bằng code.
  * **Configuration as Code (JCasC):** Quản lý *chính server Jenkins* (cái app) bằng code.

### ❓ Vấn đề nó giải quyết?

Tất cả mọi thứ bạn click trong `Manage Jenkins` (như: cài plugin, thêm Agent, cấu hình SonarQube URL, tạo Credential, đăng ký Shared Library) đều được lưu vào các file XML trên ổ cứng của server Jenkins.

**Vấn đề:**

1.  **Mong manh (Fragile):** Nếu server Jenkins "chết" hoặc ổ cứng hỏng, bạn **mất toàn bộ cấu hình**.
2.  **Không thể tái tạo (Not Reproducible):** Khi bạn cần 1 server Jenkins "staging" (thử nghiệm) giống hệt server "production" (chính thức), bạn phải *click chuột lại từ đầu* hàng trăm lần, rất dễ sai sót.
3.  **Không thể kiểm toán (Not Auditable):** Bạn không biết *ai* đã thay đổi cấu hình bảo mật, *khi nào*, và *tại sao*.

### ⚙️ Nó hoạt động như thế nào?

1.  **Cài Plugin:** Bạn cài plugin `Configuration as Code` cho Jenkins.
2.  **Viết file YAML:** Bạn tạo một file (ví dụ `jenkins.yaml`) để định nghĩa *toàn bộ* cấu hình Jenkins:
    ```yaml
    # jenkins.yaml
    jenkins:
      systemMessage: "Jenkins được quản lý bởi JCasC!"
      numExecutors: 0 # Cấm build trên Controller (Giai đoạn 4)

    # Tự động cài plugin
    plugins:
      - id: "workflow-aggregator" # Cài plugin Pipeline
      - id: "git"
      - id: "docker-workflow"
      - id: "sonar"

    # Cấu hình SonarQube (Giai đoạn 3)
    unclassified:
      sonarGlobalConfiguration:
        installations:
          - name: "MySonarQubeServer"
            serverUrl: "http://sonarqube.my-company.com"
            serverAuthenticationTokenId: "sonarqube-token" # ID của credential
            
    # Đăng ký Shared Library (Giai đoạn 5)
    globalLibraries:
      libraries:
        - name: "company-library"
          retriever:
            modernSCM:
              scm:
                git:
                  remote: "https://github.com/my-company/my-jenkins-library.git"
    ```
3.  **Trỏ Jenkins vào file YAML:** Khi khởi động Jenkins (thường là qua Docker/Kubernetes), bạn truyền một biến môi trường để bảo nó: `CASC_JENKINS_CONFIG=/đường/dẫn/tới/jenkins.yaml`.
4.  **Jenkins tự cấu hình:** Jenkins khởi động, đọc file YAML, và tự động cài plugin, cấu hình SonarQube, đăng ký Shared Library... mà bạn không cần click chuột.

### ✅ Lợi ích

  * **Tái tạo (Reproducible):** Bạn có thể tạo ra 10 server Jenkins giống hệt nhau trong 1 phút.
  * **Quản lý phiên bản (Version Control):** Bạn lưu file `jenkins.yaml` vào Git. Bạn có thể xem lịch sử, "review" (đánh giá) các thay đổi cấu hình Jenkins.
  * **Kiểm toán (Auditable):** Lịch sử Git cho bạn biết chính xác *ai* đã thêm plugin X, *khi nào*.
  * **Phục hồi thảm họa (Disaster Recovery):** Nếu server Jenkins nổ tung, bạn chỉ cần khởi động một container Jenkins mới, trỏ nó vào file `jenkins.yaml` trong Git, và toàn bộ cấu hình của bạn được phục hồi 100%.

**Tóm lại:** **Shared Library** giúp bạn quản lý *code pipeline*, trong khi **JCasC** giúp bạn quản lý *chính server Jenkins*. Cả hai đều là triết lý "Everything as Code" (Mọi thứ là Code) của DevOps hiện đại.