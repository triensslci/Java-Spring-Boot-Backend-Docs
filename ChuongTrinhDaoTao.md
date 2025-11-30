# Chương Trình Đào Tạo Chi Tiết: Java Spring Boot Backend (24 Buổi)

**Mục tiêu cuối cùng:** Học viên từ con số 0 → Deploy thành công dự án Spring Boot lên Render sử dụng GitHub Actions CI/CD

**Dự án thực hành xuyên suốt:** Hệ thống Đặt & Điều xe Taxi (Taxi Booking System)

---

## Giai đoạn 1: Khởi động & Nền tảng Spring Core (Buổi 1 - 4)

**Mục tiêu:** Hiểu cách Spring vận hành, không code máy móc. Làm quen với Git và workflow phát triển phần mềm.

### Buổi 1: Tổng quan, Môi trường & Git Workflow

#### Kiến thức:
- **Backend Architecture:** Client - Server - Database
- **Cài đặt môi trường:** 
  - JDK 17+ (kiểm tra: `java -version`)
  - **VS Code** với các extensions:
    - Extension Pack for Java (Microsoft)
    - Spring Boot Extension Pack (VMware)
    - Spring Boot Tools (VMware)
    - Spring Initializr Java Support (Microsoft)
    - Maven for Java (Microsoft)
    - GitLens (tùy chọn nhưng hữu ích)
  - Postman (hoặc Thunder Client extension trong VS Code)
  - Git (kiểm tra: `git --version`)
