# 07. Thử nghiệm và UAT

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này mô tả quy trình thử nghiệm (testing) và nghiệm thu người dùng (UAT – User Acceptance Testing) cho workflow xử lý hồ sơ trước khi đưa vào sử dụng chính thức. Bao gồm: lý do phải test, cách tổ chức nhóm tester, điều kiện tham gia, phân biệt tester và user thật, và tiêu chí hoàn thành UAT.

---

## Vì sao workflow phải qua thử nghiệm?

Không giống phần mềm thông thường, mỗi workflow được admin tạo ra là một cấu hình nghiệp vụ riêng biệt. Một lỗi trong cấu hình có thể dẫn đến:

- Hồ sơ bị "mắc kẹt" không chuyển được sang bước tiếp theo
- Phân công sai phòng ban hoặc sai người dẫn đến không ai xử lý
- Thông báo sai đối tượng hoặc thiếu thông báo quan trọng
- SLA tính sai hoặc không cảnh báo khi quá hạn
- Người dùng cuối trải nghiệm quy trình không nhất quán hoặc sai sót

**Giai đoạn TESTING** là lớp bảo vệ bắt buộc để phát hiện và xử lý các lỗi này trước khi người dùng thật bị ảnh hưởng.

---

## Thời gian thử nghiệm

### Thời gian tối thiểu bắt buộc

- **Mặc định đề xuất:** 7 ngày làm việc kể từ khi workflow chuyển sang TESTING
- Admin **không được publish** trước khi hết thời gian tối thiểu, ngay cả khi không còn lỗi mở
- Thời gian tối thiểu có thể cấu hình theo từng workflow (admin thiết lập khi tạo workflow)

### Ví dụ cấu hình thời gian test

| Loại workflow | Thời gian test tối thiểu đề xuất | Lý do |
|--------------|----------------------------------|-------|
| Workflow đơn giản (3–4 bước) | 5 ngày | Ít tình huống cần test |
| Workflow trung bình (5–7 bước) | 7 ngày | Tiêu chuẩn |
| Workflow phức tạp (8+ bước, liên phòng) | 14 ngày | Cần test nhiều tình huống edge case |

### Thời gian thực tế

- Thời gian test bắt đầu tính từ khi admin chuyển workflow sang trạng thái TESTING
- Hệ thống hiển thị: "Thời gian test còn lại: X ngày"
- Khi đủ thời gian tối thiểu VÀ không còn lỗi mở → workflow được phép chuyển sang READY_FOR_PRODUCTION

---

## Nhóm người dùng tham gia test

### Thành phần nhóm tester

Nhóm tester nên bao gồm đại diện từ tất cả các vai trò trong workflow:

| Vai trò | Mô tả trong test | Số lượng đề xuất |
|---------|-----------------|------------------|
| Tester đại diện người dùng cuối | Thực hiện nộp hồ sơ, theo dõi trạng thái, nhận thông báo | 1–2 người |
| Tester đại diện cán bộ tiếp nhận | Kiểm tra quy trình tiếp nhận, yêu cầu bổ sung | 1 người |
| Tester đại diện chuyên viên xử lý | Kiểm tra phân công, thẩm định, chuyển bước | 1–2 người |
| Tester đại diện lãnh đạo phê duyệt | Kiểm tra phê duyệt, từ chối, escalation | 1 người |
| Tester đại diện bộ phận trả kết quả | Kiểm tra trả kết quả, đóng hồ sơ | 1 người |
| Admin / Kỹ thuật | Theo dõi log, phát hiện lỗi kỹ thuật | 1 người |

### Điều kiện để được tham gia test

- Được admin chỉ định và thêm vào danh sách nhóm test của workflow
- Đã được hướng dẫn về mục tiêu test và các tình huống cần kiểm tra
- Hiểu rõ: hồ sơ tạo trong giai đoạn test là **hồ sơ giả** – không phải hồ sơ vận hành thật
- Cam kết ghi nhận phản ánh lỗi qua công cụ issue tracking của hệ thống (không báo cáo qua email/chat riêng)

---

## Phạm vi test

### Checklist tình huống cần test tối thiểu

Trong giai đoạn TESTING, nhóm tester phải thực hiện và xác nhận ít nhất các kịch bản sau:

**Luồng chính (Happy path):**
- [ ] Nộp hồ sơ mới thành công, hồ sơ đến đúng bước tiếp nhận
- [ ] Cán bộ tiếp nhận xác nhận tiếp nhận, hồ sơ chuyển đúng bước
- [ ] Chuyên viên nhận được phân công, xử lý và chuyển trình phê duyệt
- [ ] Lãnh đạo phê duyệt, hồ sơ chuyển bước trả kết quả
- [ ] Bộ phận trả kết quả xác nhận trả, hồ sơ đóng thành công

**Luồng ngoại lệ:**
- [ ] Yêu cầu bổ sung hồ sơ: người dùng nhận đúng thông báo, nộp bổ sung, cán bộ tiếp nhận lại
- [ ] Lãnh đạo từ chối: người dùng nhận thông báo kèm lý do, hồ sơ đóng
- [ ] Hồ sơ quá hạn SLA: hệ thống đánh dấu quá hạn, escalation gửi đúng người
- [ ] Chuyển liên phòng ban (nếu workflow có bước này)

