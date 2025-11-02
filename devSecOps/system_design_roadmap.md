# Lộ Trình Chi Tiết: Từ Monolith Sang Microservice - DevSecOps

## PHẦN 1: MONOLITH - NỀN TẢNG CẦN HIỂU

### 1.1 Đánh Giá & Phân Tích Monolith Hiện Tại

**Cách Tư Duy:**
- Không vội migrate - phải hiểu rõ kiến trúc hiện tại
- Map tất cả dependencies giữa các components
- Xác định bounded contexts (domain-driven design)
- Đo lường performance baseline

**Ví Dụ: E-commerce Monolith**
```
Components:
├── User Management
├── Product Catalog
├── Order Processing
├── Payment
├── Inventory
├── Shipping
└── Notification

Database: Single PostgreSQL (shared schema)
Deployment: Single WAR/JAR file
```

**Câu Hỏi Cần Trả Lời:**
1. Thành phần nào có coupling tight nhất?
2. Thành phần nào scale khác nhau?
3. Thành phần nào fail independently được?
4. Thành phần nào cần deploy riêng biệt?

### 1.2 Xác Định Service Boundaries

**Phương Pháp: Domain-Driven Design (DDD)**
- Aggregate: Nhóm entities trong cùng transaction boundary
- Bounded Context: Ranh giới giữa các domain riêng biệt
- Ubiquitous Language: Thống nhất terminology

**Ví Dụ Boundary**
```
Order Service:
- Aggregate: Order (order_id, items[], status)
- Dependencies: Product (read-only), Payment (call), Inventory (call)
- Database: order_db
- Workflow: Order → Payment → Inventory → Notification
```

**Tiêu Chí Chọn Service Đầu Tiên Để Migrate:**
1. ✅ Ít dependencies (edge service)
2. ✅ Business value cao
3. ✅ Clear boundaries
4. ❌ Không phải critical path (tiếp tục hoạt động khi fail)

Ví dụ tốt: **Notification Service** → **User Service** → **Product Service**

---

## PHẦN 2: STRANGLER FIG PATTERN - EXTRACTION AN TOÀN

### 2.1 Transform - Xây Dựng Service Đầu Tiên

**Cách Tư Duy:**
- Build song song với monolith
- Create anti-corruption layer (ACL) để translate domain models
- Test tính đúng đắn bằng cách call cả monolith và service mới

**Ví Dụ: Extract User Authentication Service**

**Bước 1: Tạo Service Mới**
```go
// user-service/main.go
package main

import (
    "github.com/gofiber/fiber/v2"
    "database/sql"
)

func main() {
    app := fiber.New()
    
    // Service riêng database
    db, _ := sql.Open("postgres", "postgres://user-service-db:5432/users")
    
    // API endpoints
    app.Post("/auth/register", RegisterUser)
    app.Post("/auth/login", LoginUser)
    app.Get("/auth/validate/:token", ValidateToken)
    
    app.Listen(":3001")
}

func LoginUser(c *fiber.Ctx) error {
    // Local transaction trong user-service
    // Return JWT token
    return c.JSON(fiber.Map{"token": "jwt_token"})
}
```

**Bước 2: API Gateway/Facade Layer**
```nginx
# nginx.conf - Facade router
upstream monolith {
    server monolith:8080;
}

upstream user_service {
    server user-service:3001;
}

server {
    listen 80;
    
    # Route auth requests to new service
    location /auth/ {
        proxy_pass http://user_service;
    }
    
    # Fallback lại monolith
    location / {
        proxy_pass http://monolith;
    }
}
```

**Bước 3: Anti-Corruption Layer (ACL) - Translate Model**
```go
// monolith-side ACL
func getUserFromNewService(userID string) (*User, error) {
    // Call new user-service
    resp, _ := http.Get("http://user-service/users/" + userID)
    
    var newServiceUser struct {
        ID    string `json:"id"`
        Email string `json:"email"`
    }
    json.NewDecoder(resp.Body).Decode(&newServiceUser)
    
    // Convert to monolith domain model
    return &User{
        ID:    newServiceUser.ID,
        Email: newServiceUser.Email,
    }, nil
}
```

### 2.2 Coexist - Song Song Cùng Monolith

**Cách Tư Duy:**
- Cả 2 service phải hoạt động đúng
- Sync data giữa monolith (write) → service mới (read)
- Kiểm tra tính consistency

