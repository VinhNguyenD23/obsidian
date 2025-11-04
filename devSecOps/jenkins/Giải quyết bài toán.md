### 1. Vận hành & Mở rộng Quy mô lớn (Scalability & High Availability)

Khi Jenkins của bạn trở nên quá quan trọng, nó không được phép "chết". Và khi có 1000 build cùng lúc, nó phải xử lý được.

* **Jenkins on Kubernetes (Quan trọng nhất):**
    * **Vấn đề:** `agent { docker }` (Giai đoạn 4) vẫn cần một "static agent" (máy chủ cố định) đã cài Docker. Nếu máy chủ đó chết, mọi thứ dừng lại.
    * **Giải pháp Enterprise:** Chạy *cả* Jenkins Controller và Agents trên cụm **Kubernetes (K8s)**.
    * **Cách hoạt động:** Thay vì `agent { docker }`, bạn sẽ dùng `agent { kubernetes { ... } }`. Khi một pipeline bắt đầu, Jenkins sẽ *yêu cầu* K8s cấp phát một `Pod` (container) mới từ template (ví dụ: Pod có cài `node:18` và `sonar-scanner`). K8s sẽ tự động tìm máy chủ (node) còn trống trong cụm (cluster) để chạy Pod đó.
    * **Lợi ích:** Mở rộng (scale) vô hạn, không phụ thuộc vào một máy chủ vật lý nào, và cực kỳ linh hoạt.
    * **🔑 Keywords:** `Kubernetes Plugin`, `Pod Templates`, `Jenkins Helm Chart`.

* **High Availability (HA - Tính Sẵn sàng Cao):**
    * **Vấn đề:** Dù chạy trên K8s, Jenkins Controller của bạn vẫn là một "con" (Pod). Nếu nó "chết", toàn bộ hệ thống CI/CD dừng lại (không ai build/deploy được) cho đến khi nó khởi động lại.
    * **Giải pháp Enterprise:** Cấu hình Jenkins ở chế độ **Active/Passive** (một con chạy, một con chờ) hoặc **Active/Active** (cả hai cùng chạy - thường là sản phẩm trả phí). Khi Controller chính chết, Controller dự phòng sẽ "lên" ngay lập tức, đảm bảo Jenkins không "chết" (zero downtime).
    * **🔑 Keywords:** `Jenkins HA`, `CloudBees Core` (bản trả phí của Jenkins), `DR (Disaster Recovery)`.

---

### 2. Bảo mật & Quản trị Nâng cao (Advanced Security & Governance)

Khi có hàng trăm lập trình viên, bạn không thể để "ai cũng có quyền admin".

* **Role-Based Authorization Strategy (RBAC):**
    * **Vấn đề:** Mặc định, Jenkins chỉ có "admin" và "user" (thấy tất cả job). Điều này rất nguy hiểm.
    * **Giải pháp Enterprise:** Dùng plugin `Role-Based Authorization Strategy`. Bạn tạo ra các "Vai trò" (Roles) và gán quyền chi tiết.
    * **Ví dụ:**
        * Role `dev-team-A`: Chỉ được `Read`, `Build`, `Configure` các job trong Folder `Team-A`.
        * Role `dev-team-B`: Chỉ được `Read`, `Build` các job trong Folder `Team-B` (không được `Configure`).
        * Role `QA-team`: Chỉ được `Read` (xem) tất cả job, nhưng chỉ được `Build` (chạy) các job `deploy-staging`.
    * **🔑 Keywords:** `Role-Based Authorization Strategy Plugin`, `Matrix Authorization`, `Folders Plugin`.

* **Quản lý Secrets bên ngoài (External Secrets Management):**
    * **Vấn đề:** Credentials Manager (Giai đoạn 5) là tốt, nhưng trong doanh nghiệp, secrets (mật khẩu, API key) thường được quản lý ở một nơi *tập trung* duy nhất.
    * **Giải pháp Enterprise:** Tích hợp Jenkins với các hệ thống quản lý bí mật bên ngoài như **HashiCorp Vault** hoặc các dịch vụ của cloud (AWS Secrets Manager, GCP Secret Manager).
    * **Lợi ích:** Jenkins không *lưu* mật khẩu, nó chỉ "hỏi mượn" Vault khi pipeline chạy. Bảo mật cao hơn và quản lý tập trung.
    * **🔑 Keywords:** `HashiCorp Vault Plugin`, `AWS Secrets Manager Credentials Provider`.

