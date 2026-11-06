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
- Một hóa đơn được gắn với một chi nhánh (`MaCN`) để xác định nơi trừ kho và tổng hợp doanh thu. Đây là thuộc tính bổ sung, không đổi hợp đồng công bố.
- Hóa đơn thanh toán thành công là bất biến; hiện chưa hỗ trợ hoàn tiền hoặc sửa/xóa sau thanh toán theo quyết định #16.
- Cách xác định tỷ lệ lấp đầy cần nhóm xác nhận theo quyết định #15; tài liệu tạm dùng số check-in đang mở tại thời điểm chụp chia cho `ChiNhanh.SucChua`.

