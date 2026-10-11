# RamanSisyphean — Nhận diện phổ Raman trên trình duyệt

[![Live](https://img.shields.io/badge/live-sisypheanhus.github.io%2Framan-2ea44f?logo=github)](https://sisypheanhus.github.io/raman/)
![Static](https://img.shields.io/badge/backend-none-blue)
![Model](https://img.shields.io/badge/model-v3%20Bayes%20%2B%20within--class%20PCA-8a2be2)
![Accuracy](https://img.shields.io/badge/degraded%20copies-95.3%25%20%7C%20handheld%2020%2F20-orange)

Kéo thả một file phổ Raman (`.txt` / `.csv`) và nhận kết quả nhận diện
vật liệu kèm xác suất hậu nghiệm — **toàn bộ tính toán chạy trong trình
duyệt**, không có server, không upload dữ liệu.

> 🔗 **Website:** <https://sisypheanhus.github.io/raman/>

## Repo này chứa gì

Đây là repo **deploy** (GitHub Pages) — chỉ gồm các file tĩnh đã được
sinh tự động từ mã nguồn nghiên cứu. Không sửa tay các file ở đây.

| File | Kích thước | Nội dung |
|------|-----------:|----------|
| `index.html` | ~87 KB | Ứng dụng chính. Nhúng sẵn toàn bộ thuật toán (`core.js`) — tiền xử lý, Bayes factor, phân loại, tìm peak, đa phương thức |
| `data.json` | 40 MB (~27 MB gzip) | Thư viện phổ tham chiếu + mô hình within-class, mã hoá base64 uint16 (lo/hi từng đoạn) |
| `catalog.html` | ~35 KB | Duyệt/xem toàn bộ phổ trong thư viện theo nguồn |
| `about.html` | ~34 KB | Mô tả phương pháp, độ chính xác, lịch sử phiên bản |
| `algorithm.html` | ~128 KB | Tài liệu thuật toán kiểu bài báo: khung RamanSisyphean, tổng quan tài liệu, benchmark LOO nghiêm ngặt |

Giao diện (28/08/2026) áp từ bản thiết kế Claude Design v3 (nền tối, nhấn cyan;
Outfit · Syne · JetBrains Mono), một file `theme.css` dùng chung được inline vào cả 4 trang
khi build. Link mẫu chia sẻ được: `index.html?demo=0` (Ethanol), `?demo=1` (THF cầm tay), `?demo=2` (Glucose, ngoài thư viện),
`?demo=mm` (đa phương thức).

## Vì sao trang web tái lập chính xác thuật toán và thư viện

```
ramanlib.py + bayes.py + multimodal.py     library_ext.npz (11 601 phổ)
            │  port 1:1                              │  build_deploy.py
            ▼                                        ▼
         core.js  ──── inline ────▶  index.html      data.json
            │
            └── test_core.mjs: so sánh với kết quả Python (> 90 kiểm tra PASS)
```

| Thành phần | Cách đảm bảo giống Python |
|------------|---------------------------|
| **Thuật toán** | `core.js` là bản port thủ công của pipeline Python: làm mượt Savitzky–Golay 9/3 trên lưới đo gốc, nội suy lưới 50–4000 cm⁻¹ (bước 2, 1 976 kênh), baseline ALS, chuẩn hoá L2, Bayesian evidence với null Legendre bậc 5, within-class PCA, lượt chấm thứ hai (dịch ±1 kênh) và khớp độ phân giải. Phổ thư viện điền 0 ngoài dải đo; phổ truy vấn được chấm trên đúng các kênh nó đo (≥ 100 kênh). Hằng số hiệu chuẩn (`LAMBDA = 0.01`, `KAPPA = 0.09`, `NULL_TAU = 0.757`, `NULL_TAU0 = 0.748`; freeze #5, 11/09/2026) đồng bộ từ `bayes.py` |
| **Kiểm chứng** | `test_core.mjs` chạy `core.js` dưới Node trên 12 phổ (máy để bàn, Wasatch cầm tay, RRUFF giữ lại) và so top-1, log-BF, hậu nghiệm, cờ unknown, bằng chứng không nền, vị trí peak với `bayes.bayes_match` (`parity_ref.json`); `test_blur.mjs` kiểm bề rộng peak/làm mờ. Phải PASS trước mỗi lần build |
| **Mô hình WC** | 804 mô hình within-class (mean + thành phần chính) được `build_deploy.py` học từ **toàn bộ 11 601 phổ** rồi encode `uint16 → base64`. `classify()` dùng `pairBfWc()` cho các lớp này → web dùng đúng lượng thông tin như Python |
| **Phổ raw** | Toàn bộ 11 601 phổ được embed vào `data.json` (đoạn đo được, ngoài đoạn = 0). Mỗi lớp ≥ 3 phổ nên mọi lớp đều có mô hình trong lớp |

## Thư viện tham chiếu (freeze #5)

| Nguồn | Số phổ | Ghi chú |
|-------|-------:|---------|
| RRUFF | 8 091 | 772 khoáng vật, excellent + fair, laser 514–785 nm |
| Dược phẩm 785 nm | 3 510 | 32 dung môi/thuốc thử × ~110 phổ lặp (Flanagan & Glavin, Sci. Data 2025, CC BY 4.0) |
| **Tổng** | **11 601** phổ, **804** lớp | + **804** mô hình within-class (PCA) |

Kèm 141 phổ huỳnh quang và 52 phổ hấp thụ cho chế độ đa phương thức (giữ nguyên từ v3, không nằm trong bài báo). VAST, ROD, NICODOM và dung môi lẻ không còn trong thư viện từ 11/09/2026.

## Độ chính xác (bài báo FEE 2026: Bayesian marginal likelihood cho nhận dạng Raman)

| Bộ đánh giá | Evidence | Đối chứng |
|---|---:|---|
| Chia cân bằng thông tin (2 797 phổ, 232 chất) | **97.3 %** | cosine 96.9 %, CNN 1D 96.0 %, CNN 2D 95.7 % |
| Bản sao suy giảm (2 000 bản) | **95.3 %** | cosine 83.5 %; ở SNR < 10: 94.0 % so với 22.6 % |
| RRUFF unrated (150 phổ, ngoài thư viện) | **97.3 %** | — |
| Wasatch WP-785XS cầm tay (20 phổ) | **20/20** | — |
| LOO nghiêm ngặt phân tầng (3 511 truy vấn; dùng để hiệu chuẩn) | 84.3 % | phổ mẫu đơn 86.5 %, cosine 86.6 % |

κ = 0.09 (ECE 0.035), τ = 0.757, τ₀ = 0.748. Điểm yếu chính: trôi trục vượt ±1 kênh (91–92 %); phổ RRUFF độ phân giải thấp 59.3 % và "poor" 32.2 %. Chi phí một truy vấn trên CPU laptop: 0.75 s lượt thứ nhất, 1.9 s cấu hình đầy đủ. Chi tiết, hạn chế và benchmark ở `algorithm.html` §9.

**Hiệu chuẩn máy.** Ngưỡng unknown τ được hiệu chuẩn trên thư viện; máy khác dịch cả thang bằng chứng. Nạp ≥ 3 phổ của chất *có trong thư viện* đo trên máy của bạn rồi bấm **"Hiệu chuẩn máy"**: τ dịch theo trung vị bằng chứng của máy đó (`bayes.instrument_tau`), lưu trong trình duyệt, bỏ bằng một chạm.

## Giấy phép dữ liệu

Phổ RRUFF, bộ dược phẩm, Wasatch ENLIGHTEN (MIT) và các bộ công khai khác giữ nguyên
giấy phép của nguồn gốc. Dùng cho mục đích nghiên cứu, học thuật.
