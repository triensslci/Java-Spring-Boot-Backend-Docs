# Buổi 5: Thiết kế CSDL & Kết nối MySQL

## Ôn lại buổi 4

Trước khi bước sang thiết kế cơ sở dữ liệu, hãy rà lại kết quả Buổi 4 để chắc chắn dữ liệu lấy từ API đã rõ ràng:
- Đã xây thành công các endpoint nhận dữ liệu bằng `@RequestParam`, `@PathVariable`, `@RequestBody` và trả về `ResponseEntity` chuẩn với status code phù hợp.
- Đã có mẫu payload cụ thể cho luồng tính giá cước (`POST /api/bookings/calculate`) cùng danh sách trường bắt buộc → đây chính là nguồn để suy ra bảng và cột trong ERD.
- Đã log/validate đầu vào, hiểu các trường hợp lỗi phổ biến để chuyển thành constraint ở tầng database (vd. không cho phép distance âm, email phải hợp lệ).

Nếu các API ở buổi 4 chưa chạy ổn định, hãy hoàn thiện trước rồi mới sang bước thiết kế ERD và cấu hình MySQL để tránh thiết kế thiếu dữ liệu.

## Kiến thức

### 1. Tư duy thiết kế ERD cho Taxi Booking System

#### 1. Định Nghĩa
ERD (Entity Relationship Diagram) là sơ đồ thể hiện các bảng (entity) và cách chúng liên kết với nhau. Hãy tưởng tượng bạn lập sơ đồ tuyến xe buýt: mỗi bến chính là một bảng, còn tuyến đường nối giữa bến chính là quan hệ giữa các bảng. Trong Taxi Booking System, chúng ta cần nhìn thấy rõ User, Booking, Feedback liên hệ ra sao để đảm bảo dữ liệu không bị trùng lặp.

#### 2. Cách Thức Hoạt Động
1. Xác định thực thể chính từ nghiệp vụ: Passenger, Driver, Booking, Feedback.
2. Liệt kê thuộc tính quan trọng cho mỗi thực thể (ví dụ: Booking phải có `pickup_location`, `status`, `fare`).
3. Vẽ mối quan hệ (1 User có nhiều Booking, 1 Booking có thể có 1 Feedback).
4. Kiểm tra tính đầy đủ bằng cách chạy thử các kịch bản nghiệp vụ (đặt xe, hoàn thành, đánh giá).
5. Tối ưu để tránh dư thừa: đặt `driver_id` và `passenger_id` trong bảng Booking thay vì tạo bảng trung gian không cần thiết.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Khi bắt đầu dự án mới, cần ERD để truyền đạt cho dev/backend/mobile.
- ✅ Khi cần giải thích cho product owner về dữ liệu nào sẽ lưu, quan hệ ra sao.
- ✅ Khi phải ước lượng xem cần bao nhiêu bảng, index nào để phục vụ truy vấn (ví dụ thống kê Booking theo driver).
- ❌ Không nên bỏ qua ERD dù dự án nhỏ vì dễ dẫn đến thiết kế bảng thiếu thuộc tính hoặc quan hệ sai.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```sql
-- Bảng người dùng chung cho Passenger/Driver/Admin
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    password VARCHAR(120) NOT NULL,
    role ENUM('ROLE_PASSENGER','ROLE_DRIVER','ROLE_ADMIN') NOT NULL
);

-- Bảng booking liên kết tới passenger và driver
CREATE TABLE bookings (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    passenger_id BIGINT NOT NULL,
    driver_id BIGINT,
    pickup_location VARCHAR(255) NOT NULL,
    dropoff_location VARCHAR(255) NOT NULL,
    status ENUM('PENDING','ACCEPTED','COMPLETED','CANCELED') NOT NULL,
    total_fare DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (passenger_id) REFERENCES users(id),
    FOREIGN KEY (driver_id) REFERENCES users(id)
);
```

