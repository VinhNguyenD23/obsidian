### Giai đoạn 4: Vận hành Microservices

#### 12. Data Consistency (Saga Pattern)

Chúng ta sẽ implement **Saga Pattern** theo kiểu **Choreography** (dùng Message Bus - RabbitMQ) để điều phối nghiệp vụ "Đặt hàng" qua nhiều service.

**Luồng nghiệp vụ (Saga):**

1. Client `POST /orders` -> `API Gateway`.
    
2. `Gateway` -> `OrderService`: `createOrder`.
    
3. `OrderService`: Tạo Order với status `PENDING` & Phát (emit) event `order_created`.
    
4. `ProductService`: Nghe event `order_created` -> Thử trừ kho.
    
5. `ProductService`:
    
    - Nếu thành công: Phát event `stock_deducted`.
        
    - Nếu thất bại: Phát event `stock_deduction_failed` (ví dụ: hết hàng).
        
6. `OrderService`:
    
    - Nghe `stock_deducted`: Cập nhật Order status -> `CONFIRMED`.
        
    - Nghe `stock_deduction_failed`: Cập nhật Order status -> `CANCELLED` (**Đây là Compensating Transaction - Giao dịch bù trừ**).
        

---

**`apps/order-service/src/order.service.ts` (Sửa lại `createOrder`)**

- _Mục tiêu:_ Chỉ tạo Order `PENDING` và phát event.
    

TypeScript

```TypeScript
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { PrismaService } from './prisma.service';

@Injectable()
export class OrderService {
  constructor(
    @Inject('SAGA_SERVICE_RMQ') private client: ClientProxy, // Client RabbitMQ
    private prisma: PrismaService,
  ) {}

  async createOrder(createOrderDto: CreateOrderRto) {
    // 1. (Giả định) Lấy giá từ bản sao (ProductCopy)
    // ... (logic tính totalAmount) ...
    
    // 2. Tạo Order với status 'PENDING'
    // KHÔNG dùng $transaction ở đây nữa vì nghiệp vụ đã bị phân tán
    const order = await this.prisma.order.create({
      data: {
        userId: createOrderDto.userId,
        totalAmount: totalAmount,
        status: 'PENDING', // Trạng thái chờ
        items: { create: createOrderDto.items },
      },
    });

    // 3. Phát sự kiện SAGA
    this.client.emit('order_created', {
      orderId: order.id,
      items: createOrderDto.items,
      // (Phải truyền Correlation ID - xem Mục 13)
    });
    
    console.log(`Order ${order.id} PENDING. Emitting 'order_created' event.`);
    return order; // Trả về cho user ngay
  }
}
```

**`apps/product-service/src/saga.controller.ts` (Listener ở ProductService)**

- _Mục tiêu:_ Nghe `order_created` và xử lý nghiệp vụ "Kho".
    

TypeScript

```TypeScript
import { Controller, Inject } from '@nestjs/common';
import { EventPattern, Payload, ClientProxy } from '@nestjs/microservices';
import { PrismaService } from './prisma.service'; // Prisma của ProductService

@Controller()
export class SagaController {
  constructor(
    @Inject('SAGA_SERVICE_RMQ') private client: ClientProxy,
    private prisma: PrismaService,
  ) {}

  @EventPattern('order_created')
  async handleOrderCreated(@Payload() data: { orderId: string; items: any[] }) {
    console.log(`Nhận event 'order_created' cho Order ${data.orderId}`);
    
    try {
      // 4. Thử trừ kho (trong 1 local transaction)
      await this.prisma.$transaction(async (tx) => {
        for (const item of data.items) {
          const product = await tx.product.findUnique({ where: { id: item.productId } });
          if (!product || product.stock < item.quantity) {
            throw new Error(`Hết hàng cho ${item.productId}`);
          }
          
          await tx.product.update({
            where: { id: item.productId },
            data: { stock: { decrement: item.quantity } },
          });
        }
      });

      // 5a. THÀNH CÔNG: Phát event 'stock_deducted'
      console.log(`Trừ kho thành công cho Order ${data.orderId}`);
      this.client.emit('stock_deducted', { orderId: data.orderId });

    } catch (error) {
      // 5b. THẤT BẠI: Phát event 'stock_deduction_failed'
      console.error(`Trừ kho thất bại cho Order ${data.orderId}`, error.message);
      this.client.emit('stock_deduction_failed', {
        orderId: data.orderId,
        reason: error.message,
      });
    }
  }
}
```

**`apps/order-service/src/saga.controller.ts` (Listener ở OrderService)**

- _Mục tiêu:_ Hoàn tất Saga (Confirm hoặc Cancel).
    

TypeScript

