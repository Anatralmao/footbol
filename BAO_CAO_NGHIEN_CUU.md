# BÁO CÁO NGHIÊN CỨU KHOA HỌC
## ĐỀ XUẤT CHIẾN THUẬT BÓNG ĐÁ BẰNG THUẬT TOÁN PHÂN LỚP NAIVE BAYES

---

## 1. TỔNG QUAN NGHIÊN CỨU

### 1.1. Mục tiêu
Nghiên cứu này nhằm xây dựng mô hình học máy sử dụng thuật toán **Gaussian Naive Bayes** để phân loại và dự đoán kết quả trận đấu bóng đá dựa trên các chỉ số chiến thuật.

### 1.2. Dữ liệu
- **Số lượng mẫu**: 6,928 trận đấu
- **Số lượng đặc trưng**: 20 biến (chọn 7 biến tối ưu)
- **Phân phối kết quả**:
  - Win: 2,667 trận (38.50%)
  - Loss: 2,667 trận (38.50%)
  - Draw: 1,594 trận (23.01%)
- **Chất lượng dữ liệu**: Không có giá trị thiếu

---

## 2. PHƯƠNG PHÁP NGHIÊN CỨU

### 2.1. Lựa chọn Đặc trưng

Sau khi phân tích ANOVA và ma trận tương quan, nghiên cứu đã chọn **7 đặc trưng tối ưu**:

| Hạng | Đặc trưng | Importance Score | F-statistic | p-value |
|------|-----------|------------------|-------------|---------|
| 1 | **xG (Expected Goals)** | 0.1503 | 738.65 | < 0.001 |
| 2 | **shots** (Số cú sút) | 0.0859 | 383.41 | < 0.001 |
| 3 | **possession_ratio** (Tỷ lệ kiểm soát bóng) | 0.0799 | 338.79 | < 0.001 |
| 4 | **avg_x_position** (Vị trí TB trên sân) | 0.0460 | 183.83 | < 0.001 |
| 5 | **avg_pass_length** (Chiều dài chuyền TB) | 0.0292 | 101.23 | < 0.001 |
| 6 | **pressures** (Áp lực phòng ngự) | 0.0180 | 87.22 | < 0.001 |
| 7 | **tackles** (Pha tranh chấp) | 0.0149 | 50.16 | < 0.001 |

**Lý do loại bỏ các biến khác**:
- `num_passes`: Tương quan cao với `possession_ratio` (r = 0.89)
- `final_third_actions`: Đa cộng tuyến nghiêm trọng (r > 0.76 với nhiều biến)
- `team_width`: Phương sai quá thấp (std ~ 0.3)
- `interceptions`, `ppda`: Ảnh hưởng yếu (F-stat < 10)

### 2.2. Tiền xử lý Dữ liệu

1. **Chia tập dữ liệu**: Train (80%) - Test (20%)
2. **Chuẩn hóa**: Sử dụng StandardScaler (mean = 0, std = 1)
3. **Stratified Split**: Đảm bảo phân phối đồng đều giữa train/test

### 2.3. Thuật toán Naive Bayes

**Gaussian Naive Bayes** được chọn vì:
- Phù hợp với dữ liệu liên tục
- Tính toán nhanh, hiệu quả
- Hoạt động tốt với dữ liệu có số chiều vừa phải
- Cung cấp xác suất dự đoán rõ ràng

**Siêu tham số tối ưu**:
- `var_smoothing`: 1.00e-12 (tìm được qua Grid Search)

---

## 3. KẾT QUẢ NGHIÊN CỨU

### 3.1. Hiệu suất Mô hình

| Metric | Train Set | Test Set | Cross-Validation (5-fold) |
|--------|-----------|----------|---------------------------|
| **Accuracy** | 54.13% | **54.69%** | 54.04% (±1.72%) |

**Nhận xét**:
- ✅ Mô hình **ổn định** (chênh lệch train-test chỉ 0.56%)
- ✅ **Không có overfitting**
- ⚠️ Accuracy ~54% cho thấy bài toán phân loại kết quả bóng đá là khó (nhiều yếu tố ngẫu nhiên)

### 3.2. Phân tích Chi tiết theo Class

#### Classification Report (Test Set):

