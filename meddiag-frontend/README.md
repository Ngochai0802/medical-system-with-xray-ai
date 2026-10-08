# Medical Diagnosis — Frontend

Giao diện React cho ba vai trò: bệnh nhân, bác sĩ và quản trị viên.

## Chạy local

Trong thư mục này:

```powershell
npm ci
npm run dev -- --port 5173
```

Ứng dụng gọi API tại `http://localhost:5255`. Xem [hướng dẫn cài đặt toàn hệ thống](../docs/SETUP.md) trước khi kiểm tra các chức năng cần backend hoặc AI.

## Lệnh

| Lệnh | Mục đích |
| --- | --- |
| `npm run dev` | Chạy Vite development server |
| `npm run build` | Tạo bản build trong `dist/` |
| `npm run lint` | Kiểm tra ESLint |
| `npm run preview` | Xem thử bản build |

## Cấu trúc

- `src/pages/`: màn hình theo vai trò và xác thực.
- `src/components/common/`: layout, bảo vệ route và thành phần dùng chung.
- `src/api/`: Axios client và các hàm gọi API.
- `src/store/`: trạng thái đăng nhập và thông báo bằng Zustand.
- `src/hooks/`: kết nối SignalR.

Thông tin dự án và kiến trúc: [README chính](../README.md).
