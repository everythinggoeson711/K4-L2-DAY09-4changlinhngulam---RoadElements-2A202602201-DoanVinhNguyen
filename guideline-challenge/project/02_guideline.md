# Annotation guideline — Đèn điều khiển ego: trạng thái + relevance tại giao lộ nhiều đầu đèn

**Version:** v1

<!--
v1 = bản nháp đầu. Sau calibration nâng lên v2 (freeze). Sau blind handoff nâng lên v3.
Mỗi lần tăng version ghi một dòng vào 08_revision_log.md.
Ví dụ trong file này chỉ dùng ảnh thuộc split example/calibration.
-->

## 1. Objective + scope

Mục đích: planner cần biết **màu của đèn điều khiển làn xe mình (ego) ở vạch dừng kế tiếp**. Annotator cần:

1. box mỗi **đầu đèn xe cơ giới quay mặt về ego** ở giao lộ đầu tiên phía trước;
2. với mỗi box, ghi đèn đang sáng màu gì (`state`) và đèn có điều khiển ego hay không (`relevance`);
3. gắn 1 tag `frame` cho cả ảnh, tóm tắt kết luận.

**Trong scope:** đầu đèn tín hiệu cho xe (ô tròn hoặc mũi tên), gắn trên cột, cần vươn (mast arm) hoặc dây treo, thấy được
**mặt ô đèn**.

**Ngoài scope (không box):**
- đèn đi bộ;
- vỏ đèn quay ngang hoặc quay lưng;
- đầu đèn quá nhỏ ở xa (xem 5.3);
- đèn đường sắt, tram, bus-only; đèn vàng nhấp nháy cảnh báo trường học;
- đèn xe, đèn đường, biển hiệu, biển báo;
- bóng phản chiếu.

## 2. Annotation unit

- **Đơn vị:** 1 ảnh tĩnh (BDD), dùng Shape. Clip LISA 30 frame: dùng Track theo mục 8 (đề xuất, chờ nhóm chốt ở v2).
- **1 instance = 1 đầu đèn (signal head)**, tức một vỏ chứa các ô đèn xếp dọc hoặc ngang.
  - Hai vỏ gắn cạnh nhau trên cùng giá là **2 instance**, kể cả khi một vỏ quay ngang và không được box.
  - Đầu đèn 5 ô (hình chữ L hoặc "doghouse") là **1 instance**.
- Mỗi ảnh có **đúng 1 tag `frame`**, kể cả ảnh không có đèn nào.

## 3. Geometry rule

- Dùng tool **Rectangle**. Box ôm sát **phần vỏ đầu đèn nhìn thấy được** (visible extent, không amodal):
  - **Tính** mái che (visor) nhìn từ phía trước.
  - **Không tính** tấm viền nền (backplate), giá hoặc cần treo, cột, quầng sáng (halo).
- **Bị che một phần** (lá cây, cột, xe): chỉ box phần vỏ nhìn thấy.
- **Bị cắt ở mép ảnh:** box đến sát mép ảnh.
- **Ban đêm không thấy vỏ:** box lõi sáng của ô đèn đang bật. Lấy phần sáng đặc, bỏ quầng loe ra xung quanh.
- **Tolerance:** mỗi cạnh lệch tối đa **max(2 px, 15% chiều cao box)** so với mép vỏ. Zoom ≥ 400% khi kéo cạnh.

## 4. Taxonomy

Bảng đầy đủ ở `03_ontology_and_cvat_setup.md`; hai nơi phải khớp nhau.

| Label (CVAT) | Kiểu | Attribute | Giá trị | Ghi chú |
|---|---|---|---|---|
| `traffic_light` | rectangle | `state` | `red`, `yellow`, `green`, `unknown` | Default `__undefined__`, **bắt buộc chọn** |
| | | `relevance` | `ego`, `other`, `unknown` | Default `__undefined__`, **bắt buộc chọn** |
| `frame` | tag (cả ảnh) | `ego_signal` | `visible`, `out_of_view`, `none`, `escalate` | Default `__undefined__`, **bắt buộc chọn** |

**`state`** là màu của ô đang sáng.
- Mũi tên được ghi theo màu của nó; ví dụ mũi tên đỏ là `red`.
- **Dùng vị trí ô sáng để xác nhận màu:** đèn dọc thì đỏ trên, vàng giữa, xanh dưới; đèn ngang thì đỏ trái, xanh phải.
  Ban đêm hoặc chạng vạng, đỏ thường trông như cam và xanh trông như trắng-xanh ngọc. Khi đó vị trí ô sáng quyết định.
