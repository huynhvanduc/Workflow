# 14. Rollback và Xử lý Khẩn cấp

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này định nghĩa cơ chế rollback phiên bản workflow, quy trình xử lý khẩn cấp khi phiên bản ACTIVE có lỗi nghiêm trọng, và các nguyên tắc đảm bảo hồ sơ đang xử lý dở không bị ảnh hưởng. Đây là phần bổ sung cho `06-workflow-lifecycle.md` và `09-governance-and-activation.md`.

---

## Khái niệm cơ bản

### Rollback là gì?

**Rollback** (khôi phục phiên bản) là hành động **đưa một phiên bản workflow cũ trở lại trạng thái ACTIVE** để thay thế phiên bản hiện tại đang có lỗi hoặc không phù hợp.

Rollback **khác** với việc tạo phiên bản mới:

| Khía cạnh | Rollback | Tạo phiên bản mới |
|-----------|----------|--------------------|
| Cấu hình | Dùng lại cấu hình của phiên bản cũ đã tồn tại | Tạo cấu hình mới từ đầu hoặc từ clone |
| Thời gian | Nhanh hơn – không cần qua đủ vòng đời test | Chậm hơn – phải qua DRAFT → TESTING → ACTIVE |
| Rủi ro | Thấp hơn – phiên bản cũ đã từng hoạt động ổn | Cao hơn – cần kiểm thử kỹ |
| Khi dùng | Phiên bản mới có lỗi nghiêm trọng sau khi ACTIVE | Thay đổi nghiệp vụ có kế hoạch |
| Yêu cầu | Phiên bản cũ còn tồn tại ở trạng thái INACTIVE | Không bắt buộc có phiên bản cũ |

### Emergency Hotfix là gì?

**Emergency Hotfix** là thao tác sửa lỗi khẩn cấp mà **không cần đi qua đầy đủ vòng đời** (DRAFT → TESTING → ACTIVE). Chỉ áp dụng cho các lỗi cấu hình nhỏ không ảnh hưởng đến cấu trúc luồng, được thực hiện trong một cửa sổ thời gian kiểm soát chặt.

> ⚠️ **Cả rollback và hotfix đều là ngoại lệ có kiểm soát.** Không được dùng tùy tiện thay cho quy trình tạo phiên bản thông thường.

---

## Các tình huống cần Rollback hoặc Hotfix

| # | Tình huống | Loại can thiệp đề xuất | Mức độ khẩn |
|---|-----------|------------------------|-------------|
| 1 | Phiên bản mới ACTIVE nhưng hồ sơ bị kẹt ở bước không thể chuyển tiếp | Rollback về phiên bản cũ | 🔴 Khẩn cấp |
| 2 | Phân công sai – hồ sơ giao cho phòng không có thẩm quyền | Rollback hoặc Hotfix (tùy phạm vi) | 🔴 Khẩn cấp |
| 3 | Thông báo gửi sai đối tượng hàng loạt | Hotfix (sửa notification rule) | 🟠 Cao |
| 4 | SLA cấu hình sai – thời hạn quá ngắn hoặc quá dài | Hotfix (sửa SLA value) | 🟠 Cao |
| 5 | Phiên bản mới có cấu trúc luồng sai (thiếu bước, transition sai) | Rollback về phiên bản cũ | 🔴 Khẩn cấp |
| 6 | Phiên bản mới gây lỗi hiển thị nhưng chưa ảnh hưởng hồ sơ thực | Tạo phiên bản sửa lỗi (quy trình thông thường) | 🟡 Trung bình |
| 7 | Admin publish nhầm phiên bản chưa hoàn chỉnh | Rollback ngay + điều tra nguyên nhân | 🔴 Khẩn cấp |

---

## Quy trình Rollback

### Bước 1 – Xác nhận sự cố và đánh giá

Trước khi rollback, admin phải xác nhận:

- [ ] Phiên bản hiện tại (vX.Y) thực sự có lỗi ảnh hưởng đến vận hành
- [ ] Đã có phiên bản cũ (vX.Z, với Z < Y) đang ở trạng thái INACTIVE và đã từng hoạt động ổn
- [ ] Rollback là phương án phù hợp (không phải hotfix hoặc tạo phiên bản mới)
- [ ] Đã ghi nhận lỗi vào hệ thống issue tracking với mức độ **Critical**

### Bước 2 – Thông báo trước khi rollback

Trước khi thực hiện rollback, admin phải gửi thông báo đến:

- Tất cả cán bộ nghiệp vụ đang sử dụng workflow này
- Nhóm kỹ thuật / giám sát hệ thống
- Cấp có thẩm quyền (trưởng bộ phận hoặc lãnh đạo phụ trách)

Nội dung thông báo cần nêu rõ: lý do rollback, phiên bản sẽ được khôi phục, thời điểm thực hiện, tác động dự kiến.

### Bước 3 – Thực hiện Rollback

