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
