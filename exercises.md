# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu em quên khai báo AGENT_API_KEY, Settings sẽ báo lỗi khi đọc cấu hình, giúp em phát hiện thiếu khóa. Nếu đặt mặc định là "changeme", ứng dụng có thể tiếp tục chạy với một khóa dễ đoán. Người khác biết khóa mặc định có thể gọi API trái phép và làm phát sinh chi phí. Vì vậy, bắt buộc cung cấp khóa giúp phát hiện lỗi cấu hình sớm.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log em thu được khi gọi `/ask` ở local: {"event":"ask_completed","level":"info","timestamp":"2026-09-28T15:56:05.408883+00:00","user_id":"exercise-2","tokens_in":1,"tokens_out":35,"cost_usd":2.115e-05}Hai việc làm với dòng log trên: có thể lọc các lượt gọi theo user_id và timestamp để biết người dùng nào gọi vào thời điểm nào; có thể cộng cost_usd và thống kê tokens_in, tokens_out để theo dõi chi phí và lượng token sử dụng. Dòng “đã trả lời xong” không cung cấp các dữ liệu này.


---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.730 MB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage dùng image nền python:3.11 đầy đủ, copy toàn bộ thư mục dự án và cài dependency trực tiếp trong image. Bản multi-stage dùng python:3.11-slim, chỉ copy thư viện đã cài từ builder cùng hai thư mục app và utils sang runtime, đồng thời không lưu cache pip. Vì vậy, bản hiện tại giảm các công cụ, thư viện hệ thống và file không cần thiết khi chạy. Chênh lệch không chỉ do multi-stage mà còn do đổi sang base image slim và giới hạn nội dung được copy.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Sau khi thêm comment vào app/main.py và build lại, em thấy các bước WORKDIR, COPY requirements.txt, RUN pip install và COPY thư viện từ builder đều báo CACHED. Dependency không thay đổi nên Docker không cần cài lại. Bước COPY app ./app chạy lại vì source thay đổi. Các bước phía sau là COPY utils ./utils và RUN useradd cũng chạy lại, dù nội dung utils không đổi, vì chúng nằm sau layer đã thay đổi. Nếu đặt COPY . . trước RUN pip install trong cùng stage, mỗi lần thay đổi source sẽ làm mất cache của bước COPY và các bước sau đó, khiến pip install phải chạy lại dù requirements.txt không thay đổi. Vì vậy, copy requirements.txt và cài dependency trước khi copy source giúp build lại nhanh hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗ hổng cho phép thực thi lệnh, kẻ tấn công có thể chạy lệnh với quyền của tiến trình ứng dụng. Nếu tiến trình chạy bằng root, họ có quyền cao trong container, dễ sửa file hệ thống hoặc cài thêm công cụ tấn công. Root trong container không tự động là root trên host. Tuy nhiên, nếu container được cấp quyền quá rộng, mount tài nguyên nhạy cảm của host hoặc có lỗ hổng thoát container, kẻ tấn công có thể tiếp tục ảnh hưởng đến máy host. Trong Dockerfile của em, USER appuser khiến ứng dụng chạy bằng user thường. Khi bị khai thác, kẻ tấn công ban đầu chỉ có quyền của user này, giúp giảm phạm vi thiệt hại. Đây là một lớp bảo vệ, không thay thế việc sửa lỗ hổng và cấu hình container an toàn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

>  Với đếm theo phút đồng hồ (hạn mức 10/phút), người dùng có thể gửi tối đa 20 request trong 2 giây. Cách đạt được: gửi 10 request lúc 10:00:59 (cuối phút 10:00) và 10 request lúc 10:01:01 (đầu phút 10:01) — mỗi phút chỉ có 10 request nhưng thực tế 20 request xảy ra trong 2 giây. Sliding window tránh được lỗ hổng này vì luôn nhìn vào 60 giây gần nhất, không có kẽ hở tại ranh giới phút.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số lượng request trong một khoảng thời gian; cost guard giới hạn tổng chi phí trong tháng. Tình huống rate limit cho qua nhưng cost guard chặn: user gửi đúng 5 request/phút (trong hạn mức 10), nhưng mỗi request gửi 50.000 token khiến chi phí vượt ngân sách tháng. Tình huống ngược lại: user gửi 20 request/phút (vượt rate limit → bị chặn 429) nhưng tổng chi phí tháng vẫn còn trong ngân sách — cost guard sẽ cho qua nếu rate limit không tồn tại.
---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Thứ tự sự kiện: (1) Redis mất kết nối 30 giây; (2) endpoint gộp kiểm tra Redis và trả 503; (3) orchestrator đọc 503 từ liveness probe → kết luận container cần restart → restart cả 3 container cùng lúc; (4) trong lúc 3 container đang khởi động lại, không có instance nào phục vụ traffic; (5) khi Redis quay lại thì cụm mới dần ổn định, nhưng đã có khoảng downtime toàn hệ thống. Nếu tách riêng: /health không check Redis nên vẫn 200, orchestrator không restart; /ready trả 503 → load balancer ngừng gửi traffic vào, không restart container → khi Redis hồi phục, /ready trả 200 và traffic vào lại bình thường, không có downtime.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lịch sử lưu trong dict Python, mỗi container có một dict riêng. Với 3  container sau load balancer, câu hỏi của user có thể đến container A, câu tiếp theo đến container B. Container B không biết lịch sử từ A nên history_length sẽ dao động ngẫu nhiên: có lúc tăng (gặp đúng container đã lưu), có lúc reset về 0 hoặc 1 (gặp container chưa có history). Khi lưu trên Redis, mọi container đều đọc/ghi chung một nơi nên history_length tăng đều đặn qua mỗi lượt hỏi, bất kể container nào xử lý.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi kiểm tra bản deploy trên Railway bằng lệnh curl với `-H "X-API-Key: $AGENT_API_KEY"`, service luôn trả về 401 dù đã set AGENT_API_KEY trong Railway dashboard. Thông báo lỗi là `{"detail":"invalid or missing API key"}`. Ban đầu em nghĩ key trên Railway bị sai, nhưng sau khi kiểm tra lại thì phát hiện nguyên nhân: trong Windows Command Prompt, biến `$AGENT_API_KEY` không tự động expand như trên Linux/macOS — cmd gửi chuỗi `$AGENT_API_KEY` nguyên văn lên server thay vì giá trị thật. Ngoài ra, JSON body dùng dấu nháy đơn `'{"question":"..."}` cũng không hợp lệ trong cmd, gây lỗi 422. Cách sửa: dùng `%AGENT_API_KEY%` thay cho `$AGENT_API_KEY` trong cmd, hoặc chuyển sang PowerShell và set biến bằng `$env:AGENT_API_KEY = "..."` rồi dùng `curl.exe`. JSON body phải dùng nháy kép với escape: `"{\"question\":\"...\"}"`.
