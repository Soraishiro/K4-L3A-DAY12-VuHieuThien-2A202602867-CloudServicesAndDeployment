# Thong Tin Deploy - Checkpoint 5

Trang thai hien tai: code va cau hinh Render da san sang, nhung moi truong thuc thi nay khong co tai khoan cloud/GitHub cua hoc vien de tao public service. Khong co secret nao duoc ghi vao file nay.

## Thong Tin Hoc Vien

| Muc         | Noi dung                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------- |
| Ho va ten   | **Vũ Hieu Thiên**                                                                                 |
| Mã học viên | **2A202602867**                                                                                   |
| Repo        | **https://github.com/Soraishiro/K4-L3A-DAY12-VuHieuThien-2A202602867-CloudServicesAndDeployment** |

## Service

| Muc         | Noi dung                                                       |
| ----------- | -------------------------------------------------------------- |
| Public URL  | **https://<tên-service>.onrender.com** (sau khi deploy Render) |
| Platform    | **Render**                                                     |
| Ngay deploy | **2026-09-28**                                                 |

## Bien Moi Truong Can Set Tren Cloud

Chi ghi ten bien, khong ghi gia tri secret.

| Bien                    | Trang thai                       | Ghi chu                            |
| ----------------------- | -------------------------------- | ---------------------------------- |
| `PORT`                  | platform tu gan                  | Khong hardcode tren dashboard      |
| `AGENT_API_KEY`         | can set                          | Secret, nhap tren Render dashboard |
| `REDIS_URL`             | render.yaml noi tu Redis service | Khong dung localhost               |
| `RATE_LIMIT_PER_MINUTE` | cau hinh san                     | 10                                 |
| `MONTHLY_BUDGET_USD`    | cau hinh san                     | 10.0                               |
| `LOG_LEVEL`             | cau hinh san                     | INFO                               |

## Kiem Tra Local Da Chay

Moi truong sandbox khong co Docker, nen day la sanity check bang Uvicorn va `REDIS_URL=fake://`, khong duoc xem la bang chung CP5 cloud.

```text
GET  /health -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready  -> 200 {"status":"ready","redis":true}
POST /ask khong X-API-Key -> 401
POST /ask co X-API-Key -> 200, co answer/user_id/history_length/cost_usd/tokens
```

## Lenh Can Chay Sau Khi Deploy

```bash
URL=https://<public-url-that>

curl -i "$URL/health"
curl -i "$URL/ready"
curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy la gi?"}'
```

Sau khi deploy that, thay `CHUA_CUNG_CAP` va `CHUA_DEPLOY`, dan output that, roi them `screenshots/dashboard.png` va `screenshots/health.png`.
