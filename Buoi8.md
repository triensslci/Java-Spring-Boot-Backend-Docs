# Buổi 8: Validation & Global Exception Handling

## Ôn lại buổi trước

Trước khi bắt đầu validation và xử lý lỗi tập trung, hãy kiểm tra lại bạn đã hoàn thành các đầu việc chính của **Buổi 7**:

- Đã **tách riêng Entity và DTO** trong module User:
  - Entity `User` chỉ dùng trong tầng Service/Repository.
  - `RegisterUserRequest` dùng cho request.
  - `UserResponseDTO` dùng cho response (không chứa password).
- Đã **refactor API đăng ký** theo pattern RequestDTO → Entity → ResponseDTO:
  - Controller nhận `RegisterUserRequest`.
  - Service map DTO → Entity, lưu DB, map Entity → ResponseDTO.
  - Response không lộ password.
- Đã **thực hành VS Code Refactoring**:
  - Extract Method để tách logic mapping.
  - Rename Symbol (`F2`) để đổi tên DTO nhất quán.
  - Organize Imports (`Shift+Alt+O`).
- Đã **test lại API đăng ký** sau khi refactor, đảm bảo vẫn hoạt động đúng.

Nếu bất kỳ mục nào phía trên chưa hoàn thành, hãy quay lại buổi 7 làm cho chắc, vì buổi 8 sẽ **thêm validation và exception handling** trên nền tảng DTO đã có sẵn.

---

## Kiến thức

Trong buổi này, trọng tâm là:

- **Validation (Xác thực dữ liệu)**: Đảm bảo dữ liệu từ client đúng định dạng, đầy đủ, hợp lệ trước khi xử lý.
- **Global Exception Handling (Xử lý lỗi tập trung)**: Bắt tất cả lỗi ở một nơi, trả về JSON lỗi đẹp, nhất quán cho client.
- Làm quen với các annotation validation: `@NotNull`, `@Email`, `@Size`, `@Pattern`.
- Xây dựng `@ControllerAdvice` để xử lý exception tập trung, không phải try-catch ở mọi Controller.

### 1. Validation là gì? Tại sao cần validate dữ liệu?

#### 1. Định Nghĩa

**Validation (Xác thực dữ liệu)** là quá trình kiểm tra dữ liệu từ client (request) có đúng định dạng, đầy đủ, và hợp lệ hay không **trước khi** xử lý nghiệp vụ.

Bạn có thể hình dung validation giống như **nhân viên bảo vệ ở cửa siêu thị**:
- Kiểm tra khách hàng có đủ tuổi không (validate age).
- Kiểm tra khách hàng có mang theo vật cấm không (validate format).
- Chỉ cho phép vào khi đã kiểm tra xong (chỉ xử lý khi validation pass).

Trong Taxi Booking System:
- Khi Passenger đăng ký, phải kiểm tra email đúng định dạng, password đủ dài, số điện thoại đúng 10 số.
- Khi tạo Booking, phải kiểm tra địa điểm đón/đến không được rỗng, khoảng cách phải dương.

#### 2. Cách Thức Hoạt Động

1. **Client gửi request** với JSON body (ví dụ: `RegisterUserRequest`).
2. **Spring Boot nhận request** và map JSON vào DTO.
3. **Validation xảy ra tự động** (nếu có `@Valid` hoặc `@Validated`):
   - Spring quét tất cả annotation validation trên các field của DTO (`@NotNull`, `@Email`, `@Size`...).
   - Nếu field nào vi phạm rule → Spring tạo `BindingResult` chứa danh sách lỗi.
   - Nếu có lỗi validation → Spring ném `MethodArgumentNotValidException`.
4. **Exception Handler bắt lỗi** (nếu có `@ControllerAdvice`):
   - Chuyển đổi `MethodArgumentNotValidException` thành JSON response đẹp.
   - Trả về status code `400 Bad Request` kèm danh sách lỗi chi tiết.
5. **Nếu validation pass**:
   - Request tiếp tục đi vào Controller → Service → Repository.

**Luồng validation trong Spring Boot:**

```
Client Request (JSON)
    ↓
@RequestBody + @Valid → DTO được map
    ↓
Spring Validation Engine kiểm tra annotation
    ↓
Có lỗi? → MethodArgumentNotValidException → @ControllerAdvice → JSON lỗi
    ↓
Không lỗi? → Controller → Service → Repository → Success Response
```

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ **Validate form đăng ký User/Driver**:
  - Email phải đúng định dạng (`@Email`).
  - Password tối thiểu 8 ký tự (`@Size(min = 8)`).
  - Số điện thoại phải đúng 10 số (`@Pattern(regexp = "^[0-9]{10}$")`).
  - Full name không được rỗng (`@NotBlank`).
- ✅ **Validate tạo Booking**:
  - Pickup location và dropoff location không được rỗng (`@NotBlank`).
  - Distance phải dương (`@Min(0.1)`).
  - Car type phải hợp lệ (`@Pattern` hoặc enum).
- ✅ **Validate update thông tin**:
  - Khi update, chỉ validate field được gửi lên (dùng `@Valid` trên DTO update).
- ❌ Không nên validate ở tầng Repository:
  - Repository chỉ lo truy cập DB, validation nên ở tầng Controller/Service.