**Thông báo:**
- [ ] Tất cả các notification gửi đúng đối tượng, đúng nội dung, đúng thời điểm
- [ ] Không gửi notification thừa hoặc notification sai người

**Phân quyền:**
- [ ] Người không được phân công không thể thao tác hồ sơ của người khác
- [ ] Người dùng cuối không nhìn thấy hồ sơ của người khác
- [ ] Tester không thể thao tác workflow ở trạng thái DRAFT hoặc ACTIVE

---

## Quy tắc sử dụng trong giai đoạn TESTING

### Quy tắc dành cho tester

1. Chỉ được thao tác với hồ sơ được tạo trong giai đoạn test, không được tác động vào hồ sơ thật
2. Phải ghi nhận mọi lỗi và góp ý qua công cụ issue tracking của hệ thống
3. Không được tự ý đóng hoặc xử lý lỗi thay cho đội kỹ thuật/admin
4. Khi hoàn thành test, phải xác nhận rõ ràng trong hệ thống: "Đã hoàn thành kiểm thử"

### Quy tắc dành cho admin

1. Workflow TESTING không hiển thị cho người dùng cuối thật
2. Nếu cần chỉnh sửa cấu hình sau khi phát hiện lỗi: quay về DRAFT, sửa, rồi mới chuyển lại TESTING (thời gian test bắt đầu tính lại)
3. Theo dõi tiến độ giải quyết issue định kỳ, không để issue tồn đọng
4. Không publish khi còn issue đang mở, ngay cả dưới áp lực thời gian

### Quy tắc về dữ liệu test

- Hồ sơ tạo trong giai đoạn TESTING được đánh dấu là "hồ sơ test"
- Hồ sơ test không được tính vào báo cáo vận hành thực tế
- Sau khi kết thúc test, hồ sơ test có thể được xóa hoặc lưu trữ riêng

---

## Phân biệt Tester và User thật

| Khía cạnh | Tester | User thật (Người dùng cuối) |
|-----------|--------|---------------------------|
| Được chỉ định bởi | Admin (thêm vào danh sách nhóm test) | Không cần chỉ định |
| Loại tài khoản | Tài khoản test hoặc tài khoản có flag "test user" | Tài khoản thường |
| Workflow được dùng | Workflow ở trạng thái TESTING | Chỉ workflow ở trạng thái ACTIVE |
| Hồ sơ tạo ra | Hồ sơ test (không phải hồ sơ vận hành) | Hồ sơ thật |
| Mục đích | Kiểm tra quy trình, phát hiện lỗi | Xử lý hành chính thật |
| Quyền phản ánh lỗi | Có – qua issue tracking | Không có kênh trực tiếp trong hệ thống |
| Khi workflow ACTIVE | Không còn được dùng workflow này ở chế độ test | Bắt đầu có thể dùng |

**Cơ chế kỹ thuật phân biệt:**
- Hệ thống lọc workflow hiển thị theo trạng thái và theo thuộc tính người dùng
- User không có trong danh sách tester sẽ không thấy workflow TESTING
- Dữ liệu hồ sơ test được gắn nhãn riêng để không lẫn với dữ liệu thật

---

## Điều kiện hoàn thành UAT

Để workflow được phép chuyển sang READY_FOR_PRODUCTION, phải thỏa mãn **đồng thời** các điều kiện:

| # | Điều kiện | Kiểm tra bởi |
|---|-----------|-------------|
| 1 | Đã trải qua đủ thời gian thử nghiệm tối thiểu | Hệ thống tự động kiểm tra |
| 2 | Không còn issue/lỗi nào ở trạng thái mở (NEW, ACKNOWLEDGED, IN_PROGRESS, RETEST_PENDING) | Hệ thống tự động kiểm tra |
| 3 | Tất cả các tình huống trong checklist tối thiểu đã được test và xác nhận | Tester xác nhận |
| 4 | Ít nhất một tester đã ký xác nhận "Hoàn thành kiểm thử" trong hệ thống | Tester thực hiện |
| 5 | Admin xem xét và đồng ý chuyển sang READY_FOR_PRODUCTION | Admin thực hiện |

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Thời gian test tối thiểu mặc định là bao nhiêu? Admin có được cấu hình khác không? | Cao |
| 2 | Nếu admin chỉnh sửa cấu hình trong lúc TESTING, có cần reset thời gian test không? | Cao |
| 3 | Tester có cần checklist rõ ràng bắt buộc, hay tự do test theo hiểu biết? | Trung bình |
| 4 | Có cần lưu evidence kiểm thử (screenshot, file log) không, hay chỉ cần xác nhận trong hệ thống? | Trung bình |
| 5 | Nếu tester phát hiện lỗi nghiêm trọng sau khi đã ký xác nhận, có cơ chế thu hồi xác nhận không? | Trung bình |
