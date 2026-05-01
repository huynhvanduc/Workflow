# 10. Checklist cho người dùng cuối

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này tổng hợp checklist nghiệp vụ từ góc nhìn của người dùng cuối (công dân, tổ chức nộp hồ sơ), bao gồm: hành trình tương tác với hệ thống, những gì người dùng cần nhận được tại mỗi bước, tiêu chí nghiệm thu từ phía người dùng, và phân biệt giữa tester và user thật trong quá trình thử nghiệm.

---

## Danh sách đối tượng cần hỗ trợ

Trong phạm vi hệ thống workflow xử lý hồ sơ, người dùng cuối bao gồm:

| Đối tượng | Mô tả | Đặc điểm cần chú ý |
|-----------|-------|-------------------|
| Công dân cá nhân | Người nộp hồ sơ thủ tục hành chính cho bản thân | Có thể ít quen với hệ thống số; cần hướng dẫn rõ |
| Tổ chức / Doanh nghiệp | Đại diện tổ chức nộp hồ sơ hành chính | Thường nộp nhiều hồ sơ, cần theo dõi danh sách |
| Người ủy quyền | Người nộp hồ sơ thay mặt người khác | Cần cơ chế ủy quyền rõ ràng (xác minh tư cách) |

---

## Hành trình người dùng cuối

### Nộp hồ sơ

**Mô tả:** Người dùng khởi đầu quá trình bằng việc chọn dịch vụ và nộp hồ sơ.

**Các bước người dùng thực hiện:**
1. Đăng nhập hệ thống (xác thực danh tính)
2. Tìm kiếm và chọn dịch vụ công tương ứng
3. Đọc thông tin yêu cầu hồ sơ (danh mục tài liệu cần nộp)
4. Điền thông tin vào biểu mẫu
5. Tải lên tài liệu đính kèm
6. Xem lại và xác nhận nộp
7. Nhận xác nhận nộp thành công (mã hồ sơ)

**Người dùng cần nhận được:**
- Giao diện biểu mẫu rõ ràng, dễ điền
- Danh sách tài liệu cần đính kèm kèm mô tả rõ (loại file, kích thước)
- Thông báo xác nhận ngay sau khi nộp: mã hồ sơ, thời gian xử lý dự kiến, ngày hẹn trả kết quả
- Lưu lịch sử hồ sơ đã nộp để tham chiếu sau

---

### Nhận yêu cầu bổ sung

**Mô tả:** Cán bộ tiếp nhận phát hiện hồ sơ thiếu và yêu cầu người dùng bổ sung.

**Những gì người dùng trải nghiệm:**
1. Nhận thông báo (push notification / email) hồ sơ cần bổ sung
2. Truy cập vào hồ sơ và xem danh sách tài liệu cần bổ sung kèm lý do cụ thể
3. Tải lên tài liệu bổ sung theo hướng dẫn
4. Xác nhận gửi bổ sung
5. Chờ cán bộ xem lại và tiếp nhận

**Người dùng cần nhận được:**
- Thông báo kịp thời và rõ ràng: thiếu tài liệu gì, tại sao thiếu, hạn bổ sung là khi nào
- Giao diện bổ sung đơn giản: không cần nộp lại toàn bộ hồ sơ, chỉ cần bổ sung phần thiếu
- Thông báo xác nhận sau khi nộp bổ sung thành công

---

### Theo dõi trạng thái xử lý

**Mô tả:** Người dùng muốn biết hồ sơ đang ở bước nào và có vấn đề gì không.

**Những gì người dùng cần:**
1. Xem danh sách hồ sơ đã nộp và trạng thái hiện tại
2. Theo dõi hồ sơ đang ở bước nào trong quy trình
3. Xem thời gian dự kiến hoàn thành
4. Nhận thông báo khi có thay đổi trạng thái

**Người dùng cần nhận được:**
- Giao diện trạng thái hồ sơ dễ đọc: đang ở bước nào, đã qua những bước nào
- Ngôn ngữ thân thiện: tránh dùng thuật ngữ kỹ thuật nội bộ (không hiển thị "TESTING", "IN_PROGRESS"...)
- Thông báo chủ động khi hồ sơ chuyển sang trạng thái mới (không cần phải vào hệ thống kiểm tra thủ công)
- Nếu hồ sơ bị chậm hơn dự kiến: thông báo với lý do chung (tránh gây lo lắng)

---

### Nhận kết quả / Thông báo hoàn tất

**Mô tả:** Hồ sơ đã được xử lý xong, người dùng nhận kết quả.

**Các trường hợp kết quả:**

| Kết quả | Thông báo cần có |
|---------|-----------------|
| Được phê duyệt | Thông báo thành công, hướng dẫn nhận kết quả (trực tiếp hoặc trực tuyến) |
| Bị từ chối | Thông báo từ chối kèm lý do rõ ràng, hướng dẫn khiếu nại hoặc nộp lại |
| Kết quả sẵn sàng trả | Mời đến nhận trực tiếp (nếu cần) hoặc download trực tuyến |

**Người dùng cần nhận được:**
- Thông báo kết quả ngay khi có, không cần đợi người dùng chủ động kiểm tra
- Lý do từ chối (nếu có): rõ ràng, có cơ sở pháp lý hoặc nghiệp vụ để người dùng hiểu
- Hướng dẫn bước tiếp theo: nhận kết quả như thế nào, ở đâu, trong thời hạn bao lâu
- Lưu hồ sơ và kết quả để người dùng tra cứu về sau

---

## Checklist nghiệp vụ từ góc nhìn người dùng cuối

Danh sách này được dùng để nghiệm thu hệ thống từ phía người dùng cuối:

### Nộp hồ sơ

