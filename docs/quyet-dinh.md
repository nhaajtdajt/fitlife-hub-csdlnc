# Quyết định làm rõ yêu cầu

Những chỗ đề chưa rõ. Cột "Đề xuất" là **chưa chốt** — cả nhóm chốt ở Sprint 0, ghi kết quả vào cột "Chốt". Chỗ nào không chắc thì hỏi cô.

| # | Câu hỏi | Đề xuất | Module bị ảnh hưởng | Chốt |
|---|---|---|---|---|
| 1 | Gói tập dùng được ở chi nhánh nào? | Mọi chi nhánh (đề: "bất kỳ chi nhánh nào") | C | Mọi chi nhánh |
| 2 | Gói PT 1-1 dùng và trừ buổi thế nào? | Chọn HLV cho từng lịch | B, C | Chốt 08/11/2026: gói PT chỉ dùng đặt lịch. Đến NgayGio của lịch chưa hủy thì tự động trừ đúng một buổi, dù có vào cổng hoặc tham dự hay không. Không trừ khi check-in hay khi xác nhận hoàn thành |
| 3 | Một hóa đơn gồm gì? | Cả gói và sản phẩm, qua `ChiTietHoaDon` | B, D | Theo đề xuất |
| 4 | Hạng tính theo "năm" hay "12 tháng qua"? (đề tự mâu thuẫn) | 12 tháng gần nhất; ngưỡng nhóm tự đặt, ví dụ Gold ≥ 20 triệu, Platinum ≥ 50 triệu | B | Chốt GD1-09: 12 tháng gần nhất, từng khoản hết hạn riêng, không reset đồng loạt; điểm tích lũy giữ nguyên. Ngưỡng nhóm dùng Silver < 20tr ≤ Gold < 50tr ≤ Platinum (ngưỡng vẫn cần cô xác nhận). Biên thời gian và lưu tổng tại B/diem-hang.md |
| 5 | Điểm Loyalty tích trên gì? | Tổng tiền sau giảm giá, 1 điểm / 100.000đ; chưa làm đổi điểm | B, D | Theo đề xuất |
| 6 | Hoa hồng PT tính thế nào? | Mức cố định mỗi buổi hoàn thành, theo chi nhánh PT thuộc về **tại ngày buổi tập** | A, C | Theo đề xuất; mức hoa hồng lưu theo chi nhánh (`ChiNhanh.HoaHongMoiBuoi`) |
| 7 | Điểm đánh giá HLV, sức chứa phòng tập (đề có nhắc ở truy vấn nhưng không mô tả) | Bổ sung `DanhGiaHLV` và cột sức chứa cho chi nhánh | A, C, D | Thêm `DanhGiaHLV`, `ChiNhanh.SucChua` |
| 8 | Quyền vào cổng và điều kiện mua PT? | Tách quyền vào cổng khỏi lịch PT | B, C, D | Chốt 08/11/2026: PT không cấp quyền vào cổng. Vào cổng dùng thẻ Gym/Yoga/Spa hoặc dịch vụ có quyền vào: thẻ theo hạn còn hiệu lực hoặc vé một lượt còn lượt. Bổ sung thẻ 1 tháng và vé một lượt bên cạnh gói hiện có. Mua PT phải có quyền vào hợp lệ hoặc mua kèm; không có thì yêu cầu mua thẻ/vé. Đặt lịch chỉ xét quyền lợi PT, không giữ vé vào cổng cho lịch. Vé một lượt bị trừ khi vào, không gắn giờ PT; ra rồi vào lại cần lượt mới nếu không có thẻ theo hạn hợp lệ. Quét cổng không đọc/trừ lịch PT. Chốt GD1-04, áp dụng toàn dự án: mọi kiểm tra thẻ/vé và tính lượt vào chỉ thực hiện tại cổng; không có điểm kiểm tra/trừ lượt dịch vụ tại quầy. PT vẫn tự trừ theo giờ hẹn theo #2; vé Spa được chọn làm quyền vào thì trừ cùng giao dịch CheckIn, không kiểm tra/trừ thêm tại quầy. Phạm vi không theo dõi riêng sử dụng Spa bên trong |
| 9 | Gói bắt đầu tính khi nào? Có bảo lưu không? | Phân biệt gói theo hạn với PT và vé lượt | B, C, D | Chốt Gym/Yoga theo hạn ngày 08/11/2026: chưa có gói hoặc toàn bộ gói cùng loại đã hết hạn thì bắt đầu ngày mua; còn gói hiện hành/chờ kích hoạt thì nối sau ngày hết hạn xa nhất của các gói đã mua cùng loại. Gym nối Gym, Yoga nối Yoga; không áp dụng cho vé một lượt. Giữ nguyên lịch sử gói cũ. Gói mới có hiệu lực từ NgayBatDau đến hết NgayHetHan; hạn Gym/Yoga = ngày bắt đầu cộng số tháng rồi trừ 1 ngày (mốc cộng tháng không có ngày tương ứng thì lấy cuối tháng đích). Gói chờ chưa được dùng vào cổng. PT đã chốt: mua thêm dùng ngay từ ngày mua, mỗi lần mua có MaGHV, số buổi và hạn riêng; không chuyển buổi hay sửa hạn gói cũ. Khi đặt chọn gói của hội viên đủ hạn tại ngày hẹn và còn buổi chưa giữ, ưu tiên hết hạn sớm nhất (cùng hạn thì MaGHV tăng dần); lưu MaGHV vào lịch, đến giờ trừ đúng gói đó. Bảo lưu vẫn chưa chốt |
| 10 | Khung giờ PT cố định hay tùy ý? | Cố định, mỗi buổi 1 tiếng | C | Chốt 08/11/2026: khung cố định một giờ, bắt đầu vào giờ tròn theo Asia/Saigon (phút/giây bằng 0), ví dụ 17h–18h, 18h–19h; không nhận 17h30–18h30. Cùng PT không có hai lịch chưa hủy ở cùng giờ bắt đầu. Hai khung liền nhau không chồng lấn. Trừ tại giờ bắt đầu, hủy khi còn ít nhất hai giờ |
| 11 | Hủy lịch PT trước bao lâu? Không đến thì sao? | Cho phép hủy trước giờ hẹn | A, B, C | Chốt 08/11/2026: chỉ cho hội viên hủy khi còn ít nhất 2 giờ, giải phóng buổi đang giữ, không trừ. Sau mốc đó từ chối hủy; đến giờ hẹn vẫn tự trừ đúng một buổi, kể cả không đến hoặc không đủ quyền vào cổng. Vắng mặt không tính hoa hồng |
| 12 | PT có được dạy ở chi nhánh khác không? | Không, chỉ dạy tại chi nhánh đang công tác | A, C | Chốt GD1-06: PT chỉ dạy tại chi nhánh công tác và phải có chức vụ PT trong toàn bộ buổi hẹn. Đổi chức vụ/chi nhánh từ ngày mai bị từ chối nếu còn lịch chưa hủy bị thay đổi làm mất điều kiện này; trả danh sách lịch vướng, giữ nguyên dữ liệu. Không tự hủy, đổi HLV hoặc chuyển lịch. Đặt lịch và đổi vị trí phải kiểm tra nhất quán lịch sử/lịch hẹn khi ghi nhận |
| 13 | Ngày công tác dùng kiểu gì? `ChucVu` lưu ở đâu? | DATE; lưu chức vụ hiện tại và chức vụ lịch sử, chốt quy ước biên ngày và ngày hiệu lực | A, C | Chốt 08/11/2026: DATE, tính cả ngày bắt đầu và ngày kết thúc. Giai đoạn mới bắt đầu đúng ngày sau ngày kết thúc cũ. Bấm đổi chức vụ/chi nhánh trong ngày D thì cũ kết thúc D, mới bắt đầu D+1 (Asia/Saigon); không đổi giữa ngày. Đóng/mở lịch sử cùng giao dịch, giữ chức vụ hiện tại trong ngày D; sang ngày mới mới đồng bộ NhanVien theo giai đoạn hiệu lực. NgayKetThuc rỗng không đủ xác định hiện hành vì có giai đoạn chờ. Xem QT-A03/A04; giữ lịch sử cũ |
| 14 | Sửa mức HoaHongMoiBuoi thì các tháng cũ có đổi không? | Lưu kết quả kỳ đã chốt | A, C | Chốt GD1-10: giữ nguyên kỳ đã chốt. KyHoaHong lưu mức tại lúc chốt theo chi nhánh/tháng, ChiTietHoaHong lưu danh sách buổi; tổng tính từ dữ liệu chốt, không dùng mức hiện tại. Không mở lại/sửa kỳ trong phạm vi này; C bảo vệ nguồn lịch của kỳ đã chốt. Kỳ chưa chốt dùng mức đang cấu hình tại lúc xác nhận chốt |
| 15 | Tỷ lệ lấp đầy tính thế nào? Quên quét ra thì sao? | Số người đang trong phòng ÷ SucChua theo chi nhánh | A, C, D | Chốt GD1-07: chỉ báo cáo lấp đầy, không chặn cổng theo sức chứa. Tại T, đếm CheckIn cùng chi nhánh có TGVao <= T và (TGRa rỗng hoặc TGRa > T); tỷ lệ = 100 × số người / SucChua. SucChua là số nguyên dương, tỷ lệ có thể vượt 100%, không cắt ngọn. Quy ước mô phỏng: giờ hoạt động trong cùng ngày Asia/Saigon, giờ mở < giờ đóng; giờ hoạt động và sức chứa cố định trong kỳ dữ liệu. Giữ thiết kế tự đóng lượt quên ra tại giờ đóng cửa, phân biệt bằng SoHieuCuaRa rỗng; đây là thời gian ra ước tính. Nếu còn lượt quá hạn chưa được đóng thì báo cáo là tạm tính, chưa coi kỳ hoàn tất. Không giả định hỗ trợ qua đêm/24 giờ hoặc lịch sử thay đổi sức chứa |
| 16 | Hủy/hoàn tiền hóa đơn thì điểm và hạng ra sao? Khi nào hạ hạng? | Chỉ sửa/bỏ hóa đơn trước khi thanh toán; đã thanh toán thì không sửa, không xóa, không hoàn tiền; xét hạ hạng bằng job hằng đêm | B, D | Theo đề xuất (đề không yêu cầu hoàn tiền) |
| 17 | Web hay WinForm, ngôn ngữ, phiên bản SQL Server, ai làm module nào? | Nhóm tự chọn | Cả nhóm | |
| 18 | Doanh thu từng gói/sản phẩm trước hay sau giảm giá? | Thống nhất với tổng thực thu hóa đơn | D, B dùng tổng hóa đơn | Chốt GD1-08: doanh thu sau giảm giá; phân bổ GiamGia theo tỷ lệ thành tiền trước giảm của toàn bộ các dòng, gồm cả gói và sản phẩm. Quy ước tính bằng đồng nguyên, làm tròn xuống rồi chia phần đồng còn thiếu theo phần dư giảm dần, hòa thì STT tăng dần. Tổng giảm phân bổ = GiamGia, tổng doanh thu dòng = TongTien. Phân bổ trước khi lọc loại hàng; không trừ giảm giá lần hai. Chi tiết tại D thiết kế logic |

