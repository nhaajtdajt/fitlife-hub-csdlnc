# Module A — Chi nhánh và Nhân sự

| Mục | Nội dung |
|---|---|
| Người phụ trách | Nguyễn Phúc Hậu |
| Phạm vi | Chi nhánh, dịch vụ/tiện ích tại chi nhánh, nhân viên, lịch sử công tác, điều chuyển và hoa hồng PT |
| Vấn đề trọng tâm | Xác định đúng chi nhánh của PT tại thời điểm diễn ra buổi dạy để tính hoa hồng chính xác |
| Giai đoạn | Giai đoạn 1 — Phân tích, thiết kế mức quan niệm và logic |

## Tài liệu

- [Mô tả quy trình nghiệp vụ](quy-trinh.md)
- [Bảng phân tích tần suất chức năng](bang-tan-suat.md)
- [ERD và ràng buộc quan niệm](erd.md)
- [Bản vẽ ERD quan niệm Draw.io](erd-a.drawio)
- [Thiết kế logic: lược đồ, khóa, PTH, dạng chuẩn, thuộc tính và ràng buộc](thiet-ke-logic.md)
- [Bản vẽ lược đồ quan hệ Draw.io](luoc-do-quan-he-a.drawio)

`erd.md` và `thiet-ke-logic.md` là tài liệu chữ dùng để ghép báo cáo. Hai file Draw.io là bản vẽ tương ứng, đã đồng bộ cùng bảy quan hệ thuộc Module A; `LichHenPT` chỉ là quan hệ ngoài do Module C sở hữu.

## Phạm vi dữ liệu

- `ChiNhanh`: thông tin chi nhánh, giờ hoạt động và sức chứa.
- Dịch vụ/tiện ích được cung cấp tại từng chi nhánh.
- `NhanVien`: hồ sơ nhân viên, chức vụ và lương cơ bản hiện tại.
- `LichSuCongTac`: chi nhánh và chức vụ theo từng giai đoạn DATE, tính cả ngày bắt đầu/kết thúc. Vị trí mới bắt đầu ngày sau ngày kết thúc cũ; yêu cầu trong ngày có hiệu lực ngày mai.
- `KyHoaHong`, `ChiTietHoaHong`: mức hoa hồng và danh sách buổi nguồn được chụp lại khi chốt từng chi nhánh–tháng.
- Dữ liệu lịch hẹn PT đã hoàn thành do Module C quản lý, được Module A sử dụng khi tính hoa hồng.

## Quyết định và giả định dùng trong thiết kế

- `NhanVien.ChucVu` lưu chức vụ hiện tại; `LichSuCongTac.ChucVu` lưu chức vụ tại từng giai đoạn. Chi nhánh hiện tại được suy ra từ lịch sử, không lưu lặp trong `NhanVien`.
- Khoảng công tác dùng `DATE`, tính cả hai biên. Yêu cầu thay đổi trong ngày D đóng giai đoạn cũ tại D và mở giai đoạn mới từ D+1; dữ liệu hiện tại chỉ đồng bộ khi đến ngày hiệu lực.
- `TienIch` là danh mục tiện ích cơ sở vật chất; `CungCapTienIch` lưu các khoảng một chi nhánh cung cấp tiện ích, không ghi đè lịch sử khi ngừng rồi cung cấp lại.
- PT chỉ dạy tại chi nhánh và với chức vụ PT hợp lệ tại ngày diễn ra buổi. Điều chuyển, đổi chức vụ hoặc nghỉ việc bị từ chối nếu làm lịch PT chưa hủy trở nên không hợp lệ.
- Hoa hồng là mức cố định mỗi buổi hoàn thành theo chi nhánh. Khi chốt, `KyHoaHong` chụp mức hiện tại và `ChiTietHoaHong` chụp danh sách buổi; kỳ cũ không thay đổi khi cấu hình mức mới.
- Các tần suất không có trong đề là ước lượng làm việc của nhóm và được ghi rõ trong bảng tần suất; chúng không phải số liệu đo thực tế.

