# Cài ZAP và cấu hình Chrome

Làm lần lượt các bước dưới đây trước khi làm [Lab 1](02-huong-dan-lab-1.md).

## Bước 1. Cài công cụ

1. Cài **Google Chrome** nếu máy chưa có.
2. Cài **Java 17 trở lên** từ [Eclipse Temurin](https://adoptium.net/). Khi cài, bật tùy chọn thêm Java vào `PATH` nếu có.
3. Mở PowerShell mới, chạy `java -version`. Thấy phiên bản từ 17 trở lên là được.
4. Vào [trang tải ZAP](https://www.zaproxy.org/download/), chọn **Windows (64) Installer** nếu dùng Windows 64 bit.
5. Mở file vừa tải → chấp nhận giấy phép → chọn **Standard** → cài xong thì mở ZAP.
6. Khi ZAP hỏi lưu phiên làm việc, có thể chọn **No, I do not want to persist this session** → **Start**.

Để tự chạy EShop, máy cần có **Node.js và npm**. Nếu cài thư viện báo `EBADENGINE`, cập nhật Node.js theo phiên bản được yêu cầu trong thông báo.

## Bước 2. Chạy EShop

Mã nguồn đã có trong thư mục `eshop-sut` đi kèm hướng dẫn. Trong VS Code, mở **thư mục có file `README.md` và thư mục `eshop-sut`**. Chọn **Terminal → New Terminal**, chạy:

```powershell
cd .\eshop-sut\backend
npm install
node server.js
```

Mở **terminal thứ hai**, cũng bắt đầu từ thư mục gốc, chạy:

```powershell
cd .\eshop-sut\frontend-web
npm install
npm run dev -- --port 5173 --strictPort
```

**Giữ cả hai terminal chạy.** Lần sau có thể bỏ qua `npm install` nếu thư viện không thay đổi.

Mở `http://localhost:5173` bằng Chrome. Thấy danh sách sản phẩm và ô **Tìm kiếm...** là được. Nếu EShop đã chạy sẵn, chỉ cần kiểm tra trang này.

> Backend hiện tại tạo lại dữ liệu mẫu mỗi lần khởi động. Dùng bản EShop riêng cho lab; sao lưu `backend/database.sqlite` khi backend đã dừng nếu cần giữ dữ liệu cũ.

## Bước 3. Kiểm tra ZAP

1. Trong ZAP, chọn **Tools → Options → Network → Local Servers/Proxies**.
2. Kiểm tra proxy chính có địa chỉ `127.0.0.1`, cổng `8080` và được bật → **OK**.
3. Chọn chế độ **Standard** ở thanh công cụ. Tắt **Break** nếu đang bật để request không bị giữ lại.

## Bước 4. Cấu hình Chrome trong ZAP

### 4.1. Cho ZAP biết Chrome nằm ở đâu

1. Mở Chrome bình thường, nhập `chrome://version` vào thanh địa chỉ.
2. Tìm dòng **Executable Path** (Đường dẫn thực thi), sao chép đường dẫn kết thúc bằng `chrome.exe`. Ví dụ:

   ```text
   C:\Program Files\Google\Chrome\Application\chrome.exe
   ```

3. Trong ZAP, chọn **Tools → Options → Selenium**.
4. Nếu thấy bảng **Browsers**, chọn dòng **Chrome** → **Modify...** → dán đường dẫn vào ô **Binary**. Nếu phiên bản ZAP có ô **Chrome Binary** riêng, dán vào ô đó.
5. Đây là đường dẫn đến `chrome.exe`; ô **Driver** dành cho `chromedriver.exe`. Giữ cấu hình Driver do ZAP cung cấp nếu chưa cần sửa.
6. Bấm **OK** để lưu các cửa sổ cấu hình.

Nếu không có mục **Selenium**, cài add-on theo mục **5.1** bên dưới. [Tham khảo cấu hình Selenium](https://www.zaproxy.org/docs/desktop/addons/selenium/options/).

### 4.2. Đổi Firefox sang Chrome trong Manual Explore

1. Chọn tab **Quick Start** → **Manual Explore**.
2. Ô **URL to explore**: nhập `http://localhost:5173`.
3. Ở vùng **Explore your application**, bấm mũi tên của danh sách trình duyệt đang hiện **Firefox** → chọn **Chrome**. Không chọn **Chrome Headless** vì chế độ đó không hiện cửa sổ.
4. Bỏ chọn **Enable HUD** nếu có → bấm **Launch Browser**.
5. Một cửa sổ Chrome mới sẽ mở trang EShop. Dùng chính cửa sổ này: nhập `test` vào ô tìm kiếm → bấm **Tìm**.
6. Quay lại ZAP → mở tab **History** phía dưới → tìm dòng:

   ```text
   GET http://localhost:3000/api/products?search=test
   ```

**Thấy dòng này là setup xong.** Chuyển sang [hướng dẫn Lab 1](02-huong-dan-lab-1.md).

ZAP tự cấu hình proxy cho Chrome khi mở theo cách này. [Tham khảo thao tác chính thức](https://www.zaproxy.org/getting-started/).

## Bước 5. Nếu ZAP không mở được Chrome

Làm lần lượt, sau mỗi phần quay lại **Manual Explore → Chrome → Launch Browser** để thử.

### 5.1. Cập nhật Chrome và công cụ điều khiển trình duyệt

1. Mở Chrome bình thường → nhập `chrome://settings/help` → đợi cập nhật hoàn tất → bấm **Relaunch** nếu được yêu cầu.
2. Trong ZAP, mở **Manage Add-ons** bằng nút Marketplace trên thanh công cụ.
3. Ở tab **Installed**, cập nhật **Selenium** và **Windows WebDrivers** nếu có bản mới. Nếu chưa cài, tìm chúng trong tab **Marketplace**, chọn rồi bấm **Install**.
4. Đóng cửa sổ Chrome do ZAP mở trước đó, thoát ZAP rồi mở lại để driver mới được áp dụng.

WebDriver là công cụ ZAP dùng để điều khiển Chrome. Nếu lỗi có chữ **session not created** hoặc **only supports Chrome version...**, driver có thể không khớp phiên bản Chrome. Xem [hướng dẫn xử lý của ZAP](https://www.zaproxy.org/faq/how-can-i-fix-browser-was-not-found/).

### 5.2. ZAP báo “browser was not found” hoặc không tìm thấy Chrome

1. Làm lại mục **4.1**, lấy đúng **Executable Path** từ Chrome trên máy mình.
2. Kiểm tra ô **Binary / Chrome Binary** trỏ đến file `chrome.exe`, không phải thư mục hoặc shortcut `.lnk`.
3. Lưu cấu hình rồi thử mở lại Chrome từ ZAP.

### 5.3. Vẫn lỗi driver sau khi cập nhật

1. Trong ZAP, chọn **Help → Support Info... → Open ZAP Home**.
2. Giữ cửa sổ thư mục vừa mở, đóng Chrome do ZAP điều khiển rồi thoát ZAP.
3. Trong **đúng thư mục ZAP Home đó**, xóa thư mục con `webdriver` nếu có. Không xóa cả ZAP Home.
4. Mở lại ZAP để công cụ tạo lại thư mục driver → thử **Launch Browser** lần nữa.

Đây là bước làm mới driver trên Windows theo [FAQ của ZAP](https://www.zaproxy.org/faq/how-can-i-fix-browser-was-not-found/).

### 5.4. Cách dự phòng: tự mở Chrome qua proxy ZAP

Nếu vẫn chưa mở được từ ZAP, giữ ZAP chạy, mở PowerShell và chạy đoạn dưới. Sửa `$chromePath` theo đường dẫn đã lấy ở mục **4.1**:

```powershell
$chromePath = 'C:\Program Files\Google\Chrome\Application\chrome.exe'
& $chromePath "--user-data-dir=$env:TEMP\eshop-zap-chrome" `
    '--proxy-server=http://127.0.0.1:8080' `
    '--proxy-bypass-list=<-loopback>' 'http://localhost:5173'
```

Nếu báo không tìm thấy file, sửa `$chromePath` thành đường dẫn `chrome.exe` trên máy. Thường nằm trong `C:\Program Files (x86)\Google\Chrome\Application` hoặc `%LOCALAPPDATA%\Google\Chrome\Application`.

Lệnh tạo cửa sổ Chrome riêng cho lab và cho request `localhost` đi qua ZAP. Nếu cửa sổ lab này đang mở, đóng nó trước khi chạy lại lệnh. [Tham khảo cấu hình proxy Chrome](https://chromium.googlesource.com/chromium/src/+/HEAD/net/docs/proxy.md).

Sau đó tìm kiếm `test` lần nữa và kiểm tra History. Nếu gặp `ERR_PROXY_CONNECTION_FAILED`, kiểm tra ZAP vẫn mở và dùng đúng cổng `8080`.
