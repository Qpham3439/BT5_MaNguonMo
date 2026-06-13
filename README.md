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

# PHẦN 2: QUY TRÌNH THỰC HIỆN ỨNG DỤNG MONITOR & ALERT DATA REAL-TIME

Hệ thống tiến hành giám sát dữ liệu **Nhiệt độ biến động liên tục theo thời gian thực** từ nguồn API. Hệ thống sở hữu cơ chế lưu trữ song song (**MariaDB lưu giá trị tức thời, InfluxDB lưu lịch sử**), trực quan hóa dữ liệu bằng **Grafana**, cung cấp giao diện người dùng bằng **Nginx + Flask API** và tự động gửi cảnh báo vượt ngưỡng an toàn về **nhóm Telegram gồm 3 thành viên**.

---

# Bước 1: Chuẩn bị cấu trúc thư mục dự án và các tệp mã nguồn

Di chuyển vào thư mục dự án:

```bash
cd ~/ma
```

Cấu trúc thư mục:

```text
~/ma/
├── docker-compose.yml
├── init.sql
├── flask-api/
│   ├── app.py
│   └── requirements.txt
└── nginx/
    ├── default.conf
    └── web/
        └── index.html
```

Tạo nhanh:

```bash
mkdir -p ~/ma/flask-api ~/ma/nginx/web
touch ~/ma/docker-compose.yml
touch ~/ma/init.sql
touch ~/ma/flask-api/app.py
touch ~/ma/flask-api/requirements.txt
touch ~/ma/nginx/default.conf
touch ~/ma/nginx/web/index.html
```

### File docker-compose.yml
```
version: '3.8'

services:
  # 1. Node-RED: Lấy dữ liệu, lưu DB, Alert Telegram
  nodered:
    image: nodered/node-red:latest
    container_name: app_nodered
    user: "root" # Thêm dòng này để tránh lỗi quyền ghi dữ liệu (Permission Denied)
    ports:
      - "1880:1880"
    volumes:
      - nodered_data:/data
    networks:
      - monitor_net
    restart: always

  # 2. MariaDB: Lưu dữ liệu tức thời
  mariadb:
    image: mariadb:10.6
    container_name: app_mariadb
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: monitor_db
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro # Gắn file khởi tạo tĩnh ở đây (Chỉ đọc)
    networks:
      - monitor_net
    restart: always

  # 3. InfluxDB: Lưu dữ liệu lịch sử (Time-series)
  influxdb:
    image: influxdb:1.8
    container_name: app_influxdb
    ports:
      - "8086:8086"
    environment:
      INFLUXDB_DB: history_db
      INFLUXDB_DATA_ENGINE: tsm1 # Đảm bảo engine chạy nhẹ, không lỗi vặt
    volumes:
      - influxdb_data:/var/lib/influxdb
    networks:
      - monitor_net
    restart: always

  # 4. Grafana: Vẽ biểu đồ lịch sử
  grafana:
    image: grafana/grafana:latest
    container_name: app_grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ALLOW_EMBEDDING: "true"
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_ANONYMOUS_ORG_ROLE: Viewer
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitor_net
    restart: always

  # 5. Flask API: Cầu nối lấy dữ liệu từ MariaDB cho Front-end
  flask-api:
    image: python:3.9-slim
    container_name: app_flask
    working_dir: /app
    volumes:
      - ./flask-api:/app
    command: sh -c "pip install -r requirements.txt && python app.py"
    ports:
      - "5000:5000"
    networks:
      - monitor_net
    depends_on:
      - mariadb
    restart: always

  # 6. Nginx: Webserver chứa Front-end giao diện chính
  nginx:
    image: nginx:alpine
    container_name: app_nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./nginx/web:/usr/share/nginx/html
    networks:
      - monitor_net
    depends_on:
      - flask-api
    restart: always

networks:
  monitor_net:
    driver: bridge

volumes:
  nodered_data:
  mariadb_data:
  influxdb_data:
  grafana_data:
```
### File init.sql

```sql
USE monitor_db;

CREATE TABLE IF NOT EXISTS real_time_data(
    id INT AUTO_INCREMENT PRIMARY KEY,
    value INT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### File default.conf

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    # Proxy các request API sang container Flask tránh lỗi CORS nếu cần
    location /api/ {
        proxy_pass http://flask-api:5000/api/;
    }
}
```

---

# Bước 2: Khởi chạy cụm dịch vụ và kiểm tra sức khỏe hệ thống

Khởi động:

```bash
cd ~/ma
sudo docker compose up -d
```

Kiểm tra:

```bash
sudo docker ps
```

Yêu cầu:

Các container:

- app_nodered
- app_mariadb
- app_influxdb
- app_grafana
- app_flask
- app_nginx

đều ở trạng thái **Up**.

<img width="1918" height="857" alt="Screenshot 2026-06-13 140938" src="https://github.com/user-attachments/assets/b1b11c1f-56a2-4da2-b115-1ce7e8d9889c" />

---

Kiểm tra database:

```bash
sudo docker exec -it app_mariadb bash
mysql -u user -p
```

```sql
USE monitor_db;
SHOW TABLES;
SELECT * FROM real_time_data;
```

<img width="1918" height="572" alt="Screenshot 2026-06-13 142431" src="https://github.com/user-attachments/assets/a494810c-e12f-4fbd-8bad-60fd4c8bf708" />


---

# Bước 3: Thiết lập luồng xử lý và Cảnh báo dữ liệu trên Node-RED

Truy cập:

```text
http://192.168.56.14:1880
```

Thiết kế Flow:

<img width="1918" height="892" alt="Screenshot 2026-06-13 165740" src="https://github.com/user-attachments/assets/49c11d95-43cd-4a69-be09-def3eeea6221" />

---

## Lấy dữ liệu nhiệt độ

Node HTTP Request:

```text
https://api.open-meteo.com/v1/forecast?latitude=21.03&longitude=105.85&current=temperature_2m
```

Thiết lập:

Method:

```text
GET
```

Return:

```text
parsed JSON
```
<img width="1917" height="980" alt="image" src="https://github.com/user-attachments/assets/58f7449f-1945-4dd9-9edc-4ab43f8f9cf2" />

---

## Lưu MariaDB

Function:

```javascript
msg.topic =
"INSERT INTO real_time_data(value) VALUES ("+
msg.payload.current.temperature_2m+
")";

return msg;
```
<img width="1918" height="950" alt="image" src="https://github.com/user-attachments/assets/cc7043e5-6dab-4322-9c47-a9c55484f02f" />

---

## Lưu InfluxDB

Function:

```javascript
msg.payload=[{
value:
msg.payload.current.temperature_2m
}];

return msg;
```

Measurement:

```text
temperature_history
```

Database:

```text
history_db
```

<img width="1918" height="1078" alt="Screenshot 2026-06-13 170024" src="https://github.com/user-attachments/assets/82319e1c-d3a2-4416-8c8d-131953355790" />


---

## Cảnh báo Telegram

Switch:

```text
msg.payload < 28
OR
msg.payload > 34
```

Function:

```javascript
let val = msg.payload;

msg.payload = {
chatId:"GROUP_CHAT_ID",
type:"message",
content:
"🚨 [ALERT]\n"+
"Nhiệt độ bất thường!\n"+
"Giá trị hiện tại: "+
val+
"°C"
};

return msg;
```

<img width="1918" height="1045" alt="image" src="https://github.com/user-attachments/assets/117a7228-f78c-4e29-9375-f0b7d48b3250" />


Bot sẽ gửi cảnh báo tới nhóm Telegram có 3 thành viên.


<img width="1913" height="1013" alt="image" src="https://github.com/user-attachments/assets/f12cb451-a0f3-44b1-a46e-c80e858530b1" />


TIP:

Khi không biết ChatID nhóm:

Thêm bot:

```text
RawDataBot
```

vào nhóm.

Gọi:

```text
https://api.telegram.org/botTOKEN/getUpdates
```

Đọc:

```text
chat.id
```
---

# Bước 4: Trực quan hóa dữ liệu bằng Grafana

Mở:

```text
http://192.168.56.14:3000
```


<img width="1918" height="1078" alt="Screenshot 2026-06-13 160424" src="https://github.com/user-attachments/assets/c1cd31b1-14ed-4001-bc54-2aef3bf7faf9" />


---

Data Source:

```text
Connections
↓

Data Sources
↓

InfluxDB
```

URL:

```text
http://influxdb:8086
```

Database:

```text
history_db
```

Save & Test.

<img width="1918" height="931" alt="Screenshot 2026-06-13 160436" src="https://github.com/user-attachments/assets/54443407-d293-44c9-b442-4dcd5d5629cd" />


<img width="1918" height="1078" alt="Screenshot 2026-06-13 160503" src="https://github.com/user-attachments/assets/6b27a29f-ed32-4e5c-89f9-9d1b70059d61" />

<img width="1907" height="1063" alt="Screenshot 2026-06-13 160708" src="https://github.com/user-attachments/assets/384d3b7f-1024-466f-b359-c462cc36cf40" />

<img width="1918" height="1078" alt="Screenshot 2026-06-13 160712" src="https://github.com/user-attachments/assets/5e8e0cca-339a-4d14-bbfe-447206a5a10b" />

---

