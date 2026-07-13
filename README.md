# Wild Animal Alert - Dự án Nghiên cứu & Học tập

Dự án **Wild Animal Alert** là ứng dụng cảnh báo động vật hoang dã được thiết kế phục vụ cho mục đích nghiên cứu, học tập của học sinh. Ứng dụng giúp gửi các thông báo khẩn cấp (Push Notification) đến thiết bị người dùng khi phát hiện các tình huống động vật hoang dã nguy hiểm xuất hiện.

---

## 🌟 Các chức năng cốt lõi

1. **Quản lý Tài khoản (User Management):** Đăng ký và đăng nhập tài khoản học sinh/người dùng đơn giản chỉ với tên đăng nhập (username) và mật khẩu (password).
2. **Đăng ký Device Token:** Nhận diện và đăng ký token thiết bị để chuẩn bị cho việc nhận thông báo từ xa.
3. **Gửi Push Notification:** Gửi tin nhắn cảnh báo trực tiếp từ máy chủ đến một thiết bị cụ thể thông qua `userid` người nhận.

---

## 📁 Cấu trúc thư mục dự án

```text
wild-animal-alert/
├── android/             # Mã nguồn ứng dụng Android (Client)
├── api/                 # Mã nguồn backend máy chủ (Server API)
├── README.md            # Tài liệu giới thiệu tổng quan dự án
└── api-doc.md           # Thiết kế cơ sở dữ liệu & tài liệu đặc tả API chi tiết
```

---

## 📖 Tài liệu liên quan

* Để xem đặc tả chi tiết về các Endpoint API (URL, Method, Request/Response, HTTP Status Code) cũng như cấu trúc bảng cơ sở dữ liệu (`User`, `Notification`), vui lòng tham khảo:
  👉 **[Tài liệu đặc tả API & Database Models (api-doc.md)](file:///Users/trungnguyen/GitHub/wild-animal-alert/api-doc.md)**