**Strategy 1: Dual Write (High Risk - Tránh Nếu Có Thể)**
```go
// In monolith - write to both
func CreateOrder(order *Order) error {
    // Write to monolith DB
    monolithDB.InsertOrder(order)
    
    // Write to new Order Service DB (risky)
    orderServiceClient.CreateOrder(order)
    
    // Problem: Inconsistency nếu 1 fails
    return nil
}
```

**Strategy 2: Event-Driven Sync (Recommended)**
```go
// Monolith publishes event
func CreateOrder(order *Order) error {
    // Write to monolith DB
    monolithDB.InsertOrder(order)
    
    // Publish event
    eventBus.Publish("order.created", OrderCreatedEvent{
        OrderID: order.ID,
        UserID: order.UserID,
        Items: order.Items,
    })
    
    return nil
}

// Order Service subscribes
func (os *OrderService) OnOrderCreated(event OrderCreatedEvent) {
    os.db.InsertOrder(Order{
        ID: event.OrderID,
        UserID: event.UserID,
        Items: event.Items,
    })
}
```

### 2.3 Eliminate - Tắt Monolith Feature

**Cách Tư Duy:**
- Chỉ tắt khi 100% confident
- Giữ feature trong monolith vào thế "read-only"
- Redirect toàn bộ traffic đến service mới

```go
// Monolith - deprecated
@Deprecated("Use new Order Service")
func GetOrder(c *fiber.Ctx) error {
    orderID := c.Params("id")
    
    // Proxy request to new service
    resp, _ := http.Get("http://order-service/orders/" + orderID)
    return c.JSON(resp)
}

// After full migration - remove feature từ monolith
```

---

## PHẦN 3: DATA MANAGEMENT - KHỚP KHÚC KHÁI KHÁI

### 3.1 Database per Service Pattern

**Vấn Đề Cốt Lõi:**
- Monolith: 1 DB chung → shared tables → tight coupling
- Microservices: Each service owns data → loose coupling BUT distributed transactions phức tạp

**Ví Dụ Decomposition:**
```
Monolith DB (PostgreSQL):
├── users table
├── products table
├── orders table
├── order_items table
├── payments table
└── inventory table

↓↓↓ After Migration ↓↓↓

User Service DB:     users, user_profiles
Product Service DB:  products, categories
Order Service DB:    orders, order_items
Payment Service DB:  payments, transactions
Inventory Service DB: inventory, stock_history
```

**Kỹ Thuật: Strangler Data Migration**
```sql
-- Phase 1: Create schema in new service DB
CREATE TABLE orders_new (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    status VARCHAR(50),
    total_amount DECIMAL(10,2),
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Phase 2: Dual write (application logic)
-- Phase 3: Sync historical data
INSERT INTO orders_new 
SELECT * FROM monolith.orders;

-- Phase 4: Verify consistency
SELECT COUNT(*) FROM orders_new 
WHERE created_at > NOW() - INTERVAL '1 hour';

-- Phase 5: Cutover (switch reads)
-- Phase 6: Stop writes to monolith orders
-- Phase 7: Decommission old table
```

### 3.2 Distributed Transactions - Saga Pattern

**Vấn Đề:**
- Monolith: 1 transaction = all or nothing (ACID)
- Microservices: Mỗi service transaction riêng → distributed transactions needed

**Use Case: Order Processing Flow**
```
Flow: Order → Payment → Inventory Update

❌ KHÔNG THỂ LÀM:
BEGIN TRANSACTION
  INSERT INTO orders
  UPDATE inventory
  INSERT INTO payments
COMMIT; -- Across multiple databases = KHÔNG POSSIBLE

✅ GIẢI PHÁP: Saga Pattern
```

**Saga Pattern - 2 Kiến Trúc:**

