# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Chu Thien Phu
- MSSV: 2A202603114
- Email: 26ai.phuct@vinuni.edu.vn
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/chuphu2004/K5-L2A-DAY02-ChuThienPhu-2A202603114-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/16qsREJkM0RxQMa1HXB8DY2BLWxjM6gTA

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema 4 lớp giữ nguyên thứ tự `car=0`, `truck=1`, `bus=2`, `van=3`; xe bán tải và SUV tính là `car`. Box ôm sát phần thân xe nhìn thấy; xe bị che vẫn gán nếu còn nhận ra lớp; xe bị cắt rìa thì box chạm đúng rìa ảnh.

Ba ca mơ hồ:

- `drive_044`: xe bị cắt bởi rìa ảnh, khó phân biệt lớp → gán `bus` vì thấy dãy cửa sổ dọc thân xe.
- `drive_030` và `drive_027`: cụm xe ô tô ở rất xa, không tách được viền từng xe → **không gán nhãn** (chỉ gán khi tách được từng xe).

Phần cần giảng viên làm rõ: ngưỡng kích thước/vị trí để một xe ở xa vẫn phải gán. So với gold (mục 4), gold không gán phần lớn xe ở vùng xa phía trên khung hình, còn em gán khá nhiều xe nhỏ tách được ở vùng này.

## 3. Annotation và self-QC

- Công cụ: CVAT, export định dạng YOLO 1.1 (`obj.names` = car, truck, bus, van).
- Số file nhãn train: 50/50 (không có file rỗng).
- Phân bố lớp bản khóa ban đầu: `car` 992, `truck` 26, `bus` 118, `van` 76 — tổng 1212 box. `truck` là lớp hiếm nhất.
- Kết quả checker: `python3 tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ.` (cả bản khóa ban đầu và bản sau rework). Không có lỗi định dạng cần xử lý.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: từ drive_002 đến drive_019
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: không phát hiện lỗi
- Commit khóa nhãn ban đầu: [`d8b3d36`](https://github.com/chuphu2004/K5-L2A-DAY02-ChuThienPhu-2A202603114-ObjectDetection/commit/d8b3d36f5275a02415f208df7ac2437719d8927f)

## 4. Gold QC và rework

| Chỉ số | Trước sửa (`d8b3d36`) | Sau sửa |
|---|---:|---:|
| TP | 470 | 476 |
| FP | 742 | 673 |
| FN | 183 | 177 |
| Precision | 0.3878 | 0.4143 |
| Recall | 0.7198 | 0.7289 |
| F1 | 0.5040 | 0.5283 |
| mean_iou_of_matches | 0.8121 | 0.8120 |
| passed (theo script, ngưỡng F1 0.80) | false | false |

- Kết quả sau sửa: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json); trước sửa: [`outputs/annotation_evaluation_before_rework.json`](../../outputs/annotation_evaluation_before_rework.json).
- Gold có 653 box (car 533, truck 46, bus 20, van 54). Bản nhãn sau sửa có 1149 box (car 941, truck 27, bus 113, van 68).
- **Ngưỡng cổng:** giảng viên thông báo hạ ngưỡng F1 xuống **≥ 0.50** (thông báo lúc 4h20). Bản sau sửa đạt F1 = 0.528 ≥ 0.50 và mean IoU = 0.812 ≥ 0.70. Script vẫn in `passed: false` vì ngưỡng 0.80 được viết cứng trong `tools/evaluate_annotations.py`; em không sửa script.

### Phân tích lệch so với gold (trước sửa)

Em so khớp box không xét lớp (IoU ≥ 0.50), vẽ gold và nhãn của em cạnh nhau trên từng ảnh, rồi cắt từng ca ra xem:

1. **ID lớp của gold khác schema README/`data.yaml`.** 108 box trùng vị trí nhưng lệch lớp theo một quy luật cố định:

   | Gold ghi | Em ghi | Số box | Quan sát trên ảnh |
   |---|---|---:|---|
   | `van` (3) | `bus` (2) | 54 | xe buýt (vd. `drive_008`, `drive_009`) |
   | `truck` (1) | `van` (3) | 36 | xe van/minivan (vd. `drive_002`, `drive_004`) |
   | `bus` (2) | `truck` (1) | 18 | xe tải, xe ben, xe cứu hộ (vd. `drive_008`, `drive_014`) |

   Gold thực tế dùng thứ tự 1=van, 2=truck, 3=bus. Nhãn `dataset/labels/eval/` cũng lệch giống vậy (vd. xe buýt ở `drive_052`, `drive_053` ghi ID 3; minivan ở `drive_059` ghi ID 1). Em giữ nhãn theo schema của đề, không sửa gold/eval và đã báo mentor về việc lệch ID này.

