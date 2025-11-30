# Buổi 3: Spring Boot & REST API Cơ bản

## Kiến thức

### 1. Cấu trúc project Spring Boot chuẩn

#### 1.1. Tổng quan cấu trúc

**Giải thích:** Khi bạn tạo một Spring Boot project (từ Spring Initializr hoặc VS Code), Spring Boot tự động tạo ra một cấu trúc folder chuẩn. Hiểu cấu trúc này giúp bạn biết nên đặt code ở đâu và tại sao.

**Cấu trúc folder chuẩn:**

```
taxi-booking-backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── taxi/
│   │   │           └── booking/
│   │   │               ├── TaxiBookingBackendApplication.java  ← Main class
│   │   │               ├── controller/     ← Xử lý HTTP request
│   │   │               ├── service/       ← Business logic
│   │   │               ├── repository/     ← Tương tác database
│   │   │               ├── entity/         ← Class đại diện cho bảng DB
│   │   │               └── dto/            ← Data Transfer Object
│   │   └── resources/
│   │       ├── application.properties      ← File cấu hình
│   │       └── static/                     ← File tĩnh (HTML, CSS, JS)
│   └── test/
│       └── java/                            ← Code test
├── pom.xml                                  ← File quản lý dependencies (Maven)
├── .gitignore                               ← File bỏ qua khi commit Git
└── README.md                                ← Mô tả project
```

**Giải thích từng phần:**

1. **`src/main/java/`**: Nơi chứa code Java chính của ứng dụng
   - Đây là nơi bạn viết tất cả code business logic
   - Package structure: `com.taxi.booking` (theo quy ước reverse domain)

2. **`src/main/resources/`**: Nơi chứa file cấu hình và tài nguyên
   - `application.properties`: Cấu hình database, port, logging, etc.
   - `static/`: File HTML, CSS, JS (nếu có frontend)
   - `templates/`: File template (nếu dùng Thymeleaf)

3. **`src/test/java/`**: Nơi chứa code test
   - Unit test, Integration test
   - Spring Boot tự động tạo folder này

4. **`pom.xml`**: File cấu hình Maven
   - Khai báo dependencies (thư viện cần dùng)
   - Cấu hình build, version Java, etc.

5. **`.gitignore`**: File bỏ qua khi commit Git
   - Bỏ qua file build, log, IDE config
   - Tránh commit file không cần thiết

---

#### 1.2. Package structure chi tiết

**Giải thích:** Trong Spring Boot, chúng ta tổ chức code theo các package (thư mục) để dễ quản lý và tìm kiếm. Mỗi package có một vai trò cụ thể.

**Cấu trúc package chuẩn cho Taxi Booking System:**

```
com.taxi.booking/
├── TaxiBookingBackendApplication.java  ← Main class (có @SpringBootApplication)
│
├── controller/                          ← Xử lý HTTP request từ client
│   ├── BookingController.java
│   ├── UserController.java
│   └── WelcomeController.java
│
├── service/                            ← Business logic (logic nghiệp vụ)
│   ├── BookingService.java
│   ├── UserService.java
│   └── PriceCalculationService.java
│
├── repository/                         ← Tương tác với database
│   ├── BookingRepository.java
│   └── UserRepository.java
│
├── entity/                            ← Class đại diện cho bảng database
│   ├── Booking.java
│   └── User.java
│
└── dto/                               ← Data Transfer Object (sẽ học ở buổi sau)
    ├── BookingRequestDTO.java
    └── BookingResponseDTO.java
```

**Giải thích từng package:**

1. **`controller/`** - Bộ điều khiển HTTP request
   - **Vai trò:** Nhận request từ client (Postman, Frontend), gọi Service để xử lý, trả về response
   - **Ví dụ:** Khi client gọi `GET /api/welcome`, `WelcomeController` sẽ nhận request này
   - **Annotation:** `@RestController`, `@RequestMapping`

2. **`service/`** - Xử lý business logic
   - **Vai trò:** Chứa logic nghiệp vụ (tính giá, validate, xử lý dữ liệu)
   - **Ví dụ:** `BookingService` có method `calculatePrice()` để tính giá cước
   - **Annotation:** `@Service`

3. **`repository/`** - Tương tác với database
   - **Vai trò:** Lưu, xóa, tìm kiếm dữ liệu trong database
   - **Ví dụ:** `UserRepository` có method `findByEmail()` để tìm user theo email
   - **Annotation:** `@Repository`

4. **`entity/`** - Đại diện cho bảng database
   - **Vai trò:** Class Java đại diện cho một bảng trong database
   - **Ví dụ:** Class `User` đại diện cho bảng `users` trong database
   - **Annotation:** `@Entity`, `@Table`

5. **`dto/`** - Data Transfer Object (sẽ học chi tiết ở buổi sau)
   - **Vai trò:** Class để truyền dữ liệu giữa client và server
   - **Ví dụ:** `BookingRequestDTO` chứa dữ liệu client gửi lên khi đặt xe

