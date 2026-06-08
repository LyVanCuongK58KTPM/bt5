# BÀI TẬP 5
## 1. Lý Thuyết:
## DOCKER LÀ GÌ?

Docker là một nền tảng mã nguồn mở cho phép lập trình viên đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (bao gồm thư viện, môi trường, cấu hình...) vào trong một đơn vị duy nhất gọi là Container. 

Để dễ hình dung, Docker hoạt động giống như những thùng container trên tàu biển:
* Trước đây, khi vận chuyển hàng hóa, người ta phải lo lắng hàng này có làm hỏng hàng kia không, xếp lên tàu thế nào. Giờ đây, mọi thứ được đóng chặt vào các thùng container tiêu chuẩn, có thể đặt lên bất kỳ con tàu nào mà không sợ ảnh hưởng đến xung quanh.
* Với phần mềm, Docker đảm bảo ứng dụng của bạn sẽ chạy y hệt nhau trên mọi môi trường, từ laptop cá nhân, máy ảo test cho đến máy chủ production, xóa bỏ hoàn toàn câu nói kinh điển: "Code chạy mượt mà trên máy em nhưng sang máy anh lại lỗi!".

---

## CÁC TỪ KHÓA ĐƯỢC SỬ DỤNG TRONG DOCKER-COMPOSE.YML

File `docker-compose.yml` được sử dụng để định nghĩa và vận hành các ứng dụng Docker đa container (Multi-container). Dưới đây là ý nghĩa và ví dụ minh họa của các từ khóa cốt lõi dùng để mô tả một Service, Network, Volume:

###  Cấu trúc phân cấp cao nhất (Top-level)
* **version**: Định nghĩa phiên bản cấu trúc file Docker Compose (Ví dụ: '3.8').
* **services**: Nơi định nghĩa các container (dịch vụ) cấu thành nên ứng dụng (như Web, Database).
* **networks**: Định nghĩa mạng ảo để các container kết nối và giao tiếp nội bộ với nhau.
* **volumes**: Định nghĩa không gian lưu trữ dữ liệu bền vững (không bị mất khi container bị xóa).

### Chi tiết các từ khóa mô tả một Service (Container)
* **image**: Chỉ định Docker Image có sẵn trên Docker Hub (hoặc local) để tạo ra container này.
* **build**: Dùng để build image trực tiếp từ một Dockerfile có sẵn trong thư mục thay vì dùng image tải sẵn.
* **ports**: Ánh xạ cổng (port) từ Máy chủ (Host) vào bên trong Container theo cú pháp HOST:CONTAINER.
* **environment**: Khai báo các biến môi trường (Environment Variables) cho ứng dụng bên trong container sử dụng.
* **volumes**: Gắn một Volume hoặc một thư mục từ máy chủ vào trong container để lưu dữ liệu hoặc đồng bộ code.
* **networks**: Chỉ định container này tham gia vào mạng ảo nào để giao tiếp với các container khác.
* **depends_on**: Thiết lập thứ tự khởi chạy. Container này phải đợi container được chỉ định khởi động trước.
* **restart**: Chính sách tự động khởi động lại container (ví dụ: always - luôn tự bật lại nếu bị lỗi hoặc máy chủ reset).

### Ví dụ minh họa file docker-compose.yml hoàn chỉnh
Dưới đây là một file ví dụ dựng một hệ thống gồm một ứng dụng Web (Node.js/Python) kết nối tới cơ sở dữ liệu MariaDB, có mạng riêng và ổ đĩa lưu trữ riêng:

```yaml
version: '3.8'

services:
  db_server:
    image: mariadb:10.6
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secret_password
      MYSQL_DATABASE: iot_data
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - backend_network

  web_app:
    build: ./web_app_directory
    ports:
      - "8080:3000"
    environment:
      DB_HOST: db_server
      DB_NAME: iot_data
    depends_on:
      - db_server
    networks:
      - backend_network

volumes:
  db_data:

networks:
  backend_network:
```
## ƯU ĐIỂM KHI TRIỂN KHAI ỨNG DỤNG SỬ DỤNG DOCKER
Khi đóng gói ứng dụng (ví dụ: các dự án web IoT, nhận diện khuôn mặt, hay web e-commerce) vào Docker, hệ thống sẽ nhận được các lợi ích vượt trội:

Tính nhất quán tối đa: Loại bỏ hoàn toàn sự khác biệt về môi trường, hệ điều hành giữa các máy tính. App chạy tốt trên Ubuntu máy bạn thì chắc chắn sẽ chạy tốt trên máy chủ của đối tác hoặc trường học.

Đóng gói gọn gàng, cách ly an toàn: Mỗi container là một môi trường độc lập. Bạn có thể chạy song song 2 container ứng dụng, một cái dùng Python 2.7, một cái dùng Python 3.10 trên cùng một máy chủ mà không sợ chúng xung đột thư viện với nhau.

