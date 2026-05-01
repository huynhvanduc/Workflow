# 08. Phản ánh lỗi và theo dõi fix

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này mô tả cơ chế ghi nhận, phân loại, và theo dõi xử lý các phản ánh lỗi và góp ý phát sinh trong giai đoạn thử nghiệm workflow. Bao gồm: định nghĩa các loại phản ánh, thông tin cần ghi nhận, vòng đời trạng thái lỗi, quy trình xử lý, và ảnh hưởng đến quyết định publish workflow.

---

## Khái niệm phản ánh / Lỗi / Góp ý

Trong giai đoạn TESTING, tester có thể ghi nhận ba loại phản ánh:

### Lỗi (Bug / Defect)

- **Định nghĩa:** Hành vi của hệ thống không đúng với thiết kế hoặc mô tả nghiệp vụ đã thống nhất
- **Ví dụ:**
  - Hồ sơ không chuyển sang bước tiếp theo sau khi cán bộ xác nhận tiếp nhận
  - Notification gửi sai người hoặc không gửi
  - SLA không được tính đúng sau khi tiếp nhận
  - Tester thấy được hồ sơ của tester khác (vi phạm phân quyền)
- **Ảnh hưởng đến publish:** Lỗi **phải được đóng** trước khi workflow được publish

### Góp ý cải thiện (Improvement / Enhancement)

- **Định nghĩa:** Đề xuất cải thiện chức năng hoặc trải nghiệm, không phải lỗi sai
- **Ví dụ:**
  - "Thông báo nên có thêm tên bước hiện tại"
  - "Giao diện xử lý hồ sơ nên có nút truy cập nhanh"
  - "Nên có tóm tắt lý do từ chối ở trang kết quả"
- **Ảnh hưởng đến publish:** Không chặn publish; có thể triển khai ở phiên bản sau

### Câu hỏi / Làm rõ (Question / Clarification)

- **Định nghĩa:** Tester không chắc về hành vi mong đợi, cần admin hoặc nghiệp vụ xác nhận
- **Ví dụ:**
  - "Khi lãnh đạo từ chối, hồ sơ có thể nộp lại không?"
  - "SLA tính theo ngày dương lịch hay ngày làm việc?"
- **Ảnh hưởng đến publish:** Cần làm rõ; nếu là câu hỏi về lỗi tiềm ẩn thì cần xác nhận trước khi publish

---

## Thông tin cần ghi nhận cho mỗi phản ánh

Khi tester ghi nhận một phản ánh, hệ thống cần lưu trữ ít nhất các thông tin sau:

| Trường | Bắt buộc? | Mô tả |
|--------|-----------|-------|
| Tiêu đề ngắn | ✓ | Tóm tắt vấn đề trong 1 dòng |
| Loại phản ánh | ✓ | Lỗi / Góp ý / Câu hỏi |
| Mô tả chi tiết | ✓ | Mô tả đầy đủ: đang làm gì, kỳ vọng gì, thực tế là gì |
| Bước tái hiện | ✓ | Các bước để tái hiện vấn đề |
| Tình huống nghiệp vụ liên quan | ✓ | Tình huống nào đang test khi phát sinh lỗi |
| Mức độ nghiêm trọng | ✓ | Critical / High / Medium / Low |
| Workflow liên quan | ✓ | Tên và phiên bản workflow đang test |
| Screenshot / Video | Khuyến khích | Bằng chứng minh họa vấn đề |
| Người ghi nhận | Tự động | Tên tester, timestamp |
| Trạng thái | Tự động | Bắt đầu từ NEW |

### Mức độ nghiêm trọng

| Mức độ | Mô tả | Ví dụ |
|--------|-------|-------|
| **Critical** | Chức năng cốt lõi bị vỡ hoàn toàn, không thể test tiếp | Không nộp được hồ sơ, hệ thống crash |
| **High** | Tính năng quan trọng bị lỗi, ảnh hưởng nghiêm trọng đến nghiệp vụ | Phân công sai phòng ban, notification sai người |
| **Medium** | Lỗi ảnh hưởng đến trải nghiệm nhưng có cách workaround | Thông báo sai nội dung nhỏ, UI hiển thị sai |
| **Low** | Lỗi nhỏ, không ảnh hưởng đến chức năng chính | Lỗi chính tả, icon hiển thị sai |