**Tại sao cần tổ chức như vậy?**

- **Dễ tìm code:** Biết chức năng ở đâu → Tìm nhanh hơn
- **Dễ bảo trì:** Sửa code ở một nơi, không ảnh hưởng nơi khác
- **Dễ test:** Test từng phần riêng biệt
- **Theo chuẩn:** Spring Boot và cộng đồng Java đều dùng cấu trúc này

---

### 2. @RestController - Tạo REST API

#### 2.1. Định nghĩa

**Giải thích:** `@RestController` là annotation đánh dấu một class là Controller xử lý REST API. REST API là cách client (Frontend, Mobile App) giao tiếp với server (Backend) thông qua HTTP request.

**So sánh dễ hiểu:**
- **REST API** giống như một nhà hàng có menu. Client xem menu (gọi API) → Nhà hàng phục vụ món (trả về dữ liệu JSON)
- **@RestController** giống như đầu bếp trưởng, nhận order từ khách và chỉ đạo làm món

**Cấu trúc cơ bản:**

```java
@RestController
@RequestMapping("/api")
public class WelcomeController {
    // Các method xử lý HTTP request
}
```

**Giải thích:**
- `@RestController`: Đánh dấu class này là REST Controller
- `@RequestMapping("/api")`: Tất cả URL trong class này sẽ bắt đầu bằng `/api`
- Method trong class: Xử lý các HTTP request cụ thể (GET, POST, PUT, DELETE)

---

#### 2.2. Cách thức hoạt động

**Quy trình từ khi client gọi API đến khi nhận response:**

```
Bước 1: Client gửi HTTP Request
    │
    │   GET http://localhost:8080/api/welcome
    │
    ▼
Bước 2: Spring Boot nhận request
    │
    │   Spring DispatcherServlet (bộ điều phối) nhận request
    │
    ▼
Bước 3: Tìm Controller phù hợp
    │
    │   Spring tìm class có @RestController
    │   và @RequestMapping("/api")
    │
    ▼
Bước 4: Tìm method phù hợp
    │
    │   Spring tìm method có @GetMapping("/welcome")
    │
    ▼
Bước 5: Gọi method và xử lý
    │
    │   WelcomeController.welcome()
    │
    ▼
Bước 6: Trả về response (JSON)
    │
    │   {"message": "Welcome to Taxi App"}
    │
    ▼
Bước 7: Client nhận response
```

**Ví dụ minh họa:**

```java
@RestController
@RequestMapping("/api")
public class WelcomeController {
    
    // Xử lý GET request: /api/welcome
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Khi client gọi `GET /api/welcome`:**
1. Spring Boot nhận request
2. Tìm `WelcomeController` (có `@RequestMapping("/api")`)
3. Tìm method `welcome()` (có `@GetMapping("/welcome")`)
4. Gọi method `welcome()`
5. Trả về `"Welcome to Taxi App"` (Spring tự động chuyển thành JSON)

**Kết quả:** Client nhận được:
```json
"Welcome to Taxi App"
```

---

#### 2.3. Trường hợp sử dụng thực tế

**Khi nào dùng @RestController:**

- ✅ **Khi xây dựng REST API:** API cho Frontend, Mobile App gọi
- ✅ **Khi trả về JSON:** Dữ liệu dạng JSON (không phải HTML)
- ✅ **Khi xây dựng Backend riêng:** Backend và Frontend tách biệt

**Khi nào KHÔNG dùng @RestController:**

- ❌ **Khi xây dựng web application có giao diện:** Dùng `@Controller` (trả về HTML)
- ❌ **Khi cần trả về View (HTML):** Dùng `@Controller` + Thymeleaf

**So sánh @RestController vs @Controller:**

| Tiêu chí | @RestController | @Controller |
|----------|----------------|-------------|
| **Trả về gì?** | JSON (tự động) | HTML View hoặc ModelAndView |
| **Dùng cho?** | REST API | Web application có giao diện |
| **Annotation tương đương** | `@Controller` + `@ResponseBody` | `@Controller` |
| **Ví dụ** | API cho Mobile App | Website có giao diện |

---

#### 2.4. Ví dụ minh họa

**Ví dụ đơn giản - Cách đúng:**

```java
package com.taxi.booking.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

// Đánh dấu class này là REST Controller
@RestController
// Tất cả URL trong class này sẽ bắt đầu bằng /api
@RequestMapping("/api")
public class WelcomeController {
    
