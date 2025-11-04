## 🗺️ Giai đoạn 2 (Chi tiết): Pipeline as Code (Declarative Pipeline)

### 🎯 Mục tiêu

Từ bỏ hoàn toàn việc "click-ops" (cấu hình bằng UI) của Freestyle Project. Bạn sẽ học cách định nghĩa _toàn bộ_ quy trình CI/CD (Build, Test, Deploy) bằng một file code duy nhất tên là **`Jenkinsfile`**. File này sẽ được lưu chung với mã nguồn dự án của bạn trong Git.

### 2.1. Tại sao lại là "Pipeline as Code" (PaC)?

Đây là triết lý _bắt buộc_ trong doanh nghiệp. Job Freestyle (Giai đoạn 1) có một nhược điểm chí mạng: **toàn bộ cấu hình của nó nằm trong Jenkins, không nằm trong code.**

- Nếu server Jenkins hỏng, bạn mất toàn bộ cấu hình build.
    
- Bạn không biết _ai_ đã thay đổi cấu hình build, _khi nào_, và _tại sao_.
    
- Rất khó để sao chép quy trình build cho 10 dự án khác nhau.
    

**PaC giải quyết tất cả:**

- **Quản lý phiên bản (Version Control):** `Jenkinsfile` nằm trong Git. Bạn có thể xem lịch sử, "revert" thay đổi, và thực hiện "code review" cho _chính quy trình build_.
    
- **Bền bỉ (Durable):** Cấu hình nằm trong Git. Nếu server Jenkins "chết", bạn chỉ cần cài server mới và trỏ nó vào Git, mọi pipeline tự động chạy lại.
    
- **Tái sử dụng (Reusable):** Dễ dàng copy `Jenkinsfile` cho dự án mới.
    
- **Audit (Kiểm toán):** Lịch sử Git cho bạn biết chính xác ai đã thay đổi logic deploy.
    

### 2.2. Tạo "Pipeline Job" - Cách Jenkins đọc `Jenkinsfile`

Đây là bước khác biệt với Freestyle. Lần này, chúng ta không "viết" logic trong Jenkins, chúng ta chỉ _bảo_ Jenkins đọc logic từ Git.

1. **Tạo Job:**
    
    - Từ trang chủ Jenkins, nhấn **New Item** (Tạo mục mới).
        
    - Đặt tên: `my-first-pipeline`.
        
    - Chọn **Pipeline** (KHÔNG chọn Freestyle project) và nhấn OK.
        
2. **Cấu hình (Configuration):**
    
    - Bạn sẽ thấy trang cấu hình đơn giản hơn Freestyle rất nhiều, vì hầu hết các tab đã biến mất.
        
    - Cuộn xuống mục **Pipeline**. Đây là phần quan trọng nhất.
        
    - **Definition (Định nghĩa):** Chọn **Pipeline script from SCM**.
        
        - Đây là bạn đang bảo Jenkins: "Đừng tìm code trong ô text bên dưới, hãy đi tìm file `Jenkinsfile` trong Git."
            
    - **SCM:** Chọn **Git**.
        
    - **Repository URL:** Dán URL của dự án (ví dụ: `https://github.com/jenkins-docs/simple-node-js-react-app.git`).
        
    - **Branch Specifier:** Để `*/main` hoặc `*/master`.
        
    - **Script Path:** Đây là tên file chứa pipeline. Mặc định là `Jenkinsfile`. **Hãy luôn dùng tên này.**
        
3. Nhấn **Save** (Lưu).
    

Xong! Job của bạn đã được tạo. Nó chưa làm gì cả cho đến khi bạn thêm file `Jenkinsfile` vào repository Git.

### 2.3. Giải phẫu một `Jenkinsfile` (Declarative Pipeline)

