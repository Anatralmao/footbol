# BÁO CÁO NGHIÊN CỨU KHOA HỌC
## ĐỀ TÀI: XÂY DỰNG HỆ THỐNG ĐỀ XUẤT CHIẾN THUẬT BÓNG ĐÁ DỰA TRÊN MÔ HÌNH GAUSSIAN NAIVE BAYES

---

## TÓM TẮT (ABSTRACT)
Nghiên cứu này đề xuất một phương pháp tiếp cận định lượng để tối ưu hóa chiến thuật bóng đá bằng thuật toán học máy Gaussian Naive Bayes. Dựa trên bộ dữ liệu 6,928 trận đấu tại 5 giải VĐQG hàng đầu châu Âu, mô hình không chỉ dự đoán kết quả trận đấu với độ chính xác **51.95%**, mà quan trọng hơn, nó trích xuất được "bộ gen chiến thắng" (Tactical DNA). Kết quả cho thấy các đội chiến thắng có chỉ số bàn thắng kỳ vọng (xG) trung bình đạt **1.82** và tỷ lệ kiểm soát bóng **53.3%**. Hệ thống đề xuất được xây dựng có khả năng đưa ra các điều chỉnh chiến thuật cụ thể cho từng đội bóng dựa trên độ lệch chuẩn so với mô hình chiến thắng.

---

## 1. GIỚI THIỆU
Trong bóng đá hiện đại, việc ra quyết định dựa trên dữ liệu (Data-driven decision making) đang trở thành xu hướng tất yếu. Tuy nhiên, đa số các nghiên cứu hiện tại tập trung vào việc dự đoán kết quả (Prediction) hơn là đưa ra giải pháp (Prescription). Nghiên cứu này đặt mục tiêu lấp đầy khoảng trống đó bằng cách sử dụng Gaussian Naive Bayes - một thuật toán có khả năng giải thích cao - để xây dựng hệ thống khuyến nghị chiến thuật.

## 2. PHƯƠNG PHÁP NGHIÊN CỨU

### 2.1. Dữ liệu và Tiền xử lý
* **Nguồn dữ liệu:** Tập dữ liệu bao gồm các thông số kỹ thuật (Possession, xG, PPDA, Passes...) của 6,928 trận đấu.
* **Phân chia dữ liệu (Data Splitting):** Để đảm bảo tính khách quan và tránh hiện tượng rò rỉ dữ liệu (Data Leakage), tập dữ liệu được chia thành tập Huấn luyện (80%) và Kiểm thử (20%) bằng kỹ thuật **Stratified Sampling**. Điều này đảm bảo tỷ lệ các lớp kết quả (Thắng/Hòa/Thua) được giữ nguyên trong cả hai tập.
* **Chuẩn hóa:** Sử dụng `StandardScaler` để đưa các biến về phân phối chuẩn ($\mu=0, \sigma=1$). Quá trình `fit` chỉ thực hiện trên tập Train, sau đó tham số được dùng để `transform` tập Test.

### 2.2. Mô hình Gaussian Naive Bayes (GNB)
Nghiên cứu lựa chọn GNB vì khả năng mô hình hóa xác suất của từng đặc trưng độc lập. Với mỗi lớp kết quả $y$ (Win/Draw/Loss), mô hình học được vector trung bình $\mu_y$ và phương sai $\sigma^2_y$.
Cơ chế đề xuất chiến thuật dựa trên việc so sánh vector đặc trưng hiện tại của đội bóng ($x_{current}$) với vector trung bình của lớp chiến thắng ($\mu_{win}$).

## 3. KẾT QUẢ NGHIÊN CỨU