```
Trạng thái trước rollback:
  vX.Z → INACTIVE
  vX.Y → ACTIVE  ← có lỗi

Thao tác admin thực hiện:
  1. Deactivate vX.Y (chuyển sang INACTIVE hoặc ARCHIVED)
  2. Restore vX.Z (chuyển từ INACTIVE → ACTIVE)

Trạng thái sau rollback:
  vX.Z → ACTIVE  ← khôi phục
  vX.Y → INACTIVE / ARCHIVED
```

**Điều kiện để rollback hợp lệ:**

| # | Điều kiện | Kiểm tra bởi |
|---|-----------|-------------|
| 1 | Phiên bản đích (vX.Z) tồn tại ở trạng thái INACTIVE | Hệ thống kiểm tra tự động |
| 2 | Phiên bản đích đã từng ACTIVE và có lịch sử vận hành | Hệ thống kiểm tra tự động |
| 3 | Admin có quyền thực hiện rollback | Hệ thống phân quyền |
| 4 | Đã ghi nhận lý do rollback vào audit log | Bắt buộc trước khi hệ thống cho phép |

### Bước 4 – Xử lý hồ sơ đang dở sau Rollback

Đây là phần quan trọng nhất và cần quyết định rõ ràng:

| Trạng thái hồ sơ | Hành xử sau Rollback | Lý do |
|------------------|---------------------|-------|
| Hồ sơ đang xử lý theo **vX.Y** (phiên bản lỗi) và đang ở bước **hợp lệ trong vX.Z** | Tiếp tục theo **vX.Z** (phiên bản được khôi phục) | Bước tương thích → không cần can thiệp |
| Hồ sơ đang ở bước **chỉ tồn tại trong vX.Y** (không có trong vX.Z) | Admin phải can thiệp thủ công: chuyển thủ công sang bước tương đương trong vX.Z | Không thể tự động giải quyết |
| Hồ sơ đã bị kẹt do lỗi của vX.Y | Hệ thống đánh dấu `BLOCKED` + thông báo admin + chờ xử lý thủ công | Cần xem xét từng trường hợp |
| Hồ sơ đã hoàn thành theo vX.Y trước khi rollback | Giữ nguyên trạng thái CLOSED – không ảnh hưởng | Hồ sơ đã kết thúc |

> **Nguyên tắc cốt lõi:** Rollback **không được tự động hủy hoặc thay đổi trạng thái hồ sơ** mà không có sự xem xét của admin. Hệ thống phải đánh dấu các hồ sơ cần can thiệp thủ công và tạo task xử lý cho admin.

### Bước 5 – Xác nhận hoàn thành và theo dõi

- [ ] Xác nhận vX.Z đang ACTIVE và hệ thống nhận hồ sơ mới theo đúng cấu hình cũ
- [ ] Kiểm tra danh sách hồ sơ bị đánh dấu `BLOCKED` – xử lý từng trường hợp
- [ ] Thông báo cho cán bộ nghiệp vụ: hệ thống đã khôi phục và cần lưu ý gì
- [ ] Lên kế hoạch điều tra và sửa lỗi trên vX.Y (hoặc tạo vX.Y+1) để chuẩn bị cho lần deploy tiếp theo

---

## Quy trình Emergency Hotfix

Emergency Hotfix cho phép sửa lỗi nhỏ trực tiếp trên workflow ACTIVE mà không cần đi qua đủ vòng đời. **Phạm vi áp dụng bị giới hạn nghiêm ngặt.**

### Phạm vi được phép Hotfix

| ✅ Được phép Hotfix | ❌ Không được phép Hotfix – phải Rollback/Tạo phiên bản mới |
|--------------------|--------------------------------------------------------------|
| Sửa template nội dung thông báo (typo, sai tên) | Thêm hoặc xóa bước trong luồng |
| Điều chỉnh giá trị SLA (số ngày) | Thay đổi điều kiện transition |
| Sửa người nhận notification | Thay đổi assignment rule |
| Cập nhật mô tả bước (không ảnh hưởng logic) | Thêm/xóa action tại một bước |
| Sửa thông tin cấu hình liên lạc (email template) | Thay đổi cấu trúc bước (role, phòng ban xử lý) |

### Quy trình Hotfix

```
1. Admin xác định lỗi và phân loại: nằm trong phạm vi hotfix?
   ├─ Có → tiếp tục bước 2
   └─ Không → chuyển sang quy trình Rollback hoặc tạo phiên bản mới

2. Admin ghi nhận yêu cầu hotfix (mô tả lỗi, thay đổi dự kiến)

3. Hệ thống tạm thời mở khóa trường cấu hình cụ thể (chỉ trường hotfix, không mở toàn bộ)

4. Admin thực hiện thay đổi

5. Hệ thống yêu cầu admin xác nhận lại (xem preview thay đổi)

6. Ghi audit log đầy đủ: ai, khi nào, thay đổi gì, lý do

7. Thông báo cho cán bộ liên quan về thay đổi
```

> ⚠️ **Hotfix không reset thời gian tồn tại phiên bản.** Phiên bản vẫn giữ số version cũ (ví dụ: v1.1 → v1.1 sau hotfix, không thành v1.2). Tuy nhiên, hệ thống phải ghi nhận rõ "đã có hotfix" trong lịch sử phiên bản.

