# Guideline gán nhãn — v0.1 — 222

- Phiên bản: v0.1 — đề xuất cá nhân cho CP1, chưa phải v1.0 của nhóm.
- Ngày: 2026-11-16.
- Người viết: 222.
- Loại nhãn: bounding box trên ảnh đường phố 2D; CVAT, xuất COCO 1.0.
- Cơ sở: guideline v0, template của lab và các tình huống do 222 ghi nhận. Quyết định dưới đây là đề xuất để họp CP2; ca thiếu bằng chứng được giữ trạng thái cần reviewer. Không coi đây là kết quả đã gán hay đã QC.

## 1. Mục tiêu

Nhãn phục vụ mô hình phát hiện người thật và phương tiện thật trong cảnh đường phố. Mô hình cần phân biệt loại đối tượng và vị trí trên ảnh, không suy đoán nghề nghiệp, ý định hay việc một người có đang điều khiển xe từ tư thế đơn lẻ.

Bỏ sót người bị che hoặc người đứng sát mép ảnh làm thiếu ví dụ quan trọng; gán tượng, đồ chơi hoặc ảnh phản chiếu thành vật thật tạo nhãn dương tính sai. Guideline ưu tiên quy tắc có thể áp dụng nhất quán trên bằng chứng nhìn thấy.

## 2. Lớp và thuộc tính

### 2.1. Định nghĩa lớp

Mỗi đối tượng có đúng một lớp; người và phương tiện là hai đối tượng riêng. Các lớp bổ sung dưới đây mới là đề xuất cá nhân, chỉ dùng khi nhóm chấp thuận và cập nhật taxonomy ở CP2.

| Lớp | Trạng thái | Định nghĩa | Ví dụ | Không thuộc lớp / dễ nhầm |
|---|---|---|---|---|
| `car` | Có trong v0 | Ô tô thật dùng chủ yếu chở người, gồm sedan, hatchback, SUV; không có thùng chở hàng riêng dạng pickup. | Xe con thật, kể cả đang được vận chuyển. | Pickup thuộc `truck`; xe đồ chơi, tượng xe, hình xe trong gương không gán. |
| `truck` | Có trong v0 | Xe tải thật có cabin và kết cấu chở hàng riêng, gồm pickup và xe chở ô tô. | Pickup chở ngựa ở img_028 theo ghi nhận của 222. | Van có thân khoang kín liền; không đổi lớp chỉ vì pickup lắp nắp thùng. |
| `van` | Có trong v0 | Xe thật có khoang kín cao liền với thân xe, dùng chở người hoặc hàng, không phải pickup gắn nắp thùng. | Xe thùng kín cỡ nhỏ. | Pickup có cabin/thùng tách biệt là `truck`; không phân biệt chỉ bằng màu hoặc chiều cao mui. |
| `motorcycle` | Có trong v0 | Xe hai bánh thật có động cơ, gồm xe số, tay ga và mô tô. | Xe máy có người ngồi ở img_010. | Xe đạp không động cơ; tuk-tuk ba bánh; người trên xe là `person`. |
| `person` | Có trong v0 | Người thật xuất hiện trực tiếp trong cảnh, dù đi bộ, ngồi, đứng, mang vật hay ngồi trên xe. | Người ở img_005, img_010, img_029 theo ghi nhận của 222. | Tượng, ma-nơ-canh, chân dung quảng cáo và phản chiếu không gán. Không cần chứng minh đang lái xe. |
| `ignore_region` | Có trong v0 | Vùng bỏ qua cho vật quá nhỏ hoặc cụm không thể tách từng vật theo R-04; không phải lớp vật thể. | Cụm vật có chiều cao nhìn thấy dưới 20 px. | Không dùng để che mọi ca khó, tượng, gương hoặc vật rõ nhưng ngoài phạm vi. |
| `tuk_tuk` | Đề xuất thêm | Xe ba bánh có động cơ dạng tuk-tuk, có vị trí người điều khiển và khoang chở người/hàng. | img_004, img_016, img_035 theo ghi nhận của 222. | Xích lô không động cơ; không ép vào `car` hoặc `motorcycle`. |
| `bus` | Đề xuất thêm | Xe thật có khoang hành khách tập thể, thân dài với nhiều hàng ghế/cửa sổ. | Ca xe buýt trong bộ ảnh cần xác nhận khi khảo sát. | Van nhỏ; không dựa riêng vào kích thước ảnh để phân lớp. |
| `bicycle` | Đề xuất thêm | Xe hai bánh không động cơ, vận hành bằng bàn đạp. | Xe đạp thật nếu có trong phạm vi ảnh. | Motorcycle; xích lô ba bánh. |
| `cyclo` | Đề xuất thêm | Xích lô ba bánh không động cơ, có vị trí đạp và khoang chở người/hàng. | img_023 theo ghi nhận của 222. | Tuk-tuk có động cơ. |

