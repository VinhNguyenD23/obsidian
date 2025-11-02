# Lộ Trình Chi Tiết: Code Examples & Case Studies

## PHẦN 1: CODE EXAMPLES THỰC TẾ

### 1.1 Strangler Fig Pattern - Complete Example

```go
// ==================== MONOLITH ====================
// monolith/main.go
package main

import (
    "github.com/gofiber/fiber/v2"
    "database/sql"
)

func main() {
    app := fiber.New()
    db, _ := sql.Open("postgres", "monolith_db")
    
    // Auth routes (sẽ migrate)
    app.Post("/auth/register", RegisterUserMonolith)
    app.Post("/auth/login", LoginMonolith)
    app.Get("/auth/validate", ValidateTokenMonolith)
    
    // Other routes
    app.Get("/products", GetProducts)
    app.Post("/orders", CreateOrder)
    
    app.Listen(":8080")
}

// Monolith auth logic
func LoginMonolith(c *fiber.Ctx) error {
    var req struct {
        Email    string
        Password string
    }
    c.BodyParser(&req)
    
    // Query monolith DB
    user := getUser(req.Email)
    if !validatePassword(user, req.Password) {
        return c.Status(401).JSON(fiber.Map{"error": "invalid credentials"})
    }
    
    token := generateJWT(user)
    return c.JSON(fiber.Map{"token": token})
}

// ==================== NEW USER SERVICE ====================
// user-service/main.go
package main

import (
    "github.com/gofiber/fiber/v2"
    "database/sql"
    "time"
)

func main() {
    app := fiber.New()
    db, _ := sql.Open("postgres", "user_service_db")
    
    // Same endpoints, separate database
    app.Post("/auth/register", RegisterUserService)
    app.Post("/auth/login", LoginService)
    app.Get("/auth/validate", ValidateTokenService)
    
    app.Listen(":3001")
}

// New service with own DB
func LoginService(c *fiber.Ctx) error {
    var req struct {
        Email    string
        Password string
    }
    c.BodyParser(&req)
    
    // Query user-service DB
    user := getUserFromServiceDB(req.Email)
    if !validatePassword(user, req.Password) {
        return c.Status(401).JSON(fiber.Map{"error": "invalid credentials"})
    }
    
    token := generateJWT(user)
    
    // Emit event for audit
    publishEvent("user.authenticated", map[string]interface{}{
        "user_id": user.ID,
        "email":   user.Email,
        "timestamp": time.Now(),
    })
    
    return c.JSON(fiber.Map{"token": token})
}

// ==================== API GATEWAY (FACADE) ====================
// api-gateway/routes.go
package main

import (
    "net/http"
    "net/http/httputil"
    "net/url"
)

type RouteConfig struct {
    monolith    *url.URL
    userService *url.URL
}

func setupRoutes(cfg RouteConfig) {
    // Route auth to new service (gradual migration)
    http.HandleFunc("/auth/", func(w http.ResponseWriter, r *http.Request) {
        // % traffic to new service (gradually increase)
        if shouldRouteToNewService(r) {
            proxy := httputil.NewSingleHostReverseProxy(cfg.userService)
            proxy.ServeHTTP(w, r)
        } else {
            proxy := httputil.NewSingleHostReverseProxy(cfg.monolith)
            proxy.ServeHTTP(w, r)
        }
    })
    
    // Other routes go to monolith
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        proxy := httputil.NewSingleHostReverseProxy(cfg.monolith)
        proxy.ServeHTTP(w, r)
    })
}

func shouldRouteToNewService(r *http.Request) bool {
    // A/B testing: route 10% of traffic to new service
    // In production, use more sophisticated routing (Nginx, HAProxy)
    userID := r.Header.Get("X-User-ID")
    return hashUserID(userID)%10 < 1  // 10% routing
}

// ==================== ANTI-CORRUPTION LAYER ====================
// monolith/acl.go
package main

import "net/http"

// Convert new service response to monolith domain model
type OldUser struct {
    ID       string
    Email    string
    Username string
}

type NewServiceUser struct {
    ID    string `json:"id"`
    Email string `json:"email"`
}

func getUserFromNewService(userID string) *OldUser {
    resp, _ := http.Get("http://user-service:3001/auth/users/" + userID)
    
    var newUser NewServiceUser
    json.NewDecoder(resp.Body).Decode(&newUser)
    
    // Convert to monolith's user model
    return &OldUser{
        ID:       newUser.ID,
        Email:    newUser.Email,
        Username: extractUsernameFromEmail(newUser.Email),
    }
}

// ==================== DATA MIGRATION ====================
// migration/dual_write.go
package main

import "database/sql"

// Phase 1: Dual write - write to both DBs
func InsertUserDualWrite(monolithDB, userServiceDB *sql.DB, user *User) error {
    // Write to monolith
    _, err := monolithDB.Exec(
        "INSERT INTO users (id, email, password_hash) VALUES ($1, $2, $3)",
        user.ID, user.Email, user.PasswordHash,
    )
    if err != nil {
        return err
    }
    
    // Write to new service DB
    _, err = userServiceDB.Exec(
        "INSERT INTO users (id, email, password_hash) VALUES ($1, $2, $3)",
        user.ID, user.Email, user.PasswordHash,
    )
    return err
}

// Phase 2: Event-driven sync (preferred)
type UserCreatedEvent struct {
    UserID       string
    Email        string
    PasswordHash string
    Timestamp    time.Time
}

func PublishUserCreatedEvent(user *User, eventBus EventBus) {
    event := UserCreatedEvent{
        UserID:       user.ID,
        Email:        user.Email,
        PasswordHash: user.PasswordHash,
        Timestamp:    time.Now(),
    }
    eventBus.Publish("user.created", event)
}

// User Service subscribes
func (us *UserService) OnUserCreated(event UserCreatedEvent) {
    us.db.Exec(
        "INSERT INTO users (id, email, password_hash) VALUES ($1, $2, $3)",
        event.UserID, event.Email, event.PasswordHash,
    )
}
```

