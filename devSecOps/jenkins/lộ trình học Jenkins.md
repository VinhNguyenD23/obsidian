## 🗺️ Giai đoạn 1: Nền tảng & Cài đặt Jenkins

### 🎯 Mục tiêu

Hiểu Jenkins là gì, cài đặt Jenkins Controller (Master) một cách an toàn, và làm quen với giao diện người dùng (UI) cũng như khái niệm "Job" cơ bản nhất (Freestyle Project).

### 💡 Giải thích chi tiết

Đây là bước "Hello, World\!" với Jenkins. Bạn sẽ cài đặt Jenkins, thường là qua Docker (cách nhanh nhất và phổ biến trong doanh nghiệp để thử nghiệm) hoặc cài đặt trực tiếp trên một máy chủ (ví dụ: Ubuntu/CentOS).

Sau khi cài đặt, bạn sẽ truy cập giao diện web, thực hiện "Post-installation setup wizard" (cài đặt plugin khuyến nghị) và tạo admin user.

Chúng ta sẽ bắt đầu với **Freestyle Project**. Đây là cách tạo job "click-ops" (dùng UI để cấu hình), giúp bạn hiểu các khối xây dựng cơ bản:

1.  **Lấy code (SCM):** Kết nối tới Git.
2.  **Chạy build (Build Steps):** Thực thi các lệnh (ví dụ: `npm install`).
3.  **Hành động sau build (Post-build Actions):** Gửi email, lưu trữ file.

Mặc dù Freestyle dễ bắt đầu, doanh nghiệp hiện đại **không** dùng nó cho các dự án chính, mà dùng Pipeline (Giai đoạn 2). Tuy nhiên, hiểu Freestyle giúp bạn hiểu các khái niệm gốc.

### 🔑 Keywords

  * Jenkins Controller (tên cũ là Master)
  * Agent (tên cũ là Slave/Node)
  * Plugin
  * Job / Project
  * Build
  * Freestyle Project
  * SCM (Source Control Management)

### ⌨️ Ví dụ (Freestyle Project)

Bạn không "code" mà là cấu hình trên UI:

1.  Tạo "New Item" -\> chọn "Freestyle project".
2.  Trong tab **Source Code Management**, chọn **Git**, điền URL repository (ví dụ: `https://github.com/your-user/my-app.git`).
3.  Trong tab **Build Steps**, chọn **Execute shell**.
4.  Nhập vào ô lệnh:

<!-- end list -->

```bash
echo "Bắt đầu build..."
npm install
npm test
echo "Build hoàn tất."
```

5.  Lưu và nhấn "Build Now". Bạn sẽ thấy một build được thực thi ở "Build History".

### 📖 Bảng Thuật Ngữ (Giai đoạn 1)

| Thuật ngữ              | Giải thích chi tiết                                                                                               |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------- |
| **Jenkins Controller** | Máy chủ trung tâm điều phối mọi hoạt động. Nó lưu trữ cấu hình, lên lịch build, và hiển thị UI.                   |
| **Agent**              | (Hay Node) Máy (vật lý hoặc ảo) thực thi các lệnh build. Controller chỉ điều phối, Agent mới là "người làm việc". |
| **Plugin**             | Các gói mở rộng chức năng cho Jenkins. Hầu hết mọi tính năng (Git, Docker, SonarQube...) đều qua plugin.          |
| **Job / Project**      | Một "công việc" được cấu hình để Jenkins thực thi, ví dụ: build một ứng dụng.                                     |
| **Build**              | Một lần thực thi cụ thể của một Job. Mỗi lần bạn nhấn "Build Now", Jenkins tạo ra một build mới với ID riêng.     |
| **Freestyle Project**  | Loại job cơ bản nhất, cấu hình hoàn toàn bằng giao diện web (click-ops).                                          |

-----

## 🗺️ Giai đoạn 2: Pipeline as Code (Declarative Pipeline)

### 🎯 Mục tiêu

Chuyển từ "click-ops" (Freestyle) sang **Pipeline as Code (PaC)**. Đây là yêu cầu *bắt buộc* trong doanh nghiệp. Bạn sẽ học cách định nghĩa toàn bộ quy trình CI/CD bằng một file tên là **Jenkinsfile** và lưu nó chung với source code.

### 💡 Giải thích chi tiết

