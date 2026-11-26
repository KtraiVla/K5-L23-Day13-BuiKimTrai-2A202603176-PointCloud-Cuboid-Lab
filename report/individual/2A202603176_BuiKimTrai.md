# Báo cáo Day 13 — Bùi Kim Trại, 2A202603176

## 1. Kết luận trước

Lớp **L23**, task **D13-2A202603176**, tài khoản `kimtrai205`; Windows, Microsoft Edge; CVAT server/UI **2.74.1** (đọc từ About). Bạn ngồi cạnh cùng lớp: MSSV **2A202603233**. Bảng A và phiếu tự kiểm được cập nhật sau khi Save trong CVAT và chạy `python scripts/lab.py sync`.

Em đã vẽ và hoàn thiện **92 cuboid / 10 frame**, gồm 81 Car, 3 Truck, 3 Bus và 5 Cyclist; trong đó 91 hộp có tâm nằm trong phạm vi 50 m. Số hộp cụ thể ở từng frame: s01f05: 10; s01f58: 8; s02f02: 7; s02f34: 8; s03f00: 12; s03f41: 5; s04f09: 13; s04f33: 9; s05f12: 13; s05f41: 7. Frame có ít hộp nhất là `s03f41` (5 hộp) do cảnh này mật độ xe thưa và có 1 hộp nằm ở 54,06 m (ngoài mốc 50 m). Phiếu tự kiểm chỉ còn **1/92 hộp có cờ cảnh báo** (xe van thân cao). Em tự tin nhất ở nhóm xe con cự ly gần có mật độ điểm dày, rõ cụm hình học; còn với các vật ở xa, xe thân cao hoặc xe hai bánh thưa điểm, em đối chiếu kỹ thêm ảnh camera để gán nhãn thận trọng nhất có thể. Em hiểu rằng số hộp đạt chuẩn và việc ít cờ tự kiểm là cơ sở tự tin bước đầu, nhưng chất lượng nhãn vẫn cần sự tỉ mỉ ở từng góc xoay và độ bám đáy.

Về thao tác trên CVAT: Em xoay và zoom khung nhìn 3D, chuyển sang góc nhìn từ trên xuống (Top) để đọc hệ trục và dời tâm nhìn bằng chuột. Mỗi vật được vẽ ở dạng **Shape**, sau đó nháy đúp để tinh chỉnh kích thước và vị trí qua 3 mặt chiếu Top/Side/Front. Em luôn bật **Cuboid orientation** để kiểm tra mũi tên hướng đầu xe khớp với ảnh camera và chạy lệnh `preview` để kiểm tra hình chiếu 3D lên 7 camera. Em cũng lưu ý ID hiển thị trên giao diện CVAT là số thứ tự chung, khác với cột `idx` theo từng frame trong Bảng A.

## 2. Đọc frame

Ảnh minh chứng: `t1-s01f05-3d.png`, `t1-s01f05-top.png`, `t1-s01f58-3d.png`, `t1-s01f58-top.png`. Trên hai ảnh Top, em có chú thích thêm các trục tọa độ để tiện theo dõi mà vẫn giữ nguyên ảnh chụp gốc từ CVAT. Trong hệ tọa độ VF, phía trước xe luôn là trục **−y**.

| Frame | Hướng trên ảnh Top đã lưu | Mặt đường | Vật xa nhất đã đọc và gán | Loại vật và phía sau/hai bên | Cụm chưa đọc chắc |
|---|---|---|---|---|---|
| s01f05 | Trước: xuống; +x: phải; +y: lên | Tương đối phẳng gần xe; khoảng −1,8 m quanh LiDAR, không áp một cao độ cho toàn cảnh | Car #10: 38,49 m | 9 Car, 1 Truck; Car #1 ở sau-trái, Truck #7 cũng sau-trái | Điểm cây, cột và vùng xa phía sau; không suy thành xe nếu thiếu cơ sở |
| s01f58 | Trước: lên; +x: trái; +y: xuống | Tương đối phẳng gần xe; khoảng −1,8 m | Car #8: 22,15 m | 8 Car; #1, #2 ở phía sau, các xe còn lại phía trước; cần cả camera hai bên/sau | Các vật nhỏ trên vỉa hè và vùng khuất sau hàng xe chưa được kết luận class |

