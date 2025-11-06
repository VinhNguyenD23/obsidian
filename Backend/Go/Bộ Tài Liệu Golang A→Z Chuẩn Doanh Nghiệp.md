# Mô tả

Tài liệu này là cẩm nang đào tạo nội bộ, mục tiêu thống nhất tiêu chuẩn phát triển, vận hành và bảo mật cho các dự án sử dụng Golang trong môi trường microservices, cloud-native.

# 1. Giới thiệu tổng thể

### 1.1. Golang dùng ở doanh nghiệp cho việc gì?

Golang (Go) là lựa chọn hàng đầu cho các hệ thống backend hiệu năng cao, đặc biệt trong kiến trúc microservices và cloud-native.

- **API Backends & Microservices:** Xây dựng các dịch vụ REST/gRPC nhanh, nhẹ, dễ dàng đóng gói và triển khai.
    
- **Hạ tầng & DevOps Tools:** Rất nhiều công cụ hạ tầng (Kubernetes, Docker, Prometheus) được viết bằng Go. Go lý tưởng để viết các công cụ CLI, controller, operator.
    
- **Xử lý dữ liệu & Pipeline:** Khả năng xử lý đồng thời (concurrency) mạnh mẽ giúp Go phù hợp cho các pipeline xử lý dữ liệu, streaming, và các worker pool.
    
- **Proxy & Gateway:** Nhờ hiệu năng mạng và I/O tốt, Go thường được dùng để viết API Gateway, service mesh proxy, và các-lớp-đệm-hiệu-năng-cao.
    

### 1.2. Triết lý của Golang

- **Simplicity (Đơn giản):** Ngôn ngữ nhỏ, cú pháp rõ ràng, ít "ma thuật". Ưu tiên sự rõ ràng (readability) hơn là sự ngắn gọn (brevity).
    
- **Fast Build (Biên dịch nhanh):** Tốc độ biên dịch cực nhanh, giúp chu trình phát triển (dev loop) và CI/CD hiệu quả.
    
- **Static Binary (Tệp nhị phân tĩnh):** Biên dịch ra một tệp nhị phân duy nhất, không phụ thuộc vào thư viện bên ngoài (trừ khi dùng CGO). Điều này giúp việc triển khai (deploy) trong container trở nên cực kỳ đơn giản.
    
- **Concurrency (Xử lý đồng thời):** Hỗ trợ concurrency là cốt lõi của ngôn ngữ (goroutine và channel), không phải là thư viện thêm vào.
    

### 1.3. Kiến trúc tổng quan hệ thống mẫu

Chúng ta sẽ xây dựng các dịch vụ theo mô hình microservices tiêu chuẩn. Một luồng request (yêu cầu) điển hình:

1. **Client/User** → **API Gateway** (Quản lý AuthN/AuthZ, Rate Limit, Caching)
    
2. **API Gateway** → **Service A (gRPC/REST)** (Ví dụ: Dịch vụ Quản lý Người dùng)
    
3. **Service A** → **Database (Postgres)** và/hoặc **Cache (Redis)**
    
4. **Service A** → **Service B (gRPC/Kafka)** (Gọi dịch vụ khác hoặc bắn sự kiện)
    
5. **Tất cả dịch vụ** → **Observability Stack** (Logging: ELK/Loki, Metrics: Prometheus, Tracing: Jaeger/OpenTelemetry)
    

### 1.4. Các tiêu chuẩn sẽ dùng

- **Clean Architecture (biến thể):** Tách biệt rõ ràng các lớp: Transport (HTTP/gRPC), Use Case (Service/Logic), và Storage (Repository).
    
- **12-Factor App:** Tuân thủ các nguyên tắc xây dựng ứng dụng cloud-native (ví dụ: config qua biến môi trường, logging ra stdout/stderr, stateless).
    
- **OWASP ASVS:** Áp dụng các tiêu chuẩn bảo mật của OWASP (Open Web Application Security Project) ở mức độ phù hợp. `[TODO: Xác định Level ASVS (1, 2, 3) cho dự án]`.
    

# 2. Nền tảng Golang (Beginner → Junior)

Phần này tập trung vào các khái niệm cốt lõi của Go để xây dựng ứng dụng.

## 2.1. Cài đặt và Workspace

### Mô tả

Quản lý project, dependencies (thư viện phụ thuộc) và cấu trúc thư mục là nền tảng để đảm bảo tính nhất quán.

### Checklist

- [ ] Hiểu `go mod` để quản lý dependencies.
    
- [ ] Biết cách dùng `go work` cho môi trường monorepo/multi-module.
    
- [ ] Nắm vững cấu trúc thư mục chuẩn của dự án (Standard Go Project Layout).
    

### Cấu trúc Repo chuẩn

