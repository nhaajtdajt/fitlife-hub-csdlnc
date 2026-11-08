# Module C — Mô tả quy trình nghiệp vụ

Phạm vi: cửa kiểm soát, phiên vào–ra, lịch hẹn PT và đánh giá HLV. Module C dùng dữ liệu công bố của A (`ChiNhanh`, `NhanVien`, `LichSuCongTac`) và B (`HoiVien`, `GoiDichVu`, `GoiHoiVien`), đồng thời công bố dữ liệu lịch hẹn cho A và check-in cho D theo [hợp đồng giao diện](../../phan-cong.md).

Số lượng lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Quyết định #1, #2, #7, #8 và #11 đã chốt; #2, #8, #11 được cập nhật ngày 08/11/2026 để tách lịch PT khỏi quyền vào cổng. GD1-06/#12 đã chốt PT chỉ dạy tại nơi công tác, chặn đổi vị trí khi còn lịch không phù hợp. GD1-07/#15 dùng sức chứa chỉ cho báo cáo và quy ước tự đóng lượt của bộ dữ liệu mô phỏng; QD10 đã chốt khung một giờ bắt đầu vào giờ tròn, không cho chọn phút tùy ý.

## Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Hội viên | Quét mã vào/ra, xem lịch trống, đặt/hủy lịch và đánh giá buổi PT |
| Lễ tân | Hỗ trợ xử lý lượt quét lỗi, tra cứu người đang có mặt và lịch hẹn |
| PT | Xem lịch dạy; xác nhận kết quả buổi tập theo quyền được phân công |
| Quản lý chi nhánh | Quản lý các cửa kiểm soát, theo dõi công suất và xử lý dữ liệu bất thường |
| Hệ thống | Xác thực quyền vào, ghi log, chống lịch trùng, trừ quyền lợi đúng một lần và tự đóng lượt |

| Dữ liệu | Nội dung |
|---|---|
| `CuaKiemSoat` | Chi nhánh, số hiệu cửa, chức năng quét và trạng thái hoạt động; khóa ghép `(MaCN, SoHieuCua)` |
| `CheckIn` | Hội viên, chi nhánh, số cửa vào/ra và thời gian vào/ra |
| `LichHenPT` | Gói đã mua MaGHV (suy ra hội viên), PT, chi nhánh, giờ bắt đầu, trạng thái và cờ đã trừ buổi |
| `DanhGiaHLV` | Một điểm số/nhận xét cho một lịch PT đã hoàn thành |
| Dữ liệu A | Chi nhánh, chức vụ nhân viên và khoảng công tác để xác nhận PT thuộc đúng chi nhánh tại giờ dạy |
| Dữ liệu B | Hội viên, loại gói, hạn và số buổi còn lại để xác định quyền vào/đặt lịch |

## Quy tắc nghiệp vụ chung

1. Mỗi hội viên chỉ có tối đa một `CheckIn` chưa có `TGRa` trên toàn hệ thống. `TGRa` phải sau `TGVao`.
2. Cửa quét phải đang hoạt động, thuộc cùng chi nhánh với lượt check-in và cho phép đúng chiều quét. Dữ liệu lịch sử không bị xóa khi cửa ngừng hoạt động.
3. Quyền vào cổng là thẻ dịch vụ theo hạn còn hiệu lực hoặc vé vào một lượt còn lượt, dùng ở mọi chi nhánh. Gói PT chỉ dùng đặt lịch, không cấp quyền vào.
4. Ưu tiên thẻ theo hạn hợp lệ; nếu dùng vé vào một lượt thì trừ đúng một lượt. Vé hết sau lần vào, không gắn với giờ PT; ra rồi vào lại cần quyền vào mới nếu không còn thẻ/vé hợp lệ.
5. Giao dịch quét vào phải nguyên tử: xác thực quyền vào, trừ vé nếu cần và tạo `CheckIn` cùng thành công hoặc cùng thất bại. Cổng không đọc lịch PT, không thay đổi `DaTruBuoi`.
6. Lịch PT kéo dài một giờ, bắt đầu vào giờ tròn theo Asia/Saigon (phút/giây bằng 0), không nhận thời điểm bắt đầu tùy ý như 17h30. Một PT không có hai lịch chưa hủy chồng nhau; với khung cố định, cặp `(MaNV, NgayGio)` của lịch chưa hủy là duy nhất.
7. Nhân viên trong lịch phải có chức vụ PT và có đúng một giai đoạn công tác tại chi nhánh lịch hẹn trong toàn bộ buổi tập.
8. Khi đặt lịch, hội viên có gói PT còn hạn tại ngày hẹn và đủ buổi chưa bị giữ; không kiểm tra quyền vào tương lai. Mỗi lịch giữ một buổi của gói phù hợp; đến giờ tự trừ đúng buổi đã giữ, không phụ thuộc check-in.
9. Trạng thái lịch chỉ gồm `Đã đặt`, `Đã hoàn thành`, `Hủy`. Chuyển trạng thái phải theo luồng cho phép; lịch hoàn thành hoặc đã hủy không được đưa lại về `Đã đặt`.
10. Chỉ cho hội viên hủy trước ít nhất hai giờ, giải phóng buổi giữ mà không trừ; hủy muộn bị từ chối. Đến giờ hẹn lịch chưa hủy tự trừ một buổi kể cả vắng mặt. `DaTruBuoi` ngăn trừ lặp; vắng mặt không tính hoa hồng.
11. Chỉ lịch `Đã hoàn thành` mới được đánh giá và tính hoa hồng. Mỗi lịch có tối đa một đánh giá, điểm nguyên từ 1 đến 5.
12. Lượt quên quét ra được tự đóng tại giờ đóng cửa của chi nhánh. Khi đó `TGRa` có giá trị nhưng `SoHieuCuaRa` rỗng, nhờ vậy phân biệt được với lượt quét ra tại cửa mà không cần thêm thuộc tính nguồn đóng. Đây là quy ước mô phỏng #15; thời gian ra tự đóng là ước tính, không phải bằng chứng khách thực sự ở lại đến giờ đó.

