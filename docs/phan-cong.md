# Phân công nhóm (2 giai đoạn)

Mỗi người **sở hữu một module** xuyên suốt cả 2 giai đoạn. Trong mỗi giai đoạn, cả nhóm đi qua cùng một chuỗi **bước**; mỗi bước mỗi người làm phần của module mình, rồi có **điểm ghép** trước khi sang bước sau. Không chia theo tuần — xong bước nào sang bước đó.

Mẫu nội dung từng phần theo [required.md](../required.md).

**Cách để ít phụ thuộc nhất:** ở bước 1.0 cả nhóm chốt một **"hợp đồng giao diện"** (bảng bên dưới): mỗi module công bố sẵn những cột mà module khác sẽ dùng. Sau đó ai cũng làm độc lập, chỉ dùng đúng những cột đã chốt, không cần chờ nhau. Ở mỗi bước dưới đây, cột **Phụ thuộc** ghi rõ "không phụ thuộc ai" hoặc "chờ ai làm gì".

## Module

| Người | Module | Thực thể chính | Vấn đề của đề phải giải |
|---|---|---|---|
| **A** | Chi nhánh & Nhân sự | `ChiNhanh`, dịch vụ/tiện ích, `NhanVien`, `LichSuCongTac`, hoa hồng PT | 5 — lịch sử điều chuyển & hoa hồng |
| **B** | Hội viên, Gói & Thân thiết | `HoiVien`, `GoiDichVu`, `GoiHoiVien` (gói đã mua), hạng, lịch sử điểm | 4 — tính lại hạng quá tải |
| **C** | Check-in & Lịch hẹn PT | `CheckIn`, `CuaKiemSoat`, `LichHenPT`, `DanhGiaHLV` | 1 — nghẽn check-in (< 50ms); 2 — overbooking PT |
| **D** | Sản phẩm, Hóa đơn & Báo cáo | `SanPham`, tồn kho theo chi nhánh, `HoaDon`, `ChiTietHoaDon` | 3 — log lớn & báo cáo chậm |

## Hợp đồng giao diện (chốt ở bước 1.0)

Mỗi dòng: bảng của ai, những cột nào được **công bố** cho người khác dùng, ai dùng. Thêm cột mới thì tự do; **đổi tên/kiểu hoặc xóa cột trong hợp đồng** thì mở issue và báo người dùng nó.

| Bảng (chủ) | Cột công bố | Ai dùng, để làm gì |
|---|---|---|
| `ChiNhanh` (A) | `MaCN`, `SucChua` | C (check-in), D (tỷ lệ lấp đầy) |
| `NhanVien` (A) | `MaNV`, `ChucVu` | C (kiểm tra là PT), D (người lập hóa đơn) |
| `LichSuCongTac` (A) | `MaNV`, `MaCN`, `NgayBatDau`, `NgayKetThuc` | C (PT thuộc chi nhánh nào tại ngày dạy) |
| `HoiVien` (B) | `MaHV` | C, D |
| `GoiDichVu` (B) | `MaGoi`, `LoaiGoi`, `Gia` | C (loại gói khi đặt PT), D (dòng hóa đơn là gói) |
| `GoiHoiVien` (B) | `MaHV`, `MaGoi`, `NgayHetHan`, `SoBuoiConLai` | C (gói còn hạn / còn buổi) |
| `LichHenPT` (C) | `MaNV`, `MaCN`, `NgayGio`, `TrangThai` | A (đếm buổi dạy tính hoa hồng) |
| `CheckIn` (C) | `MaHV`, `MaCN`, `TGVao`, `TGRa` | D (tỷ lệ lấp đầy) |
| `HoaDon` (D) | `MaHD`, `MaHV`, `NgayLap`, `TongTien` | B (cộng chi tiêu/điểm) |

Chữ ký thủ tục B công bố để D gọi (chốt ở bước 2.3): `sp_CapNhatDiemHang(@MaHV, @MaHD, @TongTien, @NgayLap)`.

Khóa chung: `MaCN`, `MaNV` (A) · `MaHV`, `MaGoi` (B) · `MaHD` (D).