Thay vì click trên UI, bạn định nghĩa quy trình build trong một file `Jenkinsfile`.

**Tại sao (Why)?**

  * **Source Control:** Quy trình build được quản lý phiên bản chung với code (ai thay đổi, thay đổi gì, khi nào).
  * **Tái sử dụng (Reusable):** Dễ dàng sao chép, chia sẻ pipeline cho các dự án khác.
  * **Bền bỉ (Durable):** Nếu máy chủ Jenkins "chết", toàn bộ cấu hình pipeline vẫn nằm trong Git, chỉ cần trỏ Jenkins mới vào Git là pipeline chạy lại.

Bạn sẽ tập trung vào **Declarative Pipeline**. Đây là cú pháp mới, có cấu trúc rõ ràng và dễ học hơn (so với Scripted Pipeline cũ).

Một Declarative Pipeline cơ bản có cấu trúc:
`pipeline` -\> `agent` -\> `stages` -\> `stage` -\> `steps`.

### 🔑 Keywords

  * Pipeline as Code (PaC)
  * Jenkinsfile
  * Declarative Pipeline (ưu tiên học)
  * Scripted Pipeline (biết để đọc code cũ)
  * `pipeline` (block)
  * `agent` (chỉ định nơi chạy)
  * `stages` (tập hợp các giai đoạn)
  * `stage` (một giai đoạn cụ thể, ví dụ: "Build", "Test")
  * `steps` (các lệnh thực thi trong một stage)
  * `post` (hành động sau khi build xong: luôn luôn, thành công, thất bại)

### ⌨️ Ví dụ (Jenkinsfile)

Đây là file `Jenkinsfile` bạn đặt ở gốc (root) của dự án (ví dụ: dự án Node.js).

```groovy
// Jenkinsfile (Cú pháp Declarative)
pipeline {
    // 1. Chỉ định nơi chạy build
    // 'any' nghĩa là chạy trên bất kỳ agent nào có sẵn
    agent any 

    // 2. Định nghĩa các giai đoạn
    stages {
        // Giai đoạn 1: Cài đặt dependencies
        stage('Install Dependencies') {
            steps {
                // 'sh' là viết tắt của "shell script"
                echo 'Đang cài đặt node modules...'
                sh 'npm install'
            }
        }

        // Giai đoạn 2: Chạy Unit Test
        stage('Unit Test') {
            steps {
                echo 'Đang chạy unit test...'
                sh 'npm test'
            }
        }

        // Giai đoạn 3: Build (ví dụ: build app React/Vue)
        stage('Build') {
            steps {
                echo 'Đang build ứng dụng...'
                sh 'npm run build'
            }
        }
    }

    // 3. Hành động sau khi build (luôn chạy)
    post {
        always {
            echo 'Pipeline đã chạy xong.'
            // Dọn dẹp workspace
            cleanWs() 
        }
        success {
            echo 'Build thành công!'
        }
        failure {
            echo 'Build thất bại!'
            // Gửi email thông báo (cần plugin)
            // mail to: 'dev-team@example.com', subject: "Build thất bại: ${env.JOB_NAME}"
        }
    }
}
```

### 📖 Bảng Thuật Ngữ (Giai đoạn 2)

| Thuật ngữ | Giải thích chi tiết |
| :--- | :--- |
| **Pipeline as Code (PaC)** | Triết lý lưu trữ và quản lý cấu hình pipeline (quy trình build/deploy) dưới dạng code (ví dụ: `Jenkinsfile`) thay vì cấu hình trên UI. |
| **Jenkinsfile** | Tên file mặc định (viết hoa chữ J) chứa định nghĩa pipeline. Jenkins tự động đọc file này từ SCM. |
| **Declarative Pipeline** | Cú pháp PaC hiện đại, có cấu trúc rõ ràng (pipeline, agent, stages, steps). Dễ viết và dễ đọc. |
| **`agent`** | Chỉ thị (directive) xác định môi trường thực thi cho toàn bộ pipeline hoặc một `stage` cụ thể. |
| **`stage`** | Đại diện cho một giai đoạn logic riêng biệt trong pipeline (ví dụ: Build, Test, Deploy). Các `stage` được trực quan hóa trên UI của Jenkins. |
| **`steps`** | Nơi chứa các lệnh thực thi thực tế (ví dụ: `sh 'npm install'`, `docker build .`). |
| **`post`** | Block tùy chọn, định nghĩa các hành động *sau khi* pipeline hoàn tất, dựa trên kết quả (success, failure, always, unstable...). |

