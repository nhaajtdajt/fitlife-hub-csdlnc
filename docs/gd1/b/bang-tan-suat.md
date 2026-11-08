# Module B — Bảng phân tích tần suất chức năng

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Chỗ bảng chung không có thì ghi **tự đặt** kèm cách tính. Quy trình tương ứng ở [quy-trinh.md](quy-trinh.md).

**Mức độ:** Cao ≥ 1.000 lần/ngày · Trung bình 100–999 lần/ngày · Thấp < 100 lần/ngày. Ngưỡng đã chốt chung ở GD1-15.

## Chức năng của module B

| Mã | Chức năng | Mô tả | Tần suất (toàn hệ thống) | Mức độ | Căn cứ |
|---|---|---|---|---|---|
| CN-B01 | Tra cứu hội viên | Tìm theo `MaHV`, SĐT hoặc CCCD; xem gói đang có, hạng, điểm | ~2.000 lần/ngày (~170/chi nhánh) | **Cao** | 1.400 hóa đơn + ~600 lượt tra cứu hỗ trợ hội viên (tự đặt, ~2,4% của 25.000 lượt đến) |
| CN-B02 | Đăng ký hội viên | Tạo hồ sơ, hạng Silver, điểm 0 | ~50 lần/ngày (~4/chi nhánh) | Thấp | Tự đặt: 70.000 hội viên × 25% mới mỗi năm ÷ 365 |
| CN-B03 | Cập nhật hội viên | Sửa SĐT, email... | ~20 lần/ngày | Thấp | Tự đặt |
| CN-B04 | Xem danh mục gói | Xem tên, loại, thời hạn, giá của ~30 gói khi tư vấn hoặc lập hóa đơn | ~1.000 lần/ngày | **Cao** | ~690 lượt mua + ~300 lượt tư vấn không mua (tự đặt) |
| CN-B05 | Mua/gia hạn gói tập | Tạo gói Gym/Yoga theo hạn nối tiếp sau gói cùng loại cuối cùng (kể cả gói chờ), hoặc vé một lượt độc lập | ~390 lần/ngày | Trung bình | G11: 300 gói dài hạn + 60 gói một tháng + 30 vé lượt (gồm 10 lượt mua lại) |
| CN-B06 | Mua gói PT | Kiểm tra có thẻ dịch vụ cấp quyền vào còn hiệu lực/vé còn lượt hoặc mua kèm trên cùng hóa đơn, rồi tạo `GoiHoiVien` có số buổi | ~100 lần/ngày | Trung bình | Bảng chung |
| CN-B07 | Mua vé Spa lẻ | Tạo `GoiHoiVien` dùng một lần | ~200 lần/ngày | Trung bình | Bảng chung |
| CN-B08 | Trừ buổi PT | Tự giảm một buổi từ gói đã giữ khi đến giờ lịch chưa hủy, độc lập check-in/tham dự | ~2.700 lượt trừ lịch/ngày (1 lịch = 1 lượt) | Cao | 2.600 hoàn thành + ~100 vắng mặt; tạm tách 300 lịch không hoàn thành thành 200 hủy đúng hạn và 100 vắng mặt; đã đối chiếu cơ cấu giả định ở GD1-15, chưa phải số đo |
| CN-B09 | Dùng vé vào một lượt (Spa/Gym/Yoga) | `SoBuoiConLai` từ 1 về 0 tại cổng khi dùng vé một lượt làm quyền vào; nằm trong giao dịch CheckIn, không có giao dịch dùng Spa tại quầy | ~230 lần/ngày | Trung bình | G12: 200 vé Spa + 30 vé Gym/Yoga được dùng tại cổng; là phần con của lượt check-in C, không cộng thêm lượt vào |
| CN-B10 | Cập nhật điểm, xét hạng khi thanh toán | Hiện trạng: cộng lại `TongTien` 12 tháng cho mỗi hóa đơn. **Nặng**, là vấn đề 4 của đề | ~1.400 lần/ngày (~117/chi nhánh) | **Cao** | Bảng chung |
| CN-B11 | Xem lịch sử điểm | Liệt kê các lần cộng điểm của một hội viên | ~50 lần/ngày | Thấp | Tự đặt |
| CN-B12 | Xét hạ hạng hằng đêm | Tính lại chi tiêu của hội viên có hóa đơn vừa quá 12 tháng. **Nặng** | 1 lần/ngày | Thấp | Quyết định #16 |
| CN-B13 | Quản lý danh mục gói | Thêm gói, sửa giá, ngừng bán | ~5 lần/tháng | Thấp | Tự đặt |
| CN-B14 | Danh sách sắp hết hạn, lâu ngày chưa quay lại | Báo cáo cấp chi nhánh, đọc thêm `CheckIn`. **Nặng** | 12 lần/ngày (1/chi nhánh) | Thấp | Tự đặt |
| CN-B15 | Thống kê phân bổ hạng | Tỷ lệ Silver / Gold / Platinum toàn hệ thống | ~1 lần/tuần | Thấp | Tự đặt |

