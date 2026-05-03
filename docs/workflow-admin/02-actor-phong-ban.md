# 02 · Actors & Mô hình Phòng ban

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## Các đối tượng sử dụng (Actors)

### Ma trận Actor – Vai trò – Phạm vi sử dụng

| Đối tượng | Vai trò chính | Được sử dụng khi Workflow ở trạng thái |
|---|---|---|
| **Admin** | Cấu hình workflow, quản lý phòng ban & SLA, xem báo cáo | `DRAFT`, `READY_FOR_TEST`, `TESTING`, `READY_FOR_PRODUCTION` |
| **Tester** | Kiểm thử workflow, ghi nhận lỗi, xác nhận fix | `TESTING` |
| **Cán bộ tiếp nhận** | Tiếp nhận hồ sơ, kiểm tra ban đầu, chuyển bộ phận | `ACTIVE` |
| **Chuyên viên xử lý** | Thẩm định, xử lý nghiệp vụ, trình duyệt | `ACTIVE` |
| **Người phê duyệt** | Duyệt / từ chối / ký duyệt hồ sơ | `ACTIVE` |
| **Bộ phận trả kết quả** | Xác nhận hoàn tất, phát hành kết quả | `ACTIVE` |
| **Người dùng cuối / Công dân** | Nộp hồ sơ, nhận yêu cầu bổ sung, theo dõi trạng thái, nhận kết quả | `ACTIVE` (chỉ khi workflow đã publish chính thức) |
| **Quản trị / Giám sát** | Giám sát vận hành, xử lý fallback & escalation | Mọi trạng thái |

> **Lưu ý quan trọng:** Workflow ở trạng thái `TESTING` **không được áp dụng cho người dùng cuối thực tế**. Chỉ tester hoặc nhóm user được chỉ định mới được thao tác.

---

## Mô hình Phòng ban

```
Người dân / Người dùng cuối
   │
   ▼
┌─────────────────────────────────┐
│  A. Bộ phận Tiếp nhận / Một cửa │
│  DeptType: RECEPTION            │
└──────────────┬──────────────────┘
               │  (hồ sơ hợp lệ)
               ▼
┌─────────────────────────────────┐
│  B. Phòng Chuyên môn            │
│  DeptType: SPECIALIZED          │
└──────────────┬──────────────────┘
               │  (cần phối hợp)
               ├──────────────────────────────────────┐
               │                                      ▼
               │                    ┌─────────────────────────────────┐
               │                    │  C. Phòng/Ban Liên quan         │
               │                    │  DeptType: COORDINATING         │
               │                    └──────────────┬──────────────────┘
               │                                   │ (kết quả phối hợp)
               │◄──────────────────────────────────┘
               │
               │  (trình phê duyệt)
               ▼
┌─────────────────────────────────┐
│  D. Lãnh đạo / Cấp phê duyệt   │
│  Role: APPROVER / LEADER        │
└──────────────┬──────────────────┘
               │  (đã duyệt)
               ▼
┌─────────────────────────────────┐
│  E. Bộ phận Trả kết quả         │
│  DeptType: RETURN_DESK          │
└──────────────┬──────────────────┘
               │
               ▼
           Người dân nhận kết quả

Luồng ngang:
┌─────────────────────────────────┐
│  F. Quản trị / Giám sát         │
│  Role: SYSTEM_ADMIN / MONITOR   │
│  (giám sát toàn bộ, fallback,   │
│   escalation, cấu hình hệ thống)│
└─────────────────────────────────┘
```

---

## Chi tiết từng bộ phận

| # | Bộ phận | Vai trò | Khi nhận notification |
|---|---|---|---|
| A | Tiếp nhận / Một cửa | Nhận, kiểm tra ban đầu, chuyển hồ sơ | Hồ sơ mới; hồ sơ bị trả; hồ sơ hoàn tất cần trả dân |
| B | Phòng Chuyên môn | Thẩm định, xử lý nghiệp vụ, trình duyệt | Hồ sơ được chuyển đến; bị lãnh đạo trả lại; cận hạn |
| C | Phòng/Ban Liên quan | Phối hợp, xác minh, bổ sung ý kiến | Có yêu cầu phối hợp mới; nhắc quá hạn phản hồi |
| D | Lãnh đạo phê duyệt | Phê duyệt / từ chối / ký duyệt | Hồ sơ trình duyệt; hồ sơ tồn đọng; cận hạn |
| E | Trả kết quả | Xác nhận hoàn tất, phát hành kết quả | Hồ sơ hoàn tất sẵn sàng trả |
| F | Quản trị / Giám sát | Giám sát vận hành, fallback, escalation | Cảnh báo hệ thống; backlog bất thường; lỗi luồng |

---

## Chiến lược thông báo theo loại bộ phận

| Bộ phận | Thông báo theo |
|---|---|
| Tiếp nhận (A) | **Phòng ban** – queue chung chưa assign |
| Chuyên môn (B) | **Người được phân công** hoặc **role** |
| Phối hợp (C) | **Phòng ban** theo từng yêu cầu phối hợp |
| Phê duyệt (D) | **Role** (approver / leader) |
| Quá hạn (mọi bước) | **Người xử lý hiện tại + Trưởng bộ phận** |

---

## Hành trình của Người dùng cuối / Công dân

Workflow từ góc nhìn người dùng cuối trải qua các trạng thái:

1. **Tạo / nộp hồ sơ**
2. **Chờ tiếp nhận**
3. **Bị yêu cầu bổ sung** _(nếu có)_
4. **Đã tiếp nhận**
5. **Đang xử lý**
6. **Đang chờ phê duyệt**
7. **Đã phê duyệt / Bị từ chối**
8. **Đã có kết quả**
9. **Đã trả kết quả**

---

*Xem tiếp: [03 · Thành phần Workflow](./03-thanh-phan-workflow.md)*
