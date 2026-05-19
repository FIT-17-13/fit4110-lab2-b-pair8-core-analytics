# Biên bản đàm phán hợp đồng API / Event

- Cặp đàm phán: Core Business ↔ Analytics (Pair 08)
- Product: B
- Producer: Core Business (Nhóm 12 — B6)
- Consumer: Analytics (B5)
- Phiên: v1.0
- Ngày: 2026-05-19

---

## Issue #1 — Định dạng lý do (reason) của decision

- Raised by: Consumer (Analytics)
- Event: `policy.decision.created`
- Concern: Analytics cần lý do từ chối (reason) để phân loại KPI cảnh báo, nhưng nếu là chuỗi text tự do thì rất khó để aggregate (group by).
- Proposal: Core Business chỉ gửi `reasonCode` (chuỗi enum như `CARD_EXPIRED`, `INVALID_GATE`) thay vì text tự do.
- Resolution: Accepted
- Rationale: Dùng mã enum giúp CSDL Analytics group by nhanh hơn và không bị lỗi chính tả. Nếu cần text chi tiết để hiển thị, Analytics sẽ match mã code trên frontend hoặc dùng từ điển.
- Impact: Core Business bỏ field `reasonText`, chỉ gửi `reasonCode`.

---

## Issue #2 — Xử lý nhiều policy cho một decision

- Raised by: Consumer (Analytics)
- Event: `policy.decision.created`
- Concern: Một lượt ra vào (decision) có thể liên quan nhiều policy không? Nếu là mảng (array) thì Analytics sẽ phải làm phẳng (flatten) dữ liệu.
- Proposal: Core Business làm rõ 1 decision chỉ map với 1 policy duy nhất (hoặc null nếu không match policy nào).
- Resolution: Accepted
- Rationale: Thiết kế của Core Business ưu tiên policy có priority cao nhất. Do đó, một decision chỉ áp dụng duy nhất 1 policy.
- Impact: Field `policyId` trong event payload là kiểu string (hoặc null), không phải array.

---

## Issue #3 — Thời gian xử lý cảnh báo (duration)

- Raised by: Consumer (Analytics)
- Event: `alert.resolved`
- Concern: Analytics cần tính "Thời gian trung bình xử lý cảnh báo". Nếu tự lấy thời điểm nhận event `alert.resolved` trừ đi thời điểm nhận event `alert.created` có thể sai số do độ trễ của queue mạng.
- Proposal: Core Business tự tính toán `durationSeconds` (hoặc milliseconds) và đính kèm vào event `alert.resolved`.
- Resolution: Accepted
- Rationale: Core Business nắm rõ thời điểm thực tế tạo và đóng alert trong DB, do đó tính toán ở nguồn sẽ chính xác tuyệt đối.
- Impact: Thêm trường `durationSeconds` vào payload của `alert.resolved`.

---

## Issue #4 — Đảm bảo Idempotency khi trùng event

- Raised by: Producer (Core Business)
- Event: Tất cả events
- Concern: Message queue (như RabbitMQ) có thể gửi lại (at-least-once delivery) dẫn đến trùng event. Nếu Analytics cộng dồn doanh số/số lượt 2 lần thì sẽ sai KPI.
- Proposal: Tất cả event bắt buộc phải có `eventId` duy nhất (UUID). Analytics tự check DB xem `eventId` này đã được xử lý chưa trước khi cập nhật thống kê.
- Resolution: Accepted
- Rationale: Đây là pattern bắt buộc trong hệ thống phân tán.
- Impact: Core Business luôn sinh `eventId` (UUID v4). Analytics thêm logic kiểm tra trùng lặp `eventId` trước khi xử lý.

---

## Issue #5 — Chuyển tiếp định dạng sang CloudEvents

- Raised by: Cả hai bên
- Event: Định dạng payload chung
- Concern: Payload JSON cần chuẩn hóa cấu trúc để sau này dễ quản lý (header, metadata nằm ở đâu).
- Proposal: Áp dụng chuẩn CloudEvents cho tất cả các thông điệp bất đồng bộ.
- Resolution: Accepted
- Rationale: Việc chia tách rõ `eventType`, `occurredAt`, `source` khỏi khối `data` giúp hạ tầng trung gian dễ dàng lọc và định tuyến message.
- Impact: Khối lượng payload thực tế chỉ nằm trong thuộc tính `data`, các trường còn lại đặt ở root theo chuẩn CloudEvents.

---

# Chốt hợp đồng sơ bộ v1.0

Producer sign-off: Nhóm 12 — Core Business (B6)
Consumer sign-off: Analytics (B5)
Witness (GV/TA):   _________________
Date:              2026-05-19
