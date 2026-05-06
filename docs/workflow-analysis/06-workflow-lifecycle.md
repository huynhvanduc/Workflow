# 06. Vòng đời Workflow

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này mô tả các trạng thái mà một workflow có thể tồn tại, ý nghĩa nghiệp vụ của từng trạng thái, các chuyển đổi (transition) được phép, và quy tắc kiểm soát quyền sử dụng theo từng đối tượng (actor).

---

## Các trạng thái workflow

Một workflow trải qua các trạng thái sau:

```
DRAFT → READY_FOR_TEST → TESTING → READY_FOR_PRODUCTION → ACTIVE → INACTIVE / ARCHIVED
```

### DRAFT (Bản nháp)

- **Ý nghĩa:** Workflow vừa được tạo mới hoặc đang được chỉnh sửa. Chưa sẵn sàng để thử nghiệm.
- **Ai được thao tác:** Admin
- **Ai được dùng:** Không ai được dùng để xử lý hồ sơ thật
- **Đặc điểm:**
  - Admin có thể chỉnh sửa toàn bộ cấu hình: bước, action, transition, notification, assignment rule
  - Không ảnh hưởng đến hồ sơ nào đang xử lý
  - Có thể xóa hoàn toàn nếu chưa cần

---

### READY_FOR_TEST (Sẵn sàng kiểm thử)

- **Ý nghĩa:** Admin đã hoàn thành cấu hình và xác nhận workflow sẵn sàng để đưa vào thử nghiệm.
- **Ai được thao tác:** Admin (chuyển trạng thái)
- **Ai được dùng:** Chưa ai được dùng thật
- **Đặc điểm:**
  - Đây là điểm chốt để admin "đóng băng" cấu hình và yêu cầu kiểm thử
  - Hệ thống có thể tự động khởi tạo giai đoạn test sau bước này
  - Admin vẫn có thể quay về DRAFT nếu phát hiện cần chỉnh sửa thêm

---

### TESTING (Đang thử nghiệm)

- **Ý nghĩa:** Workflow đang trong giai đoạn kiểm thử nội bộ. Chỉ nhóm user được chỉ định (tester) mới được thao tác.
- **Ai được thao tác:** Admin (giám sát), Tester (thực hiện test case)
- **Ai được dùng:** **Chỉ tester hoặc nhóm user được chỉ định** – user thật chưa được dùng
- **Đặc điểm:**
  - Thời gian thử nghiệm được xác định trước (ví dụ: 7 ngày, 14 ngày)
  - Tester phản ánh lỗi, góp ý thông qua cơ chế issue/feedback tracking
  - Admin có thể quay về DRAFT nếu phát hiện lỗi nghiêm trọng cần cấu hình lại
  - Hồ sơ tạo trong giai đoạn TESTING là hồ sơ test – không phải hồ sơ vận hành thật

> ⚠️ **Lưu ý quan trọng:** Workflow ở trạng thái TESTING tuyệt đối không được áp dụng cho người dùng cuối thực tế. Nếu test trong môi trường production, phải giới hạn phạm vi người dùng nghiêm ngặt.

---

### READY_FOR_PRODUCTION (Sẵn sàng đưa vào chính thức)

- **Ý nghĩa:** Giai đoạn thử nghiệm đã kết thúc, không còn lỗi mở, workflow đủ điều kiện để được kích hoạt chính thức.
- **Ai được thao tác:** Admin (xem xét và ra quyết định kích hoạt)
- **Ai được dùng:** Chưa ai được dùng thật
- **Đặc điểm:**
  - Hệ thống hoặc admin xác nhận: đã qua thời gian test tối thiểu + không còn issue mở
  - Admin thực hiện thao tác "Publish" / "Activate" để chuyển sang ACTIVE
  - Đây là bước checkpoint cuối cùng trước khi workflow đi vào vận hành

---

### ACTIVE (Đang hoạt động chính thức)

- **Ý nghĩa:** Workflow đã được kích hoạt và đang được sử dụng trong vận hành thực tế.
- **Ai được thao tác:** Cán bộ tiếp nhận, Chuyên viên xử lý, Lãnh đạo phê duyệt, Bộ phận trả kết quả
- **Ai được dùng:** **Tất cả người dùng cuối thực tế** và cán bộ nghiệp vụ được phân công
- **Đặc điểm:**
  - Workflow ACTIVE **không được chỉnh sửa trực tiếp**
  - Mọi thay đổi cấu hình phải tạo phiên bản mới (clone → DRAFT → ... → ACTIVE)
  - Hồ sơ đang xử lý theo phiên bản hiện tại vẫn tiếp tục đến khi hoàn thành
  - Admin có thể DEACTIVATE trong trường hợp khẩn cấp (có hướng dẫn riêng về xử lý hồ sơ dở dang)

