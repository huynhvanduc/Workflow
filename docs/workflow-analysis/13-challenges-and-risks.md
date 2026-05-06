# 13. Dự đoán Thách thức và Issue có thể xảy ra

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này dự đoán và phân loại các thách thức kỹ thuật, nghiệp vụ, vận hành và bảo mật có thể xảy ra trong quá trình thiết kế, phát triển và vận hành hệ thống workflow xử lý hồ sơ. Mỗi thách thức đều có đề xuất biện pháp phòng ngừa và giảm thiểu rủi ro.

---

## Phân loại thách thức

| Nhóm | Mô tả |
|---|---|
| **T** | Thách thức kỹ thuật (Technical) |
| **N** | Thách thức nghiệp vụ (Business/Domain) |
| **V** | Thách thức vận hành (Operational) |
| **B** | Thách thức bảo mật (Security) |

---

## 1. Thách thức kỹ thuật (Technical)

### T-01: Độ phức tạp của Workflow Runtime Engine

**Mô tả:** Hệ thống phải xây dựng một engine có khả năng tải định nghĩa workflow động từ database và thực thi đúng logic luồng, điều kiện chuyển bước, phân công và thông báo tại runtime – không phải compile-time.

**Rủi ro:**
- Engine xử lý sai thứ tự bước hoặc bỏ sót transition condition
- Khó debug khi hồ sơ bị kẹt ở trạng thái không mong muốn
- Khó kiểm thử đầy đủ với số lượng lớn tổ hợp workflow

**Biện pháp phòng ngừa:**
- Áp dụng pattern State Machine rõ ràng (finite state machine) cho CaseInstance
- Xây dựng bộ unit test bao phủ từng transition condition
- Cung cấp công cụ "simulation mode" cho admin để chạy thử workflow trước khi thực chiến
- Log đầy đủ mọi quyết định của engine (transition evaluation, assignment resolution)

---

### T-02: Quản lý phiên bản Workflow khi có hồ sơ đang xử lý dở

**Mô tả:** Khi phiên bản workflow mới được kích hoạt, các hồ sơ cũ vẫn tiếp tục theo phiên bản cũ. Hệ thống phải duy trì đồng thời nhiều phiên bản workflow ở runtime.

**Rủi ro:**
- Hồ sơ cũ bị "mắc kẹt" vì phiên bản workflow đã bị INACTIVE nhưng chưa hoàn thành
- Query báo cáo phức tạp khi phải join qua nhiều phiên bản
- Nhầm lẫn giữa dữ liệu của các phiên bản khi xem audit log

**Biện pháp phòng ngừa:**
- Mỗi CaseInstance phải lưu `WorkflowDefinitionId` và `WorkflowVersion` tại thời điểm tạo
- Workflow INACTIVE vẫn phải được cache/load để xử lý hồ sơ dở dang
- Báo cáo phân tách rõ ràng theo từng phiên bản workflow
- Đặt cơ chế cảnh báo khi số hồ sơ đang dùng phiên bản cũ còn nhiều

---

### T-03: Hiệu năng khi tải và thực thi Workflow Definition

**Mô tả:** Mỗi hành động trên hồ sơ đều yêu cầu load workflow definition từ database để evaluate transition. Với số lượng hồ sơ và người dùng đồng thời lớn, đây là bottleneck tiềm ẩn.

**Rủi ro:**
- Database bị quá tải bởi read query cho workflow definition
- Latency tăng cao ở giờ cao điểm (sáng đầu giờ làm việc, cuối tháng)

**Biện pháp phòng ngừa:**
- Cache toàn bộ workflow definition ACTIVE vào Redis; chỉ invalidate khi publish phiên bản mới
- Dùng read replica cho tất cả query không yêu cầu consistency tuyệt đối
- Load test và capacity planning trước khi go-live

---

### T-04: Tính toán SLA và Leo thang (Escalation) chính xác

**Mô tả:** SLA phải tính theo giờ làm việc (working hours), không phải tổng thời gian tuyệt đối. Cần loại trừ ngày nghỉ lễ, cuối tuần và giờ ngoài làm việc.

**Rủi ro:**
- SLA tính sai dẫn đến cảnh báo nhầm (false alarm) hoặc bỏ sót hồ sơ thực sự quá hạn
- Khó cấu hình lịch làm việc cho nhiều đơn vị hành chính khác nhau
- Múi giờ nếu hệ thống phục vụ đa tỉnh