GD1-11 đã chốt: từng gói đã mua liên kết dòng hóa đơn, định danh đơn vị bằng (MaHD, STT, DonVi); MaGHV riêng. Tạo hóa đơn/gói/điểm/kho nguyên tử, xử lý lại MaHD không tạo trùng. Thiết kế B chuẩn hóa chủ/mã gói qua view từ hóa đơn.

GD1-12 đã chốt: ChiTietHoaDon không lưu ThanhTien; tính từ SoLuong × DonGia chụp lúc bán khi đọc. Doanh thu sau giảm và quy tắc phân bổ #18 giữ nguyên.

## Quy ước phân loại tần suất chung (GD1-15)

- Thấp: dưới 100 lần/ngày; Trung bình: từ 100 đến dưới 1.000 lần/ngày; Cao: từ 1.000 lần/ngày. Áp dụng toàn hệ thống, không so tổng một module với số riêng một chi nhánh.
- Với lịch tuần/tháng: quy đổi trung bình theo 7 ngày/tuần, 30 ngày/tháng cho ước lượng, nhân số chi nhánh/nhân viên được áp dụng; vẫn ghi số gốc và thời điểm chạy dồn. Không tự gọi mọi tác vụ theo kỳ là thấp.
- Tần suất GĐ1 đếm lượt nghiệp vụ theo đơn vị công bố ở từng hàng. A-F09: một buổi đối chiếu; CN-B08/C-F14: một lịch trừ buổi; C-F10: một lịch xác nhận vắng. Các chức năng chốt kỳ/đồng bộ toàn hệ thống vẫn đếm theo lần chốt/đồng bộ đã ghi, không đổi thành số dòng. Không đồng nhất lượt nghiệp vụ, số dòng đọc, số câu SQL và số lần chạy tác vụ. Chi phí và số lần chạy kỹ thuật đo riêng ở GĐ2.
- Chốt xử lý PT theo đợt: gom lịch đến giờ để xử lý, mỗi lịch trừ đúng một lần tại giờ hẹn; không cố ý đợi hết giờ hoặc cuối ngày. Chạy bù chỉ để khắc phục lỗi/chậm. Chi tiết cơ chế nằm ở [ghi chú GĐ2](gd2/ghi-chu-trien-khai.md).
- Bảng phân tích từng chức năng không phải danh sách các tập giao dịch rời nhau. Không cộng tất cả hàng/module thành tổng: một thanh toán gồm D bán hàng, B tạo gói/cộng điểm; trừ vé nằm trong check-in; B trừ PT và C tác vụ đến giờ là cùng thao tác.
- Bộ giả định PT giữ 2.900 lịch = 2.600 hoàn thành + 200 hủy đúng hạn + 100 vắng. 2.700 lịch trừ buổi = 2.600 + 100, không trừ lại khi hoàn thành/vắng. Hủy/vắng là kết quả của tập lịch, không cộng vào số lịch mới. Lượt hủy bị từ chối và chạy lại chưa có ước lượng. Các con số là giả định, không phải số đo.

