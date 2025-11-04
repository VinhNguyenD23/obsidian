### Giai đoạn 1: Nền tảng & Monolith

#### 1. Core Principles (Nguyên tắc cốt lõi)

Đây là bước xác định yêu cầu, **không có code mẫu** cụ thể. Đây là giai đoạn thảo luận (architectural discussion).

- **Ví dụ thảo luận:** "Hệ thống cần P99 latency dưới 200ms (Performance) và chấp nhận eventual consistency cho việc cập nhật số lượng 'view' sản phẩm (Trade-off)."
    

#### 2. Thiết kế Monolith (Layered)

Chúng ta áp dụng kiến trúc Controller → Service → Repository (Repository ở đây được Prisma Client trừu tượng hóa).

**`src/order/dto/create-order.dto.ts` (DTO & Validation)**

- _Mục tiêu:_ Validate input (user memory: Input validated/sanitized).
    

TypeScript

```typescript
// DTO dùng class-validator (user memory)
import { IsNotEmpty, IsUUID, IsInt, Min, ArrayMinSize, ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

class OrderItemDto {
  @IsUUID()
  productId: string;

  @IsInt()
  @Min(1)
  quantity: number;
}

export class CreateOrderDto {
  @IsUUID()
  @IsNotEmpty()
  userId: string;

  @ArrayMinSize(1)
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];
}
```

**`src/order/order.controller.ts` (Controller mỏng)**

- _Mục tiêu:_ Chỉ routing, validate DTO, và gọi Service (user memory: controller mỏng).
    

TypeScript

```typescript
import { Controller, Post, Body, UseGuards } from '@nestjs/common';
import { OrderService } from './order.service';
import { CreateOrderDto } from './dto/create-order.dto';
import { JwtAuthGuard } from '../auth/jwt-auth.guard'; // Giả định đã có Auth
import { RolesGuard } from '../auth/roles.guard'; // Giả định đã có AuthZ

@Controller('orders')
export class OrderController {
  constructor(private readonly orderService: OrderService) {}

  @UseGuards(JwtAuthGuard, RolesGuard) // user memory: AuthN/AuthZ
  // @Roles('customer') // Giả định kiểm tra role
  @Post()
  async create(@Body() createOrderDto: CreateOrderDto) {
    // Controller không có logic nghiệp vụ
    // DTO đã được validate tự động bởi ValidationPipe của NestJS
    return this.orderService.createOrder(createOrderDto);
  }
}
```

#### 3. Database (Transaction trong Monolith)

Nghiệp vụ "Tạo đơn hàng" phải là một **transaction**: 1. Tạo `Order` record. 2. Trừ `Stock` (kho). Cả hai phải cùng thành công hoặc thất bại.

**`src/order/order.service.ts` (Service & Transaction)**

- _Mục tiêu:_ Chứa logic nghiệp vụ và đảm bảo tính toàn vẹn dữ liệu (user memory: ưu tiên transaction).
    

TypeScript

```typescript
import { Injectable, InternalServerErrorException, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service'; // Giả định bạn có PrismaService
import { CreateOrderDto } from './dto/create-order.dto';
import { Prisma } from '@prisma/client';

@Injectable()
export class OrderService {
  constructor(private prisma: PrismaService) {}

  async createOrder(createOrderDto: CreateOrderDto) {
    const { userId, items } = createOrderDto;

    // 1. Lấy thông tin sản phẩm và giá (để tính tổng tiền)
    const productIds = items.map((item) => item.productId);
    const products = await this.prisma.product.findMany({
      where: { id: { in: productIds } },
    });

    if (products.length !== productIds.length) {
      // user memory: not-found dùng domain exception
      throw new NotFoundException('Một số sản phẩm không tồn tại');
    }

    // 2. Thực hiện nghiệp vụ trong một transaction
    // (user memory: ưu tiên Postgres + transaction)
    try {
      return await this.prisma.$transaction(async (tx) => {
        // Biến 'tx' (Prisma Transaction Client)
        // Nó đảm bảo mọi lệnh dùng 'tx' đều nằm trong transaction

        // 2a. Kiểm tra kho và tính tổng tiền
        let totalAmount = 0;
        for (const item of items) {
          const product = products.find((p) => p.id === item.productId);
          if (product.stock < item.quantity) {
            throw new InternalServerErrorException(`Không đủ hàng cho ${product.name}`);
          }
          totalAmount += product.price * item.quantity;
        }

        // 2b. Tạo record Order
        const order = await tx.order.create({
          data: {
            userId: userId,
            totalAmount: totalAmount,
            items: {
              create: items.map((item) => ({
                productId: item.productId,
                quantity: item.quantity,
                price: products.find((p) => p.id === item.productId).price,
              })),
            },
          },
          include: { items: true },
        });

        // 2c. Trừ kho (Update nhiều record)
        const updateStockPromises = items.map((item) =>
          tx.product.update({
            where: { id: item.productId },
            data: { stock: { decrement: item.quantity } },
          }),
        );
        await Promise.all(updateStockPromises);

        return order;
      });
    } catch (error) {
      // Nếu bất kỳ lỗi nào xảy ra ở trên, $transaction sẽ tự động ROLLBACK
      console.error(error);
      throw new InternalServerErrorException('Không thể tạo đơn hàng', error.message);
    }
  }
}
```

