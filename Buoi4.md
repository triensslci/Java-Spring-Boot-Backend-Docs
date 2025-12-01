# Buổi 4: Request Handling & Response Entity

## Ôn lại buổi 3

Để xử lý request phức tạp, hãy đảm bảo bạn đã nắm chắc sản phẩm của Buổi 3:
- Đã hiểu cấu trúc chuẩn của project Spring Boot (controller/service/repository/dto) và đã tạo `BookingController` cơ bản với endpoint `GET /api/welcome`.
- Đã run/debug ứng dụng thành thạo trong VS Code, có Thunder Client/Postman collection sẵn để gọi nhanh và xem log.
- Đã luyện cách đặt tên đường dẫn REST, hiểu `@RestController`, `@RequestMapping` và cách logging đơn giản.

Nếu các phần trên chưa ổn, hãy quay lại kiểm tra API đã build ở Buổi 3 rồi mới tiến sang `@RequestParam`, `@PathVariable`, `@RequestBody` nâng cao hôm nay.

## Kiến thức

### 1. @RequestParam - Nhận dữ liệu từ query string

#### 1. Định Nghĩa
`@RequestParam` cho phép Spring tự động đọc dữ liệu nằm sau dấu `?` trên URL (query string) và gán vào biến method. Hãy tưởng tượng bạn gọi tổng đài taxi và nói: “Tôi cần lọc xe theo quận 1, loại xe 4 chỗ”. Tổng đài viên chính là `@RequestParam`, nghe từng yêu cầu nhỏ rồi ghi chú lại.

#### 2. Cách Thức Hoạt Động
1. Client gửi request dạng `GET /api/drivers?district=1&seat=4`.
2. Spring phân tích URL và thấy hai tham số `district`, `seat`.
3. Các tham số này khớp với biến method có `@RequestParam`.
4. Spring convert kiểu dữ liệu (String, int, boolean...) rồi truyền vào method trước khi code bên trong chạy.

**Những thuộc tính cần nhớ (liên hệ kiến thức naming convention ở Buổi 3):**
- `value` hoặc `name`: Đổi tên param nếu khác tên biến (`@RequestParam("pick_up_time") String pickupTime`).
- `required` (mặc định = `true`): Bắt buộc phải có. Đặt `false` nếu chỉ là tiêu chí optional.
- `defaultValue`: Giá trị mặc định khi client không gửi hoặc gửi chuỗi rỗng.
- Spring tự convert sang `int`, `double`, `boolean`, `Enum`, thậm chí `List<String>` nếu query có nhiều giá trị (`status=NEW&status=COMPLETED`).

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Lọc dữ liệu theo điều kiện tùy chọn (filter booking theo trạng thái, theo ngày).
- ✅ Phân trang: `page`, `size`.
- ✅ Tìm kiếm nhanh: `keyword`, `sortBy`.
- ❌ Không dùng cho dữ liệu phức tạp (JSON): khi đó nên dùng `@RequestBody`.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```java
// API lọc tài xế theo quận và số chỗ
@GetMapping("/drivers/filter")
public String filterDrivers(
        @RequestParam String district,          // quận muốn đón
        @RequestParam(defaultValue = "4") int seat // mặc định xe 4 chỗ
) {
    return "Đang tìm tài xế ở quận " + district + " với xe " + seat + " chỗ";
}
```

**Ví dụ đơn giản - Cách sai:**
```java
// ❌ Quên đánh @RequestParam nên Spring không map được
@GetMapping("/drivers/filter")
public String filterDrivers(String district, int seat) {
    return "Tìm tài xế";
}
```
**Tại sao sai:** Không có `@RequestParam` nên Spring không biết lấy dữ liệu query string và sẽ báo lỗi `400 Bad Request`.

