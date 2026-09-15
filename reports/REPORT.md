# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Cư Đức Quang (2A202602188) — làm cá nhân
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT local 2.75.0, `localhost:8080`; task/job #11 warm-up, #12 core; export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | Không ghi log thời gian thực; không suy từ thời điểm lưu task |
| Thời gian gán `clip_01` | Không ghi log thời gian thực; không suy từ thời điểm lưu task |
| Số track đã vẽ trong `clip_01` | 8 track, 595 bbox / 190 frame |
| Số keyframe trung bình mỗi track | 27,25 (218 keyframe / 8 track trong CVAT job #12) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe buýt ID 4 mới vào rìa phải (CVAT 50–65): bắt đầu khi nhận ra xe, bbox chạm rìa, giữ cùng ID khi thân xe lộ dần.
2. Sedan ID 5 bị xe buýt che một phần (CVAT 78–115): giữ ID, đánh dấu `Occluded`, không kết thúc track khi xe vẫn còn trong cảnh.
3. Xe đỏ ID 8 ra sát rìa phải (CVAT 164–171): sửa `Outside` quá sớm, giữ bbox đến frame cuối còn phần xe nhìn thấy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: 8 ID trong MOT; chấm với gold hiện tại không có IDSW. Chưa có video tự-QC lưu lại cho cả ba lượt.
- Lượt 2: phát hiện `Outside` sớm ở ID 4 (CVAT 56/MOT 57) và ID 8 (CVAT 164/MOT 165); người học đã sửa job #12 và export lại.
- Lượt 3: diagnostic so gold gợi ý soi midpoint/geometry ở MOT 55/ID 4, 84/ID 5 và 106/ID 6; chưa tự nhận đã đóng các finding này.

Kiểm chéo với: không có — bài làm cá nhân. Chi tiết self-QC ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: N/A. Số lỗi bạn ấy tìm được trong bản của bạn: N/A.

Ca hai người quyết khác nhau: N/A. Luật còn thiếu là ngưỡng nhận dạng xe chỉ lộ rất nhỏ ở rìa và frame đầu thực sự vắng mặt để đặt `Outside`; đã bổ sung vào `GUIDELINE_MINI.md`.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `cb569880e9745894d662f65d177ba1bee553e4b876c66b1ebc58f674f8ccf686` |
| Thời điểm khóa | Export CVAT ZIP lúc 04:02:57 UTC **trước gold**; snapshot/manifest audit tạo muộn lúc 09:17:36 UTC |
| Số row / frame / track trước khi mở reference | 595 row / 190 frame / 8 track; SHA-256 của entry `gt/gt.txt` trong ZIP export trùng snapshot và nhãn hiện tại |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.831 | 0.818 | 0.847 | 0.883 | 0.966 | 0.930 | 0.873 | 31 | 9 | 0 |
| Sau rework* | 0.831 | 0.818 | 0.847 | 0.883 | 0.966 | 0.930 | 0.873 | 31 | 9 | 0 |

*ZIP export gốc của CVAT có trên máy lúc 04:02:57 UTC, trước `gold.zip` tải lúc 04:38:32 UTC và upload lên fork lúc 04:49:29 UTC; entry nhãn trùng SHA-256 với snapshot. **Khóa hash/Coach xác nhận trước gold không được thực hiện**: manifest chỉ được lập lúc 09:17:36 UTC để audit nguồn cũ. Bản hiện tại không có rework theo gold, vì vậy hai hàng metric bằng nhau.

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** cho bản nhãn hiện tại.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| `Outside` sớm — self-QC trước lần chấm này | CVAT 56 / MOT 57 | 4 | Người học sửa trong CVAT, export lại; không còn gap 56–61 |
| `Outside` sớm — self-QC trước lần chấm này | CVAT 164 / MOT 165 | 8 | Người học sửa trong CVAT, export lại; không còn gap ở frame 164 |
| Chênh boundary/geometry so gold | MOT 51–53/ID 4, 98–100/ID 6, 133–135/ID 8; MOT 55/84/106 | 4/5/6/8 | Chỉ đánh dấu `needs-review`; chưa sửa nhãn theo gold hoặc sửa trực tiếp MOT |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json` của **lần chạy Colab do người học tải về**:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` / `configs/trackers/botsort-reid.yaml` (`with_reid: true`, `gmc_method: none`) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck) |
| device | GPU `0`; `persist=True`; 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.831 | 0.818 | 0.847 | 0.883 | 0.966 | 0.930 | 0.873 | 31 | 9 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.787 | 0.729 | 0.851 | 0.907 | 0.891 | 0.775 | 0.900 | 88 | 45 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA 0.930 thấp hơn IDF1 0.966; bản nhãn có 0 IDSW nhưng còn 31 FP, 9 FN. Nếu MOTA cao mà IDF1 thấp, một xe có thể bị đổi/tách ID kéo dài: MOTA đếm sự kiện switch, còn IDF1 tính đúng ID trên toàn quãng đời.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ReID cao hơn IDF1 0.025 và AssA 0.044, nhưng IDSW cùng bằng 2. Ở MOT 56–65, bus gold ID 4: ByteTrack gán ID 14 tại 56–57 rồi ID 15 từ 59; ReID giữ ID 9 từ 55–65, một đoạn association tốt hơn. Tuy nhiên ReID lại đổi gold ID 5 từ ID 17 sang 18 ở MOT 87 và gold ID 6 từ 24 sang 31 ở 113. Đây là so hai hệ thống ByteTrack/BoT-SORT với cùng detector input, **không cô lập causal effect của ReID** vì implementation tracker khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng 0.649 → 0.711; FN giảm 54 → 26, FP tăng 88 → 91. Cải thiện phần lớn là coverage/detection, không phải giảm số IDSW; cả hai có 16 track cho 8 gold track, ghost/fragmentation vẫn còn. Model thiếu xe bị che và phát sinh bbox ngoài scope/không khớp gold, nên cần phân biệt detector miss/FP với association switch theo frame.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

