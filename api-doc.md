# Tài liệu API & Thiết kế Cơ sở Dữ liệu (Đơn giản cho học sinh)

Ứng dụng phục vụ nghiên cứu học tập với các chức năng cơ bản: Đăng ký, Đăng nhập, Đăng ký Device Token và Gửi Push Notification.

---

## I. Mô hình Cơ sở Dữ liệu (Database Models)

Để lưu trữ dữ liệu cho ứng dụng này, chỉ cần 2 bảng (hoặc collections) đơn giản sau:

### 1. Model `User` (Người dùng)

Lưu trữ thông tin tài khoản và danh sách các Device Token để nhận thông báo.

| Tên trường (Field) | Kiểu dữ liệu | Bắt buộc (Required?) | Mô tả                       | Ghi chú                                                       |
| :----------------- | :----------- | :------------------- | :-------------------------- | :------------------------------------------------------------ |
| `id`               | String       | Có (Required)        | Mã hex 4 ký tự              | Khóa chính (Ví dụ: `"a1b2"`, `"f3e4"`)                        |
| `username`         | String       | Có (Required)        | Tên tài khoản               | Duy nhất                                                      |
| `password`         | String       | Có (Required)        | Mật khẩu                    | Lưu mật khẩu gốc hoặc mã hóa đơn giản                         |
| `deviceTokens`     | [String]     | Không (Optional)     | Mảng các token của thiết bị | Lưu nhiều token nếu user đăng nhập trên nhiều thiết bị        |
| `createdAt`        | Date         | Có (Required)        | Ngày tạo                    | Tự động sinh                                                  |

> [!NOTE]
> **Lưu ý thiết kế Cơ sở dữ liệu quan hệ (SQL):**
> Nếu dự án sử dụng hệ quản trị cơ sở dữ liệu quan hệ (như MySQL, PostgreSQL, SQLite), việc lưu trữ một mảng `deviceTokens` trực tiếp trong bảng `User` là không tối ưu (vi phạm chuẩn hóa dữ liệu). Khi đó, bạn nên tách ra thành một bảng riêng biệt tên là `UserDevice` (hoặc `Device`) để quản lý mối quan hệ `1-N` giữa `User` và `Device Token` (gồm các trường: `id`, `userId`, `deviceToken`, `createdAt`).

### 2. Model `NotificationLog` (Lịch sử thông báo)

Bắt buộc lưu trữ lịch sử tất cả các thông báo đã gửi để phục vụ debug và xem lại lịch sử.

| Tên trường (Field) | Kiểu dữ liệu | Bắt buộc (Required?) | Mô tả                     | Ghi chú                                         |
| :----------------- | :----------- | :------------------- | :------------------------ | :---------------------------------------------- |
| `id`               | String       | Có (Required)        | ID thông báo              | Khóa chính                                      |
| `receiverId`       | String       | Có (Required)        | ID người nhận             | Liên kết với `User.id`                          |
| `title`            | String       | Có (Required)        | Tiêu đề thông báo         |                                                 |
| `body`             | String       | Có (Required)        | Nội dung thông báo        |                                                 |
| `status`           | String       | Có (Required)        | Trạng thái gửi            | Giá trị: `"SUCCESS"` (Thành công) hoặc `"FAILED"` (Thất bại) |
| `errorMessage`     | String       | Không (Optional)     | Lý do lỗi                 | Chỉ có khi trạng thái gửi là `"FAILED"`          |
| `sentAt`           | Date         | Có (Required)        | Thời gian gửi             | Tự động sinh                                    |

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

Mỗi khi app trên điện thoại khởi động hoặc nhận được Token thông báo mới từ hệ thống (FCM/APNs), app sẽ gửi Token này lên Server để liên kết với `userid`. Server sẽ lưu token mới này vào danh sách `deviceTokens` của người dùng (nếu chưa tồn tại).

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

Gửi thông báo tới một user cụ thể bằng `userid` của người nhận. Server sẽ tìm danh sách `deviceTokens` tương ứng của user đó để gửi qua dịch vụ Push Notification (FCM) tới tất cả các thiết bị đang kết nối. Đồng thời, **server sẽ tự động ghi lại lịch sử gửi tin vào bảng `NotificationLog`** (bao gồm trạng thái thành công hay thất bại).

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

  - **TH 2: Danh sách Device Token của người nhận trống**
    - **HTTP Status Code:** `404 Not Found`
    - **Response Body:**
      ```json
      {
        "error": "Người nhận chưa đăng ký bất kỳ Device Token nào để nhận thông báo"
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

### 5. Xem lịch sử gửi thông báo (View Notification Logs)

Lấy danh sách lịch sử các thông báo đã gửi trong hệ thống để theo dõi hoặc debug. Hỗ trợ lọc theo `receiverId` của người nhận và hỗ trợ phân trang bằng tham số `page` và `n`.

- **URL:** `/api/notifications?receiverId=c3d4&page=1&n=20`
- **Method:** `GET`
- **Query Parameters:**
  - `receiverId` (String, Optional): Lọc theo ID người nhận.
  - `page` (Integer, Optional): Số trang muốn xem. Mặc định: `1`.
  - `n` (Integer, Optional): Số bản ghi trên mỗi trang. Mặc định: `20`.
- **Request Body (JSON):** Không yêu cầu

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Lấy danh sách thành công**
    - **HTTP Status Code:** `200 OK`
    - **Response Body:**
      ```json
      {
        "page": 1,
        "n": 20,
        "total": 45,
        "data": [
          {
            "id": "notif_9876",
            "receiverId": "c3d4",
            "title": "Cảnh báo khẩn cấp",
            "body": "Phát hiện động vật nguy hiểm gần khu vực của bạn!",
            "status": "SUCCESS",
            "errorMessage": null,
            "sentAt": "2026-07-14T08:50:00Z"
          },
          {
            "id": "notif_9877",
            "receiverId": "c3d4",
            "title": "Cảnh báo rắn độc",
            "body": "Phát hiện rắn cỏ ở khu vực giảng đường.",
            "status": "FAILED",
            "errorMessage": "Không tìm thấy bất kỳ device token hoạt động nào",
            "sentAt": "2026-07-14T08:52:00Z"
          }
        ]
      }
      ```

---

### 6. Kiểm tra trạng thái máy chủ (Health Check)

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

---

### 7. Lấy danh sách tài khoản phục vụ phát triển (Dev User List)

Dùng để liệt kê danh sách tài khoản đã đăng ký trong hệ thống phục vụ việc debug, kiểm thử trong quá trình phát triển (chỉ hiển thị UserID, UserName và Thời gian tạo tài khoản).

- **URL:** `/api/dev/users`
- **Method:** `GET`
- **Request Body (JSON):** Không yêu cầu

- **Các trường hợp phản hồi (Responses):**
  - **TH 1: Lấy danh sách thành công**
    - **HTTP Status Code:** `200 OK`
    - **Response Body:**
      ```json
      [
        {
          "userid": "a1b2",
          "username": "hocsinh1",
          "createdAt": "2026-07-13T09:10:00Z"
        },
        {
          "userid": "c3d4",
          "username": "hocsinh2",
          "createdAt": "2026-07-13T09:12:00Z"
        }
      ]
      ```
