# RamanID — Nhận diện phổ Raman trên trình duyệt

[![Live](https://img.shields.io/badge/live-sisypheanhus.github.io%2Framan-2ea44f?logo=github)](https://sisypheanhus.github.io/raman/)
![Static](https://img.shields.io/badge/backend-none-blue)
![Model](https://img.shields.io/badge/model-v3%20Bayes%20%2B%20within--class%20PCA-8a2be2)
![Accuracy](https://img.shields.io/badge/LOO%20accuracy-90.1%25%20%7C%2095.9%25%20ho%C3%A1%20ch%E1%BA%A5t-orange)

Kéo thả một file phổ Raman (`.txt` / `.csv`) và nhận kết quả nhận diện
vật liệu kèm xác suất hậu nghiệm — **toàn bộ tính toán chạy trong trình
duyệt**, không có server, không upload dữ liệu.

> 🔗 **Demo:** <https://sisypheanhus.github.io/raman/>

## Repo này chứa gì

Đây là repo **deploy** (GitHub Pages) — chỉ gồm các file tĩnh đã được
sinh tự động từ mã nguồn nghiên cứu. Không sửa tay các file ở đây.

| File | Kích thước | Nội dung |
|------|-----------:|----------|
| `index.html` | ~66 KB | Ứng dụng chính. Nhúng sẵn toàn bộ thuật toán (`core.js`) — tiền xử lý, Bayes factor, phân loại, tìm peak, đa phương thức |
| `data.json` | 16.7 MB (~5 MB gzip) | Thư viện phổ tham chiếu + mô hình within-class, mã hoá base64 float32 |
| `catalog.html` | ~27 KB | Duyệt/xem toàn bộ phổ trong thư viện theo nguồn |
| `about.html` | ~23 KB | Mô tả phương pháp, độ chính xác, lịch sử phiên bản |

## Vì sao trang web tái lập chính xác thuật toán và thư viện

```
ramanlib.py + bayes.py + multimodal.py     library_ext.npz (16 024 phổ)
            │  port 1:1                              │  build_deploy.py
            ▼                                        ▼
         core.js  ──── inline ────▶  index.html      data.json
            │
            └── test_core.mjs: so sánh với kết quả Python (17/17 PASS)
```

| Thành phần | Cách đảm bảo giống Python |
|------------|---------------------------|
| **Thuật toán** | `core.js` là bản port thủ công của pipeline Python: nội suy lưới 450–3200 cm⁻¹ (bước 2), Savitzky–Golay 9/3, baseline ALS, chuẩn hoá L2, Bayesian evidence với null Legendre bậc 5, within-class PCA. Hằng số hiệu chuẩn (`KAPPA = 0.06`, `NULL_TAU = 0.659`, `LAMBDA = 0.01`) được đồng bộ từ `bayes.py` |
| **Kiểm chứng** | `test_core.mjs` chạy `core.js` dưới Node trên đúng file phổ và so log-BF, top-1, vị trí peak với output Python — phải PASS trước mỗi lần build |
| **Mô hình WC** | 516 mô hình within-class (mean + thành phần chính) được `build_deploy.py` học từ **toàn bộ 16 024 phổ** rồi encode `float32 → base64`. `classify()` ưu tiên `pairBfWc()` cho các lớp này → web dùng đúng lượng thông tin như Python |
| **Phổ raw** | Lớp chưa đủ điều kiện dựng WC (< 3 phổ, hoặc vùng phủ chung < 30 %) được so trực tiếp bằng `pairBF()` với phổ raw trong `data.json`. Decode lại thành `Float32Array` — không mất mát |
| **Thu gọn** | Chỉ RRUFF bị thu gọn (1 phổ/khoáng vật, chọn phổ phủ rộng nhất) để `data.json` ≈ 17 MB thay vì ≈ 50 MB. VAST, ROD, dung môi giữ **nguyên vẹn**; pharma đã nằm trọn trong WC |

Hệ quả: tập kiểm định VAST (209 phổ, 34 lớp) được tái lập **100 %** trên web
— 13 lớp có WC, 21 lớp còn lại giữ đủ phổ raw — nên độ chính xác công bố
bên dưới đúng với những gì trang web tính. Khác biệt duy nhất so với Python:
khoáng vật RRUFF không có WC (≈ 2 000 lớp) chỉ so với 1 phổ thay vì
best-of-N.

## Thư viện tham chiếu trên web

| Nguồn | Số phổ (web) | Ghi chú |
|-------|-------------:|---------|
| RRUFF | 2 284 | 1 phổ/khoáng vật (phủ rộng nhất); 11 154 phổ gốc đã học vào WC |
| ROD | 1 121 | Raman Open Database — giữ nguyên |
| VAST | 209 | Đo tại phòng thí nghiệm (532 nm) — giữ nguyên |
| Pharma API | 96 | 3 phổ/hợp chất hiển thị; 3 510 phổ gốc đã học vào WC |
| NICODOM demo | 18 | |
| Dung môi | 12 | |
| **Tổng** | **3 740** raw + **516** mô hình WC | Đại diện cho 16 024 phổ gốc |

Kèm 141 phổ huỳnh quang và 52 phổ hấp thụ cho chế độ đa phương thức (v3).

## Độ chính xác (leave-one-out)

| Phiên bản | Phương pháp | Tổng thể | Hoá chất |
|-----------|-------------|---------:|---------:|
| v1 | Cosine similarity | 79.7% | — |
| v2 | Bayes đơn mẫu | 83.7% | — |
| **v3** | Bayes + within-class PCA + đa phương thức | **90.1%** | **95.9%** |

## Build lại (từ repo nguồn)

```bash
cd raman_id/web
node test_core.mjs        # parity JS ↔ Python phải PASS
python build_deploy.py    # sinh deploy/{index,catalog,about}.html + data.json
cd deploy && git add -A && git commit -m "Rebuild" && git push
```

## Giấy phép dữ liệu

Phổ RRUFF, ROD và các bộ công khai khác giữ nguyên giấy phép của nguồn
gốc; phổ VAST do nhóm tự đo. Dùng cho mục đích nghiên cứu, học thuật.
