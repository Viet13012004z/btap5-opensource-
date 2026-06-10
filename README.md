# btap5-opensource
# Nguyễn Hoàng Việt - K225480106074
# APP MONITOR + ALERT DATA REALTIME
# PHẦN 1: LÝ THUYẾT VỀ DOCKER

## 1. Docker là gì?
**Docker** là một nền tảng mã nguồn mở cho phép đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (thư viện, môi trường hệ điều hành, cấu hình...) vào trong một đơn vị duy nhất gọi là **Container**. 

* **Bản chất:** Thay vì ảo hóa toàn bộ phần cứng như Máy ảo (Virtual Machine), Docker chia sẻ chung nhân (Kernel) của hệ điều hành máy chủ. 
* **Mục đích:** Giúp ứng dụng hoạt động độc lập, đồng nhất trên bất kỳ môi trường nào (từ máy cá nhân cho đến máy chủ production) mà không gặp lỗi xung đột thư viện ("Work on my machine").

---

## 2. Các từ khóa được sử dụng trong `docker-compose.yml`

File `docker-compose.yml` được cấu trúc bằng định dạng YAML để định nghĩa và quản lý đa container. Dưới đây là các từ khóa cốt lõi:

### `version`
* **Ý nghĩa:** Chỉ định phiên bản cấu hình định dạng của Docker Compose để Docker Engine đọc hiểu các tính năng đi kèm.
* **Ví dụ:**
  ```yaml
  version: '3.8'

| Từ khóa | Ý nghĩa và Chức năng | Ví dụ minh họa |
| :--- | :--- | :--- |
| **`version`** | Chỉ định phiên bản cấu hình định dạng của Docker Compose để Docker Engine đọc hiểu các tính năng đi kèm. | `version: '3.8'` |
| **`services`** | Khởi tạo vùng định nghĩa cho các dịch vụ (các container) cấu thành nên ứng dụng tổng thể. | `services:`<br>`  web_server:`<br>`    image: nginx` |
| **`image`** | Chỉ định tên và phiên bản (tag) của Docker Image có sẵn trên Docker Hub hoặc Registry để tải về chạy container. | `image: mariadb:10.6` |
| **`build`** | Chỉ định đường dẫn tới thư mục chứa `Dockerfile` nội bộ để tự động biên dịch (build) Image riêng. | `build: ./flask-api` |
| **`ports`** | Ánh xạ cấu hình cổng mạng kết nối giữa Máy chủ (Host) và bên trong Container (`Cổng_Máy_Chủ:Cổng_Container`). | `ports:`<br>`  - "8080:80"` |
| **`environment`** | Khai báo các biến môi trường cấu hình cho container lúc khởi chạy (thiết lập cấu hình database, mật khẩu, token...). | `environment:`<br>`  - MYSQL_ROOT_PASSWORD=123`<br>`  - MYSQL_DATABASE=monitor_db` |
| **`volumes`** | Gắn kết (mount) thư mục giữa máy chủ và container nhằm mục đích lưu trữ dữ liệu bền vững, không bị mất khi xóa container. | `volumes:`<br>`  - mariadb_data:/var/lib/mysql` |
| **`networks`** | Định nghĩa hoặc tham chiếu đến các mạng ảo nội bộ cô lập để các container có thể tự do giao tiếp an toàn với nhau. | `networks:`<br>`  - monitor_net` |
| **`depends_on`** | Chỉ định thứ tự ưu tiên khởi động. Container đích sẽ đợi các container phụ thuộc được bật lên trước rồi mới chạy. | `depends_on:`<br>`  - mariadb` |

## 3. Ưu điểm khi triển khai ứng dụng sử dụng Docker

Việc sử dụng Docker để đóng gói và triển khai ứng dụng mang lại nhiều lợi ích vượt trội so với các phương pháp truyền thống (như cài đặt trực tiếp trên hệ điều hành hoặc dùng máy ảo VM):

* **Tính nhất quán tuyệt đối (Consistency):** Đảm bảo môi trường chạy code giữa máy của Lập trình viên (Dev), Kiểm thử viên (Tester) và Máy chủ thật (Production) giống nhau 100%. Loại bỏ hoàn toàn lỗi kinh điển: *"Code chạy bình thường ở máy tôi nhưng lên server lại lỗi"*.
* **Tối ưu hóa và tiết kiệm tài nguyên:** Các container chia sẻ chung nhân (Kernel) của hệ điều hành máy chủ nên có kích thước cực kỳ nhẹ (chỉ vài chục đến vài trăm MB) và khởi động chỉ mất vài giây. Hiệu năng vượt trội hơn nhiều so với việc phải chạy một hệ điều hành ảo hóa hoàn chỉnh như Máy ảo (VM).
* **Cô lập môi trường an toàn (Isolation):** Mỗi ứng dụng chạy trong một container hoàn toàn tách biệt. Bạn có thể chạy song song nhiều ứng dụng yêu cầu các phiên bản thư viện khác nhau (ví dụ: một app chạy NodeJS 14, một app chạy NodeJS 20) trên cùng một máy chủ mà không sợ xung đột hệ thống.
* **Đơn giản hóa quy trình CI/CD và Mở rộng (Scalability):** Vì ứng dụng đã được đóng gói thành một "khối" đồng nhất (Image), việc tự động hóa kiểm thử, triển khai hoặc nâng cấp lên phiên bản mới diễn ra cực kỳ nhanh chóng. Khi hệ thống bị quá tải, việc nhân bản (scale) thêm nhiều container mới chỉ mất vài giây.

---

## 4. Quy trình triển khai ứng dụng Ngoại tuyến (Offline) trên Máy chủ không có Internet

Khi ứng dụng đã được tạo và kiểm thử chạy ổn định (OK) bằng Docker trên laptop cá nhân, để triển khai nó lên một **Máy chủ thật hoàn toàn không có kết nối Internet** (môi trường mạng nội bộ, mạng quân sự, doanh nghiệp bảo mật cao), chúng ta không thể dùng lệnh `docker pull` thông thường. Thay vào đó, quy trình triển khai Offline chuẩn sẽ gồm 4 bước sau:

### Bước 1: Đóng gói và xuất các Docker Images trên Máy tính cá nhân (Có Internet)
Tại máy tính cá nhân của bạn, sau khi đã build và chạy thử mọi thứ ổn định, sử dụng lệnh `docker save` để nén các Docker Image cần thiết thành các file lưu trữ dạng `.tar`:
```bash
# Cú pháp: docker save -o [ten_file_nen.tar] [ten_docker_image:tag]