    // Xử lý GET request: GET /api/welcome
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Kết quả khi gọi `GET /api/welcome`:**
- Response: `"Welcome to Taxi App"` (dạng JSON string)

**Ví dụ đơn giản - Cách sai:**

```java
// ❌ SAI - Thiếu @RestController
@RequestMapping("/api")
public class WelcomeController {
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Tại sao sai:**
- Thiếu `@RestController` → Spring không biết đây là REST Controller
- Kết quả: Spring sẽ tìm View (HTML file) tên "Welcome to Taxi App" → Không tìm thấy → Lỗi 404

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
package com.taxi.booking.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class WelcomeController {
    
    /**
     * API chào mừng - Trả về thông điệp chào mừng
     * URL: GET /api/welcome
     * Response: "Welcome to Taxi App"
     */
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
    
    /**
     * API thông tin ứng dụng
     * URL: GET /api/info
     * Response: Thông tin về ứng dụng
     */
    @GetMapping("/info")
    public String getInfo() {
        return "Taxi Booking System v1.0.0";
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ SAI - Dùng @Controller thay vì @RestController
@Controller  // SAI - Sẽ tìm View (HTML) thay vì trả về JSON
@RequestMapping("/api")
public class WelcomeController {
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";  // Spring sẽ tìm file welcome.html → Lỗi!
    }
}
```

**Tại sao sai:**
- `@Controller` dùng cho web application có giao diện (trả về HTML)
- Khi return String, Spring sẽ tìm file HTML tương ứng → Không tìm thấy → Lỗi 404
- Với REST API, phải dùng `@RestController` để trả về JSON

---

### 3. @RequestMapping - Định nghĩa URL base

#### 3.1. Định nghĩa

**Giải thích:** `@RequestMapping` là annotation đánh dấu URL cơ sở (base URL) cho tất cả các method trong Controller. Nó giống như địa chỉ nhà, tất cả các phòng (method) đều ở trong nhà đó.

**Ví dụ dễ hiểu:**
- `@RequestMapping("/api")` = Địa chỉ nhà là `/api`
- `@GetMapping("/welcome")` = Phòng welcome
- URL đầy đủ = `/api` + `/welcome` = `/api/welcome`

**Cấu trúc cơ bản:**

```java
@RestController
@RequestMapping("/api")  // URL base cho tất cả method trong class này
public class WelcomeController {
    @GetMapping("/welcome")  // URL đầy đủ: /api/welcome
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

---

#### 3.2. Cách thức hoạt động

**Quy trình:**

```
Bước 1: Client gọi API
    │
    │   GET /api/welcome
    │
    ▼
Bước 2: Spring tìm Controller có @RequestMapping("/api")
    │
    │   Tìm thấy WelcomeController
    │
    ▼
Bước 3: Spring tìm method có @GetMapping("/welcome")
    │
    │   Tìm thấy welcome() method
    │
    ▼
Bước 4: Gọi method và trả về response
```

**Ví dụ minh họa:**

```java
@RestController
@RequestMapping("/api")
public class WelcomeController {
    
    // URL: GET /api/welcome
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
    
    // URL: GET /api/info
    @GetMapping("/info")
    public String getInfo() {
        return "Taxi Booking System";
    }
}
```

**Giải thích:**
- `@RequestMapping("/api")` áp dụng cho tất cả method trong class
- `@GetMapping("/welcome")` → URL đầy đủ: `/api/welcome`
- `@GetMapping("/info")` → URL đầy đủ: `/api/info`

---

#### 3.3. Trường hợp sử dụng thực tế

**Khi nào dùng @RequestMapping:**

- ✅ **Khi muốn nhóm các API lại:** Tất cả API trong Controller có cùng prefix
- ✅ **Khi muốn version API:** `/api/v1`, `/api/v2`
- ✅ **Khi muốn phân loại API:** `/api/public`, `/api/admin`

**Ví dụ thực tế:**

```java
// API công khai (không cần đăng nhập)
@RestController
@RequestMapping("/api/public")
public class PublicController {
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome";
    }
}

