# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Phan Trọng Hoàn |
| Mã học viên | 2A202602954 |
| Repo | https://github.com/naoh-pt/K4-L3A-DAY12-PhanTrongHoan-2A202602954-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-mrev.onrender.com |
| Platform | Render |
| Ngày deploy | 28/09/2026 — ngày xác nhận service hoạt động |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | Service đã nhận request | Render quản lý cổng; chưa đọc dashboard để xác nhận giá trị |
| `AGENT_API_KEY` | Đã kiểm tra | Thiếu khóa trả 401; khóa đúng trả 200 có câu trả lời |
| `REDIS_URL` | Kết nối Redis thành công | `/ready` trả 200; Blueprint tham chiếu day12-redis |
| `RATE_LIMIT_PER_MINUTE` | Đã kiểm tra trên cloud | 10 request đầu trả 200; request thứ 11 trong 60 giây trả 429 |
| `MONTHLY_BUDGET_USD` | Khai báo trong render.yaml | 10.0 |
| `LOG_LEVEL` | Khai báo trong render.yaml | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Kết quả HTTP kiểm tra ngày 28/09/2026:

```
GET /health → 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready → 200
{"status":"ready","redis":true}
POST /ask không có API key → 401
{"detail":"invalid or missing API key"}
POST /ask có API key đúng → 200, có câu trả lời, history_length=0 ở lượt đầu
11 request liên tiếp với cùng X-User-Id:
[200, 200, 200, 200, 200, 200, 200, 200, 200, 200, 429]
Retry-After: 60
pytest tests/test_cp5.py -v → 9 passed, 4 skipped
4 test bị bỏ qua thuộc phương án local fallback, không dùng cho bản Render.
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

Đã kiểm tra hai ảnh: dashboard hiển thị `Deploy succeeded | Live` ngày 28/09/2026; ảnh health hiển thị đúng domain Render và JSON trạng thái `ok`. Không thấy API key hoặc mật khẩu trong hai ảnh.

---

## Trạng Thái Bài Nộp

Đã deploy lên Render, không dùng phương án dự phòng; đủ hai ảnh minh chứng. Đã kiểm tra `/ask` có API key và rate limit trên cloud thành công.
