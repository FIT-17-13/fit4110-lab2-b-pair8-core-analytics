# Event Contract sơ bộ — Core Business → Analytics (Pair 08)

> File này ghi nhận thỏa thuận ban đầu cho Queue async ở Lab 02. Đặc tả chi tiết bằng AsyncAPI sẽ chuyển sang Lab 03.

## 1. Thông tin dependency

- Dependency số: 08
- Producer: Core Business (Nhóm 12 — B6)
- Consumer: Analytics (B5)
- Cơ chế: Queue async (RabbitMQ/Kafka)
- Event/topic dự kiến: `core.policy.events`, `core.alert.events`
- Người ghi: Nhóm 12 & Nhóm B5
- Ngày: 2026-05-19

## 2. Mục đích nghiệp vụ

Core Business đẩy các sự kiện quan trọng (như quyết định ra/vào cổng, cảnh báo an ninh được tạo hoặc xử lý xong) sang Analytics. Analytics dùng dữ liệu này để tổng hợp KPI vận hành (số lượt ra vào, tần suất cảnh báo, thời gian xử lý cảnh báo) và hiển thị trên dashboard.

## 3. Event name / topic

| Mục | Giá trị |
|---|---|
| Event 1 | `policy.decision.created` |
| Event 2 | `alert.created` |
| Event 3 | `alert.resolved` |
| Topic/queue | `campus.core.events` |
| Producer | `core-business-service` |
| Consumer | `analytics-service` |

## 4. Payload tối thiểu

**Event `policy.decision.created`:**
```json
{
  "eventId": "0196fb3d-4ad7-7d1e-9f49-5d5148d2babc",
  "eventType": "policy.decision.created",
  "occurredAt": "2026-05-19T08:30:00Z",
  "correlationId": "RFID-2026-001",
  "source": "core-business",
  "data": {
    "decisionId": "0196fb3d-4ad7-7d1e-9f49-5d5148d2babc",
    "policyId": "POL-001",
    "subjectId": "RFID-2026-001",
    "gateId": "GATE-01",
    "result": "ALLOW",
    "reasonCode": "VALID_CARD"
  }
}
```

**Event `alert.resolved`:**
```json
{
  "eventId": "0196fb3d-4ad7-7d1e-9f49-5d5148d2babd",
  "eventType": "alert.resolved",
  "occurredAt": "2026-05-19T09:00:00Z",
  "correlationId": "ALERT-001",
  "source": "core-business",
  "data": {
    "alertId": "ALERT-001",
    "resolvedBy": "ADMIN-01",
    "durationSeconds": 1800,
    "resolutionNote": "False alarm"
  }
}
```

## 5. Ràng buộc cần thống nhất

| Vấn đề | Quyết định tạm thời |
|---|---|
| Event id có bắt buộc không? | Có (UUID v4) để consumer xử lý trùng lặp |
| Có cần correlationId không? | Có, dùng mã thẻ hoặc mã cảnh báo để trace |
| Có cho phép gửi trùng event không? | At-least-once delivery, consumer phải thiết kế idempotent |
| Cấu trúc Event chung | Tuân theo chuẩn CloudEvents |
| Retry khi lỗi | Sẽ setup Dead-letter queue ở Lab 03 |

## 6. Issue chuyển sang Lab 03

1. Định dạng chuẩn hóa `AsyncAPI` (YAML).
2. Quyết định dùng Message Broker nào (RabbitMQ hay Kafka).
3. Cơ chế xử lý khi Analytics bị nghẽn (Backpressure).