// API admin (cần quyền admin)
@RestController
@RequestMapping("/api/admin")
public class AdminController {
    @GetMapping("/users")
    public List<User> getAllUsers() {
        // Logic lấy danh sách user
    }
}
```

---

#### 3.4. Ví dụ minh họa

**Ví dụ đơn giản - Cách đúng:**

```java
@RestController
@RequestMapping("/api")
public class WelcomeController {
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Ví dụ đơn giản - Cách sai:**

```java
// ❌ SAI - Không có @RequestMapping, URL sẽ không có prefix /api
@RestController
public class WelcomeController {
    @GetMapping("/welcome")  // URL sẽ là /welcome (thiếu /api)
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Tại sao sai:**
- Thiếu `@RequestMapping("/api")` → URL sẽ là `/welcome` thay vì `/api/welcome`
- Không tuân theo quy ước đặt tên URL (sẽ học ở phần sau)

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
package com.taxi.booking.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class WelcomeController {
    
    /**
     * API chào mừng
     * URL: GET /api/welcome
     */
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ SAI - Đặt @RequestMapping ở method thay vì class
@RestController
public class WelcomeController {
    @RequestMapping("/api/welcome")  // SAI - Nên đặt ở class
    @GetMapping
    public String welcome() {
        return "Welcome to Taxi App";
    }
}
```

**Tại sao sai:**
- `@RequestMapping` nên đặt ở class để áp dụng cho tất cả method
- Đặt ở method sẽ làm code lặp lại và khó bảo trì

---

### 4. Quy chuẩn đặt tên URL (Naming Convention)

#### 4.1. Định nghĩa

**Giải thích:** Quy chuẩn đặt tên URL là các quy tắc chung mà cộng đồng lập trình viên đồng ý sử dụng. Tuân theo quy chuẩn giúp code dễ đọc, dễ hiểu, và dễ bảo trì.

**Các quy tắc cơ bản:**

1. **Dùng chữ thường (lowercase):** `/api/welcome` ✅, `/api/Welcome` ❌
2. **Dùng dấu gạch ngang (-) thay vì underscore (_):** `/api/user-profile` ✅, `/api/user_profile` ❌
3. **Dùng danh từ số nhiều cho resource:** `/api/users` ✅, `/api/user` ❌
4. **Tránh động từ trong URL:** `/api/users` ✅, `/api/getUsers` ❌
5. **Dùng HTTP method để chỉ hành động:** GET = lấy, POST = tạo, PUT = cập nhật, DELETE = xóa

---

#### 4.2. Cách thức hoạt động

**Ví dụ minh họa quy chuẩn:**

```java
@RestController
@RequestMapping("/api")
public class UserController {
    
    // ✅ ĐÚNG - GET /api/users (lấy danh sách)
    @GetMapping("/users")
    public List<User> getAllUsers() {
        // ...
    }
    
    // ✅ ĐÚNG - GET /api/users/1 (lấy user có id = 1)
    @GetMapping("/users/{id}")
    public User getUserById(@PathVariable Long id) {
        // ...
    }
    
    // ✅ ĐÚNG - POST /api/users (tạo user mới)
    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        // ...
    }
    
    // ✅ ĐÚNG - PUT /api/users/1 (cập nhật user có id = 1)
    @PutMapping("/users/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        // ...
    }
    
    // ✅ ĐÚNG - DELETE /api/users/1 (xóa user có id = 1)
    @DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable Long id) {
        // ...
    }
}
```

**Giải thích:**
- `/api/users` (số nhiều) = Resource (tài nguyên)
- HTTP method (GET, POST, PUT, DELETE) = Hành động
- `/api/users/{id}` = Resource cụ thể (user có id = 1)

---

#### 4.3. Trường hợp sử dụng thực tế

**Bảng quy chuẩn cho các hành động CRUD:**

| Hành động | HTTP Method | URL | Ví dụ |
|-----------|-------------|-----|-------|
| **Lấy danh sách** | GET | `/api/resources` | `GET /api/users` |
| **Lấy một item** | GET | `/api/resources/{id}` | `GET /api/users/1` |
| **Tạo mới** | POST | `/api/resources` | `POST /api/users` |
| **Cập nhật** | PUT | `/api/resources/{id}` | `PUT /api/users/1` |
| **Xóa** | DELETE | `/api/resources/{id}` | `DELETE /api/users/1` |

**Ví dụ trong Taxi Booking System:**

```java
@RestController
@RequestMapping("/api/bookings")
public class BookingController {
    
    // Lấy danh sách booking
    @GetMapping
    public List<Booking> getAllBookings() {
        // ...
    }
    
    // Lấy booking theo ID
    @GetMapping("/{id}")
    public Booking getBookingById(@PathVariable Long id) {
        // ...
    }
    
    // Tạo booking mới
    @PostMapping
    public Booking createBooking(@RequestBody BookingRequest request) {
        // ...
    }
    
    // Cập nhật booking
    @PutMapping("/{id}")
    public Booking updateBooking(@PathVariable Long id, @RequestBody BookingRequest request) {
        // ...
    }
    
    // Xóa booking
    @DeleteMapping("/{id}")
    public void deleteBooking(@PathVariable Long id) {
        // ...
    }
}
```

---

#### 4.4. Ví dụ minh họa

**Ví dụ đơn giản - Cách đúng:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping
    public List<User> getAllUsers() {
        // ...
    }
    
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        // ...
    }
}
```

**Ví dụ đơn giản - Cách sai:**

```java
// ❌ SAI - Dùng động từ trong URL
@RestController
@RequestMapping("/api")
public class UserController {
    @GetMapping("/getUsers")  // SAI - Nên dùng /users
    public List<User> getAllUsers() {
        // ...
    }
    
