# Buổi 6: Entity Mapping & Basic CRUD

## Ôn lại buổi trước

Trước khi bắt đầu map Entity và viết CRUD, hãy chắc chắn bạn đã hoàn thành các bước ở **Buổi 5**:

- Đã **vẽ ERD** cho Taxi Booking System với ít nhất các bảng: `users`, `bookings`, (tuỳ chọn `feedback`).
- Đã **tạo database** `taxi_booking_dev` và **user riêng** (ví dụ: `taxi_dev`) với quyền trên schema đó.
- Đã **chạy script SQL** tạo bảng `users`, `bookings` (có khóa ngoại `passenger_id`, `driver_id` trỏ tới `users.id`).
- Đã cấu hình **datasource** trong `application-dev.properties` với:
  - `spring.datasource.url`
  - `spring.datasource.username`
  - `spring.datasource.password`
  - `spring.jpa.hibernate.ddl-auto=validate` hoặc `update` (trong giai đoạn học).
- Đã kiểm tra kết nối bằng VS Code / Navicat / Database Client:
  - Chạy được `SELECT COUNT(*) FROM users;`
  - Ứng dụng Spring Boot **khởi động không lỗi** kết nối DB.

Nếu một trong các mục trên chưa xong, hãy quay lại buổi 5 hoàn thiện trước, vì buổi 6 **phụ thuộc trực tiếp** vào schema đã có sẵn.

---

## Kiến thức

Trong buổi này, bạn sẽ học cách:

- Biến bảng trong database (`users`, `bookings`) thành **Entity Java** với annotation của JPA.
- Sử dụng **`JpaRepository`** để thao tác CRUD mà **không phải viết SQL tay**.
- Liên kết Entity với bảng thật trong MySQL đã tạo từ buổi trước.
- Dùng **VS Code Debug** để theo dõi flow Controller → Service → Repository → Database.

### 1. `@Entity` và `@Table`

#### 1. Định Nghĩa

**`@Entity`** là annotation cho Spring Data JPA biết rằng class Java này đại diện cho **một bảng** trong database.  
**`@Table`** dùng để chỉ rõ **tên bảng** tương ứng trong database (nếu khác tên class).

Bạn có thể tưởng tượng: bảng `users` trong MySQL giống như một **bảng Excel** lưu danh sách tài khoản. Class `User` trong Java là **bản thiết kế** để Spring hiểu mỗi dòng trong bảng `users` sẽ được map thành một đối tượng `User`.

#### 2. Cách Thức Hoạt Động

1. Khi ứng dụng Spring Boot khởi động, Hibernate/JPA sẽ:
   - Quét tất cả các class có annotation `@Entity`.
   - Đọc thông tin annotation `@Table`, `@Id`, `@Column`… để hiểu Entity map với bảng/cột nào.
2. Mỗi khi bạn gọi `userRepository.findAll()`:
   - JPA sinh ra câu `SELECT * FROM users` (hoặc tương đương).
   - Lấy từng dòng kết quả, map từng cột (`email`, `full_name`…) vào field của object `User`.
3. Khi bạn gọi `userRepository.save(user)`:
   - Nếu `user.id` đang `null` → JPA hiểu là **insert** bản ghi mới.
   - Nếu `user.id` đã có → JPA hiểu là **update** bản ghi hiện tại.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ Khi bạn muốn map bảng `users` thành Entity `User` để:
  - Lưu tài khoản Passenger/Driver/Admin.
  - Truy vấn danh sách user, tìm user theo email…
- ✅ Khi bạn muốn map bảng `bookings` thành Entity `Booking` để:
  - Lưu từng chuyến đi.
  - Lọc booking theo `status`, `passenger`, `driver`.
- ✅ Khi bạn có sẵn bảng do script SQL tạo (buổi 5) và muốn **không viết SQL** mà vẫn CRUD được.
- ❌ Không nên đánh dấu class DTO (`UserResponseDTO`, `RegisterRequestDTO`) bằng `@Entity`, vì:
  - DTO chỉ dùng để nhận/trả dữ liệu qua API, **không đại diện cho bảng DB**.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**

```java
// Entity đại diện cho bảng users trong database Taxi Booking
@Entity // Báo với JPA: đây là một Entity
@Table(name = "users") // Map tới bảng users (đã tạo ở buổi 5)
public class User {

    // Khóa chính, map với cột id (BIGINT) trong bảng users
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Dùng AUTO_INCREMENT của MySQL
    private Long id;

    // Map với cột email (unique) trong bảng users
    private String email;

    // Map với cột full_name
    private String fullName;

    // Map với cột password (chưa mã hóa ở buổi này)
    private String password;

    // Map với cột role (ROLE_PASSENGER, ROLE_DRIVER, ROLE_ADMIN)
    private String role;

    // getter/setter bỏ qua cho gọn
}
```

**Ví dụ đơn giản - Cách sai:**

```java
// ❌ Sai: Quên không dùng @Entity, JPA sẽ hoàn toàn bỏ qua class này
public class User {

    @Id // ❌ Dù có @Id nhưng không có @Entity thì vẫn không được JPA quản lý
    private Long id;

    private String email;
}
```

