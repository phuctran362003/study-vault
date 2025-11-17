# Tạo và build một Docker image sao mà nhiều thứ rắc rối quá zậy???

Trong suốt các bài trước, chúng ta đã sử dụng rất nhiều **Image** được tạo sẵn bởi các kỹ sư khác.

Ví dụ như chúng ta đã dùng image **Hello World**, **Redis**, hay **BusyBox**. Tất cả những image này đều là do người khác tạo ra. Chúng ta chỉ cần tải chúng về máy và tạo container từ đó để chạy.

Bây giờ với bài viết này, chúng ta sẽ bắt đầu học cách tự tạo ra **Image** riêng của mình, để có thể chạy ứng dụng của chính chúng ta trong những container.

Để tạo một **Image** chúng ta sẽ phải xem qua kha khá thứ, nhưng bạn đừng lo, đã có mình ở đây, mình sẽ giúp bạn đi qua những kiến thức này một cách dễ dàng thôi, giờ thì bắt đầu thôi!!!

À mà khoan đã, mong anh em đọc và cho mình 1 upvote và comment nếu thấy cấn cấn để mình cải thiện bài viết nhé, giờ thì nẹt gooo!!!!!!!!!

# I. Giải thích về Dockerfile

Quy trình tạo một **image** thật ra khá đơn giản. Giống như khi tán gái vậy, chỉ cần làm theo công thức là được, được gì thì mình chưa chắc 🤣

Cụ thể để tạo **Image** , chúng ta sẽ tạo ra một thứ gọi là **Dockerfile**.

**Dockerfile** thực chất chỉ là một tệp văn bản, bên trong có vài dòng cấu hình.

Những cấu hình này sẽ định nghĩa cách container hoạt động, nói rõ nó sẽ chứa những chương trình nào, và khi container bắt đầu chạy, nó sẽ làm gì.

