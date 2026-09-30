# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng chờ dưới mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Ví dụ mình nghĩ tới là lúc deploy quên nhập AGENT_API_KEY. Nếu app tự dùng changeme thì service vẫn có thể khởi động và mình dễ tưởng là ổn, trong khi ai biết khóa mặc định đều có thể gọi API. Không có giá trị mặc định thì app dừng ngay, mình nhận ra thiếu secret trước khi mở service cho người dùng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Sau khi gọi `/ask` thành công, mình nhận được dòng log: `{"event":"ask_completed","level":"info","timestamp":"2026-09-28T16:27:10.327581+00:00","user_id":"sv401","tokens_in":5,"tokens_out":37,"cost_usd":2.295e-05}`. Vì các thông tin nằm ở từng trường riêng, mình có thể lọc các lần gọi theo `user_id` hoặc thời gian, và cộng `cost_usd` hay token để theo dõi mức sử dụng. Một dòng `print("đã trả lời xong")` chỉ báo là xong, không có các trường để lọc và tổng hợp như vậy.

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
| 1 stage (bản đầu) | Chưa đo được — build dừng ở lỗi TLS handshake khi tải base image |
| Multi-stage | 82.4 MB content size; Docker Desktop báo 354 MB disk usage |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Lần build image một stage của mình không hoàn tất vì Docker bị TLS handshake timeout khi tải base image, nên mình chưa có số đo để so sánh công bằng. Với bản multi-stage, Docker Desktop hiển thị content size 82.4 MB và disk usage 354 MB. Cách tách stage giúp image chạy cuối không mang theo toàn bộ môi trường và layer chỉ cần lúc build; khi lấy được số đo một stage mình sẽ bổ sung mức chênh lệch cụ thể.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Trong Dockerfile hiện tại, requirements được copy và cài trước source code. Vì vậy nếu chỉ sửa app/main.py thì Docker có thể dùng lại layer cài thư viện từ cache; layer copy source và các bước sau nó sẽ chạy lại. Nếu copy toàn bộ source trước pip install, thay đổi một file Python cũng làm layer copy đổi, khiến pip install phải chạy lại dù requirements không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu code có lỗ hổng cho phép chạy lệnh tùy ý, kẻ tấn công có thể chiếm quyền của tiến trình trong container. Chạy root khiến họ có quyền cao bên trong container, đọc hoặc sửa được nhiều file hơn và hậu quả sẽ nặng hơn nếu container còn mount nhầm thư mục hay được cấp quyền quá rộng. USER appuser hạ quyền của tiến trình, nên giảm phạm vi thiệt hại; nó không phải bảo đảm tuyệt đối rằng container không thể bị thoát ra.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Với giới hạn 10 request mỗi phút, bộ đếm theo phút đồng hồ có thể cho qua 20 request trong khoảng hai giây: gửi 10 request ngay trước mốc phút đổi, rồi thêm 10 request ngay sau mốc đó. Mỗi nhóm nằm ở một phút khác nhau nên bộ đếm reset giữa hai nhóm. Sliding window nhìn lại đủ 60 giây sẽ thấy cả 20 request và chặn từ request vượt hạn mức.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit nhìn vào số lần gọi trong một khoảng thời gian, còn cost guard nhìn vào tổng tiền đã dùng trong tháng. Một người gửi ít request nhưng đã tiêu hết ngân sách vẫn có thể bị cost guard chặn dù chưa chạm rate limit. Ngược lại, nhiều request rất rẻ gửi dồn trong một phút có thể bị rate limit chặn dù ngân sách tháng còn nhiều.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Nếu dùng cùng một endpoint làm liveness và kiểm tra Redis, Redis mất kết nối sẽ làm cả ba instance báo unhealthy dù process vẫn còn chạy. Nếu bộ điều phối dùng kết quả đó để restart, nó có thể khởi động lại cả ba; Redis vẫn đang mất nên các instance lại tiếp tục lỗi, làm dịch vụ gián đoạn rộng hơn. Tách probe sẽ tránh việc đó: /health chỉ xác nhận process còn sống, còn /ready báo chưa sẵn sàng để load balancer tạm ngừng gửi traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Trong test CP4, hai lần gọi `/ask` liên tiếp với cùng user được kiểm tra: lần đầu có `history_length` bằng 0, lần sau bằng 2 vì lịch sử đã có một tin nhắn user và một tin nhắn assistant. Redis giữ lịch sử dùng chung nên request nào vào instance nào cũng đọc được cùng dữ liệu. Nếu thay Redis bằng dict trong Python, mỗi instance chỉ giữ dict của riêng process đó; khi load balancer chuyển request sang instance khác, `history_length` có thể thấp hơn hoặc quay về 0, còn nếu request cứ vào một instance thì số này vẫn tăng theo lịch sử ở instance đó. Mình chưa có kết quả đo thực tế khi chạy ba replica; phần này là suy luận từ test CP4 và cách lưu state trong process.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Khi mở URL gốc của service trên Render, mình thấy {"detail":"Not Found"} và ban đầu tưởng deploy bị lỗi. Trang Deploys lại báo Live, nên mình thử đúng endpoint /health và nhận 200 cùng status ok. Nguyên nhân là app không khai báo route cho đường dẫn /; cách kiểm tra đúng là dùng /health để xác nhận service và /ready để xác nhận Redis.
