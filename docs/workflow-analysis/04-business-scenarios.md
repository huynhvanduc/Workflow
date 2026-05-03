# 04. Tình huống nghiệp vụ đặc trưng

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này liệt kê và mô tả chi tiết 10 tình huống nghiệp vụ đặc trưng nhất trong quy trình xử lý hồ sơ. Mỗi tình huống bao gồm: bối cảnh, actor tham gia, luồng xử lý, điều kiện kích hoạt, và kết quả mong đợi. Đây là cơ sở để xây dựng test case cho giai đoạn UAT và thiết kế kỹ thuật.

---

## Danh sách tình huống

### 1. Người dân nộp hồ sơ mới

**Bối cảnh:** Công dân hoặc tổ chức muốn nộp hồ sơ hành chính trực tuyến.

**Actor chính:** Người dùng cuối (công dân/tổ chức)

**Luồng xử lý:**
1. Người dùng đăng nhập hệ thống và chọn dịch vụ công tương ứng
2. Hệ thống hiển thị biểu mẫu hồ sơ theo workflow ACTIVE của loại hồ sơ đó
3. Người dùng điền thông tin, đính kèm tài liệu và xác nhận nộp
4. Hệ thống tạo workflow instance mới và chuyển hồ sơ sang bước tiếp nhận
5. Hệ thống gửi thông báo xác nhận cho người dùng (mã hồ sơ, thời gian dự kiến)

**Điều kiện tiên quyết:**
- Workflow cho loại hồ sơ này đang ở trạng thái ACTIVE
- Người dùng đã đăng nhập và xác thực danh tính

**Kết quả mong đợi:**
- Hồ sơ được tạo thành công với mã định danh duy nhất
- Hồ sơ chuyển đến bước tiếp nhận, chờ cán bộ xử lý
- Người dùng nhận thông báo xác nhận đã nộp thành công

**Notification liên quan:** Gửi cho người dùng cuối, gửi cho bộ phận tiếp nhận

---

### 2. Hồ sơ bị trả để bổ sung

**Bối cảnh:** Cán bộ tiếp nhận kiểm tra hồ sơ và phát hiện thiếu tài liệu hoặc thông tin không đầy đủ.

**Actor chính:** Cán bộ tiếp nhận, Người dùng cuối

**Luồng xử lý:**
1. Cán bộ tiếp nhận xem xét hồ sơ vừa được nộp
2. Phát hiện hồ sơ thiếu tài liệu hoặc thông tin không hợp lệ
3. Cán bộ chọn action "Yêu cầu bổ sung" và ghi rõ lý do, danh sách tài liệu cần bổ sung
4. Hệ thống chuyển hồ sơ về trạng thái chờ bổ sung
5. Người dùng nhận thông báo với nội dung cụ thể về những gì cần bổ sung
6. Người dùng nộp bổ sung → hồ sơ quay lại bước tiếp nhận
7. Cán bộ kiểm tra lại và tiếp nhận hoặc yêu cầu bổ sung tiếp

**Điều kiện tiên quyết:**
- Hồ sơ đang ở bước tiếp nhận
- Cán bộ tiếp nhận đang xử lý hồ sơ này

**Kết quả mong đợi:**
- Người dùng nhận thông báo rõ ràng: thiếu gì, cần nộp gì, thời hạn bổ sung là bao lâu
- Hồ sơ không bị đóng hoặc hủy, chỉ tạm dừng chờ bổ sung
- Lịch sử yêu cầu bổ sung được ghi lại trong hệ thống

**Notification liên quan:** Gửi cho người dùng cuối với danh sách tài liệu cần bổ sung

---

### 3. Hồ sơ được tiếp nhận chính thức

**Bối cảnh:** Sau khi xác nhận hồ sơ đủ điều kiện, cán bộ tiếp nhận xác nhận tiếp nhận chính thức.

**Actor chính:** Cán bộ tiếp nhận

**Luồng xử lý:**
1. Cán bộ tiếp nhận xem xét toàn bộ hồ sơ và tài liệu đính kèm
2. Xác nhận hồ sơ đầy đủ và hợp lệ
3. Chọn action "Tiếp nhận chính thức"
4. Hệ thống ghi nhận ngày tiếp nhận, bắt đầu tính SLA
5. Hồ sơ được chuyển đến bước xử lý chuyên môn theo assignment rule đã cấu hình
6. Người được phân công nhận thông báo có hồ sơ mới cần xử lý