**Biện pháp phòng ngừa:**
- Xây dựng module `WorkingCalendar` cấu hình được theo từng đơn vị
- Lưu `SlaStartedAt` (UTC) và tính lại remaining time mỗi khi check, thay vì lưu `SlaDeadlineAt` cố định
- Unit test bao phủ các ca ngày nghỉ lễ, cuối tuần, hồ sơ tạo cuối giờ chiều thứ Sáu
- Hiển thị SLA dưới dạng "số giờ làm việc còn lại" thay vì chỉ hiện timestamp

---

### T-05: Thiết kế Audit Log bất biến và có hiệu năng cao

**Mô tả:** Audit log phải vừa bất biến (immutable), vừa đủ hiệu năng ghi trong cùng transaction nghiệp vụ, vừa có thể query nhanh qua hàng triệu bản ghi.

**Rủi ro:**
- Ghi log chậm làm tăng thời gian response của API nghiệp vụ
- Bảng audit log tăng trưởng nhanh gây chậm query sau vài tháng vận hành
- Dữ liệu log bị tamper nếu phân quyền database không chặt chẽ

**Biện pháp phòng ngừa:**
- Ghi log trong cùng transaction nhưng dùng fire-and-forget với outbox pattern nếu cần tách biệt
- Partition bảng `CaseAuditLogs` theo tháng (range partitioning trên `PerformedAt`)
- Index hợp lý: composite index (`CaseId`, `PerformedAt`), (`WorkflowId`, `ActionCode`)
- Database-level: REVOKE DELETE, UPDATE từ application user; chỉ INSERT được phép

---

### T-06: Đồng bộ hóa Notification và xử lý thất bại

**Mô tả:** Thông báo gửi bất đồng bộ qua message queue có thể thất bại (FCM lỗi, email bounce, SMS không giao được) và cần cơ chế retry, dead-letter, và báo cáo.

**Rủi ro:**
- Cán bộ không nhận được thông báo hồ sơ mới → hồ sơ bị bỏ qua
- Gửi trùng thông báo do retry không idempotent
- Không biết thông báo thực sự đã đến người dùng hay chưa

**Biện pháp phòng ngừa:**
- Lưu trạng thái gửi thông báo vào bảng `NotificationLogs` (PENDING, SENT, FAILED)
- Đảm bảo idempotency: mỗi notification event có `IdempotencyKey` duy nhất
- Hiển thị cảnh báo trong hệ thống khi có thông báo failed sau số lần retry tối đa
- Fallback: nếu push notification thất bại, thử gửi email

---

### T-07: Xây dựng Workflow Designer (giao diện kéo thả)

**Mô tả:** Giao diện thiết kế workflow dạng node-graph (drag-and-drop các bước, vẽ transition) đòi hỏi frontend phức tạp và phải validate logic trước khi lưu.

**Rủi ro:**
- Admin cấu hình workflow có dead-end (bước không có transition ra)
- Admin vẽ vòng lặp vô hạn (loop) mà không có điều kiện thoát
- Giao diện khó dùng với cán bộ admin ít kỹ thuật

**Biện pháp phòng ngừa:**
- Validate phía client và server: kiểm tra graph có end state, không có node cô lập
- Cảnh báo khi admin cố lưu cấu hình có lỗi logic (warning + blocking)
- UX: hướng dẫn trực quan, tooltip, highlight lỗi ngay trên canvas
- Cung cấp workflow template mẫu để admin bắt đầu từ đó thay vì từ đầu

---

## 2. Thách thức nghiệp vụ (Business)

### N-01: Quy trình nghiệp vụ thay đổi thường xuyên

**Mô tả:** Quy trình hành chính công thay đổi theo văn bản pháp luật, quyết định hành chính – thường không báo trước nhiều. Admin phải cập nhật workflow kịp thời.

**Rủi ro:**
- Workflow cũ vẫn đang chạy khi quy trình mới có hiệu lực
- Phiên bản mới chưa test xong khi deadline yêu cầu áp dụng đến
- Hồ sơ đang xử lý theo quy trình cũ không được tự động chuyển sang quy trình mới