Ba thông tin chỉ camera cung cấp trong hai frame: màu thân xe (xe sáng phía trước, xe xanh ở góc sau-trái); chi tiết đèn/kính/biển số để phân biệt rõ đầu–đuôi xe; ngữ cảnh làn đường và cấu trúc thân xe để hỗ trợ xác định chính xác class. Point cloud đơn lẻ không cung cấp trực tiếp màu sắc hay các chi tiết ảnh đó.

Ba thông tin chỉ LiDAR cung cấp trực tiếp: tọa độ tâm và khoảng cách chính xác theo mét; chiều cao và độ cao đáy hộp so với LiDAR và mặt đường; phân bố điểm 360° quanh xe bao gồm phía sau và các vùng khuất. Ảnh camera đơn lẻ không cho trực tiếp các số đo 3D theo mét này. Cao độ đường nêu trên là ước lượng cục bộ, có thể thay đổi nhẹ theo địa hình.

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
| (c) `t2-c-ped-cyclist.png` | s02f02 #6 / Cyclist | 265,3°, gần −y; đầu vật nhỏ còn khó xác nhận | Cả ba view có trong ảnh; gap +0,19 m | 1,29 × 0,58 × 1,42 m: chiều dài ngắn hơn khoảng xe đạp chuẩn do khuất đuôi xe; chỉ 32 điểm nên em căn chỉnh thận trọng theo cụm điểm |

Em đã tham gia vòng hiệu chuẩn cùng lớp ở đầu Bước 3. Quá trình thảo luận giúp em nắm chắc và áp dụng chặt chẽ các quy tắc trong guideline: trục +X cục bộ (mũi tên hướng) luôn chỉ về phía trước của vật thể, đáy cuboid phải chạm mặt đường thực tế và tuân thủ nguyên tắc mỗi vật chỉ gán một Shape duy nhất.

Các vật/vùng em bỏ qua vì không đủ cơ sở: Trong frame `s03f00`, em phát hiện một cụm điểm rời rạc (đã chụp ở ảnh hiện tượng điểm lạ `t4-4-diem-la.png`) nhưng không thể xác định hình dáng hay đối chiếu được trên camera. Em quyết định không vẽ hộp phỏng đoán mà ghi nhận là **LOW_VISIBILITY/UNCERTAIN_CLASS** theo đúng hướng dẫn.

## 4. Bốn hiện tượng chất lượng

Tất cả các ảnh minh chứng đều được em chụp trực tiếp từ task cá nhân `D13-2A202603176` (không mượn frame). Dưới đây là các quan sát và **giả thuyết nguyên nhân** của em dựa trên đặc tính vật lý của LiDAR:

| Hiện tượng | Frame / tên ảnh | Giả thuyết nguyên nhân | Quan sát | Hệ quả khi gán nhãn |
|---|---|---|---|---|
| Bóng che | s03f00 / `t4-1-bong-che.png` | Xe/hàng rào chắn đường truyền tia | Các vùng sau cấu trúc đặc có ít điểm hơn phần mặt đường nhìn thấy; #10 có occluded | Vùng trống chưa chắc là không có xe; em bật occluded cho xe bị che và chỉ vẽ phần có đủ cơ sở suy đoán |
| Thưa ở xa | s04f09 / `t4-2-thua-o-xa.png` | Khoảng cách làm số tia rơi lên vật giảm | Car #7 ở 37,84 m có mật độ rất thưa, chỉ rải rác vài chục điểm (phiếu tự kiểm ghi nhận 75 điểm, thưa hơn hẳn các xe ở gần) | Em tuân thủ quy tắc chỉ gán nhãn khi thấy ít nhất 15–20% thể tích; nếu thiếu cơ sở thì bỏ qua và ghi LOW_VISIBILITY |
| Phản xạ thấp / ít điểm | s01f05 / `t4-3-phan-xa-thap.png` | Bề mặt tối/kính có thể phản xạ yếu; góc quét, che khuất cũng có thể góp phần | Xe tối màu Car #6 ở 14,52 m bắt điểm rất kém (phiếu tự kiểm: 253 điểm), ít hơn một nửa so với Car #2 ở cùng cự ly 14,57 m (616 điểm) | Em không vội kết luận là vật nhỏ chỉ vì ít điểm; luôn đối chiếu 3 góc nhìn và camera để vẽ đúng kích thước thực của xe |
| Điểm lạ | s03f00 / `t4-4-diem-la.png` | Có thể là phản xạ nhiều đường hoặc điểm rời; cũng chưa loại trừ bề mặt thật | Có dải điểm lẻ ở vùng trống phía trên dải đường trong góc chụp, chưa xác định vật tương ứng; ảnh này chưa đủ để kết luận nằm dưới đường | Em không gán cụm điểm này thành vật; cần xoay ngang và đối chiếu camera để tránh nhận nhầm nhiễu thành chướng ngại vật |

