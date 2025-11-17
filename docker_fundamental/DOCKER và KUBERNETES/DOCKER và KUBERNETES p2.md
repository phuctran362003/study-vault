# Những đường quyền Docker đầu tiên!!!

Ở bài trước chúng ta đã đi qua các lý thuyết cơ bản về Docker và Kubernetes, nên ở bài viết này chúng ta sẽ động tay động chân một tí, đó là đi tìm hiểu về các lệnh để tương tác với Docker Client.

Trước khi đi sâu vào bài viết, mình muốn nhắc lại nhanh một điều quan trọng:

> Mỗi khi bạn chạy lệnh `docker run <tên-image>` Docker sẽ tạo ra một container mới dựa trên bản snapshot hệ thống có trong image đó.

Và sau khi container được tạo, Docker sẽ tự động thực thi một lệnh mặc định - **Default command** được định nghĩa sẵn bên trong image (Giống như chuyện bạn đã `npm install` các thứ rồi thì việc bạn cần làm bây giờ là gõ `npm run start`, thì đây chính là một ví dụ về **Default command** cho bạn dễ hiểu).

Chúng ta sẽ mở đầu bài viết bằng cách nghịch 1 tí với **Default command** này.

# I. Ghi đè Default command

Giờ chúng ta sẽ xem cách chạy `docker run` nhưng ghi đè lệnh mặc định đó bằng lệnh khác của riêng mình.

Cách làm rất đơn giản:

_Sau phần tên image, bạn chỉ cần thêm một lệnh khác. Docker sẽ chạy lệnh đó thay vì lệnh mặc định trong image._

Ví dụ:

```arduino
docker run <tên-image> <lệnh-mới>
```

Lệnh bạn thêm vào chính là lệnh override, và lệnh mặc định trong image sẽ bị bỏ qua.

## 1. Thử nghiệm với BusyBox

Giờ hãy thử luôn nhé. Chúng ta mở terminal và gõ theo mình:

```arduino
docker run busybox echo "Hi there"
```

Ở đây, BusyBox là một image rất nhỏ gọn, mình sẽ giải thích lý do chọn nó vào một tí nữa.

Kết quả:

![image.png](https://images.viblo.asia/99361cad-e81a-415a-a2eb-e338e538fe51.png)

Phần echo "Hi there" chính là lệnh **override**, nó sẽ chạy bên trong container. Kết quả là, bạn sẽ thấy dòng chữ "Hi there" được in ra terminal.

Bạn có thể thay đổi nội dung này tuỳ ý, ví dụ:

```arduino
docker run busybox echo "Bye there"
```

hoặc

```arduino
docker run busybox echo "How are you?"
```

Nói chung, **echo** sẽ in ra bất kỳ chuỗi nào bạn truyền vào.

## 2. Thử với lệnh `ls`

Giờ ta thử một lệnh thú vị hơn thay vì chỉ **echo**. Hãy gõ theo mình:

```arduino
docker run busybox ls
```

Nếu bạn chưa biết, thì giờ bạn sẽ biết, lệnh `ls` sẽ liệt kê tất cả file và thư mục trong thư mục hiện tại.

Khi chạy, bạn sẽ thấy nó in ra các thư mục như:

![image.png](https://images.viblo.asia/d1221b91-58cd-4a03-98af-8fe1230ff7f9.png)

Nếu bạn dùng Windows, những tên thư mục này có thể trông rất lạ, nhưng đây không phải là thư mục của máy bạn.

Đây là các thư mục tồn tại bên trong container, tức là bên trong hệ thống file của BusyBox.

## 3. Giải thích lại quá trình tạo container

Chúng ta hãy cùng nhớ lại nhé, khi bạn tạo container từ một image, Docker sẽ:

**1. Lấy snapshot hệ thống file có trong image.**

**2. Dán nó vào container như là ổ đĩa của container.**

**3. Thực thi lệnh khởi động (default hoặc override) bên trong container đó.**

Trong ví dụ này:

- Image busybox có sẵn các thư mục như `/bin, /dev, /etc, /home, /proc, /root,` v.v.
- Khi bạn chạy `docker run busybox ls`, Docker tạo container mới, dùng snapshot này làm file system, rồi chạy lệnh `ls` → và nó liệt kê ra chính các thư mục đó.

## 4. Vì sao không dùng hello-world image?

Bạn có thể thắc mắc rằng:

> “Tại sao ta không dùng image hello-world như trước?”

Ta cùng thử nhé:

```arduino
docker run hello-world ls
```

hoặc

```arduino
docker run hello-world echo "Hi there"
```

Kết quả là bạn sẽ thấy lỗi.

![image.png](https://images.viblo.asia/f96544b3-8c1a-45fb-b9d8-a2bffa77257b.png)

**Lý do:**

Các lệnh như `ls` hoặc `echo` không tồn tại bên trong image hello-world.

Image hello-world cực kỳ tối giản, nó chỉ chứa một chương trình duy nhất dùng để in ra thông điệp chào mừng. Không có bất kỳ công cụ hay lệnh cơ bản nào khác.

Trong khi đó, image busybox là một image Linux siêu nhỏ gọn, được thiết kế để gom hàng chục lệnh cơ bản của Linux vào một tệp duy nhất. Cụ thể như sau, trên hệ điều hành Linux thông thường (ví dụ Ubuntu, Debian, Alpine...), chúng có các lệnh command sau:

```bash
ls
cat
echo
sh
pwd
grep
cp
mv
```

Mỗi cái là một chương trình riêng nằm trong `/bin` hoặc`/usr/bin`.

Nhưng với BusyBox, tất cả những lệnh này được gộp vào chung trong một file thực thi duy nhất (tên là busybox). Tùy bạn gọi `busybox ls`, `busybox echo`, hoặc chỉ `ls`, `echo`, thì nó sẽ biến hình để thực thi hành vi tương ứng.

Cho nên mới có chuyện là bạn có thể ghi đè `echo` hay `ls` ở bên busybox thì được nhưng ở image hello-world thì không đấy nha!!!

# II. Liệt kê tất cả các container đang chạy

## 1. `docker ps`

Trong phần này, chúng ta sẽ tiếp tục tìm hiểu một lệnh được dùng rất thường xuyên. Lệnh mà chúng ta sẽ xem đó chính là `docker ps`.

Lệnh này dùng để liệt kê tất cả các container đang chạy trên máy của bạn. Hãy thử chạy nó ngay bây giờ để xem chuyện gì xảy ra nhé.

Mình mở terminal và gõ:

```bash
docker ps
```

Khi chạy lệnh này, bạn sẽ thấy một bảng có các cột tiêu đề.

![image.png](https://images.viblo.asia/014c4481-74a0-4509-9ae3-651db80f0b9f.png)

Hiện tại thì không có container nào đang chạy, nên bảng này đang trống.

Lý do là từ đầu đến giờ, ta chỉ chạy những container chạy rất nhanh rồi thoát ngay.

Ví dụ, khi ta chạy:

```bash
docker run busybox echo "Hi there"
```

Thì container đó chỉ khởi động, in ra dòng “Hi there”, rồi ngay lập tức dừng lại.

Vì thế, `docker ps` không thấy gì, vì nó chỉ hiển thị container đang chạy, không bao gồm container đã dừng.

Nếu ta muốn `docker ps` hiển thị được container nào đó, thì cần một container chạy lâu hơn một chút.

Ta có thể làm điều đó bằng cách thay đổi lệnh được thực thi khi container khởi động.

Thay vì `echo "Hi there"`, ta hãy thử:

```bash
docker run busybox ping google.com
```

Lệnh ping sẽ gửi các gói tin đến máy chủ Google và đo độ trễ (latency).

Nó chạy liên tục, nên container sẽ vẫn hoạt động trong một thời gian dài.

Bạn sẽ thấy đầu ra kiểu như:

![image.png](https://images.viblo.asia/d6f722dc-3706-4816-ae72-d995e73f9c34.png)

Tức là container đang chạy lệnh ping.

Bây giờ, ta mở một cửa sổ terminal thứ hai, rồi gõ:

```bash
docker ps
```

Bạn sẽ thấy container vừa tạo xuất hiện trong danh sách.

![image.png](https://images.viblo.asia/b56acfe2-da64-48d6-a484-75c9be6e26b1.png)

Trong bảng đó, bạn sẽ thấy các thông tin sau:

- CONTAINER ID → mã định danh ID của container.
- IMAGE → tên image dùng để tạo container.
- COMMAND → lệnh đang chạy.
- CREATED → thời gian container được tạo.
- STATUS → trạng thái hiện tại
- PORTS → cổng được mở ra (Hiện tại chưa có, ta sẽ biết nó sau).
- NAMES → tên ngẫu nhiên mà Docker đặt (ví dụ: gifted_gates)

Quay lại cửa sổ terminal đang chạy ping, ta có thể nhấn `Ctrl + C` để dừng container.

Lúc đó Docker sẽ kết thúc container và bạn quay lại dòng lệnh bình thường.

Nếu giờ bạn chạy lại:

```bash
docker ps
```

→ Container đó không còn xuất hiện nữa. Vì `docker ps` chỉ hiển thị container đang chạy.

## 2. `docker ps --all`

Nếu bạn muốn xem tất cả container đã từng được tạo (kể cả đã dừng), thì dùng:

```bash
docker ps --all
```

hoặc viết tắt như sau:

```bash
docker ps -a
```

Lệnh này sẽ hiển thị danh sách mọi container trong quá khứ, bao gồm:

- Container nào đang chạy.
- Và cả container đã dừng (trạng thái: “Exited”).

Trong bảng đó, bạn sẽ thấy:

![image.png](https://images.viblo.asia/b35eed4c-498b-45cf-98f6-6cfd4f694bf0.png)

- ID, Image, Command, Created, Status (Exited)
- Ports (nếu có)
- Tên container ngẫu nhiên mà Docker gán.

Trong thực tế, `docker ps` là một lệnh rất hay dùng để xem container nào đang chạy, hoặc để lấy ID của container đang chạy đó.

Vì nhiều lệnh Docker khác (như `docker stop, docker logs, docker exec..`.) cần bạn chỉ rõ container bằng ID hoặc tên.

# III. Vòng đời của một container

Trong phần II, chúng ta đã chạy lệnh `docker ps` và xem một biến thể của nó là `docker ps --all`.

Khi chạy lệnh `--all`, ta thấy rằng nó liệt kê tất cả các container mà ta từng khởi chạy trên máy, điều này khá thú vị, vì nó sẽ khiến ta tự hỏi:

- Khi nào một container thực sự bị tắt?
- Tại sao nó bị tắt?
- Và chuyện gì xảy ra khi nó bị tắt?

Bắt đầu phần này, mình muốn cho bạn thấy vòng đời của một container. Điều này sẽ giúp bạn hiểu rõ hơn chuyện gì đang diễn ra đằng sau nó.

Trước khi tìm hiểu chuyện gì xảy ra khi container bị tắt, ta hãy đi ngược lại từ lúc nó được tạo ra để xem chuyện gì xảy ra khi một container được khởi tạo lần đầu.

## 1. Điều tra hiện trường

Cho đến giờ, ta đã biết rằng để khởi chạy một container từ một image, ta dùng lệnh:

```bash
docker run
```

Nhưng nếu bạn nhớ lại những gì chúng ta đã học và làm nãy giờ, bạn sẽ thấy một điều đó là:

> “Tạo container” (Creating) và “chạy container” (Running) là hai quá trình riêng biệt.

Điều đó nghĩa là có hai lệnh khác ngoài docker run mà ta có thể dùng để khởi chạy container mới. Thực ra, chạy `docker run` tương đương với việc chạy hai lệnh liên tiếp:

```bash
docker create <tên-image>

và

docker start <container-id>
```

Dựa vào tên của nó bạn có thể đoán được mục đích của từng lệnh:

- `docker create` dùng để tạo một container từ image.
- `docker start` dùng để chạy container đó.

Lúc này, bạn có thể sẽ thắc mắc:

> “Khác biệt giữa tạo container và chạy container là gì?”

Hãy xem sơ đồ sau để hiểu rõ hơn.

![image.png](https://images.viblo.asia/3d6db9fe-ca0d-42f0-a581-cef8e8749e33.png)

Một image gồm có:

- Một bản chụp hệ thống file (file system snapshot).
- Và một startup command.

Khi tạo container, Docker sẽ chuẩn bị hệ thống file từ image này để sẵn sàng dùng cho container mới.

Nói cách khác, bước **create** chỉ là chuẩn bị môi trường file system. (Bước 1 trong ảnh)

Còn khi khởi động container, Docker sẽ thực thi lệnh startup bên trong nó, ví dụ như in “Hello World”, hoặc trong image busybox mà ta từng dùng, nó chạy lệnh `echo "Hi there"`. (Bước 2 trong ảnh)

Tóm lại:

- `docker create`: chuẩn bị hệ thống file.
- `docker start`: thực thi lệnh khởi động.

Hãy giữ nó trong đầu một chút, giờ thì hãy quay lại terminal để thử nghiệm xem sự khác biệt này thực tế như thế nào.

Trên terminal, mình sẽ thử dùng `docker create` và `docker start` với image hello-world.

Đầu tiên, chúng ta sẽ chạy:

```bash
docker create hello-world
```

Kết quả:

Khi chạy lệnh này, Docker in ra một chuỗi ký tự dài, đó là ID của container vừa được tạo. Bạn hãy copy nó lại bằng cách bôi đen và nhấn tổ hợp `Ctrl + Shift + C`.

Bây giờ, chúng ta có thể chạy lệnh bên trong container đó bằng cách gõ:

```bash
docker start -a <container_id>
```

(Biến cờ `-a`, lát nữa mình sẽ giải thích nó làm gì.)

Sau đó, chúng ta dán ID container vừa nhận được và chạy lệnh trên.

![image.png](https://images.viblo.asia/e72705fc-875a-472e-bdec-5bbc7086e3e4.png)

Khi chạy xong, ta thấy dòng chào quen thuộc “Hello from Docker!”.

## 2. Suy luận thông tin

Vậy chuyện gì vừa xảy ra?

- Đầu tiên, Docker chuẩn bị hệ thống file cho container (bước create).
- Sau đó, Docker chạy lệnh khởi động bên trong container (bước start).

Giờ, còn biến cờ `-a` kia là gì? Hãy thử lại mà không có `-a`, ta sẽ chạy lại `docker start <container_id>` (không thêm gì khác).

![image.png](https://images.viblo.asia/67495c7b-bef9-4e87-b030-e332bccc47b9.png)

Kết quả là, Docker chỉ in ra ID của container, chứ không hiển thị nội dung đầu ra (output).

Vì vậy, cờ `-a` có nghĩa là:

> Gắn terminal của chúng ta vào container, để hiển thị mọi output từ container đang chạy trực tiếp ra màn hình terminal của mình và bạn. (Lúc này bạn nên hiểu là terminal của bạn hiện tại và terminal của container là 2 cái khác nhau, muốn hiện kết quả terminal của container khi chạy ta phải thêm biến cờ `-a`).

Nếu không có `-a`, container vẫn chạy, nhưng bạn sẽ không thấy kết quả đầu ra.

Do đó, sự khác biệt nhỏ này cũng chính là điểm khác giữa `docker run` và `docker start`:

- `docker run` mặc định hiển thị toàn bộ log và output từ container.
- `docker start` mặc định KHÔNG hiển thị bất kỳ thông tin nào ra terminal. (Trừ khi bạn thêm -a.)

Vậy là ta vừa thấy sự khác biệt giữa `docker run`, `docker create`, và `docker start` rồi đó.

# IV. Chạy lại một container đã bị dừng lại

Trong phần III, chúng ta đã bắt đầu tìm hiểu vòng đời của một container bằng cách sử dụng hai lệnh `docker create` và `docker start`.

Giờ chúng ta sẽ tiếp tục tìm hiểu về trạng thái của những container mà ta đã từng chạy trên máy, nhưng hiện tại đã bị dừng lại vì một lý do nào đó.

Ta có thể liệt kê tất cả các container, bao gồm cả những cái đã dừng bằng lệnh:

```bash
docker ps --all
```

Bây giờ ta sẽ thấy container vừa chạy:

Trong ảnh trên:

- Lệnh thực thi là `echo "Hi there"`.
- Và trạng thái là **Exited**.

Khi một container ở trạng thái **Exited**, điều đó không có nghĩa là nó ngủm hẳn hay không dùng được nữa. Ta vẫn có thể khởi động lại nó bất cứ lúc nào.

Để làm vậy, ta lấy ID của container đó rồi chạy:

```bash
docker start -a <container_id>
```

Khi chạy lệnh này, bạn sẽ thấy dòng "Hi there" được in ra một lần nữa.

Khá thú vị, phải không?

Hãy xem sơ đồ để hiểu rõ hơn về chuỗi lệnh mà ta vừa thực hiện.

![image.png](https://images.viblo.asia/94254780-073a-491b-bb98-14a9052b5a4e.png)

Ta có image busybox, và khi ta chạy `docker create` hoặc `docker run`.

Docker lấy bản chụp hệ thống file (file system snapshot) của image đó và liên kết nó vào container mới được tạo.

Sau đó, ta ghi đè lệnh mặc định bằng lệnh:

```bash
echo "Hi there"
```

Lệnh đó chính trở thành lệnh chính được chạy bên trong container.

Khi lệnh `echo` hoàn thành, container tự động dừng lại (Exited).

Sau đó, khi ta chạy lại:

```bash
docker start <container_id>
```

Docker chạy lại cùng một lệnh mặc định đó lần nữa, nghĩa là echo "Hi there" được thực thi lại.

Điều quan trọng bạn cần nhớ là:

### Khi một container đã được tạo, ta không thể thay đổi lệnh mặc định của nó nữa.

Nói cách khác, ta không thể làm như sau:

```bash
docker start -a <container_id> echo "Bye there"
```

Vì điều này đang cố ghi đè lại lệnh mặc định, điều mà Docker không cho phép đối với container đã nạp lệnh mặc định.

Nếu bạn thử chạy, Docker sẽ hiểu nhầm là bạn đang khởi động nhiều container cùng lúc, và sẽ báo lỗi.

**Tóm lại:**

- **Khi container được tạo, lệnh mặc định của nó sẽ được cố định.**
- **Khi bạn khởi động lại, Docker chạy lại đúng lệnh mặc định đó.**

Có thể bạn cảm thấy phần này hơi chi tiết và rườm rà, nhưng thật ra hiểu rõ **vòng đời** của container là rất quan trọng. Nó sẽ giúp bạn debug và xử lý sự cố hiệu quả hơn nhiều sau này khi làm việc với các container thực tế.

# V. Cách xóa container đã dừng

Có thêm là phải có xóa, tính mình thấy cái gì bừa bừa bẩn bẩn là phải dọn dẹp, nên mình sẽ hướng dẫn mọi người cách xóa container đã dừng nhé!

![](https://media.tenor.com/yp2dMubnVZ4AAAAi/attack-on-titan-levi.gif)

## 1. Cách xóa những container mà mình muốn chỉ định

Đầu tiên hãy gõ lệnh `docker ps -a` để xem các container nào có trạng thái existed mà mình muốn xóa nhé!

![image.png](https://images.viblo.asia/da0e5ece-1a9c-4630-9ceb-bfa34feeb135.png)

Giờ chúng ta sẽ thấy có một container có ID là "7ecd1e44c2dd" và name là "intelligent_jemison" (Lưu ý là cái này là đối với máy mình thôi nhé, máy bạn sẽ có ID và tên khác nhé).

Bạn có thể copy Name hoặc ID của (những) container mà bạn muốn xóa và gõ lệnh sau:

```bash
docker rm <name-1> <name-2> <name-3>

hoặc 

docker rm <id-1> <id-2> <id-3>
```

Kết quả:

Ở đây mình đã copy tên của container và chạy, sau đó kiểm tra lại danh sách container thì đã không còn nữa.

## 2. Cách xóa cả lò nhà container

Để xóa toàn bộ các container đang chiếm dung lượng ổ đĩa mà không làm gì cả, chúng ta có thể chạy lệnh `docker system prune`.

Sau khi chạy lệnh, Docker sẽ hiển thị một cảnh báo.

Lưu ý rằng lệnh này không chỉ xóa container đã dừng, mà còn xóa thêm một vài thứ khác, đặc biệt là bộ nhớ đệm khi build (build cache).

Build cache ở đây là các cache layer của image mà bạn đã tải từ Docker Hub hoặc tự build, phần cache này mình sẽ nói kỹ hơn ở phần sau, bạn chỉ cần hiểu ngắn gọn đó là Build cache là dữ liệu mà Docker lưu lại từ các bước build image trước đó để tăng tốc độ build lại image.

Điều này cũng thực sự không quá nghiêm trọng đâu, chỉ là bạn sẽ phải chờ thêm vài giây khi khởi động lại container thôi.

Sau khi đọc cảnh báo, chỉ cần gõ “y” (yes) rồi nhấn Enter, Docker sẽ tiến hành xóa các container, và hiển thị thông tin về những container đã bị xóa, cũng như dung lượng ổ đĩa được giải phóng.

Sau đó, nếu bạn chạy lại lệnh:

```bash
docker ps --all
```

Bạn sẽ thấy không còn container dừng nào nữa.

Mình rất khuyên bạn ghi nhớ lệnh `docker system prune` bởi vì mỗi khi bạn ngừng sử dụng Docker một thời gian dài (vài tuần hoặc vài tháng), hoặc nếu bạn không định dùng Docker nữa, thì hãy chạy lệnh này để dọn dẹp tất cả container và dữ liệu dư thừa, tránh việc chúng chiếm dụng dung lượng ổ đĩa vô ích.

# VI. Cách xem output log

Ở phần IV, chúng ta đã nói về sự khác nhau giữa **create** và **start** một container. Trong ví dụ, ta đã dùng lệnh:

```bash
docker create busybox echo "Hi there"
```

Sau đó Docker in ra ID của container vừa tạo, và ta có thể chạy container đó bằng lệnh:

```bash
docker start <container_id>
```

Tuy nhiên, có một điểm cần chú ý. Khi chạy `docker start`, mặc định bạn sẽ không thấy output của container. Muốn xem, bạn phải thêm cờ -a, ví dụ như:

```bash
docker start -a <container_id>
```

Giờ chúng ta thử tưởng tượng, việc chạy container này mất vài phút để hoàn tất.

Nếu bạn quên thêm `-a`, bạn sẽ không thấy kết quả đầu ra, và phải chạy lại lệnh `docker start -a` một lần nữa, nghĩa là lại phải chờ thêm vài phút, khá phiền phức đúng chứ?

Để tránh chuyện đó, Docker có một lệnh rất hữu ích là:

```bash
docker logs <container_id>
```

Lệnh này cho phép bạn xem lại toàn bộ log với container đã tạo ra trước đó, mà không cần khởi động lại container.

Ví dụ nhé, hãy gõ theo mình:

```bash
docker create busybox echo "Hi there"
```

Docker sẽ in ra ID container.

Sau đó ta lưu ID đó lại rồi chạy:

```bash
docker start <container_id>
```

Lúc này bên trong container sẽ chạy lệnh `echo "Hi there"`, in ra chuỗi “Hi there” bên trong, và sau đó kết thúc ngay lập tức vì lệnh echo đã hoàn thành.

Giờ ta muốn xem lại nội dung đã in ra bên trong container đó, thì chỉ cần chạy:

```bash
docker logs <container_id>
```

Kết quả hiển thị sẽ là:

```bash
Hi there
```

Điều bạn cần nhớ đó là:

- `docker logs` chỉ đọc lại log, **không khởi động lại** container.
- Nó rất hữu ích khi bạn muốn debug hoặc kiểm tra xem container đã chạy và in ra những gì.

**Tóm lại:**

- `docker start -a` → vừa chạy container vừa xem output trực tiếp.
- `docker logs` → xem lại output sau khi container đã chạy xong.

Chúng ta sẽ cần dùng `docker logs` khá nhiều trong thực tế, nên mình mong bạn nạp nó vào bộ nhớ não bộ thật kỹ nhé :>>

# VII. Cách dừng một container chuẩn anime

Trong các lệnh Docker mà ta vừa học như `create` và `start`, có một điểm hơi **lạ** mà mình sẽ cho bạn thấy nhanh thôi.

Ở phần xóa container mình luôn dùng từ là xóa một container **đã dừng** chứ không phải một container **đang chạy**.

Giả sử ta tạo và chạy một container như sau:

```bash
docker run busybox ping google.com
```

Lệnh này sẽ tạo ra một container chạy lệnh ping [google.com](http://google.com/) (Bạn nên nhớ `docker run` là lệnh kết hợp giữa `docker create + docker start -a` nhé).

Ta sẽ thấy container đó đang ping [google.com](http://google.com/) liên tục. Và nếu ta kiểm tra bằng cách mở một terminal mới và gõ:

```bash
docker ps
```

Thì sẽ thấy container đang chạy và vẫn thực hiện lệnh ping.

## 1. Vấn đề: Làm sao để xóa container đang chạy?

Nếu bạn cố gắng xóa một container đang chạy bằng lệnh `docker rm` sẽ có một thông báo lỗi như sau:

Vậy nên việc chúng ta cần làm là phải dừng container trước, nhưng chúng ta phải làm thế nào?

Lệnh `ping` chạy vô hạn, nên container cũng sẽ không tự dừng. Ta cần gửi cho nó một tín hiệu để báo rằng:

> “Này ping, dừng lại đi, đừng có gọi Google nữa!”

Có hai cách để dừng một container đang chạy, cả hai là đều dừng container, nhưng khác nhau ở cách gửi tín hiệu:

## 2. `docker stop` em gái loli nhẹ nhàng

![](https://media.tenor.com/UkK1G2I0EisAAAAj/ast-anime.gif)

Tưởng tượng `docker stop` là đứa em gái loli dễ thương của bạn, em gái bạn giúp bạn:

- Gửi tín hiệu **SIGTERM** (terminate) tới tiến trình chính trong container.
- Báo với tiến trình trong container kiểu “Hãy tự tắt đi khi Tiến trình-kun rảnh nhé, nhưng mà nhớ làm sạch dữ liệu rồi mới thoát nhá.”
- Cho phép chương trình có thời gian dọn dẹp hoặc lưu dữ liệu trước khi thoát.
- Nhiều ứng dụng có thể lắng nghe tín hiệu này để xử lý trước khi tắt.

Nếu sau 10 giây mà container vẫn chưa dừng, Docker sẽ tự động chuyển sang gửi **SIGKILL** (Một tín hiệu của cách thứ 2).

Tức là `docker stop` là cách em gái nhẹ nhàng, lịch sự, nhưng Docker sẽ chỉ đợi 10 giây trước khi ép buộc.

## 3. `docker kill` em gái tóc hồng yandere

Nghe tới từ `kill` chắc bạn cũng đoán được đây là một cách khá dã man con ngan rồi, ở đây nó sẽ có nhiệm vụ:

> “Dừng ngay lập tức! Không được dọn dẹp, không được phản ứng gì nữa.”

Quá trình trong container bị chết ngay tức thì, không có “grace period” luôn.

## 4. Thực hành

Kiểm tra container đang chạy:

```bash
docker ps
```

![image.png](https://images.viblo.asia/e8012d71-ac22-40d4-9829-ddc2b6e1e9e6.png)

Ta thấy ping [google.com](http://google.com/) đang hoạt động.

### a. Dừng bằng `docker stop`:

Thử gõ:

```bash
docker stop <container_id>
```

→ Bạn sẽ thấy chờ khoảng 10 giây, vì ping không phản hồi tín hiệu **SIGTERM**, nên Docker phải tự động gửi **SIGKILL** sau đó.

### b. Dừng bằng `docker kill`:

Gõ lệnh:

```bash
docker ps -a
docker start <container_id>
docker kill <container_id>
```

![image.png](https://images.viblo.asia/8bb18dae-95ba-46cb-bc86-84adede3b611.png)

→ Lần này container chết ngay lập tức, không có chờ, không có dọn dẹp gì cả.

Giờ mà bạn muốn xóa container đã dừng thì dùng lệnh `docker rm` hoặc `docker system prune` thoi.

# VIII. Cách thực thi các lệnh trong các container đang chạy

## 1. Vấn đề hiện tại

Từ đầu đến giờ chắc bạn cũng nhận ra, khi chúng ta tạo ra một container, bên trong container là một không gian riêng biệt, chúng ta không dùng cách thông thường để có thể tương tác trực tiếp với chương trình trong container thông qua terminal của máy mình được, tiêu biểu như khi bạn chạy `docker start` mà không có biến cờ `-a`, bạn muốn xem log của container thì bắt buộc phải dùng lệnh `docker logs` để nó xuất ra các log từ trong container ra terminal hiện tại của bạn.

Bạn có thể xem ví dụ qua hình:

![image.png](https://images.viblo.asia/9cd1dc7e-2cb9-4e5c-a727-a0aa1ea48357.png)

Rõ ràng rằng, giả sử bạn muốn tương tác được với container đang chạy bên trong thì phải có một cơ chế gì đó, hoặc một lệnh gì đó của **Docker client** để chúng ta có thể thông qua Docker mà vẫn có thể tương tác được với chương trình bên trong container.

Nếu bạn chưa tin, thì bạn có thể cùng mình làm một ví dụ sau:

Chúng ta sẽ cùng cài redis image và chạy nó. Nhắc lại một chút, Redis là một hệ thống lưu trữ dữ liệu trong bộ nhớ (in-memory data store), thường được dùng trong các ứng dụng web.

Redis cũng có 2 phần chính:

- Redis Server: Là một dịch vụ chạy ngầm, quản lý và lưu trữ dữ liệu trong bộ nhớ.
- Redis CLI: Là một công cụ dòng lệnh cho phép bạn gửi các lệnh như SET, GET đến Redis Server và nhận kết quả để thao tác với dữ liệu.

![image.png](https://images.viblo.asia/d3875a6a-65a7-4b36-8c1b-2559f3cf9dd4.png)

Okay giờ chúng ta sẽ cùng chạy redis với Docker qua lệnh sau ha:

```bash
docker run redis
```

Khi Redis chạy, có thể bạn sẽ thấy vài cảnh báo nhỏ, cứ bỏ qua, miễn là dòng cuối cùng hiện: **Ready to accept connections**

![image.png](https://images.viblo.asia/47c548e5-e740-427d-8d55-8104b411a80f.png)

Vậy container này chính là Redis server đang chạy bên trong Docker.

Bây giờ nếu bạn chưa tin thì bạn có thể gõ lệnh sau để tương tác thử với Redis server có được không, giờ thì chúng ta cùng gõ:

```bash
redis-cli
```

Đây là lệnh chúng ta báo với Redis rằng:

> "Này, tôi đang cần tương tác với Redis server, hãy mở Redis CLI cho tôi gõ nào!"

Nhưng đây là kết quả (terminal bên trái vẫn đang chạy Redis server, termnial bên phải yêu cầu mở Redis CLI):

![image.png](https://images.viblo.asia/dea32c6b-7249-4cd2-a853-bc5da7a25cee.png)

Nói cách khác, nó không kết nối được.

![image.png](https://images.viblo.asia/01e33b07-c402-495e-96ad-ca844f00a99c.png)

Thực chất Docker client cũng không nhận được lệnh ta vừa gõ nữa cơ, bởi vì nó đâu phải là lệnh đúng của Docker đâu, nên thành ra một lệnh đi vào hư không luôn.

## 2. Cách giải quyết

Nếu muốn dùng CLI, ta phải chạy CLI bên trong cùng container, nghĩa là phải chui được vào trong container và thực thi lệnh redis-cli từ bên trong đó.

Nói dễ hiểu thì ta cần chạy thêm một chương trình thứ hai (redis-cli) bên trong container mà Redis server đang chạy.

Để làm việc này, ta sẽ dùng một lệnh Docker mới đó là `docker exec`

### Giới thiệu lệnh `docker exec`

`exec` là viết tắt của “execute” (thực thi).

**Lệnh này được dùng để chạy thêm một lệnh khác bên trong container đang hoạt động.**

Cú pháp chung như sau:

```bash
docker exec -it <container_id> <command>
```

Trong đó `-it` là hai sự kết hợp của `i` và `t`:

- `-i` = “interactive” (Cho phép nhập dữ liệu từ bàn phím)
- `-t` = “terminal” (Tạo ra một giao diện terminal mô phỏng)

→ Kết hợp lại, `-it` giúp ta nhập lệnh và tương tác trực tiếp với container.

### Thực hành chạy redis-cli bên trong container

Giờ hãy thử làm thật. Chúng ta quay lại terminal, kiểm tra xem container Redis có còn chạy không bằng cách gõ:

```bash
docker ps
```

![image.png](https://images.viblo.asia/e002f2bc-7f0f-4fa0-bfad-4dd67a4c3057.png)

Rồi vẫn đang chạy ngon. Rồi giờ chúng ta sẽ thực thi lệnh để đi vào trong container, chạy lệnh để mở Redis client, chúng ta sẽ gõ lệnh như sau:

```bash
docker exec -it <container_id> redis-cli
```

![image.png](https://images.viblo.asia/064593e8-7492-44da-bc50-c54785fc3be0.png)

Quào giờ chúng ta đã không còn bị lỗi như vừa nãy nữa, bây giờ chúng ta đã mở được một Redis client để có thể gõ được các lệnh và tương tác với Redis server rồi.

Giờ bạn có thể nhập các lệnh như:

```bash
SET myvalue 5
GET myvalue
```

Và Redis trả về giá trị 5.

Vậy là nhờ lệnh `docker exec`, ta đã chạy thêm một chương trình thứ hai bên trong container (ngoài Redis server đang chạy).

Tùy chọn `-it` cho phép ta gõ lệnh từ bàn phím và gửi trực tiếp vào container đó.

### Thử bỏ `-it` xem sao

Giờ thử một thí nghiệm nhỏ.

Chúng ta hãy nhấn `Ctrl + C` để thoát khỏi CLI.

Rồi chạy lại lệnh `docker exec`, nhưng bỏ phần `-it` đi:

![image.png](https://images.viblo.asia/a7224332-3a4a-41ce-a737-b2c0c821fb37.png)

Kết quả lệnh chạy xong ngay lập tức và trả về terminal ban đầu, không có gì hiển thị.

**Tại sao?**

Bởi vì khi Redis CLI được khởi động, nó nhận ra rằng không có luồng nhập từ bàn phím do đó nó không thể nhận được input nào cả, nên chương trình tự đóng lại và trả quyền điều khiển về terminal. (Bạn nên hiểu ở đây là Redis CLI vẫn chạy được, nhưng một chương trình terminal mà không có terminal hoặc cái gì để tương tác bỏ input vào cho nó nên nó tự tắt và trả về terminal của máy bạn, mình sẽ giải thích kỹ cơ chế này vào phần dưới đây.)

## 3. Mục đích của biến cờ `-it`

Trong phần này, mình sẽ giải thích rõ hơn cờ `-it` thực sự làm gì và tại sao nó lại cần thiết.

Đầu tiên, ta cần hiểu cách tiến trình hoạt động trong môi trường Linux.

### a. Tiến trình trong Linux và 3 kênh giao tiếp chuẩn

Để mình nhắc lại một chút:

> Mỗi container Docker bạn chạy thực chất đều hoạt động bên trong một máy ảo Linux.

Nghĩa là dù bạn đang dùng Windows hay Mac, thì bên trong container thì tất cả vẫn là Linux.

Trong môi trường Linux, mỗi tiến trình đều có gắn 3 kênh giao tiếp chuẩn (standard streams):

- Standard In (stdin) – kênh nhập dữ liệu vào tiến trình → Mọi thứ bạn gõ trong terminal được gửi vào kênh này.
- Standard Out (stdout) – kênh xuất dữ liệu từ tiến trình → Dữ liệu được tiến trình in ra và hiển thị trên màn hình.
- Standard Error (stderr) – cũng là kênh xuất dữ liệu, nhưng dành riêng cho thông báo lỗi hoặc thông tin lỗi → Thường cũng được hiển thị trong terminal của bạn.

Bạn có thể xem hình minh họa dưới đây:

![image.png](https://images.viblo.asia/2123d46d-9767-4a0e-800d-4797755239a8.png)

_Nguồn: [https://krishwebdev.hashnode.dev/learn-standard-linux-streams-and-file-manipulation](https://krishwebdev.hashnode.dev/learn-standard-linux-streams-and-file-manipulation)_

**Tóm lại:**

- **Bạn gõ lệnh → đi qua stdin.**
- **Kết quả → in ra qua stdout hoặc stderr.**

### b. Liên hệ với `docker exec -it`

Giờ chúng ta quay lại với lệnh:

```bash
docker exec -it <container_id> redis-cli
```

Thực ra `-it` không phải là một cờ duy nhất, như mình nói lúc nãy, mà là gộp từ hai cờ riêng biệt:

```none
-i

và

-t
```

Kết hợp lại thành `-it` chỉ để gọn hơn, nhưng về bản chất là như sau:

- `-i` (interactive) → kết nối terminal của bạn với **stdin** của tiến trình mới. 👉 Giúp những gì bạn gõ được gửi vào redis-cli.
- `-t` (tty) → tạo ra một terminal giả (pseudo-terminal) để hiển thị gọn gàng, có định dạng, có con trỏ, hỗ trợ autocomplete, v.v.

**Tóm lại:**

- **`-i` giúp nhập dữ liệu vào container.**
- **`-t` giúp hiển thị đẹp và tương tác dễ dàng.**

### c. Thử nghiệm bỏ biến cờ `-t`

Giờ ta thử làm lại (không có `-t`):

```bash
docker exec -i <container_id> redis-cli
```

![image.png](https://images.viblo.asia/a78d91ea-48c2-441a-8cb4-0530c74c79b3.png)

Kết quả:

- Bạn vẫn có thể nhập lệnh (con trỏ vẫn nhấp nháy chờ input).
- Nhưng không còn định dạng đẹp như trước, không có dòng lệnh màu, không có autocomplete.

Giờ bạn hãy thử nhập:

```bash
SET myvalue 5
GET myvalue
```

![image.png](https://images.viblo.asia/5391b892-1513-41a4-bacb-f31902e757d5.png)

Thì nó vẫn hoạt động bình thường, chỉ là kết quả hiển thị khá là xấu, không đẹp như khi có `-t`.

Vậy ý nghĩa thật sự của -it là cho phép bạn vừa nhập, vừa nhìn thấy kết quả một cách rõ ràng, giống như đang thao tác trực tiếp trong terminal thật.

Nếu thiếu một trong hai, trải nghiệm sẽ bị thiếu, hoặc không gõ được, hoặc hiển thị xấu.

## 4. Truy cập trực tiếp vào terminal bên trong container đang chạy

Trong phần này, ta sẽ nói về một cách dùng cuối cùng của lệnh `docker exec`, và đây cũng là cách phổ biến nhất mà bạn sẽ thường xuyên dùng trong các dự án thực tế.

### a. Mục tiêu của phần này

Một việc rất thường gặp khi làm việc với Docker là:

> Bạn muốn truy cập trực tiếp vào terminal (shell) bên trong container đang chạy.

Nói cách khác, thay vì phải chạy:

```bash
docker exec ...
docker exec ...
docker exec ...
```

Liên tục mỗi khi muốn gõ một lệnh, ta chỉ cần mở hẳn một terminal bên trong container, rồi có thể gõ bao nhiêu lệnh tùy thích, giống như đang làm việc trên máy thật.

Để làm được điều đó, chúng ta sẽ xem phần sau đây.

### b. Thực hành mở shell bên trong container

Đây là các bước thực hiện:

1. Mở lại terminal, kiểm tra container Redis đang chạy:

```bash
docker ps
```

2. Mở một cửa sổ terminal thứ hai.
3. Lấy container ID từ kết quả `docker ps`.
4. Chạy lệnh:

```bash
docker exec -it <container_id> sh
```

Lệnh này có nghĩa là:

- `docker exec`: Chạy một lệnh bên trong container.
- `-it`: Cho phép tương tác.
- `sh`: mở một trình dòng lệnh bên trong container.

![image.png](https://images.viblo.asia/a58f6c2e-39f9-4e3f-8939-a7b8735f6ea4.png)

Khi chạy xong, bạn sẽ thấy dấu nhắc lệnh kiểu như:

```bash
#
```

Điều đó nghĩa là bạn đang ở bên trong container.

![](https://media.tenor.com/n12Ei4-oFckAAAAM/%D0%BF%D0%BE%D0%B7%D0%B4%D1%80%D0%B0%D0%B2%D0%BB%D1%8F%D1%8E.gif)

### c. Gõ lệnh như trong Linux thật

Bây giờ, bạn có thể gõ các lệnh quen thuộc như:

```bash
cd ~
ls
```

![image.png](https://images.viblo.asia/9d99c75a-4e64-4624-8b24-6f5e220da24e.png)

Tạm thời chưa có file nào bên trong.

Thử quay lại thư mục gốc:

```bash
cd /
ls
```

![image.png](https://images.viblo.asia/acdf5619-8c3f-4daa-9ebc-f08e7f32f944.png)

→ Bạn sẽ thấy toàn bộ cấu trúc thư mục gốc của container.

Thử vài lệnh khác:

```bash
echo "hi there"
export B=5
echo $B
```

Tất cả đều hoạt động như trong môi trường Linux thật.

Điều này cho phép ta debug trực tiếp bên trong container rất hiệu quả.

Thậm chí bạn có thể chạy luôn Redis CLI từ đây:

```bash
redis-cli
```

→ Giao diện redis-cli được mở ra ngay trong container.

Khi muốn thoát, bạn có thể nhấn:

- Ctrl + C (nếu đang trong chương trình CLI), hoặc
- Ctrl + D (lối tắt phổ biến để thoát khỏi shell).

### d. Giải thích `sh` là gì?

Vậy chương trình sh là gì?

- **sh** là một chương trình shell (viết tắt của Shell).
- Nó là bộ xử lý lệnh (command processor) của chương trình giúp ta nhập lệnh và thực thi chúng bên trong container.

Trên máy của bạn cũng đang có một shell tương tự:

- macOS → thường dùng bash hoặc zsh
- Windows → có thể là Git Bash, PowerShell
- Linux → bash, sh, hoặc zsh

Tất cả những chương trình đó đều cho phép ta nhập lệnh vào terminal và chạy chúng.

Tương tự vậy, khi ta chạy:

```bash
docker exec -it <container_id> sh
```

Ta đang chạy chương trình sh bên trong container, để có thể nhập và chạy lệnh bên trong môi trường đó.

Hầu hết các container đều cài sẵn sh.

Một số image đầy đủ hơn (ví dụ Ubuntu, Node.js, v.v.) còn có thêm bash.

→ Vì vậy, đôi khi bạn có thể dùng:

```bash
docker exec -it <container_id> bash
```

Nếu không có bash, hãy dùng sh, lệnh này gần như luôn luôn có sẵn.

## 5. Cách khác để mở shell trong container

Trong phần trước, ta đã nói về cách mở một shell bên trong container đang chạy bằng lệnh:

```bash
docker exec -it <container_id> sh
```

Ta dùng `docker exec` để chạy thêm một lệnh mới bên trong container, mà không cần khởi động lại nó.

Nhưng thực ra, ta cũng có thể dùng luôn lệnh `docker run` ban đầu, kết hợp với cờ `-it`, để khởi động container mới và mở ngay shell bên trong khi nó vừa chạy.

Tất nhiên, nếu bạn khởi động container và chạy shell ngay từ đầu, thì container đó sẽ không chạy chương trình mặc định nào khác (ví dụ: web server).

Tuy nhiên, đôi khi việc chạy một container trống chỉ để mở shell và thử nghiệm lại rất tiện lợi.

Giờ hãy thử nhé, ta sẽ quay lại terminal và gõ:

```bash
docker run -it busybox sh
```

Khi chạy lệnh này, bạn sẽ thấy một dấu nhắc lệnh (command prompt).

Giờ bạn có thể gõ:

- `ls`để liệt kê thư mục.
- ping [google.com](http://google.com/) rồi nhấn Ctrl + C để dừng.
- hoặc echo "Hello" để in ra dòng chữ.

Đây là một cách rất hay để dò thử container xem nó có gì bên trong, thử vài lệnh, v.v.

**Nhược điểm là:**

Nếu bạn chạy container bằng `docker run -it sh`, container chỉ chạy shell thôi, không có chương trình chính nào khác.

Thực tế, thông thường ta sẽ muốn container chạy ứng dụng chính (ví dụ web server), rồi sau đó mới dùng `docker exec -it` để mở shell kiểm tra khi cần.

Mình chỉ muốn bạn biết thêm cách khác này để linh hoạt khi làm việc với Docker.

# IX. Container không tự động chia sẻ chung hệ thống tệp

Trước đó ta đã từng nói nhờ vào tính năng namespace, ta có thể tưởng tượng như ổ đĩa của máy được chia thành nhiều phần nhỏ với từng container.

Mỗi ứng dụng (ví dụ Chrome hoặc Node) chỉ thấy phần riêng của nó thôi. Tương tự như vậy, hai container không tự động chia sẻ chung hệ thống tệp. Chúng hoàn toàn tách biệt.

Giờ ta sẽ làm một ví dụ nhanh trong terminal để thấy rõ điều này.

Đầu tiên, ta mở terminal và chạy:

```bash
docker run -it busybox sh
```

Giờ ta ở trong container đầu tiên, gõ ls để xem các thư mục mặc định, chỉ có mấy thư mục hệ thống thôi.

Tiếp theo, ta mở một terminal khác, và chạy lệnh y hệt:

```bash
docker run -it busybox sh
```

=> Vậy là ta có hai container đang chạy song song, đều dùng image busybox.

Để chắc chắn, ta có thể mở thêm một terminal thứ ba, gõ:

```bash
docker ps
```

Sẽ thấy có hai container đang chạy, mỗi cái đều là shell.

Giờ ta quay lại container thứ nhất, và tạo một file mới:

```bash
touch hi_there
```

Kiểm tra lại bằng ls → thấy file hi_there đã xuất hiện.

Bây giờ sang container thứ hai, ta cũng gõ ls.

Và bạn sẽ thấy 👉 Không có file hi_there ở đây.

**Vì sao? Vì hai container này có hai hệ thống tệp hoàn toàn riêng biệt.**

Không có chia sẻ dữ liệu nào giữa chúng, trừ khi ta tự thiết lập việc đó (ví dụ dùng volume hoặc network).

**Tóm lại:**

> Các container là môi trường tách biệt, cô lập hoàn toàn với nhau. Nếu bạn không tự cấu hình để chia sẻ dữ liệu, thì container A và container B không thấy được file của nhau.

Vậy là chúng ta đã xong phần cơ bản về Docker.

# X. Tổng kết phần 2

Và đó là toàn bộ phần Docker cơ bản mà mình đã truyền tải đến cho mọi người.

Bài viết đã rất là dài rồi nên mình sẽ cắt phần 2 ở đây, sang phần 3 chúng ta sẽ tiếp tục với các kiến thức tiếp theo của Docker nhé!

Mình xin nhắc lại là bài viết của mình không thể thay thế được những kiến thức khóa học từ anh Sờ Te Phờn, các bạn hãy đăng ký và trải nghiệm thử.

Nếu bạn thấy hay thì cho mình xin 1 upvote và để lại comment những chỗ mình đã sai, hoặc muốn thảo luận thêm nhé.