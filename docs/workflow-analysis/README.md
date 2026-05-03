# Tài liệu phân tích workflow xử lý hồ sơ

Bộ tài liệu này mô tả bài toán nghiệp vụ cho chức năng cấu hình workflow xử lý hồ sơ, cơ chế thử nghiệm trước khi đưa vào sử dụng chính thức, theo dõi phản ánh/lỗi, và vai trò của các đối tượng tham gia trong hệ thống.

## Mục tiêu

- Chuẩn hóa mô tả nghiệp vụ cho workflow xử lý hồ sơ
- Cho phép admin thiết lập quy trình thay vì hard-code
- Làm rõ vai trò của các phòng ban và đối tượng người dùng
- Xác định vòng đời workflow từ nháp, thử nghiệm đến chính thức
- Làm cơ sở cho phân tích, thiết kế và triển khai sau này

## Danh sách tài liệu

| # | File | Mô tả |
|---|------|--------|
| 01 | [01-overview.md](./01-overview.md) | Tổng quan – bối cảnh, mục tiêu, phạm vi |
| 02 | [02-actors-and-departments.md](./02-actors-and-departments.md) | Đối tượng và phòng ban |
| 03 | [03-workflow-definition.md](./03-workflow-definition.md) | Khái niệm và thành phần workflow |
| 04 | [04-business-scenarios.md](./04-business-scenarios.md) | Tình huống nghiệp vụ đặc trưng |
| 05 | [05-notification-rules.md](./05-notification-rules.md) | Quy tắc notification |
| 06 | [06-workflow-lifecycle.md](./06-workflow-lifecycle.md) | Vòng đời workflow |
| 07 | [07-testing-and-uat.md](./07-testing-and-uat.md) | Thử nghiệm và UAT |
| 08 | [08-issue-feedback-tracking.md](./08-issue-feedback-tracking.md) | Phản ánh lỗi và theo dõi fix |
| 09 | [09-governance-and-activation.md](./09-governance-and-activation.md) | Nguyên tắc quản trị và điều kiện kích hoạt |
| 10 | [10-end-user-checklist.md](./10-end-user-checklist.md) | Checklist cho người dùng cuối |
| 11 | [11-audit-log.md](./11-audit-log.md) | Audit Log – Nhật ký thao tác (nhân viên & admin) |
| 12 | [12-technology-analysis.md](./12-technology-analysis.md) | Phân tích công nghệ – stack, kiến trúc, hạ tầng |
| 13 | [13-challenges-and-risks.md](./13-challenges-and-risks.md) | Dự đoán thách thức và issue có thể xảy ra |

## Gợi ý thứ tự đọc

1. [01-overview.md](./01-overview.md) – Bắt đầu từ đây để hiểu bức tranh tổng thể
2. [02-actors-and-departments.md](./02-actors-and-departments.md) – Ai tham gia, phòng ban nào liên quan
3. [03-workflow-definition.md](./03-workflow-definition.md) – Các khái niệm kỹ thuật nghiệp vụ
4. [04-business-scenarios.md](./04-business-scenarios.md) – Tình huống thực tế
5. [06-workflow-lifecycle.md](./06-workflow-lifecycle.md) – Workflow sống qua những trạng thái nào
6. [07-testing-and-uat.md](./07-testing-and-uat.md) – Cách thử nghiệm trước khi đưa vào dùng thật
7. [08-issue-feedback-tracking.md](./08-issue-feedback-tracking.md) – Ghi nhận và xử lý phản ánh lỗi
8. [09-governance-and-activation.md](./09-governance-and-activation.md) – Điều kiện và nguyên tắc công bố chính thức
9. [10-end-user-checklist.md](./10-end-user-checklist.md) – Checklist từ góc nhìn người dùng cuối
10. [12-technology-analysis.md](./12-technology-analysis.md) – Phân tích công nghệ và stack kỹ thuật
11. [13-challenges-and-risks.md](./13-challenges-and-risks.md) – Dự đoán thách thức và issue có thể xảy ra

## Tài liệu liên quan

- [Đặc tả nghiệp vụ (docs/workflow-admin)](../workflow-admin/dac-ta-nghiep-vu.md)
