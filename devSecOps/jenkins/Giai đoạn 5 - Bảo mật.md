## 🗺️ Giai đoạn 5 (Chi tiết): Bảo mật, Shared Libraries & Vận hành (Enterprise Ops)

### 🎯 Mục tiêu

Đưa Jenkins lên mức độ "Enterprise-ready" (sẵn sàng cho doanh nghiệp) bằng cách:

1. **Bảo mật (Credentials):** **Không bao giờ** viết mật khẩu, API key, hay token vào `Jenkinsfile`. Học cách lưu trữ và sử dụng chúng một cách an toàn.
    
2. **Tái sử dụng (Shared Libraries):** Tránh lặp lại code. Viết code pipeline một lần và tái sử dụng cho 100 dự án khác nhau.
    
3. **Vận hành (JCasC):** Quản lý _cấu hình của chính Jenkins_ (plugins, agents, credentials) bằng code, thay vì click chuột.
    

---

### 5.1. Trụ cột 1: Bảo mật - Credentials Manager

Đây là quy tắc **bất di bất dịch**: **KHÔNG BAO GIỜ hard-code bí mật (secrets) vào `Jenkinsfile`**. `Jenkinsfile` nằm trong Git, ai cũng có thể đọc được nó.

Jenkins cung cấp một "két sắt" gọi là **Credentials Manager** để lưu trữ an toàn mọi thứ.

#### 🔑 Keywords: Credentials, `withCredentials`, Masking

#### 1. Thêm Credentials vào Jenkins:

- Vào `Manage Jenkins` -> `Security` -> `Manage Credentials`.
    
- Nhấn vào `(global)` (hoặc một domain khác).
    
- Nhấn `Add Credentials` (Thêm Credential).
    
- Bạn sẽ thấy các **Loại** (Kind) credential phổ biến:
    
    - **Username with password:** Dùng cho Docker Hub, Nexus, Sonatype, đăng nhập SSH...
        
        - **ID:** `docker-registry-credentials` (Đây là **tên gọi** bạn sẽ dùng trong pipeline. **Rất quan trọng!**).
            
        - **Username:** `vinh`
            
        - **Password:** `my-super-secret-password`
            
    - **Secret text:** Dùng cho API token (ví dụ: SonarQube token, Slack token).
        
        - **ID:** `sonarqube-token`
            
        - **Secret:** `sqp_xxxxxxxxxxxxxxxxxxxx`
            
    - **SSH Username with private key:** Dùng để Jenkins kết nối vào Agent (như Giai đoạn 4) hoặc deploy lên server.
        

#### 2. Sử dụng Credentials trong `Jenkinsfile`:

Bạn dùng step `withCredentials` để "bọc" đoạn code cần dùng bí mật. Jenkins sẽ "tiêm" (inject) bí mật đó vào dưới dạng **biến môi trường** (environment variable) _chỉ trong phạm vi của khối đó_.

Sau khi khối `withCredentials` kết thúc, các biến môi trường đó bị _xóa ngay lập tức_. Jenkins cũng sẽ tự động **che (mask)** các biến này trong log (hiển thị `****`).

#### ⌨️ Ví dụ: Sửa lại `stage` Push Docker (Giai đoạn 3 & 4)

**Cách làm TỆ (Bad Practice):**

Groovy

```
stage('Push Docker') {
    steps {
        sh 'docker login -u vinh -p my-super-secret-password' // LỘ MẬT KHẨU!
        sh 'docker push ...'
    }
}
```

Cách làm ĐÚNG (Best Practice) dùng withCredentials:

(Giả sử bạn đã tạo Credential loại "Username with password" với ID là docker-registry-credentials)

Groovy

