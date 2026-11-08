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
- [ERD hiện hành](erd.md)
- [Thiết kế logic đầy đủ A](thiet-ke-logic.md)

## Phạm vi dữ liệu dự kiến

- `ChiNhanh`: thông tin chi nhánh, giờ hoạt động và sức chứa.
- Dịch vụ/tiện ích được cung cấp tại từng chi nhánh.
- `NhanVien`: hồ sơ nhân viên, chức vụ và lương cơ bản hiện tại.
- `LichSuCongTac`: chi nhánh và chức vụ theo từng giai đoạn DATE, tính cả ngày bắt đầu/kết thúc. Vị trí mới bắt đầu ngày sau ngày kết thúc cũ; yêu cầu trong ngày có hiệu lực ngày mai.
- Dữ liệu lịch hẹn PT đã hoàn thành do Module C quản lý, được Module A sử dụng khi tính hoa hồng.

## Các nội dung cần nhóm chốt

1. **Đã chốt GD1-05:** ChucVu lưu ở NhanVien cho hiện tại và LichSuCongTac cho từng giai đoạn; lịch sử là nguồn xác định chức vụ tại ngày cần tra cứu.
2. **Đã chốt GD1-05:** DATE, bao gồm hai biên. Bấm đổi ngày 15 thì vị trí cũ hết ngày 15, mới bắt đầu ngày 16; chức vụ hiện tại chỉ đồng bộ khi đến ngày hiệu lực theo QT-A04.
3. GD1-10: TienIch và CungCapTienIch lưu danh mục và các khoảng cung cấp riêng.
4. Mức hiện tại theo chi nhánh (#6); GD1-10 chụp mức vào KyHoaHong lúc chốt, kỳ cũ giữ nguyên.
5. GD1-10: xem trước từ buổi hoàn thành; chốt tháng lưu danh sách buổi và mức, không mở lại kỳ.

Các tài liệu hiện dùng những giả định được đánh dấu là **đề xuất**; chúng chỉ trở thành quy tắc chính thức sau khi cả nhóm thống nhất.

## Ánh xạ ERD cho chức vụ và lịch sử (GD1-05)

- Phân cấp PT/Lễ tân/Quản lý/Nhân viên bán hàng trong [ERD A](erd-a.drawio) thể hiện loại nhân viên tại thời điểm xét. Ánh xạ phần phân loại này sang NhanVien.ChucVu cho hiện tại và LichSuCongTac.ChucVu cho lịch sử; không suy ra chức vụ quá khứ từ loại hiện tại.
- LichSuCongTac có MaLS (khóa chính), MaNV/MaCN (khóa ngoại), ChucVu, NgayBatDau/NgayKetThuc kiểu DATE; các thuộc tính khác trong ERD vẫn giữ. Khi đổi chức vụ trong cùng chi nhánh cũng phải tạo giai đoạn mới.
- NgayKetThuc >= NgayBatDau nếu có; giai đoạn một ngày hợp lệ. Hai khoảng của cùng nhân viên không được giao nhau kể cả ngày biên. Bản ghi ngày mai có thể chưa có ngày kết thúc nhưng chưa phải hiện hành hôm nay.
- Thiết kế logic A đã hoàn thiện tại thiet-ke-logic.md (GD1-10). GD1-06 đã chốt: chưa cho đổi vị trí nếu còn lịch PT chưa hủy không phù hợp sau thay đổi; xem QT-A03. Đặt lịch và đổi vị trí phải bảo đảm không bỏ sót lịch mới khi diễn ra đồng thời.

Bản ghép hiện hành: [hồ sơ GĐ1 tổng](../README.md), [ERD tổng](../erd-tong.md), [lược đồ và ràng buộc tổng](../thiet-ke-tong.md).