* **Audit Trail (Theo vết Kiểm toán):**
    * **Vấn đề:** Ai đã nhấn nút "Deploy to Production" lúc 2 giờ sáng? Ai đã thay đổi cấu hình job?
    * **Giải pháp Enterprise:** Cài plugin `Audit Trail`. Nó ghi lại *mọi hành động* của người dùng (ai, làm gì, khi nào) vào một file log riêng, phục vụ cho việc kiểm toán (compliance) và điều tra sự cố.
    * **🔑 Keywords:** `Audit Trail Plugin`, `Compliance as Code`.

---

### 3. Tối ưu Tốc độ Pipeline (Pipeline Optimization)

Khi pipeline của bạn chạy mất 30 phút, đó là một sự lãng phí. Doanh nghiệp cần nó chạy trong 5 phút.

* **Parallel Stages (Chạy Song song):**
    * **Vấn đề:** Pipeline của bạn chạy tuần tự: `Build` -> `Test Backend` -> `Test Frontend` -> `Scan`.
    * **Giải pháp Enterprise:** Cho các stage *không phụ thuộc* nhau chạy song song. Ví dụ: Chạy `Test Backend`, `Test Frontend`, và `Scan` *cùng một lúc* sau khi `Build` xong.
    * **Cách làm:** Dùng block `parallel` trong `Jenkinsfile`.
    * **🔑 Keywords:** `parallel { ... }`, `Dynamic Agents` (bạn cần nhiều agent để chạy song song).

* **Caching (Lưu đệm):**
    * **Vấn đề:** `npm install` hoặc `mvn install` tải về "cả thế giới" (hàng GB thư viện) *mỗi lần* build, dù bạn dùng `agent { docker }` (vì container bị xóa).
    * **Giải pháp Enterprise:** Thiết lập cơ chế cache.
        * **Cách 1:** Mount một thư mục "cache" (ví dụ: `.npm`, `.m2`) vào agent.
        * **Cách 2 (Chuẩn):** Dùng **Artifact Repository (Nexus/Artifactory)** (đã nói ở Giai đoạn 3) làm "proxy cache". Agent sẽ tải thư viện từ Nexus (trong mạng nội bộ, siêu nhanh) thay vì từ Internet.
    * **🔑 Keywords:** `cache` step (trong Pipeline), `Nexus Proxy Repository`, `Docker Layer Caching`.

---

### 4. Continuous Deployment (CD) Nâng cao & GitOps

Đây là một mảng *rất lớn*. Giai đoạn 3-4 mới chỉ là **Continuous Integration (CI)** (tạo ra artifact/image). Doanh nghiệp cần **Continuous Deployment (CD)** (triển khai artifact đó).

* **Deployment Strategies (Chiến lược Triển khai):**
    * Bạn sẽ học cách viết `Jenkinsfile` để thực hiện các chiến lược deploy phức tạp.
    * **User Input:** Thêm `input` step vào pipeline, ví dụ: `stage('Approve Deploy to Production')`. Pipeline sẽ *dừng lại* và chờ một manager nhấn "Approve" (Phê duyệt) trên UI.
    * **Blue/Green Deployment:** Jenkins deploy phiên bản mới (Green) song song với bản cũ (Blue), sau đó chỉ cần chuyển hướng traffic.
    * **Canary Deployment:** Jenkins deploy bản mới cho 1% người dùng, nếu ổn, tăng lên 10%,...
    * **🔑 Keywords:** `input` step, `Blue/Green`, `Canary`.

* **GitOps (Khái niệm quan trọng nhất):**
    * **Vấn đề:** Dùng Jenkins (`sh 'kubectl apply ...'`) để *đẩy* (push) code lên K8s là một cách làm cũ và rủi ro (Jenkins cần quá nhiều quyền).
    * **Giải pháp Enterprise (Hiện đại):** Jenkins **không deploy**.
    * **Cách hoạt động (GitOps):**
        1.  **Jenkins (CI):** Chạy build, test, scan, build Docker image (như chúng ta đã học).
        2.  **Jenkins (CI):** *Push* image lên Registry.
        3.  **Jenkins (CI):** Mở một Pull Request vào một **Repo Git Cấu hình** (ví dụ: `app-config-repo`), tự động cập nhật file YAML trong đó: `image: my-app:v1.2.3` (thay vì `v1.2.2`).
        4.  **ArgoCD / Flux (CD):** Đây là một công cụ *khác* (chạy trong K8s). Nó "theo dõi" cái Repo Git Cấu hình. Khi thấy repo đổi (`v1.2.3`), nó tự động *kéo* (pull) cấu hình đó về và cập nhật K8s.
    * **Lợi ích:** Jenkins không cần quyền deploy. Mọi thứ được quản lý qua Git (ai duyệt PR deploy, deploy khi nào).
    * **🔑 Keywords:** `GitOps`, `ArgoCD`, `FluxCD`, `Push vs. Pull`.
