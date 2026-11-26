# Báo cáo Day 13 — Bùi Kim Trại, 2A202603176

## 1. Kết luận trước

Lớp **L23**, task **D13-2A202603176**, tài khoản `kimtrai205`; Windows, Microsoft Edge; CVAT server/UI **2.74.1** (đọc từ About). MSSV người kiểm chéo: **2A202603233**. Bảng A và phiếu tự kiểm được cập nhật sau Save và `python scripts/lab.py sync`.

Có **92 cuboid / 10 frame**, gồm 81 Car, 3 Truck, 3 Bus, 5 Cyclist; 91 hộp có tâm trong 50 m. Số hộp từng frame: s01f05: 10; s01f58: 8; s02f02: 7; s02f34: 8; s03f00: 12; s03f41: 5; s04f09: 13; s04f33: 9; s05f12: 13; s05f41: 7. Frame ít nhất là `s03f41` (5 hộp): lựa chọn hiện tại còn hạn chế và có một hộp ngoài 50 m; đây là điểm yếu về phạm vi cần ghi nhận. Phiếu tự kiểm còn **1/92 hộp có cờ**. Các xe gần có nhiều điểm là phần có cơ sở hình học rõ hơn; class của xe thân cao, đầu xe nhỏ và vật xa còn cần thận trọng. Số hộp và số cờ không chứng minh nhãn đã khớp nhãn chuẩn.

Cách dùng CVAT: xoay và zoom khung 3D, nhìn từ trên xuống để đọc trục, dời tâm nhìn bằng điều khiển camera. Vẽ dạng **Shape**, chọn/nháy đúp cuboid rồi chỉnh sáu mặt qua Top/Side/Front. Bật **Cuboid orientation**, đối chiếu đầu xe với camera và dùng `preview` để xem hình chiếu hộp; số ID giao diện CVAT khác `idx` trong mỗi frame của Bảng A.

## 2. Đọc frame

Ảnh: `t1-s01f05-3d.png`, `t1-s01f05-top.png`, `t1-s01f58-3d.png`, `t1-s01f58-top.png`. Hai ảnh Top có chú thích hướng bổ sung, giữ nguyên nội dung ảnh chụp bên dưới. Hướng trên màn hình phụ thuộc góc camera; phía trước của hệ VF luôn là **−y**.

| Frame | Hướng trên ảnh Top đã lưu | Mặt đường | Vật xa nhất đã đọc và gán | Loại vật và phía sau/hai bên | Cụm chưa đọc chắc |
|---|---|---|---|---|---|
| s01f05 | Trước: xuống; +x: phải; +y: lên | Tương đối phẳng gần xe; khoảng −1,8 m quanh LiDAR, không áp một cao độ cho toàn cảnh | Car #10: 38,49 m | 9 Car, 1 Truck; Car #1 ở sau-trái, Truck #7 cũng sau-trái | Điểm cây, cột và vùng xa phía sau; không suy thành xe nếu thiếu cơ sở |
| s01f58 | Trước: lên; +x: trái; +y: xuống | Tương đối phẳng gần xe; khoảng −1,8 m | Car #8: 22,15 m | 8 Car; #1, #2 ở phía sau, các xe còn lại phía trước; cần cả camera hai bên/sau | Các vật nhỏ trên vỉa hè và vùng khuất sau hàng xe chưa được kết luận class |

Ba thông tin camera cung cấp trong hai frame: màu thân xe (xe sáng phía trước, xe xanh ở góc sau-trái); chi tiết đèn/kính để phân biệt đầu–đuôi; ngữ cảnh làn đường và cấu trúc thân xe để hỗ trợ class. Point cloud đơn lẻ không cung cấp trực tiếp màu bề mặt hay các chi tiết ảnh đó.

Ba thông tin LiDAR cung cấp trực tiếp: vị trí tâm và khoảng cách theo mét; chiều cao/độ cao đáy so với LiDAR và mặt đường; phân bố điểm 360° gồm phía sau và vùng thiếu điểm do khuất. Ảnh camera đơn lẻ không cho trực tiếp các số đo 3D theo mét này. Cao độ đường nêu trên là ước lượng, có thể thay đổi theo vị trí.