- `unknown` dùng khi:
  - không ô nào trông sáng (ảnh chụp đúng lúc LED nhấp nháy), hoặc
  - ô sáng bị che, loá, mờ mưa, không xác định được màu **và** vị trí.
- Không có giá trị `off`: ảnh tĩnh không phân biệt được đèn tắt thật với LED nhấp nháy.

**`relevance`**
- `ego`: đèn này điều khiển làn/hướng đi của ego tại giao lộ đầu tiên (rule 5.4).
- `other`: đèn quay về phía ego nhưng điều khiển thứ khác, ví dụ mũi tên cho làn rẽ mà ego không đi, hoặc đường song
  song bị ngăn bởi dải phân cách cứng/hàng rào.
- `unknown`: đã áp dụng rule 5.4 mà vẫn không quyết được.

**`frame.ego_signal`** (quyết định theo thứ tự, dừng ở dòng đầu tiên đúng):

| # | Điều kiện | Giá trị |
|---|---|---|
| 1 | Có ≥ 2 box `relevance=ego` mang **màu khác nhau** (không tính `unknown`) | `escalate` |
| 2 | Có box `relevance=unknown` hoặc box `ego` với `state=unknown`, **và** không có box `ego` nào mang màu rõ | `escalate` |
| 3 | Có ≥ 1 box `relevance=ego` với `state` là red/yellow/green | `visible` |
| 4 | Không có box `ego`, nhưng thấy phần cứng tín hiệu của giao lộ phía trước: đèn đi bộ, vỏ đèn quay ngang/lưng, đầu đèn `other`, cần vươn mang đèn | `out_of_view` |
| 5 | Không thấy phần cứng tín hiệu nào | `none` |

## 5. Inclusion / exclusion — cây quyết định cho từng nguồn sáng/vỏ đèn

Đi lần lượt 5.1 → 5.4; dừng ngay khi có kết quả IGNORE.

**5.1 Có phải đầu đèn cho xe không?**
- **Đèn đi bộ → IGNORE.** Dấu hiệu nhận biết:
  - hộp vuông, hiện hình bàn tay hoặc người đi bộ, có thể có số đếm ngược;
  - bàn tay có màu **cam** (không phải đỏ tròn), người đi bộ màu **trắng**;
  - thường gắn **thấp hơn** trên cột, gần vỉa hè.
- Không phải đèn tín hiệu → IGNORE: đèn đuôi hoặc đèn pha xe, đèn đường, biển hiệu sáng, biển báo vàng.
- Bóng phản chiếu trên capo, kính hoặc mặt đường ướt → IGNORE.

**5.2 Có thấy mặt ô đèn không?**
- Thấy được ít nhất một ô đèn tròn/mũi tên (đang sáng hoặc tối) → tiếp tục.
- Chỉ thấy hông vỏ, mép các mái che xếp chồng (hình răng lược như chữ "E"), hoặc mặt lưng phẳng → **IGNORE**. Đầu đèn
  quay ngang phục vụ đường cắt ngang, ego không đọc được màu của nó.

**5.3 Có thuộc giao lộ đầu tiên không? (rule 1/3)**
- Tìm **đầu đèn lớn nhất đang quay mặt về ego** trong ảnh. Đầu đèn nào có chiều cao vỏ **< 1/3** chiều cao đầu đèn đó
  → **IGNORE**, vì là đèn xa hoặc của giao lộ sau.
- Ban đêm thì so kích thước lõi sáng thay cho vỏ.
- Nếu ảnh **chỉ có** đầu đèn nhỏ/xa (không có đầu đèn lớn nào) → không áp dụng rule 1/3; box chúng và đi tiếp 5.4.

**5.4 Relevance (đèn đã qua 5.1–5.3 thì đều được box):**
- `ego` khi **đủ 3 điều kiện**:
  - (a) mặt đèn quay về hướng ego đang tới;
  - (b) nằm trên hoặc cạnh lòng đường của ego, kể cả đầu đèn trên cột phía trái hay phía phải ngã tư;
  - (c) không phải mũi tên dành riêng cho làn rẽ mà ego không ở.

  **Mặc định ego đi thẳng trong làn hiện tại**, trừ khi thấy rõ mũi tên sơn trên làn của ego.
