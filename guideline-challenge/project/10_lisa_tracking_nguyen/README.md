# LISA tracking — phần của Đoàn Vĩnh Nguyên

Label cả clip LISA01–LISA30 (`dayClip5--01606` → `01635`) bằng **Track** theo `02_guideline.md` mục 8 (bản đề xuất).

| File | Nội dung |
|---|---|
| `nguyen_lisa_video.zip` | Export **CVAT for video 1.1**: 6 track, keyframe ở frame đổi state. Không có tag `frame` (CVAT bỏ tag khi export video) |
| `nguyen_lisa_images.zip` | Export **CVAT for images 1.1** của cùng job: 180 box (6/frame) + 30 tag `frame [ego_signal=visible]` |

- CVAT: v2.75.1 local · task `LAB09-DoanVinhNguyen` (id 21) · job 21 · 30 frame 1280×960
- Hai file đúng byte với export của CVAT, chỉ xoá email trong `<owner>`.

## 6 track

| Track | Box (xtl,ytl–xbr,ybr) | relevance | state |
|---|---|---|---|
| 0 Mũi tên trái trên cần vươn | 732,130–762,199 | other | red 0–29 |
| 1 Đèn tròn giữa | 926,140–956,209 | ego | red 0–14 · green 15–29 |
| 2 Đèn tròn phải (chỉ thấy lõi sáng) | 1148,197–1159,211 → 1147,232–1163,251 từ frame 15 | ego | red 0–14 · green 15–29 |
| 3 Đèn xa cột trái | 678,361–688,384 | other | red 0–14 · unknown 15 · green 16–29 |
| 4 Đèn xa giữa-trái | 810,401–823,425 | other | red 0–14 · green 15–29 |
| 5 Đèn xa giữa-phải | 829,401–841,427 | other | red 0–29 |

State được kiểm bằng màu pixel tại vị trí từng ô đèn trên cả 30 frame; camera đứng yên nên box không đổi vị trí.

## Cần nhóm quyết

1. Chấp nhận Track cho LISA (mục 8) và nâng guideline lên v2, hay giữ Shape như v1.
2. Đèn xa trong clip: `other` (đề xuất) hay IGNORE theo rule 1/3 như EC02 (LISA01) và EC07 (LISA30). Nếu nhận đề xuất,
   sửa Expected của EC02/EC07.
3. Không đưa hai file này vào `06_calibration_exports/`: `make calib` so theo bộ sample calibration trong
   `sample_pack.csv`, còn file này có 30 frame LISA nên sẽ bị báo "thừa sample".
