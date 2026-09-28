# CP5 — Deploy trên Render

## Push cấu hình

```powershell
git add render.yaml railway.toml DEPLOYMENT.md CP5_SETUP.md
git commit -m "CP5: chuyển cấu hình deploy sang Render"
git push origin main
```

`railway.toml` đã được xóa; `git add` ghi nhận việc xóa file đó.

## Tạo Blueprint

1. Đăng nhập https://dashboard.render.com và kết nối GitHub.
2. Chọn **New > Blueprint**, chọn repo của bài và branch `main`.
3. Render đọc `render.yaml`, tạo web service `day12-agent` và Key Value `day12-redis` cùng vùng Singapore. Kiểm tra cả hai dùng plan Free.
4. Nhập `AGENT_API_KEY` khi dashboard yêu cầu. Có thể dùng khóa trong `.env` local. Không upload `.env` hoặc đưa khóa vào Git.
5. `REDIS_URL` được Blueprint gán tự động từ `connectionString` của `day12-redis`; không nhập tên biến, hostname đơn lẻ, `localhost` hay `fake://`.
6. Chờ deploy hoàn tất, lấy URL HTTPS của web service.

## Nếu tạo thủ công

1. **New > Key Value**: tên `day12-redis`, vùng Singapore, plan Free.
2. **New > Web Service**: chọn repo, branch `main`, runtime Docker, Dockerfile `./Dockerfile`, vùng Singapore, plan Free.
3. Trong Environment của web service, đặt `AGENT_API_KEY`; đặt `REDIS_URL` bằng **Internal Redis URL** của Key Value. URL phải bắt đầu bằng `redis://` hoặc `rediss://`; không đưa URL chứa mật khẩu vào repo/chat.
4. Thêm `RATE_LIMIT_PER_MINUTE=10`, `MONTHLY_BUDGET_USD=10.0`, `LOG_LEVEL=INFO`.
5. Đặt Health Check Path là `/health`, giữ Docker Command mặc định từ Dockerfile.

## Hoàn thiện CP5

- Mở `<URL>/health` và `<URL>/ready`: cả hai phải trả HTTP 200.
- Gửi URL công khai để kiểm tra và điền kết quả thật vào `DEPLOYMENT.md`.
- Tự điền `DEPLOY_API_KEY` vào `.env` local bằng khóa của service Render nếu muốn chạy test có xác thực. Giữ `LOCAL_FALLBACK=false`.
- Chạy `.\.venv\Scripts\python.exe -m pytest tests/test_cp5.py -v`.
- Chụp ảnh dashboard và `/health`, lưu vào `screenshots/dashboard.png` và `screenshots/health.png`; che secret nếu hiển thị.

## Xóa bản Railway cũ

Thay đổi repo không xóa tài nguyên Railway. Trong project Railway của bài, mở Settings của từng service `day12-agent` và Redis, chọn Delete Service và xác nhận. Redis bị xóa sẽ mất dữ liệu hội thoại và chi phí trên bản Railway đó. Nếu project chỉ chứa bài lab này, có thể xóa cả project từ Project Settings. Không xóa tài nguyên của ứng dụng khác.

Tài liệu: https://render.com/docs/blueprint-spec và https://render.com/docs/key-value.
