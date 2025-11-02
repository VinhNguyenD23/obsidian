### Giai đoạn 3: Chuyển đổi (Microservices)

#### 9. Decomposition (Chia tách)

Đây là một bước tái cấu trúc. Không có code mới, mà là _tổ chức_ lại code.

**Cấu trúc Monolith (Cũ):**

```
/src
  /auth
  /order
    - order.controller.ts (Xử lý /orders)
    - order.service.ts
  /product
    - product.controller.ts (Xử lý /products)
    - product.service.ts
  /prisma
  - app.module.ts
```

**Cấu trúc Microservices (Mới - Dùng NestJS Monorepo):**

```
/apps
  /api-gateway (App 1 - Hướng ra ngoài)
    /src
      - app.module.ts
      - auth.guard.ts (Sẽ validate JWT)
      - proxy.controller.ts (Proxy request)
      
  /order-service (App 2 - Service nghiệp vụ)
    /src
      - order.controller.ts (Chỉ nhận request nội bộ/gRPC)
      - order.service.ts
      - prisma.service.ts
      - main.ts (Chạy trên port 3001)
      
  /product-service (App 3 - Service nghiệp vụ)
    /src
      - product.controller.ts (Chỉ nhận request nội bộ/gRPC)
      - product.service.ts
      - prisma.service.ts
      - main.ts (Chạy trên port 3002)
```

- **Database:** (Rất quan trọng) `order-service` và `product-service` giờ sẽ có **database riêng** của chúng. `order-service` _không được_ truy cập thẳng vào bảng `Product` của `product-service`.
    

---

#### 11. Inter-Service Communication (Giao tiếp liên service)

Đây là cốt lõi. Làm sao `OrderService` (cần tạo đơn hàng) lấy được giá và kiểm tra tồn kho từ `ProductService`? Chúng ta sẽ triển khai cả 2 cách: **Async** (cho dữ liệu ít thay đổi) và **Sync** (cho dữ liệu phải chính xác ngay lập tức).

##### Cách 1: Async (Event-Driven) - Dùng RabbitMQ

- _Mục tiêu:_ `OrderService` cần biết giá sản phẩm. Thay vì _hỏi_ `ProductService` mỗi lần, `OrderService` sẽ _lưu một bản sao_ (copy) của bảng Product. Khi `ProductService` cập nhật giá, nó sẽ _phát một sự kiện_ (event) để `OrderService` cập nhật bản sao của mình.
    

**`product-service` (Người phát - Publisher)**

- _Cài đặt:_ `npm i @nestjs/microservices amqp-connection-manager amqplib`
    

**`apps/product-service/src/main.ts` (Khởi tạo RabbitMQ Publisher)**

TypeScript

```TypeScript
import { NestFactory } from '@nestjs/core';
import { ProductModule } from './product.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.create(ProductModule);

  // Kết nối đến RabbitMQ để CÓ THỂ PHÁT event
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'product_events_queue', // Tên queue (không bắt buộc cho publisher)
      queueOptions: { durable: false },
    },
  });

  await app.startAllMicroservices();
  await app.listen(3002); // Chạy HTTP server (cho Gateway gọi vào)
}
bootstrap();
```

**`apps/product-service/src/product.service.ts` (Phát sự kiện)**

TypeScript

```TypeScript
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class ProductService {
  constructor(
    @Inject('PRODUCT_SERVICE_RMQ') private client: ClientProxy,
    private prisma: PrismaService, // Prisma của ProductService
  ) {}

  async updatePrice(productId: string, newPrice: number) {
    const product = await this.prisma.product.update({
      where: { id: productId },
      data: { price: newPrice },
    });

    // PHÁT SỰ KIỆN (Event-Driven)
    // .emit() là "fire-and-forget", không cần chờ ai nhận
    this.client.emit('product_price_changed', {
      productId: product.id,
      newPrice: product.price,
    });

    return product;
  }
}
```

**`order-service` (Người nghe - Subscriber)**

- _Mô hình DB:_ `OrderService` bây giờ có 1 bảng `ProductCopy` (chỉ chứa `id`, `name`, `price`).
    

**`apps/order-service/src/main.ts` (Khởi tạo RabbitMQ Subscriber)**

TypeScript

```TypeScript
import { NestFactory } from '@nestjs/core';
import { OrderModule } from './order.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.create(OrderModule);

  // Kết nối đến RabbitMQ để LẮNG NGHE event
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'order_service_queue', // Queue riêng của OrderService
      queueOptions: { durable: true }, // Nên 'durable'
      prefetchCount: 1, // Xử lý 1 msg/lần
    },
  });

  await app.startAllMicroservices();
  await app.listen(3001);
}
bootstrap();
```

**`apps/order-service/src/product.controller.ts` (Lắng nghe sự kiện)**

- _Lưu ý:_ Đây là Controller trong `order-service` _không phải_ để xử lý HTTP, mà để xử lý Message.
    