## Ánh xạ ERD và lược đồ quan hệ

- Phân cấp toàn phần, loại trừ PT/Lễ tân/Quản lý chi nhánh/Nhân viên bán hàng trong ERD quan niệm được ánh xạ sang `NhanVien.ChucVu` ở mức logic; không cần bốn bảng con chỉ chứa mã nhân viên.
- `LichSuCongTac` có PK `MaLS`, AK `(MaNV, NgayBatDau)` và FK tới `NhanVien`, `ChiNhanh`. Khi đổi chức vụ trong cùng chi nhánh vẫn phải tạo giai đoạn mới.
- `CungCapTienIch` dùng PK ghép `(MaCN, MaTI, NgayBatDau)`. `KyHoaHong` dùng PK ghép `(MaCN, Thang)`. `ChiTietHoaHong.MaLichHen` vừa là PK vừa là FK tới `C.LichHenPT`.
- Cả bảy quan hệ của A đạt BCNF theo các phụ thuộc hàm đã công bố. Kết luận toàn CSDL nằm trong thiết kế tổng; không suy rộng kết luận BCNF của A cho toàn hệ thống.

## Đối chiếu yêu cầu GĐ1

| Yêu cầu | Artifact A | Trạng thái |
|---|---|---|
| Quy trình nghiệp vụ chi tiết, gồm tác nhân, điều kiện, luồng, ngoại lệ và kết quả | [quy-trinh.md](quy-trinh.md), QT-A01–QT-A08 | Đủ |
| Chức năng, mô tả, tần suất và mức cao/trung bình/thấp | [bang-tan-suat.md](bang-tan-suat.md), A-F01–A-F13 | Đủ |
| ERD mức quan niệm và các ràng buộc không biểu diễn hết trên hình | [erd-a.drawio](erd-a.drawio), [erd.md](erd.md), A-R01–A-R11 trong thiết kế logic | Đủ |
| Lược đồ quan hệ, PK/FK/AK, PTH và dạng chuẩn | [thiet-ke-logic.md](thiet-ke-logic.md), [luoc-do-quan-he-a.drawio](luoc-do-quan-he-a.drawio) | Đủ |
| Mô tả quan hệ và từng thuộc tính | Mục “Ánh xạ ERD và ý nghĩa thuộc tính” trong [thiet-ke-logic.md](thiet-ke-logic.md) | Đủ |
| Ràng buộc logic theo bối cảnh, nội dung và tầm ảnh hưởng Thêm/Xóa/Sửa | A-R01–A-R11 trong [thiet-ke-logic.md](thiet-ke-logic.md) | Đủ |

Phần thiết kế vật lý, chỉ mục, phân vùng, kịch bản truy vấn và kiểm thử hiệu năng thuộc GĐ2, không nằm trong trạng thái hoàn tất ở trên. Ghi chú ôn vấn đáp cá nhân không phải artifact nộp và không được đưa vào bộ hồ sơ Git.

## Điểm ghép và giới hạn bàn giao

- A cung cấp `ChiNhanh`, `NhanVien` và `LichSuCongTac` theo hợp đồng giao diện; A đọc `LichHenPT` của C để kiểm tra lịch vướng và chốt hoa hồng.
- Các vấn đề liên module còn chờ nhóm xác nhận trong [báo cáo rà soát cuối](../ra-soat-cuoi.md), đặc biệt vòng đời cửa kiểm soát của một chi nhánh và quy tắc nhận quét/đặt lịch ngoài giờ, không làm thiếu bộ artifact nội bộ của A nhưng ảnh hưởng kết luận sẵn sàng nộp của hồ sơ tổng.
- Trước khi nộp qua Git, bảo đảm hai file Draw.io hiện hành đã được theo dõi và có thể mở lại; không đưa ghi chú cá nhân vào commit.

Bản ghép hiện hành: [hồ sơ GĐ1 tổng](../README.md), [ERD tổng](../erd-tong.md), [lược đồ và ràng buộc tổng](../thiet-ke-tong.md).
