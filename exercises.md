# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Thị Chinh  Mã học viên: 2A202602876

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định `"changeme"`, bất kỳ ai sử dụng key `"changeme"` đều có thể gọi vào endpoint `/ask`, khiến tiêu hao tiền api và tài nguyên của cloud. Còn khi `agent_api_key` không có giá trị mặc định thì người dùng sẽ nhận lỗi 401, ứng dụng dừng ngay lập tức. Điều này giúp người dùng phát hiện lỗi và sửa ngay từ đầu, thay vì phải chịu tốn kém chi phí trong âm thầm.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T16:54:58.856930+00:00", "user_id": "sv01", "tokens_in": 233, "tokens_out": 46, "cost_usd": 6.255e-05}
> 
> Hai việc làm được với log JSON mà print() không làm được:
> 1. Lưu trữ và truy xuất có cấu trúc: Log JSON có thể được lưu vào file, cơ sở dữ liệu hoặc hệ thống quản lý log (như Splunk, ELK) để dễ dàng tìm kiếm, lọc và phân tích sau này. Trong khi đó, chuỗi văn bản từ print() khó phân tích và truy xuất thông tin chi tiết
> 2. Tích hợp với hệ thống monitoring: Log JSON có thể được thu thập bởi các công cụ monitoring để theo dõi hiệu suất, phát hiện bất thường và kích hoạt cảnh báo tự động.

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
| 1 stage (bản đầu) | 1.67 GB |
| Multi-stage | 297 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch (~1.3 GB) là do image build 1 stage chứa tất cả các dependency của Python (bao gồm cả compiler, tool, và các thư viện dùng cho việc build), còn image multi-stage chỉ copy đúng các thư viện đã cài đặt sang stage runtime và code cần thiết để chạy, loại bỏ compiler và tool không cần thiết.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa một ký tự trong `app/main.py`: layer `COPY requirement.txt` và `RUN pip install` được dùng lại từ cache vì dile requirements không đổi, không phải cài lại thư viện. Còn layer `COPY app ./app` và các bước phía sau nó sẽ phải build lại từ đầu. 
> Còn nếu đặt `COPY . .` lên trước `RUN pip install`, vì `app/main.py` bị sửa, layer `COPY . .` sẽ bị mất cache, kéo theo tất cả các layer phía sau nó (bao gồm `RUN pip install`) sẽ phải build lại từ đầu, mất thời gian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu image được build với USER root:
> 1. Kẻ tấn công tìm thấy lỗ hổng trong code, chiếm quyền điều khiển. 
> 2. Do container chạy dưới quyền root, kẻ tấn công lập tức có quyền root trong container.
> 3. Từ root trong container, kẻ tấn công có thể tấn công máy chủ host, sửa file hệ điều hành, cài đặt phần mềm độc hại hoặc đánh cắp dữ liệu.

> Lệnh `USER appuser` cắt đứt chuỗi ở bước 2: khi kẻ tấn công xâm nhập vào container, họ chỉ có quyền của một user thông thường, không thể sửa file hệ điều hành, không có đủ đặc quyền để thực hiện các cuộc tấn công.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 requests trong 2 giây liên tiếp khi hạn mức là 10 requests/phút. Cách đạt được là: Gửi 10 request ở giây `00:59`. Khi đồng hồ nhảy sang giây `01:00`, tất cả các request đó đều đã nằm ngoài cửa sổ 60 giây, nên người dùng vẫn được gửi thêm 10 request nữa.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số lượng request trên một khoảng thời gian nhất định, trong khi cost guard giới hạn số tiền chi tiêu trong một tháng. 
> 
> Tình huống rate limit cho qua nhưng cost guard phải chặn: một user gửi đúng 1 request trong ngày nhưng tổng chi phí tích luỹ trong tháng của user đó đã vượt quá mức quy định, lúc này cost guard sẽ chặn request đó lại. 
> 
> Tình huống ngược lại: Tài khoản user còn nguyên ngân sách, nhưng user spam liên tục 10 request trong 10 giây, khi đó rate limit chặn hết lại trong khi chi phí tăng thêm không đáng kể so với ngân sách.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> 1. Redis mất kết nối, container 1 (agent-0) fail check health/ready, status chuyển sang red.
> 2. Hệ thống Orchestrator gọi định kỳ vào endpoint health check của cả 3 container agent. Do Redis chết, cả 3 container đều trả về lỗi 503.
> 3. Vì đây là liveness probe, Orchestrator mặc định rằng tiến trình của container đã bị treo/hỏng và ra quyết định restart đồng loạt cả 3 container agent.
> 4. 3 container bị khởi động lại, nhưng chúng vẫn không thể kết nối được tới Redis (vì Redis vẫn đang chết).
> 5. Lặp lại chu trình: fail → restart → fail → restart, dẫn đến tình trạng crash loop và không có container nào hoạt động bình thường trong suốt 30 giây.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Quan sát thấy `history_length` tăng đều đặn theo mỗi câu hỏi. Nếu lịch sử được lưu trong một dict Python thay vì Redis, con số `history_length` sẽ bị nhảy loạn xạ và ngắt quãng, bởi vì mỗi lần scale lên 3 instances, mỗi instance sẽ có một dict riêng và không chia sẻ dữ liệu với nhau, dẫn đến việc lịch sử hội thoại bị mất hoặc đè lên nhau.


---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Thông báo lỗi: `"Free plan resource provision limit exceeded. Please upgrade to provision more resources!"` Khi chạy lệnh `railway init` trên Terminal, CLI thông báo tài khoản của tôi đã vượt quá hạn mức tài nguyên của gói Free, mặc dù vẫn còn nguyên credit và hạn sử dụng. Khi đăng nhập vào dashboard của Railway kiểm tra, tôi thấy đã có sẵn 2 projects, tôi xử lý bằng cách xoá hết projects cũ không dùng đến, sau đó chạy lại lệnh `railway init` để tạo project mới thành công. 