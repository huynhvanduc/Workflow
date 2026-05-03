# 03. Khái niệm Workflow

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này định nghĩa và làm rõ các khái niệm cốt lõi của hệ thống workflow xử lý hồ sơ, bao gồm cấu trúc, thành phần, và sự khác biệt giữa cấu hình (definition) và vận hành thực tế (instance). Đây là nền tảng ngữ nghĩa chung cho cả đội nghiệp vụ và đội kỹ thuật.

---

## Workflow definition là gì?

**Workflow definition** (cấu hình quy trình) là **bản mô tả trừu tượng** về luồng xử lý hồ sơ, được admin tạo ra qua giao diện quản trị. Đây là "khuôn mẫu" xác định:

- Các bước xử lý (step) và trình tự thực hiện
- Điều kiện chuyển từ bước này sang bước khác (transition)
- Ai được phân công xử lý tại mỗi bước (assignment rule)
- Thông báo nào được gửi đến ai tại mỗi bước (notification rule)
- Thời hạn xử lý tối đa tại mỗi bước (SLA)

**Đặc điểm:**
- Workflow definition là **tĩnh** – nó mô tả "quy trình phải làm gì", không phải "đang xử lý gì"
- Một definition có thể được sử dụng để tạo ra nhiều hồ sơ (instance) theo cùng quy trình
- Definition trải qua vòng đời riêng: DRAFT → TESTING → ACTIVE → ARCHIVED (xem file `06-workflow-lifecycle.md`)

---

## Workflow instance là gì?

**Workflow instance** (phiên làm việc thực tế) là **một lần chạy cụ thể** của workflow definition khi có hồ sơ được nộp.

Ví dụ: Khi công dân A nộp hồ sơ xin cấp phép xây dựng, hệ thống tạo ra một workflow instance mới dựa trên definition "Quy trình cấp phép xây dựng v1.2". Instance này theo dõi:

- Hồ sơ đang ở bước nào
- Ai đang xử lý
- Thời gian còn lại
- Lịch sử các thao tác đã thực hiện

**Đặc điểm:**
- Workflow instance là **động** – nó thay đổi trạng thái theo thời gian khi hồ sơ được xử lý
- Một definition ACTIVE có thể có nhiều instance đang chạy song song (nhiều hồ sơ cùng loại)
- Instance lưu lại toàn bộ lịch sử xử lý để phục vụ audit và tra cứu

---

## Thành phần chính của workflow

### Workflow (Quy trình tổng thể)

**Định nghĩa:** Đơn vị cấu hình cấp cao nhất, mô tả toàn bộ quy trình xử lý một loại hồ sơ từ đầu đến cuối.

**Thuộc tính chính:**
- Tên workflow và mô tả
- Loại hồ sơ áp dụng
- Phiên bản (version)
- Trạng thái vòng đời hiện tại (DRAFT, ACTIVE, ...)
- Thời gian test tối thiểu yêu cầu

**Ví dụ:** "Quy trình cấp phép xây dựng", "Quy trình đăng ký kinh doanh hộ cá thể"

---

### Step (Bước xử lý)

**Định nghĩa:** Một giai đoạn cụ thể trong quy trình xử lý hồ sơ. Mỗi bước đại diện cho một nhiệm vụ hoặc trạng thái của hồ sơ.

**Thuộc tính chính:**
- Tên và mô tả bước
- Phòng ban / vai trò chịu trách nhiệm tại bước này
- SLA (thời hạn xử lý tối đa)
- Trạng thái kết thúc của hồ sơ khi hoàn thành bước
- Danh sách action cho phép tại bước này

**Ví dụ các bước điển hình:**
- Bước 1: Tiếp nhận hồ sơ
- Bước 2: Thẩm định chuyên môn
- Bước 3: Trình lãnh đạo phê duyệt
- Bước 4: Trả kết quả

---

### Action (Hành động)

**Định nghĩa:** Một thao tác cụ thể mà người xử lý có thể thực hiện tại một bước nhất định.