## 5. Số từ cuboid của tôi

Dữ liệu được trích xuất từ Bảng A (`submission/D13-2A202603176-bangA.csv`) và phiếu tự kiểm (`submission/D13-2A202603176-tukiem.csv`) sau lần chạy `sync` cuối cùng. Trong đó `dist_m` là khoảng cách mặt bằng $xy$, còn $r = \sqrt{x^2+y^2+z^2}$ là khoảng cách không gian 3D từ tâm LiDAR tới tâm cuboid. Độ hở mặt đường (gap) dương nghĩa là đáy hộp cao hơn mặt đường cục bộ, âm là thấp hơn; em đo theo mặt đường thực tế thay vì áp hằng số −1,7 m.

| Cuboid | z (m) | h (m) | z − h/2 (m) | dist_m (m) | r (m) | yaw (°) | Gap đường (m) |
|---|---|---|---|---|---|---|---|
| `s01f05 #1` Car | -0,990 | 1,440 | -1,710 | 5,51 | 5,595 | 271,6 | 0,07 |
| `s01f05 #7` Truck | -0,320 | 3,200 | -1,920 | 21,78 | 21,786 | 90,0 | -0,07 |
| `s02f02 #6` Cyclist | -0,930 | 1,420 | -1,640 | 34,77 | 34,782 | 265,3 | 0,19 |

Cả 3 hộp mẫu đều có đáy bám sát mặt đường cục bộ ($|\text{gap}| < 0,3\text{ m}$). Riêng đối tượng Cyclist mẫu có đáy cao hơn mặt đường 0,19 m do phần bánh xe tiếp đất có ít điểm phản xạ, em đã căn chỉnh cẩn thận để hộp không bị chìm xuống dưới đường.

- **Gần nhất:** `s01f58 #1`, dist_m = 2,98 m; r = 3,125 m.
- **Xa nhất toàn bảng:** `s03f41 #3`, dist_m = 54,06 m; r = 54,062 m. Hộp này **ngoài 50 m, không được chấm**, thuộc UNCERTAIN_SCOPE. Xa nhất trong phạm vi là `s05f12 #13`: 47,25 m; r = 47,256 m.
- **Kích thước:** Car mẫu #1 rộng 1,96 m, nhỉnh hơn tối đa sedan khoảng 0,06 m (do gương chiếu hậu chìa ra ngoài thân xe). Cyclist mẫu #6 dài 1,29 m dưới khoảng tham khảo 1,60–1,90 m của xe đạp (do góc quét khuất một phần đuôi xe; em lưu ý điều này và không chủ quan chỉ vì công cụ tự kiểm không báo cờ). Đối chiếu thêm `s04f33 #3` Cyclist: 2,07 × 0,82 × 1,66 m, hoàn toàn nằm trong khoảng chuẩn xe máy có người (1,80–2,30; 0,70–0,95; 1,40–1,80 m).
- **Yaw:** Xe phía trước cùng chiều `s01f05 #2`: 271,6°, gần 270°; xe ngược hướng `s01f05 #4`: 89,4°, gần 90°. Em không dựa vào dấu của tọa độ y để đoán hướng di chuyển mà luôn quan sát kỹ đầu/đuôi xe trên ảnh camera.

Mật độ của bốn Car trong cùng frame `s01f05`:

| Cuboid | dist_m (m) | points_in (điểm) |
|---|---|---|
| `s01f05 #1` Car | 5,51 | 2610 |
| `s01f05 #2` Car | 14,57 | 616 |
| `s01f05 #5` Car | 26,26 | 213 |
| `s01f05 #8` Car | 31,05 | 162 |

Từ xe #1 đến xe #8, khoảng cách tăng gấp 5,64 lần ($5,51\text{ m} \to 31,05\text{ m}$) thì số điểm phản xạ giảm mạnh chỉ còn 6,2% ($2610 \to 162$ điểm). Quy luật suy giảm này phù hợp với đường cong lý thuyết trên slide bài giảng (khoảng 1968 → 718 → 214 → 72 điểm ở các bin cự ly tương ứng). Em cũng lưu ý mức suy giảm thực tế giữa các xe còn phụ thuộc thêm vào góc chiếu và diện tích bề mặt của từng loại xe.