```
stage('Push Docker Image') {
    agent { label 'linux && docker' }
    steps {
        script {
            def myImage = docker.build("vinh/my-node-app:${env.BUILD_NUMBER}", ".")

            // 1. Yêu cầu "mượn" credential có ID 'docker-registry-credentials'
            // 2. Jenkins sẽ cung cấp 2 biến: DOCKER_USER và DOCKER_PASS
            withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                            usernameVariable: 'DOCKER_USER', 
                                            passwordVariable: 'DOCKER_PASS')]) {
                
                // 3. Sử dụng các biến an toàn. 
                // Các biến này sẽ bị che trong log nếu bị in ra.
                echo "Đang đăng nhập với user: ${DOCKER_USER}" // OK
                
                // Dùng stdin để tránh lộ password trong 'ps' (process list)
                sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                
                // 4. Push image
                myImage.push()

                // 5. Đăng xuất
                sh "docker logout"

            } // 6. Biến DOCKER_USER và DOCKER_PASS bị hủy ở đây
        }
    }
}
```

> **Ghi chú:** Plugin `Docker Pipeline` (`docker.withRegistry`) đã làm việc này cho bạn một cách tự động. `docker.withRegistry('...', 'docker-registry-credentials')` là cách viết tắt của `withCredentials` + `docker login` + `docker push` + `docker logout`.

---

### 5.2. Trụ cột 2: Tái sử dụng - Shared Libraries

Vấn đề: Pipeline ở Giai đoạn 4 rất tốt, nhưng nó dài 80 dòng. Nếu bạn có 50 dự án Node.js, bạn sẽ copy-paste 80 dòng này vào 50 Jenkinsfile khác nhau?

Nếu sếp yêu cầu thêm 1 stage "Security Scan" thì sao? Bạn sẽ phải sửa 50 file!

**Shared Library** là giải pháp. Bạn định nghĩa các hàm (functions) pipeline trong một **repository Git riêng**, sau đó _gọi_ các hàm đó từ `Jenkinsfile` của dự án.

#### 🔑 Keywords: Shared Library, Groovy, `vars`

#### 1. Cấu trúc Repo Shared Library:

Bạn tạo một repo Git mới (ví dụ: `jenkins-shared-library`) với cấu trúc:

```
jenkins-shared-library/
|
+-- vars/  <-- Thư mục quan trọng nhất
|   |
|   +-- standardNodeBuild.groovy  <-- Tên file là tên hàm
|   +-- standardJavaBuild.groovy
|   +-- notifySlack.groovy
|
+-- resources/
    |
    +-- config.json
```

**Nội dung file `vars/standardNodeBuild.groovy`:**

Groovy

```
// vars/standardNodeBuild.groovy

// Tên file 'standardNodeBuild' trở thành tên hàm
// 'call()' là hàm đặc biệt sẽ được thực thi khi bạn gọi tên đó

def call(Map config = [:]) {
    // config là một map để truyền tham số, ví dụ:
    // config.sonarProjectKey
    
    // BẠN COPY-PASTE TOÀN BỘ PIPELINE GIAI ĐOẠN 4 VÀO ĐÂY
    pipeline {
        agent none 

        stages {
            
            stage('Build & Test') {
                agent { docker { image "node:${config.nodeVersion ?: 18}-alpine" } } // Thêm tham số
                steps {
                    echo "Đang chạy npm install & test"
                    sh 'npm install'
                    sh 'npm test'
                    stash name: 'build-output', includes: 'dist/'
                }
            }

            stage('SonarQube Analysis') {
                // ... (giống hệt Giai đoạn 4) ...
                // Sử dụng config.sonarProjectKey
            }
            
            stage('SonarQube Quality Gate') {
                // ... (giống hệt Giai đoạn 4) ...
            }

            stage('Build & Push Docker Image') {
                // ... (giống hệt Giai đoạn 4) ...
                // Sử dụng config.dockerImageName
            }
        } // end stages
        
        post {
            // ... (post block) ...
        }
    }
}
```

#### 2. Cấu hình Jenkins:

- Vào `Manage Jenkins` -> `System Configuration` (Configure System).
    
