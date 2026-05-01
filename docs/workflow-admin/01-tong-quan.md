# 01 · Tổng quan & Bài toán

> **Thuộc tài liệu:** [Quản trị Workflow xử lý hồ sơ](./README.md)

---

## Bối cảnh

Hệ thống hiện tại xử lý hồ sơ theo luồng cứng (hard-coded), dẫn đến các vấn đề:

- Khi quy trình nghiệp vụ thay đổi, cần **sửa code và triển khai lại**.
- Thông báo cho cán bộ / bộ phận phụ thuộc thủ công, dễ **sót và chậm trễ**.
- Không phản ánh được **sự đa dạng** giữa các loại hồ sơ (mỗi loại có luồng riêng).

---

## Mục tiêu

Xây dựng hệ thống cho phép **admin cấu hình linh hoạt** quy trình xử lý hồ sơ (workflow), bao gồm:

- Các **bước xử lý** và thứ tự của chúng.
- **Điều kiện chuyển trạng thái** giữa các bước.
- **Phân công** người / bộ phận xử lý.
- **Quy tắc thông báo** tự động.

Thay thế hoàn toàn cách hard-code hiện tại – **không cần sửa code** khi nghiệp vụ thay đổi.

---

## Phạm vi

| Trong phạm vi | Ngoài phạm vi |
|---|---|
| Cấu hình workflow (admin) | Tích hợp cổng dịch vụ công bên ngoài |
| Thực thi workflow khi xử lý hồ sơ | Hệ thống lưu trữ hồ sơ vật lý |
| Thông báo qua Push Notification / Email | Chữ ký số (giai đoạn sau) |
| Giai đoạn thử nghiệm có kiểm soát | Báo cáo thống kê nâng cao (giai đoạn sau) |
| Quản lý phòng ban & vai trò | Quản lý tài khoản người dùng cuối |

---

## Nguyên tắc cốt lõi

> **Admin thiết lập quy trình; người dùng nghiệp vụ chỉ thao tác theo quy trình đã được duyệt.**

- Workflow phải **trải qua giai đoạn thử nghiệm** trước khi đưa vào vận hành chính thức.
- Người dùng cuối **chỉ nhìn thấy và sử dụng workflow đã được kích hoạt** (`ACTIVE`).
- Mọi thay đổi workflow đều được **ghi log đầy đủ** (ai, lúc nào, lý do).

---

*Xem tiếp: [02 · Actors & Phòng ban](./02-actor-phong-ban.md)*
