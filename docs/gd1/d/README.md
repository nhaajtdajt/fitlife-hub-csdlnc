# Module D — Sản phẩm, Hóa đơn và Báo cáo

| Mục | Nội dung |
|---|---|
| Người phụ trách | Module D |
| Phạm vi | Danh mục sản phẩm, tồn kho theo chi nhánh, lập hóa đơn và báo cáo vận hành/doanh thu |
| Vấn đề trọng tâm | Log hóa đơn lớn và báo cáo doanh thu chậm; chuẩn bị dữ liệu để phân vùng, tổng hợp và tối ưu ở giai đoạn 2 |
| Giai đoạn | Giai đoạn 1 — Phân tích, thiết kế mức quan niệm và logic |

## Tài liệu

- [Mô tả quy trình](quy-trinh.md)
- [Bảng phân tích tần suất](bang-tan-suat.md)
- [ERD và ràng buộc quan niệm](erd.md)
- [Thiết kế logic](thiet-ke-logic.md)

## Phạm vi dữ liệu

- `SanPham`: danh mục hàng hóa bán lẻ.
- `TonKho`: số lượng từng sản phẩm tại từng chi nhánh.
- `HoaDon`: giao dịch bán hàng, người mua, nhân viên lập, chi nhánh, tiền và phương thức thanh toán.
- `ChiTietHoaDon`: các dòng sản phẩm hoặc gói dịch vụ của hóa đơn.
- `CheckIn` của Module C được đọc để lập báo cáo tỷ lệ lấp đầy; không thuộc quyền sở hữu của D.

## Quyết định và giả định dùng trong thiết kế

- Hóa đơn có thể gồm cả gói dịch vụ và sản phẩm; gói do Module B quản lý, sản phẩm do D quản lý.
- `HoaDon.TongTien` là số tiền cuối cùng sau giảm giá và là giá trị D công bố cho B để cộng chi tiêu/điểm.
- Dòng hóa đơn lưu đơn giá tại thời điểm bán. Sửa giá danh mục sau đó không làm thay đổi hóa đơn cũ.
- GD1-08/#18: mọi doanh thu tính sau giảm giá; phân bổ trên toàn hóa đơn theo tỷ lệ ThanhTien trước giảm, gồm gói và sản phẩm, rồi mới lọc báo cáo. Làm tròn theo đồng, phân phần dư giảm dần/hòa theo STT để khớp tổng. Xem [quy tắc phân bổ](thiet-ke-logic.md#phân-bổ-giảm-giá-và-doanh-thu-dòng-gd1-08); không thêm cột lưu kết quả tính ra.
- Một hóa đơn được gắn với một chi nhánh (`MaCN`) để xác định nơi trừ kho và tổng hợp doanh thu. Đây là thuộc tính bổ sung, không đổi hợp đồng công bố.
- Hóa đơn thanh toán thành công là bất biến; hiện chưa hỗ trợ hoàn tiền hoặc sửa/xóa sau thanh toán theo quyết định #16.
- GD1-07/#15: chỉ báo cáo lấp đầy, không chặn cổng. Tỷ lệ = 100 × số phiên có TGVao <= T và (TGRa rỗng hoặc TGRa > T) / SucChua nguyên dương; không giới hạn 100%. QT-D5 nêu thời điểm chụp theo giờ, lượt tự đóng là ước tính và báo cáo tạm tính khi còn lượt quá hạn chưa đóng.


GD1-12: bỏ ThanhTien khỏi bảng cơ sở và ERD; tính SoLuong × DonGia khi đọc/in/báo cáo. Phân bổ giảm giá GD1-08 và TongTien hóa đơn không thay đổi; đơn giá lịch sử vẫn được lưu.

GD1-13: bảng tầm ảnh hưởng đã phân biệt ràng buộc miền, tổng liên bảng, điều kiện tại lúc thanh toán và bất biến lịch sử. Dấu − không cho phép bỏ qua FK hoặc quy định cấm xóa.

GD1-14: [ERD Mermaid](erd.md) đã parse/render bằng Mermaid 10.9.3 và kiểm tra ảnh; [bản SVG](erd.svg) là bản xuất từ sơ đồ này. Khi sửa ERD cần xuất lại SVG.

Bản ghép hiện hành: [hồ sơ GĐ1 tổng](../README.md), [ERD tổng](../erd-tong.md), [lược đồ và ràng buộc tổng](../thiet-ke-tong.md).
