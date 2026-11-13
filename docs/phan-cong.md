# Phân công nhóm (2 giai đoạn)

Mỗi người **sở hữu một module** xuyên suốt cả 2 giai đoạn. Trong mỗi giai đoạn, cả nhóm đi qua cùng một chuỗi **bước**; mỗi bước mỗi người làm phần của module mình, rồi có **điểm ghép** trước khi sang bước sau. Không chia theo tuần — xong bước nào sang bước đó.

Mẫu nội dung từng phần theo [required.md](../required.md).

**Cách để ít phụ thuộc nhất:** ở bước 1.0 cả nhóm chốt một **"hợp đồng giao diện"** (bảng bên dưới): mỗi module công bố sẵn những cột mà module khác sẽ dùng. Sau đó ai cũng làm độc lập, chỉ dùng đúng những cột đã chốt, không cần chờ nhau. Ở mỗi bước dưới đây, cột **Phụ thuộc** ghi rõ "không phụ thuộc ai" hoặc "chờ ai làm gì".

## Module

> Cập nhật 13/11/2026: toàn bộ bảng dưới đây dùng tên quan hệ và cột của [lược đồ logic](gd1/luoc-do-logic.png), [thiết kế tổng](gd1/thiet-ke-tong.md) và [quyết định](quyet-dinh.md) hiện hành.

| Người | Thành viên | Module | Quan hệ sở hữu | Vấn đề của đề phải giải |
|---|---|---|---|---|
| **A** | Nguyễn Phúc Hậu (23120252) | Chi nhánh & Nhân sự | `CHI_NHANH`, `NHAN_VIEN`, `LICH_SU_CONG_TAC`, `HOA_HONG_CHI_NHANH` | 5 — lịch sử điều chuyển & hoa hồng |
| **B** | Trương Nhật Đạt (23120231) | Hội viên, Gói & Thân thiết | `HOI_VIEN`, `HANG_MUC_CUNG_CAP` (loại gói), `GOI_HOI_VIEN` | 4 — tính lại hạng quá tải |
| **C** | Ngô Gia An (23120205) | Check-in & Lịch hẹn PT | `CHECK_IN`, `LICH_HEN_PT`, `DANH_GIA_HLV` | 1 — nghẽn check-in (< 50ms); 2 — overbooking PT |
| **D** | Phạm Anh Kiệt (24120197) | Sản phẩm, Hóa đơn & Báo cáo | `HANG_MUC_CUNG_CAP` (loại Sản phẩm), `TON_KHO`, `HOA_DON`, `CHI_TIET_HOA_DON` | 3 — log lớn & báo cáo chậm |

## Hợp đồng giao diện

Cột mỗi module công bố cho module khác dùng. Đổi tên/kiểu hoặc xóa cột trong hợp đồng thì báo người dùng nó.

| Bảng (chủ) | Cột công bố | Ai dùng, để làm gì |
|---|---|---|
| `CHI_NHANH` (A) | `MaCN`, `GioMoCua`, `GioDongCua`, `SucChua` | B: chi nhánh đăng ký; C: giờ mở cửa cho lịch PT, đóng phiên cuối ngày; D: sức chứa cho báo cáo lấp đầy |
| `NHAN_VIEN` (A) | `MaNV`, `ChucVu`, `MaCNHienTai` | C: nhân viên của lịch phải là PT; D: chi nhánh lập hóa đơn |
| `LICH_SU_CONG_TAC` (A) | `MaNV`, `MaCN`, `NgayBatDau`, `NgayKetThuc` | C: kiểm tra PT đúng chi nhánh tại ngày tập |
| `HOI_VIEN` (B) | `MaHV` | C, D |
| `HANG_MUC_CUNG_CAP` (B, D) | `MaHM`, `LoaiHangMuc`, `Gia`, `ThoiHanThang`, `SoBuoiBanDau` | B tạo gói; C kiểm tra loại gói; D lập hóa đơn |
| `GOI_HOI_VIEN` (B) | `MaGHV`, `MaHV`, `MaHM`, `NgayKichHoat`, `NgayHetHan`, `SoBuoiConLai` | C: quét cổng, đặt lịch, trừ vé/buổi |
| `LICH_HEN_PT` (C) | `MaLH`, `MaNV`, `MaCN`, `NgayTap`, `ThoiGianBatDau`, `TrangThai` | A: tính hoa hồng, kiểm tra lịch vướng |
| `CHECK_IN` (C) | `MaCN`, `ThoiGianVao`, `ThoiGianRa` | B: lâu ngày chưa quay lại; D: báo cáo lấp đầy |
| `HOA_DON`, `CHI_TIET_HOA_DON` (D) | `MaHD`, `MaHV`, `NgayLap`, `TongTien`; `MaHM`, `SoLuong` | B: tạo gói, cộng điểm/hạng trong cùng giao dịch thanh toán |

