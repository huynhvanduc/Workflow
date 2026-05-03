# 08 · Nguyên tắc quản trị & Tiêu chí kích hoạt

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## Nguyên tắc quản trị

### 1 · Workflow ACTIVE không sửa trực tiếp

Khi cần thay đổi, admin **tạo phiên bản mới (clone)** từ bản đang active.  
Bản gốc tiếp tục chạy cho đến khi bản mới được kích hoạt thành công.  
Điều này đảm bảo **không gián đoạn hồ sơ đang xử lý**.

### 2 · Workflow mới phải qua giai đoạn test trước khi ACTIVE

Không cho phép chuyển thẳng từ `DRAFT` → `ACTIVE`.  
Bắt buộc đi qua: `DRAFT` → `READY_FOR_TEST` → `TESTING` → `READY_FOR_PRODUCTION` → `ACTIVE`.

### 3 · Còn lỗi mở thì không được publish

Hệ thống tự động **chặn action "Publish to Production"** nếu Issue Log còn issue ở trạng thái `OPEN` hoặc `IN_PROGRESS`.

### 4 · Hết thời gian thử nghiệm mới được xét publish

Không cho phép publish trước ngày `EndDate` của giai đoạn thử nghiệm.  
Admin phải chờ đủ thời gian thử nghiệm đã cam kết.

### 5 · User nghiệp vụ chỉ sử dụng workflow ACTIVE

Các trạng thái `DRAFT`, `TESTING`, `ARCHIVED`... **hoàn toàn ẩn** với người dùng thông thường.  
Người dùng cuối không thể thao tác trên workflow chưa được kích hoạt chính thức.

### 6 · Versioning rõ ràng

Mỗi lần tạo bản mới từ workflow active, **số version tăng lên**.  
Lịch sử tất cả các version được **lưu trữ và có thể tra cứu**.

### 7 · Audit log đầy đủ

Mọi thay đổi trạng thái của workflow (ai thay đổi, lúc nào, lý do) đều được **ghi nhận và không thể xóa**.

---

## Tiêu chí kích hoạt (Activation Criteria)

Để workflow được phép chuyển sang trạng thái `ACTIVE`, **tất cả** các điều kiện sau phải thoả mãn:

| # | Điều kiện | Kiểm tra bởi |
|---|---|---|
| 1 | Đã hết thời gian thử nghiệm (`EndDate` đã qua) | Hệ thống tự động |
| 2 | Không còn issue `OPEN` hoặc `IN_PROGRESS` trong Issue Log | Hệ thống tự động |
| 3 | Workflow có ít nhất 1 bước `Start` và 1 bước `End` hợp lệ | Hệ thống tự động |
| 4 | Mỗi bước có `AssignmentRule` hợp lệ | Hệ thống tự động |
| 5 | Mỗi bước không phải `End` có ít nhất 1 `Transition` đi tiếp | Hệ thống tự động |
| 6 | Admin xác nhận publish (manual sign-off) | Admin |

---

## Quy tắc vô hiệu hoá và lưu trữ

| Hành động | Điều kiện | Kết quả |
|---|---|---|
| `DEACTIVATE` | Có bản mới đã được activate thay thế, hoặc quyết định tắt dịch vụ | Chuyển sang `INACTIVE`; hồ sơ đang xử lý theo bản cũ vẫn tiếp tục |
| `ARCHIVE` | Workflow không còn được dùng vĩnh viễn | Chuyển sang `ARCHIVED`; chỉ admin xem được lịch sử |
| `REACTIVATE` | Admin quyết định kích hoạt lại bản cũ (trường hợp khẩn cấp) | Chuyển từ `INACTIVE` → `ACTIVE` |

---

## Ma trận phân quyền theo trạng thái

| Trạng thái | Admin | Tester | Cán bộ nghiệp vụ | Người dùng cuối |
|---|---|---|---|---|
| `DRAFT` | ✅ Toàn quyền | ❌ | ❌ | ❌ |
| `READY_FOR_TEST` | ✅ Toàn quyền | ❌ | ❌ | ❌ |
| `TESTING` | ✅ Toàn quyền | ✅ Chạy thử, ghi lỗi | ❌ | ❌ |
| `READY_FOR_PRODUCTION` | ✅ Xét duyệt, publish | ❌ | ❌ | ❌ |
| `ACTIVE` | 👁 Xem, clone, deactivate | ❌ | ✅ Thao tác hồ sơ | ✅ Nộp & theo dõi |
| `INACTIVE` | ✅ Reactivate, archive | ❌ | ❌ | ❌ |
| `ARCHIVED` | 👁 Chỉ xem lịch sử | ❌ | ❌ | ❌ |

---

*Quay lại: [README – Mục lục](./README.md)*
