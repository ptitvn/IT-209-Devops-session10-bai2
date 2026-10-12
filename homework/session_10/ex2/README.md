# BÀI 2: ĐÓNG GÓI ỨNG DỤNG WEB BACKEND BẰNG SINGLE-STAGE DOCKERFILE

## 1. Thông Tin Bài Nộp
- **Học phần:** IT-209 - DevOps
- **Bài tập:** Session 10 - Bài 2: Single-stage Dockerfile cho Web Backend
- **Đường dẫn GitHub:** `homework/session_10/ex2/`
- **Kho lưu trữ:** `https://github.com/ptitvn/IT-209-Devops-session10-bai2.git`

---

## 2. Mục Tiêu & Yêu Cầu
- Viết tệp **Dockerfile Single-stage** chuẩn hóa để đóng gói ứng dụng Web Backend Node.js/Express thành Docker Image.
- Đảm bảo đầy đủ các chỉ thị Dockerfile bắt buộc:
  - `FROM`: Sử dụng base image `node:18-alpine`.
  - `WORKDIR`: Thư mục làm việc `/app`.
  - `ENV`: Khai báo biến môi trường `PORT=3000`.
  - `COPY`: Sao chép file định nghĩa dependency và mã nguồn ứng dụng.
  - `RUN`: Cài đặt thư viện dependencies (`npm install`).
  - `EXPOSE`: Khai báo cổng `3000`.
  - `CMD`: Khởi chạy ứng dụng dưới dạng Exec form `["node", "server.js"]`.
- Xây dựng image `hw-backend:v1` và triển khai container `app-single-stage` ánh xạ cổng `8080:3000`.
- Kiểm thử container và gọi API bằng `curl`.

---

## 3. Cấu Trúc Thư Mục Dự Án
```text
homework/
└── session_10/
    └── ex2/
        ├── Dockerfile                     # Single-stage Dockerfile cho ứng dụng
        ├── package.json                   # Định nghĩa dependencies (express)
        ├── server.js                      # Mã nguồn ứng dụng REST API Express
        └── README.md                      # Báo cáo và hướng dẫn thực thi
```

---

## 4. Chi Tiết Mã Nguồn & Dockerfile

### 4.1. Mã nguồn `package.json`
```json
{
  "name": "ex2-backend",
  "version": "1.0.0",
  "description": "Single-stage Dockerfile Web Backend",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### 4.2. Mã nguồn `server.js`
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.json({
        status: 'success',
        message: 'Hello from Docker Single-stage!'
    });
});

app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}`);
});
```

### 4.3. Nội dung `Dockerfile`
```dockerfile
# Base Image: node:18-alpine nhẹ và tối ưu cho môi trường container
FROM node:18-alpine

# Thiết lập thư mục làm việc bên trong container
WORKDIR /app

# Thiết lập biến môi trường PORT
ENV PORT=3000

# Copy file định nghĩa phụ thuộc để tối ưu layer caching
COPY package*.json ./

# Cài đặt các dependencies
RUN npm install

# Copy toàn bộ mã nguồn vào container
COPY . .

# Khai báo port container lắng nghe
EXPOSE 3000