**Tại sao sai:**  
- Không có `@Entity` nên Hibernate không coi `User` là entity → **không tạo mapping**, `JpaRepository<User, Long>` sẽ không hoạt động đúng.  
- Nếu cố gắng tạo `UserRepository extends JpaRepository<User, Long>`, lúc chạy sẽ gặp lỗi vì JPA không thấy `User` trong danh sách entity đã đăng ký.

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
// Entity User dùng chung cho Passenger / Driver / Admin trong Taxi Booking
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Họ tên đầy đủ
    @Column(name = "full_name", nullable = false, length = 100)
    private String fullName;

    @Column(nullable = false, unique = true, length = 120)
    private String email;

    @Column(nullable = false)
    private String password;

    // ROLE_PASSENGER, ROLE_DRIVER, ROLE_ADMIN
    @Column(nullable = false, length = 30)
    private String role;

    // Số điện thoại để tài xế / hành khách liên lạc
    private String phone;

    // getter/setter...
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ Sai: Đặt @Table name không trùng với tên bảng thật trong DB
@Entity
@Table(name = "user") // Trong DB bạn đã tạo bảng tên "users"
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;
}
```

**Tại sao sai:**  
- Database đang có bảng `users` (số nhiều), nhưng bạn lại map vào bảng `user` (số ít) → khi chạy với `ddl-auto=validate`, Hibernate sẽ báo **không tìm thấy bảng `user`**.  
- Nếu để `ddl-auto=update`, Hibernate có thể tự tạo thêm bảng mới tên `user`, làm bạn **bị trùng bảng** và dữ liệu tách ra 2 chỗ rất khó debug.  
- Nguyên tắc: luôn kiểm tra tên bảng trong script SQL buổi 5 và đảm bảo `@Table(name = "tên_bảng")` trùng khớp.

---

### 2. `@Id` và `@GeneratedValue`

#### 1. Định Nghĩa

- **`@Id`**: đánh dấu **field khóa chính** của Entity – tương ứng với cột `PRIMARY KEY` trong bảng.
- **`@GeneratedValue`**: chỉ cho JPA biết **cách sinh giá trị khóa chính** (tự động tăng, sequence…).

Hãy tưởng tượng bảng `bookings` như một danh sách phiếu đặt xe, mỗi phiếu phải có **một số thứ tự duy nhất** (`id`). `@Id` chính là “số thứ tự” đó, còn `@GeneratedValue` quyết định cách hệ thống tạo số tiếp theo.

#### 2. Cách Thức Hoạt Động

1. Khi bạn gọi `bookingRepository.save(newBooking)`:
   - Nếu `newBooking.getId() == null`:
     - JPA sẽ để MySQL tự sinh `AUTO_INCREMENT` (với `GenerationType.IDENTITY`).
     - Sau khi insert, JPA lấy lại giá trị `id` vừa được sinh và gán vào object `newBooking`.
   - Nếu `newBooking.getId()` đã có giá trị:
     - JPA hiểu là update bản ghi có `id` đó.
2. Với kiểu `GenerationType.IDENTITY`:
   - MySQL chịu trách nhiệm tăng dần `id` (1, 2, 3…).
   - Phù hợp cho bảng như `users`, `bookings` trong Taxi Booking.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ Bảng `users`: dùng `Long id` + `GenerationType.IDENTITY`.
- ✅ Bảng `bookings`: dùng `Long id` + `GenerationType.IDENTITY`.
- ✅ Bảng `feedback`: cũng tương tự.
- ❌ Không nên tự set `id` bằng tay kiểu random string nếu không có lý do rõ ràng, vì:
  - Dễ trùng.
  - Khó debug khi join bảng.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**

```java
// Entity Booking đơn giản với id tự tăng
@Entity
@Table(name = "bookings")
public class Booking {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // map với BIGINT AUTO_INCREMENT trong MySQL

    private String pickupLocation;
    private String dropoffLocation;
}
```

**Ví dụ đơn giản - Cách sai:**

```java
@Entity
@Table(name = "bookings")
public class Booking {

    // ❌ Sai: Không đặt @Id, JPA không biết cột nào là khóa chính
    private Long id;

    private String pickupLocation;
}
```

**Tại sao sai:**  
- JPA **bắt buộc** mỗi Entity phải có **1 field khóa chính** được đánh dấu bằng `@Id`.  
- Nếu không, khi khởi động app, Hibernate sẽ báo lỗi “No identifier specified for entity” và ứng dụng không chạy được.

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
// Booking có khóa chính id, map tới bảng bookings đã tạo từ buổi 5
@Entity
@Table(name = "bookings")
public class Booking {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "pickup_location", nullable = false)
    private String pickupLocation;

    @Column(name = "dropoff_location", nullable = false)
    private String dropoffLocation;

    @Column(name = "status", nullable = false)
    private String status; // PENDING, ACCEPTED, COMPLETED, CANCELED

    // Các field khác bỏ qua cho gọn...
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ Sai: Tự set id bằng String không khớp với kiểu BIGINT trong DB
@Entity
@Table(name = "bookings")
public class Booking {

    @Id
    private String id; // Trong DB bạn đã tạo id BIGINT AUTO_INCREMENT

    private String pickupLocation;
}
```

**Tại sao sai:**  
- Kiểu dữ liệu trong Entity (`String`) **không khớp** với kiểu cột trong DB (`BIGINT`).  
- Khi chạy, Hibernate có thể báo lỗi mapping, hoặc SQL lỗi khi cố gắng insert/đọc dữ liệu.  
- Luôn đảm bảo **kiểu dữ liệu Entity và DB trùng nhau** (Long ↔ BIGINT, String ↔ VARCHAR…).

---

### 3. `JpaRepository` và CRUD cơ bản

#### 1. Định Nghĩa

`JpaRepository` là một **interface có sẵn** của Spring Data JPA, giúp bạn có ngay các hàm CRUD (Create, Read, Update, Delete) **mà không cần tự viết SQL hay implementation**.

**CRUD là gì?**  
CRUD là bộ 4 thao tác cơ bản nhất khi làm việc với dữ liệu:
- **Create** (Tạo mới): Ví dụ thêm user mới vào bảng `users` khi passenger đăng ký tài khoản.
- **Read** (Đọc dữ liệu): Ví dụ lấy danh sách tất cả booking đang ở trạng thái `PENDING`.
- **Update** (Cập nhật): Ví dụ đổi trạng thái một booking từ `PENDING` → `ACCEPTED` khi tài xế nhận chuyến.
- **Delete** (Xoá): Ví dụ admin xoá tài khoản driver không còn hoạt động (hoặc đánh dấu là đã xoá – soft delete).

