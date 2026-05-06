# 02. Đối tượng và phân quyền người dùng

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này xác định tất cả đối tượng (actor) tham gia vào hệ thống workflow xử lý hồ sơ, mô tả vai trò, quyền hạn, phạm vi thao tác và phạm vi xem dữ liệu của từng đối tượng.

---

## Danh sách đối tượng tham gia

Hệ thống có sáu nhóm đối tượng chính:

| Ký hiệu | Tên đối tượng | Mã vai trò hệ thống |
|---------|---------------|---------------------|
| A | Bộ phận Tiếp nhận / Một cửa | `RECEPTION` |
| B | Phòng Chuyên môn | `SPECIALIST` |
| C | Phòng/Ban Liên quan (phối hợp) | `COORDINATOR` |
| D | Lãnh đạo / Cấp phê duyệt | `APPROVER` / `LEADER` |
| E | Bộ phận Trả kết quả | `RETURN_DESK` |
| F | Quản trị / Giám sát | `SYSTEM_ADMIN` / `MONITOR` |
| – | Người dân / Người nộp hồ sơ | `SUBMITTER` / `CITIZEN` |

---

## Mô tả chi tiết từng đối tượng

### A. Bộ phận Tiếp nhận / Một cửa (`RECEPTION`)

**Vai trò nghiệp vụ:** Tiếp nhận hồ sơ ban đầu, kiểm tra tính hợp lệ, chuyển tiếp sang phòng chuyên môn.

**Quyền thao tác trên hồ sơ:**

| Hành động | Mô tả |
|-----------|-------|
| Tiếp nhận hồ sơ (`RECEIVE`) | Xác nhận nhận hồ sơ từ người dân |
| Yêu cầu bổ sung (`REQUEST_SUPPLEMENT`) | Yêu cầu người dân cung cấp thêm tài liệu |
| Chuyển hồ sơ (`TRANSFER`) | Chuyển hồ sơ sang phòng chuyên môn |
| Trả hồ sơ (`RETURN`) | Trả hồ sơ không hợp lệ về cho người dân |
| Đóng hồ sơ / Trả kết quả | Xác nhận người dân đã nhận kết quả |

**Nhận thông báo khi:**
- Có hồ sơ mới nộp vào hệ thống
- Hồ sơ bị lãnh đạo trả về do cần bổ sung
- Hồ sơ đã hoàn tất, sẵn sàng trả kết quả cho dân
- Hồ sơ trong queue bị cận hạn SLA

---

### B. Phòng Chuyên môn (`SPECIALIST`)

**Vai trò nghiệp vụ:** Thẩm định, xử lý nghiệp vụ chuyên sâu, trình lãnh đạo phê duyệt.

**Quyền thao tác trên hồ sơ:**

| Hành động | Mô tả |
|-----------|-------|
| Tiếp nhận từ Tiếp nhận | Xác nhận nhận hồ sơ từ bộ phận một cửa |
| Yêu cầu phối hợp (`REQUEST_COLLABORATION`) | Gửi yêu cầu xin ý kiến phòng ban liên quan |
| Trình duyệt (`SUBMIT_FOR_APPROVAL`) | Trình hồ sơ lên lãnh đạo phê duyệt |
| Phân công lại (`REASSIGN`) | Chuyển hồ sơ cho chuyên viên khác trong phòng |
| Ghi chú nội bộ | Thêm ghi chú trao đổi nội bộ |

**Nhận thông báo khi:**
- Hồ sơ được chuyển đến phòng
- Được phân công xử lý hồ sơ cụ thể
- Lãnh đạo từ chối và trả hồ sơ về kèm lý do
- Hồ sơ sắp hết hoặc đã hết hạn SLA

---

### C. Phòng/Ban Liên quan – Phối hợp (`COORDINATOR`)

**Vai trò nghiệp vụ:** Cung cấp ý kiến chuyên môn, xác minh hoặc bổ sung thông tin theo yêu cầu phối hợp từ phòng chuyên môn.