Tám frame lướt dùng **BEV cố định trong `preview`**, đầu xe hướng lên (khác góc tự xoay của ảnh CVAT):

| Frame | Phía trước trên BEV preview | Điều cần chú ý từ nhãn/phiếu tự kiểm |
|---|---|---|
| s02f02 | Trên (−y) | Bus gần bên trái; Cyclist #6 ở 34,77 m chỉ 32 điểm |
| s02f34 | Trên (−y) | Car #6 ở 45,35 m chỉ 48 điểm |
| s03f00 | Trên (−y) | Có bóng che; Car #10 đã bật occluded |
| s03f41 | Trên (−y) | Car #3 ở 54,06 m, ngoài phạm vi chấm 50 m |
| s04f09 | Trên (−y) | Car #7 ở 37,84 m chỉ 75 điểm; Bus ở phía sau-trái |
| s04f33 | Trên (−y) | Cyclist #3 gần, Car #9 ở 46,40 m chỉ 61 điểm |
| s05f12 | Trên (−y) | Các xe hai bên/đằng sau; Car #13 xa 47,25 m |
| s05f41 | Trên (−y) | Xe thân cao #5 còn cờ height; có Bus rất gần |

## 3. Ba cuboid có hướng dẫn

| Vật / ảnh | Frame, idx / class | Hướng yaw | Đáy và ba view | Đối chiếu kích thước bảng 4.3 |
|---|---|---|---|---|
| (a) `t2-a-car.png` | s01f05 #1 / Car | 271,6°, gần −y; đối chiếu camera góc sau | Top/Side/Front có trong ảnh; gap +0,07 m, gần đường | 4,62 × 1,96 × 1,44 m; rộng nhỉnh hơn mốc sedan 1,90 m |
| (b) `t2-b-bus-truck.png` | s01f05 #7 / Truck | 90,0°, gần +y | Cả ba view có trong ảnh; gap −0,07 m, gần đường | 6,14 × 2,31 × 3,20 m, trong khoảng xe tải nhẹ |
| (c) `t2-c-ped-cyclist.png` | s02f02 #6 / Cyclist | 265,3°, gần −y; đầu vật nhỏ còn khó xác nhận | Cả ba view có trong ảnh; gap +0,19 m | 1,29 × 0,58 × 1,42 m: chiều dài ngắn hơn cả mốc xe đạp; chỉ 32 điểm, cần kiểm độ bao phủ, không coi hết cờ là đúng |

Đã tham gia vòng hiệu chuẩn theo xác nhận của học viên. Chưa có ghi chép cụ thể về độ lệch giữa các bạn hay kết luận riêng của coach, nên không tự ghi một kết quả thống nhất. Khi đối chiếu dùng quy tắc trong guideline: +X cục bộ chỉ đầu vật, đáy chạm đường, một vật một Shape.

Vùng không gán thành vật: các điểm rời trong ảnh `s03f00` ở mục 4 chưa xác định được vật tương ứng. Ghi **LOW_VISIBILITY/UNCERTAIN_CLASS**, không tạo thêm cuboid để giải thích mọi cụm điểm.

## 4. Bốn hiện tượng chất lượng

Các ảnh đều từ task cá nhân, không mượn frame. Mọi nguyên nhân dưới đây là **giả thuyết**, không phải kết luận đã xác minh.

