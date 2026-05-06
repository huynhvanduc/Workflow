# 12. Phân tích Công nghệ

> **Trạng thái tài liệu:** Phân tích – Bản nháp  
> **Cập nhật lần cuối:** 2026-05

---

## Mục tiêu

Tài liệu này phân tích và đề xuất công nghệ phù hợp để triển khai hệ thống workflow xử lý hồ sơ hành chính công, dựa trên yêu cầu nghiệp vụ đã được xác định trong các tài liệu phân tích trước.

---

## 1. Tổng quan kiến trúc đề xuất

Hệ thống được thiết kế theo kiến trúc **Layered Architecture** (phân tầng), với khả năng mở rộng sang kiến trúc **Modular Monolith** khi quy mô tăng lên. Các thành phần chính:

```
┌────────────────────────────────────────────────────────┐
│                    React Frontend                       │
│         (Admin Portal · Cán bộ Portal · Citizen Portal)│
└────────────────────────────┬───────────────────────────┘
                             │ HTTPS / REST API
┌────────────────────────────▼───────────────────────────┐
│                  .NET Backend (ASP.NET Core)             │
│   ┌─────────────────────────────────────────────────┐  │
│   │  API Layer (Controllers, Middleware, Auth)       │  │
│   ├─────────────────────────────────────────────────┤  │
│   │  Application Layer (Use Cases, CQRS Handlers)   │  │
│   ├─────────────────────────────────────────────────┤  │
│   │  Domain Layer (Workflow Engine, Business Rules) │  │
│   ├─────────────────────────────────────────────────┤  │
│   │  Infrastructure Layer (DB, Cache, Messaging,    │  │
│   │                        Notification)            │  │
│   └─────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
         │            │             │             │
    PostgreSQL      Redis      Message Bus   Push/Email
    (Primary DB)   (Cache)    (Background)  (Notification)
```

---

## 2. Backend

### 2.1 Ngôn ngữ và Framework

| Thành phần | Công nghệ | Phiên bản đề xuất | Lý do |
|---|---|---|---|
| Runtime | .NET | 8.0 LTS | Hỗ trợ dài hạn, hiệu năng cao, cross-platform |
| Web Framework | ASP.NET Core | 8.0 | Tích hợp sẵn DI, middleware pipeline, minimal API |
| ORM | Entity Framework Core | 8.x | Hỗ trợ migrations, LINQ, code-first modeling |
| Validation | FluentValidation | 11.x | Validation phức tạp cho cấu hình workflow |
| Mapping | AutoMapper hoặc Mapster | – | Chuyển đổi giữa domain model và DTO |

### 2.2 Mô hình thiết kế

**CQRS (Command Query Responsibility Segregation)** được áp dụng vì:
- Workflow configuration commands (tạo, sửa, publish) và queries (báo cáo, audit log) có đặc tính đọc/ghi rất khác nhau
- Tách biệt phần đọc để tối ưu hiệu năng query báo cáo mà không ảnh hưởng đến write path

**Domain-Driven Design (DDD):**
- `WorkflowDefinition`, `Step`, `Transition`, `CaseInstance` là các Aggregate roots
- Workflow lifecycle logic được đóng gói trong domain model, không rải rác tại Controller
- Domain events (WorkflowPublished, CaseStepChanged) cho phép decoupled integration

### 2.3 Workflow Engine

Hệ thống cần tự xây dựng một **Workflow Runtime Engine** nhỏ gọn, bởi các lý do:
- Workflow được admin cấu hình động (không cố định ở compile-time)
- Cần kiểm soát hoàn toàn vòng đời và phiên bản

**Các thành phần của Engine:**

| Thành phần | Trách nhiệm |
|---|---|
| `WorkflowLoader` | Đọc định nghĩa workflow từ database và build runtime graph |
| `StepResolver` | Xác định bước hiện tại của một CaseInstance |
| `TransitionEvaluator` | Đánh giá điều kiện chuyển trạng thái (transition condition) |
| `AssignmentResolver` | Xác định người/phòng ban được phân công tại mỗi bước |
| `ParallelApprovalEvaluator` | Khởi tạo nhánh song song, theo dõi trạng thái từng nhánh, đánh giá `CompletionRule` và `RejectionPolicy` sau mỗi phản hồi |
| `SLAMonitor` | Theo dõi thời hạn và kích hoạt cảnh báo / leo thang |
| `NotificationDispatcher` | Gửi thông báo đúng đối tượng theo quy tắc đã cấu hình |

