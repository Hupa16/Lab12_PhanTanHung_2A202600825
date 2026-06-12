# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found in develop/app.py
1. **Hardcoded Secrets**: API Key (`OPENAI_API_KEY`) và Database credentials (`DATABASE_URL`) bị ghi cứng trực tiếp vào mã nguồn, dễ bị rò rỉ khi đẩy lên các kho lưu trữ công khai (GitHub).
2. **Thiếu Quản lý Cấu hình (Configuration Management)**: Các biến cấu hình quan trọng (`DEBUG`, `MAX_TOKENS`) bị định nghĩa tĩnh, không linh hoạt thay đổi theo môi trường (dev, staging, production) mà không phải sửa code.
3. **Sử dụng `print()` thay vì Structured Logging**: Ghi nhận log thô sơ bằng `print()`, in cả mã bảo mật ra log, và định dạng text thô khiến các hệ thống thu thập log tập trung (Datadog, Loki...) không thể phân tích và lọc hiệu quả.
4. **Không có Health Check Endpoints**: Thiếu các endpoint `/health` (Liveness) và `/ready` (Readiness) để nền tảng điều phối (Railway, Render, K8s) kiểm tra tình trạng sống/chết của ứng dụng và tự động khởi động lại nếu xảy ra sự cố.
5. **Ghi cứng Host và Port**: Thiết lập host là `localhost` và port cố định `8000`. Khi deploy lên Cloud, ứng dụng cần bind vào `0.0.0.0` để tiếp nhận kết nối bên ngoài và đọc `PORT` do nền tảng cấp phát động thông qua biến môi trường.
6. **Bật chế độ Debug/Reload mặc định**: Thiết lập `reload=True` khi chạy uvicorn mà không có điều kiện kiểm tra môi trường, gây tốn tài nguyên và tăng rủi ro bảo mật trong môi trường production.

### Exercise 1.3: Comparison table
| Feature | Develop (Basic) | Production (Advanced) | Why Important? |
|---------|---------|------------|----------------|
| **Config** | Hardcode tĩnh | Load động qua env vars (`pydantic-settings`) | Cho phép thay đổi cấu hình (port, credentials, api key) theo môi trường mà không cần sửa code hay rebuild image. |
| **Health Check** | Không có | Có endpoint `/health` và `/ready` | Giúp hệ thống container orchestration kiểm tra trạng thái liveness (khởi động lại nếu lỗi) và readiness (chỉ route traffic khi sẵn sàng). |
| **Logging** | Dùng `print()` text thô | Dùng JSON Structured Logging | Giúp dễ dàng thu gom, tìm kiếm và phân tích log tự động trên production; tránh ghi đè các secrets vào log. |
| **Shutdown** | Tắt đột ngột (Abrupt exit) | Xử lý SIGTERM để Graceful Shutdown | Đảm bảo hoàn thành các request đang xử lý dở dang và giải phóng/đóng kết nối cơ sở dữ liệu/Redis an toàn trước khi tắt ứng dụng. |

---

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image**: Ở bản develop là `python:3.11` (full distribution, dung lượng khoảng 1GB). Ở bản production sử dụng `python:3.11-slim` (chỉ chứa các gói runtime tối giản cần thiết).
2. **Working directory**: Thiết lập là `/app` làm thư mục làm việc chính trong container.
3. **Tại sao COPY requirements.txt trước?**: Để tận dụng cơ chế Docker layer caching. Nếu file `requirements.txt` không thay đổi, Docker sẽ lấy kết quả cài dependencies từ cache ở lần build trước mà không cần chạy lại `pip install`, giúp tối ưu hóa thời gian build.
4. **CMD vs ENTRYPOINT**: 
   - `CMD` định nghĩa lệnh mặc định và có thể bị ghi đè hoàn toàn nếu truyền tham số vào cuối lệnh `docker run`.
   - `ENTRYPOINT` định nghĩa lệnh bắt buộc chạy, không thể bị ghi đè trực tiếp mà chỉ có thể nhận thêm tham số truyền vào từ `docker run`. Khi dùng chung, `CMD` đóng vai trò là danh sách tham số mặc định cho `ENTRYPOINT`.

### Exercise 2.3: Image size comparison
- **Develop (Basic)**: ~ 1025 MB
- **Production (Multi-stage)**: ~ 235 MB
- **Difference**: Tiết kiệm ~ 77% dung lượng đĩa và tối ưu thời gian deploy lên cloud.

---

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- **URL**: https://your-agent.railway.app (Sẽ được điền chính xác sau khi deploy ở Part 6)
- **Screenshot**: [Screenshots Dashboard](screenshots/dashboard.png)

---

## Part 4: API Security

### Exercise 4.1-4.3: Test results
- **Gọi API không truyền API Key**:
  `HTTP 401 Unauthorized` - `{"detail":"Invalid or missing API key. Include header: X-API-Key: <key>"}`
- **Gọi API với API Key đúng**:
  `HTTP 200 OK` - `{"question":"Hello","answer":"[Mock LLM Response] Hello"}`
- **Test Rate Limiting (gọi dồn dập > 10 req/min)**:
  `HTTP 429 Too Many Requests` - `{"detail":"Rate limit exceeded: 10 req/min"}`

### Exercise 4.4: Cost guard implementation
- **Cách tiếp cận**: 
  - Tính toán số lượng tokens từ câu hỏi đầu vào (input tokens ≈ số từ * 2) và câu trả lời đầu ra (output tokens).
  - Quy đổi ra chi phí USD tương ứng với biểu giá của mô hình (GPT-4o-mini).
  - Lưu và cộng dồn chi phí đã tiêu thụ của người dùng trong ngày vào hệ thống lưu trữ.
  - Chặn cuộc gọi tiếp theo và trả về `402 Payment Required` (hoặc `503` tùy thiết lập) nếu tổng chi phí vượt quá giới hạn ngân sách hàng ngày (`daily_budget_usd`).

---

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
- **Liveness & Readiness**:
  - Endpoint `/health` kiểm tra xem process của app còn sống không.
  - Endpoint `/ready` ping thử tới Redis để đảm bảo dịch vụ lưu trữ phụ trợ (backing service) sẵn sàng nhận traffic. Nếu mất kết nối với Redis, trả về `503 Service Unavailable`.
- **Graceful Shutdown**:
  - Lắng nghe tín hiệu `SIGTERM` từ hệ thống điều phối container.
  - Khi nhận tín hiệu, set `is_ready = False` để dừng tiếp nhận request mới và chờ 30 giây để hoàn thành các request hiện tại trước khi thoát.
- **Stateless Design**:
  - Không lưu trữ dữ liệu lịch sử chat hay tokens của người dùng trong RAM của server (in-memory dict).
  - Chuyển toàn bộ dữ liệu lưu trữ sang **Redis** sử dụng các key dạng: `session:{session_id}` cho lịch sử chat, `rate_limit:{user_id}` cho rate limit và `cost:{user_id}:{date}` cho Cost Guard.
  - Nhờ đó, ta có thể chạy đồng thời nhiều instance của Agent đằng sau Load Balancer (Nginx) mà không lo mất đồng bộ dữ liệu.
