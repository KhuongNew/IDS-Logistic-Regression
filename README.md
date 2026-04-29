# IDS Logistic Regression

Project này xây dựng mô hình `Logistic Regression` để phát hiện tấn công mạng trên bộ dữ liệu `CICIDS2017`, phục vụ bài Lab 6 môn An toàn/Bảo mật hệ thống thông tin.

## Mục tiêu

- Đọc và gộp nhiều file CSV của bộ dữ liệu CICIDS2017.
- Làm sạch dữ liệu, loại bỏ giá trị lỗi và cột không hữu ích.
- Chọn tập đặc trưng cốt lõi để huấn luyện mô hình phân loại lưu lượng mạng.
- Huấn luyện và đánh giá mô hình `Logistic Regression`.
- Lưu mô hình và các chỉ số đánh giá để tái sử dụng.

## Cấu trúc thư mục

```text
IDS-Logistic-Regression/
├── data/                # Dữ liệu đầu vào CICIDS2017 (8 file CSV)
├── deliverables/        # Nơi chứa file nộp/báo cáo nếu cần
├── logs/                # Thư mục log
├── models/              # Model đã huấn luyện và metrics
├── notebooks/           # Notebook chính của project
├── requirements.txt     # Thư viện cần cài đặt
└── README.md
```

## Dữ liệu sử dụng

Project sử dụng 8 file CSV trong thư mục `data/`, tương ứng với các phiên thu thập của bộ dữ liệu `CICIDS2017`:

- `Monday-WorkingHours.pcap_ISCX.csv`
- `Tuesday-WorkingHours.pcap_ISCX.csv`
- `Wednesday-workingHours.pcap_ISCX.csv`
- `Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv`
- `Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv`
- `Friday-WorkingHours-Morning.pcap_ISCX.csv`
- `Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv`
- `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv`

## Quy trình xử lý

Notebook chính: [ML_IDS_LR.ipynb](/d:/hoc/ATBM%20HTTT/Lab%206/IDS-Logistic-Regression/notebooks/ML_IDS_LR.ipynb)

Pipeline trong notebook gồm các bước chính:

1. Gộp 8 file CSV thành một bảng dữ liệu chung.
2. Chuẩn hóa tên cột, thay `inf/-inf` bằng `NaN`.
3. Xóa dòng trùng lặp và loại bỏ các cột có phương sai bằng 0.
4. Tối ưu bộ nhớ bằng cách downcast kiểu số.
5. Đổi tên một số cột để thống nhất schema.
6. Chọn 18 đặc trưng cốt lõi cho mô hình.
7. Gộp các nhãn `Web Attack` về cùng một lớp và lọc các lớp có dưới 50 mẫu.
8. Chia tập train/test theo tỷ lệ `80/20` với `stratify`.
9. Huấn luyện pipeline gồm:
   - `SimpleImputer(strategy="median")`
   - `StandardScaler()`
   - `RandomUnderSampler()` cho lớp đa số
   - `LogisticRegression(solver="saga", class_weight="balanced", max_iter=3000)`
10. Đánh giá bằng `Accuracy`, `Balanced Accuracy`, `F1-macro`, `F1-weighted`, `classification report` và `confusion matrix`.

## Đặc trưng sử dụng

Mô hình dùng 18 đặc trưng:

- `Protocol`
- `Flow Duration`
- `Tot Fwd Pkts`
- `Tot Bwd Pkts`
- `TotLen Fwd Pkts`
- `TotLen Bwd Pkts`
- `Fwd Pkt Len Mean`
- `Bwd Pkt Len Mean`
- `Flow Byts/s`
- `Flow Pkts/s`
- `Pkt Len Mean`
- `Pkt Len Std`
- `SYN Flag Cnt`
- `ACK Flag Cnt`
- `FIN Flag Cnt`
- `RST Flag Cnt`
- `PSH Flag Cnt`
- `URG Flag Cnt`

## Kích thước dữ liệu sau xử lý

- Dữ liệu gốc sau khi gộp: `2,830,743` dòng, `79` cột
- Sau khi xóa trùng lặp: giảm `308,381` dòng
- Sau khi loại cột phương sai bằng 0: còn `71` cột
- Sau khi chọn đặc trưng và lọc nhãn: `2,522,315` mẫu, `18` đặc trưng
- Tập train: `2,017,852` mẫu
- Tập test: `504,463` mẫu

## Các lớp nhãn chính

Sau bước lọc, dữ liệu còn các lớp:

- `BENIGN`
- `DoS Hulk`
- `DDoS`
- `PortScan`
- `DoS GoldenEye`
- `FTP-Patator`
- `DoS slowloris`
- `DoS Slowhttptest`
- `SSH-Patator`
- `Web Attack`
- `Bot`

## Kết quả mô hình

Metrics được lưu tại [ids_logistic_regression_best_metrics.json](/d:/hoc/ATBM%20HTTT/Lab%206/IDS-Logistic-Regression/models/ids_logistic_regression_best_metrics.json):

- `Accuracy`: `0.7042`
- `Balanced Accuracy`: `0.8614`
- `F1-macro`: `0.3885`
- `F1-weighted`: `0.7865`
- `Training time`: `1689.49` giây

Nhận xét nhanh:

- Mô hình nhận diện khá tốt các lớp phổ biến như `DoS Hulk`, `DDoS`, `PortScan`.
- `Balanced Accuracy` cao cho thấy mô hình vẫn bắt được nhiều lớp thiểu số.
- `F1-macro` còn thấp, phản ánh hiệu năng chưa đồng đều giữa các lớp hiếm.

## Model đã lưu

Artifact đã có sẵn trong thư mục `models/`:

- [ids_logistic_regression_best.pkl](/d:/hoc/ATBM%20HTTT/Lab%206/IDS-Logistic-Regression/models/ids_logistic_regression_best.pkl): chứa pipeline huấn luyện, label encoder, danh sách feature và metrics.
- [ids_logistic_regression_best_metrics.json](/d:/hoc/ATBM%20HTTT/Lab%206/IDS-Logistic-Regression/models/ids_logistic_regression_best_metrics.json): chứa các chỉ số đánh giá chính.

## Cài đặt

Tạo môi trường ảo và cài dependencies:

```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

Các thư viện chính:

- `pandas`
- `numpy`
- `scikit-learn`
- `imbalanced-learn`
- `matplotlib`
- `seaborn`
- `jupyter`
- `joblib`

## Cách chạy

Chạy notebook:

```bash
jupyter notebook notebooks/ML_IDS_LR.ipynb
```

Hoặc mở bằng JupyterLab:

```bash
jupyter lab
```

Notebook sẽ:

- đọc dữ liệu từ thư mục `data/`
- huấn luyện mô hình Logistic Regression
- hiển thị biểu đồ EDA và confusion matrix
- lưu model và metrics vào thư mục `models/`

## Hàm dự đoán mẫu mới

Notebook có định nghĩa hàm:

```python
def predict_one(sample_dict):
    sample_df = pd.DataFrame([sample_dict])[selected_features]
    pred_id = pipeline.predict(sample_df)[0]
    pred_label = le.inverse_transform([pred_id])[0]
    return pred_label
```

Để dùng hàm này, `sample_dict` cần chứa đầy đủ 18 đặc trưng theo đúng tên cột đã chọn.