**Điều kiện tiên quyết:**
- Hồ sơ đã đầy đủ tài liệu theo yêu cầu
- Cán bộ tiếp nhận có thẩm quyền xác nhận tiếp nhận

**Kết quả mong đợi:**
- Hồ sơ chính thức đi vào luồng xử lý chuyên môn
- Ngày tiếp nhận được ghi nhận và SLA bắt đầu chạy
- Người dùng nhận thông báo: hồ sơ đã được tiếp nhận, mã tiếp nhận, ngày hẹn trả kết quả

**Notification liên quan:** Gửi cho người dùng cuối (xác nhận tiếp nhận + ngày hẹn), gửi cho chuyên viên được phân công

---

### 4. Hồ sơ được phân công cho chuyên viên

**Bối cảnh:** Sau khi tiếp nhận, hệ thống hoặc trưởng phòng phân công hồ sơ cho chuyên viên cụ thể để thẩm định.

**Actor chính:** Hệ thống (theo assignment rule) hoặc Trưởng phòng

**Luồng xử lý:**
1. Hồ sơ đến bước thẩm định chuyên môn
2. Hệ thống áp dụng assignment rule đã cấu hình:
   - Nếu rule là "theo phòng ban": thông báo cho toàn phòng, chuyên viên tự nhận việc
   - Nếu rule là "round-robin": tự động giao cho chuyên viên tiếp theo trong danh sách
   - Nếu rule là "thủ công": trưởng phòng chọn người phân công
3. Chuyên viên được phân công nhận thông báo
4. Chuyên viên xác nhận nhận việc và bắt đầu xử lý

**Điều kiện tiên quyết:**
- Hồ sơ đã được tiếp nhận chính thức
- Assignment rule cho bước này đã được cấu hình hợp lệ

**Kết quả mong đợi:**
- Hồ sơ có người xử lý rõ ràng, không bị rơi vào tình trạng không ai nhận
- Chuyên viên biết deadline xử lý (SLA của bước)
- Lịch sử phân công được ghi lại

**Notification liên quan:** Gửi cho chuyên viên được phân công

---

### 5. Hồ sơ được chuyển liên phòng ban

**Bối cảnh:** Trong quá trình thẩm định, phát hiện cần tham khảo ý kiến của phòng ban khác trước khi đưa ra kết luận.

**Actor chính:** Chuyên viên xử lý, Phòng ban được tham vấn

**Luồng xử lý:**
1. Chuyên viên đang xử lý nhận thấy cần ý kiến từ phòng ban khác
2. Chọn action "Chuyển lấy ý kiến" và chọn phòng ban cần tham vấn
3. Hồ sơ chuyển sang bước lấy ý kiến liên phòng, SLA mới bắt đầu tính
4. Phòng ban được tham vấn nhận thông báo và xử lý trong thời hạn
5. Phòng ban tham vấn cập nhật ý kiến và chuyển hồ sơ về phòng chủ trì
6. Chuyên viên chủ trì tiếp tục xử lý với ý kiến đã có

**Điều kiện tiên quyết:**
- Workflow đã được cấu hình hỗ trợ bước chuyển liên phòng
- Phòng ban được tham vấn có trong danh sách cho phép của workflow

**Kết quả mong đợi:**
- Hồ sơ được các phòng liên quan đều xem và có ý kiến
- Toàn bộ ý kiến được lưu lại trong hệ thống
- Không có phòng nào bị quên hoặc bỏ qua

**Notification liên quan:** Gửi cho phòng ban được tham vấn, gửi nhắc nhở trước deadline lấy ý kiến

---

### 6. Chuyên viên trình lãnh đạo phê duyệt

**Bối cảnh:** Sau khi hoàn thành thẩm định, chuyên viên chuẩn bị tờ trình và đưa lên lãnh đạo phê duyệt.

**Actor chính:** Chuyên viên xử lý, Lãnh đạo phê duyệt

