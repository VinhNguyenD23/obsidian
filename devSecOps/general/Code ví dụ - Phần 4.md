### Giai đoạn 5: DevSecOps Toàn diện

#### 16. CI/CD Pipeline & Security (Shift-Left)

- _Mục tiêu:_ Xây dựng một đường ống (pipeline) tự động tích hợp quét bảo mật (SAST, SCA, DAST) và deploy.
    
- _Công cụ:_ GitLab CI (ví dụ), SonarQube (SAST), Snyk (SCA), OWASP ZAP (DAST).
    
- _Code mẫu:_ `.gitlab-ci.yml` (Đặt ở root của repository, vd: `product-service`)
    

YAML

```YAML
# .gitlab-ci.yml (Ví dụ cho Product Service)
# Định nghĩa các giai đoạn (stages)
stages:
  - build
  - test
  - security_scan # Giai đoạn cho SAST & SCA
  - deploy_staging
  - dast_scan # Giai đoạn cho DAST (chỉ chạy trên staging)
  - deploy_prod

# 1. Build App
build_app:
  stage: build
  image: node:18-alpine
  script:
    - npm ci # Cài đặt dependencies
    - npm run build # Build app (tsc)
  artifacts:
    paths:
      - dist/
      - node_modules/

# 2. Run Unit Tests (user memory: Test pass)
run_tests:
  stage: test
  image: node:18-alpine
  script:
    - npm test
  dependencies:
    - build_app

# 3. Security Scan (SAST & SCA) - "Shift Left"
sast_scan (SonarQube):
  stage: security_scan
  image: sonarsource/sonar-scanner-cli:latest
  script:
    # SONAR_TOKEN và SONAR_HOST_URL được cấu hình trong CI/CD Variables
    - sonar-scanner 
      -Dsonar.projectKey=product-service
      -Dsonar.sources=.
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
  # Cho phép fail nếu SonarQube phát hiện lỗi (Quality Gate)
  allow_failure: false 

sca_scan (Snyk):
  stage: security_scan
  image: snyk/snyk:node
  script:
    # SNYK_TOKEN được cấu hình trong CI/CD Variables
    - snyk auth $SNYK_TOKEN
    # Quét và báo cáo lỗi (critical, high)
    - snyk test --severity-threshold=high
    # Ghi lại snapshot dependency cho Snyk Monitor
    - snyk monitor
  # Đây là "Security Gate". Nếu Snyk tìm thấy lỗ hổng 'high', pipeline DỪNG LẠI.
  allow_failure: false 

# 4. Deploy lên Staging
deploy_staging:
  stage: deploy_staging
  script:
    # Giả định dùng K8s (kubectl)
    - echo "Deploying to Staging environment..."
    - kubectl apply -f k8s/staging.yaml
  environment:
    name: staging
    url: http://staging.api.example.com

# 5. DAST Scan (OWASP ZAP)
dast_scan (OWASP ZAP):
  stage: dast_scan
  image: owasp/zap2docker-stable
  script:
    # Chạy DAST scan (baseline) vào URL của Staging
    - zap-baseline.py 
      -t $STAGING_URL/products # STAGING_URL từ bước deploy
      -r report.html
  artifacts:
    paths: [report.html]
  # Không nên fail pipeline ngay, chỉ báo cáo (tùy chính sách)
  allow_failure: true 

# 6. Deploy Production (user memory: Migration qua CI)
deploy_production:
  stage: deploy_prod
  script:
    # 1. Chạy migration (user memory: Migration version)
    - echo "Running DB Migrations..."
    - npm run migration:run # Giả định 'prisma migrate deploy' hoặc 'typeorm migration:run'
    
    # 2. Deploy app (vd: K8s)
    - echo "Deploying to Production..."
    - kubectl apply -f k8s/production.yaml
  environment:
    name: production
    url: http://api.example.com
  # Chỉ chạy khi 'main' (hoặc 'master') branch
  only:
    - main
  # Yêu cầu duyệt thủ công (manual approval)
  when: manual 
```

---

#### 17. Container & Orchestration (K8s) Security

- _Mục tiêu:_ Xây dựng Docker image an toàn (tối ưu, non-root) và áp dụng chính sách mạng Zero Trust trong K8s.
    

**Code mẫu 1: `Dockerfile` Tối ưu & An toàn (Best Practice)**

- _Mục tiêu:_ (user memory: dockerfile best practice, cực kỳ tối ưu và phải nhẹ).
    

Dockerfile

```Dockerfile
# ----- Giai đoạn 1: Build -----
# (Multi-stage build)
FROM node:18-alpine AS builder
WORKDIR /app

# Chỉ copy package.json để tận dụng cache
COPY package*.json ./
# Cài đặt (bao gồm devDependencies để build)
RUN npm ci

COPY . .
# Build Typescript ra Javascript
RUN npm run build

# Xóa devDependencies để chuẩn bị cho production
RUN npm prune --production

# ----- Giai đoạn 2: Production -----
# Dùng base image GỌN NHẸ
FROM node:18-alpine AS production
WORKDIR /app

# 1. (Security) Tạo 1 user KHÔNG PHẢI root (tên là 'node')
# Alpine không có sẵn user 'node', ta phải tự tạo
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
# (Nếu dùng image 'node:18-slim' thì user 'node' đã có sẵn)

# Copy artifact từ giai đoạn 'builder'
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./

# 2. (Security) Đổi sang user 'node'
# Mọi tiến trình sau đây sẽ chạy với quyền 'appuser' (non-root)
USER appuser

# 3. (Security) Config các thư viện bảo mật (user memory: helmet)
# (Helmet được config trong code NestJS, đây là ghi chú)

# 4. (Vận hành) Expose port
EXPOSE 3000

# 5. Chạy app
CMD [ "node", "dist/main" ]
```

