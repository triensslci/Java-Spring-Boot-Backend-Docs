# Buổi 7: DTO Pattern & Object Mapping

## Ôn lại buổi trước

Trước khi bắt đầu chuẩn hóa DTO và mapping object, hãy kiểm tra lại bạn đã hoàn thành các đầu việc chính của **Buổi 6**:

- Đã **tạo Entity `User` và `Booking`** map đúng với các bảng `users`, `bookings` trong MySQL:
  - Có `@Entity`, `@Table`.
  - Có `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)`.
  - Kiểu dữ liệu field khớp với kiểu cột trong DB.
- Đã **tạo `UserRepository`** extends `JpaRepository<User, Long>` và dùng được các method cơ bản:
  - `save`, `findAll`, `findById`, (tùy chọn: `findByEmail`).
- Đã **xây dựng `UserService` và `UserController`** tối thiểu:
  - Endpoint `POST /api/users/register` nhận trực tiếp `User` hoặc DTO đơn giản và lưu vào bảng `users`.
  - API chạy được, lưu user thật vào DB.
- Đã **test API đăng ký bằng Thunder Client/Postman** và kiểm tra dữ liệu trong MySQL:
  - Kiểm tra ít nhất 2 user: 1 Passenger, 1 Driver.
- Đã **thử debug flow** Controller → Service → Repository trong VS Code:
  - Biết đặt breakpoint, step over (`F10`), step into (`F11`) để quan sát dữ liệu đi qua từng tầng.

Nếu bất kỳ mục nào phía trên chưa hoàn thành, hãy quay lại buổi 6 làm cho chắc, vì buổi 7 sẽ **refactor thẳng trên luồng đăng ký User/Driver** đang có.

---

## Kiến thức

Trong buổi này, trọng tâm là **tách bạch Entity ↔ DTO** để:

- Không bao giờ **trả thẳng Entity** (lộ password, dữ liệu thừa, vòng lặp vô hạn do quan hệ).
- Chuẩn hóa flow: **RequestDTO → Entity (Service) → ResponseDTO**.
- Làm quen với **Object Mapping** (tự viết tay trước, sau đó mới tính tới MapStruct/BeanUtils).
- Tập **refactor bằng VS Code**: Rename, Extract Method, Organize Imports.

### 1. DTO là gì? Tại sao không trả Entity trực tiếp?

#### 1. Định Nghĩa

**DTO (Data Transfer Object)** là một class đơn giản dùng để **truyền dữ liệu** giữa các lớp/tầng hoặc giữa Backend và Client.  
DTO **không đại diện cho bảng trong database**, mà đại diện cho **payload của API** (dữ liệu request/response).

Bạn có thể hình dung:

- **Entity** giống như **hồ sơ gốc trong kho lưu trữ** (có rất nhiều thông tin, cả nhạy cảm).
- **DTO** giống như **bản photo đã che bớt thông tin nhạy cảm** để gửi cho khách hàng/đối tác.

#### 2. Cách Thức Hoạt Động

1. **Client gửi RequestDTO**:
   - Ví dụ: `RegisterUserRequest` chứa `fullName`, `email`, `password`, `role`.
2. **Service map RequestDTO → Entity**:
   - Tạo object `User`, gán field từ DTO vào Entity.
   - Thêm business rule nếu cần (mặc định role, chuẩn hóa email, v.v.).
3. **Repository lưu Entity vào DB**:
   - `userRepository.save(user)` → insert/update bảng `users`.
4. **Service map Entity → ResponseDTO**:
   - Tạo object `UserResponseDTO`, chỉ copy **các field an toàn**: `id`, `fullName`, `email`, `role`.
   - Tuyệt đối **không đưa password** hoặc thông tin nội bộ vào ResponseDTO.
5. **Controller trả ResponseDTO cho Client**:
   - `ResponseEntity.status(HttpStatus.CREATED).body(dto);`

Luồng này đảm bảo:

- DB được map bằng **Entity**.
- API giao tiếp với client bằng **DTO**.
- Mỗi bên chỉ thấy đúng lượng dữ liệu cần thiết.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ Khi xây dựng API đăng ký/đăng nhập User trong Taxi Booking:
  - **RequestDTO**: `RegisterUserRequest`, `LoginRequest`.
  - **ResponseDTO**: `UserResponseDTO`, `LoginResponseDTO` (chứa token).
- ✅ Khi trả danh sách Booking lịch sử cho Passenger:
  - ResponseDTO chỉ cần: `id`, `pickupLocation`, `dropoffLocation`, `status`, `totalFare`, `createdAt`.
  - Không cần gửi hết mọi quan hệ phức tạp, tránh payload nặng.
- ✅ Khi cần **ẩn field nhạy cảm**:
  - Password, token, internal id, field dùng cho tính toán nội bộ.
- ❌ Không dùng DTO để map với bảng DB:
  - DTO không cần annotation JPA như `@Entity`, `@Table`, `@Id`.
  - DTO không được dùng trực tiếp trong `JpaRepository`.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng: Tách Entity và DTO**

```java
// Entity map với bảng users trong MySQL
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String fullName;
    private String email;
    private String password; // Mật khẩu lưu trong DB (sẽ mã hóa ở buổi Security)
    private String role;     // ROLE_PASSENGER, ROLE_DRIVER, ROLE_ADMIN

    // getter/setter...
}
```

```java
// DTO nhận dữ liệu đăng ký từ client (Request)
public class RegisterUserRequest {
    public String fullName;
    public String email;
    public String password;
    public String role; // ROLE_PASSENGER hoặc ROLE_DRIVER
}
```

```java
// DTO trả dữ liệu ra ngoài (Response) - KHÔNG có password
public class UserResponseDTO {
    public Long id;
    public String fullName;
    public String email;
    public String role;
}
```

**Ví dụ đơn giản - Cách sai: Trả thẳng Entity**

```java
// ❌ Sai: Controller trả thẳng Entity User cho client
@RestController
@RequestMapping("/api/users")
public class UserControllerBad {

    @Autowired
    private UserRepository userRepository;

    @PostMapping("/register")
    public User register(@RequestBody User user) {
        // Lưu thẳng Entity và trả ngược Entity cho client
        return userRepository.save(user);
    }
}
```

**Tại sao sai:**

- Client sẽ nhận được **password** (dù là plain-text hay đã mã hóa) → rủi ro bảo mật cực lớn.
- Sau này Entity `User` có thêm quan hệ phức tạp (Booking, Feedback), JSON trả ra có thể:
  - Rất nặng, nhiều field client không cần.
  - Bị vòng lặp vô hạn (infinite recursion) nếu map 2 chiều (`User` ↔ `Booking`).