**Luồng xử lý:**
1. Chuyên viên hoàn thành soạn thảo kết quả thẩm định và đề xuất
2. Chọn action "Trình phê duyệt" và điền thông tin tờ trình nếu cần
3. Hồ sơ chuyển sang bước chờ phê duyệt của lãnh đạo
4. Lãnh đạo được phân công nhận thông báo có hồ sơ chờ phê duyệt
5. Lãnh đạo xem xét toàn bộ hồ sơ, kết quả thẩm định và đề xuất

**Điều kiện tiên quyết:**
- Chuyên viên đã hoàn thành đầy đủ nội dung thẩm định
- Lãnh đạo có thẩm quyền phê duyệt loại hồ sơ này

**Kết quả mong đợi:**
- Hồ sơ đến đúng lãnh đạo có thẩm quyền (theo assignment rule của bước phê duyệt)
- Lãnh đạo nhận đầy đủ thông tin để ra quyết định
- SLA phê duyệt bắt đầu tính từ khi trình

**Notification liên quan:** Gửi cho lãnh đạo phê duyệt

---

### 7. Lãnh đạo phê duyệt

**Bối cảnh:** Lãnh đạo xem xét và đồng ý với kết quả thẩm định, phê duyệt hồ sơ.

**Actor chính:** Lãnh đạo phê duyệt

**Luồng xử lý:**
1. Lãnh đạo xem xét hồ sơ và kết quả thẩm định của chuyên viên
2. Đồng ý với đề xuất, chọn action "Phê duyệt"
3. Có thể thêm ghi chú hoặc điều kiện kèm theo
4. Hồ sơ chuyển sang bước trả kết quả
5. Bộ phận trả kết quả nhận thông báo có hồ sơ sẵn sàng trả

**Điều kiện tiên quyết:**
- Lãnh đạo đã xem xét toàn bộ hồ sơ
- Hồ sơ đang ở bước chờ phê duyệt và được phân công cho lãnh đạo này

**Kết quả mong đợi:**
- Quyết định phê duyệt được ghi nhận với thông tin đầy đủ: người phê duyệt, thời điểm, ghi chú
- Hồ sơ chuyển đến bước trả kết quả
- Người dùng cuối nhận thông báo hồ sơ đã được phê duyệt

**Notification liên quan:** Gửi cho người dùng cuối (kết quả phê duyệt), gửi cho bộ phận trả kết quả

---

### 8. Lãnh đạo từ chối

**Bối cảnh:** Lãnh đạo không đồng ý với kết quả thẩm định hoặc hồ sơ không đủ điều kiện, ra quyết định từ chối.

**Actor chính:** Lãnh đạo phê duyệt

**Luồng xử lý:**
1. Lãnh đạo xem xét hồ sơ và kết quả thẩm định
2. Không đồng ý, chọn action "Từ chối"
3. **Bắt buộc** nhập lý do từ chối cụ thể
4. Hệ thống ghi nhận quyết định từ chối và lý do
5. Hồ sơ chuyển sang trạng thái "Đã từ chối" (đóng)
6. Người dùng cuối nhận thông báo với lý do từ chối rõ ràng

**Điều kiện tiên quyết:**
- Lãnh đạo phải nhập lý do từ chối (không được để trống)

**Kết quả mong đợi:**
- Hồ sơ được đóng với lý do từ chối được ghi lại rõ ràng
- Người dùng cuối hiểu lý do và có cơ sở để khiếu nại hoặc nộp lại
- Lịch sử quyết định được lưu đầy đủ cho mục đích audit

**Notification liên quan:** Gửi cho người dùng cuối với lý do từ chối (bắt buộc có lý do cụ thể)

---

### 9. Hồ sơ quá hạn xử lý

**Bối cảnh:** Hồ sơ không được xử lý trong thời hạn SLA đã cấu hình tại một bước nào đó.

**Actor chính:** Hệ thống (phát hiện tự động), Người xử lý, Lãnh đạo / Giám sát vận hành