**Biện pháp phòng ngừa:**
- Cho phép admin rút ngắn thời gian test trong trường hợp khẩn cấp (với xác nhận cấp trên)
- Xây dựng cơ chế "Emergency Activation" với audit trail đặc biệt
- Quy trình rõ ràng về xử lý hồ sơ đang dở khi cần chuyển phiên bản gấp

---

### N-02: Phân công tự động không phù hợp thực tế

**Mô tả:** Assignment rule được admin cấu hình dựa trên role/department, nhưng thực tế cán bộ có thể nghỉ phép, thuyên chuyển, hoặc phòng ban thay đổi cơ cấu.

**Rủi ro:**
- Hồ sơ bị phân công cho cán bộ đang nghỉ, không ai xử lý → vi phạm SLA
- Thay đổi cơ cấu phòng ban làm hỏng assignment rule đã cấu hình
- Cán bộ bị quá tải vì không có cơ chế cân bằng tải (load balancing)

**Biện pháp phòng ngừa:**
- Cơ chế `Fallback Assignment`: khi không tìm được người phân công, gửi về trưởng phòng hoặc admin
- Cảnh báo admin khi assignment rule không resolve được (zero assignees)
- Hỗ trợ "out of office" cho cán bộ để tự động redirect khi họ vắng mặt
- Dashboard: theo dõi workload phân phối theo cán bộ trong phòng ban

---

### N-03: Điều kiện chuyển bước (Transition Condition) quá phức tạp

**Mô tả:** Một số loại hồ sơ có điều kiện chuyển bước dựa trên dữ liệu nội dung hồ sơ (ví dụ: trị giá > 500 triệu thì phải qua thêm bước phê duyệt cấp cao). Dữ liệu này nằm ngoài workflow engine.

**Rủi ro:**
- Không thể cấu hình điều kiện phức tạp qua giao diện đơn giản
- Admin phải nhờ kỹ thuật để implement condition mới
- Condition sai dẫn đến hồ sơ đi sai luồng

**Biện pháp phòng ngừa:**
- Xây dựng Expression Engine đơn giản cho phép admin viết điều kiện dạng rule (ví dụ: `case.amount > 500000000`)
- Validate và test-run condition trước khi lưu
- Kiểm soát scope: chỉ cho phép access các trường dữ liệu được whitelist
- Fallback: các condition phức tạp hơn vẫn cần kỹ thuật implement dưới dạng plugin/handler

---

### N-04: Đa phòng ban và phối hợp song song

**Mô tả:** Một hồ sơ có thể cần phối hợp đồng thời với nhiều phòng ban trước khi tiếp tục (parallel collaboration steps). Cần xác định khi nào hồ sơ được phép tiếp tục.

**Rủi ro:**
- Không biết phải đợi tất cả phòng ban phản hồi hay chỉ cần một phòng là đủ
- Nếu một phòng không phản hồi đúng hạn, toàn bộ luồng bị kẹt
- Khó hiển thị trạng thái tổng hợp cho cán bộ chuyên môn và lãnh đạo

**Biện pháp phòng ngừa:**
- Workflow step có thể có loại `PARALLEL_COLLABORATION` với rule: ALL/ANY
- SLA riêng cho từng sub-request phối hợp; cảnh báo nếu một bên chưa phản hồi
- Dashboard: hiển thị rõ trạng thái từng yêu cầu phối hợp trong hồ sơ

---

### N-05: Đảm bảo chất lượng giai đoạn UAT trước khi ACTIVE

**Mô tả:** Tester được chỉ định cần thực hiện test đủ các tình huống nhưng không có checklist bắt buộc, dẫn đến test không đầy đủ.

**Rủi ro:**
- Workflow được publish sau khi đóng hết issue nhưng chưa cover hết edge case
- Lỗi mới phát hiện sau khi ACTIVE không có quy trình xử lý nhanh
- Tester xác nhận hoàn thành nhưng thực tế chưa test đủ trường hợp

**Biện pháp phòng ngừa:**
- Yêu cầu tester điền checklist tối thiểu trước khi được phép "xác nhận hoàn thành test"
- Lưu evidence (screenshot, timestamp) cho từng test case đã chạy
- Sau khi ACTIVE, cung cấp nút "Quick Deactivate" cho admin với quy trình cụ thể
- Quy trình hotfix: cho phép tạo phiên bản vá nhanh với thời gian test rút ngắn có approval