-----

## 🗺️ Giai đoạn 3: Tích hợp Công cụ & Quản lý Artifacts

### 🎯 Mục tiêu

Làm cho pipeline "chuẩn doanh nghiệp" bằng cách tích hợp các công cụ thiết yếu:

1.  **Phân tích code (SonarQube):** Đảm bảo chất lượng code.
2.  **Build Docker Image:** "Đóng gói" ứng dụng.
3.  **Lưu trữ Artifacts (Nexus/Artifactory):** Lưu trữ "thành phẩm" (ví dụ: file `.jar`, `.war`, image Docker) một cách an toàn.

### 💡 Giải thích chi tiết

Pipeline của bạn không chỉ để chạy test, mà còn để tạo ra sản phẩm.

**1. Phân tích code (Static Analysis):**
Bạn sẽ thêm một `stage` "Code Analysis". Stage này sẽ gọi `sonar-scanner` (công cụ của SonarQube) để phân tích code. Jenkins sẽ chờ kết quả từ SonarQube. Bạn có thể cấu hình "Quality Gate" (cổng chất lượng) – nếu code không đạt chuẩn (ví dụ: \>10 bug, độ che phủ \<80%), SonarQube sẽ báo thất bại, và Jenkins sẽ *dừng* pipeline lại.

**2. Build & Push Docker Image:**
Thay vì chỉ chạy `npm build`, bạn sẽ thêm `stage` để build Docker image (`docker build -t my-app:latest .`) và đẩy (push) image đó lên một Docker Registry (ví dụ: Docker Hub, hoặc registry nội bộ của công ty).

**3. Quản lý Artifacts:**

  * **Artifacts cơ bản:** Là các file được tạo ra từ quá trình build (ví dụ: `dist` folder, file `.jar`). Bạn dùng `archiveArtifacts` để Jenkins lưu lại.
  * **Artifacts chuyên nghiệp:** Trong doanh nghiệp, bạn không lưu artifact trên Jenkins, mà đẩy chúng lên một **Artifact Repository Manager** (như Sonatype Nexus hoặc JFrog Artifactory). Đây là "kho" chứa các phiên bản phần mềm của công ty.

### 🔑 Keywords

  * Static Code Analysis
  * SonarQube / SonarCloud
  * Quality Gate
  * Docker Plugin (docker-pipeline)
  * Docker Registry
  * Artifact
  * `archiveArtifacts` (step)
  * Artifact Repository Manager
  * Sonatype Nexus / JFrog Artifactory

### ⌨️ Ví dụ (Jenkinsfile với SonarQube & Docker)