    @GetMapping("/getUserById/{id}")  // SAI - Nên dùng /users/{id}
    public User getUserById(@PathVariable Long id) {
        // ...
    }
}
```

**Tại sao sai:**
- Dùng động từ (`getUsers`, `getUserById`) → Không tuân theo quy chuẩn REST
- HTTP method (GET) đã chỉ rõ hành động, không cần động từ trong URL

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
@RestController
@RequestMapping("/api/bookings")
public class BookingController {
    
    // ✅ ĐÚNG - GET /api/bookings (lấy danh sách)
    @GetMapping
    public List<Booking> getAllBookings() {
        // ...
    }
    
    // ✅ ĐÚNG - GET /api/bookings/1 (lấy booking có id = 1)
    @GetMapping("/{id}")
    public Booking getBookingById(@PathVariable Long id) {
        // ...
    }
    
    // ✅ ĐÚNG - POST /api/bookings (tạo booking mới)
    @PostMapping
    public Booking createBooking(@RequestBody BookingRequest request) {
        // ...
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ SAI - Không tuân theo quy chuẩn
@RestController
@RequestMapping("/api")
public class BookingController {
    
    @GetMapping("/getAllBookings")  // SAI
    public List<Booking> getAllBookings() {
        // ...
    }
    
    @GetMapping("/booking/{id}")  // SAI - Nên dùng /bookings/{id}
    public Booking getBookingById(@PathVariable Long id) {
        // ...
    }
    
    @PostMapping("/createBooking")  // SAI - Nên dùng POST /api/bookings
    public Booking createBooking(@RequestBody BookingRequest request) {
        // ...
    }
}
```

**Tại sao sai:**
- Dùng động từ trong URL → Không RESTful
- Dùng số ít (`/booking`) thay vì số nhiều (`/bookings`) → Không tuân theo quy chuẩn

---

### 5. VS Code với Spring Boot

#### 5.1. Run Spring Boot app trong VS Code

**Giải thích:** VS Code có thể chạy Spring Boot app trực tiếp mà không cần dùng Terminal. Điều này giúp bạn làm việc nhanh hơn.

**Cách 1: Dùng nút Run (Khuyên dùng - Dễ nhất)**

1. **Mở file có `@SpringBootApplication`:**
   - Tìm file `TaxiBookingBackendApplication.java` (hoặc file main application)
   - File này thường có annotation `@SpringBootApplication`

2. **Click nút Run:**
   - Bạn sẽ thấy nút "Run" (▶️) bên cạnh method `main`
   - Click vào nút đó → VS Code sẽ tự động chạy ứng dụng

3. **Xem kết quả:**
   - Xem Terminal (ở dưới cùng VS Code)
   - Nếu thấy: `Started TaxiBookingBackendApplication in X.XXX seconds` → Thành công!

**Ví dụ:**

```java
@SpringBootApplication
public class TaxiBookingBackendApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaxiBookingBackendApplication.class, args);
        // ↑ Click nút Run bên cạnh dòng này
    }
}
```

**Cách 2: Dùng phím tắt F5 (Debug mode)**

1. **Mở file có `@SpringBootApplication`**
2. **Nhấn `F5`** (hoặc `Cmd+F5` / `Ctrl+F5`)
3. **Chọn "Java"** trong danh sách hiện ra
4. **VS Code sẽ chạy ứng dụng ở chế độ debug**

**Lưu ý:**
- Nếu lần đầu chạy, VS Code có thể hỏi bạn chọn "Java" → Chọn "Java"
- VS Code sẽ tự động tạo file `.vscode/launch.json` để lưu cấu hình

---

#### 5.2. Debug Spring Boot app trong VS Code

**Giải thích:** Debug giúp bạn dừng code tại một điểm cụ thể và xem giá trị các biến, từng bước chạy code. Điều này rất hữu ích khi tìm lỗi.

**Các bước debug:**

1. **Set breakpoint (điểm dừng):**
   - Click vào số dòng bên trái (bên cạnh code)
   - Hoặc đặt con trỏ ở dòng code → Nhấn `F9`
   - Bạn sẽ thấy dấu chấm đỏ (breakpoint)

2. **Chạy ở chế độ debug:**
   - Nhấn `F5` (hoặc click nút "Run and Debug" ở thanh bên trái)
   - Chọn "Java" nếu được hỏi

3. **Khi code chạy đến breakpoint:**
   - Code sẽ dừng lại
   - Bạn có thể:
     - Xem giá trị các biến (ở panel bên trái)
     - Xem call stack (các method đã gọi)
     - Step over (`F10`): Chạy dòng tiếp theo
     - Step into (`F11`): Đi sâu vào method
     - Step out (`Shift+F11`): Thoát khỏi method hiện tại
     - Continue (`F5`): Tiếp tục chạy đến breakpoint tiếp theo

**Ví dụ thực hành:**

```java
@RestController
@RequestMapping("/api")
public class WelcomeController {
    
    @GetMapping("/welcome")
    public String welcome() {
        String message = "Welcome to Taxi App";  // ← Đặt breakpoint ở đây
        return message;  // ← Hoặc ở đây
    }
}
```

**Các bước:**
1. Đặt breakpoint ở dòng `String message = ...`
2. Nhấn `F5` để chạy debug
3. Gọi API `GET /api/welcome` từ Postman
4. Code sẽ dừng ở breakpoint
5. Hover chuột vào `message` → Xem giá trị
6. Nhấn `F10` để chạy dòng tiếp theo
7. Nhấn `F5` để tiếp tục