### 1.2 Saga Pattern - Distributed Transactions

```go
// ==================== ORCHESTRATION APPROACH ====================
// order-saga/orchestrator.go
package main

import (
    "context"
    "errors"
    "time"
)

type OrderSaga struct {
    orderService     OrderServiceClient
    paymentService   PaymentServiceClient
    inventoryService InventoryServiceClient
    notifyService    NotificationServiceClient
}

// Saga execution with compensating transactions
func (s *OrderSaga) ExecuteCreateOrder(ctx context.Context, order *Order) error {
    // Step 1: Create order
    createdOrder, err := s.orderService.CreateOrder(ctx, order)
    if err != nil {
        return err
    }
    
    // Step 2: Process payment
    payment := &Payment{
        OrderID: createdOrder.ID,
        Amount:  createdOrder.Total,
    }
    paymentResult, err := s.paymentService.ProcessPayment(ctx, payment)
    if err != nil {
        // Compensating: Cancel order
        s.orderService.CancelOrder(ctx, createdOrder.ID)
        return errors.New("payment failed: order cancelled")
    }
    
    // Step 3: Update inventory
    invReq := &InventoryUpdateRequest{
        OrderID: createdOrder.ID,
        Items:   createdOrder.Items,
    }
    invResult, err := s.inventoryService.UpdateInventory(ctx, invReq)
    if err != nil {
        // Compensating: Refund + cancel order
        s.paymentService.RefundPayment(ctx, paymentResult.ID)
        s.orderService.CancelOrder(ctx, createdOrder.ID)
        return errors.New("inventory update failed: order cancelled, payment refunded")
    }
    
    // Step 4: Send confirmation
    s.notifyService.SendOrderConfirmation(ctx, &NotificationRequest{
        OrderID: createdOrder.ID,
        Email:   order.Email,
    })
    
    // Success
    s.orderService.ConfirmOrder(ctx, createdOrder.ID)
    return nil
}

// ==================== CHOREOGRAPHY APPROACH ====================
// Services publish events, others subscribe
package main

import "encoding/json"

// Order Service
type OrderService struct {
    db       *sql.DB
    eventBus EventBus
}

func (os *OrderService) CreateOrder(ctx context.Context, order *Order) (*Order, error) {
    order.Status = "PENDING"
    
    // Insert to DB
    err := os.db.QueryRow(
        "INSERT INTO orders (id, user_id, total, status) VALUES ($1, $2, $3, $4) RETURNING id",
        order.ID, order.UserID, order.Total, order.Status,
    ).Scan(&order.ID)
    
    if err != nil {
        return nil, err
    }
    
    // Publish event (async)
    os.eventBus.Publish("order.created", OrderCreatedEvent{
        OrderID: order.ID,
        UserID:  order.UserID,
        Total:   order.Total,
        Items:   order.Items,
    })
    
    return order, nil
}

// Payment Service subscribes to order.created
type PaymentService struct {
    db       *sql.DB
    eventBus EventBus
}

func (ps *PaymentService) Start(ctx context.Context) {
    ps.eventBus.Subscribe("order.created", func(msg interface{}) {
        var event OrderCreatedEvent
        json.Unmarshal(msg.([]byte), &event)
        
        // Process payment
        payment := Payment{OrderID: event.OrderID, Amount: event.Total}
        err := ps.processPayment(payment)
        
        if err != nil {
            // Emit failure event
            ps.eventBus.Publish("payment.failed", PaymentFailedEvent{
                OrderID: event.OrderID,
                Reason:  err.Error(),
            })
        } else {
            // Emit success event
            ps.eventBus.Publish("payment.completed", PaymentCompletedEvent{
                OrderID: event.OrderID,
            })
        }
    })
}

// Order Service listens to payment results
func (os *OrderService) Subscribe() {
    os.eventBus.Subscribe("payment.completed", func(msg interface{}) {
        var event PaymentCompletedEvent
        json.Unmarshal(msg.([]byte), &event)
        
        os.db.Exec(
            "UPDATE orders SET status = 'PAYMENT_CONFIRMED' WHERE id = $1",
            event.OrderID,
        )
    })
    
    os.eventBus.Subscribe("payment.failed", func(msg interface{}) {
        var event PaymentFailedEvent
        json.Unmarshal(msg.([]byte), &event)
        
        // Compensating: Cancel order
        os.db.Exec(
            "UPDATE orders SET status = 'CANCELLED' WHERE id = $1",
            event.OrderID,
        )
    })
}
```