## QT-C1. Quét vào và xác thực quyền vào cổng

**Ai làm:** hội viên tại cửa vào; lễ tân chỉ hỗ trợ khi có lỗi.

**Điều kiện trước:** cửa đang hoạt động, có chức năng `Vào` hoặc `Hai chiều`, nhận diện được chi nhánh và mã hội viên.

1. Cửa kiểm soát gửi mã hội viên, `MaCN`, `SoHieuCua` và thời điểm quét.
2. Hệ thống xác nhận hội viên tồn tại và chưa có lượt check-in đang mở.
3. Hệ thống chỉ lấy thẻ theo hạn thỏa `NgayBatDau <= NgayQuet <= NgayHetHan` hoặc vé vào đã mua còn lượt. Loại gói chờ kích hoạt và gói PT khỏi quyền vào cổng.
4. Ưu tiên thẻ theo hạn hợp lệ; nếu không có thì chọn vé vào còn lượt, gồm vé Spa. Không tra lịch PT để chọn quyền vào. Chỉ trừ vé đã chọn; vào bằng thẻ Gym/Yoga không tự trừ thêm Spa. Hệ thống chỉ kiểm tra tại cổng, không có bước xác nhận sử dụng Spa tại quầy.
5. Nếu dùng vé một lượt, hệ thống xác định đúng vé và kiểm tra lại số lượt; kiểm tra lại phiên đang mở để chống quét lặp.
6. Trong một giao dịch, giảm lượt của vé nếu cần và tạo `CheckIn` có `TGVao`, chưa có `TGRa`. Không cập nhật gói PT hoặc lịch PT.
7. Cửa kiểm soát mở cổng và trả kết quả. Không đếm số người hoặc so SucChua để quyết định mở cổng, kể cả khi tỷ lệ lấp đầy đã trên 100%. Đường xử lý mục tiêu dưới 50 ms; báo cáo/tổng hợp không chạy đồng bộ trong giao dịch này.

**Ngoại lệ:** mã không hợp lệ; cửa không tồn tại, ngừng hoạt động hoặc không cho phép chiều vào; đã có phiên đang mở; không có quyền hợp lệ; gói vừa hết lượt do giao dịch khác. Đây không bao gồm điều kiện sức chứa. Mọi lỗi đều không mở cổng, không tạo log và không trừ quyền lợi một phần.

## QT-C2. Quét ra

**Ai làm:** hội viên tại cửa ra.

1. Cửa kiểm soát gửi mã hội viên, `MaCN`, `SoHieuCua` và thời điểm quét.
2. Hệ thống tìm đúng một `CheckIn` đang mở của hội viên.
3. Hệ thống kiểm tra cửa thuộc cùng chi nhánh, cho phép chiều ra và thời gian ra sau thời gian vào.
4. Hệ thống cập nhật `TGRa`, `SoHieuCuaRa`, rồi mở cổng.

**Ngoại lệ:** không có hoặc có nhiều lượt đang mở; cửa sai chi nhánh, ngừng hoạt động hoặc không cho phép chiều ra; thời gian không hợp lệ. Hệ thống không tự chọn một dòng khi dữ liệu mâu thuẫn mà chuyển cho lễ tân xử lý.

## QT-C3. Tự đóng lượt quên quét ra

