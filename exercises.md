# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay các dòng trả lời mẫu bên dưới mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Lê Thị Linh Mã học viên: 2A202601441

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu quên đặt `AGENT_API_KEY`, ứng dụng sẽ không khởi động được ngay. Nhờ vậy lỗi cấu hình được phát hiện trước khi nhận traffic, thay vì chạy với khóa mặc định `changeme` và vô tình để người khác gọi API.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> log JSON: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T04:18:37+00:00","user_id":"cp5-test","tokens_in":5,"tokens_out":44,"cost_usd":0.00002715}`. Dòng log này cho phép lọc request theo `user_id`, đồng thời thống kê token và chi phí. Với `print()` thường, dữ liệu không có field cấu trúc để hệ thống log tự động tìm kiếm, lập dashboard hoặc cảnh báo.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f Dockerfile-1-stage -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản               | Dung lượng |
| ----------------- | ---------- |
| 1 stage (bản đầu) | 286 MB     |
| Multi-stage       | 270 MB     |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Multi-stage chỉ copy dependency cần thiết từ stage builder sang runtime, còn 1-stage cài đặt nằm trực tiếp trong runtime image

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, Docker dùng lại layer `FROM`, `COPY requirements.txt` và `RUN pip install`; chỉ các layer `COPY app/` sau đó phải build lại. Nếu đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source sẽ làm layer copy thay đổi và Docker phải cài lại toàn bộ dependency, khiến build lâu hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app có lỗ hổng cho phép thực thi lệnh, kẻ tấn công có thể chạy lệnh với quyền của process trong container. Nếu process chạy bằng root thì tác động sẽ nghiêm trọng hơn, đặc biệt khi có cấu hình hoặc mount không an toàn.`USER appuser` giới hạn process ở user thường, giảm quyền mà kẻ tấn công có thể sử dụng sau khi khai thác lỗi.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi tối đa 20 request trong khoảng 2 giây: 10 request lúc 10:00:59 và thêm 10 request lúc 10:01:00. Hai nhóm nằm ở hai phút đồng hồ khác nhau nên bộ đếm reset theo phút cho qua, dù thực tế có 20 request gần như liên tiếp.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ/số request để bảo vệ hạ tầng; cost guard giới hạn tổng chi phí user được phép tiêu trong tháng để bảo vệ ngân sách. Một user gửi mỗi phút nhưng mỗi request xử lý tài liệu rất lớn có thể qua rate limit nhưng bị cost guard chặn. Ngược lại, nhiều request nhỏ có thể chưa tốn nhiều tiền nhưng vẫn bị rate limit chặn vì gửi quá nhanh.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> `/health` chỉ kiểm tra process và không gọi Redis, còn `/ready` kiểm tra Redis. Nếu Redis mất kết nối 30 giây, `/ready` trả 503 để load balancer ngừng gửi request mới; `/health` vẫn có thể trả 200 vì process còn sống. Nếu gộp hai endpoint và health cũng phụ thuộc Redis, cả ba container có thể bị đánh dấu unhealthy và bị restart hàng loạt dù lỗi Redis chỉ là tạm thời.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis, request rơi vào container nào cũng đọc được history chung. Vì mỗi request lưu một message user và một message assistant, `history_length` thường tăng `0, 2, 4, 6, 8`. Nếu dùng dict Python, mỗi replica có bộ nhớ riêng nên history bị reset hoặc tăng không đều khi request chuyển container, và mất khi container restart.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Một lỗi khi deploy Railway là Uvicorn báo `Invalid value for '--port': '$PORT' is not a valid integer`. Mình xem deployment log và thấy Railway chạy `startCommand` trực tiếp nên không expand biến `$PORT`. Mình bỏ `startCommand` trong `railway.toml` để dùng `CMD` dạng `sh -c` trong Dockerfile, nơi `${PORT:-8000}` được expand đúng. Sau đó deployment online, `/health` và `/ready` đều trả 200.
