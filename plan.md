# ĐẶC TẢ YÊU CẦU POC: HỆ THỐNG GIÁM SÁT THI CỬ NGẦM (REAL-TIME OCR)

## 1. MỤC TIÊU CỦA BẢN POC NÀY
- Dựng luồng truyền tải dữ liệu WebSocket thời gian thực (Real-time).
- Tích hợp Tesseract.js tại Server. Tối ưu hiệu năng: Báo động ngay lập tức khi có sự kiện, quét OCR trên luồng ảnh stream tuần tự.
- Chỉ dùng HTML thô (Raw HTML) và Vanilla JS.

## 2. KIẾN TRÚC LOGIC VÀ CẤU HÌNH ĐỘNG
- Payload Cấu hình:
  {"action": "UPDATE_CONFIG", "mode": "PRACTICAL", "blacklist": ["chatgpt", "zalo", "gemini", "messenger"]}

- Server Rule Engine (Tesseract.js - Ngôn ngữ 'eng'):
  - Chuẩn hóa text OCR: `text = text.toLowerCase().replace(/\s+/g, '')`.
  - **CHẾ ĐỘ TRẮC NGHIỆM (THEORY):** Mọi event đều bị gán `alert_level = 'HIGH'`. KHÔNG cần chạy OCR.
  - **CHẾ ĐỘ THỰC HÀNH (PRACTICAL):** - **Nhóm Event `BLUR`, `VISIBILITY_CHANGE`:** Chạy OCR quét 1 tấm ảnh đính kèm. Dính Blacklist -> `HIGH`. Không dính -> `LOW`.
    - **Nhóm Event `COPY` (Xử lý Real-time Streaming):**
      - Khi nhận `COPY_DETECTED`: Server lập tức broadcast cờ `MEDIUM` cho Admin (Báo hiệu: "Thí sinh vừa copy, đang theo dõi 10s...").
      - Khi nhận chuỗi `COPY_FRAME` (5 ảnh gửi tuần tự lên trong 10s): 
        - Server chạy OCR từng ảnh một. 
        - Nếu 1 ảnh phát hiện Blacklist -> Đánh cờ `HIGH` gửi Admin, đồng thời **đánh dấu phiên COPY này đã vi phạm**. Các ảnh `COPY_FRAME` tiếp theo của phiên này KHÔNG CẦN chạy OCR nữa (để tiết kiệm CPU), chỉ cần forward thẳng ảnh xuống Admin làm bằng chứng.

## 3. CẤU TRÚC DỮ LIỆU GIAO TIẾP
- **Client gửi Server:**
  - *Khi vừa bôi đen copy:* `{"event_type": "COPY_DETECTED", "copied_text": "..."}`
  - *Mỗi 2 giây sau đó (Lặp 5 lần):* `{"event_type": "COPY_FRAME", "image": "base64..."}`
  - *Các sự kiện khác:* `{"event_type": "BLUR", "image": "base64..."}`

- **Server gửi Admin:**
  - `{"exam_mode": "PRACTICAL", "event_type": "COPY_DETECTED", "alert_level": "MEDIUM", "reason": "COPY_ATTEMPT"}`
  - `{"exam_mode": "PRACTICAL", "event_type": "COPY_FRAME", "alert_level": "HIGH", "matched_keyword": "chatgpt", "image": "base64..."}`

## 4. YÊU CẦU TRIỂN KHAI CHO AI AGENT (NODE.JS + WS + TESSERACT.JS)
@Agent: Khởi tạo project Node.js (`npm i express ws tesseract.js`). Tạo 3 file sau:

### File 1: server.js (Core Engine)
- Dùng `express` serve HTML cổng 3000. Dùng `ws` mở WebSocket cổng 8080.
- State quản lý phiên COPY: `let isCurrentCopySessionHigh = false;`
- Hàm `analyzeImage(imageBase64, blacklist)`: Dùng `Tesseract.recognize(img, 'eng')`, xử lý text và so sánh.
- Lắng nghe WS:
  - Nếu event = `COPY_DETECTED`: Reset `isCurrentCopySessionHigh = false`. Broadcast `MEDIUM` cho Admin.
  - Nếu event = `COPY_FRAME`: 
    - Nếu `isCurrentCopySessionHigh === true`: Bỏ qua OCR, forward thẳng ảnh kèm `alert_level: 'HIGH'` cho Admin (bằng chứng bổ sung).
    - Nếu `false`: Chạy `analyzeImage`. Nếu dính blacklist -> Set `isCurrentCopySessionHigh = true` -> Broadcast `HIGH` kèm ảnh. Nếu không -> Broadcast `MEDIUM` kèm ảnh.
  - Xử lý các event khác theo Rule (Mục 2). Console.log đầy đủ.

### File 2: student.html (Nơi kích hoạt Data)
- UI Thô: Nút "Start Screen Share", `<input type="text">` test Blur, `<p>` test Copy, `<video hidden>`, `<canvas hidden>`.
- Logic JS:
  - Hàm `captureFrame()`: Vẽ `<video>` ra `<canvas>` (Scale width 1280px, nén jpeg 0.4), trả về base64.
  - Bắt `blur`, `visibilitychange`: Chụp 1 ảnh, gửi JSON Event tương ứng.
  - Sự kiện `copy`: 
    - Gửi ngay JSON `COPY_DETECTED` kèm text.
    - Chạy `let count = 0; let interval = setInterval(...)`. Mỗi 2 giây: gọi `captureFrame()`, gửi JSON `COPY_FRAME` kèm ảnh. Tăng count. Đủ 5 lần thì `clearInterval`.

### File 3: admin.html (Bảng điều khiển & Nhận Log)
- UI Cấu hình: `<select id="mode">`, `<input type="text" id="blacklist" value="chatgpt,zalo,gemini">`, `<button>`.
- UI Log: `<div id="logArea"></div>`
- Logic JS:
  - Nhận WS từ Server:
    - Nếu `LOW` -> Bỏ qua.
    - Nếu event `COPY_DETECTED` -> Render thẻ `div` báo: `[MEDIUM] - Thí sinh copy text: {copied_text} - Bắt đầu theo dõi...`
    - Nếu event `COPY_FRAME` hoặc `BLUR` (Alert = MEDIUM / HIGH): Cập nhật trạng thái màu sắc (Vàng/Đỏ) và gắn thêm thẻ `<img>` vào giao diện để admin thấy chuỗi bằng chứng.