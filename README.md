# LoPet InternFE

LoPet InternFE là giao diện người dùng cho mạng xã hội thú cưng LoPet, được xây dựng bằng Vue 3 và Vite. Ứng dụng hỗ trợ các luồng đăng nhập, đăng ký, đặt lại mật khẩu, bảng tin bài viết, hồ sơ cá nhân, kết bạn, nhóm cộng đồng, nhắn tin, thông báo, báo cáo vi phạm và trang quản trị.

## Công nghệ sử dụng

- Vue 3
- Vite
- Vue Router
- Axios
- Socket.IO Client
- Toastify / Vue3 Toastify
- ESLint, Prettier
- Vitest

## Chức năng chính

- Xác thực người dùng: đăng ký, đăng nhập, đăng xuất, gửi và xác thực OTP, đặt lại mật khẩu.
- Bảng tin: xem danh sách bài viết, tạo bài viết có file upload, cập nhật, xóa, like và unlike.
- Hồ sơ cá nhân: tạo profile, xem profile theo tài khoản, cập nhật thông tin, ảnh đại diện và ảnh bìa.
- Bạn bè: danh sách bạn bè, lời mời kết bạn, gợi ý bạn bè, gửi/chấp nhận/từ chối lời mời, xóa bạn.
- Cộng đồng: tạo nhóm, xem nhóm đã tạo/đã tham gia/gợi ý, tham gia nhóm, rời nhóm, cập nhật và xóa nhóm.
- Bài viết trong nhóm: tạo và xem bài viết theo nhóm.
- Bình luận: xem bình luận theo bài viết và tạo bình luận.
- Nhắn tin: xem danh sách tin nhắn, gửi tin nhắn dạng multipart, xem chi tiết và cập nhật trạng thái.
- Thông báo: lấy danh sách thông báo, tạo thông báo và cập nhật trạng thái.
- Báo cáo vi phạm: tạo báo cáo, xem danh sách báo cáo và cập nhật trạng thái xử lý.
- Quản trị: quản lý người dùng, báo cáo và quảng cáo.

## Cấu trúc thư mục

```text
LoPet_InternFE/
├── public/                 # Ảnh, icon và tài nguyên tĩnh
├── src/
│   ├── api/                # Cấu hình API và axios instance
│   ├── assets/             # Tài nguyên dùng trong component
│   ├── components/         # Component giao diện chính
│   │   └── admin/          # Component khu vực quản trị
│   ├── layout/             # Màn hình đăng nhập, đăng ký, reset password, nhóm
│   ├── router/             # Cấu hình Vue Router
│   ├── service/            # Các service gọi API backend
│   │   └── admin/          # Service cho chức năng admin
│   ├── App.vue
│   ├── main.js
│   └── socket.js           # Cấu hình kết nối Socket.IO
├── index.html
├── vite.config.js
├── vitest.config.js
├── eslint.config.js
└── package.json
```

## Yêu cầu môi trường

- Node.js
- npm
- Backend LoPet đang chạy và cung cấp API tương ứng với các endpoint `/v1/...`

## Cài đặt

```sh
npm install
```

## Cấu hình môi trường

Tạo hoặc cập nhật file `.env` ở thư mục gốc dự án:

```env
PORT=5173
VITE_BACKEND_API=http://localhost:8080
```

Trong đó:

- `VITE_BACKEND_API`: URL backend được dùng làm `baseURL` cho toàn bộ request axios.
- `PORT`: cổng mong muốn khi chạy môi trường phát triển. Vite mặc định chạy ở `5173`.

Lưu ý: frontend tự đọc `accessToken` từ `localStorage` và gắn vào header `Authorization: Bearer <token>` cho các request sau khi đăng nhập.

## Chạy dự án

Chạy môi trường phát triển:

```sh
npm run dev
```

Sau đó mở địa chỉ Vite hiển thị trên terminal, thường là:

```text
http://localhost:5173
```

Build production:

```sh
npm run build
```

Xem thử bản build:

```sh
npm run preview
```

## Kiểm tra và định dạng mã nguồn

Chạy unit test bằng Vitest:

```sh
npm run test:unit
```

Chạy ESLint và tự động sửa lỗi có thể sửa:

```sh
npm run lint
```

Định dạng mã nguồn trong thư mục `src/`:

```sh
npm run format
```

## Route chính

| Route | Chức năng |
| --- | --- |
| `/` | Chuyển hướng đến `/login` |
| `/login` | Đăng nhập |
| `/register` | Đăng ký |
| `/resetPassword` | Đặt lại mật khẩu |
| `/setNewPassword` | Thiết lập mật khẩu mới |
| `/verificationCode` | Xác thực mã OTP |
| `/resetNewPassword` | Đặt lại mật khẩu mới |
| `/home` | Trang chủ / bảng tin |
| `/create-post` | Tạo bài viết |
| `/friend` | Bạn bè |
| `/message` | Nhắn tin |
| `/profile/me` | Hồ sơ cá nhân |
| `/profile/:accountId` | Hồ sơ người dùng khác |
| `/about` | Thông tin hồ sơ |
| `/photo` | Thư viện ảnh |
| `/edit` | Chỉnh sửa hồ sơ |
| `/groups` | Cộng đồng |
| `/groupjoin/:id` | Nhóm đã tham gia |
| `/groupjoins/:id` | Trang nhóm |
| `/admin/denounce` | Quản lý báo cáo |
| `/admin/user` | Quản lý người dùng |
| `/admin/advertisement` | Quản lý quảng cáo |

## Ghi chú phát triển

- API được cấu hình tại `src/api/apiService.js`.
- URL backend lấy từ `src/api/config.js` thông qua biến môi trường `VITE_BACKEND_API`.
- Nếu backend trả về `401` hoặc `403`, frontend sẽ xóa token trong `localStorage` và chuyển người dùng về `/login`.
- Một số request upload dùng `multipart/form-data`, ví dụ tạo/cập nhật bài viết, profile, nhóm, tin nhắn và quảng cáo.
- `src/socket.js` đang cấu hình kết nối Socket.IO đến `http://localhost:8080` và gửi `userId` cùng `accessToken` khi kết nối.
