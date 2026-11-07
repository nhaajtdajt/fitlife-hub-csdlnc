# Module C — Thiết kế mức logic

Kiểu SQL Server cụ thể và chỉ mục thuộc giai đoạn 2. Thiết kế này giữ đúng các khóa chung `MaCN`, `MaNV`, `MaHV`; `MaCheckIn`, `MaLichHen` và `MaDanhGia` là khóa nội bộ của C. Cửa kiểm soát được nhận diện bằng khóa tự nhiên ghép `(MaCN, SoHieuCua)`. Thực thể A/B chỉ nêu các cột C cần tham chiếu.

## Lược đồ quan hệ

```text
CuaKiemSoat(MaCN, SoHieuCua, ChucNang, DangHoatDong)
CheckIn(MaCheckIn, MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa)
LichHenPT(MaLichHen, MaHV, MaNV, MaCN, NgayGio, ThoiDiemDat, TrangThai, DaTruBuoi, LyDoKetThuc)
DanhGiaHLV(MaDanhGia, MaLichHen, Diem, NhanXet, ThoiDiemDanhGia)
```

Các quan hệ ngoài module: `ChiNhanh(MaCN, SucChua)`, `NhanVien(MaNV, ChucVu)`, `LichSuCongTac(MaNV, MaCN, NgayBatDau, NgayKetThuc)`, `HoiVien(MaHV)`, `GoiDichVu(MaGoi, LoaiGoi, Gia)` và `GoiHoiVien(MaHV, MaGoi, NgayHetHan, SoBuoiConLai)`.

## Khóa, phụ thuộc hàm và dạng chuẩn

| Quan hệ | Khóa chính / khóa ứng viên | Phụ thuộc hàm chính | Dạng chuẩn và ghi chú |
|---|---|---|---|
| `CuaKiemSoat` | PK `{MaCN, SoHieuCua}` | `(MaCN, SoHieuCua) → ChucNang, DangHoatDong` | BCNF. Số hiệu cửa chỉ duy nhất trong phạm vi một chi nhánh; mọi định thức không tầm thường là khóa. |
| `CheckIn` | PK `{MaCheckIn}` | `MaCheckIn → MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa` | BCNF. “Một lượt đang mở/hội viên” là ràng buộc có điều kiện, không phải PTH trên toàn quan hệ. Không lưu mô tả hội viên/chi nhánh/cửa. |
| `LichHenPT` | PK `{MaLichHen}` | `MaLichHen → MaHV, MaNV, MaCN, NgayGio, ThoiDiemDat, TrangThai, DaTruBuoi, LyDoKetThuc` | BCNF. Tính duy nhất `(MaNV, NgayGio)` chỉ áp dụng cho dòng chưa hủy nên là ràng buộc có điều kiện, không coi là khóa ứng viên toàn quan hệ. |
| `DanhGiaHLV` | PK `{MaDanhGia}`; AK `{MaLichHen}` | `MaDanhGia → MaLichHen, Diem, NhanXet, ThoiDiemDanhGia`; `MaLichHen → MaDanhGia, Diem, NhanXet, ThoiDiemDanhGia` | BCNF. `MaLichHen` là khóa ứng viên `UNIQUE`, bảo đảm một lịch có tối đa một đánh giá. Không lặp `MaHV`, `MaNV` vì có thể suy ra qua lịch. |

Các khóa ngoại:

- `CuaKiemSoat.MaCN → ChiNhanh.MaCN`.
- `CheckIn.MaHV → HoiVien.MaHV`; `MaCN → ChiNhanh.MaCN`.
- `(CheckIn.MaCN, CheckIn.SoHieuCuaVao) → CuaKiemSoat(MaCN, SoHieuCua)`.
- `(CheckIn.MaCN, CheckIn.SoHieuCuaRa) → CuaKiemSoat(MaCN, SoHieuCua)`; `SoHieuCuaRa` có thể rỗng.
- `LichHenPT.MaHV → HoiVien.MaHV`; `MaNV → NhanVien.MaNV`; `MaCN → ChiNhanh.MaCN`.
- `DanhGiaHLV.MaLichHen → LichHenPT.MaLichHen`; `MaLichHen` có ràng buộc `UNIQUE`.

