# 05. Quy tắc Notification

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này xác định các quy tắc thông báo (notification rule) trong hệ thống workflow xử lý hồ sơ: khi nào gửi thông báo, gửi cho ai, qua kênh nào, và nội dung thông báo mẫu. Đây là cơ sở để admin cấu hình notification và đội kỹ thuật thiết kế cơ chế gửi thông báo.

---

## Kênh thông báo ưu tiên

| Kênh | Mức độ ưu tiên | Mô tả |
|------|---------------|-------|
| Push notification | **Ưu tiên cao nhất** | Gửi qua app hoặc trình duyệt; kịp thời, không cần email |
| Email | Bổ sung | Dùng cho thông báo quan trọng, có nội dung dài |
| SMS | Bổ sung | Chỉ dùng cho thông báo khẩn, ngắn gọn |
| In-app notification | Luôn có | Hiển thị trong giao diện hệ thống, ngay cả khi không có push |

**Nguyên tắc:**
- Một sự kiện có thể gửi qua nhiều kênh cùng lúc
- Push notification là kênh mặc định; admin có thể cấu hình thêm email/SMS theo từng workflow
- Không spam: tránh gửi quá nhiều thông báo không cần thiết cho cùng một sự kiện

---

## Các thời điểm cần gửi notification

### Nhóm sự kiện hồ sơ

| Sự kiện | Mô tả |
|---------|-------|
| Hồ sơ được nộp thành công | Người dùng xác nhận nộp xong |
| Hồ sơ bị yêu cầu bổ sung | Cán bộ yêu cầu bổ sung tài liệu |
| Hồ sơ được tiếp nhận chính thức | Cán bộ xác nhận tiếp nhận |
| Hồ sơ được phân công | Hệ thống phân công cho chuyên viên |
| Hồ sơ chuyển sang bước mới | Mỗi khi hồ sơ di chuyển giữa các bước |
| Hồ sơ được phê duyệt | Lãnh đạo phê duyệt |
| Hồ sơ bị từ chối | Lãnh đạo từ chối kèm lý do |
| Hồ sơ sẵn sàng trả kết quả | Bộ phận trả kết quả xác nhận |
| Hồ sơ đã hoàn thành / đóng | Hồ sơ được đóng thành công |

### Nhóm sự kiện SLA / Cảnh báo

| Sự kiện | Mô tả |
|---------|-------|
| SLA sắp hết hạn (ngưỡng cảnh báo) | Còn X ngày/giờ trước khi hết SLA tại bước hiện tại |
| SLA đã hết hạn (quá hạn) | Hồ sơ vượt quá thời hạn xử lý tại một bước |
| Leo thang cấp 1 | Quá hạn vượt ngưỡng 1, thông báo trưởng phòng |
| Leo thang cấp 2 | Quá hạn vượt ngưỡng 2, thông báo giám đốc |

### Nhóm sự kiện workflow (admin)

| Sự kiện | Mô tả |
|---------|-------|
| Workflow chuyển sang TESTING | Thông báo nhóm tester |
| Workflow đủ điều kiện publish | Thông báo admin |
| Workflow được ACTIVE | Thông báo tất cả cán bộ liên quan |
| Workflow bị DEACTIVATE | Thông báo admin và cán bộ đang dùng |

---

## Nhóm người nhận notification

### Theo phòng ban

Gửi thông báo cho **tất cả thành viên** của một phòng ban khi có hồ sơ cần xử lý hoặc tham vấn.

**Khi nào dùng:**
- Bước tiếp nhận: thông báo toàn bộ phận một cửa
- Bước lấy ý kiến liên phòng: thông báo toàn phòng được tham vấn
- Khi cấu hình assignment rule kiểu "theo phòng ban"

**Lưu ý:** Tránh dùng tùy tiện vì có thể gây nhiễu. Chỉ nên dùng khi không thể xác định người cụ thể.

---

### Theo vai trò

Gửi thông báo cho **tất cả người có vai trò nhất định** trong hệ thống.

**Khi nào dùng:**
- Thông báo cho tất cả "Lãnh đạo có thẩm quyền phê duyệt" loại hồ sơ X
- Thông báo cho tất cả "Chuyên viên thẩm định" khi có hồ sơ mới vào phòng