#### 4. Caching (Bộ đệm)

Chúng ta sẽ cache các API `GET` tốn kém (ví dụ: `GET /products/:id`) dùng Redis. Cách làm sạch nhất trong NestJS là dùng **Interceptor**.

**`src/cache/cache.interceptor.ts` (Caching Interceptor)**

TypeScript

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';
import { Cache } from 'cache-manager'; // NestJS tích hợp sẵn
import { CACHE_MANAGER, Inject } from '@nestjs/common';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

  async intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Promise<Observable<any>> {
    const http = context.switchToHttp();
    const request = http.getRequest();

    // Chỉ cache GET request
    if (request.method !== 'GET') {
      return next.handle();
    }

    // Tạo cache key (ví dụ: 'cache:/products/123-abc')
    const key = `cache:${request.originalUrl}`;
    
    // 1. Thử lấy từ Cache (Cache-Aside Pattern)
    const cachedResponse = await this.cacheManager.get(key);
    if (cachedResponse) {
      // Cache Hit
      return of(cachedResponse); // Trả về ngay, không cần gọi service
    }

    // 2. Cache Miss
    // Gọi 'next.handle()' để chạy logic (controller -> service)
    return next.handle().pipe(
      tap(async (response) => {
        // 3. Ghi kết quả vào cache trước khi trả về
        // Set TTL (Time-to-Live) là 5 phút (300 giây)
        await this.cacheManager.set(key, response, { ttl: 300 });
      }),
    );
  }
}
```

**`src/product/product.controller.ts` (Sử dụng Interceptor)**

TypeScript

```typescript
import { Controller, Get, Param, UseInterceptors } from '@nestjs/common';
import { ProductService } from './product.service';
import { CacheInterceptor } from '../cache/cache.interceptor'; // Import interceptor

@Controller('products')
export class ProductController {
  constructor(private readonly productService: ProductService) {}

  @Get(':id')
  @UseInterceptors(CacheInterceptor) // Áp dụng caching cho API này
  async findOne(@Param('id') id: string) {
    // Lần đầu, code này sẽ chạy và cache
    // Lần 2 (trong 5 phút), code này KHÔNG chạy, data lấy từ Redis
    return this.productService.findOne(id);
  }
}
```

---

### Giai đoạn 2: Mở rộng Monolith

#### 5. Vertical vs. Horizontal Scaling (Stateless App)

Để scale ngang (Horizontal), app phải là **Stateless**. Code mẫu ở đây là "cách không nên làm" (Stateful) và "cách nên làm" (Stateless).

**`BadPractice.controller.ts` (STATEFUL - KHÔNG NÊN DÙNG)**

- _Vấn đề:_ Dữ liệu `views` lưu trong RAM. Nếu scale ngang (chạy 2 instance), LB (Load Balancer) trỏ request 1 vào Instance A (views=1), request 2 vào Instance B (views=1). Tổng views là 1 (sai, phải là 2).
    

TypeScript

```typescript
@Controller('bad-product')
export class BadProductController {
  // Lỗi: Lưu state (trạng thái) trong bộ nhớ service
  private views: number = 0; 

  @Get(':id/view')
  getView(@Param('id') id: string) {
    this.views++;
    return `Product ${id} has ${this.views} views (on this instance)`;
  }
}
```

**`GoodPractice.controller.ts` (STATELESS - NÊN DÙNG)**

- _Giải pháp:_ Đẩy State (trạng thái) ra ngoài (DB hoặc Redis).
    

TypeScript

```typescript
@Controller('good-product')
export class GoodProductController {
  // KHÔNG lưu state ở đây
  constructor(private prisma: PrismaService) {}

