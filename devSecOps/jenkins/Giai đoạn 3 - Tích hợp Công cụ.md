## 🗺️ Giai đoạn 3 (Chi tiết): Tích hợp Công cụ & Quản lý Artifacts

### 🎯 Mục tiêu

Nâng cấp `Jenkinsfile` từ việc chỉ chạy test (Giai đoạn 2) sang một quy trình CI/CD hoàn chỉnh. Cụ thể:

1. **Phân tích Chất lượng Code (Static Analysis):** Tích hợp SonarQube để đảm bảo code "sạch" và an toàn.
    
2. **Đóng gói (Packaging):** Build ứng dụng thành một **Docker Image**.
    
3. **Lưu trữ "Thành phẩm" (Artifacts):** Đẩy (push) Docker image này lên một kho chứa riêng (Docker Registry) và lưu trữ các báo cáo (như báo cáo test, báo cáo Sonar).
    

---

### 3.1. Tích hợp Phân tích Code (SonarQube)

Đây là một bước _cực kỳ quan trọng_ trong doanh nghiệp. Bạn không chỉ muốn code chạy được, bạn muốn code _tốt_ (dễ bảo trì, ít bug, không có lỗ hổng bảo mật).

#### 🔑 Keywords: Static Analysis, SonarQube, Quality Gate

#### 1. Cài đặt (Phía Jenkins):

- **Cài đặt Plugin:** Vào `Manage Jenkins` -> `Manage Plugins` -> `Available` -> tìm và cài:
    
    - `SonarQube Scanner for Jenkins`
        
- **Cấu hình Plugin:** Vào `Manage Jenkins` -> `System Configuration` (hoặc `Configure System`):
    
    - Tìm mục **SonarQube servers**, nhấn **Add SonarQube**.
        
    - **Name:** `MySonarQubeServer` (Tên bạn tự đặt).
        
    - **Server URL:** `http://your-sonarqube-url:9000` (URL của server SonarQube bạn đã cài).
        
    - **Server authentication token:** Thêm token của SonarQube vào **Credentials Manager** (sẽ học kỹ ở Giai đoạn 5), sau đó chọn nó ở đây.
        
- **Cấu hình Tool:** Vào `Manage Jenkins` -> `Global Tool Configuration`:
    
    - Tìm **SonarQube Scanner**, nhấn **Add SonarQube Scanner**.
        
    - **Name:** `SonarScanner 5.0` (tên tự đặt).
        
    - Chọn **Install automatically** (Jenkins tự tải về).
        

#### 2. Cập nhật `Jenkinsfile`:

Bạn cần thêm 2 `stage` mới vào `Jenkinsfile`.

- **Stage 1: Chạy phân tích (Analysis):**
    
    - Sử dụng biến môi trường `withSonarQubeEnv('MySonarQubeServer')` (tên bạn đặt ở trên) để Jenkins tự động "inject" URL và token.
        
    - Gọi tool `SonarScanner 5.0` (tên bạn đặt ở trên) để chạy lệnh `sonar-scanner`.
        
- **Stage 2: Chờ Quality Gate (Quan trọng nhất):**
    
    - Sau khi phân tích, Jenkins sẽ "hỏi" SonarQube: "Code này có qua 'cổng chất lượng' không?".
        
    - **Quality Gate** là một bộ luật bạn định nghĩa trong SonarQube (ví dụ: "Độ che phủ test phải > 80%", "Không có bug nghiêm trọng").
        
    - Step `waitForQualityGate abortPipeline: true` sẽ khiến Jenkins _dừng và báo lỗi_ pipeline nếu Quality Gate thất bại.
        

---

### 3.2. Đóng gói ứng dụng (Docker)

Sau khi code đã "sạch" và "pass test", chúng ta cần đóng gói nó. Cách chuẩn nhất hiện nay là build một **Docker image**.

#### 🔑 Keywords: Docker, Dockerfile, Docker Registry

#### 1. Cài đặt (Phía Jenkins):

- **Cài đặt Plugin:** Vào `Manage Jenkins` -> `Manage Plugins` -> `Available` -> tìm và cài:
    
    - `Docker Pipeline` (hoặc `Docker` và `docker-workflow`).
        
