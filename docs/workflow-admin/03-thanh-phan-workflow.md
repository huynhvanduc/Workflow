# 03 · Các thành phần của Workflow

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## Quan hệ giữa các thành phần

```
WorkflowDefinition
    │
    ├── Step (1..n)
    │     ├── Action (1..n)
    │     ├── AssignmentRule (1..1)            ← chỉ dùng khi IsParallel = false
    │     ├── ParallelApprovalBranch (0..n)    ← chỉ dùng khi IsParallel = true
    │     ├── NotificationRule (0..n)
    │     └── SlaConfig (0..1)
    │
    └── Transition (0..n)  [từ Step → Step, kèm điều kiện]
```

---

## 3.1 WorkflowDefinition – Định nghĩa Workflow

Là "bản thiết kế" tổng thể cho một loại quy trình xử lý hồ sơ.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh duy nhất |
| `Code` | Mã nghiệp vụ (vd: `HO_SO_XAY_DUNG`) |
| `Name` | Tên hiển thị |
| `Version` | Phiên bản (tăng khi tạo bản mới từ workflow đang active) |
| `Status` | Trạng thái vòng đời (xem [05 · Vòng đời](./05-vong-doi-workflow.md)) |
| `DocumentTypeId` | Loại hồ sơ áp dụng |
| `EffectiveFrom` | Ngày bắt đầu hiệu lực |
| `EffectiveTo` | Ngày kết thúc hiệu lực (nếu có) |
| `CreatedBy` | Admin tạo |

---

## 3.2 Step – Bước xử lý

Mỗi workflow có nhiều bước nối tiếp nhau. Mỗi bước đại diện cho một giai đoạn xử lý cụ thể.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `WorkflowDefinitionId` | Thuộc workflow nào |
| `Name` | Tên bước (vd: "Tiếp nhận", "Thẩm định", "Phê duyệt") |
| `StepType` | Loại bước: `Start`, `Process`, `Approval`, `End` |
| `Order` | Thứ tự trong luồng |
| `IsParallel` | `true` = bước phê duyệt song song; `false` = bước thông thường |
| `CompletionRule` | *(chỉ dùng khi `IsParallel = true`)* Điều kiện hoàn thành: `ALL_APPROVED` / `MAJORITY` / `ANY_ONE` |
| `RejectionPolicy` | *(chỉ dùng khi `IsParallel = true`)* Xử lý khi có nhánh từ chối: `FAIL_FAST` / `WAIT_ALL` |
| `SlaConfig` | Cấu hình SLA gắn với bước này (với bước song song = SLA của cả bước, không phải từng nhánh) |

### Giải thích CompletionRule

| Giá trị | Ý nghĩa |
|---|---|
| `ALL_APPROVED` | Tất cả các nhánh đều phải phê duyệt thì bước mới hoàn thành |
| `MAJORITY` | Hơn một nửa số nhánh phê duyệt là đủ |
| `ANY_ONE` | Chỉ cần một nhánh phê duyệt là bước hoàn thành |

### Giải thích RejectionPolicy

| Giá trị | Ý nghĩa |
|---|---|
| `FAIL_FAST` | Ngay khi một nhánh từ chối → toàn bộ bước thất bại ngay lập tức, các nhánh còn lại dừng lại |
| `WAIT_ALL` | Tiếp tục chờ tất cả nhánh phản hồi, sau đó mới đánh giá kết quả cuối cùng |

---

## 3.2.1 ParallelApprovalBranch – Nhánh phê duyệt song song

*(Chỉ tồn tại khi bước có `IsParallel = true`)*

Mỗi nhánh đại diện cho một bộ phận / vai trò cần phê duyệt độc lập trong cùng một bước.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `StepId` | Thuộc bước nào |
| `BranchName` | Tên nhánh (vd: "Phòng Tài nguyên", "Phòng Quy hoạch", "Phòng PCCC") |
| `Order` | Thứ tự hiển thị |
| `AssignType` | `DEPARTMENT` / `ROLE` / `USER` – ai xử lý nhánh này |
| `TargetDepartmentCode` | Phòng ban đích (nếu `AssignType = DEPARTMENT`) |
| `TargetRoleCode` | Vai trò đích (nếu `AssignType = ROLE`) |
| `TargetUserId` | User cụ thể (nếu `AssignType = USER`) |
| `SlaHours` | SLA riêng cho nhánh này (tính từ lúc bước bắt đầu) |