---

## 3. Thách thức vận hành (Operational)

### V-01: Admin cấu hình sai gây ảnh hưởng hồ sơ thật

**Mô tả:** Dù đã có giai đoạn TESTING, vẫn có khả năng lỗi thoát qua UAT và chỉ phát hiện sau khi ACTIVE khi hồ sơ thật vào hệ thống.

**Rủi ro:**
- Hồ sơ bị chuyển sai bước hoặc phân công sai người
- SLA cấu hình quá ngắn gây vi phạm hàng loạt
- Notification gửi sai đối tượng

**Biện pháp phòng ngừa:**
- Emergency deactivate: Admin có thể tắt workflow đang ACTIVE trong vài giây
- Rollback plan: phiên bản cũ (INACTIVE) có thể được re-activate sau khi review
- Alerting: cảnh báo ngay khi phát hiện pattern bất thường (đột ngột tăng SLA breach, notification failure)
- Runbook: tài liệu hướng dẫn xử lý từng kịch bản sự cố phổ biến

---

### V-02: Dữ liệu hồ sơ dở dang khi Deactivate khẩn cấp

**Mô tả:** Khi phải DEACTIVATE workflow khẩn cấp, các hồ sơ đang xử lý dở sẽ ở trạng thái không xác định.

**Rủi ro:**
- Cán bộ không thể tiếp tục xử lý hồ sơ, không biết làm gì tiếp
- Hệ thống không giao được kết quả cho người dân đúng hạn
- Dữ liệu không nhất quán nếu một số action đã được thực hiện một nửa

**Biện pháp phòng ngừa:**
- Khi DEACTIVATE, hồ sơ đang dở được chuyển vào **lưu trữ tồn đọng** (không hủy)
- Admin có thể xem danh sách hồ sơ tồn đọng và xử lý theo từng trường hợp
- Thông báo tự động đến người dân khi hồ sơ bị chuyển tồn đọng
- Không bao giờ tự động xóa hoặc hủy hồ sơ khi deactivate

---

### V-03: Quản lý tài khoản và phân quyền khi tổ chức thay đổi

**Mô tả:** Cán bộ thuyên chuyển, phòng ban sáp nhập/chia tách, vai trò thay đổi là điều thường xảy ra trong cơ quan hành chính.

**Rủi ro:**
- Cán bộ cũ vẫn có quyền truy cập sau khi thuyên chuyển
- Phân công tự động dựa trên role cũ không còn chính xác
- Audit log ghi tên/phòng ban tại thời điểm hành động nhưng giao diện hiển thị tên/phòng hiện tại

**Biện pháp phòng ngừa:**
- Audit log dùng snapshot: lưu tên, phòng ban, vai trò **tại thời điểm hành động** – không foreign key về bảng master (đã được thiết kế trong `CaseAuditLog`)
- Quy trình offboarding: checklist thu hồi quyền truy cập khi cán bộ thuyên chuyển
- Review định kỳ danh sách phân quyền (quarterly access review)

---

### V-04: Tăng trưởng dữ liệu Audit Log

**Mô tả:** Audit log được lưu trong hot storage 1 tuần rồi chuyển vào lưu trữ tồn đọng. Khối lượng dữ liệu tích lũy trong tồn đọng sẽ rất lớn theo thời gian.

**Rủi ro:**
- Query audit log chậm sau nhiều tháng vận hành
- Chi phí lưu trữ tăng cao
- Backup và restore chậm do kích thước database lớn

**Biện pháp phòng ngừa:**
- Partitioning theo tháng từ đầu (không refactor sau)
- Archive dữ liệu cũ hơn 2 năm sang cold storage (object storage: S3, MinIO) nhưng vẫn có thể query
- Compression: PostgreSQL TOAST, bảng audit log dùng columnar storage nếu cần
- Capacity planning: ước tính tăng trưởng dữ liệu hàng năm và lên kế hoạch hardware/storage

---

### V-05: Vận hành đa địa phương (multi-tenant)

**Mô tả:** Nếu hệ thống được triển khai cho nhiều tỉnh/huyện/xã, cần kiểm soát phân tách dữ liệu và cấu hình giữa các đơn vị.