---

### Theo người được phân công

Gửi thông báo cho **cá nhân cụ thể** đang được phân công xử lý hồ sơ.

**Khi nào dùng:**
- Nhắc nhở chuyên viên về hồ sơ cần xử lý
- Cảnh báo SLA sắp hết cho người đang xử lý
- Thông báo khi hồ sơ được chuyển về từ bước phê duyệt

**Ưu tiên hơn thông báo theo phòng ban:** Sau khi phân công, nên chuyển sang thông báo cá nhân thay vì toàn phòng.

---

### Theo người dùng cuối / Công dân

Gửi thông báo cho **người nộp hồ sơ** để cập nhật tình trạng hồ sơ của họ.

**Khi nào gửi:**
- Xác nhận nộp hồ sơ thành công
- Hồ sơ bị yêu cầu bổ sung (kèm nội dung cụ thể)
- Hồ sơ được tiếp nhận chính thức (kèm ngày hẹn trả)
- Hồ sơ được phê duyệt
- Hồ sơ bị từ chối (kèm lý do)
- Kết quả sẵn sàng trả
- (Tùy chọn) Thông báo khi hồ sơ bị chậm hơn dự kiến

**Nguyên tắc với người dùng cuối:**
- Ngôn ngữ thân thiện, dễ hiểu, không dùng thuật ngữ kỹ thuật nội bộ
- Luôn có mã hồ sơ để người dùng tra cứu dễ dàng
- Không tiết lộ thông tin nội bộ (ai đang xử lý, lý do nội bộ...)

---

## Notification theo từng tình huống nghiệp vụ

| Tình huống | Gửi cho | Nội dung tóm tắt |
|-----------|---------|-----------------|
| Nộp hồ sơ mới | Người dùng cuối, bộ phận tiếp nhận | "Hồ sơ [mã] đã được gửi thành công. Chúng tôi sẽ xử lý và thông báo kết quả sớm nhất." |
| Yêu cầu bổ sung | Người dùng cuối | "Hồ sơ [mã] cần bổ sung: [danh sách tài liệu]. Vui lòng bổ sung trước [ngày]." |
| Tiếp nhận chính thức | Người dùng cuối, chuyên viên được phân công | "Hồ sơ [mã] đã được tiếp nhận. Ngày hẹn trả kết quả: [ngày]." |
| Phân công chuyên viên | Chuyên viên | "Bạn được phân công xử lý hồ sơ [mã]. Hạn xử lý: [ngày]." |
| Chuyển liên phòng | Phòng ban được tham vấn | "Hồ sơ [mã] cần ý kiến của phòng [tên]. Hạn phản hồi: [ngày]." |
| Trình phê duyệt | Lãnh đạo phê duyệt | "Hồ sơ [mã] đang chờ phê duyệt của bạn. Hạn: [ngày]." |
| Phê duyệt | Người dùng cuối, bộ phận trả kết quả | "Hồ sơ [mã] đã được phê duyệt. Kết quả sẽ được thông báo sớm." |
| Từ chối | Người dùng cuối | "Hồ sơ [mã] không được chấp thuận. Lý do: [lý do]. Bạn có thể khiếu nại hoặc nộp lại." |
| Sắp quá hạn | Người đang xử lý | "Hồ sơ [mã] còn [X] ngày trước khi quá hạn SLA tại bước hiện tại." |
| Quá hạn | Người xử lý, trưởng phòng | "Hồ sơ [mã] đã quá hạn xử lý tại bước [tên bước]. Cần xử lý ngay." |
| Leo thang | Lãnh đạo cấp cao | "Hồ sơ [mã] đã quá hạn [X] ngày, chưa được xử lý. Đề nghị kiểm tra." |
| Trả kết quả | Người dùng cuối | "Kết quả hồ sơ [mã] đã sẵn sàng. [Mô tả hình thức nhận kết quả]." |

---

## Nội dung thông báo mẫu

### Mẫu cho người dùng cuối – Xác nhận nộp hồ sơ

