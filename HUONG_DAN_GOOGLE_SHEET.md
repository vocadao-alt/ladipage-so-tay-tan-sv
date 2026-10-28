# HƯỚNG DẪN CẤU HÌNH GOOGLE APPS SCRIPT CHO SỔ TAY TÂN SINH VIÊN

Tài liệu này hướng dẫn bạn cài đặt mã Google Apps Script để tự động đồng bộ mọi lượt gửi từ trang Landing Page vào bảng tính Google Sheet của bạn.

---

## 📌 THÔNG TIN BẢNG TÍNH ĐÍCH

- **ID Bảng tính**: `1OSvil1reU79m7g79HFMUqBejy8s1LUwDTh_mL_ryDOg`
- **Tên trang tính (Sheet Tab)**: `Landing-page-so-tay-sinh-vien`
- **URL Web App chính thức**: `https://script.google.com/macros/s/AKfycbzwE1M-foNz7g1ugCTENm7BxD4j9zeqluL4tC0fIK_7kv9Hl1KZ_aB5K2VDIiE0EA9P/exec`
- **Các cột dữ liệu được lưu tự động**:
  1. `Thời gian gửi` (Timestamp)
  2. `Họ và tên`
  3. `Vai trò` (Tân sinh viên, Học sinh THPT, Phụ huynh, ...)
  4. `Địa chỉ Email`
  5. `Nội dung bạn mong muốn tìm hiểu hoặc chủ đề bạn cần giải đáp`

---

## 📝 BƯỚC 1: DÁN MÃ VÀO GOOGLE APPS SCRIPT

