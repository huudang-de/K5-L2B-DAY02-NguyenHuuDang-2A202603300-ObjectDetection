# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` |là xe 4-7 chỗ thông thường  |Sedan, Hatchback, SUV, Crossover như Xpander, Innova, CX-5, Xe bán tải (pickup truck như Ford Ranger, Hilux) |
| `truck` |là xe tải chở hàng thùng lớn, xe ben, container/đầu kéo | Một số hãng như Hino, Howo|
| `bus` |là xe dài trường, xe buýt, thường thì chạy theo tuyến, có mã số |Có mã số, xe 29 chỗ (trường học), xe 45 chỗ, xe buýt nối toa (xe buýt 2 toa) |
| `van` |là thân xe dạng hộp chữ nhật liền khối, trần cao, đuôi thẳng đứng vuông góc, thường có cửa trượt |Suzuki Carry, Ford Trasit, Hyundai Solati... |

## Quy tắc chung

- Box bám sát vật thể: box kéo vừa khít, không thừa không thiếu, Box phải ôm sát các mép ngoài cùng của thân xe (kể cả lốp xe, cản trước/sau, gương chiếu hậu ngoài), hạn chế tối đa khoảng trống nền đường/bầu trời.
- Vật thể bị che khuất: là vật bị che khuất bởi vật khác, khi ta khéo ô thì lẫn cả 2 vật thể. Nếu xe A bị xe B che mất nửa thân, chúng ta vẽ box quanh phần nhìn thấy của chiếc xe. Nếu A lẫn b thì nên lưu lại để hỏi, tuyệt đối không vẽ 1 box to gộp chung 2 xe.
- Vật thể bị cắt bởi rìa ảnh: Nếu xe chỉ lọt một nửa vào khung hình thì viền box đặt ở viền ảnh cho đến hết phần một nửa trong khung hình, nếu không biết là cái gì thì lưu lại để hỏi.
- Vật thể rất nhỏ/mơ hồ: Nếu xe ở quá xa, chỉ là vệt mờ không thể khẳng định chắc chắn thuộc 1 trong 4 lớp -> Bỏ qua (không gán).

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| drive_001 |Có một hàng xe càng về sau ảnh càng mờ |Bỏ qua | Quá mờ để có thể cho model học |
| drive_001 |Có cả 2 vật thể là oto và người cùng chung 1 box |Chỉ vẽ box cho ô tô, bỏ qua người đi bộ  | Người đi bộ nằm ngoài 4 lớp của schema|
| drive_008 |Có 1 vật bị cắt bởi rìa ảnh không rõ là loại xe nào |Bỏ qua (không gán) | Phần thân xe lọt vào khung hình quá ít, không đủ đặc trưng để xác định chắc chắn lớp; guideline quy định không đoán mò lớp khi thiếu dữ liệu thị giác|