**Ví dụ trong dự án Taxi - Cách đúng:**
```java
// Controller lọc booking theo trạng thái và số trang
@GetMapping("/api/bookings/search")
public ResponseEntity<List<BookingSummaryDTO>> searchBookings(
        @RequestParam String status,            // ví dụ: PENDING / COMPLETED
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "5") int size
) {
    // Gọi service filter theo status + phân trang
    List<BookingSummaryDTO> result = bookingService.findByStatus(status, page, size);
    return ResponseEntity.ok(result);
}
```

**Ví dụ trong dự án Taxi - Cách sai:**
```java
// ❌ Dùng @PathVariable cho dữ liệu filter linh hoạt
@GetMapping("/api/bookings/search/{status}")
public List<Booking> searchBookings(@PathVariable String status) {
    // Không thể nhận thêm page/size linh hoạt từ query string
    return bookingService.findByStatus(status, 0, 5);
}
```
**Tại sao sai:** `@PathVariable` cố định vào URL, khó thêm các filter phụ như `page`, `size`, `dateFrom`. Filter nên linh hoạt → dùng `@RequestParam`.

**Ví dụ mở rộng - Biến không bắt buộc:**
```java
@GetMapping("/api/bookings/report")
public ResponseEntity<String> report(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(required = false) String passengerPhone
) {
    if (passengerPhone == null) {
        return ResponseEntity.ok("Báo cáo toàn hệ thống - trang " + page);
    }
    return ResponseEntity.ok("Báo cáo cho khách " + passengerPhone);
}
```

**Mẹo VS Code & Thunder Client:**
- Dùng tab **Params** trong Thunder Client để nhập `district`, `seat` nhanh, VS Code tự build URL.
- Tạo Environment với biến `{{baseUrl}}` = `http://localhost:8080` để tái sử dụng cho các request khác.
- Khi code, nhấn `Shift+Alt+F` (hoặc `Cmd+Alt+L` nếu dùng plugin) để format giúp tham số thẳng hàng.

**Lỗi phổ biến:**
- Quên `@RequestParam` → Spring ném lỗi `MissingServletRequestParameterException`.
- Sai kiểu dữ liệu (ví dụ `@RequestParam int seat` nhưng client truyền `"bốn"`) → lỗi convert, nên validate hoặc dùng `@RequestParam(required = false) Integer seat`.

---

### 2. @PathVariable - Lấy ID nằm trên đường dẫn

#### 1. Định Nghĩa
`@PathVariable` giúp lấy giá trị ngay trong URL, ví dụ `/api/bookings/15`. Bạn có thể hình dung đường dẫn giống như biển số xe: chỉ cần nhìn biển số là biết đây là xe nào.

#### 2. Cách Thức Hoạt Động
1. Định nghĩa URL `@GetMapping("/bookings/{id}")`.
2. Khi client gọi `/bookings/15`, Spring ánh xạ `15` vào biến `id`.
3. Spring convert kiểu (Long, UUID...) rồi truyền cho method.

**Mẹo nhớ:**
- Có thể đổi tên: `@PathVariable("booking_id") Long bookingId`.
- Hỗ trợ nhiều biến: `/api/users/{userId}/bookings/{bookingId}`.
- Nếu client gửi `/bookings/abc` nhưng biến là `Long` → lỗi 400, nên validate format trước khi gọi Service.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Lấy một phần tử cụ thể dựa trên ID (booking, user, driver).
- ✅ Hành động CRUD có đường dẫn rõ ràng như `/resource/{id}`.
- ❌ Không phù hợp cho filter linh hoạt (nên dùng `@RequestParam`).

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```java
// Lấy thông tin booking bằng id
@GetMapping("/bookings/{id}")
public String getBooking(@PathVariable Long id) {
    return "Chi tiết booking #" + id;
}
```

**Ví dụ đơn giản - Cách sai:**
```java
// ❌ Dùng @RequestParam cho đường dẫn cố định
@GetMapping("/bookings/{id}")
public String getBooking(@RequestParam Long id) {
    return "Booking";
}
```
**Tại sao sai:** URL đã chứa `{id}` nên phải dùng `@PathVariable`. Nếu dùng `@RequestParam`, Spring không tìm thấy tham số query và sẽ báo lỗi 400.