**Ví dụ đơn giản - Cách sai:**
```sql
-- ❌ Tách passenger và driver thành 2 bảng riêng, làm mất tính dùng chung
CREATE TABLE passengers (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100),
    phone VARCHAR(20)
);

CREATE TABLE drivers (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100),
    phone VARCHAR(20)
);

CREATE TABLE bookings (
    passenger_phone VARCHAR(20), -- chỉ lưu số điện thoại, không có khóa ngoại
    driver_phone VARCHAR(20)
);
```
**Tại sao sai:** Booking chỉ lưu số điện thoại nên không ràng buộc được khóa ngoại, khi người dùng đổi số sẽ mất liên kết. Việc tách passenger/driver thành 2 bảng riêng khiến việc quản lý role khó khăn và dư thừa thông tin.

**Ví dụ trong dự án Taxi - Cách đúng:**
```sql
-- Feedback gắn với booking để biết chuyến nào được đánh giá
CREATE TABLE feedback (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    booking_id BIGINT NOT NULL,
    rating TINYINT CHECK (rating BETWEEN 1 AND 5),
    comment VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (booking_id) REFERENCES bookings(id)
);
```

**Ví dụ trong dự án Taxi - Cách sai:**
```sql
-- ❌ Lưu rating trực tiếp trong bảng users
ALTER TABLE users ADD COLUMN last_rating INT;
```
**Tại sao sai:** Một User có thể tham gia nhiều chuyến, nên rating cần gắn với từng Booking. Lưu rating trong `users` làm mất lịch sử đánh giá và không thể thống kê theo chuyến.

#### 5. Sơ đồ quan hệ trực quan
```
┌───────────-────┐          1     n         ┌─────────────--──┐
│    USERS       │------------------------->│   BOOKINGS      │
│ id (PK)        │        passenger_id      │ id (PK)         │
│ full_name      │                          │ passenger_id FK │
│ role (enum)    │<-------------------------│ driver_id FK    │
│ email (unique) │          1     n         │ status          │
│ password       │                          │ total_fare      │
└──────┬───-─────┘                          └──────┬──────--──┘
       │                                          1│ 1
       │                                           │
       │                                           │ booking_id
       │                                           │
       │                             ┌─────────────▼─────────────┐
       └────────────────────────────>│         FEEDBACK          │
                  (driver_id)        │ id (PK)                   │
                                     │ booking_id (FK)           │
                                     │ rating (1-5)              │
                                     │ comment                   │
                                     └───────────────────────────┘
```
- **User** đóng hai vai trò Passenger/Driver thông qua cột `role`, giúp tái sử dụng bảng và giảm duplication.
- **Booking** giữ hai khóa ngoại `passenger_id`, `driver_id` để biết ai đặt xe, ai nhận chuyến. Quan hệ này là 1-nhiều (một User có thể có nhiều Booking).
- **Feedback** có quan hệ 1-1 với Booking để lưu đánh giá của từng chuyến. Dùng khóa ngoại `booking_id` nhằm đảm bảo mỗi chuyến tối đa một feedback.
- Khi cần thêm bảng khác (Payments, DriverLocation), chỉ việc nối thêm khóa ngoại tương ứng từ Booking hoặc User theo cùng nguyên tắc.

---

### 2. Cấu hình Datasource trong `application.properties`

#### 1. Định Nghĩa
Datasource là “thông tin đăng nhập bãi xe” của ứng dụng: bao gồm địa chỉ bãi (URL), người gác cổng (driver), người được phép vào (username) và mật khẩu. Không có bộ thông tin này, Spring Boot không thể mở cổng đến MySQL. (Tham khảo hướng dẫn cấu hình datasource từ Spring/Hibernate community, 2024.)

#### 2. Cách Thức Hoạt Động
1. Khi Spring Boot khởi động, nó đọc file `application.properties` và nạp các key `spring.datasource.*`.
2. Spring tạo một `DataSource` (pool của HikariCP) dựa trên URL, driver, username và password.
3. Hibernate/JPA sử dụng DataSource này mỗi khi cần thực thi SQL (select/insert…).
4. Nếu URL hoặc credential sai, Hikari sẽ log lỗi và app dừng ngay, giúp bạn phát hiện sớm.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Kết nối MySQL local (`jdbc:mysql://localhost:3306/taxi_booking_dev`) trong lúc phát triển.
- ✅ Kết nối tới MySQL chạy trong Docker (thay host bằng tên container) hoặc dịch vụ cloud (RDS, PlanetScale).
- ✅ Tạo nhiều profile: `application-dev.properties`, `application-prod.properties` để dùng user khác nhau.
- ❌ Không commit trực tiếp password thật vào Git; nên dùng biến môi trường hoặc VS Code `launch.json`.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/taxi_booking?useSSL=false&serverTimezone=Asia/Ho_Chi_Minh
spring.datasource.username=root
spring.datasource.password=secret123
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Log SQL để dễ debug trong VS Code terminal
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

