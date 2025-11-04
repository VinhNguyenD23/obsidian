## 🚀 Hướng 1: Vận hành & Mở rộng (Jenkins trên Kubernetes)

### ❓ Vấn đề

`agent { docker }` (Giai đoạn 4) rất tốt, nhưng nó đòi hỏi bạn phải có một (hoặc vài) máy chủ "tĩnh" (Static Agent) đã cài sẵn Docker.

- **Single Point of Failure (SPOF):** Nếu máy chủ host Docker đó "chết", không pipeline nào chạy được.
    
- **Lãng phí tài nguyên:** Máy chủ đó phải "luôn bật" (always-on), ngay cả khi không có build.
    
- **Controller cũng là SPOF:** Chính Jenkins Controller của bạn cũng là một "con" (VM/Container). Nếu nó "chết", toàn bộ hệ thống CI/CD dừng lại.
    

### 💡 Giải pháp Enterprise: Jenkins on Kubernetes (K8s)

Doanh nghiệp giải quyết cả hai vấn đề này bằng cách cho _toàn bộ_ Jenkins (cả Controller và Agents) chạy trên **Kubernetes**.

#### 1. Controller HA (High Availability)

Bạn không cài Jenkins Controller bằng `docker run`. Bạn cài nó bằng **Helm Chart** (`jenkins/jenkins`).

- **Helm Chart là gì?** Nó là một "gói" cài đặt cho K8s.
    
- **Nó làm gì?** Nó tự động tạo:
    
    1. Một `Deployment` cho Jenkins Controller: K8s tự đảm bảo Pod này _luôn chạy_. Nếu Pod "chết", K8s lập tức tạo Pod mới.
        
    2. Một `PersistentVolumeClaim (PVC)`: Đây là phần _quan trọng sống còn_. Nó là "ổ cứng mạng" chứa toàn bộ dữ liệu `$JENKINS_HOME` (jobs, plugins, config). Khi Pod Controller "chết" và được K8s tạo lại, nó sẽ _gắn (attach)_ lại cái PVC này vào, giúp Jenkins "sống lại" mà **không mất dữ liệu**.
        
    3. Một `Service`: Để cung cấp địa chỉ IP/DNS cố định cho Jenkins UI.
        

#### 2. Dynamic Agents trên K8s (Thay thế `agent { docker }`)

Đây là phần "thay đổi cuộc chơi".

- **Plugin:** `Kubernetes Plugin`.
    
- **Cách hoạt động:**
    
    1. Bạn không tạo "Static Agent" nữa.
        
    2. Trong `Manage Jenkins` -> `Manage Nodes and Clouds`, bạn kết nối Jenkins với K8s API.
        
    3. Bạn định nghĩa các **Pod Templates** (Mẫu Pod) ngay trong JCasC/UI.
        
        - **Pod Template là gì?** Nó là một file YAML mô tả cái Pod (container) bạn muốn dùng để build. Ví dụ: "Tôi muốn một Pod có 2 container: `node:18` (để build) và `sonar-scanner` (để quét)".
            
    4. Khi pipeline chạy đến `agent { kubernetes { ... } }`, Jenkins sẽ _bảo_ K8s: "Hãy tạo cho tôi một Pod từ template `node-18-sonar`".
        
    5. K8s tự động tìm một máy chủ còn trống trong cụm (cluster) và tạo Pod đó.
        
    6. Jenkins chạy các `steps` bên trong Pod đó.
        
    7. Khi `stage` (hoặc pipeline) kết thúc, Pod đó bị **hủy hoàn toàn**.
        

#### ⌨️ Ví dụ: JCasC (Định nghĩa Pod Template)

Bạn sẽ định nghĩa các agent của mình trong file `jenkins.yaml` (JCasC):

YAML