**Thuộc tính chính:**
- Tên hành động (hiển thị trên giao diện)
- Loại hành động: chuyển tiếp (forward), trả về (return), từ chối (reject), đóng hồ sơ (close)
- Transition được kích hoạt khi thực hiện hành động
- Có yêu cầu ghi chú/lý do khi thực hiện không
- Điều kiện tiên quyết để action này xuất hiện

**Ví dụ:**
- "Tiếp nhận" → chuyển sang bước thẩm định
- "Yêu cầu bổ sung" → trả về người dùng với yêu cầu kèm lý do
- "Phê duyệt" → chuyển sang bước trả kết quả
- "Từ chối" → đóng hồ sơ với lý do từ chối

---

### Transition (Chuyển trạng thái)

**Định nghĩa:** Quy tắc xác định hồ sơ sẽ đến bước nào tiếp theo sau khi một action được thực hiện.

**Thuộc tính chính:**
- Bước nguồn (from step)
- Bước đích (to step)
- Action kích hoạt transition này
- Điều kiện bổ sung (nếu có): ví dụ, chỉ chuyển đến bước X nếu loại hồ sơ là Y

**Ví dụ:**
- `Tiếp nhận` --[action: Phê duyệt tiếp nhận]--> `Thẩm định`
- `Thẩm định` --[action: Trình phê duyệt]--> `Chờ phê duyệt lãnh đạo`
- `Chờ phê duyệt lãnh đạo` --[action: Phê duyệt]--> `Trả kết quả`
- `Chờ phê duyệt lãnh đạo` --[action: Từ chối]--> `Đã từ chối (đóng)`

---

### Assignment Rule (Quy tắc phân công)

**Định nghĩa:** Quy tắc xác định ai sẽ được giao xử lý hồ sơ tại mỗi bước.

**Các loại assignment rule:**

| Loại | Mô tả | Ví dụ |
|------|-------|-------|
| Theo phòng ban | Giao cho toàn bộ thành viên trong phòng ban | Giao cho Phòng Tài nguyên Môi trường |
| Theo vai trò | Giao cho người có vai trò nhất định | Giao cho "Chuyên viên thẩm định" |
| Theo người cụ thể | Giao cho một cá nhân được chỉ định | Giao cho Nguyễn Văn A |
| Round-robin | Phân phối luân phiên trong nhóm | Lần lượt giữa 3 chuyên viên |
| Theo tải công việc | Giao cho người có ít hồ sơ nhất | Người ít việc nhất trong phòng |

**Lưu ý:** Assignment rule được cấu hình theo từng bước trong workflow definition. Admin có thể thay đổi rule cho từng bước độc lập với các bước khác.

---

### Notification Rule (Quy tắc thông báo)

**Định nghĩa:** Quy tắc xác định khi nào gửi thông báo, gửi cho ai, và nội dung thông báo là gì.

**Thuộc tính chính:**
- Sự kiện kích hoạt: bước bắt đầu, hành động thực hiện, SLA sắp hết, quá hạn
- Đối tượng nhận: theo phòng ban, vai trò, người được phân công, người dùng cuối
- Kênh thông báo: push notification, email, SMS (ưu tiên push notification)
- Nội dung thông báo: mẫu (template) với biến động (tên hồ sơ, mã hồ sơ, deadline...)

**Chi tiết:** Xem thêm file `05-notification-rules.md`

---

### SLA / Overdue / Escalation

**SLA (Service Level Agreement – Thời hạn dịch vụ):**
- Thời gian tối đa được phép xử lý tại một bước nhất định
- Được cấu hình theo đơn vị ngày làm việc hoặc giờ
- SLA của toàn bộ workflow = tổng SLA của từng bước

**Overdue (Quá hạn):**
- Trạng thái xảy ra khi hồ sơ vượt quá thời hạn SLA mà chưa hoàn thành bước
- Hệ thống cần ghi nhận và đánh dấu hồ sơ quá hạn rõ ràng
- Cần cảnh báo cho người xử lý và người giám sát