#### A. Choreography (Event-Driven, Decentralized)
```go
// Order Service
func CreateOrder(order *Order) error {
    order.Status = "PENDING"
    os.db.Save(order)
    
    // Publish event
    eventBus.Publish("order.created", OrderCreatedEvent{
        OrderID: order.ID,
        UserID: order.UserID,
        Amount: order.TotalAmount,
    })
    return nil
}

// Payment Service (subscribes to order.created)
func OnOrderCreated(event OrderCreatedEvent) {
    payment := Payment{OrderID: event.OrderID, Amount: event.Amount}
    
    if err := ps.processPayment(payment); err != nil {
        // Compensating action: Emit event to cancel order
        eventBus.Publish("payment.failed", PaymentFailedEvent{
            OrderID: event.OrderID,
        })
        return
    }
    
    eventBus.Publish("payment.completed", PaymentCompletedEvent{
        OrderID: event.OrderID,
    })
}

// Order Service (subscribes to payment.completed & payment.failed)
func OnPaymentCompleted(event PaymentCompletedEvent) {
    os.db.UpdateOrderStatus(event.OrderID, "PAYMENT_CONFIRMED")
}

func OnPaymentFailed(event PaymentFailedEvent) {
    // Compensating transaction
    os.db.UpdateOrderStatus(event.OrderID, "CANCELLED")
}
```

**Ưu Điểm Choreography:**
- Decentralized, loosely coupled
- Easy to add new services

**Nhược Điểm:**
- Circular dependencies có thể xảy ra
- Khó debug (spaghetti logic)

#### B. Orchestration (Centralized, Explicit)
```go
// Order Saga Orchestrator
type OrderSaga struct {
    orderService    OrderServiceClient
    paymentService  PaymentServiceClient
    inventoryService InventoryServiceClient
}

func (s *OrderSaga) ExecuteOrderCreation(order *Order) error {
    // Step 1: Create order
    createdOrder, err := s.orderService.CreateOrder(order)
    if err != nil {
        return err
    }
    
    // Step 2: Process payment
    payment := Payment{OrderID: createdOrder.ID, Amount: createdOrder.Total}
    paymentResult, err := s.paymentService.ProcessPayment(payment)
    if err != nil {
        // Compensating: Cancel order
        s.orderService.CancelOrder(createdOrder.ID)
        return err
    }
    
    // Step 3: Update inventory
    invResult, err := s.inventoryService.UpdateInventory(createdOrder.ID)
    if err != nil {
        // Compensating: Refund payment & cancel order
        s.paymentService.RefundPayment(paymentResult.ID)
        s.orderService.CancelOrder(createdOrder.ID)
        return err
    }
    
    // Step 4: Send notification
    s.notificationService.SendOrderConfirmation(createdOrder)
    
    return nil
}
```

**Ưu Điểm Orchestration:**
- Clear flow, easy to understand
- Centralized error handling
- Easy to debug

**Nhược Điểm:**
- Orchestrator là single point of failure
- Tight coupling vào orchestrator

**Khi Nào Dùng Cái Nào:**
- Choreography: < 5 services, simple flows
- Orchestration: > 5 services, complex business logic, strict order requirements

---

## PHẦN 4: DATA CONSISTENCY - EVENTUAL CONSISTENCY

### 4.1 CQRS (Command Query Responsibility Segregation)

**Vấn Đề:**
- Monolith: 1 database model → dễ query
- Microservices: Data spread across services → complex queries, N+1 problems

**Giải Pháp: Separate Read & Write Model**
```
Write Model (Command):
├── Order Service (write)
└── DB: PostgreSQL (normalized)

Read Model (Query):
├── Order View Service
└── DB: MongoDB (denormalized)

Sync: Event Streaming
```

**Ví Dụ Implementation:**
```go
// ============ WRITE SIDE ============
// Order Service - normalized schema
type Order struct {
    ID        string
    UserID    string
    Status    string
}

func (os *OrderService) CreateOrder(order *Order) error {
    // Write to normalized DB
    os.writeDB.InsertOrder(order)
    
    // Emit event
    os.eventBus.Publish("order.created", OrderCreatedEvent{
        OrderID: order.ID,
        UserID:  order.UserID,
    })
    return nil
}

// ============ READ SIDE ============
// Order View Service - denormalized, optimized for queries
type OrderView struct {
    OrderID      string
    UserName     string    // Denormalized from User Service
    UserEmail    string
    Items        []OrderItem
    TotalAmount  float64
    Status       string
    CreatedAt    time.Time
}

// Subscribe to events
func (ovs *OrderViewService) OnOrderCreated(event OrderCreatedEvent) {
    // Fetch from upstream services
    user := ovs.userService.GetUser(event.UserID)
    order := ovs.orderService.GetOrder(event.OrderID)
    
    // Build denormalized view
    view := OrderView{
        OrderID:     order.ID,
        UserName:    user.Name,
        UserEmail:   user.Email,
        Items:       order.Items,
        TotalAmount: order.Total,
        Status:      order.Status,
    }
    
    // Write to read model (MongoDB)
    ovs.readDB.InsertOrderView(view)
}

// Fast read query
func (ovs *OrderViewService) GetOrderSummary(orderID string) *OrderView {
    return ovs.readDB.FindByID(orderID)  // Single query, all data here
}
```

