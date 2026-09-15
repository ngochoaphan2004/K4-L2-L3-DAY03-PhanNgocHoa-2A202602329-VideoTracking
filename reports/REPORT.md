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
| Số row / frame / track trước khi mở reference | `549 rows / 190 frames / 8 tracks` |

| Bản | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Bản pre-gold | 0.8288 | 0.8169 | 0.8418 | 0.8792 | 0.9697 | 0.9407 | 0.8686 | 5 | 29 | 0 |
| Sau rework | 0.8288 | 0.8169 | 0.8418 | 0.8792 | 0.9697 | 0.9407 | 0.8686 | 5 | 29 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox trôi (IoU thấp) | 111–112 | 6 | Xe minivan đi sau cột biển báo trung tâm IoU còn 0.52; thêm keyframe ở frame 111 để nắn lại bbox ôm sát phần nhìn thấy, tránh trôi nội suy. |
| Bbox trôi (IoU thấp) | 84–85 | 5 | Xe con xám mới nhô ra sau xe buýt IoU chỉ 0.52; chỉnh lại keyframe đầu cho khít góc và thân xe. |
| Bbox trôi (IoU thấp) | 168 | 8 | Xe sedan đỏ ở sát rìa dưới IoU còn 0.56; nắn lại mép dưới bbox chạm chuẩn rìa ảnh trước khi ra khỏi khung. |
| Bỏ sót frame (FN) | 138–140 | 8 | Gold gán sớm 3 frame khi xe đỏ mới nhô một vệt mui; đối chiếu guideline nhóm đã thống nhất chỉ gán khi lộ diện trên 20% để nhất quán. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25` / `0.7` / `960` / `[2, 5, 7]` (car, bus, truck) |
| device | `0` (GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| bạn vs gold | 0.8288 | 0.8169 | 0.8418 | 0.8792 | 0.9697 | 0.9407 | 0.8686 | 5 | 29 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.8215 | 0.7657 | 0.8815 | 0.9146 | 0.9115 | 0.8106 | 0.9082 | 96 | 7 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- MOTA của tôi (0.9407) thấp hơn IDF1 (0.9697) một chút; cả hai đều rất cao do không bị lỗi ID switch nào.
- MOTA cao mà IDF1 thấp nghĩa là: Model phát hiện đúng xe ở từng frame, nhưng bị đổi ID hoặc chia vụn track liên tục (lỗi association).
- MOTA không phạt nặng lỗi ID vì nó chỉ đếm mỗi lần đổi ID đúng 1 lỗi tại frame xảy ra switch. Ngược lại, IDF1 đo tính nhất quán suốt quãng đời, nên track bị cắt đôi sẽ bị phạt trên toàn bộ nửa thời gian còn lại.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Số liệu:** BoT-SORT + ReID tốt hơn ByteTrack ở IDF1 (0.9001 vs 0.8746) và AssA (0.8204 vs 0.7761); cả hai đều có 2 lần ID switch.
- **Dẫn chứng:** Ở xe buýt lớn (GT track 4, frame 60–149), ByteTrack bị nhảy ID ở frame 59–60 làm cắt đôi track; còn BoT-SORT + ReID giữ nguyên đúng 1 ID suốt 95 frame.
- **Lưu ý:** Không thể khẳng định riêng ReID tạo ra khác biệt này (chưa cô lập causal effect), vì ByteTrack và BoT-SORT là hai tracker khác nhau cả về Kalman filter và quy trình liên kết.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- BoT-SORT + ReID có DetA tăng (0.6487 lên 0.7110), FN giảm mạnh từ 54 xuống 26, còn FP tăng nhẹ (88 lên 91).
- Lỗi còn lại chủ yếu là do **detector (YOLO)**: Số lượng FP rất lớn (~90 box) do nhận nhầm vật thể tĩnh bên đường, và DetA thấp hơn nhiều so với AssA.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Frame 16–116, pred_track 7:** Model nhận nhầm bốt ki-ốt bán hàng ven đường (x≈490, y≈211) thành xe và theo dấu suốt 43 frame (FP). Tôi đúng vì nhận biết bằng mắt đây là vật thể tĩnh ngoài schema nên không gán.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Frame 138–140, xe sedan đỏ (pred_track 8):** Model phát hiện xe đỏ từ frame 138 (trùng với bản gold), trong khi tôi đợi đến frame 141 mới gán. Evidence này cho thấy xe đỏ thực sự đã vào hình từ frame 138 và có thể gán sớm hơn khi xe có màu tương phản rõ rệt.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong `GUIDELINE_MINI.md`:**
  - **Định lượng rõ ngưỡng rìa ảnh:** Quy định rõ xe lộ diện từ 20% diện tích trở lên mới bắt đầu gán, và bấm `outside` ngay khi diện tích còn dưới 10% để tránh lệch frame giữa các thành viên.
  - **Bổ sung vật cản tĩnh và xe đỗ:** Hướng dẫn chi tiết cách vẽ bbox khi xe bị cột biển báo che cắt ngang, và khóa toạ độ cố định đối với xe đỗ dài hạn để không bị rung lắc IoU.
- **Đổi trong quy trình làm việc:**
  - **Gán dứt điểm từng xe:** Theo dõi trọn vẹn một xe từ lúc xuất hiện đến khi rời khung rồi mới sang xe khác; không nhảy qua lại giữa các xe để tránh ID switch.
  - **Duy trì tự kiểm 3 lượt:** Lượt 1 tua nhanh chỉ nhìn số ID; Lượt 2 kiểm tra kỹ frame đầu và frame cuối (chắc chắn đã bấm phím `O`); Lượt 3 kiểm tra giữa hai keyframe xa nhau để bắt lỗi trôi bbox do nội suy.
  - **Chạy script kiểm tra sớm:** Chạy ngay `check_mot_labels.py` sau khi export để lọc lỗi box treo trước khi nộp hoặc kiểm chéo.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)