**Ví dụ đơn giản - Cách sai:**
```properties
# ❌ Sai port và thiếu timezone → dễ lỗi khi parse DATETIME
spring.datasource.url=jdbc:mysql://localhost:13306/taxi
spring.datasource.username=root
spring.datasource.password=
```
**Tại sao sai:** Port 13306 không chạy MySQL (mặc định 3306), password rỗng không an toàn, thiếu `serverTimezone` khiến MySQL 8 cảnh báo và có thể trả về thời gian sai.

**Ví dụ trong dự án Taxi - Cách đúng:**
```properties
# Sử dụng profile riêng cho môi trường dev
spring.profiles.active=dev

# application-dev.properties
spring.datasource.url=jdbc:mysql://localhost:3306/taxi_booking_dev?characterEncoding=UTF-8
spring.datasource.username=taxi_dev
spring.datasource.password=S3cureDev!
```

**Ví dụ trong dự án Taxi - Cách sai:**
```properties
# ❌ Dùng root chung cho tất cả môi trường và commit lên Git
spring.datasource.username=root
spring.datasource.password=root
```
**Tại sao sai:** Khi deploy lên server, tài khoản root có toàn quyền, dễ bị khai thác nếu lộ repo. Nên tạo user riêng `taxi_app` chỉ có quyền trên schema cần thiết.

---

### 3. `spring.jpa.hibernate.ddl-auto` và kiểm soát schema

#### 1. Định Nghĩa
`spring.jpa.hibernate.ddl-auto` giống công tắc điều khiển “thợ xây Hibernate”: bạn chọn chế độ `create`, `update`, `validate` hay `none` để quyết định xem Hibernate có được phép tạo/xoá/cập nhật bảng dựa trên Entity hay chỉ đứng ngoài kiểm tra. (Theo tài liệu Hibernate ORM, 2024.)

#### 2. Cách Thức Hoạt Động
1. Hibernate đọc metadata của toàn bộ Entity khi ứng dụng khởi động.
2. Nếu `ddl-auto=create` hoặc `create-drop`, Hibernate tạo lại bảng từ đầu (và xóa khi tắt app nếu là `create-drop`).
3. Nếu `ddl-auto=update`, Hibernate so sánh schema hiện tại với Entity và bổ sung cột/bảng cần thiết (không xóa cột cũ).
4. Nếu `ddl-auto=validate`, Hibernate chỉ kiểm tra sự khớp giữa Entity và bảng. Sai khác → app dừng.
5. `ddl-auto=none` bỏ qua hoàn toàn, phù hợp khi bạn muốn dùng script migration (Flyway/Liquibase).

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ `create-drop`: demo nhanh hoặc viết thử nghiệm, không tiếc dữ liệu.
- ✅ `update`: môi trường dev nội bộ cần giữ dữ liệu mẫu nhưng vẫn muốn Entity mới tự sinh cột.
- ✅ `validate`: staging/production để đảm bảo schema đúng trước khi app chạy, tránh mất dữ liệu.
- ❌ Không để `create`/`create-drop` trên môi trường thật vì mỗi lần restart là mất sạch booking của khách.

#### 4. Bảng so sánh tác động của từng cấu hình

