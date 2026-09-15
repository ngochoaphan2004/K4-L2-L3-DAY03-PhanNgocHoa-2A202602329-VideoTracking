# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Phan Ngọc Hòa`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `10` phút |
| Thời gian gán `clip_01` | `20` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `2.89` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất và xuất hiện mờ sau xe lớn (Xe xám đi sau xe buýt):**
   - *Khó khăn:* Xe xám bị thân xe buýt che khuất gần hết và nhìn qua kính xe buýt rất mờ, dễ đoán sai biên dạng bbox và vị trí bắt đầu track.
   - *Cách xử lý:* Không gán khi xe còn quá mờ; đợi đến frame xe vượt qua khỏi thân xe buýt và lộ rõ hình dạng xe con 4 bánh mới bắt đầu tạo track và vẽ bbox ôm phần nhìn thấy được.
2. **Xe SUV trắng đỗ cố định suốt toàn bộ video (ID 1 đỗ từ frame 1 đến frame 190):**
   - *Khó khăn:* Xe đỗ bất động một chỗ từ đầu đến cuối clip và có người đi bộ đi lại gần xe, dễ phân vân liệu xe không chuyển động có cần duy trì track suốt cả clip hay không.
   - *Cách xử lý:* Vẫn duy trì cùng track ID 1 xuyên suốt từ frame 1 đến frame 190 theo đúng schema bài lab; giữ cố định toạ độ và kích thước bbox để tránh rung lắc IoU hoặc trôi bbox do nội suy, không bấm outside khi xe chưa rời khung hình.
3. **Xe di chuyển nhanh và bị cắt bởi rìa ảnh (Xe sedan bạc ID 2 và xe sedan đỏ ID 8):**
   - *Khó khăn:* Xe đi với tốc độ cao sát góc camera, chỉ xuất hiện một phần thân xe ở mép viền, dễ bị trôi bbox khi nhảy frame xa và dễ quên bấm outside làm treo bbox.
   - *Cách xử lý:* Đặt keyframe dày hơn bình thường (cách 2–3 frame); bbox vẽ chạm đúng mép viền ảnh không đoán phần ngoài; bấm phím O (outside) ngay tại frame đầu tiên xe biến mất khỏi khung hình.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `14ef7f34c803648bb9f8f69db5a5e2ed7e4c4ee84b9c14bf0771a3cc36e92fc2` |
| Thời điểm khóa | `2026-09-15T04:45:07.716222+00:00` |
| Số row / frame / track trước khi mở reference | `549/190/1,  2, 3, 4, 5, 6, 7, 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| | | | |
| | | | |
| | | | |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `...` |
| weights / hai tracker | `...` |
| conf / IoU / imgsz / classes | `...` |
| device | `...` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | | | | | | | | | | |
| ByteTrack control vs gold | | | | | | | | | | |
| BoT-SORT + ReID vs gold | | | | | | | | | | |
| ReID vs bạn | | | | | | | | | | |

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

`...`

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)


