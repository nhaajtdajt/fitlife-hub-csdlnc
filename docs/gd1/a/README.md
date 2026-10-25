# Module A — Chi nhánh và Nhân sự

| Mục | Nội dung |
|---|---|
| Người phụ trách | Nguyễn Phúc Hậu |
| Phạm vi | Chi nhánh, dịch vụ/tiện ích tại chi nhánh, nhân viên, lịch sử công tác, điều chuyển và hoa hồng PT |
| Vấn đề trọng tâm | Xác định đúng chi nhánh của PT tại thời điểm diễn ra buổi dạy để tính hoa hồng chính xác |
| Giai đoạn | Giai đoạn 1 — Phân tích, thiết kế mức quan niệm và logic |

## Tài liệu

- [Mô tả quy trình](quy-trinh.md)
- [Bảng phân tích tần suất](bang-tan-suat.md)

## Phạm vi dữ liệu dự kiến

- `ChiNhanh`: thông tin chi nhánh, giờ hoạt động và sức chứa.
- Dịch vụ/tiện ích được cung cấp tại từng chi nhánh.
- `NhanVien`: hồ sơ nhân viên, chức vụ và lương cơ bản hiện tại.
- `LichSuCongTac`: chi nhánh mà nhân viên thuộc về trong từng khoảng thời gian.
- Dữ liệu lịch hẹn PT đã hoàn thành do Module C quản lý, được Module A sử dụng khi tính hoa hồng.

## Các nội dung cần nhóm chốt

1. `ChucVu` chỉ lưu trong `NhanVien` hay cần lưu thêm theo từng giai đoạn công tác.
2. Thời điểm điều chuyển có độ chính xác đến ngày hay ngày giờ.
3. Dịch vụ/tiện ích của chi nhánh được chuẩn hóa thành bảng riêng hay lưu theo phương án khác.
4. Mức hoa hồng PT là cố định toàn hệ thống hay thay đổi theo chi nhánh, chức vụ hoặc thời gian.
5. Hoa hồng được ghi nhận ngay khi hoàn thành buổi dạy hay được tổng hợp khi chốt tháng.

Các tài liệu hiện dùng những giả định được đánh dấu là **đề xuất**; chúng chỉ trở thành quy tắc chính thức sau khi cả nhóm thống nhất.