**Ưu Điểm CQRS:**
- Read & write models tuned independently
- Read queries ultra-fast (denormalized)
- Scalable reads (read replicas)
- Audit trail (all events stored)

**Nhược Điểm:**
- Eventual consistency (read lags behind write)
- More complex infrastructure
- Data duplication

### 4.2 Event Sourcing

**Concept:**
- Không store current state
- Store ALL events (immutable log)
- Rebuild state từ events

```go
// Event log
EventLog:
[
  {ID: "evt1", Type: "OrderCreated", OrderID: "ord123", UserID: "usr1"},
  {ID: "evt2", Type: "PaymentProcessed", OrderID: "ord123", Amount: 100},
  {ID: "evt3", Type: "InventoryUpdated", OrderID: "ord123", Items: [...]},
]

// Rebuild order state từ events
func (es *EventStore) GetOrderState(orderID string) *Order {
    events := es.GetEventsByOrderID(orderID)
    order := &Order{}
    
    for _, event := range events {
        switch event.Type {
        case "OrderCreated":
            order.ID = event.OrderID
            order.Status = "CREATED"
        case "PaymentProcessed":
            order.Status = "PAID"
        case "InventoryUpdated":
            order.Status = "READY"
        }
    }
    return order
}
```

**Ưu Điểm:**
- Complete audit trail
- Debugging: replay events to understand what happened
- Temporal queries (what was state at time X?)

---

## PHẦN 5: SERVICE COMMUNICATION

### 5.1 Synchronous vs Asynchronous

**Synchronous (REST/gRPC):**
```go
// Order Service calls Payment Service directly
func (os *OrderService) CreateOrder(order *Order) error {
    order.Status = "PENDING"
    os.db.Save(order)
    
    // Blocking call
    paymentResp, err := http.Post(
        "http://payment-service:3002/payments",
        "application/json",
        encodePayment(order),
    )
    if err != nil {
        // Payment failed
        order.Status = "FAILED"
        os.db.Save(order)
        return err
    }
    
    order.Status = "PAID"
    os.db.Save(order)
    return nil
}
```

**Ưu:** Simple, easy to understand, strong consistency
**Nhược:** Tight coupling, timeout issues, cascading failures

**Asynchronous (Message Queue):**
```go
// Order Service publishes event
func (os *OrderService) CreateOrder(order *Order) error {
    order.Status = "PENDING"
    os.db.Save(order)
    
    // Non-blocking publish
    os.messageQueue.Publish("payment.queue", PaymentEvent{
        OrderID: order.ID,
        Amount:  order.Total,
    })
    
    return nil
}

// Payment Service subscribes
func (ps *PaymentService) StartListener() {
    ps.messageQueue.Subscribe("payment.queue", func(msg PaymentEvent) {
        payment := Payment{OrderID: msg.OrderID, Amount: msg.Amount}
        err := ps.ProcessPayment(payment)
        
        if err != nil {
            ps.messageQueue.Publish("payment.failed.queue", msg)
        } else {
            ps.messageQueue.Publish("payment.completed.queue", msg)
        }
    })
}
```

**Ưu:** Loose coupling, resilience, scalable
**Nhược:** Async debugging harder, eventual consistency

### 5.2 API Gateway Pattern

**Tại Sao Cần:**
- Single entry point
- Auth, rate limiting, logging centrally
- Service discovery
- Protocol translation

```nginx
# nginx api-gateway.conf
upstream auth_service {
    server auth-service:3001;
}

upstream order_service {
    server order-service:3002;
}

upstream user_service {
    server user-service:3003;
}

server {
    listen 80;
    
    # Auth middleware - applied to all routes
    location / {
        # Validate JWT token
        if ($http_authorization = "") {
            return 401;
        }
        
        # Extract user_id from token
        proxy_set_header X-User-ID $jwt_claim_sub;
    }
    
    # Route /orders to order-service
    location /orders {
        proxy_pass http://order_service;
        
        # Rate limiting
        limit_req zone=api burst=10 nodelay;
    }
    
    # Route /users to user-service
    location /users {
        proxy_pass http://user_service;
    }
}
```

