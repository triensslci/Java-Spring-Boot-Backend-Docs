# 📚 Java Spring Boot Backend Training Course

Khóa học Java Spring Boot Backend từ con số 0 đến Deploy thành công lên Render sử dụng GitHub Actions CI/CD.

## 🎯 Mục tiêu khóa học

Sau khi hoàn thành khóa học, học viên sẽ có thể:

- ✅ Hiểu và vận dụng được Spring Framework Core (IoC, DI, Bean Lifecycle)
- ✅ Xây dựng RESTful API với Spring Boot
- ✅ Làm việc với Database (JPA/Hibernate, MySQL)
- ✅ Xử lý Authentication & Authorization (JWT)
- ✅ Viết Unit Test và Integration Test
- ✅ Deploy ứng dụng lên Render
- ✅ Thiết lập CI/CD pipeline với GitHub Actions
- ✅ Xây dựng hoàn chỉnh một dự án thực tế: **Taxi Booking System**

## 📖 Cấu trúc khóa học

Khóa học được chia thành **24 buổi học**, mỗi buổi là một file riêng biệt:

- **Buoi1.md** - Buoi24.md: Chi tiết từng buổi học
- **ChuongTrinhDaoTao.md**: Tổng quan toàn bộ chương trình đào tạo

### Giai đoạn 1: Khởi động & Nền tảng Spring Core (Buổi 1 - 4)
- Tổng quan, Môi trường & Git Workflow
- Tư duy OOP & Spring Core (DI/IoC)
- Spring Boot & REST API Cơ bản
- Request/Response & DTO Pattern

### Giai đoạn 2: Database & JPA (Buổi 5 - 8)
- Database Design & MySQL
- JPA/Hibernate Cơ bản
- Relationships (One-to-Many, Many-to-Many)
- Query Methods & Custom Queries

### Giai đoạn 3: Business Logic & Validation (Buổi 9 - 12)
- Service Layer & Business Logic
- Exception Handling
- Validation & Error Response
- Pagination & Sorting

### Giai đoạn 4: Security & Authentication (Buổi 13 - 16)
- Spring Security Cơ bản
- JWT Authentication
- Role-based Authorization
- Password Encryption

### Giai đoạn 5: Advanced Features (Buổi 17 - 20)
- File Upload/Download
- Email Service
- Scheduled Tasks
- Caching với Redis

### Giai đoạn 6: Testing & Deployment (Buổi 21 - 24)
- Unit Testing với JUnit & Mockito
- Integration Testing
- Deploy lên Render
- CI/CD với GitHub Actions

## 🚀 Yêu cầu tiên quyết

### Kiến thức cơ bản
- ✅ Hiểu cơ bản về Java (Class, Object, Interface, Inheritance)
- ✅ Hiểu cơ bản về OOP (Object-Oriented Programming)
- ✅ Biết sử dụng Git cơ bản (khuyến nghị)