| Hiện tượng | Frame / tên ảnh | Giả thuyết nguyên nhân | Quan sát | Hệ quả khi gán nhãn |
|---|---|---|---|---|
| Bóng che | s03f00 / `t4-1-bong-che.png` | Xe/hàng rào chắn đường truyền tia | Các vùng sau cấu trúc đặc có ít điểm hơn phần mặt đường nhìn thấy; #10 có occluded | Vùng trống chưa chắc là đường trống; bật occluded khi vật bị che, bao phần có đủ cơ sở suy luận |
| Thưa ở xa | s04f09 / `t4-2-thua-o-xa.png` | Khoảng cách làm số tia rơi lên vật giảm | Car #7 ở 37,84 m chỉ 75 điểm, thưa hơn các xe gần | Áp quy tắc thấy khoảng 15–20% thể tích hoặc đủ cơ sở suy luận; thiếu cơ sở thì bỏ qua và ghi LOW_VISIBILITY |
| Phản xạ thấp / ít điểm | s01f05 / `t4-3-phan-xa-thap.png` | Bề mặt tối/kính có thể phản xạ yếu; góc quét, che khuất cũng có thể góp phần | Car #6: 253 điểm ở 14,52 m; Car #2: 616 điểm ở 14,57 m; khoảng cách gần bằng nhau nhưng số điểm khác rõ | Không kết luận vật nhỏ chỉ vì ít điểm; kiểm camera và ba view, không nới hộp theo nhiễu |
| Điểm lạ | s03f00 / `t4-4-diem-la.png` | Có thể là phản xạ nhiều đường hoặc điểm rời; cũng chưa loại trừ bề mặt thật | Có dải điểm lẻ ở vùng trống phía trên dải đường trong góc chụp, chưa xác định vật tương ứng; ảnh này chưa đủ để kết luận nằm dưới đường | Không gán cụm đó thành vật; cần xoay ngang và đối chiếu camera trước khi kết luận nguyên nhân |

## 5. Số từ cuboid của tôi

Nguồn: `submission/D13-2A202603176-bangA.csv` và `submission/D13-2A202603176-tukiem.csv` sau lần sync cuối. `dist_m` là khoảng cách ngang; r là khoảng cách 3D tới tâm. Gap dương: đáy cao hơn đường ước lượng, âm: thấp hơn; không thay cao độ đường cục bộ bằng hằng số −1,7 m.

| Cuboid | z (m) | h (m) | z − h/2 (m) | dist_m (m) | r (m) | yaw (°) | Gap đường (m) |
|---|---|---|---|---|---|---|---|
| `s01f05 #1` Car | -0,990 | 1,440 | -1,710 | 5,51 | 5,595 | 271,6 | 0,07 |
| `s01f05 #7` Truck | -0,320 | 3,200 | -1,920 | 21,78 | 21,786 | 90,0 | -0,07 |
| `s02f02 #6` Cyclist | -0,930 | 1,420 | -1,640 | 34,77 | 34,782 | 265,3 | 0,19 |

Cả ba đáy gần mặt đường ước lượng (|gap| < 0,3 m); Cyclist mẫu cao hơn đường 0,19 m nên vẫn cần xem lại điểm bánh/đáy, không khẳng định tiếp đất chính xác.

- **Gần nhất:** `s01f58 #1`, dist_m = 2,98 m; r = 3,125 m.
- **Xa nhất toàn bảng:** `s03f41 #3`, dist_m = 54,06 m; r = 54,062 m. Hộp này **ngoài 50 m, không được chấm**, thuộc UNCERTAIN_SCOPE. Xa nhất trong phạm vi là `s05f12 #13`: 47,25 m; r = 47,256 m.
- **Kích thước:** Car mẫu #1 rộng 1,96 m, hơn tối đa sedan 0,06 m; gương/phần chìa ra hoặc cách căn mép là giả thuyết cần đối chiếu. Cyclist mẫu #6 dài 1,29 m dưới mốc xe đạp 1,60–1,90 m, không được suy ra là đúng chỉ vì hết cờ. Đối chiếu thêm `s04f33 #3` Cyclist: 2,07 × 0,82 × 1,66 m, trong các khoảng xe máy có người (1,80–2,30; 0,70–0,95; 1,40–1,80 m).
- **Yaw:** xe phía trước cùng chiều `s01f05 #2`: 271,6°, gần 270°; xe ngược hướng `s01f05 #4`: 89,4°, gần 90°. Không coi dấu của y là bằng chứng về hướng di chuyển: phải nhìn đầu/đuôi trên camera.