---

## PHẦN 6: RESILIENCE PATTERNS

### 6.1 Circuit Breaker

**Vấn Đề:**
- Service A gọi Service B
- Service B fail → A keeps retrying → cascade failure → system down

**Giải Pháp: Circuit Breaker (3 states)**
```
CLOSED (Normal)
  ↓ (failures > threshold)
OPEN (Reject fast, prevent cascading)
  ↓ (timeout pass)
HALF_OPEN (Test if recovered)
  ↓ (success) → CLOSED
  ↓ (failure) → OPEN
```

**Implementation:**
```go
import "github.com/grpc-ecosystem/go-grpc-middleware/retry"

// Initialize circuit breaker
breaker := resilience4j.NewCircuitBreaker(&resilience4j.Config{
    FailureThreshold:      50,           // % failures to open
    SuccessThreshold:      100,          // % success to close
    WaitTimeout:           5 * time.Second, // half-open duration
    RecordExceptions:      []string{"timeout", "500"},
})

// Use in service call
func (os *OrderService) ProcessPayment(order *Order) error {
    return breaker.ExecuteFunc(func() error {
        return os.paymentService.Charge(order.Total)
    })
}

// Result:
// Normal: request succeeds
// Service down: fails fast → return cached response or default
// Recovering: circuit opens for new requests, retries if recovered
```

### 6.2 Rate Limiter & Retry Logic

```go
// Token Bucket Algorithm
type RateLimiter struct {
    capacity  int
    tokens    int
    refillRate int // tokens per second
}

func (rl *RateLimiter) AllowRequest() bool {
    rl.refill()
    if rl.tokens > 0 {
        rl.tokens--
        return true
    }
    return false
}

// Retry with exponential backoff
func RetryWithBackoff(fn func() error, maxRetries int) error {
    backoff := time.Second
    
    for i := 0; i < maxRetries; i++ {
        if err := fn(); err == nil {
            return nil
        }
        
        time.Sleep(backoff)
        backoff *= 2 // exponential backoff
    }
    return errors.New("max retries exceeded")
}
```

---

## PHẦN 7: DEVSECOPS INTEGRATION

### 7.1 Build Stage - Image Security

**Cách Tư Duy:**
- Catch security issues EARLY
- Minimize attack surface
- Automate scanning

**Multi-Stage Dockerfile (Reduce Size & Vulnerabilities):**
```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine as builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
# Build only binary, no dev tools
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Stage 2: Runtime (minimal)
FROM alpine:3.18
# Add only necessary tools
RUN apk add --no-cache ca-certificates
# DON'T include: apt, curl, wget, shells, debuggers
WORKDIR /root/
COPY --from=builder /app/app .
# Run as non-root
RUN adduser -D appuser
USER appuser
CMD ["./app"]
```

**Image Scanning:**
```bash
# Scan for vulnerabilities
docker scout cves ./Dockerfile

# Policy enforcement in CI
trivy image --severity HIGH,CRITICAL --exit-code 1 myregistry/order-service:latest
```

### 7.2 Deploy Stage - Secrets Management

**Problem:**
```yaml
# ❌ WRONG: Secrets in env vars
env:
  - name: DB_PASSWORD
    value: "super_secret_123"  # Exposed in pod spec
```

**Solution: External Secrets Manager**
```yaml
# ✅ RIGHT: Reference external secret
apiVersion: v1
kind: Pod
metadata:
  name: order-service
spec:
  containers:
  - name: app
    image: order-service:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password

---
# Secret stored in secure vault (Hashicorp Vault / AWS Secrets Manager)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  password: c3VwZXJfc2VjcmV0XzEyMw==  # base64 only (not encrypted by default!)
```

**Better: Use External Secrets Operator**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret/data"

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  secretStoreRef:
    name: vault-backend
  target:
    name: db-secret
  data:
  - secretKey: password
    remoteRef:
      key: database/prod
