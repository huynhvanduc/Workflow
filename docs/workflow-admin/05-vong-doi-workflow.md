# 05 · Vòng đời Workflow & Giai đoạn thử nghiệm

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## Sơ đồ trạng thái

```
         ┌──────────────────────────────────────────────────────────────────┐
         │                                                                  │
         ▼                                                                  │
     [DRAFT]                                                                │
      (Admin tạo mới / clone từ bản active)                                │
         │                                                                  │
         │  Admin xác nhận sẵn sàng test                                    │
         ▼                                                                  │
  [READY_FOR_TEST]                                                          │
      (Chờ bắt đầu kiểm thử)                                               │
         │                                                                  │
         │  Bắt đầu giai đoạn thử nghiệm                                   │
         ▼                                                                  │
     [TESTING]  ◄──────── Ghi nhận lỗi / phản ánh trong giai đoạn này      │
         │                                                                  │
         │  Hết thời gian thử nghiệm                                        │
         │  VÀ không còn lỗi ở trạng thái "mở"                             │
         ▼                                                                  │
 [READY_FOR_PRODUCTION]                                                     │
      (Chờ admin xét duyệt lần cuối)                                       │
         │                                                                  │
         │  Admin publish                                                    │
         ▼                                                                  │
     [ACTIVE]  ◄─── Chỉ workflow ACTIVE mới cho user nghiệp vụ sử dụng    │
         │                                                                  │
         │  Admin vô hiệu hoá hoặc có bản mới thay thế                     │
         ▼                                                                  │
 [INACTIVE / ARCHIVED] ──────────────────────────────────────────────────┘
      (Lưu trữ, không dùng nữa; admin có thể reactivate nếu cần)
```

---

## Mô tả từng trạng thái

| Trạng thái | Ý nghĩa | Ai có thể thao tác |
|---|---|---|
| `DRAFT` | Bản nháp đang soạn thảo | Admin |
| `READY_FOR_TEST` | Đã hoàn thiện thiết kế, sẵn sàng đưa vào môi trường test | Admin |
| `TESTING` | Đang trong giai đoạn thử nghiệm có kiểm soát | Admin + Tester |
| `READY_FOR_PRODUCTION` | Đã test xong, không còn lỗi mở, chờ publish | Admin |
| `ACTIVE` | Đang hoạt động chính thức | User nghiệp vụ (chỉ đọc / thực hiện) |
| `INACTIVE` | Tạm tắt, không nhận hồ sơ mới | Admin (có thể reactivate) |
| `ARCHIVED` | Lưu trữ vĩnh viễn, không dùng lại | Admin (chỉ xem) |

---

## Giai đoạn thử nghiệm (Testing Period)

```
Bắt đầu TESTING
    │
    ├── Thời gian thử nghiệm: [StartDate] → [EndDate]  (admin cấu hình)
    │
    ├── Trong giai đoạn này:
    │     ├── Tester / admin vận hành hồ sơ thử
    │     ├── Ghi nhận lỗi / phản ánh vào Issue Log
    │     └── Mỗi issue có trạng thái: OPEN / IN_PROGRESS / RESOLVED / CLOSED
    │
    └── Điều kiện chuyển sang READY_FOR_PRODUCTION:
          ├── [EndDate] đã qua  (hết thời gian thử nghiệm)
          └── Tất cả issues đều ở trạng thái RESOLVED hoặc CLOSED
                  (không còn issue OPEN / IN_PROGRESS)
```

### Quy tắc về người dùng trong giai đoạn TESTING

- Workflow ở trạng thái `TESTING` **không được áp dụng cho người dùng cuối thực tế**.
- Chỉ **tester hoặc nhóm user được chỉ định** mới được thao tác.
- Nếu test trong production, phải **giới hạn phạm vi user** rõ ràng.
- Chỉ khi workflow ở trạng thái `ACTIVE` thì **người dùng cuối thực tế** mới được phép sử dụng.

---

## Quyền của Admin theo trạng thái

| Trạng thái hiện tại | Hành động Admin được phép |
|---|---|
| `DRAFT` | Chỉnh sửa thiết kế; xóa; chuyển sang `READY_FOR_TEST` |
| `READY_FOR_TEST` | Bắt đầu giai đoạn test (chuyển sang `TESTING`) |
| `TESTING` | Chỉnh sửa (nếu cần fix lỗi); kết thúc test khi đủ điều kiện |
| `READY_FOR_PRODUCTION` | Publish (chuyển sang `ACTIVE`) |
| `ACTIVE` | Clone tạo bản mới; vô hiệu hoá |
| `INACTIVE` | Reactivate; lưu trữ vĩnh viễn |
| `ARCHIVED` | Chỉ xem lịch sử |

---

*Xem tiếp: [06 · Issue Log](./06-issue-log.md)*