**Ví dụ trong dự án Taxi - Cách đúng:**
```java
// API lấy booking cụ thể
@GetMapping("/api/bookings/{bookingId}")
public ResponseEntity<BookingDetailDTO> getBookingDetail(
        @PathVariable("bookingId") Long id // đặt tên rõ ràng
) {
    BookingDetailDTO detail = bookingService.getDetail(id);
    return ResponseEntity.ok(detail);
}
```

**Ví dụ trong dự án Taxi - Cách sai:**
```java
// ❌ Cố gắng lấy ID từ query string dù URL đã có {bookingId}
@GetMapping("/api/bookings/{bookingId}")
public BookingDetailDTO getBookingDetail(@RequestParam Long bookingId) {
    return bookingService.getDetail(bookingId);
}
```
**Tại sao sai:** Không thống nhất cách truyền tham số, gây lỗi `Required request parameter 'bookingId' for method parameter type Long is not present`.

**Ví dụ nâng cao - nhiều PathVariable:**
```java
// GET /api/users/5/bookings/12/feedback
@GetMapping("/api/users/{userId}/bookings/{bookingId}/feedback")
public ResponseEntity<FeedbackDTO> getFeedback(
        @PathVariable Long userId,
        @PathVariable Long bookingId
) {
    FeedbackDTO dto = feedbackService.findByUserAndBooking(userId, bookingId);
    return ResponseEntity.ok(dto);
}
```

**VS Code pro tip:** Dùng `F2` (Rename Symbol) để đổi tên đồng loạt `bookingId` → tránh quên cập nhật ở `@PathVariable`.

---

### 3. @RequestBody - Nhận JSON từ client

#### 1. Định Nghĩa
`@RequestBody` bảo Spring đọc phần thân (body) của HTTP request, parse JSON → Object Java. Giống như bạn gửi một form đặt xe đầy đủ thông tin; Spring chính là nhân viên đọc form và điền vào phiếu nội bộ.

#### 2. Cách Thức Hoạt Động
1. Client gửi request `POST /api/bookings` kèm JSON (distance, pickupLocation...).
2. Spring dùng Jackson để chuyển JSON thành object (DTO).
3. Object được truyền vào method, bạn xử lý bình thường.
4. Nếu dữ liệu thiếu hoặc sai format → Spring trả về lỗi 400.

**Trước khi test cần chuẩn bị:**
- Header `Content-Type: application/json`.
- JSON hợp lệ (dấu nháy kép, không thừa dấu phẩy).
- DTO phải có constructor mặc định hoặc dùng `record`.
- Nếu dùng Lombok, đảm bảo VS Code bật Lombok Annotations Support để generate getter/setter khi compile.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Tạo mới hoặc cập nhật dữ liệu phức tạp (BookingRequest, UserProfile).
- ✅ Khi payload có nhiều trường hoặc nested object.
- ❌ Không dùng cho filter đơn giản (dùng `@RequestParam`).

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```java
// DTO đơn giản để minh họa
record CalculatorRequest(int a, int b) {}

@PostMapping("/calculator/add")
public int add(@RequestBody CalculatorRequest request) {
    return request.a() + request.b();
}
```

**Ví dụ đơn giản - Cách sai:**
```java
// ❌ Thiếu @RequestBody
@PostMapping("/calculator/add")
public int add(CalculatorRequest request) {
    return request.a() + request.b();
}
```
**Tại sao sai:** Spring nghĩ `CalculatorRequest` là form data → Không parse JSON → request luôn null.