Chữ ký thủ tục B công bố để D gọi (chốt ở bước 2.3): `sp_CapNhatDiemHang(@MaHV, @MaHD, @TongTien, @NgayLap)`.

Khóa chung: `MaCN`, `MaNV` (A) · `MaHV`, `MaHM`, `MaGHV` (B) · `MaHD` (D).

---

# GIAI ĐOẠN 1 — Phân tích, mức quan niệm & logic

## Bước 1.0 — Chốt chung _(cả nhóm)_
- Chốt hợp đồng giao diện ở trên và [28 quyết định](quyet-dinh.md).
- Chọn công cụ theo dõi tiến độ (GitHub Issues), web hay winform, ngôn ngữ, phiên bản SQL Server.
- **Xong khi:** hợp đồng không còn ô trống, `quyet-dinh.md` đã chốt đủ.

## Bước 1.1 — Mô tả quy trình & bảng tần suất
Mỗi người viết cho module mình. Quy trình viết cụ thể hơn đề: ai làm, theo thứ tự nào, khi nào thất bại. Bảng tần suất: chức năng – mô tả – tần suất – mức độ. Quy ước chung toàn hệ thống (xem `quyet-dinh.md`): thấp <100 lần/ngày, trung bình 100–999, cao >=1.000; quy đổi tuần/tháng và tách số dòng/độ nặng.

| Người | Ví dụ mẫu | Phụ thuộc |
|---|---|---|
| **A** | _Điều chuyển nhân viên:_ quản lý nhân sự gửi yêu cầu → hệ thống đóng giai đoạn công tác cũ (ghi ngày kết thúc) và mở giai đoạn mới. Trước khi ghi, chặn nếu còn lịch PT chưa hủy không phù hợp sau thay đổi; dùng dữ liệu C. ~10 lần/tháng, **thấp**. _Tính hoa hồng tháng:_ ~2 lần/tháng/chi nhánh, **thấp** nhưng truy vấn nặng. | **Không phụ thuộc ai.** Buổi dạy PT lấy từ `LICH_HEN_PT` (cột đã chốt ở 1.0) |
| **B** | _Mua gói:_ hội viên thanh toán → tạo `GOI_HOI_VIEN` bắt đầu ngày mua; hết hạn = ngày mua + thời hạn tháng − 1 ngày (vé lượt: hạn 7 ngày). ~390 lần/ngày, **trung bình**. _Cộng điểm/hạng khi thanh toán:_ mỗi hóa đơn cộng một lần vào `HOI_VIEN`, ~1.400 lần/ngày, **cao** (vấn đề 4). | **Không phụ thuộc ai.** Hóa đơn lấy từ `HOA_DON` (cột đã chốt) |
| **C** | _Check-in:_ quét mã QR của gói → kiểm tra gói còn hạn, không phải PT → ghi log vào/ra. Đề ghi >10.000 lượt quét đồng thời toàn hệ thống trong 17h–20h; không đồng nhất với số lượt cộng dồn. Tần suất minh họa và các đơn vị tải xem bảng giả định chung. _Đặt lịch PT:_ chọn PT, ngày giờ → tạo lịch Đã đặt. | **Không phụ thuộc ai.** Gói lấy từ `GOI_HOI_VIEN` (cột đã chốt); gói PT không mở cổng |
| **D** | _Lập hóa đơn:_ nhân viên chọn gói/sản phẩm → tính tổng, giảm giá → chọn hình thức thanh toán. ~1.400 lần/ngày, **cao**. _Báo cáo doanh thu:_ 12 lần/ngày (mỗi chi nhánh một lần), **thấp** nhưng nặng. | **Không phụ thuộc ai** |

**Xong khi:** mỗi người có `docs/gd1/<module>/quy-trinh.md` và bảng tần suất.

