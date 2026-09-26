# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: class `traffic_light` (state, relevance) + tag `frame.ego_signal`; cây quyết định 5.1–5.4; rule 1/3; bảng frame | Chốt contract "đèn điều khiển ego ở vạch dừng kế tiếp" | `01_problem_statement.md`; khảo sát ảnh BDD02, BDD07, BDD11, BDD12, BDD26, LISA01/30 |
| v1 (đề xuất, chưa lên v2) | Mục 8: clip LISA label bằng Track (1 đầu đèn = 1 track, keyframe ở frame đổi state, frame chuyển tiếp không ô nào sáng → `unknown`, outside khi mất đèn); đèn xa trong clip box thành track `relevance=other`; thêm card EC14 | Label thử cả 30 frame LISA: state chỉ đổi ở 1 frame nên Shape từng frame lặp 30 lần cùng một quyết định; đèn xa có chu kỳ riêng (đổi xanh ở frame 15–16), IGNORE thì mất tín hiệu này | `project/10_lisa_tracking_nguyen/` (export CVAT task 21), card EC14 |
