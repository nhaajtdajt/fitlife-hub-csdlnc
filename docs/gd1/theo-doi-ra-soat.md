# Theo dõi xử lý các vấn đề giai đoạn 1

Ngày rà soát: 08/11/2026. Bản đối chiếu ban đầu: `a157ee2`.

Danh sách giữ đủ 17 mục của lần rà soát trước, gồm lỗi, thiếu đầu ra và điểm cần thống nhất. Mã mục cố định để cùng xử lý từng việc; không đổi số hoặc xóa mục sau khi giải quyết. Các quyết định còn mở được ghi riêng, không mặc định là lỗi hay đã được duyệt.

## Cách theo dõi

- Trạng thái: Mở → Đang xử lý / Chờ chốt → Đã kiểm tra.
- Mỗi lần xử lý: chốt cách làm, sửa các tài liệu bị ảnh hưởng, kiểm tra một tình huống cụ thể, rồi ghi kết quả ở cuối mục.
- Chỉ đánh dấu Đã kiểm tra khi tài liệu của các module liên quan đã khớp nhau; thống nhất trong chat chưa đủ.
- Rà cuối 08/11/2026: mở lại GD1-10/11/13/15/17 theo [8 nhóm phát hiện LC-01…LC-08](ra-soat-cuoi.md). Giữ đủ 17 mã gốc: 12 mục giữ trạng thái đã kiểm tra, 5 mục mở lại. Hồ sơ chưa phải bản cuối sẵn sàng nộp; các kết quả kiểm tra cấu trúc trước đây vẫn được giữ nhưng không đủ chứng minh đầy đủ nghiệp vụ. Kiểm thử triển khai và khóa giao dịch SQL Server thuộc GĐ2.
- Căn cứ: [yêu cầu](../../required.md), [đề](../../de.md), [phân công](../phan-cong.md), [quyết định chung](../quyet-dinh.md).

## Rà lại ranh giới GĐ1/GĐ2 theo yêu cầu người dùng

Đối chiếu required.md và de.md: GĐ1 cần quy trình, bảng tần suất có ước lượng và mức cao/trung bình/thấp, ERD, lược đồ/khóa/PTH/dạng chuẩn, thuộc tính và ràng buộc/bảng ảnh hưởng. Không yêu cầu cài đặt hoặc chứng minh hiệu năng.

- Giữ ở GĐ1: liên kết gói–hóa đơn, lưu lịch sử, số liệu phục vụ điểm/hạng, quy tắc không trùng/không âm/không cộng lặp và yêu cầu các thay đổi cùng thành công hoặc thất bại.
- Các đoạn thứ tự khóa dòng/khóa kỳ, cách hiện thực View, lựa chọn lịch chạy tác vụ chi tiết, chỉ mục/partition/thủ tục, kiểm thử đồng thời SQL Server và đo dưới 50 ms là ghi chú triển khai GĐ2, không phải điều kiện hoàn tất GĐ1. Đã tách hướng triển khai khỏi quy trình/ràng buộc GĐ1 sang [ghi chú GĐ2](../gd2/ghi-chu-trien-khai.md), giữ nguyên văn các đoạn cũ để truy vết. Chưa cài đặt/kiểm chứng cơ chế đó. Các mục GĐ2 trong kế hoạch phân công vẫn giữ đúng giai đoạn.
- Các kiểm tra mô hình đã chạy chỉ đối chiếu tính nhất quán thiết kế, không thay thế demo hoặc phân tích hiệu năng GĐ2.
- GD1-15 từng mở lại vì bốn hàng chưa có nhãn. Đã hoàn thiện theo lượt nghiệp vụ: A-F09 ~2.600/Cao, CN-B08 ~2.700/Cao, C-F10 ~100/Trung bình, C-F14 ~2.700/Cao mỗi ngày. Không coi đây là số lần chạy tác vụ; B08/C14 không cộng trùng.
- GD1-16 chỉ chỉnh căn cứ/đơn vị của số liệu tần suất, tách số đề cho và giả định. Chưa lập kịch bản tải, ma trận truy vấn hoặc benchmark.

## Thứ tự xử lý

| Mã | Nội dung | Phụ trách/phối hợp | Loại | Trạng thái |
|---|---|---|---|---|
| GD1-01 | Trừ buổi PT và hủy đúng hạn mâu thuẫn | B, C; A dùng kết quả | Mâu thuẫn | Đã kiểm tra |
| GD1-02 | Chọn lịch để trừ khi có nhiều lịch PT trong ngày | B, C | Thiếu quy tắc | Đã kiểm tra |
| GD1-03 | Giữ/trừ quyền lợi theo từng gói đã mua | B, C | Thiếu thiết kế liên kết | Đã kiểm tra |
| GD1-04 | Luồng vé Spa tại cổng/quầy | B, C | Luồng bổ sung đã bỏ | Đã kiểm tra |
| GD1-05 | Đổi chức vụ phải giữ lịch sử và đồng bộ hiện tại | A | Thiếu quy trình | Đã kiểm tra |
| GD1-06 | Kiểm tra chức vụ PT tại thời điểm buổi tập | A, C | Thiếu dữ liệu giao diện | Đã kiểm tra |
| GD1-07 | Chặn sức chứa và báo cáo lấp đầy chưa khớp | A, C, D | Cần thống nhất | Đã kiểm tra |
| GD1-08 | Doanh thu theo gói trước hay sau giảm giá | D | Cần thống nhất | Đã kiểm tra |
| GD1-09 | Phương án lưu điểm/hạng còn để sang GĐ2 | B, D | Thiếu thiết kế | Đã kiểm tra |
| GD1-10 | Hoàn thiện thiết kế logic A | A | Thiếu đầu ra | Mở lại |
| GD1-11 | Hoàn thiện ERD và thiết kế logic B | B | Thiếu đầu ra | Mở lại |
| GD1-12 | Kết luận BCNF của ChiTietHoaDon chưa đúng | D | Lỗi dạng chuẩn | Đã kiểm tra |
| GD1-13 | Dấu trong bảng tầm ảnh hưởng D chưa đúng | D | Lỗi ràng buộc | Mở lại |
| GD1-14 | Cú pháp ERD Mermaid D | D | Lỗi cú pháp | Đã kiểm tra |
| GD1-15 | Ngưỡng cao/trung bình/thấp khác nhau | A, B, C, D | Thống nhất đơn vị nghiệp vụ, ngưỡng và 52 nhãn | Mở lại |
| GD1-16 | Tải trung bình và tải đồng thời check-in bị lẫn | C chủ trì; cả nhóm | Tách số đề/giả định, trung bình/đỉnh/đồng thời và dòng log | Đã kiểm tra |
| GD1-17 | Ghép ERD, lược đồ và kết luận toàn CSDL | A, B, C, D | Đã ghép 20 quan hệ, 33 FK, 14 ràng buộc tổng và kết luận 2NF | Mở lại |

01–09 giải quyết nghiệp vụ và dữ liệu trước khi hoàn thiện 10–11. Các mục 12–14 có thể làm độc lập. 15–16 hoàn thiện số liệu chung. 17 làm sau cùng. Khi xử lý một mục, giải quyết kèm quyết định liên quan trong bảng cuối tài liệu.

## Chi tiết và điều kiện hoàn tất

### GD1-01 Trừ buổi PT và hủy đúng hạn

