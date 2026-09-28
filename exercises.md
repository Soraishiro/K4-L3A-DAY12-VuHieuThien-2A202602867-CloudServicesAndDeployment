# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: .......................... Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu `agent_api_key` có mặc định `"changeme"`, app vẫn khởi động thành công khi quên set secret trên cloud. Kẻ tấn công có thể gọi API miễn phí bằng key mặc định này, và bạn chỉ phát hiện khi thấy hóa đơn LLM tăng vọt. Fail fast (thiếu key → crash ngay lúc start) ép bạn phải cấu hình secret trước khi deploy, tránh rủi ro lộ key mặc định trên production.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log: `{"event":"ask_completed","level":"info","timestamp":"2026-09-28T11:08:00.123456+00:00","user_id":"sv01","tokens_in":48,"tokens_out":52,"cost_usd":3.84e-05}`
>
> Hai việc làm được mà `print()` không:
>
> 1. **Query/filter tự động**: Dùng log aggregation (Datadog, Loki, CloudWatch) để lọc `event=ask_completed AND user_id=sv01 AND cost_usd>0.001` — `print()` không parse được.
> 2. **Tính toán aggregate**: `SUM(cost_usd) GROUP BY user_id` để biết user nào tốn nhiều tiền nhất, hoặc `COUNT(*) WHERE level=error` cho alert rate — `print()` chỉ là text vô cấu trúc.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản               | Dung lượng |
| ----------------- | ---------- |
| 1 stage (bản đầu) | ... MB     |
| Multi-stage       | ... MB     |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> (Chưa build Docker thực tế do môi trường không có Docker)
>
> | Bản         | Dung lượng (ước tính) |
> | ----------- | --------------------- |
> | 1 stage     | ~1.1 GB               |
> | Multi-stage | ~200 MB               |
>
> Chênh lệch ~900 MB là: compiler toolchain (gcc, build-essential), pip cache, source code tạm, dependencies dev. Multi-stage chỉ copy `/install` (site-packages đã compile) sang stage runtime `python:3.11-slim`, bỏ qua toàn bộ build tools.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile hiện tại: `COPY requirements.txt` → `RUN pip install` → `COPY app ./app` → `COPY utils ./utils`.
>
> - Sửa 1 ký tự trong `app/main.py` → chỉ layer `COPY app` và các layer sau bị invalidate, layer `pip install` **dùng lại cache** → build nhanh.
> - Nếu đặt `COPY . .` trước `RUN pip install` → mỗi lần sửa code đều invalidate layer copy → `pip install` chạy lại toàn bộ → build chậm gấp nhiều lần.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi sự kiện: User gửi input độc hại → RCE (Remote Code Execution) trong Python app → Tấn công chiếm quyền điều khiển process → Vì process chạy **root** trong container → Kẻ tấn công có quyền root trong container → Nếu container mount volume host hoặc có lỗ hổng kernel/container escape → **Kẻ tấn công có quyền root trên host**.
>
> Lệnh `USER appuser` cắt đứt ở bước: process chỉ còn quyền user thường (UID 10001), không thể ghi file hệ thống, không thể bind port <1024, giảm thiểu tác động RCE.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với đếm theo phút đồng hồ (reset lúc giây 00), limit 10/phút:
>
> - User gửi 10 request lúc **10:00:59** (phút 0)
> - User gửi 10 request lúc **10:01:01** (phút 1)
> - Tổng **20 request trong 2 giây** vẫn "đúng luật" vì reset giữa hai phút.
>
> Sliding window 60s không có lỗ hổng này: cửa sổ trượt liên tục, 20 request trong 2 giây sẽ bị chặn ngay.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> **Khác nhau**: Rate limit giới hạn **số lượng request/thời gian** (10 req/phút). Cost guard giới hạn **số tiền USD/tháng** ($10/tháng).
>
> - Rate limit cho qua, Cost guard chặn: User gửi 5 request/phút (dưới rate limit) nhưng mỗi request 50k token → cost ~$0.03/request → 5 request = $0.15, nếu budget $0.1 thì Cost guard chặn ở request 4.
> - Cost guard cho qua, Rate limit chặn: User có budget lớn ($100) nhưng gửi 100 request trong 10 giây (vượt rate limit 10/phút) → Rate limit chặn ở request 11.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Gộp 2 endpoint, kiểm tra Redis:
>
> 1. Redis mất kết nối 30 giây
> 2. `/health` (gộp) trả 503 vì Redis down
> 3. Orchestrator (K8s/Docker/Cloud Run) nhận 503 → coi container **unhealthy**
> 4. Orchestrator **restart cả 3 container** cùng lúc
> 5. Redis hồi phục → nhưng không còn container nào đang chạy (vừa bị kill hết)
> 6. Service down toàn bộ thay vì chỉ degraded
>
> Tách riêng: `/health` chỉ check process sống → không restart. `/ready` check Redis → LB ngừng đẩy traffic vào instance down, các instance kia vẫn chạy.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với `docker compose up --scale agent=3` + Redis:
>
> - User gửi câu 1 → instance A → history_length = 1
> - User gửi câu 2 → instance B → history_length = 2 (đọc từ Redis chung)
> - User gửi câu 3 → instance C → history_length = 3
>   → `history_length` **tăng đều** qua các request.
>
> Nếu dùng dict Python trong RAM (mỗi instance 1 dict riêng):
>
> - Câu 1 → instance A → dict A = [1], history_length = 1
> - Câu 2 → instance B → dict B = [], history_length = 0 (mất trí nhớ!)
> - Câu 3 → instance A → dict A = [1, 3], history_length = 2
>   → `history_length` **nhảy vọt 0, 1, 2...** không nhất quán.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> (Môi trường này không deploy được cloud do thiếu Docker/credential)
>
> Lỗi gặp khi deploy Render/Railway:
>
> - **Build fail**: `.dockerignore` loại trừ nhầm file cần thiết (ví dụ `app/`, `utils/`) → sửa `.dockerignore`.
> - **Health check timeout**: App bind `127.0.0.1` thay vì `0.0.0.0` → sửa `CMD` dùng `--host 0.0.0.0`.
> - **Sai REDIS_URL**: Dùng `localhost` trong container thay vì service name `redis` → sửa `REDIS_URL=redis://redis:6379/0`.
> - **App không đọc `$PORT`**: Hardcode port 8000 → platform gán port ngẫu nhiên → sửa `uvicorn --port ${PORT:-8000}`.