**Rủi ro:**
- Dữ liệu hồ sơ của đơn vị này lọt sang đơn vị khác
- Cấu hình workflow của một đơn vị ảnh hưởng đến đơn vị khác
- Hiệu năng database giảm khi tất cả đơn vị dùng chung instance

**Biện pháp phòng ngừa:**
- Thiết kế multi-tenant từ đầu: mỗi bảng có `TenantId`; tất cả query đều filter theo tenant
- Row-Level Security (RLS) tại tầng database như một lớp bảo vệ bổ sung
- Phân tách schema theo tenant (schema-per-tenant) nếu yêu cầu isolation cao hơn
- Xem xét deployment riêng biệt cho các tỉnh lớn (database riêng)

---

## 4. Thách thức bảo mật (Security)

### B-01: Leo thang đặc quyền của Admin

**Mô tả:** Admin có quyền rất rộng: cấu hình workflow, xem tất cả hồ sơ, deactivate hệ thống. Nếu tài khoản admin bị chiếm, thiệt hại rất lớn.

**Rủi ro:**
- Admin giả mạo publish workflow có lỗi cố ý để chuyển hướng hồ sơ
- Admin xem dữ liệu hồ sơ nhạy cảm của người dân ngoài phạm vi cần thiết
- Tài khoản admin bị brute-force hoặc phishing

**Biện pháp phòng ngừa:**
- Bắt buộc MFA (Multi-Factor Authentication) cho tài khoản Admin
- Nguyên tắc least privilege: tách `SYSTEM_ADMIN` (cấu hình) và `MONITOR` (chỉ xem)
- Ghi lại tất cả thao tác admin vào Config Audit Log không thể sửa xóa
- Alert khi admin thực hiện thao tác nhạy cảm (deactivate, publish) ngoài giờ làm việc
- Periodic review: lãnh đạo nhận báo cáo định kỳ về các thao tác admin

---

### B-02: Lộ thông tin hồ sơ nhạy cảm qua Audit Log

**Mô tả:** Audit log chứa nhiều thông tin nhạy cảm (lý do từ chối, ghi chú nội bộ, thông tin cá nhân người dân). Quyền truy cập log cần được kiểm soát chặt.

**Rủi ro:**
- Cán bộ xem log của hồ sơ không thuộc phạm vi mình xử lý
- Người dân không được xem thông tin nội bộ nhưng API có thể bị khai thác
- Export log chứa PII bị rò rỉ ra ngoài

**Biện pháp phòng ngừa:**
- API endpoint xem log phải enforce authorization: chỉ trả về dữ liệu trong phạm vi được phép
- Citizen view là endpoint riêng biệt, chỉ trả về thông tin đã được sanitize
- Mã hóa dữ liệu nhạy cảm trong log hoặc trong file export
- Ghi lại mọi hành động export log (meta-audit)

---

### B-03: Tấn công vào Workflow Transition Logic

**Mô tả:** Nếu transition condition hoặc action handler bị khai thác, kẻ tấn công có thể ép hồ sơ chuyển sang bước sai (ví dụ: bỏ qua bước phê duyệt).

**Rủi ro:**
- Hồ sơ được approve mà không qua bước phê duyệt bắt buộc
- Cán bộ thực hiện action không được phép tại bước hiện tại

**Biện pháp phòng ngừa:**
- Server-side validation: mọi action request phải được validate lại tại server (không tin client)
- Engine luôn kiểm tra: người thực hiện có đúng role không, bước có cho phép action này không
- Transition chỉ được thực hiện nếu engine evaluate và approve – không có bypass
- Penetration testing tập trung vào workflow transition logic

---

### B-04: Injection vào Expression Engine (Transition Condition)

**Mô tả:** Nếu admin được phép viết expression tùy ý để làm transition condition, có nguy cơ code injection nếu expression được eval trực tiếp.

**Rủi ro:**
- Admin nhập biểu thức độc hại truy cập database hoặc hệ thống file
- Kẻ tấn công chiếm tài khoản admin và inject mã nguy hiểm vào condition

**Biện pháp phòng ngừa:**
- Không dùng `eval()` hay dynamic code execution cho expression
- Xây dựng safe expression parser giới hạn trong whitelist operator và field
- Sandbox: expression chỉ được access dữ liệu được khai báo, không có I/O
- Review expression do admin nhập trước khi lưu (validation + static analysis)

---