### 1.3 CQRS Pattern - Separate Read/Write

```go
// ==================== WRITE MODEL ====================
// order-service/write_model.go
package main

import "database/sql"

type Order struct {
    ID     string
    UserID string
    Status string
    Total  float64
}

type OrderWriteRepository struct {
    db *sql.DB
}

// Normalized schema for writes
func (r *OrderWriteRepository) Create(order *Order) error {
    return r.db.QueryRow(
        `INSERT INTO orders (id, user_id, status, total) 
         VALUES ($1, $2, $3, $4) RETURNING id`,
        order.ID, order.UserID, order.Status, order.Total,
    ).Scan(&order.ID)
}

// ==================== READ MODEL ====================
// order-view-service/read_model.go
package main

type OrderView struct {
    OrderID      string
    UserName     string
    UserEmail    string
    Items        []OrderItem
    TotalAmount  float64
    Status       string
    CreatedAt    time.Time
}

type OrderViewRepository struct {
    db *sql.DB  // MongoDB or other read-optimized DB
}

// Denormalized, optimized for queries
func (r *OrderViewRepository) FindByID(id string) (*OrderView, error) {
    var view OrderView
    
    // Single query, all data here (no joins)
    err := r.db.QueryRow(
        `SELECT order_id, user_name, user_email, items, total_amount, status, created_at
         FROM order_views WHERE order_id = $1`,
        id,
    ).Scan(&view.OrderID, &view.UserName, &view.UserEmail, 
           &view.Items, &view.TotalAmount, &view.Status, &view.CreatedAt)
    
    return &view, err
}

// ==================== SYNC (Write → Read) ====================
// event-processor.go - subscribes to write model events
package main

import "encoding/json"

type EventProcessor struct {
    eventBus EventBus
    writeRepo *OrderWriteRepository
    readRepo  *OrderViewRepository
}

func (ep *EventProcessor) Start() {
    // Listen to all order events
    ep.eventBus.Subscribe("order.created", func(msg interface{}) {
        var event OrderCreatedEvent
        json.Unmarshal(msg.([]byte), &event)
        
        // Build denormalized view
        order := ep.writeRepo.GetByID(event.OrderID)
        user := getUserService(order.UserID)  // Call User Service
        
        view := OrderView{
            OrderID:     order.ID,
            UserName:    user.Name,
            UserEmail:   user.Email,
            Items:       order.Items,
            TotalAmount: order.Total,
            Status:      order.Status,
            CreatedAt:   time.Now(),
        }
        
        // Write to read model
        ep.readRepo.Insert(view)
    })
    
    ep.eventBus.Subscribe("payment.completed", func(msg interface{}) {
        var event PaymentCompletedEvent
        json.Unmarshal(msg.([]byte), &event)
        
        // Update view
        ep.readRepo.UpdateStatus(event.OrderID, "PAYMENT_CONFIRMED")
    })
}

// ==================== QUERY SERVICE ====================
// Fast queries from read model
func (s *OrderService) GetOrderSummary(orderID string) (*OrderView, error) {
    // Single query from denormalized data
    return s.readRepo.FindByID(orderID)
}

// Complex query with filters
func (s *OrderService) GetUserOrders(userID string) ([]OrderView, error) {
    rows, err := s.readRepo.db.Query(
        `SELECT order_id, user_name, total_amount, status 
         FROM order_views 
         WHERE user_id = $1 
         ORDER BY created_at DESC`,
        userID,
    )
    // Parse and return
}
```

