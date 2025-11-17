# Hướng dẫn Cài đặt và Cấu hình MinIO trên VPS

Tài liệu này hướng dẫn các bước để cài đặt MinIO, cấu hình dịch vụ, và thiết lập reverse proxy với Nginx và SSL.

## I. Cài đặt MinIO

### Bước 1: Tải và Cài đặt MinIO Binary

Tải file thực thi của MinIO, cấp quyền thực thi và di chuyển vào thư mục `/usr/local/bin` để có thể gọi từ bất kỳ đâu.

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin
```

### Bước 2: Tạo User và Thư mục Dữ liệu

Tạo một user hệ thống riêng cho MinIO để tăng cường bảo mật và tạo thư mục để lưu trữ dữ liệu.

```bash
sudo useradd -r minio-user -s /sbin/nologin
sudo mkdir -p /mnt/data/minio
sudo chown minio-user:minio-user /mnt/data/minio
```

### Bước 3: Tạo File Cấu hình MinIO

Tạo file môi trường để định nghĩa các biến cần thiết cho MinIO.

Tạo file tại `/etc/default/minio`:
```bash
sudo nano /etc/default/minio
```

Thêm nội dung sau vào file:
```ini
MINIO_VOLUMES="/mnt/data/minio"
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin
```
**Lưu ý:** Thay đổi `MINIO_ROOT_USER` và `MINIO_ROOT_PASSWORD` bằng thông tin đăng nhập an toàn hơn cho môi trường production.

### Bước 4: Tạo Service MinIO với systemd

Tạo một service unit cho `systemd` để quản lý tiến trình MinIO (tự động khởi động, restart khi lỗi).

Tạo file `/etc/systemd/system/minio.service`:
```bash
sudo nano /etc/systemd/system/minio.service
```

Thêm nội dung sau vào file:
```ini
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target

[Service]
User=minio-user
Group=minio-user
EnvironmentFile=/etc/default/minio
ExecStart=/usr/local/bin/minio server $MINIO_VOLUMES --address ":9000" --console-address ":9001"
Restart=always
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
- **Port 9000**: Cổng API (S3-compatible) dùng cho SDK, `mc` (MinIO Client), và các công cụ upload.
- **Port 9001**: Cổng giao diện web (Console UI) cho quản trị viên.

### Bước 5: Khởi động Dịch vụ MinIO

Kích hoạt và khởi động service MinIO vừa tạo.

```bash
sudo systemctl daemon-reload
sudo systemctl enable minio
sudo systemctl start minio
```

### Bước 6: Kiểm tra Trạng thái

Kiểm tra xem dịch vụ MinIO đã chạy thành công hay chưa.

```bash
systemctl status minio
```

### Bước 7: Truy cập Giao diện Web

Truy cập vào giao diện quản trị của MinIO qua trình duyệt:
`http://<your-vps-ip>:9001`

Sử dụng thông tin đăng nhập đã cấu hình ở **Bước 3**:
- **Username:** `minioadmin`
- **Password:** `minioadmin`

### Bước 8: Mở Port Firewall (Nếu cần)

Nếu bạn đang sử dụng firewall (ví dụ `ufw`), hãy mở các port 9000 và 9001.

```bash
sudo ufw allow 9000
sudo ufw allow 9001
```

---

## II. Cấu hình Reverse Proxy với Nginx

Các bước sau hướng dẫn cấu hình Nginx làm reverse proxy cho MinIO, cho phép truy cập qua tên miền và bảo mật bằng SSL.

### Bước 1: Tạo File Cấu hình Nginx

Tạo một file cấu hình mới cho trang MinIO trong Nginx.

```bash
sudo nano /etc/nginx/sites-available/minio
```

### Bước 2: Kích hoạt Cấu hình

Tạo symbolic link để kích hoạt cấu hình và kiểm tra cú pháp Nginx, sau đó reload lại dịch vụ.

```bash
sudo ln -s /etc/nginx/sites-available/minio /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Bước 3: Cài đặt SSL bằng Certbot

Sử dụng Certbot để tự động lấy và cài đặt chứng chỉ SSL cho tên miền của bạn.

```bash
sudo certbot --nginx -d minio.fpt-devteam.fun
```
**Lưu ý:** Thay `minio.fpt-devteam.fun` bằng tên miền của bạn.

Sau khi chạy thành công, Certbot sẽ tự động cập nhật file `/etc/nginx/sites-available/minio` với cấu hình HTTPS.

### Bước 4: Nội dung File Cấu hình Nginx (Sau khi có SSL)

Đây là ví dụ về file cấu hình Nginx sau khi Certbot đã hoàn tất.

```nginx
server {
    listen 443 ssl;
    server_name minio.fpt-devteam.fun;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/minio.fpt-devteam.fun/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/minio.fpt-devteam.fun/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # MinIO Web Console proxy
    location / {
        proxy_pass http://127.0.0.1:9001;
        
        # Advanced proxy settings
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Support for WebSocket (for MinIO >= 2023)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Disable buffering for large uploads
        proxy_request_buffering off;
        
        # Connection timeout for MinIO
        proxy_connect_timeout 300;
    }

    # Get the real client IP
    real_ip_header X-Real-IP;
    
    # No limit on upload file size
    client_max_body_size 0;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name minio.fpt-devteam.fun;
    return 301 https://$host$request_uri;
}
```
