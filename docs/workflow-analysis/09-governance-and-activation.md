# 09. Nguyên tắc quản trị và điều kiện kích hoạt

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này xác định các nguyên tắc quản trị (governance) cho workflow xử lý hồ sơ, điều kiện để một workflow được công bố chính thức (publish/activate), các điều kiện chặn (blocking conditions), khuyến nghị về phiên bản (versioning), và các rủi ro nghiệp vụ cần kiểm soát.

---

## Nguyên tắc quản trị chính

### 1. Workflow ACTIVE không được chỉnh sửa trực tiếp

Khi một workflow đang ở trạng thái **ACTIVE**, tất cả các thao tác chỉnh sửa cấu hình (thêm bước, thay đổi điều kiện, sửa notification) đều bị **khóa hoàn toàn**.

**Lý do:**
- Tránh thay đổi đột ngột ảnh hưởng đến hồ sơ đang xử lý dở dang
- Bảo đảm tính nhất quán: hồ sơ được tạo theo quy trình nào thì kết thúc theo quy trình đó
- Ngăn ngừa lỗi vận hành khó phát hiện do cấu hình thay đổi ngầm

**Cách thực hiện đúng:**
1. Clone workflow đang ACTIVE → tạo phiên bản mới ở trạng thái DRAFT
2. Chỉnh sửa trên phiên bản mới
3. Đưa phiên bản mới qua vòng đời đầy đủ (DRAFT → TESTING → ACTIVE)
4. Phiên bản cũ chuyển sang INACTIVE sau khi phiên bản mới được kích hoạt

---

### 2. Workflow mới phải qua thử nghiệm trước khi ACTIVE

Không có ngoại lệ: mọi workflow mới hoặc phiên bản mới đều phải trải qua giai đoạn **TESTING** với nhóm tester được chỉ định.

**Lý do:**
- Phát hiện lỗi cấu hình, logic sai, bước thiếu trước khi ảnh hưởng đến người dùng thật
- Cho phép cán bộ nghiệp vụ xác nhận luồng phù hợp thực tế trước khi vận hành
- Tạo cơ sở dữ liệu kiểm thử (test evidence) cho mục đích audit và truy xuất

---

### 3. Còn lỗi mở thì không được publish

Nếu có bất kỳ **issue/lỗi nào chưa được đóng (status ≠ CLOSED hoặc REJECTED)**, hệ thống hoặc quy trình quản trị phải **chặn** không cho phép chuyển workflow sang ACTIVE.

**Lý do:**
- Đảm bảo chất lượng: không đưa workflow có lỗi đã biết vào vận hành
- Buộc đội triển khai xử lý dứt điểm từng phản ánh trước khi công bố

> **Điều kiện kỹ thuật:** `count(issues WHERE status NOT IN ('CLOSED', 'REJECTED')) = 0`

---

### 4. Hết thời gian thử nghiệm tối thiểu mới được xét publish

Ngay cả khi không còn lỗi mở, workflow chỉ được phép chuyển sang **READY_FOR_PRODUCTION** sau khi đã trải qua thời gian thử nghiệm tối thiểu được xác định trước.

**Lý do:**
- Cần đủ thời gian để tester phát hiện các lỗi tiềm ẩn, edge case
- Tránh tình trạng "publish vội" sau khi đóng hết lỗi nhưng chưa test kỹ

> **Giá trị mặc định đề xuất:** 7 ngày (có thể do admin cấu hình theo từng loại workflow)

---

### 5. Người dùng cuối thực tế chỉ sử dụng workflow ACTIVE

Người dùng cuối (công dân, người nộp hồ sơ) chỉ có thể tương tác với workflow đã được kích hoạt chính thức.

**Lý do:**
- Tránh nhầm lẫn giữa hồ sơ test và hồ sơ thật
- Bảo vệ trải nghiệm người dùng: không để user tiếp cận quy trình chưa được kiểm duyệt
- Ngăn ngừa rủi ro pháp lý nếu hồ sơ "thật" bị xử lý theo workflow thử nghiệm

---

## Điều kiện để workflow được publish (Activation Criteria)

Để workflow được chuyển từ **READY_FOR_PRODUCTION → ACTIVE**, phải thỏa mãn **đồng thời** tất cả điều kiện sau:

| # | Điều kiện | Kiểm tra bởi |
|---|-----------|-------------|
| 1 | Workflow đã ở trạng thái TESTING đủ thời gian tối thiểu | Hệ thống tự động |
| 2 | Không còn issue/lỗi nào ở trạng thái mở (NEW, IN_PROGRESS, RETEST_PENDING...) | Hệ thống tự động |
| 3 | Ít nhất một tester đã xác nhận hoàn thành kiểm thử | Tester + Admin xác nhận |
| 4 | Admin thực hiện thao tác "Publish" / "Activate" | Admin thủ công |
| 5 | (Tùy chọn) Cấp có thẩm quyền phê duyệt việc công bố | Lãnh đạo / Admin cấp cao |

---

## Điều kiện chặn publish (Blocking Conditions)

Hệ thống **phải từ chối** thao tác publish nếu bất kỳ điều kiện nào dưới đây còn tồn tại:

| # | Điều kiện chặn | Hành động yêu cầu |
|---|----------------|-------------------|
| 1 | Thời gian thử nghiệm chưa đạt tối thiểu | Chờ đủ thời gian |
| 2 | Còn issue ở trạng thái NEW, ACKNOWLEDGED, IN_PROGRESS | Xử lý và đóng tất cả issue |
| 3 | Còn issue ở trạng thái RETEST_PENDING (chờ kiểm tra lại) | Tester phải retest và cập nhật trạng thái |
| 4 | Cấu hình workflow còn thiếu bước bắt buộc (theo validation rule) | Admin hoàn thiện cấu hình |
| 5 | Chưa có tester nào xác nhận hoàn thành test | Tester thực hiện xác nhận |

> **Giao diện đề xuất:** Hiển thị rõ danh sách điều kiện còn thiếu khi admin cố gắng publish, kèm link trực tiếp đến từng issue cần xử lý.

---

## Khuyến nghị về versioning

### Nguyên tắc đặt tên phiên bản

Đề xuất sử dụng định dạng: `vMAJOR.MINOR`

- **MAJOR**: tăng khi thay đổi cấu trúc luồng (thêm/bỏ bước, đổi điều kiện chuyển trạng thái)
- **MINOR**: tăng khi thay đổi nhỏ (sửa notification, sửa tên bước, điều chỉnh SLA)

Ví dụ: `v1.0` → `v1.1` (sửa notification) → `v2.0` (thêm bước phê duyệt mới)

### Quản lý phiên bản song song

- Tại một thời điểm, chỉ có **một phiên bản ACTIVE** cho mỗi loại hồ sơ
- Hồ sơ đang xử lý dở theo phiên bản cũ: tiếp tục đến khi hoàn thành, **không bị chuyển sang phiên bản mới**
- Phiên bản cũ chuyển sang INACTIVE khi phiên bản mới ACTIVE; lưu lại cho mục đích tra cứu

### Lịch sử phiên bản

- Hệ thống phải lưu trữ toàn bộ lịch sử các phiên bản workflow
- Mỗi hồ sơ phải ghi lại phiên bản workflow đã sử dụng khi tạo
- Admin có thể tra cứu cấu hình của phiên bản bất kỳ trong lịch sử

---

## Rủi ro nghiệp vụ và biện pháp kiểm soát

| # | Rủi ro | Mức độ | Biện pháp kiểm soát |
|---|--------|--------|---------------------|
| 1 | Admin publish workflow có lỗi, ảnh hưởng đến hồ sơ thật của người dùng | Cao | Chặn publish khi còn issue mở; yêu cầu xác nhận từ tester |
| 2 | Admin sửa trực tiếp workflow ACTIVE, gây nhầm lẫn cho hồ sơ đang xử lý | Cao | Khóa chỉnh sửa khi ACTIVE; bắt buộc tạo phiên bản mới |
| 3 | Người dùng cuối nộp hồ sơ theo workflow đang test, tạo dữ liệu sai | Cao | Ẩn workflow TESTING khỏi giao diện người dùng cuối |
| 4 | Workflow bị deactivate khẩn cấp khi có hồ sơ đang xử lý dở | Trung bình | Admin có quyền deactivate không cần approval; cần có hướng dẫn xử lý hồ sơ dở; không tự động hủy hồ sơ |
| 5 | Tester xác nhận test nhưng chưa cover đủ các tình huống | Trung bình | Quy định checklist test tối thiểu; lưu evidence |
| 6 | Nhiều phiên bản workflow tồn tại song song gây nhầm lẫn cho cán bộ | Thấp | Hiển thị rõ phiên bản trên giao diện xử lý hồ sơ |
| 7 | Audit log không đầy đủ, không truy xuất được ai đã publish/deactivate | Trung bình | Ghi đầy đủ audit log: ai, khi nào, thao tác gì |
| 8 | Workflow clone từ phiên bản cũ nhưng không cập nhật các thay đổi mới | Thấp | Cảnh báo khi clone: liệt kê những gì đã thay đổi so với gốc |

---

## Checklist quản trị trước khi publish

Trước khi admin thực hiện thao tác Publish, nên kiểm tra:

- [ ] Workflow đã trải qua đủ thời gian thử nghiệm tối thiểu
- [ ] Không còn issue/lỗi nào ở trạng thái mở
- [ ] Ít nhất một tester đã xác nhận kết quả kiểm thử
- [ ] Cấu hình notification đã được kiểm tra và xác nhận
- [ ] Cấu hình phân công (assignment rule) đã được xác nhận phù hợp cơ cấu tổ chức thực tế
- [ ] Đã thông báo cho cán bộ nghiệp vụ sẽ sử dụng workflow này
- [ ] Đã ghi nhận version và thông tin release (ai publish, thời điểm nào, mô tả thay đổi)

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên | Câu trả lời |
|---|---------|----------------|-------------|
| 1 | Thời gian thử nghiệm tối thiểu mặc định là bao lâu? Admin có được cấu hình không? | Cao | Đang chờ xác nhận |
| 2 | Ai có quyền DEACTIVATE khẩn cấp workflow đang ACTIVE? Cần approval flow không? | Cao | **Admin – không cần approval flow** |
| 3 | Hồ sơ đang dở khi deactivate khẩn cấp: tự động hủy, giữ nguyên, hay cần admin xử lý từng cái? | Cao | Đang chờ xác nhận |
| 4 | Có cần bước phê duyệt từ cấp trên trước khi admin được publish không? | Trung bình | Đang chờ xác nhận |
| 5 | Audit log cần lưu bao lâu? Có quy định pháp lý nào liên quan không? | Trung bình | Đang chờ xác nhận |
