# Báo Cáo Lab 21 - CI/CD for AI Systems

**AIInAction - VinUni | Day 21**

**Họ Tên:** Hồ Thành Tiến
**MSSV:** 2A202600868

---

## 1. Bộ Siêu Tham Số Đã Chọn Và Lý Do

### 1.1 Kết Quả Thí Nghiệm

Đã thực hiện 5 lần chạy thí nghiệm với các bộ siêu tham số khác nhau trên tập dữ liệu Wine Quality (2998 mẫu huấn luyện, 500 mẫu đánh giá):

| Run | n_estimators | max_depth | min_samples_split | Accuracy | F1-Score |
|-----|-------------|-----------|-------------------|----------|----------|
| 1   | 100         | 5         | 2                 | 0.5640   | 0.5534   |
| 2   | 50          | 3         | 2                 | 0.5580   | 0.5185   |
| 3   | 200         | 10        | 5                 | 0.6440   | 0.6417   |
| **4*** | **300** | **20**   | **2**             | **0.6780** | **0.6767** |
| 5   | 500         | 20        | 2                 | 0.6780   | 0.6766   |

*(\*) Bộ tham số được chọn*

### 1.2 Bộ Tham Số Được Chọn

**`n_estimators=300, max_depth=20, min_samples_split=2`**

**Lý do lựa chọn:**

- **max_depth=20:** Cho phép cây đủ sâu để học các pattern phức tạp trong dữ liệu Wine Quality (12 đặc trưng hóa học), tăng accuracy từ 0.564 (max_depth=5) lên 0.678.
- **n_estimators=300:** Tăng số cây giúp ổn định kết quả dự đoán. Thử nghiệm cho thấy 500 cây không cải thiện hơn 300 cây (cùng cho accuracy 0.678), nên 300 là điểm tối ưu về hiệu suất/tốc độ.
- **min_samples_split=2:** Cho phép phân chia tối đa các nút, phù hợp với tập dữ liệu kích thước trung bình (2998 mẫu).

---

## 2. Kết Quả So Sánh Bước 2 và Bước 3

| Chỉ số | Bước 2 (2998 mẫu) | Bước 3 (5996 mẫu) |
|--------|-------------------|-------------------|
| accuracy | 0.6780 | 0.6780 |
| f1_score | 0.6767 | ~0.677 |

Accuracy giữ nguyên do tập eval.csv cố định (500 mẫu held-out). Pipeline Bước 3 được kích hoạt hoàn toàn tự động bởi commit dữ liệu `data: bo sung 2998 mau du lieu moi (train_phase2)` mà không cần tác động thủ công.

---

## 3. Khó Khăn Gặp Phải Và Cách Giải Quyết

### Khó khăn 1: Xung đột phiên bản Python 3.13 và scikit-learn

**Vấn đề:** `requirements.txt` chỉ định `scikit-learn==1.4.2` không tương thích với Python 3.13 (yêu cầu `numpy==2.0.0rc1` không tồn tại).

**Giải pháp:** Cài đặt phiên bản mới nhất của scikit-learn không pin version cụ thể, Python 3.13 tự động chọn phiên bản tương thích.

### Khó khăn 2: Lỗi BOM (Byte Order Mark) trong file YAML

**Vấn đề:** Khi ghi file `params.yaml` bằng PowerShell `Set-Content`, file bị thêm ký tự BOM (`EF BB BF`) ở đầu, khiến MLflow từ chối tên tham số vì có ký tự Unicode ẩn.

**Giải pháp:** Sử dụng Python để ghi file với `encoding='utf-8'` thuần túy (không BOM).

### Khó khăn 3: Giá trị max_depth=None trong YAML

**Vấn đề:** Khi ghi `max_depth: None` vào YAML, Python đọc ra chuỗi `'None'` thay vì giá trị `null`, dẫn đến lỗi `InvalidParameterError` trong scikit-learn.

**Giải pháp:** Thay thế `None` bằng giá trị số nguyên lớn (`max_depth=20`) để đạt hiệu quả tương đương mà không gây lỗi.

---

*Lab 21 - AIInAction VinUni | Hồ Thành Tiến - MSSV: 2A202600868 | 2026*