Chúng ta tuân theo [Standard Go Project Layout](https://github.com/golang-standards/project-layout "null") đã được điều chỉnh.

```
/my-service
├── /cmd          # Điểm vào (main) của các ứng dụng
│   └── /server   # Ứng dụng server chính
│       └── main.go
├── /internal     # Code nghiệp vụ nội bộ, không thể import từ bên ngoài
│   ├── /handler  # Lớp HTTP/gRPC (adapters)
│   ├── /service  # Lớp nghiệp vụ (use cases)
│   ├── /repository # Lớp truy cập dữ liệu (adapters)
│   └── /domain   # Các struct, interface cốt lõi
├── /pkg          # Code có thể chia sẻ (share) và import bởi dự án khác (cẩn trọng khi dùng)
├── /configs      # Tệp cấu hình mẫu (config.yml.example)
├── /deploy       # Tệp YAML cho Kubernetes, Dockerfile
│   ├── /docker
│   │   └── Dockerfile
│   └── /k8s
│       └── deployment.yaml
├── go.mod        # Quản lý dependencies
├── go.sum
└── .golangci.yml # Cấu hình linter
```

### Lỗi thường gặp

- Bỏ _tất cả_ code vào `/pkg` ngay cả khi chúng không cần chia sẻ. Điều này vi phạm tính đóng gói.
    
- Dùng `go work` khi không cần thiết (chỉ dùng khi phát triển đồng thời nhiều module liên quan).
    

## 2.2. Cú pháp ngôn ngữ cốt lõi

### Mô tả

Nắm vững các thành phần cơ bản: type, struct, interface, pointer và cách xử lý lỗi.

### Checklist

- [ ] Định nghĩa `struct` để mô hình hóa dữ liệu.
    
- [ ] Định nghĩa `interface` để trừu tượng hóa hành vi.
    
- [ ] Hiểu khi nào dùng method receiver là `value` (nhận giá trị) và khi nào dùng `pointer` (con trỏ).
    
- [ ] Xử lý lỗi (error) như một giá trị trả về, không dùng `panic` cho logic thông thường.
    

### Best Practice: Method Receiver

- Dùng **Pointer Receiver (`*T`)**:
    
    1. Khi bạn cần **thay đổi** trạng thái của `struct`.
        
    2. Khi `struct` lớn và việc copy (sao chép) nó tốn kém.
        
    3. Để đảm bảo tính nhất quán (nếu một số method dùng pointer, hãy dùng tất cả là pointer).
        
- Dùng **Value Receiver (`T`)**:
    
    1. Khi `struct` nhỏ, bất biến (immutable) và là kiểu dữ liệu cơ bản (slice, map).
        

### Ví dụ Code: User Domain

```go
package domain

import (
	"errors"
	"strings"
)

// User định nghĩa đối tượng người dùng.
// Struct tag `json` định nghĩa cách nó được serialize/deserialize.
type User struct {
	ID    string `json:"id"`
	Email string `json:"email"`
	// Không bao giờ export field password ra JSON
	passwordHash string
}

// NewUser là hàm khởi tạo (constructor) an toàn.
func NewUser(email, password string) (*User, error) {
	// Giả sử có hàm hash
	hashedPassword := hashPassword(password) // (implementation chi tiết ở lớp khác)

	u := &User{
		// ID sẽ được gán bởi DB hoặc UUID
		Email:        email,
		passwordHash: hashedPassword,
	}

	if err := u.Validate(); err != nil {
		return nil, err
	}
	return u, nil
}

// Validate sử dụng pointer receiver (*User) vì nó có thể được gọi
// trên một đối tượng đã tồn tại và logic validate có thể phức tạp.
// Mặc dù ở đây nó không thay đổi state, nhưng việc dùng pointer là nhất quán.
func (u *User) Validate() error {
	if u.Email == "" || !strings.Contains(u.Email, "@") {
		return errors.New("invalid email")
	}
	if len(u.passwordHash) < 8 { // Giả định hash luôn dài hơn 8
		return errors.New("invalid password (hash too short)")
	}
	return nil
}

// CheckPassword dùng pointer receiver để nhất quán.
func (u *User) CheckPassword(password string) bool {
	// Giả sử có hàm verify
	return verifyPassword(u.passwordHash, password)
}

// (Các hàm hashPassword/verifyPassword không được định nghĩa ở đây)
func hashPassword(password string) string { return "hashed_" + password }
func verifyPassword(hash, password string) bool { return hash == "hashed_"+password }
```

### Lỗi thường gặp

- Trả về `nil` thay vì `error` khi không có lỗi (phải trả về `nil` tường minh).
    
- `panic` khi validation thất bại (chỉ `panic` khi gặp lỗi không thể phục hồi, ví dụ: không thể kết nối DB lúc khởi động).
    

## 2.3. Collection và Built-in

### Mô tả

`slice` và `map` là hai cấu trúc dữ liệu quan trọng nhất trong Go.

### Checklist

- [ ] Khởi tạo `map` và `slice` bằng `make()`.
    
- [ ] Hiểu sự khác biệt giữa `len()` và `cap()` của `slice`.
    
- [ ] Biết cách dùng `append` an toàn.
    

### Lỗi thường gặp

1. **Nil Map Panic:** Ghi vào một `map` chưa được khởi tạo.
    
    ```go
    // LỖI: panic: assignment to entry in nil map
    var m map[string]string
    m["key"] = "value"
    
    // SỬA: Luôn dùng make()
    m := make(map[string]string)
    m["key"] = "value"
    ```
    
2. **Slice Copy không mong muốn:** `append` có thể thay đổi mảng gốc nếu `cap` còn đủ.
    
    ```go
    a := []int{1, 2, 3}
    b := a[:2] // b share cùng mảng backing với a
    b = append(b, 99) //
    // a bây giờ là [1, 2, 99], không phải [1, 2, 3] như mong đợi!
    
    // SỬA: Dùng copy() nếu cần slice độc lập
    c := make([]int, 2)
    copy(c, a[:2])
    c = append(c, 99) // a vẫn là [1, 2, 3]
    ```
    

## 2.4. Concurrency cơ bản

### Mô tả

Goroutine và Channel là nền tảng cho xử lý đồng thời trong Go. `Context` là bắt buộc để quản lý request-scoped-data, cancellation và timeout.

### Checklist

- [ ] Biết cách khởi chạy hàm bằng `go func()`.
    
- [ ] Sử dụng `channel` (buffered/unbuffered) để giao tiếp.
    
- [ ] Dùng `select` để chờ trên nhiều channel.
    
- [ ] **BẮT BUỘC:** Truyền `context.Context` làm tham số đầu tiên cho mọi hàm có I/O hoặc có thể bị hủy (cancel).
    

### Ví dụ: Worker Pool đơn giản

Một worker pool xử lý các "job" (công việc).

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

// processJob là công việc cần thực hiện
func processJob(ctx context.Context, jobID int) error {
	select {
	case <-time.After(500 * time.Millisecond): // Giả lập công việc tốn thời gian
		fmt.Printf("Processed job %d\n", jobID)
		return nil
	case <-ctx.Done(): // Bị hủy (ví dụ: timeout)
		fmt.Printf("Cancelled job %d\n", jobID)
		return ctx.Err()
	}
}

func main() {
	const numJobs = 10
	const numWorkers = 3

	// Context với timeout 2 giây
	// Toàn bộ công việc phải hoàn thành trong 2s
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	jobs := make(chan int, numJobs)
	results := make(chan error, numJobs)
	var wg sync.WaitGroup

	// Khởi tạo workers
	for w := 0; w < numWorkers; w++ {
		wg.Add(1)
		go func(workerID int) {
			defer wg.Done()
			// Worker sẽ sống cho đến khi channel 'jobs' bị đóng
			for jobID := range jobs {
				fmt.Printf("Worker %d starting job %d\n", workerID, jobID)
				
				// Truyền context vào hàm xử lý
				err := processJob(ctx, jobID)
				
				// Gửi kết quả (hoặc lỗi)
				select {
				case results <- err:
				case <-ctx.Done(): // Nếu context đã bị hủy, không gửi nữa
					return
				}
			}
			fmt.Printf("Worker %d shutting down\n", workerID)
		}(w)
	}

	// Đẩy jobs vào channel
	for j := 0; j < numJobs; j++ {
		jobs <- j
	}
	close(jobs) // Báo cho worker biết đã hết jobs

	// Đợi tất cả worker hoàn thành
	wg.Wait()
	close(results) // Đóng channel results sau khi wg.Wait()
	fmt.Println("All workers finished")

	// Thu thập kết quả
	for err := range results {
		if err != nil {
			fmt.Printf("A job failed: %v\n", err)
		}
	}
	fmt.Println("Finished processing results")
}
```

### Lỗi thường gặp

- **Quên truyền `context`**: Mất khả năng cancel hoặc set timeout cho I/O (DB, HTTP call).
    
- **Leak Goroutine**: Khởi chạy goroutine mà không có cách nào để nó kết thúc (ví dụ: chờ channel mãi mãi). Luôn dùng `context` hoặc `close(channel)` để ra tín hiệu kết thúc.
    

## 2.5. Testing cơ bản

### Mô tả

Go tích hợp sẵn `testing` package mạnh mẽ.

### Checklist

- [ ] Viết unit test (hàm `TestXxx`).
    
- [ ] Sử dụng **Table-Driven Tests** (kiểm thử theo bảng) để bao phủ nhiều ca.
    
- [ ] Viết benchmark (hàm `BenchmarkXxx`) để đo hiệu năng.
    

### Ví dụ: Table-Driven Test cho HTTP Handler (Mock)

Giả sử chúng ta có một handler đơn giản:

```go
package main

import (
	"encoding/json"
	"net/http"
)

func GetUserHandler(w http.ResponseWriter, r *http.Request) {
	userID := r.URL.Query().Get("id")
	if userID == "" {
		http.Error(w, "missing id", http.StatusBadRequest)
		return
	}

	if userID == "1" {
		w.Header().Set("Content-Type", "application/json")
		w.WriteHeader(http.StatusOK)
		json.NewEncoder(w).Encode(map[string]string{"id": "1", "name": "Admin"})
		return
	}

	http.Error(w, "not found", http.StatusNotFound)
}
```

Và đây là file test `main_test.go`:

```go
package main

import (
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"
)

func TestGetUserHandler(t *testing.T) {
	// Định nghĩa các ca kiểm thử
	tests := []struct {
		name       string // Tên ca kiểm thử
		queryParam string // Input
		wantCode   int    // Output mong đợi (HTTP Status)
		wantBody   string // Output mong đợi (Body)
	}{
		{
			name:       "Success case",
			queryParam: "id=1",
			wantCode:   http.StatusOK,
			wantBody:   `{"id":"1","name":"Admin"}`,
		},
		{
			name:       "Not found case",
			queryParam: "id=2",
			wantCode:   http.StatusNotFound,
			wantBody:   "not found",
		},
		{
			name:       "Missing param case",
			queryParam: "",
			wantCode:   http.StatusBadRequest,
			wantBody:   "missing id",
		},
	}

	// Chạy vòng lặp qua các ca
	for _, tt := range tests {
		// t.Run() giúp nhóm các ca kiểm thử
		t.Run(tt.name, func(t *testing.T) {
			// Tạo request
			req, err := http.NewRequest("GET", "/user?"+tt.queryParam, nil)
			if err != nil {
				t.Fatal(err)
			}

			// Tạo ResponseRecorder (một http.ResponseWriter mock)
			rr := httptest.NewRecorder()
			handler := http.HandlerFunc(GetUserHandler)

			// Gọi handler
			handler.ServeHTTP(rr, req)

			// Kiểm tra Status Code
			if status := rr.Code; status != tt.wantCode {
				t.Errorf("handler returned wrong status code: got %v want %v",
					status, tt.wantCode)
			}

			// Kiểm tra Body
			// Trim whitespace/newline mà http.Error thêm vào
			gotBody := strings.TrimSpace(rr.Body.String())
			
			// Nếu body mong đợi là JSON, cần so sánh cẩn thận hơn
			// (Ở đây chúng ta so sánh string đơn giản)
			if tt.wantBody == `{"id":"1","name":"Admin"}` {
				// Cần so sánh JSON chuẩn, nhưng ví dụ này dùng Contains
				if !strings.Contains(gotBody, `"id":"1"`) {
					 t.Errorf("handler returned unexpected body: got %v want (contains) %v",
                    gotBody, tt.wantBody)
				}
			} else if gotBody != tt.wantBody {
				t.Errorf("handler returned unexpected body: got %v want %v",
					gotBody, tt.wantBody)
			}
		})
	}
}
```

# 3. Thiết kế ứng dụng backend với Go

Phần này đi sâu vào xây dựng một microservice hoàn chỉnh.

## 3.1. Kiến trúc chuẩn (Clean Architecture

)

### Mô tả

Chúng ta áp dụng một biến thể của Clean Architecture (hoặc Hexagonal) để đảm bảo tính linh hoạt (testability) và dễ bảo trì.

- **So sánh:**
    
    - _Layered (N-Tier):_ Thường thấy (Controller, Service, DAO). Nhược điểm: Service dễ bị phình to, các lớp phụ thuộc lẫn nhau.
        
    - _Clean/Hexagonal:_ Tách biệt nghiệp vụ (Domain, Use Case) khỏi chi tiết kỹ thuật (Transport, Storage). Quy tắc phụ thuộc: bên ngoài (Infra) phải phụ thuộc vào bên trong (Domain).
        

### Chọn mẫu: Handler → Service → Repository

1. **Handler (lớp Transport/Infra):**
    
    - Chịu trách nhiệm về HTTP/gRPC.
        
    - Parse/validate request (binding).
        
    - Gọi `Service`.
        
    - Format response (JSON/gRPC).
        
    - _Không_ chứa logic nghiệp vụ.
        
2. **Service (lớp Use Case/Application):**
    
    - Chứa logic nghiệp vụ.
        
    - Điều phối `Repository` (và các service khác).
        
    - Quản lý transaction.
        
    - _Không_ biết về HTTP (không giữ `http.ResponseWriter`).
        
3. **Repository (lớp Interface/Infra):**
    
    - Định nghĩa `interface` (ví dụ: `UserRepository`) ở lớp `Domain` hoặc `Service`.
        
    - Implement `interface` (ví dụ: `PostgresUserRepository`) ở lớp `Infra/Repository`.
        
    - Chịu trách nhiệm truy cập dữ liệu (SQL, Redis, API ngoài).
        

### Dependency Inversion (Đảo ngược phụ thuộc)

`Handler` → `Service` → `UserRepository (Interface)`

`PostgresUserRepository (Implementation)` → `UserRepository (Interface)`

`Service` không biết về Postgres. Nó chỉ biết về `interface` mà nó cần.

## 3.2. HTTP API với chi tiết

### Mô tả

Xây dựng lớp API (REST) mạnh mẽ.

### Checklist

- [ ] Chọn router (bộ định tuyến) phù hợp.
    
- [ ] Triển khai middleware chuẩn (logging, recovery, CORS, Request ID).
    
- [ ] Validate input (đầu vào).
    
- [ ] Định dạng JSON response (phản hồi) nhất quán.
    

### Chọn Router

- `net/http` (stdlib): Tốt cho việc đơn giản, nhưng thiếu param-matching, middleware .
    
- `chi`: Rất tốt. Tương thích `net/http`, nhẹ, đủ tính năng (middleware, routing). **Đây là lựa chọn khuyến nghị.**
    
- `gin`: Phổ biến, nhanh, nhiều tính năng. Nhược điểm: context riêng, không hoàn toàn tương thích `net/http` thuần.
    
- `fiber`: Cực nhanh (dựa trên fasthttp). Nhược điểm: không tương thích `net/http`, API giống Express.js.
    

### Middleware

Middleware là các hàm "bọc" (wrap) một `http.Handler`.

```go
// Ví dụ: Middleware ghi log Request ID (giả sử đã có hàm xử lý Request ID)
func LoggerMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Lấy request ID từ context (do middleware trước đó gán)
		reqID := GetRequestID(r.Context())
		
		log.Info().Str("request_id", reqID).Str("path", r.URL.Path).Msg("Handling request")
		
		// Bọc ResponseWriter để bắt status code
		// (Chi tiết hơn, dùng thư viện)
		
		next.ServeHTTP(w, r)
		
		log.Info().Str("request_id", reqID).Msg("Handled request")
	})
}
```

### Validation

Dùng thư viện như `github.com/go-playground/validator`.

```go
type CreateUserRequest struct {
	Email    string `json:"email" validate:"required,email"`
	Password string `json:"password" validate:"required,min=8"`
}

