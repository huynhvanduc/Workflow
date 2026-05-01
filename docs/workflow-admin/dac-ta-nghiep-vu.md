# Đặc tả nghiệp vụ: Quản trị Workflow xử lý hồ sơ

> **Trạng thái tài liệu:** Phân tích – Use Case  
> **Tech stack tham chiếu:** .NET Backend · React Frontend  
> **Kênh thông báo ưu tiên:** Push Notification  
> **Cập nhật lần cuối:** 2026-05

---

## Mục lục

1. [Bài toán tổng quát](#1-bài-toán-tổng-quát)
2. [Quyền cấu hình Workflow của Admin](#2-quyền-cấu-hình-workflow-của-admin)
3. [Các thành phần chính của Workflow](#3-các-thành-phần-chính-của-workflow)
4. [Phòng ban / Bộ phận tham gia xử lý](#4-phòng-ban--bộ-phận-tham-gia-xử-lý)
5. [Tình huống nghiệp vụ đặc trưng](#5-tình-huống-nghiệp-vụ-đặc-trưng)
6. [Vòng đời quản trị Workflow](#6-vòng-đời-quản-trị-workflow)
7. [Nguyên tắc quản trị](#7-nguyên-tắc-quản-trị)
8. [Sơ đồ tổng hợp](#8-sơ-đồ-tổng-hợp)

---

## 1. Bài toán tổng quát

Hệ thống hiện tại xử lý hồ sơ theo luồng cứng (hard-coded), dẫn đến các vấn đề:

- Khi quy trình nghiệp vụ thay đổi, cần sửa code và triển khai lại.
- Thông báo cho cán bộ/bộ phận phụ thuộc thủ công, dễ sót và chậm trễ.
- Không phản ánh được sự đa dạng giữa các loại hồ sơ (mỗi loại có luồng riêng).

**Mục tiêu:** Xây dựng hệ thống cho phép **admin cấu hình linh hoạt** quy trình xử lý hồ sơ (workflow), bao gồm các bước, điều kiện chuyển trạng thái, phân công xử lý và quy tắc thông báo – thay thế hoàn toàn cho cách hard-code hiện tại.

---

## 2. Quyền cấu hình Workflow của Admin

Admin (quản trị hệ thống) có toàn quyền:

| Quyền | Mô tả |
|---|---|
| Tạo mới workflow | Định nghĩa quy trình cho từng loại hồ sơ |
| Chỉnh sửa bản nháp | Thay đổi workflow ở trạng thái DRAFT hoặc TESTING |
| Tạo phiên bản mới | Clone workflow đang ACTIVE để chỉnh sửa, không động chạm bản đang chạy |
| Kích hoạt workflow | Đưa workflow qua vòng đời đến ACTIVE sau khi test thành công |
| Vô hiệu hoá / lưu trữ | Tắt workflow khi không còn dùng |
| Quản lý bộ phận & vai trò | Định nghĩa phòng ban, vị trí, quy tắc phân công |
| Cấu hình SLA | Đặt thời hạn và leo thang cho từng bước |
| Xem báo cáo vận hành | Theo dõi hồ sơ tồn đọng, quá hạn, lỗi luồng |

> **Nguyên tắc cốt lõi:** Admin thiết lập quy trình; người dùng nghiệp vụ chỉ thao tác theo quy trình đã được duyệt.

---

## 3. Các thành phần chính của Workflow

### 3.1 WorkflowDefinition – Định nghĩa Workflow

Là "bản thiết kế" tổng thể cho một loại quy trình.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh duy nhất |
| `Code` | Mã nghiệp vụ (vd: `HO_SO_XAY_DUNG`) |
| `Name` | Tên hiển thị |
| `Version` | Phiên bản (tăng khi tạo bản mới từ workflow đang active) |
| `Status` | Trạng thái vòng đời (xem mục 6) |
| `DocumentTypeId` | Loại hồ sơ áp dụng |
| `EffectiveFrom` | Ngày bắt đầu hiệu lực |
| `EffectiveTo` | Ngày kết thúc hiệu lực (nếu có) |
| `CreatedBy` | Admin tạo |

---

### 3.2 Step – Bước xử lý

Mỗi workflow có nhiều bước nối tiếp nhau.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `WorkflowDefinitionId` | Thuộc workflow nào |
| `Name` | Tên bước (vd: "Tiếp nhận", "Thẩm định", "Phê duyệt") |
| `StepType` | Loại bước: `Start`, `Process`, `Approval`, `End` |
| `Order` | Thứ tự trong luồng |
| `IsParallel` | Cho phép xử lý song song không |
| `SlaConfig` | Cấu hình SLA gắn với bước này |

---

### 3.3 Action – Hành động trong bước

Là các thao tác người dùng có thể thực hiện tại một bước.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `StepId` | Thuộc bước nào |
| `Code` | Mã hành động (vd: `APPROVE`, `REJECT`, `REQUEST_SUPPLEMENT`, `TRANSFER`) |
| `Name` | Tên hiển thị |
| `RequiredFields` | Danh sách trường bắt buộc nhập khi thực hiện |
| `ConfirmationMessage` | Thông báo xác nhận trước khi thực hiện |

Ví dụ hành động điển hình:

```
RECEIVE          – Tiếp nhận hồ sơ
REQUEST_SUPPLEMENT – Yêu cầu bổ sung hồ sơ
TRANSFER         – Chuyển sang bộ phận / bước khác
APPROVE          – Phê duyệt
REJECT           – Từ chối
RETURN           – Trả lại bước trước
CLOSE            – Đóng hồ sơ / trả kết quả
```

---

### 3.4 Transition – Quy tắc chuyển bước

Xác định bước tiếp theo sau khi thực hiện một hành động, có thể kèm điều kiện.

| Thuộc tính | Mô tả |
|---|---|
| `Id` | Định danh |
| `FromStepId` | Bước xuất phát |
| `ToStepId` | Bước đích |
| `TriggerActionCode` | Hành động kích hoạt chuyển |
| `Condition` | Điều kiện (expression) để chuyển, bỏ trống = luôn chuyển |
| `Priority` | Ưu tiên khi có nhiều transition cùng action |

Ví dụ:

```
FromStep: Thẩm định  →  Action: APPROVE  →  ToStep: Phê duyệt   (Condition: Amount > 500M)
FromStep: Thẩm định  →  Action: APPROVE  →  ToStep: Hoàn tất    (Condition: Amount <= 500M)
FromStep: Thẩm định  →  Action: REJECT   →  ToStep: Trả kết quả
```

---

### 3.5 AssignmentRule – Quy tắc phân công

Xác định ai/bộ phận nào chịu trách nhiệm xử lý tại mỗi bước.

| Thuộc tính | Mô tả |
|---|---|
| `StepId` | Áp dụng cho bước nào |
| `AssignType` | `DEPARTMENT` / `ROLE` / `USER` / `ROUND_ROBIN` / `LOAD_BALANCE` |
| `TargetDepartmentCode` | Phòng ban đích (nếu AssignType = DEPARTMENT) |
| `TargetRoleCode` | Vai trò đích (nếu AssignType = ROLE) |
| `TargetUserId` | User cụ thể (nếu AssignType = USER) |
| `FallbackRule` | Quy tắc dự phòng khi không tìm thấy người |

Ví dụ:

```
Bước "Tiếp nhận"  → AssignType: DEPARTMENT  → TargetDept: RECEPTION
Bước "Thẩm định"  → AssignType: ROLE        → TargetRole: SPECIALIST
Bước "Phê duyệt"  → AssignType: ROLE        → TargetRole: APPROVER
```

---

### 3.6 NotificationRule – Quy tắc thông báo

Cấu hình khi nào, gửi cho ai, qua kênh nào.

| Thuộc tính | Mô tả |
|---|---|
| `StepId` / `ActionCode` | Kích hoạt khi bước hoặc hành động này xảy ra |
| `TriggerEvent` | `ON_ENTER`, `ON_ACTION`, `ON_OVERDUE`, `ON_ESCALATION` |
| `RecipientType` | `ASSIGNED_USER`, `DEPARTMENT`, `ROLE`, `SPECIFIC_USER`, `SUBMITTER` |
| `RecipientRef` | Tham chiếu cụ thể (dept code, role code, user id) |
| `Channel` | `PUSH_NOTIFICATION` / `EMAIL` / `SMS` / `IN_APP` |
| `TemplateCode` | Mã mẫu nội dung thông báo |
| `DelayMinutes` | Độ trễ gửi (vd: 0 = ngay lập tức) |

Kênh ưu tiên hiện tại: **Push Notification**.

---

### 3.7 SLA / Overdue / Escalation

Cấu hình thời hạn và leo thang khi quá hạn, gắn vào từng bước.

| Thuộc tính | Mô tả |
|---|---|
| `DurationHours` | Thời gian tối đa (giờ) cho phép xử lý tại bước |
| `WarningBeforeHours` | Gửi cảnh báo trước bao nhiêu giờ |
| `OverdueAction` | Hành động khi quá hạn: `NOTIFY` / `ESCALATE` / `AUTO_TRANSFER` |
| `EscalationTargetRole` | Leo thang lên vai trò nào |
| `EscalationLevel` | Mức leo thang (1 → 2 → 3 ...) |

Luồng tổng quát:

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

## 4. Phòng ban / Bộ phận tham gia xử lý

```
Người dân
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
           Người dân

Luồng ngang:
┌─────────────────────────────────┐
│  F. Quản trị / Giám sát         │
│  Role: SYSTEM_ADMIN / MONITOR   │
│  (giám sát toàn bộ, fallback,   │
│   escalation, cấu hình hệ thống)│
└─────────────────────────────────┘
```

### Chi tiết từng bộ phận

| # | Bộ phận | Vai trò | Khi nhận notification |
|---|---|---|---|
| A | Tiếp nhận / Một cửa | Nhận, kiểm tra ban đầu, chuyển hồ sơ | Hồ sơ mới; hồ sơ bị trả; hồ sơ hoàn tất cần trả dân |
| B | Phòng Chuyên môn | Thẩm định, xử lý nghiệp vụ, trình duyệt | Hồ sơ được chuyển đến; bị lãnh đạo trả lại; cận hạn |
| C | Phòng/Ban Liên quan | Phối hợp, xác minh, bổ sung ý kiến | Có yêu cầu phối hợp mới; nhắc quá hạn phản hồi |
| D | Lãnh đạo phê duyệt | Phê duyệt / từ chối / ký duyệt | Hồ sơ trình duyệt; hồ sơ tồn đọng; cận hạn |
| E | Trả kết quả | Xác nhận hoàn tất, phát hành kết quả | Hồ sơ hoàn tất sẵn sàng trả |
| F | Quản trị / Giám sát | Giám sát vận hành, fallback, escalation | Cảnh báo hệ thống; backlog bất thường; lỗi luồng |

### Chiến lược thông báo theo loại bộ phận

| Bộ phận | Thông báo theo |
|---|---|
| Tiếp nhận (A) | **Phòng ban** – queue chung chưa assign |
| Chuyên môn (B) | **Người được phân công** hoặc **role** |
| Phối hợp (C) | **Phòng ban** theo từng yêu cầu phối hợp |
| Phê duyệt (D) | **Role** (approver / leader) |
| Quá hạn (mọi bước) | **Người xử lý hiện tại + Trưởng bộ phận** |

---

## 5. Tình huống nghiệp vụ đặc trưng

Dưới đây là 10 tình huống làm cơ sở thiết kế workflow và notification rule.

---

### TH-01: Hồ sơ mới nộp – Thông báo bộ phận tiếp nhận

**Mô tả:** Người dân nộp hồ sơ trực tuyến hoặc tại quầy.  
**Kết quả mong đợi:** Toàn bộ cán bộ tiếp nhận (hoặc trưởng bộ phận) nhận push notification ngay lập tức.  
**Trigger:** `ON_ENTER` bước Tiếp nhận.  
**Recipient:** `DeptType = RECEPTION`.

---

### TH-02: Hồ sơ thiếu thông tin – Yêu cầu bổ sung

**Mô tả:** Cán bộ tiếp nhận phát hiện hồ sơ chưa đủ, thực hiện action `REQUEST_SUPPLEMENT`.  
**Kết quả mong đợi:** Người dân (submitter) nhận thông báo kèm danh sách tài liệu cần bổ sung.  
**Trigger:** `ON_ACTION = REQUEST_SUPPLEMENT`.  
**Recipient:** `SUBMITTER`.

---

### TH-03: Chuyển hồ sơ sang Phòng chuyên môn

**Mô tả:** Sau khi hồ sơ hợp lệ, cán bộ tiếp nhận chuyển sang phòng chuyên môn.  
**Kết quả mong đợi:** Phòng chuyên môn (hoặc trưởng phòng) nhận thông báo có hồ sơ mới cần xử lý.  
**Trigger:** `ON_ENTER` bước Chuyên môn.  
**Recipient:** `DeptType = SPECIALIZED` hoặc `Role = SPECIALIST`.

---

### TH-04: Phân công chuyên viên xử lý

**Mô tả:** Trưởng phòng chuyên môn phân công một chuyên viên cụ thể.  
**Kết quả mong đợi:** Chuyên viên đó nhận thông báo hồ sơ được assign cho mình.  
**Trigger:** `ON_ACTION = ASSIGN`.  
**Recipient:** `ASSIGNED_USER`.

---

### TH-05: Yêu cầu phối hợp phòng liên quan

**Mô tả:** Chuyên môn cần xin ý kiến từ phòng liên quan (vd: Địa chính, Tư pháp).  
**Kết quả mong đợi:** Phòng liên quan nhận thông báo có yêu cầu phối hợp kèm deadline phản hồi.  
**Trigger:** `ON_ACTION = REQUEST_COLLABORATION`.  
**Recipient:** `TargetDepartment` được chỉ định trong action.

---

### TH-06: Trình lãnh đạo phê duyệt

**Mô tả:** Chuyên môn hoàn thiện hồ sơ, thực hiện action `SUBMIT_FOR_APPROVAL`.  
**Kết quả mong đợi:** Lãnh đạo có thẩm quyền nhận thông báo có hồ sơ chờ phê duyệt.  
**Trigger:** `ON_ENTER` bước Phê duyệt.  
**Recipient:** `Role = APPROVER` theo cấp thẩm quyền.

---

### TH-07: Lãnh đạo từ chối – Trả lại chuyên môn

**Mô tả:** Lãnh đạo phát hiện vấn đề, thực hiện action `REJECT`.  
**Kết quả mong đợi:** Chuyên môn (người phụ trách) nhận thông báo kèm lý do từ chối.  
**Trigger:** `ON_ACTION = REJECT`.  
**Recipient:** `ASSIGNED_USER` tại bước Chuyên môn trước đó.

---

### TH-08: Hồ sơ hoàn tất – Thông báo trả kết quả

**Mô tả:** Lãnh đạo phê duyệt, hồ sơ chuyển sang trạng thái hoàn tất.  
**Kết quả mong đợi:** Bộ phận trả kết quả nhận thông báo; người dân nhận thông báo hồ sơ đã xong.  
**Trigger:** `ON_ENTER` bước Trả kết quả.  
**Recipient:** `DeptType = RETURN_DESK` + `SUBMITTER`.

---

### TH-09: Hồ sơ sắp / đã quá hạn (SLA)

**Mô tả:** Hồ sơ chưa được xử lý xong trong thời hạn quy định.  
**Kết quả mong đợi:**  
- Trước 4 giờ hết hạn: cảnh báo cho người xử lý.  
- Khi quá hạn: leo thang thông báo lên trưởng bộ phận.  
- Nếu tiếp tục quá hạn (cấp 2): leo thang lên lãnh đạo.  
**Trigger:** `ON_OVERDUE`, `ON_ESCALATION`.  
**Recipient:** `ASSIGNED_USER` → `DEPT_MANAGER` → `LEADER`.

---

### TH-10: Không tìm thấy người xử lý (Fallback)

**Mô tả:** Đến bước phân công nhưng không có ai phù hợp (role trống, bộ phận không có nhân sự online).  
**Kết quả mong đợi:** Hệ thống gửi cảnh báo cho quản trị viên để can thiệp thủ công.  
**Trigger:** `ON_ASSIGNMENT_FAILURE`.  
**Recipient:** `Role = SYSTEM_ADMIN`.

---

## 6. Vòng đời quản trị Workflow

### 6.1 Sơ đồ trạng thái

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

### 6.2 Mô tả từng trạng thái

| Trạng thái | Ý nghĩa | Ai có thể thao tác |
|---|---|---|
| `DRAFT` | Bản nháp đang soạn thảo | Admin |
| `READY_FOR_TEST` | Đã hoàn thiện thiết kế, sẵn sàng đưa vào môi trường test | Admin |
| `TESTING` | Đang trong giai đoạn thử nghiệm có kiểm soát | Admin + Tester |
| `READY_FOR_PRODUCTION` | Đã test xong, không còn lỗi mở, chờ publish | Admin |
| `ACTIVE` | Đang hoạt động chính thức | User nghiệp vụ (chỉ đọc/thực hiện) |
| `INACTIVE` | Tạm tắt, không nhận hồ sơ mới | Admin (có thể reactivate) |
| `ARCHIVED` | Lưu trữ vĩnh viễn, không dùng lại | Admin (chỉ xem) |

### 6.3 Giai đoạn thử nghiệm (Testing Period)

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

### 6.4 Issue Log trong giai đoạn thử nghiệm

| Trường | Mô tả |
|---|---|
| `WorkflowDefinitionId` | Workflow liên quan |
| `Title` | Tiêu đề lỗi / phản ánh |
| `Description` | Mô tả chi tiết |
| `Severity` | `CRITICAL` / `MAJOR` / `MINOR` |
| `Status` | `OPEN` → `IN_PROGRESS` → `RESOLVED` → `CLOSED` |
| `ReportedBy` | Người ghi nhận |
| `ResolvedBy` | Người xử lý |
| `ResolvedAt` | Thời điểm giải quyết |

---

## 7. Nguyên tắc quản trị

1. **Workflow ACTIVE không sửa trực tiếp.**  
   Khi cần thay đổi, admin tạo phiên bản mới (clone) từ bản đang active. Bản gốc tiếp tục chạy cho đến khi bản mới được kích hoạt.

2. **Workflow mới phải qua giai đoạn test trước khi ACTIVE.**  
   Không cho phép chuyển thẳng từ DRAFT → ACTIVE.

3. **Còn lỗi mở (OPEN / IN_PROGRESS) thì không được publish.**  
   Hệ thống tự động chặn action "Publish to Production" nếu issue log còn issue chưa đóng.

4. **Hết thời gian thử nghiệm mới được xét publish.**  
   Không cho phép publish trước ngày `EndDate` của giai đoạn thử nghiệm.

5. **User nghiệp vụ chỉ sử dụng workflow ACTIVE.**  
   Các trạng thái khác (DRAFT, TESTING, ARCHIVED...) hoàn toàn ẩn với người dùng thông thường.

6. **Versioning rõ ràng.**  
   Mỗi lần tạo bản mới từ workflow active, số version tăng lên. Lịch sử tất cả các version được lưu trữ và có thể tra cứu.

7. **Audit log đầy đủ.**  
   Mọi thay đổi trạng thái của workflow (ai thay đổi, lúc nào, lý do) đều được ghi nhận.

---

## 8. Sơ đồ tổng hợp

### 8.1 Luồng hồ sơ – Phòng ban – Notification

```
[Người dân]
    │ nộp hồ sơ
    ▼
[RECEPTION] ──────────────── notify RECEPTION khi hồ sơ mới
    │ chuyển hồ sơ hợp lệ
    ▼
[SPECIALIZED] ─────────────── notify người được assign
    │ cần phối hợp
    ├──────────────────────────►[COORDINATING] ── notify khi có yêu cầu phối hợp
    │◄──────────────────────────     │ phản hồi
    │
    │ trình duyệt
    ▼
[APPROVER/LEADER] ─────────── notify lãnh đạo khi có hồ sơ trình
    │ duyệt                         │ từ chối
    │                               └──────────►[SPECIALIZED] ── notify người xử lý
    │ phê duyệt
    ▼
[RETURN_DESK] ─────────────── notify RETURN_DESK + notify người dân
    │
    ▼
[Người dân nhận kết quả]

[SYSTEM_ADMIN] ─────────────── nhận cảnh báo fallback, escalation, lỗi luồng (xuyên suốt)
```

### 8.2 Vòng đời workflow (tóm tắt)

```
DRAFT → READY_FOR_TEST → TESTING → READY_FOR_PRODUCTION → ACTIVE → INACTIVE/ARCHIVED
                              ▲
                              │  Ghi nhận & xử lý Issue Log trong giai đoạn này
                              │  Chỉ chuyển sang READY_FOR_PRODUCTION khi:
                              │    - Hết thời gian thử nghiệm
                              │    - Không còn issue OPEN/IN_PROGRESS
```

### 8.3 Quan hệ giữa các thành phần

```
WorkflowDefinition
    │
    ├── Step (1..n)
    │     ├── Action (1..n)
    │     ├── AssignmentRule (1..1)
    │     ├── NotificationRule (0..n)
    │     └── SlaConfig (0..1)
    │
    └── Transition (0..n)  [từ Step → Step, điều kiện]
```

---

*Tài liệu này phản ánh kết quả phân tích use case giai đoạn khởi đầu. Nội dung sẽ được bổ sung chi tiết khi team bước vào giai đoạn thiết kế kỹ thuật và triển khai.*