---

### INACTIVE / ARCHIVED (Vô hiệu hóa / Lưu trữ)

- **Ý nghĩa:** Workflow không còn được sử dụng. Lưu lại cho mục đích lịch sử, tra cứu và audit.
- **Ai được thao tác:** Admin
- **Ai được dùng:** Không ai – workflow này không nhận hồ sơ mới
- **Đặc điểm:**
  - Hồ sơ đã hoàn thành vẫn được lưu lại và tra cứu được
  - Không thể tạo hồ sơ mới theo workflow đã ARCHIVED
  - Admin có thể restore (khôi phục) nếu cần thiết, nhưng phải qua lại vòng đời từ DRAFT

---

## Sơ đồ chuyển trạng thái

```
          +----------+
          |  DRAFT   |◄────────────────────────────────┐
          +----+-----+                                  │
               │ Admin xác nhận sẵn sàng test           │ Admin phát hiện lỗi,
               ▼                                        │ cần cấu hình lại
     +-----------------+                                │
     | READY_FOR_TEST  |────────────────────────────────┘
     +-------+---------+
             │ Bắt đầu giai đoạn test
             ▼
         +---------+
         | TESTING |◄──── Tester phản ánh lỗi
         +----+----+
              │ Hết thời gian test + không còn lỗi mở
              ▼
  +------------------------+
  | READY_FOR_PRODUCTION   |
  +----------+-------------+
             │ Admin xác nhận Publish
             ▼
          +--------+
          | ACTIVE |
          +---+----+
              │ Admin vô hiệu hóa
              ▼
   +--------------------+
   | INACTIVE / ARCHIVED|
   +--------------------+
```

---

## Ma trận quyền sử dụng theo trạng thái

| Trạng thái | Admin | Tester | Người dùng cuối | Cán bộ nghiệp vụ |
|------------|-------|--------|-----------------|------------------|
| DRAFT | Cấu hình, chỉnh sửa | ✗ | ✗ | ✗ |
| READY_FOR_TEST | Chuyển trạng thái | ✗ | ✗ | ✗ |
| TESTING | Giám sát, xem kết quả | Thực hiện test | ✗ | ✗ |
| READY_FOR_PRODUCTION | Xem xét, Publish | ✗ | ✗ | ✗ |
| ACTIVE | Giám sát, báo cáo | ✗ | Nộp hồ sơ, theo dõi | Xử lý hồ sơ |
| INACTIVE/ARCHIVED | Tra cứu, restore | ✗ | ✗ | Tra cứu lịch sử |

---

## Quy tắc nghiệp vụ quan trọng

### Quy tắc bất biến (non-negotiable)

1. **Workflow ACTIVE không được sửa trực tiếp.** Mọi thay đổi phải tạo phiên bản mới.
2. **Người dùng cuối thực tế chỉ được dùng workflow ở trạng thái ACTIVE.**
3. **Workflow ở trạng thái TESTING chỉ dành cho tester hoặc nhóm user được chỉ định.**
4. **Điều kiện chuyển sang ACTIVE: hết thời gian test tối thiểu VÀ không còn issue/lỗi mở.**

### Quy tắc về phiên bản

- Khi cần cập nhật workflow đang ACTIVE:
  1. Clone workflow hiện tại → tạo phiên bản mới (DRAFT)
  2. Chỉnh sửa phiên bản mới
  3. Đưa phiên bản mới qua đầy đủ vòng đời: DRAFT → TESTING → ACTIVE
  4. Khi phiên bản mới ACTIVE, phiên bản cũ tự động hoặc thủ công chuyển sang INACTIVE

- Hồ sơ đang xử lý dở theo phiên bản cũ: tiếp tục theo phiên bản cũ cho đến khi hoàn thành (không bị chuyển đột ngột sang phiên bản mới)

---

## Ghi chú và câu hỏi mở

| # | Ghi chú / Câu hỏi | Trạng thái |
|---|-------------------|------------|
| 1 | Thời gian test tối thiểu bắt buộc: **15 ngày** | ✅ Đã chốt |
| 2 | Có cho phép admin rút ngắn thời gian test trong trường hợp khẩn không? | ✅ Đã chốt: Có |
| 3 | Khi DEACTIVATE khẩn cấp, hồ sơ đang xử lý dở **chuyển vào lưu trữ tồn đọng** | ✅ Đã chốt |
| 4 | Cần xác nhận: hệ thống có tự động chuyển TESTING → READY_FOR_PRODUCTION sau khi hết thời gian, hay admin phải thao tác thủ công? | ✅ Đã chốt: Không tự động; hệ thống gửi thông báo để admin thao tác thủ công |