**Quyết định về `rider`:** giữ mọi người thật trong `person`; không thêm lớp `rider`, thuộc tính `is_driver` hay hành động `driving`. Tư thế ngồi không đủ xác nhận đang điều khiển xe. Người ở img_005 và img_010 vẫn là `person` nếu đạt tiêu chí nhìn thấy; xe được gán riêng.

**Vật ngoài phạm vi:** không tạo lớp `statue`, `toy`, `reflection`, `cargo` hay `animal` cho bài phát hiện người/phương tiện này. Bỏ qua tượng, hình in, đồ chơi, phản chiếu và động vật theo R-02/R-07. Phương tiện thật chưa có lớp phù hợp phải được ghi để nhóm chốt thêm lớp hoặc loại khỏi phạm vi; không tự ép sang lớp gần nhất. Các lớp đề xuất chỉ trở thành nhãn hợp lệ sau khi taxonomy nhóm được khóa.

### 2.2. Thuộc tính

Giữ ba thuộc tính v0; chưa đề xuất thêm thuộc tính. Mờ, phản chiếu và tượng được xử lý bằng quy tắc lựa chọn đối tượng, không dùng thuộc tính để biến một vật ngoài phạm vi thành mẫu dương tính.

| Thuộc tính | Giá trị | Quy tắc quyết định |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần diện tích hình chiếu đối tượng bị vật khác che, so với toàn bộ hình chiếu dự kiến: `0` = 0%; `1` = trên 0% đến 30%; `2` = trên 30% đến 70%; `3` = trên 70% và vẫn nhận dạng được. Đây là ước lượng thị giác, không phải diện tích box trống. Không tính phần ngoài mép ảnh vào phần trăm che. |
| `truncated` | `no`, `yes` | `yes` khi bản thân vật thể bị khung ảnh cắt mất một phần; `no` nếu vật nằm trong ảnh, kể cả bị vật khác che. Box chạm biên chưa đủ kết luận nếu đối tượng vẫn trọn vẹn. |
| `can_review` | `false`, `true` | `true` khi đã xác định được vật và có box ứng viên nhưng còn nghi ngờ lớp, ranh giới, mức che hoặc quy tắc; ghi lý do. Không dùng một lớp tùy ý chỉ để có chỗ tick cờ. |

Với `ignore_region`, không suy diễn mức che của cả vùng: dùng `occluded_level=0`, `truncated=no` và `can_review=false` làm giá trị quy ước; chỉ đổi `can_review=true` nếu phạm vi vùng còn cần kiểm tra. Cần nhóm xác nhận quy ước này ở CP2.

## 3. Quy tắc vẽ box và ngưỡng quyết định

### R-01 — Điều kiện gán và cách xử lý thiếu thông tin

Gán khi đồng thời: vật thật xuất hiện trực tiếp; thuộc lớp đã thống nhất; có thể phân biệt một cá thể và xác định ranh giới ứng viên; box phần nhìn thấy cao **ít nhất 20 px**. Không bắt buộc phải thấy đủ một bộ phận như đầu, bánh hoặc toàn thân. Không suy ra một đối tượng bị che hoàn toàn chỉ từ bối cảnh.

