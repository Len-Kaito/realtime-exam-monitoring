# Real-Time Exam Monitoring System

POC hệ thống giám sát thi trực tuyến thời gian thực sử dụng Node.js, WebSocket và Tesseract.js OCR.

## Tính năng

- Theo dõi trạng thái chia sẻ màn hình của thí sinh.
- Chỉ chấp nhận chế độ chia sẻ **Entire Screen**.
- Phát hiện:
  - Rời khỏi trang (`BLUR`)
  - Chuyển tab (`VISIBILITY_CHANGE`)
  - Sao chép nội dung (`COPY`)
- OCR ảnh chụp màn hình bằng Tesseract.js.
- Blacklist từ khóa tùy chỉnh (ChatGPT, Gemini, Zalo,...).
- Dashboard giám thị nhận cảnh báo thời gian thực.
- Hỗ trợ 2 chế độ:
  - **THUC_HANH**
  - **TRAC_NGHIEM**

## Công nghệ sử dụng

- Node.js
- Express
- WebSocket (ws)
- Tesseract.js
- HTML/CSS/JavaScript

## Cài đặt

```bash
npm install
npm start
```