Trong dự án Taxi, hầu hết API bạn viết đều xoay quanh CRUD cho các entity chính: `User`, `Booking`, `Feedback`.

Nó giống như bạn gọi một dịch vụ "tài xế công nghệ": bạn chỉ cần đưa "địa chỉ" (Entity class) và "kiểu khóa chính", Spring Data sẽ lo phần còn lại (tạo code CRUD, sinh SQL).

**Generic Types trong JpaRepository:**
Khi khai báo `JpaRepository<User, Long>`, bạn đang truyền 2 tham số kiểu (generic):
- **Tham số đầu tiên (`User`)**: Entity class mà Repository này sẽ làm việc với. Đây là class Java đã được đánh dấu `@Entity`, đại diện cho bảng `users` trong database.
- **Tham số thứ hai (`Long`)**: Kiểu dữ liệu của khóa chính (Primary Key) của Entity. Vì `User` có field `id` kiểu `Long`, nên bạn phải khai báo `Long` ở đây.

**Tại sao cần 2 tham số này?**
- Spring Data JPA cần biết Entity nào để map với bảng nào trong database.
- Cần biết kiểu khóa chính để các method như `findById(Long id)`, `deleteById(Long id)` có thể hoạt động đúng.

**Các method có sẵn trong JpaRepository:**
Khi bạn extend `JpaRepository<User, Long>`, bạn tự động có sẵn các method sau (không cần viết code):

**Create & Update:**
- `save(T entity)`: Lưu hoặc cập nhật entity. Nếu `entity.id == null` → INSERT, nếu `entity.id` đã có → UPDATE.
- `saveAll(Iterable<T> entities)`: Lưu nhiều entity cùng lúc.

**Read (Đọc dữ liệu):**
- `findById(ID id)`: Tìm entity theo khóa chính, trả về `Optional<T>` (có thể rỗng nếu không tìm thấy).
- `findAll()`: Lấy tất cả entity trong bảng, trả về `List<T>`.
- `existsById(ID id)`: Kiểm tra entity có tồn tại không, trả về `boolean`.
- `count()`: Đếm tổng số entity trong bảng, trả về `long`.

**Delete (Xóa):**
- `deleteById(ID id)`: Xóa entity theo khóa chính.
- `delete(T entity)`: Xóa entity cụ thể.
- `deleteAll()`: Xóa tất cả entity trong bảng (nguy hiểm, cẩn thận khi dùng).

**Tại sao `findById` trả về `Optional<T>`?**
`Optional` là một class đặc biệt trong Java giúp xử lý trường hợp giá trị có thể **null** một cách an toàn. Thay vì trả về `User` (có thể null), Spring Data JPA trả về `Optional<User>` để bạn phải kiểm tra rõ ràng xem có tìm thấy hay không.

**Ví dụ sử dụng Optional:**
```java
// Cách cũ (nguy hiểm, có thể NullPointerException):
User user = userRepository.findById(1L); // user có thể null
String email = user.getEmail(); // ❌ Lỗi nếu user == null

// Cách mới (an toàn với Optional):
Optional<User> userOpt = userRepository.findById(1L);
if (userOpt.isPresent()) {
    User user = userOpt.get();
    String email = user.getEmail(); // ✅ An toàn
} else {
    // Xử lý trường hợp không tìm thấy
}

// Hoặc dùng lambda (ngắn gọn hơn):
userRepository.findById(1L).ifPresent(user -> {
    System.out.println(user.getEmail());
});
```

#### 2. Cách Thức Hoạt Động

1. **Bạn tạo interface:**
   ```java
   public interface UserRepository extends JpaRepository<User, Long> {
       // Có thể thêm method tùy chỉnh ở đây (sẽ học sau)
   }
   ```
   - Chỉ cần khai báo interface, **không cần** viết class implement.
   - Spring Data JPA sẽ tự động tạo implementation cho bạn khi ứng dụng chạy.

2. **Khi ứng dụng Spring Boot khởi động:**
   - Spring Data JPA quét tất cả interface extends `JpaRepository`.
   - Tự động tạo **proxy class** (class giả lập) implement interface đó.
   - Proxy class này chứa code thực thi các method CRUD, tự sinh SQL tương ứng.
   - Đăng ký proxy class này vào **IoC Container** như một Spring Bean.
   - Bạn có thể inject `UserRepository` vào bất kỳ class nào (Service, Controller) bằng `@Autowired` hoặc constructor injection.

3. **Khi bạn inject `UserRepository` vào Service và gọi method:**
   - **`userRepository.save(user)`**: 
     - Nếu `user.getId() == null` → JPA sinh SQL `INSERT INTO users (email, full_name, ...) VALUES (?, ?, ...)`.
     - Nếu `user.getId()` đã có giá trị (ví dụ: 5) → JPA sinh SQL `UPDATE users SET email = ?, full_name = ?, ... WHERE id = 5`.
     - Sau khi INSERT, JPA tự động lấy giá trị `id` vừa được MySQL sinh (AUTO_INCREMENT) và gán vào object `user`.
   - **`userRepository.findById(5L)`**: 
     - JPA sinh SQL `SELECT * FROM users WHERE id = 5`.
     - Nếu tìm thấy → map dữ liệu từ database vào object `User`, bọc trong `Optional<User>`.
     - Nếu không tìm thấy → trả về `Optional.empty()`.
   - **`userRepository.findAll()`**: 
     - JPA sinh SQL `SELECT * FROM users`.
     - Map tất cả dòng kết quả thành `List<User>`.
   - **`userRepository.deleteById(5L)`**: 
     - JPA sinh SQL `DELETE FROM users WHERE id = 5`.
     - Nếu không tìm thấy id → có thể ném exception hoặc không làm gì (tùy cấu hình).
   - **`userRepository.existsById(5L)`**: 
     - JPA sinh SQL `SELECT COUNT(*) FROM users WHERE id = 5`.
     - Trả về `true` nếu COUNT > 0, `false` nếu COUNT = 0.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ Repository cho bảng `users`:
  - CRUD tài khoản Passenger/Driver/Admin.
  - Tìm user theo email, theo role, theo phone...
