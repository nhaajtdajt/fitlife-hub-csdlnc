# Module B — Hội viên, gói dịch vụ và điểm/hạng

- [Quy trình](quy-trinh.md)
- [Tần suất](bang-tan-suat.md)
- [ERD hiện hành](erd.md)
- [Thiết kế logic đầy đủ](thiet-ke-logic.md)
- [Giữ/trừ buổi PT](goi-pt.md)
- [Điểm và hạng 12 tháng](diem-hang.md)

GD1-11: gói đã mua gắn dòng hóa đơn và số thứ tự đơn vị; giao dịch hoàn tất nguyên tử và xử lý lại không tạo trùng. MaHV/MaGoi của gói được đọc qua vGoiHoiVien, không lưu lặp trong bảng cơ sở. HoiVien giữ hạng dư thừa theo quyết định GD1-09, chỉ đạt 2NF; không tuyên bố cả module BCNF. Ngưỡng hạng vẫn cần cô xác nhận, bảo lưu/đổi điểm chưa triển khai.

Bản ghép hiện hành: [hồ sơ GĐ1 tổng](../README.md), [ERD tổng](../erd-tong.md), [lược đồ và ràng buộc tổng](../thiet-ke-tong.md).
