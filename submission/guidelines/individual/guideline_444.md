# Guideline gán nhãn đối tượng 2D — Phiên bản cá nhân (v0) — Thành viên 444

> **Tài liệu đặc tả kỹ thuật cá nhân (Pha 1)** được xây dựng độc lập dựa trên việc rà soát toàn bộ 40 ảnh thực tế trong `data/images/`. Tài liệu này đóng vai trò là bản đề xuất ban đầu (v0 cá nhân) kèm các lập luận và gợi ý phương án thống nhất cho phiên họp nhóm Pha 2.

- **Phiên bản:** v0 (Bản cá nhân hoàn chỉnh)
- **Ngày lập:** 2026-11-16
- **Tác giả:** Thành viên 444 (Nhóm Group-01)
- **Tình trạng:** Khóa bản độc lập để nộp cho Coach trước phiên họp nhóm

---

## 1. Mục tiêu và Định vị bài toán

- **Bài toán ứng dụng:** Huấn luyện mô hình thị giác máy tính phát hiện đối tượng 2D (2D Object Detection) phục vụ:
  - Hệ thống hỗ trợ lái xe an toàn và tự hành (ADAS/Autonomous Driving).
  - Hệ thống camera thông minh điều hành giao thông đô thị (ITS).
- **Mục tiêu nhận diện cốt lõi:** Xác định tọa độ hình chữ nhật (bounding box) và phân loại các **thực thể tham gia giao thông thực tế (dynamic traffic participants)** có khả năng tự di chuyển hoặc trực tiếp chiếm dụng không gian vật lý lòng đường.
- **Mức độ rủi ro & Ưu tiên đánh giá:**
  - *Bỏ sót (False Negative - `bo_sot`):* Bỏ sót người đi bộ hoặc người đi xe máy là lỗi nguy hiểm nhất (nguy cơ va chạm chết người).
  - *Gán thừa (False Positive - `thua`):* Gán nhãn lên tượng, tranh ảnh billboard, ma-nơ-canh, bóng phản chiếu sẽ kích hoạt lỗi "phanh ảo" (phantom braking) làm dồn toa giao thông $\rightarrow$ Bị phạt lỗi `thua` theo chuẩn QC của lab.
  - *Sai lệch kích thước / lỏng box (`sai_vi_tri_kich_thuoc`):* Gây ước lượng sai khoảng cách tiếp xúc (Time-to-Collision) và cự ly an toàn.

---

## 2. Danh mục Lớp & Xử lý ngoại lệ (Taxonomy)

### 2.1. Danh mục 6 Lớp chuẩn

Mỗi đối tượng hợp lệ phải thuộc đúng **một lớp duy nhất**:

| Lớp | Định nghĩa chuẩn (1 câu) | Ví dụ thực tế | Phản ví dụ / Tuyệt đối không gán |
|---|---|---|---|
| `car` | Xe ô tô con từ 4 đến 9 chỗ ngồi dùng chở người (sedan, SUV, hatchback, crossover, wagon, coupe). | Xe sedan trong mưa (`img_015`), xe wagon (`img_024`), xe chở trên xe tải (`img_002`), xe diễu hành chế chạy trên phố (`img_019`). | Xe bán tải (`truck`), xe van chở hàng (`van`), tượng xe bia mộ (`img_027`), mô hình xe đồ chơi (`img_038`). |
| `truck` | Xe cơ giới chuyên dụng chở hàng có thùng hàng riêng biệt, xe đầu kéo, xe chuyên chở và xe bán tải (pickup). | Xe chở xe Renault 380 (`img_002`), xe bán tải pickup (`img_001`, `img_012`), bán tải có nắp thùng ngủ/thùng cao (`img_037`), xe tải nhẹ giao hàng (`img_007`, `img_036`). | Xe thùng kín thân liền một khối (`van`), ô tô con chở người (`car`), tượng xe bán tải bằng đá (`img_027`). |
| `van` | Xe ô tô thân kín một khối cỡ nhỏ hoặc vừa chở khách hoặc bưu kiện nhẹ (minivan, blind van). | Xe chuyển phát nhanh bưu điện thân liền, minivan chở khách gia đình. | Xe bán tải có nắp thùng cao composite (`truck`), SUV cỡ lớn (`car`), xe buýt cỡ lớn (`img_032`). |
| `motorcycle` | Phương tiện cơ giới hai bánh (xe máy, mô tô phân khối lớn, xe ga, scooter) và xe ba bánh có thùng bên (sidecar). | Xe máy đường phố TP.HCM (`img_005`), xe máy chợ đêm (`img_008`), mô tô độ (`img_011`), xe trong hầm (`img_026`), xe sidecar có thùng bên (`img_022`). | Xe đạp thô sơ, xe xích lô (`img_006`, `img_009`), xe tuk-tuk 3 bánh (`img_004`, `img_035`). |
| `person` | Con người còn sống ở mọi tư thế: đi bộ, đứng, ngồi, lái xe hoặc ngồi sau phương tiện hở. | Người phụ nữ đứng cạnh xe (`img_001`), người đi xe máy (`img_005`, `img_026`), người qua đường (`img_014`), người cosplay Predator (`img_011`). | Tượng điêu khắc đá/đồng (`img_017`), hình ảnh người in trên pano quảng cáo (`img_018`), ma-nơ-canh (`img_033`), búp bê đồ chơi (`img_038`). |
| `ignore_region` | Vùng bỏ qua: cụm đám đông hoặc phương tiện ở rất xa ($<20\text{px}$) hoặc vùng hỗn loạn không gán từng vật. | Cụm người ở hậu cảnh bờ biển (`img_025`), cụm xe dồn toa mờ nhạt ở cuối đường chân trời. | Bất kỳ vật thể đơn lẻ nào nhìn rõ có chiều cao $\ge 20\text{px}$. |

---

## 3. Hệ thống Quy tắc vẽ Bounding Box (Core Annotation Protocols)

Annotator tuân thủ nghiêm ngặt 10 quy tắc kỹ thuật sau:

* **`R-01` — Nguyên tắc Modal Bounding Box (Chỉ ôm pixel nhìn thấy):** Bounding box là hình chữ nhật nhỏ nhất bao quanh toàn bộ các điểm ảnh thực tế nhìn thấy được của vật thể. Tuyệt đối **không vẽ ước lượng (Amodal)** bao trùm phần bị vật cản khác che khuất.
* **`R-02` — Phân tách Người và Phương tiện hở (Rider & Motorcycle):**
  - Khi có người điều khiển xe máy (`motorcycle`): Bắt buộc vẽ **2 box riêng biệt**:
    1. Box `person`: Ôm trọn cơ thể người từ đỉnh nón bảo hiểm đến bàn chân đặt trên để chân.
    2. Box `motorcycle`: Ôm trọn chiếc xe từ bánh trước, bánh sau đến đuôi xe và gương chiếu hậu.
  - Xe chở 2 người (`img_026`): Vẽ **3 box độc lập** (2 box `person` cho 2 người và 1 box `motorcycle` cho xe).
* **`R-03` — Loại trừ Người trong Phương tiện kín (Cabin Exclusion):**
  - Người ngồi bên trong ô tô con (`car`), cabin xe tải (`truck`), hoặc xe thùng (`van`) nhìn qua kính chắn gió: **KHÔNG vẽ box `person`**. Chiếc xe được coi là một thực thể bao trùm toàn bộ không gian bên trong.
* **`R-04` — Xe máy có ghế/thùng bên hông (Sidecar — `img_022`):**
  - Gán **1 box `motorcycle` duy nhất** bao trọn toàn bộ cụm phương tiện (khung xe máy 2 bánh + thùng thuyền bên hông).
  - Người lái xe máy: 1 box `person`.
  - Người ngồi trong thùng thuyền bên: 1 box `person` riêng (đánh dấu `occluded_level=1` hoặc `2` vì bị thành thùng che nửa người).
