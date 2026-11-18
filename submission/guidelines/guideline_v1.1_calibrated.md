# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603287 - Phạm Đình Lực

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Xếp vào `motorbike` nếu có tay ga, yên liền dày hoặc không có bàn đạp cơ học; chỉ xếp `bicycle` khi có bàn đạp đạp chân rõ ràng. | `img_005.jpg` | `CAL-01` |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Các xe ở gần camera (chiều cao ≥ 20 px) vẫn gán từng box `motorbike` độc lập nếu phân biệt được yên/tay lái; chỉ dùng `ignore_region` cho cụm ≥ 5 xe dính chùm ở xa (< 20 px). | `img_021.jpg` | `CAL-04` |
| 3 | Hàng nhô ra trên xe đạp / ba gác: box xe có gồm hàng không? | Giữ quy tắc mục 4.1.5: chỉ tính vào box xe nếu phần hàng nhô ra ≤ 20% chiều dài xe; nếu nhô xa hơn thì box xe chỉ ôm phần khung phương tiện. | `img_021.jpg` | `CAL-04` |
| 4 | Các câu hỏi mở còn lại (người ngồi trong mui kín, kính chắn gió, box đêm, ngưỡng tin cậy tối thiểu) | Chưa chốt cứng ở v1.1 do bộ 24 ảnh batch chủ yếu là cảnh ban ngày và ít ca kính xe trong suốt; giữ nguyên quy tắc `can_review` ở v1.0 để đánh giá thêm ở lô dữ liệu sau. | `img_017.jpg` | `CAL-04` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Hai chiếc tuk-tuk đỗ cạnh nhau, hoặc sạp bán hàng có tấm bạt che / xe buýt lớn | **YOLOE-26s** (`conf=0.15`) vẽ 2 box thiếu cạnh cho 2 xe kèm 1 box lớn bao trùm cả 2 xe; đồng thời nhận nhầm tấm bạt che sạp hàng và xe buýt thành `three_wheeler`. | Xóa box bao gộp và các box dương tính giả từ bạt che; sửa nhãn xe buýt về `bus`; vẽ lại 2 box `three_wheeler` độc lập ôm sát từng xe. | Bổ sung vào **Mục 2 & Mục 4.1**: Không vẽ box gộp nhiều phương tiện; kiểm tra cấu trúc bánh xe và khung gầm trước khi gán `three_wheeler`. |
| `img_018.jpg` | Hai xe máy chạy song song trong khung hình bị nhòe chuyển động (motion blur) | **YOLO26m** (`conf=0.3`) gộp 2 xe máy thành 1 box `car` hoặc nhận nhầm xe máy thành `bicycle`. | Tách thành 2 box `motorbike` riêng biệt (kèm box `person` cho từng người lái) dựa vào tỷ lệ bánh xe và thân xe có động cơ. | Bổ sung vào **Mục 4.1.4 & 4.3**: Khi ảnh bị nhòe, dựa vào cụm động cơ và tỷ lệ khung xe để tách từng `motorbike`, không gộp nhóm thành `car`. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc phân loại xe điện, xử lý cụm xe máy dày đặc, giới hạn hàng nhô ra và bổ sung quy tắc chống gộp box / nhận nhầm vật thể tĩnh (bạt che, xe buýt) từ lỗi của YOLO26 & YOLOE. | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` |