MOT 104–109: xe gold ID 6 hiện một phần bên phải/đằng sau bus, nhãn tay giữ ID 6 liên tục; ReID chỉ có một bbox ID 24 ở 104 rồi vắng 105–109, đến 110 mới có ID 31. Ở MOT 106 gold có ID 6, model không có — sai coverage; bbox tay ở 106 còn hơi rộng so gold (IoU 0.503), nên chỉ khẳng định đúng sự tồn tại/ID, không nhận bbox đã hoàn hảo.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID đổi ID 17 → 18 ở MOT 87 cho sedan gold/nhãn tay ID 5. Chuỗi 85–94 của nhãn tay và gold giữ ID 5, cùng quỹ đạo qua phần bị bus che; đây là model split, không phải lý do tách annotation. Nhưng bbox tay MOT 84–88 rộng hơn phần xe lộ ra (IoU so gold 0.528 tại 84), nên cần soi lại geometry trong CVAT, không sửa MOT trực tiếp chỉ để tăng điểm.

## 6. Nếu phải gán thêm 10 clip nữa

Trong `GUIDELINE_MINI.md`, chốt ngưỡng bắt đầu/kết thúc track cho xe chỉ lộ vài pixel ở rìa bằng frame đầu xác định được xe bốn bánh và frame đầu hoàn toàn vắng mặt. Quy trình đề xuất: đặt keyframe dày tại rìa/occlusion, kiểm midpoint sau mỗi đoạn dài, lưu bằng nút Save rồi reload xác nhận, ghi thời gian gán và khóa pre-gold **trước** khi nhận reference/model. Tác giả cần xác nhận đây là reflection của mình.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` — source export trước gold, manifest audit lập muộn; chưa đạt formal lock trước gold
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` — self-QC; peer N/A
- [x] `reports/REPORT.md` (file này)