**Ví dụ trong dự án Taxi - Cách đúng:**
```java
// DTO đặt xe
public record BookingRequestDTO(
        String passengerPhone, // số điện thoại khách
        double distanceInKm,   // quãng đường
        String carType         // loại xe: SEDAN/SUV
) {}

@PostMapping("/api/bookings")
public ResponseEntity<BookingResponseDTO> createBooking(
        @RequestBody BookingRequestDTO request
) {
    BookingResponseDTO response = bookingService.createBooking(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

**Ví dụ trong dự án Taxi - Cách sai:**
```java
// ❌ Trộn @RequestParam và @RequestBody không cần thiết
@PostMapping("/api/bookings")
public BookingResponseDTO createBooking(
        @RequestParam double distanceInKm,   // sai: dữ liệu nên nằm trong body JSON
        @RequestBody BookingRequestDTO request
) {
    return bookingService.createBooking(request);
}
```
**Tại sao sai:** Request sẽ vừa cần query string vừa cần JSON → khó dùng cho Mobile/Web. Toàn bộ dữ liệu nên gói trong một JSON duy nhất.

**Sample JSON (dùng cho Thunder Client/Postman):**
```json
{
  "passengerPhone": "0987654321",
  "distanceInKm": 12.5,
  "carType": "SUV"
}
```

**Debug trong VS Code:**
1. Đặt breakpoint ở dòng `bookingService.createBooking(request);`.
2. Nhấn `F5` (Run and Debug) → chọn Java.
3. Gọi API → VS Code dừng ở breakpoint → xem giá trị `request` trong panel VARIABLES.
4. Dùng `Step Into (F11)` để xem logic Service xử lý payload.

**Lỗi phổ biến:**
- Thiếu `@RequestBody` → biến luôn `null`.
- Không có constructor mặc định (khi không dùng record) → Jackson không tạo object.
- Gửi `Content-Type` sai → Jackson không biết parse.

---

### 4. HTTP Methods trong REST API

#### 1. Định Nghĩa
HTTP Method giống như hành động bạn yêu cầu tài xế thực hiện:
- GET: Hỏi thông tin
- POST: Tạo mới
- PUT: Cập nhật toàn bộ
- PATCH: Cập nhật một phần (Buổi sau sẽ học)
- DELETE: Xóa

#### 2. Cách Thức Hoạt Động
1. Client gửi request với method cụ thể (GET/POST/PUT/DELETE).
2. Spring chỉ định method handler tương ứng (@GetMapping, @PostMapping...).
3. Logic xử lý dựa trên ý nghĩa method, giúp API chuẩn REST.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ GET `/api/bookings/{id}`: Xem chi tiết booking.
- ✅ POST `/api/bookings`: Tạo booking mới.
- ✅ PUT `/api/bookings/{id}`: Cập nhật toàn bộ booking (lịch trình, loại xe).
- ✅ DELETE `/api/bookings/{id}`: Hủy booking.
- ❌ Không dùng GET cho hành động có side-effect (như tạo booking) vì dễ bị cache và vi phạm chuẩn REST.

| HTTP Method | Tính chất | Khi áp dụng vào Taxi Booking |
|-------------|-----------|------------------------------|
| GET | Chỉ đọc, an toàn và idempotent | Lấy danh sách booking, xem chi tiết User/Driver |
| POST | Tạo mới, **không** idempotent | Đặt chuyến, đăng ký tài khoản |
| PUT | Cập nhật toàn bộ resource, idempotent | Cập nhật booking (đổi điểm đón/trả, loại xe) |
| PATCH | Cập nhật một phần | Đổi trạng thái booking (PENDING → COMPLETED) |
| DELETE | Xóa resource theo ID | Hủy booking, xóa tài khoản bị khóa |

**Idempotent nghĩa là gì?**: Gọi cùng một request nhiều lần vẫn cho kết quả như nhau (GET, PUT, DELETE). POST không idempotent nên phải cẩn thận tránh tạo trùng dữ liệu.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```java
@GetMapping("/api/bookings")
public List<Booking> listBookings() { return bookingService.findAll(); }

