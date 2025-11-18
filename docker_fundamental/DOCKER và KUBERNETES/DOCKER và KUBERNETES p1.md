# Tuyệt thế chân kinh ở đây này!!!

Hôm nay mình sẽ mang tới cho anh em một chủ đề không quá mới, nhưng vẫn giữ được sức thu hút rất lớn đối với các bạn mới tiếp cận các kiến thức lập trình hiện đại giống mình.

Nó giống như bí kỹ chân kinh mà Dev nào cũng đã từng nghe danh qua, với uy lực cao cường, một khi đã thông thạo thì việc thống nhất giang hồ Dev đã không còn xa.

VÀ ĐÓ CHÍNH LÀ **DOCKER và KUBERNETES - Tuyệt thế chân kinh ở đây này!!!**, bài viết thứ ba nằm trong chuỗi series "Bạn mù tịt về Microservices, tôi cũng thế"

Mong anh em đọc và cho mình 1 upvote và comment nếu thấy cấn cấn để mình cải thiện bài viết, giờ thì nẹt gooo!!!!!!!!!

---

# I. Vấn đề đối với dự án trước

Nếu anh em đã quên dự án ở bài trước thì mình sẽ để link bài viết ở đây để anh em đọc lại nha [Đọc ở đây](https://viblo.asia/p/thuc-hanh-tao-ra-mang-xa-hoi-facebook-voi-microservices-cung-hang-triu-user-oKLnqBw1JQO)!!!

Web facebook của chúng ta hiện giờ gần như đã hoàn chỉnh rồi. Vì vậy, giờ là lúc chúng ta bắt đầu nghĩ xem làm sao đưa nó lên môi trường internet để nó có thể truy cập được bởi mọi người. Trước tiên, hãy nói nhanh về cách ứng dụng này đang hoạt động trên máy local của chúng ta.

Bạn có thể hình dung như sau. Trên máy của chúng, chúng ta có 4 service: **Post**, **Comment**, **Query** và **Event bus**. Tất cả chúng đang chạy ở các **port riêng biệt**.

![image.png](https://images.viblo.asia/dd5999a9-993a-4ebb-ba2c-bdeb282fe6c9.png)

Và mỗi service có thể gửi request trực tiếp đến các service khác, chúng giao tiếp với nhau qua `localhost` rất đơn giản và trực tiếp. Mọi thứ trên máy local rất trơn tru, các service giao tiếp trực tiếp với nhau mà không gặp rào cản nào.

> **💡 Câu hỏi đặt ra:**
> Giờ nếu chúng ta muốn giữ nguyên cấu trúc này, và chỉ thay đổi ít nhất có thể và để đưa nó lên internet, thì làm sao làm được?

Một cách rất đơn giản là chúng ta có thể thuê một **Virtual machine** (máy ảo) trên DigitalOcean, AWS, Azure hay bất kỳ nhà cung cấp nào. Khi có VM rồi, ta chỉ cần chuyển toàn bộ source code của các service sang đó. “Chuyển” ở đây chỉ đơn giản là copy code và chạy các service y xì chóc như bây giờ.

Các service vẫn có thể giao tiếp với nhau qua `localhost`, ví dụ như truy cập port `8080`, `8081`, `8082`... Và đúng, cách này hoàn toàn có thể chạy được.

**⚠️ Nhưng vấn đề bắt đầu khi ta muốn mở rộng**

Giờ giả sử, service **Comment** của ta bắt đầu bị quá tải. Do có quá nhiều người dùng cùng lúc tạo comment, và một lúc nào đó chúng ta cần chạy nhiều **instance** (bản sao) của service này để chịu tải tốt hơn.

![image.png](https://images.viblo.asia/c0d76d3c-94ee-4405-ad97-803bbb50cdd9.png)

Trên cùng máy chủ đó, ta tạo thêm 2 bản copy của **Comment service**. Giờ ta có 3 bản chạy song song, 1 bản gốc, và 2 bản mới. Khi có request tạo comment, ta có thể **load balance** (cân bằng tải), tức là chia ngẫu nhiên request đến 1 trong 3 service đó. Từ đây chúng ta sẽ phát sinh những vấn đề sau:

## Vấn đề 1: Port và Event Bus

Mỗi bản copy mới này sẽ cần chạy ở một port khác nhau, ví dụ **8083** và **8084**.

Nhưng mà nhớ nhé, **Event Bus** cần biết chính xác IP và port của mọi service để gửi event. Nên khi ta thêm 2 bản mới, ta phải vào sửa code của **Event Bus**, thêm 2 dòng nữa để nó gửi đến port **8083** và **8084**.

Điều này nghĩa là số lượng service đang chạy bị ràng buộc chặt vào code. Nếu ta muốn tăng hoặc giảm số **Comment service**, ta lại phải sửa code và re-deploy **Event Bus**. Rõ ràng, cách này không ổn chút nào.

![image.png](https://images.viblo.asia/d8a3625a-ba6c-4067-9539-e4fdcbd8df9b.png)

## Vấn đề 2: Nhiều máy chủ hơn

Giả sử bây giờ, một máy chủ không chịu nổi tải, ta quyết định thuê thêm một **VM** thứ hai và chạy 2 bản copy kia ở đó.

**Event Bus** giờ không chỉ cần biết port, mà còn phải biết địa chỉ IP của máy thứ hai. Nghĩa là ta lại phải sửa code của **Event Bus** để chỉ định đúng IP, ví dụ thay `localhost` bằng `192.168.0.2` chẳng hạn.

![image.png](https://images.viblo.asia/51744553-5312-4d6b-9eb1-36f489b6411a.png)

Và chắc bạn cũng đoán được rồi, việc này thực sự rất phiền phức và khó bảo trì khi mà cứ phải tìm rồi thay liên tục thủ công như vậy.

## Vấn đề 3: Linh hoạt theo thời gian

Giả sử website đông người dùng vào buổi sáng 10h, và gần như không ai dùng vào 1h sáng.

Ta có thể muốn mở thêm server phụ lúc 10h sáng, rồi tắt bớt server lúc 1h sáng để tiết kiệm chi phí.

Nhưng nếu làm vậy, **Event Bus** lại phải cần biết được server nào đang bật, server nào tắt và không gửi event đến máy đã tắt.

Để xử lý điều đó, ta lại phải thêm code kiểu như:

> “Nếu bây giờ không phải 1h sáng thì gửi event đến các service phụ.”

Tất nhiên, đó là cách làm cực kỳ tệ. Không ai muốn viết code như vậy trong thực tế. Vì coder là những Lazy guy không muốn OT mà =)))))

