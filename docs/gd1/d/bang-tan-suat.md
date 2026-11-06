# Module D — Bảng phân tích tần suất chức năng

Số liệu dùng chung lấy từ [bảng giả định](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Tần suất chức năng bổ sung chưa có trong đề được ghi là giả định. Quy ước mức độ theo tài liệu nhóm: **Cao** từ 1.000 lần/ngày; **Trung bình** 100–999 lần/ngày; **Thấp** dưới 100 lần/ngày hoặc theo kỳ. Mức độ chỉ phản ánh số lần, không phản ánh chi phí mỗi lần.

## Chức năng của Module D

| Mã | Chức năng | Mô tả | Tần suất toàn hệ thống | Mức độ | Căn cứ/nhận xét |
|---|---|---|---:|---|---|
| D-F01 | Tra cứu danh mục sản phẩm | Xem tên, loại, giá và trạng thái sản phẩm tại quầy | ~750 lần/ngày | Trung bình | Tạm lấy một lượt tra cứu cho mỗi hóa đơn có bán sản phẩm; giả định G9 |
| D-F02 | Quản lý sản phẩm | Thêm/sửa thông tin, giá hoặc ngừng bán sản phẩm | ~5 lần/tháng | Thấp | Giả định; danh mục khoảng 150 mã, thay đổi ít |
| D-F03 | Xem tồn kho | Tra số lượng theo sản phẩm và chi nhánh | ~100 lần/ngày | Trung bình | Giả định, khoảng 8 lượt/chi nhánh/ngày |
| D-F04 | Nhập/điều chỉnh tồn kho | Cộng/trừ tồn kho khi nhập hàng hoặc kiểm kê | ~24 lần/ngày | Thấp | Giả định 2 lần/chi nhánh/ngày; không tính trừ kho bán hàng |
| D-F05 | Lập hóa đơn có sản phẩm | Tạo hóa đơn bán lẻ, ghi dòng sản phẩm và trừ tồn | ~750 hóa đơn/ngày | Trung bình | 25.000 check-in/ngày × 3% lượt đến mua sản phẩm |
| D-F06 | Lập hóa đơn có gói dịch vụ | Ghi nhận thanh toán gói, dùng mã/giá gói của B và gọi cập nhật điểm/hạng | ~690 hóa đơn/ngày | Trung bình | 390 gói tập + 100 PT + 200 vé Spa theo giả định chung |
| D-F07 | Lập hóa đơn tổng | Thanh toán hóa đơn có thể chứa gói, sản phẩm hoặc cả hai | ~1.400 lần/ngày (~117/chi nhánh/ngày) | **Cao** | Tổng hóa đơn/ngày theo giả định chung; khoảng 42.000/tháng |
| D-F08 | Tra cứu/đối chiếu hóa đơn | Tìm hóa đơn và dòng theo mã, hội viên, chi nhánh, nhân viên hoặc thời gian | ~300 lần/ngày | Trung bình | Giả định, trung bình 25 lượt/chi nhánh/ngày |
| D-F09 | Báo cáo doanh thu | Tổng hợp doanh thu theo ngày/tháng/quý/năm và so sánh chi nhánh | 12 lần/ngày (1 lần/chi nhánh) + 1 lần/tuần toàn hệ thống | Thấp | Theo mẫu ở phân công; ít lần nhưng đọc lịch sử lớn |
| D-F10 | Báo cáo sản phẩm bán chạy/tồn kho | Xếp hạng mặt hàng, xem tồn kho theo chi nhánh | 12 lần/ngày (1 lần/chi nhánh) | Thấp | Giả định theo lịch quản lý cuối ngày |
| D-F11 | Báo cáo tỷ lệ lấp đầy | Tính số người đang tập theo thời điểm/khung giờ chia sức chứa | ~12 lần/ngày theo chi nhánh | Thấp | Đọc `CheckIn` của C; cần thống nhất xử lý check-in quên quét ra |

## Quy mô dữ liệu liên quan

- Khoảng 1.400 hóa đơn/ngày, 42.000/tháng; 500.000 hóa đơn cho một năm trong bộ dữ liệu demo.
- Khoảng 150 mã sản phẩm và 1.800 dòng tồn kho (150 × 12 chi nhánh).
- `ChiTietHoaDon` dự kiến khoảng 750.000 dòng/năm theo bảng giả định chung; số dòng trung bình trên mỗi hóa đơn xấp xỉ 1,5–2.
- Check-in nguồn cho báo cáo khoảng 750.000 dòng/tháng toàn hệ thống; bảng này do C quản lý.

## Nhận xét

1. `HoaDon` ghi khoảng 1.400 lần/ngày nhưng cũng là bảng nguồn cho doanh thu, điểm/hạng và đối chiếu, nên cả đường ghi hóa đơn lẫn truy vấn theo ngày/chi nhánh đều quan trọng.
2. `ChiTietHoaDon` là nơi lưu đơn giá lịch sử và phân biệt hàng hóa với gói dịch vụ. Các báo cáo bán chạy và doanh thu theo loại dựa trên bảng này.
3. `TonKho` chỉ có tối đa khoảng 1.800 cặp sản phẩm–chi nhánh nhưng cập nhật phải nhất quán với thanh toán để không âm kho.
4. Báo cáo doanh thu và lấp đầy chạy ít hơn giao dịch quầy nhưng có thể đọc nhiều tháng dữ liệu; ở giai đoạn 2 cần đo riêng chi phí và cân nhắc tổng hợp/partition theo mẫu truy vấn thực tế.
5. Số 750 sản phẩm/ngày là số hóa đơn có sản phẩm theo giả định tỷ lệ 3% lượt đến, không phải số lượng món bán ra.

