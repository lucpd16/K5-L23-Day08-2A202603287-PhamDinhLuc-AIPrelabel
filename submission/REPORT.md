# Báo cáo Day 08 — 2A202603287 - Phạm Đình Lực

> Toàn bộ số liệu trong báo cáo được tổng hợp trực tiếp từ các tệp kết quả tại `submission/qc/` (`threshold_sweep.md`, `review_stats.md`, `qc_report_auto.md`), `submission/model_log.csv` và `submission/time_log.csv`.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| `person` / `bicycle` / `motorbike` / `car` / `bus` / `truck` | `YOLO26m` (`yolo26m.pt`) | `0.3` (mặc định) | Chạy quét ngưỡng trên 4 ảnh gold (`threshold_sweep.md`) thì không có mức nào `<= 0.5` giúp các lớp này chạm mục tiêu recall 90% (cao nhất chỉ tới 67%–86%). Riêng `bus` và `truck` có 0 mẫu trong gold (`< 5` vật) nên tôi giữ nguyên mặc định `0.3` (`r01`, `r02`). |
| `three_wheeler` | `YOLOE-26s` (`yoloe-26s-seg.pt`), prompt: `three_wheeler` | `0.15` | Ở mức `0.15`, YOLOE-26s bắt đủ 100% xe ba bánh trên tập gold (5/5 xe, vượt mốc `90%`) với precision `56%`. Tôi dùng thêm con này để bù cho YOLO26m vốn không có lớp xe ba bánh (`r03`). |