- **Yêu cầu:** Agent chạy build (hoặc chính Jenkins Controller, nếu bạn đang chạy Docker-in-Docker) _phải_ cài đặt Docker CLI và Docker daemon phải đang chạy.
    

#### 2. Cập nhật `Jenkinsfile`:

Bạn sẽ sử dụng các `step` do plugin Docker cung cấp:

- `docker.build(...)`: Tương đương lệnh `docker build -t <tag> .`
    
- `docker.withRegistry(...)`: Tương đương `docker login` và `docker logout` một cách an toàn (sẽ dùng Giai đoạn 5).
    
- `myImage.push()`: Tương đương `docker push <tag>`.
    

Bạn cũng cần một file Dockerfile trong dự án của mình.

Ví dụ Dockerfile cho app Node.js:

Dockerfile

```
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build # Giả sử đây là app React/Vue

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/build ./build
COPY --from=builder /app/package*.json ./
RUN npm install --omit=dev # Chỉ cài production dependencies
EXPOSE 3000
CMD [ "node", "build/index.js" ] # Ví dụ
```

---

### 3.3. Quản lý Artifacts (Thành phẩm)

Artifacts không chỉ là file `.jar` hay `.zip`. Nó có thể là:

1. **Báo cáo (Reports):** Báo cáo test (`junit.xml`), báo cáo Sonar.
    
2. **Gói phần mềm (Packages):** File `.jar`, `.war`, `.tgz`.
    
3. **Images:** Docker Images.
    

Đối với Báo cáo/Gói phần mềm:

Bạn dùng step archiveArtifacts (như Giai đoạn 2) để Jenkins tạm thời lưu lại.

- **Best Practice:** Đẩy (push) các gói phần mềm (như `.jar`) lên một **Artifact Repository Manager** (kho quản lý) như **Sonatype Nexus** hoặc **JFrog Artifactory**. Jenkins không phải là nơi để lưu trữ file lâu dài.
    

Đối với Docker Images:

Bạn phải đẩy chúng lên một Docker Registry.

- **Registry công cộng:** Docker Hub.
    
- **Registry riêng tư (Enterprise):** Nexus, Artifactory, AWS ECR, Google GCR, Azure ACR.
    

---

### ⌨️ Ví dụ: `Jenkinsfile` (Hoàn chỉnh Giai đoạn 3)

Đây là `Jenkinsfile` kết hợp cả SonarQube và Docker, sử dụng các biến môi trường để quản lý tên.

Groovy