Tạo Dashboard.

Query:

```sql
SELECT mean("value")
FROM "temperature_history"
WHERE $timeFilter
GROUP BY time($__interval)
fill(null)
```

---

Refresh:

```text
5s
```

Time Range:

```text
Last 5 minutes
```

Graph:

```text
Time Series
```

Connect null values:

```text
Always
```

Đặt tên Dashboard:

```text
bang1
```

<img width="1918" height="1078" alt="Screenshot 2026-06-13 161046" src="https://github.com/user-attachments/assets/89755cca-502c-4e12-af65-c0e92b390e40" />


---

# Bước 5: Tích hợp giao diện Web

### requirements.txt

```txt
flask
flask-cors
mysql-connector-python
```

---

### app.py

```
from flask import Flask, jsonify
from flask_cors import CORS
import mysql.connector

app = Flask(__name__)
CORS(app) # Cho phép Front-end gọi API khác port công khai

def get_latest_data():
    try:
        # Kết nối tới container mariadb qua tên service trong mạng docker
        conn = mysql.connector.connect(
            host="mariadb",
            user="user",
            password="userpassword",
            database="monitor_db"
        )
        cursor = conn.cursor(dictionary=True)
        # Lấy bản ghi mới nhất dựa vào timestamp hoặc id tự tăng
        cursor.execute("SELECT * FROM real_time_data ORDER BY id DESC LIMIT 1")
        result = cursor.fetchone()
        cursor.close()
        conn.close()
        return result
    except Exception as e:
        return {"error": str(e)}

@app.route('/api/current', methods=['GET'])
def current_data():
    data = get_latest_data()
    if not data or "error" in data:
        return jsonify({"status": "error", "message": "Không có dữ liệu"}), 500
    return jsonify({"status": "success", "data": data})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

### index.html

```
<!DOCTYPE html>
<html lang="vi">

<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Hệ Thống Giám Sát Nhiệt Độ Real-time</title>

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

<style>

body{
background:#f4f6f9;
font-family:'Segoe UI';
}

.monitor-card{
background:white;
border-radius:15px;
box-shadow:0 4px 15px rgba(0,0,0,.1);
padding:25px;
margin-bottom:25px;
}

.temp-display{
font-size:3rem;
font-weight:bold;
color:#ff4d4d;
}

.status-badge{
font-size:1rem;
padding:6px 12px;
border-radius:20px;
}

</style>

</head>

<body>

<div class="container py-4">

<header class="text-center pb-3 mb-4 border-bottom">

<h1>
🌡️ HỆ THỐNG GIÁM SÁT & CẢNH BÁO NHIỆT ĐỘ REALTIME
</h1>

<p class="text-muted">
Bài Tập Lớn - Hệ Thống Monitor Docker
</p>

</header>


<div class="row">

<div class="col-lg-4">

<div class="monitor-card text-center">

<h3>Nhiệt Độ Tức Thời (MariaDB)</h3>

<div class="temp-display">

<span id="temp-value">
--
</span>

°C

</div>

<br>

<div>

Cập nhật lúc:

<span id="temp-time">
--:--:--
</span>

</div>

<br>

<span
id="status-tag"
class="badge bg-success status-badge">

Hệ thống ổn định

</span>

</div>

</div>



<div class="col-lg-8">

<div class="monitor-card">

<h3>
Đồ Thị Lịch Sử (Grafana)
</h3>

<div class="ratio ratio-16x9">

<iframe
src="http://192.168.56.14:3000/d-solo/adjph5j/das1?orgId=1&refresh=5s&panelId=1"
width="100%"
height="100%"
frameborder="0">
</iframe>

</div>

</div>

</div>

</div>

</div>


<script>

async function fetchCurrentTemp(){

try{

const response =
await fetch(
"http://192.168.56.14:5000/api/current"
);

if(!response.ok){

throw new Error();

}

const res =
await response.json();

if(
res.status==="success"
){

const temp =
Number(
res.data.value
);

document
.getElementById(
"temp-value"
)
.innerText =
temp;

document
.getElementById(
"temp-time"
)
.innerText =
new Date(
res.data.timestamp
)
.toLocaleTimeString();


const tag =
document.getElementById(
"status-tag"
);

if(
temp<28
){

tag.className =
"badge bg-warning status-badge";

tag.innerText =
"❄ ALERT LOW";

}

else if(
temp>34
){

tag.className =
"badge bg-danger status-badge";

tag.innerText =
"🔥 ALERT HIGH";

}

else{

tag.className =
"badge bg-success status-badge";

tag.innerText =
"✓ Hệ thống ổn định";

}

}

}

catch(err){

console.log(err);

document
.getElementById(
"temp-value"
)
.innerText =
"Lỗi API";

}

}

