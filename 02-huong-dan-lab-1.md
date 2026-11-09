# Lab 1 — Kiểm tra XSS bằng ZAP và Chrome

**Chuẩn bị:** hoàn tất [cài ZAP và cấu hình Chrome](01-cai-dat-zap-va-chrome.md), giữ EShop đang chạy. Chỉ làm trên EShop localhost của môn học.

**Bạn sẽ làm:** bắt request tìm kiếm → cho ZAP thử các payload → kiểm tra XSS trên Chrome. Payload là chuỗi dữ liệu dùng để thử lỗi.

## Bước 1. Mở ZAP, bật Chrome và bắt request tìm kiếm

1. Mở **ZAP**. Nếu được hỏi lưu phiên, chọn **No, I do not want to persist this session** → **Start**.
2. Chọn tab **Quick Start** → bấm **Manual Explore**.
3. Trong ô **URL to explore**, nhập `http://localhost:5173`.
4. Ở danh sách trình duyệt đang hiện **Firefox**, đổi sang **Chrome**. Bỏ chọn **Enable HUD** nếu có.
5. Bấm **Launch Browser**. Nếu Chrome không mở, xem **Bước 5** trong [hướng dẫn cài đặt](01-cai-dat-zap-va-chrome.md).
6. Trong cửa sổ Chrome vừa mở, nhập `test` vào ô **Tìm kiếm...** của EShop → bấm **Tìm**.
7. Quay lại ZAP → tab **History** phía dưới → chọn dòng `GET http://localhost:3000/api/products?search=test`.

**Kết quả cần thấy:** tab **Request** có `search=test`. Chụp ảnh dòng request này.

## Bước 2. Chọn chỗ thay payload

1. Chuột phải dòng request vừa chọn → **Attack → Fuzz...**.
2. Trong cửa sổ mới, tìm đoạn `search=test`.
3. Bôi đen **chỉ chữ `test`** → bấm **Add...**.

ZAP sẽ thay chữ `test` bằng từng dòng trong file payload. Không bôi đen cả `search=`.

## Bước 3. Nạp file XSS và chạy

1. Trong cửa sổ **Payloads**, bấm **Add...**.
2. Ở **Type**, chọn **File** → bấm nút chọn file.
3. Chọn file `xss-payloads.txt` trong thư mục `lab1-zap` vừa tải → xác nhận thêm file.
4. Kiểm tra danh sách có **17 dòng**.
5. Trong **Processors...** của danh sách payload, thêm **URL Encode** nếu chưa có → xác nhận. Chỉ thêm một lần để payload được gửi đúng trên URL.
6. Bấm **OK** để quay về cửa sổ Fuzzer → **Start Fuzzer**.

**Kết quả cần thấy:** bảng kết quả hiện các request đã gửi. Chụp ảnh bảng này.

Nếu không có menu **Fuzz**, mở **Manage Add-ons** trong ZAP, cài **Fuzzer** rồi thử lại. [Tham khảo URL Encode](https://www.zaproxy.org/docs/desktop/addons/fuzzer/processors/).

## Bước 4. Xem kết quả

Chọn một dòng trong bảng Fuzzer:

1. Mở **Request**: kiểm tra chữ `test` đã được thay bằng payload.
2. Mở **Response**: xem mã trạng thái và nội dung trả về.

| Kết quả | Ý nghĩa |
| --- | --- |
| `200`, nội dung `[]` | Không có sản phẩm phù hợp; đây là kết quả bình thường |
| `500`, có `Database Error` | Payload có dấu `'` có thể gây lỗi truy vấn SQL |

**Các kết quả trên chưa chứng minh XSS.** Fuzzer gửi request đến API; cần làm bước 5 để kiểm tra JavaScript có chạy trên giao diện không.

## Bước 5. Kiểm tra XSS trên Chrome

1. Quay lại Chrome, mở trang `http://localhost:5173`.
2. Dán nguyên dòng sau vào **ô tìm kiếm trên trang**:

   ```html
   <img src=x onerror=alert(document.domain)>
   ```

3. Bấm **Tìm**. Hộp thoại có thể xuất hiện ngay khi dán, trước lúc bấm nút.
4. Nếu xuất hiện hộp thoại chứa chữ **localhost**, chụp ảnh **trước khi bấm OK**. Đây là bằng chứng JavaScript đã chạy.
5. Tải lại trang, nhập `ZAPX13_benign_marker` → bấm **Tìm**. Chuỗi này chỉ hiển thị văn bản, không được bật hộp thoại.

**Lưu ý:** dán payload vào ô tìm kiếm, không dán vào thanh địa chỉ. Nếu hộp thoại bị tự đóng trong Chrome do ZAP mở, dùng cách tự mở Chrome bằng PowerShell ở cuối [hướng dẫn setup](01-cai-dat-zap-va-chrome.md) rồi thử lại.

## Bước 6. Giải thích và nộp bài

EShop đưa từ khóa tìm kiếm vào HTML bằng `dangerouslySetInnerHTML`, nên đoạn `onerror=...` có thể chạy JavaScript. Đây là XSS phía trình duyệt.

Nộp:

- Ảnh request tìm kiếm trong **History**.
- Ảnh kết quả **Fuzzer**.
- Ảnh hộp thoại **localhost**, kèm payload đã dùng.