```
// Jenkinsfile (Giai đoạn 3: SonarQube + Docker)
pipeline {
    agent any // Sẽ tối ưu ở Giai đoạn 4

    // Định nghĩa các biến dùng chung
    environment {
        SONAR_PROJECT_KEY = 'my-node-app'
        SONAR_SERVER_NAME = 'MySonarQubeServer' // Tên cấu hình trong Manage Jenkins
        SONAR_SCANNER_NAME = 'SonarScanner 5.0' // Tên cấu hình trong Global Tools
        
        // Tên image (ví dụ: tài khoản-dockerhub/tên-app)
        // ${env.BUILD_NUMBER} là biến toàn cục của Jenkins (ví dụ: 1, 2, 3...)
        DOCKER_IMAGE_NAME = "vinh/my-node-app:${env.BUILD_NUMBER}"
    }

    stages {
        // --- Giai đoạn 2: Build & Test ---
        stage('Install & Test') {
            steps {
                echo 'Đang cài đặt dependencies và chạy test...'
                sh 'npm install'
                sh 'npm test'
            }
        }

        // --- GIAI ĐOẠN 3: TÍCH HỢP ---

        // Stage 3.1: Phân tích code với SonarQube
        stage('SonarQube Analysis') {
            steps {
                // 1. Chỉ định tool SonarScanner
                script {
                    // toolName: tên đã cấu hình trong Global Tool Configuration
                    def scannerHome = tool name: "${env.SONAR_SCANNER_NAME}"
                    
                    // 2. "Bọc" lệnh bằng withSonarQubeEnv để inject token/URL
                    // serverName: tên đã cấu hình trong Configure System
                    withSonarQubeEnv(serverName: "${env.SONAR_SERVER_NAME}") {
                        // 3. Chạy lệnh
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} -Dsonar.sources=."
                    }
                }
            }
        }

        // Stage 3.2: Chờ Quality Gate (BẮT BUỘC)
        stage('SonarQube Quality Gate') {
            steps {
                echo "Đang chờ SonarQube Quality Gate..."
                // Dừng chờ sau 5 phút
                timeout(time: 5, unit: 'MINUTES') {
                    // abortPipeline: true = Báo lỗi cả pipeline nếu Quality Gate thất bại
                    waitForQualityGate abortPipeline: true
                }
                echo "Quality Gate đã vượt qua!"
            }
        }

        // Stage 3.3: Build Docker Image
        stage('Build Docker Image') {
            steps {
                echo "Đang build Docker image: ${env.DOCKER_IMAGE_NAME}"
                
                // Dùng plugin 'Docker Pipeline'
                script {
                    // docker.build() trả về một đối tượng image
                    def myImage = docker.build(env.DOCKER_IMAGE_NAME, ".")
                    
                    // Chúng ta sẽ push ở stage sau
                }
            }
        }

        // Stage 3.4: Push Docker Image (Lưu trữ)
        stage('Push Docker Image') {
            steps {
                echo "Đang push Docker image: ${env.DOCKER_IMAGE_NAME}"
                
                // Tạm thời, chúng ta sẽ hard-code. Giai đoạn 5 sẽ sửa!
                // Đây là cách làm "rất tệ" (bad practice) vì lộ mật khẩu
                // sh "docker login -u my-docker-user -p my-super-secret-password"
                // sh "docker push ${env.DOCKER_IMAGE_NAME}"
                
                // Cách làm TỐT HƠN (sẽ học kỹ ở Giai đoạn 5)
                // 'docker-registry-credentials' là ID của secret đã lưu trong Jenkins
                docker.withRegistry('https://registry.hub.docker.com', 'docker-registry-credentials') {
                    // Tự động login, push, và logout
                    def myImage = docker.image(env.DOCKER_IMAGE_NAME)
                    myImage.push()
                }
            }
        }
    } // end stages

    post {
        always {
            cleanWs()
            
            // Ví dụ: Lưu lại báo cáo test (nếu npm test có xuất ra)
            // JUnit là plugin để đọc kết quả test và vẽ biểu đồ
            junit 'reports/junit-report.xml' 
        }
        success {
            echo 'Pipeline thành công! Sẵn sàng để deploy.'
        }
        failure {
            echo 'Pipeline thất bại!'
            // Gửi thông báo (sẽ học sau)
            // slackSend channel: '#dev-alerts', message: "Build thất bại: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

### 📖 Bảng Thuật Ngữ (Giai đoạn 3)

|**Thuật ngữ**|**Giải thích chi tiết**|
|---|---|
|**Static Analysis**|Phân tích mã nguồn mà _không cần thực thi_ nó. Dùng để tìm lỗi tiềm ẩn (bugs), lỗ hổng bảo mật (vulnerabilities), và code "bốc mùi" (code smells). SonarQube là công cụ phổ biến nhất.|
|**SonarQube**|Một nền tảng (platform) quản lý chất lượng code. Nó thu thập kết quả phân tích và hiển thị trên web, cho phép quản lý các "Quality Gate".|
|**Quality Gate**|Một bộ điều kiện (ví dụ: độ che phủ test > 80%, không có bug nghiêm trọng) mà code phải vượt qua. Nếu thất bại, build sẽ bị "failed".|
|**Artifact**|(Tạo tác) Sản phẩm đầu ra của quá trình build. Có thể là file `.jar`, `.war`, `.zip`, `dist` folder, hoặc Docker image.|
|**Artifact Repository**|(Kho lưu trữ Artifact) Hệ thống chuyên dụng để lưu trữ, quản lý phiên bản và phân phối các artifact (ví dụ: **Nexus**, **Artifactory**). Đây là "kho" nội bộ của công ty.|
|**Docker Registry**|Một kho lưu trữ _chuyên dụng_ cho Docker image (ví dụ: Docker Hub, AWS ECR, Google GCR, Nexus 3 cũng hỗ trợ).|
|**Dockerfile**|Một file text chứa các "chỉ dẫn" (instructions) để Docker build ra một image (ví dụ: `FROM node:18`, `COPY . .`, `RUN npm install`).|
