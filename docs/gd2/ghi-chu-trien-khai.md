# Ghi chú hướng triển khai GĐ2

Hướng xem xét ban đầu, **chưa cài đặt hay đo đạc**. Căn cứ là thiết kế GĐ1 đã chốt: [quyết định](../quyet-dinh.md), [thiết kế logic](../gd1/thiet-ke-tong.md). Mọi lựa chọn chỉ mục/phân vùng phải được chứng minh bằng số đo có và không có giải pháp.

| Vấn đề của đề | Hướng giải từ thiết kế GĐ1 | Việc cần làm ở GĐ2 |
|---|---|---|
| 1 — Check-in < 50 ms | Quét theo mã gói: đọc một dòng `GOI_HOI_VIEN` theo khóa chính, ghi một dòng `CHECK_IN`; không kiểm tra phiên mở khi vào | Chỉ mục cho quét ra (tìm phiên mở theo các gói của hội viên); phân vùng `CHECK_IN` theo tháng; đo thời gian truy vấn và mức đồng thời |
| 2 — Trùng lịch PT | Ràng buộc duy nhất `(MaNV, NgayTap, ThoiGianBatDau)` trên lịch chưa hủy (R16) | Chỉ mục duy nhất có điều kiện `TrangThai <> 1`; demo hai giao dịch đặt cùng lúc, một bên bị từ chối |
| 3 — Báo cáo chậm | Lưu sẵn `HOA_DON.MaCN`, `TongTien`; báo cáo lấp đầy đọc `CHECK_IN` | Phân vùng `HOA_DON`, `CHECK_IN` theo tháng; cân nhắc bảng tổng hợp doanh thu ngày/chi nhánh và lấp đầy theo giờ |
| 4 — Tính hạng quá tải | Lưu sẵn `TongChiTieu`, `DiemTichLuy`, `Hang`; thanh toán chỉ sửa một dòng hội viên; reset ngày 1/1 | Thủ tục cộng điểm gọi trong giao dịch thanh toán; so sánh với cách cộng lại hóa đơn 12 tháng |
| 5 — Hoa hồng sai | `LICH_SU_CONG_TAC`, `HOA_HONG_CHI_NHANH` theo khoảng ngày; lịch tháng cũ không sửa | Chỉ mục `(MaNV, NgayBatDau)`, `(MaCN, NgayBatDau)`; thủ tục tính hoa hồng tháng cho PT điều chuyển giữa tháng |

Các job: đồng bộ nhân viên đầu ngày (QT-A05), đóng phiên cuối ngày (QT-C3), hoàn thành lịch và trừ buổi khi tới giờ (QT-C7), reset hạng đầu năm (QT-B5). Chạy lại phải cho cùng kết quả.
