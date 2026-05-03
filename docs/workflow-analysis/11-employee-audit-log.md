# 11. Audit Log cho Nhân viên

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này đặc tả yêu cầu nghiệp vụ cho cơ chế **audit log nhân viên** – ghi lại mọi thao tác mà cán bộ/nhân viên thực hiện khi xử lý hồ sơ trong hệ thống workflow.

Mục tiêu cụ thể:

- Đảm bảo **truy xuất nguồn gốc** (traceability): biết ai làm gì, lúc nào, trên hồ sơ nào.
- Hỗ trợ **giám sát vận hành**: phát hiện bất thường, tắc nghẽn, hoặc vi phạm quy trình.
- Đáp ứng **yêu cầu kiểm toán** nội bộ và các quy định pháp lý liên quan.
- Cung cấp **bằng chứng** khi có tranh chấp hoặc khiếu nại về quá trình xử lý hồ sơ.

---

## Phạm vi áp dụng

Audit log nhân viên áp dụng cho tất cả các đối tượng sau khi tương tác với hồ sơ trong workflow đang **ACTIVE**:

| Đối tượng | Mã vai trò | Thao tác điển hình |
|---|---|---|
| Cán bộ tiếp nhận | `RECEPTION_OFFICER` | Tiếp nhận, kiểm tra, chuyển hồ sơ, yêu cầu bổ sung |
| Chuyên viên xử lý | `SPECIALIST` | Thẩm định, xử lý nghiệp vụ, trình duyệt, yêu cầu phối hợp |
| Lãnh đạo phê duyệt | `APPROVER` | Phê duyệt, từ chối, trả lại |
| Cán bộ phối hợp | `COORDINATOR` | Cung cấp ý kiến phối hợp, xác minh |
| Bộ phận trả kết quả | `RETURN_OFFICER` | Xác nhận hoàn tất, phát hành kết quả |
| Quản trị / Giám sát | `SYSTEM_ADMIN` | Can thiệp khẩn cấp, chuyển hồ sơ thủ công, cấu hình |

> **Lưu ý:** Audit log cho thao tác **cấu hình workflow của Admin** (tạo/sửa/kích hoạt workflow) được mô tả trong [09-governance-and-activation.md](./09-governance-and-activation.md). Tài liệu này tập trung vào thao tác xử lý hồ sơ của nhân viên nghiệp vụ.

---

## Các thao tác cần ghi log

### Nhóm 1: Thao tác trực tiếp trên hồ sơ

| Mã hành động | Tên hiển thị | Mô tả |
|---|---|---|
| `RECEIVE` | Tiếp nhận hồ sơ | Cán bộ xác nhận nhận hồ sơ vào xử lý |
| `REQUEST_SUPPLEMENT` | Yêu cầu bổ sung | Yêu cầu người dân bổ sung tài liệu hoặc thông tin |
| `TRANSFER` | Chuyển bộ phận | Chuyển hồ sơ sang bộ phận / bước tiếp theo |
| `ASSIGN` | Phân công xử lý | Phân công hồ sơ cho chuyên viên cụ thể |
| `REQUEST_COLLABORATION` | Yêu cầu phối hợp | Gửi yêu cầu xin ý kiến tới phòng liên quan |
| `SUBMIT_COLLABORATION` | Phản hồi phối hợp | Gửi kết quả ý kiến phối hợp |
| `SUBMIT_FOR_APPROVAL` | Trình duyệt | Chuyển hồ sơ lên lãnh đạo phê duyệt |
| `APPROVE` | Phê duyệt | Lãnh đạo duyệt hồ sơ |
| `REJECT` | Từ chối | Từ chối hồ sơ kèm lý do |
| `RETURN` | Trả lại bước trước | Trả hồ sơ về bước trước để chỉnh sửa |
| `CLOSE` | Đóng / Trả kết quả | Hoàn tất và phát hành kết quả cho người dân |