- ✅ Repository cho bảng `bookings`:
  - Lưu chuyến đi mới.
  - Lấy danh sách booking theo passenger, theo driver, theo status.
  - Đếm số booking, kiểm tra booking có tồn tại không.
- ✅ Dùng trong Service layer để:
  - Tách biệt **logic nghiệp vụ** (Service) khỏi **logic truy vấn DB** (Repository).
  - Service chỉ cần gọi method của Repository, không cần biết SQL là gì.
- ❌ Không nên nhét logic nghiệp vụ phức tạp vào Repository; hãy để Repository chỉ lo **truy cập dữ liệu**, còn nghiệp vụ để Service xử lý.

**Query Method Naming Convention (Quy tắc đặt tên method tự động sinh query):**

Spring Data JPA có khả năng **tự động sinh SQL query** dựa trên tên method bạn đặt trong Repository interface. Bạn chỉ cần đặt tên method theo quy tắc, không cần viết SQL.

**Quy tắc cơ bản:**
- Bắt đầu bằng: `find...`, `get...`, `read...`, `query...`, `count...`, `exists...`
- Tiếp theo là tên Entity (có thể bỏ qua nếu rõ ràng từ context)
- Sau đó là các điều kiện: `By...`, `And...`, `Or...`, `OrderBy...`

**Ví dụ:**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Tìm user theo email
    // SQL: SELECT * FROM users WHERE email = ?
    Optional<User> findByEmail(String email);
    
    // Tìm user theo email và role
    // SQL: SELECT * FROM users WHERE email = ? AND role = ?
    Optional<User> findByEmailAndRole(String email, String role);
    
    // Tìm tất cả user có role là PASSENGER
    // SQL: SELECT * FROM users WHERE role = ?
    List<User> findByRole(String role);
    
    // Đếm số user có role là DRIVER
    // SQL: SELECT COUNT(*) FROM users WHERE role = ?
    long countByRole(String role);
    
    // Kiểm tra email đã tồn tại chưa
    // SQL: SELECT COUNT(*) > 0 FROM users WHERE email = ?
    boolean existsByEmail(String email);
    
    // Tìm user theo fullName, sắp xếp theo email tăng dần
    // SQL: SELECT * FROM users WHERE full_name = ? ORDER BY email ASC
    List<User> findByFullNameOrderByEmailAsc(String fullName);
    
    // Tìm user có email chứa chuỗi (LIKE)
    // SQL: SELECT * FROM users WHERE email LIKE %?%
    List<User> findByEmailContaining(String keyword);
}
```

**Lưu ý quan trọng:**
- Tên method phải **khớp với tên field** trong Entity (ví dụ: `findByEmail` → field `email`, không phải `emailAddress`).
- Nếu field trong Entity là `fullName` (camelCase), trong method bạn có thể dùng `findByFullName` hoặc `findByfullName` (Spring Data JPA tự động xử lý).
- Tham số của method phải **theo đúng thứ tự** với điều kiện trong tên method (ví dụ: `findByEmailAndRole(String email, String role)` - email trước, role sau).

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**

```java
// Repository cho bảng users - chỉ cần khai báo interface
public interface UserRepository extends JpaRepository<User, Long> {

    // Spring Data JPA sẽ tự sinh query: SELECT * FROM users WHERE email = ?
    // Trả về Optional để an toàn khi không tìm thấy
    Optional<User> findByEmail(String email);
}
```

**Cách sử dụng trong Service:**

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    // Ví dụ 1: Tìm user theo ID
    public UserResponseDTO getUserById(Long id) {
        Optional<User> userOpt = userRepository.findById(id);
        
        if (userOpt.isPresent()) {
            User user = userOpt.get();
            // Map sang DTO và trả về
            return mapToDTO(user);
        } else {
            throw new IllegalArgumentException("Không tìm thấy user với ID: " + id);
        }
    }

    // Ví dụ 2: Kiểm tra email đã tồn tại chưa
    public boolean isEmailExists(String email) {
        return userRepository.findByEmail(email).isPresent();
        // Hoặc: return userRepository.existsById(...) nếu có method tương ứng
    }

    // Ví dụ 3: Lấy tất cả user
    public List<UserResponseDTO> getAllUsers() {
        List<User> users = userRepository.findAll();
        return users.stream()
            .map(this::mapToDTO)
            .collect(Collectors.toList());
    }

    // Ví dụ 4: Đếm số lượng user
    public long countUsers() {
        return userRepository.count();
    }

    // Ví dụ 5: Lưu user mới (CREATE)
    public UserResponseDTO createUser(RegisterUserRequest request) {
        User newUser = new User();
        newUser.setEmail(request.email);
        newUser.setFullName(request.fullName);
        newUser.setPassword(request.password);
        newUser.setRole(request.role);
        
        // save() sẽ tự động INSERT vì newUser.getId() == null
        User savedUser = userRepository.save(newUser);
        // Sau khi save, savedUser.getId() đã có giá trị (ví dụ: 1, 2, 3...)
        
        return mapToDTO(savedUser);
    }

    // Ví dụ 6: Cập nhật user (UPDATE)
    public UserResponseDTO updateUser(Long id, UpdateUserRequest request) {
        // Tìm user hiện tại
        Optional<User> userOpt = userRepository.findById(id);
        if (!userOpt.isPresent()) {
            throw new IllegalArgumentException("Không tìm thấy user với ID: " + id);
        }

        User user = userOpt.get();
        // Cập nhật thông tin
        user.setFullName(request.fullName);
        user.setPhone(request.phone);
        
        // save() sẽ tự động UPDATE vì user.getId() đã có giá trị
        User updatedUser = userRepository.save(user);
        
        return mapToDTO(updatedUser);
    }

    // Ví dụ 7: Xóa user (DELETE)
    public void deleteUser(Long id) {
        // Kiểm tra user có tồn tại không
        if (!userRepository.existsById(id)) {
            throw new IllegalArgumentException("Không tìm thấy user với ID: " + id);
        }
        
        userRepository.deleteById(id);
        // Hoặc: userRepository.delete(user) nếu đã có object User
    }

    private UserResponseDTO mapToDTO(User user) {
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = user.getId();
        dto.email = user.getEmail();
        dto.fullName = user.getFullName();
        dto.role = user.getRole();
        return dto;
    }
}
```