@PostMapping("/api/bookings")
public Booking createBooking(@RequestBody Booking booking) { return bookingService.save(booking); }
```

**Ví dụ đơn giản - Cách sai:**
```java
// ❌ Dùng GET để tạo mới
@GetMapping("/api/bookings/create")
public Booking createBooking(...) { ... }
```
**Tại sao sai:** GET phải an toàn, không thay đổi dữ liệu. Các proxy/cache có thể lặp lại request → sinh ra nhiều booking rác.

**Ví dụ trong dự án Taxi - Cách đúng:**
```java
@PutMapping("/api/bookings/{id}/status")
public ResponseEntity<Void> updateStatus(
        @PathVariable Long id,
        @RequestParam String status // ví dụ COMPLETED
) {
    bookingService.updateStatus(id, status);
    return ResponseEntity.noContent().build();
}
```

**Ví dụ trong dự án Taxi - Cách sai:**
```java
// ❌ Dùng POST để lấy danh sách
@PostMapping("/api/bookings/list")
public List<Booking> getBookings() {
    return bookingService.findAll();
}
```
**Tại sao sai:** POST dành cho tạo mới. Việc lấy danh sách nên dùng GET để tận dụng cache và rõ nghĩa.

**Checklist trước khi tạo endpoint mới:**
1. URL đã theo chuẩn `/api/{resource}` chưa (tham chiếu Buổi 3)?
2. HTTP Method đúng với hành động?
3. Đã viết JavaDoc/Comment ngắn giải thích nghiệp vụ?
4. Đã lưu request tương ứng vào Thunder Client Collection để đồng đội dùng chung?

---

### 5. ResponseEntity - Kiểm soát status code & header

#### 1. Định Nghĩa
`ResponseEntity` giống như phong bì gửi khách: bạn không chỉ gửi nội dung mà còn chọn được tem (HTTP status) và ghi chú thêm (header). Đây là cách chủ động kiểm soát response thay vì trả về object trần.

#### 2. Cách Thức Hoạt Động
1. Tạo `ResponseEntity` bằng các factory method (`ok`, `status`, `created`...).
2. Gắn body (nội dung JSON) + status code + header nếu cần.
3. Spring gửi trả về cho client.

**Những cách khởi tạo nhanh:**
- `ResponseEntity.ok(body)` → 200 OK.
- `ResponseEntity.status(HttpStatus.CREATED).body(body)` → 201 Created.
- `ResponseEntity.noContent().build()` → 204 No Content (không có body).
- `ResponseEntity.badRequest().body(error)` → 400.
- `ResponseEntity.notFound().build()` → 404.
- `ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error)` → 500.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Trả về status chính xác (201 khi tạo booking, 204 khi xóa).
- ✅ Thêm header (Location, Pagination info).
- ✅ Trả về lỗi có message rõ ràng (400, 404, 500).

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```java
@PostMapping("/calculator/divide")
public ResponseEntity<String> divide(@RequestBody DivideRequest req) {
    if (req.divisor() == 0) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body("Không thể chia cho 0");
    }
    double result = req.dividend() / req.divisor();
    return ResponseEntity.ok("Kết quả: " + result);
}
```

**Ví dụ đơn giản - Cách sai:**
```java
// ❌ Luôn trả về 200, kể cả khi dữ liệu sai
@PostMapping("/calculator/divide")
public String divide(@RequestBody DivideRequest req) {
    if (req.divisor() == 0) {
        return "Lỗi chia 0"; // client vẫn nhận status 200 → khó xử lý
    }
    return "Kết quả";
}
```
**Tại sao sai:** Status code không phản ánh lỗi → Client nghĩ request thành công.

**Ví dụ trong dự án Taxi - Cách đúng:**
```java
@PostMapping("/api/bookings/calculate")
public ResponseEntity<FareEstimateDTO> calculateFare(
        @RequestBody FareRequestDTO request
) {
    FareEstimateDTO fare = priceService.calculate(request);
    return ResponseEntity.status(HttpStatus.CREATED) // 201: tạo báo giá mới
            .header("X-Taxi-Currency", "VND")
            .body(fare);
}
```

**Ví dụ trong dự án Taxi - Cách sai:**
```java
// ❌ Trả về null khi không tìm thấy booking
@GetMapping("/api/bookings/{id}")
public BookingDetailDTO getBooking(@PathVariable Long id) {
    Booking booking = bookingService.findById(id);
    if (booking == null) {
        return null; // Client nhận 200 + body null, không biết lỗi
    }
    return mapper.toDTO(booking);
}
```
**Tại sao sai:** Không sử dụng status 404 → Client khó phân biệt giữa lỗi và dữ liệu rỗng. Nên trả về `ResponseEntity.notFound().build()`.

**Ví dụ bổ sung - Thêm header Location khi tạo resource:**
```java
@PostMapping("/api/bookings")
public ResponseEntity<BookingResponseDTO> createBooking(
        @RequestBody BookingRequestDTO request,
        UriComponentsBuilder uriBuilder
) {
    BookingResponseDTO response = bookingService.createBooking(request);
    URI location = uriBuilder.path("/api/bookings/{id}")
            .buildAndExpand(response.id())
            .toUri();
    return ResponseEntity.created(location)
            .header("X-Taxi-Currency", "VND")
            .body(response);
}
```

**Ghi nhớ:** Khi error, hãy trả về DTO lỗi rõ ràng thay vì `null`. Buổi 8 sẽ học Global Exception Handler để chuẩn hóa phần này.

---

### 6. Ghép các annotation vào một luồng đặt xe hoàn chỉnh

Đây là lúc kết nối toàn bộ mảnh ghép đã học (tương tự cách Buổi 3 ghép cấu trúc Controller). Hãy nhìn BookingController dưới đây:

```java
@RestController
@RequestMapping("/api/bookings")
public class BookingController {

