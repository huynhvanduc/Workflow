# 01. Tổng quan

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Bối cảnh bài toán

Hệ thống xử lý hồ sơ hành chính công hiện tại vận hành theo luồng cứng (hard-coded). Mỗi khi quy trình nghiệp vụ thay đổi – dù chỉ thêm một bước phê duyệt hoặc đổi phòng ban tiếp nhận – đội kỹ thuật phải can thiệp trực tiếp vào mã nguồn, kiểm thử lại và triển khai lại toàn bộ hệ thống.

Điều này dẫn đến:

- Phụ thuộc hoàn toàn vào đội kỹ thuật cho mọi thay đổi nghiệp vụ nhỏ
- Chi phí và thời gian triển khai cao, chậm đáp ứng yêu cầu thực tế
- Rủi ro lỗi cao mỗi khi phải sửa code trong môi trường production
- Không thể kiểm soát và chuẩn hóa quy trình ở cấp nghiệp vụ

---

## Vấn đề hiện tại

Dưới đây là các điểm đau (pain points) đang tồn tại trong cách xử lý hồ sơ hiện tại:

### Quy trình phụ thuộc code

- Luồng xử lý hồ sơ bị hard-code, không thể cấu hình từ giao diện quản trị
- Thêm bước, bỏ bước, đổi điều kiện chuyển trạng thái đều phải sửa code

### Phân công thủ công

- Không có cơ chế tự động phân công hồ sơ cho cán bộ phù hợp theo phòng ban / vai trò
- Việc chuyển hồ sơ giữa các bộ phận thực hiện thủ công, dễ bỏ sót

### Thông báo thiếu nhất quán

- Thông báo cho cán bộ xử lý và người dùng cuối phụ thuộc thủ công hoặc gọi trực tiếp
- Dễ sót, chậm, không đồng nhất giữa các loại hồ sơ

### Không có giai đoạn kiểm thử

- Khi thay đổi quy trình, không có cơ chế thử nghiệm an toàn trước khi áp dụng thật
- Mọi thay đổi đều ảnh hưởng ngay đến hồ sơ đang xử lý trong production

### Khó theo dõi và báo cáo

- Không có cơ chế theo dõi SLA, cảnh báo quá hạn tự động
- Báo cáo vận hành phụ thuộc truy vấn thủ công từ database

---

## Mục tiêu nghiệp vụ

Xây dựng hệ thống cho phép **admin cấu hình linh hoạt** quy trình xử lý hồ sơ (workflow) mà không cần thay đổi mã nguồn.

Cụ thể, hệ thống mới phải cho phép:

1. **Admin định nghĩa workflow** qua giao diện quản trị:
   - Cấu hình các bước (step), điều kiện chuyển trạng thái (transition)
   - Gán quy tắc phân công (assignment rule) theo phòng ban hoặc vai trò
   - Thiết lập quy tắc thông báo (notification rule) cho từng bước và từng đối tượng
   - Cài đặt SLA và cơ chế leo thang khi quá hạn

2. **Quy trình an toàn trước khi công bố:**
   - Workflow mới tạo phải trải qua giai đoạn thử nghiệm (TESTING) trước khi đưa vào sử dụng thật
   - Chỉ sau khi test thành công và không còn lỗi mở, workflow mới được kích hoạt (ACTIVE)

3. **Người dùng cuối được hỗ trợ xuyên suốt:**
   - Nộp hồ sơ và theo dõi trạng thái xử lý
   - Nhận thông báo kịp thời khi hồ sơ chuyển bước
   - Nhận yêu cầu bổ sung kèm lý do rõ ràng

---

## Phạm vi phân tích

Bộ tài liệu này tập trung vào phân tích nghiệp vụ. Các nội dung trong phạm vi bao gồm:

**Trong phạm vi (in scope):**

- Cơ chế admin cấu hình workflow qua giao diện
- Vòng đời workflow: từ DRAFT đến ACTIVE và ARCHIVED
- Giai đoạn thử nghiệm, điều kiện công bố chính thức
- Quy tắc phân công và thông báo theo vai trò / phòng ban
- Hành trình người dùng cuối khi nộp và theo dõi hồ sơ
- Ghi nhận và theo dõi phản ánh lỗi trong giai đoạn test