// Trong Handler:
var validate = validator.New()

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
	var req CreateUserRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "invalid json", http.StatusBadRequest)
		return
	}

	if err := validate.Struct(req); err != nil {
		// Trả về lỗi validation rõ ràng
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	
	// ... gọi service ...
}
```

### Response Format (Envelope)

Luôn trả về một cấu trúc JSON nhất quán.

```go
type APIResponse struct {
	Data  interface{} `json:"data,omitempty"`
	Error *APIError   `json:"error,omitempty"`
	Meta  interface{} `json:"meta,omitempty"` // Dùng cho phân trang (pagination)
}

type APIError struct {
	Code    string `json:"code"` // Mã lỗi nghiệp vụ
	Message string `json:"message"`
}

// Hàm trợ giúp (helper)
func RespondSuccess(w http.ResponseWriter, code int, data interface{}) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(code)
	json.NewEncoder(w).Encode(APIResponse{Data: data})
}

func RespondError(w http.ResponseWriter, code int, errCode, message string) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(code)
	json.NewEncoder(w).Encode(APIResponse{Error: &APIError{Code: errCode, Message: message}})
}
```

## 3.3. Giao tiếp dịch vụ (Service-to-Service)

### Mô tả

Trong microservices, các dịch vụ cần nói chuyện với nhau.

- **REST (HTTP/JSON):**
    
    - Ưu điểm: Đơn giản, dễ debug, con người đọc được.
        
    - Nhược điểm: Chậm (text-based), tốn băng thông, không có hợp đồng (contract) mạnh.
        
- **gRPC (HTTP/2 + Protobuf):**
    
    - Ưu điểm: Hiệu năng cao (binary), contract rõ ràng (`.proto`), streaming, sinh code (code-gen).
        
    - Nhược điểm: Khó debug hơn (cần gRPC-UI, grpcurl), thiết lập phức tạp hơn.
        
    - **Khuyến nghị dùng gRPC cho giao tiếp nội bộ (backend-to-backend).**
        

### Ví dụ: gRPC (Protobuf)

Định nghĩa file `.proto`:

```go
// /proto/user/v1/user.proto
syntax = "proto3";

