# 11. Audit Log – Nhật ký thao tác

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này mô tả yêu cầu nghiệp vụ và thiết kế logic cho tính năng **Audit Log** – nhật ký ghi lại toàn bộ hành động của **nhân viên nghiệp vụ (cán bộ)** khi xử lý hồ sơ, và hành động của **admin** khi cấu hình workflow.

Audit Log đáp ứng đồng thời ba nhu cầu:

1. **Tuân thủ pháp lý:** Hồ sơ hành chính công cần truy xuất nguồn gốc "ai làm gì, lúc nào, tại bước nào".
2. **Giám sát vận hành:** Admin/Lãnh đạo có thể theo dõi tiến trình xử lý, phát hiện bất thường.
3. **Kiểm soát cấu hình:** Lưu lịch sử thay đổi workflow và hỗ trợ so sánh diff giữa các phiên bản.

---

## Phạm vi

| Loại nhật ký | Đối tượng ghi | Mô tả |
|---|---|---|
| **Hồ sơ (Case Audit Log)** | Nhân viên nghiệp vụ | Ghi hành động xử lý hồ sơ: tiếp nhận, chuyển bước, phê duyệt, từ chối... |
| **Cấu hình Workflow (Config Audit Log)** | Admin | Ghi thao tác cấu hình: tạo, sửa, publish, deactivate workflow |

> **Ưu tiên triển khai:** Case Audit Log cho nhân viên nghiệp vụ được triển khai trước.

---

## 1. Case Audit Log – Nhật ký xử lý hồ sơ

### 1.1 Mục đích

Ghi lại **mọi hành động** mà cán bộ nghiệp vụ thực hiện trên một hồ sơ trong suốt vòng đời của hồ sơ đó. Dữ liệu này:

- Giúp lãnh đạo tra cứu "ai đã làm gì với hồ sơ số X".
- Hỗ trợ kiểm toán nội bộ và thanh tra.
- Làm căn cứ giải quyết tranh chấp hoặc khiếu nại.
- Cung cấp dữ liệu cho báo cáo vận hành và phân tích hiệu suất.

---

### 1.2 Các hành động được ghi

| Mã hành động | Tên hiển thị | Khi nào ghi |
|---|---|---|
| `CASE_CREATED` | Tạo hồ sơ | Người dân/cán bộ nộp hồ sơ mới |
| `CASE_RECEIVED` | Tiếp nhận hồ sơ | Cán bộ tiếp nhận xác nhận nhận hồ sơ |
| `CASE_ASSIGNED` | Phân công xử lý | Hệ thống hoặc trưởng phòng phân công cho cán bộ |
| `CASE_REASSIGNED` | Phân công lại | Chuyển hồ sơ sang cán bộ khác |
| `STEP_ENTERED` | Vào bước mới | Hồ sơ chuyển sang bước tiếp theo |
| `ACTION_PERFORMED` | Thực hiện hành động | Cán bộ thực hiện action (APPROVE, REJECT, TRANSFER, ...) |
| `SUPPLEMENT_REQUESTED` | Yêu cầu bổ sung | Cán bộ yêu cầu người dân bổ sung hồ sơ |
| `SUPPLEMENT_RECEIVED` | Nhận bổ sung | Hồ sơ bổ sung được nộp |
| `COLLABORATION_REQUESTED` | Yêu cầu phối hợp | Gửi yêu cầu sang phòng ban khác |
| `COLLABORATION_RESPONDED` | Phản hồi phối hợp | Phòng ban liên quan phản hồi |
| `CASE_APPROVED` | Phê duyệt | Lãnh đạo phê duyệt |
| `CASE_REJECTED` | Từ chối | Lãnh đạo hoặc cán bộ từ chối |
| `CASE_RETURNED` | Trả lại bước trước | Hồ sơ bị trả lại để xử lý lại |
| `CASE_COMPLETED` | Hoàn tất | Hồ sơ hoàn tất, chuyển trả kết quả |
| `CASE_CLOSED` | Đóng hồ sơ | Hồ sơ được đóng chính thức |
| `SLA_WARNING` | Cảnh báo SLA | Hệ thống tự động – hồ sơ sắp quá hạn |
| `SLA_BREACHED` | Vi phạm SLA | Hệ thống tự động – hồ sơ đã quá hạn |
| `ESCALATED` | Leo thang | Hệ thống tự động – leo thang lên cấp trên |
| `COMMENT_ADDED` | Ghi chú nội bộ | Cán bộ thêm ghi chú/bình luận nội bộ |
| `ATTACHMENT_ADDED` | Đính kèm tài liệu | Thêm tài liệu vào hồ sơ |
| `ATTACHMENT_REMOVED` | Xóa tài liệu | Xóa tài liệu khỏi hồ sơ |

