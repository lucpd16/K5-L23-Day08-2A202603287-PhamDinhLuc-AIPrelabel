# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 519; đối tượng sau duyệt: 437; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 310 | 310/519 = 59.7% | 519 box máy |
| edited (sửa) | 83 | 83/519 = 16.0% | 519 box máy |
| deleted (xoá) | 121 | 121/519 = 23.3% | 519 box máy |
| added (vẽ thêm) | 44 | 44/437 = 10.1% | 437 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 39.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 158 | 37 | 74 | 23 |
| bicycle | 4 | 2 | 2 | 1 |
| motorbike | 83 | 34 | 19 | 14 |
| car | 37 | 5 | 11 | 3 |
| bus | 3 | 0 | 0 | 1 |
| truck | 5 | 0 | 0 | 1 |
| three_wheeler | 20 | 5 | 15 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 50 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 38 | 28 | 12 | 57.6% (KTC 95%: 46–69%) | 76.0% (KTC 95%: 63–86%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - missing: 3/3
  - extra: 3/3
  - wrong_class: 2/3
  - lọt: img_027.jpg (wrong_class), img_017.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