```groovy
// Jenkinsfile (Thêm Sonar & Docker)
pipeline {
    agent any

    // Định nghĩa các biến môi trường
    environment {
        // URL của SonarQube Server (cấu hình trong Jenkins settings)
        SONAR_SERVER = 'http://sonarqube.my-company.com' 
        // Tên project trên SonarQube
        SONAR_PROJECT_KEY = 'my-node-app'
        // Tên Docker image
        DOCKER_IMAGE_NAME = 'my-registry/my-node-app'
    }

    stages {
        // ... Các stage 'Install' và 'Test' như Giai đoạn 2 ...

        // Giai đoạn 3: Phân tích code
        stage('Code Analysis') {
            steps {
                echo "Đang chạy SonarQube analysis..."
                // Giả sử bạn đã cài đặt SonarScanner
                // và cấu hình SonarQube server trong 'Manage Jenkins'
                // 'withSonarQubeEnv' sẽ tự động cung cấp biến SONAR_HOST_URL và SONAR_AUTH_TOKEN
                withSonarQubeEnv('MySonarQubeServer') { 
                    sh "sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.sources=."
                }
            }
        }

        // Giai đoạn 4: Chờ Quality Gate (Cực kỳ quan trọng)
        stage('Quality Gate') {
            steps {
                echo "Đang chờ kết quả Quality Gate từ SonarQube..."
                // Dừng pipeline nếu Quality Gate thất bại
                // timeout: Dừng chờ sau 10 phút
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // Giai đoạn 5: Build & Push Docker Image
        stage('Build and Push Image') {
            steps {
                echo "Đang build Docker image: ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                
                // Dùng plugin 'Docker Pipeline'
                script {
                    // Xây dựng image
                    def myImage = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}", ".")

                    // Đăng nhập vào registry (Credentials sẽ học ở Giai đoạn 5)
                    // 'docker-registry-credentials' là ID của credential đã lưu trong Jenkins
                    docker.withRegistry('https://my-registry', 'docker-registry-credentials') {
                        // Push image
                        myImage.push()
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

### 📖 Bảng Thuật Ngữ (Giai đoạn 3)

| Thuật ngữ | Giải thích chi tiết |
| :--- | :--- |
| **Static Analysis** | Phân tích mã nguồn mà không cần thực thi nó. Dùng để tìm lỗi tiềm ẩn, lỗ hổng bảo mật, và code "bốc mùi" (code smells). |
| **Quality Gate** | Một bộ điều kiện (ví dụ: độ che phủ test \> 80%, không có bug nghiêm trọng) mà code phải vượt qua. Nếu thất bại, build sẽ bị "failed". |
| **Artifact** | (Tạo tác) Sản phẩm đầu ra của quá trình build. Có thể là file `.jar`, `.war`, `.zip`, `dist` folder, hoặc Docker image. |
| **Artifact Repository** | (Kho lưu trữ Artifact) Hệ thống chuyên dụng để lưu trữ, quản lý phiên bản và phân phối các artifact (ví dụ: Nexus, Artifactory). |
| **Docker Registry** | Một kho lưu trữ dành riêng cho Docker image (ví dụ: Docker Hub, Google GCR, Amazon ECR). |

-----

## 🗺️ Giai đoạn 4: Quản lý Agent & Môi trường Build

### 🎯 Mục tiêu

Ngừng chạy build trên Controller (máy chủ Jenkins) và học cách sử dụng **Agents** (các máy thực thi) để:

1.  **Scale (Mở rộng):** Chạy nhiều build song song.
2.  **Isolation (Cách ly):** Chạy các build trong môi trường sạch, chuyên biệt (ví dụ: build app Java cần JDK 11, app Node cần Node.js 18).

### 💡 Giải thích chi tiết

Trong môi trường doanh nghiệp, **Jenkins Controller KHÔNG BAO GIỜ được dùng để chạy build**. Controller chỉ làm nhiệm vụ điều phối. Mọi công việc build phải được thực thi trên **Agents**.

Bạn sẽ học 2 loại Agent:

1.  **Static Agents:** Các máy chủ (VM, bare metal) được cài đặt sẵn và kết nối vĩnh viễn với Controller. Chúng được gán **Label** (nhãn), ví dụ: `linux`, `windows`, `java11`.
2.  **Dynamic Agents:** (Nâng cao/Phổ biến) Agents được tạo ra *theo yêu cầu* khi pipeline cần, và bị *hủy* khi build xong. Cách phổ biến nhất là dùng **Docker (agent as container)** hoặc **Kubernetes**.

Sử dụng dynamic agent (ví dụ `agent { docker { ... } }`) là best practice:

  * **Môi trường sạch (Clean Environment):** Mỗi build bắt đầu với một container "sạch", không bị ảnh hưởng bởi rác từ các build trước.
  * **Tài nguyên hiệu quả (Efficient):** Không cần duy trì các máy chủ "chờ" việc.
  * **Chuyên biệt (Specialized):** Bạn có thể dùng đúng image (ví dụ: `node:18-alpine`) cho đúng dự án.

Bạn sẽ học cách sử dụng `agent { label 'label-name' }` để chọn static agent, và `agent { docker { image 'image-name' } }` để chạy build bên trong một Docker container.

### 🔑 Keywords

  * Agent / Node / Executor
  * Static Agent
  * Dynamic Agent
  * Label
  * `agent { label '...' }`
  * `agent { docker { ... } }`
  * `agent { kubernetes { ... } }` (nâng cao)
  * Executor

### ⌨️ Ví dụ (Jenkinsfile sử dụng Dynamic Agent - Docker)

Đây là cách "chuẩn" để chạy build Node.js. Jenkins sẽ tự động pull image `node:18-alpine`, khởi động container, chạy các lệnh *bên trong* container đó, và cuối cùng là hủy container.

```groovy
// Jenkinsfile (Sử dụng Dynamic Docker Agent)
pipeline {
    // 1. Chỉ định agent là một Docker container
    // Jenkins sẽ tự động pull image này và chạy các step bên trong nó.
    agent {
        docker { 
            image 'node:18-alpine' 
            // args '-v /tmp:/tmp' // Có thể mount volume nếu cần
        }
    }

    stages {
        stage('Verify Environment') {
            steps {
                echo 'Kiểm tra môi trường bên trong container:'
                sh 'node -v'  // Sẽ in ra v18.x.x
                sh 'npm -v'   // Sẽ in ra phiên bản npm tương ứng
            }
        }

        stage('Install & Test') {
            steps {
                echo 'Chạy install và test bên trong container Node 18'
                sh 'npm install'
                sh 'npm test'
            }
        }
    }
    
    post {
        always {
            // cleanWs() vẫn cần thiết để dọn dẹp workspace trên agent 
            // (có thể là agent vật lý host cái Docker daemon)
            cleanWs() 
        }
    }
}
```

### 📖 Bảng Thuật Ngữ (Giai đoạn 4)

| Thuật ngữ | Giải thích chi tiết |
| :--- | :--- |
| **Agent (Node)** | Máy thực thi các build step do Controller chỉ định. |
| **Executor** | Một "luồng" (thread) thực thi trên một Agent. Nếu một Agent có 2 executor, nó có thể chạy 2 build (hoặc 2 stage song song) cùng lúc. |
| **Label** | Một "nhãn" (tag) dùng để nhóm các Agent (ví dụ: `linux`, `gpu`, `performance-test`). Pipeline dùng label để yêu cầu một loại agent cụ thể. |
| **Static Agent** | Agent được cài đặt cố định và luôn kết nối với Controller. Dùng cho các tác vụ đặc thù cần phần cứng riêng. |
| **Dynamic Agent** | Agent được tạo ra "theo yêu" cầu (on-demand) từ một "cloud" (ví dụ: Docker, Kubernetes, AWS EC2) khi build bắt đầu và bị hủy khi build kết thúc. |

-----

## 🗺️ Giai đoạn 5: Bảo mật, Shared Libraries & Vận hành (Enterprise Ops)

### 🎯 Mục tiêu

Đưa Jenkins lên mức độ "Enterprise-ready" bằng cách tập trung vào 3 trụ cột:

1.  **Bảo mật (Security):** Quản lý secrets (mật khẩu, API key) và phân quyền (ai được làm gì).
2.  **Tái sử dụng (Reusability):** Tránh lặp code trong `Jenkinsfile` bằng **Shared Libraries**.
3.  **Quản lý cấu hình (Configuration):** Quản lý *cấu hình của chính Jenkins* bằng code (JCasC).

### 💡 Giải thích chi tiết

**1. Bảo mật - Credentials Manager:**
**TUYỆT ĐỐI KHÔNG** hard-code mật khẩu, token, SSH key vào `Jenkinsfile`. Jenkins cung cấp **Credentials Manager** để lưu trữ các thông tin nhạy cảm này một cách an toàn.

Trong pipeline, bạn sẽ dùng `withCredentials` để "inject" secret vào môi trường (env) hoặc file một cách an toàn *chỉ* trong lúc step đó chạy. Secret sẽ được che (masked) trong log.

**2. Tái sử dụng - Shared Libraries:**
Khi bạn có 100 dự án Node.js, chúng sẽ có `Jenkinsfile` gần giống hệt nhau (install, test, sonar, docker build). Nếu cần thay đổi quy trình (ví dụ: thêm 1 bước bảo mật), bạn phải sửa 100 file.

**Shared Libraries** giải quyết vấn đề này. Bạn viết các hàm Groovy (ví dụ: `buildNodeApp()`, `pushToNexus()`) trong một repository Git riêng, sau đó "import" thư viện này vào `Jenkinsfile` của bạn.

`Jenkinsfile` của dự án lúc này siêu ngắn gọn:

```groovy
@Library('my-shared-library') _ // Import thư viện