---

### 1.3 Cấu trúc dữ liệu – CaseAuditLog

```
CaseAuditLog {
    Id                 : Guid          // Định danh duy nhất của bản ghi log
    CaseId             : Guid          // Hồ sơ liên quan
    CaseCode           : string        // Mã hồ sơ (hiển thị)
    WorkflowId         : Guid          // Workflow đang áp dụng
    WorkflowVersion    : string        // Phiên bản workflow (vd: v1.2)
    StepId             : Guid?         // Bước đang xử lý (nếu có)
    StepName           : string?       // Tên bước (snapshot, không foreign key)
    ActionCode         : string        // Mã hành động (xem bảng 1.2)
    ActionLabel        : string        // Tên hiển thị của hành động
    PerformedBy        : Guid?         // User thực hiện (null nếu hệ thống tự động)
    PerformedByName    : string?       // Tên đầy đủ (snapshot)
    PerformedByRole    : string?       // Vai trò tại thời điểm thực hiện (snapshot)
    PerformedByDept    : string?       // Phòng ban (snapshot)
    PerformedAt        : datetime      // Thời điểm thực hiện (UTC)
    IsSystemGenerated  : bool          // true nếu do hệ thống tự động (SLA, escalation)
    Reason             : string?       // Lý do (bắt buộc với REJECT, RETURN, REASSIGNED)
    PreviousStepId     : Guid?         // Bước trước khi chuyển (cho STEP_ENTERED)
    NextStepId         : Guid?         // Bước tiếp theo (cho STEP_ENTERED)
    AssignedToUserId   : Guid?         // Cán bộ được phân công (cho CASE_ASSIGNED)
    AssignedToUserName : string?       // Tên cán bộ được phân công (snapshot)
    Metadata           : json?         // Dữ liệu bổ sung tùy loại action
    IpAddress          : string?       // IP của người thực hiện
    UserAgent          : string?       // Thiết bị / trình duyệt
}
```

> **Nguyên tắc snapshot:** Tên bước, tên người dùng, tên phòng ban được lưu trực tiếp vào bản ghi log tại thời điểm phát sinh. Không dùng foreign key cho các trường này để tránh mất dữ liệu lịch sử khi cơ cấu tổ chức thay đổi.

---

### 1.4 Quy tắc ghi log

1. **Bất biến (Immutable):** Bản ghi audit log **không được sửa hoặc xóa** sau khi tạo. Chỉ cho phép INSERT.
2. **Ghi ngay lập tức:** Log phải được tạo trong cùng transaction với hành động nghiệp vụ. Không ghi log bất đồng bộ để tránh mất dữ liệu.
3. **Không phụ thuộc trạng thái hiện tại:** Log snapshot tên/vai trò/phòng ban tại thời điểm phát sinh, không đọc lại từ bảng master.
4. **Lý do bắt buộc với một số hành động:** `REJECT`, `RETURN`, `CASE_REASSIGNED` yêu cầu cán bộ nhập lý do trước khi ghi log.
5. **Hành động hệ thống tự động:** Các sự kiện SLA, escalation được ghi với `PerformedBy = null` và `IsSystemGenerated = true`.

---

### 1.5 Giao diện xem nhật ký hồ sơ

#### Ai được xem

| Vai trò | Phạm vi xem |
|---|---|
| Cán bộ xử lý | Chỉ các hồ sơ mình đang/đã xử lý |
| Trưởng phòng | Toàn bộ hồ sơ trong phòng ban |
| Lãnh đạo phê duyệt | Hồ sơ đã qua bước phê duyệt của mình |
| Admin / Giám sát | Tất cả hồ sơ trong hệ thống |
| Người dân | Xem nhật ký rút gọn của hồ sơ chính mình (không thấy nội dung nội bộ) |