- `other` khi:
  - là mũi tên rẽ trái/phải trong khi ego đi thẳng; hoặc
  - đèn thuộc đường khác, bị ngăn bởi hàng rào hoặc dải phân cách cứng; hoặc
  - mặt đèn quay chéo rõ rệt sang đường cắt ngang nhưng vẫn thấy ô sáng.
- `unknown` khi không quyết được đèn thuộc giao lộ đầu tiên hay giao lộ sau. Ví dụ: ban đêm chỉ thấy vài đốm xanh xa, và
  không thấy vạch dừng hay vạch qua đường để định vị.

## 6. Visibility / occlusion

| Tình huống | Quyết định |
|---|---|
| Vỏ bị che một phần nhưng ô sáng còn thấy | LABEL, box phần nhìn thấy, `state` theo màu thấy được |
| Ô sáng bị che hoặc loá, không rõ màu lẫn vị trí | LABEL với `state=unknown` |
| Bị cắt mép ảnh, còn thấy ≥ 1 ô đèn | LABEL, box đến mép |
| Nhỏ/xa, dưới rule 1/3 | IGNORE |
| Ảnh chỉ có đèn nhỏ/xa | LABEL nếu thấy màu; `relevance` theo 5.4 (thường `unknown`) |
| Mưa, giọt nước trên kính làm nhoè | Nhận ra hình dạng đầu đèn và vị trí ô sáng thì LABEL; không thì IGNORE |
| Phản chiếu trên capo, kính, đường ướt | IGNORE |
| Ban đêm: đèn đường, đèn xe lẫn với đèn tín hiệu | Chỉ box nguồn sáng có dạng ô đèn tín hiệu (tròn, màu đỏ/vàng/xanh, nằm trên cột/cần) |

## 7. Ambiguity / escalation

| Quyết định | Khi nào | Thể hiện trong CVAT |
|---|---|---|
| **LABEL** | Qua đủ 5.1–5.3 | Box `traffic_light` và chọn đủ `state`, `relevance` |
| **IGNORE** | Trượt ở 5.1, 5.2 hoặc 5.3 | **Không vẽ box.** Tag `frame` vẫn phản ánh phần cứng tín hiệu đã thấy (mục 4) |
| **UNKNOWN** | Đã box nhưng không quyết được màu hoặc relevance | `state=unknown` và/hoặc `relevance=unknown` |
| **ESCALATE** | Rule 1 hoặc rule 2 của bảng `frame` | Tag `frame [ego_signal=escalate]` |

Nguyên tắc:
- **Không đoán.** Phân vân giữa `ego` và `other` quá 30 giây thì chọn `unknown`, rồi để bảng `frame` quyết có escalate
  hay không.
- **Không bao giờ** dùng đèn đi bộ hay đèn của giao lộ sau để "bù" khi không thấy đèn ego. Không thấy đèn ego thì chọn
  `out_of_view`.
- `unknown` và `escalate` là kết quả hợp lệ, không bị coi là lỗi. Lỗi là đoán sai.

## 8. Temporal rule

**Ảnh tĩnh (BDD):** không áp dụng. Mỗi ảnh label độc lập bằng Shape.

**Clip LISA 30 frame (đề xuất của Nguyên, chờ nhóm chốt ở v2):** label bằng **Track**, không dùng Shape.
- **1 đầu đèn vật lý = 1 track** suốt chuỗi frame. `relevance` cố định cho cả track (immutable); `state` đổi theo frame.
- **Keyframe** ở frame đầu, ở **mọi frame đổi `state`** (đặt state mới ngay tại frame đó) và ở frame cuối. Box lệch khỏi
  đèn thì kéo lại, CVAT tự tạo keyframe.
- **Frame chuyển tiếp:** nếu ở frame đó không ô nào sáng (ví dụ đỏ vừa tắt, xanh chưa lên) → `state=unknown` đúng một
  frame đó, **không** kéo màu của frame trước/sau sang. Dùng frame lân cận để lấy ngữ cảnh, không để bịa state.