fetchCurrentTemp();

setInterval(
fetchCurrentTemp,
2000
);

</script>

</body>

</html>
```

---

Build lại:

```bash
sudo docker compose up -d --build flask-api nginx
```

Mở:

```text
http://192.168.56.14
```

Ban đầu nếu hiện:

```text
Lỗi API
```

Sửa toàn bộ:

```text
localhost
```

thành:

```text
IP máy ảo
```

bao gồm:

```text
:5000
:3000
```

Refresh:

```text
CTRL + F5
```

Kết quả:

- Nhiệt độ realtime cập nhật tự động
- Dữ liệu lấy từ MariaDB
- Biểu đồ cập nhật từ InfluxDB
- Cảnh báo đổi màu theo ngưỡng


<img width="1918" height="1078" alt="Screenshot 2026-06-13 163635" src="https://github.com/user-attachments/assets/6381b042-55f9-4e16-9738-edc83a9cfca6" />


---

## Nhúng Grafana

Grafana

→ Share

→ Embed

→ Copy iframe

Yêu cầu có:

```text
from=now-5m
to=now
refresh=5s
```

Dán vào:

```html
<iframe>
```

<img width="1918" height="1078" alt="Screenshot 2026-06-13 163635" src="https://github.com/user-attachments/assets/004a6434-dd11-40fb-9ff5-928937a95ece" />


---

# Bước 6: Đóng gói và khôi phục Offline

Tạo thư mục:

```bash
mkdir -p ~/ma/backup
```

Xuất Image:

```bash
sudo docker save -o ~/ma/backup/nodered.tar nodered/node-red:latest

sudo docker save -o ~/ma/backup/mariadb.tar mariadb:10.6

sudo docker save -o ~/ma/backup/influxdb.tar influxdb:1.8

sudo docker save -o ~/ma/backup/grafana.tar grafana/grafana:latest

sudo docker save -o ~/ma/backup/nginx.tar nginx:alpine

sudo docker save -o ~/ma/backup/flask-api.tar python:3.9-slim
```

<img width="1918" height="677" alt="Screenshot 2026-06-13 164205" src="https://github.com/user-attachments/assets/732f9113-8850-4094-a1bf-a9a733bd0aa6" />


---

Xóa hệ thống:

```bash
cd ~/ma

sudo docker compose down

sudo docker rmi \
nodered/node-red:latest \
mariadb:10.6 \
influxdb:1.8 \
grafana/grafana:latest \
nginx:alpine \
python:3.9-slim
```

<img width="1853" height="672" alt="Screenshot 2026-06-13 164321" src="https://github.com/user-attachments/assets/e53e3c56-a9ec-4570-98a8-6203cf8d5b8a" />


<img width="1887" height="761" alt="Screenshot 2026-06-13 164414" src="https://github.com/user-attachments/assets/a0dedf6e-133c-45a8-901a-963d584350f1" />

---

Khôi phục:

```bash
sudo docker load -i ~/ma/backup/nodered.tar

sudo docker load -i ~/ma/backup/mariadb.tar

sudo docker load -i ~/ma/backup/influxdb.tar

sudo docker load -i ~/ma/backup/grafana.tar

sudo docker load -i ~/ma/backup/nginx.tar

sudo docker load -i ~/ma/backup/flask-api.tar
```

Khởi động:

```bash
sudo docker compose up -d
```

Kiểm tra:

```bash
sudo docker ps
```

<img width="1846" height="480" alt="Screenshot 2026-06-13 164457" src="https://github.com/user-attachments/assets/71602397-6c31-496e-b3dc-02ab14c1fe61" />


<img width="1918" height="1078" alt="Screenshot 2026-06-13 163635" src="https://github.com/user-attachments/assets/e4bbb3d0-42f2-4090-82b6-7440c4fa45c2" />

---

# KẾT LUẬN

Thông qua bài thực hành này, em đã triển khai thành công hệ thống **Monitor & Alert Data Real-time** sử dụng kiến trúc đa dịch vụ bằng Docker Compose.

Hệ thống thực hiện đầy đủ quy trình thu thập dữ liệu thời gian thực, lưu trữ song song bằng MariaDB và InfluxDB, trực quan hóa bằng Grafana, cung cấp giao diện Web thông qua Flask API và Nginx, đồng thời gửi cảnh báo tự động qua Telegram khi dữ liệu vượt ngưỡng cho phép.

Việc đóng gói toàn bộ hệ thống ra các tệp `.tar` và khôi phục hoàn chỉnh trong môi trường Offline đã chứng minh khả năng di động, khả năng tái triển khai và tính độc lập của Docker trong thực tế.
