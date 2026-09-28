# Thông Tin Deploy — Checkpoint 5

Trạng thái hiện tại: Đã deploy thành công lên Render. Public URL hoạt động, Redis kết nối tốt.

## Thông Tin Học Viên

| Mục         | Nội dung                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------- |
| Họ và tên   | **Vũ Hiệu Thiên**                                                                                 |
| Mã học viên | **2A202602867**                                                                                   |
| Repo        | **https://github.com/Soraishiro/K4-L3A-DAY12-VuHieuThien-2A202602867-CloudServicesAndDeployment** |

## Service

| Mục         | Nội dung                                  |
| ----------- | ----------------------------------------- |
| Public URL  | **https://day12-agent-xra5.onrender.com** |
| Platform    | **Render**                                |
| Ngày deploy | **2026-09-28**                            |

## Biến Môi Trường Cần Set Trên Cloud

Chỉ ghi tên biến, không ghi giá trị secret.

| Biến                    | Trạng thái                          | Ghi chú                            |
| ----------------------- | ----------------------------------- | ---------------------------------- |
| `PORT`                  | platform tự gán                     | Không hardcode trên dashboard      |
| `AGENT_API_KEY`         | ✅ đã set                           | Secret, nhập trên Render dashboard |
| `REDIS_URL`             | ✅ render.yaml nối từ Redis service | Không dùng localhost               |
| `RATE_LIMIT_PER_MINUTE` | ✅ cấu hình sẵn                     | 10                                 |
| `MONTHLY_BUDGET_USD`    | ✅ cấu hình sẵn                     | 10.0                               |
| `LOG_LEVEL`             | ✅ cấu hình sẵn                     | INFO                               |

## Kiểm Tra Deploy Thật

```text
GET  /health → 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready  → 200 {"status":"ready","redis":true}
POST /ask không X-API-Key → 401
POST /ask có X-API-Key → 200, có answer/user_id/history_length/cost_usd/tokens
```

## Lệnh Cần Chạy Sau Khi Deploy

```bash
URL=https://day12-agent-xra5.onrender.com

curl.exe -i "$URL/health"
curl.exe -i "$URL/ready"
curl.exe -i -X POST "$URL/ask" -H "Content-Type: application/json" -d '{"question":"Hello"}'
curl.exe -i -X POST "$URL/ask" -H "Content-Type: application/json" -H "X-API-Key: $AGENT_API_KEY" -H "X-User-Id: sv01" -d '{"question":"Deploy là gì?"}'
```

## Kết Quả Chạy Thật

```text
/health: 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
/ready: 200 {"status":"ready","redis":true}
/ask no key: 401 {"detail":"invalid or missing API key"}
/ask with key: 200 {"answer":"...","user_id":"sv01","history_length":2,"cost_usd":3.84e-05,"tokens":{"in":48,"out":52}}
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — Render dashboard (web service + Redis, trạng thái Live)
- `screenshots/health.png` — Terminal output 4 lệnh curl trên