### Nhóm 2: Thao tác quản lý hồ sơ

| Mã hành động | Tên hiển thị | Mô tả |
|---|---|---|
| `VIEW_RECORD` | Xem hồ sơ | Cán bộ mở xem nội dung hồ sơ |
| `DOWNLOAD_ATTACHMENT` | Tải tệp đính kèm | Tải xuống tài liệu đính kèm của hồ sơ |
| `ADD_NOTE` | Thêm ghi chú nội bộ | Thêm ghi chú nội bộ không hiển thị cho người dân |
| `EDIT_NOTE` | Sửa ghi chú nội bộ | Chỉnh sửa ghi chú đã thêm trước đó |
| `MANUAL_OVERRIDE` | Can thiệp thủ công | Admin/Giám sát can thiệp đặc biệt ngoài quy trình thông thường |

### Nhóm 3: Thao tác liên quan SLA

| Mã hành động | Tên hiển thị | Mô tả |
|---|---|---|
| `SLA_WARNING_ACK` | Xác nhận cảnh báo SLA | Nhân viên đã đọc cảnh báo sắp hết hạn |
| `SLA_ESCALATION_RECEIVED` | Nhận leo thang SLA | Ghi nhận hồ sơ được leo thang do quá hạn |
| `SLA_EXTENSION_REQUEST` | Yêu cầu gia hạn | Yêu cầu kéo dài thời hạn xử lý (nếu có quy trình) |

---

## Cấu trúc dữ liệu audit log

### Bảng `EmployeeAuditLog`

| Trường | Kiểu dữ liệu | Bắt buộc | Mô tả |
|---|---|---|---|
| `Id` | UUID | ✓ | Định danh duy nhất của bản ghi log |
| `Timestamp` | DateTime (UTC) | ✓ | Thời điểm thao tác xảy ra (múi giờ UTC) |
| `ActorId` | UUID | ✓ | ID người thực hiện thao tác |
| `ActorName` | String | ✓ | Tên đầy đủ (snapshot lúc ghi, không join realtime) |
| `ActorRole` | String | ✓ | Vai trò tại thời điểm thao tác (vd: `SPECIALIST`) |
| `ActorDepartment` | String | ✓ | Phòng ban tại thời điểm thao tác |
| `ActionCode` | String | ✓ | Mã hành động (xem bảng trên) |
| `ActionName` | String | ✓ | Tên hành động (snapshot) |
| `RecordId` | UUID | ✓ | ID hồ sơ bị tác động |
| `RecordCode` | String | ✓ | Mã hồ sơ (vd: `HS-2026-001234`) |
| `WorkflowDefinitionId` | UUID | ✓ | Workflow đang áp dụng |
| `WorkflowVersion` | String | ✓ | Phiên bản workflow (vd: `v2.1`) |
| `StepId` | UUID | ✓ | Bước trong workflow đang xử lý |
| `StepName` | String | ✓ | Tên bước (snapshot) |
| `PreviousStatus` | String | Tùy | Trạng thái hồ sơ trước thao tác |
| `NewStatus` | String | Tùy | Trạng thái hồ sơ sau thao tác |
| `Reason` | String | Tùy | Lý do (bắt buộc với REJECT, RETURN, MANUAL_OVERRIDE) |
| `Metadata` | JSON | Tùy | Dữ liệu bổ sung tùy hành động (vd: danh sách tài liệu yêu cầu bổ sung) |
| `IpAddress` | String | ✓ | Địa chỉ IP của thiết bị thực hiện |
| `UserAgent` | String | Tùy | Trình duyệt / ứng dụng |
| `SessionId` | String | Tùy | ID phiên đăng nhập |

> **Nguyên tắc snapshot:** Các trường tên (ActorName, ActorRole, StepName, ActionName) được lưu theo giá trị **tại thời điểm ghi log**, không phụ thuộc vào dữ liệu master có thể thay đổi sau này. Điều này đảm bảo tính toàn vẹn lịch sử.