* **`R-05` — Xe bán tải có chỗ ngủ tích hợp / Nắp thùng cao (`img_037`):**
  - Gán nhãn **`truck`**, box bao trọn cả cabin xe và phần nắp thùng ngủ/thùng cao composite tích hợp phía sau.
* **`R-06` — Xe chở xe chuyên dụng (Car Carrier — `img_002`):**
  - Vẽ 1 box `truck` bao toàn bộ chiếc xe tải chuyên dụng (kể cả khung sàn rơ-moóc chở xe).
  - Vẽ từng box `car` riêng cho từng chiếc ô tô con đang nằm trên sàn xe nếu chiều cao $\ge 20\text{px}$ (đánh dấu `occluded_level=1` hoặc `2` do bị khung sắt che).
* **`R-07` — Hàng hóa trên phương tiện & Hàng quá khổ (`img_012`, `img_029`):**
  - Hàng hóa nằm trong lòng thùng xe tải/bán tải (`img_012`): Box `truck` bao trọn cả phần hàng hóa trong thùng.
  - Xe máy chở hàng cồng kềnh dài hơn xe (`img_029`): Box `motorcycle` **chỉ bao trọn phần khung xe máy thực tế** nhìn thấy; phần hàng hóa cồng kềnh nhô ra ngoài **không tính vào box xe**, đồng thời bắt buộc tick `can_review=true`.
* **`R-08` — Bài trừ Thực thể phi thực & Phản chiếu ảo (Tránh lỗi `thua`):**
  - **TUYỆT ĐỐI KHÔNG GÁN NHÃN** cho: Tượng điêu khắc đá/đồng (`img_017`, `img_027`), hình ảnh người in trên biển quảng cáo/billboard (`img_018`), ma-nơ-canh trước cửa hàng (`img_033`), mô hình đồ chơi/búp bê trong tủ kính (`img_038`), và hình ảnh phản chiếu qua gương chiếu hậu (`img_020`).
  - **Ngoại lệ:** Người mặc trang phục hóa trang (Cosplay Predator — `img_011`): **VẪN GÁN `person`** vì đây là người thật đang sống tham gia hoạt động trên đường phố.
* **`R-09` — Xe quá mờ do mưa, tốc độ, thiếu sáng (`img_013`, `img_015`):**
  - Nếu mắt người vẫn nhận diện được hình bóng (silhouette) và xác định được ranh giới 4 cạnh tương đối: **Vẫn vẽ box**, đánh dấu `occluded_level=3` và tick `can_review=true`.
  - Nếu xe quá mờ, vỡ nét hoàn toàn không thể xác định biên giới: **Không vẽ box đơn lẻ**. Nếu là một vùng nhiều xe mờ ở xa: khoanh vùng gán lớp `ignore_region`.
* **`R-10` — Đám đông xe máy & Ngưỡng kích thước nhỏ (`img_005`, `img_008`, `img_030`):**
  - Tiền cảnh & Trung cảnh (chiều cao $\ge 20\text{px}$): Kiên trì vẽ tách từng cặp box (1 `person` + 1 `motorcycle`). Box chồng lấn nhau là bình thường, đánh dấu `occluded_level=1` hoặc `2`.
  - Hậu cảnh xa (chiều cao $< 20\text{px}$ hoặc dồn cục không rõ biên): Vẽ box bao quanh toàn cụm và gán lớp `ignore_region`.
  - Vật đơn lẻ $< 20\text{px}$: Không vẽ box lẻ.

---

## 4. Bảng tra cứu toàn bộ các ca khó từ 40 ảnh & Gợi ý thống nhất cho nhóm