Chúng ta sẽ học cú pháp **Declarative Pipeline**. Nó có cấu trúc rõ ràng, dễ đọc và là chuẩn mực hiện nay. Cú pháp này được viết bằng một ngôn ngữ gọi là **Groovy**, nhưng bạn không cần biết Groovy để bắt đầu.

Đây là cấu trúc "xương sống" của một `Jenkinsfile`:

Groovy

```
// Jenkinsfile
pipeline { // 1. Block gốc: Bất cứ thứ gì cũng phải nằm trong đây

    // 2. Chỉ định nơi chạy: Chạy trên agent (máy thực thi) nào?
    agent any // 'any' = Chạy trên bất kỳ agent nào đang rảnh

    // 3. Tập hợp các Giai đoạn (Stages)
    stages {
        
        // 4. Một Giai đoạn (Stage)
        // Đây là một "bước" logic, sẽ được hiển thị trên UI
        stage('Tên Giai Đoạn 1: Build') { 
            // 5. Các bước thực thi (Steps)
            steps {
                // Các lệnh thực tế chạy ở đây
                echo 'Bắt đầu build...'
                sh 'npm install' // 'sh' = chạy lệnh shell
            }
        }

        stage('Tên Giai Đoạn 2: Test') {
            steps {
                echo 'Bắt đầu test...'
                sh 'npm test'
            }
        }
    }
    
    // 6. Hành động sau Build (Post)
    post { 
        // Luôn luôn chạy, dù thành công hay thất bại
        always {
            echo 'Pipeline đã chạy xong.'
            cleanWs() // Step đặc biệt: dọn dẹp thư mục làm việc
        }
        // Chỉ chạy khi toàn bộ pipeline thành công
        success {
            echo 'Build thành công!'
        }
        // Chỉ chạy khi có một stage nào đó thất bại
        failure {
            echo 'Build thất bại!'
        }
    }
}
```

### 2.4. Thực hành: Chuyển Job Giai đoạn 1 sang `Jenkinsfile`

Bây giờ, hãy tạo một file tên `Jenkinsfile` (với chữ `J` viết hoa) ở thư mục gốc của dự án Node.js (cùng cấp với `package.json`).

**Nội dung file `Jenkinsfile`:**

Groovy

```
// Jenkinsfile (Cú pháp Declarative)
pipeline {
    // 1. CHỌN MÔI TRƯỜNG
    // Yêu cầu chạy trên bất kỳ agent nào có sẵn.
    agent any 

    // 2. ĐỊNH NGHĨA CÁC GIAI ĐOẠN
    // 'stages' là một khối chứa các 'stage' con
    stages {
        
        // Giai đoạn 1: Cài đặt (Giống hệt 'Execute shell' trong Freestyle)
        stage('Install Dependencies') {
            steps {
                echo "--- BẮT ĐẦU CÀI ĐẶT DEPENDENCIES ---"
                // 'sh' là một "step" (bước) để chạy lệnh shell.
                // Jenkins sẽ tự động 'fail' stage này nếu lệnh trả về lỗi.
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
            }
        }

        // Giai đoạn 2: Kiểm thử (Giống hệt 'Execute shell' trong Freestyle)
        stage('Unit Test') {
            steps {
                echo "--- BẮT ĐẦU CHẠY UNIT TEST ---"
                sh 'npm test'
            }
        }

        // Giai đoạn 3: Lưu trữ (Giống hệt 'Archive artifacts' trong Freestyle)
        stage('Archive') {
            steps {
                echo "--- ĐANG LƯU TRỮ ARTIFACTS ---"
                // 'archiveArtifacts' là một "step" (giống 'sh')
                // được cung cấp bởi Jenkins để lưu file.
                // Chúng ta lưu lại file package-lock.json làm ví dụ
                archiveArtifacts artifacts: 'package-lock.json', followSymlinks: false
            }
        }
    }

    // 3. HÀNH ĐỘNG SAU KHI BUILD
    post {
        // 'always' đảm bảo rằng dù build thành công (success) 
        // hay thất bại (failure), các lệnh này luôn được chạy.
        always {
            echo 'Hoàn tất pipeline.'
            
            // 'cleanWs' là step dọn dẹp workspace 
            // (thư mục code đã clone) để chuẩn bị cho lần build sau.
            // Đây là một Best Practice.
            cleanWs() 
        }
        success {
            // Gửi email, thông báo Slack... (sẽ học sau)
            echo 'Pipeline thành công!'
        }
        failure {
            // Gửi email, thông báo Slack... (sẽ học sau)
            echo 'Pipeline thất bại!'
        }
    }
}
```