  @Get(':id/view')
  async getView(@Param('id') id: string) {
    // Mọi state được đọc/ghi từ nguồn bên ngoài (DB/Cache)
    const product = await this.prisma.product.update({
      where: { id: id },
      data: { views: { increment: 1 } },
    });
    return `Product ${id} has ${product.views} views (total)`;
  }
}
```

#### 6. Load Balancer (LB)

Đây là file **config của hạ tầng (Infra)**, không phải code app.

**`nginx.conf` (Ví dụ config Nginx làm LB)**

- _Mục tiêu:_ Phân phối traffic đến 2 instance app (stateless) đang chạy ở port 3000 và 3001.
    

Nginx

```nginx
# Định nghĩa nhóm các server (app instances)
upstream my_nest_app {
    # server 127.0.0.1:3000; # Instance 1
    # server 127.0.0.1:3001; # Instance 2
    # (Đây là IP nội bộ của các container/VM)
    server app_instance_1_ip:3000;
    server app_instance_2_ip:3000;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        # Chuyển tiếp request đến nhóm 'upstream'
        proxy_pass http://my_nest_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Cấu hình health check (tùy chọn)
    }
}
```

#### 7. Database Scaling (Read Replicas)

Đây là một config phức tạp. **Prisma** không hỗ trợ Read Replicas một cách tường minh ở cấp client. Bạn phải dùng proxy (như PgBouncer).

Tuy nhiên, **TypeORM** (user memory) hỗ trợ điều này rất tốt. Đây là ví dụ config TypeORM:

**`app.module.ts` (Ví dụ config TypeORM cho Read Replicas)**

TypeScript

```typescript
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      // Cấu hình REPLICATION
      replication: {
        // MASTER (Cho WRITE: INSERT, UPDATE, DELETE)
        master: {
          host: 'master.db.host',
          port: 5432,
          username: 'write_user',
          password: 'password',
          database: 'mydb',
        },
        // REPLICAS (Cho READ: SELECT)
        slaves: [
          {
            host: 'replica1.db.host',
            port: 5432,
            username: 'read_user',
            password: 'password',
            database: 'mydb',
          },
          {
            host: 'replica2.db.host',
            port: 5432,
            username: 'read_user',
            password: 'password',
            database: 'mydb',
          },
        ],
      },
      // ...
    }),
  ],
})
export class AppModule {}
```

- Với config này, TypeORM sẽ tự động:
    
    - Chạy query `UPDATE`/`INSERT`/`DELETE` (hoặc query trong transaction) trên `master`.
        
    - Chạy query `SELECT` (không nằm trong transaction) trên các `slaves` (theo kiểu round-robin).
        

#### 8. Message Queue (MQ) / Async

Chúng ta sẽ dùng **NestJS + BullMQ** (user memory: `rate-limiter`, `pino`... ngụ ý bạn biết về các thư viện hệ sinh thái Nest).

Khi user đặt hàng (Mục 3), chúng ta không muốn API bị "treo" để chờ gửi email. Chúng ta sẽ đẩy việc đó vào Queue.

**`src/app.module.ts` (Cấu hình BullMQ)**

TypeScript

```typescript
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';

@Module({
  imports: [
    BullModule.forRoot({
      // Cấu hình Redis (BullMQ dùng Redis)
      redis: {
        host: 'localhost',
        port: 6379,
      },
    }),
    // Đăng ký queue
    BullModule.registerQueue({
      name: 'email-queue', // Tên queue
    }),
    // ...
  ],
})
export class AppModule {}
```

**`src/order/order.service.ts` (Sửa lại - Producer)**

- _Mục tiêu:_ API trả về ngay sau khi publish message.
    

TypeScript

```typescript
import { Injectable } from '@nestjs/common';
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class OrderService {
  // Tiêm Queue vào Service
  constructor(
    @InjectQueue('email-queue') private emailQueue: Queue,
    private prisma: PrismaService,
  ) {}

  async createOrder(createOrderDto: CreateOrderDto) {
    // ... (Toàn bộ logic $transaction từ Mục 3 ở đây) ...
    const order = await this.prisma.$transaction(async (tx) => {
      // ... logic tạo order và trừ kho ...
      return createdOrder;
    });

    // Sau khi transaction THÀNH CÔNG
    // Đẩy việc gửi email vào Queue
    await this.emailQueue.add(
      'send-order-confirmation', // Tên job
      { // Dữ liệu (payload)
        orderId: order.id,
        userEmail: 'user@example.com', // Giả định lấy được email
      },
      { 
        attempts: 3, // Thử lại 3 lần nếu thất bại
        backoff: 5000 // Thử lại sau 5 giây
      }
    );

    // API trả về cho user ngay lập tức, không cần chờ email gửi
    return order; 
  }
}
```

**`src/email/email.consumer.ts` (Worker - Consumer)**

- _Mục tiêu:_ Một tiến trình riêng, lắng nghe queue và xử lý (gửi email).
    

TypeScript

```typescript
import { Process, Processor } from '@nestjs/bull';
import { Job } from 'bull';
import { EmailService } from './email.service'; // Giả định có 1 service gửi mail

// 'email-queue' là tên queue đã đăng ký
@Processor('email-queue') 
export class EmailConsumer {
  constructor(private emailService: EmailService) {}

  // Lắng nghe job có tên 'send-order-confirmation'
  @Process('send-order-confirmation')
  async handleSendOrderEmail(job: Job) {
    const { orderId, userEmail } = job.data;
    
    console.log(`Bắt đầu gửi email cho Order ${orderId}...`);
    try {
      // Logic gửi email (có thể mất 3 giây)
      await this.emailService.sendMail(
        userEmail,
        'Xác nhận đơn hàng',
        `Cảm ơn đã đặt hàng ${orderId}`,
      );
      console.log(`Gửi email cho Order ${orderId} thành công.`);
    } catch (error) {
      console.error(`Gửi email cho Order ${orderId} thất bại`, error);
      throw error; // Throw error để BullMQ biết và retry (thử lại)
    }
  }
}
```