| Giá trị `ddl-auto` | Khi nên dùng | Tác động lên schema | Rủi ro nếu đặt sai |
|--------------------|-------------|----------------------|--------------------|
| `create` | Demo nhanh, dữ liệu test | Hibernate drop toàn bộ bảng rồi tạo mới khi app start | Restart production sẽ xóa sạch booking thật |
| `create-drop` | Unit test tạm thời | Tạo bảng khi start, drop khi app dừng | Deploy production → database biến mất khi server shutdown |
| `update` | Dev nội bộ cần giữ dữ liệu mẫu | Thêm cột/bảng mới theo Entity, giữ nguyên dữ liệu cũ | Đổi tên field → còn cả cột cũ và mới, dữ liệu lệch |
| `validate` | Staging/Production | Chỉ kiểm tra schema, thiếu cột → app không chạy | Nếu nhầm sang `update`, schema có thể bị chỉnh tự động không kiểm soát |
| `none` | Khi dùng Flyway/Liquibase | Hibernate không động vào DB, bạn tự chạy migration | Quên chạy migration → app lỗi SQL vì schema chưa cập nhật |

#### 5. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```properties
# Trong giai đoạn học hoặc tạo POC
spring.jpa.hibernate.ddl-auto=update
```

**Ví dụ đơn giản - Cách sai:**
```properties
# ❌ Triển khai production nhưng vẫn để create-drop
spring.jpa.hibernate.ddl-auto=create-drop
```
**Tại sao sai:** Mỗi lần restart server sẽ xóa toàn bộ bảng và dữ liệu booking thật của khách.

**Ví dụ trong dự án Taxi - Cách đúng:**
```properties
# application-prod.properties
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.sql.init.mode=never
```

**Ví dụ trong dự án Taxi - Cách sai:**
```properties
# ❌ Quên set ddl-auto nên Hibernate mặc định = create-drop khi dùng H2 profile
spring.datasource.url=jdbc:mysql://localhost:3306/taxi_booking
```
**Tại sao sai:** Khi chuyển từ H2 sang MySQL mà không đặt `ddl-auto`, team dễ quên và bị mất schema bất ngờ. Luôn ghi rõ để tránh phụ thuộc mặc định.

---

### 4. Kết nối MySQL bằng VS Code Database Tools

#### 1. Định Nghĩa
`Database Client for VS Code` (tên gói trên Open VSX: `vscode-mysql-client2`) là extension giúp VS Code trở thành “MySQL Workbench mini”. Bạn có thể duyệt bảng, xem schema, chạy query mà không cần rời khỏi editor. Đây là cách tối ưu khi vừa code Spring Boot vừa cần kiểm chứng dữ liệu Taxi Booking. (Tham khảo mô tả extension trên Open VSX.)

#### 2. Cách Thức Hoạt Động
1. Mở VS Code → Extensions (`Cmd+Shift+X`) → tìm “Database Client for VS Code” (ID `cweijan.vscode-mysql-client2`) → Install.
2. Sau khi cài, panel **Database Client** xuất hiện trong Activity Bar. Chọn `+` → *Add Connection*.
3. Điền thông tin:
   - **DB Type:** MySQL
   - **Host:** `localhost`
   - **Port:** `3306`
   - **User/Password:** user riêng (`taxi_dev`)
   - **Database:** `taxi_booking_dev`
   - **Safe Mode:** bật để extension yêu cầu xác nhận trước khi chạy lệnh nguy hiểm (`DROP`, `TRUNCATE`).
4. Extension lưu thông tin trong `~/Library/Application Support/Code/User/settings.json` hoặc `.vscode/settings.json` (nếu bạn chọn “Save to Workspace”) → team khác mở project lên là dùng được ngay.
5. Khi cần xem dữ liệu, click phải connection → `Connect`. Bạn có thể:
   - Expand bảng để xem cột, index.
   - Click `Query` → VS Code mở một `.sql` tab, bạn gõ `SELECT * FROM bookings LIMIT 5;` rồi nhấn `Cmd+Enter` để chạy.
   - Kết quả hiển thị trong panel Result, có thể export CSV cho QA.
6. Extension hỗ trợ snippets, history, thậm chí hiển thị ER diagram đơn giản dựa trên foreign key → rất hữu ích để nhắc lại quan hệ User ↔ Booking ↔ Feedback.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Sau khi gọi API `POST /api/bookings`, mở Database Client → chạy `SELECT` để kiểm tra record mới xuất hiện chưa.
- ✅ Khi cần đối chiếu schema đã triển khai với file `db/init/users.sql`.
- ✅ Trong buổi học, giảng viên yêu cầu share màn hình và trình diễn dữ liệu: chỉ cần mở panel Database Client ngay trong VS Code.
- ✅ Kết hợp với Live Share: mentor có thể xem query học viên chạy và chỉnh sửa trực tiếp.
- ❌ Không thao tác trực tiếp trên database production (dù extension hỗ trợ), trừ khi đã bật VPN, có quyền hạn và ghi log đầy đủ.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```json
{
  "database-client.connections": [
    {
      "name": "Taxi Local",
      "driver": "MySQL",
      "server": "localhost",
      "port": 3306,
      "database": "taxi_booking_dev",
      "username": "taxi_dev",
      "password": "S3cureDev!",
      "safeMode": true
    }
  ]
}
```

**Ví dụ đơn giản - Cách sai:**
```json
{
  "database-client.connections": [
    {
      "name": "Taxi Prod",
      "driver": "MySQL",
      "server": "prod-db.company.com",
      "database": "taxi_booking",
      "username": "root",
      "password": "root",
      "safeMode": false
    }
  ]
}
```
**Tại sao sai:** Lưu credential production kèm quyền root trong VS Code cá nhân rất nguy hiểm; chỉ một lần mất máy hoặc bật Live Share là lộ ngay. Ngoài ra tắt Safe Mode khiến câu lệnh `DROP` chạy tức thì.

**Ví dụ trong dự án Taxi - Cách đúng:**
```sql
-- Query kiểm tra các chuyến mới được tạo
SELECT id, passenger_id, driver_id, status, total_fare
FROM bookings
WHERE status = 'PENDING'
ORDER BY created_at DESC
LIMIT 5;
```

**Ví dụ trong dự án Taxi - Cách sai:**
```sql
-- ❌ Quên WHERE khiến toàn bộ chuyến bị đổi trạng thái
UPDATE bookings SET status = 'COMPLETED';
```
**Tại sao sai:** Không có điều kiện `WHERE` nên mọi booking đều chuyển sang COMPLETED. Khi thao tác trong extension, hãy bật `Safe Mode` và đọc lại câu lệnh trước khi `Cmd+Enter`.

**Quy trình VS Code mẫu:**
1. Dùng `Cmd+Shift+P` → `Database Client: New Query`.
2. Gõ câu SQL và nhấn `Cmd+Enter`.
3. Nếu muốn lưu script, bấm `Cmd+S` để cất vào thư mục `db/playground`.
4. Khi cần chia sẻ, dùng `Copy Results` → paste vào Slack/Notion cho mentor.

---

### 5. Kết nối MySQL bằng Navicat

#### 1. Định Nghĩa
Navicat for MySQL (hoặc Navicat Premium) là một công cụ GUI giúp bạn quản lý MySQL thông qua giao diện trực quan: tạo database, bảng, chạy query, xem dữ liệu… giống như một “remote điều khiển” cho MySQL. Khi code trong VS Code, bạn có thể mở thêm Navicat để quan sát dữ liệu `taxi_booking_dev` thay vì phải gõ lệnh trong terminal. Điều này đặc biệt hữu ích cho người mới, vì mọi thứ (bảng, cột, khóa ngoại) đều được hiển thị bằng hình ảnh.

#### 2. Cách Thức Hoạt Động
1. **Cài đặt Navicat:**
   - Tải Navicat bản dùng thử từ trang chủ (chọn phiên bản hỗ trợ MySQL).
   - Cài đặt như ứng dụng bình thường trên macOS (kéo vào thư mục `Applications`).
2. **Tạo kết nối mới đến MySQL local:**
   - Mở Navicat → `File` → `New Connection` → chọn **MySQL**.
   - Điền thông tin kết nối:
     - **Connection Name:** `Taxi Local` (tên tùy ý, dễ nhận biết).
     - **Host:** `127.0.0.1` hoặc `localhost`.
     - **Port:** `3306` (hoặc port bạn đã cấu hình cho MySQL).
     - **User Name:** `taxi_dev` (user riêng cho dự án Taxi).
     - **Password:** password mạnh bạn đã tạo cho user `taxi_dev`.
   - Bấm **Test Connection**:
     - Nếu hiện “Connection Successful” → bấm **OK** để lưu kết nối.
     - Nếu báo lỗi, kiểm tra lại:
       - MySQL đã chạy chưa?
       - Port đúng chưa?
       - User/password đúng chưa?
3. **Chọn database `taxi_booking_dev`:**
   - Sau khi kết nối thành công, ở panel bên trái bạn sẽ thấy danh sách database.
   - Click phải vào `taxi_booking_dev` → chọn **Open Database**.
   - Mở tiếp thư mục **Tables** để xem các bảng (vd. `users`, `bookings`… sau khi bạn tạo).
4. **Chạy query kiểm tra dữ liệu:**
   - Click phải lên database `taxi_booking_dev` → **New Query**.
   - Gõ SQL, ví dụ:
     ```sql
     -- Kiểm tra 5 chuyến đi mới nhất trong hệ thống Taxi
     SELECT id, passenger_id, driver_id, status, total_fare, created_at
     FROM bookings
     ORDER BY created_at DESC
     LIMIT 5;
     ```
   - Bấm nút **Run** (hình tia sét) hoặc phím tắt (tùy OS).
   - Kết quả hiển thị ở panel phía dưới, bạn có thể:
     - Sắp xếp theo cột.
     - Lọc nhanh bằng filter.
     - Export ra CSV/Excel nếu cần gửi cho mentor/QA.
5. **Đồng bộ với VS Code:**
   - Vừa mở project Spring Boot trong VS Code để code, vừa mở Navicat để quan sát dữ liệu.
   - Sau khi gọi API (ví dụ `POST /api/bookings`), quay sang Navicat → bấm **Refresh** bảng `bookings` để xem record mới được thêm.

#### 3. Trường Hợp Sử Dụng Thực Tế
- ✅ Khi bạn **chưa quen câu lệnh SQL** nhưng vẫn muốn xem dữ liệu: Navicat hiển thị bảng dạng lưới giống Excel, dễ đọc.
- ✅ Khi cần **chỉnh sửa dữ liệu test nhanh** (ví dụ đổi status một booking từ `PENDING` → `COMPLETED` để debug): có thể double click vào cell và sửa giá trị.
- ✅ Khi cần **vẽ sơ đồ quan hệ**: một số phiên bản Navicat hỗ trợ reverse-engineer ERD từ database, giúp bạn nhìn lại quan hệ `users` ↔ `bookings` ↔ `feedback`.
- ✅ Khi làm việc nhóm, mentor có thể yêu cầu bạn **chụp màn hình Navicat** để kiểm tra schema/dữ liệu.
- ❌ Không dùng Navicat để thao tác thẳng trên database **production** nếu bạn chưa có quy trình backup/kiểm soát, vì chỉ một câu lệnh sai (ví dụ `DELETE`/`UPDATE` không có `WHERE`) có thể phá hủy dữ liệu thật.

#### 4. Ví Dụ Minh Họa

**Ví dụ đơn giản - Cách đúng:**
```sql
-- Sử dụng Navicat để kiểm tra số lượng user hiện có
SELECT COUNT(*) AS total_users
FROM users;