    private final BookingService bookingService;

    public BookingController(BookingService bookingService) {
        this.bookingService = bookingService;
    }

    // Bước 1: Khách xem giá → @RequestBody + ResponseEntity
    @PostMapping("/calculate")
    public ResponseEntity<FareEstimateDTO> calculate(@RequestBody FareRequestDTO request) {
        FareEstimateDTO estimate = bookingService.calculateFare(request);
        return ResponseEntity.status(HttpStatus.CREATED)
                .header("X-Taxi-Estimate-Version", "v1")
                .body(estimate);
    }

    // Bước 2: Khách đặt xe → @RequestBody + ResponseEntity.created
    @PostMapping
    public ResponseEntity<BookingResponseDTO> create(
            @RequestBody BookingRequestDTO request,
            UriComponentsBuilder uriBuilder
    ) {
        BookingResponseDTO response = bookingService.createBooking(request);
        URI location = uriBuilder.path("/api/bookings/{id}")
                .buildAndExpand(response.id())
                .toUri();
        return ResponseEntity.created(location).body(response);
    }

    // Bước 3: Khách xem chi tiết → @PathVariable
    @GetMapping("/{id}")
    public ResponseEntity<BookingDetailDTO> getDetail(@PathVariable Long id) {
        return ResponseEntity.ok(bookingService.getDetail(id));
    }

