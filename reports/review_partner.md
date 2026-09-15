# Kiểm tra chéo Day 3 — làm cá nhân, peer review N/A

| Trường | Giá trị |
| --- | --- |
| Author | Cư Đức Quang (2A202602188) |
| Reviewer | Không có — bài làm cá nhân; không giả lập reviewer |
| Pair ID | N/A |
| CVAT version | Local 2.75.0, task #12 / job #12 |
| Thời điểm review | 15/09/2026; tự QC trước khi export lại MOT |

## Danh sách finding

Các finding dưới đây là **self-QC/diagnostic**, không phải nhận xét của một peer.
CVAT dùng frame 0-based; MOT dùng frame 1-based.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa | Closure |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 56 | 57 | 4 | Track boundary | Xe buýt vẫn hiện ở rìa phải, nhưng `Outside` làm trống frame 56–61; Outside chỉ dùng khi xe đã vắng mặt | Người học bỏ `Outside` sớm trong CVAT, export lại MOT | fixed — validator không còn gap track 4 |
| 2 | 164 | 165 | 8 | Track boundary | Xe đỏ vẫn lộ ở rìa phải, nhưng `Outside` sớm làm trống frame 164 | Người học bỏ `Outside` sớm trong CVAT, export lại MOT | fixed — validator không còn gap track 8 |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS tự kiểm | `clip_01`: 8 Rectangle Track `vehicle`; validator 0 lỗi |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS theo phép chấm | bản nhãn vs gold: IDSW 0, IDF1 0.966 |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS theo timeline hiện có | ID 5 giữ liên tục qua đoạn bị xe buýt che, CVAT frame 78–115 |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | hai lỗi sớm đã sửa; so gold còn chênh boundary ID 4/6/8 vài frame |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | NEEDS-REVIEW | eval vs gold báo IoU thấp ở MOT frame 55, 84, 106; không tự sửa chỉ để khớp gold |
| Frame giữa hai keyframe không bị interpolation drift | NEEDS-REVIEW | midpoint quanh MOT frame 55, 84, 106 cần người học soi lại trong CVAT |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | 595 bbox, 8 ID, frame 1–190, validator 0 lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | PASS cho 2 finding trước gold | người học xác nhận đã sửa job #12; hai gap biến mất sau export |

## Self-QC attestation

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS theo evaluator | IDSW 0 so với gold; kiểm tiếp bằng mắt nếu sửa thêm |
| 2 — endpoint/scope | ĐÃ SỬA; còn boundary difference | CVAT 56–61 / ID 4 và 164 / ID 8 đã sửa; eval vẫn báo chênh vào/ra vài frame |
| 3 — geometry/interpolation | NEEDS-REVIEW | MOT 55, 84, 106 có IoU gần ngưỡng 0.5 |

## Exit ticket

1. Finding quan trọng nhất: `Outside` phải đặt tại frame đầu không còn phần xe nhìn thấy; hai finding ở ID 4 và 8 đã đóng `fixed`.
2. Peer `not-a-defect`: N/A vì không có reviewer.
3. Rule cần làm rõ: ngưỡng bắt đầu/kết thúc track khi xe chỉ lộ rất nhỏ ở rìa ảnh; ghi ở `GUIDELINE_MINI.md`.
