# Thông Tin Deploy — Checkpoint 5

> File này được CP5 dùng để tìm địa chỉ service công khai.
>
> **Chỉ ghi tên biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Vu Duy Diep |
| Mã học viên | 2A202602703 |
| Repo | https://github.com/VuDuyDiepAI/K4-L3A-DAY12-VuDuyDiep-2A202602703-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-pshm.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và nguồn giá trị, không ghi giá trị secret:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| PORT | ✅ | Render tự gán |
| AGENT_API_KEY | ✅ | Đặt trong Render Dashboard; không lưu trong repo |
| REDIS_URL | ✅ | Render Key Value day12-redis; /ready xác nhận Redis đã kết nối |
| RATE_LIMIT_PER_MINUTE | ✅ | 10 |
| MONTHLY_BUDGET_USD | ✅ | 10.0 |
| LOG_LEVEL | ✅ | INFO |

## Lệnh Kiểm Tra

Dùng Public URL ở trên:

~~~bash
# 1. Liveness — mong đợi 200
curl -i https://day12-agent-pshm.onrender.com/health

# 2. Readiness — mong đợi 200 khi đã nối được Redis
curl -i https://day12-agent-pshm.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-pshm.onrender.com/ask -H "Content-Type: application/json" -d '{"question":"Hello"}'

# 4. Có API key — chạy trên máy cá nhân sau khi đặt AGENT_API_KEY trong môi trường
curl -i -X POST https://day12-agent-pshm.onrender.com/ask -H "Content-Type: application/json" -H "X-API-Key: $AGENT_API_KEY" -H "X-User-Id: sv-test" -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần bằng API key; các lần vượt hạn mức phải trả 429
for i in $(seq 1 15); do curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-pshm.onrender.com/ask -H "Content-Type: application/json" -H "X-API-Key: $AGENT_API_KEY" -H "X-User-Id: sv-test" -d '{"question":"test"}'; done; echo
~~~

## Kết Quả Chạy Thật

Các kết quả bên dưới được kiểm tra trực tiếp trên service Render ngày 2026-09-28:

~~~text
GET /health -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready -> 200 {"status":"ready","redis":true}
POST /ask (không gửi API key) -> 401 {"detail":"invalid or missing API key"}
POST /ask (có API key) -> chưa kiểm tra trong phiên này; không ghi secret vào tài liệu.
Rate limit 15 lần -> chưa kiểm tra trong phiên này; cần API key lưu riêng trên máy.
~~~

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục screenshots/:

- screenshots/dashboard.png — trang quản lý service trên Render
- screenshots/health.png — kết quả gọi /health từ trình duyệt hoặc curl
