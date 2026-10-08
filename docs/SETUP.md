# Cài đặt và chạy local

Các lệnh dưới đây dùng PowerShell, bắt đầu từ thư mục gốc của repository. Mở terminal riêng cho mỗi dịch vụ.

## 1. Chuẩn bị

- .NET SDK 10.
- Node.js đáp ứng yêu cầu của Vite 8 và npm.
- Python có thể cài các phiên bản trong `AI_Service/requirements.txt`. Có thể bắt đầu với Python 3.11; chưa kiểm thử cài mới trong đợt chỉnh tài liệu này.
- SQL Server và ODBC Driver 17 for SQL Server (tên driver đang được khai báo trong Python).
- Hai file trọng số: `AI_Service/weights/model_xquang_phoi.pth` và `AI_Service/weights/model_xray_final.h5`.

Chạy npm trong `meddiag-frontend`, nơi chứa ứng dụng Vite; `package.json` ở thư mục gốc không có lệnh khởi chạy giao diện.

## 2. Cấu hình API và database

Tạo hoặc cập nhật `MedicalDiagnosis.API/appsettings.Development.json` với cấu hình local. Nếu file đã tồn tại, giữ các thiết lập đang dùng và chỉ sửa giá trị cần thiết:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SQL_SERVER;Database=MedicalDiagnosisDB;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Jwt": {
    "Key": "REPLACE_WITH_A_RANDOM_LOCAL_SECRET_AT_LEAST_32_BYTES",
    "Issuer": "MedicalDiagnosis",
    "Audience": "MedicalDiagnosis"
  }
}
```

Thay tên SQL Server và JWT key bằng giá trị riêng của bạn. Không commit cấu hình chứa bí mật.

Khôi phục dependency:

```powershell
dotnet restore MedicalDiagnosis.slnx
```

Nếu chưa có công cụ EF Core:

```powershell
dotnet tool install --global dotnet-ef --version 10.0.5
```

Áp dụng migrations vào database local dùng cho demo:

```powershell
dotnet ef database update --project MedicalDiagnosis.Infrastructure --startup-project MedicalDiagnosis.API -- --environment Development
```

Database có dữ liệu seed trong `MedicalDiagnosis.Infrastructure/Data/AppDbContext.cs` và các migrations. Kiểm tra tài khoản demo theo mã nguồn; không sử dụng dữ liệu cá nhân thật. Các script trong `Database/` phục vụ kiểm tra/tối ưu, không thay thế migrations.

Chạy API:

```powershell
dotnet run --project MedicalDiagnosis.API --launch-profile http
```

Swagger: http://localhost:5255/swagger.

## 3. Cấu hình AI Service

Trong `AI_Service/main.py`, cập nhật:

- `WWWROOT_PATH`: đường dẫn tuyệt đối tới `MedicalDiagnosis.API/wwwroot` trên máy bạn.
- `CONNECTION_STRING`: đúng SQL Server, database và driver ODBC. Database phải trùng với API.

Hai giá trị trên hiện được khai báo trực tiếp trong code, chưa đọc từ `.env`.

Đặt trọng số đúng tên trong `AI_Service/weights/`. Bộ lọc cũng đọc `weights/class_names.json` nếu có; file này phải chứa danh sách nhãn đúng thứ tự khi huấn luyện. Nếu thiếu, code dùng `anh_thuong`, `khong_phai_phoi`, `phoi`; cần xác nhận thứ tự này phù hợp trọng số của bạn.

```powershell
cd AI_Service
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Nếu sử dụng phản hồi chat Gemini, sao chép `.env.example` thành `.env` rồi điền API key riêng:

```powershell
Copy-Item .env.example .env
```

Không có key thì endpoint chat AI sẽ báo thiếu cấu hình. Không đưa key lên GitHub.

Khởi chạy từ chính thư mục `AI_Service` vì đường dẫn trọng số là đường dẫn tương đối:

```powershell
.\.venv\Scripts\python.exe -m uvicorn main:app --reload --port 8000
```

FastAPI docs: http://localhost:8000/docs. Xem log để xác nhận cả hai mô hình đã tải; phản hồi `GET /` chưa đủ xác nhận AI sẵn sàng.

## 4. Chạy giao diện

Từ thư mục gốc, trong terminal khác:

```powershell
cd meddiag-frontend
npm ci
npm run dev -- --port 5173
```

Mở http://localhost:5173. Frontend hiện gọi API tại `http://localhost:5255`; CORS của API cho phép `http://localhost:5173`. Giữ đúng cổng khi chạy theo hướng dẫn này.

Kiểm tra riêng frontend:

```powershell
npm run lint
npm run build
```

## 5. Kiểm tra thủ công

1. Mở Swagger, FastAPI docs và giao diện.
2. Đăng ký tài khoản bệnh nhân, đăng nhập và kiểm tra phân quyền.
3. Tải ảnh thử được phép sử dụng; kiểm tra kết quả, log AI và ảnh heatmap.
4. Dùng tài khoản quản trị/bác sĩ để kiểm tra phân công ca và ghi nhận chẩn đoán.
5. Kiểm tra đặt lịch, nhắn tin giữa hai tài khoản và thông báo.

Đây là các bước kiểm tra đề xuất, chưa phải báo cáo kiểm thử đã đạt.

## Lỗi thường gặp

| Triệu chứng | Kiểm tra |
| --- | --- |
| AI không khởi động | Dependency Python, driver ODBC và file trọng số của bộ lọc |
| Không tìm thấy ảnh | `WWWROOT_PATH`, đường dẫn lưu trong database và thư mục uploads |
| Không kết nối SQL Server | Tên instance, quyền Windows Authentication và database ở cả API/Python |
| Giao diện không gọi được API | API đã chạy ở cổng 5255; giao diện ở cổng 5173 |
| Không có phản hồi chat AI | `GEMINI_API_KEY`, kết nối mạng và phản hồi từ dịch vụ Gemini |
| Không có heatmap | Log tải model và Grad-CAM; API còn chạy không đồng nghĩa suy luận thành công |
