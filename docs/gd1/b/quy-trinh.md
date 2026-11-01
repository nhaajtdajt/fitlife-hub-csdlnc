# Module B — Mô tả quy trình nghiệp vụ

Phạm vi: hội viên, gói dịch vụ, gói đã mua, hạng và điểm. Hóa đơn thuộc D, check-in và lịch hẹn PT thuộc C; tài liệu này chỉ dùng các cột của họ đã có trong [hợp đồng giao diện](../../phan-cong.md).

Ký hiệu `#N` là quyết định số N trong [quyet-dinh.md](../../quyet-dinh.md). Các quyết định #9, #11 chưa chốt, tạm dùng theo cột "Đề xuất". Những điểm cần xác nhận được gom ở [cuối tài liệu](#điểm-cần-xác-nhận).

## QT-B1. Đăng ký và cập nhật hội viên

**Ai làm:** lễ tân tại bất kỳ chi nhánh nào.

**Đăng ký:**
1. Hội viên cung cấp họ tên, SĐT, email, CCCD, giới tính, ngày sinh.
2. Lễ tân tìm theo CCCD và SĐT để chắc chắn chưa có hồ sơ.
3. Hệ thống cấp `MaHV`, ghi ngày đăng ký là hôm nay, chi nhánh đăng ký, hạng Silver, điểm 0.

**Cập nhật:** lễ tân tìm hồ sơ theo `MaHV`, SĐT hoặc CCCD rồi sửa thông tin. Hội viên nghỉ tập vẫn giữ hồ sơ vì còn hóa đơn và check-in tham chiếu đến.

**Khi nào thất bại:**
- CCCD đã có hồ sơ: không tạo mới, dùng lại hồ sơ cũ (hội viên cũ quay lại).
- SĐT đã thuộc hồ sơ khác: từ chối, vì SĐT dùng để tra cứu.
- Thiếu thông tin bắt buộc hoặc sai định dạng (CCCD không đủ 12 số, ngày sinh ở tương lai): yêu cầu nhập lại.

## QT-B2. Mua hoặc gia hạn gói

**Ai làm:** lễ tân hoặc nhân viên bán hàng, cùng lúc lập hóa đơn (D).