#### Nội dung hiển thị

**Dòng thời gian (Timeline View):**

```
[2026-05-01 09:15]  Nguyễn Văn A (Cán bộ tiếp nhận – Bộ phận Một cửa)
                    → Tiếp nhận hồ sơ

[2026-05-01 09:20]  Nguyễn Văn A (Cán bộ tiếp nhận – Bộ phận Một cửa)
                    → Phân công cho Trần Thị B (Chuyên viên – Phòng Chuyên môn)

[2026-05-01 14:30]  Trần Thị B (Chuyên viên – Phòng Chuyên môn)
                    → Yêu cầu bổ sung: "Cần bổ sung giấy phép xây dựng bản gốc"

[2026-05-02 08:45]  Hệ thống
                    → Cảnh báo SLA: Hồ sơ còn 4 giờ đến hạn xử lý tại bước Chuyên môn

[2026-05-02 10:00]  Trần Thị B (Chuyên viên – Phòng Chuyên môn)
                    → Trình lãnh đạo phê duyệt

[2026-05-02 11:30]  Lê Văn C (Lãnh đạo – Ban Giám đốc)
                    → Phê duyệt
```

#### Bộ lọc tìm kiếm

- Lọc theo hồ sơ (`CaseCode`)
- Lọc theo người thực hiện
- Lọc theo bước trong workflow
- Lọc theo loại hành động (`ActionCode`)
- Lọc theo khoảng thời gian
- Lọc theo kết quả (phê duyệt / từ chối / trả lại)

---

### 1.6 Nhật ký rút gọn cho người dân (Citizen View)

Người dân chỉ được xem phiên bản rút gọn, **không tiết lộ thông tin nội bộ**:

```
Trạng thái hồ sơ #HS-2026-001234

[01/05/2026 09:15]  Hồ sơ đã được tiếp nhận tại Bộ phận Một cửa
[01/05/2026 14:30]  Hồ sơ đang được xem xét. Cần bổ sung thêm tài liệu.
                    → Vui lòng nộp bổ sung: Giấy phép xây dựng bản gốc
[02/05/2026 11:30]  Hồ sơ đã được phê duyệt. Dự kiến trả kết quả: 05/05/2026
```

Thông tin **không hiển thị** với người dân:
- Tên cán bộ cụ thể xử lý (chỉ hiện tên phòng ban)
- Ghi chú nội bộ giữa các cán bộ
- Chi tiết lý do từ chối nội bộ (chỉ hiện thông báo chính thức)

---

## 2. Config Audit Log – Nhật ký cấu hình Workflow

### 2.1 Mục đích

Ghi lại mọi thay đổi mà admin thực hiện trên cấu hình workflow: tạo mới, sửa bước, publish, deactivate. Hỗ trợ:

- Tra cứu "ai đã publish workflow này lúc nào".
- So sánh diff giữa hai phiên bản workflow.
- Điều tra sự cố khi workflow hoạt động không như mong đợi.

---

### 2.2 Các hành động được ghi

| Mã hành động | Mô tả |
|---|---|
| `WORKFLOW_CREATED` | Admin tạo workflow mới |
| `WORKFLOW_UPDATED` | Admin sửa workflow đang ở DRAFT/TESTING |
| `WORKFLOW_CLONED` | Admin clone từ workflow đang ACTIVE |
| `STATUS_CHANGED` | Workflow chuyển trạng thái (DRAFT → READY_FOR_TEST, ...) |
| `WORKFLOW_PUBLISHED` | Admin publish workflow lên ACTIVE |
| `WORKFLOW_DEACTIVATED` | Admin vô hiệu hóa workflow |
| `WORKFLOW_ARCHIVED` | Admin lưu trữ workflow |
| `STEP_ADDED` | Thêm bước vào workflow |
| `STEP_UPDATED` | Sửa cấu hình bước |
| `STEP_REMOVED` | Xóa bước khỏi workflow |
| `ACTION_ADDED` | Thêm action vào bước |
| `ACTION_UPDATED` | Sửa action |
| `ACTION_REMOVED` | Xóa action |
| `TRANSITION_ADDED` | Thêm transition |
| `TRANSITION_UPDATED` | Sửa điều kiện transition |
| `TRANSITION_REMOVED` | Xóa transition |
| `ASSIGNMENT_RULE_UPDATED` | Sửa quy tắc phân công |
| `NOTIFICATION_RULE_UPDATED` | Sửa quy tắc thông báo |
| `SLA_CONFIG_UPDATED` | Sửa cấu hình SLA |