TypeScript

```TypeScript
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';
import { PrismaService } from './prisma.service'; // Prisma của OrderService

@Controller()
export class ProductEventController {
  constructor(private prisma: PrismaService) {}

  // Lắng nghe sự kiện 'product_price_changed'
  @EventPattern('product_price_changed')
  async handleProductPriceChanged(@Payload() data: { productId: string; newPrice: number }) {
    console.log(`Nhận được sự kiện cập nhật giá: ${data.productId}`);
    
    // Cập nhật bản sao (copy) của Product trong DB của OrderService
    await this.prisma.productCopy.update({
      where: { id: data.productId },
      data: { price: data.newPrice },
    });
  }
}
```

##### Cách 2: Sync (Đồng bộ) - Dùng gRPC

- _Mục tiêu:_ Khi tạo đơn hàng (Mục 3), `OrderService` phải _kiểm tra tồn kho (stock)_ tại _thời điểm đó_. Dùng bản sao (Async) là quá rủi ro (có thể bị stale). Chúng ta phải gọi Sync.
    
- _Công cụ:_ gRPC (Hiệu năng cao hơn REST cho giao tiếp nội bộ).
    

**Bước 1: Tạo file `.proto`**

- Đây là "hợp đồng" (contract) định nghĩa service.
    

**`proto/product.proto`**

Protocol Buffers

```TypeScript
syntax = "proto3";

package product;

// Service 'Product'
service ProductService {
  // RPC 'CheckStock'
  rpc CheckStock (CheckStockRequest) returns (CheckStockResponse);
}

// Dữ liệu request
message CheckStockRequest {
  string productId = 1;
  int32 quantity = 2;
}

// Dữ liệu response
message CheckStockResponse {
  bool isAvailable = 1;
  string productId = 2;
  int32 currentStock = 3;
}
```

**Bước 2: `product-service` (Implement gRPC Server)**

- _Cài đặt:_ `npm i @grpc/grpc-js @grpc/proto-loader`
    

**`apps/product-service/src/main.ts` (Chạy gRPC Server)**

TypeScript

```TypeScript
import { NestFactory } from '@nestjs/core';
import { ProductModule } from './product.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { join } from 'path';

async function bootstrap() {
  // Chạy gRPC server (KHÔNG chạy HTTP)
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    ProductModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'product', // Tên package trong .proto
        protoPath: join(__dirname, '..', '..', '..', 'proto/product.proto'), // Đường dẫn tới file .proto
        url: 'localhost:50051', // Port cho gRPC
      },
    },
  );
  await app.listen();
}
bootstrap();
```

**`apps/product-service/src/product.controller.ts` (Implement RPC)**

TypeScript

```TypeScript
import { Controller } from '@nestjs/common';
import { GrpcMethod } from '@nestjs/microservices';
import { ProductService } from './product.service';

interface CheckStockRequest {
  productId: string;
  quantity: number;
}

interface CheckStockResponse {
  isAvailable: boolean;
  productId: string;
  currentStock: number;
}

@Controller()
export class ProductGrpcController {
  constructor(private productService: ProductService) {}

  // Implement gRPC method 'CheckStock'
  @GrpcMethod('ProductService', 'CheckStock')
  async checkStock(data: CheckStockRequest): Promise<CheckStockResponse> {
    const product = await this.productService.findOne(data.productId); // Giả định
    
    if (!product) {
      return { isAvailable: false, productId: data.productId, currentStock: 0 };
    }

    const isAvailable = product.stock >= data.quantity;
    return {
      isAvailable: isAvailable,
      productId: data.productId,
      currentStock: product.stock,
    };
  }
}
```

**Bước 3: `order-service` (Sử dụng gRPC Client)**

- _Mục tiêu:_ Sửa lại `OrderService` (Mục 3) để gọi gRPC `CheckStock` thay vì query DB trực tiếp.
    

**`apps/order-service/src/order.module.ts` (Đăng ký gRPC Client)**

TypeScript

```TypeScript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { join } from 'path';
// ...
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'PRODUCT_PACKAGE', // Tên để Inject
        transport: Transport.GRPC,
        options: {
          package: 'product',
          protoPath: join(__dirname, '..', '..', '..', 'proto/product.proto'),
          url: 'localhost:50051', // Địa chỉ gRPC server
        },
      },
    ]),
    // ...
  ],
  controllers: [OrderController],
  providers: [OrderService],
})
export class OrderModule {}
```

**`apps/order-service/src/order.service.ts` (Sửa lại logic `createOrder`)**

TypeScript