1. Tìm hội viên; chưa có hồ sơ thì làm QT-B1 trước.
2. Chọn một hoặc nhiều gói đang bán. Một hội viên được giữ nhiều gói cùng lúc, dùng ở mọi chi nhánh (#1).
3. D lập hóa đơn, hội viên thanh toán.
4. Thanh toán thành công thì mỗi gói trên hóa đơn tạo một dòng `GoiHoiVien`, với `NgayBatDau` là ngày mua (#9):

   | Loại gói | `NgayHetHan` | `SoBuoiConLai` |
   |---|---|---|
   | Gym, Yoga | `NgayBatDau` + 3/6/12 tháng | Không áp dụng |
   | PT 1-1 | `NgayBatDau` + thời hạn gói | Số buổi của gói, ví dụ 24 |
   | Vé Spa lẻ | Không có (vé không hết hạn, #8) | 1 |

5. Cập nhật điểm và hạng theo QT-B4.

Gia hạn là mua một gói mới: tạo dòng mới, không sửa dòng cũ, nên lịch sử mua được giữ nguyên.

Gói dùng được đến hết ngày `NgayHetHan`. Ngày cuối tháng lấy theo tháng đích: mua ngày 31/08/2026 gói 6 tháng thì hết hạn ngày 28/02/2027.

**Khi nào thất bại:**
- Thanh toán thất bại: không có hóa đơn, không có gói, không cộng điểm. Các việc này nằm trong cùng một giao dịch.
- Gói đã ngừng bán: không cho chọn.
- Mua gói PT khi không có gói Gym/Yoga còn hạn và cũng không mua kèm trên cùng hóa đơn: từ chối (#8).
- Hóa đơn đã thanh toán thì không sửa, không xóa, không hoàn tiền (#16): gói đã tạo và điểm đã cộng giữ nguyên. Lập sai chỉ sửa được trước khi thanh toán.

## QT-B3. Sử dụng buổi PT và vé Spa

**Ai làm:** luồng xử lý lịch hẹn của C (PT), lễ tân (Spa). B quy định cách trừ.

**Quyền sử dụng (#8):**

| Chức năng | Điều kiện |
|---|---|
| Vào cổng (C) | Có ít nhất một trong ba: gói Gym/Yoga còn hạn; vé Spa còn số lần (không xét hạn); gói PT còn buổi, còn hạn **và** hôm nay có lịch PT. Gói PT không có lịch hôm nay thì không mở cổng |
| Đặt lịch PT (C) | Có gói PT còn buổi và còn hạn tại ngày hẹn, **và** có gói Gym/Yoga còn hạn tại ngày hẹn. Thiếu gói Gym/Yoga thì gia hạn hoặc mua thêm ngay khi đặt (QT-B2), thanh toán xong mới đặt được |
| Dùng Spa (lễ tân) | Khách vào cổng bằng vé Spa: vé đã trừ khi quét cổng, được dùng trong lượt đó. Khách vào bằng gói Gym/Yoga: có vé Spa còn số lần |

Vào cổng đọc `GoiHoiVien` và lịch PT hôm nay của hội viên (`LichHenPT` của C). Đặt lịch PT cần thêm loại gói, vì gói PT và vé Spa đều có `SoBuoiConLai`.

Mua gói PT phải đang có gói Gym/Yoga còn hạn hoặc mua kèm trên cùng hóa đơn (QT-B2). Gói Gym/Yoga có thể hết hạn trước gói PT, nên đặt lịch vẫn phải kiểm tra lại.

**Gói PT 1-1:**
1. Hội viên đặt lịch và chọn HLV từng lần (#2); việc này thuộc C.
2. Quét cổng vào ngày có lịch PT thì `SoBuoiConLai` giảm 1. Mỗi lịch chỉ trừ một lần, nên quét lại trong ngày không trừ nữa.
3. Hủy trước 2 tiếng không trừ buổi. Hủy muộn hoặc không đến vẫn trừ (#11).
4. Có nhiều gói PT thì trừ gói còn buổi có `NgayHetHan` sớm nhất, để hội viên ít mất buổi nhất.

**Vé Spa:** `SoBuoiConLai` từ 1 về 0; vé chỉ dùng một lần (#8). Khách không có gói Gym/Yoga còn hạn thì trừ ngay khi quét cổng. Khách có gói Gym/Yoga còn hạn vào cổng bằng gói đó, nên vé chỉ bị trừ khi dùng Spa tại quầy.

**Khi nào thất bại:**
- `SoBuoiConLai` bằng 0: không được đặt thêm hoặc dùng thêm.
- Gói PT quá `NgayHetHan`: buổi còn lại không dùng được nữa, vì chưa làm bảo lưu (#9).
- Không có gói Gym/Yoga còn hạn tại ngày hẹn: chưa đặt được lịch PT cho đến khi gia hạn hoặc mua thêm.
- Hai thao tác trừ cùng lúc trên một gói: `SoBuoiConLai` không được âm.

## QT-B4. Cập nhật điểm và xét hạng khi thanh toán

**Ai làm:** hệ thống, ngay sau khi D lưu hóa đơn.

**Hiện trạng (vấn đề 4 của đề):**
1. D lưu hóa đơn.
2. Hệ thống cộng `TongTien` mọi hóa đơn của hội viên trong 12 tháng qua.
3. So tổng với ngưỡng để nâng hoặc hạ hạng, rồi tính điểm.
4. In hóa đơn.

Bước 2 chạy khoảng 1.400 lần/ngày trên bảng `HoaDon` khoảng 500.000 dòng cho 12 tháng, nên nhân viên phải chờ ngay tại quầy.

**Quy tắc nghiệp vụ:**
- Điểm cộng thêm = phần nguyên của `TongTien` ÷ 100.000, với `TongTien` là số sau giảm giá (#5). Ví dụ 2.350.000đ được 23 điểm; phần lẻ 50.000đ bỏ, không cộng dồn sang hóa đơn sau (tự đặt).
- Điểm chưa đổi được thành quà và không hết hạn (#5).
- Mỗi lần cộng ghi một dòng lịch sử điểm: hội viên, hóa đơn, số điểm, thời điểm.
- Hạng theo chi tiêu 12 tháng gần nhất (#4):

  | Hạng | Chi tiêu 12 tháng |
  |---|---|
  | Silver | < 20 triệu |
  | Gold | 20 triệu đến < 50 triệu |
  | Platinum | ≥ 50 triệu |

- Hóa đơn mới chỉ có thể làm **tăng** hạng, và có hiệu lực ngay. Ví dụ đang 18 triệu, mua thêm 3 triệu thành 21 triệu thì lên Gold ngay trên hóa đơn này. Hạ hạng làm ở QT-B5 (#16).

**Yêu cầu sau cải tiến:** lúc thanh toán không phải cộng lại 12 tháng hóa đơn. Cách lưu trữ cụ thể chọn ở bước 2.3.

**Khi nào thất bại:**
- Hóa đơn đã được cộng điểm: không cộng lần hai.
- Cập nhật điểm lỗi: hủy cả giao dịch lập hóa đơn, để không có hóa đơn đã thu tiền mà thiếu điểm.

## QT-B5. Xét hạ hạng hằng đêm

**Ai làm:** job hệ thống, mỗi đêm một lần sau giờ đóng cửa (#16).

**Vì sao cần:** chi tiêu tính trên 12 tháng gần nhất nên tự giảm theo thời gian, kể cả khi hội viên không mua gì. Hiện trạng chỉ xét khi có hóa đơn mới, nên người không mua nữa giữ hạng mãi. Ví dụ hội viên chi 55 triệu vào tháng 10/2025 rồi không mua thêm: đến tháng 11/2026 chi tiêu 12 tháng là 0, phải về Silver.

1. Chọn các hội viên có hóa đơn vừa quá 12 tháng.
2. Tính lại chi tiêu 12 tháng của họ.
3. So với ngưỡng ở QT-B4, cập nhật hạng nếu đổi.

**Khi nào thất bại:** job lỗi giữa chừng thì chạy lại được mà không trừ chi tiêu hai lần cho cùng một hóa đơn.

## QT-B6. Quản lý danh mục gói dịch vụ

**Ai làm:** ban giám đốc.

1. Thêm gói mới với tên, loại, thời hạn, số buổi PT (nếu có), giá.
2. Sửa giá hoặc đánh dấu ngừng bán. Không xóa gói đã có người mua.
3. Giá mới chỉ áp dụng cho hóa đơn sau đó. Hóa đơn cũ giữ đơn giá riêng (D), gói đã mua giữ nguyên quyền lợi.

**Khi nào thất bại:**
- Giá không dương.
- Thời hạn sai: Gym, Yoga, PT phải là 3/6/12 tháng; vé Spa không có thời hạn.
- Gói PT không có số buổi, hoặc gói khác PT lại có số buổi.

## QT-B7. Theo dõi hội viên

**Quản lý chi nhánh**, mỗi ngày một lần, xem danh sách hội viên đăng ký tại chi nhánh mình:
- Sắp hết hạn: có gói với `NgayHetHan` trong 7 ngày tới và chưa có gói mới cùng loại.
- Lâu ngày chưa quay lại: không có `CheckIn.TGVao` trong 30 ngày qua dù còn gói hiệu lực.

Hai ngưỡng 7 và 30 ngày do nhóm tự đặt.

**Ban giám đốc** xem tỷ lệ phân bổ Silver / Gold / Platinum toàn hệ thống.

Danh sách không có ai thì hiển thị rỗng; không có trường hợp lỗi nghiệp vụ riêng.

## Điểm cần xác nhận

| # | Nội dung | Đang dùng | Cần ai xác nhận |
|---|---|---|---|
| 1 | Hạng tính theo 12 tháng gần nhất và ngưỡng 20/50 triệu (#4) | Như quyết định #4 | Cô |
| 2 | Quyết định #9, #11 chưa chốt | Cột "Đề xuất" | Cả nhóm |
| 3 | Gia hạn sớm: tính từ ngày mua (#9) khiến hội viên mất số ngày còn lại của gói cũ | Giữ theo #9, chưa nối tiếp | Cả nhóm |
| 4 | Khách chỉ có vé Spa: vé trừ khi quét cổng; ra ngoài rồi quay lại trong ngày có được vào không | Không, vé đã về 0 nên cổng từ chối | C |
| 5 | Luồng buổi PT: đặt lịch phải chặn khi số lịch `Đã đặt` đạt `SoBuoiConLai`; C thực hiện trừ buổi lúc quét cổng và cần đánh dấu lịch đã trừ; đã quét cổng rồi mới hủy lịch thì có trả lại buổi không; đề chỉ có 3 trạng thái nên "không đến" cần được phân biệt với hủy thường | Như QT-B3; đã quét cổng thì không trả lại | C |
| 6 | Điều kiện còn hạn thống nhất là ngày kiểm tra ≤ `NgayHetHan`; vào cổng lấy hôm nay, đặt PT lấy ngày hẹn | Như QT-B2, QT-B3 | C |
| 7 | Thêm chi nhánh đăng ký vào `HoiVien` để lập danh sách cấp chi nhánh, vì gói không gắn chi nhánh (#1) | Như QT-B1, QT-B7 | Cả nhóm |
| 8 | Hợp đồng chưa có cách để D yêu cầu tạo gói khi lập hóa đơn; chốt ở bước 2.3 | Như QT-B2 | D |