## Bước 1.2 — ERD module & ràng buộc quan niệm
Thực thể của module khác vẽ thành hình chữ nhật chỉ có các thuộc tính cần dùng (ví dụ C vẽ `GÓI HỘI VIÊN`, không vẽ các thuộc tính còn lại của hội viên).

| Người | Ví dụ mẫu (ERD) | Ràng buộc ghi bằng chữ | Phụ thuộc |
|---|---|---|---|
| **A** | `NHÂN VIÊN` 1–n `CÔNG TÁC` n–1 `CHI NHÁNH`; `CÔNG TÁC` có ngày bắt đầu, ngày kết thúc | Các khoảng công tác của cùng một nhân viên không được chồng nhau | **Không phụ thuộc ai** |
| **B** | `HỘI VIÊN` 1–n `GÓI HỘI VIÊN` n–1 `GÓI DỊCH VỤ`; `HỘI VIÊN` có tổng chi tiêu, điểm, hạng | Gói bắt đầu ngày mua; ngày hết hạn = ngày mua + thời hạn tháng − 1 ngày | **Không phụ thuộc ai** |
| **C** | `GÓI HỘI VIÊN` 1–n `CHECK IN` n–1 `CHI NHÁNH`; `LỊCH HẸN PT` nối `GÓI HỘI VIÊN`, `NHÂN VIÊN`, `CHI NHÁNH` | Một PT không có 2 lịch chưa hủy cùng ngày giờ; nhân viên trong lịch hẹn phải có chức vụ PT | **Không phụ thuộc ai** |
| **D** | `HÓA ĐƠN` n–n `HẠNG MỤC CUNG CẤP` qua mối "chứa"; `SẢN PHẨM` n–n `CHI NHÁNH` qua tồn kho | Tổng tiền hóa đơn = Σ số lượng × đơn giá − giảm giá | **Không phụ thuộc ai** |

**Điểm ghép — Ghép ERD tổng _(cả nhóm, lần đầu làm chung)_:** gộp 4 ERD, kiểm tra khóa chung khớp tên và kiểu, loại thực thể trùng. Review chéo. **Cả nhóm chờ đủ 4 ERD** mới ghép.

## Bước 1.3 — Lược đồ quan hệ, khóa, phụ thuộc hàm, dạng chuẩn
Mỗi quan hệ: khóa – tập phụ thuộc hàm – dạng chuẩn (chuẩn hóa nếu chưa đạt) – mô tả thuộc tính. Khóa ngoại sang bảng của người khác chỉ cần khai tên cột trong hợp đồng.

| Người | Ví dụ mẫu | Phụ thuộc |
|---|---|---|
| **A** | `LICH_SU_CONG_TAC(MaLSCT, MaNV, MaCN, NgayBatDau, NgayKetThuc)`. Khóa `MaLSCT`; AK `(MaNV, NgayBatDau)`. PTH: `MaLSCT → MaNV, MaCN, NgayBatDau, NgayKetThuc`. BCNF. Chức vụ cố định nên chỉ lưu ở `NHAN_VIEN.ChucVu`. | **Không phụ thuộc ai** (cần ERD tổng đã ghép từ bước 1.2) |
| **B** | `GOI_HOI_VIEN(MaGHV, MaHV, MaHM, NgayKichHoat, NgayHetHan, SoBuoiConLai)`. PK `MaGHV`; BCNF. `HOI_VIEN` đạt 2NF vì `TongChiTieu → Hang` (cố ý lưu sẵn hạng để giải vấn đề 4). | **Không phụ thuộc ai** khi thiết kế; đối chiếu khóa ngoại với D |
| **C** | `CHECK_IN(MaCI, MaGHV, MaCN, ThoiGianVao, ThoiGianRa, ThietBiVao, ThietBiRa)` đạt BCNF. `LICH_HEN_PT` đạt 2NF vì `{MaNV, NgayTap} → MaCN` (cố ý giữ `MaCN`). | **Không phụ thuộc ai** — `MaGHV` là khóa ngoại sang bảng của B, `MaCN` lấy từ A; chỉ cần các cột đã chốt, **không chờ A/B xong lược đồ** |
| **D** | `CHI_TIET_HOA_DON(MaCTHD, MaHD, MaHM, SoLuong, DonGia)`; AK `(MaHD, MaHM)`. Không lưu thành tiền, tính `SoLuong × DonGia` khi đọc. `HOA_DON` đạt 2NF vì `{MaNV, NgayLap} → MaCN` (cố ý giữ `MaCN`). | **Không phụ thuộc ai** |

