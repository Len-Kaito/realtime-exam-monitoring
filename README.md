# Real-Time Exam Monitoring System (Hệ thống giám sát thi trực tuyến)

Hệ thống giám sát thi cử trực tuyến theo thời gian thực dựa trên Node.js, WebSockets, và Tesseract.js (OCR). Hệ thống giúp giám thị (Admin) theo dõi hành vi của thí sinh, phát hiện gian lận và cảnh báo theo thời gian thực.

## 🚀 Tính năng nổi bật

- **Bắt buộc chia sẻ toàn bộ màn hình**: Thí sinh bắt buộc phải chia sẻ toàn bộ màn hình (Entire Screen). Nếu chia sẻ sai (chỉ chia sẻ một tab hoặc một cửa sổ) hoặc ngắt chia sẻ, hệ thống sẽ cảnh báo lập tức.
- **Phát hiện chuyển Tab / Rời khỏi màn hình (Blur & Visibility Change)**: Hệ thống ghi nhận mọi thao tác rời khỏi bài thi, bao gồm cả việc chuyển sang phần mềm khác.
- **Theo dõi sao chép (Copy Detection)**: Nếu thí sinh bôi đen và sao chép nội dung, hệ thống sẽ cảnh báo và lưu lại chuỗi văn bản đã sao chép.
- **Phân tích hình ảnh (OCR) thời gian thực**: Khi phát hiện hành vi đáng ngờ (chuyển tab, sao chép), hệ thống tự động chụp màn hình hiện tại và dùng AI OCR đọc chữ trên màn hình. Nếu có chứa các từ khóa cấm (VD: *chatgpt, zalo, messenger...*), mức độ cảnh báo sẽ được nâng lên mức CAO.
- **Giao diện Giám thị (Admin Dashboard) trực quan**:
  - Phân loại log cảnh báo thành các thẻ (Tabs): *Tất cả*, *Chia sẻ màn hình*, *Sao chép*, *Hành vi*.
  - Hiển thị theo thời gian thực trạng thái kết nối và chia sẻ màn hình của thí sinh.
  - Tùy chỉnh danh sách từ khóa cấm trực tiếp trên giao diện.
- **Hai chế độ giám sát linh hoạt**:
  - **Chế độ Thực hành**: Bật OCR để phân tích từ khóa cấm trên ảnh. Phân loại cảnh báo Cao/Trung Bình/Thấp dựa trên mức độ nghiêm trọng và từ khóa.
  - **Chế độ Trắc nghiệm**: Siết chặt kỷ luật, cấm mọi hành vi chuyển tab hay rời trang. Mọi vi phạm đều bị đánh dấu mức CAO (Vi phạm quy chế).

## 🛠️ Cài đặt & Khởi động

1. **Yêu cầu hệ thống**:
   - Máy tính đã cài đặt [Node.js](https://nodejs.org/) (Khuyên dùng bản LTS mới nhất).

2. **Cài đặt thư viện**:
   Mở terminal tại thư mục dự án và chạy:
   ```bash
   npm install
   ```
   *(Hệ thống sử dụng các thư viện chính: `express`, `ws`, `tesseract.js`)*

3. **Khởi chạy Server**:
   ```bash
   node server.js
   ```

4. **Sử dụng**:
   - **Giao diện Thí sinh**: Mở trình duyệt và truy cập `http://localhost:3000/student`
   - **Giao diện Giám thị (Admin)**: Mở trình duyệt và truy cập `http://localhost:3000/admin`

## 📂 Cấu trúc thư mục

- `server.js`: Máy chủ HTTP & WebSocket, đóng vai trò xử lý logic nhận/gửi tín hiệu và chạy nền tảng OCR (Tesseract.js).
- `student.html`: Giao diện dành cho thí sinh. Cấu hình tự động gửi sự kiện và stream màn hình.
- `admin.html`: Giao diện quản lý dành cho giám thị. Nhận luồng log sự kiện thời gian thực và hiển thị trạng thái sinh viên.
- `package.json`: Chứa thông tin cấu hình và các package phụ thuộc.

## 📝 Chú ý
Do hệ thống phân tích hình ảnh (OCR) đòi hỏi sức mạnh xử lý CPU, Tesseract.js được khởi tạo sẵn worker tĩnh trong bộ nhớ.
Môi trường test khuyên dùng là trình duyệt Chrome / Edge để hỗ trợ tốt nhất API ghi hình (`getDisplayMedia`).
