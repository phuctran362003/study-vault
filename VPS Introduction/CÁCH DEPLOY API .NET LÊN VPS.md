# 1. Tạo Access Token trên Docker Hub

Vì Docker Hub không còn hỗ trợ đăng nhập bằng mật khẩu trong CI/CD, bạn cần tạo một Personal Access Token (PAT):

1. Đăng nhập vào [Docker Hub](https://hub.docker.com/).
2. Nhấp vào ảnh đại diện của bạn ở góc trên bên phải và chọn "Account Settings".
3. Chuyển đến tab "Security".
4. Nhấp vào nút "New Access Token".
5. Đặt tên mô tả cho token (ví dụ: github-actions-token).
6. Nhấp "Generate" và sao chép token ngay lập tức vì nó chỉ hiển thị một lần.

# 2. Thêm Secrets vào GitHub Repository

Để GitHub Actions có thể sử dụng thông tin đăng nhập Docker Hub, bạn cần thêm hai secrets vào repository của mình:

1. Truy cập trang chính của repository trên GitHub.
2. Nhấp vào tab "Settings".
3. Trong thanh bên trái, chọn "Secrets and variables" > "Actions".
4. Nhấp vào nút "New repository secret".
5. Thêm các secrets sau:

- Name: DOCKERHUB_USERNAMEValue: Tên người dùng Docker Hub của bạn (ví dụ: your_dockerhub_username).
- Name: DOCKERHUB_TOKENValue: Access Token bạn đã tạo ở bước 1.

# 3. Sử dụng Secrets trong GitHub Actions Workflow

Trong file workflow (ví dụ: docker-build.yml), bạn có thể sử dụng các secrets này để đăng nhập vào Docker Hub:

|   |
|---|
|- name: Đăng nhập vào Docker Hub<br><br>  uses: docker/login-action@v2<br><br>  with:<br><br>    username: ${{ secrets.DOCKERHUB_USERNAME }}<br><br>    password: ${{ secrets.DOCKERHUB_TOKEN }}|

Sau đó, bạn có thể tiếp tục với các bước build và push Docker image như đã cấu hình trong workflow của mình.

Lưu ý: Đảm bảo rằng bạn không chia sẻ hoặc công khai các secrets này. Chúng được mã hóa và chỉ có thể được sử dụng trong các workflows của repository tương ứng.