---

# GIAI ĐOẠN 1 — Phân tích, mức quan niệm & logic

## Bước 1.0 — Chốt chung _(cả nhóm)_
- Chốt hợp đồng giao diện ở trên và [7 quyết định](quyet-dinh.md) (hỏi cô chỗ chưa chắc, nhất là #4).
- Chọn công cụ theo dõi tiến độ (GitHub Issues), web hay winform, ngôn ngữ, phiên bản SQL Server.
- **Xong khi:** hợp đồng không còn ô trống, `quyet-dinh.md` có cột "Chốt" điền đủ.

## Bước 1.1 — Mô tả quy trình & bảng tần suất
Mỗi người viết cho module mình. Quy trình viết cụ thể hơn đề: ai làm, theo thứ tự nào, khi nào thất bại. Bảng tần suất: chức năng – mô tả – tần suất (lần/ngày, giờ…) – mức độ (cao/TB/thấp).

| Người | Ví dụ mẫu | Phụ thuộc |
|---|---|---|
| **A** | _Điều chuyển PT:_ quản lý chi nhánh gửi yêu cầu → hệ thống đóng bản ghi công tác cũ (ghi ngày kết thúc) và mở bản ghi mới. ~10 lần/tháng, **thấp**. _Tính hoa hồng tháng:_ 1 lần/tháng/chi nhánh, **thấp** nhưng truy vấn nặng. | **Không phụ thuộc ai.** Buổi dạy PT lấy từ `LichHenPT` (cột đã chốt ở 1.0) |
| **B** | _Mua gói:_ hội viên thanh toán → tạo `GoiHoiVien` với ngày hết hạn = ngày bắt đầu + thời hạn gói. ~200 lần/ngày, **trung bình**. _Xét hạng:_ hiện chạy mỗi khi có hóa đơn (hiện trạng của đề). | **Không phụ thuộc ai.** Hóa đơn lấy từ `HoaDon` (cột đã chốt) |
| **C** | _Check-in:_ quét QR → kiểm tra gói còn hạn → ghi log vào/ra. ~10.000 lượt trong khung 17h–20h (số của đề), **cao nhất hệ thống**. _Đặt lịch PT:_ chọn PT, ngày giờ → tạo lịch "Đã đặt". | **Không phụ thuộc ai.** Gói còn hạn lấy từ `GoiHoiVien` (cột đã chốt) |
| **D** | _Lập hóa đơn:_ nhân viên chọn gói/sản phẩm → tính tổng, giảm giá → chọn hình thức thanh toán. ~1.000 lần/ngày, **cao**. _Báo cáo doanh thu tháng:_ 1 lần/ngày/chi nhánh, **thấp** nhưng nặng. | **Không phụ thuộc ai** |

**Xong khi:** mỗi người có `docs/gd1/<module>/quy-trinh.md` và bảng tần suất.

## Bước 1.2 — ERD module & ràng buộc quan niệm
Thực thể của module khác vẽ thành hình chữ nhật chỉ có các cột trong hợp đồng (ví dụ C vẽ `HoiVien(MaHV)`, không vẽ các thuộc tính còn lại).

| Người | Ví dụ mẫu (ERD) | Ràng buộc ghi bằng chữ | Phụ thuộc |
|---|---|---|---|
| **A** | `NhanVien` 1–n `LichSuCongTac` n–1 `ChiNhanh`; `LichSuCongTac` có `NgayBatDau`, `NgayKetThuc` | Các khoảng công tác của cùng một nhân viên không được chồng nhau | **Không phụ thuộc ai** |
| **B** | `HoiVien` 1–n `GoiHoiVien` n–1 `GoiDichVu`; `HoiVien` có hạng và điểm | Ngày hết hạn gói = ngày bắt đầu + thời hạn của gói | **Không phụ thuộc ai** |
| **C** | `HoiVien` 1–n `CheckIn` n–1 `ChiNhanh`; `LichHenPT` nối `HoiVien`, PT (`NhanVien`), `ChiNhanh` | Một PT không có 2 lịch chưa hủy cùng ngày giờ; nhân viên trong lịch hẹn phải là PT | **Không phụ thuộc ai** |
| **D** | `HoaDon` 1–n `ChiTietHoaDon` n–1 `GoiDichVu`/`SanPham`; `SanPham` n–n `ChiNhanh` qua tồn kho | Tổng tiền hóa đơn = tổng thành tiền các dòng − giảm giá | **Không phụ thuộc ai** |

**Điểm ghép — Ghép ERD tổng _(cả nhóm, lần đầu làm chung)_:** gộp 4 ERD, kiểm tra khóa chung khớp tên và kiểu, loại thực thể trùng. Review chéo. **Cả nhóm chờ đủ 4 ERD** mới ghép.

## Bước 1.3 — Lược đồ quan hệ, khóa, phụ thuộc hàm, dạng chuẩn
Mỗi quan hệ: khóa – tập phụ thuộc hàm – dạng chuẩn (chuẩn hóa nếu chưa đạt) – mô tả thuộc tính. Khóa ngoại sang bảng của người khác chỉ cần khai tên cột trong hợp đồng.

| Người | Ví dụ mẫu | Phụ thuộc |
|---|---|---|
| **A** | `LichSuCongTac(MaLS, MaNV, MaCN, NgayBatDau, NgayKetThuc, ChucVu)`. Khóa `MaLS`. PTH: `MaLS → MaNV, MaCN, NgayBatDau, NgayKetThuc, ChucVu`. BCNF. `ChucVu` ở đây là chức vụ **tại giai đoạn đó**; `NhanVien.ChucVu` là chức vụ **hiện tại**. | **Không phụ thuộc ai** (cần ERD tổng đã ghép từ bước 1.2) |
| **B** | `GoiHoiVien(MaGHV, MaHV, MaGoi, NgayBatDau, NgayHetHan, SoBuoiConLai)`. PTH: `MaGHV → …`. Nếu thêm cột `TenGoi` thì có `MaGoi → TenGoi` mà `MaGoi` không khóa → phải bỏ (nằm ở `GoiDichVu`). | **Không phụ thuộc ai** |
| **C** | `CuaKiemSoat(MaCN, SoHieuCua, ChucNang, DangHoatDong)` có khóa ghép `(MaCN, SoHieuCua)`. `CheckIn(MaCheckIn, MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa)` tham chiếu cửa bằng khóa ghép. Mỗi quan hệ đạt BCNF. | **Không phụ thuộc ai** — `MaHV` là khóa ngoại sang bảng của B, `MaCN` lấy từ A; chỉ cần các cột đã chốt, **không chờ A/B xong lược đồ** |
| **D** | `ChiTietHoaDon(MaHD, STT, LoaiDong, MaGoi, MaSP, SoLuong, DonGia, ThanhTien)`. `ThanhTien = SoLuong × DonGia` là thuộc tính dẫn xuất — nhóm quyết định giữ (nhanh) hay bỏ (không dư thừa) và ghi lý do. | **Không phụ thuộc ai** |

## Bước 1.4 — Ràng buộc mức logic
Theo mẫu: **bối cảnh – nội dung – bảng tầm ảnh hưởng** (dấu `+` nếu Thêm/Xóa/Sửa có thể vi phạm).

Ví dụ mẫu (C): bối cảnh `LichHenPT`; nội dung: không tồn tại 2 lịch cùng `MaNV`, `NgayGio` mà cả hai chưa hủy.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LichHenPT | + | − | + (`MaNV`, `NgayGio`, `TrangThai`) |

| Người | Ràng buộc nên có (tối thiểu 3 cái mỗi người) | Phụ thuộc |
|---|---|---|
| **A** | Khoảng công tác không chồng; nhân viên điều chuyển phải có chi nhánh đến khác chi nhánh đi; `NhanVien.ChucVu` bằng `ChucVu` của giai đoạn công tác hiện hành | **Không phụ thuộc ai** |
| **B** | Ngày hết hạn đúng; số buổi còn lại ≥ 0; hạng phải khớp tổng chi tiêu | **Không phụ thuộc ai** |
| **C** | Không trùng lịch PT; check-in chỉ hợp lệ khi hội viên có gói còn hạn (ràng buộc **liên module** — bảng `GoiHoiVien` của B cũng có trong bảng tầm ảnh hưởng) | Ràng buộc liên module: ghi thêm bảng của B bị ảnh hưởng, **báo B ở điểm ghép**, không cần chờ |
| **D** | Tổng tiền đúng; tồn kho không âm; hóa đơn phải có ít nhất 1 dòng | **Không phụ thuộc ai** |

**Điểm ghép — Lược đồ tổng _(cả nhóm)_:** gộp lược đồ, kiểm tra khóa ngoại giữa module; với mỗi ràng buộc liên module, chủ bảng bị ảnh hưởng xác nhận. **Kết luận dạng chuẩn của cả lược đồ.** Hoàn thiện tài liệu GĐ1.

---

# GIAI ĐOẠN 2 — Mức vật lý, cài đặt, ứng dụng

## Bước 2.0 — Khung CSDL
Mỗi người viết script tạo bảng module mình (`sql/01_tables_<module>.sql`). Bảng có khóa ngoại sang module khác: **viết thì độc lập, chạy thì theo thứ tự** `A → B → C và D`.

| Người | Việc | Phụ thuộc |
|---|---|---|
| **A** | Script `ChiNhanh`, `NhanVien`, `LichSuCongTac` (chạy đầu tiên) | **Không phụ thuộc ai** |
| **B** | Script `HoiVien`, `GoiDichVu`, `GoiHoiVien`, bảng hạng/điểm | Khi **chạy**: chờ A xong (khóa ngoại sang `ChiNhanh`/`NhanVien` nếu có). Khi **viết**: không chờ |
| **C** | Script `CheckIn`, `LichHenPT`, `DanhGiaHLV` | Khi chạy: chờ A và B xong. Khi viết: không chờ |
| **D** | Script `SanPham`, tồn kho, `HoaDon`, `ChiTietHoaDon` | Khi chạy: chờ A và B xong. Khi viết: không chờ |

**Điểm ghép:** gộp, chạy được từ CSDL trống trên SQL Server, một lệnh tạo xong toàn bộ bảng.

## Bước 2.1 — Kịch bản (có cả đọc và ghi)
Mỗi người đề xuất 2–3 kịch bản; cả nhóm gộp thành **8–10 kịch bản liên quan nhau**.

| Người | Ví dụ mẫu | Phụ thuộc |
|---|---|---|
| **A** | _KB-A1:_ tính hoa hồng PT tháng 10 cho PT chuyển chi nhánh ngày 15 (đọc nặng). _KB-A2:_ điều chuyển PT sang chi nhánh khác (ghi). | **Không phụ thuộc ai** khi viết |
| **B** | _KB-B1:_ thanh toán hóa đơn → cập nhật điểm/hạng (ghi). _KB-B2:_ thống kê phân bổ hạng (đọc). | **Không phụ thuộc ai** khi viết |
| **C** | _KB-C1:_ quét QR lúc 19h, kiểm tra gói + ghi log (đọc + ghi dồn dập). _KB-C2:_ hai hội viên cùng đặt một PT một khung giờ (ghi đồng thời). | **Không phụ thuộc ai** khi viết |
| **D** | _KB-D1:_ lập hóa đơn gồm gói + sản phẩm (ghi). _KB-D2:_ báo cáo doanh thu theo tháng / tỷ lệ lấp đầy theo giờ (đọc, bảng lớn). | **Không phụ thuộc ai** khi viết |

**Điểm ghép _(cả nhóm)_:** nối kịch bản thành chuỗi liên quan nhau: mua gói (D lập hóa đơn → B cập nhật hạng) → check-in (C) → đặt PT (C) → PT dạy xong, tính hoa hồng (A) → quản lý xem doanh thu (D). Chọn ra 8–10 kịch bản.

## Bước 2.2 — Ma trận tham chiếu & mẫu phân tích truy vấn
- **Ma trận tham chiếu:** mỗi truy vấn đọc (R)/ghi (W) những bảng nào, kèm tần suất → nhận xét bảng nào nóng.
- **Mẫu phân tích truy vấn:** câu lệnh – bảng – điều kiện lọc – cột trả về – tần suất → nhận xét.

Ví dụ mẫu (C, xem khung giờ trống của PT):

| Mục | Giá trị |
|---|---|
| Câu lệnh | `SELECT NgayGio FROM LichHenPT WHERE MaNV=@pt AND NgayGio>=@ngay AND NgayGio<DATEADD(day,1,@ngay) AND TrangThai<>N'Hủy'` |
| Điều kiện lọc | `MaNV` (=), `NgayGio` (khoảng) |
| Tần suất | ~15.000 lần/ngày |
| Nhận xét | Chưa có index → quét toàn bảng; ứng viên index `(MaNV, NgayGio)` |

**Phụ thuộc:** cả 4 người **không phụ thuộc ai**. Truy vấn có dùng bảng của người khác thì ghi bảng đó ở cột R/W, chỉ cần các cột trong hợp đồng.

## Bước 2.3 — Đề xuất giải pháp & viết script

| Người | Giải pháp gợi ý (ví dụ mẫu) | Phụ thuộc |
|---|---|---|
| **A** | Index `(MaNV, NgayBatDau)` trên `LichSuCongTac`; truy vấn dùng `NgayDay BETWEEN NgayBatDau AND ISNULL(NgayKetThuc,'9999-12-31')` để xác định chi nhánh tại ngày dạy; stored procedure tính hoa hồng | **Viết thì không chờ ai** (dùng `LichHenPT(MaNV, MaCN, NgayGio, TrangThai)` đã chốt). **Chạy thử** cần dữ liệu `LichHenPT` của C → sau bước 2.4 |
| **B** | Lưu sẵn `TongChiTieu12T`, `Diem`, `Hang` trong bảng hội viên; thủ tục `sp_CapNhatDiemHang` cộng dồn khi có hóa đơn (không cộng lại 12 tháng); job đêm trừ hóa đơn quá 12 tháng | **Việc đầu tiên của B ở bước này: công bố chữ ký thủ tục** (D cần gọi). Phần còn lại không chờ ai |
| **C** | Index `(MaHV, NgayHetHan)` trên gói hội viên để check-in < 50ms; **partition `CheckIn` theo tháng** (phục vụ cả vấn đề 1 lẫn 3); unique filtered index `(MaNV, NgayGio) WHERE TrangThai<>N'Hủy'` chặn overbooking | **Không phụ thuộc ai** khi viết. Index trên `GoiHoiVien` là bảng của B: C đề xuất, **B tạo index trong script của B** |
| **D** | Partition `HoaDon` theo tháng; bảng tổng hợp doanh thu theo ngày/chi nhánh và tỷ lệ lấp đầy theo giờ (đọc từ `CheckIn` của C), cập nhật bằng trigger hoặc job | Khi viết: chờ B công bố chữ ký thủ tục (để gọi sau khi lập hóa đơn) — nếu B chưa xong, D gọi một thủ tục rỗng cùng chữ ký. **D không partition `CheckIn`** — việc đó thuộc C, D chỉ đọc nó |

**Xong khi:** mỗi người có script trong `sql/` + file giải thích lý do chọn giải pháp ở `docs/gd2/<module>/`.

## Bước 2.4 — Sinh dữ liệu
Chọn công cụ sinh hoặc tự code. **≥ 70.000 dòng/bảng** cho bảng cần minh họa. Sinh theo thứ tự khóa ngoại nên có chờ nhau:

| Người | Sinh dữ liệu cho | Phụ thuộc |
|---|---|---|
| **A** | `ChiNhanh` (12), `NhanVien`, `LichSuCongTac` | **Làm đầu tiên, không chờ ai** |
| **B** | `HoiVien`, `GoiDichVu`, `GoiHoiVien` | **Chờ A** (nếu bảng có khóa ngoại sang `ChiNhanh`/`NhanVien`) |
| **C** | `CheckIn`, `LichHenPT` | **Chờ A và B xong**: cần `HoiVien`, `NhanVien`, `LichSuCongTac` để PT của buổi hẹn thuộc đúng chi nhánh tại ngày đó |
| **D** | `SanPham`, tồn kho, `HoaDon`, `ChiTietHoaDon` | **Chờ A và B xong**: cần `HoiVien`, `NhanVien`, `GoiDichVu` |

Cách làm: A và B xong thì commit file dữ liệu/script sinh; C và D bắt đầu cùng lúc (hai người này không chờ nhau). Kiểm tra toàn vẹn khóa ngoại sau khi sinh.

## Bước 2.5 — Đo hiệu năng & phân tích kết quả
Mỗi người chạy kịch bản của mình **có và không có** giải pháp, chụp hình, giải thích.

Ví dụ mẫu (C, xem lịch PT): bật `SET STATISTICS IO, TIME ON`, chạy trước và sau khi tạo index, chụp execution plan.

| | Không có index | Có index |
|---|---|---|
| Plan | Scan | Seek |
| Logical reads | _(đo thật)_ | _(đo thật)_ |
| Thời gian | _(đo thật)_ | _(đo thật)_ |

Ảnh vào `docs/gd2/<module>/img/`. Con số phải đo trên dữ liệu thật của nhóm.

**Phụ thuộc:** sau khi bước 2.4 xong thì cả 4 người **không phụ thuộc ai**. Lưu ý D đo báo cáo lấp đầy cần `CheckIn` đã có dữ liệu (C đã sinh ở 2.4); A đo hoa hồng cần `LichHenPT` (C đã sinh). Nên **không đo cùng lúc trên cùng một máy** (số đo sẽ lệch nhau); chia lượt hoặc mỗi người dùng một bản sao CSDL.

## Bước 2.6 — Ứng dụng

| Người | Việc | Phụ thuộc |
|---|---|---|
| **A** | **Việc đầu tiên: dựng khung ứng dụng** (kết nối CSDL, menu, form mẫu), rồi làm màn hình điều chuyển nhân viên, bảng hoa hồng PT | Không chờ ai |
| **B** | Màn hình đăng ký gói, xem hạng/điểm, thống kê phân bổ hạng | Thiết kế màn hình và gọi thủ tục không chờ ai; **cắm vào ứng dụng thì chờ khung của A** |
| **C** | Màn hình check-in, đặt lịch PT (demo hai cửa sổ cùng đặt → một bên báo lỗi) | Như B: chờ khung của A để cắm vào |
| **D** | Màn hình lập hóa đơn, báo cáo doanh thu, tồn kho, tỷ lệ lấp đầy | Như B; lập hóa đơn gọi `sp_CapNhatDiemHang` của B (thủ tục đã có từ 2.3) |

**Điểm ghép — Ghép ứng dụng _(cả nhóm)_:** chạy thử chuỗi kịch bản ở bước 2.1 trên ứng dụng.

## Bước 2.7 — Báo cáo & nộp
- Mỗi người viết phần module mình: phân tích → thiết kế → cài đặt → đánh giá. **Không phụ thuộc ai.**
- Thông tin cá nhân, thông tin nhóm, **đánh giá cá nhân và nhóm**.
- **Một người ghép** báo cáo cuối (chờ đủ 4 phần), rà lại `de.md` và `required.md`, nộp Moodle.

---

## Review vòng tròn

Ở mỗi điểm ghép: A review B → B review C → C review D → D review A, qua Pull Request. Người review ghi lỗi, chỗ lệch cột trong hợp đồng, chỗ không hiểu.

## Quy ước repo

- Mỗi người làm trên nhánh riêng (`a/...`, `b/...`), merge vào `main` qua PR có ít nhất 1 người review.
- Tài liệu: `docs/gd1/<module>/`, `docs/gd2/<module>/` (`a`, `b`, `c`, `d`).
- Script SQL Server: `sql/` — tên file có số thứ tự chạy, ví dụ `sql/01_tables_a.sql`.
- Ảnh đo hiệu năng: `docs/gd2/<module>/img/`. Ứng dụng: `app/`.
