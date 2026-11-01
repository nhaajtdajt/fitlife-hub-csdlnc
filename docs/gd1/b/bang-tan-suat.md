# Module B — Bảng phân tích tần suất chức năng

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Chỗ bảng chung không có thì ghi **tự đặt** kèm cách tính. Quy trình tương ứng ở [quy-trinh.md](quy-trinh.md).

**Mức độ:** Cao ≥ 1.000 lần/ngày · Trung bình 100–999 lần/ngày · Thấp < 100 lần/ngày. Ngưỡng này suy ra từ các ví dụ trong [phan-cong.md](../../phan-cong.md) (200/ngày là trung bình, 1.000/ngày là cao, 12/ngày là thấp).

## Chức năng của module B

| Mã | Chức năng | Mô tả | Tần suất (toàn hệ thống) | Mức độ | Căn cứ |
|---|---|---|---|---|---|
| CN-B01 | Tra cứu hội viên | Tìm theo `MaHV`, SĐT hoặc CCCD; xem gói đang có, hạng, điểm | ~2.000 lần/ngày (~170/chi nhánh) | **Cao** | 1.400 hóa đơn + ~600 lượt hỏi tại quầy (tự đặt, ~2,4% của 25.000 lượt đến) |
| CN-B02 | Đăng ký hội viên | Tạo hồ sơ, hạng Silver, điểm 0 | ~50 lần/ngày (~4/chi nhánh) | Thấp | Tự đặt: 70.000 hội viên × 25% mới mỗi năm ÷ 365 |
| CN-B03 | Cập nhật hội viên | Sửa SĐT, email... | ~20 lần/ngày | Thấp | Tự đặt |
| CN-B04 | Xem danh mục gói | Xem tên, loại, thời hạn, giá của ~30 gói khi tư vấn hoặc lập hóa đơn | ~1.000 lần/ngày | **Cao** | ~690 lượt mua + ~300 lượt tư vấn không mua (tự đặt) |
| CN-B05 | Mua/gia hạn gói tập | Tạo `GoiHoiVien` Gym/Yoga | ~390 lần/ngày | Trung bình | Bảng chung |
| CN-B06 | Mua gói PT | Kiểm tra đang có gói Gym/Yoga còn hạn hoặc mua kèm trên cùng hóa đơn, rồi tạo `GoiHoiVien` có số buổi | ~100 lần/ngày | Trung bình | Bảng chung |
| CN-B07 | Mua vé Spa lẻ | Tạo `GoiHoiVien` dùng một lần | ~200 lần/ngày | Trung bình | Bảng chung |
| CN-B08 | Trừ buổi PT | Giảm `SoBuoiConLai` khi quét cổng vào ngày có lịch, hủy muộn hoặc không đến | ~2.700 lần/ngày | **Cao** | 2.600 buổi diễn ra + ~100 hủy muộn/không đến (tự đặt, ~1/3 của 290 lịch hủy) |
| CN-B09 | Dùng vé Spa | `SoBuoiConLai` từ 1 về 0, khi quét cổng (khách chỉ có vé Spa) hoặc tại quầy (khách có gói Gym/Yoga) | ~200 lần/ngày | Trung bình | Coi như mọi vé mua đều được dùng |
| CN-B10 | Cập nhật điểm, xét hạng khi thanh toán | Hiện trạng: cộng lại `TongTien` 12 tháng cho mỗi hóa đơn. **Nặng**, là vấn đề 4 của đề | ~1.400 lần/ngày (~117/chi nhánh) | **Cao** | Bảng chung |
| CN-B11 | Xem lịch sử điểm | Liệt kê các lần cộng điểm của một hội viên | ~50 lần/ngày | Thấp | Tự đặt |
| CN-B12 | Xét hạ hạng hằng đêm | Tính lại chi tiêu của hội viên có hóa đơn vừa quá 12 tháng. **Nặng** | 1 lần/ngày | Thấp | Quyết định #16 |
| CN-B13 | Quản lý danh mục gói | Thêm gói, sửa giá, ngừng bán | ~5 lần/tháng | Thấp | Tự đặt |
| CN-B14 | Danh sách sắp hết hạn, lâu ngày chưa quay lại | Báo cáo cấp chi nhánh, đọc thêm `CheckIn`. **Nặng** | 12 lần/ngày (1/chi nhánh) | Thấp | Tự đặt |
| CN-B15 | Thống kê phân bổ hạng | Tỷ lệ Silver / Gold / Platinum toàn hệ thống | ~1 lần/tuần | Thấp | Tự đặt |

`GoiHoiVien` có khoảng **690 dòng mới mỗi ngày** (CN-B05 + CN-B06 + CN-B07). Ví dụ ~200 lần/ngày trong `phan-cong.md` chỉ là minh họa, nên tài liệu này dùng số của bảng chung.

## Chức năng của module khác dùng bảng của B

Các chức năng này do C giữ số chính, không cộng vào bảng trên. B cần biết để thiết kế `GoiHoiVien`.

| Chức năng | Thao tác trên bảng của B | Tần suất | Mức độ |
|---|---|---|---|
| Check-in: kiểm tra có gói hợp lệ, trừ buổi nếu có lịch PT hôm nay, trừ vé nếu chỉ có vé Spa (C) | Đọc `GoiHoiVien` theo `MaHV`, `SoBuoiConLai`, `NgayHetHan`; ghi `SoBuoiConLai` khi phải trừ | ~25.000 lần/ngày; 10.000 lần trong 17h–20h (~55 lần/phút, đỉnh vài lần/giây) | **Cao** |
| Đặt lịch PT: kiểm tra gói PT còn buổi, gói Gym/Yoga còn hạn tại ngày hẹn (C) | Đọc `GoiHoiVien` theo `MaHV`, `SoBuoiConLai`, `NgayHetHan`, kèm `GoiDichVu.LoaiGoi` | ~2.900 lần/ngày | **Cao** |

## Nhận xét

- **`GoiHoiVien` đọc nhiều hơn ghi rất nhiều.** Khoảng 28.000 lần đọc so với khoảng 3.600 lần ghi mỗi ngày (690 thêm mới + 2.700 trừ buổi PT + 200 trừ vé Spa), và lượt đọc dồn vào giờ cao điểm. Đây là ứng viên tốt để đánh index ở giai đoạn 2. Check-in còn đọc lịch PT hôm nay và trừ buổi ngay khi quét cổng, nên phần lớn ~2.700 lần trừ buổi có thể rơi vào giờ cao điểm; bước 2.3 phải đo xem vẫn đạt < 50ms không. Đặt lịch PT cần thêm loại gói từ `GoiDichVu`.
- **CN-B10 không quá dày nhưng mỗi lần rất đắt.** Khoảng 1.400 lần/ngày không phải tần suất lớn nhất, nhưng mỗi lần cộng lại trên bảng `HoaDon` khoảng 500.000 dòng, ngay trong lúc nhân viên chờ in hóa đơn. Cái cần sửa là chi phí mỗi lần, không phải số lần.
- **CN-B12 tần suất thấp nhưng bắt buộc phải có.** Chi tiêu 12 tháng tự giảm theo thời gian, nên chỉ xét khi có hóa đơn mới thì không bao giờ hạ hạng được người đã ngừng mua. Job chạy đêm nên không ảnh hưởng giờ cao điểm.
