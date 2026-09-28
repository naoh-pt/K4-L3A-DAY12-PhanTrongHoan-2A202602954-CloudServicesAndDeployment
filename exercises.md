# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: điền câu trả lời vào phần bên dưới mỗi câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phan Trọng Hoàn  Mã học viên: 2A202602954

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> ví dụ khi deploy lên môi trường production nhưng quên khai báo key, nếu có giá trị mặc định changeme, ứng dụng vẫn deploy thành công nhưng toàn bộ request gọi đến đều thất bại hoặc tệ hơn là bị khai thác key mặc định để truy cập trái phép. nhờ fail fast mà ứng dụng không thể deploy và sớm phát hiện ra lỗi.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Một dòng log thực tế khi gọi `/ask` trên container local:

```json
{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T11:32:02.066498+00:00", "user_id": "exercise-log", "tokens_in": 3, "tokens_out": 41, "cost_usd": 2.505e-05}
```

Có thể lọc log theo `user_id` và cộng `cost_usd` để theo dõi chi phí của từng user. Ngoài ra, có thể thống kê số request và token theo thời gian để tìm những lượt gọi dùng nhiều tài nguyên. Dòng `print("đã trả lời xong")` không chứa các trường cần thiết cho hai việc đó.


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
| 1 stage (bản đầu) | 1.73 GB (khoảng 1730 MB) |
| Multi-stage | 309 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Chênh lệch đến từ việc đổi base image đầy đủ `python:3.11` sang `python:3.11-slim`, giảm các công cụ và thư viện hệ điều hành không cần cho runtime. Bản mới chỉ copy môi trường Python từ builder cùng `app/` và `utils/`, còn bản đầu copy toàn bộ repo sau khi áp dụng `.dockerignore`. Multi-stage cho phép chọn thành phần đưa vào runtime; mức giảm ở bài này còn nhờ đổi base image và giới hạn source được copy. Cả hai bản vẫn cài cùng `requirements.txt`, bao gồm thư viện test.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi rebuild sau khi sửa source ở CP3/CP4, log Docker báo `CACHED` cho tạo venv, copy `requirements.txt`, cài dependency, tạo user và copy venv từ builder. Bước `COPY app/ ./app/` chạy lại và image được xuất lại; không phải cài lại dependency.

Nếu đặt `COPY . .` trước `RUN pip install`, thay đổi source làm mất cache của bước copy và các bước sau, bao gồm cài thư viện. Vì vậy copy file dependency và cài thư viện trước giúp những lần sửa code build nhanh hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Lỗ hổng trong app có thể cho kẻ tấn công thực thi lệnh bằng quyền của process Python. Nếu process chạy root, kẻ tấn công có quyền root trong container. Khi có thêm volume nhạy cảm, cấu hình đặc quyền hoặc lỗ hổng thoát container, họ có thể gây hại tới host.

Root trong container không tự động là root trên host. Lệnh `USER agent` giảm quyền của process ngay khi app bị chiếm, nhưng vẫn cần cấu hình cách ly phù hợp. Kiểm tra container của bài cho kết quả `uid=10001(agent) gid=10001(agent)`, không phải UID 0.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Tối đa 20 request: gửi 10 request lúc 10:00:59 và 10 request lúc 10:01:00. Quota được reset khi sang phút mới nên hai đợt sát nhau vẫn được cho qua, dù có tới 20 request trong khoảng 2 giây.

Sliding window đếm 60 giây gần nhất nên vẫn tính cả đợt trước, chặn request vượt hạn mức 10.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit giới hạn số request trong 60 giây; cost guard giới hạn tổng chi phí của từng user trong tháng UTC.

User mới gọi một request trong phút nhưng đã tiêu hơn 10 USD trong tháng: rate limit cho qua, cost guard trả 402. Ngược lại, user mới tiêu 0,01 USD nhưng gửi request thứ 11 trong 60 giây: rate limit trả 429 dù vẫn còn ngân sách.

Kiểm tra thực tế local cho thấy request thứ 11 trả 429. Khi đặt chi phí của user kiểm tra lên 999 USD trong Redis, `/ask` trả 402 trước khi gọi mock LLM.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Redis mất kết nối → endpoint chung của cả ba instance trả lỗi → readiness khiến load balancer ngừng gửi traffic tới chúng → nếu orchestrator được cấu hình restart sau đủ số lần lỗi liveness, các instance có thể bị restart → Redis chưa phục hồi nên instance mới tiếp tục lỗi. Restart app không khắc phục được Redis và có thể làm gián đoạn thêm request.

Restart phụ thuộc nền tảng và cấu hình; Docker HEALTHCHECK chỉ đánh dấu unhealthy, không tự restart. Khi thử dừng Redis local với hai endpoint tách riêng, `/health` vẫn trả 200 còn `/ready` trả 503 với `{"status":"not ready","redis":false}`.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Đã chạy ba agent bằng file bổ sung `docker-compose.scale.yml` để tránh trùng cổng. Gửi lần lượt ba request cùng `X-User-Id` tới ba container cho kết quả `history_length` là 0, 2, 4. Mỗi lượt thêm hai message user và assistant nên container sau thấy dữ liệu của container trước. TTL đo được là 604800 giây (7 ngày), lịch sử giữ tối đa 20 message.

Nếu dùng dict Python, mỗi container có lịch sử riêng. Ba request đầu vào ba container khác nhau có thể đều trả 0. Những lần tiếp theo có thể thấy 2, 4 hoặc giá trị thấp hơn tùy container nhận request; restart container làm mất lịch sử trong dict.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Khi deploy lên Railway, `/health` trả 200 nhưng `/ready` trả 500. Đọc Deploy Logs thấy:

```text
ValueError: Redis URL must specify one of the following schemes (redis://, rediss://, unix://)
```

Traceback dẫn đến `get_redis_client()` và `redis.from_url()`, cho thấy giá trị `REDIS_URL` không phải URL Redis hợp lệ. Cách sửa là dùng URL kết nối thật hoặc reference variable từ service Redis, thay vì tên biến hay hostname đơn lẻ.

Sau đó chuyển sang Render và cấu hình Blueprint lấy `REDIS_URL` từ `connectionString` của `day12-redis`; web service và Redis cùng vùng Singapore. Kiểm tra trên `https://day12-agent-mrev.onrender.com` cho thấy `/health` trả 200, `/ready` trả 200 với `{"status":"ready","redis":true}` và `/ask` không có API key trả 401. Như vậy bản Render đã kết nối Redis thành công.
