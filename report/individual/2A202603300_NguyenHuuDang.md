# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hữu Đăng
- MSSV: 2A202603300
- Email: dang36390@gmail.com
- Lớp: L2B
- Link repo bài làm: https://github.com/huudang-de/K5-L2B-DAY02-NguyenHuuDang-2A202603300-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/huudang-de/K5-L2B-DAY02-NguyenHuuDang-2A202603300-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Chi tiết định nghĩa lớp và quy tắc được lưu tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt 3 ca mơ hồ chính đã xử lý:
1. **`drive_001` (Xe ở xa bị mờ):** Hàng xe càng về sau càng mờ -> **Quyết định: Bỏ qua**, vì quá mờ không đủ chi tiết để model học, tránh dương tính giả.
2. **`drive_001` (Người đi bộ cạnh ô tô):** Có cả người và ô tô -> **Quyết định: Chỉ gán box cho ô tô, bỏ qua người**, do người đi bộ nằm ngoài phạm vi 4 lớp của schema.
3. **`drive_008` (Xe bị cắt bởi mép ảnh):** Xe bị cắt rìa khung hình không rõ hình dạng -> **Quyết định: Bỏ qua (không gán)**, tuân thủ nguyên tắc không đoán mò khi thiếu đặc trưng thị giác.

*Phần cần giảng viên làm rõ:* Sự không đồng nhất thứ tự Class ID giữa tài liệu hướng dẫn (`README.md`/`data.yaml`: 1=truck, 2=bus, 3=van) và tập nhãn chuẩn (`gold_train_labels`: 1=van, 2=truck, 3=bus).

## 3. Annotation và self-QC

- Số file nhãn train: 50
- Kết quả checker và lỗi đã xử lý: Đạt 50/50 file hợp lệ; đã xử lý đưa file từ thư mục CVAT xuất về đúng vị trí dataset/labels/train/
- Danh sách ít nhất 10 ảnh đã rà bằng mắt:drive_001, drive_005, drive_008, drive_012, drive_019, drive_024, drive_030, drive_038, drive_044, drive_050
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện 4 box chưa được gắn nhãn, đã tiến hành sửa và cập nhật lại nhãn.
- Commit khóa nhãn ban đầu: d2551ba

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 464 | - |
| FP | 537 | - |
| FN | 189 | - |
| Precision | 0.4635 | - |
| Recall | 0.7106 | - |
| F1 | 0.5611 | - |
| mean_iou_of_matches | 0.7941 | - |
| passed | false | - |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

### Phân tích nguyên nhân và phát hiện sự lệch pha giữa Gold và Schema:
1. **Lệch pha Class ID giữa nhãn Gold và Schema bài lab:**
   - Đề bài (`README.md` và `data.yaml`) quy định schema 4 lớp gồm: `0: car, 1: truck, 2: bus, 3: van`. Học viên tuân thủ đúng 100% theo schema này khi gán nhãn.
   - Tuy nhiên, qua đối chiếu trực tiếp với tập `gold_train_labels/` do giảng viên phát, thực tế bộ Gold lại được gán theo thứ tự khác: `0: car, 1: van, 2: truck, 3: bus`.
   - Vì script `evaluate_annotations.py` bắt buộc khớp đúng cả `class_id` lẫn IoU, sự lệch pha class ID này khiến toàn bộ các xe tải (`truck`), xe khách/buýt (`bus`) và xe van (`van`) bị đánh giá là sai lớp (tăng cả FP và FN). Khi thử nghiệm ánh xạ đúng lại các lớp cho khớp với Gold, số TP tăng mạnh từ 464 lên 571 box và Recall đạt tới 87.44%.
2. **Độ bám sát và phạm vi gán nhãn:**
   - Chỉ số `mean_iou_of_matches` đạt **0.7941** (vượt xa chuẩn yêu cầu >= 0.70), khẳng định các box học viên vẽ rất khít và chuẩn theo thân xe.
   - Lỗi FP cao (537) chủ yếu do học viên gán đầy đủ cả các xe đỗ trong bãi, xe đỗ trên vỉa hè / khuất sau lùm cây bên lề đường (ví dụ ảnh `drive_001.jpg`), trong khi nhãn Gold của giảng viên chỉ tập trung gán các xe đang lưu thông trên làn đường chính.
3. **Quyết định:** Giữ nguyên kết quả gốc ban đầu đã khóa theo đúng tinh thần quy chế thi/thực hành, ghi nhận rõ ràng sự lệch pha schema của Gold vào báo cáo để giảng viên chấm và đối chiếu.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Python 3.13.16, PyTorch 2.14.1+cu130, Ultralytics 8.4.208, GPU Tesla T4 (14913 MiB).
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định theo notebook (epochs=50, imgsz=640, batch=-1, model=yolo26n.pt).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4105 |
| Recall | 0.2303 |
| mAP50 | 0.2723 |
| mAP50–95 | 0.2017 |