# Lệnh khởi chạy ứng dụng (Exec form)
CMD ["node", "server.js"]
```

### 4.4. Giải thích các chỉ thị Dockerfile
| Chỉ thị | Cú pháp | Ý nghĩa & Vai trò kỹ thuật |
| :--- | :--- | :--- |
| **`FROM`** | `FROM node:18-alpine` | Sử dụng base image Node.js 18 phiên bản Alpine Linux siêu nhẹ (~170MB), giảm thiểu bề mặt tấn công bảo mật và tăng tốc độ tải/build. |
| **`WORKDIR`** | `WORKDIR /app` | Thiết lập thư mục làm việc mặc định trong container. Các lệnh tiếp theo (`COPY`, `RUN`, `CMD`) sẽ thực thi tương đối với thư mục này. |
| **`ENV`** | `ENV PORT=3000` | Khai báo biến môi trường trong runtime container, ứng dụng Express sẽ đọc giá trị này qua `process.env.PORT`. |
| **`COPY`** | `COPY package*.json ./` | Sao chép riêng tệp phụ thuộc trước khi copy toàn bộ mã nguồn để tận dụng **Docker Layer Caching**. Nếu mã nguồn thay đổi nhưng `package.json` không đổi, bước `npm install` sẽ dùng cache giúp build cực nhanh. |
| **`RUN`** | `RUN npm install` | Chạy lệnh cài đặt các gói phụ thuộc (Express) vào thư mục `/app/node_modules` bên trong container trong quá trình build image. |
| **`COPY`** | `COPY . .` | Sao chép toàn bộ mã nguồn còn lại của ứng dụng vào image. |
| **`EXPOSE`** | `EXPOSE 3000` | Tài liệu hóa cổng mạng mà ứng dụng lắng nghe bên trong container (metadata). |
| **`CMD`** | `CMD ["node", "server.js"]` | Lệnh mặc định được thực thi khi container khởi động. Dùng **Exec form** (dạng mảng JSON) giúp tiến trình nhận trực tiếp tín hiệu `SIGTERM` / `SIGINT` từ Docker daemon để shutdown an toàn. |

---

## 5. Hướng Dẫn Thực Thi & Kiểm Thử

### Bước 1: Di chuyển vào thư mục bài tập & Build Docker Image
```bash
cd homework/session_10/ex2
docker build -t hw-backend:v1 .
```

### Bước 2: Khởi chạy Container
```bash
docker run -d --name app-single-stage -p 8080:3000 hw-backend:v1
```

### Bước 3: Kiểm tra trạng thái Container
```bash
docker ps
```

### Bước 4: Kiểm tra Logs của Container
```bash
docker logs app-single-stage
```

### Bước 5: Kiểm tra Endpoint API qua cURL
```bash
curl -i http://localhost:8080/
```

---

## 6. Kết Quả Kiểm Thử Thực Tế

### 6.1. Build Image thành công (`hw-backend:v1`)
```text
#0 building with "desktop-linux" instance using docker driver
#1 [internal] load build definition from Dockerfile
#2 [internal] load metadata for docker.io/library/node:18-alpine
#3 [1/5] FROM docker.io/library/node:18-alpine
#4 [2/5] WORKDIR /app
#5 [3/5] COPY package*.json ./
#6 [4/5] RUN npm install
#6 added 68 packages, and audited 69 packages in 3s
#7 [5/5] COPY . .
#8 naming to docker.io/library/hw-backend:v1 done
```

### 6.2. Trạng thái Container (`docker ps`)
```text
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                                         NAMES
a43fac79defd   hw-backend:v1   "docker-entrypoint.s…"   2 seconds ago   Up 2 seconds   0.0.0.0:8080->3000/tcp, [::]:8080->3000/tcp   app-single-stage
```

### 6.3. Logs Container (`docker logs app-single-stage`)
```text
Server listening on port 3000
```

### 6.4. Kết quả phản hồi HTTP cURL (`curl -i http://localhost:8080/`)
```http
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 64
ETag: W/"40-upZb7Fps+hszzRU9BoKtiCsmGCQ"
Date: Mon, 12 Oct 2026 08:56:32 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"status":"success","message":"Hello from Docker Single-stage!"}
```

---

## 7. Đánh Giá Kỹ Thuật Single-stage Build

### Ưu điểm
- **Đơn giản & Dễ tiếp cận:** Cấu trúc Dockerfile ngắn gọn, dễ hiểu và dễ bảo trì cho các dự án nhỏ hoặc thử nghiệm ban đầu.
- **Tối ưu Layer Caching:** Tách riêng lệnh `COPY package*.json` và `RUN npm install` giúp tránh cài đặt lại dependencies khi chỉ sửa mã nguồn logic (`server.js`).
- **Nhanh gọn:** Thời gian build nhanh chóng do không cần trung gian nhiều stage.

### Nhược điểm & Hạn chế so với Multi-stage
- **Kích thước Image lớn hơn:** Single-stage lưu giữ toàn bộ build tools, npm cache và các công cụ phát triển không cần thiết cho môi trường production.
- **Rủi ro bảo mật:** Sự xuất hiện của npm runtime package manager và các công cụ build trong container production có thể làm tăng bề mặt tấn công so với distroless hoặc multi-stage minimal runtime image.