    // Bước 4: Khách lọc lịch sử → @RequestParam
    @GetMapping
    public ResponseEntity<List<BookingSummaryDTO>> list(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "5") int size,
            @RequestParam(required = false) String status
    ) {
        return ResponseEntity.ok(bookingService.list(page, size, status));
    }
}
```

**Cách luyện tập với VS Code & Thunder Client:**
1. Mở Thunder Client → tạo Collection “Booking Flow”.
2. Tạo 4 request tương ứng (calculate, create, detail, list) và chạy lần lượt.
3. Trong VS Code, đặt breakpoint ở từng method để thấy request đi qua Controller nào.
4. Quan sát tab **Call Stack** để hiểu DispatcherServlet → Controller → Service đã vận hành ra sao (nhắc lại kiến thức IoC ở Buổi 2).
5. Mỗi lần chạy lại request, DevTools sẽ auto reload (nếu bạn đã bật ở Buổi 3) nên hãy bật Auto Save (`Cmd+,` → Auto Save = afterDelay).

Sau khi hiểu luồng tổng thể này, bạn sẽ dễ dàng thêm tính năng mới mà không bị rối annotation.

## Thực hành

### Bài tập 1: API Calculator 4 phép tính
**Mục tiêu:** Nắm `@RequestParam`, `@PathVariable`, `@RequestBody`, `ResponseEntity`.

**Yêu cầu:**
1. Tạo `CalculatorController` với base URL `/api/calculator`.
2. Viết các endpoint:
   - `GET /add?a=3&b=5` → dùng `@RequestParam`.
   - `GET /subtract/{a}/{b}` → dùng `@PathVariable`.
   - `POST /multiply` nhận JSON `{ "a": 2, "b": 4 }`.
   - `POST /divide` trả về `400 Bad Request` nếu chia cho 0 (dùng `ResponseEntity`).
3. Test bằng Thunder Client (VS Code) hoặc Postman.

**Kết quả mong đợi:**
- Các phép tính trả về kết quả chính xác.
- Thấy rõ sự khác nhau giữa query string, path và body.
- Lỗi chia 0 trả về status 400 với message dễ hiểu.

**Gợi ý:**
- Dùng `record` cho DTO nhanh gọn.
- Chạy app bằng phím tắt `Cmd+F5` (Debug) để quan sát biến.

### Bài tập 2: Tạo API kiểm tra giá cước nhanh
**Mục tiêu:** Phối hợp `@RequestParam` và `@RequestBody`.

**Yêu cầu:**
1. Tạo `FarePreviewController`.
2. Endpoint `POST /api/fares/preview?currency=VND`.
3. Body JSON gồm `distanceInKm`, `carType`.
4. Nếu `currency` khác `VND` → trả 400 với message “Chỉ hỗ trợ VND ở buổi này”.
5. In log ra Terminal mỗi khi method được gọi (dùng `System.out.println`).

**Kết quả mong đợi:**
- Biết thêm cách kết hợp query string + body.
- Nhớ kiểm tra dữ liệu đầu vào.

**Gợi ý:** Trong VS Code, dùng `log` snippet (`log` + `Tab`) để tạo nhanh `System.out.println`.

---

## Dự án Taxi

### Bài tập: API `POST /api/bookings/calculate`

**Mục tiêu:** Áp dụng toàn bộ kiến thức buổi 4 để tạo API ước tính giá cước.

**Yêu cầu chi tiết:**
1. **DTO yêu cầu (`FareRequestDTO`):**
   - `double distanceInKm`
   - `String carType` (SEDAN, SUV, BIKE)
   - `String pickupDistrict`
2. **DTO phản hồi (`FareEstimateDTO`):**
   - `double baseFare`
   - `double distanceFare`
   - `double totalFare`
   - `String currency` (VND)
   - `String note`
3. **Logic tính giá (mô phỏng đơn giản):**
   - Base fare: SEDAN = 15000, SUV = 20000, BIKE = 8000.
   - Distance fare: `distanceInKm * 9000` (SEDAN/SUV) hoặc `distanceInKm * 5000` (BIKE).
   - Nếu `pickupDistrict` là “District 1” → cộng 5% phụ phí.
4. **Yêu cầu kỹ thuật:**
   - Nhận JSON qua `@RequestBody`.
   - Trả về `ResponseEntity<FareEstimateDTO>` với status `201 CREATED`.
   - Đặt thêm header `X-Taxi-Estimate-Version: v1`.
   - Nếu `distanceInKm <= 0` → trả 400 với message rõ ràng.
5. **Test bằng Thunder Client:**
   - Lưu request thành Collection “Taxi Booking”.
   - Ghi chú lại các biến đầu vào để dễ thay đổi trong tương lai.

**Kết quả mong đợi:**
- API trả về JSON chi tiết, dễ dùng cho Mobile/Web.
- Học viên hiểu cách xử lý logic nghiệp vụ nhẹ bằng `if/else`.
- Nhớ kiểm soát status code và header bằng `ResponseEntity`.

**Gợi ý thêm trong VS Code:**
- Dùng `Ctrl+Shift+P` → “Format Document” để giữ code đẹp.
- Tạo task trong `README` cá nhân ghi lại công thức tính giá để buổi sau có thể tái sử dụng trong Service layer.

---

## Tổng kết buổi 4

**Những gì đã học:**
1. ✅ Hiểu sự khác nhau giữa `@RequestParam`, `@PathVariable`, `@RequestBody` và khi nào dùng từng loại.
2. ✅ Hiểu ý nghĩa các HTTP Method (GET/POST/PUT/DELETE) trong REST và khái niệm idempotent.
3. ✅ Biết sử dụng `ResponseEntity` để chủ động đặt status code, header và body.
4. ✅ Biết cách debug payload JSON, query string và path variable trong VS Code.
5. ✅ Nhìn được luồng đặt xe hoàn chỉnh thông qua `BookingController` (calculate → create → detail → list).

**Kiến thức quan trọng:**
- **`@RequestParam`:** Dùng cho filter linh hoạt (page, size, status...), có `required`, `defaultValue`, dễ kết hợp với phân trang.
- **`@PathVariable`:** Dùng cho ID hoặc phần bắt buộc trong URL, có thể đổi tên và có nhiều biến trên một route.
- **`@RequestBody`:** Dùng cho dữ liệu JSON phức tạp (DTO), yêu cầu `Content-Type: application/json`, dễ debug bằng breakpoint.
- **HTTP Methods:** 
  - GET: Chỉ đọc, an toàn, idempotent. 
  - POST: Tạo mới, không idempotent. 
  - PUT/PATCH: Cập nhật toàn bộ/một phần. 
  - DELETE: Xóa theo ID.
- **`ResponseEntity`:** Cách chuẩn để trả về 200/201/204/400/404/500 kèm body + header (Location, X-*), rất quan trọng khi xây dựng API thực tế.

**Chuẩn bị cho buổi 5:**
- ✅ Đã quen với việc gửi/nhận JSON và query string qua Postman/Thunder Client.
- ✅ Đã hiểu luồng Controller → Service ở mức logic đơn giản (chưa có DB).
- 🔜 Sẵn sàng kết nối với MySQL, map Entity và thao tác CRUD thực tế.

**Kiểm tra lại trước buổi 5:**
- [ ] Đã tạo `CalculatorController` và test đủ 4 phép tính với status code chuẩn.
- [ ] Đã tạo `FarePreviewController` và xử lý đúng trường hợp currency ≠ `VND`.
- [ ] Đã hoàn thành API `POST /api/bookings/calculate` và nhận về JSON `FareEstimateDTO` hợp lý.
- [ ] Đã thử debug ít nhất 1 endpoint có `@RequestBody` để xem giá trị DTO.
- [ ] Đã dùng Thunder Client (hoặc Postman) lưu lại các request quan trọng vào Collection.

**Bài tập về nhà (tùy chọn):**
- Viết thêm endpoint `PATCH /api/bookings/{id}/status` chỉ cập nhật trạng thái booking (PENDING → COMPLETED/CANCELLED) và trả về `204 No Content`.
- Thêm param filter `fromDate`, `toDate` cho API `GET /api/bookings` (dùng `@RequestParam(required = false)`).
- Thử viết một `ErrorResponseDTO` đơn giản (gồm `message`, `status`) và trả về trong các trường hợp 400 để quen dần với việc chuẩn hóa lỗi cho buổi 8.

﻿