```
              precision    recall  f1-score   support

        Draw     0.4375    0.0439    0.0798       319
        Loss     0.5331    0.7697    0.6299       534
         Win     0.5712    0.6248    0.5968       533

    accuracy                         0.5469      1386
   macro avg     0.5139    0.4794    0.4355      1386
weighted avg     0.5257    0.5469    0.4905      1386
```

**Phân tích**:

1. **Win Class** (Thắng):
   - Precision: 57.12% - Khi dự đoán Win, đúng 57% trường hợp
   - Recall: 62.48% - Bắt được 62% các trận Win thực tế
   - F1-score: 0.5968 - Cân bằng khá tốt
   - **Kết luận**: Mô hình dự đoán Win tương đối tốt

2. **Loss Class** (Thua):
   - Precision: 53.31%
   - Recall: 76.97% - **Cao nhất** - Mô hình nhận diện Loss rất tốt
   - F1-score: 0.6299 - **Cao nhất**
   - **Kết luận**: Mô hình dự đoán Loss tốt nhất

3. **Draw Class** (Hòa):
   - Precision: 43.75%
   - Recall: 4.39% - **Rất thấp** - Chỉ bắt được 4% các trận hòa
   - F1-score: 0.0798 - **Thấp nhất**
   - **Kết luận**: Mô hình **khó dự đoán Draw** (điều này hợp lý vì Draw phụ thuộc nhiều yếu tố ngẫu nhiên)

### 3.3. Confusion Matrix

```
             Pred Win  Pred Draw  Pred Loss
Actual Win        333         15        185
Actual Draw       130         14        175
Actual Loss       120          3        411
```

**Insights**:
- Win: 333/533 dự đoán đúng (62.5%)
- Draw: 14/319 dự đoán đúng (4.4%) - **Cần cải thiện**
- Loss: 411/534 dự đoán đúng (77%) - **Tốt nhất**

### 3.4. Phân tích Đặc trưng theo Class

#### Mean values của từng đặc trưng (đã chuẩn hóa):

| Đặc trưng | Win | Draw | Loss | Phân biệt |
|-----------|-----|------|------|-----------|
| **xG** | +0.476 | -0.003 | -0.474 | ⭐⭐⭐ Rất cao |
| **shots** | +0.362 | -0.010 | -0.356 | ⭐⭐⭐ Cao |
| **possession_ratio** | +0.351 | -0.017 | -0.341 | ⭐⭐⭐ Cao |
| **avg_x_position** | +0.250 | +0.039 | -0.273 | ⭐⭐ Trung bình |
| **pressures** | -0.164 | -0.000 | +0.164 | ⭐ Thấp (ngược chiều) |
| **tackles** | -0.123 | +0.176 | +0.017 | ⭐ Thấp |
| **avg_pass_length** | -0.221 | +0.151 | +0.131 | ⭐ Thấp |

**Nhận xét quan trọng**:
- Đội **Win** có: xG cao, shots nhiều, kiểm soát bóng tốt, vị trí cao
- Đội **Loss** có: xG thấp, ít shots, ít kiểm soát, chịu nhiều áp lực
- Đội **Draw**: Ở giữa, khó phân biệt rõ ràng

---

## 4. DEMO DỰ ĐOÁN

### Case 1: Đội tấn công mạnh
**Input**:
```
xg: 2.5, shots: 18, possession: 60%, 
avg_x_position: 65, pressures: 140, 
tackles: 30, avg_pass_length: 18
```
**Dự đoán**: Win (93.74% xác suất)

### Case 2: Đội phòng ngự
**Input**:
```
xg: 0.8, shots: 8, possession: 40%, 
avg_x_position: 52, pressures: 180, 
tackles: 42, avg_pass_length: 22
```
**Dự đoán**: Loss (89.52% xác suất)

### Case 3: Đội cân bằng
**Input**:
```
xg: 1.5, shots: 13, possession: 50%, 
avg_x_position: 58, pressures: 160, 
tackles: 36, avg_pass_length: 20
```
**Dự đoán**: Win (46.68% xác suất) vs Loss (33.62%) vs Draw (19.70%)

---

## 5. THẢO LUẬN

### 5.1. Ưu điểm của Nghiên cứu