**Ngoài phạm vi (out of scope):**

- Thiết kế kiến trúc kỹ thuật (database schema, API contract)
- Triển khai giao diện người dùng chi tiết (wireframe, prototype)
- Tích hợp với hệ thống bên ngoài (DVCQG, cổng dịch vụ công quốc gia)
- Phân tích hiệu năng và bảo mật hệ thống

---

## Kết quả mong muốn

Sau khi hoàn thành bộ tài liệu phân tích nghiệp vụ này, kết quả kỳ vọng:

- **Đội nghiệp vụ** có thể hiểu rõ và xác nhận luồng xử lý hồ sơ mà không cần đọc code
- **Đội kỹ thuật** có đủ ngữ cảnh để thiết kế hệ thống, API và database schema phù hợp
- **Đội kiểm thử** có đủ tiêu chí nghiệm thu (acceptance criteria) để lập test case
- **Admin hệ thống** hiểu rõ quyền hạn, trách nhiệm và giới hạn khi cấu hình workflow
- **Ban lãnh đạo** nắm được nguyên tắc quản trị và rủi ro cần kiểm soát

---

## Giả định ban đầu

Các giả định được đặt ra trong quá trình phân tích:

| # | Giả định |
|---|----------|
| 1 | Mỗi hồ sơ chỉ áp dụng đúng một workflow tại một thời điểm (không song song nhiều workflow) |
| 2 | Admin là người dùng hệ thống có quyền cao nhất, được tin tưởng tuyệt đối khi cấu hình |
| 3 | Giai đoạn thử nghiệm (TESTING) chỉ áp dụng cho user được chỉ định, không phải toàn bộ user |
| 4 | Khi workflow đang ACTIVE, không chỉnh sửa trực tiếp – mọi thay đổi phải tạo phiên bản mới |
| 5 | Thông báo ưu tiên kênh push notification; các kênh khác (email, SMS) là bổ sung |
| 6 | Workflow có thể áp dụng cho nhiều loại hồ sơ khác nhau với cùng cấu trúc luồng |
| 7 | Người dùng cuối chỉ nhìn thấy và tương tác với workflow ở trạng thái ACTIVE |
| 8 | Hệ thống hỗ trợ đa phòng ban: hồ sơ có thể đi qua nhiều bộ phận trong một workflow |

---

## Câu hỏi cần chốt thêm

Các câu hỏi nghiệp vụ đã được xác nhận với stakeholder:

| # | Câu hỏi | Mức độ ưu tiên | Người phụ trách | Kết quả xác nhận |
|---|---------|----------------|-----------------|------------------|
| 1 | Một hồ sơ có thể áp dụng nhiều workflow song song không, hay chỉ dùng một? | Cao | Nghiệp vụ | ✅ Một hồ sơ chỉ dùng đúng một workflow |
| 2 | Thời gian thử nghiệm tối thiểu bắt buộc là bao lâu? Admin có được rút ngắn không? | Cao | Nghiệp vụ | ✅ Không có giới hạn tối thiểu bắt buộc; thời gian thử nghiệm do admin chỉ định |
| 3 | Khi workflow ACTIVE bị vô hiệu hóa, hồ sơ đang xử lý dở sẽ xử lý thế nào? | Cao | Kỹ thuật + Nghiệp vụ | ✅ Hệ thống thông báo yêu cầu người dùng nộp lại hồ sơ (nếu đã có workflow thay thế) hoặc tạm dừng xử lý (nếu chưa có workflow thay thế) |
| 4 | Có cần cơ chế phân quyền theo đơn vị hành chính (tỉnh/huyện/xã) không? | Trung bình | Nghiệp vụ | ✅ Có |
| 5 | Mức độ tùy chỉnh notification theo từng địa phương có được phép không? | Trung bình | Nghiệp vụ | ✅ Có |
| 6 | Có cần audit log cho mọi thao tác cấu hình workflow của admin không? | Trung bình | Kỹ thuật | ✅ Có |
| 7 | Workflow có thể được clone/sao chép từ workflow đã có để tạo nhanh không? | Thấp | Nghiệp vụ | ✅ Có |
| 8 | Ai có quyền DEACTIVATE một workflow đang ACTIVE trong tình huống khẩn cấp? | Cao | Nghiệp vụ | ✅ Admin |