```TypeScript
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';
import { PrismaService } from './prisma.service'; // Prisma của OrderService

@Controller()
export class SagaController {
  constructor(private prisma: PrismaService) {}

  // 6a. Nghe 'stock_deducted'
  @EventPattern('stock_deducted')
  async handleStockDeducted(@Payload() data: { orderId: string }) {
    console.log(`SAGA: Order ${data.orderId} -> CONFIRMED`);
    await this.prisma.order.update({
      where: { id: data.orderId },
      data: { status: 'CONFIRMED' },
    });
  }

  // 6b. Nghe 'stock_deduction_failed' (Compensating Transaction)
  @EventPattern('stock_deduction_failed')
  async handleStockDeductionFailed(@Payload() data: { orderId: string; reason: string }) {
    console.warn(`SAGA: Order ${data.orderId} -> CANCELLED (Reason: ${data.reason})`);
    
    // Giao dịch bù trừ: Hủy đơn hàng
    await this.prisma.order.update({
      where: { id: data.orderId },
      data: { status: 'CANCELLED' },
    });
  }
}
```

---

#### 13. Distributed Tracing / Logging (Correlation ID)

- _Vấn đề:_ Làm sao theo dõi 1 request (Saga) đi qua 3-4 service?
    
- _Giải pháp:_ **Correlation ID** (user memory) và **Structured Logging** (user memory: `pino`).
    

**Bước 1: Cấu hình `pino-http` (Structured Logging) trong `main.ts` của MỌI service**

TypeScript

```TypeScript
// apps/api-gateway/src/main.ts (Ví dụ)
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import pino from 'pino-http'; // Dùng pino-http
import { v4 as uuidv4 } from 'uuid';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    // Tắt logger mặc định của Nest
    logger: false, 
  });

  // Sử dụng Pino (user memory)
  app.use(pino({
    // Cấu hình log, ví dụ:
    transport: { target: 'pino-pretty' }, // Cho dev
    
    // Tự động gán Correlation ID
    genReqId: function (req, res) {
      // Lấy ID từ header nếu có, hoặc tạo mới
      const existingId = req.headers['x-correlation-id'];
      if (existingId) return existingId;
      
      const id = uuidv4();
      res.setHeader('x-correlation-id', id); // Trả về cho client
      return id;
    },
    // Đổi tên 'req.id' thành 'correlationId' cho đẹp
    customProps: (req, res) => ({
      correlationId: req.id,
    }),
  }));
  
  await app.listen(3000);
}
bootstrap();
```

- _Kết quả:_ Mọi log (vd: `console.log`) giờ sẽ là **JSON** và tự động chứa `correlationId`.
    

**Bước 2: Truyền (Propagate) Correlation ID**

Chúng ta phải "chuyền tay" ID này qua mọi ranh giới (network boundary).

**`apps/api-gateway/src/proxy.controller.ts` (Sửa lại Mục 10)**

- _Mục tiêu:_ Lấy ID từ `pino` (đã gán vào `req.id`) và gửi cho service tiếp theo.
    

TypeScript

```TypeScript
  // ... bên trong hàm proxyRequest
  private async proxyRequest(req: Request, res: Response, url: string) {
    try {
      const { status, data, headers } = await firstValueFrom(
        this.httpService.request({
          // ...
          headers: { 
            'x-correlation-id': req.id, // Lấy ID từ pino
            // ...
          },
        }),
      );
      res.status(status).json(data);
    } // ...
  }
```

**`apps/order-service/src/order.service.ts` (Sửa lại Mục 12)**

- _Mục tiêu:_ Truyền ID vào Message Queue (RabbitMQ).
    

TypeScript

```TypeScript
  // ...
  // Giả định order.service nhận được req object (hơi phức tạp trong Nest)
  // Cách đơn giản hơn là dùng 1 interceptor để lấy 'x-correlation-id'
  // và lưu vào 1 AsyncLocalStorage (Context)
  //
  // Giả sử ta lấy được correlationId
  async createOrder(createOrderDto: CreateOrderRto, correlationId: string) {
    // ...
    
    // Khi emit event
    this.client.emit('order_created', 
      { // Payload
        orderId: order.id,
        items: createOrderDto.items,
      },
      { // Options (cho RabbitMQ)
        headers: {
          'x-correlation-id': correlationId // TRUYỀN ID VÀO HEADER MESSAGE
        }
      }
    );
    // ...
  }
```

- **Kết quả:** Khi bạn lọc log (trên Kibana/Loki) theo `correlationId: "abc-123"`, bạn sẽ thấy _toàn bộ_ log của Saga đó, từ API Gateway -> OrderService -> ProductService -> quay lại OrderService.
    

---

#### 14. Service Discovery & Config (Kubernetes)

- _Vấn đề:_ Chúng ta đã hardcode IP (vd: `localhost:50051`) ở Mục 11 (gRPC).
    
- _Giải pháp:_ Dùng DNS nội bộ của Kubernetes.
    

**`k8s/product-service.yaml` (File config của K8s)**

- _Mục tiêu:_ Định nghĩa 2 thứ: `Deployment` (chạy app) và `Service` (tạo DNS).
    

YAML

