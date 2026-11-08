# Module C — Thiết kế mức logic

Kiểu SQL Server cụ thể và chỉ mục thuộc giai đoạn 2. Thiết kế này giữ đúng các khóa chung `MaCN`, `MaNV`, `MaHV`, `MaGHV`; `MaCheckIn`, `MaLichHen` và `MaDanhGia` là khóa nội bộ của C. Cửa kiểm soát được nhận diện bằng khóa tự nhiên ghép `(MaCN, SoHieuCua)`. Thực thể A/B chỉ nêu các cột C cần tham chiếu.

## Lược đồ quan hệ

```text
CuaKiemSoat(MaCN, SoHieuCua, ChucNang, DangHoatDong)
CheckIn(MaCheckIn, MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa)
LichHenPT(MaLichHen, MaGHV, MaNV, MaCN, NgayGio, ThoiDiemDat, TrangThai, DaTruBuoi, LyDoKetThuc)
DanhGiaHLV(MaDanhGia, MaLichHen, Diem, NhanXet, ThoiDiemDanhGia)
```

Các quan hệ ngoài module: `ChiNhanh(MaCN, SucChua, GioMoCua, GioDongCua)`, `NhanVien(MaNV, ChucVu)`, `LichSuCongTac(MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc)`, `HoiVien(MaHV)`, `GoiDichVu(MaGoi, LoaiGoi, Gia)` và `vGoiHoiVien(MaGHV, MaHV, MaGoi, NgayBatDau, NgayHetHan, SoBuoiConLai)`.

## Khóa, phụ thuộc hàm và dạng chuẩn

| Quan hệ | Khóa chính / khóa ứng viên | Phụ thuộc hàm chính | Dạng chuẩn và ghi chú |
|---|---|---|---|
| `CuaKiemSoat` | PK `{MaCN, SoHieuCua}` | `(MaCN, SoHieuCua) → ChucNang, DangHoatDong` | BCNF. Số hiệu cửa chỉ duy nhất trong phạm vi một chi nhánh; mọi định thức không tầm thường là khóa. |
| `CheckIn` | PK `{MaCheckIn}` | `MaCheckIn → MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa` | BCNF. “Một lượt đang mở/hội viên” là ràng buộc có điều kiện, không phải PTH trên toàn quan hệ. Không lưu mô tả hội viên/chi nhánh/cửa. |
| `LichHenPT` | PK `{MaLichHen}` | `MaLichHen → MaGHV, MaNV, MaCN, NgayGio, ThoiDiemDat, TrangThai, DaTruBuoi, LyDoKetThuc` | BCNF theo PTH đã nêu. Không lưu MaHV vì MaGHV xác định chủ gói; nhờ đó không tạo PTH không khóa MaGHV → MaHV trong lịch. Tính duy nhất `(MaNV, NgayGio)` chỉ áp dụng cho dòng chưa hủy nên là ràng buộc có điều kiện, không coi là khóa ứng viên toàn quan hệ. |
| `DanhGiaHLV` | PK `{MaDanhGia}`; AK `{MaLichHen}` | `MaDanhGia → MaLichHen, Diem, NhanXet, ThoiDiemDanhGia`; `MaLichHen → MaDanhGia, Diem, NhanXet, ThoiDiemDanhGia` | BCNF. `MaLichHen` là khóa ứng viên `UNIQUE`, bảo đảm một lịch có tối đa một đánh giá. Không lặp `MaHV`, `MaNV` vì có thể suy ra qua lịch. |

Các khóa ngoại:

- `CuaKiemSoat.MaCN → ChiNhanh.MaCN`.
- `CheckIn.MaHV → HoiVien.MaHV`; `MaCN → ChiNhanh.MaCN`.
- `(CheckIn.MaCN, CheckIn.SoHieuCuaVao) → CuaKiemSoat(MaCN, SoHieuCua)`.
- `(CheckIn.MaCN, CheckIn.SoHieuCuaRa) → CuaKiemSoat(MaCN, SoHieuCua)`; `SoHieuCuaRa` có thể rỗng.
- `LichHenPT.MaGHV → GoiHoiVien.MaGHV` (bắt buộc); `MaNV → NhanVien.MaNV`; `MaCN → ChiNhanh.MaCN`.
- `DanhGiaHLV.MaLichHen → LichHenPT.MaLichHen`; `MaLichHen` có ràng buộc `UNIQUE`.