- **Lệnh `merge_prelabels.py` đã dùng:**
  ```powershell
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM đã chọn:** `SAM 2.1 base` (`sam2.1_b.pt`, lượt `r04`) chạy trên GPU L4 của Google Colab Pro, có chạy thử thêm `MobileSAM` (`mobile_sam.pt`) để so sánh. Tôi chọn `sam2.1_b.pt` vì chạy trên L4 rất nhanh (~0.45 giây/ảnh), không làm nặng máy tính ở nhà và cắt viền mấy phần hở như mái che hay bánh xe tuk-tuk mượt hơn bản nhẹ `mobile_sam.pt`.
- **Lỗi không sửa được bằng đổi ngưỡng:** Lỗi **nhận nhầm bản chất vật thể** — ví dụ `YOLO26m` không có nhãn `three_wheeler` nên cứ gặp tuk-tuk là gán thành `car` hoặc `truck`; còn `YOLOE-26s` thì hay nhận nhầm xe buýt hoặc tấm bạt che sạp hàng thành `three_wheeler`. Những lỗi này có tăng hay giảm `--conf` cũng không hết, buộc phải xếp `yoloe_all` ưu tiên trước `y26_all` lúc gộp (`--iou 0.6`) rồi tự tay vào CVAT xóa và sửa lại.
- **Box điểm cao mà sai / điểm thấp mà đúng:**
  - *Điểm cao mà sai:* Ở `img_002.jpg` và `img_013.jpg`, `YOLOE-26s` bắt nhầm xe bán tải và tấm bạt sạp hàng thành `three_wheeler` với điểm khá cao (`> 0.35`), hoặc vẽ 1 khung to đùng bao cả 2 chiếc tuk-tuk đứng cạnh nhau.
  - *Điểm thấp mà đúng:* Nhiều chiếc `three_wheeler` ở góc nghiêng hoặc bị che một phần chỉ có `conf` tầm `0.16–0.22` (nếu để mặc định `0.3` là rớt hết).
- **Giới hạn chọn ngưỡng trên tập gold:** Ngưỡng `three_wheeler=0.15` được chọn và đo trên chính 4 ảnh `gold_manual.json` (50 vật), nên con số trên gold hơi **lạc quan** — lúc đưa sang 24 ảnh batch thì ngưỡng `0.15` hơi thấp nên sinh ra khá nhiều box `three_wheeler` thừa phải ngồi xóa.

## 2. AI error profile (từ `qc/review_stats.md`)

*(Hệ thống đã tự lọc bỏ 25 box liên quan đến 12 bẫy nhân tạo để đánh giá đúng chất lượng thật của máy).*

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| `accepted` (giữ nguyên, IoU ≥ 0.9) | 310 | 310/519 = 59.7% | 519 box máy |
| `edited` (chỉnh kích thước / đổi lớp) | 83 | 83/519 = 16.0% | 519 box máy |
| `deleted` (xóa box thừa / box giả) | 121 | 121/519 = 23.3% | 519 box máy |
| `added` (vẽ thêm vật sót) | 44 | 44/437 = 10.1% | 437 đối tượng sau duyệt |

*(Trong đống `deleted + added` có 5 ca là đổi lớp đúng chỗ, nên số vật máy bỏ sót thật là `39` vật).*

- **Lớp nào máy tệ nhất? Vì sao:**
  - Về số lượng box rác phải xóa (`deleted`), **`person`** nhiều nhất (**74 box**) vì người đi bộ hoặc ngồi bán hàng trong chợ đông hay bị che nửa người, hoặc đứng tụm lại thành cụm nhỏ `< 20 px` nên tôi phải xóa box lẻ để khoanh chung thành `ignore_region`.
  - Về độ "ảo" của mô hình, **`three_wheeler`** tệ nhất khi phải xóa tới **15 box** trong khi chỉ giữ lại được 20 box (`accepted`). Nguyên nhân là ngưỡng `0.15` làm YOLOE vẽ đè nhiều box lên cùng 1 xe hoặc nhận nhầm cả bạt che sạp hàng (`img_013.jpg`). Ngoài ra ở mấy tấm ảnh bị nhòe chuyển động, `YOLO26m` hay gộp 2 chiếc xe máy chạy sát nhau thành 1 box `car` hoặc nhận nhầm xe máy thành xe đạp.
- **So với gold (4 ảnh, 50 đối tượng):**
  - Precision của máy: **57.6%** (`38 TP / (38 TP + 28 FP)`, KTC 95%: `46%–69%`).
  - Recall của máy: **76.0%** (`38 TP / 50 đối tượng gold`, KTC 95%: `63%–86%`).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Tỉ lệ bắt bẫy:** **10/12 = 83.3%** (KTC 95%: `55%–95%`, vừa đủ qua mốc `>= 80%`).
  - Chi tiết từng loại: `missing` **3/3**, `extra` **3/3**, `loose` **2/3**, `wrong_class` **2/3**.
  - **2 bẫy bị lọt:** `img_027.jpg` (bẫy `wrong_class`) và `img_017.jpg` (bẫy `loose`).
  - **Vì sao lọt:** Lúc làm tới `img_027.jpg`, ảnh khá rối nên thấy khung vẽ ôm gọn xe rồi tôi lại chủ quan không kiểm tra lại tên nhãn bên bảng phải. Còn ở `img_017.jpg`, box bẫy chỉ kéo rộng hơn xe một chút vào phần nền tối phía sau, duyệt một mạch nhiều ảnh nên mắt bị mỏi và lướt qua.
- **Nhìn từ `accepted` (59.7%) và bắt bẫy (83.3%):** Tỉ lệ giữ nguyên 59.7% kèm việc xóa tới 23.3% box thừa và bắt được 10/12 bẫy cho thấy tôi không bấm đồng ý bừa. Tuy nhiên, việc bắt sạch 6/6 bẫy thiếu/thừa (`missing`, `extra`) nhưng lại để sổng 2 bẫy có sẵn khung (`loose`, `wrong_class`) cho thấy khi thấy AI đã vẽ sẵn một cái khung trông có vẻ hợp lý, mình rất dễ bị **chủ quan theo gợi ý của máy** mà quên soi kỹ chi tiết.

## 4. Máy có đáng dùng không?

- **Thời gian thực hiện (từ `time_log.csv`):** Gán tay 4 ảnh gold mất **5.50 phút/ảnh** (`22.0` phút / 4 ảnh), còn duyệt 24 ảnh batch có pre-label mất **2.83 phút/ảnh** (`68.0` phút / 24 ảnh). Tuy nhiên tốc độ nhanh hơn này có một phần là do tôi làm 4 ảnh gold trước nên đã quen phím tắt CVAT và nhớ quy tắc gán hơn.
- **Kết luận:** **Có đáng dùng**, nhất là với các lớp phổ biến như `person`, `motorbike`, `car`, `bus`, `truck`. Giữ lại được gần `60%` box có sẵn giúp đỡ mỏi tay hơn hẳn khi gặp những ảnh đường phố đông nghẹt xe. Riêng lớp `three_wheeler` thì model hiện tại bắt nhiễu quá nhiều, công ngồi xóa box thừa gần bằng vẽ tay mới.
- **Ba quy tắc tự đặt ra để đỡ phụ thuộc vào máy:**
  1. **Bấm nút con mắt ẩn hết nhãn AI trong vài giây đầu** mỗi khi mở ảnh mới để tự nhìn bằng mắt xem có bao nhiêu người và xe trước.
  2. **Kiểm tra tách bạch 2 bước với mọi box có sẵn:** đọc tên lớp trên bảng *Objects* trước rồi mới nhìn độ khít 4 cạnh viền (tránh lỗi như `img_027.jpg`).
  3. **Dùng thanh `Filters` lọc riêng `three_wheeler` và các box `conf < 0.35`** sau mỗi 5 ảnh để dọn sạch mấy box bao gộp nhiều xe.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | `sam2.1_b.pt` | 4 (`2 pos, 2 neg`) | Không (đã dùng 2 điểm âm tách tài xế ra) | Mask phủ trọn mái vòm, thân và bánh xe tuk-tuk rất sát; phần áo và đầu tài xế được tách riêng rõ ràng. | `tuktuk_img013_sam2.jpg` |
| `img_013.jpg` | `mobile_sam.pt` | 4 (`2 pos, 2 neg`) | Còn dính nhẹ tay và vai người lái | Viền mask hơi răng cưa ở chỗ nan hoa bánh xe và dễ ăn lẹm sang người ngồi cạnh. | `tuktuk_img013_sam1.jpg` |
| `img_004.jpg` | `sam2.1_b.pt` | 3 (`2 pos, 1 neg`) | Không | Cắt gọn phần khung xe ba bánh ở gần, không bị dính nền tiệm phía sau. | `tuktuk_img004_sam2.jpg` |
| `img_002.jpg` | `sam2.1_b.pt` | 2 (`1 pos, 1 neg`) | Không | Ôm sát phần đầu và thân xe tuk-tuk xanh, tách được người lái ra ngoài. | `tuktuk_img002_sam2.jpg` |

- **Đối chiếu guideline mục 4.2:** Quy tắc *bỏ qua lỗ thủng nhỏ hơn 5% diện tích* và *tách riêng người lái khỏi xe* (mục 4.2.2 & 4.2.4) giúp tôi đặt điểm `--pos`/`--neg` gọn hơn, không phải chấm chi chít vào mấy khe nan sắt nhỏ. Có điều guideline v1.0 chưa nói rõ cách xử lý các thanh khung mái che quá mảnh dễ bị SAM cắt đứt đoạn.
- **Interactor SAM trong CVAT vs chạy offline:** Tôi không bật AI Interactor trực tiếp trên CVAT để tránh nặng máy khi đang chạy Docker và giữ đúng quy tắc không dùng AI ở Part 1; chạy `sam2_points.py` trên Colab GPU L4 giúp lưu lại đúng tọa độ click và dễ so sánh trực tiếp giữa `sam2.1_b.pt` với `mobile_sam.pt`.

## 6. Calibration (Part 6)

- **Số quyết định đã chốt:** **4 quyết định** (`CAL-01` đến `CAL-04` trong `submission/qc/calibration_decisions.csv`). Quan trọng nhất là **`CAL-02`** (xử lý lỗi `YOLOE-26s` vẽ box bao trùm 2 chiếc tuk-tuk cạnh nhau và nhận nhầm xe buýt/tấm bạt sạp hàng thành `three_wheeler`), vì đây là nguyên nhân chính khiến tôi phải xóa rất nhiều box thừa.
- **Quy tắc mới đã thêm vào `guideline_v1.1_calibrated.md`:**
  1. Phân biệt rõ `bicycle` và `motorbike` với xe đạp điện/xe máy điện dựa trên bàn đạp cơ học và yên xe (`CAL-01`).
  2. Không dùng box gộp chung nhiều xe đứng cạnh nhau; kiểm tra kỹ bánh xe và khung gầm trước khi chốt `three_wheeler` để tránh nhận nhầm bạt che hay xe buýt (`CAL-02`).
  3. Tách riêng từng box `motorbike` trong ảnh bị nhòe chuyển động thay vì gộp chung thành `car` (`CAL-03`).
  4. Chốt rõ mức áp dụng `ignore_region` (`>= 5` xe dính chùm ở xa `< 20 px`) và giới hạn hàng hóa nhô ra tối đa `20%` chiều dài xe (`CAL-04`).
- **Nguyên nhân gốc:** Do cả hai phía: **guideline v1.0 còn bỏ ngỏ** mấy ca như xe điện hay cụm xe đông gần camera, cộng thêm **model bị nhiễu** khi hạ ngưỡng `three_wheeler=0.15` trên đường phố đông đúc.

## 7. QC và quyết định (Part 6)

- **Mẫu kiểm định:** **5 ảnh** rút từ lô batch (`0` ảnh `can_review` + `5` ảnh ngẫu nhiên với `--seed 42`, tổng `84` đối tượng). Người kiểm: tự kiểm chéo cá nhân (**hạn chế:** tự kiểm lại bài mình vừa duyệt nên rất dễ bị lặp lại đúng những chỗ mình đã nhìn sót trước đó).
- **Tỉ lệ lỗi trong `qc_report_auto.md`:**
  - Lỗi theo **đối tượng**: **3/84 = 3.6%**.
  - Lỗi theo **ảnh**: **3/5 = 60.0%** (cao hơn nhiều so với mức trần `<= 2%` để Release).
- **Top loại lỗi & lỗi hệ thống:** `bo_sot` (**3/3 lỗi**, chiếm `100%` số lỗi tìm thấy, nằm ở `img_003.jpg`, `img_006.jpg` và `img_035.jpg`). Đây là **lỗi hệ thống** lặp lại ở các vùng rìa đám đông hoặc vật bị che khuất một phần lúc duyệt nhanh, nên cần siết lại cách rà ảnh theo `v1.1` chứ không chỉ sửa lẻ vài box.
- **Giới hạn thống kê của mẫu nhỏ:** Với `3/5` ảnh lỗi, **cận trên khoảng tin cậy 95% (Wilson)** lên tới **88%** (theo `qc_report_auto.md`). Mẫu nhỏ 5 ảnh chỉ đủ để phát hiện các lỗi lặp lại rõ rệt chứ không thể kết luận chắc chắn toàn bộ lô ảnh đạt chuẩn dưới `2%`.
- **Script kiểm được và không kiểm được:** Các script (`validate_labels.py`, `qc_report.py`) bắt lỗi định dạng file, tên lớp, chiều cao box `< 20 px` và tính toán số liệu rất chuẩn, nhưng **không thể biết** box vẽ có ôm sát viền xe chưa hay người duyệt có nhìn sót đối tượng ở xa không.

**Quyết định: REWORK**, ngày **2026-11-18**.
- **Lý do (theo cây quyết định `Hold → Rework → Release`):** Không chọn *Hold* vì các thắc mắc ở guideline đã được chốt sang `v1.1` và bộ ảnh gốc không bị lỗi. Chọn **Rework** vì tỉ lệ ảnh lỗi trong mẫu QC lên tới `3/5 = 60.0%` (`> 2%`) và bị lặp lại **lỗi hệ thống `bo_sot`** (`3/3` lỗi tại `img_003.jpg`, `img_006.jpg`, `img_035.jpg`).
- **Việc cần làm tiếp theo:** Mở lại Task `batch-review` trên CVAT, rà kỹ lại các ảnh đông người/xe theo quy tắc `guideline_v1.1_calibrated.md` để vẽ bổ sung các vật bị sót (`bo_sot`), xuất lại `submission/labels/qc_fixed.json` rồi chạy lại `build_release.py`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Tôi có dùng AI để hỏi cách chạy từng pha của bài lab, nhờ gỡ lỗi database PostgreSQL của CVAT `v2.81.0` và hỗ trợ sắp xếp lại câu chữ cho báo cáo mạch lạc hơn; còn toàn bộ việc gán tay 4 ảnh gold, sửa 24 ảnh batch trên CVAT, chạy inference trên Colab L4 và soi lỗi QC đều do tôi tự làm và kiểm tra thực tế.
- **Gold có bị nhiễm không:** **Không.** Tôi gán tay xong xuôi 4 ảnh `gold_manual.json` trước khi chạy bất kỳ lệnh AI nào và không dùng AI Interactor trong CVAT.
- **Máy / GPU / CVAT dùng; vướng mắc cài đặt:**
  - *Cấu hình:* Máy cá nhân chạy Windows 11 (PowerShell) để chạy script local và Docker CVAT `v2.81.0` (`http://localhost:8080`), kết hợp Google Colab Pro dùng **GPU L4** để chạy `YOLO26m`, `YOLOE-26s` và `SAM 2.1`.
  - *Vướng mắc cài đặt:* Lúc đầu tạo Project trên CVAT `v2.81.0` bị dính lỗi `Server Error (500)` (`column "iou_threshold" is of type jsonb`). Kiểm tra log container thì thấy lỗi lệch kiểu dữ liệu trong PostgreSQL nên tôi đã chạy lệnh `ALTER TABLE` trực tiếp trong `cvat_db` để ép cột `iou_threshold` sang `double precision` thì tạo project bình thường.
- **Dùng cache `data/cache/` thay vì tự chạy?** **Không.** Tôi tự chạy toàn bộ model pre-label và tách mask SAM trên GPU L4 của Colab Pro.
- **Lỗi / bug script đã gặp:** Lúc chạy `sam2_points.py` trên `img_017.jpg` bị báo `FileNotFoundError` do lệch tọa độ mẫu nên tôi đã đổi sang `img_002.jpg` để xuất đủ ảnh mask; ngoài ra lúc đầu điền thiếu 3 dòng trong `qc_review.csv` nên `qc_report.py` báo cảnh báo, sau đó tôi đã soi nốt cả 5 ảnh mẫu và chạy lại báo cáo đầy đủ.
- **Một điều sẽ làm khác lần sau:** Với những ảnh đông xe máy và người đi bộ đan xen nhau, tôi sẽ phóng to từng góc ảnh để rà thêm một lượt cuối cùng trước khi export, tránh để lặp lại lỗi `bo_sot` như đợt kiểm QC này.