pipeline {
    agent any
    stages {
        stage('Build and Deploy') {
            steps {
                // Gọi 1 hàm duy nhất từ shared library
                // Hàm này chứa logic của 5-6 stage bên trong nó
                standardNodeBuildDeploy() 
            }
        }
    }
}
```

**3. Vận hành - Jenkins Configuration as Code (JCasC):**
PaC (Giai đoạn 2) là quản lý *pipeline* bằng code. JCasC là quản lý *cấu hình của Jenkins* (plugins, agents, credentials, security settings...) bằng các file YAML. Điều này giúp bạn:

  * Tái tạo lại server Jenkins y hệt trong vài phút.
  * Quản lý thay đổi cấu hình Jenkins qua Git (code review).

### 🔑 Keywords

  * Credentials Manager
  * `withCredentials` (step)
  * Masking (che log)
  * Shared Library
  * Groovy
  * Jenkins Configuration as Code (JCasC)
  * Role-Based Authorization Strategy (RBAC) (Plugin)
  * Matrix Security

### ⌨️ Ví dụ (Jenkinsfile dùng Credentials & Shared Library)

**1. Ví dụ `withCredentials` (Ẩn mật khẩu Docker):**

```groovy
// ... (các stage khác) ...
        stage('Build and Push Image') {
            steps {
                script {
                    def myImage = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}", ".")

                    // 'docker-registry-credentials' là ID của Secret Text hoặc User/Pass
                    // đã lưu trong Jenkins Credentials Manager.
                    // Jenkins sẽ inject biến DOCKER_USER và DOCKER_PASS
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                                    usernameVariable: 'DOCKER_USER', 
                                                    passwordVariable: 'DOCKER_PASS')]) {
                        
                        // Đăng nhập an toàn. Các biến này chỉ tồn tại trong block này
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin https://my-registry"
                        
                        myImage.push()

                        // Xóa thông tin đăng nhập sau khi push
                        sh "docker logout https://my-registry"
                    }
                }
            }
        }