Không khai FK từ `LichHenPT` đến `GoiHoiVien` ở phiên bản hợp đồng hiện tại vì B chưa công bố khóa của lần mua gói. Trước giai đoạn 2 phải chốt `MaGHV` hoặc thủ tục B quản lý việc giữ/trừ quyền lợi; không được tự chọn tùy ý một dòng theo `(MaHV, MaGoi)`.

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
| | `MaHV`, `MaNV`, `MaCN` | Hội viên, PT và nơi diễn ra buổi tập. `MaNV`, `MaCN` được công bố cho A. |
| | `NgayGio` | Thời điểm bắt đầu khung một giờ; công bố cho A. |
| | `ThoiDiemDat` | Thời điểm tạo lịch, phục vụ đối soát và quy tắc hủy. |
| | `TrangThai` | `Đã đặt`, `Đã hoàn thành` hoặc `Hủy`; công bố cho A. |
| | `DaTruBuoi` | 0/1, bảo đảm quyền lợi PT của lịch bị trừ tối đa một lần. |
| | `LyDoKetThuc` | Lý do hủy đúng hạn, hủy muộn hoặc không đến; rỗng khi đang đặt/hoàn thành bình thường. |
| `DanhGiaHLV` | `MaDanhGia` | Mã đánh giá duy nhất, khóa chính. |
| | `MaLichHen` | FK đến lịch được đánh giá và có `UNIQUE`, bảo đảm một lịch tối đa một đánh giá. |
| | `Diem` | Số nguyên từ 1 đến 5. |
| | `NhanXet` | Nội dung tùy chọn, có giới hạn độ dài ở thiết kế vật lý. |
| | `ThoiDiemDanhGia` | Thời điểm gửi đánh giá, không trước lúc buổi tập hoàn thành. |

## Kết luận chuẩn hóa

Bốn quan hệ của C đạt BCNF theo tập PTH đã nêu vì mọi định thức của PTH không tầm thường là khóa. Danh mục cửa được tách khỏi log; đánh giá không lặp hội viên/PT; lịch không sao chép thông tin chức vụ hoặc nơi công tác từ A; check-in không sao chép thông tin gói từ B. Các điều kiện “duy nhất khi chưa đóng/chưa hủy” và kiểm tra khoảng thời gian là ràng buộc nghiệp vụ có điều kiện, cần index/ràng buộc/giao dịch ở giai đoạn vật lý chứ không tạo thêm PTH toàn quan hệ.

## Ràng buộc logic và bảng tầm ảnh hưởng

Ký hiệu `+`: thao tác có thể làm vi phạm; `−`: thao tác không thể trực tiếp làm vi phạm. Với ràng buộc trạng thái/lịch sử, xóa vật lý nhìn chung bị cấm dù việc xóa có thể làm mệnh đề kiểm tra trở nên đúng.