✅ **Lựa chọn đặc trưng khoa học**: Dựa trên ANOVA, tránh đa cộng tuyến  
✅ **Mô hình ổn định**: Không overfitting, CV score nhất quán  
✅ **Dự đoán Win/Loss tốt**: F1-score > 0.59  
✅ **Giải thích được**: Naive Bayes cung cấp xác suất rõ ràng  
✅ **Tốc độ nhanh**: Phù hợp cho ứng dụng thực tế  

### 5.2. Hạn chế

⚠️ **Accuracy ~54%**: Chưa cao, nhưng hợp lý do:
- Bóng đá có nhiều yếu tố ngẫu nhiên (thời tiết, trọng tài, chấn thương...)
- Chỉ dùng dữ liệu chiến thuật, thiếu yếu tố tâm lý, phong độ
- Draw rất khó dự đoán (recall chỉ 4.4%)

⚠️ **Giả định độc lập**: Naive Bayes giả định các đặc trưng độc lập có điều kiện, nhưng trong thực tế một số đặc trưng có tương quan nhẹ (đã cố gắng giảm thiểu)

### 5.3. So sánh với Baseline

- **Random Guess**: 33.33% (1/3 classes)
- **Majority Class**: 38.5% (luôn dự đoán Win/Loss)
- **Mô hình của chúng ta**: **54.69%** 
- **Cải thiện**: +16.36% so với random, +16.19% so với majority

---

## 6. KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN

### 6.1. Kết luận

Nghiên cứu đã thành công xây dựng mô hình Gaussian Naive Bayes để phân loại kết quả trận đấu bóng đá với độ chính xác **54.69%** trên tập test. Mô hình hoạt động tốt nhất khi dự đoán **Loss** (F1 = 0.63), tốt với **Win** (F1 = 0.60), nhưng gặp khó khăn với **Draw** (F1 = 0.08).

**xG (Expected Goals)** là đặc trưng quan trọng nhất, theo sau là số cú sút và tỷ lệ kiểm soát bóng.

### 6.2. Hướng phát triển

1. **Cải thiện mô hình**:
   - Thử các thuật toán khác: Random Forest, Gradient Boosting, Neural Networks
   - Kết hợp nhiều mô hình (ensemble)
   - Thêm đặc trưng: phong độ gần đây, head-to-head, yếu tố sân nhà

2. **Xử lý class imbalance**:
   - Sử dụng SMOTE cho class Draw
   - Điều chỉnh class weights
   - Oversampling/undersampling

3. **Feature engineering**:
   - Tạo biến tương tác (xG * possession)
   - Chuẩn hóa theo đối thủ
   - Time-series features (rolling average)

4. **Ứng dụng thực tế**:
   - Xây dựng hệ thống đề xuất chiến thuật real-time
   - Tích hợp vào công cụ phân tích cho HLV
   - API dự đoán kết quả trận đấu

---

## 7. TÀI LIỆU THAM KHẢO

1. Dataset: tactical_dataset_with_results.csv (6,928 trận đấu)
2. Scikit-learn Documentation - Naive Bayes
3. Football Analytics Research Papers
4. Expected Goals (xG) Methodology

---

## PHỤ LỤC

### A. Mã nguồn
- File: `football_tactics_classification.py`
- Mô hình đã train: `football_tactics_naive_bayes.pkl`

### B. Biểu đồ
- `football_tactics_results.png`: Confusion matrix, metrics, CV scores
- `feature_distributions.png`: Phân phối các đặc trưng theo kết quả

### C. Công thức Gaussian Naive Bayes

Xác suất hậu nghiệm:

```
P(y|X) ∝ P(y) × ∏ P(xi|y)
```

Với Gaussian distribution:

```
P(xi|y) = (1/√(2π σ²y)) × exp(-(xi - μy)²/(2σ²y))
```

Trong đó:
- μy: mean của đặc trưng xi trong class y
- σ²y: variance của đặc trưng xi trong class y

---

**Người thực hiện**: [Tên của bạn]  
**Ngày hoàn thành**: 2026-02-10  
**Công cụ**: Python 3, Scikit-learn, Pandas, NumPy, Matplotlib, Seaborn