-- Kiểm tra các chuyến đang ở trạng thái PENDING
SELECT id, passenger_id, driver_id, status
FROM bookings
WHERE status = 'PENDING'
ORDER BY created_at DESC
LIMIT 10;
```
- Bạn chạy 2 câu lệnh trên trong cửa sổ Query của Navicat.
- Kết quả giúp bạn:
  - Biết đã có bao nhiêu user test trong hệ thống.
  - Dễ dàng quan sát các booking đang chờ được driver nhận.

**Ví dụ đơn giản - Cách sai:**
```sql
-- ❌ Cập nhật toàn bộ booking mà quên điều kiện WHERE
UPDATE bookings
SET status = 'COMPLETED';
```
**Tại sao sai:** Câu lệnh này sẽ chuyển **tất cả** booking (kể cả PENDING, IN_PROGRESS) sang COMPLETED. Nếu chạy trên database production sẽ phá hỏng toàn bộ luồng nghiệp vụ. Khi dùng Navicat, hãy:
- Luôn đọc kỹ lại câu lệnh trước khi bấm Run.
- Ưu tiên chạy trên database dev (`taxi_booking_dev`), không truy cập trực tiếp DB thật.

**Ví dụ trong dự án Taxi - Cách đúng (Navicat + API):**
1. Mở VS Code, chạy ứng dụng Spring Boot (`mvn spring-boot:run` hoặc Run trong VS Code).
2. Gọi API tạo booking mới (vd. `POST /api/bookings`) bằng Thunder Client/Postman.
3. Mở Navicat:
   - Refresh bảng `bookings`.
   - Chạy query:
     ```sql
     -- Xem các chuyến vừa tạo trong 5 phút gần nhất
     SELECT id, passenger_id, driver_id, status, total_fare, created_at
     FROM bookings
     WHERE created_at >= NOW() - INTERVAL 5 MINUTE
     ORDER BY created_at DESC;
     ```
4. Đối chiếu dữ liệu trong Navicat với payload bạn đã gửi từ VS Code để chắc chắn API hoạt động đúng.

**Ví dụ trong dự án Taxi - Cách sai (kết hợp Navicat & app):**
1. Ứng dụng đang bật `spring.jpa.hibernate.ddl-auto=validate` (chỉ kiểm tra schema, không tự sửa).
2. Bạn vào Navicat, mở bảng `bookings` và **xóa nhầm cột** `passenger_id`.
3. Restart ứng dụng:
   - Hibernate validate schema thất bại vì thiếu cột `passenger_id`.
   - App không khởi động được.

**Tại sao sai:** Sửa cấu trúc bảng (thêm/xóa cột) trực tiếp trong Navicat mà không thông qua migration/script sẽ khiến schema **lệch** so với Entity và Flyway/Liquibase. Nguyên tắc:
- Dùng Navicat để **xem dữ liệu, chạy SELECT, thao tác dữ liệu test nhỏ**.
- Khi muốn thay đổi cấu trúc bảng (thêm cột, đổi kiểu dữ liệu) → hãy viết **script SQL/migration** rồi chạy có kiểm soát (review, commit vào repo).

## Thực hành

### Bài tập: Vẽ ERD Taxi Booking

**Mục tiêu:** Hiểu mối quan hệ giữa User, Booking, Feedback trước khi map Entity.

**Yêu cầu:**
1. Liệt kê tất cả entity cần có (ít nhất: `users`, `bookings`, `feedback`).
2. Xác định khóa chính, khóa ngoại cho từng bảng.
3. Minh họa quan hệ bằng draw.io, Excalidraw hoặc bảng Markdown.
4. Ghi chú thêm các chỉ số cần index (ví dụ `status`, `created_at`).

**Kết quả mong đợi:**
- Sơ đồ ERD rõ ràng, dễ đọc.
- Có giải thích vì sao cần khóa ngoại `passenger_id`, `driver_id`.
- Thể hiện được quan hệ 1-nhiều giữa User và Booking.

**Gợi ý:** Trong VS Code, cài extension Draw.io Integration để mở sơ đồ ngay trong editor.

---

### Bài tập: Cấu hình MySQL & chạy thử kết nối

**Mục tiêu:** Tự cài MySQL, chỉnh `application.properties`, chạy app và kiểm tra log.

**Yêu cầu:**
1. Cài MySQL 8, tạo database `taxi_booking_dev`.
2. Tạo user `taxi_dev` với password mạnh, cấp quyền trên schema vừa tạo.
3. Điền thông tin vào `application-dev.properties`.
4. Chạy ứng dụng (`mvn spring-boot:run` hoặc Run trên VS Code).
5. Quan sát log để chắc chắn kết nối thành công (`HikariPool-1 - Starting...`).

**Kết quả mong đợi:**
- Ứng dụng khởi động không lỗi.
- Có log `Initialized JPA EntityManagerFactory`.
- Database xuất hiện bảng `bookings` sau khi tạo Entity (sẽ thực hiện ở buổi sau).

**Gợi ý:** Dùng Terminal tích hợp trong VS Code (`Ctrl+\``) để chạy `mysql -u taxi_dev -p`.

