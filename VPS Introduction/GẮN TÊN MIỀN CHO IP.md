SAU NÀY MUỐN SETUP DOMAIN TRÊN VPS VỚI NGINX THÌ PHẢI LÀM NHƯ NÀO?

Context: sau khi mày mua tên miền tên là ae-tao-fullstack.site thì phải làm sao để gắn vào server nó để có https ?

1. Thêm record DNS trên Cloudflare (hoặc nhà cung cấp DNS)

Type: A

Name: subdomain (Ví dụ: skibidi nếu bạn muốn ae-tao-fullstack.site)

Content: 103.211.201.162 (IP VPS của mày)

Proxy Status: DNS Only (hoặc Proxied nếu muốn Cloudflare bảo vệ)

TTL: Auto

2. Tạo file cấu hình Nginx cho subdomain

sudo nano /etc/nginx/sites-available/<tên-của-subdomain>

sudo nano /etc/nginx/sites-available/arwoh.ae-tao-fullstack-api.site

sudo nano /etc/nginx/sites-available/ae-tao-fullstack-api.site

- Thêm cấu hình sau (đổi subdomain.example.com và port nếu cần):

server {

    listen 80;

    server_name subdomain.example.com;

    location / {

        proxy_pass [http://localhost:PORT](http://localhost:PORT); # Đổi PORT theo ứng dụng

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 300;

        proxy_send_timeout 300;

        proxy_read_timeout 300;

        send_timeout 300;

    }

}

3. Kích hoạt cấu hình

Tạo symbolic link:

sudo ln -s /etc/nginx/sites-available/<tên-của-subdomain> /etc/nginx/sites-enabled/

4. Kiểm tra cấu hình và restart Nginx:

sudo nginx -t

Nếu không có lỗi, restart Nginx:

sudo systemctl restart nginx

6. (Khuyến nghị) Cấu hình HTTPS với Let’s Encrypt:

sudo apt install certbot python3-certbot-nginx -y

sudo certbot --nginx -d <tên-của-subdomain>.ae-tao-fullstack-api.site

7. Kiểm tra kết quả

Truy cập: [https://<tên-của-subdomain>.ae-tao-fullstack-api.site](https://%3ctên-của-subdomain%3e.ae-tao-fullstack-api.site)

Kết quả khá ngon