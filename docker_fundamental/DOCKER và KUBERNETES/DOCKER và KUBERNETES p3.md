# Tạo và build một **Docker image** sao mà nhiều thứ rắc rối quá zậy???

Trong suốt các bài trước, chúng ta đã sử dụng rất nhiều **Image** được tạo sẵn bởi các kỹ sư khác.

Ví dụ như chúng ta đã dùng **image** **Hello World**, **Redis**, hay **BusyBox**. Tất cả những **image** này đều là do người khác tạo ra. Chúng ta chỉ cần tải chúng về máy và tạo **container** từ đó để chạy.

Bây giờ với bài viết này, chúng ta sẽ bắt đầu học cách tự tạo ra **Image** riêng của mình, để có thể chạy ứng dụng của chính chúng ta trong những **container**.

Để tạo một **Image** chúng ta sẽ phải xem qua kha khá thứ, nhưng bạn đừng lo, đã có mình ở đây, mình sẽ giúp bạn đi qua những kiến thức này một cách dễ dàng thôi, giờ thì bắt đầu thôi!!!

À mà khoan đã, mong anh em đọc và cho mình 1 upvote và comment nếu thấy cấn cấn để mình cải thiện bài viết nhé, giờ thì nẹt gooo!!!!!!!!!

# I. Giải thích về **Dockerfile**

Quy trình tạo một **image** thật ra khá đơn giản. Giống như khi tán gái vậy, chỉ cần làm theo công thức là được, được gì thì mình chưa chắc 🤣

Cụ thể để tạo **Image**, chúng ta sẽ tạo ra một thứ gọi là **Dockerfile**.

**Dockerfile** thực chất chỉ là một tệp văn bản, bên trong có vài dòng cấu hình.

Những cấu hình này sẽ định nghĩa cách **container** hoạt động, nói rõ nó sẽ chứa những chương trình nào, và khi **container** bắt đầu chạy, nó sẽ làm gì.

