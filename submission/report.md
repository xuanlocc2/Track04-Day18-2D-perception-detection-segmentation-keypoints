# Báo cáo lab 2D Perception

- Sinh viên: Nguyễn Văn Xuân Lộc
- MSSV: 2A202602870

Link notebook đã chạy: https://github.com/xuanlocc2/Track04-Day18-2D-perception-detection-segmentation-keypoints/blob/main/lab_2d_perception_student.ipynb

## Kết quả bắt buộc

Môi trường: cuda; Ultralytics 8.4.171; GPU: Tesla T4.
Fine-tune: 40 epoch, imgsz 640, thời gian 5.0 phút.
Box mAP50-95: 0.9140; Pose mAP50: 0.9950; Pose mAP50-95: 0.4357.
Val: OKS trung bình 0.726; không phát hiện 0/53; nghi đảo trái/phải 9/53.

## Phân tích lỗi (Q11)

Trong Frame_17.jpg (OKS 0.618) và Frame_18.jpg (0.628), các dự đoán bàn chân trước/sau co về vị trí giữa hai chân, trong khi nhãn thật nằm ở các đầu chân đang bước xa nhau; đây là lỗi định vị và nhập các vị trí chân, phù hợp với sai số right_front_paw lớn nhất (khoảng 0.26 lần căn diện tích object). Tôi sẽ bổ sung ảnh ở nhiều pha bước, kiểm tra nhãn bàn chân và dùng crop/độ phân giải phù hợp để các đầu chân có đủ chi tiết.
Trong Frame_67.jpg (0.621) và Frame_31.jpg (0.633), chân phía xa bị chồng lên chân phía gần hoặc thân, các điểm hock/wrist dự đoán bị kéo về phía chân đang nhìn rõ; sự mơ hồ trái/phải cũng cần kiểm tra vì 9/53 ảnh có OKS tăng hơn 0.02 sau hoán đổi nhãn. Tôi sẽ bổ sung dữ liệu có chân tách rõ và cả hai hướng quay, kiểm tra quy ước trái/phải, rồi đánh giá riêng các cặp keypoint thay vì kết luận mọi trường hợp này đều là đảo nhãn.

## Bonus

AP 1D đã qua kiểm tra; kết quả PR nằm trong notebook.
So sánh hai model 4C: xem bonus_4c.md. Benchmark ONNX CPU: xem bonus_onnx.md và onnx_latency.json.

## Giới hạn

Val gốc chỉ có hổ quay phải; val lật là dữ liệu biến đổi từ cùng tập, không phải tập kiểm thử độc lập. Latency chỉ đo bus.jpg, phụ thuộc CPU/GPU và không bao gồm camera hoặc đọc ảnh từ đĩa. Hai head cần được so sánh cả tốc độ và độ chính xác khi triển khai.
