# 07 · Checklist Người dùng cuối, Tester & Vai trò nội bộ

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## A. Checklist – Admin (Quản trị hệ thống)

- [ ] Đã tạo đầy đủ **phòng ban** và **vai trò** trước khi thiết kế workflow.
- [ ] Workflow được thiết kế với **ít nhất 1 bước Start và 1 bước End**.
- [ ] Mỗi bước có **AssignmentRule** hợp lệ (không để trống).
- [ ] Mỗi bước có **ít nhất 1 Transition** đi tiếp hoặc kết thúc.
- [ ] Đã cấu hình **NotificationRule** cho các sự kiện quan trọng (`ON_ENTER`, `ON_ACTION`, `ON_OVERDUE`).
- [ ] Đã cấu hình **SLA** cho các bước có thời hạn xử lý.
- [ ] Đã đặt **thời gian thử nghiệm** (`StartDate` – `EndDate`) trước khi chuyển sang `TESTING`.
- [ ] Không publish workflow khi còn **issue OPEN / IN_PROGRESS**.
- [ ] Khi cần thay đổi workflow đang `ACTIVE`, **clone thành phiên bản mới** thay vì sửa trực tiếp.
- [ ] Mọi thay đổi trạng thái workflow đều có **lý do ghi kèm** trong audit log.

---

## B. Checklist – Tester (Kiểm thử)

- [ ] Đã được cấp quyền tester trên workflow đang ở trạng thái `TESTING`.
- [ ] Thực hiện đầy đủ **các tình huống nghiệp vụ đặc trưng** (xem [TH-01 đến TH-10](./04-tinh-huong-nghiep-vu.md)).
- [ ] Ghi nhận **mọi bất thường** vào Issue Log, kể cả lỗi nhỏ.
- [ ] Điền đủ thông tin: `Title`, `Description`, `Severity`, tình huống tái hiện.
- [ ] Khi admin/developer fix xong, **xác nhận lại** bằng cách kiểm thử lại tình huống tương tự.
- [ ] Chuyển issue sang `CLOSED` sau khi **xác nhận fix đúng**.
- [ ] Nếu fix chưa đúng, **mở lại issue** kèm ghi chú rõ lý do.
- [ ] Không đóng issue chỉ vì hết thời gian – phải xác nhận thực sự.

---

## C. Checklist – Cán bộ tiếp nhận

- [ ] Chỉ xử lý hồ sơ trên workflow đang ở trạng thái `ACTIVE`.
- [ ] Kiểm tra đầy đủ thông tin hồ sơ trước khi tiếp nhận chính thức.
- [ ] Sử dụng action `REQUEST_SUPPLEMENT` (không từ chối ngay) khi hồ sơ thiếu tài liệu.
- [ ] Ghi rõ **danh sách tài liệu còn thiếu** khi yêu cầu bổ sung.
- [ ] Chuyển hồ sơ đúng bộ phận theo quy trình workflow đã cấu hình.

---

## D. Checklist – Chuyên viên xử lý

- [ ] Xác nhận đã tiếp nhận hồ sơ được phân công (không bỏ sót trong queue).
- [ ] Hoàn thành xử lý **trước thời hạn SLA** của bước.
- [ ] Khi cần phối hợp, sử dụng action `REQUEST_COLLABORATION` đúng phòng ban.
- [ ] Trình duyệt đúng cấp thẩm quyền theo cấu hình workflow.
- [ ] Khi bị trả lại, đọc kỹ **lý do từ chối** trước khi chỉnh sửa hồ sơ.

---

## E. Checklist – Người phê duyệt (Lãnh đạo)

- [ ] Phê duyệt đúng thời hạn, không để hồ sơ tồn đọng quá SLA.
- [ ] Khi từ chối, ghi rõ **lý do cụ thể** để chuyên viên có thể điều chỉnh.
- [ ] Không phê duyệt hồ sơ khi còn thiếu thông tin cần thiết.

---

## F. Checklist – Người dùng cuối / Công dân

- [ ] Chỉ nộp và theo dõi hồ sơ trên workflow đã được **kích hoạt chính thức** (`ACTIVE`).
- [ ] Nhận và đọc thông báo đúng hạn, đặc biệt khi có yêu cầu bổ sung.
- [ ] Bổ sung tài liệu **đúng và đủ** trong thời hạn được thông báo.
- [ ] Có thể xem **hồ sơ đang ở bước nào** trong hành trình xử lý.
- [ ] Nhận thông báo khi hồ sơ **đổi trạng thái** quan trọng (tiếp nhận, phê duyệt, có kết quả).
- [ ] Không truy cập vào workflow đang trong giai đoạn thử nghiệm.

---

## G. Tiêu chí nghiệm thu từ góc nhìn người dùng cuối

| # | Tiêu chí | Trạng thái |
|---|---|---|
| 1 | Người dùng chỉ nhìn thấy workflow đã được công bố chính thức | ☐ |
| 2 | Người dùng nhận được thông báo **đúng thời điểm** khi hồ sơ đổi trạng thái | ☐ |
| 3 | Người dùng **không** truy cập nhầm vào workflow đang thử nghiệm | ☐ |
| 4 | Người dùng có thể biết hồ sơ đang ở **bước nào** trong quy trình | ☐ |
| 5 | Khi hồ sơ bị yêu cầu bổ sung, người dùng nhận được **lý do rõ ràng** | ☐ |
| 6 | Khi hồ sơ bị từ chối, người dùng nhận được **thông báo và lý do** | ☐ |
| 7 | Nhắc nhở bổ sung hồ sơ được gửi **trước khi hết hạn** | ☐ |

---

*Xem tiếp: [08 · Nguyên tắc quản trị](./08-nguyen-tac-quan-tri.md)*