Không đặt ngưỡng “độ tin cậy 80%” vì người gán không đo được xác suất này. Với vật bị che trên 70%, chỉ gán nếu phần còn thấy vẫn đủ nhận dạng lớp và tách cá thể, đặt `occluded_level=3`; còn nghi ngờ thì dùng cơ chế reviewer. Không xác định nổi lớp/box: ghi vấn đề cấp ảnh với vị trí, câu hỏi và lý do; không tạo box đoán.

### R-02 — Vật thật, vật mô phỏng và phản chiếu

Không gán người/xe chỉ tồn tại dưới dạng tượng, ma-nơ-canh, đồ chơi, hình in/quảng cáo hoặc phản chiếu trong gương, kính, mũ bảo hiểm. Không thêm class hoặc attribute cho các đối tượng bị loại này. Nếu vật thật và ảnh phản chiếu cùng xuất hiện, chỉ gán vật thật trực tiếp.

Xe thật đặt trưng bày vẫn có thể được gán; xe mô hình/trang trí thì không. Không kết luận “không chạy” đồng nghĩa với “không phải xe”. Khi chưa phân biệt được vật thật với mô hình, ghi reviewer thay vì mặc định loại.

### R-03 — Box phần nhìn thấy, che và cắt mép

Vẽ hình chữ nhật nhỏ nhất chứa các phần nhìn thấy thuộc cùng đối tượng; không kéo ra phần khuất hoặc ra ngoài ảnh. Các phần nhìn thấy rời nhau của một vật vẫn dùng một box; box có thể chứa khoảng bị che giữa chúng. Gương xe gắn liền thuộc xe; bóng đổ, vệt nhòe và vật lân cận không thuộc box.

Người bị cắt ở mép vẫn gán nếu đạt R-01, đặt `truncated=yes`. Một vật vừa bị che vừa bị cắt mép có thể mang cả hai thuộc tính. Nếu gần ngưỡng mức che hoặc khó xác định ranh giới, tick `can_review`.

### R-04 — Vật nhỏ và đám đông

Chiều cao box phần nhìn thấy **< 20 px**: không gán lớp vật riêng; đánh dấu vùng `ignore_region` ôm sát vật/cụm. Đúng 20 px được gán nếu đạt R-01. Không gom vùng ignore rộng sang đối tượng đủ lớn có thể tách được.

Đám đông: tách người đạt R-01 khi có thể xác định từng cá thể. Chỉ dùng vùng ignore cho phần cụm không thể tách; không áp đặt một số lượng người tùy ý làm ngưỡng. Tránh phủ vùng ignore lên box hợp lệ đã gán.

### R-05 — Người trên phương tiện

Mỗi người thật là một `person`, mỗi phương tiện là một đối tượng riêng. Box người không bao xe; box xe không bao người. Không cần phân biệt người điều khiển, hành khách, người ngồi nghỉ hoặc bấm điện thoại. Không đoán vai trò từ vị trí sát camera.

### R-06 — Xe chở xe hoặc xe xếp lên nhau

Gán riêng từng xe thật còn nhận dạng được và đạt R-01; không gộp xe chở và xe được chở. Với xe hỏng, chỉ gán nếu vẫn nhận dạng được một phương tiện hoàn chỉnh về cấu trúc cơ bản; thân vỏ rời hoặc bộ phận trang trí không tính là một xe riêng. Thiếu căn cứ xác định tình trạng xe thì đưa reviewer.

### R-07 — Hàng hóa, động vật và vật mang theo

Box phương tiện chứa kết cấu xe, không kéo lên theo ngựa, bao tải hay hàng hóa vượt ngoài xe. Box người chứa cơ thể và đồ mặc sát người như mũ/quần áo, không mở rộng theo vật lớn người đang mang. Nếu vật mang/hàng che cơ thể hoặc xe, cập nhật `occluded_level`. Không tạo box hàng hóa/ngựa vì ngoài phạm vi.