```yaml
# jenkins.yaml
unclassified:
  kubernetes:
    clouds:
      - name: "kubernetes"
        # ... (kết nối K8s API) ...
        templates:
          # Đây là 1 Pod Template
          - name: "node-18-builder"
            label: "node-18" # Tên bạn gọi trong Jenkinsfile
            containers:
              # Container 1: Môi trường build chính
              - name: "node"
                image: "node:18-alpine"
                command: "sleep"
                args: "999999" # Giữ cho container "sống"
              # Container 2: Công cụ phụ trợ
              - name: "sonar"
                image: "sonarsource/sonar-scanner-cli:latest"
                command: "sleep"
                args: "999999"
            # Container bắt buộc để Jenkins kết nối vào
            - name: "jnlp"
              image: "jenkins/inbound-agent:latest"
```

#### ⌨️ Ví dụ: Jenkinsfile (Sử dụng Pod Template)

`Jenkinsfile` của bạn giờ đây siêu sạch:

Groovy

```Groovy
// Jenkinsfile
pipeline {
    // Yêu cầu K8s tạo 1 Pod từ template có label 'node-18'
    agent {
        kubernetes {
            label 'node-18'
        }
    }
    
    stages {
        stage('Build & Test') {
            steps {
                // 'container' bảo Jenkins chạy step này
                // trong container 'node' của Pod
                container('node') {
                    sh 'node -v'
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('Sonar Scan') {
            steps {
                // 'container' bảo Jenkins chạy step này
                // trong container 'sonar' của Pod
                container('sonar') {
                    withSonarQubeEnv('MySonarQubeServer') {
                        sh 'sonar-scanner -Dsonar.projectKey=...'
                    }
                }
            }
        }
    }
}
```

**✅ Lợi ích:** Mở rộng gần như vô hạn, tiết kiệm chi phí (chỉ chạy khi cần), không còn SPOF.

---

## 🔒 Hướng 2: Bảo mật Nâng cao (RBAC & External Secrets)

### ❓ Vấn đề

- **RBAC:** Mặc định, Jenkins chỉ có `admin` (làm mọi thứ) và `authenticated` (thấy mọi thứ). Trong công ty 100 người, Team A có thể thấy và _chạy_ job deploy của Team B. Cực kỳ nguy hiểm.
    
- **Secrets:** `Credentials Manager` (Giai đoạn 5) là tốt, nhưng nó lưu secrets _trong Jenkins_. Doanh nghiệp muốn một "két sắt" trung tâm duy nhất cho _tất cả_ ứng dụng (cả Jenkins, app, v.v.).
    

### 💡 Giải pháp Enterprise

#### 1. RBAC (Role-Based Authorization Strategy)

- **Plugin:** `Role-Based Authorization Strategy`.
    
- **Cách hoạt động:**
    
    1. Bạn kích hoạt plugin này trong `Manage Jenkins` -> `Configure Global Security`.
        
    2. Bạn vào `Manage Jenkins` -> `Manage and Assign Roles`.
        
    3. Bạn tạo ra 3 loại Role:
        
        - **Global Roles:** Quyền chung (ví dụ: `developer` chỉ có quyền `Overall/Read`).
            
        - **Item Roles (Quan trọng nhất):** Quyền trên _Job_ hoặc _Folder_.
            
        - **Node Roles:** Quyền trên _Agents_.
            
- **Ví dụ cấu hình (Assign Roles):**
    
    - **User `vinh`:** Gán `Global Role: admin`.
        
    - **Group `team-a-devs`:** Gán `Global Role: developer` + `Item Role: team-a-access`.
        
    - **Group `qa-team`:** Gán `Global Role: developer` + `Item Role: qa-access`.
        
- **Ví dụ cấu hình (Manage Roles):**
    
    - **Item Role `team-a-access`:**
        
        - **Pattern:** `team-a-.*` (Tự động áp dụng cho mọi job/folder bắt đầu bằng `team-a-`).
            
        - **Permissions:** `Job/Read`, `Job/Build`, `Job/Configure`, `Job/Delete`.
            
    - **Item Role `qa-access`:**
        
        - **Pattern:** `.*-deploy-staging` (Mọi job kết thúc bằng `-deploy-staging`).
            
        - **Permissions:** `Job/Read`, `Job/Build` (QA được _chạy_ deploy Staging, nhưng không được _cấu hình_ hay _xóa_).
            