**Code mẫu 2: `NetworkPolicy` của Kubernetes (Zero Trust)**

- _Mục tiêu:_ Áp dụng "Firewall nội bộ". Chúng ta sẽ cấu hình để: `OrderService` **ĐƯỢC PHÉP** gọi gRPC (port 50051) của `ProductService`, nhưng `PaymentService` (ví dụ) thì **KHÔNG**.
    

YAML

```YAML
# k8s/network-policy-product.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: product-service-ingress
  namespace: default # Áp dụng cho namespace 'default'
spec:
  # 1. Policy này áp dụng cho Pod nào?
  # -> Các Pod có label 'app: product-service'
  podSelector:
    matchLabels:
      app: product-service
      
  # 2. Loại policy: Ingress (Traffic đi vào)
  policyTypes:
    - Ingress

  # 3. Quy tắc Ingress
  ingress:
    # Chỉ cho phép traffic TỪ:
    - from:
        # (A) Các Pod có label 'app: order-service'
        - podSelector:
            matchLabels:
              app: order-service
        # (B) Các Pod có label 'app: api-gateway'
        - podSelector:
            matchLabels:
              app: api-gateway
              
      # VÀ (AND) chỉ cho phép VÀO port:
      ports:
        # (A) Cho phép Gateway gọi HTTP (port 3002 - ví dụ)
        - protocol: TCP
          port: 3002
        # (B) Cho phép OrderService gọi gRPC (port 50051 - từ Mục 11)
        - protocol: TCP
          port: 50051
          
# Kết quả: MỌI traffic khác (vd: từ PaymentService) đến ProductService
# sẽ bị BLOCK, kể cả khi chúng ở chung 1 cluster.
```

---

#### 18. Monitoring & Alerting (Security Monitoring)

- _Mục tiêu:_ Sử dụng stack (Prometheus + Grafana) để giám sát các chỉ số bảo mật, không chỉ là performance.
    
- _Code mẫu 1:_ Expose metrics (chỉ số) từ NestJS (dùng `prom-client`).
    
- _Code mẫu 2:_ Cấu hình Alert (cảnh báo) trong Prometheus.
    

**Code mẫu 1: `metrics.interceptor.ts` (Expose metrics từ NestJS)**

- _Cài đặt:_ `npm i prom-client`
    

TypeScript

```TypeScript
// src/common/metrics.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { register, Counter, Histogram } from 'prom-client';

// Tạo metrics (nên làm ở Module riêng)
// Đảm bảo chỉ chạy 1 lần
register.clear(); // Xóa register cũ (cho hot-reload)

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 1, 1.5, 2, 5], // Phân loại thời gian (giây)
});

// Thêm 1 endpoint /metrics (thường ở AppModule hoặc MetricsModule)
// @Get('/metrics')
// async getMetrics() {
//   return register.metrics();
// }


@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const httpContext = context.switchToHttp();
    const request = httpContext.getRequest();
    const response = httpContext.getResponse();
    
    // Bắt đầu đếm thời gian
    const end = httpRequestDuration.startTimer();
    
    return next.handle().pipe(
      tap(() => {
        const route = request.route?.path || request.originalUrl;
        const statusCode = response.statusCode;
        
        const labels = { 
          method: request.method, 
          route: route, 
          status_code: statusCode 
        };

        // Ghi nhận metrics
        httpRequestsTotal.inc(labels);
        end(labels); // Kết thúc đếm thời gian
      }),
    );
  }
}
// Sau đó: app.useGlobalInterceptors(new MetricsInterceptor());
```

**Code mẫu 2: `prometheus-rules.yml` (Cấu hình Alert cho Ops/Security)**

- _Mục tiêu:_ Cảnh báo khi phát hiện dấu hiệu bất thường (vd: tấn công Brute-force).
    

YAML

```YAML
groups:
- name: ApplicationSecurityAlerts
  rules:
  
  # 1. Cảnh báo khi tỷ lệ lỗi 5xx (Server Error) cao
  - alert: HighErrorRate
    expr: |
      sum(rate(http_requests_total{status_code=~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m]))
      > 0.05 # Lớn hơn 5% trong 5 phút
    for: 2m # Phải kéo dài 2 phút
    labels:
      severity: critical
    annotations:
      summary: "Ty le loi 5xx cao bat thuong ({{ $value | printf '%.2f' }}%)"

  # 2. Cảnh báo Security (user memory: AuthN/AuthZ)
  # Phát hiện tấn công Brute-force hoặc quét lỗi bảo mật
  - alert: HighAuthFailureRate
    expr: |
      # Đếm tổng số lỗi 401 (Unauthorized) và 403 (Forbidden)
      sum(rate(http_requests_total{status_code=~"401|403"}[5m])) > 10
      # Lớn hơn 10 request/giây, trong 5 phút
    for: 1m
    labels:
      severity: high
    annotations:
      summary: "Phat hien ty le loi 401/403 cao!"
      description: "Co dau hieu tan cong Brute-force hoac quet loi phan quyen. ({{ $value }} req/s)"

  # 3. Cảnh báo Latency cao (P99)
  - alert: HighRequestLatency
    expr: |
      # Lấy P99 latency (từ Histogram)
      histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route))
      > 1.5 # P99 lớn hơn 1.5 giây
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "P99 latency cao bat thuong ({{ $value }}s)"
```