package user.v1;
option go_package = "[github.com/my-org/my-service/gen/proto/user/v1;userv1](https://github.com/my-org/my-service/gen/proto/user/v1;userv1)";

service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}

message User {
  string id = 1;
  string email = 2;
}

message GetUserRequest {
  string id = 1;
}

message GetUserResponse {
  User user = 1;
}
```

Sau khi chạy `protoc` (công cụ sinh code), bạn sẽ implement server và sử dụng client.

### Idempotency (Tính bất biến)

Khi gọi API (đặc biệt là `POST`, `PATCH`), nếu request bị lặp lại (ví dụ: do timeout, retry), làm sao để hệ thống không tạo 2 bản ghi?

- **Best Practice:** Client (bên gọi) tạo ra một `Idempotency-Key` (ví dụ: UUID) và gửi lên header.
    
- Server (bên nhận) lưu key này (vào DB hoặc Redis) cùng với kết quả.
    
- Nếu nhận được request với key đã tồn tại, server không xử lý lại mà trả về kết quả đã lưu.
    

## 3.4. Truy cập dữ liệu (Data Access)

### Mô tả

Tương tác với cơ sở dữ liệu (Postgres, Redis).

### Lựa chọn (Postgres)

1. `database/sql` + `pq`/`pgx`:
    
    - Tiêu chuẩn của Go. Linh hoạt.
        
    - Nhược điểm: Viết SQL bằng tay, dễ lỗi (typo), "scan" dữ liệu thủ công.
        
2. `GORM` (ORM):
    
    - Ưu điểm: Trừu tượng hóa SQL, nhanh (cho CRUD), tự động migration.
        
    - Nhược điểm: "Ma thuật", khó tối ưu query phức tạp, chậm hơn SQL thuần.
        
3. `sqlc` (Code-gen):
    
    - **Lựa chọn khuyến nghị (cân bằng).**
        
    - Bạn viết SQL thuần (trong file `.sql`).
        
    - `sqlc` sinh ra code Go an toàn (type-safe) để gọi các query đó.
        

### Migration

Dùng `golang-migrate` để quản lý thay đổi schema DB.

Tạo file migration: `.../db/migrations/000001_create_users_table.up.sql`

```sql
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### Transaction Pattern (Mô hình giao dịch)

Nghiệp vụ (Service) nên là bên quản lý transaction, không phải Repository.

```go
// Interface (trong domain)
type UserRepository interface {
    // Repository nhận vào DB executor (có thể là *sql.DB hoặc *sql.Tx)
	Create(ctx context.Context, exec *sql.Tx, user *User) error
	GetByID(ctx context.Context, exec *sql.DB, id string) (*User, error)
}

// Implementation (trong infra)
type PostgresUserRepo struct {
    // Không giữ *sql.Tx ở đây
}

func (r *PostgresUserRepo) Create(ctx context.Context, tx *sql.Tx, user *User) error {
    // tx.ExecContext...
    return nil
}
func (r *PostgresUserRepo) GetByID(ctx context.Context, db *sql.DB, id string) (*User, error) {
    // db.QueryRowContext...
    return nil, nil
}


// Service (lớp nghiệp vụ)
type UserService struct {
	db   *sql.DB
	repo UserRepository
}

func (s *UserService) RegisterUser(ctx context.Context, email, password string) (*User, error) {
	// 1. Bắt đầu Transaction
	tx, err := s.db.BeginTx(ctx, nil)
	if err != nil {
		return nil, fmt.Errorf("could not begin tx: %w", err)
	}
	// 2. Đảm bảo Rollback nếu có lỗi (panic hoặc return err)
	defer tx.Rollback() // Rollback sẽ bị bỏ qua nếu Commit() được gọi

	// 3. Tạo user (giả sử)
	user, err := NewUser(email, password)
	if err != nil {
		return nil, err // Lỗi validation
	}

	// 4. Truyền tx vào Repository
	if err := s.repo.Create(ctx, tx, user); err != nil {
		// (Kiểm tra lỗi duplicate email ở đây)
		return nil, fmt.Errorf("could not create user: %w", err)
	}
	
	// 5. (Tùy chọn) Ghi vào bảng outbox (nếu dùng Outbox Pattern)
	// if err := s.outboxRepo.Create(ctx, tx, ...); err != nil { ... }

	// 6. Commit Transaction
	if err := tx.Commit(); err != nil {
		return nil, fmt.Errorf("could not commit tx: %w", err)
	}

	return user, nil
}
```

### Lỗi thường gặp

- **Không set `context` timeout cho DB query:** Query bị treo có thể làm cạn kiệt connection pool. Luôn dùng `...Context()` (ví dụ: `db.QueryRowContext(ctx, ...)`).
    
- **Repository tự mở transaction:** Vi phạm nguyên tắc single-responsibility. Service mới là bên biết _khi nào_ cần gộp nhiều thao tác repo thành một transaction.
    

## 3.5. Cấu hình và Secrets

### Mô tả

Quản lý cấu hình (config) theo 12-Factor App (qua biến môi trường).

### Checklist

- [ ] Tách biệt config theo môi trường (local, staging, prod).
    
- [ ] Dùng thư viện (Viper) hoặc `godotenv` để load file `.env` (chỉ cho local dev).
    
- [ ] **KHÔNG BAO GIỜ** hardcode secret (password, API key) trong code.
    
- [ ] Đọc config từ biến môi trường (environment variables) khi chạy (prod).
    

### Ví dụ: Struct AppConfig và Init (dùng Viper)

`configs/config.go`

```go
package configs

import (
	"[github.com/spf13/viper](https://github.com/spf13/viper)"
	"strings"
)

type Config struct {
	Server ServerConfig
	DB     DBConfig
	JWT    JWTConfig
}

type ServerConfig struct {
	Port string `mapstructure:"port"`
}

type DBConfig struct {
	URL string `mapstructure:"url"`
}

type JWTConfig struct {
	Secret string `mapstructure:"secret"`
}

func LoadConfig() (*Config, error) {
	// 1. Đặt tên file (nếu có)
	viper.SetConfigName("config")    // Tên file (config.yml, config.json)
	viper.AddConfigPath("./configs") // Đường dẫn cho local
	viper.AddConfigPath(".")
	viper.SetConfigType("yaml")

	// 2. Đọc từ biến môi trường (ENV)
	// Cho phép ENV ghi đè file (ví dụ: DB_URL sẽ ghi đè db.url)
	viper.AutomaticEnv()
	viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))

	// 3. Đặt giá trị mặc định
	viper.SetDefault("server.port", "8080")

	var config Config

	// Thử đọc từ file (cho local)
	if err := viper.ReadInConfig(); err != nil {
		// Bỏ qua nếu không tìm thấy file, vì ENV là ưu tiên
		if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
			return nil, err
		}
	}
	
	// 4. Unmarshal vào struct
	if err := viper.Unmarshal(&config); err != nil {
		return nil, err
	}

	// 5. [TODO] Validate config (ví dụ: DB_URL không được rỗng)

	return &config, nil
}
```

# 4. Bảo mật (DevSec theo Go)

Tích hợp bảo mật vào quy trình phát triển.

## 4.1. Nguyên tắc

- **OWASP Top 10:**
    
    - A01 (Broken Access Control): Kiểm tra quyền (AuthZ) ở middleware/service.
        
    - A02 (Cryptographic Failures): Dùng TLS, hash password (Argon2, bcrypt).
        
    - A03 (Injection): Dùng prepared statements (tránh SQL injection), không dùng `exec` OS command.
        
    - A05 (Security Misconfiguration): Cấu hình CORS chặt, tắt debug mode ở prod.
        
- **Validate input, Escape output:** Không bao giờ tin tưởng dữ liệu từ client. Dùng `html/template` (thay vì `text/template`) để tự động escape HTML (tránh XSS).
    