B đã công bố MaGHV là khóa chính từng lần mua. Lịch lưu MaGHV bất biến; hội viên suy ra qua vGoiHoiVien.MaHV. Không lưu MaHV lặp trên lịch và không dùng (MaHV, MaGoi) để định danh lần mua. Tra cứu lịch sử, kiểm tra chủ lịch/đánh giá phải nối qua gói. Xem [thiết kế giữ/trừ PT](../b/goi-pt.md) và [ERD hiện hành](erd.md).

## Thông tin thuộc tính

| Quan hệ | Thuộc tính | Ý nghĩa / miền giá trị |
|---|---|---|
| `CuaKiemSoat` | `MaCN`, `SoHieuCua` | Khóa ghép xác định cửa trong một chi nhánh; `SoHieuCua` là số nguyên dương và không được tái sử dụng cho cửa khác trong cùng chi nhánh. |
| | `ChucNang` | Một trong `Vào`, `Ra`, `Hai chiều`; quy định chiều quét cửa được phép nhận. |
| | `DangHoatDong` | Cờ cho phép cửa nhận lượt quét mới; không dùng để xóa lịch sử. |
| `CheckIn` | `MaCheckIn` | Mã phiên vào–ra duy nhất. |
| | `MaHV`, `MaCN` | Hội viên và chi nhánh phát sinh lượt. Đây là các cột C công bố cho D. |
| | `SoHieuCuaVao` | Số hiệu cửa ghi nhận lượt vào tại `MaCN`, bắt buộc. |
| | `SoHieuCuaRa` | Số hiệu cửa ghi nhận lượt ra tại cùng `MaCN`; rỗng khi đang mở hoặc đóng tự động. |
| | `TGVao`, `TGRa` | Thời gian vào và thời gian ra. `TGRa` rỗng khi hội viên đang ở trong chi nhánh. |
| `LichHenPT` | `MaLichHen` | Mã lịch hẹn PT duy nhất. |
| | `MaGHV` | FK bắt buộc đến đúng lần mua gói PT; không đổi sau khi đặt. Hội viên là vGoiHoiVien.MaHV, không lưu lặp trên lịch. |
| | `MaNV`, `MaCN` | PT và nơi diễn ra buổi tập, công bố cho A. |
| | `NgayGio` | Thời điểm bắt đầu khung một giờ; công bố cho A. |
| | `ThoiDiemDat` | Thời điểm tạo lịch, phục vụ đối soát và quy tắc hủy. |
| | `TrangThai` | `Đã đặt`, `Đã hoàn thành` hoặc `Hủy`; công bố cho A. |
| | `DaTruBuoi` | 0/1, tác vụ đến giờ hẹn chuyển 0 → 1 cùng giao dịch trừ gói đã giữ; không do check-in hoặc xác nhận hoàn thành kích hoạt. |
| | `LyDoKetThuc` | Lý do hủy đúng hạn hoặc không đến; rỗng khi đang đặt/hoàn thành bình thường. Hủy muộn bị từ chối, không tạo kết quả hủy. |
| `DanhGiaHLV` | `MaDanhGia` | Mã đánh giá duy nhất, khóa chính. |
| | `MaLichHen` | FK đến lịch được đánh giá và có `UNIQUE`, bảo đảm một lịch tối đa một đánh giá. |
| | `Diem` | Số nguyên từ 1 đến 5. |
| | `NhanXet` | Nội dung tùy chọn, có giới hạn độ dài ở thiết kế vật lý. |
| | `ThoiDiemDanhGia` | Thời điểm gửi đánh giá, không trước lúc buổi tập hoàn thành. |

## Kết luận chuẩn hóa

Bốn quan hệ của C đạt BCNF theo tập PTH đã nêu vì mọi định thức của PTH không tầm thường là khóa. Danh mục cửa được tách khỏi log; đánh giá không lặp hội viên/PT; lịch dùng MaGHV và không lặp MaHV từ gói, không sao chép chức vụ từ A; check-in không sao chép thông tin gói từ B. Các điều kiện “duy nhất khi chưa đóng/chưa hủy” và kiểm tra khoảng thời gian là ràng buộc nghiệp vụ có điều kiện, cần index/ràng buộc/giao dịch ở giai đoạn vật lý chứ không tạo thêm PTH toàn quan hệ.

## Ràng buộc logic và bảng tầm ảnh hưởng

Ký hiệu `+`: thao tác có thể làm vi phạm; `−`: thao tác không thể trực tiếp làm vi phạm. Với ràng buộc trạng thái/lịch sử, xóa vật lý nhìn chung bị cấm dù việc xóa có thể làm mệnh đề kiểm tra trở nên đúng.

