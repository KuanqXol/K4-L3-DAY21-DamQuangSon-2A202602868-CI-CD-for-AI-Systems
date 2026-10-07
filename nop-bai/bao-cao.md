# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Đàm Quang Sơn |
| MSSV | 2A2002602868 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/KuanqXol/K4-L3-DAY21-DamQuangSon-2A2002602868-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |
| 4 | 150 | 0.1 | 4 | 0.7156 | 0.8760 |

**Bộ siêu tham số đã chọn:** `n_estimators=150`, `learning_rate=0.1`, `max_depth=4`.

**Lý do:** Cấu hình này đạt F1-score cao nhất (0.7156) trên tập kiểm thử holdout và vượt qua ngưỡng chất lượng 0.65 của hệ thống. Đáng chú ý, lần chạy 1 có accuracy cao nhất (0.8780) nhưng F1 chỉ đạt 0.7109, cho thấy mô hình thiên vị lớp đa số và bỏ sót nhiều cá nhân thu nhập cao. Thực nghiệm cũng phản ánh sự đánh đổi rõ rệt: khi tăng n_estimators lên 200 và max_depth lên 5, F1 không tăng thêm mà giảm nhẹ về 0.7149 do hiện tượng quá khớp (overfitting) cục bộ. Do đó, bộ siêu tham số thứ tư là điểm cân bằng tối ưu giữa năng lực dự đoán lớp thiểu số và khả năng tổng quát hóa của mô hình.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Census Income mất cân bằng lớp rõ rệt: nhóm thu nhập cao (>50K) chỉ chiếm khoảng 24.8%, trong khi nhóm thu nhập thấp chiếm 75.2%. Nếu mô hình dự đoán tất cả là thu nhập thấp, nó vẫn đạt Accuracy 75.2% nhưng hoàn toàn vô dụng vì không phát hiện được bất kỳ ai thuộc nhóm mục tiêu. Do đó, Accuracy mang lại cảm giác ảo tưởng về chất lượng. F1-score trên lớp dương (trung bình điều hòa giữa Precision và Recall) đo lường chính xác năng lực phát hiện và độ tin cậy đối với nhóm thu nhập cao. Việc tính trực tiếp F1 cho lớp dương thay vì dùng `average="weighted"` (bị chi phối bởi lớp đa số) hay `average="macro"` (bình quân hóa ngang hàng hai lớp) giúp thước đo bám sát mục tiêu nghiệp vụ cốt lõi.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi import `FallbackAsyncAdaptedQueuePool` khi kích hoạt MLflow tracking. | Phiên bản `mlflow==2.13.0` xung đột với bản phát hành mới của SQLAlchemy 2.1+. | Cố định `sqlalchemy==2.0.30` trong `requirements.txt` để đảm bảo tính tương thích. |
| Job Release thất bại khi nạp mô hình trên Cloud VM. | Khác biệt phiên bản scikit-learn giữa môi trường train và serve gây lỗi giải mã cấu trúc `CyHalfBinomialLoss`. | Cài đặt cố định `scikit-learn==1.4.2` trên máy chủ suy luận đồng nhất với môi trường CI. |
| Cấu hình xác thực DVC với Google Cloud Storage trên GitHub Actions runner. | Đường dẫn tệp khóa Service Account cục bộ không được đưa vào Git để tránh lộ bảo mật. | Sử dụng GitHub Secret lưu khóa JSON, ghi ra tệp tạm trên runner và cấu hình DVC remote qua cờ `--local`. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7156 | 0.8760 |
| Bước 3 (thêm `train_batch2`) | 0.7248 | 0.8800 |

**Nhận xét:** Khi bổ sung thêm 22,361 mẫu từ `train_batch2`, kích thước tập huấn luyện tăng gấp đôi lên 44,722 mẫu giúp thuật toán học thêm được nhiều không gian đặc trưng đa dạng. Nhờ đó, cả F1-score (tăng từ 0.7156 lên 0.7248) và Accuracy (tăng từ 0.8760 lên 0.8800) trên tập holdout đều cải thiện rõ rệt, chứng minh tính hiệu quả của quy trình Continuous Training tự động khi có luồng dữ liệu mới.