**Quyền thao tác trên hồ sơ:**

| Hành động | Mô tả |
|-----------|-------|
| Phản hồi phối hợp (`RESPOND_COLLABORATION`) | Gửi ý kiến/kết quả xác minh về phòng chuyên môn |
| Từ chối phối hợp | Từ chối tham gia với lý do cụ thể |
| Yêu cầu gia hạn | Xin thêm thời gian để hoàn thiện phản hồi |

**Nhận thông báo khi:**
- Có yêu cầu phối hợp mới từ phòng chuyên môn
- Nhắc nhở khi sắp hết hạn phản hồi

---

### D. Lãnh đạo / Cấp phê duyệt (`APPROVER` / `LEADER`)

**Vai trò nghiệp vụ:** Xem xét và ra quyết định phê duyệt, từ chối hoặc trả hồ sơ về xử lý lại.

**Quyền thao tác trên hồ sơ:**

| Hành động | Mô tả |
|-----------|-------|
| Phê duyệt (`APPROVE`) | Đồng ý hồ sơ, chuyển sang trả kết quả |
| Từ chối (`REJECT`) | Từ chối hồ sơ kèm lý do rõ ràng |
| Trả về chuyên môn (`RETURN`) | Yêu cầu xem xét lại, kèm ý kiến |

**Nhận thông báo khi:**
- Có hồ sơ mới được trình duyệt
- Hồ sơ chờ phê duyệt sắp/đã quá hạn SLA
- Hệ thống leo thang (escalation) hồ sơ quá hạn lên cấp trên

---

### E. Bộ phận Trả kết quả (`RETURN_DESK`)

**Vai trò nghiệp vụ:** Xác nhận hồ sơ đã hoàn tất, phát hành kết quả, bàn giao cho người dân.

**Quyền thao tác trên hồ sơ:**

| Hành động | Mô tả |
|-----------|-------|
| Xác nhận trả kết quả (`CLOSE`) | Đánh dấu hồ sơ đã được trả thành công |
| Hoãn trả kết quả | Ghi nhận lý do hoãn (người dân chưa đến nhận) |

**Nhận thông báo khi:**
- Hồ sơ đã hoàn tất, sẵn sàng để trả kết quả cho người dân

---

### F. Quản trị / Giám sát (`SYSTEM_ADMIN` / `MONITOR`)

**Vai trò nghiệp vụ:**  
- **SYSTEM_ADMIN:** Toàn quyền cấu hình hệ thống, quản lý workflow, xử lý sự cố.  
- **MONITOR:** Chỉ xem và giám sát, không thay đổi cấu hình.

**Quyền cấu hình Workflow (SYSTEM_ADMIN):**

| Quyền | Mô tả |
|-------|-------|
| Tạo mới workflow | Định nghĩa quy trình cho từng loại hồ sơ |
| Chỉnh sửa bản nháp | Thay đổi cấu hình workflow ở trạng thái DRAFT hoặc TESTING |
| Tạo phiên bản mới | Clone workflow đang ACTIVE để chỉnh sửa, không ảnh hưởng bản đang chạy |
| Kích hoạt workflow | Đưa workflow qua vòng đời đến ACTIVE sau khi test thành công |
| Vô hiệu hóa / Lưu trữ | Tắt workflow khi không còn sử dụng |
| Quản lý bộ phận & vai trò | Định nghĩa phòng ban, vị trí, quy tắc phân công |
| Cấu hình SLA | Đặt thời hạn và leo thang cho từng bước |
| Xem báo cáo vận hành | Theo dõi hồ sơ tồn đọng, quá hạn, lỗi luồng |

> **Nguyên tắc cốt lõi:** Admin thiết lập quy trình; người dùng nghiệp vụ chỉ thao tác theo quy trình đã được duyệt.

