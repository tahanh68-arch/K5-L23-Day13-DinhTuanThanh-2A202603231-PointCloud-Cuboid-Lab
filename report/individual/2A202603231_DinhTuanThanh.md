# Báo cáo Day 13 — Đinh Tuấn Thành, 2A202603231

## 1. Kết luận trước

Task `D13-2A202603231` (CVAT task 26), thực hiện trên Windows, Microsoft Edge, CVAT 2.81.0. Mười frame và số cuboid: s01f23: 12; s01f40: 6; s02f07: 14; s02f22: 11; s03f16: 10; s03f50: 6; s04f07: 9; s04f52: 5; s05f07: 15; s05f42: 12.

Bảng A có **100 cuboid**: 56 Car, 6 Bus, 5 Truck, 19 Pedestrian và 14 Cyclist. Frame s04f52 ít nhất, với 5 nhãn; đây là số vật đã gán, chưa chứng minh cảnh chỉ có 5 vật hoặc đã gán đủ. Phiếu tự kiểm gắn cờ **27/100 cuboid**. Vật gần, có nhiều điểm và camera rõ dễ kiểm hơn; đáy hộp, kích thước vật bị che và đầu–đuôi vật xa còn cần rà soát. Chưa có thông tin xác nhận MSSV người kiểm chéo.

Cách thao tác CVAT:

- Xoay, thu/phóng và dời góc nhìn trong khung 3D; dùng góc nhìn top để đọc vị trí tương đối và trục tọa độ.
- Chọn Shape/Cuboid để vẽ; nháy đúp hộp để tập trung, đối chiếu Top, Side, Front rồi chỉnh tâm, kích thước và góc quay.
- Bật **Cuboid orientation**: vạch đỏ ngắn là +X của hộp, chỉ về đầu/hướng đi của vật. So với các camera tương ứng và ảnh `preview`; không áp hướng ego cho mọi vật.

## 2. Đọc frame (Bước 2)

Ảnh: `t1-s01f23-3d.png`, `t1-s01f23-top.png`, `t1-s01f40-3d.png`, `t1-s01f40-top.png`. Với góc top đã chụp, **+x sang phải, +y lên trên; phía trước ego là −y, ở cạnh dưới ảnh**. Đây là trục toàn cảnh, khác trục riêng của từng cuboid.

| Frame | Mặt đường | Khoảng cách đã có nhãn | Loại vật và phía sau/hai bên | Chưa chắc |
|---|---|---|---|---|
| s01f23 | Vùng gần tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | Nhãn xa nhất khoảng 23,64 m; không phải giới hạn nhìn của cảm biến | Cảnh giao lộ; xe con, xe buýt, người và xe hai bánh. Xe buýt phía sau lệch bên; camera bên hỗ trợ phân biệt người và xe | Cụm nhỏ phía xa và vùng thiếu điểm sau xe lớn chưa đủ để suy luận thêm vật |
| s01f40 | Vùng gần tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | Nhãn xa nhất khoảng 25,49 m | Xe con sáng màu sát bên; xe tối màu phía sau; người và xe hai bánh phía vỉa hè trong camera bên | Cây che phần thấp; xe hai bánh không rõ người lái cần phân biệt với xe dựng |

Camera bổ sung ba thông tin khó đọc từ cloud: màu sáng/tối của xe; dấu hiệu đầu–đuôi qua thân/kính xe; người đang ngồi trên xe hay đứng/dắt xe. LiDAR bổ sung ba thông tin metric: khoảng cách tới ego; độ cao vật so với đường; kích thước và hình học 3D của bề mặt trả điểm.

Tám frame còn lại dùng quy ước phía trước −y; giữ cùng góc top thì nằm ở cạnh dưới. Ghi chú sau dựa trên nhãn và phiếu tự kiểm, không khẳng định đã tìm hết hiện tượng.

| Frame | Số hộp | Điểm cần chú ý |
|---|---:|---|
| s02f07 | 14 | Car #13 xa nhất tập nhãn; Car #11 thưa điểm |
| s02f22 | 11 | Car #3 và #11 có cảnh báo đáy–đường |
| s03f16 | 10 | Truck #1 gần nhất; một số Car bị cờ kích thước |
| s03f50 | 6 | Chưa kết luận hiện tượng đặc biệt |
| s04f07 | 9 | Car #6 cần kiểm chiều cao và đáy |
| s04f52 | 5 | Ít nhãn nhất; cần phân biệt ít vật với bỏ sót |
| s05f07 | 15 | Dùng bốn Car để so số điểm theo khoảng cách |
| s05f42 | 12 | Hộp #10–#12 bị cờ đáy, cần xem đường cục bộ |