**Ví dụ đơn giản - Cách sai:**

```java
// ❌ Sai: Không extends JpaRepository, mất toàn bộ CRUD mặc định
public interface UserRepository {

    // Tự khai báo mà không có implementation
    User save(User user);
    
    // Không có method findAll(), findById(), deleteById()...
}
```

**Tại sao sai:**  
- Interface `UserRepository` không extend `JpaRepository` nên Spring Data JPA **không biết cách tự sinh code**.  
- Bạn sẽ phải tự viết class implement interface này, tự viết SQL → **mất hết lợi thế** của Spring Data.
- Khi inject `UserRepository` vào Service, Spring sẽ báo lỗi vì không tìm thấy bean implementation.

**Ví dụ sai khác - Quên xử lý Optional:**

```java
@Service
public class UserServiceBad {

    @Autowired
    private UserRepository userRepository;

    // ❌ Sai: Không kiểm tra Optional, có thể NullPointerException
    public UserResponseDTO getUserById(Long id) {
        Optional<User> userOpt = userRepository.findById(id);
        User user = userOpt.get(); // ❌ Lỗi nếu userOpt.isEmpty()
        return mapToDTO(user);
    }
}
```

**Tại sao sai:**  
- Nếu không tìm thấy user với `id` đó, `userOpt.get()` sẽ ném `NoSuchElementException`.  
- Luôn phải kiểm tra `isPresent()` hoặc dùng `orElse()`, `orElseThrow()` trước khi gọi `get()`.

**Ví dụ trong dự án Taxi - Cách đúng:**

```java
// Repository thực tế cho Taxi Booking
public interface BookingRepository extends JpaRepository<Booking, Long> {

    // Tìm tất cả booking của một passenger cụ thể
    // Spring Data JPA tự sinh: SELECT * FROM bookings WHERE passenger_id = ?
    List<Booking> findByPassengerId(Long passengerId);

    // Tìm tất cả booking đang ở trạng thái PENDING
    // Spring Data JPA tự sinh: SELECT * FROM bookings WHERE status = ?
    List<Booking> findByStatus(String status);
    
    // Tìm booking theo passenger và status
    // Spring Data JPA tự sinh: SELECT * FROM bookings WHERE passenger_id = ? AND status = ?
    List<Booking> findByPassengerIdAndStatus(Long passengerId, String status);
    
    // Đếm số booking của một passenger
    // Spring Data JPA tự sinh: SELECT COUNT(*) FROM bookings WHERE passenger_id = ?
    long countByPassengerId(Long passengerId);
}
```

**Sử dụng trong BookingService:**

```java
@Service
public class BookingService {

    @Autowired
    private BookingRepository bookingRepository;

    // Lấy tất cả booking đang chờ của một passenger
    public List<BookingResponseDTO> getPendingBookingsForPassenger(Long passengerId) {
        List<Booking> bookings = bookingRepository.findByPassengerIdAndStatus(
            passengerId, 
            "PENDING"
        );
        return bookings.stream()
            .map(this::mapToDTO)
            .collect(Collectors.toList());
    }

    // Đếm số booking đã hoàn thành của một passenger
    public long countCompletedBookings(Long passengerId) {
        return bookingRepository.countByPassengerId(passengerId);
    }

    // Tạo booking mới
    public BookingResponseDTO createBooking(CreateBookingRequest request) {
        Booking newBooking = new Booking();
        newBooking.setPickupLocation(request.pickupLocation);
        newBooking.setDropoffLocation(request.dropoffLocation);
        newBooking.setStatus("PENDING");
        newBooking.setPassengerId(request.passengerId);
        
        // save() tự động INSERT vì newBooking.getId() == null
        Booking savedBooking = bookingRepository.save(newBooking);
        
        return mapToDTO(savedBooking);
    }

    // Cập nhật trạng thái booking (ví dụ: tài xế nhận chuyến)
    public BookingResponseDTO acceptBooking(Long bookingId, Long driverId) {
        Optional<Booking> bookingOpt = bookingRepository.findById(bookingId);
        if (!bookingOpt.isPresent()) {
            throw new IllegalArgumentException("Không tìm thấy booking với ID: " + bookingId);
        }

        Booking booking = bookingOpt.get();
        booking.setStatus("ACCEPTED");
        booking.setDriverId(driverId);
        
        // save() tự động UPDATE vì booking.getId() đã có giá trị
        Booking updatedBooking = bookingRepository.save(booking);
        
        return mapToDTO(updatedBooking);
    }
}
```

**Ví dụ trong dự án Taxi - Cách sai:**

