## 🗺️ Giai đoạn 4 (Chi tiết): Quản lý Agent & Môi trường Build

### 🎯 Mục tiêu

Ngừng chạy build trên Controller (máy chủ Jenkins). Bạn sẽ học cách "phân công" công việc cho các **Agents** (còn gọi là Nodes, hay "máy thợ"). Điều này là _bắt buộc_ trong doanh nghiệp để:

1. **Bảo mật & Ổn định (Security & Stability):** Không làm sập Controller. Nếu build thất bại (ví dụ: chạy hết 100% CPU, đầy ổ cứng), nó chỉ làm sập Agent, Controller vẫn sống khỏe để điều phối các job khác.
    
2. **Mở rộng (Scalability):** Chạy 100 build cùng lúc trên 100 Agent khác nhau. Controller không thể làm điều này.
    
3. **Môi trường chuyên biệt (Specialized Environments):** Controller không thể cài _mọi thứ_ (Java 8, Java 11, Node 16, Node 18, Python...). Thay vào đó, bạn sẽ có các Agent chuyên biệt: Agent `java-11`, Agent `node-18`, Agent `windows-build`.
    

---

### 4.1. Jenkins Controller vs. Agent

Hãy tưởng tượng Jenkins là một nhà hàng:

- **Jenkins Controller (Master):** Là người quản lý/bếp trưởng. Nhận đơn hàng (lịch build, user nhấn "Build Now"), xem đơn hàng cần gì (đọc `Jenkinsfile`), nhưng _không tự mình_ nấu.
    
- **Agents (Nodes/Slaves):** Là các đầu bếp (máy thợ). Mỗi đầu bếp có một "nhãn" (label) về kỹ năng của mình (ví dụ: `chuyên-đồ-á`, `chuyên-đồ-âu`).
    
- **Executors (Luồng thực thi):** Là số tay/số chảo mà một đầu bếp có. Nếu Agent có 2 Executor, nó có thể "nấu" 2 món (chạy 2 build) cùng lúc.
    

**NGUYÊN TẮC VÀNG:** Luôn cấu hình Controller có **0 executors**. Điều này _cấm_ nó chạy bất kỳ build nào và chỉ cho phép nó làm nhiệm vụ điều phối.

- **Cách làm:** `Manage Jenkins` -> `Manage Nodes and Clouds` -> chọn node `master` (hoặc `Built-In Node`) -> `Configure` -> `Number of executors`: **0** -> `Save`.
    

---

### 4.2. Loại 1: Static Agents (Agent "Cứng")

Đây là các máy chủ (vật lý hoặc VM) được cài đặt cố định, kết nối 24/7 với Jenkins Controller. Bạn phải tự tay cài đặt các công cụ (Java, Node.js, Docker...) lên các máy này.

#### 🔑 Keywords: Static Agent, Label, SSH Launcher

#### 1. Cách tạo (Ví dụ: Thêm một Agent Linux qua SSH):

1. **Chuẩn bị Agent:** Bạn cần một máy chủ Linux (ví dụ: một VM Ubuntu) có cài sẵn `openjdk-11-jre` (để chạy agent của Jenkins) và các công cụ build (ví dụ: `nodejs`, `npm`, `docker`).
    
2. **Tạo SSH Key:** Trên Controller, tạo một cặp SSH key. Bạn sẽ copy public key vào file `~/.ssh/authorized_keys` trên máy Agent, và lưu private key vào **Credentials Manager** (Giai đoạn 5) trên Jenkins.
    
3. **Thêm Node trên Jenkins:**
    
    - `Manage Jenkins` -> `Manage Nodes and Clouds` -> `New Node`.
        
    - **Node Name:** `my-linux-agent-01`
        
    - Chọn **Permanent Agent** (Agent cố định) -> OK.
        
    - **Remote root directory:** Thư mục làm việc trên Agent (ví dụ: `/home/jenkins/workspace`).
        
    - **Labels:** Đây là phần _quan trọng nhất_. Gõ vào đây các "tag" mô tả Agent. Ví dụ: `linux docker nodejs` (phân cách bằng dấu cách).
        
    - **Launch method:** Chọn **Launch agents via SSH**.
        
    - **Host:** Địa chỉ IP hoặc tên miền của máy Agent.
        
    - **Credentials:** Chọn private key SSH bạn đã lưu ở Credentials Manager.
        
    - **Host Key Verification Strategy:** Chọn `Non-verifying Verification Strategy` (dễ nhất cho lần đầu) hoặc `Known hosts file` (an toàn hơn).
        
    - `Save` và nhấn `Launch agent`.
        

Nếu thành công, Agent sẽ kết nối và báo "Idle" (rảnh rỗi).