---

#### 5.3. Xem logs trong VS Code Terminal

**Giải thích:** Khi chạy Spring Boot app, tất cả logs (thông báo) sẽ hiển thị trong Terminal của VS Code. Đây là nơi bạn xem lỗi, thông báo, và output của ứng dụng.

**Cách xem logs:**

1. **Mở Terminal:**
   - Cách 1: `View → Terminal` (hoặc `Ctrl+` ` / `Cmd+` `)
   - Cách 2: Click tab "Terminal" ở dưới cùng VS Code

2. **Khi chạy app:**
   - Logs sẽ tự động hiển thị trong Terminal
   - Bạn sẽ thấy:
     - Thông báo khởi động Spring Boot
     - Port mà app đang chạy (thường là 8080)
     - Các log từ code của bạn (nếu có `System.out.println()`)

**Ví dụ logs khi chạy thành công:**

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.x.x)

2024-01-01 10:00:00.123  INFO 12345 --- [           main] c.t.b.TaxiBookingBackendApplication       : Starting TaxiBookingBackendApplication
2024-01-01 10:00:01.456  INFO 12345 --- [           main] c.t.b.TaxiBookingBackendApplication       : Started TaxiBookingBackendApplication in 1.333 seconds
```

**Giải thích:**
- Dòng đầu: Logo Spring Boot
- `INFO`: Mức độ log (INFO, WARN, ERROR)
- `Starting...`: Ứng dụng đang khởi động
- `Started... in 1.333 seconds`: Ứng dụng đã khởi động thành công, mất 1.333 giây

**Cách in log từ code:**

```java
@RestController
@RequestMapping("/api")
public class WelcomeController {
    
    @GetMapping("/welcome")
    public String welcome() {
        System.out.println("API welcome được gọi!");  // In log ra Terminal
        return "Welcome to Taxi App";
    }
}
```

**Khi gọi API, Terminal sẽ hiển thị:**
```
API welcome được gọi!
```

---

#### 5.4. Auto-reload với Spring Boot DevTools (Tùy chọn)

**Giải thích:** Spring Boot DevTools giúp ứng dụng tự động reload (tải lại) khi bạn sửa code. Thay vì phải dừng và chạy lại app mỗi lần sửa code, DevTools sẽ tự động làm điều đó.

**Cách cài đặt:**

1. **Thêm dependency vào `pom.xml`:**
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
       <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   ```

2. **Reload Maven dependencies:**
   - Click vào file `pom.xml`
   - Nhấn `Cmd+Shift+P` / `Ctrl+Shift+P` → Gõ "Java: Reload Projects"
   - Hoặc đợi VS Code tự động reload

3. **Chạy ứng dụng:**
   - Chạy app như bình thường
   - Khi bạn sửa code và lưu file, app sẽ tự động reload

**Lưu ý:**
- DevTools chỉ reload khi bạn **lưu file** (Save)
- DevTools không reload khi thay đổi cấu trúc class (thêm/xóa method) → Phải restart app
- DevTools chỉ hoạt động trong môi trường development (không hoạt động trong production)

**Ví dụ:**
1. Chạy app
2. Sửa code trong `WelcomeController`:
   ```java
   return "Welcome to Taxi App - Updated!";
   ```
3. Lưu file (`Cmd+S` / `Ctrl+S`)
4. DevTools tự động reload → Bạn sẽ thấy log "Restarting..." trong Terminal
5. Gọi lại API → Kết quả đã được cập nhật

---

## Thực hành

### Bài tập 1: Tạo API Hello World

**Mục tiêu:** Tạo API đơn giản nhất để làm quen với Spring Boot REST API

**Yêu cầu:**

1. **Tạo package `controller`:**
   - Trong VS Code, right-click vào package `com.taxi.booking`
   - Chọn "New Folder" → Gõ `controller`

2. **Tạo class `WelcomeController`:**
   - Right-click vào package `controller`
   - Chọn "New File" → Gõ `WelcomeController.java`

3. **Viết code:**
   ```java
   package com.taxi.booking.controller;

   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   @RequestMapping("/api")
   public class WelcomeController {
       
       @GetMapping("/hello")
       public String hello() {
           return "Hello World from Spring Boot!";
       }
   }
   ```

4. **Chạy ứng dụng:**
   - Mở file `TaxiBookingBackendApplication.java`
   - Click nút "Run" bên cạnh method `main`
   - Đợi app khởi động (xem Terminal)

5. **Test API:**
   - Mở Postman hoặc Thunder Client
   - Gọi `GET http://localhost:8080/api/hello`
   - Kết quả mong đợi: `"Hello World from Spring Boot!"`

**Kết quả mong đợi:**
- ✅ App chạy thành công (không có lỗi)
- ✅ Gọi API thành công và nhận được response
- ✅ Response là JSON string: `"Hello World from Spring Boot!"`

---

### Bài tập 2: Run và Debug app trong VS Code

