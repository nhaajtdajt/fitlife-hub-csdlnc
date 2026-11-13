# Bảng phân tích tần suất — Module D

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung); chỗ bảng chung không có thì ghi **tự đặt**. Quy trình tương ứng ở [quy-trinh.md](quy-trinh.md).

**Mức độ** (toàn hệ thống, theo [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung)): Cao ≥ 1.000 lần/ngày · Trung bình 100–999 lần/ngày · Thấp < 100 lần/ngày.

## Chức năng

| Mã | Chức năng | Mô tả | Loại | Tần suất | Mức độ | Căn cứ / nhận xét |
|---|---|---|---|---|---|---|
| D-F01 | Tra cứu danh mục sản phẩm | Xem tên, giá, trạng thái sản phẩm khi bán hàng (QT-D1) | Đọc | ~750 lần/ngày | Trung bình | Một lượt cho mỗi hóa đơn có sản phẩm (G9) |
| D-F02 | Quản lý danh mục sản phẩm | Thêm, sửa giá, ngừng kinh doanh (QT-D1) | Ghi | ~5 lần/tháng | Thấp | Tự đặt; ~150 sản phẩm |
| D-F03 | Mở/ngừng bán sản phẩm tại chi nhánh | Tạo hoặc xóa dòng `TON_KHO` (QT-D2) | Ghi | ~10 lần/tháng | Thấp | Tự đặt |
| D-F04 | Xem tồn kho | Số lượng theo sản phẩm và chi nhánh | Đọc | ~100 lần/ngày | Trung bình | Tự đặt, ~8 lượt/chi nhánh/ngày |
| D-F05 | Nhập hàng, điều chỉnh tồn | Cộng/trừ `SoLuongTon`, không âm (QT-D3) | Ghi | ~24 lần/ngày | Thấp | Tự đặt, 2 lần/chi nhánh/ngày; không gồm trừ kho khi bán |
| D-F06 | Lập và thanh toán hóa đơn | Ghi hóa đơn, các dòng, trừ kho; gọi B tạo gói và cộng điểm trong cùng giao dịch (QT-D4) | Đọc + ghi | ~1.400 hóa đơn/ngày (~42.000/tháng) | **Cao** | 690 hóa đơn có gói + 750 có sản phẩm − 40 có cả hai |
| D-F07 | Tra cứu, đối chiếu hóa đơn | Lọc theo mã, hội viên, ngày, chi nhánh, nhân viên (QT-D5) | Đọc | ~300 lần/ngày | Trung bình | Tự đặt, ~25 lượt/chi nhánh/ngày |
| D-F08 | Báo cáo doanh thu | Cộng `TongTien` theo ngày/tháng/quý/năm, chi nhánh; doanh thu theo hàng mục trước giảm giá (QT-D6) | Đọc nặng | 12 lần/ngày + ~1 lần/tuần toàn hệ thống | Thấp | Tự đặt. Ít lần nhưng đọc nhiều tháng hóa đơn; đề nêu là chậm > 1 phút (vấn đề 3) |
| D-F09 | Báo cáo bán chạy, tồn kho | Xếp hạng hàng mục theo số lượng; xem tồn theo chi nhánh (QT-D6) | Đọc | 12 lần/ngày | Thấp | Tự đặt, 1/chi nhánh/ngày |
| D-F10 | Báo cáo tỷ lệ lấp đầy | Số người có mặt tại từng mốc giờ chia sức chứa (QT-D7) | Đọc nặng | ~12 lần/ngày | Thấp | Tự đặt. Đọc `CHECK_IN` của C (~750.000 dòng/tháng); đề nêu là chậm > 1 phút (vấn đề 3) |

## Quy mô dữ liệu

- `HOA_DON`: ~1.400 dòng/ngày, ~42.000/tháng; bộ dữ liệu GĐ2 ~500.000 dòng (một năm).
- `CHI_TIET_HOA_DON`: ~1,5 dòng/hóa đơn, ~750.000 dòng/năm.
- `HANG_MUC_CUNG_CAP`: ~180 dòng (~30 gói + ~150 sản phẩm); `TON_KHO`: tối đa ~1.800 dòng.

## Nhận xét

1. `HOA_DON` ghi ~1.400 lần/ngày và là nguồn cho doanh thu, lịch sử điểm, đối chiếu; cả đường ghi lẫn truy vấn theo ngày và chi nhánh đều quan trọng.
2. D-F06 là giao dịch dài nhất hệ thống (hóa đơn + dòng + kho + gói + điểm). Phần cộng điểm đã được làm nhẹ ở B-F08 để không kéo dài thời gian chờ in hóa đơn.
3. `TON_KHO` nhỏ nhưng phải trừ nhất quán khi hai hóa đơn cùng bán món cuối cùng, để không âm kho.
4. D-F08 và D-F10 chạy ít nhưng đọc dữ liệu lớn; đây là hai báo cáo vấn đề 3 yêu cầu tối ưu (phân vùng, bảng tổng hợp) ở GĐ2.