Mật độ của bốn Car trong cùng frame `s01f05`:

| Cuboid | dist_m (m) | points_in (điểm) |
|---|---|---|
| `s01f05 #1` Car | 5,51 | 2610 |
| `s01f05 #2` Car | 14,57 | 616 |
| `s01f05 #5` Car | 26,26 | 213 |
| `s01f05 #8` Car | 31,05 | 162 |

Từ #1 đến #8, xa gấp 5,64 lần, số điểm còn 6,2%. Xu hướng giảm phù hợp đường cong tham khảo 1968 → 718 → 214 → 72 điểm ở các bin 0–10 → 10–20 → 20–30 → 30–40 m; không coi bốn vật khác nhau là thí nghiệm giữ nguyên kích thước, bề mặt và che khuất.

Các sửa đổi đã lưu và xuất lại (đối chiếu bản nhãn trước sửa với Bảng A hiện tại):

| Cuboid | Trước sửa → hiện tại | Kết quả hiện tại |
|---|---|---|
| s04f09 #9 Car | Rộng 2,489 → 2,150 m; dịch tâm theo mép vật | Gap +0,22 m; không còn cờ chiều rộng |
| s04f09 #12 Bus | Rộng 3,635 → 2,800 m; dịch tâm | Gap +0,12 m; không còn cờ chiều rộng |
| s05f12 #12 Cyclist | z −1,064 → −1,280 m; h 1,208 → 1,600 m | Đáy −2,080 m, gap +0,05 m; không còn cờ đáy |

**Cờ giữ lại:** `s05f41 #5 Car`, cao 2,36 m, vượt mốc rộng của công cụ 2,30 m. Camera cho thấy vật thân cao dạng van; chưa xác định chắc công năng chở khách/chở hàng, nên giữ class và hình học hiện tại, ghi **UNCERTAIN_CLASS/chiều cao cần kiểm**, không đổi class hoặc ép thấp hộp chỉ để hết cờ. Điều này chưa chứng minh class Car và chiều cao đều đúng.

**Kiểm chéo:** người kiểm có MSSV **2A202603233**. Theo xác nhận của học viên, bạn đã đối chiếu hai số trong báo cáo với Bảng A/phiếu tự kiểm trên màn hình:

| Số được kiểm | Giá trị trong báo cáo | Kết quả đối chiếu |
|---|---|---|
| Đáy cuboid `s01f05 #1` | −1,710 m | Khớp Bảng A |
| `points_in` của `s01f05 #8` | 162 điểm | Khớp phiếu tự kiểm |

Cả hai số đều khớp, không cần sửa số liệu sau kiểm chéo.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Model dự đoán vị trí và kích thước trong không gian thật nên nhãn 3D cần đơn vị mét; pixel thay đổi theo camera và khoảng cách, không thể thay cho độ dài vật. Vật thưa điểm cần quy tắc riêng để model không học hộp ôm một mẩu quan sát hoặc hộp được suy đoán không có cơ sở.

- **Công cụ AI:** ChatGPT/Codex.
- **Câu hỏi chính:** cách dùng lệnh/CVAT, rà soát cảnh báo, đọc số đo và hoàn thiện bài nộp.
- **Cách dùng kết quả:** AI đã hỗ trợ xem dữ liệu/ảnh, đề xuất và hỗ trợ chỉnh một số cuboid trong các lượt trước, tính số liệu, soạn báo cáo và thêm chú thích hướng. Báo cáo khai rõ các phần có AI hỗ trợ; kết quả cần được học viên tự đối chiếu, không coi số cờ bằng điểm số.

Điều còn chưa chắc: đầu/độ dài Cyclist ít điểm; class và chiều cao của van `s05f41 #5`; vật ngoài 50 m ở `s03f41`; nguyên nhân của dải điểm lẻ ở `s03f00`. Không có kết quả nhãn chuẩn nên chưa biết recall, IoU hay số hộp ngược đầu thực tế.