**Nhận thông báo khi:**
- Không tìm thấy người xử lý (fallback) tại bất kỳ bước nào
- Phát sinh lỗi luồng hệ thống
- Backlog hồ sơ bất thường (cảnh báo tồn đọng)
- Leo thang cấp cao nhất (escalation level cuối)

---

### Người dân / Người nộp hồ sơ (`SUBMITTER` / `CITIZEN`)

**Vai trò nghiệp vụ:** Nộp hồ sơ và theo dõi tiến trình xử lý.

**Quyền thao tác:**

| Hành động | Mô tả |
|-----------|-------|
| Nộp hồ sơ | Tạo hồ sơ mới trong hệ thống |
| Bổ sung hồ sơ | Nộp tài liệu bổ sung theo yêu cầu |
| Theo dõi trạng thái | Xem tiến trình xử lý (nhật ký rút gọn) |
| Nhận thông báo | Nhận push notification khi hồ sơ chuyển trạng thái |

**Giới hạn:** Người dân chỉ thấy thông tin hồ sơ của chính mình; không thấy tên cán bộ cụ thể xử lý, ghi chú nội bộ, hay lý do từ chối nội bộ.

---

## Sơ đồ luồng phòng ban

```
Người dân
   │
   ▼
┌─────────────────────────────────┐
│  A. Bộ phận Tiếp nhận / Một cửa │
│  Role: RECEPTION                │
└──────────────┬──────────────────┘
               │  (hồ sơ hợp lệ)
               ▼
┌─────────────────────────────────┐
│  B. Phòng Chuyên môn            │
│  Role: SPECIALIST               │
└──────────────┬──────────────────┘
               │  (cần phối hợp)
               ├──────────────────────────────────────┐
               │                                      ▼
               │                    ┌─────────────────────────────────┐
               │                    │  C. Phòng/Ban Liên quan         │
               │                    │  Role: COORDINATOR              │
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
│  Role: RETURN_DESK              │
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

---

## Ma trận phân quyền theo trạng thái Workflow

| Trạng thái Workflow | Admin | Tester | Người dùng cuối (Dân) | Cán bộ nghiệp vụ |
|---------------------|-------|--------|------------------------|------------------|
| `DRAFT` | Cấu hình, chỉnh sửa | ✗ | ✗ | ✗ |
| `READY_FOR_TEST` | Chuyển trạng thái | ✗ | ✗ | ✗ |
| `TESTING` | Giám sát, xem kết quả | Thực hiện test | ✗ | ✗ |
| `READY_FOR_PRODUCTION` | Xem xét, Publish | ✗ | ✗ | ✗ |
| `ACTIVE` | Giám sát, báo cáo | ✗ | Nộp hồ sơ, theo dõi | Xử lý hồ sơ |
| `INACTIVE` / `ARCHIVED` | Tra cứu, restore | ✗ | ✗ | Tra cứu lịch sử |

---

## Ma trận phân quyền xem dữ liệu

### Xem nhật ký hồ sơ (Case Audit Log)

| Vai trò | Phạm vi xem |
|---------|-------------|
| Cán bộ xử lý | Chỉ hồ sơ mình đang/đã xử lý |
| Trưởng phòng | Toàn bộ hồ sơ trong phòng ban |
| Lãnh đạo phê duyệt | Hồ sơ đã qua bước phê duyệt của mình |
| Admin / Giám sát | Tất cả hồ sơ trong hệ thống |
| Người dân | Nhật ký rút gọn của hồ sơ chính mình (không thấy nội dung nội bộ) |

### Xem nhật ký cấu hình Workflow (Config Audit Log)

| Vai trò | Phạm vi xem |
|---------|-------------|
| Admin | Toàn bộ lịch sử cấu hình tất cả workflow |
| Admin cấp cao | Như Admin + có thể export |
| Giám sát (Monitor) | Chỉ xem, không export |

### Xem báo cáo vận hành

| Vai trò | Quyền |
|---------|-------|
| Admin / SYSTEM_ADMIN | Xem toàn bộ báo cáo vận hành: tồn đọng, quá hạn, lỗi luồng |
| MONITOR | Xem báo cáo **theo phòng ban** (không xem tên cán bộ cụ thể), không thay đổi cấu hình |
| Lãnh đạo | Xem báo cáo thuộc phạm vi phê duyệt của mình |
| Trưởng phòng | Xem tình trạng hồ sơ trong phòng ban |
| Cán bộ | Không có quyền xem báo cáo tổng hợp |

---

## Chiến lược phân công thông báo theo bộ phận

| Bộ phận | Thông báo theo |
|---------|---------------|
| Tiếp nhận (A) | **Phòng ban** – queue chung chưa assign |
| Chuyên môn (B) | **Người được phân công** hoặc **vai trò (role)** |
| Phối hợp (C) | **Phòng ban** theo từng yêu cầu phối hợp |
| Phê duyệt (D) | **Vai trò** (`APPROVER` / `LEADER`) |
| Quá hạn (mọi bước) | **Người xử lý hiện tại + Trưởng bộ phận** |

---

## Nguyên tắc phân quyền

1. **Admin thiết lập – Nghiệp vụ vận hành:** Admin cấu hình quy trình; cán bộ nghiệp vụ chỉ thao tác theo quy trình đã duyệt. Cán bộ không thể thay đổi cấu hình bước, điều kiện chuyển trạng thái hay quy tắc thông báo.

2. **Người dùng cuối chỉ thấy ACTIVE:** Workflow ở trạng thái DRAFT, TESTING, READY_FOR_TEST, READY_FOR_PRODUCTION hoàn toàn ẩn với người dùng cuối (người dân và cán bộ nghiệp vụ thông thường).

3. **Tester được chỉ định và phải là cán bộ nội bộ:** Trong giai đoạn TESTING, chỉ nhóm tester được admin chỉ định mới có quyền tương tác với workflow. Tester phải là cán bộ nội bộ của tổ chức – không sử dụng nhân sự bên ngoài. Người dùng cuối thực tế không được tiếp cận.

4. **Phạm vi xem theo vai trò:** Dữ liệu hồ sơ, nhật ký và báo cáo được hiển thị theo phạm vi vai trò – không ai xem được dữ liệu ngoài phạm vi được phép.

5. **Audit log bất biến:** Chỉ service account ứng dụng được ghi log; không có user nào (kể cả Admin) được sửa hoặc xóa bản ghi audit log.

6. **Phân quyền theo đơn vị hành chính:** Hệ thống hỗ trợ phân quyền theo đơn vị hành chính (tỉnh/huyện/xã). Người dùng chỉ có thể xem và xử lý hồ sơ thuộc phạm vi đơn vị hành chính được phân công.

7. **DEACTIVATE khẩn cấp:** Admin có quyền DEACTIVATE workflow đang ACTIVE ngay lập tức mà không cần approval flow. Thao tác này được ghi nhận đầy đủ trong audit log.

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên | Câu trả lời |
|---|---------|----------------|-------------|
| 1 | Có cần phân quyền theo đơn vị hành chính (tỉnh/huyện/xã) không? | Trung bình | **Có** |
| 2 | Một người dùng có thể đảm nhiệm nhiều vai trò cùng lúc không (vd: vừa là Chuyên viên vừa là Trưởng phòng)? | Cao | Đang chờ xác nhận |
| 3 | Ai có quyền DEACTIVATE khẩn cấp workflow đang ACTIVE? Cần approval flow không? | Cao | **Admin – không cần approval flow** |
| 4 | Tester có phải là cán bộ nội bộ hay có thể là bên ngoài? | Trung bình | **Cán bộ nội bộ** |
| 5 | MONITOR có được xem tên cán bộ cụ thể trong báo cáo, hay chỉ xem theo phòng ban? | Thấp | **Xem theo phòng ban** |