**Mục tiêu:** Làm quen với cách chạy và debug Spring Boot app trong VS Code

**Yêu cầu:**

1. **Run app bằng nút Run:**
   - Mở file `TaxiBookingBackendApplication.java`
   - Click nút "Run" (▶️) bên cạnh method `main`
   - Quan sát Terminal → Xem logs khởi động

2. **Dừng app:**
   - Nhấn `Ctrl+C` trong Terminal
   - Hoặc click nút "Stop" (nếu có)

3. **Debug app:**
   - Mở file `WelcomeController.java`
   - Đặt breakpoint ở dòng `return "Hello World...";` (click vào số dòng bên trái)
   - Nhấn `F5` để chạy debug
   - Chọn "Java" nếu được hỏi
   - Gọi API `GET /api/hello` từ Postman
   - Code sẽ dừng ở breakpoint
   - Hover chuột vào các biến để xem giá trị
   - Nhấn `F10` để chạy dòng tiếp theo
   - Nhấn `F5` để tiếp tục

4. **Thử các phím tắt debug:**
   - `F9`: Toggle breakpoint (bật/tắt breakpoint)
   - `F10`: Step over (chạy dòng tiếp theo)
   - `F11`: Step into (đi sâu vào method)
   - `Shift+F11`: Step out (thoát khỏi method)
   - `F5`: Continue (tiếp tục chạy)

**Kết quả mong đợi:**
- ✅ Đã chạy app thành công bằng nút Run
- ✅ Đã debug app và thấy code dừng ở breakpoint
- ✅ Đã thử các phím tắt debug
- ✅ Hiểu cách sử dụng debug để tìm lỗi

---

### Bài tập 3: Tạo nhiều API endpoint

**Mục tiêu:** Tạo nhiều API endpoint khác nhau trong cùng một Controller

**Yêu cầu:**

1. **Mở file `WelcomeController.java`**

2. **Thêm các API endpoint:**
   ```java
   @RestController
   @RequestMapping("/api")
   public class WelcomeController {
       
       @GetMapping("/hello")
       public String hello() {
           return "Hello World from Spring Boot!";
       }
       
       @GetMapping("/welcome")
       public String welcome() {
           return "Welcome to Taxi App!";
       }
       
       @GetMapping("/info")
       public String getInfo() {
           return "Taxi Booking System v1.0.0";
       }
       
       @GetMapping("/status")
       public String getStatus() {
           return "Server is running!";
       }
   }
   ```

3. **Chạy app và test từng API:**
   - `GET /api/hello` → `"Hello World from Spring Boot!"`
   - `GET /api/welcome` → `"Welcome to Taxi App!"`
   - `GET /api/info` → `"Taxi Booking System v1.0.0"`
   - `GET /api/status` → `"Server is running!"`

4. **Quan sát logs trong Terminal:**
   - Mỗi lần gọi API, xem có log nào xuất hiện không

**Kết quả mong đợi:**
- ✅ Đã tạo 4 API endpoint khác nhau
- ✅ Tất cả API đều hoạt động và trả về response đúng
- ✅ Hiểu cách tổ chức nhiều endpoint trong một Controller

---

## Dự án Taxi

### Bài tập: Tạo cấu trúc packages và API Welcome

**Mục tiêu:** Tạo cấu trúc packages chuẩn cho dự án Taxi và viết API welcome đầu tiên

**Yêu cầu:**

1. **Tạo cấu trúc packages chi tiết:**

   Trong VS Code, tạo các package sau:
   ```
   com.taxi.booking/
   ├── TaxiBookingBackendApplication.java
   ├── controller/
   ├── service/
   ├── repository/
   ├── entity/
   └── dto/
   ```

   **Cách tạo package:**
   - Right-click vào `com.taxi.booking`
   - Chọn "New Folder" → Gõ tên package (ví dụ: `controller`)
   - Lặp lại cho các package khác

2. **Tạo Controller package và WelcomeController:**

   **File:** `src/main/java/com/taxi/booking/controller/WelcomeController.java`
   
   ```java
   package com.taxi.booking.controller;

   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;

   /**
    * Controller xử lý các API chào mừng và thông tin cơ bản
    */
   @RestController
   @RequestMapping("/api")
   public class WelcomeController {
       
       /**
        * API chào mừng - Trả về thông điệp chào mừng
        * URL: GET /api/welcome
        * Response: "Welcome to Taxi App"
        */
       @GetMapping("/welcome")
       public String welcome() {
           return "Welcome to Taxi App";
       }
   }
   ```

3. **Chạy ứng dụng trong VS Code:**

   - Mở file `TaxiBookingBackendApplication.java`
   - Click nút "Run" (▶️) bên cạnh method `main`
   - Đợi app khởi động (xem Terminal)
   - Nếu thấy: `Started TaxiBookingBackendApplication in X.XXX seconds` → Thành công!