| ID | Bối cảnh | Nội dung | Bảng tầm ảnh hưởng (Thêm / Xóa / Sửa) |
|---|---|---|---|
| C-R01 | `CheckIn` | `TGRa` rỗng khi hội viên còn trong phòng tập; nếu có thì phải lớn hơn `TGVao`. `SoHieuCuaRa` có thể rỗng khi lượt được hệ thống tự đóng. | `CheckIn` (+ / − / + `TGVao, TGRa, SoHieuCuaRa`) |
| C-R02 | `CheckIn` | Mỗi hội viên có tối đa một lượt đang mở trên toàn hệ thống. | `CheckIn` (+ / − / + `MaHV, TGRa`) |
| C-R03 | `CheckIn`, `CuaKiemSoat` | Cửa vào/ra được xác định bằng `(MaCN, SoHieuCua)`, phải hoạt động và cho phép đúng chiều quét tại thời điểm nhận quét. | `CheckIn` (+ / − / + `MaCN, SoHieuCuaVao, SoHieuCuaRa`); `CuaKiemSoat` (+ / − / + `MaCN, SoHieuCua, ChucNang, DangHoatDong`) |
| C-R04 | `CheckIn`, dữ liệu thẻ/vé vào | Chỉ mở cổng khi thẻ theo hạn thỏa `NgayBatDau <= Ngay(TGVao) <= NgayHetHan` hoặc vé đã mua còn lượt; gói chờ không cấp quyền vào; PT không cấp quyền vào. Trừ vé và tạo log nguyên tử, không âm, không trừ lặp; ra rồi vào lại cần quyền vào mới. | `CheckIn` (+ / − / + `MaHV, TGVao`); `GoiHoiVien` (− / − / + `NgayBatDau, NgayHetHan, SoBuoiConLai`); `GoiDichVu` (− / − / + `LoaiGoi`) |
| C-R05 | `CheckIn`, `ChiNhanh` | Lượt tự đóng (TGRa có, SoHieuCuaRa rỗng) có TGRa bằng mốc đóng cửa của ngày vào, sau TGVao, theo quy ước mô phỏng #15. Chạy bù không ghi đè lượt đã đóng. SucChua không phải điều kiện mở cổng. | `CheckIn` (+ / − / + `MaCN, TGVao, TGRa, SoHieuCuaRa`); `ChiNhanh` (− / + / + `MaCN, GioMoCua, GioDongCua`) |
| C-R06 | `LichHenPT` | Không có hai lịch chưa hủy của cùng `MaNV` tại cùng `NgayGio`; khung cố định kéo dài một giờ, NgayGio có phút/giây bằng 0 theo QD10. | `LichHenPT` (+ / − / + `MaNV, NgayGio, TrangThai`) |
| C-R07 | `LichHenPT`, `NhanVien`, `LichSuCongTac` | Nhân viên có ChucVu = PT trong đúng một khoảng LichSuCongTac bao phủ buổi tập tại MaCN của lịch; không dùng chức vụ hiện tại để xét lịch quá khứ/tương lai. Ngày công tác DATE tính cả hai biên theo #13. Đổi vị trí bị từ chối nếu làm lịch chưa hủy mất điều kiện này, kể cả buổi kéo qua mốc hiệu lực; đặt lịch và đổi vị trí phải kiểm tra nhất quán dữ liệu khi ghi nhận. | `LichHenPT` (+ / − / + `MaNV, MaCN, NgayGio`); `NhanVien` (− / − / −); `LichSuCongTac` (+ / + / + `MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc`) |
| C-R08 | `LichHenPT`, `GoiHoiVien`, `GoiDichVu` | MaGHV thuộc hội viên đặt, loại PT và đủ hạn tại ngày hẹn. Với mỗi gói: 0 <= COUNT(lịch cùng MaGHV, Đã đặt, DaTruBuoi=0) <= SoBuoiConLai; đếm cả lịch quá giờ chưa trừ. Không kiểm tra quyền vào khi giữ lịch. | `LichHenPT` (+ / − / + `MaGHV, NgayGio, TrangThai, DaTruBuoi`); `GoiHoiVien` (− / + / + `MaGHV, MaHD, STT, NgayBatDau, NgayHetHan, SoBuoiConLai`); `GoiDichVu` (− / + / + `MaGoi, LoaiGoi`) |
| C-R09 | `LichHenPT` | Chỉ cho phép `Đã đặt → Đã hoàn thành` hoặc `Đã đặt → Hủy`; trạng thái thuộc tập cho phép và trạng thái kết thúc không đảo ngược. | `LichHenPT` (+ / − / + `TrangThai`) |
| C-R10 | `LichHenPT`, dữ liệu gói PT | Đến `NgayGio`, lịch chưa hủy được tự trừ một buổi của đúng MaGHV đã giữ và đổi `DaTruBuoi` 0 → 1 trong cùng giao dịch, độc lập check-in/tham dự. Chỉ hủy được khi còn ít nhất 2 giờ: giải phóng giữ chỗ, cờ 0. Hủy muộn bị từ chối. Hoàn thành/vắng mặt phải có cờ 1, không trừ thêm. Tác vụ chạy lại xử lý lịch đến hạn chưa trừ. | `LichHenPT` (+ / − / + `MaGHV, NgayGio, TrangThai, DaTruBuoi, LyDoKetThuc`); `GoiHoiVien` (− / − / + `SoBuoiConLai`); thời gian tới `NgayGio` kích hoạt tác vụ |
| C-R11 | `DanhGiaHLV`, `LichHenPT` | `MaLichHen` trong đánh giá là duy nhất; chỉ lịch hoàn thành được đánh giá; điểm nguyên 1–5 và thời điểm đánh giá không trước buổi tập. | `DanhGiaHLV` (+ / − / + `MaLichHen, Diem, ThoiDiemDanhGia`); `LichHenPT` (− / − / + `TrangThai, NgayGio`) |
| C-R12 | Dữ liệu lịch sử C | Không xóa vật lý check-in, lịch đã kết thúc hoặc đánh giá đã dùng cho báo cáo/hoa hồng; sửa điều chỉnh phải có quyền và khả năng truy vết. | `CheckIn` (− / + / +); `LichHenPT` (− / + / +); `DanhGiaHLV` (− / + / +) |
| C-R13 | `LichHenPT`, `GoiHoiVien`, `GoiDichVu` | MaGHV của lịch và chủ/loại/hạn gói đã mua không đổi trong luồng hiện tại; không xóa gói có lịch. Mua mới không chuyển lịch/buổi của gói cũ. Không sửa LoaiGoi của mã danh mục đã bán. | `LichHenPT` (− / − / + `MaGHV`); `GoiHoiVien` (− / + / + `MaGHV, MaHD, STT, NgayBatDau, NgayHetHan`); `GoiDichVu` (− / − / + `LoaiGoi`) |