**Ai làm:** tác vụ hệ thống theo từng chi nhánh.

1. Theo quy ước mô phỏng #15, mỗi chi nhánh mở/đóng trong cùng ngày Asia/Saigon, giờ hoạt động cố định trong kỳ dữ liệu. Tác vụ lấy lượt còn mở có TGVao thuộc ngày đó và trước mốc đóng cửa đã đến; lượt ngoài giờ cấu hình được liệt kê là dữ liệu bất thường, không tự ghi TGRa <= TGVao.
2. Trong giao dịch, đọc lại lượt và chỉ cập nhật nếu TGRa vẫn rỗng. Hệ thống đặt `TGRa` bằng thời điểm đóng cửa áp dụng cho ngày đó; `SoHieuCuaRa` giữ rỗng vì không phát sinh lượt quét tại cửa ra.
3. Hệ thống ghi số dòng đã đóng và danh sách bất thường để quản lý đối chiếu.

Tác vụ phải chạy lặp an toàn: không ghi đè giờ ra thực tế hoặc cập nhật lại dòng đã đóng. Nếu tác vụ thất bại, chạy bù các ngày còn thiếu theo mốc đóng cửa của từng ngày, không dùng thời điểm chạy bù làm giờ ra. Báo cáo có lượt quá hạn chưa đóng được ghi là tạm tính; hoàn tất tác vụ rồi tính lại. Không tự đóng phiên của ngày mới khi chạy bù ngày cũ. Hoạt động qua đêm/24 giờ nằm ngoài giả định mô phỏng hiện tại.

## QT-C4. Xem lịch trống và đặt lịch PT

**Ai làm:** hội viên hoặc lễ tân thao tác thay hội viên.

1. Người dùng chọn ngày, chi nhánh và có thể chọn PT.
2. Hệ thống lấy các PT có chức vụ phù hợp và giai đoạn công tác bao phủ toàn bộ các khung một giờ tại chi nhánh đó.
3. Hệ thống loại các khung đã có lịch chưa hủy và trả danh sách còn trống.
4. Hội viên chọn PT/khung giờ. Hệ thống lấy gói thuộc hội viên, loại PT, thỏa `NgayBatDau <= NgayHen <= NgayHetHan` và `SoBuoiConLai - SoLichGiu > 0`; `SoLichGiu` đếm lịch cùng MaGHV ở trạng thái Đã đặt, DaTruBuoi = 0, kể cả lịch quá giờ chưa xử lý. Chọn NgayHetHan sớm nhất, hòa thì MaGHV tăng dần. Quyền vào cổng kiểm tra riêng lúc quét.
5. Kiểm tra lại lịch sử công tác gồm giai đoạn chờ: toàn bộ buổi hẹn phải có chức vụ PT/đúng chi nhánh. Kiểm tra chủ sở hữu, loại/hạn gói, số buổi khả dụng và khung PT tại lúc ghi nhận. Gói vừa hết khả dụng thì chọn lại ứng viên hợp lệ; không đổi PT/khung. Tạo `LichHenPT` có `MaGHV`, trạng thái `Đã đặt`, `DaTruBuoi = 0`; không lưu MaHV lặp, hội viên suy ra qua gói. Thêm lịch tăng số giữ chỗ, chưa giảm số buổi. Mọi bước lỗi hoàn tác; đặt lịch và đổi vị trí đồng thời không được bỏ sót thay đổi của nhau.
6. Ràng buộc duy nhất tại CSDL quyết định kết quả cuối cùng khi hai giao dịch cùng chọn một PT/khung. Chỉ một giao dịch thành công; giao dịch còn lại nhận thông báo khung vừa được đặt và phải tải lại lịch.

**Ngoại lệ:** PT không còn làm tại chi nhánh vào ngày hẹn; gói không đủ hạn/buổi; số buổi đã bị các lịch khác giữ chỗ; khung đã bị đặt; ngày giờ ở quá khứ. Không tạo lịch một phần hoặc tự đổi PT/khung.

## QT-C5. Hủy lịch, tự trừ buổi đến giờ và ghi nhận kết quả

### Hủy lịch

1. Hội viên chọn lịch `Đã đặt`; hệ thống lấy MaGHV bất biến của lịch, kiểm tra lại gói và lịch trong cùng giao dịch. Kiểm tra chủ sở hữu qua vGoiHoiVien.MaHV, trạng thái và thời gian hiện tại.
2. Chỉ chấp nhận nếu thời điểm yêu cầu ≤ `NgayGio - 2 giờ`; chuyển `Hủy`, ghi lý do hủy đúng hạn, giải phóng buổi giữ chỗ và giữ `DaTruBuoi = 0` trong cùng giao dịch.
3. Khi còn dưới hai giờ, từ chối hủy và giữ nguyên lịch. Check-in trước đó không ảnh hưởng quyền hủy hay số buổi PT.