### Công cụ cần cài đặt
- **JDK 17+** ([Download](https://adoptium.net/))
- **VS Code** với các extensions:
  - Extension Pack for Java (Microsoft)
  - Spring Boot Extension Pack (VMware)
  - Spring Boot Tools (VMware)
  - Spring Initializr Java Support (Microsoft)
  - Maven for Java (Microsoft)
  - GitLens (tùy chọn)
- **Postman** hoặc **Thunder Client** (extension trong VS Code)
- **Git** ([Download](https://git-scm.com/))
- **MySQL** (sẽ cài đặt trong Buổi 5)

## 📁 Cấu trúc thư mục

```
Java-Spring-Boot-Backend-Docs/
├── README.md                    # File này
├── ChuongTrinhDaoTao.md         # Tổng quan chương trình đào tạo
├── Buoi1.md                     # Buổi 1: Tổng quan, Môi trường & Git
├── Buoi2.md                     # Buổi 2: Tư duy OOP & Spring Core
├── Buoi3.md                     # Buổi 3: Spring Boot & REST API
├── ...
└── Buoi24.md                    # Buổi 24: CI/CD với GitHub Actions
```

## 🎓 Cách sử dụng tài liệu

### Cho học viên mới bắt đầu:
1. **Bắt đầu từ Buoi1.md**: Đọc và làm theo từng bước
2. **Tham khảo ChuongTrinhDaoTao.md**: Xem tổng quan để hiểu toàn bộ lộ trình
3. **Làm bài tập thực hành**: Mỗi buổi đều có phần "Thực hành" và "Dự án Taxi"
4. **Không bỏ qua phần lý thuyết**: Mỗi khái niệm đều có giải thích chi tiết và ví dụ

### Cho giảng viên:
- Sử dụng `ChuongTrinhDaoTao.md` để lên kế hoạch giảng dạy
- Sử dụng các file `BuoiX.md` để chuẩn bị bài giảng chi tiết
- Tất cả ví dụ và bài tập đều liên quan đến dự án **Taxi Booking System**

## 🚕 Dự án thực hành: Taxi Booking System

Xuyên suốt khóa học, học viên sẽ xây dựng một hệ thống **Đặt & Điều xe Taxi** hoàn chỉnh:

### Các tính năng chính:
- 👤 **Quản lý người dùng**: Đăng ký, đăng nhập, phân quyền (Passenger, Driver, Admin)
- 🚖 **Đặt xe**: Tạo booking, tìm driver gần nhất, tính giá cước
- 📍 **Quản lý chuyến đi**: Nhận chuyến, hoàn thành chuyến, hủy chuyến
- ⭐ **Đánh giá**: Feedback và rating cho driver
- 📊 **Thống kê**: Dashboard cho admin

### Công nghệ sử dụng:
- **Backend**: Spring Boot 3.x
- **Database**: MySQL
- **Security**: Spring Security + JWT
- **Build Tool**: Maven
- **Testing**: JUnit 5, Mockito
- **Deployment**: Render
- **CI/CD**: GitHub Actions

## 📝 Đặc điểm của tài liệu

### 1. Ngôn ngữ đơn giản, dễ hiểu
- ✅ Giải thích bằng ngôn ngữ đời thường, tránh thuật ngữ khó hiểu
- ✅ So sánh với các khái niệm quen thuộc trong cuộc sống
- ✅ Dịch thuật ngữ kỹ thuật sang tiếng Việt rõ nghĩa

### 2. Cấu trúc khái niệm đầy đủ
Mỗi khái niệm mới đều có đầy đủ 4 phần:
- **Định nghĩa**: Giải thích khái niệm là gì
- **Cách thức hoạt động**: Mô tả quy trình hoạt động
- **Trường hợp sử dụng**: Khi nào nên dùng, khi nào không nên
- **Ví dụ minh họa**: Cả ví dụ đúng và sai, với giải thích chi tiết

### 3. Tập trung vào thực hành
- ✅ Mỗi buổi đều có bài tập thực hành
- ✅ Tất cả ví dụ đều liên quan đến dự án Taxi Booking System
- ✅ Code examples có comment giải thích bằng tiếng Việt

### 4. Tích hợp VS Code
- ✅ Hướng dẫn sử dụng VS Code để phát triển
- ✅ Keyboard shortcuts và extensions hữu ích
- ✅ Cách debug, run, test trong VS Code

## 🔄 Đồng bộ tài liệu

**Lưu ý quan trọng:** Tài liệu được đồng bộ 2 chiều:
- `ChuongTrinhDaoTao.md` ↔ `BuoiX.md`
- Khi chỉnh sửa một file, phải cập nhật file tương ứng

## 🛠️ Cài đặt môi trường

### Bước 1: Cài đặt JDK 17+
```bash
# Kiểm tra version
java -version

# Nếu chưa có, cài đặt:
# macOS
brew install openjdk@17

# Windows
# Tải từ https://adoptium.net/
```

### Bước 2: Cài đặt VS Code
1. Tải VS Code từ [code.visualstudio.com](https://code.visualstudio.com/)
2. Mở VS Code → Extensions (Cmd+Shift+X / Ctrl+Shift+X)
3. Cài đặt các extensions:
   - Extension Pack for Java
   - Spring Boot Extension Pack
   - Spring Boot Tools
   - Spring Initializr Java Support
   - Maven for Java

### Bước 3: Cài đặt Git
```bash
# Kiểm tra version
git --version

# Nếu chưa có, tải từ https://git-scm.com/
```

### Bước 4: Cài đặt Postman
- Tải từ [postman.com](https://www.postman.com/)
- Hoặc cài Thunder Client extension trong VS Code

## 📚 Tài liệu tham khảo

### Tài liệu chính thức
- [Spring Framework Documentation](https://spring.io/projects/spring-framework)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA Documentation](https://spring.io/projects/spring-data-jpa)
- [Spring Security Documentation](https://spring.io/projects/spring-security)

### Tài liệu bổ sung
- [Java Documentation](https://docs.oracle.com/javase/)
- [Maven Documentation](https://maven.apache.org/guides/)
- [MySQL Documentation](https://dev.mysql.com/doc/)

## 🤝 Đóng góp

Nếu bạn phát hiện lỗi hoặc muốn cải thiện tài liệu:
1. Tạo Issue để báo lỗi hoặc đề xuất
2. Fork repository và tạo Pull Request
3. Đảm bảo tuân thủ cấu trúc và quy tắc viết tài liệu

## 📄 License

Tài liệu này được tạo cho mục đích giáo dục và có thể được sử dụng tự do.

## 📑 Phụ lục: Danh sách tài liệu

### 📋 Tài liệu tổng quan
- **[ChuongTrinhDaoTao.md](./ChuongTrinhDaoTao.md)** - Tổng quan toàn bộ chương trình đào tạo 24 buổi

### 📚 Các buổi học chi tiết

#### Giai đoạn 1: Khởi động & Nền tảng Spring Core (Buổi 1 - 4)
- ✅ **[Buoi1.md](./Buoi1.md)** - Tổng quan, Môi trường & Git Workflow
- ✅ **[Buoi2.md](./Buoi2.md)** - Tư duy OOP & Spring Core (DI/IoC)
- ✅ **[Buoi3.md](./Buoi3.md)** - Spring Boot & REST API Cơ bản
- ✅ **[Buoi4.md](./Buoi4.md)** - Request Handling & Response Entity

#### Giai đoạn 2: Database & JPA (Buổi 5 - 8)
- ✅ **[Buoi5.md](./Buoi5.md)** - Thiết kế CSDL & Kết nối MySQL
- ✅ **[Buoi6.md](./Buoi6.md)** - Entity Mapping & Basic CRUD
- ✅ **[Buoi7.md](./Buoi7.md)** - DTO Pattern & Object Mapping
- ⏳ **[Buoi8.md](./Buoi8.md)** - Validation & Global Exception Handling *(Đang cập nhật)*

#### Giai đoạn 3: Business Logic & Validation (Buổi 9 - 12)
- ⏳ **[Buoi9.md](./Buoi9.md)** - JPA Relationships (One-to-Many, Many-to-Many) *(Đang cập nhật)*
- ⏳ **[Buoi10.md](./Buoi10.md)** - Query Methods & Custom Queries *(Đang cập nhật)*
- ⏳ **[Buoi11.md](./Buoi11.md)** - Service Layer & Business Logic *(Đang cập nhật)*
- ⏳ **[Buoi12.md](./Buoi12.md)** - Pagination & Sorting *(Đang cập nhật)*

#### Giai đoạn 4: Security & Authentication (Buổi 13 - 16)
- ⏳ **[Buoi13.md](./Buoi13.md)** - Spring Security Cơ bản *(Đang cập nhật)*
- ⏳ **[Buoi14.md](./Buoi14.md)** - JWT Authentication *(Đang cập nhật)*
- ⏳ **[Buoi15.md](./Buoi15.md)** - Role-based Authorization *(Đang cập nhật)*
- ⏳ **[Buoi16.md](./Buoi16.md)** - Password Encryption *(Đang cập nhật)*

#### Giai đoạn 5: Advanced Features (Buổi 17 - 20)
- ⏳ **[Buoi17.md](./Buoi17.md)** - File Upload/Download *(Đang cập nhật)*
- ⏳ **[Buoi18.md](./Buoi18.md)** - Email Service *(Đang cập nhật)*
- ⏳ **[Buoi19.md](./Buoi19.md)** - Scheduled Tasks *(Đang cập nhật)*
- ⏳ **[Buoi20.md](./Buoi20.md)** - Caching với Redis *(Đang cập nhật)*

#### Giai đoạn 6: Testing & Deployment (Buổi 21 - 24)
- ⏳ **[Buoi21.md](./Buoi21.md)** - Unit Testing với JUnit & Mockito *(Đang cập nhật)*
- ⏳ **[Buoi22.md](./Buoi22.md)** - Integration Testing *(Đang cập nhật)*
- ⏳ **[Buoi23.md](./Buoi23.md)** - Deploy lên Render *(Đang cập nhật)*
- ⏳ **[Buoi24.md](./Buoi24.md)** - CI/CD với GitHub Actions *(Đang cập nhật)*

### 📊 Trạng thái tài liệu
- ✅ **Đã hoàn thành** - Tài liệu đã được tạo và sẵn sàng sử dụng
- ⏳ **Đang cập nhật** - Tài liệu đang được soạn thảo hoặc chưa có

---

## 🎉 Bắt đầu học ngay!

👉 **Bắt đầu từ [Buoi1.md](./Buoi1.md)** để bắt đầu hành trình học Spring Boot!

---

**Chúc bạn học tập hiệu quả! 🚀**