- **VS Code cơ bản:** 
  - Mở folder project, Terminal tích hợp
  - Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`)
  - Run và Debug Spring Boot app từ VS Code
- **Git cơ bản:** `init`, `add`, `commit`, `push`, `.gitignore`. Cách tạo Pull Request (PR)
- **Git Branching:** `main` vs `develop`, cách tạo và merge branch (quan trọng cho teamwork và CI/CD)

#### Thực Hành:
- Setup môi trường development
- Cài đặt và cấu hình VS Code với Java extensions
- Khởi tạo Spring Boot Project:
  - **Cách 1:** Dùng Spring Initializr website → Download → Mở trong VS Code
  - **Cách 2:** Dùng VS Code Command Palette → "Spring Initializr: Create a Maven Project"
- Push source code đầu tiên lên GitHub (dùng VS Code Source Control panel hoặc Terminal)

#### Dự án Taxi:
- Khởi tạo cấu trúc dự án `taxi-booking-backend` chuẩn (Maven, Dependencies cơ bản)
- Mở project trong VS Code, kiểm tra Java extension hoạt động

---

### Buổi 2: Tư duy OOP & Spring Core (DI/IoC)

#### Kiến thức:
- Ôn tập **Interface**, **Abstract Class** trong Java
  - Interface: Abstract method, Default method (Java 8+), Static method (Java 8+)
  - Abstract Class: Concrete method (có thể override), Abstract method (phải implement)
  - Abstract Class có thể implement Interface
  - **Sự khác biệt `extends` vs `implements`:** extends = kế thừa code từ 1 class, implements = thực hiện hợp đồng từ nhiều interface
- **Inversion of Control (IoC) Container:** "Kho chứa" Bean - Spring tự động tạo, lưu trữ và cung cấp các đối tượng (Bean) khi cần
- **Dependency Injection (DI):** Tự động cung cấp đối tượng - Spring tự động tạo và gán đối tượng cần thiết vào nơi cần dùng
- **So sánh IoC Container vs DI:** IoC Container = Công cụ (nhà kho), DI = Cách thức (dịch vụ giao hàng). IoC Container sử dụng DI để cung cấp Bean
- **Các Annotation quan trọng:**
  - Tạo Bean: `@Component`, `@Service`, `@Repository`, `@Controller`
  - Cấu hình: `@Configuration`, `@Bean`
  - Inject: `@Autowired`
  - Chọn Bean: `@Primary`, `@Qualifier`
  - Đọc config: `@Value`
- **Bean Lifecycle:** Singleton vs Prototype
- **VS Code Productivity:**
  - Go to Definition: `F12` hoặc `Cmd+Click` / `Ctrl+Click`
  - Find References: `Shift+F12`
  - Quick Fix: `Cmd+.` / `Ctrl+.`
  - IntelliSense: Auto-complete, parameter hints
  - Peek Definition: `Alt+F12`

#### Thực Hành:
- Tạo một `Coach` interface và inject các loại Coach khác nhau vào `MainApp`
- Practice navigation: Go to definition, find references trong VS Code

#### Dự án Taxi:
- Chưa áp dụng (tập trung lý thuyết cốt lõi)

---

### Buổi 3: Spring Boot & REST API Cơ bản

#### Kiến thức:
- Cấu trúc project Spring Boot chuẩn (giải thích kỹ các folder đã tạo ở Buổi 1)
- `@RestController`, `@RequestMapping`
- Quy chuẩn đặt tên URL (Naming convention)
- **VS Code với Spring Boot:**
  - Run Spring Boot app: Click vào `@SpringBootApplication` class → Click "Run" hoặc dùng `F5`
  - Debug: Set breakpoint → `F5` để debug
  - Xem logs trong VS Code Terminal
  - Auto-reload với Spring Boot DevTools (nếu có)

#### Thực Hành:
- API Hello World
- Run và Debug app trong VS Code

#### Dự án Taxi:
- Tạo cấu trúc packages chi tiết: `controller`, `service`, `repository`, `entity`
- Viết API `GET /api/welcome`: Trả về "Welcome to Taxi App"
- Run app trong VS Code và test bằng Postman/Thunder Client

---

### Buổi 4: Request Handling & Response Entity

#### Kiến thức:
- Nhận dữ liệu: `@RequestParam` (cho filter), `@PathVariable` (cho ID), `@RequestBody` (cho JSON)
- **HTTP Methods:** GET, POST, PUT, DELETE
- **ResponseEntity:** Kiểm soát HTTP Status Code (200, 201, 400, 404, 500)

#### Thực Hành:
- API Calculator (Cộng trừ nhân chia)

#### Dự án Taxi:
- Viết API `POST /api/bookings/calculate`: Nhập vào khoảng cách (km), loại xe → Trả về giá tiền ước tính (Logic if-else đơn giản, chưa cần DB)

---

## Giai đoạn 2: Database & Data JPA (Buổi 5 - 10)

**Mục tiêu:** Thiết kế CSDL và thao tác CRUD thành thạo.

### Buổi 5: Thiết kế CSDL & Kết nối MySQL

#### Kiến thức:
- Tư duy thiết kế **ERD (Entity Relationship Diagram)**
- Cấu hình `application.properties`: Datasource URL, Username, Password
- **Hibernate ddl-auto:** `update` vs `create-drop`
- **VS Code Database Tools:**
  - Extension: **MySQL** (của WeChat) hoặc **SQLTools** (của Matheus Teixeira)
  - Kết nối MySQL từ VS Code, xem và query database trực tiếp
  - Hoặc dùng MySQL Workbench/DBeaver (tool bên ngoài)

#### Thực Hành:
- Cài đặt MySQL Workbench/DBeaver hoặc MySQL extension trong VS Code
- Kết nối MySQL từ VS Code (nếu dùng extension)

#### Dự án Taxi:
- Thiết kế bảng `users` (id, email, password, role) và `bookings`
- Kết nối project với MySQL local
- Test kết nối bằng cách run app và kiểm tra logs trong VS Code Terminal

---

### Buổi 6: Entity Mapping & Basic CRUD

#### Kiến thức:
- `@Entity`, `@Table`, `@Id`, `@GeneratedValue`
- **JpaRepository interface:** `save`, `findById`, `findAll`, `delete`
- **VS Code Debugging:**
  - Set breakpoint: Click vào số dòng bên trái hoặc `F9`
  - Debug mode: `F5` hoặc click vào "Run and Debug" panel
  - Step over (`F10`), Step into (`F11`), Step out (`Shift+F11`)
  - Xem variables, call stack trong Debug panel
  - Debug Spring Boot app: Chọn "Java" configuration

#### Thực Hành:
- CRUD quản lý Sinh viên
- Practice debugging: Set breakpoint trong Controller, Service, Repository

#### Dự án Taxi:
- Tạo Entity `User`
- Viết API đăng ký tài khoản đơn giản (lưu thẳng vào DB, chưa mã hóa pass)
- Debug API đăng ký: Set breakpoint để xem flow từ Controller → Service → Repository

---

### Buổi 7: DTO Pattern & Object Mapping

#### Kiến thức:
- Tại sao không trả về Entity? (Lộ password, vòng lặp vô hạn, dư thừa data)
- **Pattern:** RequestDTO → Entity → ResponseDTO
- Thư viện: **MapStruct** hoặc **BeanUtils**
- **VS Code Refactoring:**
  - Rename Symbol: `F2` (rename variable, method, class)
  - Extract Method: Select code → Right click → "Extract Method"
  - Extract Variable: Select expression → Right click → "Extract Variable"
  - Organize Imports: `Shift+Alt+O` / `Shift+Option+O`

#### Thực Hành:
- Refactor bài Sinh viên dùng DTO
- Practice refactoring tools trong VS Code

#### Dự án Taxi:
- Tạo `RegisterRequestDTO`, `UserResponseDTO`
- Refactor API đăng ký dùng DTO (dùng rename, extract method nếu cần)

---

### Buổi 8: Validation & Global Exception Handling

#### Kiến thức:
- **Validation:** `@NotNull`, `@Email`, `@Size`, `@Pattern` (Regex)
- Xử lý lỗi tập trung: `@ControllerAdvice`, `@ExceptionHandler`

#### Thực Hành:
- Validate Form đăng ký

#### Dự án Taxi:
- Validate số điện thoại (phải 10 số), email đúng định dạng
- Bắt lỗi `UserAlreadyExistsException` trả về JSON lỗi đẹp (message, status)

---

### Buổi 9: JPA Relationships (Quan hệ bảng)

#### Kiến thức:
- `@OneToMany`, `@ManyToOne` (Quan trọng nhất)
- `@OneToOne`, `@ManyToMany`
- **Fetch Type:** LAZY vs EAGER

#### Thực Hành:
- Mapping Class - Student

#### Dự án Taxi:
- Mapping quan hệ: 1 User (Passenger) có nhiều Booking
- Tạo Entity `Booking` liên kết với User (`passenger_id`, `driver_id`)

---

### Buổi 10: Advanced JPA & Database Migration

#### Kiến thức:
- **Query Methods:** `findByEmail`, `existsByPhone`...
- **JPQL (Java Persistence Query Language)** & Native Query
- **Database Migration:** Flyway hoặc Liquibase (quản lý schema version)
- **Multiple Profiles:** `application-dev.properties` vs `application-prod.properties`

#### Thực Hành:
- Tìm sinh viên có điểm > 8.0
- Setup Flyway migration scripts

#### Dự án Taxi:
- Viết API `GET /api/bookings/history`: Tìm tất cả chuyến đi của một user cụ thể
- Viết API `GET /api/bookings/pending`: Tìm chuyến đi có status = 'PENDING'
- Tạo file migration đầu tiên cho bảng `users` và `bookings`

---

## Giai đoạn 3: Bảo mật & Nghiệp vụ Nâng cao (Buổi 11 - 18)

**Mục tiêu:** Biến API thành hệ thống an toàn và thông minh.

### Buổi 11: Spring Security Fundamentals

#### Kiến thức:
- Cơ chế **Authentication (Xác thực)** vs **Authorization (Phân quyền)**
- `SecurityFilterChain`, `BCryptPasswordEncoder`

#### Thực Hành:
- Login bằng Basic Auth (Username/Password cứng)

#### Dự án Taxi:
- Mã hóa mật khẩu User khi đăng ký
- Cấu hình Security cho phép truy cập public vào `/auth/**`

---

### Buổi 12: JWT (JSON Web Token) Implementation

#### Kiến thức:
- Cấu trúc JWT (Header, Payload, Signature)
- Quy trình: Login → Server ký Token → Client lưu Token → Gửi kèm Header

#### Thực Hành:
- Tạo class `JwtUtils`

#### Dự án Taxi:
- Viết API `POST /auth/login` trả về Access Token
- Viết `JwtFilter` để chặn các request không có token

---

### Buổi 13: Role-based Authorization

#### Kiến thức:
- Phân quyền dựa trên Role (`ROLE_ADMIN`, `ROLE_DRIVER`...)
- Annotation `@PreAuthorize("hasRole('ADMIN')")`
- Lấy thông tin người đang login từ `SecurityContextHolder`

#### Thực Hành:
- API Admin xóa User

#### Dự án Taxi:
- Chặn API: Chỉ DRIVER mới được gọi API nhận chuyến (`/accept`)
- Logic: Khi tạo Booking, tự động lấy ID của người đang login làm `passenger_id`

---

### Buổi 14: File Upload & Static Resources

#### Kiến thức:
- `MultipartFile`
- Lưu file vào thư mục server (hoặc Cloudinary - optional)
- Serve static content

#### Thực Hành:
- Upload ảnh profile

#### Dự án Taxi:
- API Upload Avatar cho Driver
- Lưu đường dẫn ảnh vào bảng User

---

### Buổi 15: Pagination & Sorting (Phân trang)

#### Kiến thức:
- `Pageable`, `PageRequest`
- Đối tượng trả về `Page<T>`

#### Thực Hành:
- Phân trang danh sách sản phẩm

#### Dự án Taxi:
- Nâng cấp API lịch sử chuyến đi: Trả về 10 chuyến mới nhất mỗi trang

---

### Buổi 16: Email Service & Scheduling

#### Kiến thức:
- `JavaMailSender`
- **Scheduling:** `@Scheduled` (Cron Job)

#### Thực Hành:
- Gửi email test sau 1 phút

#### Dự án Taxi:
- Gửi Email xác nhận khi chuyến đi hoàn thành (COMPLETED)
- Job chạy ngầm: Mỗi 5 phút quét DB, hủy các chuyến PENDING quá 30 phút

---

### Buổi 17: Caching với Redis (Tăng tốc)

#### Kiến thức:
- Khái niệm **Caching**. Tại sao dùng Redis?
- `@Cacheable`, `@CacheEvict`

#### Thực Hành:
- Cài Redis (Docker), cache danh sách Category

#### Dự án Taxi:
- Cache bảng giá cước (ít thay đổi)
- (Nâng cao) Lưu vị trí tài xế tạm thời trên Redis

---

### Buổi 18: Unit Testing (JUnit 5 & Mockito)

#### Kiến thức:
- Tại sao cần Test? **Unit Test** vs **Integration Test**
- **Mocking Dependencies:** `@Mock`, `@InjectMocks`
- **VS Code Testing:**
  - Run test: Click vào icon ▶️ bên cạnh test method hoặc class
  - Run all tests: Click vào "Run Tests" trong Test Explorer panel
  - Debug test: Set breakpoint trong test → Right click → "Debug Test"
  - Xem test coverage: Extension "Coverage Gutters" (optional)
  - Test Explorer panel: Xem tất cả tests, filter, run tests theo package/class

#### Thực Hành:
- Test `CalculatorService`
- Run và debug tests trong VS Code

#### Dự án Taxi:
- Viết Test cho `BookingService`: Test logic tính tiền, Test logic nhận chuyến (tranh chấp)
- Run tất cả tests từ VS Code Test Explorer
- Debug test để xem flow và variables

---

## Giai đoạn 4: Deployment & Hoàn thiện (Buổi 19 - 24)

**Mục tiêu:** Đưa sản phẩm ra thực tế.

### Buổi 19: Docker Containerization (Tùy chọn - có thể bỏ qua nếu tập trung vào Render)

#### Kiến thức:
- Container là gì? **Dockerfile**, Image, Container
- **Docker Compose** để chạy cả App và MySQL (cho local development)

#### Thực Hành:
- Đóng gói App Hello World

#### Dự án Taxi:
- Viết Dockerfile cho Backend (tùy chọn, Render không bắt buộc)
- Viết `docker-compose.yml` chạy App + DB (cho local)

**Lưu ý:** Render có thể build trực tiếp từ source code, không nhất thiết cần Docker. Buổi này hữu ích cho hiểu biết về containerization nhưng không bắt buộc cho deployment lên Render.

---

### Buổi 20: GitHub Actions CI/CD Pipeline & Production Database Setup

#### Kiến thức:
- **CI/CD là gì?** Continuous Integration / Continuous Deployment
- **GitHub Actions:** Workflow file (`.github/workflows/deploy.yml`)
- Các bước trong pipeline: Build → Test → Deploy
- **Secrets trong GitHub:** Lưu trữ thông tin nhạy cảm (DB password, API keys)
- **Production Database:** PostgreSQL trên Render (khác MySQL local)
- **@Profile** annotation để chọn config theo môi trường
- **VS Code với GitHub Actions:**
  - Tạo folder `.github/workflows` trong VS Code
  - Edit YAML file với syntax highlighting và IntelliSense
  - Extension "GitHub Actions" (của GitHub) để preview workflow
  - Xem workflow runs trực tiếp trong VS Code (nếu có extension)
  - Commit và push workflow file lên GitHub

#### Thực Hành:
- Tạo PostgreSQL database trên Render
- Tạo workflow file trong VS Code: `.github/workflows/deploy.yml`
- Build và test Spring Boot app
- Trigger workflow khi push code lên branch `main`

#### Dự án Taxi:
- Tạo PostgreSQL database trên Render, lấy connection string
- Cấu hình profile `prod` với PostgreSQL connection string
- Tạo `.github/workflows/deploy.yml` trong VS Code:
  - Build JAR file với Maven
  - Chạy Unit Tests (nếu có)
  - Deploy lên Render bằng Render API hoặc webhook
- Setup GitHub Secrets: `RENDER_API_KEY`, `RENDER_SERVICE_ID` (nếu dùng Render API)
- Commit và push workflow file, verify GitHub Actions chạy

---

### Buổi 21: Deploy lên Render với GitHub Actions & Production Best Practices

#### Kiến thức:
- **Render Platform:** Web Service vs Background Worker
- **Environment Variables** trên Render: DATABASE_URL, JWT_SECRET, etc.
- **Build Command** và **Start Command** trên Render
- **Spring Boot Actuator:** Health checks, metrics (`/actuator/health`)
- **Logging:** SLF4J + Logback, cấu hình log levels cho production
- **Error Handling:** Global exception handler cho production (không expose stack trace)
- **CORS Configuration:** Cho phép frontend gọi API
- **API Documentation:** Swagger/OpenAPI setup

#### Thực Hành:
- Tạo Render Web Service
- Kết nối với GitHub repository (hoặc dùng GitHub Actions để auto-deploy)
- Cấu hình Build & Start commands
- Thêm Actuator dependencies và cấu hình health check
- Setup Swagger UI: `/swagger-ui.html`

#### Dự án Taxi:
- Deploy backend lên Render (Free tier)
- Kết nối với PostgreSQL database trên Render
- Setup environment variables: `SPRING_PROFILES_ACTIVE=prod`, `DATABASE_URL`, `JWT_SECRET`
- Thêm Actuator, expose `/actuator/health` endpoint
- Cấu hình CORS cho phép frontend gọi API
- Thêm Swagger documentation cho tất cả endpoints
- Test API trên production URL bằng Postman
- Kiểm tra logs trên Render dashboard
- Test health check endpoint

---

### Buổi 22: Dự án cuối kỳ - Sprint 1 (Setup & Auth & Deployment Setup)

#### Nội dung:
- Tổng hợp lại kiến thức
- Học viên tự rà soát lại project Taxi hoặc chọn đề tài mới tương đương (E-commerce, Job Board)

#### Yêu cầu:
- Hoàn thiện ERD và Module Authentication
- Setup GitHub Actions workflow cơ bản
- Setup PostgreSQL database trên Render
- Deploy lên Render (có thể chưa hoàn chỉnh, nhưng phải có deployment pipeline)

---

### Buổi 23: Dự án cuối kỳ - Sprint 2 (Core Logic & Hoàn thiện Deployment)

#### Nội dung:
- Coding session tại lớp
- Giảng viên review code 1-1
- **Quan trọng:** Hoàn thiện deployment pipeline và production setup

#### Yêu cầu:
- Hoàn thiện các luồng nghiệp vụ chính (Đặt xe, Nhận xe)
- **Bắt buộc:** GitHub Actions workflow hoạt động đầy đủ (Build → Test → Deploy)
- **Bắt buộc:** App chạy ổn định trên Render production
- **Bắt buộc:** Health check endpoint hoạt động
- **Bắt buộc:** Swagger documentation đầy đủ
- Test toàn bộ API trên production environment
- Cấu hình CORS, logging, error handling cho production

---

### Buổi 24: Demo Day & Tổng kết

#### Nội dung:
- Học viên thuyết trình demo sản phẩm:
  - **Bắt buộc:** Demo trên production URL (Render)
  - Demo bằng Postman hoặc Frontend đơn giản
  - **Bắt buộc:** Giải thích và demo quy trình CI/CD:
    - Push code lên GitHub
    - GitHub Actions tự động trigger
    - Build và deploy lên Render
    - Test API trên production
- Giảng viên đóng vai "Technical Interviewer" phỏng vấn:
  - Hỏi về deployment process (GitHub Actions → Render)
  - Hỏi về cách xử lý lỗi trên production
  - Hỏi về database migration (Flyway)
  - Hỏi về environment variables và security
  - Hỏi về cách monitor và debug trên production
- Tư vấn lộ trình tiếp theo (Microservices, DevOps, Kubernetes, AWS, etc.)

---

## Phụ Lục: Chi Tiết Dự Án Thực Hành (Taxi Booking System)

### 1. Thiết Kế Cơ Sở Dữ Liệu (Database Design)

Hệ thống sử dụng mô hình quan hệ (Relational Database) với các bảng chính sau:

#### 1.1. Bảng `users` (Người dùng)
Lưu trữ thông tin chung cho cả Hành khách (Passenger), Tài xế (Driver) và Quản trị viên (Admin).

- `id` (PK, Long, Auto Increment): Khóa chính
- `email` (String, Unique): Tên đăng nhập
- `password` (String): Mật khẩu đã mã hóa (BCrypt)
- `full_name` (String): Họ tên đầy đủ
- `phone` (String): Số điện thoại liên hệ
- `role` (String/Enum): Phân quyền (`ROLE_PASSENGER`, `ROLE_DRIVER`, `ROLE_ADMIN`)
- `balance` (BigDecimal): Ví tiền ảo (để thanh toán giả lập)
- `vehicle_type` (String, Nullable): Loại xe (chỉ dành cho Driver, VD: 4 chỗ, 7 chỗ, xe máy)
- `is_active` (Boolean): Trạng thái kích hoạt tài khoản

#### 1.2. Bảng `bookings` (Chuyến xe)
Lưu trữ thông tin giao dịch đặt xe.

- `id` (PK, Long, Auto Increment): Khóa chính
- `passenger_id` (FK → `users.id`): Người đặt xe
- `driver_id` (FK → `users.id`, Nullable): Tài xế nhận chuyến (null khi mới đặt)
- `pickup_location` (String): Địa điểm đón
- `dropoff_location` (String): Địa điểm đến
- `distance_km` (Double): Khoảng cách ước tính
- `total_price` (BigDecimal): Giá cước chuyến đi
- `status` (String/Enum): Trạng thái chuyến đi (`PENDING`, `ACCEPTED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`)
- `created_at` (DateTime): Thời gian đặt
- `completed_at` (DateTime): Thời gian hoàn thành

#### 1.3. Bảng `feedbacks` (Đánh giá) - Optional

- `id` (PK): Khóa chính
- `booking_id` (FK → `bookings.id`): Chuyến xe nào
- `rating` (Integer): 1-5 sao
- `comment` (String): Nội dung góp ý

---

### 2. Các Luồng Nghiệp Vụ Chính (Business Flows)

#### 2.1. Luồng Đặt Xe (Booking Flow - Passenger)

1. Passenger đăng nhập và chọn điểm đi, điểm đến
2. Passenger gọi API tính giá (`POST /api/bookings/calculate`)
3. System trả về giá cước dự kiến dựa trên khoảng cách
4. Passenger xác nhận đặt xe (`POST /api/bookings`)
5. System tạo bản ghi Booking với trạng thái `PENDING` và lưu vào DB

#### 2.2. Luồng Nhận Chuyến (Acceptance Flow - Driver)

1. Driver đăng nhập, gọi API xem các chuyến đang chờ (`GET /api/bookings/available`)
2. Driver chọn một chuyến và bấm "Nhận chuyến" (`POST /api/bookings/{id}/accept`)
3. System kiểm tra tính nhất quán (Concurrency check):
   - Nếu chuyến vẫn còn `PENDING`: Gán `driver_id` = Driver hiện tại, cập nhật status → `ACCEPTED`. Trả về thành công
   - Nếu chuyến đã bị người khác nhận hoặc hủy: Trả về lỗi

#### 2.3. Luồng Hoàn Thành Chuyến (Completion Flow)

1. Driver đến đón khách → Update status `IN_PROGRESS`
2. Driver đưa khách đến nơi → Update status `COMPLETED`
3. System thực hiện trừ tiền trong ví (nếu có tính năng ví) hoặc ghi nhận doanh thu
4. System gửi email hóa đơn cho Passenger (Feature Buổi 16)

#### 2.4. Luồng Hủy Chuyến Tự Động (Cancellation Flow)

1. System (Scheduler) chạy ngầm mỗi 5 phút
2. Quét bảng `bookings` tìm các chuyến có status `PENDING` và `created_at` quá 15 phút so với hiện tại
3. Cập nhật status → `CANCELLED`

---

## Phụ Lục: VS Code Tips & Tricks cho Spring Boot Development

### Extensions Cần Thiết:

1. **Extension Pack for Java** (Microsoft) - Bộ extension cơ bản:
   - Language Support for Java
   - Debugger for Java
   - Test Runner for Java
   - Maven for Java
   - Project Manager for Java

2. **Spring Boot Extension Pack** (VMware) - Hỗ trợ Spring Boot:
   - Spring Boot Tools
   - Spring Boot Dashboard
   - Spring Initializr Java Support

3. **GitLens** (GitKraken) - Nâng cao Git experience

4. **Thunder Client** (Raneem) - Test API trực tiếp trong VS Code (thay thế Postman)

5. **MySQL** (WeChat) hoặc **SQLTools** - Kết nối database

6. **GitHub Actions** (GitHub) - Preview và edit workflow files

### Keyboard Shortcuts Quan Trọng:

- `F5` - Run/Debug
- `F9` - Toggle breakpoint
- `F10` - Step over
- `F11` - Step into
- `Shift+F11` - Step out
- `F12` - Go to definition
- `Shift+F12` - Find all references
- `F2` - Rename symbol
- `Cmd+.` / `Ctrl+.` - Quick fix
- `Cmd+Shift+P` / `Ctrl+Shift+P` - Command Palette
- `Shift+Alt+O` / `Shift+Option+O` - Organize imports

### Workflow Phát Triển trong VS Code:

1. **Mở Project:**
   - File → Open Folder → Chọn folder project
   - VS Code tự động detect Java project và load Maven dependencies

2. **Run Spring Boot App:**
   - Tìm class có `@SpringBootApplication`
   - Click vào "Run" button bên cạnh `main` method
   - Hoặc dùng Command Palette → "Spring Boot: Run"

3. **Debug:**
   - Set breakpoint (click vào số dòng)
   - Run với Debug mode (`F5`)
   - Xem variables, call stack trong Debug panel

4. **Test API:**
   - Dùng Thunder Client extension
   - Hoặc Postman (tool bên ngoài)
   - Test trực tiếp trong VS Code

5. **Git Operations:**
   - Source Control panel (icon bên trái)
   - Commit, push, pull trực tiếp trong VS Code
   - Xem diff, history với GitLens

6. **Maven Commands:**
   - Command Palette → "Java: Run Maven Goal"
   - Hoặc dùng Terminal tích hợp: `./mvnw clean install`

### Troubleshooting VS Code với Java:

- **Java không được detect:** Kiểm tra `JAVA_HOME` environment variable
- **Maven dependencies không load:** Command Palette → "Java: Clean Java Language Server Workspace"
- **Extension không hoạt động:** Reload VS Code (`Cmd+Shift+P` → "Reload Window")
- **IntelliSense không hiện:** Đợi Java Language Server index project (xem progress bar dưới)

---

## Phụ Lục: Checklist Deployment lên Render với GitHub Actions

### Setup Môi Trường Development (VS Code):

- [ ] **Cài đặt VS Code** và các extensions cần thiết:
  - Extension Pack for Java
  - Spring Boot Extension Pack
  - GitLens (optional)
  - Thunder Client hoặc Postman
  - MySQL/SQLTools extension
- [ ] **Cấu hình Java:** Kiểm tra `java -version` và `JAVA_HOME`
- [ ] **Test VS Code:** Mở project, verify IntelliSense hoạt động
- [ ] **Test Run/Debug:** Run Spring Boot app từ VS Code thành công

### Trước khi Deploy:

- [ ] **Database Migration:** Đã setup Flyway/Liquibase và tạo migration scripts
- [ ] **Multiple Profiles:** Đã cấu hình `application-prod.properties` với PostgreSQL
- [ ] **Environment Variables:** Đã xác định tất cả biến môi trường cần thiết (DATABASE_URL, JWT_SECRET, etc.)
- [ ] **Health Check:** Đã thêm Spring Boot Actuator và expose `/actuator/health`
- [ ] **CORS:** Đã cấu hình CORS cho phép frontend gọi API
- [ ] **Error Handling:** Global exception handler không expose stack trace trên production
- [ ] **Logging:** Đã cấu hình logging phù hợp cho production
- [ ] **Swagger:** Đã thêm Swagger documentation (optional nhưng recommended)

### Setup Render:

- [ ] Tạo PostgreSQL database trên Render, lấy connection string
- [ ] Tạo Web Service trên Render
- [ ] Cấu hình Build Command: `./mvnw clean package -DskipTests` (hoặc tương đương)
- [ ] Cấu hình Start Command: `java -jar target/your-app.jar --spring.profiles.active=prod`
- [ ] Setup Environment Variables trên Render:
  - `SPRING_PROFILES_ACTIVE=prod`
  - `DATABASE_URL=<postgresql-connection-string>`
  - `JWT_SECRET=<your-secret-key>`
  - Các biến khác nếu cần

### Setup GitHub Actions:

- [ ] Tạo folder `.github/workflows` trong VS Code
- [ ] Tạo file `deploy.yml` trong VS Code (có syntax highlighting)
- [ ] Workflow bao gồm các bước:
  - Checkout code
  - Setup JDK
  - Build với Maven
  - Run tests (optional)
  - Deploy lên Render (qua Render API hoặc webhook)
- [ ] Setup GitHub Secrets nếu cần (RENDER_API_KEY, etc.)
- [ ] Commit và push workflow file lên GitHub (dùng VS Code Source Control)
- [ ] Test workflow bằng cách push code lên branch `main`
- [ ] Xem workflow runs trên GitHub (hoặc dùng GitHub extension trong VS Code)

### Sau khi Deploy:

- [ ] Test health check endpoint: `https://your-app.onrender.com/actuator/health`
- [ ] Test các API endpoints chính bằng Postman
- [ ] Kiểm tra logs trên Render dashboard
- [ ] Test quy trình CI/CD: Push code → GitHub Actions chạy → App tự động deploy
- [ ] Verify database connection và data migration

### Troubleshooting thường gặp:

- **Build failed:** Kiểm tra JDK version, Maven dependencies
- **App không start:** Kiểm tra Start Command, Environment Variables, logs
- **Database connection error:** Kiểm tra DATABASE_URL, firewall settings
- **GitHub Actions không trigger:** Kiểm tra file path `.github/workflows/deploy.yml`, branch name
- **Deploy nhưng API không hoạt động:** Kiểm tra CORS, security configuration, port binding

---

**Kết thúc chương trình**

