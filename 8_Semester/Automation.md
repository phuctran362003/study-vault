# Hướng dẫn thiết lập GPTComet và script tự động commit + push trên Windows

## 1. Mục đích

Tài liệu này hướng dẫn chi tiết quy trình thiết lập và sử dụng GPTComet để tự động tạo commit message bằng AI và tự động push lên Git trên môi trường Windows, không dùng Python.

## 2. GPTComet là gì?

GPTComet là một công cụ mã nguồn mở cho phép tạo commit message tự động dựa trên thay đổi trong Git staging area. Nó khai thác mô hình ngôn ngữ nhân tạo (AI) để sinh ra thông điệp commit chính xác và chứa ý nghĩa.

- **Repo chính thức:** [https://github.com/belingud/gptcomet](https://github.com/belingud/gptcomet)

## 3. Các bước thiết lập và cài đặt

### Bước 1: Cài GPTComet

1.  Tải GPTComet theo hướng dẫn từ repo trên.
2.  Sau khi cài xong, mở PowerShell và chạy lệnh `gptcomet` để đảm bảo đã cài đặt đúng.

**Mục đích:** Đảm bảo GPTComet đã sẵn sàng sử dụng trong PowerShell.

### Bước 2: Tạo script tự động commit + push

1.  Mở **Notepad** (hoặc trình soạn thảo văn bản bất kỳ).
2.  Dán nội dung sau vào file:

    ```powershell
    # Force UTF-8 for proper Unicode display
    [Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()

    Write-Host "Running GPTComet commit..."

    # Step 1: Add all files
    git add .

    # Step 2: Check if there are staged changes
    $staged = git diff --cached --name-only
    if (-not $staged) {
        Write-Host "Error: no staged changes found. Exiting."
        exit 1
    }

    # Step 3: Auto-confirm 'yes' for GPTComet prompt
    $commitOutput = echo y | gptcomet commit | Out-String
    Write-Host $commitOutput

    # Step 4: Check if commit was successful
    if ($commitOutput -match "Successfully created commit") {
        Write-Host "`nCommit succeeded. Executing git push..."
        git push
    } else {
        Write-Host "`nCommit failed or was cancelled. Skipping git push."
    }
    ```

3.  Lưu lại với tên `gcommit.ps1` và đặt tại một thư mục dễ truy cập, ví dụ:
    `C:\Users\<TÊN_BẠN>\Desktop\Scripts\gcommit.ps1`

**Mục đích:** Script này sẽ chạy `gptcomet commit`, và nếu commit thành công, nó sẽ tự động thực hiện `git push`.

### Bước 3: Cho phép PowerShell chạy script

1.  Mở PowerShell với quyền user thường.
2.  Chạy lệnh sau:
    ```powershell
    Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
    ```

**Mục đích:** Cho phép chạy các script tự tạo (không cần ký số) trong tài khoản Windows hiện tại.

### Bước 4: Bỏ chặn file (nếu cần)

Trong trường hợp Windows vẫn chặn script, hãy chạy lệnh này trong PowerShell:

```powershell
Unblock-File -Path "C:\Users\<TÊN_BẠN>\Desktop\Scripts\gcommit.ps1"
```

**Mục đích:** Xóa cờ `Zone.Identifier` khỏi file, vốn thường được thêm vào các file tải về từ Internet hoặc tạo ra từ các ứng dụng không xác định.

### Bước 5: Tạo alias `gcommit`

1.  Mở PowerShell và chạy lệnh sau để mở file cấu hình profile:
    ```powershell
    notepad $PROFILE
    ```
2.  Nếu file chưa tồn tại, Windows sẽ hỏi bạn có muốn tạo mới không. Chọn **Yes**.
3.  Dán đoạn mã sau vào cuối file:
    ```powershell
    function gcommit {
        & "C:\Users\<TÊN_BẠN>\Desktop\Scripts\gcommit.ps1"
    }
    ```
4.  Lưu và đóng file. Khởi động lại PowerShell để áp dụng thay đổi.

**Mục đích:** Tạo một lệnh tắt (`gcommit`) để gọi script một cách dễ dàng từ bất kỳ đâu trong PowerShell.

## 4. Cách sử dụng

1.  Mở PowerShell và điều hướng đến thư mục dự án Git của bạn.
2.  Thêm các file bạn muốn commit vào staging area (`git add .`).
3.  Chạy lệnh:
    ```powershell
    gcommit
    ```
4.  GPTComet sẽ gợi ý một commit message. Sau khi bạn xác nhận, script sẽ tự động push lên remote repository. Tất cả chỉ với một lệnh duy nhất.

## 5. Ghi chú

- GPTComet yêu cầu có kết nối Internet và token truy cập mô hình AI (ví dụ: OpenAI API key).
- Script này hoạt động tốt với mọi loại repository (ví dụ: .NET, JavaScript,...) và không phụ thuộc vào Python.
- Các bước thiết lập ở trên chỉ cần thực hiện một lần duy nhất.

## 6. Kết luận

Với GPTComet và alias `gcommit`, bạn đã có một quy trình làm việc tự động, nhất quán và chuyên nghiệp. Điều này giúp cải thiện tốc độ làm việc và đồng bộ hóa quy chuẩn commit cho cá nhân hoặc nhóm phát triển.
