# Module D — Bảng phân tích tần suất chức năng

Số liệu dùng chung lấy từ [bảng giả định](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Tần suất chức năng bổ sung chưa có trong đề được ghi là giả định. Quy ước mức độ theo tài liệu nhóm: **Cao** từ 1.000 lần/ngày; **Trung bình** 100–999 lần/ngày; **Thấp** dưới 100 lần/ngày sau quy đổi. Mức độ chỉ phản ánh số lần, không phản ánh chi phí mỗi lần.

## Chức năng của Module D

| Mã | Chức năng | Mô tả | Tần suất toàn hệ thống | Mức độ | Căn cứ/nhận xét |
|---|---|---|---:|---|---|
| D-F01 | Tra cứu danh mục sản phẩm | Xem tên, loại, giá và trạng thái sản phẩm khi bán hàng | ~750 lần/ngày | Trung bình | Tạm lấy một lượt tra cứu cho mỗi hóa đơn có bán sản phẩm; giả định G9 |
| D-F02 | Quản lý sản phẩm | Thêm/sửa thông tin, giá hoặc ngừng bán sản phẩm | ~5 lần/tháng | Thấp | Giả định; danh mục khoảng 150 mã, thay đổi ít |
| D-F03 | Xem tồn kho | Tra số lượng theo sản phẩm và chi nhánh | ~100 lần/ngày | Trung bình | Giả định, khoảng 8 lượt/chi nhánh/ngày |
| D-F04 | Nhập/điều chỉnh tồn kho | Cộng/trừ tồn kho khi nhập hàng hoặc kiểm kê | ~24 lần/ngày | Thấp | Giả định 2 lần/chi nhánh/ngày; không tính trừ kho bán hàng |
| D-F05 | Lập hóa đơn có sản phẩm | Tạo hóa đơn bán lẻ, ghi dòng sản phẩm và trừ tồn | ~750 hóa đơn/ngày | Trung bình | 25.000 check-in/ngày × 3% lượt đến mua sản phẩm |
| D-F06 | Lập hóa đơn có gói dịch vụ | Ghi nhận thanh toán gói, dùng mã/giá gói của B và gọi cập nhật điểm/hạng | ~690 hóa đơn/ngày | Trung bình | G11: 390 Gym/Yoga (gồm một tháng/vé lượt) + 100 PT + 200 Spa; giả định một đơn vị gói/vé trên mỗi hóa đơn có gói |
| D-F07 | Lập hóa đơn tổng | Thanh toán hóa đơn có thể chứa gói, sản phẩm hoặc cả hai | ~1.400 lần/ngày (~117/chi nhánh/ngày) | **Cao** | 690 có gói + 750 có sản phẩm − 40 có cả hai = 1.400 hóa đơn riêng biệt; khoảng 42.000/tháng |
| D-F08 | Tra cứu/đối chiếu hóa đơn | Tìm hóa đơn và dòng theo mã, hội viên, chi nhánh, nhân viên hoặc thời gian | ~300 lần/ngày | Trung bình | Giả định, trung bình 25 lượt/chi nhánh/ngày |
| D-F09 | Báo cáo doanh thu | Tổng hợp TongTien sau giảm; doanh thu theo gói phân bổ giảm trên toàn hóa đơn trước khi lọc/nhóm | 12 lần/ngày (1 lần/chi nhánh) + 1 lần/tuần toàn hệ thống | Thấp | Theo mẫu ở phân công; ít lần nhưng đọc lịch sử lớn |
| D-F10 | Báo cáo sản phẩm bán chạy/tồn kho | Xếp hạng mặt hàng, xem tồn kho theo chi nhánh | 12 lần/ngày (1 lần/chi nhánh) | Thấp | Giả định theo lịch quản lý cuối ngày |
| D-F11 | Báo cáo tỷ lệ lấp đầy | Tính số người đang tập theo thời điểm/khung giờ chia sức chứa | ~12 lần/ngày theo chi nhánh | Thấp | Đọc CheckIn của C theo ảnh chụp đầu khung giờ; ghi rõ lượt tự đóng và trạng thái tạm tính theo #15, không chặn cổng |

## Quy mô dữ liệu liên quan

- Khoảng 1.400 hóa đơn/ngày, 42.000/tháng; 500.000 hóa đơn cho một năm trong bộ dữ liệu demo.
- Khoảng 150 mã sản phẩm và 1.800 dòng tồn kho (150 × 12 chi nhánh).
- `ChiTietHoaDon` dự kiến khoảng 750.000 dòng/năm theo bảng giả định chung; số dòng trung bình trên mỗi hóa đơn xấp xỉ 1,5–2.
- Check-in nguồn cho báo cáo khoảng 750.000 dòng/tháng toàn hệ thống; bảng này do C quản lý.

## Nhận xét

1. `HoaDon` ghi khoảng 1.400 lần/ngày nhưng cũng là bảng nguồn cho doanh thu, điểm/hạng và đối chiếu, nên cả đường ghi hóa đơn lẫn truy vấn theo ngày/chi nhánh đều quan trọng.
2. `ChiTietHoaDon` là nơi lưu đơn giá lịch sử và phân biệt hàng hóa với gói dịch vụ. Bán chạy theo số lượng; doanh thu theo loại dùng tiền dòng sau phân bổ giảm giá của toàn hóa đơn (GD1-08). Tính phần dư cần đọc các dòng cùng MaHD; đo chi phí ở GĐ2, không cộng thành giao dịch mới.
3. `TonKho` chỉ có tối đa khoảng 1.800 cặp sản phẩm–chi nhánh nhưng cập nhật phải nhất quán với thanh toán để không âm kho.
4. Báo cáo doanh thu và lấp đầy chạy ít hơn giao dịch bán hàng nhưng có thể đọc nhiều tháng dữ liệu; ở giai đoạn 2 cần đo riêng chi phí và cân nhắc tổng hợp/partition theo mẫu truy vấn thực tế.
5. Số 750 sản phẩm/ngày là số hóa đơn có sản phẩm theo giả định tỷ lệ 3% lượt đến, không phải số lượng món bán ra.


**GD1-16:** đã bổ sung cơ cấu thẻ một tháng/vé lượt trong G11, không cộng trùng vé mua lại hoặc hóa đơn hỗn hợp. Đây là giả định quy mô; nhiều đơn vị gói trên cùng hóa đơn được nghiệp vụ cho phép và cần ước lượng lại khi thay cơ cấu. Số dòng CheckIn khác tổng thao tác quét vào/ra.

**GD1-15:** áp dụng [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung-gd1-15); nhãn theo lượt nghiệp vụ/ngày toàn hệ thống, độ nặng ghi riêng. Một lượt là đơn vị của chức năng ở từng hàng, không mặc định là một câu SQL hay một lần chạy tác vụ. Không cộng các chức năng bao hàm nhau hoặc lượt xử lý theo lô thành tổng giao dịch.