// ...
```

**2. Ví dụ JCasC (file `jenkins.yaml`):**

```yaml
# Đây là file cấu hình JCasC (Jenkins Configuration as Code)
jenkins:
  systemMessage: "Welcome to our Enterprise Jenkins! Managed by JCasC."
  
  # Cài đặt security realm (ví dụ: dùng user/pass của Jenkins)
  security:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD_ENV_VAR}" # Lấy pass từ biến môi trường
        - id: "developer"
          password: "${DEV_PASSWORD_ENV_VAR}"

  # Cấu hình an ninh (phân quyền)
  authorization:
    matrix:
      permissions:
        - "Overall/Read:developer" # Developer chỉ được quyền đọc
        - "Job/Read:developer"
        - "Overall/Administer:admin" # Admin có full quyền

# Cấu hình plugin SonarQube
unclassified:
  sonarGlobalConfiguration:
    installations:
      - name: "MySonarQubeServer"
        serverUrl: "http://sonarqube.my-company.com"
        # Token của SonarQube được lấy từ Credentials Manager
        serverAuthenticationTokenId: "sonarqube-token"
```

### 📖 Bảng Thuật Ngữ (Giai đoạn 5)

| Thuật ngữ               | Giải thích chi tiết                                                                                                                                                                           |
| :---------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Credentials Manager** | Nơi lưu trữ tập trung và an toàn các thông tin nhạy cảm (mật khẩu, API key, file certificate) trong Jenkins.                                                                                  |
| **`withCredentials`**   | Một step trong pipeline, cho phép "mượn" một credential đã lưu và inject nó vào build một cách an toàn (ẩn trong log).                                                                        |
| **Shared Library**      | Một repository Git chứa các đoạn code Groovy (hàm) có thể tái sử dụng, dùng để chuẩn hóa và đơn giản hóa các file `Jenkinsfile` trong nhiều dự án.                                            |
| **JCasC**               | (Jenkins Configuration as Code) Triết lý quản lý *cấu hình hệ thống* của Jenkins (plugin, bảo mật, agent...) bằng các file YAML, thay vì click trên UI.                                       |
| **RBAC**                | (Role-Based Authorization Strategy) Một plugin phổ biến cho phép tạo ra các "Vai trò" (ví dụ: Developer, QA, DevOps) và gán quyền chi tiết cho từng vai trò, thay vì gán quyền cho từng user. |