- **Recovery Middleware:** Bắt `panic` và trả về lỗi 500 chung chung, không để lộ stacktrace.
    

## 4.2. AuthN/AuthZ (Xác thực / Ủy quyền)

### Mô tả

Dùng JWT (JSON Web Token) cho xác thực API.

### Checklist

- [ ] Dùng JWT với `exp` (expiry) ngắn (ví dụ: 15 phút).
    
- [ ] Cung cấp Refresh Token (lưu ở DB/Redis) để lấy JWT mới.
    
- [ ] Ký bằng `RS256` (dùng cặp public/private key) thay vì `HS256` (secret chung) nếu cần bảo mật cao hơn.
    
- [ ] Triển khai middleware kiểm tra (AuthN) và phân quyền (AuthZ).
    

### Ví dụ: Middleware kiểm tra Role (AuthZ)

Giả sử middleware AuthN (xác thực JWT) đã chạy trước và đặt `UserClaims` vào `context`.

```go
// Giả sử struct này được đặt vào context
type UserClaims struct {
	UserID string   `json:"user_id"`
	Roles  []string `json:"roles"`
}

// Context key (tránh dùng string thuần)
type contextKey string
const userClaimsKey contextKey = "userClaims"

// Helper lấy claims
func GetClaims(ctx context.Context) (*UserClaims, bool) {
	claims, ok := ctx.Value(userClaimsKey).(*UserClaims)
	return claims, ok
}

// CheckRole là một factory, trả về middleware kiểm tra role
func CheckRole(requiredRoles []string) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			claims, ok := GetClaims(r.Context())
			if !ok {
				// Lỗi này không nên xảy ra nếu AuthN middleware chạy đúng
				RespondError(w, http.StatusUnauthorized, "auth_required", "Authentication required")
				return
			}

			// Kiểm tra
			roleMap := make(map[string]bool)
			for _, role := range claims.Roles {
				roleMap[role] = true
			}

			for _, reqRole := range requiredRoles {
				if !roleMap[reqRole] {
					// Không đủ quyền
					RespondError(w, http.StatusForbidden, "forbidden", "Insufficient permissions")
					return
				}
			}

			// Đủ quyền, đi tiếp
			next.ServeHTTP(w, r)
		})
	}
}

// Cách dùng (với router 'chi'):
// r.With(CheckRole([]string{"admin"})).Post("/admin/users", ...)
```

## 4.3. An toàn giao tiếp

- **HTTPS/TLS:** Luôn dùng TLS (HTTPS) ở môi trường staging/prod. (Thường được xử lý ở Ingress/Load Balancer).
    
- **Verify HMAC:** Khi nhận webhook từ bên thứ 3 (ví dụ: Stripe, GitHub), phải kiểm tra chữ ký (signature) HMAC để đảm bảo request là hợp lệ.
    

## 4.4. An toàn file và Serialization

- **Deserialization:** Cẩn thận khi `json.Unmarshal` vào `interface{}`. Kẻ tấn công có thể gửi các cấu trúc JSON phức tạp (nested) gây DoS (cạn kiệt bộ nhớ/CPU). Luôn unmarshal vào `struct` cụ thể.
    
- **File Upload:** Giới hạn kích thước file, kiểm tra loại file (MIME type), không lưu file ở thư mục thực thi.
    

## 4.5. Static Analysis (Phân tích tĩnh)

### Mô tả

`golangci-lint` là công cụ bắt buộc. Nó chạy hàng chục linter khác nhau để tìm lỗi, lỗ hổng bảo mật, và anti-pattern.

### Ví dụ: Cấu hình `.golangci.yml`

Đây là file cấu hình cơ bản, rất quan trọng.

```yml
run:
  timeout: 5m
  skip-dirs:
    - gen # Bỏ qua thư mục code tự sinh (ví dụ: proto)
    - vendor

linters-settings:
  # Cấu hình cụ thể cho từng linter
  govet:
    check-shadowing: true
  gocyclo:
    min-complexity: 15 # Cảnh báo nếu hàm quá phức tạp
  goimports:
    local-prefixes: [github.com/my-org/my-service](https://github.com/my-org/my-service) # Tên module của bạn
  
  # Cấu hình bảo mật
  gosec:
    # G104: Unchecked errors (Cảnh báo lỗi không được kiểm tra)
    # G102: Insecure TLS (Kiểm tra TLS không an toàn)
    includes:
      - G101 # Hardcoded credentials
      - G102
      - G104
      - G110 # Potential DoS vulnerability (e.g. decompression bomb)

linters:
  # Tắt các linter không cần thiết
  disable-all: true
  # Bật các linter quan trọng
  enable:
    - govet
    - goimports
    - misspell
    - errcheck # Kiểm tra lỗi chưa được check (if err != nil)
    - staticcheck
    - unused
    - gosec # Linter bảo mật
    - gocyclo
    - bodyclose # Kiểm tra quên đóng response body
    - revive

issues:
  max-issues-per-linter: 0
  max-same-issues: 0
```

# 5. Observability và Vận hành

Ứng dụng phải có khả năng "quan sát" (O11y) để debug và giám sát.

## 5.1. Logging chuẩn

### Mô tả

Không dùng `fmt.Println()`. Dùng structured logging (log dưới dạng JSON).

- **Lựa chọn:** `zerolog` (nhanh nhất), `zap` (phổ biến), `logrus` (cũ hơn). **Khuyến nghị: `zerolog`.**
    

### Checklist

- [ ] Log ra `stdout/stderr` (12-Factor App).
    
- [ ] Log định dạng JSON (để ELK/Loki parse).
    
- [ ] **Field bắt buộc:**
    
    - `ts` (timestamp)
        
    - `level` (debug, info, error)
        
    - `msg` (message)
        
    - `service` (tên service, ví dụ: "user-service")
        
    - `trace_id`, `span_id` (nếu có, từ OpenTelemetry)
        
- [ ] **Mask (che)** dữ liệu nhạy cảm (password, token) trước khi log.
    

### Ví dụ: Logger Middleware (với zerolog)

```go
package main

import (
	"context"
	"net/http"
	"os"
	"time"
	"[github.com/rs/zerolog](https://github.com/rs/zerolog)"
	"[github.com/rs/zerolog/log](https://github.com/rs/zerolog/log)"
)

// Khởi tạo logger toàn cục (ở main)
func InitLogger() {
	// [TODO: Cấu hình level (debug/info) theo ENV]
	// [TODO: Masking (che) dữ liệu]
	log.Logger = zerolog.New(os.Stdout).With().
		Timestamp().
		Str("service", "my-user-service"). // Tên service
		Logger()
}

// AddLoggerToContext đưa logger (đã có field) vào context
func AddLoggerToContext(ctx context.Context, logger zerolog.Logger) context.Context {
	return logger.WithContext(ctx)
}

// GetLogger từ context, nếu không có trả về logger mặc định
func GetLogger(ctx context.Context) zerolog.Logger {
	return log.Ctx(ctx)
}


// LoggerMiddleware (cho HTTP)
func LoggerMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()

		// Tạo logger con cho request này
		// [TODO: Lấy trace_id từ header/context nếu dùng OpenTelemetry]
		reqLogger := log.With().
			Str("method", r.Method).
			Str("path", r.URL.Path).
			Logger()
		
		// Đưa logger vào context để các hàm (service, repo) có thể dùng
		ctx := AddLoggerToContext(r.Context(), reqLogger)
		r = r.WithContext(ctx)

		// [TODO: Bọc response writer để lấy status code]
		// ...
		
		next.ServeHTTP(w, r)
		
		duration := time.Since(start)
		
		// Log khi request hoàn thành
		reqLogger.Info().
			// Int("status_code", ...). // Lấy từ response writer đã bọc
			Dur("duration_ms", duration).
			Msg("HTTP request completed")
	})
}

// Cách dùng trong service:
// func (s *UserService) GetUser(ctx context.Context, id string) {
//     logger := GetLogger(ctx)
//     logger.Info().Str("user_id", id).Msg("Fetching user")
//     // ...
// }
```