> **Lưu ý:** Không nên dùng các workflow engine thương mại (Windows Workflow Foundation, Camunda, Temporal) vì chúng thiếu tính linh hoạt để tích hợp với mô hình phân quyền và SLA tùy chỉnh theo nghiệp vụ hành chính công.

---

## 3. Frontend

### 3.1 Công nghệ chính

| Thành phần | Công nghệ | Lý do |
|---|---|---|
| Framework | React 18+ | Đã xác định trong tech stack; hệ sinh thái lớn, component-based |
| Language | TypeScript | Type safety, giảm bug khi xử lý cấu hình workflow phức tạp |
| Build Tool | Vite | Nhanh hơn Create React App, HMR hiệu quả |
| State Management | Zustand hoặc Redux Toolkit | Quản lý trạng thái phức tạp của form cấu hình workflow |
| UI Component Library | Ant Design hoặc Material UI | Bảng dữ liệu, form phức tạp, timeline, drag-and-drop |
| HTTP Client | Axios | Interceptors, error handling tập trung |
| Form | React Hook Form + Zod | Validate form cấu hình workflow nhiều trường |
| Workflow Designer | React Flow | Vẽ sơ đồ workflow dạng node-graph (drag & drop bước, transition) |

### 3.2 Các portal cần xây dựng

| Portal | Đối tượng | Tính năng chính |
|---|---|---|
| **Admin Portal** | SYSTEM_ADMIN | Thiết kế workflow, quản lý SLA, audit log, báo cáo |
| **Staff Portal** | Cán bộ nghiệp vụ | Xử lý hồ sơ, xem nhật ký, nhận thông báo |
| **Citizen Portal** | Người dân | Nộp hồ sơ, theo dõi trạng thái, nhận thông báo |

---

## 4. Cơ sở dữ liệu

### 4.1 Database chính

**PostgreSQL 15+** được đề xuất làm database chính:

| Tiêu chí | Lý do chọn PostgreSQL |
|---|---|
| JSON/JSONB support | Lưu `OldValue`/`NewValue` trong audit log, `Metadata` trong CaseAuditLog |
| ACID compliance | Đảm bảo tính toàn vẹn khi ghi audit log trong cùng transaction |
| Full-text search | Tìm kiếm hồ sơ, nhật ký không cần Elasticsearch cho quy mô vừa |
| Row-level security | Hỗ trợ phân quyền theo phòng ban tại tầng database |
| Mature ecosystem | Tích hợp tốt với EF Core, công cụ backup/restore phong phú |

### 4.2 Các bảng dữ liệu cốt lõi

| Nhóm | Bảng | Mô tả |
|---|---|---|
| **Workflow Config** | `WorkflowDefinitions` | Định nghĩa workflow và phiên bản |
| | `WorkflowSteps` | Các bước trong workflow (bao gồm cả bước song song) |
| | `ParallelApprovalBranches` | Các nhánh phê duyệt của bước song song (`IsParallel = true`) |
| | `WorkflowActions` | Hành động có thể thực hiện tại mỗi bước |
| | `WorkflowTransitions` | Điều kiện chuyển bước |
| | `AssignmentRules` | Quy tắc phân công tự động (bước thông thường) |
| | `NotificationRules` | Quy tắc gửi thông báo |
| | `SlaConfigs` | Cấu hình SLA và leo thang |
| **Case Runtime** | `CaseInstances` | Hồ sơ đang xử lý |
| | `CaseStepStates` | Trạng thái từng bước của hồ sơ |
| | `CaseAssignments` | Phân công cán bộ xử lý |
| | `ParallelApprovalStates` | Trạng thái từng nhánh của bước song song (PENDING / APPROVED / REJECTED) |
| **Audit** | `CaseAuditLogs` | Nhật ký xử lý hồ sơ |
| | `WorkflowConfigAuditLogs` | Nhật ký cấu hình workflow |
| **Org** | `Departments` | Phòng ban / bộ phận |
| | `UserRoles` | Phân quyền người dùng |

