# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Nguyễn Thành Đạt
Ngày: 16/9/2026

---

## 1. Quá trình gán nhãn

| Mục                                 | Giá trị    |
| ------------------------------------ | ------------ |
| Công cụ                            | CVAT         |
| Thời gian gán`clip_02` (warm-up) | `30` phút |
| Thời gian gán`clip_01`           | 120phút     |
| Số track đã vẽ trong`clip_01`  | 8            |
| Số keyframe trung bình mỗi track  | 4.6          |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Thời gian gán quá lâu do quá nhiều frame và xe di chuyển liên tục.** Phương án xử lý: áp dụng keyframe thưa ở những đoạn xe chạy đều, sau đó review và tinh chỉnh lại các điểm uốn (rẽ, phanh).
2. **Xe bị che khuất tạm thời rồi xuất hiện lại.** Phương án xử lý: áp dụng luật mặc định (dưới 2 giây / 25 frame) để giữ nguyên `track_id` cũ thay vì tạo track mới.
3. **Xe ở xa mờ hoặc sát rìa khung hình.** Phương án xử lý: bắt đầu track từ frame đầu tiên xác định chắc chắn là xe bốn bánh và kết thúc bằng phím `O` (outside) ngay khi xe rời khung.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1:  Bắt được các ID bị nhấp nháy, đổi số đột ngột hoặc ID bị trùng lặp trong cùng một frame.
- Lượt 2: Phát hiện các track quên bấm `outside` khiến bbox treo lơ lửng ở cuối clip, hoặc track bắt đầu trễ hơn thực tế.
- Lượt 3: Kiểm tra các đoạn giữa keyframe xa nhau để đảm bảo bbox không bị trôi (drift) lệch khỏi xe.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                               | Giá trị                                                            |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | `47e21d41f458284cd40a0db760fffa705dc51351c06c6c46da6a1fb3ee74b382` |
| Thời điểm khóa                                     | `2026-09-15T09:57:45.934632+00:00`                                 |
| Số row / frame / track trước khi mở reference      | `583`/ 190/ 8                                                      |

|               |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| Bản pre-gold | 0.847 | 0.832 | 0.865 | 0.893 | 0.974 | 0.948 | 0.881 | 20 | 10 |    0 |
| Sau rework    | 0.895 | 0.902 | 0.890 | 0.920 | 0.985 | 0.970 | 0.910 |  5 |  3 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame        | ID | Đã sửa thế nào                                                                                          |
| ---------- | ------------ | -- | ------------------------------------------------------------------------------------------------------------ |
| Bbox thừa | 97-100       | 6  | Bấm phím`O` (outside) để cắt bỏ bbox xuất hiện trước khi xe vào khung                           |
| Bbox treo  | 149-151      | 4  | Bấm phím`O` (outside) đúng frame xe rời khung để tránh bbox tồn tại ở nơi không có vật thể |
| Bbox trôi | 82-85        | 5  | Thêm keyframe trung gian quanh các frame IoU thấp để kéo bbox khít lại với vật thể                |
| Bbox trôi | 106, 111-113 | 6  | Thêm keyframe và tinh chỉnh góc khung hình cho khớp với track gold                                    |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                               | Giá trị                                           |
| ---------------------------------- | --------------------------------------------------- |
| Python / ultralytics / torch / lap | `3.11 / 8.4.145 / 2.x / 0.5.13`                   |
| weights / hai tracker              | `yolo26n.pt + bytetrack.yaml / botsort-reid.yaml` |
| conf / IoU / imgsz / classes       | `0.25 / 0.70 / 960 / [2, 5, 7]`                   |
| device                             | `cuda / cpu`                                      |

| So sánh                  |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| bạn vs gold              | 0.847 | 0.832 | 0.865 | 0.893 | 0.974 | 0.948 | 0.881 | 20 | 10 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 |    2 |
| BoT-SORT + ReID vs gold   | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 |    2 |
| ReID vs bạn              | 0.823 | 0.771 | 0.878 | 0.923 | 0.906 | 0.808 | 0.917 | 82 | 27 |    3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`...`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`...`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`...`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`...`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`...`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

**Sửa đổi trong `GUIDELINE_MINI.md`**:

  - Bổ sung quy định rõ ràng hơn về cách xử lý các trường hợp xe di chuyển sát mép khung hình hoặc bị khuất một phần lớn nhưng vẫn di chuyển liên tục.

  - Thêm hướng dẫn cụ thể về ngưỡng thời gian và cách đánh số ID thống nhất cho các trường hợp xe đi ra rồi quay lại khung hình ở các góc khuất phức tạp.

**Thay đổi trong quy trình làm việc**:

  - Tăng tần suất tự kiểm tra sớm (chạy script kiểm tra định dạng và lướt nhanh 3 lượt tua) ngay sau khi hoàn thành từng phần thay vì dồn lại cuối sprint.

- Tận dụng hiệu quả hơn các phím tắt và chế độ keyframe thưa trước, sau đó mới tinh chỉnh chi tiết để tối ưu thời gian thao tác trên CVAT.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