4. **Test API bằng Postman/Thunder Client:**

   **Cách 1: Dùng Thunder Client (trong VS Code)**
   - Click icon Thunder Client ở thanh bên trái VS Code
   - Click "New Request"
   - Method: `GET`
   - URL: `http://localhost:8080/api/welcome`
   - Click "Send"
   - Xem response: `"Welcome to Taxi App"`

   **Cách 2: Dùng Postman**
   - Mở Postman
   - Tạo request mới: `GET http://localhost:8080/api/welcome`
   - Click "Send"
   - Xem response: `"Welcome to Taxi App"`

5. **Kiểm tra logs trong Terminal:**

   - Xem Terminal trong VS Code
   - Khi gọi API, bạn sẽ thấy log (nếu có)
   - Ví dụ: `GET "/api/welcome", parameters={}`

6. **Thử debug API:**

   - Đặt breakpoint trong method `welcome()`
   - Chạy app ở chế độ debug (`F5`)
   - Gọi API từ Postman/Thunder Client
   - Code sẽ dừng ở breakpoint
   - Xem giá trị các biến
   - Nhấn `F10` để chạy tiếp
   - Xem response trả về

**Kết quả mong đợi:**
- ✅ Đã tạo cấu trúc packages chuẩn: `controller`, `service`, `repository`, `entity`, `dto`
- ✅ Đã tạo `WelcomeController` với API `GET /api/welcome`
- ✅ App chạy thành công trong VS Code
- ✅ API hoạt động và trả về `"Welcome to Taxi App"`
- ✅ Đã test API bằng Postman/Thunder Client
- ✅ Đã thử debug API và thấy code dừng ở breakpoint

**Lưu ý quan trọng:**
- Đảm bảo app chạy trên port 8080 (mặc định của Spring Boot)
- Nếu port 8080 đã được dùng, Spring Boot sẽ tự động chọn port khác (xem logs trong Terminal)
- Nếu có lỗi, kiểm tra:
  - Đã import đúng các annotation chưa?
  - Package name có đúng không?
  - App đã chạy chưa?

**Troubleshooting:**
- **Lỗi "Port 8080 already in use":** Đổi port trong `application.properties`: `server.port=8081`
- **Lỗi "Cannot resolve symbol":** Reload Maven dependencies: `Cmd+Shift+P` → "Java: Reload Projects"
- **API không hoạt động:** Kiểm tra app đã chạy chưa (xem Terminal)

---

## Tổng kết buổi 3

**Những gì đã học:**
1. ✅ Hiểu cấu trúc project Spring Boot chuẩn
2. ✅ Hiểu package structure và vai trò từng package
3. ✅ Hiểu `@RestController` và cách tạo REST API
4. ✅ Hiểu `@RequestMapping` và cách định nghĩa URL base
5. ✅ Hiểu quy chuẩn đặt tên URL (Naming Convention)
6. ✅ Biết cách run và debug Spring Boot app trong VS Code
7. ✅ Biết cách xem logs trong VS Code Terminal
8. ✅ Đã tạo API đầu tiên cho dự án Taxi

**Kiến thức quan trọng:**
- **Cấu trúc project:** `controller/`, `service/`, `repository/`, `entity/`, `dto/`
- **@RestController:** Đánh dấu class là REST Controller, trả về JSON
- **@RequestMapping:** Định nghĩa URL base cho Controller
- **Quy chuẩn URL:**
  - Dùng chữ thường, dấu gạch ngang
  - Dùng danh từ số nhiều cho resource
  - Dùng HTTP method để chỉ hành động
- **VS Code với Spring Boot:**
  - Run: Click nút Run hoặc `F5`
  - Debug: Set breakpoint → `F5`
  - Logs: Xem trong Terminal

**Chuẩn bị cho buổi 4:**
- ✅ Đã tạo cấu trúc packages chuẩn
- ✅ Đã tạo API đầu tiên (`GET /api/welcome`)
- ✅ Đã biết cách run và debug app trong VS Code
- ✅ Sẵn sàng học Request Handling và Response Entity

**Kiểm tra lại trước buổi 4:**
- [ ] Đã tạo cấu trúc packages: `controller`, `service`, `repository`, `entity`, `dto`
- [ ] Đã tạo `WelcomeController` với API `GET /api/welcome`
- [ ] App chạy thành công và API hoạt động
- [ ] Đã test API bằng Postman/Thunder Client
- [ ] Đã thử debug API và thấy code dừng ở breakpoint
- [ ] Hiểu quy chuẩn đặt tên URL

**Bài tập về nhà (tùy chọn):**
- Tạo thêm các API endpoint khác trong `WelcomeController`:
  - `GET /api/info` → Trả về thông tin ứng dụng
  - `GET /api/status` → Trả về trạng thái server
- Thử tạo Controller mới: `HealthController` với API `GET /api/health`
- Đọc thêm về REST API tại https://restfulapi.net/
- Xem lại các annotation đã học: `@RestController`, `@RequestMapping`, `@GetMapping`