| ID Ảnh | Tình huống thực tế | Rule ID | Quyết định cá nhân của 444 | Gợi ý thống nhất cho phiên họp nhóm (Pha 2) |
|---|---|---|---|---|
| `img_001` | Người phụ nữ đứng cạnh xe bán tải pickup | `R-01`, `R-02` | 1 `person` cho người phụ nữ; 1 `truck` cho xe bán tải. | Thống nhất xe bán tải là `truck` (có thùng hàng riêng). |
| `img_002` | Xe chuyên dụng chở 6 ô tô con trên sàn | `R-06` | 1 `truck` cho xe tải; từng box `car` riêng cho xe con trên sàn $\ge 20\text{px}$. | Thống nhất gán `car` cho xe trên sàn để detector học đặc trưng visual ô tô; đánh dấu `occluded_level=1/2`. |
| `img_003` | Người bán mít đẩy xe đạp chở hàng | `R-02` | 1 `person` cho người bán; xe đạp thô sơ không gán box. | Xe đạp ngoài taxonomy cơ giới, chỉ gán người để tránh false positive phương tiện. |
| `img_004` | Hai xe Tuk-tuk đỗ trên vỉa hè | `R-TUKTUK` | Tạm gán `car`, tick `can_review=true`. | Đề xuất nhóm tạm quy ước `car` ở v1.0 vì có cabin che chở; nếu bất đồng thì đưa vào `ignore_region` hoặc thêm lớp ở v1.1. |
| `img_005` | Đám đông xe máy dày đặc tại TP.HCM | `R-10`, `R-02` | Tiền cảnh vẽ cặp 1 `person` + 1 `motorcycle`. Hậu cảnh xa $< 20\text{px}$ phủ `ignore_region`. | Thống nhất ngưỡng cắt 20px để cả nhóm có cùng ranh giới, tránh lệch độ phủ. |
| `img_006` | Xích lô chở du khách tại Huế | `R-CYCLO` | Chỉ gán `person` cho người đạp và du khách; xe xích lô không gán box. | Thống nhất xe xích lô không phải phương tiện cơ giới, không gán box xe. |
| `img_007` | Xe tải giao hàng trong kho tối | `R-09` | Gán `truck` cho các xe tải nhìn rõ; vùng quá tối không rõ biên thì bỏ qua. | Đánh dấu `occluded_level` phù hợp với độ che phủ ánh sáng. |
| `img_008` | Chợ đêm có người đi bộ và xe máy đỗ | `R-02`, `R-10` | Xe máy đỗ không người: 1 `motorcycle`. Người đi bộ: 1 `person`. Xe có người lái: tách 2 box. | Thống nhất xe máy đỗ tĩnh vẫn là `motorcycle`. |
| `img_009` | Đoàn xích lô du lịch | `R-CYCLO` | Chỉ gán `person` cho tất cả người ngồi/đạp xích lô. | Thống nhất không tạo nhãn phương tiện cho xích lô. |
| `img_010` | Người bán phật thủ bằng xe đạp | `R-02` | 1 `person` cho người bán; xe đạp và thúng hoa quả không gán box xe. | Đồng nhất với ca `img_003`. |
| `img_011` | Cosplay Predator đứng cạnh mô tô độ | `R-08`, `R-02` | 1 `person` (người cosplay), 1 `motorcycle` (xe mô tô độ). | Giải thích cho nhóm: Predator là người sống thật đang mặc trang phục, bắt buộc phải bảo vệ trong giao thông. |
| `img_012` | Bán tải chở đầy rau củ trên thùng | `R-07` | 1 `truck` bao gồm cả thùng xe và khối rau củ trên thùng. | Hàng trong thùng xe tính là một phần thể tích phương tiện. |
| `img_013` | Phố đêm mưa ướt, ánh đèn phản chiếu | `R-09` | Gán các xe nhìn rõ ranh giới; bóng phản chiếu dưới mặt đường ướt KHÔNG gán. | Thống nhất không bao giờ gán vệt phản chiếu trên mặt đường ướt. |
| `img_014` | Người đi bộ băng qua đường giữa dòng xe máy | `R-01`, `R-02` | Gán độc lập từng `person` đi bộ và từng cặp người/xe máy; áp dụng modal box ôm phần nhìn thấy. | Thống nhất không vẽ đoán chân người khi bị đầu xe máy che. |
| `img_015` | Xe ô tô chạy trong mưa lớn qua kính gạt nước | `R-09` | 1 `car`, đánh dấu `occluded_level=3` và tick `can_review=true`. | Mưa nhòe nhưng vẫn thấy rõ bóng dáng sedan; gán để xe tự lái không bị bỏ sót vật cản trong mưa. |
| `img_016` | Giao thông Bangkok giờ cao điểm | `R-01` | Gán xe con, xe máy, xe tải theo quy tắc chuẩn. | Nhóm thống nhất ranh giới phân loại. |
| `img_017` | Tượng kỵ sĩ Duke of Wellington | `R-08` | **TUYỆT ĐỐI KHÔNG GÁN** `person` hay phương tiện. | Cảnh báo nhóm: gán tượng là lỗi `thua` theo cheat-sheet QC, bị trừ điểm nặng. |
| `img_018` | Pano quảng cáo in ảnh chân dung người | `R-08` | **TUYỆT ĐỐI KHÔNG GÁN** `person`. | Ảnh in 2D không tham gia giao thông, gán là lỗi `thua`. |
| `img_019` | Xe hình chuột hoạt hình Dangermouse | `R-01` | Gán `car` (vì là xe tự chế hoạt động thật trên đường phố). | Dù hình thù kỳ dị nhưng xe có động cơ, bánh xe và người lái lưu thông trên đường. |
| `img_020` | Cầu vồng và xe trong gương chiếu hậu | `R-08` | **TUYỆT ĐỐI KHÔNG GÁN** bất kỳ box nào trong mặt gương. | Ảnh ảo trong gương không phải chướng ngại vật phía trước, gán sẽ gây phanh ảo. |
| `img_022` | Xe máy có thùng bên (Sidecar) | `R-04` | 1 `motorcycle` bao toàn bộ xe máy và thùng phụ; 2 `person` cho người lái và người ngồi thùng. | Thống nhất sidecar là 1 phương tiện duy nhất, không tách rời thùng phụ. |
| `img_024` | Xe AMC Pacer nhìn qua cửa sổ | `R-01` | 1 `car` ôm phần thân xe nhìn thấy. | Áp dụng modal bbox. |
| `img_025` | Người đi dạo bãi biển ở hậu cảnh xa | `R-10` | Người ở xa $< 20\text{px}$ gom thành các cụm `ignore_region`. | Thống nhất không vẽ lắt nhắt từng chấm nhỏ ở hậu cảnh. |
| `img_026` | Cặp đôi đi chung xe máy trong hầm | `R-02` | Vẽ 3 box: 2 box `person` (người lái + người ngồi sau) và 1 box `motorcycle`. | Giải thích cho nhóm: mỗi người là một mục tiêu bảo vệ độc lập. |
| `img_027` | Bia mộ đá hoa cương hình xe bán tải | `R-08` | **TUYỆT ĐỐI KHÔNG GÁN** `truck` hay `car`. | Tượng đá bia mộ trong nghĩa trang không phải xe, gán là lỗi `thua`. |
| `img_028` | Xe bán tải chở tượng ngựa gỗ trên thùng | `R-07` | 1 `truck` bao trọn cả xe và tượng ngựa gỗ trên thùng. | Tượng ngựa gỗ là hàng hóa chở trên xe, tính vào box xe tải. |
| `img_029` | Xe máy chở hàng cồng kềnh dài hơn xe | `R-07` | 1 `person`, 1 `motorcycle` ôm thân xe máy thật; hàng nhô ra ngoài không tính vào xe, tick `can_review=true`. | Thống nhất không làm méo mó aspect ratio của xe máy. |
| `img_030` | Đoàn xe máy dừng đèn đỏ | `R-02` | Tách độc lập từng box người và xe máy. | Đồng nhất với `img_005`. |
| `img_032` | Xe buýt công cộng tại trạm dừng | `R-BUS` | Tạm gán `truck` (xe thương mại hạng nặng), tick `can_review=true`. | Đề xuất nhóm tạm xếp vào `truck` ở v1.0 để giữ tương thích, sau đó thảo luận thêm lớp `bus` ở v1.1. |
| `img_033` | Ma-nơ-canh trước cửa hàng | `R-08` | **TUYỆT ĐỐI KHÔNG GÁN** `person`. | Ma-nơ-canh vô tri, gán sẽ gây phanh ảo và bị phạt lỗi `thua`. |
| `img_035` | Xe Auto Rickshaw chở đầy trẻ em | `R-TUKTUK`, `R-02` | Gán `car` cho rickshaw (tick `can_review`); các trẻ em thò người hẳn ra ngoài gán `person` riêng. | Hành khách thò người ra ngoài có nguy cơ va chạm trực tiếp cần bảo vệ. |
| `img_036` | Xe tải nhẹ CMC Veryca thùng lửng | `R-01` | Gán `truck`. | Xe tải nhẹ có thùng hàng riêng biệt thuộc `truck`. |
| `img_037` | Bán tải có nắp thùng ngủ/thùng cao composite | `R-05` | Gán `truck`, box bao trọn cả cabin và phần nắp thùng cao. | Thùng cao là phụ kiện gắn liền trên khung gầm bán tải, vẫn thuộc `truck`. |
| `img_038` | Tủ kính cửa hàng bày mô hình xe và búp bê | `R-08` | **TUYỆT ĐỐI KHÔNG GÁN** `car` hay `person`. | Đồ chơi trong nhà không tham gia giao thông, gán là lỗi `thua`. |
| `img_040` | Giao thông hỗn hợp Bangkok (ô tô, tuk-tuk, xe máy) | `R-01`, `R-TUKTUK` | Gán xe con, xe máy, tuk-tuk theo đúng quy ước đã lập. | Thống nhất áp dụng đồng bộ bộ quy tắc cho các ảnh giao thông phức tạp. |