### 1.4 Circuit Breaker & Resilience

```go
// ==================== CIRCUIT BREAKER IMPLEMENTATION ====================
// resilience/circuit_breaker.go
package main

import (
    "errors"
    "sync"
    "time"
)

type CircuitBreakerState string

const (
    CLOSED     CircuitBreakerState = "CLOSED"     // Normal
    OPEN       CircuitBreakerState = "OPEN"       // Failing, reject fast
    HALF_OPEN  CircuitBreakerState = "HALF_OPEN"  // Testing recovery
)

type CircuitBreaker struct {
    mu                   sync.RWMutex
    state                CircuitBreakerState
    failureCount         int
    successCount         int
    failureThreshold     int  // Failures to open
    successThreshold     int  // Successes to close
    timeout              time.Duration
    lastFailureTime      time.Time
}

func NewCircuitBreaker(failureThreshold, successThreshold int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        state:            CLOSED,
        failureThreshold: failureThreshold,
        successThreshold: successThreshold,
        timeout:          timeout,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    // Check if should transition from OPEN to HALF_OPEN
    if cb.state == OPEN {
        if time.Since(cb.lastFailureTime) > cb.timeout {
            cb.state = HALF_OPEN
            cb.failureCount = 0
            cb.successCount = 0
        } else {
            return errors.New("circuit breaker is OPEN")
        }
    }
    
    // Execute function
    err := fn()
    
    if err != nil {
        cb.failureCount++
        cb.lastFailureTime = time.Now()
        
        // Transition to OPEN if threshold reached
        if cb.failureCount >= cb.failureThreshold {
            cb.state = OPEN
        }
        return err
    }
    
    // Success
    cb.successCount++
    
    // Transition to CLOSED if recovered
    if cb.state == HALF_OPEN && cb.successCount >= cb.successThreshold {
        cb.state = CLOSED
        cb.failureCount = 0
    }
    
    return nil
}

// ==================== USAGE ====================
type PaymentServiceClient struct {
    breaker *CircuitBreaker
}

func (pc *PaymentServiceClient) ProcessPayment(amount float64) error {
    return pc.breaker.Call(func() error {
        // Call payment service
        return pc.callRemoteService(amount)
    })
}

// ==================== RETRY LOGIC ====================
package main

import "math"

func RetryWithExponentialBackoff(
    fn func() error,
    maxRetries int,
    initialBackoff time.Duration,
) error {
    backoff := initialBackoff
    
    for attempt := 0; attempt < maxRetries; attempt++ {
        err := fn()
        if err == nil {
            return nil
        }
        
        // Don't sleep on last attempt
        if attempt < maxRetries-1 {
            time.Sleep(backoff)
            backoff = time.Duration(float64(backoff) * math.Pow(2, 1))  // 2x exponential
        }
    }
    
    return errors.New("max retries exceeded")
}

// Usage
err := RetryWithExponentialBackoff(
    func() error {
        return paymentService.Charge(100)
    },
    3,  // max 3 retries
    100 * time.Millisecond,
)
```

### 1.5 DevSecOps - Container Security

```dockerfile
# ==================== MULTI-STAGE BUILD ====================
# Dockerfile
FROM golang:1.21-alpine as builder

WORKDIR /app

# Copy only go.mod/go.sum first (cache layer)
COPY go.mod go.sum ./
RUN go mod download

# Copy source
COPY . .

# Build without CGO, static binary
RUN CGO_ENABLED=0 GOOS=linux go build \
    -a -installsuffix cgo \
    -ldflags="-w -s" \
    -o app .

# ==================== RUNTIME STAGE ====================
# Use minimal base image
FROM alpine:3.18

# Install only necessary runtime dependencies
RUN apk add --no-cache ca-certificates

# Create non-root user
RUN adduser -D appuser

WORKDIR /home/appuser

# Copy only binary from builder
COPY --from=builder /app/app .

# Set permissions
RUN chown appuser:appuser /home/appuser/app && \
    chmod 755 /home/appuser/app

# Run as non-root
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/home/appuser/app", "health"]

EXPOSE 3002

CMD ["./app"]
```

### 1.6 Kubernetes Security & DevSecOps

