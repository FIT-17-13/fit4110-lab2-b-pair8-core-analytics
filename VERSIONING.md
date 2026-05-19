# Versioning — Event Contract: Core Business ↔ Analytics

## Phiên bản hiện tại

| Phiên bản | Ngày | Trạng thái | Ghi chú |
|---|---|---|---|
| v1.0.0 | 2026-05-19 | Signed-off | Hợp đồng sự kiện sơ bộ giữa Core Business (B6) và Analytics (B5) |

## Quy tắc versioning

Dùng **Semantic Versioning** (SemVer):

- **MAJOR** (x.0.0): Breaking change — thay đổi không tương thích ngược (VD: xóa field bắt buộc trong event)
- **MINOR** (1.x.0): Feature mới tương thích ngược (VD: thêm event mới, thêm field tùy chọn)
- **PATCH** (1.0.x): Sửa lỗi, cập nhật description, không ảnh hưởng logic

## Lịch sử thay đổi

### v1.0.0 — 2026-05-19

- Khởi tạo Event Contract sơ bộ cho pair-08 (Core Business → Analytics)
- Events: `policy.decision.created`, `alert.created`, `alert.resolved`
- Payload: JSON chuẩn CloudEvents với `eventId`, `correlationId`.
- Đàm phán 5 issue với Analytics, đã sign-off để chuẩn bị cho Lab 03 (AsyncAPI).