docker save -o nodered_image.tar nodered/node-red:latest
docker save -o mariadb_image.tar mariadb:latest
docker save -o influxdb_image.tar influxdb:1.8
docker save -o nginx_image.tar nginx:latest
docker save -o flask_api_image.tar custom-flask-api:latest
```
### Bước 2: Di chuyển tài nguyên sang Máy chủ thật bằng thiết bị vật lý
Sao chép toàn bộ các file nén .tar vừa tạo cùng với các file cấu hình dự án từ máy tính cá nhân sang một thiết bị lưu trữ ngoại vi (như USB, ổ cứng di động) hoặc truyền qua mạng LAN nội bộ để đưa sang Máy chủ thật. Các file cần mang đi bao gồm:

1. Các file đóng gói hình ảnh hệ thống (.tar) ở Bước 1.

2. File điều phối hệ thống docker-compose.yml.

3. Thư mục chứa mã nguồn giao diện Frontend tĩnh hoặc các file cấu hình liên quan (nếu có).

### Bước 3: Nạp (Load) lại Docker Images trên Máy chủ Offline
Tại giao diện dòng lệnh (Terminal) của Máy chủ thật, di chuyển vào thư mục lưu trữ các file .tar đã copy sang. Thực hiện lệnh docker load để giải nén và nạp ngược các Image này vào hệ thống quản lý cục bộ của máy chủ:

```bash
docker load -i nodered_image.tar
docker load -i mariadb_image.tar
docker load -i influxdb_image.tar
docker load -i nginx_image.tar
docker load -i flask_api_image.tar
```
### Bước 4: Khởi chạy ứng dụng bằng Docker Compose
Khi Máy chủ thật đã sở hữu toàn bộ các Docker Image cần thiết ở môi trường local, bạn chỉ cần di chuyển đến thư mục chứa file docker-compose.yml và kích hoạt toàn bộ hệ thống lên bằng lệnh:
```
 docker compose up -d
```

### Thực Hành: 

## Cấu trúc thư mục:
```
monitorsystem/
├── docker-compose.yml             # File cấu hình điều phối toàn bộ 6 container
├── flask-api/                     # Thư mục chứa mã nguồn Backend (Python)
│   ├── Dockerfile                 # File đóng gói môi trường chạy cho Flask
│   ├── app.py                     # Mã nguồn xử lý API lấy dữ liệu từ MariaDB
│   └── requirements.txt           # Danh sách các thư viện Python cần cài đặt
└── frontend/                      # Thư mục chứa mã nguồn Giao diện (Nginx)
    └── index.html                 # Trang web hiển thị số realtime và nhúng Grafana
```
### docker-compose.yml

```
 version: '3.8'

services:
  nodered:
    image: nodered/node-red:latest
    container_name: nodered_app
    ports:
      - "1880:1880"
    volumes:
      - nodered_data:/data
    networks:
      - monitor_net
    depends_on:
      - mariadb
      - influxdb

  mariadb:
    image: mariadb:latest
    container_name: mariadb_db
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: realtime_db
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - monitor_net

  influxdb:
    image: influxdb:1.8
    container_name: influxdb_db
    environment:
      INFLUXDB_DB: history_db
    ports:
      - "8086:8086"
    volumes:
      - influxdb_data:/var/lib/influxdb
    networks:
      - monitor_net

  grafana:
    image: grafana/grafana:latest
    container_name: grafana_app
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true
      - GF_AUTH_ANONYMOUS_ENABLED=true
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitor_net

  flask-api:
    build: ./flask-api
    container_name: flask_api
    ports:
      - "5000:5000"
    networks:
      - monitor_net
    depends_on:
      - mariadb

  nginx:
    image: nginx:latest
    container_name: nginx_server
    ports:
      - "80:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
    networks:
      - monitor_net

volumes:
  nodered_data:
  mariadb_data:
  influxdb_data:
  grafana_data:

networks:
  monitor_net:
    driver: bridge
```