- Cuộn xuống mục **Global Pipeline Libraries**.
    
- Nhấn `Add`.
    
- **Name:** `my-company-library` (Tên thư viện).
    
- **Default version:** `main` (hoặc `master`).
    
- **Retrieval Method:** Chọn `Modern SCM`.
    
- **Source Code Management:** Chọn `Git`, điền URL của repo `jenkins-shared-library`.
    
- `Save`.
    

#### 3. Sử dụng trong `Jenkinsfile` của dự án:

Bây giờ, `Jenkinsfile` của 50 dự án Node.js của bạn sẽ trở nên _siêu ngắn gọn_:

Groovy

```
// Jenkinsfile (của dự án my-node-app)

// 1. Import thư viện (tên đã đặt trong Manage Jenkins)
@Library('my-company-library') _

// 2. Gọi hàm (tên file .groovy) và truyền tham số
standardNodeBuild(
    nodeVersion: 18,
    sonarProjectKey: 'my-node-app-key',
    dockerImageName: "vinh/my-node-app"
)
```

**Kết quả:**

- `Jenkinsfile` của dự án giờ chỉ còn vài dòng, siêu dễ đọc.
    
- Nếu sếp muốn thêm 1 stage "Security Scan", bạn chỉ cần **sửa file `standardNodeBuild.groovy`** trong Shared Library. Ngay lập tức, _cả 50 dự án_ sẽ tự động có stage mới ở lần build tiếp theo.
    

---

### 5.3. Trụ cột 3: Vận hành - Jenkins Configuration as Code (JCasC)

Phần này là "siêu nâng cao" (ops-level).

- **Pipeline as Code (PaC):** Quản lý _quy trình build_ bằng code.
    
- **Configuration as Code (JCasC):** Quản lý _cấu hình của Jenkins_ (plugin, agent, security, thư viện shared...) bằng code.
    

Bạn dùng plugin **Configuration as Code**. Nó cho phép bạn định nghĩa _toàn bộ_ cấu hình trong `Manage Jenkins` bằng một (hoặc nhiều) file **YAML**.

#### 🔑 Keywords: JCasC, YAML, Configuration as Code

#### ⌨️ Ví dụ: File `jenkins.yaml` (JCasC)

YAML

```
# jenkins.yaml
jenkins:
  # Cấu hình hệ thống (giống trong /configure)
  systemMessage: "Welcome to our Enterprise Jenkins! (Managed by JCasC)"
  numExecutors: 0 # Cấm chạy build trên Controller! (Giai đoạn 4)

  # Cấu hình bảo mật
  security:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD_ENV_VAR}" # Lấy pass từ biến môi trường
        - id: "vinh"
          password: "${VINH_PASSWORD_ENV_VAR}"
  
  # Cấu hình Global Tool (giống trong /global-tool-configuration)
  tools:
    sonarScanner:
      - name: "SonarScanner 5.0"
        installations:
          - install:
              id: "5.0.1.3006" # Tự động cài

  # Cấu hình Global Pipeline Library (giống 5.2)
  globalLibraries:
    libraries:
      - name: "my-company-library"
        retriever:
          modernSCM:
            scm:
              git:
                remote: "https://github.com/my-company/jenkins-shared-library.git"
                credentialsId: "github-ssh-key" # Dùng credential đã lưu

# Cấu hình plugin (ví dụ: SonarQube)
unclassified:
  sonarGlobalConfiguration:
    installations:
      - name: "MySonarQubeServer"
        serverUrl: "http://sonarqube.my-company.com"
        serverAuthenticationTokenId: "sonarqube-token" # Dùng credential đã lưu
```

**Cách dùng:** Bạn trỏ Jenkins vào file YAML này (qua biến môi trường). Khi Jenkins khởi động, nó đọc file này và tự cấu hình. Bạn có thể lưu file này vào Git, thực hiện code review cho _thay đổi cấu hình Jenkins_.