---

## Quy tắc ghi log

### Ghi log bắt buộc

1. **Mọi hành động thay đổi trạng thái hồ sơ** (RECEIVE, TRANSFER, APPROVE, REJECT, CLOSE, ...) đều **phải ghi log** – không có ngoại lệ.
2. **Thao tác MANUAL_OVERRIDE** của Admin luôn phải có trường `Reason` và được ghi đặc biệt với độ ưu tiên cao.
3. **Ghi log phải xảy ra trong cùng transaction** với thao tác nghiệp vụ – không được ghi log sau khi transaction đã commit để tránh mất đồng bộ.

### Ghi log tùy chọn (có thể bật/tắt theo cấu hình)

- `VIEW_RECORD`: Có thể bật/tắt theo chính sách; mặc định **tắt** để tránh log quá lớn.
- `DOWNLOAD_ATTACHMENT`: Mặc định **bật** do liên quan tài liệu pháp lý.
- `ADD_NOTE` / `EDIT_NOTE`: Mặc định **bật**.

### Bất biến (immutability)

- Bản ghi audit log **không được phép sửa hoặc xóa** sau khi đã tạo, kể cả bởi Admin.
- Nếu cần đính chính, phải tạo một bản ghi mới với `ActionCode = CORRECTION` và tham chiếu đến bản ghi gốc.

---

## Quy định truy cập

| Vai trò | Quyền xem | Phạm vi |
|---|---|---|
| `SYSTEM_ADMIN` | Xem toàn bộ | Tất cả hồ sơ, tất cả nhân viên |
| `DEPT_MANAGER` | Xem theo phòng ban | Chỉ hồ sơ và nhân viên thuộc phòng ban mình |
| `SPECIALIST` / `RECEPTION_OFFICER` | Xem giới hạn | Chỉ log liên quan hồ sơ được phân công cho mình |
| `APPROVER` | Xem giới hạn | Chỉ log liên quan hồ sơ đã trình duyệt lên mình |
| `AUDITOR` (kiểm toán nội bộ) | Xem toàn bộ (chỉ đọc) | Tất cả, không có quyền thao tác |

> **Nguyên tắc:** Nhân viên không được xem log của đồng nghiệp trừ khi có vai trò Quản lý hoặc Kiểm toán.

---

## Tra cứu và lọc log

Giao diện tra cứu audit log cần hỗ trợ lọc theo:

| Tiêu chí lọc | Mô tả |
|---|---|
| Khoảng thời gian | Từ ngày – đến ngày |
| Mã hồ sơ / RecordCode | Xem toàn bộ lịch sử của một hồ sơ |
| Nhân viên (ActorId) | Xem tất cả thao tác của một nhân viên cụ thể |
| Phòng ban | Xem thao tác của toàn bộ phòng ban |
| Loại hành động (ActionCode) | Lọc theo nhóm thao tác |
| Trạng thái thay đổi | Lọc theo chuyển trạng thái cụ thể |
| Workflow / Phiên bản | Lọc theo workflow hoặc version cụ thể |

---

## Thời hạn lưu trữ

| Loại log | Thời hạn lưu | Ghi chú |
|---|---|---|
| Log thay đổi trạng thái hồ sơ | Tối thiểu **5 năm** | Tuân theo quy định lưu trữ hồ sơ hành chính |
| Log xem / tải tài liệu | Tối thiểu **2 năm** | Có thể rút ngắn theo chính sách nội bộ |
| Log ghi chú nội bộ | Tối thiểu **5 năm** | Đi kèm với vòng đời hồ sơ |
| Log can thiệp thủ công (MANUAL_OVERRIDE) | **Vĩnh viễn** | Không được xóa |

> **Lưu ý:** Thời hạn cụ thể cần được xác nhận với bộ phận pháp chế trước khi triển khai.