---

## Dự án Taxi

### Bài tập: Thiết kế bảng `users` và `bookings`, kiểm tra kết nối

**Mục tiêu:** Chuẩn bị sẵn schema cho buổi 6 (mapping Entity & CRUD) và đảm bảo Spring Boot kết nối MySQL trơn tru.

**Yêu cầu chi tiết:**
1. **Script SQL:**
   - Tạo bảng `users` (id, full_name, email unique, password, role, created_at).
   - Tạo bảng `bookings` (id, passenger_id, driver_id, pickup_location, dropoff_location, status, total_fare, created_at).
   - Đảm bảo `passenger_id` và `driver_id` là khóa ngoại tới `users`.
2. **Cấu hình Spring Boot:**
   - Tạo file `src/main/resources/application-dev.properties`.
   - Điền datasource + `spring.jpa.hibernate.ddl-auto=validate` để kiểm tra script tạo đúng.
3. **Kiểm tra bằng VS Code:**
   - Dùng SQLTools kết nối tới `taxi_booking_dev`.
   - Chạy `SELECT COUNT(*) FROM users;` xem schema đã sẵn sàng.
   - Ghi chú lại kết quả vào file `README` cá nhân hoặc log VS Code.

**Kết quả mong đợi:**
- Schema đúng chuẩn, không lỗi khóa ngoại.
- Spring Boot chạy `mvn spring-boot:run -Dspring-boot.run.profiles=dev` mà không báo lỗi kết nối.
- Sẵn sàng cho buổi 6 để map Entity.