## Kết luận cho phần này

> Rõ ràng, hướng này chúng ta đang đi sai đường. Ta đang cố gắng biến **Event Bus** thành **một bộ não điều khiển** biết mọi thứ. Nó phải biết service nào đang chạy, port nào, IP nào, giờ nào được phép gửi, giờ nào không...
>
> Và đó là cơn ác mộng bảo trì. Cực kỳ phức tạp, và gần như không thể quản lý khi hệ thống lớn lên.

### **🧭 Vậy ta cần gì?**

Chúng ta cần một công cụ:

- Biết được service nào đang chạy.
- Có thể tự tạo thêm bản copy (**scale out**) khi cần.
- Có thể tự phát hiện service nào ngủm.
- Và tự định tuyến (**routing**) kết nối giữa các service.

### **Và đó chính là thứ sẽ dẫn chúng ta đến hai công nghệ mã nguồn mở cực kỳ quan trọng:**

### **Docker và Kubernetes.**

---

# II. Tại sao lại là Docker?

Trong phần trên, chúng ta đã nói về một số vấn đề mà ta có thể sẽ gặp phải khi triển khai ứng dụng của mình. Vì vậy, trong phần này, chúng ta sẽ nói về **Docker**, một công cụ cực kỳ hữu ích.

**Docker** sẽ giúp chúng ta đơn giản hóa rất nhiều thứ, đặc biệt là khi ta bắt đầu đưa ứng dụng của mình lên môi trường **production**.

## 1. Docker là gì?

**Docker** là công cụ giúp ta tạo ra những thứ gọi là **Container**.

> Một **container** giống như một môi trường máy tính độc lập, nó chứa tất cả mọi thứ cần thiết để chạy một chương trình duy nhất.

Chúng ta sẽ tạo một **container** riêng biệt cho từng service trong hệ thống. Ví dụ:

- Một container để chạy **event-bus**.
- Một container cho **post**.
- Một container cho **comment**.
- Một container cho **query**.