**Luồng xử lý:**
1. Hệ thống theo dõi SLA của từng hồ sơ theo từng bước
2. Khi đến ngưỡng cảnh báo (ví dụ: còn 1 ngày): gửi nhắc nhở cho người đang xử lý
3. Khi vượt quá SLA: đánh dấu hồ sơ là "Quá hạn", gửi cảnh báo leo thang
4. Escalation level 1: thông báo trưởng phòng
5. Escalation level 2 (nếu vẫn chưa xử lý): thông báo giám đốc / cấp cao hơn
6. Hồ sơ vẫn tiếp tục được xử lý bình thường – escalation chỉ là cảnh báo, không tự động hủy

**Điều kiện tiên quyết:**
- SLA đã được cấu hình cho bước tương ứng
- Escalation rule đã được thiết lập trong workflow

**Kết quả mong đợi:**
- Hồ sơ quá hạn được nhận diện và cảnh báo kịp thời
- Lãnh đạo biết về tình trạng quá hạn để can thiệp nếu cần
- Người dùng cuối có thể theo dõi trạng thái và biết hồ sơ đang bị chậm

**Notification liên quan:** Gửi cảnh báo cho người xử lý, trưởng phòng, giám đốc theo mức leo thang; tùy chọn thông báo cho người dùng cuối về sự chậm trễ

---

### 10. Hồ sơ có kết quả và chờ trả cho người dùng

**Bối cảnh:** Hồ sơ đã hoàn tất xử lý và được phê duyệt, bộ phận trả kết quả chuẩn bị trả cho người dùng.

**Actor chính:** Bộ phận trả kết quả, Người dùng cuối

**Luồng xử lý:**
1. Hồ sơ đến bước trả kết quả sau khi được phê duyệt
2. Bộ phận trả kết quả nhận thông báo và chuẩn bị kết quả (văn bản, giấy tờ, quyết định)
3. Người dùng được thông báo đến nhận kết quả (trực tiếp) hoặc kết quả được gửi trực tuyến
4. Người dùng nhận kết quả
5. Bộ phận trả kết quả xác nhận "Đã trả" trong hệ thống
6. Hệ thống đóng hồ sơ và ghi nhận thời gian hoàn thành

**Điều kiện tiên quyết:**
- Hồ sơ đã được phê duyệt và có kết quả đầy đủ
- Bộ phận trả kết quả đã chuẩn bị tài liệu

**Kết quả mong đợi:**
- Người dùng nhận được kết quả đúng hạn
- Hồ sơ được đóng với đầy đủ thông tin: ai trả, khi nào trả, hình thức trả
- Hệ thống có thể báo cáo thời gian xử lý thực tế so với SLA cam kết

**Notification liên quan:** Gửi cho người dùng cuối (thông báo kết quả đã sẵn sàng / mời đến nhận)

---

### 11. Phê duyệt song song liên phòng ban

**Bối cảnh:** Một hồ sơ (vd: cấp phép xây dựng) cần được phê duyệt đồng thời từ nhiều phòng ban độc lập (Phòng Tài nguyên, Phòng Quy hoạch, Phòng PCCC) trước khi chuyển sang bước tiếp theo.

**Actor chính:** Nhiều phòng ban phê duyệt (mỗi phòng xử lý một nhánh độc lập)

**Cấu hình bước:** `IsParallel = true`, `CompletionRule = ALL_APPROVED`, `RejectionPolicy = FAIL_FAST`

**Luồng xử lý:**
1. Hồ sơ chuyển đến bước phê duyệt liên phòng
2. Hệ thống **đồng thời** tạo trạng thái PENDING cho mỗi nhánh và gửi thông báo đến tất cả phòng ban liên quan
3. Mỗi phòng ban xem xét hồ sơ và ra quyết định độc lập (APPROVE hoặc REJECT)
4. Sau mỗi phản hồi, engine đánh giá lại `CompletionRule`:
   - Nếu một phòng **từ chối** và `RejectionPolicy = FAIL_FAST` → bước thất bại ngay, hồ sơ chuyển sang nhánh từ chối
   - Nếu tất cả phòng **phê duyệt** → bước hoàn thành, hồ sơ chuyển sang bước tiếp theo
5. Người dùng cuối nhận thông báo kết quả cuối cùng

**Điều kiện tiên quyết:**
- Bước được cấu hình `IsParallel = true` với ít nhất 2 `ParallelApprovalBranch`
- Tất cả phòng ban trong danh sách đã được thiết lập trong hệ thống