Các sửa đổi em đã lưu và xuất lại (đối chiếu bản nhãn trước sửa với Bảng A hiện tại):

| Cuboid | Trước sửa → hiện tại | Kết quả hiện tại |
|---|---|---|
| s04f09 #9 Car | Rộng 2,489 → 2,150 m; dịch tâm theo mép vật | Gap +0,22 m; không còn cờ chiều rộng |
| s04f09 #12 Bus | Rộng 3,635 → 2,800 m; dịch tâm | Gap +0,12 m; không còn cờ chiều rộng |
| s05f12 #12 Cyclist | z −1,064 → −1,280 m; h 1,208 → 1,600 m | Đáy −2,080 m, gap +0,05 m; không còn cờ đáy |

**Cảnh báo em chủ động giữ lại:** Cuboid `s05f41 #5 Car` có chiều cao 2,36 m, vượt ngưỡng cảnh báo 2,30 m của công cụ tự kiểm. Khi đối chiếu ảnh camera, đây là một xe van thân cao; do chưa đủ căn cứ để xếp vào Truck hay Bus nên em giữ nguyên class Car và kích thước hình học thật ôm trọn cụm điểm, chấp nhận giữ lại cờ tự kiểm và ghi nhận là đối tượng cần kiểm tra thêm, chứ không gượng ép bóp méo hộp chỉ để xóa cờ.

**Kiểm chéo với bạn học:** Em đã thực hiện đổi kiểm chéo với bạn ngồi cạnh có MSSV **2A202603233**. Em và bạn đã đối chiếu trực tiếp trên màn hình 2 số liệu giữa báo cáo và Bảng A / phiếu tự kiểm:

| Số được kiểm | Giá trị trong báo cáo | Kết quả đối chiếu với Bảng A / Phiếu tự kiểm |
|---|---|---|
| Đáy cuboid `s01f05 #1` | −1,710 m | Khớp Bảng A (z_bottom = −1.710 m) |
| `points_in` của `s01f05 #8` | 162 điểm | Khớp phiếu tự kiểm (points_in = 162) |

Cả hai chỉ số đều hoàn toàn trùng khớp, không cần chỉnh sửa lại số liệu.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Từ góc nhìn mô hình học máy:
- Nhãn 3D bắt buộc phải đo bằng mét trong không gian thực thay vì pixel trên ảnh 2D, vì khoảng cách và kích thước vật lý theo mét là bất biến, giúp xe tự hành tính toán được quỹ đạo và khoảng cách an toàn tránh va chạm; trong khi kích thước pixel thay đổi theo góc camera và khoảng cách xa/gần.
- Đối tượng thưa điểm ở xa cần có quy tắc gán nhãn chặt chẽ (như chỉ vẽ khi thấy đủ 15–20% thể tích hoặc đủ cơ sở hình học) để mô hình không học các nhãn vẽ mò thiếu căn cứ, tránh làm sai lệch dữ liệu huấn luyện.

(b) Sử dụng công cụ AI:
- **Công cụ:** ChatGPT / Codex.
- **Mục đích:** Tra cứu cú pháp lệnh PowerShell/Python, hỗ trợ kiểm tra format file theo hướng dẫn và hỗ trợ tính toán công thức khoảng cách 3D.
- **Cách dùng kết quả:** Em dùng AI như một công cụ hỗ trợ rà soát. Mọi thao tác vẽ và chỉnh sửa cuboid trên giao diện CVAT 3D, đọc camera, phân tích hiện tượng và viết báo cáo đều do em tự thực hiện và chịu trách nhiệm.

(c) Những điều em còn chưa chắc chắn:
- Hướng mũi tên và độ dài thực tế của người đi xe đạp khi quá ít điểm phản xạ (`s02f02 #6`).
- Phân loại chính xác cho xe van thân cao (`s05f41 #5`).
- Vật thể nằm ngoài cự ly 50 m ở `s03f41` (`dist_m = 54,06 m`).
- Bản chất vật lý chính xác của dải điểm rời ở frame `s03f00` (giả thuyết là phản xạ nhiều đường hoặc tán xạ).