- Code khó thay đổi: chỉ cần đổi Entity là **toàn bộ API** bị ảnh hưởng.

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
// Service chuyển từ RegisterUserRequest -> User -> UserResponseDTO
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponseDTO registerNewUser(RegisterUserRequest request) {
        // Kiểm tra email trùng (logic nghiệp vụ)
        userRepository.findByEmail(request.email).ifPresent(u -> {
            throw new IllegalArgumentException("Email đã được sử dụng");
        });

        // Map RequestDTO -> Entity
        User user = new User();
        user.setFullName(request.fullName);
        user.setEmail(request.email);
        user.setPassword(request.password); // Buổi Security sẽ mã hóa
        user.setRole(request.role);

        User saved = userRepository.save(user);

        // Map Entity -> ResponseDTO
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = saved.getId();
        dto.fullName = saved.getFullName();
        dto.email = saved.getEmail();
        dto.role = saved.getRole();
        return dto;
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ Sai: Vừa dùng Entity cho DB, vừa dùng chính Entity cho response
@RestController
@RequestMapping("/api/users")
public class UserControllerMixed {

    @Autowired
    private UserRepository userRepository;

    @PostMapping("/register")
    public User register(@RequestBody RegisterUserRequest request) {
        User user = new User();
        user.setFullName(request.fullName);
        user.setEmail(request.email);
        user.setPassword(request.password);
        user.setRole(request.role);

        // Trả thẳng Entity ra ngoài
        return userRepository.save(user);
    }
}
```

**Tại sao sai:**

- Mặc dù đã dùng DTO ở request, nhưng response vẫn **lộ password**.
- Sau này khi Entity `User` có thêm rất nhiều field nội bộ (ví dụ `balance`, `isActive`, `vehicleType`), client sẽ thấy hết, khó kiểm soát.

---

### 2. Pattern RequestDTO → Entity → ResponseDTO

#### 1. Định Nghĩa

**RequestDTO → Entity → ResponseDTO** là pattern chuẩn để xử lý dữ liệu trong API:

- **RequestDTO**: Dữ liệu **đi vào hệ thống** (client → server).
- **Entity**: Dữ liệu **lưu trong database** (server ↔ database).
- **ResponseDTO**: Dữ liệu **trả ra cho client** (server → client).

Trong Taxi Booking System:

- Passenger gửi `RegisterUserRequest` → Service convert thành `User` → lưu DB → trả về `UserResponseDTO`.
- Khi xem lịch sử chuyến đi:
  - DB trả `List<Booking>` → Service convert từng `Booking` thành `BookingHistoryItemDTO`.

#### 2. Cách Thức Hoạt Động

1. **Controller**:
   - Nhận JSON từ client và map vào **RequestDTO** qua `@RequestBody`.
   - Gọi Service với DTO này.
2. **Service**:
   - Validate nghiệp vụ (email trùng, role hợp lệ…).
   - Map **RequestDTO → Entity**:
     - Tạo Entity mới (`User`, `Booking`…).
     - Copy các field bắt buộc từ DTO sang Entity.
   - Gọi Repository để lưu/truy vấn DB.
   - Map **Entity → ResponseDTO**:
     - Tạo DTO để trả ra, chỉ copy field cần hiển thị.
3. **Controller**:
   - Nhận ResponseDTO từ Service và trả về client qua `ResponseEntity`.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ API đăng ký User/Driver:
  - `RegisterUserRequest` → `User` → `UserResponseDTO`.
- ✅ API tạo Booking:
  - `CreateBookingRequest` (pickup, dropoff, carType) → `Booking` → `BookingDetailDTO`.
- ✅ API xem lịch sử Booking:
  - `List<Booking>` → `List<BookingHistoryItemDTO>`.
- ✅ API login:
  - `LoginRequest` (email, password) → Entity `User` để check → `LoginResponseDTO` (token, user info).
- ❌ Không bypass pattern để:
  - Nhận trực tiếp Entity trong `@RequestBody`.
  - Trả trực tiếp Entity trong `ResponseBody`.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng: Flow đăng ký User dùng DTO**

```java
// RequestDTO
public class RegisterUserRequest {
    public String fullName;
    public String email;
    public String password;
    public String role;
}
```

```java
// ResponseDTO
public class UserResponseDTO {
    public Long id;
    public String fullName;
    public String email;
    public String role;
}
```

```java
// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/register")
    public ResponseEntity<UserResponseDTO> register(@RequestBody RegisterUserRequest request) {
        UserResponseDTO response = userService.registerNewUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

**Ví dụ đơn giản - Cách sai: Trộn RequestDTO và ResponseDTO**

```java
// ❌ Sai: Dùng chung một DTO cho cả request và response
public class UserDTO {
    public Long id;
    public String fullName;
    public String email;
    public String password; // vừa dùng để nhận request, vừa trả response
}
```

```java
// ❌ Sai: Controller nhận/trả cùng một DTO UserDTO
@RestController
@RequestMapping("/api/users")
public class UserControllerBad2 {

    @PostMapping("/register")
    public UserDTO register(@RequestBody UserDTO dto) {
        // Lưu user (bỏ qua cho ngắn)
        // ...
        return dto; // Trả ngược lại cả password cho client
    }
}
```

**Tại sao sai:**

- DTO duy nhất vừa dùng cho request vừa cho response nên:
  - **Password** phải có trong DTO để gửi lên → khi trả về cũng có password.
  - Rất khó kiểm soát field tác động tới client.
- Về lâu dài, yêu cầu giữa request và response thường khác nhau:
  - Request có password, confirmPassword, captcha…
  - Response thì không cần các trường đó.

**Ví dụ trong dự án Taxi - Cách đúng: Booking history**

```java
// DTO cho từng item lịch sử chuyến đi của Passenger
public class BookingHistoryItemDTO {
    public Long id;
    public String pickupLocation;
    public String dropoffLocation;
    public String status;
    public Double distanceKm;
    public Double totalFare;
}
```

```java
// Service convert từ List<Booking> -> List<BookingHistoryItemDTO>
@Service
public class BookingService {

    private final BookingRepository bookingRepository;

    public BookingService(BookingRepository bookingRepository) {
        this.bookingRepository = bookingRepository;
    }

    public List<BookingHistoryItemDTO> getHistoryForPassenger(Long passengerId) {
        List<Booking> bookings = bookingRepository.findByPassengerId(passengerId);
        List<BookingHistoryItemDTO> result = new ArrayList<>();

        for (Booking booking : bookings) {
            BookingHistoryItemDTO dto = new BookingHistoryItemDTO();
            dto.id = booking.getId();
            dto.pickupLocation = booking.getPickupLocation();
            dto.dropoffLocation = booking.getDropoffLocation();
            dto.status = booking.getStatus();
            dto.distanceKm = booking.getDistanceKm();
            dto.totalFare = booking.getTotalPrice();
            result.add(dto);
        }
        return result;
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ Sai: Trả thẳng List<Booking> ra ngoài
@RestController
@RequestMapping("/api/bookings")
public class BookingControllerBad {

    private final BookingRepository bookingRepository;

    public BookingControllerBad(BookingRepository bookingRepository) {
        this.bookingRepository = bookingRepository;
    }

    @GetMapping("/history/{passengerId}")
    public List<Booking> getHistory(@PathVariable Long passengerId) {
        return bookingRepository.findByPassengerId(passengerId);
    }
}
```

**Tại sao sai:**

- Trả toàn bộ Entity `Booking` (sau này có thể chứa quan hệ phức tạp, field nội bộ).
- Không kiểm soát được payload trả về, dễ gây quá tải băng thông và lộ cấu trúc hệ thống.

---

### 3. Object Mapping: Tự viết tay vs dùng MapStruct/BeanUtils

#### 1. Định Nghĩa

**Object Mapping** là quá trình **chuyển dữ liệu từ một object sang object khác**:

- Từ `RegisterUserRequest` sang `User`.
- Từ `User` sang `UserResponseDTO`.
- Từ `Booking` sang `BookingHistoryItemDTO`.

Có 2 cách chính:

- **Mapping thủ công (viết tay)**: Tự set từng field một.
- **Mapping bằng thư viện**: Dùng MapStruct, ModelMapper, BeanUtils, v.v.

Trong giai đoạn học này, bạn nên **làm quen với mapping thủ công** để hiểu rõ luồng dữ liệu, sau đó mới cân nhắc thư viện.

#### 2. Cách Thức Hoạt Động

1. **Mapping thủ công**:
   - Viết code gán từng field:
     - `user.setEmail(request.email);`
     - `dto.email = user.getEmail();`
   - Dễ đọc, rõ ràng, không phụ thuộc framework.
   - Nhược điểm: Gõ hơi nhiều nhưng **rất tốt để luyện tay**.
2. **Mapping bằng thư viện (ví dụ MapStruct/BeanUtils)**:
   - Bạn định nghĩa cấu hình mapping (annotation, config).
   - Thư viện sinh code mapping hoặc dùng reflection để copy field.
   - Tiết kiệm thời gian khi project lớn, nhiều DTO lặp lại.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ **Mapping thủ công**:
  - Dự án nhỏ, số field ít.
  - Giai đoạn học cơ bản (như khoá này).
  - Cần kiểm soát chặt từng bước transform dữ liệu.
- ✅ **MapStruct / BeanUtils**:
  - Dự án lớn, rất nhiều DTO và Entity.
  - Team muốn giảm code lặp, tập trung vào logic.
- ❌ Không nên vội dùng thư viện mapping khi:
  - Chưa hiểu rõ DTO vs Entity.
  - Mới làm quen với Spring Boot và Taxi Booking System.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng: Mapping thủ công rõ ràng**

```java
// Hàm helper map từ RegisterUserRequest -> User
public class UserMapper {

    // Map RequestDTO -> Entity
    public static User toEntity(RegisterUserRequest request) {
        User user = new User();
        user.setFullName(request.fullName);
        user.setEmail(request.email);
        user.setPassword(request.password);
        user.setRole(request.role);
        return user;
    }

    // Map Entity -> ResponseDTO
    public static UserResponseDTO toResponseDTO(User user) {
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = user.getId();
        dto.fullName = user.getFullName();
        dto.email = user.getEmail();
        dto.role = user.getRole();
        return dto;
    }
}
```

**Ví dụ đơn giản - Cách sai: Dùng reflection bừa bãi**

```java
// ❌ Sai: Tự nghĩ ra "mapper" generic dùng reflection, khó debug
public class GenericMapper {

    public static <T> T map(Object source, Class<T> targetClass) {
        try {
            T target = targetClass.getDeclaredConstructor().newInstance();
            // Dùng reflection copy tất cả field trùng tên (code bỏ qua cho ngắn)
            return target;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

**Tại sao sai (cho người mới):**

- Reflection phức tạp, khó debug, hiệu năng kém hơn mapping tay.
- Người mới khó hiểu dữ liệu đang đi như thế nào.
- Dễ copy cả những field **không nên** copy (như password, field nội bộ).

**Ví dụ trong dự án Taxi - Cách đúng: tách mapper riêng**

```java
// BookingMapper cho Taxi Booking
public class BookingMapper {

    public static Booking toEntity(CreateBookingRequest request, Long passengerId) {
        Booking booking = new Booking();
        booking.setPassengerId(passengerId);
        booking.setPickupLocation(request.pickupLocation);
        booking.setDropoffLocation(request.dropoffLocation);
        booking.setStatus("PENDING");
        booking.setDistanceKm(request.distanceKm);
        booking.setTotalPrice(request.estimatedPrice);
        return booking;
    }

    public static BookingHistoryItemDTO toHistoryItemDTO(Booking booking) {
        BookingHistoryItemDTO dto = new BookingHistoryItemDTO();
        dto.id = booking.getId();
        dto.pickupLocation = booking.getPickupLocation();
        dto.dropoffLocation = booking.getDropoffLocation();
        dto.status = booking.getStatus();
        dto.distanceKm = booking.getDistanceKm();
        dto.totalFare = booking.getTotalPrice();
        return dto;
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai: Map trực tiếp trong Controller**

```java
// ❌ Sai: Controller quá béo, vừa nhận request vừa tự map, vừa gọi repository
@RestController
@RequestMapping("/api/bookings")
public class BookingControllerFat {

    @Autowired
    private BookingRepository bookingRepository;

    @PostMapping
    public BookingHistoryItemDTO create(@RequestBody CreateBookingRequest request) {
        Booking booking = new Booking();
        booking.setPickupLocation(request.pickupLocation);
        booking.setDropoffLocation(request.dropoffLocation);
        // ... gán thêm vài field ...

        Booking saved = bookingRepository.save(booking);

        BookingHistoryItemDTO dto = new BookingHistoryItemDTO();
        dto.id = saved.getId();
        dto.pickupLocation = saved.getPickupLocation();
        // ...
        return dto;
    }
}
```

**Tại sao sai:**

- Controller **ôm hết mọi việc**: mapping, logic, thao tác DB.
- Sau này muốn reuse mapping/logic ở chỗ khác rất khó.
- Trái với cấu trúc 3 tầng đã học buổi 6 (Controller → Service → Repository).

---

### 4. VS Code Refactoring với DTO & Mapping

#### 1. Định Nghĩa

Refactoring là việc **cải thiện cấu trúc code mà không thay đổi hành vi**.  
Trong VS Code, bạn có sẵn rất nhiều công cụ để refactor khi chuyển từ Entity thuần sang DTO pattern:

- **Rename Symbol (`F2`)**.
- **Extract Method**.
- **Extract Variable**.
- **Organize Imports**.

#### 2. Cách Thức Hoạt Động

1. **Rename Symbol (`F2`)**:
   - Đặt con trỏ vào tên class/method/field.
   - Nhấn `F2` → nhập tên mới → Enter.
   - VS Code rename tất cả chỗ liên quan trong project (có preview).
2. **Extract Method**:
   - Bôi đen khối code mapping lặp lại.
   - Chuột phải → `Refactor...` → `Extract to method`.
   - VS Code tạo method mới (ví dụ `mapToUserResponseDTO`) và thay thế đoạn code cũ.
3. **Organize Imports (`Shift+Alt+O`)**:
   - Tự động xóa import thừa, sắp xếp lại import.
4. **Debug sau refactor**:
   - Đặt breakpoint vào method mapping mới.
   - Chạy debug để đảm bảo kết quả mapping không đổi.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ Khi chuyển từ code cũ (trả Entity) sang code mới (trả DTO):
  - Dùng Extract Method để tách phần mapping thành hàm riêng trong Service/Mapper.
- ✅ Khi đổi tên DTO từ `UserDTO` thành `UserResponseDTO`:
  - Dùng `F2` để đổi nhất quán toàn project, tránh sót.
- ✅ Khi vừa thêm MapStruct/BeanUtils vào project:
  - Dùng refactor để dần dần thay thế mapping tay bằng mapper tự động.

#### 4. Ví Dụ Minh Họa

**Ví dụ đúng: Extract mapping ra hàm riêng trong Service**

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponseDTO registerNewUser(RegisterUserRequest request) {
        // ... check email trùng ...

        User user = new User();
        user.setFullName(request.fullName);
        user.setEmail(request.email);
        user.setPassword(request.password);
        user.setRole(request.role);

        User saved = userRepository.save(user);
        return toUserResponseDTO(saved); // Đoạn này được Extract Method
    }

    // Method riêng để map Entity -> DTO
    private UserResponseDTO toUserResponseDTO(User saved) {
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = saved.getId();
        dto.fullName = saved.getFullName();
        dto.email = saved.getEmail();
        dto.role = saved.getRole();
        return dto;
    }
}
```

**Ví dụ sai: Không refactor, copy-paste mapping khắp nơi**

```java
// ❌ Sai: Copy-paste mapping ở nhiều chỗ khác nhau
public class SomeService {

    public UserResponseDTO doSomething(User user) {
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = user.getId();
        dto.fullName = user.getFullName();
        dto.email = user.getEmail();
        dto.role = user.getRole();
        return dto;
    }
}

public class AnotherService {

    public UserResponseDTO anotherThing(User user) {
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = user.getId();
        dto.fullName = user.getFullName();
        dto.email = user.getEmail();
        dto.role = user.getRole();
        return dto;
    }
}
```

**Tại sao sai:**

- Có 2 khối code mapping **giống hệt nhau**, khó bảo trì.
- Nếu sau này đổi thêm field trong `UserResponseDTO`, bạn phải sửa ở **rất nhiều nơi**.
- Nên dùng VS Code `Extract Method` để gom logic mapping lại một chỗ.

**Ví dụ trong dự án Taxi - Cách đúng: sử dụng VS Code Refactor**

- Trong VS Code:
  1. Bôi đen đoạn mapping `User` → `UserResponseDTO`.
  2. Chuột phải → `Refactor...` → `Extract to method in class 'UserService'`.
  3. Đặt tên method: `toUserResponseDTO`.
  4. VS Code tạo sẵn method private, thay chỗ cũ bằng lời gọi method mới.
- Sau đó:
  - Đặt breakpoint trong method `toUserResponseDTO`.
  - Gửi request đăng ký User để kiểm tra mapping vẫn đúng.

**Ví dụ trong dự án Taxi - Cách sai: rename bằng Find & Replace text thuần**

- Bạn dùng `Ctrl+F` → `Replace All` để đổi `UserDTO` thành `UserResponseDTO` trong toàn project.

**Tại sao sai:**

- Dễ phá hỏng những chỗ:
  - Comment, string literal, SQL, tên file… không nên đổi.
- Dùng `F2` (Rename Symbol) an toàn hơn vì VS Code hiểu được ngữ cảnh code Java.

---

## Thực hành

### Bài tập: Refactor API đăng ký User sang DTO chuẩn

**Mục tiêu:**  
Tách Entity `User` khỏi payload API, chuẩn hóa flow `RegisterUserRequest → User → UserResponseDTO`.

**Yêu cầu:**

1. Tạo 2 DTO:
   - `RegisterUserRequest` (request).
   - `UserResponseDTO` (response).
2. Refactor `UserService.registerNewUser`:
   - Nhận `RegisterUserRequest`.
   - Map sang `User` (Entity).
   - Lưu DB, sau đó map `User` sang `UserResponseDTO`.
3. Refactor `UserController`:
   - Endpoint `POST /api/users/register` nhận `RegisterUserRequest`.
   - Trả về `201 CREATED` + body là `UserResponseDTO`.
4. Dùng VS Code:
   - Extract method `toUserResponseDTO` trong `UserService`.
   - Organize imports (`Shift+Alt+O`) sau khi thêm DTO.

**Kết quả mong đợi:**

- API đăng ký vẫn hoạt động đúng.
- Response JSON **không chứa password**.
- Entity `User` chỉ dùng trong tầng Service/Repository, không xuất hiện trong Controller response.

**Gợi ý:**

- Dùng lại code ví dụ trong phần kiến thức, chỉnh cho khớp với project của bạn.
- Thử bật debug và xem giá trị trước/sau khi mapping.

---

### Bài tập: Tạo DTO cho lịch sử chuyến đi của Passenger

**Mục tiêu:**  
Biết thiết kế DTO cho case thực tế trong Taxi Booking System: xem lịch sử chuyến đi.

**Yêu cầu:**

1. Tạo DTO `BookingHistoryItemDTO` với các field:
   - `id`, `pickupLocation`, `dropoffLocation`, `status`, `distanceKm`, `totalFare`.
2. Trong `BookingService`:
   - Viết method `List<BookingHistoryItemDTO> getHistoryForPassenger(Long passengerId)`.
   - Lấy dữ liệu từ `BookingRepository.findByPassengerId(passengerId)`.
   - Map từng `Booking` sang `BookingHistoryItemDTO`.
3. Trong `BookingController`:
   - Tạo endpoint `GET /api/bookings/history/{passengerId}` trả về `List<BookingHistoryItemDTO>`.

**Kết quả mong đợi:**

- Gọi API trả về danh sách JSON gọn, rõ, không dư thừa thông tin nội bộ.
- Không dùng trực tiếp `List<Booking>` cho response.

**Gợi ý:**

- Có thể tạo `BookingMapper` để tách logic mapping ra riêng, tái sử dụng sau này.

---

### Bài tập: Luyện VS Code Refactoring với DTO

**Mục tiêu:**  
Làm quen với các thao tác refactor trong VS Code để code sạch hơn, ít lặp hơn.

**Yêu cầu:**

1. Mở `UserService` trong VS Code.
2. Bôi đen phần mapping `User` → `UserResponseDTO`:
   - Dùng `Refactor...` → `Extract Method` → đặt tên `toUserResponseDTO`.
3. Dùng `F2` để:
   - Đổi tên DTO từ `UserDTO` cũ (nếu có) thành `UserResponseDTO` mới.
4. Dùng `Shift+Alt+O` để:
   - Dọn dẹp import thừa sau khi refactor.

**Kết quả mong đợi:**

- Code dễ đọc hơn, nhiều method nhỏ, rõ ràng.
- Không còn duplicate mapping giống hệt ở nhiều class.

**Gợi ý:**

- Sau khi refactor, luôn chạy lại test/endpoint tương ứng để đảm bảo hành vi không đổi.

---

## Dự án Taxi

### Bài tập: Chuẩn hóa module User với DTO & Mapping

**Mục tiêu:**  
Hoàn thiện module User trong Taxi Booking System với DTO chuẩn, sẵn sàng cho Validation và Security ở các buổi sau.

**Yêu cầu chi tiết:**

1. **DTO:**
   - `RegisterUserRequest`:
     - `fullName`, `email`, `password`, `role`, `phone` (tuỳ chọn).
   - `UserResponseDTO`:
     - `id`, `fullName`, `email`, `role`, `phone`.
2. **Mapper (tuỳ chọn nhưng khuyến khích):**
   - Class `UserMapper` chứa:
     - `User toEntity(RegisterUserRequest request)`.
     - `UserResponseDTO toResponseDTO(User user)`.
3. **Service:**
   - Cập nhật `UserService.registerNewUser(RegisterUserRequest request)`:
     - Dùng `UserMapper` để map DTO ↔ Entity.
     - Xử lý logic email trùng, set role mặc định nếu không truyền.
4. **Controller:**
   - Đảm bảo `POST /api/users/register`:
     - Nhận `RegisterUserRequest`.
     - Trả về `UserResponseDTO`.
5. **Kiểm thử:**
   - Gửi request đăng ký Passenger/Driver qua Thunder Client.
   - Kiểm tra JSON response không chứa password.
   - Kiểm tra DB để chắc chắn user vẫn được lưu đúng.

**Kết quả mong đợi:**

- Module User sạch sẽ: Entity tách riêng, DTO tách riêng, có mapping rõ ràng.
- Sẵn sàng thêm Validation (`@NotBlank`, `@Email`, `@Size`) ở buổi 8 mà không cần đổi lại cấu trúc lớn.

**Gợi ý:**

- Ghi TODO trong code tại chỗ set password:
  - `// TODO: mã hóa password ở buổi Spring Security`.
- Đặt breakpoint trong mapper để quan sát dữ liệu đi qua.

---

## Tổng kết buổi 7

**Những gì đã học:**

1. Hiểu rõ **DTO là gì** và lý do **không nên trả thẳng Entity** cho client.
2. Nắm được pattern chuẩn **RequestDTO → Entity → ResponseDTO** trong API.
3. Biết cách **mapping object** (tự viết tay) giữa DTO và Entity trong Taxi Booking System.
4. Thực hành **refactor với VS Code** (Rename, Extract Method, Organize Imports) để code sạch hơn.

**Kiến thức quan trọng:**

- DTO dùng cho **payload API**, Entity dùng cho **database**, không trộn lẫn.
- Không bao giờ expose field nhạy cảm (như password) ra ngoài ResponseDTO.
- Pattern RequestDTO → Entity → ResponseDTO giúp code dễ bảo trì, dễ mở rộng.
- VS Code hỗ trợ rất tốt việc refactor: `F2`, Extract Method, Organize Imports.

**Chuẩn bị cho buổi 8:**

- Hoàn thiện refactor module User sang DTO đầy đủ.
- Liệt kê các rule dữ liệu cho đăng ký User (email hợp lệ, password tối thiểu 8 ký tự, role hợp lệ…).
- Suy nghĩ về các lỗi thường gặp khi đăng ký (email trùng, format sai) để chuyển thành **Validation + Exception Handling**.

**Kiểm tra lại trước buổi 8:**

- [ ] Đã tách riêng Entity `User` và các DTO (`RegisterUserRequest`, `UserResponseDTO`).
- [ ] Tất cả endpoint User **không trả Entity trực tiếp**, chỉ trả DTO.
- [ ] Module UserService đã sử dụng mapping rõ ràng giữa DTO ↔ Entity.
- [ ] Đã test lại API đăng ký sau khi refactor, đảm bảo vẫn hoạt động.
- [ ] Đã quen tay với các thao tác refactor cơ bản trong VS Code (F2, Extract Method, Organize Imports).

**Bài tập về nhà (tùy chọn):**

- Thiết kế thêm các DTO khác cho Taxi Booking:
  - `CreateBookingRequest`, `BookingDetailDTO`, `DriverProfileDTO`.
- Thử tự viết một `BookingMapper` đầy đủ cho các use case:
  - Tạo booking mới.
  - Xem danh sách booking pending/accepted.