- ❌ Không nên validate bằng cách tự viết if-else dài dòng:
  - Dùng annotation validation của Spring Boot (`javax.validation` hoặc `jakarta.validation`) để code ngắn gọn, dễ đọc.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng: Dùng annotation validation**

```java
// DTO với validation annotation
public class RegisterUserRequest {
    
    @NotBlank(message = "Họ tên không được để trống")
    @Size(min = 2, max = 100, message = "Họ tên phải từ 2 đến 100 ký tự")
    public String fullName;
    
    @NotBlank(message = "Email không được để trống")
    @Email(message = "Email không đúng định dạng")
    public String email;
    
    @NotBlank(message = "Mật khẩu không được để trống")
    @Size(min = 8, max = 50, message = "Mật khẩu phải từ 8 đến 50 ký tự")
    public String password;
    
    @NotBlank(message = "Số điện thoại không được để trống")
    @Pattern(regexp = "^[0-9]{10}$", message = "Số điện thoại phải đúng 10 chữ số")
    public String phone;
    
    @NotBlank(message = "Vai trò không được để trống")
    @Pattern(regexp = "^(ROLE_PASSENGER|ROLE_DRIVER)$", message = "Vai trò phải là ROLE_PASSENGER hoặc ROLE_DRIVER")
    public String role;
}
```

```java
// Controller với @Valid để kích hoạt validation
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping("/register")
    public ResponseEntity<UserResponseDTO> register(
        @Valid @RequestBody RegisterUserRequest request  // @Valid kích hoạt validation
    ) {
        UserResponseDTO response = userService.registerNewUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

**Ví dụ đơn giản - Cách sai: Không dùng validation, tự kiểm tra bằng if-else**

```java
// ❌ Sai: Controller tự validate bằng if-else dài dòng
@RestController
@RequestMapping("/api/users")
public class UserControllerBad {
    
    @PostMapping("/register")
    public ResponseEntity<?> register(@RequestBody RegisterUserRequest request) {
        // Tự validate bằng tay - code dài, khó bảo trì
        if (request.fullName == null || request.fullName.trim().isEmpty()) {
            return ResponseEntity.badRequest().body("Họ tên không được để trống");
        }
        if (request.email == null || !request.email.contains("@")) {
            return ResponseEntity.badRequest().body("Email không đúng định dạng");
        }
        if (request.password == null || request.password.length() < 8) {
            return ResponseEntity.badRequest().body("Mật khẩu phải tối thiểu 8 ký tự");
        }
        // ... còn rất nhiều if-else nữa ...
        
        // Logic xử lý...
    }
}
```

**Tại sao sai:**

- Code dài dòng, khó đọc, khó bảo trì.
- Validation logic nằm rải rác trong Controller, khó tái sử dụng.
- Response lỗi không nhất quán (có thể trả String, có thể trả Map...).
- Khó test: phải test từng case if-else riêng lẻ.

**Ví dụ trong dự án Taxi - Cách đúng: Validate đăng ký User**

```java
// RegisterUserRequest với đầy đủ validation
public class RegisterUserRequest {
    
    @NotBlank(message = "Họ tên không được để trống")
    @Size(min = 2, max = 100, message = "Họ tên phải từ 2 đến 100 ký tự")
    public String fullName;
    
    @NotBlank(message = "Email không được để trống")
    @Email(message = "Email phải đúng định dạng (ví dụ: user@example.com)")
    public String email;
    
    @NotBlank(message = "Mật khẩu không được để trống")
    @Size(min = 8, max = 50, message = "Mật khẩu phải từ 8 đến 50 ký tự")
    @Pattern(regexp = ".*[A-Z].*", message = "Mật khẩu phải có ít nhất 1 chữ hoa")
    @Pattern(regexp = ".*[0-9].*", message = "Mật khẩu phải có ít nhất 1 chữ số")
    public String password;
    
    @NotBlank(message = "Số điện thoại không được để trống")
    @Pattern(regexp = "^[0-9]{10}$", message = "Số điện thoại phải đúng 10 chữ số")
    public String phone;
    