### Ví dụ cấu hình bước song song

```
Bước "Phê duyệt liên phòng" (IsParallel = true, CompletionRule = ALL_APPROVED, RejectionPolicy = FAIL_FAST)
  ├── Nhánh 1: Phòng Tài nguyên & Môi trường  (SlaHours: 48)
  ├── Nhánh 2: Phòng Quy hoạch – Kiến trúc    (SlaHours: 48)
  └── Nhánh 3: Phòng Cảnh sát PCCC            (SlaHours: 24)
```

### Luồng thực thi khi bước song song bắt đầu

```
Bước song song bắt đầu
    │
    ├── Tạo đồng thời ParallelApprovalState cho mỗi nhánh (Status = PENDING)
    ├── Gửi thông báo cho tất cả các nhánh cùng lúc
    │
    ├── [Nhánh 1 phê duyệt]  → Status = APPROVED
    ├── [Nhánh 2 phê duyệt]  → Status = APPROVED
    └── [Nhánh 3 phê duyệt]  → Status = APPROVED
              │
              ▼ (engine kiểm tra CompletionRule sau mỗi nhánh phản hồi)
    ┌─ CompletionRule thỏa mãn? ─┐
    │ Có                         │ Không
    ▼                             ▼
Bước hoàn thành               Tiếp tục chờ
→ Chuyển sang bước tiếp theo
```

---

## 3.3 Action – Hành động trong bước

Là các thao tác người dùng có thể thực hiện tại một bước.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `StepId` | Thuộc bước nào |
| `Code` | Mã hành động (vd: `APPROVE`, `REJECT`, `REQUEST_SUPPLEMENT`) |
| `Name` | Tên hiển thị |
| `RequiredFields` | Danh sách trường bắt buộc nhập khi thực hiện |
| `ConfirmationMessage` | Thông báo xác nhận trước khi thực hiện |

### Hành động điển hình

| Mã | Tên | Mô tả |
|---|---|---|
| `RECEIVE` | Tiếp nhận | Tiếp nhận hồ sơ hợp lệ |
| `REQUEST_SUPPLEMENT` | Yêu cầu bổ sung | Yêu cầu người dân bổ sung tài liệu |
| `TRANSFER` | Chuyển bộ phận | Chuyển sang bộ phận / bước khác |
| `ASSIGN` | Phân công | Phân công chuyên viên xử lý |
| `REQUEST_COLLABORATION` | Yêu cầu phối hợp | Xin ý kiến phòng liên quan |
| `SUBMIT_FOR_APPROVAL` | Trình duyệt | Trình hồ sơ lên lãnh đạo phê duyệt |
| `APPROVE` | Phê duyệt | Duyệt hồ sơ |
| `REJECT` | Từ chối | Từ chối / trả lại bước trước |
| `RETURN` | Trả lại | Trả lại bước trước kèm lý do |
| `CLOSE` | Đóng hồ sơ | Trả kết quả cho người dân |

---

## 3.4 Transition – Quy tắc chuyển bước

Xác định bước tiếp theo sau khi thực hiện một hành động, có thể kèm điều kiện.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `FromStepId` | Bước xuất phát |
| `ToStepId` | Bước đích |
| `TriggerActionCode` | Hành động kích hoạt chuyển |
| `Condition` | Điều kiện (expression) để chuyển; bỏ trống = luôn chuyển |
| `Priority` | Ưu tiên khi có nhiều transition cùng action |

### Ví dụ Transition

```
FromStep: Thẩm định  →  Action: APPROVE  →  ToStep: Phê duyệt   (Condition: Amount > 500M)
FromStep: Thẩm định  →  Action: APPROVE  →  ToStep: Hoàn tất    (Condition: Amount <= 500M)
FromStep: Thẩm định  →  Action: REJECT   →  ToStep: Trả kết quả
```

---

## 3.5 AssignmentRule – Quy tắc phân công

> **Lưu ý:** AssignmentRule chỉ áp dụng cho bước **thông thường** (`IsParallel = false`). Với bước song song (`IsParallel = true`), quy tắc phân công được cấu hình riêng cho từng **ParallelApprovalBranch**.