#### 2. External Secrets (HashiCorp Vault)

- **Plugin:** `HashiCorp Vault Plugin`.
    
- **Cách hoạt động:**
    
    1. Công ty cài đặt một server **HashiCorp Vault** (két sắt trung tâm).
        
    2. Jenkins được cấp một "danh tính" (ví dụ: K8s Service Account) để _xác thực_ với Vault.
        
    3. Secrets (ví dụ: `prod/db/password`) được lưu trong Vault, _không phải Jenkins_.
        
    4. Trong pipeline, bạn dùng `withVault` (thay vì `withCredentials`).
        
    5. Plugin sẽ:
        
        - Tạm thời xác thực Jenkins với Vault.
            
        - "Mượn" (fetch) cái secret `prod/db/password`.
            
        - Inject nó vào một biến môi trường (ví dụ: `$DB_PASS`).
            
        - Khi block `withVault` kết thúc, biến môi trường bị hủy.
            
    6. **Kết quả:** Secret _không bao giờ_ được lưu trên Jenkins, chỉ "bay ngang qua" (in-memory) và bị che `****` trong log.
        

#### ⌨️ Ví dụ: Jenkinsfile (Sử dụng Vault)

Groovy

```Groovy
// Jenkinsfile
stage('Deploy to Production') {
    steps {
        // Yêu cầu plugin "mượn" secret từ Vault
        // và inject vào biến môi trường 'DB_PASSWORD'
        withVault(credentials: [
            VaultKVSecret(
                path: 'secret/data/production/database', // Đường dẫn trong Vault
                key: 'password', // Key của secret
                envVar: 'DB_PASSWORD' // Tên biến môi trường
            )
        ]) {
            
            // Giờ bạn có thể dùng $DB_PASSWORD
            // Nó sẽ bị che là '****' trong log
            sh "echo Deploying with pass: ${DB_PASSWORD}"
            sh "./deploy-script.sh --password ${DB_PASSWORD}"
            
        } // Biến $DB_PASSWORD bị hủy ở đây
    }
}
```

---

## ⚡ Hướng 3: Tối ưu Pipeline & GitOps (Parallel & ArgoCD)

### ❓ Vấn đề

- **Tốc độ:** Pipeline của bạn chạy _tuần tự_. `Build` (5 phút) -> `Test Backend` (10 phút) -> `Test Frontend` (10 phút). Tổng: 25 phút. Rất lãng phí.
    
- **Rủi ro (Push Model):** `Jenkinsfile` của bạn có stage `Deploy` chạy `sh 'kubectl apply -f ...'`.
    
    - **Rủi ro bảo mật:** Jenkins (hoặc agent) phải có quyền `cluster-admin` (quyền tối thượng) trên K8s. Nếu Jenkins bị hack, K8s của bạn "bay màu".
        
    - **Rủi ro vận hành:** Nếu pipeline `Deploy` thất bại giữa chừng, K8s của bạn rơi vào trạng thái "nửa vời" (nửa cũ, nửa mới). Jenkins không biết điều đó.
        

### 💡 Giải pháp Enterprise

#### 1. Tối ưu tốc độ (Parallel Stages)

Bạn cho các stage _không phụ thuộc_ nhau chạy song song.

- **Yêu cầu:** Bạn _phải_ có nhiều agent (K8s agents là hoàn hảo cho việc này).
    

#### ⌨️ Ví dụ: Jenkinsfile (Parallel)

Groovy