### 3.1. Hiệu năng mô hình
Trên tập kiểm thử (Test set), mô hình đạt các chỉ số sau:
* **Accuracy:** 51.95% (Cao hơn ngẫu nhiên 33.3%).
* **Precision (Win):** 55.5% - Trong các trận mô hình dự báo Thắng, 55.5% thực tế là Thắng.
* **Recall (Loss):** 75.3% - Mô hình nhận diện các trận Thua rất tốt.
* **Vấn đề lớp "Draw":** F1-score chỉ đạt 0.08. Mô hình gặp khó khăn lớn trong việc phân biệt trận Hòa, thường nhầm lẫn sang Thua.

### 3.2. "Bộ gen chiến thắng" (Tactical DNA)
Dựa trên tham số $	heta$ (theta_) của mô hình GNB, chúng tôi xác định được bộ chỉ số lý tưởng cho một đội chiến thắng trung bình:

| Chỉ số (Feature) | Giá trị trung bình (Win Class) | Ý nghĩa chiến thuật |
| :--- | :--- | :--- |
| **xG (Expected Goals)** | **1.82** | Cần tạo ra cơ hội chất lượng cao tương đương gần 2 bàn/trận. |
| **Possession Ratio** | **53.3%** | Kiểm soát thế trận ở mức chủ động, nhưng không cần quá áp đảo (không cần >60%). |
| **Shots** | **14.75** | Tần suất dứt điểm cần đạt khoảng 15 lần/trận. |
| **PPDA** | **Thấp** | Chỉ số pressing tầm cao (PPDA thấp tốt hơn) để đoạt bóng sớm. |

### 3.3. Demo Hệ thống Đề xuất
Thử nghiệm với một mẫu đội bóng đang có phong độ kém (Dự báo: Loss):
* *Thông số hiện tại:* xG = 0.8, Possession = 45%.
* *Hệ thống đề xuất:*
    1.  Tăng cường độ rủi ro trong chuyền bóng để nâng xG lên ngưỡng 1.8 (+1.0).
    2.  Đẩy cao đội hình (Avg X Position) thêm 5m để hỗ trợ kiểm soát bóng lên mức 53%.

## 4. KẾT LUẬN
Mô hình Gaussian Naive Bayes tuy đơn giản nhưng cung cấp một khung tham chiếu định lượng (Baseline) vững chắc cho các HLV. Hạn chế lớn nhất là khả năng dự báo trận Hòa và giả định các biến độc lập (trong khi xG và Shots có tương quan cao).

---

## PHỤ LỤC: TỰ ĐÁNH GIÁ HIỆU QUẢ (CRITICAL REVIEW)

### 1. Ưu điểm
* **Tính khoa học (Academic Rigor):** Đã khắc phục hoàn toàn lỗi Data Leakage. Việc chia tập Train/Test trước khi Scale là chính xác.
* **Tính ứng dụng (Applicability):** Đã chuyển đổi thành công từ bài toán "Dự báo" sang "Đề xuất". Việc trích xuất `gnb.theta_` (means) để làm benchmark cho các đội bóng là một cách tiếp cận thông minh, tận dụng đặc tính dễ giải thích (Explainable AI) của Naive Bayes.
* **Trung thực:** Báo cáo đúng con số 52% thay vì con số ảo 54-55% trước đó, và thừa nhận sự thất bại trong việc dự đoán cửa Hòa (Draw).

### 2. Nhược điểm (Cần cải thiện)
* **Vấn đề giả định độc lập (Independence Assumption):** Naive Bayes giả định các biến độc lập với nhau. Trong bóng đá, `Shots` và `xG` tương quan rất mạnh (sút nhiều thì xG cao), `Possession` và `Passes` cũng vậy. Việc dùng GNB vi phạm giả định này, làm giảm độ tin cậy của xác suất dự báo.
* **Hiệu năng thấp với lớp Draw:** F1-score của Draw quá thấp (0.08) làm ảnh hưởng đến tổng thể.
* **Thiếu yếu tố phi kỹ thuật:** Mô hình hiện tại chỉ nhìn vào thông số trận đấu (Technical stats) mà bỏ qua Phong độ gần đây (Form), Lợi thế sân nhà (Home advantage), hay Chất lượng cầu thủ (Squad value).