1. Mở file Google Sheet của bạn trên trình duyệt (hoặc mở trực tiếp link: [Bảng tính Google Sheet của bạn](https://docs.google.com/spreadsheets/d/1OSvil1reU79m7g79HFMUqBejy8s1LUwDTh_mL_ryDOg/edit)).
2. Trên thanh menu, chọn **Tiện ích mở rộng** (Extensions) -> **Apps Script**.
3. Xóa toàn bộ nội dung trong trình soạn thảo `Code.gs` và dán đoạn mã sau vào:

```javascript
/**
 * GOOGLE APPS SCRIPT CHO LANDING PAGE SỔ TAY TÂN SINH VIÊN
 * Bảng tính ID: 1OSvil1reU79m7g79HFMUqBejy8s1LUwDTh_mL_ryDOg
 * Tên trang tính: Landing-page-so-tay-sinh-vien
 */

var SPREADSHEET_ID = "1OSvil1reU79m7g79HFMUqBejy8s1LUwDTh_mL_ryDOg";
var SHEET_NAME = "Landing-page-so-tay-sinh-vien";

// 1. Hàm kiểm tra kết nối qua trình duyệt (doGet)
function doGet(e) {
  try {
    var ss = SpreadsheetApp.openById(SPREADSHEET_ID);
    var sheet = ss.getSheetByName(SHEET_NAME);
    
    if (!sheet) {
      sheet = ss.insertSheet(SHEET_NAME);
      initHeaders(sheet);
    }
    
    var lastRow = sheet.getLastRow();
    
    var response = {
      status: "success",
      message: "Google Apps Script đang hoạt động tốt và kết nối thành công!",
      spreadsheet_name: ss.getName(),
      spreadsheet_url: ss.getUrl(),
      sheet_tab_name: sheet.getName(),
      total_rows: lastRow
    };
    
    return ContentService
      .createTextOutput(JSON.stringify(response, null, 2))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: "error", message: err.toString() }, null, 2))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// 2. Hàm nhận dữ liệu từ Form (doPost)
function doPost(e) {
  var lock = LockService.getScriptLock();
  // Khóa tối đa 10 giây để chống xung đột ghi đồng thời
  lock.tryLock(10000);
  
  try {
    var ss = SpreadsheetApp.openById(SPREADSHEET_ID);
    var sheet = ss.getSheetByName(SHEET_NAME);
    
    // Nếu tab chưa tồn tại, tự động tạo mới
    if (!sheet) {
      sheet = ss.insertSheet(SHEET_NAME);
    }
    
    // Nếu bảng tính còn trống (chưa có header), khởi tạo tiêu đề
    if (sheet.getLastRow() === 0) {
      initHeaders(sheet);
    }
    
    // Lấy dữ liệu gửi lên (hỗ trợ cả x-www-form-urlencoded và JSON)
    var params = {};
    if (e && e.parameter && Object.keys(e.parameter).length > 0) {
      params = e.parameter;
    } else if (e && e.postData && e.postData.contents) {
      try {
        params = JSON.parse(e.postData.contents);
      } catch (jsonErr) {
        // fallback
      }
    }
    
    var timestamp = Utilities.formatDate(new Date(), "Asia/Ho_Chi_Minh", "dd/MM/yyyy HH:mm:ss");
    var name = params.name || "";
    var role = params.role_label || params.role || "";
    var email = params.email || "";
    var message = params.message || "";
    
    // Thêm dòng mới vào sheet
    sheet.appendRow([timestamp, name, role, email, message]);
    
    var output = {
      result: "success",
      message: "Dữ liệu đã được ghi nhận thành công!",
      row: sheet.getLastRow()
    };
    
    return ContentService
      .createTextOutput(JSON.stringify(output))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (err) {
    var errOutput = {
      result: "error",
      message: err.toString()
    };
    return ContentService
      .createTextOutput(JSON.stringify(errOutput))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

// 3. Hàm khởi tạo tiêu đề cột và định dạng giao diện đẹp mắt
function initHeaders(sheet) {
  var headers = [
    "Thời gian gửi",
    "Họ và tên",
    "Vai trò",
    "Địa chỉ Email",
    "Nội dung bạn mong muốn tìm hiểu hoặc chủ đề bạn cần giải đáp"
  ];
  sheet.appendRow(headers);
  
  var headerRange = sheet.getRange(1, 1, 1, headers.length);
  headerRange.setBackground("#1e3a5f");     // Màu xanh navy
  headerRange.setFontColor("#ffffff");      // Chữ trắng
  headerRange.setFontWeight("bold");        // In đậm
  headerRange.setHorizontalAlignment("center");
  headerRange.setVerticalAlignment("middle");
  sheet.setFrozenRows(1);                   // Cố định dòng 1
  sheet.setRowHeight(1, 38);               // Chiều cao dòng tiêu đề
  
  // Tự động giãn độ rộng các cột
  sheet.setColumnWidth(1, 160); // Thời gian
  sheet.setColumnWidth(2, 180); // Họ và tên
  sheet.setColumnWidth(3, 220); // Vai trò
  sheet.setColumnWidth(4, 220); // Email
  sheet.setColumnWidth(5, 420); // Nội dung
}
```

4. Nhấn biểu tượng **Lưu** (Ctrl + S / Cmd + S).

---

## 🚀 BƯỚC 2: TRIỂN KHAI THÀNH WEB APP (DEPLOY)

> [!IMPORTANT]
> Bước cấu hình quyền này rất quan trọng để mọi người vào xem landing page đều gửi được dữ liệu.

1. Ở góc trên bên phải của màn hình Apps Script, bấm nút **Triển khai** (Deploy) -> chọn **Tùy chọn triển khai mới** (New deployment).
2. Tại mục **Chọn loại** (hình bánh răng bên trái), chọn **Ứng dụng web** (Web app).
3. Điền các trường cấu hình như sau:
   - **Mô tả** (Description): `LadiPage So tay Sinh vien v1`
   - **Thực thi dưới dạng** (Execute as): **Tôi** (`Me - your_email@gmail.com`)
   - **Ai có quyền truy cập** (Who has access): **Bất kỳ ai** (`Anyone`) *(Bắt buộc chọn Anyone để khách truy cập website không cần đăng nhập Google vẫn gửi được)*
4. Bấm nút **Triển khai** (Deploy).
5. Nếu Google hiển thị thông báo yêu cầu cấp quyền:
   - Bấm **Ủy quyền truy cập** (Authorize access) -> Chọn tài khoản Google của bạn.
   - Bấm **Nâng cao** (Advanced) -> Bấm **Đi tới... (không an toàn)** / `Go to ... (unsafe)`.
   - Bấm **Cho phép** (Allow).
6. Sau khi hoàn tất, Google sẽ cấp cho bạn một **URL ứng dụng web** (Web app URL) có định dạng:
   `https://script.google.com/macros/s/AKfycb.../exec`
7. Hãy sao chép (Copy) đường dẫn này.

---

## 🔗 BƯỚC 3: CẬP NHẬT VÀO LANDING PAGE

Gửi lại đường dẫn Web App URL bạn vừa nhận được ở bước trên cho tôi, hoặc dán trực tiếp vào:
- File [index.html](file:///c:/Users/Admin/Downloads/LadiPage%20S%E1%BB%95%20tay%20T%C3%A2n%20SV/index.html) tại dòng:
  `const GOOGLE_SHEET_WEBAPP_URL = "DÁN_URL_VÀO_ĐÂY";`
  và thuộc tính `action="DÁN_URL_VÀO_ĐÂY"` của thẻ `<form id="inquiryForm">`.
