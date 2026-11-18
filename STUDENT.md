# Thông tin học viên

- Họ và tên: Phạm Đình Lực
- MSHV: 2A202603287
- Lớp (L2 / L3 / khác): L2b
- Tài khoản GitHub: lucpd16
- Repo bài nộp (URL): https://github.com/lucpd16/K5-L23-DAY08-2A202603287-PhamDinhLuc-AIPrelabel
- Commit chốt bài (hash): 64c1f05c3b53645905262516d687844152ef38c3
- Máy: Windows 11 (Local chạy script & CVAT Docker venv) + Google Colab Pro (GPU L4/G4/A100)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 310/519 (59.7%) / 83/519 (16.0%) / 121/519 (23.3%) (mẫu số: 519 box máy); added: 44/437 (10.1%) (mẫu số: 437 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 57.6% / 76.0% (mẫu số: 50 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 5.50 phút/ảnh; duyệt có pre-label 2.83 phút/ảnh
- Lỗi QC: 60.0% theo ảnh (3/5), 3.6% theo đối tượng (3/84)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh lỗi trong mẫu QC đạt 3/5 (60.0% > 2%) kèm lỗi hệ thống bỏ sót đối tượng (bo_sot 3/3) nên cần rà soát bổ sung theo guideline v1.1.