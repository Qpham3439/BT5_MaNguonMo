# BT5_MaNguonMo
Bài tập 05 của sinh viên: K225480106057 - Phạm Mạnh Quỳnh - môn Phát triển ứng dụng với mã nguồn mở

# BÀI TẬP 5 – APP MONITOR + ALERT DATA REALTIME

## Yêu cầu

### Phần 1. Lý thuyết

* Docker là gì.
* Các keyword trong `docker-compose.yml` dùng để mô tả:

  * service
  * network
  * volume
  * …

  Trình bày:

  * Ý nghĩa
  * Cú pháp
  * Ví dụ minh hoạ
* Ưu điểm khi triển khai ứng dụng bằng Docker.
* Quy trình triển khai ứng dụng Docker từ máy cá nhân lên máy chủ không có Internet.

---

### Phần 2. Thực hành

Xây dựng hệ thống **APP MONITOR + ALERT DATA REALTIME** bằng **Docker Compose** gồm nhiều service:

* **Node-RED**

  * Lấy dữ liệu realtime từ nguồn thực tế (thời tiết, chứng khoán, giá vàng,...).

* **MariaDB**

  * Lưu dữ liệu tức thời.

* **InfluxDB**

  * Lưu dữ liệu lịch sử.

* **Grafana**

  * Trực quan hoá và hiển thị biểu đồ.

* **Nginx + Frontend (HTML/CSS/JS)**

  * Hiển thị dữ liệu realtime.
  * Gọi API Flask bằng AJAX hoặc Socket.
  * Nhúng dashboard Grafana bằng iframe.

* **Flask API**

  * Truy vấn dữ liệu từ MariaDB.

* **Telegram Bot**

  * Gửi cảnh báo khi dữ liệu vượt ngưỡng:

    * `< A`: ALERT LOW
    * `A → B`: NORMAL
    * `> B`: ALERT HIGH

* Xuất toàn bộ container ra file nén.

* Xoá toàn bộ container đang chạy.

* Khôi phục hệ thống từ file đã xuất.

# PHẦN 1. LÝ THUYẾT

## 1. Docker là gì?

Docker là nền tảng hỗ trợ đóng gói, triển khai và chạy ứng dụng dưới dạng **container**.

Container là môi trường thực thi độc lập, chứa đầy đủ:

* Mã nguồn ứng dụng
* Thư viện cần thiết
* Runtime
* Công cụ hệ thống
* Cấu hình môi trường

Nhờ vậy ứng dụng có thể chạy giống nhau trên nhiều máy khác nhau mà không bị lỗi do khác môi trường.

Ví dụ:

Máy cá nhân:

```text
Windows + Python 3.12
```

Máy chủ:

```text
Ubuntu + Python 3.12
```

Nếu triển khai bằng Docker thì ứng dụng sẽ hoạt động giống nhau trên cả hai môi trường.

### Một số thành phần trong Docker

| Thành phần     | Ý nghĩa                               |
| -------------- | ------------------------------------- |
| Docker Engine  | Dịch vụ chạy container                |
| Image          | Mẫu tạo container                     |
| Container      | Phiên bản đang chạy của image         |
| Dockerfile     | File mô tả cách tạo image             |
| Docker Compose | Công cụ chạy nhiều container cùng lúc |
| Volume         | Lưu dữ liệu lâu dài                   |
| Network        | Giao tiếp giữa các container          |

---

## 2. Docker Compose là gì?

Docker Compose là công cụ giúp quản lý và triển khai nhiều container cùng lúc bằng một file cấu hình.

File cấu hình thường có tên:

```text
docker-compose.yml
```

Ví dụ:

Một ứng dụng có:

* Frontend
* API
* Database

Thay vì chạy:

```bash
docker run ...
docker run ...
docker run ...
```

chỉ cần:

```bash
docker compose up -d
```

Docker sẽ tự tạo toàn bộ hệ thống.

Ví dụ đơn giản:

```yaml
services:

  web:
    image: nginx
    ports:
      - "80:80"

  db:
    image: mariadb
```

---

## 3. Các keyword thường dùng trong docker-compose.yml

### 3.1 services

Khai báo các container sẽ chạy.

Ví dụ:

```yaml
services:
  api:
  nginx:
```

---

### 3.2 image

Chỉ định image sử dụng.

Ví dụ:

```yaml
image: nginx
```

Docker sẽ tải image nginx.

---

### 3.3 build

Build image từ Dockerfile.

Ví dụ:

```yaml
build: ./api
```

Docker sẽ tìm:

```text
./api/Dockerfile
```

---

### 3.4 container_name

Đặt tên container.

Ví dụ:

```yaml
container_name: flask_api
```

---

### 3.5 ports

Ánh xạ cổng.

Cú pháp:

```text
HOST:CONTAINER
```

Ví dụ:

```yaml
ports:
 - "8080:80"
```

Truy cập:

```text
localhost:8080
```

---

### 3.6 environment

Khai báo biến môi trường.

Ví dụ:

```yaml
environment:
 MYSQL_ROOT_PASSWORD: admin
```

---

### 3.7 volumes

Lưu dữ liệu lâu dài.

Ví dụ:

```yaml
volumes:
 - db_data:/var/lib/mysql
```

Khi container bị xoá dữ liệu vẫn còn.

---

### 3.8 networks

Cho phép container giao tiếp.

Ví dụ:

```yaml
networks:
 monitor_net:
```

Gắn:

```yaml
networks:
 - monitor_net
```

---

### 3.9 depends_on

Thiết lập thứ tự khởi động.

Ví dụ:

```yaml
depends_on:
 - mariadb
```

Container hiện tại chạy sau mariadb.

---

### 3.10 restart

Chính sách tự khởi động lại.

Ví dụ:

```yaml
restart: always
```

Các chế độ:

```text
no
always
unless-stopped
on-failure
```

---

### 3.11 command

Ghi đè lệnh chạy mặc định.

Ví dụ:

```yaml
command: python app.py
```

---

### 3.12 expose

Cho phép container khác truy cập cổng nội bộ.

Ví dụ:

```yaml
expose:
 - 5000
```

---

### Ví dụ docker-compose hoàn chỉnh

```yaml
version: "3.9"

services:

 api:
  build: ./api
  container_name: flask_api

  ports:
   - "5000:5000"

  environment:
   DB_HOST: mariadb

  depends_on:
   - mariadb

  networks:
   - monitor_net

 mariadb:
  image: mariadb

  environment:
   MYSQL_ROOT_PASSWORD: admin

  volumes:
   - db_data:/var/lib/mysql

  networks:
   - monitor_net

networks:
 monitor_net:

volumes:
 db_data:
```

---

## 4. Ưu điểm khi triển khai ứng dụng bằng Docker

### 4.1 Môi trường đồng nhất

Ứng dụng chạy giống nhau ở mọi máy.

---

### 4.2 Triển khai nhanh

Có thể chạy toàn bộ hệ thống bằng:

```bash
docker compose up -d
```

---

### 4.3 Dễ mở rộng

Có thể tăng số lượng container.

Ví dụ:

```bash
docker compose up --scale api=3
```

---

### 4.4 Dễ sao lưu và phục hồi

Container và image có thể export/import.

---

### 4.5 Tiết kiệm tài nguyên

Container dùng chung kernel hệ điều hành nên nhẹ hơn máy ảo.

---

### 4.6 Dễ bảo trì

Mỗi thành phần chạy độc lập.

Ví dụ:

```text
Frontend
API
Database
```

---

## 5. Triển khai ứng dụng Docker lên máy chủ KHÔNG có Internet

Giả sử ứng dụng đã chạy ổn định trên laptop.

### Bước 1. Build image

```bash
docker compose build
```

Kiểm tra:

```bash
docker images
```

---

### Bước 2. Export image

Ví dụ:

```bash
docker save monitor_api > api.tar
docker save monitor_nginx > nginx.tar
docker save monitor_db > db.tar
```

Gộp:

```bash
tar -cvf all_images.tar *.tar
```

---

### Bước 3. Copy sang máy chủ

Có thể dùng:

```text
USB
LAN
Ổ cứng ngoài
```

---

### Bước 4. Import image

Giải nén:

```bash
tar -xvf all_images.tar
```

Load:

```bash
docker load < api.tar
docker load < nginx.tar
docker load < db.tar
```

---

### Bước 5. Copy source và docker-compose

Chép:

```text
docker-compose.yml
.env
frontend/
api/
```

---

### Bước 6. Khởi động hệ thống

```bash
docker compose up -d
```

Kiểm tra:

```bash
docker ps
```

---

### Bước 7. Kiểm tra hoạt động

Kiểm tra:

```bash
docker logs

docker ps
```

Truy cập:

```text
http://IP_MAY_CHU
```

Nếu truy cập thành công thì quá trình triển khai hoàn tất.