#### 2. Sử dụng trong `Jenkinsfile`:

Bạn sẽ thay `agent any` bằng `agent { label '...' }`.

Groovy

```
// Jenkinsfile
pipeline {
    // Yêu cầu: "Chỉ chạy pipeline này trên một agent
    // CÓ ĐẦY ĐỦ cả 3 nhãn: 'linux', 'docker', và 'nodejs'"
    agent {
        label 'linux && docker && nodejs'
    }

    stages {
        stage('Build on Static Agent') {
            steps {
                echo "Đang chạy trên Agent có nhãn 'linux docker nodejs'"
                sh 'node -v'  // Lệnh này sẽ dùng 'node' đã cài sẵn trên agent
                sh 'docker version' // Lệnh này dùng 'docker' đã cài sẵn trên agent
            }
        }
    }
}
```

Ưu điểm: Tốc độ build nhanh (vì không mất thời gian khởi động).

Nhược điểm: Khó bảo trì. Nếu 100 dự án cần 100 phiên bản Node.js khác nhau, bạn sẽ "rối" khi cài đặt.

---

### 4.3. Loại 2: Dynamic Agents (Agent "Động" - Best Practice)

Đây là cách làm "chuẩn" của doanh nghiệp. Agent được _tạo ra_ (cấp phát) khi pipeline cần, và _bị hủy_ ngay sau khi build xong. Cách phổ biến nhất là dùng **Docker** hoặc **Kubernetes**.

#### 🔑 Keywords: Dynamic Agent, `agent { docker { ... } }`

#### 1. Agent as Container (Dùng `agent { docker { ... } }`)

Đây là cách dễ nhất để bắt đầu với Dynamic Agent.

- **Yêu cầu:** Agent (có thể là static agent `my-linux-agent-01` ở trên) phải cài sẵn Docker.
    
- **Cách hoạt động:** Khi Jenkins thấy `agent { docker { image '...' } }`, nó sẽ:
    
    1. Tìm một agent có cài Docker (ví dụ: agent có label `docker`).
        
    2. Trên agent đó, nó chạy `docker pull <image>`.
        
    3. Nó chạy `docker run <image>` để khởi động một container mới.
        
    4. Nó thực thi _tất cả_ các `steps` của bạn _bên trong_ container đó.
        
    5. Sau khi build xong, nó `docker stop` và `docker rm` container đó.
        

#### 2. Sử dụng trong `Jenkinsfile`:

Đây là cách "chuẩn" để build dự án Node.js mà _không cần_ cài Node.js lên Agent.

Groovy

```
// Jenkinsfile (Giai đoạn 4: Sử dụng Dynamic Docker Agent)
pipeline {
    // 1. Chỉ định agent là một Docker container
    // Jenkins sẽ tự động pull image 'node:18-alpine'
    // và chạy TẤT CẢ các stage bên trong container này.
    agent {
        docker {
            // Dùng image Node.js 18
            image 'node:18-alpine' 
            
            // (Tùy chọn) Chỉ định agent vật lý nào sẽ host container này
            // label 'docker' // Yêu cầu chạy trên agent có label 'docker'
            
            // (Tùy chọn) Mount volume, ví dụ: cache
            // args '-v ~/.npm:/root/.npm' 
        }
    }

    stages {
        stage('Verify Environment') {
            steps {
                // Các lệnh này đang chạy BÊN TRONG container 'node:18-alpine'
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
        
        // VẤN ĐỀ: Làm sao để chạy Docker-in-Docker?
        // Stage 'Build Docker Image' (từ GĐ 3) sẽ thất bại
        // vì container 'node:18-alpine' không cài Docker client.
        
        /*
        stage('Build Docker Image') {
            steps {
                sh 'docker build .' // Lỗi! 'docker' command not found
            }
        }
        */
    }
}
```

Ưu điểm: Môi trường siêu sạch cho mỗi lần build. Không cần quản lý tool (Node, Java) trên agent.

Nhược điểm: Hơi chậm (mất thời gian pull image).

---

### 4.4. Giải quyết vấn đề "Docker-in-Docker" (D-in-D)

Bạn đã thấy vấn đề ở trên: `stage('Build Docker Image')` cần lệnh `docker`, nhưng `agent` của chúng ta là container `node:18-alpine` (không có Docker).

**Giải pháp:** Sử dụng **nhiều `agent` khác nhau cho các `stage` khác nhau**. Đây là một kỹ thuật nâng cao cực kỳ mạnh mẽ.

- `pipeline { agent none }`: Khai báo rằng pipeline này không có agent _chung_.
    
- Mỗi `stage` sẽ tự định nghĩa `agent` riêng của mình.
    

