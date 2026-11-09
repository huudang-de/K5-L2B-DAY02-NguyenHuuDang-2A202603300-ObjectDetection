# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hữu Đăng
- MSSV: 2A202603300
- Email: dang36390@gmail.com
- Lớp: L2B
- Link repo bài làm: https://github.com/huudang-de/K5-L2B-DAY02-NguyenHuuDang-2A202603300-ObjectDetection
- Link Colab đã chạy:

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

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| | | | |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.

## 8. Công cụ AI và tài liệu tham khảo

Ghi công cụ đã dùng, dùng ở bước nào và phần đã tự kiểm tra; không dùng thì ghi rõ. Liệt kê nguồn ngoài repo được sử dụng. Không ghi mật khẩu/token hoặc đính kèm gold train.
