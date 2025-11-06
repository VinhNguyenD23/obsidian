# Phân tích Toàn diện các Lý thuyết NestJS Nền tảng cho Lập trình viên Junior

Báo cáo này trình bày chi tiết các lý thuyết và khái niệm kiến trúc cốt lõi của NestJS, được thiết kế đặc biệt cho các lập trình viên ở cấp độ junior. Mục tiêu là cung cấp một sự hiểu biết có cấu trúc, đi từ triết lý thiết kế cơ bản đến các thành phần xử lý cụ thể.

## PHẦN 1: TRIẾT LÝ VÀ KIẾN TRÚC NỀN TẢNG

Phần này giới thiệu các khái niệm ở mức độ cao nhất, giải thích lý do NestJS tồn tại và các quyết định kiến trúc đã định hình nên framework này.

### NestJS là gì? Vượt xa định nghĩa "Framework Node.js"

NestJS là một framework "progressive" (cấp tiến) của Node.js, được thiết kế để xây dựng các ứng dụng phía máy chủ (server-side) hiệu quả, đáng tin cậy và có khả năng mở rộng.1 Nó sử dụng TypeScript toàn phần 2 và theo mặc định, được xây dựng dựa trên thư viện HTTP mạnh mẽ là Express. Ngoài ra, nó cũng có thể được cấu hình để sử dụng Fastify.3

Thuật ngữ "progressive" ngụ ý rằng NestJS cung cấp một cấu trúc mạnh mẽ ngay từ đầu, nhưng vẫn linh hoạt để phát triển cùng với dự án.2 Không giống như các framework "không có chính kiến" (unopinionated) như Express, vốn mang lại sự linh hoạt nhưng thường dẫn đến sự thiếu hụt các giải pháp kiến trúc trong các dự án lớn.3

NestJS ra đời (năm 2017) để giải quyết chính xác vấn đề về kiến trúc này.5 Nó không tìm cách thay thế Express; thay vào đó, nó cung cấp một lớp trừu tượng (abstraction) và kiến trúc mạnh mẽ _bên trên_ Express (hoặc Fastify), mang lại một cấu trúc ứng dụng "sẵn sàng để sử dụng" (ready-to-roll).1 Điều này giúp tổ chức code rõ ràng, dễ bảo trì và mở rộng, đặc biệt là cho các ứng dụng quy mô doanh nghiệp (enterprise-grade).4

### Triết lý Thiết kế: "Angular cho Backend"

Triết lý của NestJS chịu ảnh hưởng sâu sắc và lấy cảm hứng từ Angular.1 Nó áp dụng các mô hình thiết kế đã được kiểm chứng của Angular cho việc phát triển backend. Sự tương đồng này không chỉ dừng lại ở cú pháp mà còn thể hiện ở ba khái niệm kiến trúc chính:

1. **Tính Mô-đun (Modularity):** Toàn bộ ứng dụng được tổ chức thành các Modules, giúp đóng gói và phân định rõ ràng các chức năng.6
    
2. **Dependency Injection (DI):** NestJS triển khai một hệ thống Dependency Injection (Tiêm phụ thuộc) mạnh mẽ, tương tự như Angular.3
    
3. **Decorators:** Framework sử dụng rộng rãi các Decorator của TypeScript (ví dụ: `@Module()`, `@Controller()`, `@Injectable()`) để định nghĩa metadata (siêu dữ liệu).6
    

Bằng cách áp dụng các mẫu kiến trúc này, NestJS cung cấp một "cách làm chuẩn" (opinionated).6 Điều này làm giảm gánh nặng về việc phải đưa ra các quyết định kiến trúc, giúp các nhóm phát triển duy trì sự nhất quán. Các lợi ích trực tiếp của triết lý này bao gồm khả năng mở rộng (scalability), khả năng bảo trì (maintainability) và quan trọng nhất là khả năng kiểm thử (testability).5

### Phân tích Tệp Khởi động: main.ts và NestFactory

Tệp `main.ts` là điểm vào (entry file) của mọi ứng dụng NestJS.2 Tệp này chứa một hàm `bootstrap` bất đồng bộ, chịu trách nhiệm khởi tạo và khởi chạy ứng dụng.

Hàm cốt lõi trong `main.ts` là `NestFactory.create()`.2

TypeScript

