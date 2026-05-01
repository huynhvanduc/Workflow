# 02. Đối tượng và phòng ban

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này xác định tất cả các đối tượng (actor) tham gia vào hệ thống workflow xử lý hồ sơ, mô tả vai trò và trách nhiệm của từng đối tượng, liệt kê các phòng ban liên quan, và làm rõ mối quan hệ giữa đối tượng với từng giai đoạn xử lý.

---

## Danh sách đối tượng tham gia

### Admin (Quản trị viên hệ thống)

- **Mô tả:** Người có quyền cao nhất trong hệ thống, chịu trách nhiệm cấu hình và quản lý toàn bộ workflow.
- **Trách nhiệm chính:**
  - Tạo, chỉnh sửa, xóa workflow definition
  - Cấu hình các bước (step), hành động (action), điều kiện chuyển trạng thái (transition)
  - Thiết lập quy tắc phân công (assignment rule) và thông báo (notification rule)
  - Cài đặt SLA và cơ chế leo thang khi quá hạn
  - Quản lý vòng đời workflow: từ DRAFT đến ACTIVE và ARCHIVED
  - Giám sát quá trình thử nghiệm và ra quyết định publish
- **Phạm vi quyền:** Toàn hệ thống; không bị giới hạn bởi phòng ban hay loại hồ sơ
- **Số lượng điển hình:** 1–3 người cho mỗi đơn vị triển khai

---

### Tester (Người kiểm thử)

- **Mô tả:** Người được chỉ định để thử nghiệm workflow trong giai đoạn TESTING. Có thể là cán bộ nghiệp vụ am hiểu quy trình, hoặc chuyên viên kiểm thử phần mềm.
- **Trách nhiệm chính:**
  - Thực hiện các test case theo kịch bản nghiệp vụ được giao
  - Nộp hồ sơ giả lập, xử lý từng bước trong workflow thử nghiệm
  - Phát hiện và phản ánh lỗi, góp ý về luồng xử lý
  - Xác nhận hoàn thành kiểm thử khi đạt tiêu chí nghiệm thu
- **Phạm vi quyền:** Chỉ được thao tác với workflow ở trạng thái TESTING và trong phạm vi nhóm test
- **Phân biệt với user thật:** Tester dùng tài khoản test, hồ sơ tạo ra là hồ sơ giả – không phải hồ sơ vận hành thật

---

### Người dùng cuối / Công dân

- **Mô tả:** Người nộp hồ sơ và nhận kết quả xử lý. Đây là đối tượng cuối cùng được phục vụ bởi hệ thống.
- **Trách nhiệm chính:**
  - Nộp hồ sơ theo mẫu và quy trình do admin cấu hình
  - Theo dõi trạng thái xử lý hồ sơ
  - Bổ sung hồ sơ theo yêu cầu của cán bộ tiếp nhận
  - Nhận kết quả và thông báo hoàn tất
- **Phạm vi quyền:** Chỉ thao tác với hồ sơ của bản thân; không xem được hồ sơ của người khác
- **Quy tắc quan trọng:** Người dùng cuối **chỉ** được tiếp cận workflow ở trạng thái **ACTIVE**. Workflow đang TESTING không hiển thị cho đối tượng này.

---

### Cán bộ tiếp nhận / Bộ phận một cửa

- **Mô tả:** Người tiếp nhận hồ sơ tại điểm tiếp xúc đầu tiên (quầy một cửa hoặc cổng dịch vụ trực tuyến).
- **Trách nhiệm chính:**
  - Kiểm tra tính đầy đủ và hợp lệ của hồ sơ
  - Tiếp nhận hồ sơ chính thức vào hệ thống
  - Yêu cầu người dùng bổ sung hồ sơ nếu thiếu
  - Chuyển hồ sơ đã tiếp nhận đến bước xử lý tiếp theo
- **Phạm vi quyền:** Thao tác với hồ sơ ở giai đoạn tiếp nhận; không tham gia vào các bước chuyên môn

---

### Chuyên viên xử lý

