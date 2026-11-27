# Báo cáo Day 13 — Nguyễn Thị Ngọc Mai, 2A202603188

## 1. Kết luận trước

- **Mã số/task:** 2A202603188, task `D13-2A202603188`.
- **10 frame:** `s01f18`, `s01f59`, `s02f20`, `s02f40`, `s03f25`, `s03f42`, `s04f23`, `s04f42`, `s05f06`, `s05f24`.
- **Máy làm bài:** Windows 11; Google Chrome; tài khoản CVAT `nmai`; CVAT Server/UI **2.83.0** (About).
- **Người ngồi cạnh:** MSSV 2A202603079.

Bảng A có **76 cuboid**. Frame ít cuboid nhất là `s03f25` với 2 cuboid; hai vật nằm xa nên có ít điểm LiDAR hơn và không thể ghi rõ vật che cận. Phiếu `selfcheck` có cảnh báo cho **2/76 cuboid**, đều liên quan đến chiều rộng của xe con vượt khoảng tham chiếu. Tôi tin phần đáy và kích thước xe gần nhất là tương đối chắc, còn yếu nhất là kích thước và hướng của vật xa; class của vật thưa điểm cũng còn có khả năng hiểu sai.

Trong CVAT, tôi đổi góc nhìn bằng cách xoay khung 3D, chọn chế độ nhìn từ trên xuống và dời khung Top/Side/Front để nhìn toàn cảnh. Để vẽ, tôi chọn Shape, chọn label, tạo cuboid rồi nháy đúp để chỉnh; sau đó kiểm ranh giới ở Top, Side và Front. Tôi bật Cuboid orientation để xem mũi tên đầu xe và chạy `python scripts/lab.py preview` để so cuboid với ảnh camera.

## 2. Đọc frame (Bước 2)