- [ ] Người dùng có thể tìm thấy dịch vụ cần nộp dễ dàng (tìm kiếm hoặc danh mục)
- [ ] Người dùng xem được danh sách tài liệu cần chuẩn bị trước khi bắt đầu điền
- [ ] Biểu mẫu có hướng dẫn điền rõ ràng cho từng trường
- [ ] Người dùng có thể lưu nháp hồ sơ và tiếp tục điền sau
- [ ] Sau khi nộp, người dùng nhận ngay xác nhận với mã hồ sơ và thời gian xử lý dự kiến
- [ ] Chỉ workflow ở trạng thái **ACTIVE** mới hiển thị cho người dùng cuối thật

### Theo dõi và bổ sung

- [ ] Người dùng xem được danh sách và trạng thái tất cả hồ sơ đã nộp
- [ ] Trạng thái hồ sơ hiển thị bằng ngôn ngữ thân thiện, không phải mã kỹ thuật
- [ ] Người dùng nhận thông báo khi hồ sơ chuyển sang trạng thái mới
- [ ] Khi cần bổ sung: người dùng nhận thông báo với danh sách cụ thể và hạn bổ sung
- [ ] Giao diện bổ sung đơn giản, chỉ yêu cầu phần còn thiếu, không nộp lại toàn bộ
- [ ] Người dùng nhận xác nhận sau khi nộp bổ sung thành công

### Nhận kết quả

- [ ] Người dùng nhận thông báo ngay khi kết quả có sẵn
- [ ] Nếu được phê duyệt: thông báo rõ cách nhận kết quả (địa điểm, thời gian, hoặc download)
- [ ] Nếu bị từ chối: lý do từ chối được trình bày rõ ràng, dễ hiểu
- [ ] Người dùng có thể lưu / tải về kết quả xử lý
- [ ] Hồ sơ và kết quả được lưu trong lịch sử để tra cứu về sau

---

## Checklist phân biệt Tester và User thật

Trong giai đoạn thử nghiệm, hệ thống phải đảm bảo:

### Đối với Tester

- [ ] Tester được admin thêm vào danh sách nhóm test của workflow cụ thể
- [ ] Tester được phép thao tác với workflow ở trạng thái TESTING
- [ ] Hồ sơ tester tạo ra được đánh dấu là "hồ sơ test" – không lẫn với hồ sơ thật
- [ ] Tester có thể ghi nhận phản ánh lỗi qua công cụ issue tracking trong hệ thống
- [ ] Khi workflow chuyển sang ACTIVE, tester không còn quyền test workflow đó nữa

### Đối với User thật

- [ ] User thật **không thể nhìn thấy** workflow đang ở trạng thái TESTING
- [ ] User thật **không thể nộp hồ sơ** theo workflow TESTING, dù biết URL trực tiếp
- [ ] User thật **không bị nhầm lẫn** giữa quy trình test và quy trình thật
- [ ] Không có dữ liệu test nào hiển thị trên giao diện của user thật
- [ ] Khi workflow chuyển sang ACTIVE, user thật mới có thể dùng

---

## Tiêu chí nghiệm thu từ góc nhìn người dùng

Các tiêu chí này được dùng trong quá trình UAT để xác nhận hệ thống sẵn sàng cho người dùng thật:

| # | Tiêu chí | Cách kiểm tra | Mức độ ưu tiên |
|---|---------|--------------|----------------|
| 1 | Người dùng nộp hồ sơ thành công không cần hỗ trợ kỹ thuật | Tester giả lập user thao tác độc lập | Bắt buộc |
| 2 | Thông báo xác nhận nộp hồ sơ đến trong vòng 1 phút | Đo thời gian thực tế | Bắt buộc |
| 3 | Trạng thái hồ sơ được cập nhật và hiển thị chính xác | Tester kiểm tra sau mỗi bước xử lý | Bắt buộc |
| 4 | Thông báo yêu cầu bổ sung có đủ thông tin (cái gì, tại sao, hạn khi nào) | Tester kiểm tra nội dung thông báo | Bắt buộc |
| 5 | Người dùng nhận thông báo kết quả trước khi tự vào kiểm tra | Kiểm tra thứ tự nhận thông báo | Bắt buộc |
| 6 | Lý do từ chối được trình bày rõ ràng, dễ hiểu với người dùng thường | Nhóm người không có kiến thức kỹ thuật đọc và hiểu được | Bắt buộc |
| 7 | Người dùng có thể tra cứu lịch sử hồ sơ đã nộp dễ dàng | Tester kiểm tra giao diện lịch sử | Quan trọng |
| 8 | Hệ thống không hiển thị thông tin nội bộ không cần thiết cho người dùng | Kiểm tra giao diện người dùng cuối | Quan trọng |
| 9 | Thông báo bằng tiếng Việt chuẩn, không có lỗi chính tả hoặc thuật ngữ khó hiểu | Review nội dung tất cả notification | Quan trọng |
| 10 | Giao diện hoạt động đúng trên thiết bị di động | Tester test trên mobile | Quan trọng |

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Người dùng có cần xác thực định danh điện tử (VneID, VNPT eKYC...) khi nộp hồ sơ không? | Cao |
| 2 | Người dùng có thể lưu nháp hồ sơ và tiếp tục điền vào lần sau không? | Cao |
| 3 | Trường hợp người dùng muốn rút hồ sơ đã nộp: có hỗ trợ không? Điều kiện rút là gì? | Trung bình |
| 4 | Người dùng có được thông báo nếu hồ sơ bị chậm hơn dự kiến không? | Trung bình |
| 5 | Có cần hỗ trợ đa ngôn ngữ (tiếng Anh, tiếng dân tộc thiểu số) không? | Thấp |