# Bảng giả định số liệu chung

Đề cho 12 chi nhánh và ghi “> 10.000 lượt quét thẻ/QR check-in đồng thời toàn hệ thống” trong 17h–20h; log check-in và lịch sử tập luyện tăng > 300.000 dòng/tháng (de.md, vấn đề 1 và 3). Không thay câu này bằng 10.000 lượt cộng dồn trong ba giờ. Bảng dưới là bộ giả định làm việc của GĐ1, không phải số đo và không chứng minh đáp ứng yêu cầu tải của đề. Cả A/B/C/D dùng chung; cơ chế và phép đo thuộc GĐ2.

## Giả định gốc

| # | Giả định | Giá trị | Cách tính / căn cứ |
|---|---|---|---|
| G1 | Mốc minh họa số lượt vào thành công cộng dồn trong 17h–20h | ~10.000 lượt/3 giờ toàn hệ thống | Giả định làm tròn dùng tính tần suất; không phải diễn giải hay đáp ứng yêu cầu “> 10.000 ... đồng thời” của đề |
| G2 | Tỷ lệ lượt vào trong 17h–20h so với cả ngày | 40% | Giả định nhóm để tính tần suất; không khẳng định đây là tỷ lệ đo tại Việt Nam |
| G3 | Số lần đến tập của một hội viên đang hoạt động | 2,5 lần/tuần | IHRSA: trung bình ~2 lần/tuần (104–106 lượt/năm) [3]; chỉ tính người còn đi tập nên lấy cao hơn |
| G4 | Tỷ lệ hội viên dùng PT 1-1 | 13% | IHRSA: 12–15% hội viên dùng PT [4] |
| G5 | Số buổi PT của một hội viên PT | 2 buổi/tuần | Lịch tập PT phổ biến 2–3 buổi/tuần |
| G6 | Số buổi một PT dạy mỗi ngày | 5 buổi | Thực tế PT toàn thời gian dạy 4–8 buổi/ngày [5] |
| G7 | Tỷ lệ lịch PT không hoàn thành (hủy đúng hạn hoặc vắng mặt) | ~10% | Giả định cập nhật theo #11: trong ~2.900 lịch/ngày, ~200 hủy đúng hạn + ~100 vắng mặt; cần nhóm duyệt |
| G8 | Mốc thời hạn dùng ước lượng ban đầu | 6 tháng | Chỉ giải thích mốc cũ ~390 lượt mua/ngày; sau bổ sung gói 1 tháng/vé lượt, dùng cơ cấu G11, không coi 6 tháng là trung bình mới |
| G9 | Tỷ lệ lượt đến có mua sản phẩm | 3% | Nhóm tự đặt |
| G10 | Phân bổ hạng Silver / Gold / Platinum | 70% / 25% / 5% | Nhóm tự đặt (phụ thuộc ngưỡng ở quyết định #4) |
| G11 | Cơ cấu ~390 lượt mua Gym/Yoga/ngày | 300 gói 3/6/12 tháng + 60 gói 1 tháng + 30 vé một lượt | Giả định làm việc GD1-16, gồm mua mới/gia hạn; trong 30 vé có 10 lượt mua lại khi quay vào sau khi đã ra |
| G12 | Lượt dùng vé tại cổng/ngày | 230 = 200 Spa + 30 Gym/Yoga | Giả định vé mua được dùng trong ngày; đã nằm trong tổng 25.000 lượt vào, không cộng lần nữa |

## Số liệu suy ra (dùng cho bảng tần suất)

| Số liệu | Giá trị | Từ | Module dùng |
|---|---|---|---|
| Lượt check-in mỗi ngày (toàn hệ thống) | ~25.000 (~2.100/chi nhánh) | G1 ÷ G2 | C, D |
| Dòng `CheckIn` mỗi tháng | ~750.000 | 25.000 phiên mới × 30 ngày; một phiên một dòng, quét ra cập nhật dòng cũ; là ước lượng, không xác nhận cơ cấu log đề cho | C, D |
| Trung bình của mốc minh họa G1 | ~55,56 lượt/phút = ~0,926 lượt/giây toàn hệ thống | 10.000 ÷ 180 phút hoặc ÷ 10.800 giây; không suy ra đỉnh/đồng thời | C |
| Hội viên đang hoạt động | ~70.000 (~5.800/chi nhánh) | 25.000 × 7 ÷ G3 | B, C |
| Tổng hội viên trong CSDL (kể cả đã hết hạn) | ~120.000 | Tự đặt, gồm hội viên cũ | B |
| Hội viên có gói PT | ~9.000 | 70.000 × G4 | B, C |
| Buổi PT diễn ra mỗi ngày | ~2.600 | 9.000 × G5 ÷ 7 | A, C |
| Lượt đặt lịch PT mỗi ngày | ~2.900 | 2.600 ÷ (1 − G7) | C |
| Lượt xem lịch trống PT mỗi ngày | ~14.500 | ~5 lần xem mỗi lượt đặt | C |
| Dòng `LichHenPT` mỗi tháng | ~87.000 | 2.900 × 30 | A, C |
| Số PT | ~520 (~45/chi nhánh) | 2.600 ÷ G6 | A, C |
| Tổng nhân viên | ~700 (~58/chi nhánh: 45 PT, 6 lễ tân, 5 bán hàng, 2 quản lý) | Tự đặt theo số PT | A |
| Điều chuyển nhân viên | ~10 lần/tháng | ~1,5%/tháng | A |
| Lượt mua Gym/Yoga mỗi ngày | ~390 đơn vị gói/vé | G11; giữ mốc quy mô cũ nhưng bổ sung cơ cấu, không suy bằng thời hạn 6 tháng nữa | B, D |
| Hóa đơn gói PT mỗi ngày | ~100 | Gói ~24 buổi dùng ~3 tháng | B, D |
| Hóa đơn vé Spa lẻ mỗi ngày | ~200 | Tự đặt | D |
| Hóa đơn bán sản phẩm mỗi ngày | ~750 | 25.000 × G9 | D |
| **Tổng hóa đơn mỗi ngày** | **~1.400** (~42.000/tháng) | Giả định 690 hóa đơn có gói + 750 có sản phẩm − 40 có cả hai; không cộng trùng hóa đơn hỗn hợp | B, D |
| Lượt cập nhật điểm/hạng mỗi ngày | ~1.400 | Mỗi hóa đơn một lần | B |
| Số gói dịch vụ / số mã sản phẩm | ~30 / ~150 (tồn kho 150 × 12 = 1.800 dòng) | Tự đặt | B, D |

## Phân biệt tải cổng và giả định — GD1-16

- **Lượt trong khoảng:** G1 là mốc minh họa cộng dồn trong ba giờ; G1/G2 cho ~25.000 lượt vào thành công/ngày. Một người có thể vào nhiều lần, nên đây không phải 25.000 người khác nhau. Lượt quay lại bằng vé mới đã nằm trong tổng này.
- **Tốc độ trung bình:** ~55,56 lượt/phút chỉ đúng với mốc minh họa G1. Chia đều 12 chi nhánh chỉ cho trung bình ~2.083 lượt vào/chi nhánh/ngày, không khẳng định các chi nhánh có tải bằng nhau.
- **Tốc độ đỉnh:** số lượt trong giây/phút bận nhất; chưa được đề định lượng và không suy được từ trung bình. Bỏ kết luận cũ “đỉnh vài lượt/giây”.
- **Mức đồng thời:** số yêu cầu đang được xử lý chồng thời gian, khác lượt/giây và khác số người đang trong phòng. Giữ nguyên yêu cầu >10.000 của đề; khoảng chồng thời gian/cách tạo tải cần làm rõ trước kiểm thử GĐ2. Không tự giảm yêu cầu xuống 0,926 lượt/giây hoặc tự diễn giải thành cùng một mili giây.
- **Số dòng:** mỗi lượt vào thành công tạo một CheckIn; quét ra cập nhật chính dòng đó. ~25.000 vào + tối đa khoảng 25.000 quét ra là ~50.000 thao tác cổng/ngày, không phải 50.000 dòng mới. Quên quét ra được đóng tự động thay cho thao tác quét ra tương ứng. Quét lặp phiên mở/từ chối không tạo phiên mới; số yêu cầu lỗi/lặp chưa có ước lượng, phải bổ sung khi phân tích tải thực tế.
- **Mục tiêu <50 ms:** mục tiêu tra cứu quyền vào cổng trong đề, chưa phải kết quả đạt được; phải phân biệt thời gian truy vấn với toàn bộ lượt quét/mở cổng khi đo ở GĐ2. Số liệu trung bình không chứng minh đạt mục tiêu.
- **Gói/vé mới:** G11 là giả định cơ cấu trong tổng 390, không tự cộng thêm vào tổng cũ. 360 gói theo hạn gồm 60 gói một tháng; 30 vé lượt gồm 10 lượt mua lại. Vé lượt không gia hạn nối tiếp. Tổng tạo gói/vé ~690/ngày = 390 + 100 PT + 200 Spa; trừ vé tại cổng ~230/ngày là phần con của check-in.
- **Giả định hóa đơn và đơn vị:** để ước lượng GĐ1, lấy mỗi hóa đơn có gói tạo một đơn vị gói/vé; ~690 hóa đơn có gói và ~750 có sản phẩm, trong đó ~40 hóa đơn có cả hai, cho ~1.400 hóa đơn riêng biệt. Hệ thống vẫn cho nhiều gói/đơn vị trên một hóa đơn và mua PT kèm quyền vào; khi cơ cấu này tăng phải ước lượng lại số đơn vị và hóa đơn riêng, không dùng phép đồng nhất một-một như ràng buộc nghiệp vụ. Các số này là giả định quy mô, không phải dự báo bán hàng từ 70.000 hội viên.

## Khối lượng sinh dữ liệu (giai đoạn 2)

Không cần sinh đủ khối lượng thật. Đề chỉ yêu cầu **≥ 70.000 dòng** cho bảng cần minh họa. Đề xuất:

| Bảng | Số dòng sinh | Ghi chú |
|---|---|---|
| `ChiNhanh` / `NhanVien` / `LichSuCongTac` | 12 / ~700 / ~1.000 | Nhỏ, đúng như thực tế |
| `HoiVien` | ~120.000 | |
| `GoiHoiVien` | ~200.000 | Mỗi hội viên vài gói theo thời gian |
| `CheckIn` | ~2.000.000 (khoảng 3 tháng) | Đủ để partition theo tháng có ý nghĩa |
| `LichHenPT` | ~260.000 (3 tháng) | |
| `HoaDon` / `ChiTietHoaDon` | ~500.000 / ~750.000 (1 năm) | 1 năm để có đủ dữ liệu xét hạng 12 tháng |

## Nguồn tham khảo

1. [What gym check-ins reveal about member behavior — Wellyx](https://wellyx.com/blog/what-gym-check-ins-reveal-about-member-behavior/)
2. [When South Africa Goes to Gym: 2.4 Million Check-Ins Analysed — Itensity](https://itensityonline.com/?p=948336)
3. [Gym membership statistics (số liệu IHRSA) — Wod.guru](https://wod.guru/blog/en/gym-membership-statistics/)
4. [IHRSA Fitness Training Report Shows Consumer Preferences — Health & Fitness Association](https://www.healthandfitness.org/ihrsa-fitness-training-report-shows-consumer-preferences/)
5. [What Does a Personal Trainer Do on a Daily Basis? — FitSW](https://www.fitsw.com/blog/what-does-a-personal-trainer-do-on-a-daily-basis/)

Các nguồn là số liệu ngành ở Mỹ/thế giới, phần lớn từ 2014–2016; chưa tìm được số liệu công khai của chuỗi gym Việt Nam. Dòng "Tự đặt" là giả định của nhóm, có thể đổi.