```yaml
# ==================== DEPLOYMENT WITH SECURITY ====================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
      annotations:
        container.apparmor.security.beta.kubernetes.io/order-app: runtime/default
    spec:
      # Service account for RBAC
      serviceAccountName: order-service
      
      # Security context for pod
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsReadOnlyRootFilesystem: true
        
      containers:
      - name: order-app
        image: myregistry/order-service:v1.0
        imagePullPolicy: Always
        
        # Ports
        ports:
        - containerPort: 3002
          name: http
          protocol: TCP
        
        # Resource limits (prevent DoS)
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        
        # Security context for container
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
        
        # Environment variables from ConfigMap (non-sensitive)
        env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: order-config
              key: log-level
        
        # Secrets from external manager (not in YAML)
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        
        # Liveness & readiness probes
        livenessProbe:
          httpGet:
            path: /health
            port: 3002
          initialDelaySeconds: 10
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 3002
          initialDelaySeconds: 5
          periodSeconds: 5
        
        # Volume for temp files
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      
      # Volumes
      volumes:
      - name: tmp
        emptyDir: {}

---
# ==================== NETWORK POLICY ====================
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: order-service-netpol
spec:
  podSelector:
    matchLabels:
      app: order-service
  
  # Ingress: who can call
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
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
  
  # Allow DNS
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53

---
# ==================== RBAC ====================
apiVersion: v1
kind: ServiceAccount
metadata:
  name: order-service

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: order-service-role
rules:
# Read ConfigMaps
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
# Read Secrets (only needed ones)
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
  resourceNames: ["db-credentials"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: order-service-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: order-service-role
subjects:
- kind: ServiceAccount
  name: order-service
```

---

## PHẦN 2: CASE STUDIES THỰC TẾ

### Case Study 1: E-commerce Migration Timeline

**Before:**
- Monolith (Java Spring Boot): 2M LOC
- Single PostgreSQL: 50 tables, 1TB data
- 100+ engineers working on same codebase
- Deployment every 2 weeks (risky)
- Scaling: whole app must scale

**Week 1-4: Analysis Phase**
- Identify components: User (500K), Product (100K), Order (5M), Payment, Inventory
- Choose User Service first (edge service, clear boundary, low risk)
- Baseline: 500 req/s, 2s p99 latency, 3x per week deployments

**Week 5-8: Extract User Service**
- Built separate PostgreSQL for users
- API Gateway routes: 5% to new service, 95% to monolith
- Tested: identical responses from both

**Week 9-12: Data Sync**
- Dual write: new registrations go to both DBs
- Migrated 500K existing users
- Verified consistency

**Week 13-16: Traffic Cutover**
- 25% → new service
- 50% → new service
- 100% → new service
- 0% → monolith (removed from monolith codebase)

**Result:**
- User Service: Independent scaling, own deployment pipeline
- Team: Dedicated 3-person team can move fast
- Deployment: Can deploy independently, 50+ times per week

### Case Study 2: Distributed Transaction (Payment) Issue

**Scenario:**
Order Service, Payment Service, Inventory Service

**Problem (Without Saga):**
```
Step 1: Create Order → ✅ Success
Step 2: Process Payment → ❌ Fails (bank timeout)
Step 3: Update Inventory → ⏸️ Blocked
Result: Order created but not paid, inventory not updated ❌
```

**Solution (Saga Pattern):**
```
Step 1: Create Order (PENDING) → ✅
Step 2: Process Payment → ❌ Timeout
Compensating:
  - Cancel Order (PENDING → CANCELLED)
  - Refund Payment (if partial)
Result: System in consistent state ✅
```

**Real Impact:**
- Before: 2% of orders had inconsistent state (manual cleanup)
- After: 0% inconsistent orders (automatic rollback)
- Recovery Time: 1s (vs 30min manual fix)

---

## PHẦN 3: DECISION MATRIX - KHI NÀO DÙNG GÌ?

### Monolith vs Microservices

| Criteria | Monolith | Microservices |
|----------|----------|---------------|
| Team size | < 10 | > 30 |
| Deployment frequency | < 1x/week | > 10x/week |
| Scaling needs | Uniform | Varied |
| Technology diversity | No | Yes |
| Complexity | Low-medium | High |
| **Decision** | ✅ | ✅ |

### CQRS vs Simple Pattern

| Use CQRS if | Skip CQRS if |
|------------|------------|
| Complex reads from multiple services | Simple queries, same DB |
| Read load >> write load | Balanced read/write |
| Need denormalized views | Normalized schema ok |
| Eventual consistency acceptable | Strong consistency needed |

### Choreography vs Orchestration

| Choreography | Orchestration |
|------------|--------------|
| < 5 services | > 5 services |
| Simple flow | Complex business logic |
| Async-friendly | Explicit control needed |
| Hard to debug | Easy to trace |

