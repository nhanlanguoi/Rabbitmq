# Chạy và Kiểm tra Ứng dụng Spring Boot với RabbitMQ

Phần này hướng dẫn cách chạy ứng dụng Spring Boot đã được cấu hình với RabbitMQ và cách kiểm tra chức năng gửi và nhận tin nhắn.

## 1. Chạy Ứng dụng

### a. Đảm bảo RabbitMQ Server đang chạy
Trước khi khởi động ứng dụng Spring Boot, hãy chắc chắn rằng RabbitMQ server của bạn đã được cài đặt và đang hoạt động.

* Bạn có thể kiểm tra giao diện quản lý của RabbitMQ (RabbitMQ Management Plugin) bằng cách truy cập vào địa chỉ sau trong trình duyệt web:
    `http://localhost:15672/`
* Thông tin đăng nhập mặc định thường là:
    * **User:** `guest`
    * **Password:** `guest`

### b. Chạy ứng dụng Spring Boot
Có hai cách phổ biến để chạy ứng dụng Spring Boot của bạn:

* **Cách 1: Sử dụng Maven (trong terminal)**
    Mở terminal hoặc command prompt, điều hướng đến thư mục gốc của dự án (nơi chứa tệp `pom.xml`), và chạy lệnh sau:
    ```bash
    mvn spring-boot:run
    ```

* **Cách 2: Chạy từ IDE**
    Nếu bạn đang sử dụng một Môi trường Phát triển Tích hợp (IDE) như IntelliJ IDEA, Eclipse, hoặc VSCode:
    1.  Mở dự án của bạn trong IDE.
    2.  Tìm đến lớp chính của ứng dụng (thường có tên là `*Application.java`, ví dụ: `RabbitmqDemoApplication.java`).
    3.  Chạy lớp này như một ứng dụng Java (thường có tùy chọn "Run" hoặc biểu tượng tam giác màu xanh).

## 2. Kiểm tra Chức năng

Sau khi ứng dụng Spring Boot đã khởi động thành công và kết nối với RabbitMQ, bạn có thể tiến hành kiểm tra.

### a. Gửi Tin nhắn
Sử dụng một công cụ như Postman, Insomnia, hoặc lệnh `curl` trong terminal để gửi một HTTP POST request đến endpoint đã được tạo trong `MessageController`.

* **Để gửi tin nhắn dạng chuỗi đơn giản:**
    Gửi một `POST` request đến:
    `http://localhost:8080/api/v1/messages/send?message=HelloRabbitMQ`

    Ví dụ với `curl`:
    ```bash
    curl -X POST "http://localhost:8080/api/v1/messages/send?message=HelloRabbitMQ"
    ```

* **Để gửi tin nhắn dạng đối tượng `CustomMessage` (nếu bạn đã triển khai endpoint này):**
    Gửi một `POST` request đến:
    `http://localhost:8080/api/v1/messages/send-custom`

    Với **JSON body** như sau:
    ```json
    {
      "id": "123",
      "content": "This is a custom message"
    }
    ```
    Ví dụ với `curl`:
    ```bash
    curl -X POST -H "Content-Type: application/json" \
    -d '{
          "id": "123",
          "content": "This is a custom message"
        }' \
    http://localhost:8080/api/v1/messages/send-custom
    ```

### b. Kiểm tra Console (Log Output)
Quan sát cửa sổ console nơi ứng dụng Spring Boot của bạn đang chạy:

1.  **Log từ `MessageProducer`:** Bạn sẽ thấy một dòng log (thường là `INFO`) cho biết tin nhắn đã được gửi đi. Ví dụ:
    ```
    INFO --- [nio-8080-exec-1] c.e.r.producer.MessageProducer     : Sending message -> HelloRabbitMQ
    ```
    Hoặc cho `CustomMessage`:
    ```
    INFO --- [nio-8080-exec-2] c.e.r.producer.MessageProducer     : Sending custom message -> CustomMessage{id='123', content='This is a custom message'}
    ```

2.  **Log từ `MessageConsumer`:** Ngay sau đó (hoặc gần như đồng thời), bạn sẽ thấy một dòng log khác cho biết tin nhắn đã được nhận và xử lý bởi consumer. Ví dụ:
    ```
    INFO --- [ntContainer#0-1] c.e.r.consumer.MessageConsumer     : Received message -> HelloRabbitMQ
    ```
    Hoặc cho `CustomMessage`:
    ```
    INFO --- [ntContainer#0-1] c.e.r.consumer.MessageConsumer     : Received custom message -> CustomMessage{id='123', content='This is a custom message'}
    ```
    *(Lưu ý: Tên thread (`[ntContainer#0-1]`) có thể khác nhau)*

### c. Kiểm tra Giao diện Quản lý RabbitMQ (RabbitMQ Management UI)
Kiểm tra lại giao diện quản lý của RabbitMQ để xác nhận trạng thái của queues và exchanges:

1.  Truy cập `http://localhost:15672/` trong trình duyệt.
2.  Đăng nhập (nếu được yêu cầu).
3.  **Vào tab "Queues":**
    * Bạn sẽ thấy hàng đợi đã được định nghĩa (ví dụ: `myQueue`) được tạo ra.
    * Các cột như "Ready" (sẵn sàng), "Unacked" (chưa xác nhận), "Total" (tổng số) sẽ hiển thị số lượng tin nhắn. Nếu tin nhắn được xử lý nhanh, bạn có thể thấy số lượng này về 0 ngay lập tức.
    * Nếu bạn gửi nhiều tin nhắn liên tục trước khi consumer kịp xử lý, bạn có thể thấy số lượng tin nhắn "Ready" tăng lên.
4.  **Vào tab "Exchanges":**
    * Bạn sẽ thấy exchange đã được định nghĩa (ví dụ: `myExchange`) được tạo ra.
    * Bạn có thể nhấp vào tên exchange để xem chi tiết và các bindings của nó tới queues.

Nếu tất cả các bước trên đều cho kết quả như mong đợi, hệ thống RabbitMQ đơn giản của bạn với Spring Boot đã hoạt động chính xác!