## 3. Ba cuboid mẫu (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. `idx` lấy từ CSV, có thể khác ID hiển thị trên CVAT.

| Mẫu | Frame, idx, class | Hướng | Đáy và L/W/H (m) | Đối chiếu |
|---|---|---|---|---|
| (a) Xe con gần | s01f40, #1, Car | 274,8°, gần −y | Đáy −1,628; 4,526/1,897/1,368 | 3D và camera nhận ra xe gần. Dài/rộng trong khoảng sedan; cao thấp hơn mốc 1,40 m khoảng 0,032 m, cần kiểm nóc/đáy |
| (b) Xe buýt | s01f23, #5, Bus | 269,6°, gần −y | Đáy −1,697; 13,033/3,535/3,310 | Ảnh có top và view phụ. Cao trong khoảng xe buýt thường, dài và đặc biệt rộng vượt bảng 4.3; cần rà mặt bên |
| (c) Xe đạp có người | s01f40, #2, Cyclist | 261,9°, lệch −y khoảng 8,1° | Đáy −1,493; 1,629/0,546/1,764 | Camera bên giúp nhận ra người cùng xe; ba kích thước trong khoảng xe đạp có người; cần kiểm đáy theo đường cục bộ |

Chưa có thông tin xác nhận vòng hiệu chuẩn và kết luận chung của lớp, nên không ghi kết quả so với bạn khác. Ca cần hỏi thêm: s01f40, phía vỉa hè trong camera bên, xe hai bánh bị cây che và không rõ người lái (**UNCERTAIN_CLASS / LOW_VISIBILITY**). Chưa có nhật ký đầy đủ để xác nhận từng ca đã bỏ qua hay đã gán; không suy đoán số vật bỏ qua.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Cả bốn trường hợp từ task cá nhân 26, không mượn frame. Nguyên nhân là **giả thuyết**, chưa phải kết luận lỗi cảm biến.

| Hiện tượng | Frame / ảnh | Quan sát và giả thuyết | Hệ quả khi gán |
|---|---|---|---|
| Bóng che | s01f23 / `t4-1-bong-che.png` | Vùng sau xe buýt thiếu điểm; thân xe lớn có thể chặn tia | Không kéo hộp sang khoảng trống hoặc tự thêm vật. Bật occluded khi vật thật sự bị che, bao phần khuất suy luận được |
| Thưa ở xa | s02f07 / `t4-2-thua-o-xa.png` | Car #11 ở 35,50 m có 102 điểm; khoảng cách và góc quét có thể làm giảm điểm | So camera và kích thước tham chiếu; không căn chặt theo vài điểm lẻ |
| Phản xạ thấp / ít điểm bất thường | s01f40 / `t4-3-phan-xa-thap.png` | Car #3 tối màu phía sau, 14,56 m, 795 điểm, trả điểm không đều; vật liệu, kính, góc tới đều có thể ảnh hưởng | Không coi vùng thiếu điểm trên thân là biên vật; so camera và các view phụ |
| Điểm lạ | s01f23 / `t4-4-diem-la.png` | Điểm rời mờ ở vùng dưới-trái khung 3D, thấp hơn lớp đường; cần mở ảnh gốc và phóng lớn để thấy. Có thể do phản xạ nhiều đường hoặc nhiễu, chưa xác định cơ chế | Không kéo đáy theo điểm lẻ; kiểm lớp đường liên tục và camera trước khi coi là vật |

Trong CSV hiện tại, occluded đều bằng 0. Vì vậy phần bóng che là nhận xét cần kiểm thuộc tính, chưa xác nhận mọi thuộc tính đã đúng.

## 5. Số từ cuboid của tôi (Bước 5)

Nguồn: `submission/D13-2A202603231-bangA.csv` và `submission/D13-2A202603231-tukiem.csv`. Khoảng cách dùng m, góc dùng °. dist_m = √(x²+y²); r tính từ x, y, z đã làm tròn trong CSV. z đáy dùng cột xuất; tính lại từ số đã làm tròn có thể lệch 0,001 m.