```
Tiêu đề: Xác nhận nộp hồ sơ thành công

Kính gửi [Tên người dùng],

Hồ sơ của bạn đã được ghi nhận thành công.
- Mã hồ sơ: [Mã hồ sơ]
- Loại hồ sơ: [Tên loại hồ sơ]
- Ngày nộp: [Ngày giờ]
- Thời gian xử lý dự kiến: [X ngày làm việc]
- Ngày hẹn trả kết quả (dự kiến): [Ngày]

Bạn có thể theo dõi tình trạng hồ sơ tại: [Link]

Trân trọng,
[Tên đơn vị]
```

---

### Mẫu cho người dùng cuối – Yêu cầu bổ sung

```
Tiêu đề: Hồ sơ [Mã] cần bổ sung tài liệu

Kính gửi [Tên người dùng],

Hồ sơ của bạn (Mã: [Mã hồ sơ]) cần được bổ sung tài liệu sau:

[Danh sách tài liệu cần bổ sung]

Lý do: [Lý do cụ thể từ cán bộ tiếp nhận]
Hạn bổ sung: [Ngày]

Vui lòng đăng nhập và bổ sung tài liệu tại: [Link]
Nếu bạn không bổ sung trước ngày trên, hồ sơ có thể bị trả lại.

Trân trọng,
[Tên đơn vị]
```

---

### Mẫu cho cán bộ – Thông báo có hồ sơ mới

```
Tiêu đề: Hồ sơ mới cần xử lý

Bạn có hồ sơ mới cần xử lý:
- Mã hồ sơ: [Mã]
- Loại: [Loại hồ sơ]
- Bước hiện tại: [Tên bước]
- Hạn xử lý: [Ngày] ([X] ngày làm việc còn lại)

Xem chi tiết: [Link]
```

---

### Mẫu cảnh báo SLA – Gửi cho người xử lý

```
Tiêu đề: ⚠️ Hồ sơ [Mã] sắp quá hạn

Hồ sơ [Mã] đang chờ xử lý tại bước "[Tên bước]".
Hạn còn lại: [X] ngày ([Ngày hết hạn]).

Vui lòng xử lý hoặc liên hệ cấp trên nếu cần hỗ trợ.

Xem hồ sơ: [Link]
```

---

## Quy tắc tối thiểu cho MVP

Trong phiên bản MVP, cần đảm bảo tối thiểu các notification sau:

| # | Sự kiện | Người nhận | Kênh | Bắt buộc? |
|---|---------|-----------|------|-----------|
| 1 | Nộp hồ sơ thành công | Người dùng cuối | Push + In-app | ✓ Bắt buộc |
| 2 | Yêu cầu bổ sung | Người dùng cuối | Push + In-app | ✓ Bắt buộc |
| 3 | Tiếp nhận chính thức | Người dùng cuối | Push + In-app | ✓ Bắt buộc |
| 4 | Có hồ sơ mới cần xử lý | Cán bộ được phân công | Push + In-app | ✓ Bắt buộc |
| 5 | Phê duyệt | Người dùng cuối | Push + In-app | ✓ Bắt buộc |
| 6 | Từ chối | Người dùng cuối | Push + In-app | ✓ Bắt buộc |
| 7 | Kết quả sẵn sàng trả | Người dùng cuối | Push + In-app | ✓ Bắt buộc |
| 8 | Cảnh báo SLA | Người đang xử lý | Push + In-app | ✓ Bắt buộc |
| 9 | Leo thang quá hạn | Trưởng phòng, Giám đốc | Push | ✓ Bắt buộc |
| 10 | Workflow ACTIVE | Cán bộ liên quan | In-app | Có thể thêm sau |

---

## Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Có cho phép người dùng cuối tắt/bật từng loại notification không? | Trung bình |
| 2 | Thông báo quá hạn gửi vào giờ hành chính hay bất kỳ lúc nào? | Trung bình |
| 3 | Có cần lưu lịch sử notification (ai nhận, khi nào, đã đọc chưa) không? | Trung bình |
| 4 | Admin có thể tùy chỉnh nội dung thông báo theo từng workflow không? | Thấp |
| 5 | Khi người nhận offline, notification có được retry không? Retry bao nhiêu lần? | Trung bình |