    @NotBlank(message = "Vai trò không được để trống")
    @Pattern(regexp = "^(ROLE_PASSENGER|ROLE_DRIVER)$", 
             message = "Vai trò phải là ROLE_PASSENGER hoặc ROLE_DRIVER")
    public String role;
}
```

```java
// Controller với @Valid
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping("/register")
    public ResponseEntity<UserResponseDTO> register(
        @Valid @RequestBody RegisterUserRequest request
    ) {
        // Nếu validation fail, Spring sẽ ném MethodArgumentNotValidException
        // và @ControllerAdvice sẽ bắt, không bao giờ chạy đến dòng này
        UserResponseDTO response = userService.registerNewUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai: Validate không đầy đủ**

```java
// ❌ Sai: Chỉ validate một vài field, bỏ sót field quan trọng
public class RegisterUserRequest {
    
    @NotBlank
    public String fullName;
    
    // ❌ Thiếu @Email, chỉ có @NotBlank
    @NotBlank
    public String email;
    
    // ❌ Thiếu @Size, password có thể quá ngắn
    @NotBlank
    public String password;
    
    // ❌ Không có validation cho phone và role
    public String phone;
    public String role;
}
```

**Tại sao sai:**

- Email có thể nhận giá trị không phải email (ví dụ: "abc").
- Password có thể quá ngắn (ví dụ: "123").
- Phone và role không được kiểm tra → có thể nhận giá trị sai, gây lỗi khi lưu DB.

---

### 2. Các Annotation Validation Phổ Biến

#### 1. Định Nghĩa

Spring Boot sử dụng **Bean Validation API** (JSR-303/JSR-380) với các annotation có sẵn để validate dữ liệu. Mỗi annotation có một mục đích cụ thể:

- **`@NotNull`**: Field không được null (nhưng có thể là chuỗi rỗng "").
- **`@NotBlank`**: Field không được null, không được rỗng, và không được chỉ có khoảng trắng (dùng cho String).
- **`@NotEmpty`**: Field không được null và không được rỗng (dùng cho Collection, Array, String).
- **`@Email`**: Field phải đúng định dạng email.
- **`@Size`**: Kiểm tra độ dài (min, max) cho String, Collection, Array.
- **`@Min` / `@Max`**: Kiểm tra giá trị số tối thiểu/tối đa.
- **`@Pattern`**: Kiểm tra String có khớp với regex pattern không.
- **`@Positive` / `@Negative`**: Kiểm tra số dương/âm.

Bạn có thể hình dung các annotation này giống như **các loại thẻ kiểm tra**:
- `@NotBlank` = "Thẻ kiểm tra: Không được để trống".
- `@Email` = "Thẻ kiểm tra: Phải đúng định dạng email".
- `@Size(min = 8)` = "Thẻ kiểm tra: Phải có ít nhất 8 ký tự".

#### 2. Cách Thức Hoạt Động

1. **Khi bạn đặt annotation trên field DTO**:
   ```java
   @NotBlank(message = "Email không được để trống")
   @Email(message = "Email không đúng định dạng")
   public String email;
   ```
   - Spring Boot đăng ký validator cho field này.
2. **Khi request đến Controller với `@Valid`**:
   ```java
   @PostMapping("/register")
   public ResponseEntity<?> register(@Valid @RequestBody RegisterUserRequest request) {
       // ...
   }
   ```
   - Spring Boot tự động gọi validator cho từng field.
3. **Nếu field vi phạm rule**:
   - Validator tạo `ConstraintViolation` chứa thông tin lỗi.
   - Spring ném `MethodArgumentNotValidException` (cho `@RequestBody`) hoặc `ConstraintViolationException` (cho `@RequestParam`, `@PathVariable`).
4. **Exception Handler bắt lỗi**:
   - Chuyển đổi exception thành JSON response với danh sách lỗi chi tiết.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ **`@NotBlank`**: Dùng cho String bắt buộc (fullName, email, password...).
- ✅ **`@Email`**: Dùng cho field email trong form đăng ký/đăng nhập.
- ✅ **`@Size(min = 8, max = 50)`**: Dùng cho password (tối thiểu 8 ký tự, tối đa 50).
- ✅ **`@Pattern(regexp = "^[0-9]{10}$")`**: Dùng cho số điện thoại (phải đúng 10 số).
- ✅ **`@Min(0.1)`**: Dùng cho distance, price (phải dương).
- ✅ **`@NotNull`**: Dùng cho field không phải String (Long, Integer, Boolean...).
- ❌ Không nên dùng `@NotNull` cho String (nên dùng `@NotBlank`).
- ❌ Không nên validate quá chặt (ví dụ: password phải có ký tự đặc biệt, số, chữ hoa, chữ thường) nếu không thực sự cần thiết → khó cho user.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng: Sử dụng đúng annotation cho từng trường hợp**

```java
public class CreateBookingRequest {
    
    // String bắt buộc → dùng @NotBlank
    @NotBlank(message = "Địa điểm đón không được để trống")
    public String pickupLocation;
    
    @NotBlank(message = "Địa điểm đến không được để trống")
    public String dropoffLocation;
    
    // Số dương → dùng @Min
    @Min(value = 0, message = "Khoảng cách phải lớn hơn hoặc bằng 0")
    public Double distanceKm;
    
    // Enum/String có giá trị cố định → dùng @Pattern
    @Pattern(regexp = "^(4_SEATER|7_SEATER|MOTORBIKE)$", 
             message = "Loại xe phải là 4_SEATER, 7_SEATER hoặc MOTORBIKE")
    public String carType;
}
```

**Ví dụ đơn giản - Cách sai: Dùng sai annotation**

```java
public class CreateBookingRequestBad {
    
    // ❌ Sai: Dùng @NotNull cho String, không kiểm tra chuỗi rỗng
    @NotNull(message = "Địa điểm đón không được null")
    public String pickupLocation; // Có thể nhận "" (chuỗi rỗng)
    
    // ❌ Sai: Dùng @NotBlank cho số, @NotBlank chỉ dùng cho String
    @NotBlank(message = "Khoảng cách không được để trống")
    public Double distanceKm; // ❌ Lỗi compile hoặc không hoạt động đúng
    
    // ❌ Sai: Không có validation cho carType, có thể nhận giá trị sai
    public String carType; // Có thể nhận "INVALID_TYPE"
}
```

**Tại sao sai:**

- `@NotNull` cho String không kiểm tra chuỗi rỗng `""` → vẫn pass validation dù không có dữ liệu.
- `@NotBlank` chỉ dùng cho String, không dùng cho số → compile error hoặc không hoạt động.
- Thiếu validation cho `carType` → có thể nhận giá trị không hợp lệ, gây lỗi khi xử lý nghiệp vụ.

**Ví dụ trong dự án Taxi - Cách đúng: Validate đầy đủ cho RegisterUserRequest**

```java
public class RegisterUserRequest {
    
    @NotBlank(message = "Họ tên không được để trống")
    @Size(min = 2, max = 100, message = "Họ tên phải từ 2 đến 100 ký tự")
    public String fullName;
    
    @NotBlank(message = "Email không được để trống")
    @Email(message = "Email phải đúng định dạng (ví dụ: user@example.com)")
    public String email;
    
    @NotBlank(message = "Mật khẩu không được để trống")
    @Size(min = 8, max = 50, message = "Mật khẩu phải từ 8 đến 50 ký tự")
    public String password;
    
    @NotBlank(message = "Số điện thoại không được để trống")
    @Pattern(regexp = "^[0-9]{10}$", message = "Số điện thoại phải đúng 10 chữ số")
    public String phone;
    
    @NotBlank(message = "Vai trò không được để trống")
    @Pattern(regexp = "^(ROLE_PASSENGER|ROLE_DRIVER)$", 
             message = "Vai trò phải là ROLE_PASSENGER hoặc ROLE_DRIVER")
    public String role;
}
```

**Ví dụ trong dự án Taxi - Cách sai: Validate không đầy đủ hoặc sai cách**

```java
// ❌ Sai: Chỉ validate một phần, bỏ sót nhiều field
public class RegisterUserRequestBad {
    
    @NotBlank
    public String fullName;
    
    // ❌ Thiếu @Email, chỉ có @NotBlank
    @NotBlank
    public String email;
    
    // ❌ Thiếu @Size, password có thể quá ngắn
    @NotBlank
    public String password;
    
    // ❌ Không có validation cho phone và role
    public String phone;
    public String role;
}
```

**Tại sao sai:**

- Email có thể nhận giá trị không phải email (ví dụ: "abc", "not-an-email").
- Password có thể quá ngắn (ví dụ: "123") → không đảm bảo bảo mật.
- Phone và role không được kiểm tra → có thể nhận giá trị sai, gây lỗi khi lưu DB hoặc xử lý nghiệp vụ.

---

### 3. Global Exception Handling với @ControllerAdvice

#### 1. Định Nghĩa

**`@ControllerAdvice`** là một annotation của Spring Boot cho phép bạn tạo một class **xử lý exception tập trung** cho toàn bộ ứng dụng. Thay vì phải try-catch ở mọi Controller, bạn chỉ cần tạo một class duy nhất để bắt tất cả lỗi và trả về JSON response đẹp, nhất quán.

Bạn có thể hình dung `@ControllerAdvice` giống như **trung tâm xử lý khiếu nại**:
- Thay vì mỗi bộ phận tự xử lý khiếu nại riêng (Controller tự try-catch).
- Tất cả khiếu nại đều được gửi về một trung tâm duy nhất (`@ControllerAdvice`).
- Trung tâm này xử lý và trả lời nhất quán, chuyên nghiệp.

Trong Taxi Booking System:
- Khi validation fail → `MethodArgumentNotValidException` → `@ControllerAdvice` bắt và trả JSON lỗi đẹp.
- Khi email trùng → `UserAlreadyExistsException` (custom) → `@ControllerAdvice` bắt và trả JSON lỗi đẹp.
- Khi không tìm thấy user → `UserNotFoundException` (custom) → `@ControllerAdvice` bắt và trả `404 Not Found`.

#### 2. Cách Thức Hoạt Động

1. **Bạn tạo class `GlobalExceptionHandler`** với annotation `@ControllerAdvice`:
   ```java
   @ControllerAdvice
   public class GlobalExceptionHandler {
       // Các method xử lý exception ở đây
   }
   ```
   - Spring Boot tự động phát hiện class này khi khởi động.
   - Đăng ký class này như một **interceptor** cho tất cả Controller.

2. **Bạn viết method xử lý exception** với annotation `@ExceptionHandler`:
   ```java
   @ExceptionHandler(MethodArgumentNotValidException.class)
   public ResponseEntity<ErrorResponse> handleValidationException(
       MethodArgumentNotValidException ex
   ) {
       // Xử lý lỗi validation, trả về JSON đẹp
   }
   ```
   - `@ExceptionHandler` chỉ định loại exception nào method này sẽ xử lý.
   - Method này sẽ được gọi **tự động** khi exception tương ứng xảy ra ở bất kỳ Controller nào.

3. **Khi exception xảy ra trong Controller**:
   - Spring Boot tìm method `@ExceptionHandler` phù hợp trong `@ControllerAdvice`.
   - Gọi method đó và trả về response từ method đó.
   - Controller gốc **không cần** try-catch.

4. **Response trả về nhất quán**:
   - Tất cả lỗi đều trả về cùng format JSON (ví dụ: `{ "message": "...", "status": 400, "errors": [...] }`).

**Luồng xử lý exception:**

```
Controller method throw exception
    ↓
Spring Boot tìm @ExceptionHandler phù hợp trong @ControllerAdvice
    ↓
@ExceptionHandler method xử lý exception
    ↓
Trả về ResponseEntity với JSON lỗi đẹp
    ↓
Client nhận được response lỗi nhất quán
```

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ **Xử lý validation errors**:
  - Bắt `MethodArgumentNotValidException` (từ `@Valid @RequestBody`).
  - Bắt `ConstraintViolationException` (từ `@Valid @RequestParam`, `@PathVariable`).
  - Trả về danh sách lỗi chi tiết với status `400 Bad Request`.
- ✅ **Xử lý business logic errors**:
  - Bắt custom exception như `UserAlreadyExistsException`, `BookingNotFoundException`.
  - Trả về status code phù hợp (409 Conflict, 404 Not Found...).
- ✅ **Xử lý lỗi hệ thống**:
  - Bắt `Exception` chung (catch-all) để xử lý lỗi không lường trước.
  - Trả về `500 Internal Server Error` với message an toàn (không expose stack trace trên production).
- ❌ Không nên bắt `Exception` chung mà không log:
  - Phải log lỗi để debug, nhưng không trả stack trace cho client.
- ❌ Không nên có nhiều `@ControllerAdvice` class:
  - Nên gom tất cả exception handling vào một class duy nhất để dễ quản lý.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng: Tạo GlobalExceptionHandler**

```java
// DTO để trả về lỗi cho client
public class ErrorResponse {
    public int status;
    public String message;
    public List<String> errors; // Danh sách lỗi chi tiết (cho validation)
    public long timestamp;
    
    public ErrorResponse(int status, String message, List<String> errors) {
        this.status = status;
        this.message = message;
        this.errors = errors;
        this.timestamp = System.currentTimeMillis();
    }
}
```

```java
// Global Exception Handler
@ControllerAdvice
public class GlobalExceptionHandler {
    
    // Xử lý lỗi validation (từ @Valid @RequestBody)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
        MethodArgumentNotValidException ex
    ) {
        // Lấy danh sách lỗi từ BindingResult
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
        
        ErrorResponse response = new ErrorResponse(
            400,
            "Dữ liệu không hợp lệ",
            errors
        );
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    // Xử lý lỗi IllegalArgumentException (từ Service)
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(
        IllegalArgumentException ex
    ) {
        ErrorResponse response = new ErrorResponse(
            400,
            ex.getMessage(),
            null
        );
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    // Xử lý lỗi chung (catch-all)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        // Log lỗi để debug (không trả stack trace cho client)
        System.err.println("Lỗi không lường trước: " + ex.getMessage());
        ex.printStackTrace();
        
        ErrorResponse response = new ErrorResponse(
            500,
            "Đã xảy ra lỗi hệ thống. Vui lòng thử lại sau.",
            null
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

**Ví dụ đơn giản - Cách sai: Try-catch ở mọi Controller**

```java
// ❌ Sai: Mỗi Controller tự try-catch, code lặp lại nhiều lần
@RestController
@RequestMapping("/api/users")
public class UserControllerBad {
    
    @PostMapping("/register")
    public ResponseEntity<?> register(@RequestBody RegisterUserRequest request) {
        try {
            // Logic xử lý...
            return ResponseEntity.ok(...);
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            return ResponseEntity.status(500).body(Map.of("error", "Lỗi hệ thống"));
        }
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<?> getUser(@PathVariable Long id) {
        try {
            // Logic xử lý...
            return ResponseEntity.ok(...);
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            return ResponseEntity.status(500).body(Map.of("error", "Lỗi hệ thống"));
        }
    }
    
    // ... mỗi method đều phải try-catch giống nhau → code lặp lại rất nhiều
}
```

**Tại sao sai:**

- Code lặp lại rất nhiều (mỗi Controller method đều có try-catch giống nhau).
- Response lỗi không nhất quán (có thể trả Map, có thể trả String...).
- Khó bảo trì: muốn đổi format lỗi phải sửa ở rất nhiều chỗ.
- Controller bị "béo", vừa xử lý nghiệp vụ vừa xử lý lỗi.

**Ví dụ trong dự án Taxi - Cách đúng: GlobalExceptionHandler đầy đủ**

```java
// Custom Exception cho Taxi Booking System
public class UserAlreadyExistsException extends RuntimeException {
    public UserAlreadyExistsException(String message) {
        super(message);
    }
}

public class BookingNotFoundException extends RuntimeException {
    public BookingNotFoundException(String message) {
        super(message);
    }
}
```

```java
// Global Exception Handler cho Taxi Booking System
@ControllerAdvice
public class GlobalExceptionHandler {
    
    // Xử lý lỗi validation
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
        MethodArgumentNotValidException ex
    ) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
        
        ErrorResponse response = new ErrorResponse(
            400,
            "Dữ liệu không hợp lệ",
            errors
        );
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    // Xử lý lỗi email trùng (custom exception)
    @ExceptionHandler(UserAlreadyExistsException.class)
    public ResponseEntity<ErrorResponse> handleUserAlreadyExistsException(
        UserAlreadyExistsException ex
    ) {
        ErrorResponse response = new ErrorResponse(
            409, // Conflict
            ex.getMessage(),
            null
        );
        
        return ResponseEntity.status(HttpStatus.CONFLICT).body(response);
    }
    
    // Xử lý lỗi không tìm thấy booking
    @ExceptionHandler(BookingNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleBookingNotFoundException(
        BookingNotFoundException ex
    ) {
        ErrorResponse response = new ErrorResponse(
            404,
            ex.getMessage(),
            null
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }
    
    // Xử lý lỗi chung
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        // Log lỗi (dùng Logger thay vì System.out trong production)
        System.err.println("Lỗi không lường trước: " + ex.getMessage());
        ex.printStackTrace();
        
        ErrorResponse response = new ErrorResponse(
            500,
            "Đã xảy ra lỗi hệ thống. Vui lòng thử lại sau.",
            null
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

```java
// Service sử dụng custom exception
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public UserResponseDTO registerNewUser(RegisterUserRequest request) {
        // Kiểm tra email trùng
        if (userRepository.findByEmail(request.email).isPresent()) {
            throw new UserAlreadyExistsException("Email đã được sử dụng: " + request.email);
        }
        
        // Logic đăng ký...
        // ...
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai: Không có GlobalExceptionHandler, trả lỗi không nhất quán**

```java
// ❌ Sai: Controller tự xử lý lỗi, trả response không nhất quán
@RestController
@RequestMapping("/api/users")
public class UserControllerBad {
    
    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody RegisterUserRequest request) {
        try {
            UserResponseDTO response = userService.registerNewUser(request);
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            // ❌ Trả Map
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        } catch (Exception e) {
            // ❌ Trả String
            return ResponseEntity.status(500).body("Lỗi hệ thống");
        }
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<?> getUser(@PathVariable Long id) {
        try {
            UserResponseDTO response = userService.getUserById(id);
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            // ❌ Trả format khác với method register
            return ResponseEntity.badRequest().body("Lỗi: " + e.getMessage());
        }
    }
}
```

**Tại sao sai:**

- Response lỗi không nhất quán (có method trả Map, có method trả String).
- Client khó xử lý vì không biết format lỗi sẽ như thế nào.
- Code lặp lại try-catch ở mọi Controller method.
- Khó bảo trì: muốn đổi format lỗi phải sửa ở rất nhiều chỗ.

---

### 4. Dependency cho Validation

#### 1. Định Nghĩa

Để sử dụng validation trong Spring Boot, bạn cần thêm **dependency** vào file `pom.xml` (nếu dùng Maven) hoặc `build.gradle` (nếu dùng Gradle). Dependency này cung cấp các annotation validation như `@NotNull`, `@Email`, `@Size`...

Trong Spring Boot 3.x, bạn cần dependency `spring-boot-starter-validation` (hoặc `validation-api` + `hibernate-validator`).

Bạn có thể hình dung dependency giống như **bộ công cụ**:
- Không có dependency → không có annotation validation → không thể validate.
- Có dependency → có đầy đủ annotation → có thể validate dữ liệu.

#### 2. Cách Thức Hoạt Động

1. **Thêm dependency vào `pom.xml`**:
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-validation</artifactId>
   </dependency>
   ```
   - Maven/Gradle tự động tải về các thư viện validation.
2. **Spring Boot tự động cấu hình**:
   - Khi khởi động, Spring Boot phát hiện dependency validation.
   - Tự động kích hoạt validation engine.
   - Các annotation `@Valid`, `@Validated` sẽ hoạt động.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ **Khi dùng Spring Boot 3.x**: Thêm `spring-boot-starter-validation`.
- ✅ **Khi dùng Spring Boot 2.x**: Có thể cần thêm `validation-api` và `hibernate-validator` riêng.
- ❌ Không cần thêm dependency nếu chỉ dùng `@NotNull`, `@Email` cơ bản trong Spring Boot 3.x (đã có sẵn trong `spring-boot-starter-web`).

#### 4. Ví Dụ Minh Họa

**Ví dụ đúng: Thêm dependency vào pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <dependencies>
        <!-- Spring Boot Web (đã có sẵn validation cơ bản) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Validation dependency (khuyến nghị thêm để đầy đủ tính năng) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Các dependency khác... -->
    </dependencies>
</project>
```

**Ví dụ sai: Quên thêm dependency, validation không hoạt động**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <dependencies>
        <!-- ❌ Thiếu spring-boot-starter-validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Validation annotation có thể compile được, nhưng không hoạt động khi chạy -->
    </dependencies>
</project>
```

**Tại sao sai:**

- Code có thể compile được (annotation `@NotNull`, `@Email` có thể import được).
- Nhưng khi chạy, `@Valid` không kích hoạt validation → request vẫn đi qua dù dữ liệu sai.
- Phải thêm `spring-boot-starter-validation` để validation engine hoạt động.

---

## Thực hành

### Bài tập: Thêm Validation cho RegisterUserRequest

**Mục tiêu:**  
Biết cách sử dụng các annotation validation để kiểm tra dữ liệu đầu vào.

**Yêu cầu:**

1. Thêm dependency `spring-boot-starter-validation` vào `pom.xml` (nếu chưa có).
2. Cập nhật `RegisterUserRequest` với các validation:
   - `fullName`: `@NotBlank`, `@Size(min = 2, max = 100)`.
   - `email`: `@NotBlank`, `@Email`.
   - `password`: `@NotBlank`, `@Size(min = 8, max = 50)`.
   - `phone`: `@NotBlank`, `@Pattern(regexp = "^[0-9]{10}$")`.
   - `role`: `@NotBlank`, `@Pattern(regexp = "^(ROLE_PASSENGER|ROLE_DRIVER)$")`.
3. Cập nhật `UserController.register`:
   - Thêm `@Valid` trước `@RequestBody`.
4. Test bằng Thunder Client:
   - Gửi request thiếu field → kiểm tra response lỗi.
   - Gửi request email sai định dạng → kiểm tra response lỗi.
   - Gửi request password quá ngắn → kiểm tra response lỗi.

**Kết quả mong đợi:**

- Khi gửi request không hợp lệ, nhận được response lỗi từ Spring Boot (có thể chưa đẹp, sẽ cải thiện ở bài tập tiếp theo).
- Khi gửi request hợp lệ, API vẫn hoạt động bình thường.

**Gợi ý:**

- Đọc log trong VS Code Terminal để xem thông báo lỗi validation chi tiết.
- Thử gửi request với các trường hợp sai khác nhau để hiểu rõ cách validation hoạt động.

---

### Bài tập: Tạo GlobalExceptionHandler

**Mục tiêu:**  
Xây dựng exception handler tập trung để trả về JSON lỗi đẹp, nhất quán.

**Yêu cầu:**

1. Tạo class `ErrorResponse`:
   - Field: `status` (int), `message` (String), `errors` (List<String>), `timestamp` (long).
2. Tạo class `GlobalExceptionHandler` với `@ControllerAdvice`:
   - Method xử lý `MethodArgumentNotValidException`:
     - Trích xuất danh sách lỗi từ `BindingResult`.
     - Trả về `400 Bad Request` + `ErrorResponse`.
   - Method xử lý `IllegalArgumentException`:
     - Trả về `400 Bad Request` + `ErrorResponse`.
   - Method xử lý `Exception` chung (catch-all):
     - Log lỗi (dùng `System.err.println` tạm thời).
     - Trả về `500 Internal Server Error` + `ErrorResponse`.
3. Test bằng Thunder Client:
   - Gửi request validation fail → kiểm tra JSON lỗi đẹp.
   - Gửi request gây `IllegalArgumentException` (ví dụ: email trùng) → kiểm tra JSON lỗi đẹp.

**Kết quả mong đợi:**

- Tất cả lỗi đều trả về cùng format JSON (`ErrorResponse`).
- Response lỗi dễ đọc, có message và danh sách lỗi chi tiết (cho validation).

**Gợi ý:**

- Dùng `ex.getBindingResult().getFieldErrors()` để lấy danh sách lỗi validation.
- Format message lỗi: `"fieldName: errorMessage"` để client dễ hiểu.

---

### Bài tập: Tạo Custom Exception cho Taxi Booking

**Mục tiêu:**  
Tạo custom exception để xử lý lỗi nghiệp vụ rõ ràng hơn.

**Yêu cầu:**

1. Tạo custom exception:
   - `UserAlreadyExistsException`: Khi email đã tồn tại.
   - `UserNotFoundException`: Khi không tìm thấy user.
2. Cập nhật `UserService`:
   - Thay `IllegalArgumentException` bằng `UserAlreadyExistsException` khi email trùng.
   - Thay `IllegalArgumentException` bằng `UserNotFoundException` khi không tìm thấy user.
3. Cập nhật `GlobalExceptionHandler`:
   - Thêm method xử lý `UserAlreadyExistsException` → trả `409 Conflict`.
   - Thêm method xử lý `UserNotFoundException` → trả `404 Not Found`.
4. Test bằng Thunder Client:
   - Đăng ký user với email đã tồn tại → kiểm tra `409 Conflict`.
   - Lấy user không tồn tại → kiểm tra `404 Not Found`.

**Kết quả mong đợi:**

- Mỗi loại lỗi nghiệp vụ có status code phù hợp (409, 404...).
- Response lỗi có message rõ ràng, dễ hiểu.

**Gợi ý:**

- Custom exception nên extends `RuntimeException` để không bắt buộc khai báo `throws`.
- Status code:
  - `409 Conflict`: Khi resource đã tồn tại (email trùng).
  - `404 Not Found`: Khi resource không tồn tại (user không tìm thấy).
  - `400 Bad Request`: Khi dữ liệu không hợp lệ (validation fail).

---

## Dự án Taxi

### Bài tập: Hoàn thiện Validation và Exception Handling cho Taxi Booking System

**Mục tiêu:**  
Áp dụng validation và exception handling vào toàn bộ module User trong Taxi Booking System.

**Yêu cầu chi tiết:**

1. **Validation cho RegisterUserRequest**:
   - `fullName`: `@NotBlank`, `@Size(min = 2, max = 100)`.
   - `email`: `@NotBlank`, `@Email`.
   - `password`: `@NotBlank`, `@Size(min = 8, max = 50)`, `@Pattern` (có ít nhất 1 chữ hoa, 1 chữ số).
   - `phone`: `@NotBlank`, `@Pattern(regexp = "^[0-9]{10}$")`.
   - `role`: `@NotBlank`, `@Pattern(regexp = "^(ROLE_PASSENGER|ROLE_DRIVER)$")`.

2. **Custom Exception**:
   - `UserAlreadyExistsException`: Khi email trùng.
   - `UserNotFoundException`: Khi không tìm thấy user.

3. **GlobalExceptionHandler**:
   - Xử lý `MethodArgumentNotValidException` → `400 Bad Request`.
   - Xử lý `UserAlreadyExistsException` → `409 Conflict`.
   - Xử lý `UserNotFoundException` → `404 Not Found`.
   - Xử lý `Exception` chung → `500 Internal Server Error`.

4. **Cập nhật UserService**:
   - Dùng `UserAlreadyExistsException` thay vì `IllegalArgumentException` khi email trùng.
   - Dùng `UserNotFoundException` thay vì `IllegalArgumentException` khi không tìm thấy user.

5. **Test toàn diện**:
   - Test validation: Gửi request thiếu field, sai định dạng → kiểm tra JSON lỗi đẹp.
   - Test custom exception: Đăng ký email trùng → kiểm tra `409 Conflict`.
   - Test custom exception: Lấy user không tồn tại → kiểm tra `404 Not Found`.

**Kết quả mong đợi:**

- Tất cả API User đều có validation đầy đủ.
- Tất cả lỗi đều trả về JSON nhất quán (`ErrorResponse`).
- Status code phù hợp với từng loại lỗi (400, 404, 409, 500).
- Code sạch sẽ, dễ bảo trì, không có try-catch rải rác trong Controller.

**Gợi ý:**

- Ghi chú lại các rule validation để dùng cho các module khác (Booking, Feedback) sau này.
- Đặt breakpoint trong `GlobalExceptionHandler` để quan sát cách exception được xử lý.

---

## Tổng kết buổi 8

**Những gì đã học:**

1. Hiểu rõ **Validation là gì** và tại sao cần validate dữ liệu đầu vào.
2. Nắm được các **annotation validation phổ biến**: `@NotNull`, `@NotBlank`, `@Email`, `@Size`, `@Pattern`.
3. Biết cách sử dụng **`@Valid`** để kích hoạt validation trong Controller.
4. Xây dựng **`@ControllerAdvice`** để xử lý exception tập trung, trả về JSON lỗi đẹp, nhất quán.
5. Tạo **custom exception** để xử lý lỗi nghiệp vụ rõ ràng hơn (UserAlreadyExistsException, UserNotFoundException).

**Kiến thức quan trọng:**

- Validation giúp đảm bảo dữ liệu đầu vào đúng định dạng, đầy đủ, hợp lệ trước khi xử lý.
- `@NotBlank` dùng cho String bắt buộc, `@Email` dùng cho email, `@Size` dùng cho độ dài, `@Pattern` dùng cho regex.
- `@ControllerAdvice` + `@ExceptionHandler` giúp xử lý exception tập trung, code sạch sẽ, dễ bảo trì.
- Custom exception giúp phân biệt rõ các loại lỗi nghiệp vụ, trả về status code phù hợp.
- Response lỗi nên nhất quán (dùng `ErrorResponse` DTO) để client dễ xử lý.

**Chuẩn bị cho buổi 9:**

- Hoàn thiện validation và exception handling cho module User.
- Ghi chú lại các rule validation để áp dụng cho module Booking sau này.
- Suy nghĩ về các quan hệ giữa các Entity (User ↔ Booking) để chuẩn bị cho JPA Relationships.

**Kiểm tra lại trước buổi 9:**

- [ ] Đã thêm dependency `spring-boot-starter-validation` vào `pom.xml`.
- [ ] Đã thêm validation đầy đủ cho `RegisterUserRequest` (fullName, email, password, phone, role).
- [ ] Đã tạo `GlobalExceptionHandler` với `@ControllerAdvice` và xử lý được `MethodArgumentNotValidException`.
- [ ] Đã tạo custom exception (`UserAlreadyExistsException`, `UserNotFoundException`) và xử lý trong `GlobalExceptionHandler`.
- [ ] Đã test validation và exception handling bằng Thunder Client, đảm bảo response lỗi đẹp, nhất quán.
- [ ] Đã cập nhật `UserService` để dùng custom exception thay vì `IllegalArgumentException`.

**Bài tập về nhà (tùy chọn):**

- Thêm validation cho các DTO khác trong Taxi Booking System:
  - `CreateBookingRequest`: Validate pickupLocation, dropoffLocation, distanceKm, carType.
  - `UpdateUserRequest`: Validate các field có thể update (fullName, phone).
- Thử nghiệm các annotation validation nâng cao:
  - `@DecimalMin` / `@DecimalMax` cho số thập phân (price, distance).
  - `@Future` / `@Past` cho ngày tháng (nếu có field date trong Booking).
- Tạo thêm custom exception cho Booking:
  - `BookingNotFoundException`: Khi không tìm thấy booking.
  - `BookingAlreadyAcceptedException`: Khi booking đã được driver khác nhận.