| Frame, idx, class | z | h | z đáy | dist_m | r | yaw_deg | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| s01f40, #1, Car | −0,944 | 1,368 | −1,628 | 3,13 | 3,272 | 274,8 | ground_gap +0,18 m, dưới ngưỡng cờ 0,3 m |
| s01f23, #5, Bus | −0,041 | 3,310 | −1,697 | 14,00 | 13,996 | 269,6 | ground_gap +0,06 m; rộng còn đáng nghi |
| s01f40, #2, Cyclist | −0,611 | 1,764 | −1,493 | 9,97 | 9,990 | 261,9 | ground_gap −0,11 m; so đường tại vị trí vật |

Gần nhất: **s03f16 #1 Truck**, dist_m 2,80 m, r 2,910 m. Xa nhất: **s02f07 #13 Car**, dist_m 48,22 m, r 48,236 m, 55 điểm; phiếu chưa ước lượng được ground_gap. Không kết luận chìm chỉ từ đáy −2,002 m khi chưa biết đường cục bộ. Ở hàng Bus, r tính lại nhỏ hơn dist_m đã làm tròn khoảng 0,004 m là do độ chính xác hiển thị.

So bảng 4.3, Cyclist s01f40 #2 phù hợp xe đạp có người ở ba chiều; Car s01f40 #1 hơi thấp so với khoảng sedan. Thiếu điểm nóc hoặc đặt mặt hộp chưa sát đều là khả năng cần kiểm. Xe theo −y có yaw gần 270° (Car s01f40 #1: 274,8°); xe ngược hướng có yaw gần 90° (Car s05f07 #11: 90,0°). Hướng đầu xe vẫn cần camera xác nhận.

| Cuboid cùng frame s05f07 | dist_m (m) | points_in |
|---|---:|---:|
| #1 Car | 4,77 | 4979 |
| #3 Car | 11,61 | 1299 |
| #11 Car | 26,57 | 253 |
| #15 Car | 39,73 | 84 |

Khoảng cách tăng 8,33 lần thì số điểm còn khoảng 1/59 (1,69%). Xu hướng giảm phù hợp bài giảng: khoảng 1968, 718, 214, 72 và 19 điểm ở dải 0–10, 10–20, 20–30, 30–40 và 60–80 m. Đây là các xe khác nhau; chưa thể suy ra quy luật chính xác vì kích thước, che khuất và góc nhìn cũng ảnh hưởng.

Các cờ còn cần rà: s01f23 #1 có ground_gap −0,77 m và cao 2,418 m; s01f23 #5 Bus rộng 3,535 m; s02f22 #3 có ground_gap +0,54 m. Giữ số xuất từ task để phản ánh tình trạng hiện tại, chưa coi các hộp này là đúng và không sửa CSV để tắt cờ. Chưa có thông tin xác nhận kiểm chéo, MSSV đối tác và hai số đã đối chiếu; cần bổ sung khi thực sự thực hiện.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Model học vị trí, kích thước và hướng trong không gian thực nên nhãn 3D dùng mét; kích thước pixel thay đổi theo camera và khoảng cách. Vật thưa điểm cần quy tắc riêng vì vài điểm không mô tả toàn thể tích; bám sát điểm lẻ dễ làm hộp thiếu thân hoặc sai đáy.

- **Công cụ:** ChatGPT/Codex và công cụ đề xuất cuboid chạy cục bộ.
- **Yêu cầu chính:** thiết lập repo/CVAT, xử lý lỗi lệnh, thử tạo hộp từ cloud/camera, kiểm hướng vạch đỏ, hỗ trợ tổng hợp báo cáo và ảnh.
- **Cách dùng:** AI hỗ trợ tổng hợp số liệu hai CSV và hồ sơ; không coi đề xuất hoặc selfcheck là nhãn chuẩn.

Còn chưa chắc về đầu–đuôi vật ít điểm, đáy theo đường cục bộ, xe buýt quá rộng, thuộc tính occluded và khả năng bỏ sót vật sau cây. Chưa xác định các lỗi được phát hiện trước hay sau hạn nộp; chưa có kết quả kiểm chéo và hiệu chuẩn. Báo cáo ghi rõ các giới hạn thay vì khẳng định toàn bộ nhãn đúng.
