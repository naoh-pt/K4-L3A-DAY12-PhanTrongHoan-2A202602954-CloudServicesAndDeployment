# CP5 — Deploy trên Railway

## Chuẩn bị

Push cấu hình mới trước khi tạo service:

```powershell
git add railway.toml DEPLOYMENT.md CP5_SETUP.md
git commit -m "CP5: chuẩn bị cấu hình cloud deployment"
git push origin main
```

Không upload `.env`. Chỉ nhập secret trong dashboard của nền tảng.

## Railway

1. Đăng nhập https://railway.com, tạo project từ GitHub repo của bài, branch `main`.
2. Thêm Redis vào cùng project.
3. Trong Variables của agent, đặt `AGENT_API_KEY` bằng khóa trong `.env` local (hoặc khóa riêng cho cloud). Thêm reference variable `REDIS_URL` trỏ đến `REDIS_URL` của service Redis; nếu service có tên `Redis`, giá trị tham chiếu là `${{Redis.REDIS_URL}}`. Thêm `RATE_LIMIT_PER_MINUTE=10`, `MONTHLY_BUDGET_USD=10.0`, `LOG_LEVEL=INFO`.
4. Giữ lệnh khởi động mặc định từ Dockerfile, không nhập `$PORT` dưới dạng exec command riêng.
5. Generate Domain trong Networking để lấy URL HTTPS. Kiểm tra kế hoạch và chi phí hiển thị trước khi tạo tài nguyên.

## Hoàn thiện bài nộp

1. Mở `<URL>/health` và `<URL>/ready`: cả hai phải trả HTTP 200.
2. Gửi lại URL công khai để kiểm tra CP5 và điền kết quả thật vào `DEPLOYMENT.md`.
3. Nếu muốn chạy thêm test có key, tự điền `DEPLOY_API_KEY` trong `.env` local bằng khóa của agent trên cloud; không gửi khóa qua chat. Giữ `LOCAL_FALLBACK=false`.
4. Chạy:

```powershell
.\.venv\Scripts\python.exe -m pytest tests/test_cp5.py -v
```

5. Chụp dashboard và kết quả `/health`, lưu lần lượt vào `screenshots/dashboard.png` và `screenshots/health.png`; che secret nếu đang hiển thị.
6. Ghi platform, ngày deploy, URL, nguồn Redis và output thật trong `DEPLOYMENT.md`. Không đánh dấu biến môi trường đã set trước khi thực hiện.

Tài liệu chính thức: https://docs.railway.com/config-as-code/reference.
