# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Phan Ngọc Hòa`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | xe máy / mô tô |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Quỹ đạo và đặc trưng của xe liên tục, xác định được bằng mắt và khớp với track_buffer của tracker. |
| Xe bị che lâu hơn ngưỡng trên | Sẽ được tạo track mới và được đánh ID mới | Mất tính liên tục không-thời gian; gán nhãn không đoán mò để đảm bảo tính khách quan và online tracker cũng xóa track sau khi hết buffer. |
| Xe rời khung hình rồi quay lại | Sẽ được tạo track mới và được đánh ID mới | Bài toán Single-Camera MOT kết thúc vòng đời track khi ra khỏi khung hình. |
| Hai xe cắt nhau / chồng lên nhau | Vẫn vẽ phần hiện ra ngoài của xe bị che, còn nếu che hoàn toàn thì theo luật 1 và 2 | Chỉ vẽ phần nhìn thấy để tránh ô nhiễm đặc trưng và giảm IoU/MOTP; khi che 100% không còn pixel để vẽ nên tuân theo luật 1 & 2 để nhất quán hệ thống. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: xác định được nữa thân xe (nữa trước hoặc nữa sau)|
| Xe đang đỗ, không di chuyển | gán cho xe môt track như thường |
| Keyframe đặt dày ở đâu | lúc xe thay đổi tốc độ, chuyển hướng, dừng xe, gần ra hoặc đi ra khỏi frame |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: clip 1 / frame 81 / 001
- Tình huống: Xe màu xám đi sau chiếc xe buýt, có thể lờ mờ thấy được là một chiếc xe sedan khi nhìn xuyên qua chiếc xe buýt, nhưng lại quá mờ
- Quyết định: không đặt track ở lúc đó, mà đặt khi nhìn thấy rõ được xe, khi xe đi qua hẳn chiếc xe buýt
- Lý do: Vì ảnh quá mờ và bị che bởi xe buýt, sẽ khiến mô mình đoán sai khi đặt track như vậy

### Ca 2
- Clip / frame / ID: clip 1 / frame 1 - 190 / ID 1
- Tình huống: Xe SUV màu trắng đỗ cố định một chỗ sát lề đường bên trái từ frame 1 đến hết clip (frame 190), có người đi bộ đi lại gần xe. Dễ phân vân liệu xe đỗ bất động cả clip có cần gán và duy trì track suốt thời gian đó hay không.
- Quyết định: Vẫn duy trì một track ID 1 duy nhất từ frame 1 đến frame 190, giữ nguyên toạ độ và kích thước bbox xuyên suốt toàn bộ video.
- Lý do: Theo quy chuẩn schema bài lab, xe đang đỗ vẫn là `vehicle` hợp lệ trong khung hình; duy trì track cố định giúp tránh bị phạt lỗi bỏ sót (FN) và không làm rung lắc bbox.

### Ca 3
- Clip / frame / ID: clip 2 / frame 138 - 141 / ID 8
- Tình huống: Xe sedan màu đỏ bắt đầu xuất hiện từ mép dưới bên phải ở frame 138, nhưng ban đầu chỉ nhô lên một mảng nhỏ mui xe sát rìa dưới của ảnh, chưa thấy rõ kính lái hay bánh xe.
- Quyết định: Chưa tạo track ở frame 138 mà đợi đến frame 141 khi phần đầu xe và kính lái nhô lên đủ rõ để nhận diện chắc chắn là xe con 4 bánh; bbox vẽ chạm đúng mép dưới ảnh, tuyệt đối không vẽ đoán phần thân xe bên ngoài khung hình.
- Lý do: Tuân thủ quy tắc bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh và không đoán phần ngoài ảnh, tránh tạo bbox ảo làm suy giảm chất lượng IoU (MOTP) và gây nhiễu cho mô hình.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Ngưỡng bắt đầu track ở rìa ảnh**: Trước đây chỉ ghi frame đầu tiên xác định được xe, làm người gán sớm người gán muộn lệch 3-4 frame. **Viết lại**: Chỉ bắt đầu gán khi xe lộ diện từ 20% diện tích trở lên và thấy rõ bộ phận cấu trúc như đèn, kính lái hoặc bánh xe; không gán khi mới chỉ thấy một vệt màu nhỏ sát mép.
- **Che khuất bởi vật cản tĩnh như cột biển báo**: Luật cũ chỉ xét hai xe cắt nhau, thiếu trường hợp vật cản tĩnh ở giữa đường. **Viết lại**: Nếu xe bị cột hoặc biển báo che dưới 25 frame thì vẫn giữ nguyên một ID, bbox chỉ ôm phần nhìn thấy và bật occluded, không tách track mới và không vẽ trùm lên vật cản.
- **Thời điểm bấm outside khi xe rời khung**: Trước đây chưa chốt frame kết thúc, dễ làm sót bbox treo ngoài lề. **Viết lại**: Bấm outside bằng phím O ngay tại frame đầu tiên diện tích nhìn thấy của xe còn dưới 10% hoặc không còn điểm nhận diện trong khung hình.