## 5.2. Metrics

### Mô tả

Đo lường các chỉ số (ví dụ: số lượng request, thời gian xử lý) và export ra cho **Prometheus**.

### Checklist

- [ ] Dùng `prometheus/client_golang`.
    
- [ ] Export endpoint `/metrics`.
    
- [ ] **Các chỉ số quan trọng (RED):**
    
    - `Counter`: Đếm số lượng request (`http_requests_total`).
        
    - `Histogram`: Đo lường thời gian xử lý (`http_request_duration_seconds`).
        
    - `Counter`: Đếm số lỗi (`http_requests_errors_total`).
        

### Ví dụ

```go
import (
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promauto"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

// Định nghĩa metrics (toàn cục)
var (
	httpRequestsTotal = promauto.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Total number of HTTP requests.",
		},
		[]string{"method", "path", "status_code"}, // labels
	)
	
	httpRequestDuration = promauto.NewHistogramVec(
		prometheus.HistogramOpts{
			Name: "http_request_duration_seconds",
			Help: "Duration of HTTP requests.",
			// Buckets (ms): 50, 100, 250, 500, 1000, 2500, 5000
			Buckets: []float64{0.05, 0.1, 0.25, 0.5, 1, 2.5, 5},
		},
		[]string{"method", "path"},
	)
)

// Trong main.go, đăng ký handler
// r.Handle("/metrics", promhttp.Handler())

// Trong middleware (sau khi xử lý xong):
// statusCode := ... (lấy từ response writer)
// httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, fmt.Sprint(statusCode)).Inc()
// httpRequestDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration.Seconds())
```

## 5.3. Tracing (Truy vết)

### Mô tả

Theo dõi một request khi nó đi qua nhiều microservices. **Tiêu chuẩn: OpenTelemetry (OTel).**

### Checklist

- [ ] Tích hợp OTel SDK.
    
- [ ] Tự động tạo `span` cho HTTP request (dùng OTel middleware).
    
- [ ] Tự động tạo `span` cho DB query (dùng OTel-instrumented DB driver).
    
- [ ] Truyền `trace_id` qua context và log (xem 5.1).
    

_Ghi chú: Cài đặt OTel tương đối phức tạp, thường cần một thư viện wrapper riêng của công ty._

## 5.4. Healthcheck và Readiness

### Mô tả

Giúp Kubernetes (K8s) biết khi nào service còn "sống" và khi nào "sẵn sàng" nhận request.

- `/healthz` (Liveness Probe): Kiểm tra service có bị "chết" (treo) không. Chỉ cần trả về `200 OK`. Nếu probe này thất bại, K8s sẽ restart container.
    
- `/readyz` (Readiness Probe): Kiểm tra service có _sẵn sàng_ nhận traffic mới không. Phải kiểm tra các kết nối phụ thuộc (DB, Cache, service khác). Nếu thất bại, K8s sẽ ngừng gửi traffic đến container này.
    

### Ví dụ

```go
func HealthzHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("OK"))
}

// Giả sử service có biến *sql.DB
func (s *MyService) ReadyzHandler(w http.ResponseWriter, r *http.Request) {
	// Kiểm tra DB
	ctx, cancel := context.WithTimeout(r.Context(), 1*time.Second) // Đặt timeout
	defer cancel()

	if err := s.db.PingContext(ctx); err != nil {
		log.Error().Err(err).Msg("Readiness probe failed (DB ping)")
		http.Error(w, "DB not ready", http.StatusServiceUnavailable)
		return
	}
	
	// [TODO: Kiểm tra Redis, Kafka, ...]

	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Ready"))
}
```

## 5.5. Graceful Shutdown (Tắt ứng dụng an toàn)

### Mô tả

Khi K8s (hoặc user) gửi tín hiệu `SIGTERM` (tín hiệu yêu cầu tắt), service không được tắt ngay lập tức. Nó phải:

1. Ngừng nhận request mới (Readiness probe thất bại).
    
2. Hoàn thành các request đang xử lý.
    
3. Đóng các kết nối (DB, Kafka) một cách an toàn.
    

### Ví dụ: Cấu trúc `main.go` chuẩn

```go
package main

import (
	"context"
	"errors"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
	"[github.com/rs/zerolog/log](https://github.com/rs/zerolog/log)"
	// ... (import router, config, service, ...)
)

func main() {
	// [TODO: Init Logger (5.1), Load Config (3.5)]
	// [TODO: Init DB, Init Service, Init Repo, Init Handler]
	// [TODO: Init Router (chi), đăng ký middleware (Log, Recovery, Auth)]
	// [TODO: Đăng ký /healthz, /readyz]

	// Ví dụ router (giả)
	router := http.NewServeMux() 
	router.HandleFunc("/healthz", HealthzHandler)
	// ...

	// Cấu hình HTTP Server
	srv := &http.Server{
		Addr:    ":8080", // [TODO: Lấy từ Config]
		Handler: router,
		// [TODO: Set Read/Write/Idle Timeouts]
	}

	// 1. Chạy server trong một goroutine
	go func() {
		log.Info().Str("addr", srv.Addr).Msg("Starting server...")
		if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			log.Fatal().Err(err).Msg("Server startup failed")
		}
	}()

	// 2. Tạo channel lắng nghe tín hiệu OS
	quit := make(chan os.Signal, 1)
	// Lắng nghe SIGINT (Ctrl+C) và SIGTERM (từ K8s)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

	// 3. Block main goroutine cho đến khi nhận được tín hiệu
	sig := <-quit
	log.Info().Str("signal", sig.String()).Msg("Received signal, starting graceful shutdown")

	// 4. Tạo context timeout cho shutdown
	// (K8s mặc định cho 30s)
	ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
	defer cancel()

	// 5. [TODO] Tắt Readiness Probe (để K8s ngừng gửi traffic)
	// (Cách đơn giản là dùng một biến atomic)

	// 6. Gọi Shutdown() cho HTTP server
	// (Nó sẽ chờ các request đang xử lý hoàn thành)
	if err := srv.Shutdown(ctx); err != nil {
		log.Error().Err(err).Msg("Server shutdown failed")
	} else {
		log.Info().Msg("Server shut down gracefully")
	}

	// 7. [TODO] Đóng các kết nối khác (DB, Redis)
	// db.Close()
	
	log.Info().Msg("Shutdown complete")
}
```

# 6. DevOps / DevSecOps cho Go service

Quy trình tự động hóa build, test, và deploy.

## 6.1. Build & Release

### Mô tả

Build Go ra tệp nhị phân (binary) tĩnh.

### Checklist

- [ ] Dùng `CGO_ENABLED=0` để đảm bảo build tĩnh (static binary), trừ khi cần CGO.
    
- [ ] Dùng `GOOS` và `GOARCH` để build chéo (ví dụ: build trên Mac (darwin/amd64) cho Linux (linux/amd64)).
    
- [ ] **Nhúng (embed) thông tin phiên bản (version)** vào tệp nhị phân dùng `ldflags`.
    

### Ví dụ: Lệnh Build

Giả sử có biến `main.Version`.

```bash
# Lấy Git commit hash
VERSION=$(git rev-parse --short HEAD)

# Build
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags="-w -s -X 'main.Version=${VERSION}'" \
    -o ./build/my-service \
    ./cmd/server/main.go

# -w -s: Tối ưu size (bỏ DWARF và symbol table)
# -X 'main.Version=...': Ghi đè biến Version trong package main
```

## 6.2. Docker hóa (Dockerization)

### Mô tả

Đóng gói ứng dụng vào container.

### Checklist

- [ ] Dùng **Multi-stage build** (bắt buộc) để giữ image nhỏ gọn và an toàn.
    
- [ ] Chạy container với **non-root user**.
    
