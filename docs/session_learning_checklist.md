# Danh sách kiểm tra bài học: Observability Lab

Danh sách này giúp theo dõi tiến trình nắm bắt kiến thức của bạn trong phiên làm việc này. Chúng ta sẽ đi từng bước một.

## Bài học 1: Khắc phục lỗi tương thích Python và Môi trường ảo

### 1. Vấn đề (The Problem)
- [x] Tại sao lệnh `pip install` lại thất bại lúc ban đầu?
- [x] Vấn đề cốt lõi liên quan đến phiên bản Python 3.14 là gì?
- [x] Tại sao các thư viện như `pydantic-core` hay `jiter` (sử dụng Rust/C dưới nền) lại bị ảnh hưởng?

### 2. Giải pháp (The Solution)
- [x] Tại sao chúng ta lại chọn giải quyết bằng cách cài đặt Python 3.12 thay vì cố gắng sửa code của thư viện?
- [x] Tại sao chúng ta phải xóa và tạo lại thư mục `.venv`?
- [x] Những trường hợp ngoại lệ (edge cases) nào có thể xảy ra khi xử lý việc này?

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Việc quản lý phiên bản Python và thư viện đóng vai trò quan trọng như thế nào trong dự án thực tế?
- [x] Tác động của việc sử dụng các phiên bản phần mềm "pre-release" (tiền phát hành) lên độ ổn định của hệ thống.
- [x] Cách thiết lập môi trường chuẩn để tránh các lỗi tương tự trong tương lai.

---

## Bài học 2: Correlation IDs (ID Tương quan)

### 1. Vấn đề (The Problem)
- [x] Correlation ID là gì và tại sao chúng ta lại cần nó trong ghi log (logging)?
- [x] Chuyện gì sẽ xảy ra với quá trình debug (gỡ lỗi) khi hệ thống nhận hàng nghìn request chạy song song nếu không có Correlation ID?
> **Câu trả lời của bạn:** Correlation_id chỉ là một đoạn mã được tạo ra, nên việc tìm thấy đoạn log chứa ID này thì cũng không giải quyết được vấn đề nào cả. Chúng ta sẽ phải load toàn bộ correlation id đó ra để kiểm xem thử họ lỗi ở đâu, lỗi như thế nào.

### 2. Giải pháp (The Solution)
- [x] Cách tạo và nhúng `x-request-id` vào mỗi request bằng Middleware trong FastAPI.
- [x] Làm thế nào để tự động gắn ID này vào mọi dòng log mà không phải truyền nó qua tham số (parameter) của từng hàm?
- [x] Những trường hợp ngoại lệ (Edge cases): Điều gì xảy ra nếu client (người gọi API) đã tự gửi lên một `x-request-id` từ trước?

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Tác động của Correlation ID đến kiến trúc Microservices (truy vết chuỗi request đi qua nhiều dịch vụ khác nhau).
- [x] Mối liên hệ giữa Correlation ID và Observability (Khả năng quan sát toàn diện hệ thống).

---

## Bài học 3: Enrich Logs (Làm giàu thông tin Log)

### 1. Vấn đề (The Problem)
- [x] Tại sao chỉ có `correlation_id` là chưa đủ?
- [x] Chuyện gì sẽ xảy ra nếu ta cần thống kê toàn bộ lịch sử lỗi của một **user cụ thể** hoặc một **tính năng cụ thể**?
> **Câu trả lời của bạn:** Để giải quyết vấn đề này thì em nghĩ nên thêm user_id để xem thử request này là của ai để fix, thời gian để biết họ request khi nào, request này xử lý trong bao lâu. feature đã gọi. Nếu user_id là thông tin cá nhân nhạy cảm thì phải chia ra để xử lý ví dụ như thông tin về email, số điện thoại thì phải mã hóa nó đi hoặc xóa nó đi, xử lý nó thành dữ liệu duy nhất để chỉ log những field đã duyệt. Việc làm giàu log thì giúp người debug biết lỗi rõ hơn nhanh hơn, hỗ trợ nhiều cho việc fix bug. Đưa ra cảnh báo.

### 2. Giải pháp (The Solution)
- [x] Cập nhật file `app/main.py` để lấy các thông tin ngữ cảnh (`user_id`, `session_id`, `feature`, `env`) từ Payload của Request.
- [x] Dùng `bind_contextvars()` để gài các thông tin này vào mọi dòng log.
- [x] Edge cases: Điều gì xảy ra nếu `user_id` là thông tin cá nhân nhạy cảm (PII - Personally Identifiable Information)? Tại sao lại phải băm (hash) `user_id`?

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Lợi ích của việc làm giàu log (Enrich Logs) đối với hệ thống Dashboard (Bảng điều khiển) và Alerting (Cảnh báo).

---

## Bài học 4: Sanitize Data (Che khuất Dữ liệu nhạy cảm - PII)

### 1. Vấn đề (The Problem)
- [x] Tại sao thông tin PII (thẻ tín dụng, email, CMND/CCCD) lọt vào Log lại nguy hiểm hơn lọt vào Database?
- [x] Những hậu quả pháp lý nào có thể xảy ra nếu hệ thống ghi lại toàn bộ tin nhắn raw của người dùng?
> **Câu trả lời của bạn:** Việc lọt vào log sẽ có nhiều lập trình viên có thể đọc được khi họ fixbug. Database thì chỉ có những người có đủ uy tín mới được xem và biết. Nhưng log là thứ mà lập trình viên nhìn vào để biết chỗ mà sửa lỗi. Gây ra nhiều hậu quá pháp lý. Vì vậy ta cần một thứ để mã hóa nó đi. Chỉ những data đã đi qua nó thì mới được lưu và trong log.