Khởi động siêu nhanh: Khác với máy ảo (Virtual Machine) phải khởi động lại toàn bộ hệ điều hành mất vài phút, Docker Container dùng chung nhân (kernel) của máy chủ nên việc khởi động hay tắt ứng dụng chỉ diễn ra trong vòng vài giây.

Dễ dàng quản lý và triển khai: File docker-compose.yml đóng vai trò như một bản thiết kế hệ thống. Chỉ với một câu lệnh đơn giản, toàn bộ hạ tầng phức tạp bao gồm mã nguồn, database, mạng kết nối đều tự động dựng lên gọn gàng.

Tiết kiệm tài nguyên: Container tiêu tốn rất ít dung lượng RAM và CPU so với máy ảo thông thường, giúp bạn tận dụng tối đa sức mạnh của phần cứng máy chủ.


## CÁC BƯỚC TRIỂN KHAI APP DOCKER LÊN MÁY CHỦ THẬT KHÔNG CÓ INTERNET
Khi ứng dụng đã chạy test ổn định bằng Docker trên laptop cá nhân, quy trình di chuyển và cài đặt lên một máy chủ vật lý bị cô lập mạng hoàn toàn (Môi trường Offline/Air-gapped) được thực hiện qua các bước sau:

Bước 1: Đóng gói (Build và Export) các Docker Images trên laptop (Nơi có Internet)
Trước tiên, trên máy tính cá nhân cần build hoàn chỉnh toàn bộ ứng dụng thành các Docker Image cố định, sau đó dùng lệnh nén chúng thành file .tar.

Khởi chạy lệnh build để đóng gói app thành image cục bộ:

```Bash
docker compose build
```
Kiểm tra danh sách các image đang có bằng lệnh docker images để lấy chính xác tên và tag của các image ứng dụng cũng như image cơ sở dữ liệu (ví dụ: mariadb:10.6, app2_web_app).

Sử dụng lệnh docker save để xuất các image này thành các file nén dạng tệp tin thông thường:

```Bash
docker save -o web_app_image.tar app2_web_app:latest
docker save -o mariadb_image.tar mariadb:10.6
```
Bước 2: Chuẩn bị bộ cài đặt Docker cho Máy chủ Offline
Vì máy chủ thật không có mạng, bạn không thể gõ lệnh apt-get install docker được. Bạn cần tải sẵn file cài đặt Docker ngoại tuyến ngay trên laptop của mình.

Truy cập vào trang tải file của Docker (đối với hệ điều hành của máy chủ, ví dụ Ubuntu là các file .deb, CentOS là .rpm).

Tải toàn bộ các file gói cài đặt (docker-ce, docker-ce-cli, containerd.io, và plugin docker-compose-plugin) về máy tính của bạn.

Bước 3: Sao chép dữ liệu sang Máy chủ thật
Cắm USB hoặc ổ cứng di động vào laptop của bạn.

Sao chép toàn bộ các file sau vào USB:

Các file cài đặt Docker Offline (.deb hoặc .rpm).

Các file image đã nén (web_app_image.tar, mariadb_image.tar).

File cấu hình hệ thống: docker-compose.yml.

Cắm USB vào máy chủ thật và sao chép toàn bộ dữ liệu vào một thư mục làm việc trên máy chủ.

Bước 4: Cài đặt Docker và Nạp Image trên Máy chủ thật
Bật terminal của máy chủ thật lên và tiến hành thiết lập:

Cài đặt Docker Offline: Gõ lệnh cài đặt trực tiếp từ các file gói vật lý có trong thư mục (ví dụ trên hệ điều hành Ubuntu/Debian):

```Bash
sudo dpkg -i *.deb
```
Nạp (Import) các Image vào hệ thống Docker của máy chủ: Dùng lệnh docker load để giải nén các file .tar ngược trở lại thành Docker Image:

```Bash
docker load -i web_app_image.tar
docker load -i mariadb_image.tar
```
Kiểm tra lại xem Docker trên máy chủ đã nhận đủ image chưa bằng lệnh: sudo docker images.

Bước 5: Khởi chạy ứng dụng bằng Docker Compose
Khi các image đã nằm sẵn trong máy chủ, lệnh Docker Compose sẽ lấy trực tiếp tài nguyên đó ra chạy mà không cần kết nối internet ra ngoài để tải nữa:

Di chuyển vào thư mục chứa file docker-compose.yml trên máy chủ.

Gõ lệnh kích hoạt toàn bộ hệ thống chạy ngầm:
```Bash
sudo docker compose up -d
```
##. Thực hành:
- Cấu hình file docker-compose.yml
  ```bash
  sudo nano docker-compose.yml
  ```
  