```java
// ❌ Sai: Dùng @Repository trên class trống, không extends JpaRepository
@Repository
public class BookingRepository {

    // Không có code, cũng không extends interface nào
    // Spring không thể tự sinh CRUD
}
```

**Tại sao sai:**  
- Đây chỉ là một class rỗng, Spring không thể tự sinh CRUD như `save`, `findAll`…  
- Bạn sẽ phải tự inject `EntityManager` và viết query tay. Điều này **không phù hợp cho người mới học** và trái với mục tiêu dùng Spring Data JPA để đơn giản hoá CRUD.

**Ví dụ sai khác - Nhầm lẫn giữa INSERT và UPDATE:**

```java
@Service
public class BookingServiceBad {

    @Autowired
    private BookingRepository bookingRepository;

    // ❌ Sai: Tự set id bằng tay, có thể gây lỗi
    public BookingResponseDTO createBooking(CreateBookingRequest request) {
        Booking newBooking = new Booking();
        newBooking.setId(999L); // ❌ Không nên tự set id
        newBooking.setPickupLocation(request.pickupLocation);
        
        // save() sẽ cố UPDATE bản ghi có id = 999, không phải INSERT mới
        // Nếu id = 999 không tồn tại → có thể lỗi hoặc tạo bản ghi không mong muốn
        Booking savedBooking = bookingRepository.save(newBooking);
        
        return mapToDTO(savedBooking);
    }
}
```

**Tại sao sai:**  
- Khi bạn tự set `id` cho entity mới, JPA sẽ hiểu là bạn muốn **UPDATE** bản ghi có `id` đó, không phải INSERT mới.  
- Nếu `id` đó không tồn tại trong database, có thể gây lỗi hoặc tạo dữ liệu không mong muốn.  
- **Nguyên tắc:** Để `id = null` khi tạo entity mới, để JPA tự động INSERT và MySQL tự sinh `id` qua AUTO_INCREMENT.

---

### 4. Luồng Controller → Service → Repository trong CRUD User

#### 1. Định Nghĩa

Trong Spring Boot, một request CRUD điển hình sẽ đi qua 3 tầng:

- **Controller**: Nhận request HTTP (JSON), validate sơ bộ, gọi Service.
- **Service**: Chứa logic nghiệp vụ (vd: kiểm tra email đã tồn tại chưa).
- **Repository**: Thao tác trực tiếp với database qua JPA.

Luồng này giúp code **dễ đọc, dễ test và dễ mở rộng**. Khi debug trong VS Code, bạn có thể đặt breakpoint ở cả 3 tầng để quan sát dữ liệu chạy qua.

#### 2. Cách Thức Hoạt Động

1. Client gửi request `POST /api/users/register` với JSON.
2. `UserController`:
   - Dùng `@RequestBody` map JSON → `RegisterUserRequest`.
   - Gọi `userService.registerNewUser(request)`.
3. `UserService`:
   - Check email đã tồn tại chưa bằng `userRepository.findByEmail(...)`.
   - Nếu hợp lệ, tạo object `User` và gọi `userRepository.save(user)`.
4. `UserRepository` (JpaRepository):
   - Thực hiện `INSERT INTO users ...`.
5. Kết quả trả về `UserResponseDTO` để client không thấy password.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ API đăng ký tài khoản Passenger/Driver.
- ✅ API cập nhật thông tin User.
- ✅ API xoá User (soft delete hoặc hard delete).

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**

```java
// DTO nhận dữ liệu từ client khi đăng ký (request)
public class RegisterUserRequest {
    public String fullName;
    public String email;
    public String password;
    public String role; // ROLE_PASSENGER hoặc ROLE_DRIVER
}

// DTO trả dữ liệu cho client (response) - không trả password
public class UserResponseDTO {
    public Long id;
    public String fullName;
    public String email;
    public String role;
}
```

```java
// Service xử lý nghiệp vụ đăng ký
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponseDTO registerNewUser(RegisterUserRequest request) {
        // 1. Kiểm tra email đã tồn tại chưa
        userRepository.findByEmail(request.email).ifPresent(user -> {
            throw new IllegalArgumentException("Email đã được sử dụng");
        });

        // 2. Tạo entity User từ request
        User user = new User();
        user.setFullName(request.fullName);
        user.setEmail(request.email);
        user.setPassword(request.password); // Buổi sau sẽ mã hóa
        user.setRole(request.role);

        // 3. Lưu vào DB
        User saved = userRepository.save(user);

        // 4. Map sang DTO để trả về
        UserResponseDTO dto = new UserResponseDTO();
        dto.id = saved.getId();
        dto.fullName = saved.getFullName();
        dto.email = saved.getEmail();
        dto.role = saved.getRole();
        return dto;
    }
}
```

```java
// Controller nhận request HTTP và gọi Service
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/register")
    public ResponseEntity<UserResponseDTO> register(@RequestBody RegisterUserRequest request) {
        UserResponseDTO dto = userService.registerNewUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(dto);
    }
}
```

**Ví dụ đơn giản - Cách sai:**

```java
// ❌ Sai: Nhét hết logic vào Controller, không có Service, DTO
@RestController
@RequestMapping("/api/users")
public class UserControllerBad {

    @Autowired
    private UserRepository userRepository;

    @PostMapping("/register")
    public User register(@RequestBody User user) {
        // Không kiểm tra email trùng, không tách DTO
        return userRepository.save(user); // Trả thẳng Entity, lộ password
    }
}
```

**Tại sao sai:**  
- **Controller quá “béo”**, chứa luôn cả logic nghiệp vụ.  
- Nhận và trả thẳng `User` Entity → dễ lộ password, khó thay đổi về sau.  
- Không có check email trùng, không dùng DTO → trái với best practice sẽ học ở buổi 7.

**Ví dụ trong dự án Taxi - Cách đúng (tối giản):**

