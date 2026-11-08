# Module B — Mô tả quy trình nghiệp vụ

Phạm vi: hội viên, gói dịch vụ, gói đã mua, hạng và điểm. Hóa đơn thuộc D, check-in và lịch hẹn PT thuộc C; tài liệu này chỉ dùng các cột của họ đã có trong [hợp đồng giao diện](../../phan-cong.md).

Ký hiệu `#N` là quyết định số N trong [quyet-dinh.md](../../quyet-dinh.md). Quyết định #9 đã chốt gia hạn nối tiếp cho Gym/Yoga theo hạn; PT đã chốt mua thêm dùng ngay, từng lần mua giữ buổi/hạn riêng; bảo lưu vẫn chờ chốt. Quyết định #2, #8, #11 đã cập nhật ngày 08/11/2026. Những điểm cần xác nhận được gom ở [cuối tài liệu](#điểm-cần-xác-nhận).

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
2. Chọn một hoặc nhiều gói đang bán, dùng ở mọi chi nhánh (#1). Có thể có nhiều bản ghi mua cùng loại Gym/Yoga nhưng các khoảng hiệu lực của gói theo hạn cùng loại không chồng nhau; gói mua trước chờ đến ngày bắt đầu.
3. B tính và D hiển thị ngày bắt đầu/hết hạn của từng gói trước khi hội viên xác nhận thanh toán. Khi ghi nhận, kiểm tra lại đuôi chuỗi theo hội viên + loại gói và tính lại ngày nối tiếp để hai giao dịch mua đồng thời không tạo khoảng chồng nhau; nếu ngày dự kiến thay đổi, cần xác nhận lại trước khi hoàn tất thanh toán.
4. Thanh toán thành công thì tạo các dòng `GoiHoiVien` trong cùng giao dịch với hóa đơn. Với Gym/Yoga theo hạn: `NgayBatDau = max(NgayMua, NgayHetHanXaNhatCungLoai + 1 ngày)`; nếu chưa có gói cùng loại thì lấy `NgayMua`. Tính cả gói đang dùng và gói đã mua chờ kích hoạt; nhiều gói cùng loại trên cùng hóa đơn cũng được xếp nối tiếp theo thứ tự dòng/đơn vị. Vé một lượt dùng được từ ngày mua. PT bắt đầu ngày mua, mỗi đơn vị gói tạo một `MaGHV` riêng với số buổi/hạn riêng. Mua thêm không cần chờ gói cũ hết buổi/hết hạn, không chuyển buổi hoặc sửa hạn gói cũ.

   | Loại gói | `NgayHetHan` | `SoBuoiConLai` |
   |---|---|---|
   | Gym, Yoga theo hạn | Cộng 1/3/6/12 tháng vào `NgayBatDau`, rồi trừ 1 ngày | Không áp dụng |
   | Vé vào một lượt Gym/Yoga | Không có | 1 |
   | PT 1-1 | `NgayBatDau` + thời hạn gói | Số buổi của gói, ví dụ 24 |
   | Vé Spa lẻ | Không có (vé không hết hạn, #8) | 1 |

5. Cập nhật điểm và hạng theo QT-B4.

Gia hạn Gym/Yoga theo hạn là mua bản ghi mới nối tiếp, không sửa ngày hoặc ép gói cũ hết hạn. Phạm vi nối tiếp là `(MaHV, LoaiGoi)`: Gym nối Gym, Yoga nối Yoga, dù khác mã gói/thời hạn; không xếp vé một lượt vào chuỗi này.

Gói theo hạn được dùng khi `NgayBatDau <= NgaySuDung <= NgayHetHan`. Với Gym/Yoga, mốc cộng tháng không có ngày tương ứng thì lấy cuối tháng đích, sau đó trừ một ngày để tính ngày hết hạn bao gồm cả ngày cuối. Ví dụ: gói cũ hết 15/11/2026, mua thêm một tháng ngày 08/11 thì gói mới dùng từ 16/11 đến hết 15/12/2026. Mua thêm một tháng nữa trước khi gói này bắt đầu thì nối từ 16/12/2026 đến hết 15/01/2027. Gói chưa đến ngày bắt đầu không cấp quyền vào cổng.

Trạng thái chờ/đang hiệu lực/hết hạn có thể suy ra từ ngày, không cần tác vụ kích hoạt để mở quyền. Điểm và chi tiêu ghi nhận khi thanh toán, không đợi ngày gói bắt đầu.

**Khi nào thất bại:**
- Thanh toán thất bại: không có hóa đơn, không có gói, không cộng điểm. Các việc này nằm trong cùng một giao dịch.
- Gói đã ngừng bán: không cho chọn.
- Mua gói PT khi chưa có thẻ dịch vụ cấp quyền vào còn hạn hoặc vé vào còn lượt: yêu cầu mua kèm thẻ/vé (có lựa chọn 1 tháng hoặc một lượt) trên cùng hóa đơn; chưa mua thì chưa hoàn tất mua PT (#8). Quyền này không được giữ riêng cho giờ PT.
- Hóa đơn đã thanh toán thì không sửa, không xóa, không hoàn tiền (#16): gói đã tạo và điểm đã cộng giữ nguyên. Lập sai chỉ sửa được trước khi thanh toán.

## QT-B3. Sử dụng buổi PT và quyền vào cổng

**Ai làm:** C quản lý lịch và tác vụ đến giờ hẹn; cổng kiểm tra quyền vào và trừ vé một lượt, gồm vé Spa. B quản lý quyền lợi đã mua.

| Chức năng | Điều kiện và tác động |
|---|---|
| Vào cổng | Có thẻ dịch vụ cấp quyền vào thỏa `NgayBatDau <= NgaySuDung <= NgayHetHan`, hoặc vé vào đã mua còn lượt. Gói chờ kích hoạt không dùng được. PT không phải thẻ vào cổng. Chỉ trừ vé được dùng cho lượt vào, không trừ PT |
| Đặt lịch PT | Gói PT còn hạn tại ngày hẹn, còn buổi chưa bị giữ cho lịch khác. Không đòi thẻ Gym/Yoga còn hạn tại ngày hẹn; quyền vào được kiểm tra riêng khi quét |
| Mua gói PT | Phải có quyền vào hợp lệ hoặc mua kèm thẻ/vé theo QT-B2; vé mua kèm không dành riêng cho lịch PT |
| Vé Spa tại cổng | Khi vé Spa được chọn làm quyền vào, trừ một lượt cùng giao dịch tạo CheckIn. Không có bước kiểm tra hoặc trừ vé Spa tại quầy |

**Gói PT:**
1. Chọn HLV và khung giờ; hệ thống chọn `MaGHV` thuộc hội viên, loại PT, đủ hạn tại ngày hẹn và còn buổi chưa giữ. Ưu tiên `NgayHetHan` sớm nhất, cùng hạn thì `MaGHV` tăng dần. Tạo lịch gắn `MaGHV`, giữ một buổi nhưng chưa giảm `SoBuoiConLai`.
2. Hội viên được hủy khi thời điểm yêu cầu không muộn hơn `NgayGio - 2 giờ` (đúng mốc hai giờ vẫn được hủy). Giải phóng quyền lợi giữ chỗ, không trừ buổi.
3. Sau mốc đó, từ chối hủy. Tại `NgayGio`, tác vụ tự giảm một buổi của gói đã giữ cho lịch và đặt `DaTruBuoi = 1` trong cùng giao dịch, dù hội viên đã vào, chưa vào hoặc không đến.
4. Tác vụ chạy lại không trừ lần hai. Khi hoàn thành hoặc xác nhận vắng mặt không trừ thêm; chỉ buổi thực sự hoàn thành mới được tính hoa hồng.
5. Số buổi khả dụng của một gói = `SoBuoiConLai` trừ số lịch của chính `MaGHV` có `TrangThai = Đã đặt` và `DaTruBuoi = 0` (kể cả lịch quá giờ đang chờ tác vụ xử lý). Không lưu thêm bộ đếm giữ chỗ. Không đổi gói của lịch sau khi tạo; mua mới hoặc hủy lịch khác không làm phân bổ lại lịch đã có.
6. Đặt/hủy/trừ buổi phải kiểm tra lại quyền lợi và trạng thái tại lúc ghi nhận; đặt lịch phải phù hợp chức vụ/chi nhánh cả giai đoạn chờ (GD1-06). Hai lần đặt đồng thời không được giữ chung buổi cuối. Quy tắc và tình huống kiểm tra tại [thiết kế gói PT](goi-pt.md).

**Vé vào một lượt:** từ 1 về 0 khi tạo thành công một phiên check-in. Vé có thể dùng trước giờ PT, không được giữ cho lịch. Sau khi ra, muốn vào lại cần vé/lượt mới hoặc thẻ theo hạn còn hiệu lực. Quét lặp trong phiên đang mở không tạo phiên mới hoặc trừ thêm. Vé Gym/Yoga một lượt là lựa chọn bổ sung theo quyết định #8; vé Spa được xử lý tại cổng như vé một lượt. Phạm vi hệ thống chỉ xác thực quyền vào và ghi nhận phiên vào–ra, không theo dõi riêng việc sử dụng Spa bên trong. Nếu dùng thẻ Gym/Yoga còn hiệu lực để vào thì không tự trừ thêm vé Spa.

**Ví dụ:** mua một buổi PT lúc 18h và vé Gym một lượt; vào 8h rồi ra 9h làm hết vé Gym nhưng chưa trừ PT. Đến 18h, PT tự bị trừ nếu lịch chưa hủy đúng hạn. Muốn vào tập lúc 18h phải mua lượt vào mới nếu không có quyền vào khác.

**Khi nào thất bại:**
- Không đủ buổi PT chưa giữ chỗ hoặc gói PT không còn hạn tại ngày hẹn: không đặt được.
- Không có thẻ/vé vào hợp lệ: cổng từ chối; lịch PT vẫn giữ nguyên và đến giờ vẫn trừ buổi.
- Hủy khi còn dưới hai giờ: từ chối, lịch vẫn `Đã đặt`.
- Hai thao tác cùng giữ/trừ một quyền lợi: không được vượt số buổi hoặc làm số dư âm.
- Tác vụ đến giờ lỗi: hoàn tác, ghi nhận để thử lại các lịch đến hạn chưa trừ; không đánh dấu đã trừ khi cập nhật gói thất bại.

## QT-B4. Cập nhật điểm và xét hạng khi thanh toán

**Ai làm:** hệ thống, ngay sau khi D lưu hóa đơn.

**Hiện trạng (vấn đề 4 của đề):**
1. D lưu hóa đơn.
2. Hệ thống cộng `TongTien` mọi hóa đơn của hội viên trong 12 tháng qua.
3. So tổng với ngưỡng để nâng hoặc hạ hạng, rồi tính điểm.
4. In hóa đơn.

Bước 2 chạy khoảng 1.400 lần/ngày trên bảng `HoaDon` khoảng 500.000 dòng cho 12 tháng, nên nhân viên phải chờ khi hoàn tất thanh toán.

**Quy tắc nghiệp vụ:**
- Điểm cộng thêm = phần nguyên của `TongTien` ÷ 100.000, với `TongTien` là số sau giảm giá (#5). Ví dụ 2.350.000đ được 23 điểm; phần lẻ 50.000đ bỏ, không cộng dồn sang hóa đơn sau (tự đặt).
- Điểm chưa đổi được thành quà và không hết hạn (#5).
- Mỗi lần cộng ghi một dòng lịch sử điểm: hội viên, hóa đơn, số điểm, thời điểm. GD1-08 không đổi đầu vào: nhận TongTien sau giảm của toàn hóa đơn đúng một lần, không cộng điểm riêng từng dòng phân bổ rồi cộng lại.
- Hạng theo chi tiêu 12 tháng gần nhất (#4):

  | Hạng | Chi tiêu 12 tháng |
  |---|---|
  | Silver | < 20 triệu |
  | Gold | 20 triệu đến < 50 triệu |
  | Platinum | ≥ 50 triệu |

- Đồng bộ các khoản quá hạn trước, rồi cộng hóa đơn mới và tính hạng ngay. Khoản mua mới chỉ tăng tổng sau đồng bộ; hạng so với giá trị cũ có thể giảm nếu có khoản vừa hết hạn. Ví dụ đang 18 triệu, mua thêm 3 triệu thành 21 triệu thì lên Gold ngay trên hóa đơn này. Tác vụ QT-B5 hỗ trợ cập nhật nền; đọc/sử dụng hạng vẫn phải đồng bộ tới thời điểm hiện tại.

**Thiết kế GD1-09:** lưu tổng chi tiêu, điểm và hạng trên HoiVien; GhiNhanLoyalty khóa theo MaHD chống cộng/trừ lặp. Đồng bộ khoản quá hạn trước khi dùng hạng hoặc thanh toán. Xem [thiết kế điểm/hạng](diem-hang.md); chỉ mục và thủ tục cụ thể thuộc GĐ2.

**Khi nào thất bại:**
- Hóa đơn đã được cộng điểm: không cộng lần hai.
- Cập nhật điểm lỗi: hủy cả giao dịch lập hóa đơn, để không có hóa đơn đã thu tiền mà thiếu điểm.

## QT-B5. Xét hạ hạng hằng đêm

**Ai làm:** job hệ thống, mỗi đêm một lần sau giờ đóng cửa (#16).

**Vì sao cần:** chi tiêu tính trên 12 tháng gần nhất nên tự giảm theo thời gian, kể cả khi hội viên không mua gì. Hiện trạng chỉ xét khi có hóa đơn mới, nên người không mua nữa giữ hạng mãi. Ví dụ hội viên chi 55 triệu vào tháng 10/2025 rồi không mua thêm: đến tháng 11/2026 chi tiêu 12 tháng là 0, phải về Silver.

1. Chọn mọi khoản đã đến mốc hết hạn 12 tháng nhưng GhiNhanLoyalty.DaLoaiKhoi12T còn 0, gồm cả phần tồn do tác vụ lỗi.
2. Khóa hội viên rồi đọc lại, trừ đúng khoản khỏi TongChiTieu12T và bật cờ trong cùng giao dịch; không trừ điểm.
3. Cập nhật hạng theo tổng mới. Không reset toàn bộ và không xóa lịch sử. Quy tắc biên thời gian/chống lặp ở [thiết kế điểm/hạng](diem-hang.md).

**Khi nào thất bại:** job lỗi giữa chừng thì chạy lại được mà không trừ chi tiêu hai lần cho cùng một hóa đơn.

## QT-B6. Quản lý danh mục gói dịch vụ

**Ai làm:** ban giám đốc.

1. Thêm gói mới với tên, loại, thời hạn, số buổi PT (nếu có), giá.
2. Sửa giá hoặc đánh dấu ngừng bán. Không xóa gói đã có người mua.
3. Giá mới chỉ áp dụng cho hóa đơn sau đó. Hóa đơn cũ giữ đơn giá riêng (D), gói đã mua giữ nguyên quyền lợi.

**Khi nào thất bại:**
- Giá không dương.
- Thời hạn sai: Gym/Yoga theo hạn cho phép 1/3/6/12 tháng; PT giữ 3/6/12 tháng; vé vào một lượt và vé Spa không có thời hạn.
- Gói PT thiếu số buổi dương; vé một lượt không có đúng một lượt ban đầu; gói theo hạn lại khai số lượt hữu hạn.

## QT-B7. Theo dõi hội viên

**Quản lý chi nhánh**, mỗi ngày một lần, xem danh sách hội viên đăng ký tại chi nhánh mình:
- Sắp hết hạn: có gói theo hạn đang hiệu lực với `NgayHetHan` trong 7 ngày tới và chưa mua gói nối tiếp cùng loại. Gói đã hết hạn từ trước không được coi là gói nối tiếp; hội viên chỉ có gói chờ chưa được xem là đang có gói hiệu lực.
- Lâu ngày chưa quay lại: không có `CheckIn.TGVao` trong 30 ngày qua dù còn gói hiệu lực.

Hai ngưỡng 7 và 30 ngày do nhóm tự đặt.

**Ban giám đốc** xem tỷ lệ phân bổ Silver / Gold / Platinum toàn hệ thống.

Danh sách không có ai thì hiển thị rỗng; không có trường hợp lỗi nghiệp vụ riêng.

## Điểm cần xác nhận

| # | Nội dung | Đang dùng | Cần ai xác nhận |
|---|---|---|---|
| 1 | GD1-09 đã chốt 12 tháng gần nhất, điểm không hết hạn; còn xác nhận ngưỡng 20/50 triệu với cô (#4) | Thiết kế tại diem-hang.md | Cô xác nhận ngưỡng |
| 2 | #9 đã chốt Gym/Yoga nối tiếp, PT mua thêm dùng ngay và giữ hạn/buổi riêng; bảo lưu còn chờ | Chưa triển khai bảo lưu | Cả nhóm |
| 3 | Gia hạn sớm Gym/Yoga theo hạn | Nối sau gói cùng loại có ngày hết hạn xa nhất, kể cả gói chờ; giữ gói cũ | Đã chốt 08/11/2026 |
| 4 | Vé vào một lượt đã dùng, ra rồi quay lại | Đã chốt: cần mua lượt mới nếu không có quyền vào khác còn hợp lệ, dù có lịch PT | Đã chốt 08/11/2026 |
| 5 | Liên kết lịch với đúng gói PT và giữ buổi khi đặt | LichHenPT.MaGHV; đếm lịch Đã đặt/chưa trừ trên từng gói, kiểm tra nhất quán khi ghi | Đã chốt và thiết kế GD1-03 |
| 6 | Kiểm tra đủ hai biên của gói theo hạn: `NgayBatDau <= NgayKiemTra <= NgayHetHan`; vào cổng lấy ngày quét, đặt PT lấy ngày hẹn. Vé một lượt xét đã mua/còn lượt, không so hạn rỗng | B công bố thêm NgayBatDau cho C | Đã đồng bộ B/C; PT mua thêm dùng ngay theo GD1-03 |
| 7 | Thêm chi nhánh đăng ký vào `HoiVien` để lập danh sách cấp chi nhánh, vì gói không gắn chi nhánh (#1) | Như QT-B1, QT-B7 | Cả nhóm |
| 8 | Hợp đồng dữ liệu tạo gói theo dòng/đơn vị đã chốt GD1-11 và tích hợp GD1-17; chữ ký thủ tục thuộc GĐ2 | Như QT-B2 | D |

**GD1-11 — tạo gói từ dòng hóa đơn:** mỗi GoiHoiVien lưu (MaHD,STT,DonVi), FK dòng và UNIQUE bộ ba; dòng gói số lượng n tạo đúng n gói, dòng sản phẩm tạo 0. Xử lý lại cùng MaHD trả gói đã có; không tạo lại gói/cộng điểm/trừ kho. Hóa đơn/gói/điểm/kho phải cùng được ghi nhận đầy đủ hoặc cùng hoàn tác khi lỗi. Thông tin chủ và danh mục đọc qua vGoiHoiVien. Chi tiết tại [thiết kế B](thiet-ke-logic.md).