```Groovy
stage('Build & Test') {
    steps {
        sh 'npm install' // Cài đặt chung
        
        // Bắt đầu khối chạy song song
        parallel (
            "Backend Tests": {
                agent { kubernetes { label 'java-11-agent' } } // Agent riêng
                steps {
                    echo "Đang chạy Backend Tests..."
                    sh 'mvn test'
                }
            },
            "Frontend Tests": {
                agent { kubernetes { label 'node-18-agent' } } // Agent riêng
                steps {
                    echo "Đang chạy Frontend Tests..."
                    sh 'npm test'
                }
            },
            "Code Linting": {
                agent { kubernetes { label 'linter-agent' } } // Agent riêng
                steps {
                    echo "Đang chạy Linter..."
                    sh './run-linter.sh'
                }
            }
        ) // Kết thúc khối song song
    }
}
```

**Kết quả:** Thời gian `Test` giờ là `max(10 phút, 10 phút, 3 phút)` = **10 phút** (thay vì 23 phút).

#### 2. GitOps (Pull Model) - Giải pháp cho Deploy

Đây là sự thay đổi về _triết lý_ lớn nhất trong DevOps hiện đại.

- **Tư duy cũ (Push Model):** Jenkins "đẩy" (push) cấu hình lên K8s.
    
- **Tư duy mới (Pull Model - GitOps):** K8s "kéo" (pull) cấu hình từ Git.
    

**Công cụ:** **ArgoCD** (hoặc FluxCD).

**Luồng hoạt động (Workflow) chuẩn doanh nghiệp:**

1. **Repo 1 (Application Repo):** Lập trình viên push code `feature-A` (code Java/Node.js).
    
2. **Jenkins (CI):**
    
    - Trigger bởi Repo 1.
        
    - Chạy `Build`, `Test`, `Sonar`, `Build Docker Image` (ví dụ: `my-app:v1.2.3`).
        
    - Push image `my-app:v1.2.3` lên Docker Registry.
        
    - **Đây là bước mới:** Jenkins _không chạy `kubectl`_.
        
    - Jenkins _checkout_ **Repo 2 (Config Repo)**.
        
    - Nó sửa file `deployment.yaml` trong Repo 2: `image: my-app:v1.2.2` -> `image: my-app:v1.2.3`.
        
    - Jenkins _commit_ và _push_ thay đổi này lên Repo 2.
        
    - **== VAI TRÒ CỦA JENKINS KẾT THÚC ==**
        
3. **Repo 2 (Config Repo):** Repo này chứa _toàn bộ_ file YAML mô tả trạng thái của K8s. Git là **Single Source of Truth** (Nguồn Chân lý Duy nhất).
    
4. **ArgoCD (CD):**
    
    - Đây là một công cụ _riêng biệt_ chạy _bên trong_ K8s.
        
    - Nó được cấu hình để "theo dõi" Repo 2.
        
    - Nó phát hiện: "Hey, Repo 2 vừa có commit mới, nó muốn `image` là `v1.2.3`".
        
    - Nó so sánh: "Trạng thái _hiện tại_ trên K8s là `v1.2.2`".
        
    - Nó kết luận: "Trạng thái _không đồng bộ_ (Out of Sync)".
        
    - Nó tự động _kéo_ (pull) file YAML mới từ Repo 2 và chạy `kubectl apply` để cập nhật K8s.
        

**✅ Lợi ích:**

- **Bảo mật:** Jenkins không cần bất kỳ quyền nào vào K8s.
    
- **Kiểm toán:** Bạn muốn deploy? Bạn phải _mở Pull Request_ vào Repo 2. Lịch sử Git cho bạn biết _ai_ đã duyệt deploy, deploy _cái gì_, _khi nào_.
    
- **Tin cậy:** Trạng thái của K8s luôn được "khóa" với Git. Nếu ai đó "lỡ tay" `kubectl delete` một service, ArgoCD sẽ thấy "Out of Sync" và _tự động tạo lại_ ngay lập tức (tự chữa lành - self-healing).
