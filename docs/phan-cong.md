# Phân công nhóm

Mỗi người **sở hữu một module** từ đầu đến lúc nộp. Cả nhóm đi theo **sprint đồng bộ**, cuối mỗi sprint có buổi ghép và review chéo.
Nhiệm vụ chi tiết từng người, từng sprint nằm trong **GitHub Issues** (lọc theo label `module-A`…`module-D` và milestone `Sprint N`).

## Module

| Người | Module | Thực thể chính | Vấn đề của đề phải giải |
|---|---|---|---|
| **A** — _(điền tên)_ | Chi nhánh & Nhân sự | `ChiNhanh`, dịch vụ/tiện ích chi nhánh, `NhanVien`, `LichSuCongTac`, hoa hồng PT | 5 — lịch sử điều chuyển & hoa hồng |
| **B** — _(điền tên)_ | Hội viên, Gói dịch vụ & Thân thiết | `HoiVien`, `GoiDichVu`, gói hội viên đã mua, hạng, lịch sử điểm | 4 — tính lại hạng quá tải |
| **C** — _(điền tên)_ | Check-in & Lịch hẹn PT | `CheckIn`, cửa/thiết bị, `LichHenPT`, `DanhGiaHLV` | 1 — nghẽn check-in (< 50ms); 2 — overbooking PT |
| **D** — _(điền tên)_ | Sản phẩm, Hóa đơn & Báo cáo | `SanPham`, tồn kho theo chi nhánh, `HoaDon`, `ChiTietHoaDon`, thanh toán | 3 — log lớn & báo cáo chậm (phối hợp C cho bảng `CheckIn`) |

## Khóa dùng chung (chốt ở Sprint 0)

Module khác chỉ tham chiếu bằng khóa này, không cần biết bên trong thiết kế thế nào.

| Khóa | Thực thể | Chủ sở hữu |
|---|---|---|
| `MaCN` | Chi nhánh | A |
| `MaNV` | Nhân viên | A |
| `MaHV` | Hội viên | B |
| `MaGoi` | Gói dịch vụ | B |
| `MaHD` | Hóa đơn | D |

Muốn đổi tên/kiểu một khóa chung → mở issue, chủ sở hữu duyệt, báo cả nhóm.

## Sprint (10 tuần)

| Sprint | Tuần | Mỗi người (trong module mình) | Mốc cả nhóm |
|---|---|---|---|
| 0 | 1 | — | Chốt khóa chung, [quyết định](quyet-dinh.md), chọn web/winform |
| 1 | 2–3 | Quy trình nghiệp vụ, bảng tần suất, ERD, ràng buộc | Ghép ERD tổng, review chéo |
| 2 | 4 | Lược đồ quan hệ: khóa, PTH, dạng chuẩn, mô tả thuộc tính, ràng buộc | Ghép lược đồ tổng, kết luận dạng chuẩn |
| 3 | 5–7 | Kịch bản, ma trận tham chiếu, phân tích truy vấn, giải pháp + script | Gộp script CSDL, sinh dữ liệu ≥ 70.000 dòng, chốt 8–10 kịch bản |
| 4 | 8–9 | Đo có/không giải pháp, phân tích kết quả, màn hình ứng dụng | Ghép ứng dụng |
| 5 | 10 | Viết phần báo cáo của mình | Ghép báo cáo, nộp Moodle |

Mẫu nội dung từng phần theo [required.md](../required.md).

## Review vòng tròn

A review B → B review C → C review D → D review A. Review qua Pull Request: ghi lỗi, chỗ lệch khóa ngoại, chỗ không hiểu. Ghép ERD lần đầu làm chung cả nhóm; các lần ghép sau xoay vòng người chủ trì.

## Quy ước repo

- Mỗi người làm trên nhánh riêng (`a/...`, `b/...`), merge vào `main` qua PR có ít nhất 1 người review.
- Tài liệu GĐ1/GĐ2: `docs/gd1/<module>/`, `docs/gd2/<module>/` (module viết thường: `a`, `b`, `c`, `d`).
- Script SQL Server: `sql/` — tên file có số thứ tự chạy, ví dụ `sql/01_tables_a.sql`.
- Ảnh đo hiệu năng: `docs/gd2/<module>/img/`.
- Ứng dụng: `app/`.
