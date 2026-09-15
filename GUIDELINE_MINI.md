# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Cư Đức Quang — làm cá nhân (MSSV 2A202602188)
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung: giữ đúng một Rectangle Track `vehicle` trong CVAT local; không dùng Shape.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ ID cũ nếu che dưới 25 frame | xe vẫn ở trong cảnh; bật `Occluded`, không dùng `Outside` |
| Xe bị che lâu hơn 25 frame | mở track mới theo mặc định lab | bằng chứng nối cùng xe không còn đủ chắc |
| Xe rời khung hình rồi quay lại | track mới | `Outside` chấm dứt lần xuất hiện trước |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ ID riêng | theo chuyển động trước/sau crossing, không đổi số theo bbox gần nhất |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu ở frame đầu tiên nhận ra xe bốn bánh từ phần nhìn thấy; nếu chỉ là vài pixel không rõ loại thì chờ frame sau |
| Xe đang đỗ, không di chuyển | vẫn giữ một track xuyên suốt lúc xe còn trong cảnh |
| Keyframe đặt dày ở đâu | lúc vào/ra rìa ảnh, đổi scale, crossing hoặc occlusion; kiểm midpoint giữa keyframe xa nhau |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01`, CVAT frame 50–65 (MOT 51–66), ID 4.
- Tình huống: xe buýt chỉ lộ một dải hẹp khi đi vào từ rìa phải.
- Quyết định trong bản nhãn: bắt đầu một track ở frame 50, bbox chạm rìa phải; không chia ID khi thân xe hiện dần.
- Lý do: phần đầu xe đã nhận ra được; không đoán phần nằm ngoài ảnh.

### Ca 2
- Clip / frame / ID: `clip_01`, CVAT frame 78–115 (MOT 79–116), ID 5.
- Tình huống: sedan xám ở làn phía sau bị xe buýt che một phần rồi hiện rõ.
- Quyết định trong bản nhãn: giữ ID 5 và đánh dấu `Occluded` ở đoạn bị che.
- Lý do: xe vẫn trong cảnh và diễn tiến vị trí trước/sau che liên tục; `Outside` chỉ dành cho frame không còn bbox.

### Ca 3
- Clip / frame / ID: `clip_01`, CVAT frame 164–171 (MOT 165–172), ID 8.
- Tình huống: xe đỏ bị cắt sát rìa phải khi đi ra; một `Outside` sớm ở frame 164 từng làm trống một frame còn thấy xe.
- Quyết định trong bản nhãn sau QC: giữ bbox khi còn thấy xe, đặt `Outside` tại frame 171.
- Lý do: rời khung chỉ khi không còn phần xe nhìn thấy; bbox cuối chạm rìa ảnh, không kéo sang ngoài.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Cần quy định rõ frame đầu tiên một xe rất nhỏ ở rìa được coi là đủ nhận dạng; hiện bản nhãn và reference chênh vài frame ở ID 4/6/8.
- Với xe chỉ còn vài pixel ở rìa, kiểm frame kế và ghi quyết định visible-box trước khi bấm `Outside`.