### R-08 — Mờ chuyển động

Mờ không tự động bị loại. Vẫn gán khi phân biệt được lớp, cá thể, ranh giới ứng viên và đạt 20 px; box không bao đuôi vệt chuyển động. Khi không xác định được ranh giới hoặc lớp, ghi vấn đề cấp ảnh; nếu có box hợp lý nhưng còn nghi ngờ thì tick `can_review`. Không gán một vệt mờ thành nhiều vật.

### R-09 — Van, pickup và xe chưa rõ loại

Ưu tiên cấu trúc thân xe: pickup có cabin và thùng hàng riêng vẫn là `truck` khi gắn nắp thùng cao; xe khoang kín liền thân là `van`. Nếu ảnh không lộ đủ dấu hiệu cấu trúc, không quyết định từ màu, kích thước hay tên gọi suy đoán; ghi reviewer. Phương tiện chưa có lớp phải theo quyết định taxonomy chung ở CP2.

### R-10 — Xe chủ (ego vehicle)

Không gán phần xe đang mang camera, như mép capo hoặc gương của xe chủ. Phương tiện thật khác trong cảnh vẫn gán theo R-01. Nếu chưa chứng minh được bộ phận thuộc xe chủ, ghi ca để reviewer xác nhận.

## 4. Ca khó và quyết định cá nhân

Các tình huống sau dựa trên ghi nhận của 222, cần đối chiếu lại ảnh gốc trước khi gửi CP1. “Cần reviewer” là câu hỏi còn mở, không phải kết quả đã xác minh. Các quyết định dùng lớp bổ sung phụ thuộc việc nhóm chấp thuận taxonomy ở CP2.