---

### 2.3 Cấu trúc dữ liệu – WorkflowConfigAuditLog

```
WorkflowConfigAuditLog {
    Id                 : Guid          // Định danh duy nhất
    WorkflowId         : Guid          // Workflow liên quan
    WorkflowVersion    : string        // Phiên bản tại thời điểm thay đổi
    EntityType         : string        // Thực thể bị thay đổi: WORKFLOW / STEP / ACTION / TRANSITION / ...
    EntityId           : Guid?         // Id của thực thể (StepId, ActionId, ...)
    ActionCode         : string        // Mã hành động (bảng 2.2)
    ActionLabel        : string        // Tên hiển thị
    PerformedBy        : Guid          // Admin thực hiện
    PerformedByName    : string        // Tên admin (snapshot)
    PerformedAt        : datetime      // Thời điểm (UTC)
    OldValue           : json?         // Giá trị trước khi thay đổi (JSON snapshot)
    NewValue           : json?         // Giá trị sau khi thay đổi (JSON snapshot)
    ChangeReason       : string?       // Lý do thay đổi (admin nhập khi publish/deactivate)
    IpAddress          : string?       // IP của admin
}
```

---

### 2.4 So sánh diff giữa hai phiên bản (Version Diff)

Admin có thể chọn **hai phiên bản bất kỳ** của cùng một workflow để xem sự khác biệt.

#### Cấu trúc kết quả diff

```
WorkflowDiff {
    WorkflowId          : Guid
    WorkflowCode        : string
    VersionA            : string       // Phiên bản cũ hơn (vd: v1.0)
    VersionB            : string       // Phiên bản mới hơn (vd: v2.0)
    GeneratedAt         : datetime

    Changes             : [
        DiffItem {
            EntityType      : string   // WORKFLOW / STEP / ACTION / TRANSITION / ...
            EntityName      : string   // Tên hiển thị của thực thể
            ChangeType      : string   // ADDED / MODIFIED / REMOVED
            Fields          : [
                FieldDiff {
                    FieldName   : string
                    OldValue    : any?
                    NewValue    : any?
                }
            ]
        }
    ]

    Summary {
        StepsAdded          : int
        StepsRemoved        : int
        StepsModified       : int
        ActionsAdded        : int
        ActionsRemoved      : int
        TransitionsChanged  : int
        NotificationRulesChanged : int
        SlaConfigChanged    : int
    }
}
```

#### Ví dụ hiển thị diff

```
So sánh workflow "Hồ sơ xây dựng" – v1.0 vs v2.0

BƯỚC THAY ĐỔI:
─────────────────────────────────────────────────────────────────
[MODIFIED] Bước "Thẩm định"
  ├── SLA.DurationHours: 48 giờ  →  72 giờ
  └── SLA.EscalationTargetRole: DEPT_MANAGER  →  VICE_DIRECTOR

[ADDED] Bước "Kiểm tra pháp lý" (sau bước Thẩm định)
  ├── StepType: PROCESS
  ├── AssignmentRule: Role = LEGAL_OFFICER
  └── SLA.DurationHours: 24 giờ

[REMOVED] Bước "Xác minh thực địa"
  └── (đã được gộp vào bước Thẩm định)

TRANSITION THAY ĐỔI:
─────────────────────────────────────────────────────────────────
[MODIFIED] Thẩm định → Phê duyệt (Action: APPROVE)
  └── Condition: Amount > 500M  →  Amount > 1000M

[ADDED] Kiểm tra pháp lý → Phê duyệt (Action: APPROVE)

NOTIFICATION THAY ĐỔI:
─────────────────────────────────────────────────────────────────
[MODIFIED] Thông báo quá hạn tại bước Phê duyệt
  └── EscalationTargetRole: DIRECTOR  →  VICE_DIRECTOR

TỔNG KẾT: 1 bước thêm · 1 bước xóa · 1 bước sửa · 2 transition thay đổi
```

---

### 2.5 Giao diện quản lý Config Audit Log