- Giữ cấu trúc 3 tầng: `UserController` → `UserService` → `UserRepository`.  
- Sử dụng DTO ngay từ buổi này để **chuẩn bị** cho buổi 7 (DTO Pattern).

---

### 5. Debug CRUD trong VS Code

#### 1. Định Nghĩa

Debug là chế độ giúp bạn **chạy code từng bước** để:

- Xem giá trị biến tại từng dòng.
- Xem flow gọi hàm (call stack) từ Controller → Service → Repository.
- Dễ dàng phát hiện lỗi logic trong khi thao tác CRUD với DB.

Trong VS Code, debug Spring Boot rất trực quan: đặt breakpoint, nhấn F5, gửi request là có thể xem dữ liệu đi qua từng tầng.

#### 2. Cách Thức Hoạt Động

1. Mở class `UserController` / `UserService` trong VS Code.
2. Đặt breakpoint:
   - Click vào **vị trí số dòng** bên trái.
   - Hoặc nhấn `F9`.
3. Chạy ứng dụng ở chế độ Debug:
   - Mở **Run and Debug** panel → chọn cấu hình Java/Spring Boot → `Start Debugging`.
   - Hoặc click vào nút Debug bên cạnh phương thức `main`.
4. Gửi request bằng Thunder Client/Postman:
   - Ví dụ: `POST http://localhost:8080/api/users/register`.
5. Khi request đến breakpoint:
   - Ứng dụng tạm dừng.
   - Bạn có thể:
     - Dùng `F10` (Step Over): chạy qua từng dòng trong method hiện tại.
     - Dùng `F11` (Step Into): nhảy vào method con (ví dụ từ Controller vào Service).
     - Xem biến trong panel **Variables**.

#### 3. Trường Hợp Sử Dụng Thực Tế

- ✅ Khi CRUD không hoạt động như mong đợi (không lưu DB, không trả data…).
- ✅ Khi muốn kiểm tra dữ liệu đi qua các tầng (Controller/Service/Repository).
- ✅ Khi học quy trình thực thi Spring Boot, nhất là lần đầu làm việc với JPA.

#### 4. Ví Dụ Minh Họa

**Ví dụ đúng (Quy trình debug):**

1. Đặt breakpoint ở dòng đầu tiên trong `UserService.registerNewUser`.
2. Chạy debug Spring Boot (`F5`).
3. Gửi request đăng ký:
   ```json
   {
     "fullName": "Nguyen Van A",
     "email": "a@example.com",
     "password": "12345678",
     "role": "ROLE_PASSENGER"
   }
   ```
4. Khi dừng ở breakpoint:
   - Kiểm tra giá trị `request.email`.
   - Bước vào (Step Into) lời gọi `userRepository.save(user)` để xem log SQL trong terminal.

**Ví dụ sai (lỗi thường gặp):**

- Đặt breakpoint nhưng **chạy ứng dụng ở chế độ Run bình thường**, không phải Debug → breakpoint **không bao giờ dừng**.  
- Gửi request sai URL hoặc sai port (vd: 8081 thay vì 8080) → tưởng code lỗi nhưng thực ra request **không tới ứng dụng**.

**Tại sao sai:**  
- Debug chỉ hoạt động khi app chạy ở chế độ **Debug**, VS Code tạo kết nối debug với JVM.  
- Nếu gõ sai URL/port, request không đi qua Controller, nên breakpoint sẽ không bao giờ bị kích hoạt.

---

## Thực hành

### Bài tập: Tạo Entity `User` và `Booking` từ ERD

**Mục tiêu:** Biết cách chuyển từ bảng trong MySQL sang Entity trong Java với annotation JPA.

**Yêu cầu:**
1. Dựa trên ERD buổi 5 và script SQL đã tạo, viết 2 Entity:
   - `User` map tới bảng `users`.
   - `Booking` map tới bảng `bookings` (chưa cần quan hệ phức tạp, chỉ cần field cơ bản).
2. Đảm bảo:
   - Có `@Entity`, `@Table`.
   - Có `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)`.
   - Kiểu dữ liệu của field khớp với kiểu cột trong DB.
3. Chạy ứng dụng với `spring.jpa.hibernate.ddl-auto=validate` để kiểm tra mapping đúng.

**Kết quả mong đợi:**
- Ứng dụng khởi động **không báo lỗi** về schema.
- Bảng `users` / `bookings` giữ nguyên, không bị tạo thêm bảng lạ.

**Gợi ý:**
- So sánh tên bảng/cột trong script SQL với annotation `@Table`, `@Column`.
- Nếu gặp lỗi, đọc log Hibernate kỹ, VS Code sẽ highlight dòng stacktrace liên quan.

---

### Bài tập: Tạo `UserRepository` và CRUD đơn giản

**Mục tiêu:** Sử dụng `JpaRepository` để thao tác CRUD User **không cần SQL tay**.

**Yêu cầu:**
1. Tạo interface `UserRepository`:
   - `public interface UserRepository extends JpaRepository<User, Long> { ... }`
   - Thêm method: `Optional<User> findByEmail(String email);`
2. Tạo `UserService` với các method:
   - `UserResponseDTO registerNewUser(RegisterUserRequest request);`
   - `List<UserResponseDTO> getAllUsers();`
   - `UserResponseDTO getUserById(Long id);`
3. Tạo `UserController`:
   - `POST /api/users/register`
   - `GET /api/users`
   - `GET /api/users/{id}`
4. Test bằng Thunder Client:
   - Gửi vài request đăng ký user.
   - Lấy danh sách user.

**Kết quả mong đợi:**
- Dữ liệu user được lưu thật vào bảng `users`.
- Gọi `GET /api/users` trả về đầy đủ danh sách vừa tạo (không có password).