---

## Trạng thái xử lý lỗi

Vòng đời của một phản ánh lỗi đi qua các trạng thái sau:

### NEW (Mới ghi nhận)

- **Ý nghĩa:** Phản ánh vừa được tester tạo, chưa có người tiếp nhận
- **Ai xử lý:** Chờ admin hoặc đội kỹ thuật xem xét
- **Chuyển sang:** ACKNOWLEDGED (khi được tiếp nhận) hoặc REJECTED (nếu không hợp lệ)

---

### ACKNOWLEDGED (Đã tiếp nhận)

- **Ý nghĩa:** Admin hoặc đội kỹ thuật đã xem xét và xác nhận phản ánh là hợp lệ
- **Ai xử lý:** Admin gán người phụ trách và đánh giá mức độ ưu tiên
- **Chuyển sang:** IN_PROGRESS (khi bắt đầu xử lý)

---

### IN_PROGRESS (Đang xử lý)

- **Ý nghĩa:** Đội kỹ thuật hoặc admin đang tiến hành điều tra và sửa lỗi
- **Ai xử lý:** Developer hoặc admin (chỉnh sửa cấu hình workflow)
- **Chuyển sang:** FIXED (sau khi sửa xong, cần tester kiểm tra lại)

---

### FIXED (Đã sửa – Chờ xác nhận)

- **Ý nghĩa:** Lỗi đã được sửa, đang chờ tester kiểm tra lại để xác nhận
- **Ai xử lý:** Tester thực hiện retest
- **Chuyển sang:**
  - RETEST_PENDING nếu tester chưa kịp kiểm tra lại
  - CLOSED nếu tester xác nhận lỗi đã được sửa
  - IN_PROGRESS nếu tester phát hiện lỗi vẫn còn hoặc sinh ra lỗi mới

---

### RETEST_PENDING (Chờ kiểm tra lại)

- **Ý nghĩa:** Lỗi đã được báo cáo là sửa xong nhưng tester chưa có thời gian kiểm tra lại
- **Ai xử lý:** Tester cần ưu tiên thực hiện retest
- **Chuyển sang:** CLOSED (xác nhận OK) hoặc IN_PROGRESS (vẫn còn lỗi)

> ⚠️ **Lưu ý quan trọng:** Trạng thái RETEST_PENDING được coi là **lỗi chưa đóng** theo điều kiện publish. Tester cần xử lý kịp thời để không chặn việc publish workflow.

---

### CLOSED (Đã đóng)

- **Ý nghĩa:** Tester đã xác nhận lỗi được sửa thành công và không còn tái hiện
- **Ai xử lý:** Tester thực hiện xác nhận đóng
- **Trạng thái cuối:** Lỗi đã được giải quyết hoàn toàn

---

### REJECTED (Bị từ chối)

- **Ý nghĩa:** Phản ánh không hợp lệ: không phải lỗi thật, nằm ngoài phạm vi, hoặc trùng lặp
- **Ai xử lý:** Admin hoặc đội kỹ thuật ra quyết định
- **Trường hợp dùng:**
  - Lỗi do người dùng thao tác sai, không phải lỗi hệ thống
  - Phản ánh trùng với issue đã có
  - Vấn đề thuộc ngoài phạm vi của workflow đang test
  - Đây là hành vi đúng theo thiết kế, tester hiểu nhầm

> Khi REJECT, admin phải ghi rõ lý do để tester hiểu và không tạo lại phản ánh tương tự.

---

## Sơ đồ vòng đời trạng thái