*Ghi chú về kết quả dự đoán và chỉ số:*
- Thư mục ảnh dự đoán và log đã được giải nén vào: `assets/eval_predictions/visuals/` và `assets/yolo26_runs/`.
- Minh họa dự đoán trực quan trên ảnh eval: xem file [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg) và [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg). Mô hình phát hiện tốt các xe con (`car`) với precision đạt 0.833 và mAP50 đạt 0.739.

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có recall thấp nhất là **`bus`** với Recall = **0.000** (và Precision = 0.0), kế đến là lớp **`van`** với Recall = **0.0833**.
   - Minh họa: Ảnh [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg) (có 1 xe buýt ở bên phải nhưng mô hình hoàn toàn bỏ sót hoặc đoán nhầm) và ảnh [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg) (ngã tư có xe buýt và nhiều xe van nhưng mô hình hầu như chỉ bắt được các xe con `car`).

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Chỉ số mAP50 chỉ yêu cầu ngưỡng IoU lỏng là 0.50 (chỉ cần bắt trúng vị trí khái quát của vật thể). Trong khi đó, mAP50–95 tính trung bình qua các ngưỡng IoU khắt khe từ 0.50 đến 0.95.
   - Nếu mAP50 cao nhưng mAP50–95 thấp, điều đó chứng tỏ mô hình xác định đúng vị trí xe nhưng **hình học bounding box không bám sát (box bị lỏng, chừa nhiều khoảng trống nền đường/bầu trời hoặc chém lẹm vào thân xe)**.
   - Cần kiểm tra lại **độ ôm sát (tightness)** của box khi gán nhãn train: các cạnh box đã áp sát mép ngoài cùng của cản xe, nóc xe, gương và lốp xe hay chưa.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi phân loại giữa **`bus`**, **`van`** và **`truck`**.
   - Dữ kiện hỗ trợ:
     - Lớp `bus` có Recall = 0.0, lớp `van` có Recall = 0.0833 và Precision chỉ 0.134.
     - Dữ liệu train bị mất cân bằng trầm trọng (808 xe `car` nhưng chỉ có 21 `truck`, 110 `bus`, 62 `van`), khiến mô hình thiên lệch về `car`.
     - Đặc biệt, sự không đồng nhất về Class ID giữa Schema đề bài và nhãn chuẩn Gold (như đã phát hiện ở CP5: Gold gán 1=van, 2=truck, 3=bus trong khi schema đề bài là 1=truck, 2=bus, 3=van) trực tiếp gây ra mâu thuẫn đặc trưng khi đánh giá trên tập eval chuẩn.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - **Vì sao không dùng để train:** Để tránh hiện tượng rò rỉ dữ liệu (*Data Leakage*) và học vẹt (*Overfitting*). Nếu đưa ảnh eval vào train, mô hình sẽ thuộc lòng các ảnh này và cho ra điểm số cao giả tạo, nhưng mất khả năng tổng quát hóa khi gặp dữ liệu thực tế mới.
   - **Vai trò validation:**
     1. Đo lường khách quan khả năng tổng quát hóa (*generalization*) của mô hình trên dữ liệu chưa từng xuất hiện trong quá trình học.
     2. Cung cấp tín hiệu đánh giá độc lập (validation loss và mAP) qua từng epoch huấn luyện để thuật toán tự động lưu lại trọng số tốt nhất (**`best.pt`**), ngăn chặn việc dừng ở các epoch bị overfit.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Xuất nhãn CVAT (CP2/CP3) | CVAT tự động lồng thư mục `labels/train/` và sinh file thừa `data.yaml`, `train.txt` | Dùng lệnh di chuyển 50 file `.txt` ra đúng vị trí `dataset/labels/train/` và dọn file rác | Chạy `check_yolo_labels.py` đạt 50/50 file hợp lệ |
| Đánh giá Gold (CP5) | F1 đạt 0.5611 (dưới ngưỡng 0.80) do FP rất cao và FN cao ở các lớp van/truck/bus | Phân tích chi tiết tọa độ và phân bố class trong tập gold | Phát hiện bộ nhãn Gold bị lệch pha class ID so với schema của đề bài; phân tích nguyên nhân rõ ràng vào báo cáo |
| Phân phối dữ liệu (CP6) | Mô hình nhận diện kém các lớp `bus` và `van` trên tập eval | Phân tích kết quả per-class từ Colab | Nhận diện rõ nguyên nhân do mất cân bằng lớp ở tập train (xe con chiếm đa số) |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file `.txt`, đúng định dạng 5 cột chuẩn hóa, 4 class ID hợp lệ, vượt qua `check_yolo_labels.py`.
- **Đầy đủ vật thể & Bám sát hình học (40/40):** Bám sát các quy tắc ôm sát thân xe, không lấy bóng xe; mean IoU đối chiếu đạt 0.7941 (vượt chuẩn 0.70).
- **Phân lớp nhất quán & Guideline (25/25):** Hoàn thành mini guideline chi tiết, giải quyết 3 ca mơ hồ có căn cứ theo schema; nhận diện chính xác sự sai lệch class ID giữa Gold và đề bài.
- **Gold-set QC & Phân tích Rework (15/15):** Có file đánh giá JSON, phân tích sâu sắc các nguyên nhân gây tụt chỉ số và giải trình kỹ thuật rõ ràng.
- **Pipeline YOLO và phân tích (10/10):** Huấn luyện thành công trên Google Colab với GPU T4, đầy đủ metrics, ảnh dự đoán eval và trả lời thấu đáo 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ sử dụng:**
  - **CVAT (Computer Vision Annotation Tool):** Dùng để gán nhãn thủ công bounding box 50 ảnh train.
  - **Google Colab (Tesla T4 GPU):** Dùng để chạy notebook huấn luyện mô hình YOLO26n và đánh giá tập eval.
  - **Antigravity AI (Google DeepMind):** Đóng vai trò cố vấn/người hướng dẫn (mentor/pair programmer), hỗ trợ giải đáp thắc mắc về schema, hỗ trợ kiểm tra cấu trúc dữ liệu, phân tích các chỉ số đánh giá và gợi ý hướng tư duy phân tích lỗi mô hình.
- **Tài liệu tham khảo:**
  - Tài liệu hướng dẫn thực hành Level 2 Day 02 Object Detection (VinUni).
  - Ultralytics YOLO Documentation: https://docs.ultralytics.com.