---

## Thời gian tồn tại phiên bản cũ

Sau khi một phiên bản chuyển sang INACTIVE (do phiên bản mới ACTIVE hoặc do rollback), phiên bản đó được lưu lại theo quy tắc:

| Điều kiện | Thời gian lưu tối thiểu |
|-----------|------------------------|
| Còn hồ sơ đang xử lý dở theo phiên bản này | Lưu đến khi hồ sơ cuối cùng hoàn thành + 90 ngày |
| Không còn hồ sơ dở | Lưu tối thiểu 365 ngày sau khi INACTIVE |
| Phiên bản đã bị xác định là lỗi nghiêm trọng (lý do rollback) | Lưu theo quy định audit – không xóa |
| Phiên bản ARCHIVED theo yêu cầu admin | Lưu vĩnh viễn (chỉ xóa khi có yêu cầu pháp lý) |

> **Lý do giữ lại:** Phục vụ tra cứu hồ sơ, audit, và làm cơ sở cho rollback nếu cần.

---

## Ma trận quyền hạn thực hiện Rollback và Hotfix

| Thao tác | `SYSTEM_ADMIN` | Admin cấp cao | `MONITOR` |
|----------|:--------------:|:-------------:|:---------:|
| Xem lịch sử phiên bản | ✅ | ✅ | ✅ |
| Khởi tạo yêu cầu rollback | ✅ | ✅ | ❌ |
| Phê duyệt và thực hiện rollback | ✅ | ✅ | ❌ |
| Thực hiện emergency hotfix | ✅ | ✅ | ❌ |
| Xử lý hồ sơ BLOCKED thủ công | ✅ | ✅ | ❌ |
| Xem audit log rollback | ✅ | ✅ | ✅ |

> **Đề xuất:** Rollback nên yêu cầu **ít nhất một xác nhận thứ hai** từ admin hoặc lãnh đạo khác để tránh thao tác nhầm.

---

## Audit Log cho Rollback và Hotfix

Mọi thao tác rollback và hotfix phải được ghi vào audit log với các trường bắt buộc:

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| `timestamp` | Thời điểm thực hiện | ✅ |
| `actor` | Tài khoản thực hiện | ✅ |
| `action_type` | `ROLLBACK` hoặc `HOTFIX` | ✅ |
| `workflow_id` | ID workflow bị tác động | ✅ |
| `from_version` | Phiên bản trước khi thay đổi | ✅ |
| `to_version` | Phiên bản sau khi thay đổi (rollback) hoặc giữ nguyên (hotfix) | ✅ |
| `reason` | Lý do thực hiện (mô tả tự do, bắt buộc nhập) | ✅ |
| `issue_ref` | Mã issue/lỗi liên quan (nếu có) | Khuyến nghị |
| `affected_instances` | Danh sách ID hồ sơ bị ảnh hưởng | ✅ |
| `approver` | Người xác nhận thứ hai (nếu có) | Khuyến nghị |

---

## Liên kết với các tài liệu khác

| Tài liệu | Nội dung liên quan |
|----------|--------------------|
| [06-workflow-lifecycle.md](./06-workflow-lifecycle.md) | Vòng đời trạng thái workflow, quy tắc phiên bản |
| [09-governance-and-activation.md](./09-governance-and-activation.md) | Điều kiện publish, điều kiện chặn, rủi ro quản trị |
| [11-audit-log.md](./11-audit-log.md) | Cấu trúc và yêu cầu audit log toàn hệ thống |
| [08-issue-feedback-tracking.md](./08-issue-feedback-tracking.md) | Quy trình ghi nhận và theo dõi lỗi phát sinh |

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| ~~1~~ | ~~Rollback có yêu cầu phê duyệt từ cấp trên trước khi thực hiện không?~~ | ✅ **Đã chốt:** Có |
| ~~2~~ | ~~Hồ sơ đang dở theo phiên bản lỗi: hệ thống tự xử lý hay bắt buộc admin can thiệp thủ công từng cái?~~ | ✅ **Đã chốt:** Hệ thống tự xử lý |
| 3 | Có giới hạn số lần rollback trong một khoảng thời gian nhất định không (để tránh lạm dụng)? | Trung bình |
| 4 | Thời gian giữ phiên bản INACTIVE: 365 ngày hay theo quy định lưu trữ hồ sơ hành chính (thường là 5–10 năm)? | Trung bình |
| 5 | Hotfix có cần một người xác nhận thứ hai không, hay admin đơn lẻ được phép thực hiện? | Cao |
| ~~6~~ | ~~Khi rollback, hồ sơ mới nộp trong khoảng thời gian phiên bản lỗi đang ACTIVE được xử lý theo phiên bản nào?~~ | ✅ **Đã chốt:** Phiên bản ổn định gần nhất |
| 7 | Có cần cơ chế "simulation" cho phép admin xem trước tác động của rollback lên các hồ sơ đang dở trước khi xác nhận không? | Trung bình |