Theo giả định G11 và cơ cấu hóa đơn GD1-16, `GoiHoiVien` có khoảng **690 dòng mới mỗi ngày** (CN-B05 + CN-B06 + CN-B07). Ví dụ ~200 lần/ngày trong `phan-cong.md` chỉ là minh họa, nên tài liệu này dùng số của bảng chung.

## Chức năng của module khác dùng bảng của B

Các chức năng này do C giữ số chính, không cộng vào bảng trên. B cần biết để thiết kế `GoiHoiVien`.

| Chức năng | Thao tác trên bảng của B | Tần suất | Mức độ |
|---|---|---|---|
| Check-in: kiểm tra thẻ dịch vụ/vé vào hợp lệ, chỉ trừ vé vào nếu dùng (C) | Đọc `GoiHoiVien` theo `MaHV`, `SoBuoiConLai`, `NgayBatDau`, `NgayHetHan`; ghi `SoBuoiConLai` khi phải trừ | ~25.000 lượt vào thành công/ngày; G1 minh họa ~10.000 lượt/3 giờ (~55,56 lượt/phút trung bình); không suy ra đỉnh/đồng thời | **Cao** |
| Đặt lịch PT: kiểm tra gói PT đủ hạn tại ngày hẹn và buổi chưa bị giữ (C) | Xét lại lịch sử vị trí theo GD1-06; kiểm tra gói theo MaHV/loại/hạn, đếm lại lịch Đã đặt/chưa trừ theo MaGHV trước khi thêm lịch | ~2.900 lần/ngày | **Cao** |

## Nhận xét

- **`GoiHoiVien` đọc nhiều hơn ghi rất nhiều.** Riêng kiểm tra vào cổng và đặt PT có khoảng 27.900 lượt nghiệp vụ cần đọc/ngày (25.000 + 2.900); các nghiệp vụ chính làm đổi gói có khoảng 3.620 lượt/ngày (690 thêm mới + 2.700 trừ PT + 230 trừ vé). Đây không phải tổng số câu SQL, dòng đọc/ghi hay toàn bộ lượt đọc hệ thống. Lượt đọc dồn vào giờ cao điểm. Đặt lịch còn đọc các lịch giữ chỗ theo MaGHV; phải tính thêm tải này khi phân tích vật lý. Đây là ứng viên tốt để đánh index ở giai đoạn 2. Check-in không đọc lịch hoặc trừ PT; ~2.700 lượt trừ PT là tác vụ độc lập tại giờ hẹn. Trừ vé vào vẫn nằm trong giao dịch check-in và cần đo mục tiêu < 50ms. Đặt lịch PT cần thêm loại gói từ `GoiDichVu`.
- **CN-B10 không quá dày nhưng mỗi lần rất đắt.** Khoảng 1.400 lần/ngày không phải tần suất lớn nhất, nhưng mỗi lần cộng lại trên bảng `HoaDon` khoảng 500.000 dòng, ngay trong lúc nhân viên chờ in hóa đơn. Cái cần sửa là chi phí mỗi lần, không phải số lần.
- **CN-B12 tần suất thấp nhưng bắt buộc phải có.** Chi tiêu 12 tháng tự giảm theo thời gian, nên chỉ xét khi có hóa đơn mới thì không bao giờ hạ hạng được người đã ngừng mua. Job chạy đêm nên không ảnh hưởng giờ cao điểm.

**GD1-16:** đã tính thẻ một tháng/vé lượt trong G11–G12; lượt mua lại đã nằm trong tổng mua và tổng vào cổng. Các số là giả định làm việc, không phải số đo; xem [quy ước tải chung](../../quyet-dinh.md#phân-biệt-tải-cổng-và-giả-định--gd1-16).

**GD1-09:** phần cộng lại toàn bộ hóa đơn ở trên mô tả hiện trạng. Thiết kế mới lưu TongChiTieu12T/DiemTichLuy/Hang và GhiNhanLoyalty theo MaHD; thanh toán chỉ xử lý khoản đến hạn chưa loại và hóa đơn mới. Tác vụ đêm dùng cùng cơ chế chống lặp; đọc/sử dụng hạng đồng bộ trước khi trả kết quả. Tần suất nghiệp vụ không đổi, chi phí truy tìm khoản đến hạn và tranh chấp theo hội viên phải đo ở GĐ2.

**GD1-15:** áp dụng [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung-gd1-15); nhãn theo lượt nghiệp vụ/ngày toàn hệ thống, độ nặng ghi riêng. Một lượt là đơn vị của chức năng ở từng hàng, không mặc định là một câu SQL hay một lần chạy tác vụ. Không cộng các chức năng bao hàm nhau hoặc lượt xử lý theo lô thành tổng giao dịch.

**Xử lý theo đợt:** trừ PT tại giờ hẹn; không chờ cuối giờ/cuối ngày. Cách gom lịch, kích hoạt tác vụ và chạy bù thuộc [ghi chú GĐ2](../../gd2/ghi-chu-trien-khai.md). CN-B08 và C-F14 là cùng một nghiệp vụ, không cộng trùng.