- [ ] Load config qua biến môi trường (ENV) lúc runtime.
    

### Ví dụ: Dockerfile tối ưu

`/deploy/docker/Dockerfile`

```dockerfile
# --- Stage 1: Builder ---
# Dùng image Go chính thức (alpine là bản nhỏ gọn)
FROM golang:1.21-alpine AS builder

# Set thư mục làm việc
WORKDIR /app

# Copy go.mod và go.sum TRƯỚC để tận dụng Docker layer caching
COPY go.mod go.sum ./
RUN go mod download

# Copy toàn bộ source code
COPY . .

# [TODO: Lấy version (xem 6.1)]
ARG VERSION=dev

# Build ứng dụng
# CGO_ENABLED=0 và -installsuffix để build tĩnh
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-w -s -X 'main.Version=${VERSION}'" \
    -o /my-service \
    ./cmd/server/main.go


# --- Stage 2: Final Image ---
# Dùng image "trống" (scratch) hoặc alpine (nếu cần shell/tools)
# "distroless" của Google là lựa chọn tốt
FROM gcr.io/distroless/static-debian11 AS final
# HOẶC: FROM scratch

WORKDIR /

# Copy tệp nhị phân từ stage "builder"
COPY --from=builder /my-service /my-service

# [TODO] Copy các file tĩnh nếu cần (ví dụ: /configs/*.yml)
# COPY --from=builder /app/configs /configs

# (Tùy chọn: Thêm non-root user)
# USER nonroot:nonroot
# (Nếu dùng 'scratch', bạn phải tự tạo user ở stage 1.
#  Nếu dùng 'distroless', user 'nonroot' (65532) đã có sẵn)
USER 65532

# Port mà ứng dụng sẽ chạy (chỉ mang tính thông báo)
EXPOSE 8080

# Lệnh chạy khi container khởi động
ENTRYPOINT ["/my-service"]
```

## 6.3. CI/CD Pipeline

### Mô tả

Quy trình tự động khi code được push lên (ví dụ: GitHub Actions).

### Pipeline chuẩn

1. **Lint:** Chạy `golangci-lint` (xem 4.5).
    
2. **Test:** Chạy `go test ./... -cover` (Unit test).
    
3. **Build:** Chạy `go build` (kiểm tra code biên dịch được).
    
4. **Scan (Phân tích):**
    
    - `go vulncheck` (kiểm tra lỗ hổng thư viện).
        
    - `trivy` (quét bí mật, SAST).
        
5. **(Nếu là main/master):** Build Docker image.
    
6. **(Nếu là main/master):** Push image lên (AWS ECR, GCP GCR, ...).
    
7. **(Nếu là main/master):** Deploy (ví dụ: cập nhật K8s manifest).
    

### Ví dụ: GitHub Actions (YAML)

`.github/workflows/ci.yml`

```yml
name: Go CI/CD

on:
  push:
    branches: [ "main", "develop" ]
  pull_request:
    branches: [ "main", "develop" ]

jobs:
  test-lint:
    name: Test & Lint
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21' # [TODO: Đặt phiên bản Go của dự án]

      - name: Run golangci-lint
        uses: golangci/golangci-lint-action@v4
        with:
          version: v1.55 # [TODO: Đặt phiên bản linter]
          args: --timeout=5m
          # Chỉ chạy trên các file đã thay đổi (cho PR)
          only-new-issues: ${{ github.event_name == 'pull_request' }}

      - name: Run Unit Tests
        run: go test -v -race -coverprofile=coverage.out ./...

      - name: Run Go Vuln Check (Security)
        # Quét lỗ hổng trong các thư viện đang dùng
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...
  
  build-image:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    # Chỉ chạy khi push lên 'main'
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: test-lint # Chạy sau khi test-lint thành công
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub # [TODO: Thay bằng Registry của công ty, ví dụ ECR, GCR]
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./deploy/docker/Dockerfile
          push: true
          tags: my-org/my-service:latest, my-org/my-service:${{ github.sha }}
          cache-from: type=inline
          cache-to: type=inline,mode=max

  # [TODO] Thêm job 'deploy' (ví dụ: dùng kubectl, helm, argocd)
```

## 6.4. Security trong Pipeline (DevSecOps)

### Mô tả

Tích hợp quét bảo mật vào CI.

### Checklist

- [ ] **Trivy (Scan Image):** Quét lỗ hổng (CVE) trong image Docker (quét các gói của HĐH và thư viện).
    
- [ ] **go vulncheck:** (Như ví dụ 6.3) Quét lỗ hổng trong các thư viện Go.
    
- [ ] **Detect Secrets:** Quét tìm bí mật (API key, password) bị hardcode.
    

_Ghi chú: Thêm bước Trivy vào GitHub Actions (sau khi build image)._

## 6.5. Triển khai (Kubernetes)

### Mô tả

Chạy ứng dụng trên K8s.

### Checklist

- [ ] `Deployment`: Định nghĩa cách chạy (image, số replica).
    
- [ ] `Service`: Mở port của `Deployment` ra (ClusterIP, LoadBalancer).
    
- [ ] `ConfigMap`: Cung cấp config (không chứa secret).
    
- [ ] `Secret`: Cung cấp secret (DB password, JWT secret).
    
- [ ] **BẮT BUỘC:** Đặt `resources.requests` và `resources.limits` (CPU, Mem) cho container.
    
- [ ] Dùng `RollingUpdate` (mặc định) để deploy không downtime.
    

### Ví dụ: K8s Deployment tối thiểu

`/deploy/k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-user-service
  labels:
    app: my-user-service
spec:
  replicas: 2 # [TODO: Chỉnh theo môi trường]
  selector:
    matchLabels:
      app: my-user-service
  strategy:
    type: RollingUpdate # Cập nhật cuốn chiếu
  template:
    metadata:
      labels:
        app: my-user-service
    spec:
      containers:
      - name: server
        image: my-org/my-service:latest # [TODO: Dùng tag cụ thể, không dùng latest]
        ports:
        - containerPort: 8080
        
        # --- Cấu hình Healthcheck (RẤT QUAN TRỌNG) ---
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /readyz
            port: 8080
          initialDelaySeconds: 10 # Chờ lâu hơn liveness
          periodSeconds: 5

        # --- Cấu hình Resource (BẮT BUỘC) ---
        resources:
          # Yêu cầu tối thiểu
          requests:
            cpu: "100m" # 0.1 core
            memory: "128Mi"
          # Giới hạn tối đa
          limits:
            cpu: "500m" # 0.5 core
            memory: "256Mi"

        # --- Cấu hình ENV từ ConfigMap và Secret ---
        envFrom:
        - configMapRef:
            name: my-user-service-config # Chứa config (ví dụ: LOG_LEVEL)
        - secretRef:
            name: my-user-service-secret # Chứa (ví dụ: DB_URL, JWT_SECRET)
```

## 6.6. Logging tập trung

Log (từ 5.1) được viết ra `stdout`. Hệ thống (K8s, Docker) sẽ thu thập chúng và đẩy về một nơi tập trung (ELK Stack, Loki, Splunk).

# 7. Các chủ đề nâng cao (Senior)

## 7.1. Concurrency nâng cao

- **Worker Pool linh hoạt:** Dùng `context` và `sync.WaitGroup` để quản lý pool (như ví dụ 2.4).
    
- **Fan-in, Fan-out:** Pattern phân chia công việc (fan-out) cho nhiều worker và gộp kết quả (fan-in).
    
- **Tránh leak Goroutine:** Đảm bảo mọi goroutine đều có đường thoát (qua `ctx.Done()` hoặc `close(channel)`).
    

## 7.2. Performance Tuning (Tinh chỉnh hiệu năng)

### Mô tả

Dùng `pprof` (công cụ profiling của Go) để tìm điểm nóng (hotspot).

### Checklist

- [ ] Import `net/http/pprof` (chỉ cho server debug) để xem pprof qua HTTP.
    