```yaml
# 1. Deployment: Cách chạy app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 2 # Chạy 2 pod (instance)
  template:
    spec:
      containers:
      - name: product-service
        image: my-registry/product-service:1.2.0
        ports:
        - containerPort: 50051 # Port gRPC
        # Lấy DB_URL từ Secret (user memory: env schema-validated)
        envFrom:
        - secretRef:
            name: product-db-secret # Tên của K8s Secret

---
# 2. Service: Tạo DNS nội bộ
apiVersion: v1
kind: Service
metadata:
  name: product-service-dns # Tên DNS
spec:
  # Tên DNS này sẽ trỏ đến các Pod có label app: product-service
  selector:
    app: product-service 
  ports:
  - protocol: TCP
    port: 50051       # Port mà service khác gọi vào
    targetPort: 50051 # Port trên container (ở trên)
```

**`apps/order-service/src/order.module.ts` (Sửa lại Mục 11)**

- _Mục tiêu:_ Dùng tên DNS của K8s, không dùng IP.
    

TypeScript

```TypeScript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'PRODUCT_PACKAGE',
        transport: Transport.GRPC,
        options: {
          package: 'product',
          protoPath: join(/* ... */),
          // THAY ĐỔI Ở ĐÂY:
          // 'localhost:50051' -> 'tên-service-dns.namespace.svc.cluster.local:port'
          url: 'product-service-dns.default.svc.cluster.local:50051',
        },
      },
    ]),
  ],
  // ...
})
export class OrderModule {}
```

---

#### 15. Resiliency (Circuit Breaker)

- _Vấn đề:_ Ở Mục 11, `OrderService` gọi `checkStock` (gRPC Sync). Nếu `ProductService` bị chậm, `OrderService` sẽ bị treo (cascading failure).
    
- _Giải pháp:_ Dùng thư viện `opossum` (một Circuit Breaker) để bọc (wrap) lệnh gọi gRPC.
    

**`apps/order-service/src/order.service.ts` (Sửa lại Mục 11)**

TypeScript

```TypeScript
import { Injectable, Inject, OnModuleInit, InternalServerErrorException } from '@nestjs/common';
import { ClientGrpc } from '@nestjs/microservices';
import CircuitBreaker from 'opossum'; // Import thư viện

// ... (Interface ProductServiceClient)

@Injectable()
export class OrderService implements OnModuleInit {
  private productServiceClient: ProductServiceClient;
  private stockBreaker: CircuitBreaker; // Khai báo Circuit Breaker

  constructor(
    @Inject('PRODUCT_PACKAGE') private client: ClientGrpc,
    // ...
  ) {}

  onModuleInit() {
    this.productServiceClient = this.client.getService<ProductServiceClient>('ProductService');
    
    // Cấu hình Breaker (vd: nếu 50% lỗi trong 10s, "MỞ" (OPEN) cầu dao)
    const options: CircuitBreaker.Options = {
      timeout: 3000, // Lỗi nếu request gRPC quá 3 giây
      errorThresholdPercentage: 50, // Mở nếu 50% request lỗi
      resetTimeout: 10000, // Thử đóng (HALF-OPEN) lại sau 10 giây
    };

    // Bọc hàm gRPC trong Breaker
    // Chúng ta không thể bọc `this.productServiceClient.checkStock` trực tiếp
    // Chúng ta phải tạo 1 hàm async mới để bọc
    const grpcCheckStock = async (request: { productId: string; quantity: number }) => {
      // (Bản thân gRPC client của NestJS (Observable) cần chuyển sang Promise)
      // Giả sử hàm checkStock đã là Promise (như ví dụ Mục 11)
      return this.productServiceClient.checkStock(request);
    };

    this.stockBreaker = new CircuitBreaker(grpcCheckStock, options);
    
    // (Ops/DevOps) Lắng nghe sự kiện
    this.stockBreaker.on('open', () => console.warn('CIRCUIT OPEN: checkStock'));
    this.stockBreaker.on('close', () => console.info('CIRCUIT CLOSE: checkStock'));
    this.stockBreaker.on('fallback', () => console.warn('CIRCUIT FALLBACK: checkStock'));
  }

  async createOrder(createOrderDto: CreateOrderDto) {
    // ...
    try {
      await this.prisma.$transaction(async (tx) => {
        for (const item of items) {
          // THAY ĐỔI Ở ĐÂY:
          // Thay vì gọi: await this.productServiceClient.checkStock(...)
          // Gọi qua Breaker:
          const stockCheck = await this.stockBreaker.fire({
            productId: item.productId,
            quantity: item.quantity,
          });

          if (!stockCheck.isAvailable) {
            throw new InternalServerErrorException(`Không đủ hàng`);
          }
        }
        // ...
      });
    } catch (error) {
      if (error.code === 'EOPENBREAKER') { // Lỗi từ Circuit Breaker
        throw new InternalServerErrorException('Hệ thống sản phẩm đang bận, thử lại sau');
      }
      throw error; // Lỗi nghiệp vụ (vd: hết hàng)
    }
  }
}
```