**Escalation (Leo thang):**
- Cơ chế tự động khi hồ sơ quá hạn: thông báo leo lên cấp cao hơn
- Ví dụ: quá hạn 1 ngày → cảnh báo chuyên viên; quá hạn 3 ngày → cảnh báo trưởng phòng; quá hạn 5 ngày → cảnh báo giám đốc
- Ngưỡng leo thang và đối tượng nhận cảnh báo được cấu hình trong workflow

---

## Phân biệt cấu hình và vận hành thực tế

| Khía cạnh | Cấu hình (Definition) | Vận hành (Instance) |
|-----------|----------------------|---------------------|
| Bản chất | Khuôn mẫu quy trình | Một lần chạy cụ thể của khuôn mẫu |
| Ai tạo ra | Admin | Hệ thống (khi hồ sơ được nộp) |
| Khi nào thay đổi | Khi admin chỉnh sửa | Khi cán bộ xử lý thao tác |
| Số lượng | Một definition cho mỗi loại quy trình | Nhiều instance cùng lúc |
| Lưu trữ | Cấu hình nghiệp vụ | Dữ liệu xử lý hồ sơ |
| Có thể chỉnh sửa khi ACTIVE? | **Không** (phải tạo phiên bản mới) | Không thể thay đổi cấu hình, chỉ cập nhật trạng thái |

---

## Nguyên tắc thiết kế workflow

Khi admin thiết kế workflow, cần tuân theo các nguyên tắc sau:

1. **Mỗi bước chỉ có một trách nhiệm rõ ràng:** Tránh một bước gộp nhiều nhiệm vụ khác nhau.
2. **Transition phải có điều kiện rõ ràng:** Không để hồ sơ "mắc kẹt" do thiếu transition.
3. **Mỗi bước cần có ít nhất một action kết thúc:** Không để hồ sơ không có cách thoát ra.
4. **SLA phải thực tế:** Tổng SLA của workflow phải phù hợp với quy định pháp lý về thời hạn giải quyết.
5. **Assignment rule phải dự phòng:** Cần có phương án dự phòng khi người được phân công vắng mặt.
6. **Notification không được quá nhiễu:** Chỉ thông báo khi thực sự cần thiết, tránh spam.

---

## Phạm vi MVP đề xuất

Các thành phần tối thiểu cần có trong phiên bản đầu (MVP):

| Thành phần | MVP | Ghi chú |
|-----------|-----|---------|
| Workflow với nhiều bước | ✓ | Tối thiểu 4–6 bước |
| Action: chuyển tiếp, trả về, từ chối | ✓ | Các action cơ bản |
| Transition đơn giản (không điều kiện) | ✓ | Điều kiện phức tạp để sau |
| Assignment rule theo phòng ban / vai trò | ✓ | Round-robin và load balancing để sau |
| Notification cơ bản qua push | ✓ | Email/SMS là bổ sung |
| SLA theo bước | ✓ | Escalation nâng cao để sau |
| Vòng đời DRAFT → ACTIVE | ✓ | Bao gồm giai đoạn TESTING |
| Conditional transition | Không | Có thể thêm ở phiên bản sau |
| Assignment theo tải công việc | Không | Thêm khi có dữ liệu vận hành |
| Workflow song song (parallel steps) | Không | Phức tạp, để phiên bản sau |

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Workflow có hỗ trợ bước song song (nhiều phòng ban xử lý đồng thời một hồ sơ) không? | Cao |
| 2 | Transition có điều kiện dựa vào dữ liệu hồ sơ (ví dụ: chỉ phê duyệt nếu giá trị > X) có cần không? | Cao |
| 3 | SLA tính theo giờ làm việc hay giờ dương lịch? Có tính ngày lễ không? | Trung bình |
| 4 | Khi chuyên viên vắng mặt, hệ thống tự động chuyển phân công hay chờ admin can thiệp? | Trung bình |
| 5 | Một workflow có thể áp dụng cho nhiều loại hồ sơ khác nhau không, hay mỗi loại hồ sơ có workflow riêng? | Trung bình |
