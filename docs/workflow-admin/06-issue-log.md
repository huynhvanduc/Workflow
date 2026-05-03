# 06 · Ghi nhận lỗi & Trạng thái xử lý (Issue Log)

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## Mục đích

Issue Log được dùng trong **giai đoạn thử nghiệm** (`TESTING`) để:

- Tester và admin **ghi nhận các lỗi hoặc phản ánh** phát sinh khi chạy hồ sơ thử.
- Theo dõi **tiến trình xử lý** từng lỗi.
- Đảm bảo **điều kiện publish**: không còn issue nào ở trạng thái `OPEN` hoặc `IN_PROGRESS`.

---

## Cấu trúc Issue Log

| Trường | Kiểu | Mô tả |
|---|---|---|
| `Id` | UUID | Định danh duy nhất |
| `WorkflowDefinitionId` | FK | Workflow liên quan |
| `Title` | string | Tiêu đề lỗi / phản ánh |
| `Description` | text | Mô tả chi tiết, bước nào xảy ra, điều kiện tái hiện |
| `Severity` | enum | `CRITICAL` / `MAJOR` / `MINOR` |
| `Status` | enum | `OPEN` → `IN_PROGRESS` → `RESOLVED` → `CLOSED` |
| `ReportedBy` | FK User | Người ghi nhận |
| `ReportedAt` | datetime | Thời điểm ghi nhận |
| `AssignedTo` | FK User | Người được giao xử lý (admin) |
| `ResolvedBy` | FK User | Người xác nhận đã xử lý xong |
| `ResolvedAt` | datetime | Thời điểm giải quyết |
| `ClosedBy` | FK User | Người đóng issue (tester xác nhận) |
| `ClosedAt` | datetime | Thời điểm đóng |
| `Comment` | text | Ghi chú thêm khi chuyển trạng thái |

---

## Vòng đời của một Issue

```
[OPEN]
  │  Admin hoặc developer tiếp nhận
  ▼
[IN_PROGRESS]
  │  Đã fix / đã điều chỉnh workflow
  ▼
[RESOLVED]
  │  Tester xác nhận fix đúng, không còn lỗi
  ▼
[CLOSED]
```

> **Lưu ý:** Tester **phải xác nhận** (`CLOSED`) mới được tính là đã xử lý xong. Nếu fix chưa đúng, tester có thể **mở lại** issue (`OPEN`) kèm ghi chú.

---

## Mức độ nghiêm trọng (Severity)

| Mức | Ý nghĩa | Ưu tiên xử lý |
|---|---|---|
| `CRITICAL` | Workflow không chạy được hoặc dữ liệu sai nghiêm trọng | Khẩn cấp – xử lý ngay |
| `MAJOR` | Tính năng quan trọng bị lỗi nhưng có thể workaround | Cao – xử lý trong ngày |
| `MINOR` | Lỗi nhỏ, giao diện, thông báo chưa đúng | Bình thường – xử lý trong sprint |

---

## Điều kiện để chuyển sang READY_FOR_PRODUCTION

Hệ thống tự động kiểm tra trước khi cho phép admin publish:

1. **Thời gian thử nghiệm** đã kết thúc (`EndDate` đã qua).
2. **Không còn** issue nào ở trạng thái `OPEN` hoặc `IN_PROGRESS`.

Nếu một trong hai điều kiện chưa thoả, nút "Publish to Production" sẽ bị **vô hiệu hoá** và hệ thống hiển thị lý do cụ thể.

---

*Xem tiếp: [07 · Checklist người dùng](./07-checklist-nguoi-dung.md)*