| ID | Bối cảnh | Nội dung | Bảng tầm ảnh hưởng (Thêm / Xóa / Sửa) |
|---|---|---|---|
| C-R01 | `CheckIn` | `TGRa` rỗng khi hội viên còn trong phòng tập; nếu có thì phải lớn hơn `TGVao`. `SoHieuCuaRa` có thể rỗng khi lượt được hệ thống tự đóng. | `CheckIn` (+ / − / + `TGVao, TGRa, SoHieuCuaRa`) |
| C-R02 | `CheckIn` | Mỗi hội viên có tối đa một lượt đang mở trên toàn hệ thống. | `CheckIn` (+ / − / + `MaHV, TGRa`) |
| C-R03 | `CheckIn`, `CuaKiemSoat` | Cửa vào/ra được xác định bằng `(MaCN, SoHieuCua)`, phải hoạt động và cho phép đúng chiều quét tại thời điểm nhận quét. | `CheckIn` (+ / − / + `MaCN, SoHieuCuaVao, SoHieuCuaRa`); `CuaKiemSoat` (+ / − / + `MaCN, SoHieuCua, ChucNang, DangHoatDong`) |
| C-R04 | `CheckIn`, dữ liệu gói | Chỉ mở cổng khi có Gym/Yoga còn hạn, vé Spa còn lượt hoặc gói PT hợp lệ kèm lịch trong ngày; thao tác trừ quyền lợi và tạo log là nguyên tử, số buổi không âm. | `CheckIn` (+ / − / + `MaHV, TGVao`); `GoiHoiVien` (− / − / + `NgayHetHan, SoBuoiConLai`); `LichHenPT` (− / − / + `TrangThai, DaTruBuoi`) |
| C-R05 | `CheckIn`, `ChiNhanh` | Nếu nhóm chốt chặn công suất, số lượt đang mở tại chi nhánh không vượt `SucChua`; lượt quên ra phải được tự đóng theo giờ hoạt động. | `CheckIn` (+ / − / + `MaCN, TGVao, TGRa`); `ChiNhanh` (− / − / + `SucChua`) |
| C-R06 | `LichHenPT` | Không có hai lịch chưa hủy của cùng `MaNV` tại cùng `NgayGio`; khung cố định kéo dài một giờ. | `LichHenPT` (+ / − / + `MaNV, NgayGio, TrangThai`) |
| C-R07 | `LichHenPT`, `NhanVien`, `LichSuCongTac` | Nhân viên là PT và có đúng một khoảng công tác bao phủ buổi tập tại `MaCN` của lịch. | `LichHenPT` (+ / − / + `MaNV, MaCN, NgayGio`); `NhanVien` (− / − / + `ChucVu`); `LichSuCongTac` (+ / + / + `MaNV, MaCN, NgayBatDau, NgayKetThuc`) |
| C-R08 | `LichHenPT`, dữ liệu gói | Tại ngày hẹn, hội viên có Gym/Yoga còn hạn và gói PT còn hạn/còn buổi; số lịch đang giữ quyền lợi không vượt số buổi khả dụng. | `LichHenPT` (+ / − / + `MaHV, NgayGio, TrangThai, DaTruBuoi`); `GoiHoiVien` (− / − / + `NgayHetHan, SoBuoiConLai`); `GoiDichVu` (− / − / + `LoaiGoi`) |
| C-R09 | `LichHenPT` | Chỉ cho phép `Đã đặt → Đã hoàn thành` hoặc `Đã đặt → Hủy`; trạng thái thuộc tập cho phép và trạng thái kết thúc không đảo ngược. | `LichHenPT` (+ / − / + `TrangThai`) |
| C-R10 | `LichHenPT`, dữ liệu gói | `DaTruBuoi` chỉ đổi 0 → 1 và mỗi lần đổi đi kèm đúng một lần giảm quyền lợi; hoàn thành/hủy muộn/không đến phải bằng 1, hủy đúng hạn bằng 0. | `LichHenPT` (+ / − / + `TrangThai, DaTruBuoi, LyDoKetThuc`); `GoiHoiVien` (− / − / + `SoBuoiConLai`) |
| C-R11 | `DanhGiaHLV`, `LichHenPT` | `MaLichHen` trong đánh giá là duy nhất; chỉ lịch hoàn thành được đánh giá; điểm nguyên 1–5 và thời điểm đánh giá không trước buổi tập. | `DanhGiaHLV` (+ / − / + `MaLichHen, Diem, ThoiDiemDanhGia`); `LichHenPT` (− / − / + `TrangThai, NgayGio`) |
| C-R12 | Dữ liệu lịch sử C | Không xóa vật lý check-in, lịch đã kết thúc hoặc đánh giá đã dùng cho báo cáo/hoa hồng; sửa điều chỉnh phải có quyền và khả năng truy vết. | `CheckIn` (− / + / +); `LichHenPT` (− / + / +); `DanhGiaHLV` (− / + / +) |

## Điểm ghép với A/B/D

- A xác nhận kiểu thời gian và quy tắc khoảng công tác, đồng thời chỉ tính hoa hồng cho `LichHenPT.TrangThai = Đã hoàn thành` tại đúng chi nhánh lịch.
- B xác nhận một API/stored procedure nguyên tử cho ba thao tác: kiểm tra quyền tại thời điểm, giữ chỗ khi đặt lịch và trừ đúng một quyền lợi theo `MaLichHen`. Nếu chọn công bố `MaGHV`, hợp đồng giao diện phải được cập nhật trước khi viết khóa ngoại/script.
- D dùng điều kiện `TGVao <= T AND (TGRa IS NULL OR TGRa > T)` để tính hiện diện tại T; `SoHieuCuaRa` rỗng không đồng nghĩa hội viên còn ở trong phòng nếu `TGRa` đã có giá trị.