![image.png](https://images.viblo.asia/94cce101-70b6-4836-9ddd-972bd73a7118.png)

Sau khi tạo xong Dockerfile, chúng ta sẽ chuyển nó cho Docker client, tức là CLI mà ta vẫn dùng trong terminal từ trước đến giờ.

Rồi Docker client sẽ gửi file này cho Docker server. Docker server chính là thành phần xử lý công việc nặng cho chúng ta.

Nó sẽ đọc **Dockerfile**, xem từng dòng cấu hình và sau đó xây dựng một **Image** hoàn chỉnh, để ta có thể dùng image đó khởi động container mới.

![image.png](https://images.viblo.asia/74b45219-1822-4909-9a08-966fc6bd5cb3.png)

Phần **Dockerfile** chính là nơi chứa toàn bộ phần khó của việc này.

Thực tế, hầu hết các Dockerfile mà bạn viết ra đều sẽ có cấu trúc gần giống nhau.

Trong mọi Dockerfile, ta luôn phải chỉ định:

- Một base image: Đây thường là bước đầu tiên khi tạo Dockerfile. (Lát nữa chúng ta sẽ nói rõ base image là gì.)
- Cấu hình và chạy thêm vài lệnh bổ sung.
- Cài đặt các phần mềm phụ thuộc hoặc chương trình bổ sung, để đảm bảo container có đủ những gì cần thiết để chạy ứng dụng của ta.
- Cuối cùng, ta sẽ chỉ định lệnh khởi động (**startup command**) cho image.

Mỗi khi ta dùng image đó để tạo container mới, **startup command** sẽ được chạy tự động, nhằm khởi động container, giống chuyện như bật máy vậy.

Ví dụ cho một Dockerfile:

![image.png](https://images.viblo.asia/5db24bfd-d25b-408e-be66-b50a1c9398e6.png)

# II. Viết Dockerfile đầu tiên

Chúng ta sẽ cùng nhau viết **Dockerfile** song song cùng với việc giải thích để bạn dễ dàng hiểu hơn nhé.

Trước hết, hãy xem thử chúng ta đang định xây dựng gì.

> Chúng ta sẽ tạo một Dockerfile để sinh ra một image, mà mỗi khi chạy container từ image đó, nó sẽ khởi động Redis server.

Trong bài viết trước chúng ta đã nhiều lần dùng sẵn image Redis được tạo bởi người khác để chạy trên máy. Nhưng bây giờ mình sẽ hướng dẫn bạn xây dựng Redis image gần như là từ đầu, nghe có vẻ kinh khủng nhưng chỉ có vài dòng thôi bạn đừng lo.

Vì vậy, mục tiêu của **Dockerfile** lần này là tạo ra một image có thể chạy Redis server khi container khởi động.

Giờ thì bắt đầu thôi.

## 1. Thực hành

Trước hết hãy tạo folder thực hành của riêng bạn, sau đó bạn hãy tạo giúp mình một file là `Dockerfile`. Lưu ý là file này không có phần mở rộng gì hết, chỉ có phần tên file thôi nhé.

Giờ thì mình sẽ thêm một vài dòng comment để định hướng chúng ta sẽ làm những gì.

Nó sẽ tuân theo mẫu cấu trúc cơ bản như sau:

```Dockerfile
# 1. Sử dụng một Docker image có sẵn làm base
# 2. Tải và cài đặt dependency
# 3. Chỉ định lệnh sẽ chạy khi container khởi động
```

Giờ chúng sẽ bắt đầu viết mã thực tế trong file này. Đây là nội dung Dockerfile:

```Dockerfile
FROM alpine  <- Chỉ định base image
RUN apk add --update redis      <- Tải chương trình
CMD ["redis-server"]        <- Chỉ định lệnh chạy khi container khởi động
```

Giờ chúng ta lưu file lại, mở lại terminal, đảm bảo rằng bạn đang ở trong thư mục mà bạn vừa tạo, và chạy lệnh sau:

```bash
docker build .
```

Dấu chấm ở cuối nghĩa là build Image dựa trên **Dockerfile** trong thư mục hiện tại.

Sau khi chạy lệnh, bạn sẽ thấy một loạt thông báo xuất hiện.

Cuối cùng sẽ có dòng như:

```bash
Trường hợp 1: (Phiên bản Docker cũ hơn)
Successfully built <image_id>

Hoặc

Trường hợp 2: (Phiên bản Docker mới)
View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/<build-history-id>
```

Với trường hợp cũ thì bạn sẽ thấy luôn Image ID luôn, nhưng với phiên bản mới hơn thì **BuildKit** được bật mặc định nên chúng ta phải thêm một bước nữa.

Gõ theo mình để tìm ra image ID chúng ta vừa tạo:

```bash
docker images

hoặc

docker image ls
```

Kết quả:

Ta sẽ copy lại image ID của cái mà mới tạo đó, rồi chạy tiếp lệnh:

```bash
docker run <image_id>
```

Kết quả:

Sau khi chạy, bạn sẽ thấy kết quả đầu ra giống như khi dùng image Redis chính thức, và dòng cuối cùng sẽ là:

```bash
Ready to accept connections tcp
```

Điều đó nghĩa là Redis server đang chạy thành công trong container mà bạn vừa build!

![a cartoon drawing of two anime characters ram and rem](https://media.tenor.com/z3uaokB8S7wAAAAi/anime-dancing.gif)

## 2. Giải thích

Bây giờ chúng ta sẽ phân tích kỹ từng bước xem chính xác Docker đã làm gì với file và terminal của chúng ta.

### a. Các Instruction

Ở trong Dockerfile chúng ta đã ví dụ ở trên:

```bash
FROM ...
RUN ...
CMD ...
```

Mỗi dòng trong Dockerfile bắt đầu bằng một từ khóa gọi là instruction, ví dụ ở đây là `FROM`, `RUN`, và `CMD`.

Mỗi instruction nói cho Docker server biết phải làm gì để chuẩn bị image.

Ví dụ:

- `FROM` → chỉ định base image mà ta sẽ build trên đó.
- `RUN` → chạy một lệnh (thường dùng để cài đặt phần mềm, thư viện, v.v.) trong quá trình build image.
- `CMD` → chỉ định lệnh mặc định sẽ chạy khi container được khởi động từ image này.

### b. Các Arguments

Sau mỗi instruction, ta luôn có một phần tham số là nội dung cụ thể của chỉ thị đó.

Ví dụ:

- `FROM alpine` → argument là `alpine` (Chọn base image là Alpine Linux, một hệ điều hành rất nhẹ).
- `RUN apk add` ... → argument là lệnh dùng package manager của Alpine là apk để cài Redis.
- `CMD ["echo", "Hello"]`→ argument là lệnh mặc định khi container chạy.

### c. Ghi nhớ

Mỗi dòng trong **Dockerfile** = 1 instruction + 1 argument.

Ba lệnh quan trọng nhất bạn cần nhớ đó là:

- `FROM` → Chọn base image.
- `RUN` → Chạy lệnh khi build.
- `CMD` → Lệnh mặc định khi chạy container.

Ngoài ra có một số Instruction khác nữa nhưng 3 Instruction trên là nền tảng cơ bản bạn cần phải ghi nhớ trước.

### d. Giải thích kỹ về các Instruction

Ở phần trên mình chỉ nói sơ qua các Instruction mà chưa giải thích kỹ lý do tại sao chúng ta phải làm như vậy, chúng ta sẽ bắt đầu bằng một ví dụ so sánh để dễ hiểu hơn về cấu trúc và mục đích của các dòng cấu hình trong **Dockerfile**.

---

Hãy tưởng tượng, việc viết một Dockerfile giống như bạn được đưa cho một chiếc máy tính hoàn toàn mới, chưa có hệ điều hành, và được yêu cầu cài đặt Google Chrome lên đó.

![image.png](https://images.viblo.asia/11c6ad44-b410-4561-aff2-cd87ace21fcb.png)

Giờ hãy nghĩ xem, nếu mình đưa bạn một máy tính trống không có hệ điều hành, và nói “hãy cài Chrome lên đi”, bạn sẽ làm gì?

- Việc đầu tiên bạn làm chắc là bật máy lên.
- Nhưng khi bật lên, màn hình có thể báo lỗi kiểu như: “Không tìm thấy ổ khởi động, không có hệ điều hành nào được cài đặt.”

Tức là cái máy tính trống trơn, chẳng biết phải làm gì. Vì vậy, bước đầu tiên bạn phải làm là cài một hệ điều hành.

Chỉ khi có hệ điều hành, bạn mới có thể làm các bước tiếp theo, ví dụ như mở trình duyệt, vào trang [chrome.google.com](http://chrome.google.com/), tải trình cài đặt, mở trình quản lý file, và chạy file cài đặt đó.

Tất cả các bước này đều cần có hệ điều hành trước đã. Không có hệ điều hành thì:

- Bạn không có trình duyệt để tải Chrome.
- Bạn không có File Explorer để mở file.
- Và cũng không có công cụ nào để chạy file .exe.

**Nói cách khác, mọi thao tác đều phụ thuộc vào việc có sẵn một hệ điều hành ban đầu.**

Sau khi cài hệ điều hành xong, bạn mới có thể chạy file cài đặt của Chrome và mở được trình duyệt. Quá trình đó rất giống với những gì ta đã làm trong **Dockerfile**.

Khi ta viết:

```bash
FROM alpine
```

Thì việc đó giống như cài đặt một hệ điều hành ban đầu cho chiếc máy ảo (container) của chúng ta.

Bởi mặc định, khi bạn tạo một image mới, nó rỗng hoàn toàn, không có chương trình nào, không có cấu trúc thư mục, không có công cụ để tải hoặc cài đặt phần mềm.

Nên lệnh `FROM alpine` có nhiệm vụ tạo một điểm khởi đầu, một môi trường cơ bản có sẵn một vài chương trình giúp ta tùy chỉnh và cài thêm phần mềm khác.

Giờ bạn có thể thắc mắc:

> “Tại sao lại chọn Alpine? Alpine là gì?”

Để trả lời, ta hãy liên tưởng lại: tại sao bạn chọn Windows, macOS, Ubuntu, hay bản Linux khác để dùng hằng ngày?

> Câu trả lời là: **bạn chọn hệ điều hành nào tùy theo nhu cầu.**

- Bạn dùng Windows vì nó hỗ trợ phần mềm bạn cần.
- Bạn dùng macOS vì nó có sẵn terminal tiện lợi.

Mỗi hệ điều hành đều có một tập hợp chương trình mặc định hữu ích cho mục đích riêng.

Tương tự, ta chọn Alpine làm base image vì nó cung cấp sẵn một số công cụ rất tiện cho việc cài đặt và chạy Redis mà ta đang làm trong khi thực hành bài viết này.

Cụ thể, trong dòng thứ hai của **Dockerfile**, ta có:

```Dockerfile
RUN apk add --update redis
```

**Lưu ý**: Đây không phải là lệnh của Docker, mà là một lệnh bên trong Alpine. Apk là trình quản lý gói (package manager) của Alpine (tên đầy đủ là Alpine Package Keeper). Nhờ công cụ này mà ta có thể tự động tải và cài Redis từ kho phần mềm của Alpine.

Vì vậy, việc chọn Alpine làm base image vì nó nhẹ, đơn giản, và có sẵn công cụ **apk** để giúp cài Redis một cách nhanh chóng.

Tiếp sau đây mình sẽ nói kỹ hơn điều gì đã xảy ra với từng Instruction khi ta build.

# III. Giải thích kỹ về các Instruction trong quá trình build

Trong phần này, chúng ta sẽ cùng hiểu rõ hơn chính xác điều gì đã xảy ra trong terminal của chúng ta khi chạy lệnh `docker build`.

Chúng ta sẽ đi qua từng bước một, không phải để phân tích cú pháp của từng dòng trong **Dockerfile**, mà để hiểu mỗi dòng đã ảnh hưởng như thế nào đến image mà Docker tạo ra.

## **1. Hiểu về docker build**

Trước tiên, mình nên nói một chút về lệnh `docker build`.

Khi ta gõ lệnh:

```bash
docker build .
```

Ta đang trao Dockerfile cho Docker client, và nó sẽ gửi cho server để xử lý.

Cụ thể như sau:

- `build` → dùng để tạo image từ Dockerfile.
- Dấu `.` ở cuối → gọi là **build context**.

**Build context** là toàn bộ tập hợp file và thư mục của dự án, tức là mọi thứ trong thư mục hiện tại. Đó là những gì ta gói lại để Docker có thể truy cập trong quá trình build container. Sau này, khi **Dockerfile** phức tạp hơn, ta sẽ thấy rõ hơn vai trò của build context.

## **2. Mỗi dòng trong Dockerfile là một bước build**

Sau khi chạy lệnh, ta thấy rất nhiều dòng kết quả xuất hiện trong terminal.

Điều đầu tiên cần chú ý là mỗi dòng trong **Dockerfile** tương ứng với một **step**.

Ví dụ:

- **Step 1/2** → Tương ứng dòng đầu tiên trong **Dockerfile**.
- **Step 2/2** → Dòng thứ hai.

Ủa gì vậy bồ? Bồ kêu mỗi dòng Instruction là một **step** mà? Vậy sao có 2 step, vậy step thứ 3 ở đâu?

Thực ra ở phiên bản Docker cũ hơn thì nó có 3 step thật, bao gồm cả step `CMD`. Nhưng khi Docker đổi engine từ **Classic builder** sang **BuildKit** thì nó xem `CMD` chỉ là một metadata, một thông tin cho Image, không tác động vào quá trình build image nên nó sẽ không hiển thị vào.

## **3. Phân tích từng bước**

Ở đây mình muốn nói rõ trước cho mọi người hiểu, vì hiện tại Docker đổi engine build từ **Classic builder** sang **BuildKit** nên thứ bạn thấy trên terminal hiện tại là phiên bản mới hơn, và khó hình dung được bản chất vấn đề hơn.

Nên mình sẽ đề xuất mọi người tạm thời tắt **BuildKit** đi, rồi build, sau đó mình sẽ giải thích cơ chế mới ha.

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

### Step 1: `FROM alpine`

Đầu tiên **Docker server** sẽ kiểm tra xem local cache (bộ nhớ đệm cục bộ) có sẵn image tên là **alpine** chưa.

Nếu chưa có, Docker sẽ kết nối tới **Docker Hub** (nơi chứa hàng ngàn image công khai) và tải về image tên là **alpine**.

Khi hoàn tất, terminal sẽ báo:

```bash
Downloaded newer image for alpine:latest
```

Nghĩa là base image alpine đã được tải xong. 👉 Đây là bước khởi đầu đơn giản, chỉ là lấy base image để bắt đầu build.

### Step 2: `RUN apk add --update redis`

Đây mới là phần thú vị. Khi Docker server đọc dòng `RUN ...`, thì nó sẽ:

- Nhìn lại image của bước trước đó (ở đây là alpine).
- Tạo một container tạm thời từ image đó.

Trên terminal, bạn sẽ thấy dòng như:

```bash
Running in <container_id>
```

Nghĩa là có một **container tạm** được tạo ra để thực thi lệnh này. Sau khi lệnh `RUN` kết thúc, bạn thấy dòng:

```bash
Removing intermediate container <container_id>
```

Tức là Docker xóa container tạm thời đó sau khi đã dùng xong. Bên trong container này, Docker thực hiện lệnh:

```bash
apk add --update redis
```

Khi lệnh chạy, nó sẽ tải và cài đặt Redis vào hệ thống file của container. Bạn hãy tưởng tượng kiểu như container đó có thêm một thư mục mới `/redis`, chứa chương trình Redis này vậy.

Sau khi cài xong, Docker sẽ:

- Dừng container đó.
- Chụp lại toàn bộ snapshot hệ thống file (Ở ví dụ này là bao gồm Redis vừa được cài).
- Và lưu snapshot đó thành một image tạm mới (có ID ví dụ như 332ab...).

Kết quả của step 2 là:

> Một image mới chứa đầy đủ những thay đổi và container tạm thì bị xóa đi.

### Step 3: `CMD ["redis-server"]`

Tiếp theo Docker lại:

- Lấy image vừa tạo từ bước 2 (Lúc này đã có Redis).
- Tạo thêm một container tạm thời từ image đó.

Sau đó, lệnh `CMD` được dùng không phải để chạy ngay, mà để đặt cấu hình startup mặc định cho image. Nói cách khác, Docker chỉ ghi chú rằng:

> Khi container được khởi chạy thật sự, lệnh mặc định cần thực thi là `redis-server`.

Rồi container tạm thời này cũng bị dừng, Docker chụp lại snapshot của file system + lệnh khởi động mặc định, và tạo ra image cuối cùng.

### Tóm tắt toàn bộ quy trình build

Như vậy, trong quá trình docker build, mỗi dòng trong **Dockerfile** đều làm theo chu trình:

1. Lấy image của bước trước.
2. Tạo container tạm từ image đó.
3. Thực thi lệnh (nếu có) trong container.
4. Chụp snapshot hệ thống file (và nếu có, cả lệnh `CMD` khởi động).
5. Tạo image mới từ snapshot đó.
6. Xóa container tạm.

Và cứ như thế, bước sau kế thừa kết quả của bước trước, cho đến khi hết các dòng trong **Dockerfile**. Image cuối cùng chính là image hoàn chỉnh mà bạn build ra được.

## **4. Cơ chế cache trong Docker build**

Trước đó chúng ta đã có nhắc tới khá nhiều về việc cache trong quá trình build, cũng giống như việc bạn cache lại data ở phía BE hoặc FE giúp làm giảm thiểu số lần truy vấn xuống database và tăng thời gian phản hồi, thì việc cache trong quá trình build image cũng có ý nghĩa tương tự.

Nó sẽ cache lại những step chúng ta đã làm, chỉ thực sự build những step có sự thay đổi với lần trước để tăng tốc quá trình build và giảm thiểu khả năng lỗi có thể xảy ra. Đây là thứ giúp Docker có được hiệu suất cực kỳ cao khi tạo image mới.

Hãy cùng xem lại `Dockerfile` của chúng ta:

```bash
FROM alpine
RUN apk add --update redis
CMD ["redis-server"]
```

Giờ chúng ta sẽ thử tính năng cache khi build của Docker như sau, bạn thêm một dòng instruction `RUN`, với nội dung đầy đủ của file sẽ là như sau:

```bash
FROM alpine
RUN apk add --update redis
RUN apk add --update gcc   <---- Dòng mới
CMD ["redis-server"]
```

Bạn không cần quan tâm `apk add --update gcc` dùng để làm gì, bạn chỉ cần hiểu rằng mục tiêu của chúng ta là xem việc cache hoạt động như thế nào khi ta thêm một instruction mới và build lại image.

Và giờ chúng ta sẽ gõ lệnh build lại

```bash
docker build .
```

Ta được kết quả sau:

Hãy để ý kỹ chỗ mình khoanh đỏ

```bash
 => CACHED [2/3] RUN apk add --update redis
```

Ở đây Docker đã cache lại cho chúng ta step 2 mà chúng ta build hồi nãy, bây giờ nó chỉ cần build step 3 rồi cache lại nữa là xong. Tương tự, bạn thử gõ build một lần nữa xem:

```bash
 => CACHED [2/3] RUN apk add --update redis
 => CACHED [3/3] RUN apk add --update gcc
```

Bạn sẽ thấy rằng nó sẽ sử dụng lại cache build của 2 step trước đó mà không cần build lại nữa.

### a. Cách xem thử build cache

Bạn có thể xem các cache layer của mình thông qua lệnh sau:

```bash
docker builder du
```

Chúng ta sẽ có kết quả như sau:

```bash
ID                      RECLAIMABLE  SIZE      LAST USED
kz4d9ynk4s3z            true         123MB     5 hours ago
z7sq6r8j1v2p            false        54MB      2 hours ago
total:                  177MB
```

Ý nghĩa các cột như sau:

|Cột|Giải thích|
|---|---|
|**ID**|Mã định danh của cache (build cache object)|
|**RECLAIMABLE**|`true` nghĩa là cache đó có thể xóa được (không bị ràng buộc bởi image nào đang dùng)|
|**SIZE**|Dung lượng cache|
|**LAST USED**|Thời điểm cuối cùng cache được dùng trong một lần build|

### b. Cách xóa build cache

Bạn có thể xóa tất cả các cache bằng lệnh sau:

```bash
docker builder prune -a
```

Docker sẽ hỏi xác nhận (`Are you sure you want to continue? [y/N]`), bạn hãy gõ y rồi enter.

Lệnh này xóa toàn bộ cache build gồm các layer tạm, dữ liệu trung gian, build context…

Còn nếu bạn muốn xóa chỉ định những cache layer mà bạn muốn ta có thể dùng lệnh sau:

```bash
docker buildx prune --filter id=<cache_id>
```

## **5. BuildKit**

Như mình đã giải thích ở trước đó BuildKit là một build engine của Docker ra mắt từ phiên bản 23.0 để thay thế build engine cũ khá cồng kềnh và chậm chạp, bạn có thể thấy nhược điểm ở trên là các step luôn tốn thời gian tạo và xóa container tạm một cách tuần tự và điều này thì nó cực kỳ tốn thời gian. Bạn có thể đọc qua Docs ở đây: [https://docs.docker.com/build/buildkit/](https://docs.docker.com/build/buildkit/) . Mình sẽ giải thích sơ qua cách làm của engine mới nhé:

Như mình đã nói ở trên, build engine cũ bị một vấn đề là việc tạo xóa tuần tự container tạm là một việc rất tốn thời gian, nên ở engine mới thì Docker có một cách tiếp cận khác.

BuildKit có 3 thành phần chính:

- **Frontend**: Dịch Dockerfile thành LLB graph (Low-Level Build – Một biểu đồ các bước và dependency).
- **LLB graph**: Một biểu đồ giúp xem bước nào độc lập có thể chạy song song.
- **Worker backend**: Khởi chạy mỗi bước trong sandbox riêng biệt (Có thể là container nhẹ hơn hoặc “chroot namespace” tùy môi trường, nhưng không cần tạo container thật qua Docker daemon.)

Kết quả của mỗi bước sau đó được lưu thành cache layer.

![image.png](https://images.viblo.asia/a642974f-7127-4d09-b9d3-c47275e00117.png)

Với cơ chế mới này thì BuildKit có những ưu điểm sau:

- Build song song.
- Cache thông minh.
- Secret và SSH an toàn.

Thế nên ở bản mới này khá là khó để bạn hình dung cách tạo ra một Image thực tế lúc ban đầu như thế nào, nên mình mới khuyên các bạn dùng build engine cũ để cho dễ hình dung.

# IV. Gắn thẻ tên (Tag) cho các build image

Tại sao chúng ta lại cần đặt tên cho các image, tại sao việc này lại cần thiết?

Bây giờ chúng ta thử gõ lệnh này nhé:

```bash
# Liệt kê các image đang có của chúng ta

docker images

hoặc 

docker image ls
```

Kết quả:

![image.png](https://images.viblo.asia/ef1f6cf8-8785-4a97-910b-d4dd5a487ef2.png)

Hồi nãy chúng ta đã chạy 2, 3 lần lệnh build image, kết quả là sinh ra được các image tương ứng với các ID khác nhau.

Chúng ta sẽ phải ghi nhớ hoặc viết lại ở đâu đó để đánh dấu rằng với mỗi ID ta sẽ có những mục đích khác nhau, và điều này thực sự rất là một cách tệ để làm việc đó.

Nên từ đó chúng ta đã có khái niệm là "Tagging", bạn có thể gắn thẻ tên cho image của mình và lần sau gọi lại thì chỉ cần viết lại chúng thui!

Một tag có một **quy ước chung** như sau:

```none
<docker-id>/<tên-dự-án>:<phiên-bản>
```

Trong đó:

- Phần docker ID là tài khoản của bạn trên Docker Hub.
- Phần tên dự án là tùy bạn thích đặt sao thì đặt.
- Và sau dấu hai chấm là phiên bản của image. (Thông thường, khi build phiên bản mới nhất, bạn sẽ đặt là latest. Nhưng thực tế thì giá trị này nên là số phiên bản, ví dụ 1.0, 2.1...)

Hầu hết các dự án Docker đều tuân theo quy ước này.

Nói là hầu hết nhưng bạn thấy đấy, các image khác ví dụ như redis, hello-world, hay busybox lại chỉ có mỗi tên ngắn gọn mà không cần chỉ định quá nhiều với docker-id.

Đó là vì chúng là các image cộng đồng được tạo và chia sẻ công khai bởi những người dùng khác trong cộng đồng Docker. Chúng là những image được public rất phổ biến. Còn khi bạn tự tạo image riêng của mình, bạn luôn phải thêm tiền tố là Docker ID của bạn ở trước.

Oke, vậy để gắn thẻ tên chúng ta sẽ dùng biến cờ `-t` và giá trị phía sau thì tuân theo quy tắc trên, các bạn có thể gõ như sau:

```bash
docker build -t <docker-id>/<tên-dự-án>:latest .
```

Ví dụ:

Kết quả:

![image.png](https://images.viblo.asia/98b91176-545e-4f3a-bcc1-4090de007f0d.png)

**Lưu ý: Mình đã che Docker id của mình lại rồi nên bạn chỉ thấy được tag của image thôi nha!**

Lúc này bạn đã có thể thấy được cả version và tên image bạn vừa tạo rồi đó, EZ đúng không nào?

# V. Đảo ngược tu tiên (Docker commit)

Trước khi chúng ta tiếp tục, có một thứ thú vị nhỏ mình muốn chỉ cho bạn trong toàn bộ quá trình build image này.

**Lưu ý những gì mình sắp chỉ cho bạn trong phần này không phải là thứ bạn sẽ thường xuyên làm, thậm chí có thể sẽ không bao giờ cần làm sau này. Nhưng mình nghĩ nó sẽ giúp bạn hiểu rõ hơn mối quan hệ giữa image và container trong Docker.**

Khi chúng ta nói về chuỗi hành động xảy ra trong lúc build image, mình có nói rằng:

- Ở mỗi bước, Docker sẽ lấy image từ bước trước.
- Tạo ra một container từ image đó.
- Thực hiện một số thao tác trong container.
- Và sau đó tạo một image mới từ container đang chạy tạm thời đó.

Đến giờ, chúng ta đều hiểu rằng image được dùng để tạo container. Nhưng nếu nhìn kỹ lại, có vẻ chiều ngược lại cũng đúng, tức là từ container ta cũng có thể tạo ra image. Nếu bạn nhìn lại:

![image.png](https://images.viblo.asia/0e408727-1d51-4687-ace8-dc2511eb2757.png)

Rõ ràng trong các bước thì container tạo ra image như bình thường. Chúng ta hoàn toàn có thể tự tạo một container thủ công, chạy một số lệnh trong đó, thay đổi hệ thống file bên trong, rồi tạo một image mới có thể sử dụng sau này.

Nói cách khác, ta có thể tự tay thực hiện lại toàn bộ những gì mà Dockerfile làm, nhưng theo một cách thủ công hơn.

Ví dụ: ta có thể tạo một container, rồi chạy lệnh `apk add redis` để cài Redis, và cuối cùng tạo một image mới từ container đó. Giờ thì chúng ta cùng thử nha, ta sẽ có các bước như sau:

- Tạo một container mới thủ công từ image Alpine.
- Cài Redis bên trong.
- Đặt default command.
- Và cuối cùng là tạo ra image mới có thể dùng về sau.

Tất cả những bước này chính là những gì Dockerfile làm, nhưng giờ chúng ta sẽ làm thủ công nhé.

Mở terminal, bạn hãy gõ lệnh:

```bash
docker run -it alpine sh
```

Ngay sau đó chúng ta sẽ thấy dấu thăng bên trong container, tức là ta đang thao tác trong container thật.

![image.png](https://images.viblo.asia/92e13adc-41f2-4ef2-938a-692e02c899bf.png)

Bây giờ trong container này, chúng ta sẽ cài Redis thủ công bằng lệnh:

```bash
apk add --update redis
```

![image.png](https://images.viblo.asia/a62a18fb-57df-45a0-b01b-22faa13664ca.png)

Lúc này container đang chạy của chúng ta đã bị thay đổi hệ thống file, cụ thể là Redis đã được cài đặt vào đó.

Giờ chúng ta sẽ mở một cửa sổ terminal thứ hai, **lưu ý là không đóng terminal vừa nãy nhé!**

Trước hết chúng ta sẽ kiểm tra xem container có đang chạy không bằng lệnh:

```bash
docker ps
```

![image.png](https://images.viblo.asia/0d8c1b25-e911-4a82-9c31-5f981ee74a2f.png)

Oke, chúng ta đã thấy được container chúng ta chạy, giờ hãy cũng tìm hiểu lệnh `docker commit` này cần những tham số gì rồi mình giải thích sau nha. Gõ theo mình:

```bash
docker commit --help
```

![image.png](https://images.viblo.asia/814dee1b-307c-49e6-a1fe-26e9875d7d10.png)

Chúng ta sẽ nhìn vào hình, chỗ mình khoanh đỏ là cách sử dụng của lệnh này, còn phía dưới là mô tả về công dụng, cũng như các cách khác để thực thi lệnh này.

Lệnh này sẽ giúp chúng ta tạo một image từ container chúng ta chỉ định.

Ở đây chúng ta sẽ có 4 option, nhưng chúng ta chỉ cần quan tâm đến option -c, cài các default command cho image.

Ở cuối lệnh docker commit thì gắn tag cho nó thế là xong, giờ chúng ta cùng thử ha:

```bash
#Dành cho CMD
docker commit -c "CMD [\"redis-server\"]" <container_id> <docker-id>/<name>:<version>

#PowerShell
docker commit -c "CMD ['redis-server']" <container_id> <docker-id>/<name>:<version>
```

Ta được như sau:

![image.png](https://images.viblo.asia/d23e787d-ff75-49ee-b250-197c5b2af5a5.png)

Lúc này bạn đã có được image rồi, giờ thì thử chạy nó xem nào!

**Nhắc lại:** Trong thực tế, bạn không nên dùng `docker commit` để tạo image. Hãy luôn dùng Dockerfile, vì nó giúp bạn lặp lại quy trình build một cách dễ dàng, có kiểm soát và có thể tái sử dụng sau này.

# VI. Truyền nội công (COPY instruction)

Okay chúng ta đã sắp đi hết chặng đường rồi, ở phần này mình sẽ hướng dẫn cho các bạn thêm một instruction mới mà chúng ta sẽ thường viết trong một `Dockerfile`.

Để ví dụ sát với thực tế, chúng ta sẽ triển khai viết một Dockerfile để tạo một NodeJS app nhé.

## 1. Set up dự án

Vẫn trong thư mục cũ, các bạn làm theo hướng dẫn của trang ExpressJS đây ha: [https://expressjs.com/en/starter/installing.html](https://expressjs.com/en/starter/installing.html) (Yêu cầu bạn đã cài NodeJS trước đó nhé)

Về cơ bản chúng ta sẽ có dự án kiểu như sau:

![image.png](https://images.viblo.asia/7f297781-7eb9-435e-826c-9891c5433a7f.png)

Trong package.json mình chỉ thêm script này để chạy cho quen tay thui `"start": "node index.js"`

## 2. Minh họa các bước làm

Các bước mà chúng ta sẽ viết trong `Dockerfile` này sẽ rất giống với những gì chúng ta đã làm trước đó khi tạo image cho Redis.

Dockerfile của chúng ta sẽ cần:

- Chỉ định base image.
- Chạy lệnh để cài dependencies. Trong trường hợp này là `npm install`
- Khai báo lệnh mặc định khi container khởi động, tức là `npm start`

Khi làm với image Redis trước đó, ta đã:

- Dùng `FROM alpine` để chỉ định base image.
- Dùng `apk add redis` để cài đặt Redis.
- Và đặt default command để chạy server Redis.

Giờ đây, với Node.js, quy trình gần như tương tự:

- Dùng `FROM ...`.
- **BƯỚC ẨN MÀ CHÚNG TA SẼ TÌM HIỂU Ở PHẦN 4.**
- Chạy `npm install` để cài dependencies.
- Và đặt lệnh mặc định là `npm start` để khởi động ứng dụng khi container chạy.

Vậy nên, chiến lược lần này là tạo một file **Dockerfile** và viết vào đó các lệnh tương tự như vậy.

## 3. Bắt đầu viết Dockerfile

### a. FROM

Giờ chúng ta sẽ bắt đầu với instruction đầu tiên đó là `FROM` ha, đây sẽ là nơi chúng ta chọn base image đầu tiên.

Ở đây bạn sẽ có 2 lựa chọn:

- Dùng một image đã có sẵn NodeJS làm base image.
- Dùng một image chưa có NodeJS sẵn (Như các ví dụ trước là `alpine`), và dùng instruction để cài NodeJS lên đó trước.

Nghe thôi thì bạn cũng biết là chọn cái dễ rồi đúng không, mắc mọe gì chọn cái khó làm chi, vậy giờ chúng ta sẽ tìm một image có sẵn NodeJS sẵn nhé! OKay giờ bạn sẽ check sơ qua docker hub giúp mình ha: [https://hub.docker.com/_/node](https://hub.docker.com/_/node)

Ở ngay đầu trang chỗ `Supported tags and respective Dockerfile links`, bạn sẽ thấy được hàng loạt các phiên bản NodeJS image mà bạn có thể sử dụng.

Nếu ứng dụng của bạn cần Node v25, bạn có thể dùng tag 25, và trong Dockerfile, bạn sẽ viết:

```Dockerfile
FROM node:25
```

Docker sẽ tự tải về image NodeJS chứa Node.js v25 được cài sẵn.

Vì đây là phần ví dụ thôi nên chúng ta sẽ một phiên bản image nào nó nhỏ gọn nhẹ để phù hợp với tiêu chí thực hành này của chúng ta thôi nhé.

Và trong Docker thì các tag `alpine` nghĩa là phiên bản nhẹ nhất, nhỏ gọn nhất của image đó. Nhiều repository phổ biến như Node, Python, Postgres, v.v. đều có phiên bản `alpine` để giảm dung lượng tối đa.

![image.png](https://images.viblo.asia/917e647f-da65-41a8-8b06-8434cb7c6902.png)

Ví dụ image NodeJS thông thường có thể bao gồm:

- git.
- curl.
- Một số package manager.
- Vài trình soạn thảo cơ bản.

Còn node:alpine thì loại bỏ tất cả những thứ đó, chỉ giữ lại phần core như:

- Node.js
- npm
- Và một số lệnh cơ bản của Linux (ls, cat, echo, v.v.)

Vì vậy, ta sẽ chọn image `node:alpine` làm base image.

Vậy dòng đầu tiên của file Dockerfile sẽ là:

```Dockerfile
FROM node:alpine
```

## 4. Bước ẩn mở khóa

Đây là instruction mới mà mình muốn giới thiệu với các bạn, như từ bài đầu nói về Docker tới giờ, mình và các bạn đã hiểu được một container luôn luôn được cô lập ở phần cứng, tách biệt với máy thật của chúng ta, chúng ta muốn tương tác được với container thì phải thông qua một số lệnh của Docker.

Vậy giờ chúng ta sẽ có một câu hỏi, chúng ta muốn container đọc được code của app chúng ta và chạy nó lên thì phải làm như thế nào?

- Một là container sẽ trỏ tới đọc file thật của chúng ta đang làm. (Cách này cực kỳ rủi ro vì nếu như chiếm được quyền điều khiển container thì nó có thể thông qua mối liên kết này để tiến hành hack máy chúng ta luôn.)
- Hai là chúng ta đưa một bản sao code của chúng ta vào trong container, rồi cho nó chạy. (Cách này giống việc bạn code ở máy mình, sau đó nén dự án lại file sau đó ném qua máy khác giải nén rồi chạy vậy.)

![image.png](https://images.viblo.asia/8613ebb7-2867-435d-b52c-d70a108cc720.png)

Và đương nhiên rồi, chúng ta sẽ chọn cách hai rồi. Chúng ta sẽ copy dự án của chúng ta mà không bao gồm các thư mục thư viện tải về như `node_modules`, bởi vì chúng ta sẽ đưa code sang trước rồi dùng instruction để khởi chạy lệnh cài dependency sau, chứ dại gì mang một cục to sang chi cho mất thời gian đúng không?

Để copy được dự án, chúng ta sẽ dùng instruction sau:

```Dockerfile
COPY <đường_dẫn_trên_máy> <đường_dẫn_trong_container>
```

Ví dụ:

```Dockerfile
COPY ./ ./
```

Ở đây, dấu `./` nghĩa là nói rằng đây là thư mục hiện tại.

Bây giờ chúng ta có một chi tiết kỹ thuật nhỏ cần chú ý đó là cái <đường_dẫn_trên_máy> tức đối số đầu tiên của `COPY` được tính tương đối so với build context.

Bạn còn nhớ không, khi ta chạy lệnh build:

```bash
docker build .
```

Dấu chấm `.` ở cuối lệnh đó chính là **build context**. Nó nói với Docker rằng:

> “Hãy lấy thư mục hiện tại làm nơi chứa các file có thể copy vào image.”

Vì vậy trong lệnh `COPY ./ ./` phần `./` đầu tiên có nghĩa là toàn bộ thư mục hiện tại của chúng ta, và phần `./` thứ hai nghĩa là thư mục gốc bên trong container.

Tóm lại `COPY ./ ./` sẽ copy mọi file trong thư mục dự án hiện tại vào container.

Vậy bây giờ file Dockerfile sẽ là:

```Dockerfile
FROM node:alpine
COPY ./ ./
```

## 5. Hoàn thiện Dockerfile và thêm .dockerignore

Bây giờ thì chúng ta chỉ cần thêm instruction để cài dependency và đặt một default command cho image nữa thôi là được:

```Dockerfile
FROM node:alpine
COPY ./ ./
RUN npm install
CMD ["npm", "start"]
```

Giống như mình nói ở trên, chúng ta copy thì chỉ cần copy những file cần thiết thôi, những thứ có thể cài hoặc làm bên container mà làm quá trình copy dài ra thì phải loại ra trong quá trình copy, khá giống với việc bạn config file .gitignore để đưa lên repo thì bạn đâu cần push node_modules làm gì đúng không?

Bây giờ chúng ta sẽ tạo một file tên là `.dockerignore` cùng cấp với Dockerfile với nội dung như sau:

```dockerignore
node_modules
```

Quá đẹp, giờ thì mở terminal và build thôi nào:

```bash
docker build -t <repo-name-tùy-bạn-đặt> .               <- Ở đây bạn chỉ cần chỉ định repo name thôi, còn docker-id vì chưa đẩy lên Docker hub nên chưa cần, version mặc định là latest 
```

Kết quả:

![image.png](https://images.viblo.asia/4f628dee-ad4f-4df5-a532-cfcfd12f7428.png)

Bây giờ tất cả các dependency của npm đã được cài đặt thành công!

Giờ đến bước cuối cùng, chúng ta sẽ chạy container từ image đó.

```bash
docker run <repo-name-tùy-bạn-đặt>
```

Kết quả:

![image.png](https://images.viblo.asia/6cc270b8-1132-4baf-aca5-97e8042d7aec.png)

Chúng ta thử vào [http://localhost:3000/](http://localhost:3000/) xem thử kết quả nào:

![image.png](https://images.viblo.asia/1e7191b4-20db-4c8f-952d-ed7d95230de0.png)

What? Chuyện quái gì đang xảy ra vậy, tại sao đã chạy thành công và đã mở đúng port rồi mà vẫn không vào được?

Vậy là tuy container đã chạy, và server bên trong đang hoạt động, nhưng chúng ta vẫn không thể truy cập từ bên ngoài.

Để giải quyết vấn đề này, chúng ta sẽ đi tới phần tiếp theo nhé!

## 6. Ánh xạ port

Chắc bạn cũng đoán ra được lý do tại sao chúng ta không thể truy cập vào [http://localhost:3000/](http://localhost:3000/) rồi, về cơ bản container và máy chúng ta là khác nhau, nên không có chuyện có thể truy cập được port của container thông qua máy mình dễ đến thế.

![image.png](https://images.viblo.asia/e4bf8de8-c662-476f-8190-cadf8a2a51a0.png)

Do đó, để request từ máy tính hoặc từ trình duyệt có thể đi vào container, ta cần thiết lập port mapping một cách rõ ràng.

Port mapping nghĩa là:

> Bất cứ khi nào có request đến một port cụ thể trên máy thật, hãy tự động chuyển tiếp request đó đến một port bên trong container.

Ví dụ:

Nếu có ai đó truy cập localhost:3000, thì ta có thể cấu hình để Docker chuyển yêu cầu này vào port 3000 trong container, nơi mà ứng dụng Node của chúng ta đang lắng nghe và xử lý request.

![image.png](https://images.viblo.asia/ee0b5bc6-fcfd-428a-a4d3-75081369d357.png)

Một điều rất quan trọng mà bạn cần nhớ đó là:

**Câu chuyện port mapping này chỉ liên quan đến các request đi vào. Container của Docker vẫn có thể tự do gửi request ra ngoài Internet.**

Chúng ta đã thấy điều đó khi chạy `npm install`, khi đó Docker đã tải các gói npm từ mạng mà không gặp vấn đề gì.

Vì vậy:

- Gửi ra ngoài (outgoing) → luôn được phép.
- Nhận vào container (incoming) → cần thiết lập port mapping.

Để cấu hình port mapping, chúng ta sẽ không chỉnh trong **Dockerfile**, mà chỉ thay đổi khi chạy container.

Nói cách khác, port mapping là một thiết lập runtime, chỉ áp dụng lúc container được khởi chạy, không phải lúc build image.

Cú pháp để chạy container với port mapping là:

```bash
docker run -p <port_máy_thật>:<port_trong_container> <image_name>
```

Giờ ta quay lại terminal để thực hành. Trước hết, dừng container đang chạy bằng cách nhấn `Ctrl + C`.

Sau đó, chạy lại container nhưng lần này thêm cờ `-p`:

```bash
docker run -p 3000:3000 <image_name>
```

Ta được kết quả sau:

![image.png](https://images.viblo.asia/822d7e0f-c04a-47e9-8594-fd9ac81c3164.png)

Giờ thì ứng dụng của chúng ta đã chạy ngon nghẻ rồi nhá!

Tuy nhiên, có một điểm thú vị cần lưu ý thêm:

**Hai con số port không nhất thiết phải giống nhau.**

Ví dụ, ta có thể chạy:

```bash
docker run -p 5000:3000 <tên-image>
```

Câu này nghĩa là:

> “Bất kỳ request nào đến localhost:5000 trên máy thật, hãy chuyển đến port 3000 bên trong container.”

Đây là cách mà rất nhiều ứng dụng thực tế sử dụng, port bên ngoài có thể khác hoàn toàn so với port bên trong.

## 7. Bonus 1: Instruction `WORKDIR`

Khi copy toàn bộ dự án vào container theo cách hiện tại, chúng ta vô tình khiến cấu trúc thư mục bên trong container trở nên không gọn gàng và khó quản lý, nếu bạn chưa tin thì hãy xem đây, khi mình đi vào container và list các thư mục bằng lệnh sau:

```bash
docker exec -it <container-id> sh

/ # ls

Dockerfile         etc                lib                node_modules       package.json       run                sys                var
bin                home               media              opt                proc               sbin               tmp
dev                index.js           mnt                package-lock.json  root               srv                usr
```

Các file thì nằm tứ lung tung, nếu dự án của ta có thư mục trùng tên với các thư mục hệ thống như `var`, `root`, `run`, hoặc `lib` ta có thể vô tình ghi đè các thư mục mặc định của hệ thống, điều đó chắc chắn là không nên.

Vì vậy ta sẽ chỉnh lại **Dockerfile** một chút. Thay vì copy vào `/` (Thư mục gốc), ta sẽ copy vào một thư mục con mà ta sẽ tự tạo.

instruction `WORKDIR` sẽ cho phép ta đặt thư mục làm việc mặc định bên trong container. Tất cả các lệnh sau đó (`COPY`, `RUN`, v.v.) sẽ được thực thi tương đối theo thư mục này.

Nói cách khác, nếu ta đặt:

```Dockerfile
WORKDIR /usr/app
```

và sau đó thực hiện `COPY . .`, thì các file sẽ được copy vào `/usr/app`, chứ không phải `/` nữa.

Hãy thử áp dụng ngay, thêm dòng sau vào Dockerfile, trước phần `COPY`:

```Dockerfile
WORKDIR /usr/app
```

Nếu thư mục `/usr/app`chưa tồn tại, Docker sẽ tự tạo nó.

Tại sao lại chọn `/usr/app`?

Thực ra trong thế giới Node.js, vị trí này không quá quan trọng. Thư mục `/usr` là nơi an toàn để đặt ứng dụng, vì nó dành cho các dữ liệu người dùng. Một số người dùng Linux “chuẩn” có thể tranh luận rằng nên để ở `/var` hoặc `/home`, nhưng `/usr/app` vẫn là lựa chọn rất ổn và phổ biến.

Bây giờ ta thoát khỏi container bằng lệnh exit, sau đó rebuild lại image:

```bash
docker build -t <repo-name> .
```

Sau đó chạy lại container với image vừa tạo.

```bash
docker run -p 3000:3000 <repo-name>
```

Mở một terminal thứ hai, tiến hành xâm nhập vào container:

```bash
#Xem container chúng ta vừa chạy để lấy id
docker ps

docker exec -it <container-id> sh
```

Kết quả:

![image.png](https://images.viblo.asia/6e21a31a-6c42-4aa8-89a3-ad06a4c8eb59.png)

Khi vào trong, bạn sẽ thấy chúng ta đang ở ngay thư mục `/usr/app`, vì Docker đã đặt đây là working directory mặc định.

Nếu ta gõ ls, sẽ thấy toàn bộ file dự án nằm trong `/usr/app`.

Còn nếu `cd /` rồi `ls` lại, bạn sẽ thấy không còn file dự án nào ở thư mục gốc nữa.

![image.png](https://images.viblo.asia/31d3c572-4c41-42ca-8faa-7f2163ec9021.png)

Rất gọn gàng và an toàn. Ứng dụng của ta được cô lập trong thư mục riêng, không gây xung đột với hệ thống nữa.

## 8. Bonus 2: Cách tận dụng cache khi build của Docker

Cho đến hiện tại, những thứ chúng ta đang viết trong Dockerfile là không sai, nhưng nó chưa tối ưu khi gặp trường hợp thế này:

Giả sử bạn thay đổi một tí ti ở code, sau đó chúng ta phải build lại image, ở đây nó chỉ cache lại quá trình build được tới bước `WORKDIR`:

```Dockerfile
FROM node:alpine # Đã cache

WORKDIR /usr/app <- Chỉ cache được tới đây

COPY ./ ./ <- Tới đây đã có sự thay đổi nên phải bắt đầu build lại từ đây

RUN npm install

CMD ["npm", "start"]
```

![image.png](https://images.viblo.asia/493cb626-0aba-4296-b3fc-60423921d57e.png)

Tức là dù thay đổi code 1 tí, nhưng cứ mỗi lần thay đổi là phải `npm install` lại, đáng lý cái này phải được đặt trước cái `COPY ./ ./`, để mà code có thay đổi thì phần dependency đã được cache và không cần chạy lại, từ đó sẽ làm giảm thời gian build.

Do đó, ở bước copy đầu tiên, ta chỉ copy file package.json vào container.

```Dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY package.json .

...
```

Sau đó ta chạy npm install để cài đặt dependencies.

```Dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY package.json .

RUN npm install

...
```

Khi cài xong rồi, ta mới copy toàn bộ source code vào container:

```Dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY package.json .

RUN npm install

COPY . .

CMD ["npm", "start"]
```

Giờ ta thử build lại, sau đó thay đổi file index.js, rồi lại build lại, ta được kết quả sau:

![image.png](https://images.viblo.asia/2d2c108a-047d-438c-8bc3-17d012d7bdce.png)

Kết quả build lần này ta đã thấy nó build cực kỳ nhanh vì đã không còn cần phải install lại các dependency nữa, quá tuyệt đúng không nào?

![a girl with a bandage on her eye is pointing at something](https://media.tenor.com/z3WvqfHGlloAAAAi/anime-dab.gif)

# VII. Tổng kết phần 3

Và đó là tất cả những gì cơ bản nhất về Docker, thực sự đây là một bài viết rất rất dài, mọi người có thể sẽ thấy ngợp với lượng thông tin mà mình cung cấp, nhưng mà dài dài vậy mới đã đúng không :3

Và qua phần 4 mình sẽ tới với một nội dung cực kỳ thú vị đó chính là Kubernetes, đây coi như là một bước đà chuẩn bị cho những kiến thức sắp tới.

Mình xin nhắc lại là bài viết của mình không thể thay thế được những kiến thức khóa học từ anh Sờ Te Phờn, các bạn hãy đăng ký và trải nghiệm thử.

Nếu bạn thấy hay thì cho mình xin 1 upvote và để lại comment những chỗ mình đã sai, hoặc muốn thảo luận thêm nhé.