### 4.3 Caching

**Redis** được sử dụng cho:

| Use case | Chiến lược | TTL đề xuất |
|---|---|---|
| Workflow definition đang ACTIVE | Cache-aside, invalidate khi publish mới | Until next publish |
| Session / JWT token blacklist | Write-through | Theo thời gian token |
| SLA counter (thời gian còn lại) | Sliding window | 1 giờ |
| Rate limiting admin operations | Token bucket | 1 phút |

---

## 5. Hệ thống thông báo (Notification)

### 5.1 Kiến trúc thông báo

```
Business Event
    │
    ▼
NotificationDispatcher
    │
    ├──► Push Notification (FCM/APNs)   ← Kênh ưu tiên
    ├──► Email (SMTP / SendGrid)         ← Kênh bổ sung
    └──► SMS (Twilio / VIETTEL)          ← Kênh dự phòng
```

### 5.2 Công nghệ đề xuất

| Kênh | Công nghệ | Lưu ý |
|---|---|---|
| Push Notification | Firebase Cloud Messaging (FCM) | Miễn phí, hỗ trợ cả Android/iOS/Web |
| Email | SMTP nội bộ hoặc SendGrid | SendGrid có delivery tracking, bounce handling |
| SMS | API nhà mạng nội địa (Viettel, VNPT) | Ưu tiên nhà mạng trong nước để giảm latency |
| In-app Notification | SignalR (WebSocket) | Real-time notification không cần reload trang |

### 5.3 Xử lý bất đồng bộ

Thông báo được gửi **bất đồng bộ** qua message queue để tránh chặn luồng nghiệp vụ:

- **Message Bus:** RabbitMQ hoặc Azure Service Bus
- **Background Worker:** .NET Worker Service (IHostedService)
- **Retry policy:** Exponential backoff, tối đa 3 lần retry, lưu vào dead-letter queue nếu thất bại

---

## 6. Bảo mật

### 6.1 Xác thực và phân quyền

| Thành phần | Công nghệ | Ghi chú |
|---|---|---|
| Authentication | JWT Bearer Token + Refresh Token | Access token TTL: 15 phút; Refresh token TTL: 7 ngày |
| Authorization | Policy-based Authorization (ASP.NET Core) | Phân quyền theo Role + Department + Workflow state |
| Identity Provider | Keycloak hoặc tự xây | Keycloak nếu cần SSO với hệ thống hành chính khác |
| Password hashing | BCrypt hoặc Argon2 | Không dùng MD5/SHA1 |

### 6.2 Bảo vệ API

- **Rate limiting:** Giới hạn số request per IP/user tránh abuse
- **Input validation:** FluentValidation ở Application layer; không tin tưởng dữ liệu từ client
- **CORS:** Chỉ cho phép domain được whitelist
- **HTTPS:** Bắt buộc TLS 1.2+; HSTS header
- **SQL Injection:** EF Core parameterized queries; không dùng raw SQL trừ khi cần thiết
- **XSS:** Content Security Policy header; sanitize input HTML

### 6.3 Bảo vệ Audit Log

- Bảng audit log chỉ được INSERT bởi service account ứng dụng
- Không có user nào (kể cả Admin) được UPDATE/DELETE qua application layer
- Database-level: `REVOKE UPDATE, DELETE ON CaseAuditLogs FROM app_user`
- Xem xét checksum (SHA-256) cho mỗi bản ghi để phát hiện tamper

---

## 7. Hiệu năng và Khả năng mở rộng

### 7.1 Database optimization