Xác định ai / bộ phận nào chịu trách nhiệm xử lý tại mỗi bước.

| Thuộc tính | Mô tả |
|---|---|
| `StepId` | Áp dụng cho bước nào |
| `AssignType` | `DEPARTMENT` / `ROLE` / `USER` / `ROUND_ROBIN` / `LOAD_BALANCE` |
| `TargetDepartmentCode` | Phòng ban đích (nếu `AssignType = DEPARTMENT`) |
| `TargetRoleCode` | Vai trò đích (nếu `AssignType = ROLE`) |
| `TargetUserId` | User cụ thể (nếu `AssignType = USER`) |
| `FallbackRule` | Quy tắc dự phòng khi không tìm thấy người |

### Ví dụ AssignmentRule

```
Bước "Tiếp nhận"  → AssignType: DEPARTMENT  → TargetDept: RECEPTION
Bước "Thẩm định"  → AssignType: ROLE        → TargetRole: SPECIALIST
Bước "Phê duyệt"  → AssignType: ROLE        → TargetRole: APPROVER
```

---

## 3.6 NotificationRule – Quy tắc thông báo

Cấu hình khi nào, gửi cho ai, qua kênh nào.

| Thuộc tính | Mô tả |
|---|---|
| `StepId` / `ActionCode` | Kích hoạt khi bước hoặc hành động này xảy ra |
| `TriggerEvent` | `ON_ENTER`, `ON_ACTION`, `ON_OVERDUE`, `ON_ESCALATION` |
| `RecipientType` | `ASSIGNED_USER`, `DEPARTMENT`, `ROLE`, `SPECIFIC_USER`, `SUBMITTER` |
| `RecipientRef` | Tham chiếu cụ thể (dept code, role code, user id) |
| `Channel` | `PUSH_NOTIFICATION` / `EMAIL` / `SMS` / `IN_APP` |
| `TemplateCode` | Mã mẫu nội dung thông báo |
| `DelayMinutes` | Độ trễ gửi (0 = ngay lập tức) |

> **Kênh ưu tiên hiện tại:** Push Notification.

### Thông báo hướng tới Người dùng cuối

| Sự kiện | Nội dung thông báo |
|---|---|
| Nộp hồ sơ thành công | "Hồ sơ của bạn đã được tiếp nhận" |
| Yêu cầu bổ sung | "Hồ sơ cần bổ sung: [danh sách tài liệu]" |
| Hồ sơ đã tiếp nhận chính thức | "Hồ sơ đã được tiếp nhận, đang xử lý" |
| Đang xử lý | "Hồ sơ đang trong quá trình thẩm định" |
| Đã phê duyệt | "Hồ sơ đã được phê duyệt" |
| Bị từ chối | "Hồ sơ không được chấp thuận. Lý do: [...]" |
| Có kết quả | "Hồ sơ đã có kết quả, vui lòng đến nhận" |
| Nhắc bổ sung đúng hạn | "Bạn còn [X] ngày để bổ sung hồ sơ" |

---

## 3.7 SLA / Overdue / Escalation

Cấu hình thời hạn và leo thang khi quá hạn, gắn vào từng bước.

| Thuộc tính | Mô tả |
|---|---|
| `DurationHours` | Thời gian tối đa (giờ) cho phép xử lý tại bước |
| `WarningBeforeHours` | Gửi cảnh báo trước bao nhiêu giờ |
| `OverdueAction` | Hành động khi quá hạn: `NOTIFY` / `ESCALATE` / `AUTO_TRANSFER` |
| `EscalationTargetRole` | Leo thang lên vai trò nào |
| `EscalationLevel` | Mức leo thang (1 → 2 → 3 ...) |

### Luồng SLA

```
Bắt đầu bước
  │
  ├─ [Còn trong SLA]  → Xử lý bình thường
  │
  ├─ [Sắp hết SLA]    → Gửi cảnh báo cho người xử lý + trưởng bộ phận
  │
  └─ [Quá SLA]        → Leo thang lên cấp trên + ghi nhận overdue
```

---

*Xem tiếp: [04 · Tình huống nghiệp vụ](./04-tinh-huong-nghiep-vu.md)*