#### Ai được xem

| Vai trò | Phạm vi xem |
|---|---|
| Admin | Toàn bộ lịch sử cấu hình tất cả workflow |
| Admin cấp cao | Như Admin + có thể export |
| Giám sát (Monitor) | Chỉ xem, không export |

#### Tính năng giao diện

1. **Timeline cấu hình:** Hiển thị dòng thời gian các thay đổi cấu hình theo từng workflow.
2. **So sánh phiên bản:** Chọn hai phiên bản để xem diff chi tiết.
3. **Tìm kiếm:** Lọc theo admin, thời gian, loại hành động, workflow cụ thể.
4. **Export:** Xuất nhật ký cấu hình ra CSV/Excel cho mục đích audit.

---

## 3. Chính sách lưu trữ và bảo mật

### 3.1 Thời gian lưu trữ

| Loại log | Thời gian lưu tối thiểu | Lý do |
|---|---|---|
| Case Audit Log | 10 năm | Yêu cầu lưu trữ hồ sơ hành chính công |
| Config Audit Log | 5 năm | Kiểm toán nội bộ và truy xuất sự cố |

> ⚠️ **Lưu ý:** Thời gian lưu trữ cần được xác nhận lại với đội pháp lý theo quy định hiện hành về lưu trữ hồ sơ hành chính công.

### 3.2 Bất biến và toàn vẹn dữ liệu

- Bảng audit log **chỉ INSERT**, không UPDATE/DELETE.
- Xem xét dùng cơ chế checksum hoặc append-only storage để phát hiện tamper.
- Phân quyền: Chỉ service account của ứng dụng được INSERT; không có user nào được DELETE/UPDATE trực tiếp trên bảng log.

### 3.3 Phân quyền xem

- Log có thể chứa thông tin nhạy cảm (lý do từ chối, ghi chú nội bộ).
- Áp dụng kiểm soát phân quyền chặt chẽ: mỗi vai trò chỉ xem được phạm vi được phép (xem bảng 1.5 và 2.5).
- Ghi lại việc xem audit log vào một meta-audit log (ai đã xem log của ai, khi nào).

---

## 4. Yêu cầu hiệu năng

| Yêu cầu | Tiêu chí |
|---|---|
| Ghi log không chặn luồng nghiệp vụ | Thời gian ghi log < 50ms, không gây timeout cho transaction chính |
| Truy vấn timeline một hồ sơ | < 500ms cho hồ sơ có đến 500 bản ghi log |
| Tìm kiếm toàn hệ thống | < 2 giây cho bộ lọc thông thường trên 10 triệu bản ghi |
| Export CSV | < 30 giây cho báo cáo 1 tháng |

---

## 5. Tích hợp với các tính năng khác

| Tính năng | Cách tích hợp |
|---|---|
| **Dashboard & Báo cáo** | Đọc Case Audit Log để tính thời gian xử lý trung bình, phát hiện bottleneck |
| **SLA Engine** | Ghi `SLA_WARNING` và `SLA_BREACHED` vào Case Audit Log |
| **Notification** | Ghi log khi gửi thông báo (thành công/thất bại) để trace |
| **Workflow Lifecycle** | Config Audit Log ghi mỗi khi trạng thái workflow thay đổi |
| **Citizen Portal** | Đọc Case Audit Log (phiên bản rút gọn) để hiển thị trạng thái cho người dân |

---

## 6. Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Thời gian lưu trữ log tối thiểu theo quy định pháp lý là bao lâu? | Cao |
| 2 | Lý do từ chối/trả lại có hiển thị cho người dân không, hay chỉ nội bộ? | Cao |
| 3 | Có cần ghi log truy cập (ai đã xem hồ sơ, kể cả chỉ xem mà không hành động)? | Trung bình |
| 4 | Meta-audit (ghi log việc xem log) có bắt buộc không? | Trung bình |
| 5 | Tên cán bộ trong log hiển thị cho người dân: full name hay chỉ tên phòng ban? | Trung bình |
| 6 | Export log cần hỗ trợ format nào: CSV, Excel, PDF hay cả ba? | Thấp |
| 7 | Có cần tích hợp với hệ thống quản lý văn bản điện tử (EDMS) để đồng bộ log không? | Thấp |