### 2.5. Chạy và xem kết quả (Stage View)

1. Commit và đẩy (push) file `Jenkinsfile` này lên repository Git của bạn.
    
2. Quay lại giao diện Jenkins, vào job `my-first-pipeline` và nhấn **Build Now**.
    

**Kết quả bạn sẽ thấy (Khác biệt lớn so vớii Freestyle):**

- Bạn sẽ thấy một giao diện gọi là **Stage View**.
    
- Nó hiển thị các cột trực quan: `Install Dependencies`, `Unit Test`, `Archive`.
    
- Jenkins sẽ tô màu XANH LÁ nếu stage thành công, và ĐỎ nếu thất bại.
    
- Nếu `npm test` (stage "Unit Test") thất bại, pipeline sẽ _dừng ngay lập tức_ và báo đỏ, nó sẽ _không_ chạy stage "Archive". Đây chính là logic "fail-fast" (thất bại nhanh) mà chúng ta muốn.
    
- Bạn có thể nhấn vào từng stage để xem log _chỉ riêng của stage đó_.
    

### 📖 Bảng Thuật Ngữ (Giai đoạn 2)

| **Thuật ngữ**              | **Giải thích chi tiết**                                                                                                                            |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pipeline as Code (PaC)** | Triết lý lưu trữ và quản lý cấu hình pipeline (quy trình build/deploy) dưới dạng code (ví dụ: `Jenkinsfile`) thay vì cấu hình trên UI.             |
| **Jenkinsfile**            | Tên file mặc định (viết hoa chữ J) chứa định nghĩa pipeline. Jenkins tự động đọc file này từ SCM.                                                  |
| **Declarative Pipeline**   | Cú pháp PaC hiện đại, có cấu trúc rõ ràng (pipeline, agent, stages, steps). Dễ viết và dễ đọc.                                                     |
| **Scripted Pipeline**      | Cú pháp PaC cũ, linh hoạt hơn nhưng khó viết hơn (gần như code Groovy thuần). Bạn nên biết sự tồn tại của nó, nhưng hãy tập trung vào Declarative. |
| **`pipeline`**             | Khối (block) ngoài cùng, khai báo toàn bộ là một Declarative Pipeline.                                                                             |
| **`agent`**                | Chỉ thị (directive) xác định môi trường thực thi (nơi chạy build). `agent any` là chạy trên bất kỳ máy thực thi (agent) nào có sẵn.                |
| **`stage`**                | Đại diện cho một giai đoạn logic riêng biệt trong pipeline (ví dụ: Build, Test, Deploy). Các `stage` được trực quan hóa trên UI của Jenkins.       |
| **`steps`**                | Một khối (block) bên trong `stage`, chứa các lệnh thực thi thực tế (ví dụ: `sh 'npm install'`, `echo 'hello'`).                                    |
| **`post`**                 | Block tùy chọn, định nghĩa các hành động _sau khi_ pipeline hoàn tất, dựa trên kết quả (success, failure, always, unstable...).                    |
| **`sh`**                   | Một "step" (hàm) cơ bản dùng để thực thi một lệnh shell (Linux/macOS). Tương đương với `bat` trên Windows.                                         |
| **`cleanWs`**              | Một "step" (hàm) tiện ích dùng để xóa toàn bộ nội dung trong thư mục workspace sau khi build xong.                                                 |