- **Đèn ra khỏi khung hoặc bị che hẳn** trước frame cuối → bấm **O** (outside) ở frame đầu tiên mất đèn.
- **Đèn nhỏ/xa của giao lộ sau:** trong clip vẫn box thành track riêng với `relevance=other` (không dùng rule 1/3 để
  IGNORE), để model thấy được chu kỳ đèn phía xa nhưng không lấy nó làm tín hiệu cho ego. Rule 1/3 ở 5.3 vẫn giữ cho
  ảnh tĩnh BDD.
- Đèn chỉ thấy lõi sáng (vỏ lẫn vào nền cây lúc chạng vạng) → box lõi sáng như mục 3; lõi chuyển ô (đỏ trên → xanh
  dưới) thì kéo box theo ở frame đổi state.
- Tag `frame` vẫn gắn mỗi frame một cái. **Lưu ý export:** `CVAT for video 1.1` giữ track nhưng **bỏ mất tag `frame`**;
  `CVAT for images 1.1` giữ tag nhưng tách track thành box rời. Nộp cả hai file.

## 9. Examples

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD02 | Ngã tư NYC, ban ngày. Có 4 đầu đèn quay về ego đều sáng xanh: cột trái gần ~(262,110), cần vươn giữa ~(795,140), cột phải ~(1006,150), góc trên phải ~(1219,122). Bên cạnh là 2 vỏ vàng quay ngang (~245,95; ~1240,45), hộp trắng/vàng của đèn đi bộ trên cột phải, và đèn tí hon ở giao lộ sau ~(582,245) | `traffic_light [relevance=ego;state=green]` ×4. Tag `frame [ego_signal=visible]`. Không box vỏ quay ngang, đèn đi bộ, đèn tí hon | 5.2, 5.3, 5.4 |
| LISA01 | Chạng vạng. Cần vươn mang 1 đầu đèn **mũi tên trái** sáng đỏ ~(748,150) và 1 đầu đèn tròn sáng đỏ ~(940,165); cột phải có thêm đầu đèn tròn đỏ ~(1155,205). Ba đèn thấp ở xa ~(683,368), ~(817,407), ~(832,407) nhỏ hơn 1/3 | Đèn tròn giữa và đèn phải: `[relevance=ego;state=red]`. Mũi tên trái: `[relevance=other;state=red]`. Đèn xa: không box. Tag `visible` | 4 (vị trí ô sáng: đỏ ở trên), 5.3, 5.4c |
| BDD11 | Chỉ thấy đèn đi bộ **bàn tay cam** bên phải ~(1200,255); không có đầu đèn xe nào quay về ego | 0 box `traffic_light`. Tag `frame [ego_signal=out_of_view]` | 5.1, bảng `frame` dòng 4 |
| BDD04 | Phố, ban ngày, không có đèn tín hiệu hay đèn đi bộ | 0 box. Tag `frame [ego_signal=none]` | bảng `frame` dòng 5 |

<!-- v2: thêm 2–3 dòng ví dụ từ ảnh calibration (BDD18, BDD25, LISA30…) theo kết luận calibration. -->

## 10. Common mistakes

1. **Box đèn đi bộ bàn tay cam thành `traffic_light` đỏ.** Đây là lỗi nghiêm trọng nhất. Bàn tay/người thì IGNORE, dù
   nó đang sáng.
2. **Lấy màu của giao lộ sau.** Đèn gần xanh mà đèn xa đỏ thì đèn xa bị rule 1/3 loại, không làm đổi kết luận.
3. **Box vỏ đèn quay ngang/quay lưng.** Không thấy mặt ô đèn thì không box.
4. **Quên tag `frame` hoặc để `__undefined__`.** Mỗi ảnh đúng 1 tag, kể cả ảnh không có đèn.
5. **Mũi tên rẽ đỏ, đèn tròn xanh, rồi chọn `escalate`.** Mũi tên là `other` khi ego đi thẳng, nên kết luận là
   `visible` chứ không phải xung đột.
6. **Box ôm cả backplate, cần treo hoặc quầng sáng ban đêm.** Chỉ ôm vỏ, hoặc lõi sáng khi ban đêm.
7. **Box bóng phản chiếu trên capo hoặc mặt đường ướt.**
8. **Chọn `state=red` chỉ vì đốm sáng trông đỏ/cam lúc chạng vạng.** Kiểm lại vị trí ô sáng trên vỏ trước khi chọn.
9. **Để sót default `__undefined__`** ở `state` hoặc `relevance`. Trước khi Save, lọc các object còn attribute trống.