```TypeScript
import { Injectable, Inject, OnModuleInit, InternalServerErrorException } from '@nestjs/common';
import { ClientGrpc } from '@nestjs/microservices';
import { PrismaService } from './prisma.service'; // Prisma của OrderService
import { CreateOrderDto } from './dto/create-order.dto';

// Interface cho gRPC client (từ .proto)
interface ProductServiceClient {
  checkStock(request: { productId: string; quantity: number }): Promise<{ isAvailable: boolean }>;
}

@Injectable()
export class OrderService implements OnModuleInit {
  private productServiceClient: ProductServiceClient;

  constructor(
    @Inject('PRODUCT_PACKAGE') private client: ClientGrpc,
    private prisma: PrismaService, // Prisma của OrderService
  ) {}

  onModuleInit() {
    // Khởi tạo gRPC client
    this.productServiceClient = this.client.getService<ProductServiceClient>('ProductService');
  }

  async createOrder(createOrderDto: CreateOrderDto) {
    const { userId, items } = createOrderDto;

    // 1. Lấy giá từ bản sao (copy) - Dữ liệu Async (từ RabbitMQ)
    const productCopies = await this.prisma.productCopy.findMany({
      where: { id: { in: items.map((i) => i.productId) } },
    });

    try {
      return await this.prisma.$transaction(async (tx) => {
        // 2. KIỂM TRA KHO (SYNC) - Dùng gRPC
        for (const item of items) {
          const stockCheck = await this.productServiceClient.checkStock({
            productId: item.productId,
            quantity: item.quantity,
          });

          if (!stockCheck.isAvailable) {
            throw new InternalServerErrorException(`Không đủ hàng cho ${item.productId}`);
          }
        }
        
        // 3. Tính tiền (dùng giá từ bản sao)
        let totalAmount = 0;
        // ... (logic tính tiền) ...
        
        // 4. Tạo Order (trong DB của OrderService)
        const order = await tx.order.create({ /* ... */ });

        // 5. [QUAN TRỌNG] Trừ kho
        // Chúng ta KHÔNG thể trừ kho trực tiếp (vì khác DB)
        // Chúng ta phải gọi 1 gRPC/RabbitMQ khác để YÊU CẦU ProductService trừ kho
        // Đây chính là khởi đầu của SAGA PATTERN (Mục 12)
        // Hiện tại, ta sẽ tạm bỏ qua bước này để tập trung vào gRPC read

        return order;
      });
    } catch (error) {
      throw new InternalServerErrorException('Lỗi tạo đơn hàng', error.message);
    }
  }
}
```

---

#### 10. API Gateway

- _Mục tiêu:_ Là điểm vào duy nhất (`api.example.com`). Nó xác thực (AuthN), sau đó "chuyển tiếp" (proxy) request đến service tương ứng.
    

**`apps/api-gateway/src/app.module.ts`**

- _Cài đặt:_ `npm i @nestjs/axios`
    

TypeScript

```TypeScript
import { Module } from '@nestjs/common';
import { HttpModule } from '@nestjs/axios';
import { ProxyController } from './proxy.controller';
// ... Import AuthModule (Giả định)

@Module({
  imports: [
    HttpModule.register({
      timeout: 5000,
    }),
    // AuthModule, // Module xác thực JWT
  ],
  controllers: [ProxyController],
})
export class AppModule {}
```

**`apps/api-gateway/src/proxy.controller.ts` (Proxy mỏng)**

TypeScript

```TypeScript
import { Controller, All, Req, Res, UseGuards } from '@nestjs/common';
import { Request, Response } from 'express';
import { HttpService } from '@nestjs/axios';
import { JwtAuthGuard } from './auth/jwt-auth.guard'; // (user memory: AuthN)
import { firstValueFrom } from 'rxjs';

@Controller()
export class ProxyController {
  // Định nghĩa địa chỉ các service nội bộ
  private orderServiceUrl = 'http://localhost:3001';
  private productServiceUrl = 'http://localhost:3002';

  constructor(private httpService: HttpService) {}

  // Bắt tất cả request đến /orders/*
  @UseGuards(JwtAuthGuard) // Bảo vệ tất cả API
  @All('orders/*')
  async proxyOrderService(@Req() req: Request, @Res() res: Response) {
    const url = `${this.orderServiceUrl}${req.originalUrl}`;
    await this.proxyRequest(req, res, url);
  }

  // Bắt tất cả request đến /products/*
  @UseGuards(JwtAuthGuard)
  @All('products/*')
  async proxyProductService(@Req() req: Request, @Res() res: Response) {
    const url = `${this.productServiceUrl}${req.originalUrl}`;
    await this.proxyRequest(req, res, url);
  }
  
  // Hàm proxy chung
  private async proxyRequest(req: Request, res: Response, url: string) {
    try {
      const { status, data, headers } = await firstValueFrom(
        this.httpService.request({
          method: req.method as any,
          url: url,
          data: req.body,
          headers: { 
            'X-User-Id': (req.user as any)?.userId, // Gửi thông tin user đã AuthN
            'Content-Type': req.headers['content-type'],
          },
        }),
      );
      res.status(status).json(data);
    } catch (error) {
      res
        .status(error.response?.status || 500)
        .json(error.response?.data || 'Proxy error');
    }
  }
}
```