```TypeScript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

Việc truyền `AppModule` (module gốc của ứng dụng) vào phương thức `NestFactory.create()` không chỉ đơn giản là để khởi động một máy chủ HTTP.2 Đây là thời điểm NestJS thực sự _xây dựng đồ thị phụ thuộc_ (Dependency Graph) của toàn bộ ứng dụng.8

Quá trình `bootstrap` này chính là lúc Inversion of Control (IoC) container (sẽ được thảo luận trong phần tiếp theo) được khởi tạo. Nest sẽ phân tích `AppModule`, sau đó phân tích đệ quy tất cả các `imports`, `providers`, và `controllers`.1 Nó khởi tạo các provider và "kết nối" (wire up) chúng dựa trên các phụ thuộc đã được khai báo. Chỉ sau khi toàn bộ cây phụ thuộc này được giải quyết và ứng dụng được "biên dịch" trong bộ nhớ, `app.listen()` mới được gọi để bắt đầu nhận các yêu cầu HTTP.

## PHẦN 2: "LÝ THUYẾT" TRUNG TÂM: INVERSION OF CONTROL (IOC) VÀ DEPENDENCY INJECTION (DI)

Đây là những khái niệm lý thuyết quan trọng nhất trong NestJS. Nắm vững chúng là điều kiện tiên quyết để hiểu mọi khía cạnh khác của framework.

### Lý thuyết 1: Inversion of Control (IoC) - Đảo ngược Quyền kiểm soát

Inversion of Control (IoC) là một nguyên tắc thiết kế (design principle).9 Trong lập trình truyền thống (thủ tục), code của lập trình viên sẽ gọi các thư viện để thực thi tác vụ. Ví dụ, lớp của bạn chủ động tạo ra các đối tượng mà nó cần:

TypeScript

```TypeScript
// Cách làm truyền thống (Không IoC)
class MyController {
  private myService: MyService;