---

## Tình huống nghiệp vụ điển hình

### TH-AL-01: Tra cứu lịch sử xử lý hồ sơ

**Bối cảnh:** Người dân khiếu nại hồ sơ của họ bị xử lý chậm.  
**Ai thực hiện:** Admin hoặc Trưởng phòng.  
**Luồng:** Tra cứu log theo `RecordCode` → Xem toàn bộ chuỗi thao tác theo thứ tự thời gian → Xác định bước nào bị tắc nghẽn và ai chịu trách nhiệm.

---

### TH-AL-02: Kiểm tra nhân viên thực hiện thao tác bất thường

**Bối cảnh:** Hệ thống phát hiện một nhân viên có số lượng hành động `REJECT` bất thường trong ngày.  
**Ai thực hiện:** Admin / Kiểm toán.  
**Luồng:** Lọc log theo `ActorId` + `ActionCode = REJECT` + khoảng thời gian → Xem lý do từ trường `Reason` → Quyết định có cần điều tra thêm.

---

### TH-AL-03: Can thiệp thủ công của Admin

**Bối cảnh:** Admin phải chuyển hồ sơ khẩn cấp do nhân viên phụ trách nghỉ phép đột xuất.  
**Ai thực hiện:** Admin (`SYSTEM_ADMIN`).  
**Luồng:** Thực hiện `MANUAL_OVERRIDE` → Hệ thống bắt buộc nhập `Reason` → Ghi log với flag đặc biệt → Thông báo cho Trưởng phòng liên quan.

---

### TH-AL-04: Kiểm toán định kỳ

**Bối cảnh:** Kiểm toán nội bộ yêu cầu báo cáo tất cả thao tác trên hồ sơ trong quý.  
**Ai thực hiện:** Kiểm toán viên (`AUDITOR`).  
**Luồng:** Lọc log theo khoảng thời gian → Xuất file CSV/Excel → Phân tích ngoài hệ thống.

---

## Cảnh báo và giám sát tự động

Hệ thống nên tự động phát cảnh báo khi phát hiện các pattern bất thường:

| Điều kiện | Ngưỡng gợi ý | Hành động |
|---|---|---|
| Một nhân viên thực hiện quá nhiều `REJECT` trong 1 ngày | > 10 lần | Gửi cảnh báo cho Trưởng phòng |
| Hồ sơ không có log hoạt động trong thời gian dài | > SLA × 2 | Gửi cảnh báo escalation |
| Có `MANUAL_OVERRIDE` mà không có `Reason` | Bất kỳ | Chặn ngay, yêu cầu nhập lý do |
| Cùng một hồ sơ có > 3 lần `RETURN` liên tiếp | 3 lần | Gửi cảnh báo cho lãnh đạo |
| Truy cập log vào giờ bất thường (ngoài giờ làm việc) | Tuỳ chính sách | Ghi nhận, cảnh báo nếu cần |

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên | Người phụ trách |
|---|---------|----------------|-----------------|
| 1 | Thời hạn lưu trữ chính xác theo quy định pháp lý hiện hành là bao nhiêu năm? | Cao | Pháp chế |
| 2 | Có yêu cầu mã hóa (encryption) dữ liệu audit log khi lưu trữ không? | Cao | Kỹ thuật + Bảo mật |
| 3 | Có cần cơ chế export log định kỳ sang hệ thống lưu trữ bên ngoài không? | Trung bình | Kỹ thuật |
| 4 | Có bật log `VIEW_RECORD` mặc định không, hay chỉ bật khi điều tra? | Trung bình | Nghiệp vụ |
| 5 | Giao diện tra cứu audit log dành cho DEPT_MANAGER có cần tích hợp vào dashboard không? | Trung bình | UX/UI |
| 6 | Có cần thông báo realtime cho Admin khi có `MANUAL_OVERRIDE` không? | Thấp | Nghiệp vụ |
