# Bảng phân tích tần suất — Module B

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung); chỗ bảng chung không có thì ghi **tự đặt**. Quy trình tương ứng ở [quy-trinh.md](quy-trinh.md).

**Mức độ** (toàn hệ thống, theo [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung)): Cao ≥ 1.000 lần/ngày · Trung bình 100–999 lần/ngày · Thấp < 100 lần/ngày.

## Chức năng

| Mã | Chức năng | Mô tả | Loại | Tần suất | Mức độ | Căn cứ / nhận xét |
|---|---|---|---|---|---|---|
| B-F01 | Tra cứu hội viên | Tìm theo mã, SĐT, CCCD; xem hạng, điểm, gói còn hiệu lực (QT-B8) | Đọc | ~2.000 lần/ngày | **Cao** | 1.400 lần khi lập hóa đơn + ~600 lượt hỗ trợ (tự đặt) |
| B-F02 | Đăng ký hội viên | Tạo hồ sơ với chi nhánh đăng ký, 0 chi tiêu, 0 điểm, Silver (QT-B1) | Ghi | ~50 lần/ngày | Thấp | Tự đặt: 70.000 × 25% mới mỗi năm ÷ 365 |
| B-F03 | Cập nhật hội viên | Sửa thông tin liên hệ (QT-B1) | Ghi | ~20 lần/ngày | Thấp | Tự đặt |
| B-F04 | Xem danh mục gói | Xem tên, loại, giá, thời hạn ~30 gói khi tư vấn hoặc lập hóa đơn (QT-B2) | Đọc | ~1.000 lần/ngày | **Cao** | ~690 lượt mua + ~300 lượt tư vấn không mua (tự đặt) |
| B-F05 | Mua gói Gym/Yoga, vé lượt | Tạo gói bắt đầu ngày mua; vé hạn 7 ngày, 1 lượt (QT-B2) | Ghi | ~390 gói/ngày | Trung bình | G11: 360 gói theo tháng + 30 vé lượt |
| B-F06 | Mua gói PT | Tạo gói PT có số buổi; không kiểm tra thẻ Gym (QT-B2) | Ghi | ~100 gói/ngày | Trung bình | Bảng chung |
| B-F07 | Mua vé Spa | Tạo gói Spa hạn 7 ngày, 1 lượt (QT-B2) | Ghi | ~200 gói/ngày | Trung bình | Bảng chung |
| B-F08 | Cộng điểm, cập nhật hạng khi thanh toán | Cộng `TongChiTieu`, `DiemTichLuy`, cập nhật `Hang` trên một dòng hội viên (QT-B4) | Đọc + ghi | ~1.400 lần/ngày | **Cao** | Mỗi hóa đơn một lần. Đây là vấn đề 4 của đề: hiện trạng mỗi lần cộng lại ~500.000 hóa đơn/năm; thiết kế mới chỉ sửa một dòng |
| B-F09 | Reset hạng đầu năm | Tính lại `TongChiTieu` từ hóa đơn năm mới, hạng về Silver cho ~120.000 hội viên (QT-B5) | Đọc + ghi theo lô | 1 lần/năm | Thấp | Chạy 00:00 ngày 1/1, ngoài giờ cao điểm |
| B-F10 | Xem lịch sử điểm | Liệt kê hóa đơn của hội viên kèm điểm từng hóa đơn (QT-B8) | Đọc | ~50 lần/ngày | Thấp | Tự đặt. Đọc `HOA_DON` theo `MaHV` |
| B-F11 | Quản lý danh mục gói | Thêm gói, sửa giá, ngừng kinh doanh (QT-B6) | Ghi | ~5 lần/tháng | Thấp | Tự đặt |
| B-F12 | Danh sách sắp hết hạn, lâu ngày chưa quay lại | Theo chi nhánh đăng ký; đọc thêm `CHECK_IN` 30 ngày (QT-B7) | Đọc nặng | 12 lần/ngày (1/chi nhánh) | Thấp | Tự đặt. Ít lần nhưng đọc nhiều dòng check-in |
| B-F13 | Thống kê phân bổ hạng | Tỷ lệ Silver / Gold / Platinum toàn hệ thống (QT-B7) | Đọc | ~1 lần/tuần | Thấp | Tự đặt. Đếm theo cột `Hang` lưu sẵn |

Tổng gói mới ~690 dòng `GOI_HOI_VIEN`/ngày (B-F05 + B-F06 + B-F07), được tạo bên trong giao dịch thanh toán của D.

## Chức năng của module khác dùng bảng của B

Các chức năng này do C giữ số chính, không cộng vào bảng trên.

| Chức năng (C) | Thao tác trên bảng của B | Tần suất | Mức độ |
|---|---|---|---|
| Quét vào (C-F01) | Đọc một `GOI_HOI_VIEN` theo `MaGHV` và `LoaiHangMuc`; giảm `SoBuoiConLai` khi dùng vé | ~25.000 lượt/ngày, trong đó ~230 lượt trừ vé (G12) | **Cao** |
| Đặt lịch PT (C-F06) | Đọc gói PT của hội viên, đếm lịch Đã đặt theo `MaGHV` để tính buổi trống | ~2.900 lần/ngày | **Cao** |
| Job tới giờ tập (C-F08) | Giảm `SoBuoiConLai` của gói PT đi 1 | ~2.700 lịch/ngày | **Cao** |

## Nhận xét

- **`GOI_HOI_VIEN` đọc nhiều hơn ghi rất nhiều:** ~27.900 lượt đọc/ngày (quét vào + đặt PT) so với ~3.600 lượt ghi (690 gói mới + 2.700 trừ PT + 230 trừ vé). Quét vào đọc theo khóa chính `MaGHV` nên rất nhanh; đặt PT đọc theo `MaHV` + loại gói, là ứng viên chỉ mục ở GĐ2.
- **B-F08 không phải chức năng dày nhất nhưng nằm ngay lúc khách chờ in hóa đơn.** Điều cần sửa là chi phí mỗi lần (từ quét hóa đơn 12 tháng xuống sửa một dòng), không phải số lần.
- **B-F09 chỉ một lần/năm** nhưng chạm toàn bộ hội viên; chạy lúc nửa đêm và tính lại được nếu lỗi.
