# Bảng phân tích tần suất — Module A

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung); chỗ bảng chung không có thì ghi **tự đặt**. Quy trình tương ứng ở [quy-trinh.md](quy-trinh.md).

**Mức độ** (toàn hệ thống, theo [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung)): Cao ≥ 1.000 lần/ngày · Trung bình 100–999 lần/ngày · Thấp < 100 lần/ngày. Mức độ phản ánh số lần, không phản ánh độ nặng của mỗi lần.

## Chức năng

| Mã | Chức năng | Mô tả | Loại | Tần suất | Mức độ | Căn cứ / nhận xét |
|---|---|---|---|---|---|---|
| A-F01 | Xem thông tin chi nhánh | Xem địa chỉ, giờ mở/đóng cửa, sức chứa (QT-A01) | Đọc | ~100 lần/ngày | Trung bình | Tự đặt. Bảng chỉ 12 dòng, nhiều module dùng chung |
| A-F02 | Cập nhật chi nhánh | Sửa thông tin chi nhánh (QT-A01) | Ghi | ~2 lần/tháng | Thấp | Tự đặt. Không xóa chi nhánh |
| A-F03 | Tiếp nhận nhân viên | Tạo hồ sơ và giai đoạn công tác đầu tiên trong một giao dịch (QT-A02) | Ghi | ~4 lần/tháng | Thấp | Tự đặt, ~0,5%/tháng của ~700 nhân viên |
| A-F04 | Tra cứu nhân viên hiện tại | Tìm theo mã, tên, chức vụ, chi nhánh hiện tại (QT-A04) | Đọc | ~200 lần/ngày | Trung bình | Tự đặt. Đọc `ChucVu`, `MaCNHienTai` lưu sẵn, không phải dò lịch sử |
| A-F05 | Cập nhật thông tin và lương | Sửa SĐT, họ tên, lương cơ bản (QT-A04) | Ghi | ~20 lần/ngày | Thấp | Tự đặt |
| A-F06 | Điều chuyển nhân viên | Kiểm tra lịch PT vướng, đóng giai đoạn cũ hết hôm nay, mở giai đoạn mới từ ngày mai (QT-A03) | Đọc + ghi | ~10 lần/tháng | Thấp | Bảng chung (~1,5%/tháng). Ghi nguyên tử, không chồng khoảng |
| A-F07 | Cho nhân viên nghỉ việc | Kiểm tra lịch PT vướng, đóng giai đoạn hiện tại hết hôm nay (QT-A09) | Đọc + ghi | ~4 lần/tháng | Thấp | Tự đặt |
| A-F08 | Đồng bộ chi nhánh hiện tại | Job đầu ngày cập nhật `MaCNHienTai` cho ~700 nhân viên (QT-A05) | Đọc + ghi theo lô | 1 lần/ngày | Thấp | 00:00 Asia/Saigon; chạy lại cho cùng kết quả |
| A-F09 | Tra cứu lịch sử công tác | Xem toàn bộ lịch sử hoặc chi nhánh của nhân viên tại một ngày (QT-A06) | Đọc | ~100 lần/ngày | Trung bình | Tự đặt. Điều kiện chính: `MaNV` + khoảng ngày |
| A-F10 | Cập nhật mức hoa hồng | Đóng khoảng mức cũ hết hôm nay, mở khoảng mới từ ngày mai (QT-A07) | Ghi | ~1 lần/tháng | Thấp | Tự đặt |
| A-F11 | Tính và xem hoa hồng PT theo tháng | Với mỗi buổi Đã hoàn thành: tìm chi nhánh của PT và mức hoa hồng tại `NgayTap`, nhóm theo PT và chi nhánh (QT-A08) | Đọc nặng | ~2 lần/tháng/chi nhánh ≈ 24 lần/tháng | Thấp | Ít lần nhưng mỗi lần đọc ~81.000 buổi/tháng toàn hệ thống (2.700 × 30) và đối chiếu từng buổi với `LICH_SU_CONG_TAC`, `HOA_HONG_CHI_NHANH`. Đây là truy vấn lõi của vấn đề 5 |

## Nhận xét

1. `CHI_NHANH` và `HOA_HONG_CHI_NHANH` rất nhỏ (12 và vài chục dòng) nhưng được tham chiếu rộng.
2. `LICH_SU_CONG_TAC` ghi rất ít (vài chục lần/tháng) nhưng phải bảo đảm tuyệt đối không chồng khoảng, vì mọi phép tính hoa hồng dựa vào nó.
3. Kiểm tra lịch PT vướng nằm bên trong A-F06 và A-F07; đọc các lịch Đã đặt của một nhân viên từ ngày mai trở đi, không tính thành chức năng riêng.
4. A-F11 là chức năng nặng nhất của A: tần suất thấp nhưng mỗi lần phải nối lịch PT với hai bảng lịch sử theo khoảng ngày. Mẫu truy vấn chính là `MaNV` + ngày trong khoảng `[NgayBatDau, NgayKetThuc]` và `MaCN` + ngày trong khoảng, là cơ sở chọn chỉ mục ở GĐ2.
5. Không lưu kết quả chốt kỳ, nên mỗi lần xem là một lần tính lại. Nếu đo ở GĐ2 thấy chậm, có thể cân nhắc bảng tổng hợp; GĐ1 chưa chốt.