```

### 7.3 Runtime Stage - Network Policies & RBAC

**Network Policy (Firewall for Pods):**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: order-service-policy
spec:
  podSelector:
    matchLabels:
      app: order-service
  
  # Ingress: who can call order-service
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 3002
  
  # Egress: what order-service can call
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: payment-service
    ports:
    - protocol: TCP
      port: 3003
  - to:
    - podSelector:
        matchLabels:
          app: user-service
    ports:
    - protocol: TCP
      port: 3001
```

**RBAC (Role-Based Access Control):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: order-service-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]  # Only read ConfigMaps
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]  # Only read specific secrets

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: order-service-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: order-service-role
subjects:
- kind: ServiceAccount
  name: order-service
```

---

## PHẦN 8: OBSERVABILITY - VISIBILITY VÀO SYSTEM

### 8.1 The Three Pillars: Logs, Metrics, Traces

**1. Logs (Where did things happen?)**
```go
// Structured logging
logger.Info("order.created", 
    "order_id", order.ID,
    "user_id", order.UserID,
    "amount", order.Total,
    "timestamp", time.Now(),
)
```

**2. Metrics (How is the system performing?)**
```go
// Counter: how many orders created
orderCounter.Inc()

// Histogram: order processing time distribution
orderProcessingTime.Observe(time.Since(start).Seconds())

// Gauge: current orders in queue
ordersInQueue.Set(len(queue))
```

**3. Traces (How did the request flow?)**
```
User Request
├── API Gateway (2ms)
├── Order Service (5ms)
│   ├── Database query (3ms)
│   └── Publish event (1ms)
├── Payment Service (8ms)
│   └── External payment provider (7ms)
└── Response (15ms total)
```

**Implementation - ELK Stack + Jaeger:**
```go
import (
    "go.opentelemetry.io/otel/trace"
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
)

// Distributed tracing
span := tracer.Start(ctx, "CreateOrder")
defer span.End()

// Add attributes
span.SetAttribute("order.id", order.ID)
span.SetAttribute("user.id", order.UserID)

// Nested spans for sub-operations
dbSpan := tracer.Start(ctx, "database.insert")
os.db.Insert(order)  // Automatically traced
dbSpan.End()
```

### 8.2 Chaos Engineering - Proactive Resilience Testing

**Cách Tư Duy:**
- Không đợi failure xảy ra trong production
- Deliberately break things để tìm yếu điểm
- Automated resilience testing trong CI/CD

```bash
# Using Chaos Toolkit
chaos discover --system kubernetes
chaos init --from-discovery /path/to/discovery.json

# Chaos experiment: kill a random pod
chaos run experiments/kill_pod.yaml

# Verify circuit breaker works
# Verify auto-recovery
# Measure MTTR (Mean Time To Recovery)
```

---

## PHẦN 9: ADVANCED PATTERNS

### 9.1 Service Mesh (Istio)

**Tại Sao Cần:**
- Đa số logic network handling → sidecar proxy
- mTLS encryption
- Traffic shaping
- Observability

```yaml
# Virtual Service: routing rules
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
  - order-service
  http:
  - match:
    - headers:
        user-type:
          exact: premium
    route:
    - destination:
        host: order-service
        port:
          number: 3002
      weight: 100
  - route:
    - destination:
        host: order-service
        port:
          number: 3002
      weight: 80  # 80% to v1
    - destination:
        host: order-service-v2
        port:
          number: 3002
      weight: 20  # 20% canary to v2