- **Mô tả:** Cán bộ chuyên môn thuộc phòng ban liên quan, chịu trách nhiệm thẩm định và xử lý nội dung hồ sơ.
- **Trách nhiệm chính:**
  - Nghiên cứu và thẩm định hồ sơ theo đúng chuyên môn
  - Xử lý, soạn thảo kết quả hoặc kiến nghị
  - Chuyển hồ sơ lên lãnh đạo phê duyệt hoặc chuyển phòng ban khác nếu cần
  - Cập nhật trạng thái xử lý và ghi chú vào hệ thống
- **Phạm vi quyền:** Thao tác với hồ sơ được phân công; không xem hồ sơ của chuyên viên khác (trừ khi được ủy quyền)

---

### Lãnh đạo / Người phê duyệt

- **Mô tả:** Người có thẩm quyền phê duyệt hồ sơ và đưa ra quyết định cuối cùng.
- **Trách nhiệm chính:**
  - Xem xét kết quả thẩm định của chuyên viên
  - Phê duyệt hoặc từ chối hồ sơ kèm lý do rõ ràng
  - Phê duyệt trong thời hạn SLA cho phép
  - Có thể ủy quyền phê duyệt cho cấp phó khi vắng mặt
- **Phạm vi quyền:** Thao tác với hồ sơ đang chờ phê duyệt trong phạm vi thẩm quyền

---

### Bộ phận trả kết quả

- **Mô tả:** Người hoặc bộ phận chịu trách nhiệm trả kết quả xử lý hồ sơ cho người dùng cuối.
- **Trách nhiệm chính:**
  - Xác nhận hồ sơ đã có kết quả và sẵn sàng trả
  - Thông báo cho người dùng đến nhận kết quả (trực tiếp hoặc qua hệ thống)
  - Ghi nhận việc đã trả kết quả vào hệ thống để đóng hồ sơ
- **Phạm vi quyền:** Thao tác với hồ sơ ở giai đoạn trả kết quả

---

### Quản trị / Giám sát vận hành

- **Mô tả:** Người theo dõi hiệu quả vận hành của toàn bộ hệ thống workflow, không can thiệp trực tiếp vào xử lý hồ sơ.
- **Trách nhiệm chính:**
  - Giám sát các chỉ số vận hành: SLA, tỷ lệ quá hạn, thời gian xử lý trung bình
  - Báo cáo tiến độ và kết quả xử lý hồ sơ theo định kỳ
  - Cảnh báo khi phát hiện bất thường (bottleneck, quá tải, vi phạm SLA)
  - Không có quyền thao tác trực tiếp vào hồ sơ hay cấu hình workflow
- **Phạm vi quyền:** Chỉ đọc (read-only) toàn bộ hệ thống; xem báo cáo và dashboard

---

## Danh sách phòng ban sử dụng

### Bộ phận tiếp nhận / Một cửa

- Là điểm đầu tiên nhận hồ sơ từ người dùng (cả trực tuyến lẫn trực tiếp)
- Chịu trách nhiệm kiểm tra sơ bộ và đưa hồ sơ vào hệ thống xử lý
- Có thể yêu cầu bổ sung hồ sơ trước khi tiếp nhận chính thức

### Phòng chuyên môn

- Đảm nhận việc thẩm định nội dung hồ sơ theo đúng lĩnh vực chuyên môn
- Có thể có nhiều phòng chuyên môn tham gia xử lý một hồ sơ (ví dụ: phòng tài nguyên môi trường, phòng xây dựng, phòng tài chính)
- Luồng xử lý giữa các phòng được cấu hình trong workflow

### Phòng / Bộ phận liên quan (Liên phòng)

- Các phòng ban được tham vấn ý kiến trong quá trình xử lý hồ sơ
- Hồ sơ có thể được chuyển đi lấy ý kiến và chuyển trở lại phòng chủ trì
- Thời hạn lấy ý kiến được quản lý bởi SLA của bước liên quan

### Lãnh đạo / Cấp có thẩm quyền

