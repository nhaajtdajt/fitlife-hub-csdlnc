# Hồ sơ giai đoạn 1 — FitLife Hub

Phân tích nghiệp vụ, thiết kế mức quan niệm và mức logic theo [yêu cầu GĐ1](../../required.md). Bản này đã chốt lại toàn bộ theo [lược đồ logic 13 quan hệ](luoc-do-logic.png) ngày 13/11/2026.

| Đầu ra yêu cầu | Tài liệu | Trạng thái |
|---|---|---|
| Mô tả chi tiết quy trình nghiệp vụ | [A](a/quy-trinh.md), [B](b/quy-trinh.md), [C](c/quy-trinh.md), [D](d/quy-trinh.md) | Xong |
| Bảng tần suất: chức năng, mô tả, tần suất, mức độ | [A](a/bang-tan-suat.md), [B](b/bang-tan-suat.md), [C](c/bang-tan-suat.md), [D](d/bang-tan-suat.md) — 46 chức năng, dùng [giả định chung](../quyet-dinh.md#bảng-giả-định-số-liệu-chung) | Xong |
| ERD mức quan niệm và ràng buộc ngoài hình | [ERD và 18 ràng buộc](erd-tong.md) (hình [erd.png](erd.png), nguồn [.drawio](erd.drawio)) | Xong |
| Lược đồ CSDL quan hệ | [luoc-do-logic.png](luoc-do-logic.png) (nguồn [.drawio](luoc-do-logic.drawio)) | Xong |
| Khóa, phụ thuộc hàm, dạng chuẩn từng quan hệ; kết luận dạng chuẩn | [Thiết kế logic tổng](thiet-ke-tong.md), mục 2–3 | Xong |
| Mô tả quan hệ và thuộc tính | [Thiết kế logic tổng](thiet-ke-tong.md), mục 4 | Xong |
| Ràng buộc: bối cảnh, nội dung, bảng tầm ảnh hưởng | [Thiết kế logic tổng](thiet-ke-tong.md), mục 6 (R01–R27) | Xong |
| Quyết định và giả định của nhóm | [quyet-dinh.md](../quyet-dinh.md) (28 quyết định) | Xong |

**Kết luận dạng chuẩn:** 10 quan hệ đạt BCNF; HOI_VIEN, LICH_HEN_PT, HOA_DON đạt 2NF do cố ý lưu dư để tăng tốc truy vấn; toàn lược đồ đạt 2NF.

Tóm tắt theo module: [A](a/README.md), [B](b/README.md), [C](c/README.md), [D](d/README.md).

Trước khi nộp: bổ sung tên/MSSV còn thiếu.