```

### 9.2 API Gateway Advanced

```go
// Kong API Gateway example
// Define service with rate limiting, auth, logging
type ServiceConfig struct {
    Name: "order-service",
    URL: "http://order-service:3002",
    Plugins: []Plugin{
        {
            Name: "rate-limiting",
            Config: RateLimitConfig{
                Minute: 100,  // 100 requests per minute
                Hour: 10000,
            },
        },
        {
            Name: "jwt",
            Config: JWTConfig{
                ClaimsToVerify: []string{"iss", "sub"},
            },
        },
    },
}
```

---

## PHẦN 10: LEARNING ROADMAP & RESOURCES

### Level 1: Monolith Foundation (Week 1-2)
- ✅ Understand monolithic architecture
- ✅ Learn about tightly coupled components
- ✅ Practice: Analyze a monolith codebase

### Level 2: Service Extraction (Week 3-4)
- ✅ Study Strangler Fig pattern
- ✅ Identify service boundaries using DDD
- ✅ Practice: Extract 1 service from monolith

### Level 3: Data Management (Week 5-6)
- ✅ Learn Database per Service
- ✅ Study Saga pattern (choreography & orchestration)
- ✅ Practice: Implement Saga for distributed transaction

### Level 4: Resilience (Week 7-8)
- ✅ Circuit breaker, retry logic, rate limiting
- ✅ Learn Bulkhead pattern (isolation)
- ✅ Practice: Add resilience to failing services

### Level 5: Observability (Week 9-10)
- ✅ ELK Stack setup
- ✅ Distributed tracing with Jaeger
- ✅ Practice: Debug production issue using traces

### Level 6: DevSecOps (Week 11-12)
- ✅ Container security, image scanning
- ✅ Secrets management
- ✅ RBAC, Network Policies
- ✅ Practice: Deploy secure microservice to Kubernetes

### Level 7: Advanced (Week 13+)
- ✅ Service Mesh (Istio)
- ✅ Advanced API Gateway patterns
- ✅ Chaos engineering

---

## PHẦN 11: CÂU HỎI INTERVIEW & CÁCH TRẢ LỜI

### Q1: "Why microservices? When should we migrate?"

**Tư Duy:**
- Microservices không phải silver bullet
- Chỉ migrate khi monolith thực sự là bottleneck

**Câu Trả Lời:**
"Microservices cho phép:
1. Independent scaling - nếu 1 component scale khác
2. Technology diversity - mỗi service chọn best tech
3. Team autonomy - teams can work independently
4. Faster deployment - deploy service không cần toàn hệ thống

Tuy nhiên, migrate khi:
- System quá phức tạp để maintain
- Different teams work on different features
- Scaling bottlenecks rõ ràng
- NOT vì 'it's the trend'"

### Q2: "How to migrate from monolith safely?"

**Cách Tú Duy:**
- Progressive, low-risk approach
- Verify everything before moving forward

**Câu Trả Lời:**
"Use Strangler Fig Pattern:
1. Transform: Build new service alongside monolith
2. Coexist: Route traffic gradually using API Gateway
3. Eliminate: Only remove monolith feature when 100% confident

Example:
- Week 1: Extract User Service (5% traffic)
- Week 2: 20% traffic
- Week 3: 50% traffic
- Week 4: 100% traffic, remove from monolith

Verify using dual reads/writes, canary deployments"

### Q3: "How to handle distributed transactions?"

**Cách Tư Duy:**
- Can't use traditional 2PC across databases
- Saga pattern is industry standard

**Câu Trả Lời:**
"Two approaches:

1. Choreography (Event-Driven):
   - Service A emits event
   - Service B listens & processes
   - Service B emits result event
   - Pros: Loose coupling
   - Cons: Hard to trace, circular dependencies possible

2. Orchestration (Central Coordinator):
   - Coordinator calls services sequentially
   - Coordinates compensating actions on failure
   - Pros: Clear flow
   - Cons: Orchestrator is SPOF

Example: Order → Payment → Inventory
- If Inventory fails → refund Payment → cancel Order"

### Q4: "How to ensure data consistency?"

**Cách Tư Duy:**
- Microservices = eventual consistency (trade-off)
- CQRS + Event Sourcing for consistency guarantees

**Câu Trả Lời:**
"Data consistency trade-offs:
1. Strong consistency: Expensive, slow, hard in distributed systems
2. Eventual consistency: Fast, scalable, temporary inconsistency

Solutions:
- CQRS: Separate read/write models
- Event Sourcing: All changes = events (audit trail, replay)
- Saga Pattern: Distributed transaction coordination

Example CQRS:
- Write to Order Service DB (normalized)
- Event published → Order View updated (denormalized)
- Read from Order View (fast, but eventually consistent)
- Consistency lag: usually < 1 second"

### Q5: "How to secure microservices?"

**Cách Tư Duy:**
- Security at every layer
- Shift left (build time > runtime)

**Câu Trả Lời:**
"DevSecOps approach:

Build Stage:
- Multi-stage Docker builds (reduce image size)
- Image scanning (Trivy, Scout)
- Fail build if vulnerabilities found

Deploy Stage:
- Secrets management (Vault, AWS Secrets Manager)
- RBAC (least privilege access)
- Network Policies (pod-to-pod communication)
- mTLS (Istio) for service-to-service encryption

Runtime:
- Pod Security Policies
- Runtime monitoring
- Admission controllers
- Regular security audits"