**Kết quả mong đợi:**
- Tất cả phòng ban nhận thông báo cùng lúc – không phải chờ tuần tự
- Mỗi phòng ban chỉ thấy và thao tác trên nhánh của mình
- Trạng thái từng nhánh (PENDING / APPROVED / REJECTED) hiển thị rõ ràng
- SLA của bước được tính từ lúc bước bắt đầu (song song), không phải tổng cộng các nhánh

**Notification liên quan:** Gửi đồng thời đến tất cả phòng ban liên quan (`ON_ENTER` bước song song); gửi cảnh báo khi nhánh sắp quá SLA

---

## Nhóm tình huống ưu tiên triển khai

Trong giai đoạn MVP, ưu tiên triển khai các tình huống theo thứ tự:

| Ưu tiên | Tình huống | Lý do |
|---------|-----------|-------|
| 1 | Nộp hồ sơ mới (TH1) | Điểm bắt đầu của mọi luồng xử lý |
| 2 | Tiếp nhận chính thức (TH3) | Xác nhận hồ sơ đi vào luồng chính thức |
| 3 | Phân công chuyên viên (TH4) | Cơ chế phân công tự động là tính năng cốt lõi |
| 4 | Phê duyệt (TH7) và Từ chối (TH8) | Quyết định cuối cùng của quy trình |
| 5 | Trả kết quả (TH10) | Kết thúc vòng đời hồ sơ |
| 6 | Yêu cầu bổ sung (TH2) | Tình huống phổ biến thực tế |
| 7 | Quá hạn / Escalation (TH9) | Cơ chế kiểm soát SLA |
| 8 | Phê duyệt song song (TH11) | Tính năng đã đưa vào phạm vi; cần engine hỗ trợ `IsParallel` |
| 9 | Chuyển liên phòng ban (TH5) | Cần thiết nhưng phức tạp hơn |
| 10 | Trình phê duyệt (TH6) | Thường đơn giản hóa ở MVP |

---

## Liên hệ với notification

Mỗi tình huống đều kích hoạt ít nhất một notification. Xem chi tiết tại `05-notification-rules.md`.

Tóm tắt:

| Tình huống | Người nhận thông báo |
|-----------|---------------------|
| TH1: Nộp mới | Người dùng cuối, bộ phận tiếp nhận |
| TH2: Yêu cầu bổ sung | Người dùng cuối |
| TH3: Tiếp nhận | Người dùng cuối, chuyên viên được phân công |
| TH4: Phân công | Chuyên viên được phân công |
| TH5: Chuyển liên phòng | Phòng ban được tham vấn |
| TH6: Trình phê duyệt | Lãnh đạo phê duyệt |
| TH7: Phê duyệt | Người dùng cuối, bộ phận trả kết quả |
| TH8: Từ chối | Người dùng cuối (bắt buộc có lý do) |
| TH9: Quá hạn | Người xử lý, trưởng phòng, giám đốc (theo mức) |
| TH10: Trả kết quả | Người dùng cuối |
| TH11: Phê duyệt song song | Tất cả phòng ban trong `ParallelApprovalBranch` (đồng thời) |

---

## Liên hệ với workflow setup

Để các tình huống này hoạt động đúng, workflow phải được cấu hình đầy đủ:

- **TH1, TH3:** Cần có bước tiếp nhận với action phù hợp
- **TH2:** Cần có transition "Yêu cầu bổ sung" quay lại người dùng và transition "Nộp bổ sung" quay lại tiếp nhận
- **TH4:** Cần cấu hình assignment rule cho bước thẩm định
- **TH5:** Cần có bước lấy ý kiến liên phòng và transition tương ứng
- **TH6, TH7, TH8:** Cần có bước phê duyệt với đầy đủ action và transition
- **TH9:** Cần cấu hình SLA và escalation rule cho từng bước
- **TH10:** Cần có bước trả kết quả và action "Xác nhận đã trả"
- **TH11:** Cần cấu hình bước với `IsParallel = true`, ít nhất 2 `ParallelApprovalBranch`, `CompletionRule`, `RejectionPolicy`; engine phải hỗ trợ `ParallelApprovalState` tại runtime