```
          +-------+
          |  NEW  |
          +---+---+
              |
     +--------+--------+
     |                 |
     ▼                 ▼
+----------+       +----------+
|ACKNOWLEDGED|     | REJECTED |  (Kết thúc)
+-----+----+       +----------+
      |
      ▼
+-----------+
|IN_PROGRESS|◄────────────────┐
+-----------+                 │
      │                       │
      ▼                       │
   +-------+      Lỗi vẫn còn │
   | FIXED |──────────────────┘
   +---+---+
       │
  +----+----+
  │         │
  ▼         ▼
+--------+ +----------------+
| CLOSED | |RETEST_PENDING  |
+--------+ +-------+--------+
                   │
          +--------+--------+
          │                 │
          ▼                 ▼
       +--------+      +-----------+
       | CLOSED |      |IN_PROGRESS| (Lỗi vẫn còn)
       +--------+      +-----------+
```

---

## Quy trình xử lý phản ánh

### Bước 1: Tester ghi nhận phản ánh

- Tester tạo issue mới trong hệ thống tracking
- Điền đầy đủ thông tin: tiêu đề, mô tả, bước tái hiện, mức độ
- Đính kèm screenshot nếu có

### Bước 2: Admin / Kỹ thuật tiếp nhận

- Xem xét phản ánh trong vòng 1 ngày làm việc (đề xuất)
- Đánh giá: hợp lệ hay không?
  - Nếu hợp lệ: chuyển sang ACKNOWLEDGED, gán người xử lý
  - Nếu không hợp lệ: chuyển sang REJECTED kèm lý do

### Bước 3: Xử lý lỗi

- Đội kỹ thuật điều tra và sửa lỗi (sửa code hoặc admin chỉnh sửa cấu hình)
- Cập nhật trạng thái IN_PROGRESS
- Sau khi sửa xong: chuyển sang FIXED và thông báo tester retest

### Bước 4: Tester retest

- Tester kiểm tra lại theo bước tái hiện ban đầu
- Nếu lỗi đã hết: đóng issue (CLOSED)
- Nếu lỗi vẫn còn hoặc có lỗi mới: chuyển lại IN_PROGRESS kèm mô tả chi tiết

### Bước 5: Theo dõi tổng thể

- Admin theo dõi tổng số issue còn mở theo ngày
- Ưu tiên xử lý issue Critical và High trước
- Đảm bảo không còn issue mở trước khi xem xét publish

---

## Liên hệ giữa lỗi và quyết định publish workflow

Điều kiện chặn publish dựa trên trạng thái issue:

```
count(issues WHERE status IN ('NEW', 'ACKNOWLEDGED', 'IN_PROGRESS', 'RETEST_PENDING')) = 0
```

Tức là: workflow **chỉ được publish** khi **toàn bộ issue** đã ở trạng thái **CLOSED hoặc REJECTED**.

| Trạng thái issue | Có chặn publish không? |
|-----------------|----------------------|
| NEW | ✓ Chặn |
| ACKNOWLEDGED | ✓ Chặn |
| IN_PROGRESS | ✓ Chặn |
| RETEST_PENDING | ✓ Chặn |
| FIXED | ✓ Chặn (chờ tester xác nhận) |
| CLOSED | ✗ Không chặn |
| REJECTED | ✗ Không chặn |

---

## Dashboard theo dõi issue (đề xuất giao diện)

Giao diện quản lý issue của admin nên hiển thị:

- Tổng số issue theo trạng thái (bảng tổng hợp)
- Danh sách issue đang chặn publish (status mở)
- Tiến độ theo thời gian: số issue mở theo ngày
- Thời gian còn lại của giai đoạn test
- Nút "Publish" chỉ hiển thị khi đủ điều kiện; kèm thông báo lý do nếu chưa đủ

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Issue tracking có phải là module trong hệ thống hay dùng công cụ ngoài (Jira, GitHub Issues...)? | Cao |
| 2 | Ai có quyền REJECT một phản ánh? Chỉ admin hay cả kỹ thuật? | Trung bình |
| 3 | Có cần phân loại mức độ ưu tiên (priority) tách biệt với mức độ nghiêm trọng (severity) không? | Trung bình |
| 4 | Tester có thể tạo issue cho nhiều workflow cùng lúc hay mỗi session chỉ test một workflow? | Thấp |
| 5 | Issue của version workflow cũ có bị kế thừa sang version mới không? | Trung bình |