## Bước 1.4 — Ràng buộc mức logic
Theo mẫu: **bối cảnh – nội dung – bảng tầm ảnh hưởng** (dấu `+` nếu Thêm/Xóa/Sửa có thể vi phạm).

Ví dụ mẫu (C): bối cảnh `LICH_HEN_PT`; nội dung: không tồn tại 2 lịch cùng `MaNV`, `NgayTap`, `ThoiGianBatDau` mà cả hai `TrangThai` khác Hủy (R16 trong thiết kế tổng).

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (`MaNV`, `NgayTap`, `ThoiGianBatDau`, `TrangThai`) |

| Người | Ràng buộc nên có (tối thiểu 3 cái mỗi người) | Phụ thuộc |
|---|---|---|
| **A** | Khoảng công tác không chồng; nhân viên điều chuyển phải có chi nhánh đến khác chi nhánh đi; `NHAN_VIEN.MaCNHienTai` đồng bộ từ giai đoạn bao phủ ngày hiện tại; bấm điều chuyển ngày D thì lịch sử cũ kết thúc D, mới bắt đầu D+1; từ chối điều chuyển nếu lịch chưa hủy mất điều kiện chi nhánh, đặt lịch và điều chuyển phải được kiểm tra nhất quán | **Không phụ thuộc ai** |
| **B** | Ngày hết hạn đúng; số buổi còn lại ≥ 0; hạng phải khớp tổng chi tiêu | **Không phụ thuộc ai** |
| **C** | Không trùng lịch PT; check-in chỉ hợp lệ khi gói còn hạn, còn lượt (nếu có giới hạn) và không phải gói PT; tới giờ hẹn lịch tự hoàn thành và trừ buổi (ràng buộc **liên module** — bảng `GOI_HOI_VIEN` của B cũng có trong bảng tầm ảnh hưởng) | Ràng buộc liên module: ghi thêm bảng của B bị ảnh hưởng, **báo B ở điểm ghép**, không cần chờ |
| **D** | Tổng tiền đúng; tồn kho không âm; hóa đơn phải có ít nhất 1 dòng | **Không phụ thuộc ai** |

**Điểm ghép — Lược đồ tổng _(cả nhóm)_:** gộp lược đồ, kiểm tra khóa ngoại giữa module; với mỗi ràng buộc liên module, chủ bảng bị ảnh hưởng xác nhận. **Kết luận dạng chuẩn của cả lược đồ.** Hoàn thiện tài liệu GĐ1.

---

# GIAI ĐOẠN 2 — Mức vật lý, cài đặt, ứng dụng

## Bước 2.0 — Khung CSDL
Mỗi người viết script tạo bảng module mình (`sql/01_tables_<module>.sql`). Bảng có khóa ngoại sang module khác: **viết thì độc lập, chạy thì theo thứ tự** `A → B → C và D`.

| Người | Việc | Phụ thuộc |
|---|---|---|
| **A** | Script `CHI_NHANH`, `NHAN_VIEN`, `LICH_SU_CONG_TAC`, `HOA_HONG_CHI_NHANH` (chạy đầu tiên) | **Không phụ thuộc ai** |
| **B** | Script `HOI_VIEN`, `HANG_MUC_CUNG_CAP`, `GOI_HOI_VIEN` | Khi **chạy**: chờ A xong (`HOI_VIEN.MaCN` là khóa ngoại sang `CHI_NHANH`). Khi **viết**: không chờ. B tạo bảng `HANG_MUC_CUNG_CAP`; D chỉ thêm dữ liệu loại Sản phẩm |
| **C** | Script `CHECK_IN`, `LICH_HEN_PT`, `DANH_GIA_HLV` | Khi chạy: chờ A và B xong. Khi viết: không chờ |
| **D** | Script `TON_KHO`, `HOA_DON`, `CHI_TIET_HOA_DON` | Khi chạy: chờ A và B xong. Khi viết: không chờ |

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
| Câu lệnh | `SELECT ThoiGianBatDau FROM LICH_HEN_PT WHERE MaNV=@pt AND NgayTap=@ngay AND TrangThai<>1` |
| Điều kiện lọc | `MaNV` (=), `NgayTap` (=) |
| Tần suất | ~14.500 lần/ngày |
| Nhận xét | Chưa có index → quét toàn bảng; ứng viên index `(MaNV, NgayTap)` |

