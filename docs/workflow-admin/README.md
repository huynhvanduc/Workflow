# Tài liệu Phân tích nghiệp vụ: Quản trị Workflow xử lý hồ sơ

> **Trạng thái tài liệu:** Phân tích – Use Case  
> **Tech stack tham chiếu:** .NET Backend · React Frontend  
> **Kênh thông báo ưu tiên:** Push Notification  
> **Cập nhật lần cuối:** 2026-05

---

## Cấu trúc tài liệu

Mỗi file mô tả **một khía cạnh cụ thể** của nghiệp vụ để dễ tra cứu và bảo trì.

| # | File | Nội dung |
|---|------|---------|
| 1 | [01-tong-quan.md](./01-tong-quan.md) | Bài toán tổng quát & mục tiêu hệ thống |
| 2 | [02-actor-phong-ban.md](./02-actor-phong-ban.md) | Các đối tượng sử dụng & mô hình phòng ban |
| 3 | [03-thanh-phan-workflow.md](./03-thanh-phan-workflow.md) | Các thành phần kỹ thuật của Workflow (Step, Action, Transition, Assignment, Notification, SLA) |
| 4 | [04-tinh-huong-nghiep-vu.md](./04-tinh-huong-nghiep-vu.md) | 10 tình huống nghiệp vụ đặc trưng |
| 5 | [05-vong-doi-workflow.md](./05-vong-doi-workflow.md) | Vòng đời Workflow & giai đoạn thử nghiệm |
| 6 | [06-issue-log.md](./06-issue-log.md) | Ghi nhận lỗi / phản ánh & trạng thái xử lý |
| 7 | [07-checklist-nguoi-dung.md](./07-checklist-nguoi-dung.md) | Checklist cho người dùng cuối, tester và vai trò nội bộ |
| 8 | [08-nguyen-tac-quan-tri.md](./08-nguyen-tac-quan-tri.md) | Nguyên tắc quản trị & tiêu chí kích hoạt |

---

## Sơ đồ tổng hợp nhanh

### Luồng hồ sơ – Phòng ban – Thông báo

```
[Người dân / Người dùng cuối]
    │ nộp hồ sơ
    ▼
[RECEPTION] ──── notify RECEPTION khi hồ sơ mới
    │ chuyển hồ sơ hợp lệ
    ▼
[SPECIALIZED] ── notify người được assign
    │ cần phối hợp
    ├──────────────────►[COORDINATING] ── notify khi có yêu cầu phối hợp
    │◄──────────────────
    │ trình duyệt
    ▼
[APPROVER/LEADER] ── notify lãnh đạo khi có hồ sơ trình
    │ phê duyệt          │ từ chối
    │                    └────────►[SPECIALIZED] ── notify người xử lý
    ▼
[RETURN_DESK] ── notify RETURN_DESK + notify người dân
    │
    ▼
[Người dân nhận kết quả]

[SYSTEM_ADMIN] ── nhận cảnh báo fallback, escalation, lỗi luồng (xuyên suốt)
```

### Vòng đời Workflow (tóm tắt)

```
DRAFT → READY_FOR_TEST → TESTING → READY_FOR_PRODUCTION → ACTIVE → INACTIVE/ARCHIVED
                              ▲
                              │  Ghi nhận & xử lý Issue Log trong giai đoạn này
```

### Quan hệ giữa các thành phần

```
WorkflowDefinition
    │
    ├── Step (1..n)
    │     ├── Action (1..n)
    │     ├── AssignmentRule (1..1)
    │     ├── NotificationRule (0..n)
    │     └── SlaConfig (0..1)
    │
    └── Transition (0..n)  [từ Step → Step, kèm điều kiện]
```

---

*Tài liệu này phản ánh kết quả phân tích use case giai đoạn khởi đầu. Nội dung sẽ được bổ sung chi tiết khi team bước vào giai đoạn thiết kế kỹ thuật và triển khai.*
