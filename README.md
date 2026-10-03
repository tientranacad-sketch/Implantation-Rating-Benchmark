# Dự đoán kết cục mang thai (IVF)

Benchmark và baseline cho bài toán dự đoán kết cục mang thai nhị phân (có thai / không có thai) từ dữ liệu lâm sàng dạng bảng, ảnh phôi tĩnh, và kết hợp hai nguồn này, trên hai bộ dữ liệu IVF độc lập. Đây là một phần của báo cáo *Dataset Benchmark và Phân tích Baseline cho AI trong Hỗ trợ sinh sản (IVF)* — Genome IVF-AI, Mô-đun Vision.

## Nội dung repo

| File | Bộ dữ liệu | Mô tả | Đường dẫn|
|---|---|---|---|
| `masterlist.ipynb` | PCRM / Masterlist (171 phôi, 146 nhóm bệnh nhân) | Tabular + Vision (ResNet50 6 kênh, dùng mặt nạ ICM/TE/ZP có sẵn) + Late Fusion | Dữ liệu không công khai nên **không** nằm trong repo này. |
| `embryo.ipynb` | Embryo Dataset (2.542 phôi, 1.415 cặp vợ chồng) | Tabular (36 đặc trưng lâm sàng) + Vision (ResNet50, ảnh RGB không mặt nạ) + Late Fusion | https://data.mendeley.com/datasets/x5s3ky9cpv/1|

Cả hai notebook dùng chung một khung đánh giá:

- **Chia tập theo nhóm bệnh nhân** (`StratifiedGroupKFold`) — không bao giờ chia theo dòng, vì các phôi của cùng một bệnh nhân/cặp vợ chồng chia sẻ hồ sơ lâm sàng, dễ gây rò rỉ dữ liệu nếu chia ngẫu nhiên theo dòng (notebook có thí nghiệm kiểm chứng điều này).
- **Ngưỡng phân loại theo Youden's J**, chọn trên tập validation — không chọn trên chính tập được chấm điểm.
- **Khoảng tin cậy 95%** bằng Group Bootstrap (500 lần lấy mẫu lại theo nhóm bệnh nhân).
- **Multi-modal Fusion** = nối feature vector ảnh (2048-d từ ResNet50) với feature vector
  bảng (16-d) rồi học chung qua MLP end-to-end — không phải lấy trung bình xác suất của hai mô hình riêng.

## Yêu cầu dữ liệu

**`masterlist.ipynb`** cần:
```
MasterlistAug30-2017.xlsx
Images/Images/<File Name>.bmp
GT_ICM/GT_ICM/<File Name> ICM_Mask.bmp
GT_TE/GT_TE/<File Name> TE_Mask.bmp
GT_ZP/GT_ZP/<File Name> ZP_Mask.bmp
```

**`embryo.ipynb`** cần:
```
Embryo dataset_2542.xlsx
Data set splitting/{Non-pregnant, Pregnant}/{Cross-validation, External-validation}/*.jpg
```

## Cách chạy

1. Cài thư viện: `pip install pandas numpy scikit-learn lightgbm xgboost torch torchvision opencv-python matplotlib`
2. Mở notebook tương ứng (Jupyter / Google Colab), sửa biến đường dẫn ở mục 1.
3. Chạy tuần tự từ trên xuống — mỗi notebook có mục lục markdown giải thích từng bước.
4. Bảng kết quả cuối cùng (gộp cả 3 nhánh) được in ra ở mục cuối và lưu ra file `.csv`.

**Yêu cầu phần cứng:** nhánh Tabular chạy CPU bình thường; nhánh Vision/Fusion cần GPU (đã test trên Colab, NVIDIA T4/A30) để huấn luyện trong thời gian hợp lý.

## Kết quả chính

**PCRM / Masterlist Data** (External, 5-fold theo nhóm):

| Nhánh | AUC-ROC | Sensitivity | Specificity |
|---|---|---|---|
| XGBoost (Tabular) | 0,619 | 0,677 | 0,440 |
| ResNet50 6 kênh (Vision) | 0,577 | 0,927 | 0,253 |
| Late Fusion | 0,602 | 0,510 | 0,720 |

**Embryo Data** (tập giữ riêng External, 486 phôi):

| Nhánh | AUC-ROC | Sensitivity | Specificity |
|---|---|---|---|
| LightGBM / XGBoost (Tabular) | 0,869 | 0,70–0,79 | 0,77–0,84 |
| ResNet50 (Vision) | 0,601 | 0,41 | 0,71 |
| Late Fusion | 0,524 | 0,15 | 0,85 |