---

## 5. Định lượng Thuộc tính (Attributes Specification)

* **`occluded_level` (Mức độ che khuất):**
  * `0`: Không che khuất (diện tích bị che $< 10\%$).
  * `1`: Bị che khuất nhẹ ($10\% \le \text{che} \le 35\%$).
  * `2`: Bị che khuất đáng kể ($35\% < \text{che} \le 80\%$).
  * `3`: Bị che khuất nghiêm trọng ($> 80\%$, chỉ còn thấy một phần nhỏ nhận diện được qua ngữ cảnh).
* **`truncated` (Cắt mép ảnh):**
  * `yes`: Khi viền ngoài của vật thể bị cắt ngang bởi ít nhất một cạnh ảnh ($x=0, y=0, x=W, y=H$).
  * `no`: Vật thể nằm trọn vẹn bên trong khung ảnh.
* **`can_review`:** Bắt buộc tick `true` với tất cả các ca ngoài taxonomy (Tuk-tuk, Xe buýt), hàng cồng kềnh quá khổ, xe quá mờ trong mưa, hoặc ranh giới phân vân.

---

## 6. Lịch sử phiên bản
* **v0 (2026-11-16):** Bản khởi tạo cá nhân hoàn chỉnh của thành viên 444, bao phủ toàn bộ các ca kinh điển và bảng tra cứu chi tiết từ 40 ảnh thực tế, sẵn sàng cho phiên họp nhóm Pha 2.
