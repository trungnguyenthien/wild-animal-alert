# Tài liệu API & Thiết kế Cơ sở Dữ liệu (Đơn giản cho học sinh)

Ứng dụng phục vụ nghiên cứu học tập với các chức năng cơ bản: Đăng ký, Đăng nhập, Đăng ký Device Token và Gửi Push Notification.

---

## I. Mô hình Cơ sở Dữ liệu (Database Models)

Để lưu trữ dữ liệu cho ứng dụng này, chỉ cần 2 bảng (hoặc collections) đơn giản sau:

### 1. Model `User` (Người dùng)

Lưu trữ thông tin tài khoản và Device Token dùng để nhận thông báo.

| Tên trường (Field) | Kiểu dữ liệu | Mô tả              | Ghi chú                                                       |
| :----------------- | :----------- | :----------------- | :------------------------------------------------------------ |
| `id`               | String       | Mã hex 4 ký tự     | Khóa chính (Ví dụ: `"a1b2"`, `"f3e4"`)                        |
| `username`         | String       | Tên tài khoản      | Duy nhất                                                      |
| `password`         | String       | Mật khẩu           | Lưu mật khẩu gốc hoặc mã hóa đơn giản                         |
| `deviceToken`      | String       | Token của thiết bị | Dùng cho Firebase Cloud Messaging (FCM) hoặc dịch vụ tương tự |
| `createdAt`        | Date         | Ngày tạo           | Tự động sinh                                                  |

### 2. Model `Notification` (Lịch sử thông báo - Tùy chọn)

Nếu muốn lưu lại lịch sử tin nhắn đã gửi.

| Tên trường (Field) | Kiểu dữ liệu | Mô tả              | Ghi chú                |
| :----------------- | :----------- | :----------------- | :--------------------- |
| `id`               | String       | ID thông báo       |                        |
| `receiverId`       | String       | ID người nhận      | Liên kết với `User.id` |
| `title`            | String       | Tiêu đề thông báo  |                        |
| `body`             | String       | Nội dung thông báo |                        |
| `sentAt`           | Date         | Thời gian gửi      |                        |

---

## II. Chi tiết API Endpoints

### 1. Đăng ký User (Register)

Đăng ký tài khoản mới với username và password.

- **URL:** `/api/register`
- **Method:** `POST`
- **Request Body (JSON):**

```json
{
  "username": "hocsinh1",
  "password": "123"
}
```

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Đăng ký thành công**
    - **HTTP Status Code:** `201 Created`
    - **Response Body:**
      ```json
      {
        "message": "Đăng ký thành công",
        "userid": "a1b2"
      }
      ```
      _(Ghi chú: ID `a1b2` được sinh ngẫu nhiên dưới dạng chuỗi hex 4 ký tự)_

  - **TH 2: Trùng tên tài khoản**
    - **HTTP Status Code:** `409 Conflict`
    - **Response Body:**
      ```json
      {
        "error": "Tên tài khoản đã tồn tại"
      }
      ```

  - **TH 3: Thiếu thông tin bắt buộc**
    - **HTTP Status Code:** `400 Bad Request`
    - **Response Body:**
      ```json
      {
        "error": "Username và password không được để trống"
      }
      ```

---

### 2. Đăng nhập (Login)

Xác thực tài khoản và trả về `userid` để app client lưu trữ.

- **URL:** `/api/login`
- **Method:** `POST`
- **Request Body (JSON):**

```json
{
  "username": "hocsinh1",
  "password": "123"
}
```

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Đăng nhập thành công**
    - **HTTP Status Code:** `200 OK`
    - **Response Body:**
      ```json
      {
        "message": "Đăng nhập thành công",
        "userid": "a1b2"
      }
      ```

  - **TH 2: Sai tài khoản hoặc mật khẩu**
    - **HTTP Status Code:** `401 Unauthorized`
    - **Response Body:**
      ```json
      {
        "error": "Tài khoản hoặc mật khẩu không chính xác"
      }
      ```

---

### 3. Đăng ký Device Token (Register Device Token)

Mỗi khi app trên điện thoại khởi động hoặc nhận được Token thông báo mới từ hệ thống (FCM/APNs), app sẽ gửi Token này lên Server để liên kết với `userid`.

- **URL:** `/api/device-token`
- **Method:** `POST`
- **Request Body (JSON):**

```json
{
  "userid": "a1b2",
  "deviceToken": "fcm_token_string_abc123..."
}
```

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Cập nhật thành công**
    - **HTTP Status Code:** `200 OK`
    - **Response Body:**
      ```json
      {
        "message": "Cập nhật Device Token thành công"
      }
      ```

  - **TH 2: Không tìm thấy User**
    - **HTTP Status Code:** `404 Not Found`
    - **Response Body:**
      ```json
      {
        "error": "Không tìm thấy người dùng với userid đã cung cấp"
      }
      ```

---

### 4. Gửi Push Notification cho User (Send Push Notification)

Gửi thông báo tới một user cụ thể bằng `userid` của người nhận. Server sẽ tìm `deviceToken` tương ứng của user đó để gửi qua dịch vụ Push Notification (FCM).

- **URL:** `/api/send-notification`
- **Method:** `POST`
- **Request Body (JSON):**

```json
{
  "receiverId": "c3d4",
  "title": "Cảnh báo khẩn cấp",
  "body": "Phát hiện động vật nguy hiểm gần khu vực của bạn!"
}
```

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Gửi thông báo thành công**
    - **HTTP Status Code:** `200 OK`
    - **Response Body:**
      ```json
      {
        "message": "Đã gửi thông báo thành công"
      }
      ```

  - **TH 2: Không tìm thấy Device Token của người nhận**
    - **HTTP Status Code:** `404 Not Found`
    - **Response Body:**
      ```json
      {
        "error": "Người nhận chưa đăng ký Device Token để nhận thông báo"
      }
      ```

  - **TH 3: Thiếu thông tin gửi tin**
    - **HTTP Status Code:** `400 Bad Request`
    - **Response Body:**
      ```json
      {
        "error": "Thiếu các thông tin bắt buộc (receiverId, title, hoặc body)"
      }
      ```

---

### 5. Kiểm tra trạng thái máy chủ (Health Check)

Dùng để kiểm tra nhanh xem máy chủ backend có đang hoạt động bình thường hay không.

- **URL:** `/api/health`
- **Method:** `GET`
- **Request Body (JSON):** Không yêu cầu

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Máy chủ hoạt động bình thường**
    - **HTTP Status Code:** `200 OK`
    - **Response Body:**
      ```json
      {
        "status": "UP",
        "timestamp": "2026-07-13T09:10:00Z"
      }
      ```