#### ⌨️ Ví dụ: `Jenkinsfile` (Hoàn chỉnh Giai đoạn 4 - Multi-Agent)

Đây là pipeline "chuẩn" nhất, kết hợp Giai đoạn 2, 3, và 4.

Groovy

```
// Jenkinsfile (Giai đoạn 4: Multi-Agent Pipeline)
pipeline {
    // 1. Không dùng agent chung
    agent none 

    stages {
        
        // Stage 1: Build & Test (Chạy trong container Node.js)
        stage('Build & Test') {
            // 2. Chỉ định agent riêng cho stage này
            agent {
                docker { image 'node:18-alpine' }
            }
            steps {
                echo 'Đang chạy npm install & test BÊN TRONG container node:18'
                sh 'node -v'
                sh 'npm install'
                sh 'npm test'
                
                // 3. "Đưa" kết quả build ra ngoài (stash)
                // Vì container này sẽ bị xóa, chúng ta cần "cất"
                // thư mục 'dist' (ví dụ) để stage sau dùng.
                // 'build-output' là tên tạm do ta đặt
                stash name: 'build-output', includes: 'dist/'
            }
        }

        // Stage 2: SonarQube Analysis (Chạy trong container Sonar)
        stage('SonarQube Analysis') {
            agent {
                // Dùng image chính thức của SonarScanner
                docker { image 'sonarsource/sonar-scanner-cli:latest' }
            }
            steps {
                echo 'Đang chạy Sonar analysis BÊN TRONG container sonar-scanner'
                // Tương tự Giai đoạn 3, nhưng không cần 'tool'
                withSonarQubeEnv('MySonarQubeServer') {
                    // Chạy sonar-scanner
                    sh 'sonar-scanner -Dsonar.projectKey=my-node-app -Dsonar.sources=.'
                }
            }
        }
        
        // Stage 3: Chờ Quality Gate (Chạy trên một agent bất kỳ)
        stage('SonarQube Quality Gate') {
            // Stage này không cần môi trường đặc biệt,
            // chỉ cần một agent bất kỳ để chạy lệnh 'waitForQualityGate'
            agent any 
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // Stage 4: Build & Push Docker Image
        stage('Build & Push Docker Image') {
            // 4. Yêu cầu chạy trên agent "cứng" (static)
            // đã cài Docker (ví dụ agent ta tạo ở 4.2)
            agent {
                label 'linux && docker'
            }
            steps {
                echo 'Đang chạy build image BÊN NGOÀI (trên agent có cài Docker)'
                
                // 5. Lấy lại kết quả build từ stage 1
                // 'build-output' là tên đã đặt ở stage 1
                unstash name: 'build-output'
                
                // Giờ thư mục 'dist' đã có ở đây
                // (Giả sử Dockerfile của bạn cần 'dist')
                
                script {
                    def myImage = docker.build("vinh/my-node-app:${env.BUILD_NUMBER}", ".")
                    
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-registry-credentials') {
                        myImage.push()
                    }
                }
            }
        }
    } // end stages
}
```

### 📖 Bảng Thuật Ngữ (Giai đoạn 4)

|**Thuật ngữ**|**Giải thích chi tiết**|
|---|---|
|**Agent (Node)**|Máy thực thi các build step do Controller chỉ định. **Nơi làm việc thực sự.**|
|**Controller (Master)**|Máy chủ điều phối, lên lịch build, hiển thị UI. **Không được dùng để chạy build.**|
|**Executor**|Một "luồng" (thread) thực thi trên một Agent. Nếu một Agent có 2 executor, nó có thể chạy 2 build (hoặc 2 stage song song) cùng lúc.|
|**Label**|Một "nhãn" (tag) dùng để nhóm các Agent (ví dụ: `linux`, `gpu`, `performance-test`). Pipeline dùng label để yêu cầu một loại agent cụ thể (`agent { label '...' }`).|
|**Static Agent**|Agent được cài đặt cố định và luôn kết nối với Controller. Dùng cho các tác vụ đặc thù cần phần cứng riêng.|
|**Dynamic Agent**|Agent được tạo ra "theo yêu cầu" (on-demand) từ một "cloud" (ví dụ: Docker, Kubernetes, AWS EC2) khi build bắt đầu và bị hủy khi build kết thúc.|
|**`agent { docker { ... } }`**|Cú pháp Declarative, yêu cầu Jenkins chạy stage này _bên trong_ một container Docker được tạo ra tức thời.|
|**`stash` / `unstash`**|Các "step" dùng để "cất" (stash) file/folder từ một agent (ví dụ: trong container) và "lấy ra" (unstash) ở một agent khác (ví dụ: ngoài agent host). Cực kỳ hữu ích trong multi-agent pipeline.|