### 2. Giải pháp (The Solution)
- [x] Làm thế nào để lọc/che khuất PII ở MỘT NƠI DUY NHẤT (Centralized) thay vì phải dặn dò các lập trình viên gọi hàm `scrub()` mỗi khi họ ghi log?
- [x] Cách sử dụng Processor của `structlog` để chặn các thông tin nhạy cảm.

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Mối liên hệ với các tiêu chuẩn bảo mật (GDPR, PCI-DSS, HIPAA).

---

## Bài học 5: Validate Logs & Tracing (Xác thực Log & Truy vết chi tiết)

### 1. Vấn đề (The Problem)
- [x] Con người đọc log thì hiểu, nhưng làm sao máy móc (như Datadog/Elasticsearch) biết được cấu trúc log của bạn có hợp lệ không để vẽ Biểu đồ (Dashboard)?
- [x] Nếu một request mất 10 giây để chạy, Log bình thường chỉ báo "Bắt đầu" ở giây thứ 1 và "Kết thúc" ở giây 10. Làm sao bạn biết chính xác 10 giây đó bị "nghẽn" ở hàm nào (Gọi Database? Gọi API ngoài? Xử lý ảnh?)?
> **Câu trả lời của bạn:** 1. Nếu có một lập trình lỡ tay đổi. thì mình nên có quy ước rõ ràng. Nếu thay đổi log.info(...) sẽ bị lỗi log và cảnh báo. 2. Ta nên chia ra xem luồng seasion xem mỗi luồng chạy bao nhiêu giây. Vai trò của Json schema trong trường hợp này là để giúp máy có thể hiểu được format chuẩn chỉ không bị việc đọc không hiểu log. Khác biệt giữa log ghi sự kiện là cho mình lỗi để debug còn tracing là điểm nghẽn ở đâu. Em nghĩ thế.

### 2. Giải pháp (The Solution)
- [x] Chạy script tự động xác thực (validate) cấu trúc JSON của file log dựa trên một khuôn mẫu (Schema) chuẩn.
- [x] Sử dụng công cụ Tracing (Langfuse) và decorator `@observe` để đo lường và theo dõi từng hàm một bên trong một request.

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Vai trò của JSON Schema trong việc đảm bảo hệ thống Monitoring không bị "gãy" khi có ai đó lỡ đổi tên một trường (field) trong log.
- [x] Sự khác biệt giữa Logging (ghi log sự kiện) và Tracing (truy vết luồng chạy).

---

## Bài học 6: Dashboards (Bảng điều khiển)

### 1. Vấn đề (The Problem)
- [x] Dù log có chi tiết (enriched) đến đâu, hàng triệu dòng log dạng JSON cũng là vô nghĩa đối với CEO, Giám đốc sản phẩm (PM) hay lúc bạn đang ngái ngủ lúc 3h sáng. Vì sao?
- [x] "Golden Signals" (Các tín hiệu vàng) là gì và tại sao chúng ta lại chỉ nên tập trung vào 6-8 biểu đồ cốt lõi?
> **Câu trả lời của bạn:** Việc chuyển hóa log thành dashbroads sẽ giúp ta hiểu nhanh hơn. 10k dòng log đó ta sẽ khó hiểu nó đang cho ra cái gì. Trong khi chuyển hóa thành các biểu đồ thì ta sẽ có cái nhìn trực quan hơn về các dòng dữ liệu. Ví dụ ta có 10 người chọn đáp án a,b,c ta sẽ phải đọc hết 10 dòng đó ta mới biết được bao nhiêu người chọn a hoặc b hoặc c. Trong khi chỉ cần 1 biểu đồ là ta đã có thể biết được chỉ trong vài giây.

### 2. Giải pháp (The Solution)
- [x] Các thông số: Độ trễ (Latency), Lưu lượng (Traffic), Tỷ lệ lỗi (Error rate), Chi phí (Cost), Tokens, Chất lượng (Quality).
- [x] Tại sao phải vẽ thêm đường giới hạn SLO (Service Level Objective) trên biểu đồ?

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Tại sao một Dashboard có tới 30 biểu đồ chi chít lại là một Dashboard "thất bại"?

---

## Bài học 7: Alerting (Cảnh báo Sự cố)

### 1. Vấn đề (The Problem)
- [x] Dù Dashboard có đẹp đến mấy, không một ai (kể cả Tech Lead) có thể ngồi nhìn chằm chằm vào màn hình 24/7. Làm sao để hệ thống tự gọi điện thoại cho chúng ta lúc 3h sáng khi có lỗi?
- [x] Hội chứng "Mệt mỏi vì cảnh báo" (Alert Fatigue) là gì?

### 2. Giải pháp (The Solution)
- [x] Phân tích cấu hình cảnh báo `error_rate_pct > 5 for 5m`. Tại sao lại có thêm chữ `for 5m`?
> **Câu trả lời của bạn:** Tại vì người ta cần một khoảng thời gian đủ dài để đánh giá được model đang hoạt động đúng chính xác hay không. Nếu set là 1s thì quá ngắn để nói model đó hoạt động lỗi vì đôi khi hệ thống sẽ phải có sai sót và nếu set quá dài thì mình sẽ không thể sửa kịp thời.
- [x] Khái niệm "Symptom-based Alerting" (Cảnh báo dựa trên Triệu chứng).

### 3. Ngữ cảnh rộng hơn (Broader Context)
- [x] Tại sao mọi cảnh báo đều phải đi kèm với một `runbook` (Tài liệu hướng dẫn xử lý)?