**Gợi ý VS Code:** Tạo nhiệm vụ (Task) “Start MySQL” trong `.vscode/tasks.json` để bật/tắt MySQL Docker nhanh (`Cmd+Shift+B`).

---

## Tổng kết buổi 5

**Những gì đã học:**
1. Hiểu cách phân tích nghiệp vụ Taxi để vẽ ERD chuẩn (User ↔ Booking ↔ Feedback).
2. Nắm cấu hình datasource trong `application.properties` và quản lý password an toàn.
3. Biết chọn chế độ `spring.jpa.hibernate.ddl-auto` phù hợp từng môi trường.
4. Kết nối và kiểm tra dữ liệu MySQL ngay trong VS Code bằng SQLTools/MySQL extension.

**Kiến thức quan trọng:**
- ERD là bản đồ dữ liệu, phải rõ entity, khóa chính/phụ, quan hệ 1-nhiều.
- Datasource = URL + user + password + driver; nhớ thêm `serverTimezone` cho MySQL 8.
- `ddl-auto` quyết định sinh/ktra schema, không dùng `create-drop` ngoài môi trường học.
- VS Code có thể trở thành “mini Workbench” để xem dữ liệu mà không rời khỏi editor.

**Chuẩn bị cho buổi 6:**
- Hoàn thành script tạo bảng `users`, `bookings`.
- Tạo file `application-dev.properties` với cấu hình datasource chuẩn.
- Đảm bảo app khởi động thành công và kết nối được tới MySQL.

**Kiểm tra lại trước buổi 6:**
- [ ] Đã vẽ ERD và hiểu rõ quan hệ User ↔ Booking ↔ Feedback.
- [ ] Đã tạo database `taxi_booking_dev` cùng user riêng.
- [ ] Đã cấu hình datasource + `ddl-auto` đúng trong Spring Boot.
- [ ] Đã kết nối MySQL từ VS Code và chạy thử query kiểm tra bảng.
- [ ] Đã lưu lại script SQL để reuse trong dự án.

**Bài tập về nhà (tùy chọn):**
- Viết thêm bảng `feedback` và `payments`, suy nghĩ về quan hệ 1-nhiều và khóa ngoại.
- Thử tạo sơ đồ ERD bằng công cụ miễn phí (dbdiagram.io) và export hình ảnh để đưa vào tài liệu đội nhóm.