![image.png](https://images.viblo.asia/afac159e-64d7-459d-8f61-df1a34c02948.png)

Và cứ thế, nếu ta cần nhiều bản chạy song song của một service nào đó, ví dụ **comment service**, thì chỉ cần tạo thêm một **container Docker** nữa cho service đó. Nói cách khác, mỗi **container** tương ứng 1 **instance** của chương trình.

## 2. Tại sao lại cần Docker?

Giờ hãy nói về lý do tại sao chúng ta dùng **Docker**, và nó giải quyết vấn đề gì.

Hiện tại, để chạy ứng dụng của chúng ta, ta đang phải cài rất nhiều thứ về môi trường mà nó chạy trên đó.

Ví dụ nhé ta khởi chạy **Query service** bằng cách gõ lệnh:

```bash
npm run start
```

Nhưng để chạy được lệnh đó, máy tính phải có sẵn **NPM**. Mà để có **NPM**, máy đó cũng phải cài sẵn **Node.js**.

Vậy tức là chỉ để chạy chương trình, ta đã phụ thuộc vào việc môi trường có sẵn **Node** và **NPM**. Đó là công việc hiển nhiên và đã trở thành thói quen mà chúng ta đã dễ dàng bỏ qua điều đó.

Ngoài ra còn một vấn đề khác, việc khởi chạy ứng dụng cũng phải tuân thủ đúng quy trình. Ta phải cần biết chính xác lệnh nào để chạy, ví dụ như:

```bash
npm run start
```

Hoặc

```bash
go run main.go
```

Nói cách khác, cách chạy ứng dụng không hề rõ ràng, với ngôn ngữ này hoặc framework này thì cần chạy lệnh này, qua service khác thì phải chạy lệnh khác, không hề có một cách thống nhất chung để làm hết được mấy chuyện này.

### Docker giải quyết cả hai vấn đề đó

Mục tiêu của **Docker** là xóa bỏ toàn bộ các giả định môi trường đó. Bằng cách tạo ra **Container** (Bạn chưa cần hiểu nó ngay bây giờ, mình sẽ giải thích kỹ nó ở phần sau), ta có thể gói trọn toàn bộ các **dependency** mà chương trình cần bao gồm cả **Node.js** và **NPM**, **Go** hay **Python**.

Trong **container** đó, ta còn có thể chỉ định sẵn lệnh khởi chạy cho chương trình. Vì vậy, khi người khác muốn chạy ứng dụng của bạn, họ không cần cài **Node**, không cần cài **NPM**, không cần cài **Python**, không cần cài **Go**, chỉ cần chạy **container Docker** là đủ. **Container** đã tự mang theo môi trường riêng của nó.

### Docker có thể chạy mọi thứ

Điều hay ở **Docker** là nó không chỉ giới hạn cho **Node.js**. Bất kỳ chương trình nào bạn nghĩ đến **Python**, **Java**, **Go**, **C++**,... đều có thể chạy được trong **Docker**. Nói cách khác, **Docker** khiến việc chạy bất kỳ chương trình nào trở nên cực kỳ dễ dàng.

Và bạn sẽ thấy khi ta kết hợp **Docker** với **Kubernetes**, mọi thứ sẽ trở nên mạnh mẽ và tự động hơn rất nhiều. Trong phần tiếp theo, chúng ta sẽ bắt đầu tìm hiểu **Kubernetes** và xem **Docker + Kubernetes** cùng nhau giải quyết việc triển khai, mở rộng và quản lý service như thế nào.

---

# III. Tại sao lại là Kubernetes?

Bây giờ, chúng ta sẽ bắt đầu nói về **Kubernetes**. Khi mới học về **Kubernetes**, bạn sẽ chưa thấy ngay vì sao **Docker** và **Kubernetes** lại hợp nhau đến thế. Đừng lo, mọi thứ sẽ trở nên rõ ràng hơn khi ta đi sâu hơn một chút.

Còn bây giờ, hãy tập trung hiểu cơ bản **Kubernetes** là gì, và xem nó giải quyết vấn đề **scaling** như thế nào.

## 1. Kubernetes là gì?

> **Kubernetes** là một công cụ dùng để chạy nhiều **container** khác nhau cùng lúc.

Khi sử dụng **Kubernetes**, ta sẽ viết các **file cấu hình** để mô tả cho nó biết:

- Chúng ta muốn chạy những **container** nào?
- Chạy bao nhiêu **instance** của mỗi **container**?
- Và chúng kết nối với nhau ra sao?

Sau đó, **Kubernetes** sẽ tạo các **container** đó và chạy chương trình cho ta, đồng thời xử lý luôn phần **kết nối mạng** giữa các **container**, đảm bảo các service liên lạc được với nhau dễ dàng.

Chúng ta có thể hiểu đơn giản thế này:

> **Kubernetes** giúp bạn chạy nhiều chương trình khác nhau, và giúp chúng giao tiếp với nhau trơn tru, tự động.

## 2. Cấu trúc cơ bản của Kubernetes: Cluster – Master – Nodes

Khi làm việc với **Kubernetes**, ta sẽ tạo ra một **cluster**.

Một **cluster** bao gồm nhiều **VM** (máy ảo), có thể chỉ là 1 máy, hoặc hàng trăm, hàng nghìn máy.

![image.png](https://images.viblo.asia/fb8fc592-6b5d-4491-bb25-ea66d1d9b261.png)

Mỗi máy ảo này được gọi là một **Node**. Tất cả các **Node** đều được điều khiển bởi một thành phần trung tâm gọi là **Master**.

**Master** là một chương trình có nhiệm vụ:

- Quản lý toàn bộ **cluster**.
- Giám sát các **Node**.
- Theo dõi trạng thái của các **container** đang chạy.
- Và đảm bảo mọi thứ hoạt động đúng như ta cấu hình.

## 3. Cách Kubernetes thực hiện

Giả sử ta viết một file cấu hình như sau:

> Hãy chạy 2 bản sao (**replicas**) của **Post service**, và đảm bảo có thể truy cập dễ dàng vào chúng.

Khi ta gửi file này cho **Master**, **Kubernetes** sẽ:

- Đọc file cấu hình.
- Thực hiện các bước được mô tả trong đó.
- Tạo ra 2 **container Post**.
- Gán chúng (gần như ngẫu nhiên) vào các **Node** trong **cluster** để chạy. (Nói “ngẫu nhiên” cho dễ hiểu, nhưng thực ra có một số thuật toán phân bổ thông minh để tối ưu tài nguyên.)

![image.png](https://images.viblo.asia/e4a4d8ad-5f15-4af9-a5e5-0aa0655be416.png)

## 4. Vấn đề giao tiếp giữa các service

Giờ giả sử ta có:

- **Post Service** chạy trên **Node 1** và **Node 2**,
- **Event Bus** chạy trên **Node 3**.

Nếu không có **Kubernetes**, thì **Event Bus** sẽ phải tự biết chính xác địa chỉ của từng **Post Service** trên từng **Node**.

Ví dụ:

- Node 1: `10.0.0.5:8080`
- Node 2: `10.0.0.5:8081`

Điều này rất rắc rối và khó quản lý, đặc biệt khi số lượng service ngày càng nhiều hoặc bị thay đổi IP liên tục.

## 5. Giải pháp Kubernetes Service (Communication Channel)

**Kubernetes** cung cấp cho ta một lớp giao tiếp chung (**common communication channel**).

Thay vì **Event Bus** phải gửi trực tiếp đến từng **container**, nó chỉ cần gửi request đến kênh chung này. **Kubernetes** sẽ tự động định tuyến (**route**) request đó đến đúng **container** thích hợp đang chạy trong **cluster**.

Ví dụ khi **Event Bus** gửi request đến địa chỉ của **Post Service**, **Kubernetes** sẽ đảm bảo:

- Cả hai bản sao của **Post Service** đều nhận được.
- Hoặc request được phân phối hợp lý giữa chúng.

![image.png](https://images.viblo.asia/698dc4a7-490c-4cc0-adbf-aec932535703.png)

## 6.Lợi ích lớn nhất khi dùng Kubernetes

1.  **Giao tiếp dễ dàng** giữa các service, không cần cấu hình phức tạp.
2.  **Dễ dàng scale**: chỉ cần chỉnh trong file cấu hình, **Kubernetes** tự động tạo thêm hoặc giảm bớt bản sao service.
3.  **Tự động hóa** hoàn toàn việc khởi tạo, debug, và cân bằng tải (**load balancing**).

---

# IV. Hướng dẫn Docker dành cho thiếu nhi

Mỗi khi bạn thấy ai đó nhắc đến **Docker** trong một bài viết, blog, hay diễn đàn, thì thực ra họ đang nói đến một hệ sinh thái gồm rất nhiều dự án, công cụ và phần mềm khác nhau.

Vì vậy, khi ai đó nói “Tôi dùng Docker trong dự án của mình”, thì họ có thể đang nói đến:

- **Docker Client** (trình dòng lệnh của Docker),
- hoặc **Docker Server**,
- hoặc **Docker Hub**,
- hoặc **Docker Compose**.

Tất cả những thứ đó là các công cụ và phần mềm nhỏ hợp lại để tạo thành một platform, một hệ sinh thái xoay quanh việc tạo ra và chạy cái gọi là **Container**.

Và lúc này bạn có thể sẽ hỏi:

> “**Container** là gì vậy?”

Đó là một câu hỏi hay, và cũng chính là câu hỏi mà chúng ta sẽ tìm hiểu dần xuyên suốt bài viết này.

Trước hết, chúng ta sẽ nói về **Tại sao lại dùng Docker** bằng cách xem qua một ví dụ nhỏ.

Mình sẽ cho bạn xem một sơ đồ quy trình đây có lẽ là một việc mà bạn đã từng làm, ít nhất một lần trong đời:

**👉 Cài đặt phần mềm trên máy tính cá nhân.**

Và mình khá chắc là ít nhất một lần, bạn đã gặp phải tình huống như thế này:

- Bạn tải về một bộ cài đặt (**installer**)
- Bạn chạy nó.
- Rồi đột nhiên, bị lỗi trong quá trình cài đặt.

Khi đó, bạn làm gì? Có lẽ bạn sẽ lên Google để tìm cách khắc phục, thử vài cách, không được, sau đó dùng chatGPT cả buổi trời, cuối cùng cũng sửa được lỗi. Nhưng rồi khi chạy lại bộ cài, lại xuất hiện một lỗi khác. Và bạn lại phải tiếp tục quá trình troubleshoot mệt mỏi này.

![image.png](https://images.viblo.asia/8beb97b1-d7b7-4374-862a-bf1bbd3d7980.png)

**👉 Và đây chính là vấn đề mà Docker muốn giải quyết.**

**Docker** được tạo ra để giúp việc cài đặt và chạy phần mềm trên bất kỳ máy tính nào trở nên thật dễ dàng và đơn giản. Không chỉ trên máy cá nhân của bạn, mà còn trên máy chủ web server hay nền tảng điện toán đám mây (**Cloud computing**).

Giờ mình sẽ trình diễn nhanh cho bạn thấy **Docker** giúp đơn giản hóa quá trình đó thế nào. Chúng ta sẽ thử cài đặt một phần mềm tên là **Redis**. **Redis** là một bộ lưu trữ dữ liệu trong bộ nhớ, chắc hẳn bạn đã từng nghe qua rất nhiều về nó, chúng ta sẽ truy cập trang trính thức của họ: [https://redis.io/docs/latest/operate/oss_and_stack/install/archive/install-redis/install-redis-on-windows/](https://redis.io/docs/latest/operate/oss_and_stack/install/archive/install-redis/install-redis-on-windows/) (Yeah mình đang dùng Window nên mình sẽ tìm đến bản Win)

![screencapture-redis-io-docs-latest-operate-oss-and-stack-install-archive-install-redis-install-redis-on-windows-2025-10-21-17_38_15.png](https://images.viblo.asia/47b00e6e-c35e-4031-9f56-3266d86fbca7.png)

Yah, vừa mới vào là một mớ thứ mà mình còn chưa kịp hiểu, WTH tôi chỉ muốn cài **Redis** thôi mà, có cần lằng nhằng đến như vậy không?

Vì thế, bây giờ mình sẽ cho bạn thấy cách chạy **Redis** chỉ bằng **Docker**, và bạn sẽ thấy sự khác biệt rõ ràng. Mình quay lại terminal và chỉ cần gõ một dòng lệnh duy nhất:

```bash
docker run -it redis
```

Nhấn Enter...

![Picture1.png](https://images.viblo.asia/0bf5f908-7ed7-428c-9a70-ce0f33a3d115.png)

Và chỉ sau vài giây ngắn ngủi, **Redis** đã chạy thành công trên máy của mình. Vậy đó, chỉ một lệnh duy nhất!

Và đó chính là **Docker** trong một câu ngắn gọn:

> **Docker** giúp việc chạy phần mềm trở nên dễ dàng cực kỳ.

Tóm lại, để trả lời trực tiếp câu hỏi “Tại sao dùng Docker?”:

> Bởi vì **Docker** giúp cài đặt và chạy phần mềm dễ dàng hơn nhiều, không cần lo lắng về việc cài **dependencies** hay cấu hình phức tạp.

Giờ quay lại lệnh vừa nãy, khi mình chạy lệnh đó, **Docker CLI** đã kết nối đến **Docker Hub**, và tải về một tập tin duy nhất gọi là **Image**. Vậy **Docker Image** là gì?

## 1. Docker Image là gì?

> **Image** là một tệp duy nhất chứa toàn bộ các thư viện, **dependencies** và cấu hình cần thiết để chạy một chương trình cụ thể.

Ví dụ, **image** chúng ta vừa tải chính là để chạy **Redis**. **Image** này được lưu lại trên ổ cứng của bạn. Và sau này, chúng ta có thể dùng **image** đó để tạo ra một thứ gọi là **Container**.

![image.png](https://images.viblo.asia/27d0ccdb-05ea-4260-a701-cdd70ced2b2f.png)

## 2. Container là gì?

> **Container** là một **instance** của **image**, chúng ta có thể hiểu đơn giản nó giống như một chương trình đang chạy vậy.

Chúng ta sẽ tìm hiểu chi tiết hơn về cách **container** hoạt động, nhưng bây giờ bạn chỉ cần hiểu rằng:

> **Container** là một chương trình có **không gian tài nguyên riêng biệt**.

Điều đó có nghĩa là:

- Nó có **bộ nhớ riêng**.
- **Network riêng**.
- Và **vùng lưu trữ riêng** tách biệt với hệ thống chính.

## 3. Docker Client và Docker Daemon là gì?

Bên trong **Docker** có hai công cụ cực kỳ quan trọng, mà chúng ta sẽ sử dụng xuyên suốt bài viết này.

Công cụ đầu tiên là **Docker Client**, hay còn gọi là **Docker CLI** đây là chương trình mà bạn và mình sẽ tương tác trực tiếp trong terminal.

Chúng ta sẽ gõ các lệnh trong terminal, gửi chúng đến **Docker Client**. Nhiệm vụ của **Client** là nhận lệnh và quyết định cách xử lý.

Tuy nhiên, **Docker Client** bản thân nó không trực tiếp làm gì với **container** hay **image** cả. Thực chất, nó chỉ là một công cụ trung gian, một cửa ngõ giúp ta giao tiếp với một phần mềm khác đi kèm trong bộ cài **Docker**, gọi là **Docker Server**.

![image.png](https://images.viblo.asia/ebf3a42f-4cfb-498b-98e5-c4d3d6ad06db.png)

**Docker Server** này còn có tên khác là **Docker Daemon**. Và đây mới chính là thành phần thật sự làm việc, nó tạo **container**, tạo **image**, quản lý **container**, tải lên **image**, và gần như xử lý mọi thứ liên quan đến **Docker**.

Tóm lại:

- **Docker Client** là nơi bạn và mình gõ lệnh, tương tác trực tiếp.
- **Docker Server (Daemon)** là chương trình chạy ngầm, nhận lệnh từ **Client** và thực thi mọi công việc thật sự.

Chúng ta sẽ không bao giờ cần chạm trực tiếp vào **Docker Server**. Nó sẽ tự động chạy ngầm phía sau để phục vụ **Client**.

## 4. Hướng dẫn cài đặt Docker (Cho Window)

### Bước 1. Đăng ký tài khoản DockerHub

Truy cập trang [https://app.docker.com/signup](https://app.docker.com/signup) và đăng ký tài khoản.

![image.png](https://images.viblo.asia/341887fc-418d-41b4-8353-ad0f4d19c169.png)

### Bước 2. Cài WSL

**Lưu ý: Nếu trước đây bạn đã bật WSL, bạn có thể bỏ qua bước này và bước 3, bước 4.**

**Kiểm tra bạn đã bật WSL chưa bằng lệnh `wsl -l -v`**

Mở PowerShell với quyền Administrator và chạy lệnh `wsl --install`. Lệnh này sẽ kích hoạt và cài đặt tất cả các tính năng cần thiết cũng như cài đặt Ubuntu.

![image.png](https://images.viblo.asia/4c28bc32-3e79-49f6-a0f9-fd8a0f745faa.png)

Bạn có thể xem qua thông tin ở đây: [https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command](https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command)

### Bước 3. Khởi động lại máy tính của bạn

Bạn chỉ cần tắt hết app và khởi động lại máy thôi.

### Bước 4. Tạo một tài khoản Ubuntu

Sau khi khởi động lại, Windows sẽ tự động khởi chạy hệ điều hành Ubuntu mới của bạn và nhắc bạn đặt username và password như này.

![image.png](https://images.viblo.asia/212685c4-c60a-479d-97eb-855fff25e33c.png)

### Bước 5. Cài đặt Docker Desktop

Truy cập vào trang này và cài đặt Docker Desktop nào: [https://docs.docker.com/desktop/setup/install/windows-install/](https://docs.docker.com/desktop/setup/install/windows-install/)

![image.png](https://images.viblo.asia/d5645b54-8578-45ec-a748-bb3ddb6d7e67.png)

Và giờ chỉ cần mở Docker Desktop là xong rồi!!!

![image.png](https://images.viblo.asia/f0df1c7c-e9ae-4973-8fe7-b93b5f317359.png)

Bây giờ, bạn có thể mở Command Line và gõ lệnh:

```bash
docker version
```

Khi chạy, bạn sẽ thấy một vài dòng thông tin hiện ra, điều đó có nghĩa **Docker** đã được cài đặt thành công.

---

# V. Hello World với Docker

Giờ thì chúng ta sẽ thực hành lệnh **Docker** đầu tiên như khi học bao ngôn ngữ mới khác. Chúng ta sẽ chạy một lệnh rất đơn giản, rồi cùng tìm hiểu từng bước cụ thể xem **Docker** đã làm gì phía sau khi lệnh đó được thực thi.

Cùng mình gõ:

```bash
docker run hello-world
```

Đúng vậy, đây chính là **Hello World** phiên bản của **Docker**. Nghe có vẻ cơ bản, nhưng thực ra khá thú vị đấy. Sau khi nhấn Enter, bạn sẽ thấy một loạt dòng chữ chạy nhanh trên màn hình.

![image.png](https://images.viblo.asia/9da60319-3948-4460-a734-86366f4ce86b.png)

Nếu bạn kéo lên trên một chút, bạn sẽ thấy dòng chữ:

> Hello from Docker!

Ngay bên dưới, **Docker** sẽ liệt kê các bước đã diễn ra khi bạn chạy lệnh đó. Chúng ta sẽ phân tích kỹ từng bước đó ngay sau đây, nên nếu bạn chưa hiểu rõ phần chữ hiện ra, thì cũng không sao cả.

Nếu nhìn kỹ hơn ở phần đầu của kết quả, bạn sẽ thấy dòng thông báo:

> Unable to find image 'hello-world' locally. (Nghĩa là: Không tìm thấy image ‘hello-world’ trong máy.)

Với chi tiết đó, hãy xem sơ đồ dưới đây để hiểu chính xác chuyện gì đã xảy ra khi bạn chạy lệnh vừa rồi.

![image.png](https://images.viblo.asia/ca2efac0-bc72-4caf-b90b-4359491d46f5.png)

Đầu tiên, tại cửa sổ terminal, chúng ta đã gõ lệnh:

```bash
docker run hello-world
```

Điều này khiến **Docker Client** (hay **Docker CLI**) bắt đầu chạy. **Docker Client** là chương trình nhận lệnh của bạn, xử lý một chút, rồi gửi yêu cầu đến **Docker Server** (hay còn gọi là **Docker Daemon**). Chính **Docker Server** mới là nơi thực hiện toàn bộ công việc thực sự.

Khi chúng ta gõ lệnh `docker run hello-world` thì đại khái ý của lệnh là: “Hãy tạo và chạy một container từ image có tên hello-world.”

**Image** **hello-world** này thực ra chỉ chứa một chương trình nhỏ xíu, và nhiệm vụ duy nhất của nó là in ra dòng chữ “Hello from Docker!” mà bạn thấy.

Vậy điều gì đã xảy ra thật sự khi bạn chạy lệnh đó?

**Docker Server** nhận lệnh từ **Client**, và việc đầu tiên nó làm là kiểm tra xem máy của bạn đã có sẵn **image** **hello-world** chưa. Để kiểm tra, Docker nhìn vào **image cache**, tức là nơi lưu các image đã tải trước đó. Nhưng vì bạn vừa mới cài Docker, nên **image cache** đang trống trơn. Vì vậy Docker không tìm thấy “hello-world” trong máy. Do đó, **Docker Server** kết nối đến một nơi tên là **Docker Hub**.

**Docker Hub** là một kho lưu trữ chứa hàng ngàn image công khai, bạn có thể tải về và chạy miễn phí bất cứ cái nào mà mình cần trên máy của mình.

![image.png](https://images.viblo.asia/1f277e26-bb56-4946-8d61-d06317e4d000.png)

**Docker Server** gửi yêu cầu:

> “Tôi đang cần image có tên ‘hello-world’. Bạn (Tức là **Docker hub**) có nó không?”

**Docker Hub** trả lời:

> “Có đây bro!”

Và **Docker Server** bắt đầu tải image đó về máy của bạn, rồi lưu nó vào **image cache**, để sau này nếu bạn chạy lại, Docker có thể dùng ngay mà không cần tải lại.

Sau khi image được tải xong, **Docker Server** nói:

> “Tốt rồi, bây giờ tôi sẽ dùng image này để tạo ra một container.”

**Container** này chính là phiên bản đang chạy của image, và nó chỉ thực hiện một tác vụ duy nhất, ở đây là chạy chương trình in ra dòng chào mừng.

**Docker Server** nạp image vào bộ nhớ, tạo container từ đó, chạy chương trình bên trong, và in ra “Hello from Docker!” trên màn hình.

Vậy là xong, đó chính là toàn bộ quá trình khi bạn chạy lệnh `docker run hello-world`.

**Tóm lại:**

- Docker kiểm tra image trong máy.
- Nếu chưa có, nó tải từ **Docker Hub**.
- Sau đó, nó tạo **container** từ **image** và chạy chương trình bên trong.

Và đến đây, ta đã hiểu được toàn bộ quy trình hoạt động của lệnh `docker run hello-world`.

Ở phần tiếp theo, chúng ta sẽ đi sâu hơn một chút để hiểu rõ chính xác **container** là gì, và nó khác gì so với **image**.

---

# VI. Container chúng ta đã chạy là cái quái gì?

Để hiểu **container**, bạn cần biết sơ qua cách hệ điều hành (OS) hoạt động.

Trong hầu hết các hệ điều hành, có một thành phần trung tâm gọi là **kernel**.

**Kernel** là một tiến trình phần mềm luôn chạy, chịu trách nhiệm điều phối việc truy cập giữa:

- Các chương trình đang chạy trên máy tính bạn.
- Và phần cứng vật lý (CPU, RAM, ổ cứng, mạng...).

![image.png](https://images.viblo.asia/d8d4d63b-c66b-4a60-ba0d-1a405efea171.png)

Ví dụ, ở phía trên sơ đồ ta có các chương trình như: Chrome, Terminal, Spotify, hay Node.js.

Nếu bạn từng dùng Node.js để ghi file xuống ổ đĩa, thật ra không phải Node.js nói chuyện trực tiếp với ổ cứng. Thay vào đó, Node.js sẽ nói với **kernel**:

> “Này kernel, tôi muốn ghi một file xuống ổ cứng nhé.”

**Kernel** sẽ tiếp nhận yêu cầu đó và thực sự thực hiện việc ghi dữ liệu xuống ổ đĩa. Tức là **kernel** chính là lớp trung gian giữa các chương trình và phần cứng.

Các chương trình tương tác với kernel thông qua những **System call**, hiểu nôm na là các **lời gọi hàm**.

**Kernel** cung cấp một loạt hàm, ví dụ:

- Ghi file thì gọi hàm này.
- Đọc file thì gọi hàm kia.
- Gửi dữ liệu mạng thì gọi hàm khác.

Và **kernel** sẽ xử lý, điều phối yêu cầu đó đến phần cứng tương ứng.

### **Tình huống giả định**

Giả sử bạn có 2 chương trình đang chạy:

- Chrome.
- Node.js.

Và trong một vũ trụ song song giả định, Chrome cần Python 2, còn Node.js cần Python 3 để hoạt động.

Nhưng máy bạn chỉ được phép có một bản cài Python. Vậy Chrome sẽ hoạt động, còn Node.js thì không. Chúng ta cần một cách để cả hai chương trình dùng hai môi trường khác nhau, cùng lúc trên cùng một máy.

### **Giải pháp - Vị cứu tinh Namespaces**

Hệ điều hành cung cấp một tính năng gọi là **namespacing**. Tính năng này cho phép phân tách tài nguyên của phần cứng ra từng vùng độc lập.

Ví dụ:

- Tạo một vùng ổ cứng riêng chỉ chứa Python 2 (cho Chrome).
- Tạo một vùng khác chứa Python 3 (cho Node.js).

Khi Chrome gọi hệ thống để đọc file, **kernel** sẽ biết yêu cầu đến từ Chrome, và chỉ cho nó đọc trong vùng Python 2. Tương tự, Node.js chỉ được đọc trong vùng Python 3.

Như vậy, cả hai chương trình cùng tồn tại, dùng hai môi trường khác nhau, trên cùng một hệ thống.

### Namespacing và Control Groups

![image.png](https://images.viblo.asia/dcc8fbf3-7f7f-47c5-9c6f-750a600eeab4.png)

- **Namespaces** → giúp chia tách tài nguyên như ổ đĩa, mạng, tiến trình, v.v... cho từng nhóm chương trình riêng biệt.
- **Control Groups (cgroups)** → giúp giới hạn tài nguyên bằng việc bạn có thể quy định một process chỉ được dùng tối đa 512MB RAM, hoặc chỉ được dùng 20% CPU, hoặc giới hạn tốc độ mạng, I/O…

Hai cơ chế này kết hợp lại giúp cô lập hoàn toàn một tiến trình, nó chỉ nhìn thấy tài nguyên riêng và bị giới hạn trong phạm vi mà ta đã quy định.

### Và đó chính là Container

Như vậy, một **container** không phải là thứ gì đó vật lý hay **hộp thật** trong máy bạn. Nó chỉ là một hoặc nhiều tiến trình đang chạy được:

- Cô lập bằng **namespaces**.
- Giới hạn bằng **control groups**.
- Và gắn với một tập tài nguyên riêng biệt (ổ đĩa, CPU, RAM...).

### Mối quan hệ giữa Image và Container

Khi bạn nghe nói đến **Docker image**, hãy hiểu rằng đó là một **snapshot** của hệ thống file:

- Gồm các thư mục, file, chương trình, cấu hình.
- Và một lệnh khởi động mặc định (**startup command**) (ví dụ: `npm start`, `python app.py`…).

Khi Docker tạo **container**:

- **Kernel** tạo ra một vùng ổ cứng cô lập riêng.
- Nội dung của **image** được copy vào vùng đó.
- Sau đó Docker chạy lệnh khởi động (**startup command**) bên trong vùng này → Tạo ra tiến trình chính của container (ví dụ: chạy Chrome, hoặc Node.js).

**Container** chính là tiến trình này + tập tài nguyên bị cô lập mà nó được cấp.

---

# VII. Tổng kết phần 1

Và đó là toàn bộ lý thuyết mà mình muốn truyền đạt đến mọi người hiểu về **DOCKER** và **KUBERNETES** thực sự là gì.

Bài viết đã rất là dài rồi nên mình sẽ cắt phần 1 ở đây, sang phần 2 chúng ta sẽ thực hành các lệnh Docker nhé.

Mình xin nhắc lại là bài viết của mình không thể thay thế được những kiến thức khóa học từ anh Sờ Te Phờn, các bạn hãy đăng ký và trải nghiệm thử.

Nếu bạn thấy hay thì cho mình xin 1 upvote và để lại comment những chỗ mình đã sai, hoặc muốn thảo luận thêm nhé.

Mình là NekoArcoder, xin hẹn gặp lại các bạn ở bài viết tiếp theo!!!