- Nguồn: [B QT-B3](b/quy-trinh.md), [C QT-C1 và QT-C5](c/quy-trinh.md), [BR-C18](c/business-rules.md), [C-R10](c/thiet-ke-logic.md), quyết định #2/#8/#11.
- Lỗi ban đầu: quét cổng đã trừ PT nhưng hủy trước hai giờ lại bắt cờ chưa trừ; hai quy tắc mâu thuẫn.
- Quyết định 08/11/2026: PT chỉ dùng đặt lịch; tự trừ đúng một buổi từ gói đã giữ tại giờ hẹn, dù vào cổng/tham dự hay không. Hội viên chỉ được hủy khi còn ít nhất hai giờ, không mất buổi; quá mốc từ chối hủy. Cổng dùng thẻ/vé riêng, không đọc/trừ PT.
- Đã cập nhật: quyết định chung; quy trình, tần suất B/C; business rules, logic, README C; điều kiện bán PT tại D; phần liên quan trong phân công. Vắng mặt vẫn không tính hoa hồng theo quy trình A.
- Kết quả: Đã kiểm tra nhất quán nghiệp vụ và các tình huống ở bảng dưới. Liên kết lịch–gói được bổ sung và kiểm tra tại GD1-03.

### GD1-02 Nhiều lịch PT trong ngày

- Lỗi ban đầu: cổng cần chọn duy nhất lịch PT để trừ, nên nhiều lịch trong ngày có thể làm từ chối quyền vào hợp lệ.
- Quyết định 08/11/2026 loại bỏ sự phụ thuộc: cổng chỉ xét thẻ/vé vào; tác vụ xử lý mỗi lịch tại chính giờ hẹn của lịch đó.
- Kết quả: Đã kiểm tra 0/1/nhiều lịch cho cùng một quyền vào đều không làm thay đổi kết quả ở cổng. Không cần thêm giới hạn một lịch/ngày để sửa lỗi này. Mỗi lịch phải gắn đúng gói và giữ đủ buổi theo GD1-03.

### GD1-03 Giữ và trừ đúng gói đã mua

- Nguồn: [thiết kế PT B–C](b/goi-pt.md), [ERD C](c/erd.md), [logic C](c/thiet-ke-logic.md), hợp đồng trong [phân công](../phan-cong.md).
- Vấn đề ban đầu: chưa công bố khóa lần mua gói và dữ liệu phân bổ quyền lợi. Cờ đã trừ không cho biết đã dùng gói nào; tổng buổi không đủ để xử lý nhiều gói khác hạn.
- Quyết định: PT mua thêm dùng ngay từ ngày mua; mỗi lần mua có MaGHV, số buổi và hạn riêng, không chuyển buổi/gia hạn gói cũ. Gym/Yoga theo hạn vẫn gia hạn nối tiếp theo quy tắc đã chốt.
- Thiết kế: LichHenPT lưu MaGHV bắt buộc, suy ra hội viên qua GoiHoiVien; không lưu MaHV lặp trong lịch. Chọn gói của hội viên đủ hạn tại ngày hẹn, còn buổi chưa giữ, ưu tiên hết hạn sớm nhất rồi MaGHV. Liên kết gói không đổi sau khi đặt.
- Giữ/trừ: số buổi khả dụng = số buổi còn lại − số lịch Đã đặt/chưa trừ của chính gói, gồm cả lịch quá giờ chưa xử lý. Hủy đúng hạn giải phóng giữ chỗ, không cộng số dư; đến giờ giảm số dư và bật cờ cùng giao dịch. Đặt/hủy/trừ đều kiểm tra lại trạng thái khi ghi nhận; chạy lại không trừ lần hai. Tác vụ chạy bù xét hạn tại ngày hẹn.
- Đã cập nhật: quyết định và phân công; quy trình B/C/D; tần suất B; thiết kế PT B–C; ERD, logic, business rules và README C. Ảnh ERD C cũ giữ làm lịch sử, sơ đồ hiện hành là erd.md. Thiết kế B đầy đủ vẫn thuộc GD1-11.
- Kết quả: Đã kiểm tra bằng mô hình các tình huống trong thiết kế PT: nhiều gói khác hạn, hết buổi khả dụng, hủy đúng mốc hai giờ, từ chối hủy muộn, trừ đúng gói/chống lặp, giữ lịch quá giờ, chạy bù sau hạn, mua mới dùng ngay và từ chối sai chủ. Kiểm tra thêm gói cùng hạn, hai biên ngày và 50 lần hai yêu cầu tranh một buổi qua khóa mô hình; đều đạt. Đây chưa phải kiểm thử SQL Server.

### GD1-04 Luồng vé Spa tại cổng/quầy

- Nguồn ban đầu: B QT-B3 có kiểm tra Spa tại quầy sau khi vé đã trừ ở cổng, nhưng chưa có dữ liệu đối soát. Nhận xét cũ xuất phát từ luồng bổ sung này, không phải yêu cầu bắt buộc của đề.
- Đối chiếu đề: phần Lịch hẹn & Check-in yêu cầu ghi nhận phiên vào–ra, vấn đề 1 yêu cầu xác thực quyền vào cổng; không yêu cầu xác nhận Spa tại quầy.
- Quyết định của người dùng, áp dụng toàn dự án: kiểm tra thẻ/vé và tính lượt vào chỉ tại cổng; không có điểm kiểm tra/trừ lượt dịch vụ tại quầy. PT giữ quy tắc trừ theo giờ hẹn đã chốt ở GD1-01. Vé Spa dùng làm quyền vào thì giảm từ 1 về 0 cùng giao dịch tạo CheckIn; không kiểm tra/trừ tiếp ở quầy. Giữ quy tắc ưu tiên thẻ theo hạn hợp lệ; vào bằng thẻ Gym/Yoga không tự trừ vé Spa. Không theo dõi riêng sử dụng Spa bên trong.
- Đã cập nhật: quyết định #8, quy trình và tần suất B, quy trình và README C. Không thêm bảng/cờ sử dụng Spa hoặc liên kết để đối soát cổng–quầy. Giữ nguyên mã 04 để truy vết việc loại bỏ yêu cầu thừa.
- Kết quả: Đã đối chiếu tài liệu cho các trường hợp dùng vé Spa tại cổng, quét lặp trong phiên đang mở, ra rồi vào lại và vào bằng thẻ theo hạn. Trừ vé/tạo phiên cùng thành công hoặc cùng hoàn tác; vé đã hết không cấp lượt vào mới. Kiểm thử thực thi thuộc GĐ2.

### GD1-05 Quy trình đổi chức vụ

- Nguồn: [A QT-A03/A04](a/quy-trinh.md), [ánh xạ ERD A](a/README.md), quyết định #13 và hợp đồng A–C.
- Vấn đề ban đầu: lưu chức vụ hiện tại và lịch sử đã được chọn nhưng chưa có quy trình đóng/mở giai đoạn, đồng bộ hiện tại; README vẫn hỏi lại nơi lưu chức vụ.
- Đã chốt: DATE và tính cả hai biên. Giai đoạn cũ kết thúc ngày 14 thì mới bắt đầu ngày 15. Bấm đổi chức vụ/chi nhánh trong ngày D thì cũ kết thúc hết D, mới bắt đầu D+1 theo Asia/Saigon. Không hồi tố/đổi giữa ngày; giai đoạn một ngày hợp lệ.
- Thiết kế: kiểm tra lại ngày/lịch sử và ghi kết thúc giai đoạn cũ + tạo giai đoạn mới cùng giao dịch; không sửa chức vụ lịch sử cũ. Chỉ cho một thay đổi đang chờ của mỗi nhân viên, tránh hai yêu cầu ghi đè nhau. Giữ chức vụ hiện tại trong ngày D; khi sang ngày mới đồng bộ từ giai đoạn hiệu lực. Nếu tác vụ chậm, thao tác cần vị trí hiện tại phải đọc/đồng bộ từ lịch sử trước khi dùng. Không coi NgayKetThuc rỗng là đủ để xác định hiện hành.
- Đã cập nhật: quyết định #13; A quy trình/README/ánh xạ phân cấp ERD/tần suất; hợp đồng phân công; trường ChucVu lịch sử và biên ngày trong C. Logic đầy đủ A còn ở GD1-10; chính sách xử lý lịch PT tương lai bị ảnh hưởng đã bổ sung ở GD1-06.
- Kết quả: Đã kiểm tra mô hình ngày biên, đổi trong ngày có hiệu lực hôm sau, không nhận thêm yêu cầu khi đang chờ, giai đoạn một ngày, chức vụ quá khứ, đồng bộ chạy chậm/chạy lại, cuối tháng/năm/năm nhuận và mốc nửa đêm Việt Nam. Chưa kiểm thử khóa/tác vụ SQL Server.