**Phụ thuộc:** cả 4 người **không phụ thuộc ai**. Truy vấn có dùng bảng của người khác thì ghi bảng đó ở cột R/W, chỉ cần các cột trong hợp đồng.

## Bước 2.3 — Đề xuất giải pháp & viết script

| Người | Giải pháp gợi ý (ví dụ mẫu) | Phụ thuộc |
|---|---|---|
| **A** | Index `(MaNV, NgayBatDau)` trên `LICH_SU_CONG_TAC`; truy vấn dùng `NgayBatDau <= NgayTap AND (NgayKetThuc IS NULL OR NgayTap <= NgayKetThuc)` để xác định chi nhánh tại ngày dạy; stored procedure tính hoa hồng | **Viết thì không chờ ai** (dùng `LICH_HEN_PT(MaNV, MaCN, NgayTap, ThoiGianBatDau, TrangThai)` đã chốt). **Chạy thử** cần dữ liệu `LICH_HEN_PT` của C → sau bước 2.4 |
| **B** | Lưu sẵn `TongChiTieu`, `DiemTichLuy`, `Hang` trong `HOI_VIEN`; thủ tục `sp_CapNhatDiemHang` cộng dồn khi có hóa đơn (không cộng lại cả năm); job 00:00 ngày 1/1 tính lại tổng chi tiêu và hạng | **Việc đầu tiên của B ở bước này: công bố chữ ký thủ tục** (D cần gọi). Phần còn lại không chờ ai |
| **C** | Quét cổng tra `GOI_HOI_VIEN` theo khóa chính `MaGHV`, đo mục tiêu check-in < 50ms; chỉ mục cho tìm phiên đang mở khi quét ra; **partition `CHECK_IN` theo tháng** (phục vụ cả vấn đề 1 lẫn 3); unique filtered index `(MaNV, NgayTap, ThoiGianBatDau) WHERE TrangThai<>1` chặn overbooking | **Không phụ thuộc ai** khi viết. Index trên `GOI_HOI_VIEN` là bảng của B: C đề xuất, **B tạo index trong script của B** |
| **D** | Partition `HOA_DON` theo tháng; bảng tổng hợp doanh thu theo ngày/chi nhánh và tỷ lệ lấp đầy theo giờ (đọc từ `CHECK_IN` của C), cập nhật bằng trigger hoặc job | Khi viết: chờ B công bố chữ ký thủ tục (để gọi sau khi lập hóa đơn) — nếu B chưa xong, D gọi một thủ tục rỗng cùng chữ ký. **D không partition `CHECK_IN`** — việc đó thuộc C, D chỉ đọc nó |

**Xong khi:** mỗi người có script trong `sql/` + file giải thích lý do chọn giải pháp ở `docs/gd2/<module>/`.

## Bước 2.4 — Sinh dữ liệu
Chọn công cụ sinh hoặc tự code. **≥ 70.000 dòng/bảng** cho bảng cần minh họa. Sinh theo thứ tự khóa ngoại nên có chờ nhau:

| Người | Sinh dữ liệu cho | Phụ thuộc |
|---|---|---|
| **A** | `CHI_NHANH` (12), `NHAN_VIEN`, `LICH_SU_CONG_TAC`, `HOA_HONG_CHI_NHANH` | **Làm đầu tiên, không chờ ai** |
| **B** | `HOI_VIEN`, `HANG_MUC_CUNG_CAP`, `GOI_HOI_VIEN` | **Chờ A** (`HOI_VIEN.MaCN` sang `CHI_NHANH`) |
| **C** | `CHECK_IN`, `LICH_HEN_PT`, `DANH_GIA_HLV` | **Chờ A và B xong**: cần `GOI_HOI_VIEN`, `NHAN_VIEN`, `LICH_SU_CONG_TAC` để PT của buổi hẹn thuộc đúng chi nhánh tại ngày đó |
| **D** | `TON_KHO`, `HOA_DON`, `CHI_TIET_HOA_DON` (và các dòng Sản phẩm của `HANG_MUC_CUNG_CAP`) | **Chờ A và B xong**: cần `HOI_VIEN`, `NHAN_VIEN`, `HANG_MUC_CUNG_CAP` |

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