2. **Khác quy ước về xe ở xa.** Nếu bỏ qua lớp, có 598 cặp khớp → precision ≈ 0.49, recall ≈ 0.92, F1 ≈ 0.64. Có 614 box của em không khớp gold (car 518, bus 61, van 29, truck 6), phần lớn rất nhỏ (trung vị ~695 px² ≈ 26×26 px; 342 box nhỏ hơn mức 10% nhỏ nhất của gold ~784 px²). Khi xem ảnh cạnh nhau (vd. `drive_002`), gold chỉ gán xe ở nửa dưới/gần camera và bỏ trống cả dòng xe phía xa, kể cả xe buýt khá rõ. Em chỉ bỏ sót hoàn toàn 55 xe gold (car 53, truck 2).

3. **Bất đồng lớp thật** (không do lệch ID): một số SUV/xe con gold ghi `car` còn em ghi `van` (vd. SUV trắng bên trái `drive_002`).

### Rework

Bản khóa `d8b3d36` được giữ nguyên trong lịch sử git. Rework gồm hai bước:

1. **Xóa box thừa ở vùng xa** (em sửa trong CVAT): xóa 63 box `car` ở `drive_030` (54→34), `drive_044` (41→17), `drive_006` (51→42), `drive_049` (29→20), `drive_003` (47→46). F1: 0.504 → 0.522; TP không đổi, FP giảm 63.
2. **Sửa lớp sau khi rà lại toàn bộ 220 box truck/bus/van bằng ảnh cắt** (theo schema của đề, **không** đổi theo ID của gold). 15 box sửa:

   | Ảnh:dòng | Trước | Sau | Lý do |
   |---|---|---|---|
   | `drive_021`:8 | truck | van | minivan bạc |
   | `drive_026`:28–32 (5 box) | bus | car | taxi, sedan, xe con nằm cạnh xe buýt |
   | `drive_005`:1, `drive_006`:16 | van | truck | xe cứu hộ, cùng loại với các box `truck` ở `drive_014`, `drive_019` |
   | `drive_005`:8, `drive_026`:6 | van | car | SUV/jeep là `car` theo schema |
   | `drive_049`:11, `drive_011`:33, `drive_004`:2, `drive_004`:6, `drive_006`:36 | van | car | sedan, hatchback, xe con cỡ nhỏ |

   Giữ nguyên các box chưa chắc: `drive_006`:20, `drive_012`:26 (van/car), `drive_001`:5, `drive_044`:2 (bus ở xa), `drive_046`:4. F1: 0.522 → 0.528.

Bản trước khi sửa lớp được lưu ngoài repo để đối chiếu.

**Kết luận:** box bám sát tốt (mean IoU 0.81). F1 thấp chủ yếu do (1) gold dùng ID lớp khác schema (lỗi này không sửa được bằng nhãn của em) và (2) em gán cả xe ở vùng xa mà gold bỏ qua. Sửa lớp theo schema gần như không đổi F1, cho thấy lớp em gán ban đầu đã khá đúng.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU Tesla T4 (14913 MiB, CUDA:0); Ultralytics 8.4.208, Python 3.13.16, torch 2.14.1+cu130.

- Các tham số thay đổi so với notebook: không thay đổi; chỉ điền `REPO_URL`. Cấu hình: `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`.
- Dữ liệu train: bộ nhãn **đã khóa ở commit `d8b3d36`** (trước rework). Em không chạy lại Colab sau rework, nên số liệu dưới đây là của model train trên bản khóa.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không commit file `.pt`).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4094 |
| Recall | 0.2340 |
| mAP50 | 0.2707 |
| mAP50–95 | 0.1993 |

Nguồn: `model_eval_metrics.json` trong `day2_colab_outputs.zip`.

Kết quả chạy:

- Ảnh dự đoán trên 10 ảnh eval: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/) (vd. [`drive_051.jpg`](assets/eval_predictions/visuals/drive_051.jpg), [`drive_053.jpg`](assets/eval_predictions/visuals/drive_053.jpg), [`drive_056.jpg`](assets/eval_predictions/visuals/drive_056.jpg))
- Log train theo epoch: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv), [`results.png`](assets/yolo26_runs/vehicle_annotation/results.png)
- Confusion matrix trên eval: [`assets/yolo26_runs/vehicle_eval/confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png); PR curve: [`BoxPR_curve.png`](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png)

Dữ kiện quan sát được:

- **`best.pt` rơi vào các epoch rất sớm.** Epoch 1: mAP50 = 0.271, mAP50–95 = 0.200; epoch 50: 0.206 và 0.138, dù train loss vẫn giảm (cls_loss 4.80 → 0.94). Model càng học theo nhãn train thì điểm trên eval càng thấp.
- **Confusion matrix:** `car` khớp 83 box. Trong 12 box eval có nhãn `van` (ID 3), model dự đoán **`bus` cho 11 box**; trên ảnh đó là xe buýt (`drive_052`, `drive_053`, `drive_056`, `drive_058`, `drive_060`), khớp với việc nhãn eval dùng ID 3 cho bus. AP của truck/bus/van vì vậy bị tính sai.
- **Ảnh dự đoán:** `drive_051` sedan đỏ gần camera bị đoán `van` 0.99; `drive_054` xe con đỏ bị đoán `van` 0.97; `drive_056` một xe con có hai box chồng nhau (`car` 0.87 và `van` 0.34); `drive_059` xe nhỏ ở xa đoán `truck` 0.54; `drive_055`, `drive_057` nhiều xe nhỏ cuối đường bị bỏ sót hoặc confidence thấp (0.26–0.36). Xe `car` gần camera được phát hiện tốt, box sát (0.7–0.87).
- Nhiều dự đoán bị tính là `background` (FP). Giả thuyết: một phần do model học gán cả xe ở xa (giống bộ nhãn train) trong khi nhãn eval không gán các xe này.

Trả lời bốn câu hỏi cuối notebook (số liệu lấy từ [confusion matrix trên eval](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)):

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   Theo nhãn eval, `van` (ID 3) có recall thấp nhất: 0/12 box được đoán đúng lớp. Tuy nhiên 12 box này trên ảnh đều là **xe buýt**, và model đoán `bus` cho 11/12 box (vd. xe buýt lớn ở [`drive_053`](assets/eval_predictions/visuals/drive_053.jpg) được đoán `bus` 0.86, [`drive_060`](assets/eval_predictions/visuals/drive_060.jpg) `bus` 0.88). Recall thấp này do nhãn eval lệch ID (mục 4), không phải do model không nhận ra xe buýt.
   Nếu xét theo loại xe thật, lớp yếu nhất là **van/truck**:
   - 5 box eval ID 1 (trên ảnh là minivan) chỉ có 1 box được đoán `truck`, 4 box bị đoán `car`. Ví dụ xe nhỏ ở xa trong [`drive_059`](assets/eval_predictions/visuals/drive_059.jpg) được đoán `truck` 0.54.
   - Hai lớp này ít dữ liệu train nhất (`truck` 26 box, `van` 76 box so với `car` 992).
   - `car` có recall cao nhất: 83/94 box.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   Khi đó box đã tìm đúng vật thể nhưng chưa sát. Cần kiểm tra:
   - cạnh box có lấn nền hay hụt thân xe không;
   - xe nhỏ ở xa và xe bị che/cắt khung, vì vài pixel lệch đã làm IoU tụt mạnh;
   - box của nhãn train có nhất quán không, vì box train lỏng thì model học box lỏng.

   Ở lần chạy này mAP50 = 0.271 và mAP50–95 = 0.199 (tỉ lệ ≈ 0.74), nên chênh lệch không lớn. Xe `car` gần camera có box khá sát ([`drive_054`](assets/eval_predictions/visuals/drive_054.jpg), [`drive_058`](assets/eval_predictions/visuals/drive_058.jpg)). Mean IoU nhãn train so với gold là 0.81, nên vấn đề chính của model là sai lớp và bỏ sót, không phải độ sát của box.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   **Nhầm `car` ↔ `van`.** Trên eval, model đoán `van` với confidence rất cao cho xe con:
   - sedan đỏ ở [`drive_051`](assets/eval_predictions/visuals/drive_051.jpg) (`van` 0.99);
   - xe con đỏ ở [`drive_054`](assets/eval_predictions/visuals/drive_054.jpg) (`van` 0.97);
   - xe xám ở [`drive_056`](assets/eval_predictions/visuals/drive_056.jpg) (`van` 0.95), và một xe ở ảnh này có hai box chồng nhau `car` 0.87 / `van` 0.34.

   Dữ kiện hỗ trợ:
   - Colab train trên bản khóa `d8b3d36`. Khi rà lại sau gold QC, em thấy bản này có 7 xe con/SUV bị gán `van` (vd. `drive_004`, `drive_005`, `drive_049`; xem mục 4). Model đã học theo các nhãn sai đó.
   - Guideline của em định nghĩa `van` theo hình dáng ("khối liền, đuôi/mui khác car") nhưng chưa nêu rõ ranh giới với SUV, MPV và hatchback cao, nên khi gán nhãn dễ lệch.
   - Ngoài ra, quy ước "xe ở xa có gán hay không" trong guideline khác với nhãn eval. Model học gán cả xe nhỏ ở xa, nên có nhiều dự đoán bị tính là `background` (FP).

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   Nếu train trên eval, model sẽ được chấm trên chính ảnh nó đã học. Khi đó metric bị thổi phồng và không còn cho biết model tổng quát hóa ra sao trên ảnh mới. Trong notebook, `data.yaml` đặt `val: images/eval`, nên 10 ảnh này được dùng sau mỗi epoch để tính mAP và chọn `best.pt`. Ở lần chạy này `best.pt` được chọn ở epoch rất sớm. Vì eval đã tham gia vào việc chọn checkpoint, nó là tập **validation**, không phải test set độc lập: kết quả có phần lạc quan, và với chỉ 10 ảnh thì metric dao động mạnh.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP4 — commit khóa nhãn | `Author identity unknown` khi `git commit` | Cấu hình `user.name`/`user.email` cho git | Commit và push thành công (`d8b3d36`) |
| CP5 — nhận gold | Gold phát dạng `.rar`, máy chưa có công cụ giải nén | Cài `unrar` | Giải nén đủ 50 file gold, chạy được evaluator |
| CP5 — gold QC | F1 = 0.504 < 0.80; 108 box lệch lớp theo quy luật cố định | So khớp không xét lớp, vẽ gold/nhãn cạnh nhau, xem ảnh cắt từng ca | Xác định gold/eval dùng ID 1=van, 2=truck, 3=bus khác schema; không sửa gold/eval |
| CP5 — rework | Nhiều box thừa ở vùng xa, một số box sai lớp | Xóa 63 box thừa ở 5 ảnh; rà 220 box truck/bus/van, sửa 15 box | F1 0.528, mean IoU 0.812 — đạt ngưỡng F1 ≥ 0.50 mới của giảng viên |
| CP6 — Colab | Link mở từ GitHub không lưu output | Save a copy in Drive và chia sẻ link | Có link Colab đã chạy |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Điểm | Tình trạng |
|---|---:|---|
| Hoàn thành và định dạng YOLO | 10 | 50/50 file, checker đạt |
| Đầy đủ vật thể | 20 | Recall 0.73 so với gold (0.92 nếu bỏ qua lớp); FN chủ yếu do lệch ID lớp của gold, chỉ 55 xe gold bị bỏ sót hoàn toàn |
| Độ bám sát và hình học box | 20 | Mean IoU 0.81 (mức xuất sắc ≥ 0.80) |
| Phân lớp nhất quán | 15 | Đã rà lại 220 box truck/bus/van, sửa 15 box; còn một số ca van/car mơ hồ |
| Mini guideline và quyết định mơ hồ | 10 | Có định nghĩa 4 lớp, quy tắc box, 3 ca mơ hồ |
| Gold-set annotation QC | 15 | Có JSON trước/sau rework; F1 0.528 đạt ngưỡng 0.50 mới; vòng rework và nguyên nhân được giải thích |
| Pipeline YOLO và phân tích | 10 | Colab chạy được, giữ nguyên split, có metrics, ảnh eval và nhận xét lỗi; mô hình train trên bản nhãn trước rework |

## 8. Công cụ AI và tài liệu tham khảo

- **Claude Code (Anthropic):** đọc và tóm tắt tài liệu bài lab; hướng dẫn thao tác CVAT (tạo project/task, phím tắt, export YOLO 1.1); copy file export vào repo, chạy `check_yolo_labels.py`, `evaluate_annotations.py`; viết script phân tích lệch lớp/box thừa so với gold, vẽ gold và nhãn cạnh nhau, cắt ảnh minh họa; rà 220 box truck/bus/van bằng ảnh cắt và đề xuất sửa 15 box (em đồng ý áp dụng); giải nén kết quả Colab; soạn nháp báo cáo.
- **Phần em tự làm và tự kiểm tra:** gán nhãn 50 ảnh trong CVAT, viết mini guideline và 3 ca mơ hồ, rà lại ảnh sau khi gán, xóa box thừa trong CVAT, chạy notebook trên Colab, xem ảnh so sánh gold/nhãn và ảnh dự đoán, đọc và chỉnh lại báo cáo. Em không dùng model (YOLO26 hay công cụ tự động khác) để pre-label; toàn bộ box được vẽ tay trong CVAT.
- **Công cụ khác:** CVAT (gán nhãn), Google Colab + Ultralytics (train/eval).
- Không dùng nguồn dữ liệu/nhãn ngoài repo.