| Ảnh | Tình huống / câu hỏi | Rule ID | Quyết định đề xuất: lớp, thuộc tính, box | Lý do / điểm cần chốt |
|---|---|---|---|---|
| img_004.jpg, img_016.jpg | Tuk-tuk thuộc lớp nào? | R-09, R-05 | Đề xuất `tuk_tuk`; không ép vào motorcycle. Box xe tách người; thuộc tính che/cắt theo ảnh. | V0 thiếu lớp phù hợp; cần CP2 chấp thuận lớp mới. |
| img_005.jpg | Người đội mũ, kính sát mép; chưa rõ có lái xe. | R-01, R-03, R-05 | `person` nếu đạt R-01; không thêm thuộc tính lái xe. Chỉ `truncated=yes` nếu cơ thể thực sự bị mép ảnh cắt. | Không cần xác nhận vai trò mới gán người. |
| img_010.jpg | Người ngồi trên xe, tựa và dùng điện thoại. | R-05 | Người = `person`; xe = `motorcycle`; hai box độc lập nếu đủ điều kiện. | Tư thế không làm thay đổi lớp người; không suy đoán đang lái. |
| img_011.jpg | Tượng người/quái vật và xe trưng bày. | R-02 | Tượng không gán; không thêm `statue`. Xe: cần xác nhận xe thật hay mô hình; chỉ xe thật đạt R-01 mới gán lớp phù hợp. | Trưng bày không đủ để kết luận là mô hình. |
| img_014.jpg | Người và xe máy đi qua bị mờ. | R-08, R-05 | Gán riêng nếu đủ dấu hiệu nhận dạng/ranh giới; chưa chắc dùng `can_review`; không có box đáng tin thì ghi vấn đề cấp ảnh. | Không bỏ mọi vật mờ, không đoán từ vệt mờ. |
| img_017.jpg | Tượng người cưỡi ngựa. | R-02 | Không gán người/ngựa của tượng; không thêm lớp. | Vật mô phỏng ngoài mục tiêu. |
| img_018.jpg | Người trong biển quảng cáo. | R-02 | Không gán `person`, không thêm attribute. | Là hình in, phù hợp đề xuất ban đầu của 222. |
| img_020.jpg | Ô tô trong gương. | R-02, R-10 | Không gán ảnh phản chiếu; không thêm lớp. | Tránh coi hình phản chiếu là vật trực tiếp trong cảnh. |
| img_022.jpg | Chưa xác định dạng xe. | R-09 | Cần reviewer: ghi dấu hiệu cabin, thùng, số bánh nhìn thấy và đối chiếu định nghĩa; chưa chốt lớp. | Mô tả hiện tại chưa đủ để chọn lớp có căn cứ. |
| img_023.jpg | Xích lô. | R-09, R-05 | Đề xuất `cyclo` nếu xác nhận không động cơ; người là `person`, box riêng. | Không ép xích lô vào motorcycle/tuk-tuk; cần CP2 chốt lớp. |
| img_024.jpg | Xe vàng nằm trên xe đen. | R-06 | Nếu cả hai là xe thật đạt R-01: hai box riêng. Nếu phía trên chỉ là thân vỏ/mô hình: không gán xe phía trên. Cần reviewer xác nhận. | Vị trí xếp chồng không đủ xác định tính chất xe; không gộp chung. |
| img_027.jpg | Hình xe trên mộ. | R-02 | Không gán nếu xác nhận là tượng/trang trí; không thêm lớp tượng xe. | Quy tắc loại phải dựa vào tính chất vật mô phỏng, không chỉ địa điểm. |
| img_028.jpg | Ngựa trên pickup. | R-07, R-09 | Pickup = `truck`; box chỉ kết cấu xe, không bao chiều cao ngựa. Không gán ngựa; mức che theo phần xe bị che. | Hàng/động vật không phải bộ phận xe. |
| img_029.jpg | Người mang vật lớn. | R-07, R-03 | `person`; box ôm phần cơ thể nhìn thấy, không mở rộng theo vật mang. Ghi che khuất nếu có. | Phân biệt người với vật họ mang. |
| img_030.jpg | Ma-nơ-canh theo ghi nhận của 222. | R-02 | Không gán `person` nếu đối chiếu xác nhận ma-nơ-canh; không thêm lớp. | Không dùng hình dáng giống người làm tiêu chí duy nhất. |
| img_035.jpg | Tuk-tuk chở đồ cao/nặng. | R-07, R-09 | Đề xuất `tuk_tuk`; box kết cấu xe, không bao bao tải/hàng nhô lên; thuộc tính che theo xe. | Loại xe không đổi theo hàng chở. |
| img_037.jpg | Xe trắng: van hay truck? | R-09 | Pickup gắn nắp thùng = `truck`; thân khoang kín liền = `van`. Cần reviewer xác nhận dấu hiệu cấu trúc trên ảnh. | Không chốt từ màu trắng hoặc chiều cao mui. |
| img_038.jpg | Ô tô đồ chơi. | R-02 | Không gán `car`, không thêm lớp toy. | Mô hình ngoài mục tiêu. |
| img_040.jpg | Phản chiếu trên mũ bảo hiểm. | R-02 | Không gán vật chỉ thấy trong phản chiếu; người và xe thật trực tiếp vẫn xét riêng. | Không loại toàn bộ ảnh chỉ vì có vùng phản chiếu. |

## 5. QC và phiên bản

### 5.1. Tự kiểm trước khi gửi CP1

- Tự xem đủ img_001.jpg đến img_040.jpg; đối chiếu ID và mô tả trong bảng ca khó. Bản này không xác nhận thay người viết rằng đã hoàn tất việc xem ảnh.
- Chọn hai ảnh người trên xe để kiểm R-05, bổ sung ID thật vào bảng. Chọn ít nhất ba ca khó để ghi vào `submission/individual_notes/222.md`.
- Kiểm định nghĩa lớp có ví dụ/phản ví dụ; ngưỡng mức che không chồng lấn; không có yêu cầu đoán hành động.
- Phân biệt ca loại theo quy tắc với ca chưa đủ căn cứ cần reviewer. Mang các đề xuất lớp mới và các dòng còn mở tới CP2.
- Gửi riêng guideline cá nhân; không xem bản của thành viên khác trước khi tất cả đã nộp. Giữ nguyên bản cá nhân sau họp.