**Gợi ý:**
- Tách DTO ngay từ đầu để sau này dễ thêm Validation, Security.
- Dùng VS Code để **format code** (`Shift+Alt+F`) cho dễ đọc.

---

### Bài tập: Debug flow đăng ký User

**Mục tiêu:** Thành thạo debug flow Controller → Service → Repository bằng VS Code.

**Yêu cầu:**
1. Đặt breakpoint:
   - Trong `UserController.register`.
   - Trong `UserService.registerNewUser`.
2. Chạy app ở chế độ Debug.
3. Gửi request đăng ký user mới.
4. Quan sát:
   - Giá trị `request` trong Controller.
   - Kết quả `userRepository.save(user)` trong Service.
   - Log SQL sinh ra trong terminal.

**Kết quả mong đợi:**
- Hiểu rõ dữ liệu đi qua từng tầng.
- Biết sử dụng `F10`, `F11`, `Shift+F11` để điều khiển debug.

**Gợi ý:**
- Dùng Debug panel trong VS Code để xem **Call Stack** và **Variables**.
- Thử intentionally gửi request thiếu field để xem ứng dụng phản ứng thế nào (sẽ dùng cho Validation buổi 8).

---

## Dự án Taxi

### Bài tập: Hoàn thiện `User` Entity và API đăng ký tài khoản

**Mục tiêu:** Xây dựng API đăng ký tài khoản Passenger/Driver cho Taxi Booking System.

**Yêu cầu chi tiết:**
1. **Entity `User` (map bảng `users`):**
   - Các field tối thiểu:
     - `id` (Long, PK, auto-increment).
     - `fullName` (String, not null).
     - `email` (String, unique, not null).
     - `password` (String, not null).
     - `role` (String, not null) – giá trị: `ROLE_PASSENGER`, `ROLE_DRIVER`, `ROLE_ADMIN`.
     - `phone` (String, optional).
2. **Repository:**
   - `UserRepository extends JpaRepository<User, Long>`.
   - Thêm `Optional<User> findByEmail(String email);`.
3. **Service:**
   - `UserService.registerNewUser(RegisterUserRequest request)`:
     - Nếu email đã tồn tại → ném `IllegalArgumentException` (tạm thời).
     - Nếu chưa tồn tại → tạo `User`, lưu DB, trả về `UserResponseDTO`.
4. **Controller:**
   - `POST /api/users/register`:
     - Nhận `RegisterUserRequest` qua `@RequestBody`.
     - Trả về `201 Created` + `UserResponseDTO`.
5. **Test luồng:** dùng Thunder Client/Postman:
   - Đăng ký 1 Passenger, 1 Driver.
   - Kiểm tra bảng `users` trong MySQL đã có dữ liệu đúng chưa.

**Kết quả mong đợi:**
- API đăng ký hoạt động, lưu được user vào DB.
- Học viên hiểu luồng đầy đủ Controller → Service → Repository → DB.

**Gợi ý:**
- Tạm thời **chưa cần** mã hóa password, sẽ bổ sung ở phần Security (buổi 11).  
- Ghi rõ TODO trong code để nhắc nhở mình mã hoá password sau này.

---

## Tổng kết buổi 6

**Những gì đã học:**
1. Hiểu khái niệm Entity trong JPA và cách dùng `@Entity`, `@Table` để map class Java với bảng trong MySQL.
2. Nắm được vai trò của `@Id`, `@GeneratedValue` và cách cấu hình khóa chính tự tăng.
3. Sử dụng `JpaRepository` để thực hiện CRUD cơ bản mà không cần viết SQL.
4. Thiết kế luồng Controller → Service → Repository trong API đăng ký User.
5. Biết cách sử dụng VS Code Debug để theo dõi flow CRUD và xem dữ liệu đi qua các tầng.

**Kiến thức quan trọng:**
- Entity = đại diện cho **bảng DB**, DTO = đại diện cho **payload API**, không trộn lẫn.
- `@Entity` + `@Table` phải map đúng tên bảng, kiểu dữ liệu field phải khớp kiểu cột.
- `JpaRepository<Entity, IdType>` cung cấp sẵn `save`, `findById`, `findAll`, `deleteById`.
- Tách Controller / Service / Repository giúp code dễ bảo trì, dễ debug.
- VS Code Debug (breakpoint, F10, F11) là công cụ rất mạnh để hiểu sâu cách Spring Boot hoạt động.

**Chuẩn bị cho buổi 7:**
- Tách rõ ràng Entity và DTO trong code đăng ký User.
- Suy nghĩ xem **tại sao không nên trả Entity trực tiếp** cho client (password, quan hệ, vòng lặp…).
- Ghi chú lại các field nào được phép trả về cho client (vd: không trả password, không trả tất cả thông tin nhạy cảm).

**Kiểm tra lại trước buổi 7:**
- [ ] Đã tạo Entity `User` và `Booking` map đúng với bảng trong MySQL.
- [ ] Đã tạo `UserRepository` và dùng được các method cơ bản (`save`, `findAll`, `findById`).
- [ ] Đã xây dựng `UserService` với method đăng ký user mới.
- [ ] Đã xây dựng `UserController` với endpoint `POST /api/users/register` hoạt động.
- [ ] Đã test API đăng ký bằng Thunder Client/Postman và kiểm tra dữ liệu trong bảng `users`.
- [ ] Đã thử debug flow đăng ký trong VS Code ít nhất 1 lần.

**Bài tập về nhà (tùy chọn):**
- Viết thêm các API:
  - `PUT /api/users/{id}` để cập nhật `fullName`, `phone`.
  - `DELETE /api/users/{id}` để xóa user (tuỳ chọn hard/soft delete).
- Thử viết thêm `DriverProfile` Entity (bảng riêng chứa thông tin xe, biển số) liên kết với `User` role DRIVER, để chuẩn bị cho buổi về quan hệ JPA.