  constructor() {
    this.myService = new MyService(); // <-- Quyền kiểm soát nằm ở đây
  }
}
```

Với IoC, luồng kiểm soát này bị đảo ngược.9 Thay vì code của bạn gọi framework, _framework_ sẽ gọi code của bạn.9 Bạn từ bỏ quyền kiểm soát việc tạo và quản lý vòng đời của đối tượng, và giao nó cho một thực thể bên ngoài, thường được gọi là "IoC Container".10

Trong NestJS, thay vì tự tạo `MyService`, bạn chỉ _khai báo_ rằng `MyController` _cần_ một `MyService`. IoC container (runtime của Nest) sẽ chịu trách nhiệm tạo ra `MyService` và "đưa" nó vào `MyController` khi `MyController` được tạo.11 Nguyên tắc này còn được gọi là "Nguyên tắc Hollywood": "Đừng gọi chúng tôi; chúng tôi sẽ gọi bạn."

### Lý thuyết 2: Dependency Injection (DI) - Tiêm (Chích) Phụ thuộc

Dependency Injection (DI) là một _mô hình thiết kế_ (design pattern) cụ thể được sử dụng để _triển khai_ nguyên tắc IoC.10 DI cho phép các đối tượng phụ thuộc được tạo ra _bên ngoài_ một lớp và được _cung cấp_ (tiêm) cho lớp đó.13

Trong NestJS, DI hầu như luôn được thực hiện thông qua **Constructor Injection** (Tiêm qua hàm khởi tạo).11

TypeScript

```TypeScript
// Cách làm DI (với IoC)
@Controller()
export class AppController {
  // Yêu cầu phụ thuộc được khai báo
  // DI sẽ xảy ra ở đây
  constructor(private readonly appService: AppService) {}
}
```

Việc sử dụng DI giải quyết hai vấn đề lớn của cách làm truyền thống:

1. **Khớp nối chặt (Tight Coupling):** Nếu `AppController` tự mình `new AppService()`, nó sẽ bị _phụ thuộc cứng_ vào hiện thực cụ thể của `AppService`.14
    
2. **Khả năng kiểm thử (Testability):** Khi unit test `AppController`, bạn buộc phải chạy cả `AppService` (có thể gọi đến database thật), điều này vi phạm nguyên tắc unit test.14
    

Với DI, `AppController` trở nên _khớp nối lỏng_ (loosely coupled).5 Nó chỉ biết về _sự tồn tại_ của `AppService`. Điều này cho phép dễ dàng hoán đổi các hiện thực (ví dụ: thay thế `AppService` thật bằng `MockAppService` khi kiểm thử).14

### Cơ chế Hoạt động: IoC Container, @Injectable(), và Metadata

Cơ chế DI "phép thuật" của NestJS hoạt động dựa trên sự kết hợp của ba yếu tố:

1. **@Injectable():** Đây là một decorator 16 đánh dấu một class (như `AppService`) là một "Provider". Nó báo cho IoC container của Nest rằng class này có thể được quản lý và tiêm vào các class khác.8
    
2. **Đăng ký Provider:** Class `@Injectable()` phải được đăng ký trong một module. Bằng cách thêm `AppService` vào mảng `providers` của `AppModule` (`providers:`), bạn đang yêu cầu Nest IoC container tạo một "token" và quản lý một thực thể (instance) singleton của `AppService`.8
    
3. **Metadata:** Khi `AppController` yêu cầu `appService: AppService` trong constructor 11, câu hỏi là: Làm thế nào Nest biết được kiểu `AppService` khi code đã được biên dịch sang JavaScript (vốn không có kiểu)?
    

Câu trả lời nằm ở `reflect-metadata`.11 Khi tùy chọn `emitDecoratorMetadata` trong `tsconfig.json` được bật, TypeScript sẽ lưu trữ _siêu dữ liệu_ (metadata) về các kiểu tham số của constructor. Lúc runtime, Nest sử dụng `reflect-metadata` để _đọc_ metadata này, biết rằng tham số ở vị trí có kiểu là `AppService`.11 Sau đó, IoC container tìm token `AppService` đã đăng ký và thực hiện việc tiêm.8

## PHẦN 3: BA TRỤ CỘT KIẾN TRÚC CỦA NESTJS

Mọi ứng dụng NestJS đều được xây dựng từ ba khối cơ bản này: Modules, Providers, và Controllers.1

### Trụ cột 1: Providers (và Services)

"Provider" là một khái niệm cơ bản trong Nest. Hầu hết các lớp Nest cơ bản—services, repositories, factories, helpers—đều có thể được coi là provider.16

**Service** là loại Provider phổ biến nhất. Đây là các class được đánh dấu bằng `@Injectable()`.1 Vai trò chính của chúng là đóng gói **logic nghiệp vụ** (business logic) 1, xử lý dữ liệu, hoặc tương tác với các dịch vụ bên ngoài.

Kiến trúc của Nest khuyến khích mạnh mẽ việc tách biệt logic nghiệp vụ ra khỏi Controller.17 Controller chỉ nên "gầy" (lean), tức là chỉ nhận request và ủy thác công việc.16 Service là nơi chứa _logic thực sự_ ("fat"). Đây là một ứng dụng trực tiếp của **Nguyên tắc Đơn trách nhiệm (Single Responsibility Principle)**: Controller chịu trách nhiệm về HTTP, Service chịu trách nhiệm về Business Logic.

### Trụ cột 2: Controllers

Controllers chịu trách nhiệm xử lý các **yêu cầu HTTP đến** và tạo ra các phản hồi.1 Chúng hoạt động như "giám đốc giao thông" của ứng dụng.7

Một class trở thành Controller khi được đánh dấu bằng decorator `@Controller()` 11, thường nhận một tiền tố đường dẫn (ví dụ: `@Controller('users')`). Các phương thức bên trong controller được liên kết với các endpoint cụ thể bằng các decorator phương thức HTTP như `@Get()`, `@Post()`, `@Put()`, `@Delete()`.11

NestJS sử dụng các decorator này để xây dựng một **bảng định tuyến nội bộ** (internal routing table).11 Vai trò chính của Controller là _tiêu thụ_ (consume) các Service. Chúng _tiêm_ Service vào constructor và gọi các phương thức của Service để hoàn thành yêu cầu.11

### Trụ cột 3: Modules

Modules là các khối xây dựng (building blocks) cơ bản của ứng dụng NestJS.7 Chúng là các class được đánh dấu bằng `@Module()`.7

Mục đích của Module là _tổ chức_ (organize) và _đóng gói_ (encapsulate) các thành phần liên quan (Controllers, Providers).7 Mọi ứng dụng đều có ít nhất một module gốc (root module), là `AppModule`.1

Decorator `@Module()` nhận một đối tượng mô tả cấu trúc của module:

- `providers`: Các provider/service được định nghĩa trong module này.1
    
- `controllers`: Các controller được định nghĩa trong module này.1
    
- `imports`: Danh sách các module khác mà module này cần (để sử dụng các provider đã `export` của chúng).20
    
- `exports`: Các provider mà module này muốn "công khai" cho các module khác sử dụng.20
    

Modules không chỉ là các thư mục tổ chức code. Chúng là một phần cơ bản của hệ thống DI, vì chúng _định nghĩa ranh giới_ (boundaries) và _phạm vi_ (scope).6 Theo mặc định, một `Provider` đăng ký trong `UsersModule` là _private_ và không thể được tiêm ở nơi khác.20 Để `AuthModule` sử dụng `UsersService`, `UsersModule` phải _chủ động_ `export` nó, và `AuthModule` phải `import` `UsersModule`.20 Cơ chế đóng gói này ngăn chặn "spaghetti code" và là chìa khóa cho khả năng mở rộng của ứng dụng.

## PHẦN 4: GIẢI PHẪU VÒNG ĐỜI YÊU CẦU (REQUEST LIFECYCLE)

Hiểu được thứ tự chính xác của các sự kiện khi một yêu cầu HTTP đi vào ứng dụng NestJS là rất quan trọng để gỡ lỗi và triển khai logic một cách chính xác.22

### Sơ đồ Tuần tự của Vòng đời Yêu cầu

Một yêu cầu đến sẽ đi qua một chuỗi các thành phần xử lý theo một thứ tự cố định.22 Sơ đồ tổng quát như sau:

1. **Incoming Request** (Yêu cầu đến)
    
2. **Middleware** (Toàn cục -> Module)
    
3. **Guards** (Toàn cục -> Controller -> Route)
    
4. **Interceptors (Pre-Controller)** (Toàn cục -> Controller -> Route)
    
5. **Pipes** (Toàn cục -> Controller -> Route -> Tham số Route)
    
6. **Controller** (Thực thi Method Handler)
    
7. **Service** (Được gọi bởi Controller)
    
8. **Interceptors (Post-Request)** (Route -> Controller -> Toàn cục) - _Lưu ý: Thứ tự bị đảo ngược!_
    
9. **Exception Filters** (Route -> Controller -> Toàn cục) - _Chỉ chạy nếu có lỗi chưa được xử lý!_
    
10. **Server Response** (Phản hồi đi)
    

Mỗi thành phần trong chuỗi này có một trách nhiệm duy nhất, áp dụng theo **Mô hình Chuỗi Trách nhiệm (Chain of Responsibility)**. Mỗi mắt xích trả lời một câu hỏi cụ thể:

- **Middleware:** "Tôi có cần sửa đổi đối tượng `request` và `response` thô không?" (Nó không biết handler nào sẽ chạy).24
    
- **Guards:** "Yêu cầu này có _được phép_ (authorized) chạy handler này không?".24
    
- **Interceptors (Pre):** "Tôi có cần chạy logic (ví dụ: logging, caching) _trước khi_ handler thực thi không?".26
    
- **Pipes:** "Các _tham số_ (`@Body`, `@Param`) được truyền cho handler có _hợp lệ_ (valid) và _đúng định dạng_ (transformed) không?".27
    

### Bảng: Tóm tắt Vòng đời Yêu cầu

Bảng dưới đây tóm tắt các thành phần chính trong vòng đời yêu cầu, mục đích của chúng và các interface/decorator liên quan.

|**Giai đoạn**|**Thành phần**|**Mục đích chính**|**Interface/Decorator chính**|**Chạy (Trước/Sau)**|
|---|---|---|---|---|
|1|Middleware|Thao tác Request/Response thô, logic chung (ví dụ: CORS, logging)|Function, `NestMiddleware`|Trước|
|2|Guards|Phân quyền (Authorization) - Quyết định "có/không"|`CanActivate`|Trước|
|3|Interceptors|Logic AOP (Logging, Caching, Biến đổi Response)|`NestInterceptor` (`intercept`)|Trước & Sau|
|4|Pipes|Biến đổi (Transformation) & Xác thực (Validation) Tham số|`PipeTransform`|Trước|
|5|Controller / Service|Thực thi logic nghiệp vụ chính|`@Controller`, `@Injectable`|-|
|6|Exception Filters|Xử lý lỗi tập trung và định dạng response lỗi|`ExceptionFilter` (`catch`)|Sau (chỉ khi có lỗi)|

Lưu ý quan trọng về Interceptors là chúng hoạt động theo nguyên tắc **"First In, Last Out" (FILO)**.22 Interceptor toàn cục (Global) là cái đầu tiên chạm vào request đến và là cái cuối cùng chạm vào response đi. Điều này là do chúng _bọc_ (wrap) quá trình thực thi bằng cách sử dụng RxJS Observables.22

## PHẦN 5: PHÂN TÍCH CHUYÊN SÂU CÁC THÀNH PHẦN XỬ LÝ

Phần này đi sâu vào "lý thuyết" của từng thành phần trong vòng đời request.

### Pipes (Ống dẫn): Biến đổi và Xác thực

Pipes là các class `@Injectable` triển khai interface `PipeTransform`.28 Chúng có hai mục đích chính 27:

1. **Transformation (Biến đổi):** Thay đổi dữ liệu đầu vào thành định dạng mong muốn. Ví dụ, `ParseIntPipe` (tích hợp sẵn) nhận một string từ tham số URL (ví dụ: `':id'`) và chuyển nó thành `number`.28 Nếu biến đổi thất bại, nó sẽ ném ra một ngoại lệ.28
    
2. **Validation (Xác thực):** Kiểm tra xem dữ liệu đầu vào có tuân thủ các quy tắc hay không. Ví dụ, `ValidationPipe` (tích hợp sẵn) được sử dụng để xác thực toàn bộ `request body` dựa trên các quy tắc được định nghĩa trong DTOs.30
    

Pipes chạy _trước_ khi handler của controller được gọi.22 Điều này có nghĩa là khi code logic bên trong controller của bạn thực thi, bạn _được đảm bảo_ rằng dữ liệu đã được xác thực và có đúng kiểu. Pipes _bảo vệ_ business logic của bạn khỏi dữ liệu "bẩn" (invalid or malformed data).

### Guards (Bảo vệ): Phân quyền

Guard là một class `@Injectable` triển khai interface `CanActivate`.25 Nó có một trách nhiệm duy nhất: xác định xem một yêu cầu có được tiếp tục hay không.25 Mục đích chính của Guard là **Authorization (Phân quyền)**.24

Phương thức `canActivate()` phải trả về một `boolean` (hoặc `Promise<boolean>`). Nếu `true`, request tiếp tục; nếu `false`, Nest tự động ném ra `ForbiddenException` (lỗi 403).25

Không giống như Middleware (thường dùng cho **Authentication** - Xác thực, ví dụ: giải mã JWT và gắn `req.user`), Guards có quyền truy cập vào `ExecutionContext`.24 Điều này cho phép Guard biết _chính xác_ handler (`context.getHandler()`) và controller (`context.getClass()`) nào sắp được thực thi.24

Điều này cho phép Guard thực hiện **phân quyền theo ngữ cảnh (context-aware authorization)**. Ví dụ, một `RolesGuard` có thể đọc metadata (ví dụ: `@Roles('admin')`) được đính kèm với handler (thông qua `Reflector`) và so sánh nó với vai trò của người dùng (`req.user.roles`) để đưa ra quyết định.25

### Interceptors (Thiết bị chặn): Lập trình Hướng khía cạnh (AOP)

Interceptor là một class `@Injectable` triển khai `NestInterceptor`.26 Chúng được lấy cảm hứng từ kỹ thuật **Aspect-Oriented Programming (AOP)**.26

Interceptors rất mạnh mẽ và có thể:

- Ràng buộc logic _trước_ và _sau_ khi phương thức thực thi (ví dụ: logging thời gian phản hồi).26
    
- Biến đổi kết quả trả về (ví dụ: wrap mọi response trong một đối tượng `{ data:... }`).32
    
- Ghi đè hoàn toàn một hàm (ví dụ: trả về kết quả từ cache thay vì chạy handler).26
    
- Biến đổi ngoại lệ được ném ra.26
    

"Lý thuyết" cốt lõi của Interceptor là **RxJS**. Phương thức `intercept()` nhận vào một `CallHandler`.26 Khi gọi `next.handle()`, nó không trả về một giá trị hay Promise, mà trả về một **RxJS `Observable`** (stream).26

TypeScript

```TypeScript
// logging.interceptor.ts
intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
  console.log('Before...'); // Logic chạy trước
  
  return next
   .handle()
   .pipe(
      tap(() => console.log('After...')), // Logic chạy sau
      map(data => ({ data })), // Biến đổi response
    );
}
```

Bằng cách coi response là một stream, Interceptors cung cấp một cơ chế AOP mạnh mẽ để trừu tượng hóa các "mối quan tâm xuyên suốt" (cross-cutting concerns) ra khỏi business logic.26

### Exception Filters (Bộ lọc Ngoại lệ): Xử lý Lỗi

Nest đi kèm với một lớp xử lý ngoại lệ tích hợp, chịu trách nhiệm xử lý tất cả các ngoại lệ không được xử lý (unhandled exceptions) và gửi về một phản hồi JSON thân thiện.35

Mục đích của Exception Filter là cung cấp một **nơi tập trung** để bắt các ngoại lệ và tùy chỉnh định dạng phản hồi lỗi.36 Thay vì viết các khối `try...catch` lặp đi lặp lại trong mọi controller 38, bạn tạo một bộ lọc duy nhất.

Bạn tạo một class triển khai `ExceptionFilter` và sử dụng decorator `@Catch(ExceptionType)`.37 Ví dụ, `@Catch(HttpException)` sẽ bắt tất cả các lỗi HTTP tiêu chuẩn của Nest (như `NotFoundException`, `ForbiddenException`). Sử dụng `@Catch()` sẽ bắt _mọi_ lỗi.39

Phương thức `catch(exception, host)` cho phép bạn truy cập vào đối tượng lỗi và đối tượng `response`.37 Điều này cho phép tách biệt mối quan tâm: logic nghiệp vụ chỉ cần `throw new NotFoundException('User not found')` 39; lớp Filter sẽ lo phần định dạng response JSON nhất quán cho client.37

## PHẦN 6: "LÝ THUYẾT" VỀ QUẢN LÝ VÀ XÁC THỰC DỮ LIỆU

Phần này giải thích cách NestJS xử lý dữ liệu đi vào ứng dụng, một phần quan trọng của bất kỳ API nào.

### Lý thuyết 1: Data Transfer Objects (DTOs)

DTO (Data Transfer Object) là một mô hình thiết kế được sử dụng để truyền dữ liệu giữa các lớp của ứng dụng hoặc qua mạng.41

Trong NestJS, DTO thường là một **class** (lớp) định nghĩa _cấu trúc_ (structure) hoặc _hình dạng_ (shape) của dữ liệu 41, ví dụ như `request body` (`CreateUserDto`).

Một câu hỏi phổ biến từ các lập trình viên junior là: "Tại sao không dùng `interface` của TypeScript?".44 Câu trả lời là: `interface` trong TypeScript là một cấu trúc _chỉ tồn tại lúc biên dịch_ (compile-time). Chúng bị xóa hoàn toàn khi biên dịch sang JavaScript.45

DTOs trong NestJS là các **class**, và `class` là một tính năng của ES6 _tồn tại lúc chạy_ (runtime).41 Điều này rất quan trọng vì nó cho phép DTO phục vụ một **mục đích kép**:

1. Cung cấp type-safety (an toàn kiểu) lúc _biên dịch_.
    
2. Cung cấp một _schema_ (lược đồ) lúc _runtime_ mà `ValidationPipe` có thể sử dụng để _xác thực_ dữ liệu.30
    

### Lý thuyết 2: Validation với class-validator và class-transformer

NestJS tích hợp liền mạch với hai thư viện mạnh mẽ là `class-validator` (để xác thực) và `class-transformer` (để biến đổi).30

Cơ chế này được kích hoạt tự động khi sử dụng `ValidationPipe` (thường được áp dụng toàn cục trong `main.ts`: `app.useGlobalPipes(new ValidationPipe())`).30

1. **Định nghĩa DTO:** Bạn thêm các decorator từ `class-validator` vào các thuộc tính của DTO class.30
    
    TypeScript
    
    ```TypeScript
    // create-user.dto.ts
    import { IsString, IsEmail, IsNotEmpty } from 'class-validator';
    
    export class CreateUserDto {
      @IsString()
      @IsNotEmpty()
      name: string;
    
      @IsEmail()
      email: string;
    }
    ```
    
2. **Quy trình ẩn (Transform -> Validate):** Khi `ValidationPipe` chạy (với tùy chọn `transform: true` 30), nó thực hiện một chuỗi 2 bước:
    
    - **Bước 1 (Transform):** `class-transformer` chạy trước, lấy đối tượng JavaScript thô (từ JSON) và _chuyển đổi_ nó thành một _thực thể (instance)_ của `CreateUserDto` class.47 Nó cũng cố gắng biến đổi các kiểu nguyên thủy (ví dụ: string "true" thành boolean `true`).46
        
    - **Bước 2 (Validate):** `class-validator` chạy sau, lấy `instance` đã được chuyển đổi và _xác thực_ nó dựa trên các decorator (`@IsString`, `@IsEmail`).46
        

Nếu xác thực thất bại, `ValidationPipe` sẽ tự động ném ra một `BadRequestException` (lỗi 400), ngăn không cho handler của controller được thực thi.30

## PHẦN 7: TRỪU TƯỢNG HÓA DỮ LIỆU VÀ TÍCH HỢP CƠ SỞ DỮ LIỆU

Phần này bao gồm "lý thuyết" về cách NestJS tương tác với cơ sở dữ liệu một cách có cấu trúc.

### Tích hợp ORM/ODM: Mô hình forRoot() / forFeature()

NestJS không phụ thuộc vào cơ sở dữ liệu cụ thể (database agnostic) 49 và cung cấp các gói tích hợp chuyên dụng cho các ORM/ODM phổ biến:

- **SQL (TypeORM):** `@nestjs/typeorm` 49
    
- **NoSQL (Mongoose):** `@nestjs/mongoose` 50
    

Cả hai gói này đều tuân theo một mô hình cấu hình hai cấp độ:

1. **`TypeOrmModule.forRoot({...})` / `MongooseModule.forRoot(...)`**
    
    - **Mục đích:** Thiết lập kết nối cơ sở dữ liệu chính (ví dụ: chuỗi kết nối, host, port).
        
    - **Vị trí:** Được gọi _một lần duy nhất_ trong `imports` của `AppModule` (module gốc).49
        
2. **`TypeOrmModule.forFeature([...])` / `MongooseModule.forFeature(...)`**
    
    - **Mục đích:** Đăng ký các `Entities` (TypeORM) hoặc `Schemas` (Mongoose) cụ thể với module (tính năng) hiện tại.
        
    - **Vị trí:** Được gọi trong `imports` của các _feature module_ (ví dụ: `UserModule`, `ProductModule`).49
        

Mô hình này là một ứng dụng của DI. `forRoot()` khởi tạo _Provider_ kết nối (singleton). `forFeature()` nói với IoC container của module hiện tại: "Hãy sử dụng kết nối `forRoot()` đã có, và đăng ký các repository/model này (`UserEntity`) để chúng _có thể được tiêm (injectable)_ trong phạm vi của module này".53 Điều này cho phép chúng ta tiêm `Repository<User>` vào `UserService` bằng `@InjectRepository(User)`.53

### Entities (TypeORM) và Schemas (Mongoose)

Các decorator này hoạt động như một _lớp ánh xạ_ (mapping layer) giữa thế giới Hướng đối tượng của TypeScript và thế giới Cơ sở dữ liệu.

- **TypeORM:** `Entity` là một class được đánh dấu bằng `@Entity()`.49 Các thuộc tính được ánh xạ tới các _cột_ (columns) trong _bảng_ (table) SQL bằng các decorator như `@Column()`, `@PrimaryGeneratedColumn()`.49
    
- **Mongoose:** `Schema` là một class được đánh dấu bằng `@Schema()`.50 Nó định nghĩa _hình dạng_ (shape) của các _tài liệu_ (documents) trong một _collection_ MongoDB.50 Các thuộc tính được định nghĩa bằng `@Prop()`.50
    

Thay vì viết các câu lệnh `CREATE TABLE` hoặc định nghĩa `mongoose.Schema` thủ công, bạn định nghĩa cấu trúc dữ liệu một lần bằng cách sử dụng các lớp và decorator TypeScript, và ORM/ODM sẽ lo phần còn lại.50

### Lý thuyết 3: Repository Pattern (Mô hình Kho chứa)

Repository Pattern là một mô hình thiết kế nhằm trừu tượng hóa logic truy cập dữ liệu (data access logic) ra khỏi logic nghiệp vụ (business logic).56

**Cấp độ 1 (Tích hợp sẵn):** TypeORM vốn đã hỗ trợ Repository Pattern.49 Bạn tiêm `Repository<User>` (do TypeORM cung cấp) trực tiếp vào `UserService`.53

TypeScript

```TypeScript
// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) // Tiêm Repository của TypeORM
    private usersRepository: Repository<User>,
  ) {}

  findAll() {
    return this.usersRepository.find(); // Service gọi trực tiếp ORM
  }
}
```

**Vấn đề:** Ở cấp độ này, `UserService` bị _khớp nối chặt_ (tightly coupled) với `Repository` của _TypeORM_.57 Nếu bạn muốn đổi sang một ORM khác (ví dụ: Drizzle 56 hoặc Prisma), bạn phải viết lại toàn bộ `UserService`.

**Cấp độ 2 (Mô hình Repository Tùy chỉnh):** Đây là cách triển khai "thuần túy" của mô hình.56

- Luồng dữ liệu là: `Controller` -> `Service` -> `CustomRepository`.
    
- `UserService` chỉ tiêm và biết về `MyUserRepository` (một class `@Injectable` mà bạn tự định nghĩa).
    
- `MyUserRepository` (lớp truy cập dữ liệu của bạn) sẽ tiêm `Repository<User>` (của TypeORM) và triển khai các phương thức (ví dụ: `findAll()`) bằng cách gọi `this.typeOrmRepo.find()`.
    

Mô hình này tạo ra một _lớp trừu tượng hóa_ (abstraction shield).58 Nếu bạn thay đổi cơ sở dữ liệu hoặc ORM 56, bạn _chỉ cần_ viết lại `MyUserRepository`. Toàn bộ `UserService` (logic nghiệp vụ) của bạn vẫn được giữ nguyên, giúp tăng khả năng bảo trì.56

## PHẦN 8: QUẢN LÝ CẤU HÌNH VÀ MÔI TRƯỜNG

Một lý thuyết cuối cùng cho lập trình viên junior là cách quản lý cấu hình (như API keys, chuỗi kết nối database) một cách an toàn và linh hoạt.

### Vấn đề: Hard-coding Biến môi trường

Việc hard-code các giá trị nhạy cảm hoặc thay đổi theo môi trường (development, production) trực tiếp trong code là một thực hành tồi.59 Cách làm phổ biến hơn là sử dụng `process.env.DATABASE_URL`, nhưng điều này cũng tạo ra sự phụ thuộc cứng vào biến môi trường toàn cục `process.env`.

### Giải pháp: @nestjs/config

NestJS cung cấp gói `@nestjs/config` để giải quyết vấn đề này một cách thanh lịch.59

**Cơ chế:**

1. Cài đặt: `npm i @nestjs/config`.60
    
2. Tạo tệp `.env` ở thư mục gốc.59
    
3. Trong `AppModule`, import `ConfigModule.forRoot({ isGlobal: true })`.60
    
    - `forRoot()`: Tải và phân tích (parse) tệp `.env`.60
        
    - `isGlobal: true`: Làm cho `ConfigService` có sẵn ở mọi nơi mà không cần `import` lại.61
        
4. Truy cập biến: Tiêm (inject) `ConfigService` vào bất kỳ `Service` hoặc `Provider` nào và sử dụng `this.configService.get('DATABASE_URL')`.59
    

Bằng cách này, NestJS coi cấu hình (config) như một **dependency** (phụ thuộc) giống như bất kỳ service nào khác. Điều này mang lại lợi ích lớn về **khả năng kiểm thử**. Khi unit test một `DatabaseService` 60, bạn không cần phải thiết lập `process.env` thật. Bạn có thể dễ dàng _giả lập_ (mock) `ConfigService` và cung cấp các giá trị test.60 Điều này làm cho toàn bộ ứng dụng nhất quán với mô hình DI.

Ngoài ra, `ConfigModule` còn hỗ trợ các tính năng nâng cao như xác thực schema (schema validation) để đảm bảo các biến môi trường cần thiết tồn tại khi khởi động 61, và tải các tệp cấu hình tùy chỉnh để nhóm các biến (ví dụ: `database.host`, `database.port`).61