### GD1-06 Chức vụ PT tại thời điểm lịch hẹn

- Nguồn: [A QT-A03/A04](a/quy-trinh.md), [C-R07](c/thiet-ke-logic.md), [BR-C08/BR-C16](c/business-rules.md), hợp đồng A–C và quyết định #12–#13.
- Vấn đề ban đầu: C đọc chức vụ hiện tại, hợp đồng thiếu chức vụ lịch sử, chưa xử lý lịch đã đặt khi PT đổi vị trí. GD1-05 đã bổ sung ChucVu lịch sử và biên ngày.
- Đã chốt GD1-06: chỉ dạy khi có chức vụ PT tại đúng chi nhánh công tác trong toàn bộ buổi hẹn. Nếu thay đổi từ ngày mai làm lịch chưa hủy không còn phù hợp thì từ chối đổi vị trí, trả mã lịch/ngày giờ/chi nhánh/lý do; không ghi lịch sử mới hoặc sửa lịch, không trừ/hoàn buổi.
- Phạm vi kiểm tra: các lịch chưa hủy kết thúc sau 00:00 ngày hiệu lực, gồm buổi kéo qua mốc này. Buổi kết thúc đúng mốc và lịch quá khứ không chặn thay đổi; hoa hồng cũ vẫn dùng lịch sử. Lịch đã hủy không chặn. Giải quyết lịch theo luồng hợp lệ (hủy vẫn trước ít nhất hai giờ) rồi gửi lại, tính lại ngày hiệu lực từ ngày gửi thành công.
- Chống xung đột: đặt trước thì đổi vị trí thấy lịch mới; đổi trước thì đặt phải hợp lệ với vị trí mới, gồm cả giai đoạn chờ. Không tự hủy/đổi HLV/chuyển lịch. Cách khóa cụ thể thuộc ghi chú GĐ2.
- Đã cập nhật: quyết định #12, phân công; A quy trình/README/tần suất; C quy trình/business rules/logic/README/tần suất; B thiết kế giao dịch PT, quy trình và tần suất để thống nhất thứ tự khóa. Không thay đổi cấu trúc bảng ngoài hợp đồng chức vụ lịch sử đã bổ sung ở GD1-05.
- Kết quả: kiểm tra mô hình đổi PT thành quản lý, chuyển chi nhánh, từ chối không làm đổi dữ liệu, lịch đã hủy/đã hoàn thành/hôm nay, đặt theo vị trí chờ, biên nửa đêm, lịch kéo qua biên, hai thứ tự đặt–đổi và 50 lượt hai yêu cầu đồng thời qua khóa mô hình; đều đạt. Chưa kiểm thử khóa SQL Server. Tập giờ đặt hợp lệ (#10) khi rà mục 06 còn chờ; nay đã chốt giờ tròn ở mục 17, giữ nguyên quy tắc biên ngày.

### GD1-07 Sức chứa và lấp đầy

- Nguồn: [C-R05](c/thiet-ke-logic.md), [D-R08](d/thiet-ke-logic.md), [D QT-D5](d/quy-trinh.md), [ERD D ràng buộc 8](d/erd.md), quyết định #15.
- Vấn đề ban đầu: C chỉ chặn sức chứa nếu nhóm chọn; D coi số người không vượt sức chứa là bắt buộc. Đề yêu cầu báo cáo lấp đầy, không bắt buộc chặn cổng.
- Người dùng chốt: chỉ báo cáo lấp đầy, không chặn cổng. Bỏ kiểm tra SucChua khỏi luồng mở cổng; không thêm cảnh báo tại cổng hoặc điều kiện từ chối theo tỷ lệ.
- Công thức: N(T) = số phiên cùng chi nhánh có TGVao <= T và (TGRa rỗng hoặc TGRa > T). Tỷ lệ = 100 × N(T) / SucChua nguyên dương; được vượt 100%, không cắt ngọn. Vào đúng T có tính, ra đúng T không tính. Báo cáo theo giờ chụp đầu khung, không lấy tổng lượt vào làm số người đồng thời. Sức chứa thiếu/không hợp lệ thì không tính tỷ lệ.
- Quy ước mô phỏng được ghi rõ, không phải yêu cầu bắt buộc của đề: giờ mở < giờ đóng trong cùng ngày Asia/Saigon, cấu hình giờ/sức chứa cố định trong kỳ dữ liệu. Giữ luồng tự đóng lượt quên ra tại giờ đóng cửa của ngày vào; thời gian đó là ước tính. Chạy bù đúng ngày, không ghi đè giờ ra thực tế hoặc phiên mới. D đọc SoHieuCuaRa có sẵn để thống kê lượt tự đóng; kỳ còn lượt quá hạn chưa đóng là tạm tính. Qua đêm/24 giờ và lịch sử thay đổi sức chứa ngoài phạm vi mô phỏng hiện tại.
- Đã cập nhật: quyết định #15, hợp đồng; A kiểm tra miền sức chứa/giờ; C quy trình/business rules/logic/ERD/README/tần suất; D quy trình/logic/ràng buộc ERD/README/tần suất. D-R08 chỉ kiểm tra sức chứa hợp lệ, không còn điều kiện giới hạn số người.
- Kết quả: kiểm tra mô hình biên vào/ra, tách chi nhánh, không có người, mẫu số sai, tỷ lệ 101%, lượt quá hạn tạm tính, tự đóng/chạy bù/chạy lặp, giữ giờ ra thực tế, không đóng nhầm phiên ngày mới và phân biệt ảnh chụp với tổng lượt; đều đạt. Chưa kiểm thử SQL Server/hiệu năng.

### GD1-08 Doanh thu theo gói và giảm giá

- Nguồn: [D QT-D3/QT-D5](d/quy-trinh.md), [quy tắc phân bổ trong thiết kế logic D](d/thiet-ke-logic.md#phân-bổ-giảm-giá-và-doanh-thu-dòng-gd1-08), quyết định #18.
- Vấn đề ban đầu: tổng hóa đơn dùng doanh thu sau giảm nhưng báo cáo từng gói cộng thành tiền trước giảm nên hai thước đo không khớp.
- Người dùng chốt: doanh thu sau giảm, chia giảm giá theo tỷ lệ thành tiền từng dòng. Phân bổ trên toàn hóa đơn gồm cả gói và sản phẩm rồi mới lọc/nhóm; trọng số dòng là SoLuong × DonGia chụp lúc bán.
- Làm tròn: dùng đồng nguyên; lấy phần nguyên G × A_i / S, chia các đồng còn thiếu cho dòng có phần dư lớn nhất, hòa thì STT tăng dần. Tổng giảm dòng đúng GiamGia, tổng doanh thu dòng đúng TongTien, không dòng nào âm. Tổng tiền hàng bằng 0 chỉ nhận giảm 0 và không chia; giảm toàn bộ cho doanh thu 0. Không dùng FLOAT, không cắt phần lẻ làm lệch tổng.
- Dữ liệu: GiamGiaDong/DoanhThuDong là giá trị tính ra, không thêm cột/bảng lưu; dùng chung quy tắc khi hiển thị hóa đơn/báo cáo. Không lấy giá danh mục mới để tính lại, không trừ giảm lần hai hoặc cộng TongTien lặp sau khi nối chi tiết. Tổng gói + sản phẩm khớp tổng hóa đơn; riêng gói không cần bằng toàn hóa đơn có sản phẩm. B vẫn tính điểm/hạng đúng một lần trên TongTien, không tính riêng từng dòng.
- Đã cập nhật: quyết định #18, phân công; D quy trình/logic/ràng buộc ERD/README/tần suất; B quy tắc đầu vào điểm/hạng. GD1-12 về ThanhTien dẫn xuất/BCNF và GD1-13/14 vẫn Mở, không coi đã sửa các mục đó.
- Kết quả: kiểm tra mô hình ví dụ Gym/Yoga, gói kèm sản phẩm, nhiều đơn vị/dòng, đồng dư bằng nhau, dòng/hóa đơn 0đ, không giảm/giảm toàn bộ, thứ tự dòng đảo ngược và từ chối đầu vào sai; thêm 2.836 hóa đơn duyệt tổ hợp/ngẫu nhiên đều giữ đúng tổng và không âm. Chưa kiểm thử kiểu tiền, tràn số hoặc hiệu năng SQL Server.

### GD1-09 Lưu trữ điểm và hạng

- Vấn đề ban đầu: B để thiết kế lưu tổng/điểm/hạng sang GĐ2, chưa đủ đầu ra GĐ1.
- Chốt: xét từng khoản trong 12 tháng gần nhất, không reset đồng loạt; điểm không hết hạn. Ngưỡng 20/50 triệu vẫn cần giảng viên xác nhận, không mở lại quyết định cửa sổ đã chốt.
- Đã thiết kế tại [điểm/hạng B](b/diem-hang.md): HoiVien lưu tổng chi tiêu, điểm, hạng; GhiNhanLoyalty khóa MaHD chống cộng lặp và có cờ loại khỏi tổng xét hạng chống trừ lặp. Hóa đơn là nguồn chủ hội viên/số tiền/ngày mua; không xóa lịch sử khi hết hạn.
- Đồng bộ nhất quán theo hội viên trước thanh toán/đọc hạng và bằng tác vụ nền. Mỗi thao tác cập nhật tổng, cờ, điểm/hạng cùng giao dịch; lỗi hoàn tác. Chạy bù xử lý tất cả khoản đến hạn còn chưa loại, không cộng toàn bộ 12 tháng mỗi lần mua. Quy ước mốc hết hạn, năm nhuận và dữ liệu quá hạn khi xử lý bù được mô tả rõ.
- Đã cập nhật B quy trình/tần suất, quyết định #4, phân công và giao diện giao dịch D. Thiết kế B toàn bộ vẫn thuộc GD1-11; không kết luận HoiVien BCNF vì có tổng/hạng dư thừa.
- Kết quả: kiểm tra mô hình cửa sổ trượt, hết hạn vẫn giữ điểm, hóa đơn lặp, loại khoản lặp, chạy bù, đúng mốc giờ/năm nhuận, hai thứ tự thanh toán–tác vụ, khoản cũ xử lý muộn và hóa đơn 0đ; đều đạt. Khóa/rollback và hiệu năng SQL Server chưa được kiểm thử.

### GD1-10 Thiết kế logic A

- Đã hoàn thiện [thiết kế logic A](a/thiet-ke-logic.md) và [ERD hiện hành](a/erd.md), giữ drawio cũ làm bản quan niệm trước phần chốt kỳ.
- Có bảy quan hệ: ChiNhanh, TienIch, CungCapTienIch, NhanVien, LichSuCongTac, KyHoaHong, ChiTietHoaHong; công bố PK/AK/FK, PTH/dạng chuẩn, mô tả thuộc tính và ràng buộc theo bối cảnh–nội dung–tầm ảnh hưởng. Phân cấp nhân viên ánh xạ ChucVu; cung cấp tiện ích lưu khoảng thời gian riêng.
- Chốt #14: kỳ đã chốt giữ nguyên mức và danh sách buổi, không đọc mức hiện tại để tính lại. Kỳ chưa chốt dùng mức tại lúc chốt. Không lưu lặp số buổi/thành tiền dẫn xuất; xem tổng từ mức và chi tiết kỳ. Khóa kỳ, kiểm tra đủ nguồn đã phân loại, ghi kỳ/chi tiết cùng giao dịch; chạy lại không ghi thêm. Bảo vệ nguồn lịch C và lịch sử công tác của kỳ đã chốt.
- Đồng bộ quyết định, A quy trình/README/tần suất, C quy trình/logic và thứ tự khóa giao dịch PT. Không mở lại kỳ trong phạm vi hiện tại; thiết kế lương hồi cứu ngoài phạm vi này.
- Đã kiểm tra mô hình giữ mức cũ khi đổi mức, chốt lặp, tách chi nhánh, từ chối nguồn chưa hoàn tất không đổi dữ liệu, kỳ mới dùng mức mới, kỳ rỗng và khoảng ngày giao nhau; đối chiếu đủ thuộc tính bảy bảng giữa ERD/logic. Chưa kiểm thử SQL Server hoặc render trực quan ERD mới.

### GD1-11 ERD và thiết kế logic B

- Đã hoàn thiện [thiết kế logic](b/thiet-ke-logic.md), [ERD](b/erd.md) và [README B](b/README.md), tích hợp PT/quyền vào/điểm/hạng và liên kết hóa đơn.
- Người dùng chốt: mỗi gói tham chiếu dòng hóa đơn. Dòng số lượng n tạo n MaGHV với DonVi 1..n; AK (MaHD,STT,DonVi) chống trùng, FK ghép tới ChiTietHoaDon. Hóa đơn/gói/điểm/kho cùng thành công hoặc cùng rollback; chạy lại MaHD không tạo thêm. Dòng sản phẩm không tạo gói.
- Chuẩn hóa: chủ hội viên/mã danh mục suy ra qua hóa đơn/dòng, không lưu trùng trên GoiHoiVien cơ sở; vGoiHoiVien giữ giao diện đọc cho C. Ba quan hệ danh mục/gói/ghi nhận đạt BCNF theo PTH đã nêu; HoiVien chỉ đạt 2NF do lưu hạng dư thừa có chủ đích, đã nêu phương án chuẩn hóa và ràng buộc đồng bộ. Không tuyên bố toàn B BCNF.
- Có PK/AK/FK, PTH/dạng chuẩn, miền thuộc tính, ràng buộc và bảng ảnh hưởng; đồng bộ B/C/D, quyết định và phân công. Bảo lưu/đổi điểm chưa triển khai; ngưỡng hạng còn cần cô xác nhận, giữ nguyên ghi chú.
- Đã kiểm tra mô hình dòng mua hai gói, dòng sản phẩm, xử lý lại, rollback, mua khác MaHD và một ghi nhận điểm/hóa đơn; đối chiếu cấu trúc ERD/logic và liên kết tài liệu. Chưa chạy SQL Server hoặc render trực quan ERD.

### GD1-12 Dạng chuẩn ChiTietHoaDon

- Lỗi ban đầu: lưu ThanhTien = SoLuong × DonGia nhưng bỏ sót PTH (SoLuong,DonGia) → ThanhTien khi kết luận BCNF; vế trái không phải siêu khóa.
- Người dùng chốt: bỏ ThanhTien khỏi bảng cơ sở, chỉ tính khi đọc. Giữ đơn giá chụp lúc bán; không dùng giá danh mục hiện tại.
- Đã cập nhật lược đồ, PTH/giải thích dạng chuẩn, mô tả thuộc tính, D-R02, ERD, quy trình/README D, quyết định và phân công. TongTien = SUM(SoLuong × DonGia) − GiamGia; phân bổ giảm giá sau GD1-08 giữ nguyên. Không bỏ TongTien đầu hóa đơn theo lập luận PTH đơn dòng.
- Kết quả: đối chiếu không còn ThanhTien trong bảng cơ sở/thuộc tính ERD; kiểm tra phép tính nhiều đơn vị, dòng 0đ, ví dụ phân bổ Gym/Yoga khớp 1.350.000đ. Bảng cơ sở BCNF theo PTH đã nêu; không áp kết luận đó cho view có cột tính ra. Chưa kiểm thử SQL Server; cú pháp Mermaid D vẫn thuộc GD1-14.

### GD1-13 Bảng tầm ảnh hưởng D

- Đã rà toàn bộ D-R01 đến D-R08 trong [thiết kế logic D](d/thiet-ke-logic.md), giữ đủ mã cũ.
- Sửa xóa ChiTietHoaDon tại điều kiện đúng một mã hàng thành −; xóa GiaoDichKho tại cân bằng tồn thành +. Bổ sung ảnh hưởng khi chuyển mã hóa đơn/chi nhánh/sản phẩm, xóa dòng cuối, thay đổi ghi nhận Loyalty và tổng B.
- Tách miền giá trị khỏi bất biến lịch sử, số dòng tối thiểu khỏi cấm thêm/sửa sau thanh toán; điều kiện hàng đang bán/đủ kho chỉ xét tại lúc bán, không áp hồi tố. Dấu − không cấp quyền xóa hoặc bỏ qua FK.
- Làm rõ số dư đầu kỳ đã ghi nhật ký mở đầu, không cộng hai lần; đối soát xuất gộp theo hóa đơn/sản phẩm để xử lý nhiều dòng cùng mặt hàng. Công bố FK ghép nhật ký → cặp tồn kho đúng QT-D2; xóa cha có con bị chặn bởi FK riêng.
- Kiểm tra mô hình xóa dòng giữ điều kiện loại hàng, sửa hai mã làm sai, nhập 10/xuất 2/tồn 8 và xóa từng nhật ký, chuyển xuất sang chi nhánh khác, gộp hai dòng 1+2, xóa dòng cuối và mất ghi nhận Loyalty; đều khớp dấu đã ghi. Đồng bộ README/ghi chú ERD. Chưa chạy giao dịch SQL Server.

### GD1-14 Cú pháp ERD D

- Nguồn: [ERD D](d/erd.md), CHI_NHANH/GOI_DICH_VU dùng dấu ; ngăn thuộc tính.
- Đã sửa khai báo mỗi thuộc tính một dòng; chuẩn hóa bốn thực thể tham chiếu cùng kiểu trình bày. Không thay khóa, quan hệ hoặc chính sách nghiệp vụ.
- Kiểm tra Mermaid 10.9.3: tái hiện lỗi parser tại dấu ; của CHI_NHANH, bản sửa parse và render toàn sơ đồ thành công. Đã xem ảnh chụp kết quả, đủ chín thực thể và các thuộc tính; xuất [SVG](d/erd.svg), liên kết trong README. Renderer dùng thư viện tạm, không thêm phụ thuộc dự án.
- Kết quả: Đã kiểm tra cú pháp và render; không chạy SQL Server vì thay đổi chỉ liên quan sơ đồ.

### GD1-15 Ngưỡng phân loại tần suất

- Người dùng chốt: thấp <100, trung bình từ 100 đến dưới 1.000, cao >=1.000 lần/ngày. Quy ước toàn hệ thống ghi trong quyết định chung và phân công; cả bốn bảng cùng áp dụng.
- A-F05 20 lần/ngày chuyển từ Trung bình sang Thấp. Tuần/tháng quy đổi trung bình theo 7/30 ngày và số đơn vị áp dụng, vẫn giữ kỳ gốc/giờ chạy dồn. Không mặc định tác vụ theo kỳ luôn thấp.
- Đếm theo đơn vị nghiệp vụ rõ ở từng hàng: A-F09 ~2.600 lượt đối chiếu/ngày → Cao; CN-B08/C-F14 ~2.700 lượt trừ lịch/ngày → Cao; C-F10 ~100 lượt xác nhận vắng/ngày → Trung bình. Một đợt có thể xử lý nhiều lịch; số đợt và số câu SQL không phải số lượt nghiệp vụ. Chi phí ghi riêng.
- Đối chiếu 2.900 lịch = 2.600 hoàn thành + 200 hủy đúng hạn + 100 vắng; trừ 2.700 = 2.600 + 100, không cộng trùng lúc đóng lịch. Đây là cơ cấu giả định, chưa phải đo đạc. Không cộng các hàng nghiệp vụ bao hàm nhau giữa B/C/D thành tổng giao dịch.
- Kết quả: đã bổ sung bốn hàng sau lần mở lại, đối chiếu đủ 52 nhãn theo ngưỡng và đơn vị công bố. Chốt hướng xử lý PT theo đợt nhưng vẫn trừ tại giờ hẹn, không chờ hết giờ/cuối ngày. Đo tải và cơ chế tác vụ thuộc GĐ2. Tần suất tăng thêm do vé mới và cách diễn giải tải cổng tiếp tục ở GD1-16; không coi số liệu giả định là đo thực tế.

### GD1-16 Tải check-in

- Nguồn: de.md, vấn đề 1 và 3; [giả định chung và quy ước tải](../quyet-dinh.md#phân-biệt-tải-cổng-và-giả-định--gd1-16); bảng tần suất B/C/D và phân công.
- Đã sửa lỗi diễn giải: giữ nguyên yêu cầu >10.000 lượt quét đồng thời toàn hệ thống trong 17h–20h. Mốc ~10.000 lượt cộng dồn/3 giờ chỉ là giả định minh họa G1, không thay thế hoặc chứng minh đáp ứng yêu cầu này.
- Phân biệt lượt trong khoảng, tốc độ trung bình, tốc độ đỉnh, yêu cầu xử lý đồng thời và số người có mặt. 10.000/180 ≈55,56 lượt/phút ≈0,926 lượt/giây là trung bình minh họa; đã bỏ suy luận đỉnh vài lượt/giây.
- Mốc 25.000 lượt vào/ngày tạo ~750.000 dòng CheckIn/tháng. Quét ra cập nhật dòng cũ; tối đa ~50.000 thao tác vào/ra/ngày không phải 50.000 dòng mới. Quét lỗi/lặp không tạo phiên mới; tự đóng thay cho quên quét ra. Đây chưa phải tổng yêu cầu để kiểm thử tải.
- G11–G12 bổ sung giả định cơ cấu: 300 gói dài hạn +60 gói một tháng +30 vé lượt =390 lượt mua Gym/Yoga; 30 vé gồm 10 lượt mua lại. 230 lượt dùng vé =200 Spa +30 Gym/Yoga nằm trong lượt vào cổng. Giữ 690 đơn vị gói/vé/ngày; 690 hóa đơn có gói +750 có sản phẩm −40 có cả hai =1.400 hóa đơn riêng biệt theo giả định một đơn vị gói/hóa đơn. Không biến giả định đó thành hạn chế bán nhiều gói.
- Cập nhật CN-B05/B09, thống kê đọc/ghi B, C-F01/F02 và quy mô log, D-F06/F07, quyết định chung và phân công. Các nhãn tần suất không đổi; CN-B09 tăng giả định từ 200 lên 230, vẫn Trung bình.
- Kết quả: đối chiếu đủ 52 nhãn và các phép tính/quan hệ bao hàm. Mục 16 hoàn tất ở mức tài liệu GĐ1. Mục tiêu tra cứu <50 ms chưa được đo; cách cụ thể hóa “đồng thời”, tải đỉnh, yêu cầu lỗi/lặp và kiểm thử tải phải được làm rõ ở GĐ2, không tự hạ yêu cầu của đề.

### GD1-17 Ghép và kiểm tra toàn CSDL

- Đầu ra: [hồ sơ GĐ1 tổng](README.md), [ERD tổng](erd-tong.md), [SVG phóng to](erd-tong.svg), [lược đồ/khóa/PTH/ràng buộc tổng](thiet-ke-tong.md). Không viết hoặc chạy triển khai GĐ2.
- Ghép 20 quan hệ (A7/B4/C4/D5), đủ thuộc tính và module sở hữu; đối chiếu 33 FK gồm 5 FK ghép. Thực thể tham chiếu chỉ có một bản trong mô hình tổng. Từ điển thuộc tính đầy đủ dẫn chiếu tới từng module.
- Sửa các mô tả cũ còn coi MaHV/MaGoi là cột GoiHoiVien ở C/phân công: bảng cơ sở liên kết dòng hóa đơn, chủ/loại đọc qua vGoiHoiVien. Bổ sung ảnh hưởng của nguồn HoaDon/ChiTietHoaDon cho ràng buộc C; giữ FK LichHenPT.MaGHV tới bảng cơ sở.
- Người dùng đã chốt QD10: khung cố định một giờ bắt đầu giờ tròn; hai khung liền nhau hợp lệ, giờ lệch phút bị từ chối. Đồng bộ quy trình, ràng buộc và quyết định; không đổi thời điểm trừ/hủy PT.
- Bổ sung T-R01–14: đủ bối cảnh, nội dung, bảng ảnh hưởng liên module cho mua gói, thanh toán, cổng, PT, lịch sử, hoa hồng, doanh thu/điểm, kho, lấp đầy và đánh giá. Giữ dấu vết cả 17 mục và các bảng nội module.
- Chuẩn hóa: 19 quan hệ BCNF theo PTH công bố; HoiVien 2NF do TongChiTieu12T → Hang ngoài khóa. Toàn lược đồ bảo đảm 2NF; không đổi phương án lưu hạng đã chốt và không tuyên bố toàn CSDL BCNF.
- Kiểm tra: thuộc tính ERD khớp từng cột lược đồ; tất cả FK trỏ cột tồn tại; đủ 20 tập khóa/PTH, 14 ràng buộc tổng; mô hình biên giờ tròn/liền khung/hủy hai giờ; liên kết tài liệu hợp lệ; Mermaid 10.9.3 phân tích và xuất SVG thành công, đã xem hình kiểm tra hiển thị.
- Giới hạn còn công bố: ngưỡng hạng 20/50 triệu cần giảng viên xác nhận; bảo lưu chưa đưa vào phạm vi; README thiếu tên/MSSV của một số người. Không tự điền danh tính hoặc coi các giả định đã được duyệt. Cài đặt/đo tải thuộc GĐ2.
- Kết quả: **Đã kiểm tra mục 17 ở mức hồ sơ GĐ1**.

## Những quyết định còn mở phải mang theo

Danh sách gồm quyết định còn mở và quyết định đã chốt được giữ lại để truy vết; không mặc định chúng là lỗi mới. Mã QD trỏ số trong quyet-dinh.md; mã B trỏ hàng ở phần Điểm cần xác nhận của B.

| Nguồn | Cần chốt hoặc xác nhận | Xử lý cùng |
|---|---|---|
| QD04, B1 | Đã chốt 12 tháng gần nhất và giữ điểm; chỉ ngưỡng 20/50 triệu còn cần cô xác nhận | 09 đã kiểm tra; 11 tích hợp |
| QD09, B2–B3 | Đã chốt Gym/Yoga theo hạn nối tiếp sau gói cùng loại cuối cùng đã mua; xét cả hai biên ngày, giữ lịch sử cũ. PT đã chốt mua thêm dùng ngay, mỗi lần mua có số buổi/hạn riêng và lịch gắn MaGHV. Bảo lưu vẫn chưa chốt | 03, 11 |
| QD10 | Đã chốt 08/11/2026: mỗi buổi một giờ, bắt đầu giờ tròn (phút/giây bằng 0); không nhận 17h30. Đồng bộ quyết định và tài liệu A/C/tổng | 17 đã kiểm tra |
| QD11, B2/B5 | Đã chốt 08/11/2026: tự trừ tại giờ hẹn, hủy trước ít nhất hai giờ, từ chối hủy muộn; vắng mặt không hoa hồng. Liên kết và giữ/trừ đúng gói đã thiết kế | 01, 03 đã kiểm tra |
| QD12 | Đã chốt chỉ dạy tại chi nhánh công tác, chức vụ PT phải bao phủ buổi; chặn đổi vị trí làm lịch chưa hủy không phù hợp | 06 đã kiểm tra |
| QD13 | Đã chốt DATE bao gồm hai biên; mới bắt đầu ngày sau ngày kết thúc cũ; bấm đổi trong ngày có hiệu lực ngày mai. Chức vụ hiện tại đồng bộ khi đến ngày hiệu lực | 05, 06 đã kiểm tra |
| QD14 | Đã chốt giữ nguyên kỳ cũ; lưu mức và danh sách buổi, khóa/bảo vệ nguồn; kỳ chưa chốt dùng mức khi xác nhận | 10 đã kiểm tra |
| QD15 | Đã chốt chỉ báo cáo, không chặn cổng. Công thức tại T và quy ước mô phỏng giờ hoạt động trong ngày/tự đóng ước tính được ghi rõ; chưa mở rộng qua đêm hoặc lịch sử sức chứa | 07 đã kiểm tra |
| QD17 | Phân công tên người, nền tảng/ngôn ngữ/SQL Server; phần lựa chọn triển khai phục vụ GĐ2 | 17 |
| QD18 | Đã chốt doanh thu sau giảm; phân bổ theo tỷ lệ toàn hóa đơn, làm tròn theo đồng/phần dư và giữ tổng khớp | 08 đã kiểm tra |
| B4 | Đã chốt 08/11/2026: vé một lượt hết sau lần vào; quay lại cần lượt mới hoặc thẻ còn hiệu lực. Đã bỏ luồng kiểm tra Spa tại quầy, chỉ kiểm tra tại cổng | 04 đã kiểm tra |
| B6 | Đã xác định xét cả ngày bắt đầu/hết hạn; PT xét tại ngày hẹn, kể cả tác vụ chạy bù sau hạn. Đưa vào thiết kế B đầy đủ | 03 đã kiểm tra; 11 |
| B7 | Chi nhánh đăng ký hội viên dùng cho báo cáo | 11 |
| B8 | Đã chốt liên kết dòng/đơn vị, nguyên tử và chống lặp theo MaHD; hợp đồng đọc qua view | 11 đã kiểm tra; 17 tích hợp |

## Đối chiếu với lần rà trước

| Nhận xét trước | Mã theo dõi |
|---|---|
| A thiếu logic; đổi chức vụ; ngưỡng tần suất | 10; 05; 15 |
| B thiếu ERD/logic; lưu điểm/hạng; giữ/trừ từng gói | 11; 09; 03 |
| C hủy sau trừ; chức vụ lịch sử; nhiều lịch/ngày; dấu vết Spa | 01; 06; 02; 04 |
| D BCNF; tầm ảnh hưởng; Mermaid; doanh thu theo gói | 12; 13; 14; 08 |
| Chung tải check-in; sức chứa; ghép toàn CSDL | 16; 07; 17 |

## Nhật ký xử lý

| Ngày | Mã | Quyết định và tài liệu đã cập nhật | Kiểm tra | Trạng thái |
|---|---|---|---|---|
| 08/11/2026 | Toàn bộ | Lập danh sách, bảo toàn 17 mục và các quyết định còn mở | Đối chiếu danh sách rà trước với tài liệu hiện tại | Mở |
| 08/11/2026 | GD1-01, GD1-02 | Tách PT khỏi quyền vào; trừ tại giờ hẹn; hủy trước ít nhất hai giờ; thẻ 1 tháng/vé một lượt và mua lại khi vào lại. Đồng bộ tài liệu B/C/D, quyết định và phân công | Đối chiếu các tình huống dưới đây; quét lại văn bản để loại quy tắc trừ PT tại cổng | Đã kiểm tra nghiệp vụ; GD1-03/04 vẫn Mở |
| 08/11/2026 | QD09; liên quan GD1-03/11 | Chốt Gym/Yoga theo hạn gia hạn nối tiếp, gồm gói chờ; công bố NgayBatDau cho C; đồng bộ B/C/D và phân công | Kiểm tra nối nhiều gói, hai biên hiệu lực, phân biệt Gym/Yoga, tháng nhuận và điều kiện chống mua đồng thời | Phần Gym/Yoga đã đồng bộ; GD1-03/11 vẫn Mở |
| 08/11/2026 | GD1-03; QD09 PT | Chốt PT mua thêm dùng ngay, từng MaGHV độc lập; đồng bộ hợp đồng B/C, ERD/logic C, quy trình B/C/D và thiết kế giữ/trừ buổi | Kiểm tra mô hình các tình huống PT và 50 lần tranh buổi cuối; đối chiếu văn bản/liên kết tài liệu | Đã kiểm tra mức GĐ1; GD1-04 và 13 mục khác còn Mở |
| 08/11/2026 | GD1-04 | Người dùng chốt chỉ kiểm tra tại cổng; bỏ kiểm tra/trừ Spa ở quầy và yêu cầu dữ liệu phát sinh từ luồng này; đồng bộ B/C và quyết định #8 | Đối chiếu đề và các tình huống vé tại cổng; không bổ sung bảng cho luồng đã bỏ | Đã kiểm tra mức GĐ1; còn 13 mục Mở |
| 08/11/2026 | GD1-05; QD13 | Chốt hai biên DATE, đổi vị trí từ ngày mai; đồng bộ lịch sử/hiện tại đúng ngày, công bố chức vụ lịch sử cho C | Mô hình biên ngày, thay đổi đang chờ, chức vụ quá khứ, đồng bộ lặp/chạy bù và ngày đặc biệt đều đạt | Đã kiểm tra mức GĐ1; còn 12 mục Mở |
| 08/11/2026 | GD1-06; QD12 | Chặn đổi vị trí làm lịch PT chưa hủy không phù hợp; đọc chức vụ/chi nhánh lịch sử, cùng khóa nhân viên với đặt lịch; đồng bộ A/B/C | Kiểm tra mô hình tình huống chức vụ/chi nhánh/biên ngày, hai thứ tự giao dịch và 50 lần tranh khóa; đều đạt | Đã kiểm tra mức GĐ1; còn 11 mục Mở |
| 08/11/2026 | GD1-07; QD15 | Chỉ báo cáo lấp đầy, bỏ chặn cổng theo sức chứa; đồng bộ công thức, miền sức chứa và quy ước dữ liệu quên quét ra ở A/C/D | Mô hình biên thời gian, 101%, mẫu số sai, tự đóng/chạy bù và tính tạm thời đều đạt | Đã kiểm tra mức GĐ1; còn 10 mục Mở |
| 08/11/2026 | GD1-08; QD18 | Doanh thu sau giảm, phân bổ toàn hóa đơn theo tỷ lệ từng dòng; làm tròn chính xác, giữ giao diện điểm/hạng theo tổng hóa đơn | Mô hình ví dụ/biên/đầu vào sai và 2.836 hóa đơn tổ hợp/ngẫu nhiên đều đạt | Đã kiểm tra mức GĐ1; còn 9 mục Mở |
| 08/11/2026 | GD1-09; QD04 | Chốt cửa sổ 12 tháng, điểm giữ nguyên; bổ sung lưu tổng và ghi nhận theo hóa đơn, giao dịch B–D/chạy bù | Mô hình thời gian/chống lặp/hai thứ tự xử lý đều đạt | Đã kiểm tra mức GĐ1; còn 8 mục Mở |
| 08/11/2026 | GD1-10; QD14 | Hoàn thiện logic/ERD A và lưu kỳ hoa hồng bất biến, đồng bộ nguồn C | Kiểm tra mô hình kỳ/biên ngày và ánh xạ bảy bảng; đạt | Đã kiểm tra mức GĐ1; còn 7 mục Mở |
| 08/11/2026 | GD1-11; B8 | Hoàn thiện logic/ERD B, liên kết dòng hóa đơn và đơn vị gói, view đọc tránh lưu trùng | Mô hình tạo gói/chống lặp/rollback và đối chiếu tài liệu đạt | Đã kiểm tra mức GĐ1; còn 6 mục Mở |
| 08/11/2026 | GD1-12 | Bỏ ThanhTien lưu, tính từ số lượng/đơn giá lịch sử; cập nhật PTH và ERD/công thức | Đối chiếu cấu trúc và tính tiền/phân bổ đạt | Đã kiểm tra mức GĐ1; còn 5 mục Mở |
| 08/11/2026 | GD1-13 | Rà tám ràng buộc D, sửa dấu/cột ảnh hưởng và tách miền khỏi bất biến/điều kiện lúc giao dịch | Mô hình phản ví dụ dòng hàng/kho/điểm khớp bảng | Đã kiểm tra mức GĐ1; còn 4 mục Mở |
| 08/11/2026 | GD1-14 | Sửa dấu phân cách thuộc tính Mermaid D; xuất SVG | Tái hiện lỗi trước sửa, parse/render 10.9.3 thành công sau sửa, xem ảnh kết quả | Đã kiểm tra; còn 3 mục Mở |
| 08/11/2026 | GD1-15 | Thống nhất ngưỡng toàn hệ thống, tách số dòng/chi phí, ghi rõ phần bao hàm của PT | Đối chiếu 52 nhãn, biên ngưỡng, quy đổi kỳ và cơ cấu PT đạt | Đã kiểm tra; còn 2 mục Mở |

## Tình huống đối chiếu quyết định ngày 08/11/2026

Các kết quả sau là kiểm tra tính nhất quán của đặc tả, chưa phải chạy phần mềm.

| Tình huống | Kết quả PT | Kết quả quyền vào |
|---|---|---|
| Lịch 18h; vào 8h bằng vé một lượt, ra 9h; hủy lúc 12h | Hủy hợp lệ, giải phóng buổi giữ, không trừ PT | Vé vào đã hết; không hoàn vé vì hủy PT |
| Lịch 18h; yêu cầu hủy đúng 16h | Hủy hợp lệ, không trừ PT | Độc lập lịch PT |
| Lịch 18h; yêu cầu hủy 16h01 | Từ chối hủy; đến 18h trừ một buổi | Độc lập lịch PT |
| Lịch 18h; không vào cổng hoặc không đến | Đến 18h vẫn trừ; vắng mặt không hoa hồng | Không phát sinh lượt vào thì không trừ vé vào |
| Đã dùng vé vào buổi sáng, quay lại lúc 18h | Lịch chưa hủy vẫn trừ tại 18h | Cần mua lượt mới nếu không còn quyền khác |
| Tác vụ 18h đã trừ thành công, chạy lại | Cờ đã trừ ngăn trừ lần hai | Không tác động CheckIn |
| Tác vụ lỗi rồi chạy bù | Chỉ xử lý lịch đến hạn chưa trừ; cập nhật gói và cờ cùng giao dịch | Không phụ thuộc trạng thái ở trong phòng |
| Có nhiều lịch PT cùng ngày, quét cổng | Mỗi lịch do tác vụ giờ hẹn xử lý riêng | Cổng chỉ kiểm tra thẻ/vé, không chọn lịch |
| Mua PT khi chưa có quyền vào | Chưa hoàn tất mua PT cho đến khi có thẻ/vé mua kèm | Có thể mua thẻ 1 tháng hoặc vé một lượt; không giữ vé cho giờ PT |

## Tình huống đối chiếu gia hạn Gym và Yoga

| Tình huống | Kết quả |
|---|---|
| Chưa có Gym, mua 1 tháng ngày 08/11/2026 | Bắt đầu 08/11, dùng đến hết 07/12/2026 |
| Gym cũ hết 15/11, mua thêm 1 tháng ngày 08/11 | Gói mới bắt đầu 16/11, dùng đến hết 15/12; gói cũ giữ nguyên |
| Gói chờ nói trên chưa bắt đầu, mua thêm 1 tháng | Nối từ 16/12/2026 đến hết 15/01/2027 |
| Chỉ xét gói mới 16/11–15/12, quét ngày 15/11 | Gói mới chưa cấp quyền vào; phải có quyền khác hợp lệ |
| Quét ngày 16/11 hoặc 15/12 | Gói mới có hiệu lực; quét 16/12 thì gói này hết hạn |
| Có Gym còn hạn, mua Yoga lần đầu | Yoga bắt đầu ngày mua, không nối sau Gym |
| Tất cả Gym đã hết, mua lại 08/11 | Bắt đầu 08/11, không hồi tố nối vào thời gian đã qua |
| Gói 1 tháng bắt đầu 01/02/2028 | Dùng đến hết 29/02/2028; gói sau bắt đầu 01/03 |
| Hai giao dịch mua Gym đồng thời | Tính lại đuôi chuỗi và tạo gói nguyên tử theo hội viên/loại; không cùng nhận một ngày bắt đầu |

Quy ước tháng cho Gym/Yoga: lấy mốc NgayBatDau cộng số tháng (kẹp về cuối tháng nếu thiếu ngày tương ứng), rồi trừ một ngày. Cách mua thêm/kích hoạt PT chưa được thay bằng quy tắc nối tiếp này.

### Bổ sung rà soát 08/11/2026 — hoàn thiện hai việc được yêu cầu

- Đã tách chi tiết khóa/View/thủ tục khỏi tài liệu nghiệp vụ A–D; giữ yêu cầu nguyên tử, chống lặp, chống giữ quá buổi và tính nhất quán. Ghi chú kỹ thuật cũ được lưu ở GĐ2, không xóa lịch sử rà soát.
- Đã đóng lại GD1-15: đối chiếu 52 nhãn, bốn đơn vị nghiệp vụ bổ sung, cơ cấu 2.900/2.700 lịch PT và không cộng trùng B/C. Hướng theo đợt không thay đổi giờ trừ PT.
- Còn GD1-16 (tải cổng và số liệu giả định), sau đó GD1-17 (ghép và rà soát tổng). Chưa triển khai SQL Server.

### Bổ sung rà soát 08/11/2026 — hoàn thiện GD1-16

- Giữ nguyên yêu cầu tải của đề; sửa cách gán số cộng dồn thành đồng thời và suy diễn đỉnh từ trung bình. Công bố giới hạn bộ ước lượng GĐ1.
- Đồng bộ G11–G12 và các hàng mua/dùng vé, log, hóa đơn để không đếm trùng; kiểm tra 52 nhãn và đủ 17 mã lỗi.
- Còn GD1-17: ghép ERD/lược đồ, kiểm tra tham chiếu và ràng buộc liên module, kết luận dạng chuẩn toàn hệ thống. Chưa thực hiện triển khai/benchmark GĐ2.

### Bổ sung rà soát 08/11/2026 — hoàn thiện GD1-17

- Ghép ERD/lược đồ tổng, sửa hợp đồng chủ gói qua hóa đơn, bổ sung ràng buộc liên module và kết luận dạng chuẩn. Người dùng chốt QD10 giờ tròn.
- Đối chiếu cấu trúc, khóa, đơn vị tần suất, biên thời gian và liên kết tài liệu; phân tích/xuất và xem ERD tổng.
- Đã kiểm tra đủ 17 mục tài liệu GĐ1; giữ toàn bộ lịch sử xử lý. Không triển khai SQL/GĐ2. Thông tin thành viên và các giả định cần xác nhận vẫn được liệt kê riêng trước khi nộp.

### Rà cuối 08/11/2026 — đối chiếu lại đề gốc

- Báo cáo [ra-soat-cuoi.md](ra-soat-cuoi.md) ghi LC-01…LC-08, phân biệt lỗi/thiếu tài liệu với giả định và việc GĐ2.
- Mở lại GD1-10 (vòng đời nhân viên/quy trình A), GD1-11 (sơ đồ phụ B cũ), GD1-13 (miền/bảng ảnh hưởng và bước nhật ký D), GD1-15 (mô tả đọc/ghi hoa hồng và đồng bộ A), GD1-17 (tính đầy đủ/tính nhất quán toàn hồ sơ). Các quyết định đã chốt không tự bị hủy.
- Kiểm tra lại cấu trúc 20 quan hệ/33 FK/52 nhãn đạt; chưa sửa nghiệp vụ, chưa triển khai GĐ2. Kết luận đóng toàn bộ trước đó được thay bằng kết quả rà cuối này, không xóa nhật ký cũ.

### Bàn giao nhóm ngày 08/11/2026 — tạm dừng hôm nay

Theo yêu cầu nhóm trưởng, đã đưa LC-01…LC-08 vào [bảng phối hợp và góp ý](ra-soat-cuoi.md#bảng-nhóm-theo-dõi-và-góp-ý), có module liên quan, điều kiện hoàn tất, trạng thái, phản hồi và liên kết bản sửa/người kiểm tra. Các thành viên có thể đề xuất phương án khác kèm căn cứ; chưa tự chốt thêm nghiệp vụ. Dừng tại bàn giao, không triển khai GĐ2.