### B-05: Giả mạo thông báo (Notification Spoofing)

**Mô tả:** Nếu kênh thông báo không được bảo vệ, kẻ tấn công có thể giả mạo thông báo từ hệ thống để lừa cán bộ hoặc người dân.

**Rủi ro:**
- Người dân nhận thông báo giả "hồ sơ đã duyệt" nhưng thực tế chưa
- Cán bộ nhận link giả trong email dẫn đến trang phishing

**Biện pháp phòng ngừa:**
- Email: DKIM, SPF, DMARC configuration
- Push notification: chỉ từ server FCM chính thức, không mở endpoint public để gửi notification
- Thông báo không chứa link cần click; mọi action đều thực hiện trực tiếp trong app
- Người dùng luôn có thể verify trạng thái thật của hồ sơ trong app, không chỉ dựa vào notification

---

## 5. Tổng hợp ma trận rủi ro

| Mã | Thách thức | Mức độ ảnh hưởng | Khả năng xảy ra | Độ ưu tiên xử lý |
|---|---|---|---|---|
| T-01 | Workflow Engine phức tạp | Cao | Cao | 🔴 Cao |
| T-02 | Quản lý multi-version workflow | Cao | Cao | 🔴 Cao |
| T-03 | Hiệu năng load workflow definition | Trung bình | Trung bình | 🟡 Trung bình |
| T-04 | SLA tính theo giờ làm việc | Cao | Cao | 🔴 Cao |
| T-05 | Audit log bất biến + hiệu năng | Cao | Trung bình | 🔴 Cao |
| T-06 | Notification thất bại | Trung bình | Cao | 🟡 Trung bình |
| T-07 | Workflow Designer UX | Trung bình | Trung bình | 🟡 Trung bình |
| N-01 | Quy trình nghiệp vụ thay đổi gấp | Cao | Cao | 🔴 Cao |
| N-02 | Assignment rule sai thực tế | Trung bình | Cao | 🟡 Trung bình |
| N-03 | Transition condition phức tạp | Trung bình | Trung bình | 🟡 Trung bình |
| N-04 | Phối hợp song song đa phòng ban | Trung bình | Trung bình | 🟡 Trung bình |
| N-05 | UAT không đầy đủ trước ACTIVE | Cao | Cao | 🔴 Cao |
| V-01 | Admin cấu hình sai ảnh hưởng hồ sơ thật | Cao | Trung bình | 🔴 Cao |
| V-02 | Hồ sơ dở khi deactivate khẩn cấp | Cao | Thấp | 🟡 Trung bình |
| V-03 | Tổ chức thay đổi phá vỡ phân quyền | Trung bình | Cao | 🟡 Trung bình |
| V-04 | Tăng trưởng dữ liệu audit log | Thấp | Cao | 🟢 Thấp |
| V-05 | Multi-tenant isolation | Cao | Trung bình | 🔴 Cao |
| B-01 | Leo thang đặc quyền admin | Cao | Thấp | 🟡 Trung bình |
| B-02 | Lộ thông tin nhạy cảm qua log | Cao | Trung bình | 🔴 Cao |
| B-03 | Tấn công transition logic | Cao | Thấp | 🟡 Trung bình |
| B-04 | Injection vào expression engine | Cao | Thấp | 🟡 Trung bình |
| B-05 | Giả mạo thông báo | Trung bình | Thấp | 🟢 Thấp |

---

## 6. Các issue ưu tiên cần giải quyết sớm (trước khi bắt đầu thiết kế kỹ thuật)

1. **[T-01]** Thiết kế kiến trúc Workflow Engine trước khi bắt đầu code – đây là lõi của hệ thống
2. **[T-02]** Quyết định chiến lược versioning và cách xử lý hồ sơ dở dang khi chuyển phiên bản
3. **[T-04]** Chốt yêu cầu về lịch làm việc và cách tính SLA theo giờ hành chính
4. **[N-01]** Xây dựng quy trình Emergency Activation để chuẩn bị cho tình huống thay đổi quy trình gấp
5. **[N-05]** Định nghĩa checklist UAT tối thiểu và quy trình xác nhận test trước khi hệ thống được phép sử dụng
6. **[B-02]** Thiết kế phân quyền xem audit log chặt chẽ từ đầu – khó sửa sau khi triển khai