### Tự trừ tại giờ hẹn

1. Tác vụ hệ thống lấy lịch đã đến `NgayGio`, còn `Đã đặt` và `DaTruBuoi = 0`, kể cả lịch đến hạn bị bỏ lỡ ở lần chạy trước.
2. Xác định GoiHoiVien theo MaGHV của lịch và kiểm tra lại dữ liệu gói/lịch trong cùng giao dịch. Kiểm tra trạng thái/cờ, số buổi và hạn gói tại NgayGio của lịch (không phải ngày chạy bù). Không đọc CheckIn hoặc yêu cầu hội viên đã vào cổng.
3. Trong cùng giao dịch, giảm đúng một buổi của gói đã giữ cho lịch, kết thúc giữ chỗ và đặt `DaTruBuoi = 1`. Giữ trạng thái `Đã đặt` để chờ ghi nhận kết quả buổi tập.
4. Lỗi thì hoàn tác toàn bộ và thử lại; lịch có cờ 1 không trừ lần nữa. Không tự lấy buổi của gói khác nếu liên kết/quyền lợi bị sai.

### Không đến và hoàn thành

Sau giờ kết thúc, PT/người có thẩm quyền xác nhận buổi thực sự hoàn thành hoặc vắng mặt; không suy ra tham dự PT chỉ từ check-in hay cờ đã trừ. Lịch hoàn thành chuyển `Đã hoàn thành`; lịch vắng mặt chuyển `Hủy` với `LyDoKetThuc = Không đến`. Cả hai giữ `DaTruBuoi = 1` và không trừ thêm. Chỉ lịch hoàn thành được đánh giá và tính hoa hồng. Nếu tác vụ đến giờ chưa trừ thành công, xử lý lại tác vụ đó trước khi chốt kết quả.

**Ngoại lệ chung:** lịch không tồn tại/sai chủ sở hữu; trạng thái không cho phép; hủy quá hạn; yêu cầu lặp; dữ liệu quyền lợi đã giữ không khớp. Dữ liệu lệch phải được báo đối soát, không trừ âm hoặc trừ hai lần.

## QT-C6. Đánh giá HLV

**Ai làm:** hội viên của buổi PT.

1. Hội viên chọn lịch đã hoàn thành chưa có đánh giá.
2. Hội viên nhập điểm nguyên 1–5 và nhận xét tùy chọn.
3. Hệ thống nối lịch qua MaGHV đến vGoiHoiVien.MaHV để kiểm tra chủ sở hữu, trạng thái `Đã hoàn thành` và chưa được đánh giá.
4. Hệ thống lưu đánh giá gắn duy nhất với lịch; đánh giá được dùng trong báo cáo hiệu suất HLV.

**Ngoại lệ:** lịch chưa hoàn thành/đã hủy, người gửi không phải hội viên của lịch, điểm ngoài miền hoặc lịch đã có đánh giá.

## Quan hệ với module khác

| Module | C cung cấp | C sử dụng / cần xác nhận |
|---|---|---|
| A — Chi nhánh & Nhân sự | `LichHenPT(MaLichHen, MaNV, MaCN, NgayGio, TrangThai)` cho tính hoa hồng và kiểm tra lịch vướng khi đổi vị trí | `ChiNhanh(MaCN, SucChua, GioMoCua, GioDongCua)`, `NhanVien(MaNV, ChucVu)`, `LichSuCongTac(MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc)` |
| B — Hội viên, Gói & Thân thiết | Lịch đến giờ để tự trừ quyền lợi, độc lập kết quả tham dự | `HoiVien(MaHV)`, `GoiDichVu(MaGoi, LoaiGoi)`, `vGoiHoiVien(MaGHV, MaHV, MaGoi, NgayBatDau, NgayHetHan, SoBuoiConLai)`; MaGHV đã chốt, thao tác giữ/hủy/trừ tuân theo giao dịch trong [thiết kế gói PT](../b/goi-pt.md) |
| D — Hóa đơn & Báo cáo | `CheckIn(MaHV, MaCN, TGVao, TGRa)` cho tỷ lệ lấp đầy | Không ghi bảng D |

**GD1-10 — kỳ hoa hồng:** từ chối thêm/đổi nguồn vào kỳ đã chốt. Không sửa/xóa MaNV, MaCN, NgayGio, TrangThai của buổi đã chốt. Chốt A và ghi nguồn C phải duy trì điều kiện này cả khi diễn ra đồng thời; xem [thiết kế A](../a/thiet-ke-logic.md).

**GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem [thiết kế B](../b/thiet-ke-logic.md).
