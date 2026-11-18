# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 84 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 3/84 = 3.6% | 84 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 3/5 = 60.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 310/519 = 59.7% / 83/519 = 16.0% / 121/519 = 23.3% | 519 box máy |
| Máy: added | 44/437 = 10.1% | 437 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 3 | img_003.jpg, img_006.jpg, img_035.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 3 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **88%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 60.0% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 3/3 lỗi

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