![image.png](https://images.viblo.asia/94cce101-70b6-4836-9ddd-972bd73a7118.png)

Sau khi tạo xong **Dockerfile**, chúng ta sẽ chuyển nó cho **Docker client**, tức là CLI mà ta vẫn dùng trong terminal từ trước đến giờ.

Rồi **Docker client** sẽ gửi file này cho **Docker server**. **Docker server** chính là thành phần xử lý công việc nặng cho chúng ta.

Nó sẽ đọc **Dockerfile**, xem từng dòng cấu hình và sau đó xây dựng một **Image** hoàn chỉnh, để ta có thể dùng **image** đó khởi động **container** mới.

![image.png](https://images.viblo.asia/74b45219-1822-4909-9a08-966fc6bd5cb3.png)

Phần **Dockerfile** chính là nơi chứa toàn bộ phần khó của việc này.

Thực tế, hầu hết các **Dockerfile** mà bạn viết ra đều sẽ có cấu trúc gần giống nhau.

Trong mọi **Dockerfile**, ta luôn phải chỉ định:

- Một **base image**: Đây thường là bước đầu tiên khi tạo **Dockerfile**. (Lát nữa chúng ta sẽ nói rõ **base image** là gì.)
- Cấu hình và chạy thêm vài lệnh bổ sung.
- Cài đặt các phần mềm phụ thuộc hoặc chương trình bổ sung, để đảm bảo **container** có đủ những gì cần thiết để chạy ứng dụng của ta.
- Cuối cùng, ta sẽ chỉ định lệnh khởi động (**startup command**) cho **image**.

Mỗi khi ta dùng **image** đó để tạo **container** mới, **startup command** sẽ được chạy tự động, nhằm khởi động **container**, giống chuyện như bật máy vậy.

Ví dụ cho một **Dockerfile**:

![image.png](https://images.viblo.asia/5db24bfd-d25b-408e-be66-b50a1c9398e6.png)

# II. Viết **Dockerfile** đầu tiên

Chúng ta sẽ cùng nhau viết **Dockerfile** song song cùng với việc giải thích để bạn dễ dàng hiểu hơn nhé.

Trước hết, hãy xem thử chúng ta đang định xây dựng gì.

> Chúng ta sẽ tạo một **Dockerfile** để sinh ra một **image**, mà mỗi khi chạy **container** từ **image** đó, nó sẽ khởi động **Redis server**.

Trong bài viết trước chúng ta đã nhiều lần dùng sẵn **image Redis** được tạo bởi người khác để chạy trên máy. Nhưng bây giờ mình sẽ hướng dẫn bạn xây dựng **Redis image** gần như là từ đầu, nghe có vẻ kinh khủng nhưng chỉ có vài dòng thôi bạn đừng lo.

Vì vậy, mục tiêu của **Dockerfile** lần này là tạo ra một **image** có thể chạy **Redis server** khi **container** khởi động.

Giờ thì bắt đầu thôi.

## 1. Thực hành

Trước hết hãy tạo folder thực hành của riêng bạn, sau đó bạn hãy tạo giúp mình một file là `Dockerfile`. Lưu ý là file này không có phần mở rộng gì hết, chỉ có phần tên file thôi nhé.

Giờ thì mình sẽ thêm một vài dòng comment để định hướng chúng ta sẽ làm những gì.

Nó sẽ tuân theo mẫu cấu trúc cơ bản như sau:

```Dockerfile
# 1. Sử dụng một Docker image có sẵn làm base
# 2. Tải và cài đặt dependency
# 3. Chỉ định lệnh sẽ chạy khi container khởi động
```

Giờ chúng sẽ bắt đầu viết mã thực tế trong file này. Đây là nội dung **Dockerfile**:

```Dockerfile
FROM alpine  <- Chỉ định base image
RUN apk add --update redis      <- Tải chương trình
CMD ["redis-server"]        <- Chỉ định lệnh chạy khi container khởi động
```

Giờ chúng ta lưu file lại, mở lại terminal, đảm bảo rằng bạn đang ở trong thư mục mà bạn vừa tạo, và chạy lệnh sau:

```bash
docker build .
```

Dấu chấm ở cuối nghĩa là build **Image** dựa trên **Dockerfile** trong thư mục hiện tại.

Sau khi chạy lệnh, bạn sẽ thấy một loạt thông báo xuất hiện.

Cuối cùng sẽ có dòng như:

```bash
Trường hợp 1: (Phiên bản Docker cũ hơn)
Successfully built <image_id>

Hoặc

Trường hợp 2: (Phiên bản Docker mới)
View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/<build-history-id>
```

Với trường hợp cũ thì bạn sẽ thấy luôn **Image ID** luôn, nhưng với phiên bản mới hơn thì **BuildKit** được bật mặc định nên chúng ta phải thêm một bước nữa.

Gõ theo mình để tìm ra **image ID** chúng ta vừa tạo:

```bash
docker images

hoặc

docker image ls
```

Kết quả:

Ta sẽ copy lại **image ID** của cái mà mới tạo đó, rồi chạy tiếp lệnh:

```bash
docker run <image_id>
```

Kết quả:

Sau khi chạy, bạn sẽ thấy kết quả đầu ra giống như khi dùng **image Redis** chính thức, và dòng cuối cùng sẽ là:

```bash
Ready to accept connections tcp
```

Điều đó nghĩa là **Redis server** đang chạy thành công trong **container** mà bạn vừa build!

![a cartoon drawing of two anime characters ram and rem](https://media.tenor.com/z3uaokB8S7wAAAAi/anime-dancing.gif)

## 2. Giải thích

Bây giờ chúng ta sẽ phân tích kỹ từng bước xem chính xác **Docker** đã làm gì với file và terminal của chúng ta.

### a. Các **Instruction**

Ở trong **Dockerfile** chúng ta đã ví dụ ở trên:

```bash
FROM ...
RUN ...
CMD ...
```

Mỗi dòng trong **Dockerfile** bắt đầu bằng một từ khóa gọi là **instruction**, ví dụ ở đây là `FROM`, `RUN`, và `CMD`.

Mỗi **instruction** nói cho **Docker server** biết phải làm gì để chuẩn bị **image**.

Ví dụ:

- `FROM` → chỉ định **base image** mà ta sẽ build trên đó.
- `RUN` → chạy một lệnh (thường dùng để cài đặt phần mềm, thư viện, v.v.) trong quá trình build **image**.
- `CMD` → chỉ định lệnh mặc định sẽ chạy khi **container** được khởi động từ **image** này.

### b. Các **Arguments**

Sau mỗi **instruction**, ta luôn có một phần tham số là nội dung cụ thể của chỉ thị đó.

Ví dụ:

- `FROM alpine` → **argument** là `alpine` (Chọn **base image** là **Alpine Linux**, một hệ điều hành rất nhẹ).
- `RUN apk add` ... → **argument** là lệnh dùng **package manager** của **Alpine** là **apk** để cài **Redis**.
- `CMD ["echo", "Hello"]` → **argument** là lệnh mặc định khi **container** chạy.

### c. Ghi nhớ

Mỗi dòng trong **Dockerfile** = 1 **instruction** + 1 **argument**.

Ba lệnh quan trọng nhất bạn cần nhớ đó là:

- `FROM` → Chọn **base image**.
- `RUN` → Chạy lệnh khi build.
- `CMD` → Lệnh mặc định khi chạy **container**.

Ngoài ra có một số **Instruction** khác nữa nhưng 3 **Instruction** trên là nền tảng cơ bản bạn cần phải ghi nhớ trước.

### d. Giải thích kỹ về các **Instruction**

Ở phần trên mình chỉ nói sơ qua các **Instruction** mà chưa giải thích kỹ lý do tại sao chúng ta phải làm như vậy, chúng ta sẽ bắt đầu bằng một ví dụ so sánh để dễ hiểu hơn về cấu trúc và mục đích của các dòng cấu hình trong **Dockerfile**.

---

Hãy tưởng tượng, việc viết một **Dockerfile** giống như bạn được đưa cho một chiếc máy tính hoàn toàn mới, chưa có hệ điều hành, và được yêu cầu cài đặt Google Chrome lên đó.

![image.png](https://images.viblo.asia/11c6ad44-b410-4561-aff2-cd87ace21fcb.png)

Giờ hãy nghĩ xem, nếu mình đưa bạn một máy tính trống không có hệ điều hành, và nói “hãy cài Chrome lên đi”, bạn sẽ làm gì?

- Việc đầu tiên bạn làm chắc là bật máy lên.
- Nhưng khi bật lên, màn hình có thể báo lỗi kiểu như: “Không tìm thấy ổ khởi động, không có hệ điều hành nào được cài đặt.”

Tức là cái máy tính trống trơn, chẳng biết phải làm gì. Vì vậy, bước đầu tiên bạn phải làm là cài một **hệ điều hành**.

Chỉ khi có **hệ điều hành**, bạn mới có thể làm các bước tiếp theo, ví dụ như mở trình duyệt, vào trang [chrome.google.com](http://chrome.google.com/), tải trình cài đặt, mở trình quản lý file, và chạy file cài đặt đó.

Tất cả các bước này đều cần có **hệ điều hành** trước đã. Không có **hệ điều hành** thì:

- Bạn không có trình duyệt để tải Chrome.
- Bạn không có File Explorer để mở file.
- Và cũng không có công cụ nào để chạy file .exe.

**Nói cách khác, mọi thao tác đều phụ thuộc vào việc có sẵn một hệ điều hành ban đầu.**

Sau khi cài **hệ điều hành** xong, bạn mới có thể chạy file cài đặt của Chrome và mở được trình duyệt. Quá trình đó rất giống với những gì ta đã làm trong **Dockerfile**.

Khi ta viết:

```bash
FROM alpine
```

Thì việc đó giống như cài đặt một **hệ điều hành** ban đầu cho chiếc máy ảo (**container**) của chúng ta.

Bởi mặc định, khi bạn tạo một **image** mới, nó rỗng hoàn toàn, không có chương trình nào, không có cấu trúc thư mục, không có công cụ để tải hoặc cài đặt phần mềm.

Nên lệnh `FROM alpine` có nhiệm vụ tạo một điểm khởi đầu, một môi trường cơ bản có sẵn một vài chương trình giúp ta tùy chỉnh và cài thêm phần mềm khác.

Giờ bạn có thể thắc mắc:

> “Tại sao lại chọn **Alpine**? **Alpine** là gì?”

Để trả lời, ta hãy liên tưởng lại: tại sao bạn chọn Windows, macOS, Ubuntu, hay bản Linux khác để dùng hằng ngày?

> Câu trả lời là: **bạn chọn hệ điều hành nào tùy theo nhu cầu.**

- Bạn dùng Windows vì nó hỗ trợ phần mềm bạn cần.
- Bạn dùng macOS vì nó có sẵn terminal tiện lợi.

Mỗi **hệ điều hành** đều có một tập hợp chương trình mặc định hữu ích cho mục đích riêng.

Tương tự, ta chọn **Alpine** làm **base image** vì nó cung cấp sẵn một số công cụ rất tiện cho việc cài đặt và chạy **Redis** mà ta đang làm trong khi thực hành bài viết này.

Cụ thể, trong dòng thứ hai của **Dockerfile**, ta có:

```Dockerfile
RUN apk add --update redis
```

**Lưu ý**: Đây không phải là lệnh của **Docker**, mà là một lệnh bên trong **Alpine**. **Apk** là trình **quản lý gói** (**package manager**) của **Alpine** (tên đầy đủ là **Alpine Package Keeper**). Nhờ công cụ này mà ta có thể tự động tải và cài **Redis** từ kho phần mềm của **Alpine**.

Vì vậy, việc chọn **Alpine** làm **base image** vì nó nhẹ, đơn giản, và có sẵn công cụ **apk** để giúp cài **Redis** một cách nhanh chóng.

Tiếp sau đây mình sẽ nói kỹ hơn điều gì đã xảy ra với từng **Instruction** khi ta build.

# III. Giải thích kỹ về các **Instruction** trong quá trình build

Trong phần này, chúng ta sẽ cùng hiểu rõ hơn chính xác điều gì đã xảy ra trong terminal của chúng ta khi chạy lệnh `docker build`.

Chúng ta sẽ đi qua từng bước một, không phải để phân tích cú pháp của từng dòng trong **Dockerfile**, mà để hiểu mỗi dòng đã ảnh hưởng như thế nào đến **image** mà **Docker** tạo ra.

## **1. Hiểu về docker build**

Trước tiên, mình nên nói một chút về lệnh `docker build`.

Khi ta gõ lệnh:

```bash
docker build .
```

Ta đang trao **Dockerfile** cho **Docker client**, và nó sẽ gửi cho **server** để xử lý.

Cụ thể như sau:

- `build` → dùng để tạo **image** từ **Dockerfile**.
- Dấu `.` ở cuối → gọi là **build context**.

**Build context** là toàn bộ tập hợp file và thư mục của dự án, tức là mọi thứ trong thư mục hiện tại. Đó là những gì ta gói lại để **Docker** có thể truy cập trong quá trình build **container**. Sau này, khi **Dockerfile** phức tạp hơn, ta sẽ thấy rõ hơn vai trò của **build context**.

## **2. Mỗi dòng trong Dockerfile là một bước build**

Sau khi chạy lệnh, ta thấy rất nhiều dòng kết quả xuất hiện trong terminal.

Điều đầu tiên cần chú ý là mỗi dòng trong **Dockerfile** tương ứng với một **step**.

Ví dụ:

- **Step 1/2** → Tương ứng dòng đầu tiên trong **Dockerfile**.
- **Step 2/2** → Dòng thứ hai.

Ủa gì vậy bồ? Bồ kêu mỗi dòng **Instruction** là một **step** mà? Vậy sao có 2 **step**, vậy **step** thứ 3 ở đâu?

Thực ra ở phiên bản **Docker** cũ hơn thì nó có 3 **step** thật, bao gồm cả **step** `CMD`. Nhưng khi **Docker** đổi engine từ **Classic builder** sang **BuildKit** thì nó xem `CMD` chỉ là một metadata, một thông tin cho **Image**, không tác động vào quá trình build **image** nên nó sẽ không hiển thị vào.

## **3. Phân tích từng bước**

Ở đây mình muốn nói rõ trước cho mọi người hiểu, vì hiện tại **Docker** đổi engine build từ **Classic builder** sang **BuildKit** nên thứ bạn thấy trên terminal hiện tại là phiên bản mới hơn, và khó hình dung được bản chất vấn đề hơn.

Nên mình sẽ đề xuất mọi người tạm thời tắt **BuildKit** đi, rồi build, sau đó mình sẽ giải thích cơ chế mới ha.

Để tắt tạm thời trong cửa sổ terminal hiện tại bạn gõ như sau:

```bash
#Trên cmd của window
set DOCKER_BUILDKIT=0
docker build .
```

Đối với Linux / macOS

```bash
#Trên bash, zsh, sh... của Linux / macOS
DOCKER_BUILDKIT=0 docker build .
```

Kết quả như hình:

Giờ thì ta sẽ nói về bước ảnh này nhé

### Step 1: `FROM alpine`

Đầu tiên **Docker server** sẽ kiểm tra xem **local cache** (bộ nhớ đệm cục bộ) có sẵn **image** tên là **alpine** chưa.

Nếu chưa có, **Docker** sẽ kết nối tới **Docker Hub** (nơi chứa hàng ngàn **image** công khai) và tải về **image** tên là **alpine**.

Khi hoàn tất, terminal sẽ báo:

```bash
Downloaded newer image for alpine:latest
```

Nghĩa là **base image alpine** đã được tải xong. 👉 Đây là bước khởi đầu đơn giản, chỉ là lấy **base image** để bắt đầu build.

### Step 2: `RUN apk add --update redis`

Đây mới là phần thú vị. Khi **Docker server** đọc dòng `RUN ...`, thì nó sẽ:

- Nhìn lại **image** của bước trước đó (ở đây là **alpine**).
- Tạo một **container tạm thời** từ **image** đó.

Trên terminal, bạn sẽ thấy dòng như:

```bash
Running in <container_id>
```

Nghĩa là có một **container tạm** được tạo ra để thực thi lệnh này. Sau khi lệnh `RUN` kết thúc, bạn thấy dòng:

```bash
Removing intermediate container <container_id>
```

Tức là **Docker** xóa **container tạm thời** đó sau khi đã dùng xong. Bên trong **container** này, **Docker** thực hiện lệnh:

```bash
apk add --update redis
```

Khi lệnh chạy, nó sẽ tải và cài đặt **Redis** vào **hệ thống file** của **container**. Bạn hãy tưởng tượng kiểu như **container** đó có thêm một thư mục mới `/redis`, chứa chương trình **Redis** này vậy.

Sau khi cài xong, **Docker** sẽ:

- Dừng **container** đó.
- Chụp lại toàn bộ **snapshot hệ thống file** (Ở ví dụ này là bao gồm **Redis** vừa được cài).
- Và lưu **snapshot** đó thành một **image tạm** mới (có ID ví dụ như 332ab...).

Kết quả của **step 2** là:

> Một **image** mới chứa đầy đủ những thay đổi và **container tạm** thì bị xóa đi.

### Step 3: `CMD [