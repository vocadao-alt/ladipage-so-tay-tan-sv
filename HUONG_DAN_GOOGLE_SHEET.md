# HƯỚNG DẪN KẾT NỐI VÀ XỬ LÝ LỖI GOOGLE SHEET
*(Dành cho dự án Sổ tay Sinh viên)*

---

## 🔍 NGUYÊN NHÂN VÌ SAO BẠN CHƯA THẤY DỮ LIỆU:

Khi kiểm tra từ hệ thống, đường dẫn Web App của bạn **đang hoạt động và đã ghi nhận thành công đến dòng số 5** (`{"result":"success","row":5}`).

Tuy nhiên, bạn chưa nhìn thấy trên màn hình là do 1 trong 2 khả năng sau:
1. **Google Apps Script đang ghi vào một file Google Sheet khác** trong Google Drive của bạn (ví dụ file tạo lúc đầu, khác với file `Dữ liệu khách hàng - Sổ tay Tân SV` bạn đang mở).
2. **Google Apps Script đang ghi vào một Tab khác** (ví dụ: tab Trang tính 2 thay vì Trang tính 1 ở dưới cùng màn hình).

---

## 🛠️ CÁCH KHẮC PHỤC TRIỆT ĐỂ (TRONG 1 PHÚT):

Hãy cập nhật lại đoạn mã trong Google Apps Script với bản nâng cấp dưới đây. Bản này sẽ:
- **Tự động gắn chính xác vào Tab đầu tiên** của bảng tính (`getSheets()[0]`).
- **Tích hợp tính năng tự kiểm tra (doGet):** Bạn chỉ cần bấm vào đường link Web App trên trình duyệt là sẽ thấy ngay tên file Google Sheet và đường link dẫn thẳng tới file đang nhận dữ liệu!

### Bước 1: Mở Apps Script
1. Trên Google Sheet `Dữ liệu khách hàng - Sổ tay Tân SV`, bấm menu **Tiện ích mở rộng (Extensions)** -> **Apps Script**.
2. Xóa toàn bộ mã cũ và dán toàn bộ đoạn mã mới dưới đây:

```javascript
/**
 * GOOGLE APPS SCRIPT - BẢN NÂNG CẤP CHUẨN XÁC 100%
 */

// 1. Hàm kiểm tra: Bấm vào link Web App trên trình duyệt sẽ thấy thông tin file nhận dữ liệu
function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheets()[0]; // Luôn lấy tab đầu tiên
  var lastRow = sheet.getLastRow();
  var recentData = lastRow > 0 ? sheet.getRange(1, 1, lastRow, sheet.getLastColumn()).getValues() : [];

  var info = {
    "trang_thai": "Web App dang hoat dong tot!",
    "ten_file_google_sheet": ss.getName(),
    "link_mo_file_truc_tiep": ss.getUrl(),
    "ten_tab_nhan_du_lieu": sheet.getName(),
    "tong_so_dong_hien_co": lastRow,
    "du_lieu_cac_dong": recentData
  };

  return ContentService
    .createTextOutput(JSON.stringify(info, null, 2))
    .setMimeType(ContentService.MimeType.JSON);
}

// 2. Hàm nhận dữ liệu từ Form
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    // LUÔN LẤY TAB ĐẦU TIÊN CỦA BẢNG TÍNH
    var sheet = ss.getSheets()[0];
    
    // Tự tạo tiêu đề nếu bảng tính còn trống
    if (sheet.getLastRow() === 0) {
      var headers = [
        "Thời gian gửi", 
        "Họ và tên", 
        "Nhóm đối tượng", 
        "Địa chỉ Email", 
        "Lời nhắn / Câu hỏi băn khoăn"
      ];
      sheet.appendRow(headers);
      
      var headerRange = sheet.getRange(1, 1, 1, headers.length);
      headerRange.setBackground("#1e3a5f");
      headerRange.setFontColor("#ffffff");
      headerRange.setFontWeight("bold");
      headerRange.setHorizontalAlignment("center");
      sheet.setFrozenRows(1);
      sheet.setRowHeight(1, 38);
    }

    var params = (e && e.parameter) ? e.parameter : {};
    var timestamp = Utilities.formatDate(new Date(), "Asia/Ho_Chi_Minh", "dd/MM/yyyy HH:mm:ss");
    var name = params.name || "";
    var role = params.role_label || params.role || "";
    var email = params.email || "";
    var message = params.message || "";

    // Ghi dòng mới
    sheet.appendRow([timestamp, name, role, email, message]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: "success", row: sheet.getLastRow() }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: "error", error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);

  } finally {
    lock.releaseLock();
  }
}
```

3. Bấm biểu tượng **Đĩa mềm (Lưu / Save / Ctrl + S)**.

---

### Bước 2: Triển khai phiên bản mới (CỰC KỲ QUAN TRỌNG)
Trong Google Apps Script, mỗi lần sửa code bạn **phải cập nhật phiên bản triển khai** thì link mới nhận code mới:
1. Bấm nút màu xanh **Triển khai (Deploy)** ở góc trên bên phải -> chọn **Quản lý phiên bản triển khai (Manage deployments)**.
2. Bấm vào biểu tượng **Cây bút chì (Chỉnh sửa / Edit)** ở góc trên bên phải hộp thoại.
3. Ở dòng **Phiên bản (Version)**, bấm chọn **Phiên bản mới (New version)**.
4. Đảm bảo mục **Ai có quyền truy cập (Who has access)** vẫn là **Bất kỳ ai (Anyone)**.
5. Bấm nút **Triển khai (Deploy)**.

---

### Bước 3: Kiểm tra link Web App
Sau khi cập nhật:
- Bạn hãy nhấp trực tiếp vào đường link Web App trên trình duyệt:
  `https://script.google.com/macros/s/AKfycbwWBPf5yA8aTFQdGjRqKmzDZl7FU--8rfyVqn810b6Ck2xmPZvLt5Zkqj0xkmHOPAO-/exec`
- Trang web sẽ hiện ra thông tin dạng chữ:
  - `"ten_file_google_sheet"`
  - `"link_mo_file_truc_tiep"`: Bạn bấm vào link này sẽ mở ra đúng file Google Sheet đang nhận dữ liệu!
  - `"tong_so_dong_hien_co"`
- Khi quay lại landing page bấm gửi, dữ liệu chắc chắn sẽ nhảy ngay vào tab đầu tiên của file!
