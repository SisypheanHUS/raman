# RamanID — Nhận diện phổ Raman trên trình duyệt

[![Live](https://img.shields.io/badge/live-sisypheanhus.github.io%2Framan-2ea44f?logo=github)](https://sisypheanhus.github.io/raman/)
![Static](https://img.shields.io/badge/backend-none-blue)
![Model](https://img.shields.io/badge/model-v3%20Bayes%20%2B%20within--class%20PCA-8a2be2)
![Accuracy](https://img.shields.io/badge/strict%20LOO-87.1%25%20%7C%2090.0%25%20ho%C3%A1%20ch%E1%BA%A5t-orange)

Kéo thả một file phổ Raman (`.txt` / `.csv`) và nhận kết quả nhận diện
vật liệu kèm xác suất hậu nghiệm — **toàn bộ tính toán chạy trong trình
duyệt**, không có server, không upload dữ liệu.

> 🔗 **Website:** <https://sisypheanhus.github.io/raman/>

## Repo này chứa gì

Đây là repo **deploy** (GitHub Pages) — chỉ gồm các file tĩnh đã được
sinh tự động từ mã nguồn nghiên cứu. Không sửa tay các file ở đây.

| File | Kích thước | Nội dung |
|------|-----------:|----------|
| `index.html` | ~65 KB | Ứng dụng chính. Nhúng sẵn toàn bộ thuật toán (`core.js`) — tiền xử lý, Bayes factor, phân loại, tìm peak, đa phương thức |
| `data.json` | 34 MB (~23 MB gzip) | Thư viện phổ tham chiếu + mô hình within-class, mã hoá base64 uint16 (lo/hi từng đoạn) |
| `catalog.html` | ~27 KB | Duyệt/xem toàn bộ phổ trong thư viện theo nguồn |
| `about.html` | ~20 KB | Mô tả phương pháp, độ chính xác, lịch sử phiên bản |
| `algorithm.html` | ~100 KB | Tài liệu thuật toán kiểu bài báo: khung RamanID, tổng quan tài liệu, benchmark LOO nghiêm ngặt |

## Vì sao trang web tái lập chính xác thuật toán và thư viện

```
ramanlib.py + bayes.py + multimodal.py     library_ext.npz (16 070 phổ)
            │  port 1:1                              │  build_deploy.py
            ▼                                        ▼
         core.js  ──── inline ────▶  index.html      data.json
            │
            └── test_core.mjs: so sánh với kết quả Python (18/18 PASS)
```

| Thành phần | Cách đảm bảo giống Python |
|------------|---------------------------|
| **Thuật toán** | `core.js` là bản port thủ công của pipeline Python: nội suy lưới 450–3200 cm⁻¹ (bước 2), Savitzky–Golay 9/3, baseline ALS, chuẩn hoá L2, Bayesian evidence với null Legendre bậc 5, within-class PCA. Hằng số hiệu chuẩn (`KAPPA = 0.08`, `NULL_TAU = 0.684`, `LAMBDA = 0.003`; LOO nghiêm ngặt 25/08/2026) được đồng bộ từ `bayes.py` |
| **Kiểm chứng** | `test_core.mjs` chạy `core.js` dưới Node trên đúng file phổ và so log-BF, top-1, vị trí peak với output Python — phải PASS trước mỗi lần build |
| **Mô hình WC** | 517 mô hình within-class (mean + thành phần chính) được `build_deploy.py` học từ **toàn bộ 16 070 phổ** rồi encode `uint16 → base64`. `classify()` ưu tiên `pairBfWc()` cho các lớp này → web dùng đúng lượng thông tin như Python |
| **Phổ raw** | Toàn bộ 16 070 phổ được embed vào `data.json`. Lớp chưa đủ điều kiện WC (< 3 phổ, hoặc vùng phủ chung < 30 %) được so trực tiếp bằng `pairBF()` |

## Thư viện tham chiếu

| Nguồn | Số phổ | Ghi chú |
|-------|-------:|---------|
| RRUFF | 11 154 | Phổ Raman khoáng vật chuẩn (excellent + fair) |
| Pharma API | 3 510 | Dược phẩm (785 nm, Springer/figshare, CC BY 4.0) |
| ROD | 1 121 | Raman Open Database |
| VAST | 255 | Đo tại phòng thí nghiệm (LabRAM HR Evolution, 532 nm) |
| NICODOM demo | 18 | |
| Dung môi | 12 | |
| **Tổng** | **16 070** phổ, **2 507** lớp | + **517** mô hình within-class (PCA) |

Kèm 141 phổ huỳnh quang và 52 phổ hấp thụ cho chế độ đa phương thức (v3).

## Độ chính xác (leave-one-out nghiêm ngặt trên 248 phổ VAST, thư viện 16 070 phổ)

Mô hình lớp được dựng lại không có phổ đang chấm; đồng nghĩa xuyên nguồn
(Aceton/Acetone, PE/Polyethylene…) tính là một lớp. Các số 96.x% công bố
trước 25/08/2026 đến từ giao thức có rò rỉ và đã bị thay thế.

| Phiên bản | Phương pháp | Tổng thể | Hoá chất |
|-----------|-------------|---------:|---------:|
| — | Cosine / HQI (cùng giao thức) | 81.9% | 86.7% |
| — | Không gian con kiểu SIMCA (cùng giao thức) | 82.3% | 83.9% |
| — | χ² phần dư kiểu TruScan (cùng giao thức) | 74.6% | 81.7% |
| v3 | Bayes đơn mẫu + nền Legendre | 85.5% | 90.0% |
| **v3** | Bayes + within-class PCA (+ đa phương thức) | **87.1%** | **90.0%** |

ECE 0.069 (κ = 0.08); từ chối chất lạ 35.6% ở 10% từ chối nhầm (thử loại-lớp,
chỉ là cận dưới) — chi tiết và hạn chế ở `algorithm.html` §9.

## Giấy phép dữ liệu

Phổ RRUFF, ROD và các bộ công khai khác giữ nguyên giấy phép của nguồn
gốc; phổ VAST do nhóm tự đo. Dùng cho mục đích nghiên cứu, học thuật.