Các nguồn suy ra chủ/loại gói trong C-R04/C-R08/C-R13 còn gồm HoaDon.MaHV và ChiTietHoaDon.(MaHD,STT,LoaiDong,MaGoi). Tầm ảnh hưởng nguồn: HoaDon/ChiTietHoaDon (− / + / + các cột nguồn); kiểm tra cùng FK và quy tắc bất biến sau thanh toán. MaHV/MaGoi không phải cột của GoiHoiVien cơ sở. Xem [ràng buộc tổng](../thiet-ke-tong.md).

## Điểm ghép với A/B/D

- A đã chốt DATE và hai biên ngày đều thuộc giai đoạn, bấm đổi ngày D thì vị trí mới từ D+1; đọc ChucVu lịch sử. GD1-06 chặn đổi vị trí khi còn lịch không phù hợp, trả danh sách lịch vướng và giữ nguyên dữ liệu; kiểm tra cả buổi kéo qua mốc hiệu lực. A chỉ tính hoa hồng cho `LichHenPT.TrangThai = Đã hoàn thành` tại đúng chi nhánh lịch.
- B/C dùng MaGHV đã công bố; chọn gói, giữ chỗ, hủy và tự trừ tuân theo [giao dịch PT](../b/goi-pt.md). Khi ghi nhận phải kiểm tra lại số giữ chỗ, quyền lợi và khung PT; đặt lịch phải nhất quán với thay đổi vị trí A. Quyền vào cổng kiểm tra riêng; vé một lượt không liên quan giờ PT.
- D dùng điều kiện `TGVao <= T AND (TGRa IS NULL OR TGRa > T)` để tính hiện diện tại T; `SoHieuCuaRa` rỗng không đồng nghĩa hội viên còn ở trong phòng nếu `TGRa` đã có giá trị.

**GD1-10 — kỳ hoa hồng:** từ chối thêm/đổi nguồn vào kỳ đã chốt. Không sửa/xóa MaNV, MaCN, NgayGio, TrangThai của buổi đã chốt. Chốt A và ghi nguồn C phải duy trì điều kiện này cả khi diễn ra đồng thời; xem [thiết kế A](../a/thiet-ke-logic.md).

**GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem [thiết kế B](../b/thiet-ke-logic.md).