- [ ] Dùng `go tool pprof` để phân tích:
    
    - **CPU:** Tìm hàm tốn thời gian CPU.
        
    - **Heap (Mem):** Tìm đối tượng chiếm nhiều bộ nhớ.
        
    - **Block:** Tìm nơi goroutine bị block (chờ lock, I/O).
        
- [ ] **Tối ưu Allocation:** Giảm số lần cấp phát bộ nhớ (ví dụ: dùng `sync.Pool`, `strings.Builder` thay vì `+` khi ghép chuỗi).
    

## 7.3. Event-Driven (Kiến trúc hướng sự kiện)

### Mô tả

Giao tiếp bất đồng bộ qua message broker (Kafka, NATS, RabbitMQ).

### Outbox Pattern

Vấn đề: Làm sao để đảm bảo `Commit DB` và `Gửi sự kiện Kafka` thành công (hoặc thất bại) cùng lúc?

- **Giải pháp (Outbox):**
    
    1. Trong cùng một transaction DB (xem 3.4):
        
        - `INSERT` vào bảng `users`.
            
        - `INSERT` vào bảng `outbox_events` (nội dung: "user_created", data: ...).
            
    2. `COMMIT` transaction.
        
    3. Một service riêng (hoặc goroutine) theo dõi bảng `outbox_events` (dùng polling hoặc `LISTEN/NOTIFY` của Postgres).
        
    4. Khi thấy event mới, nó đọc, gửi lên Kafka.
        
    5. Nếu gửi Kafka thành công, nó xóa/đánh dấu event trong bảng `outbox_events`.
        
- _Lợi ích: Đảm bảo "Exactly-once" (hoặc "At-least-once") delivery._
    

## 7.4. Multi-module / Monorepo Go

- **Go Work:** Dùng `go.work` file ở thư mục gốc để làm việc đồng thời trên nhiều module (ví dụ: `my-service` và `shared-pkg`).
    

## 7.5. Design Pattern áp dụng trong Go

- **Options Pattern:** Dùng "Functional Options" để khởi tạo các `struct` phức tạp một cách linh hoạt.
    
    ```go
    // Ví dụ: Server có nhiều tùy chọn
    type Server struct {
        port    int
        timeout time.Duration
    }
    type Option func(*Server)
    
    func WithPort(port int) Option {
        return func(s *Server) { s.port = port }
    }
    func WithTimeout(t time.Duration) Option {
        return func(s *Server) { s.timeout = t }
    }
    
    // Constructor
    func NewServer(opts ...Option) *Server {
        // Giá trị mặc định
        s := &Server{port: 8080, timeout: 5 * time.Second}
    
        // Áp dụng các tùy chọn
        for _, opt := range opts {
            opt(s)
        }
        return s
    }
    
    // Cách dùng:
    // s1 := NewServer() // Dùng mặc định
    // s2 := NewServer(WithPort(9000)) // Ghi đè port
    ```
    

## 7.6. Testing nâng cao

- **Integration Test:**
    
    - Dùng `testcontainers-go` để khởi chạy Docker container (Postgres, Redis) ngay trong code test.
        
    - Test chạy trên DB/Redis thật, đảm bảo tính toàn vẹn.
        
- **Mocking:** Dùng `gomock` hoặc `testify/mock` để mock các `interface` (ví dụ: mock `UserRepository` khi test `UserService`).
    

# 8. Lỗi thường gặp và Anti-Pattern

- **Global Mutable State:** Dùng biến toàn cục và bị thay đổi (mutable). Đây là nguồn gốc của race condition. Tránh tuyệt đối.
    
- **Không đóng `resp.Body`:** Khi gọi HTTP (dùng `http.Client`), quên `defer resp.Body.Close()`. Gây rò rỉ (leak) file descriptor và connection.
    
- **Không set timeout ở HTTP Client:** `http.DefaultClient` không có timeout. Một request ra ngoài bị treo sẽ làm treo hệ thống. Luôn tạo client riêng với timeout.
    
- **Bỏ qua `context` (Context Ignored):** Nhận `ctx` vào nhưng không truyền xuống (ví dụ: gọi `db.Query()` thay vì `db.QueryContext(ctx, ...)`).
    
- **Panic trong Goroutine:** `panic` trong một goroutine (không phải `main`) sẽ làm sập toàn bộ chương trình. Luôn dùng `recover` ở đầu goroutine nếu không chắc chắn.
    
- **Struct Tag `json` sai:** Sai cú pháp (ví dụ: `json:"id, omitempty"`) thay vì (`json:"id,omitempty"`) làm (de)serialize thất bại.
    
- **Kết nối DB không Pool:** Mở/đóng kết nối DB cho mỗi request. Phải dùng `sql.DB` (nó là một connection pool) và giữ nó sống suốt vòng đời ứng dụng.
    

# 9. Phụ lục

## 9.1. Checklist Review Code Go

- **Tính đúng đắn (Correctness):**
    
    - [ ] Logic có đúng yêu cầu?
        
    - [ ] Có xử lý hết các trường hợp (edge case)?
        
    - [ ] Có xử lý lỗi (check `err != nil`) ở mọi nơi?
        
- **Bảo mật (Security):**
    
    - [ ] Input (từ user/API) đã được validate?
        
    - [ ] Có nguy cơ SQL injection (nếu không dùng prepared statement)?
        
    - [ ] Dữ liệu nhạy cảm có bị log ra không?
        
- **Concurrency:**
    
    - [ ] Có nguy cơ race condition? (Dùng `go test -race`).
        
    - [ ] Goroutine có bị leak không?
        
    - [ ] `context.Context` có được truyền và tôn trọng không?
        
- **Kiến trúc (Architecture):**
    
    - [ ] Code có đặt đúng lớp không (Handler, Service, Repo)?
        
    - [ ] Có vi phạm Dependency Rule (ví dụ: Service biết về HTTP)?
        
- **Hiệu năng (Performance):**
    
    - [ ] Có cấp phát bộ nhớ (allocation) không cần thiết trong vòng lặp?
        
    - [ ] Có dùng `defer` trong vòng lặp lớn (gây tốn bộ nhớ)?
        
- **Khả năng đọc (Readability):**
    
    - [ ] Tên biến/hàm rõ ràng?
        
    - [ ] Hàm có quá dài (vi phạm Single Responsibility)?
        
- **Testing:**
    
    - [ ] Unit test có đủ (table-driven)?
        
    - [ ] Test có thất bại khi code lỗi không?
        

## 9.2. Template Project (Cấu trúc thư mục)

(Tham khảo lại mục 2.1)

```
/my-service
├── /cmd
│   └── /server
│       └── main.go
├── /internal
│   ├── /domain       # Structs, Interfaces (ví dụ: User, UserRepository)
│   ├── /handler      # HTTP/gRPC Handlers
│   ├── /service      # Business Logic (ví dụ: UserService)
│   └── /repository   # DB/Cache Implementation (ví dụ: PostgresUserRepo)
├── /pkg              # (Hạn chế dùng)
├── /configs          # config.yml.example
├── /deploy
│   ├── /docker
│   │   └── Dockerfile
│   ├── /k8s
│   │   └── deployment.yaml
│   └── /migrations   # SQL Migrations
│       └── 000001_init.up.sql
├── go.mod
├── .golangci.yml
└── README.md
```

## 9.3. Tài liệu tham khảo

- [Go Tour](https://tour.golang.org/ "null")
    
- [Effective Go](https://golang.org/doc/effective_go.html "null")
    
- [Go Standard Project Layout](https://github.com/golang-standards/project-layout "null")
    
- [Zerolog (Logging)](https://github.com/rs/zerolog "null")
    
- [Chi (Router)](https://github.com/go-chi/chi "null")
    
- [Testcontainers-Go (Integration Test)](https://golang.testcontainers.org/ "null")
    
- [GolangCI-Lint](https://golangci-lint.run/ "null")