| Chiến lược | Áp dụng cho |
|---|---|
| Index theo `CaseId`, `PerformedAt` | Truy vấn audit log theo hồ sơ |
| Index theo `WorkflowId`, `Status` | Tìm workflow ACTIVE theo loại hồ sơ |
| Partial index | Index chỉ trên hồ sơ đang xử lý (status ≠ CLOSED) |
| Table partitioning | Partition `CaseAuditLogs` theo tháng để truy vấn nhanh |
| Read replica | Tách read traffic (báo cáo) khỏi write traffic (xử lý hồ sơ) |

### 7.2 Horizontal scaling

- API layer: Stateless → dễ scale ngang (load balancer + multiple instances)
- Session: Lưu trên Redis, không lưu in-memory → không sticky session
- Background workers: Scale độc lập với API layer

### 7.3 SLA Monitoring

- **Cron job / Quartz.NET**: Chạy mỗi 5-15 phút để kiểm tra hồ sơ sắp quá hạn
- **Hoặc:** Hangfire cho scheduling linh hoạt hơn, có dashboard quản lý job

---

## 8. Hạ tầng và Triển khai

### 8.1 Môi trường

| Môi trường | Mục đích |
|---|---|
| **Development** | Lập trình viên local |
| **Testing/Staging** | Tester UAT, kiểm tra trước deploy |
| **Production** | Vận hành thực tế |

### 8.2 Containerization

- **Docker**: Container hóa toàn bộ service (API, Worker, Frontend)
- **Docker Compose**: Dùng cho môi trường development và testing
- **Kubernetes (tùy chọn)**: Khi quy mô vận hành lớn, cần auto-scaling

### 8.3 CI/CD Pipeline

```
Git Push
  │
  ▼
Build & Unit Test (GitHub Actions / Azure DevOps)
  │
  ▼
Static Analysis + Security Scan (SonarQube / Snyk)
  │
  ▼
Docker Build & Push to Registry
  │
  ▼
Deploy to Staging → Integration Test
  │
  ▼
Manual Approval → Deploy to Production
```

### 8.4 Monitoring và Logging

| Thành phần | Công nghệ |
|---|---|
| Application Logging | Serilog → Elasticsearch / Loki |
| Metrics | Prometheus + Grafana |
| Distributed Tracing | OpenTelemetry + Jaeger |
| Alerting | Grafana Alertmanager |
| Health Check | ASP.NET Core Health Checks |

---

## 9. Tóm tắt stack công nghệ

| Tầng | Công nghệ |
|---|---|
| **Frontend** | React 18, TypeScript, Vite, Ant Design, React Flow |
| **Backend** | .NET 8, ASP.NET Core, EF Core, CQRS, DDD |
| **Database** | PostgreSQL 15 |
| **Cache** | Redis |
| **Message Queue** | RabbitMQ |
| **Push Notification** | Firebase Cloud Messaging (FCM) |
| **Real-time** | SignalR |
| **Auth** | JWT + Keycloak |
| **Container** | Docker, Docker Compose |
| **CI/CD** | GitHub Actions hoặc Azure DevOps |
| **Monitoring** | Serilog, Prometheus, Grafana |

---

## 10. Câu hỏi cần chốt thêm

| # | Câu hỏi | Mức độ ưu tiên |
|---|---------|----------------|
| 1 | Hệ thống triển khai on-premise hay cloud (Azure, AWS)? Ảnh hưởng đến lựa chọn managed services | Cao |
| ~~2~~ | ~~Cần tích hợp SSO với hệ thống hành chính hiện có không? Nếu có, dùng giao thức gì (SAML, OIDC)?~~ | ✅ **Đã chốt:** Optional – hiện tại chưa triển khai |
| ~~3~~ | ~~Yêu cầu về số lượng hồ sơ đồng thời (concurrent cases) và số người dùng đồng thời?~~ | ✅ **Đã chốt:** 1000 hồ sơ / người dùng đồng thời |
| 4 | Cần tuân thủ tiêu chuẩn bảo mật nào (ISO 27001, TCVN, Bộ TT&TT)? | Trung bình |
| 5 | Có kế hoạch dùng mobile app (iOS/Android) riêng hay chỉ web? Ảnh hưởng đến push notification | Trung bình |
| 6 | Quy định về lưu trữ dữ liệu: phải lưu trong nước (data sovereignty)? | Trung bình |