- Đưa ra quyết định cuối cùng về hồ sơ
- Cấp độ phê duyệt có thể cấu hình theo từng loại hồ sơ (trưởng phòng, phó giám đốc, giám đốc)
- Trong trường hợp từ chối, phải ghi nhận lý do rõ ràng

### Bộ phận trả kết quả

- Đảm nhận việc giao trả kết quả cho người dùng cuối
- Trong nhiều trường hợp có thể là bộ phận một cửa hoặc giao trả trực tuyến
- Xác nhận "đã trả" để đóng hồ sơ chính thức

### Quản trị / Giám sát

- Theo dõi toàn bộ hệ thống, không gắn với một loại hồ sơ cụ thể
- Tiếp nhận cảnh báo tự động khi SLA bị vi phạm hoặc hồ sơ quá hạn
- Tổng hợp báo cáo định kỳ để phục vụ Ban lãnh đạo

---

## Quan hệ giữa đối tượng và phòng ban

| Đối tượng | Phòng ban điển hình | Ghi chú |
|-----------|---------------------|---------|
| Admin | Phòng CNTT / Quản trị hệ thống | Không nhất thiết gắn với phòng nghiệp vụ |
| Tester | Bất kỳ phòng ban nào được chỉ định | Thường là cán bộ nghiệp vụ am hiểu quy trình |
| Người dùng cuối | Bên ngoài tổ chức (công dân, doanh nghiệp) | Không thuộc cơ cấu nội bộ |
| Cán bộ tiếp nhận | Bộ phận một cửa / Văn phòng | Điểm tiếp xúc đầu tiên |
| Chuyên viên xử lý | Phòng chuyên môn tương ứng | Nhiều phòng tham gia trong một workflow |
| Lãnh đạo phê duyệt | Lãnh đạo phòng / Ban / Giám đốc | Cấp độ phê duyệt tùy cấu hình workflow |
| Bộ phận trả kết quả | Bộ phận một cửa hoặc phòng chuyên môn | Tùy quy trình từng loại hồ sơ |
| Giám sát vận hành | Phòng kiểm soát / Ban lãnh đạo | Quyền đọc toàn hệ thống |

---

## Vai trò xử lý theo từng giai đoạn

| Giai đoạn | Đối tượng chính | Đối tượng hỗ trợ |
|-----------|-----------------|-----------------|
| Cấu hình workflow | Admin | — |
| Thử nghiệm | Tester | Admin (giám sát) |
| Nộp hồ sơ | Người dùng cuối | — |
| Tiếp nhận hồ sơ | Cán bộ tiếp nhận | Người dùng cuối (bổ sung) |
| Xử lý chuyên môn | Chuyên viên xử lý | Phòng ban liên quan |
| Phê duyệt | Lãnh đạo | Chuyên viên (trình) |
| Trả kết quả | Bộ phận trả kết quả | Người dùng cuối (nhận) |
| Giám sát toàn bộ | Quản trị vận hành | Admin |

---

## Ghi chú nghiệp vụ

- **Một người có thể đóng nhiều vai trò:** Ví dụ, trong đơn vị nhỏ, cán bộ tiếp nhận cũng có thể là chuyên viên xử lý.
- **Phân công linh hoạt theo workflow:** Mỗi workflow có thể cấu hình phân công khác nhau cho cùng một loại đối tượng.
- **Tránh nhầm lẫn tester và user thật:** Hệ thống cần cơ chế phân biệt rõ ràng tài khoản test và tài khoản thật, đặc biệt trong môi trường production.
- **Người dùng cuối không có quyền xem cấu hình workflow:** Họ chỉ tương tác qua giao diện nộp hồ sơ và theo dõi trạng thái.

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Có cần phân quyền quản trị theo đơn vị hành chính (tỉnh/huyện/xã) không? | Cao |
| 2 | Trường hợp lãnh đạo vắng, có cơ chế ủy quyền phê duyệt tạm thời không? | Cao |
| 3 | Một tài khoản có thể vừa là tester vừa là cán bộ nghiệp vụ thật không? | Trung bình |
| 4 | Admin có thể giới hạn tester chỉ test một số bước nhất định không? | Thấp |