Trong ảnh nhìn từ trên xuống, phía trước xe là cạnh **−y**; trục **+x** chỉ về bên trái ảnh và trục **+y** chỉ lên phía trước trong khung toạ độ. Ảnh: `t1-s01f18-3d`, `t1-s01f18-top`, `t1-s01f59-3d`, `t1-s01f59-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f18 | Phía trước ở cạnh −y, tương ứng phía dưới ảnh; +x về trái, +y về phía trước | Đường tương đối phẳng; mặt đường xấp xỉ −1,7 m dưới tâm LiDAR | Xe bus khoảng 27,21 m; nhiều xe con phía sau tới khoảng 25,72 m | Xe con, xe buýt | Hai xe con ở hai bên/phía sau; xe buýt phía trước bên phải | Không nhìn rõ các vật rất xa hoặc có ít điểm ở rìa |
| s01f59 | Phía trước ở cạnh −y, tương ứng phía dưới ảnh; +x về trái, +y về phía trên | Đường tương đối phẳng; mặt đường xấp xỉ −1,7 m | Xe con khoảng 16,39 m; hai người đi bộ khoảng 13,59–14,32 m | Xe con, người đi bộ | Xe ở hai bên, phía sau và các làn ngược chiều | Không chắc một số vật quá thưa điểm ở mép |

Ba điều **chỉ camera** cho biết: màu và kiểu thân xe, vị trí đầu–đuôi xe, cũng như người đang đứng hay đang di chuyển. Ba điều **chỉ LiDAR** cho biết: khoảng cách tới tâm cuboid, độ cao z và hình dạng/bao điểm 3D giúp chỉnh đáy và kích thước.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f20 | Cạnh dưới (−y) | Có xe con, người đi bộ và xe hai bánh |
| s02f40 | Cạnh dưới (−y) | Xe tại khoảng 35,44 m nên điểm ít |
| s03f25 | Cạnh dưới (−y) | Chỉ có hai xe con, hai vật khá xa |
| s03f42 | Cạnh dưới (−y) | Xe xa nhất khoảng 47,65 m |
| s04f23 | Cạnh dưới (−y) | Mật độ xe lớn; có 20 cuboid |
| s04f42 | Cạnh dưới (−y) | Có xe con, xe hai bánh và xe ở khoảng xa |
| s05f06 | Cạnh dưới (−y) | Có xe tải ở gần khu vực trung tâm |
| s05f24 | Cạnh dưới (−y) | Có xe con và xe đạp có người ở hai bên |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f18, idx 1 | Car | Có, yaw 269,3° gần hướng −y/cùng chiều trước | Có, z_bottom = −1,842 m, ground_gap = −0,01 m | Có | Dài 4,479 m, rộng 1,960 m, cao 1,562 m; thuộc khoảng Car/SUV hợp lý |
| (b) xe buýt hoặc xe tải | s01f18, idx 4 | Bus | Có, yaw 270,7° theo hướng trước | Có, z_bottom = −1,962 m, ground_gap = 0,10 m | Có | Dài 11,714 m, rộng 2,767 m, cao 3,273 m; dài/rộng/cao gần quy tắc xe buýt thường |
| (c) người đi bộ hoặc xe hai bánh | s01f59, idx 7 | Pedestrian | Mũi tên 209,4° theo hướng mặt/thân người trong point cloud | Không hoàn toàn, z_bottom = −0,977 m và ground_gap = 0,35 m, chỉ cẩn thận lơ lửng nhẹ | Có | Dài 0,362 m, rộng 0,382 m, cao 1,189 m; dài/rộng hợp lý cho người trưởng thành, cao thấp hơn khoảng tham chiếu 1,50–1,95 m |

Vòng hiệu chuẩn: cuboid dễ lệch nhất ở chiều rộng và hướng Top, đặc biệt với xe dài và vật nhỏ. Cả lớp thống nhất giữ đáy sát mặt đường ở Side/Front, cuboid bao trọn phần vật nhưng không bao dư quá mức; vật thưa điểm chỉ gán khi có đủ camera và hình dạng LiDAR. Không có vật nghi ngờ bị bỏ qua trong ba mẫu; ở bài của tôi không ghi nhận ca UNCERTAIN_CLASS/LOW_VISIBILITY cần loại bỏ chắc chắn.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s05f24 / `t4-1-bong-che.png` / frame của tôi | Xe hoặc vật đứng trước có thể che vùng phía sau và làm thiếu điểm | Có một vùng tối/sparse phía sau xe; phần thân bị che không còn điểm LiDAR | Bật occluded và mở rộng cuboid để bao phần suy luận, không thu cuboid theo phần điểm còn thấy |
| 2. Thưa ở xa | s03f42 / `t4-2-thua-o-xa.png` / frame của tôi | Nhiễu và mật độ tia LiDAR giảm theo khoảng cách | Một xe ở xa khoảng 47,65 m chỉ còn vài cụm điểm rời rạc | Chỉ vẽ nếu vật còn đủ cơ sở suy luận; chấp nhận 15–20% thể tích điểm, không vẽ vật xa nếu không nhận ra hình dạng |
| 3. Phản xạ thấp / ít điểm bất thường | s05f24 / `t4-3-phan-xa-thap.png` / frame của tôi | Vật tối hoặc diện phản xạ thấp có thể thu ít tín hiệu | Một số cụm điểm nhỏ và thưa trên bề mặt xe/vật | Giữ kích thước hợp lý hơn kích thước chỉ theo điểm, nhưng vẫn kiểm Top/Side/Front và không ép hộp chạm điểm nếu thiếu cơ sở |
| 4. Điểm lạ | s04f23 / `t4-4-diem-la.png` / frame của tôi | Điểm rời có thể là bụi, tay nắm, ô che hoặc ngưỡng nền | Có cụm điểm lạ không liền thành thân xe hoàn chỉnh | Không gán tự động thành cuboid; chỉ gán khi điểm tạo thành hình dạng vật với camera hoặc điểm lân cận đủ rõ |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f59, idx 2, Car | −0,988 m | 1,524 m | −1,750 m | 3,08 m | 3,26 m | 270,6° | Chạm đường; lệch mặt đường nhỏ |
| s01f59, idx 7, Pedestrian | −0,383 m | 1,189 m | −0,977 m | 13,59 m | 13,60 m | 209,4° | Lơ lửng nhẹ so với mặt đường xấp xỉ −1,7 m |
| s03f42, idx 4, Car | −1,002 m | 1,919 m | −1,962 m | 47,65 m | 47,66 m | 271,1° | Chạm đường |

(a) Cuboid gần nhất là `s01f59, idx 2, Car`: `dist_m = 3,08 m`, `r = 3,26 m`. Xa nhất là `s03f42, idx 4, Car`: `dist_m = 47,65 m`, `r = 47,66 m`.

(b) Người đi bộ `s01f59, idx 7` có l × w × h = 0,362 × 0,382 × 1,189 m. Chiều cao 1,189 m thấp hơn khoảng người lớn 1,50–1,95 m, có thể do đánh dấu chỉ phần điểm thân hoặc xác định là người nhỏ; dài/rộng vẫn nằm trong khoảng người lớn. Xe con `s02f20, idx 5` có 4,828 × 2,331 × 1,813 m: chiều rộng 2,331 m vượt nhẹ khoảng Car 1,70–2,30 m và chiều cao cao hơn khoảng sedan/hatchback nhưng còn trong khoảng SUV 1,65–2,10 m; đây có thể là SUV.

(c) Xe cùng chiều phía trước `s01f59, idx 2` có yaw 270,6°, gần 270°, nên hướng đặt đúng. Xe ngược chiều `s02f20, idx 5` có yaw 91,9°, gần 90°, nên hướng ngược lại.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s05f06, idx 1, Car | 3,95 m | 5691 |
| s01f59, idx 5, Car | 16,39 m | 437 |
| s04f23, idx 5, Car | 35,66 m | 88 |
| s03f42, idx 4, Car | 47,65 m | 39 |

Khoảng cách tăng gần gấp 12 lần từ 3,95 m lên 47,65 m thì số điểm xe con giảm từ 5691 còn 39 điểm, tức mật độ giảm rất mạnh theo khoảng cách. Khuynh hướng này phù hợp đường cong slide: xe con có khoảng 1968 điểm ở 0–10 m và chỉ còn 19 điểm ở 60–80 m; số điểm cụ thể của tôi bị ảnh hưởng bởi góc nhìn và vật che.

Hai cuboid tôi đã sửa sau preview/selfcheck là `s01f18, idx 5, Car` và `s02f20, idx 5, Car`: tôi giảm/điều lại chiều rộng ở Side/Top để hướng theo điểm LiDAR và nằm sát ranh giới kích thước; sau đó giữ hai cảnh báo theo phiếu vì phép đo hiện vẫn sát giới và phần điểm/ảnh cho thấy chiều rộng có thể thuộc SUV. Các cảnh báo còn lại không thấy.

Đổi kiểm: MSSV 2A202603079 đã kiểm chéo với tôi. Hai con số được đối chiếu với bảng A là:

- `s05f06, idx 1, dist_m = 3,95 m`: khớp giữa bảng A và phiếu tự kiểm.
- `s04f23, idx 5, z_bottom = -1,384 m`: khớp giữa giá trị tính z − h/2 và bảng A.

Hai con số khớp; không có lệch cần sửa lại.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D bằng mét giúp model học kích thước, vị trí và hướng thực trong không gian, chứ không phụ thuộc độ phân giải hay kích thước pixel của từng camera. Đối với vật thưa điểm, model cần quy tắc riêng để suy phần che và không dùng ngay số điểm còn thấy làm kích thước chuẩn.

- **Công cụ AI:** không sử dụng AI để tạo cuboid hay điền số liệu.
- **Câu hỏi:** không đặt.
- **Cách dùng:** không dùng kết quả AI; toàn bộ số liệu và cảnh báo lấy từ bảng A, phiếu tự kiểm và các ảnh đã chụp.

Điều còn chưa chắc là độ chính xác hướng và kích thước của người đi bộ thưa điểm, cùng khả năng Cuboid của xe xa bị che lệch mép. Có thể phát hiện thêm lỗi sau hạn do đổi góc nhìn hoặc so lại bằng camera; phần này chỉ ghi nhận bằng lời, không chép số trong phiếu hồ sơ lỗi.