### 5.2. Ngưỡng QC của lab — giữ nguyên `taxonomy.json.rules`

| Nội dung | Ngưỡng / mẫu số |
|---|---|
| Vật nhỏ | Chiều cao box < 20 px: áp dụng R-04. |
| Ghép box để đo IAA | Ghép 1–1, IoU ≥ 0.5, không phân biệt lớp khi ghép. |
| IAA | IoU trung bình ≥ 0.85; cùng lớp ≥ 95%, chỉ trên cặp box ghép được; không có cặp ghi N/A. Báo riêng box không ghép. |
| IoU từng box | Cao ≥ 50 px: ≥ 0.85; cao < 50 px: ≥ 0.7. |
| Độ phủ | 100% lượt gán theo assignment.csv; nhóm 4 người, 40 ảnh × 2 người = 80 lượt. |
| Lỗi theo ảnh | Số ảnh có ít nhất một lỗi / số ảnh đã duyệt; ngưỡng ≤ 2%. Duyệt 40 ảnh thì 1/40 = 2,5% đã vượt ngưỡng. |
| Lỗi theo đối tượng | Số annotation đối tượng hiện có bị lỗi / số annotation đối tượng đã duyệt; ghi riêng số vật bỏ sót. |

Đây là tiêu chí áp dụng ở các pha sau, chưa phải số liệu thực tế. Kiểm script không thay thế kiểm bằng mắt: phải kiểm bỏ sót/thừa, lớp, box và thuộc tính. IAA cao không chứng minh cả hai người đều đúng.

### 5.3. Người chốt và xử lý QC

CP2: nhóm 111, 222, 333, 444 thảo luận, người điều phối 111 ghi nhận quy tắc thống nhất vào v1.0 và cập nhật taxonomy trước khi tạo bộ nhãn CVAT. Trong lượt gán độc lập không tự sửa quy tắc hoặc xem nhãn người khác; tick `can_review` hoặc ghi vấn đề cấp ảnh. CP5: chốt từng ca, ghi người quyết định, rule, phiên bản, phạm vi sửa và người kiểm lại; reviewer nên khác người sửa.

Thứ tự quyết định QC:

1. **Hold** nếu thiếu bằng chứng, còn vấn đề guideline/dữ liệu, ca chưa giải quyết hoặc mới chỉ duyệt mẫu.
2. **Rework** nếu trên 2% ảnh lỗi hoặc có lỗi lặp (ít nhất 3 lỗi cùng kiểu và chiếm ít nhất 50% tổng lỗi theo quy tắc lab); sửa đúng phạm vi và tạo vòng QC mới, giữ vòng cũ.
3. **Release** khi đã duyệt toàn bộ lô, đáp ứng ngưỡng và reviewer xác nhận đủ bằng chứng.

### 5.4. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do / ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu của lab, giữ nguyên file nguồn. | Định nghĩa lớp và ca biên còn sơ lược. |
| v0.1 | Đề xuất cá nhân 222: giữ person, đề xuất lớp phương tiện bổ sung; định lượng mức che; quy định box và vật bị loại; ghi câu hỏi còn mở. | Các tình huống 222 ghi nhận trong mục 4. Chưa phải thống nhất nhóm. |

V1.0 sẽ được lưu thành file nhóm riêng sau CP2. V1.1 sau calibration phải ghi rõ thay đổi câu chữ (`editorial`) hay đổi ý nghĩa (`semantic`); thay đổi ý nghĩa yêu cầu rà cả lô và có quyết định phạm vi `lo`. Không sửa ngưỡng lab để đạt QC.
