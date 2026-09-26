# Edge-case library

Kho nội bộ của nhóm, **không gửi cho peer**. Rule và ví dụ từ card dùng ảnh example/calibration đã được chép sang
`02_guideline.md` (mục 5, 7, 9). Card dùng ảnh blind (EC09–EC12) chỉ nằm ở đây; decision tương ứng có trong
`gold_decisions.csv`.

Card nào ghi **"dự kiến"** thì phải chốt lại sau calibration (120–140') và cập nhật Expected theo consensus.

---

CASE ID: EC01
Sample: BDD02
Scene: Ngã tư đô thị NYC, ban ngày, nhiều đầu đèn gắn trên cột và cần vươn
Observation: 4 đầu đèn xanh quay mặt về ego. Có 2 vỏ vàng nhìn ngang (thấy mép mái che hình răng lược), hộp đèn đi bộ trắng/vàng trên cột phải, và đèn tí hon ở giao lộ sau (~582,245)
Decision: LABEL 4 đầu đèn; IGNORE phần còn lại
Expected: traffic_light [relevance=ego;state=green]×4 · frame [ego_signal=visible]
Rationale: Planner chỉ cần đèn ego. Vỏ quay ngang không cho biết màu với ego; đèn giao lộ sau gây nhiễu (contract Q2)
Common mistake: Box cả vỏ quay ngang (đếm 6 thay vì 4); box đèn tí hon ở giao lộ sau
Diversity: conflict (nhiều nguồn) · small_far

---

CASE ID: EC02
Sample: LISA01
Scene: Giao lộ California lúc chạng vạng; cần vươn mang đầu đèn mũi tên trái và đầu đèn tròn
Observation: Mũi tên trái sáng ở ô trên (đỏ-cam), đèn tròn sáng ô trên, đầu đèn phụ bên phải cũng sáng ô trên. Có 3 đèn thấp ở xa
Decision: LABEL 3 đầu đèn trên cao; IGNORE 3 đèn xa (< 1/3)
Expected: đèn tròn giữa và phải: [relevance=ego;state=red]; mũi tên: [relevance=other;state=red] · frame visible
Rationale: Ego mặc định đi thẳng, nên mũi tên phục vụ làn rẽ. Lúc chạng vạng màu đỏ trông như cam; xác định màu theo vị trí ô sáng
Common mistake: Chọn state=yellow vì màu trông cam; gán mũi tên relevance=ego
Diversity: conflict · low_visibility · ambiguity semantics (mũi tên)

---

CASE ID: EC03
Sample: BDD11
Scene: Ngã tư khu dân cư, ban ngày
Observation: Chỉ có đèn đi bộ bàn tay cam ở góc phải (~1210,260); không thấy đầu đèn xe nào quay về ego
Decision: IGNORE đèn đi bộ; frame = out_of_view
Expected: 0 box traffic_light · frame [ego_signal=out_of_view]
Rationale: Box đèn đi bộ thành đèn ego đỏ gây phantom braking. out_of_view báo cho planner rằng giao lộ có đèn nhưng không thấy đèn ego
Common mistake: Box bàn tay cam thành traffic_light [state=red;relevance=ego]; chọn none thay vì out_of_view
Diversity: **critical-risk** · negative

---

CASE ID: EC04
Sample: BDD18
Scene: Ban đêm, đường đô thị
Observation: Đèn đi bộ bên phải (bàn tay cam + người trắng). Vài đốm xanh nhỏ ở xa (~585,305; ~660,305). Không có đầu đèn lớn nào; không định vị được giao lộ đầu tiên
Decision: LABEL đốm xanh (nhánh "chỉ có đèn xa" của 5.3) với relevance=unknown → ESCALATE (dự kiến)
Expected: traffic_light [relevance=unknown;state=green]×2 · frame [ego_signal=escalate] (dự kiến, chốt sau calibration)
Rationale: Không đủ bằng chứng để khẳng định đèn xanh xa điều khiển ego. Đoán "ego green" là rủi ro chạy qua giao lộ khi chưa rõ tín hiệu
Common mistake: Gán ego green; hoặc box đèn đi bộ; hoặc chọn out_of_view và bỏ đèn xanh
Diversity: **escalation** · low_visibility · small_far

---

CASE ID: EC05
Sample: BDD25
Scene: Đại lộ Midtown lúc chạng vạng, đường ướt, đèn xanh ở nhiều giao lộ liên tiếp
Observation: Đầu đèn xanh ở bên phải (~830,242; ~865,262) và ở giữa xa (~580,255); tất cả đều nhỏ. Đường ướt phản chiếu đèn
Decision: LABEL đèn ở giao lộ gần nhất; đèn xa hơn IGNORE hoặc unknown theo rule 1/3 (dự kiến)
Expected: (dự kiến) 2 đầu đèn bên phải [relevance=ego;state=green]; đèn giữa xa IGNORE nếu < 1/3 · frame visible. Không box phản chiếu trên mặt đường
Rationale: Contract yêu cầu đèn ở vạch dừng kế tiếp, không phải đèn ở các block sau
Common mistake: Box tất cả đèn xanh nhìn thấy và gán ego; box phản chiếu trên mặt đường ướt
Diversity: ambiguity · small_far · low_visibility

---

CASE ID: EC06
Sample: BDD17
Scene: Mưa, giọt nước trên kính chắn gió, phố NYC
Observation: Đốm đỏ-cam và xanh bị nhoè ở bên trái và ở giữa; khó phân biệt đèn xe với đèn đi bộ
Decision: LABEL chỉ khi nhận ra hình dạng đầu đèn và vị trí ô sáng; nhoè quá thì IGNORE; màu không chắc thì state=unknown (dự kiến)
Expected: (dự kiến) đầu đèn xanh ở giữa [relevance=ego;state=green]; đốm cam trái [state=unknown] hoặc IGNORE — chốt sau calibration
Rationale: Giọt nước làm biến dạng ô đèn; đoán màu sẽ tạo nhãn nhiễu cho classifier
Common mistake: Box mọi đốm màu; gán red cho đốm cam của đèn đi bộ
Diversity: occlusion · low_visibility

---

CASE ID: EC07
Sample: LISA30
Scene: Cùng giao lộ LISA01, 29 frame sau
Observation: Hai đèn tròn đã chuyển xanh; mũi tên trái vẫn đỏ
Decision: LABEL; frame = visible (không escalate)
Expected: đèn tròn [relevance=ego;state=green]×2; mũi tên [relevance=other;state=red] · frame visible
Rationale: Mũi tên là other, nên không tính vào điều kiện "≥ 2 đèn ego khác màu" của bảng frame
Common mistake: Thấy đỏ và xanh cùng lúc rồi chọn escalate; gán mũi tên ego
Diversity: conflict · temporal

---

CASE ID: EC08
Sample: BDD13
Scene: Phố ban ngày, sát mép phải có hộp đèn đi bộ nhìn nghiêng
Observation: Không có đầu đèn xe nào; chỉ thấy hông hộp đèn đi bộ ở mép ảnh
Decision: IGNORE; frame = out_of_view (dự kiến)
Expected: 0 box · frame [ego_signal=out_of_view]
Rationale: Có phần cứng tín hiệu, tức giao lộ có đèn, nhưng không thấy đèn ego
Common mistake: Chọn none; box hộp đèn đi bộ
Diversity: negative · ambiguity (out_of_view vs none)

---

CASE ID: EC09
Sample: BDD26 (blind)
Scene: Ban đêm, đại lộ có dải phân cách
Observation: Đầu đèn gần trên cột dải phân cách trái sáng xanh (~487,71). Ngay dưới là đốm cam hình bàn tay (~437,118). Đèn đỏ/vàng tí hon gần điểm tụ (~632,222; ~667,217)
Decision: LABEL đèn gần; IGNORE đèn đi bộ và đèn xa
Expected: traffic_light [relevance=ego;state=green]; không có box ego nào mang red/yellow · frame visible
Rationale: Critical: nếu lấy đèn đỏ xa hoặc đèn đi bộ làm đèn ego, planner sẽ phanh sai trong khi ego đang có đèn xanh
Common mistake: Box đèn đỏ xa với relevance=ego; box bàn tay cam thành red
Diversity: **critical** · low_visibility · small_far

---

CASE ID: EC10
Sample: BDD12 (blind)
Scene: Ngã tư Queens cạnh trạm xăng, trời âm u
Observation: Đèn đi bộ bàn tay đỏ-cam rất rõ (~1090,140). Cạnh đó là vỏ đèn nhìn ngang/lưng. Không có đầu đèn xe nào quay về ego
Decision: IGNORE tất cả; frame = out_of_view
Expected: 0 box traffic_light · frame [ego_signal=out_of_view]
Rationale: Critical: bàn tay đỏ là tín hiệu nổi bật nhất ảnh, rất dễ bị box thành đèn ego đỏ
Common mistake: Box bàn tay thành traffic_light [state=red;relevance=ego]; chọn frame visible hoặc none
Diversity: **critical** · negative

---

CASE ID: EC11
Sample: BDD21 (blind)
Scene: Parkway ban ngày; bên phải là hàng rào và đường song song
Observation: 1 đầu đèn xanh trên cần vươn (~336,295). Đèn xanh tí hon bên trái (~226,320). Hai khối vàng-cam bên trái (~57/75,345). Vật đỏ sau tán cây bên kia hàng rào (~1150–1210,313)
Decision: LABEL đèn cần vươn; IGNORE đèn tí hon và khối vàng-cam; vật đỏ không được là ego
Expected: traffic_light [relevance=ego;state=green] ×1 duy nhất ở relevance=ego · frame visible
Rationale: Đèn thuộc đường khác (sau hàng rào) là other. Nếu nhầm thì tạo xung đột giả, dẫn tới escalate hoặc phanh sai
Common mistake: Gán vật đỏ bên phải relevance=ego rồi escalate; box khối vàng-cam
Diversity: **conflict** · small_far · occlusion

---

CASE ID: EC12
Sample: BDD07 (blind)
Scene: Brooklyn, ban ngày; capo xe bóng phản chiếu cả cảnh
Observation: Mỗi cần vươn mang 1 đầu đèn quay về ego (xanh) và 1 vỏ quay ngang. Hộp đèn đi bộ bị nhìn nghiêng. Bóng đèn phản chiếu trên capo (~245,625; ~695,600)
Decision: LABEL 2 đầu đèn; IGNORE vỏ quay ngang, đèn đi bộ và phản chiếu
Expected: traffic_light [relevance=ego;state=green]×2 · frame visible · geometry đầu đèn trái ≈ x230–240 y126–158
Rationale: Phản chiếu và vỏ quay ngang tạo false positive cho detector
Common mistake: Box ôm cả 2 vỏ (trước + ngang) vào một box; box phản chiếu
Diversity: normal · reflection

---

CASE ID: EC13
Sample: BDD09
Scene: Đường cao tốc ban ngày
Observation: Mặt lưng màu đen của một biển báo trông giống vỏ đèn; không có ô đèn
Decision: IGNORE; frame none
Expected: 0 box · frame [ego_signal=none]
Rationale: Không thấy mặt ô đèn (5.2); không có phần cứng tín hiệu thật
Common mistake: Box mặt lưng biển báo với state=unknown
Diversity: negative

---

CASE ID: EC14
Sample: LISA01–LISA30 (cả clip, label bằng Track)
Scene: Cùng giao lộ LISA01/LISA30, camera đứng yên, chạng vạng; 3 đầu đèn gần + 3 đầu đèn nhỏ ở giao lộ phía xa
Observation: Hai đèn tròn gần đổi đỏ → xanh ở frame 15 (LISA16), không qua vàng. Mũi tên trái đỏ suốt clip. Đèn xa cột
trái: frame 15 cả ô đỏ lẫn xanh đều tắt, frame 16 mới xanh. Hai đèn xa giữa: một đèn đổi xanh ở frame 15, một đèn đỏ mờ
suốt clip
Decision: 6 track; relevance cố định theo track; state đổi theo keyframe
Expected: track mũi tên [other; red 0–29] · 2 track đèn tròn gần [ego; red 0–14, green 15–29] · track đèn xa trái
[other; red 0–14, unknown 15, green 16–29] · track đèn xa giữa-trái [other; red 0–14, green 15–29] · track đèn xa
giữa-phải [other; red 0–29] · frame [ego_signal=visible] ở mọi frame
Rationale: Mục 8 (đề xuất): 1 đầu đèn = 1 track; frame chuyển tiếp không ô nào sáng → unknown; đèn xa trong clip là
other chứ không IGNORE
Common mistake: Tách một đèn thành hai track ở frame đổi màu; kéo màu xanh của frame 16 ngược về frame 15; gán đèn xa
đang xanh làm ego
Diversity: temporal · small_far · low_visibility · conflict

---
