# 04 · Tình huống nghiệp vụ đặc trưng

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

Dưới đây là 10 tình huống làm cơ sở thiết kế workflow và notification rule.

---

## TH-01 · Hồ sơ mới nộp – Thông báo bộ phận tiếp nhận

**Mô tả:** Người dân nộp hồ sơ trực tuyến hoặc tại quầy.  
**Kết quả mong đợi:** Toàn bộ cán bộ tiếp nhận (hoặc trưởng bộ phận) nhận push notification ngay lập tức.  
**Trigger:** `ON_ENTER` bước Tiếp nhận.  
**Recipient:** `DeptType = RECEPTION`.

---

## TH-02 · Hồ sơ thiếu thông tin – Yêu cầu bổ sung

**Mô tả:** Cán bộ tiếp nhận phát hiện hồ sơ chưa đủ, thực hiện action `REQUEST_SUPPLEMENT`.  
**Kết quả mong đợi:** Người dân (submitter) nhận thông báo kèm danh sách tài liệu cần bổ sung.  
**Trigger:** `ON_ACTION = REQUEST_SUPPLEMENT`.  
**Recipient:** `SUBMITTER`.

---

## TH-03 · Chuyển hồ sơ sang Phòng chuyên môn

**Mô tả:** Sau khi hồ sơ hợp lệ, cán bộ tiếp nhận chuyển sang phòng chuyên môn.  
**Kết quả mong đợi:** Phòng chuyên môn (hoặc trưởng phòng) nhận thông báo có hồ sơ mới cần xử lý.  
**Trigger:** `ON_ENTER` bước Chuyên môn.  
**Recipient:** `DeptType = SPECIALIZED` hoặc `Role = SPECIALIST`.

---

## TH-04 · Phân công chuyên viên xử lý

**Mô tả:** Trưởng phòng chuyên môn phân công một chuyên viên cụ thể.  
**Kết quả mong đợi:** Chuyên viên đó nhận thông báo hồ sơ được assign cho mình.  
**Trigger:** `ON_ACTION = ASSIGN`.  
**Recipient:** `ASSIGNED_USER`.

---

## TH-05 · Yêu cầu phối hợp phòng liên quan

**Mô tả:** Chuyên môn cần xin ý kiến từ phòng liên quan (vd: Địa chính, Tư pháp).  
**Kết quả mong đợi:** Phòng liên quan nhận thông báo có yêu cầu phối hợp kèm deadline phản hồi.  
**Trigger:** `ON_ACTION = REQUEST_COLLABORATION`.  
**Recipient:** `TargetDepartment` được chỉ định trong action.

---

## TH-06 · Trình lãnh đạo phê duyệt

**Mô tả:** Chuyên môn hoàn thiện hồ sơ, thực hiện action `SUBMIT_FOR_APPROVAL`.  
**Kết quả mong đợi:** Lãnh đạo có thẩm quyền nhận thông báo có hồ sơ chờ phê duyệt.  
**Trigger:** `ON_ENTER` bước Phê duyệt.  
**Recipient:** `Role = APPROVER` theo cấp thẩm quyền.

---

## TH-07 · Lãnh đạo từ chối – Trả lại chuyên môn

**Mô tả:** Lãnh đạo phát hiện vấn đề, thực hiện action `REJECT`.  
**Kết quả mong đợi:** Chuyên môn (người phụ trách) nhận thông báo kèm lý do từ chối.  
**Trigger:** `ON_ACTION = REJECT`.  
**Recipient:** `ASSIGNED_USER` tại bước Chuyên môn trước đó.

---

## TH-08 · Hồ sơ hoàn tất – Thông báo trả kết quả

**Mô tả:** Lãnh đạo phê duyệt, hồ sơ chuyển sang trạng thái hoàn tất.  
**Kết quả mong đợi:**  
- Bộ phận trả kết quả nhận thông báo hồ sơ sẵn sàng trả.  
- Người dân nhận thông báo hồ sơ đã xử lý xong.  

**Trigger:** `ON_ENTER` bước Trả kết quả.  
**Recipient:** `DeptType = RETURN_DESK` + `SUBMITTER`.

---

## TH-09 · Hồ sơ sắp / đã quá hạn (SLA)

**Mô tả:** Hồ sơ chưa được xử lý xong trong thời hạn quy định.  
**Kết quả mong đợi:**  
- Trước 4 giờ hết hạn: cảnh báo cho người xử lý.  
- Khi quá hạn: leo thang thông báo lên trưởng bộ phận.  
- Nếu tiếp tục quá hạn (cấp 2): leo thang lên lãnh đạo.  

**Trigger:** `ON_OVERDUE`, `ON_ESCALATION`.  
**Recipient:** `ASSIGNED_USER` → `DEPT_MANAGER` → `LEADER`.

---

## TH-10 · Không tìm thấy người xử lý (Fallback)

**Mô tả:** Đến bước phân công nhưng không có ai phù hợp (role trống, bộ phận không có nhân sự online).  
**Kết quả mong đợi:** Hệ thống gửi cảnh báo cho quản trị viên để can thiệp thủ công.  
**Trigger:** `ON_ASSIGNMENT_FAILURE`.  
**Recipient:** `Role = SYSTEM_ADMIN`.

## TH-11 · Bước phê duyệt song song liên phòng ban

**Mô tả:** Một hồ sơ (vd: cấp phép xây dựng) cần được phê duyệt đồng thời từ nhiều phòng ban độc lập (Phòng Tài nguyên, Phòng Quy hoạch, Phòng PCCC) trước khi chuyển sang bước tiếp theo.

**Cấu hình bước:** `IsParallel = true`, `CompletionRule = ALL_APPROVED`, `RejectionPolicy = FAIL_FAST`.

**Kết quả mong đợi:**
- Khi hồ sơ vào bước này, tất cả phòng ban liên quan nhận thông báo **đồng thời**.
- Mỗi phòng ban xử lý độc lập trên nhánh của mình, không chờ nhau.
- Khi đủ điều kiện `CompletionRule` → hồ sơ tự động chuyển sang bước tiếp theo.
- Nếu một phòng từ chối (`FAIL_FAST`) → hồ sơ chuyển sang nhánh từ chối ngay lập tức.

**Trigger:** `ON_ENTER` bước Phê duyệt liên phòng.
**Recipient:** Tất cả `TargetDepartment` của từng `ParallelApprovalBranch`.

**Trạng thái runtime mỗi nhánh:** `PENDING` → `APPROVED` hoặc `REJECTED`.

---

*Xem tiếp: [05 · Vòng đời Workflow](./05-vong-doi-workflow.md)*
