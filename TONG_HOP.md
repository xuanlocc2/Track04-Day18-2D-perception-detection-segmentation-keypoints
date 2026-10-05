# Tổng hợp bài lab 2D Perception

Repo dự kiến nộp: https://github.com/xuanlocc2/Track04-Day18-2D-perception-detection-segmentation-keypoints

Các số liệu dưới đây được đọc từ output notebook chạy trên Colab của bài này, không lấy từ kết quả tham khảo của giảng viên. Gói nộp chỉ hoàn tất khi chạy ô đóng gói cuối và lưu output.

## Phần 1 — Detection và NMS

- Ảnh 640×640 có 8.400 vị trí dự đoán; ảnh letterbox 640×480 có 6.300 vị trí, từ ba stride 8, 16, 32. Vị trí dự đoán không phải số người.
- Nhãn tâm `(cx, cy, w, h)` bị đọc thành góc trên trái `(x, y, w, h)` làm box lệch phải `w/2`, xuống `h/2`.
- IoU đo giao/hợp. NMS giữ box score cao và loại box cùng class có IoU lớn hơn ngưỡng; bằng ngưỡng vẫn được giữ.
- `batched_nms` dịch tọa độ theo class để box người không loại box xe buýt. Ba hàm đã qua kiểm tra.

Latency PyTorch trên T4, trung bình 30 lần:

| Head | Conf | Preprocess ms | Inference ms | Postprocess ms | Box |
|---|---:|---:|---:|---:|---:|
| One-to-many + NMS | 0.25 | 1.68 | 9.27 | 1.20 | 5 |
| One-to-many + NMS | 0.001 | 1.85 | 10.18 | 1.54 | 203 |
| One-to-one | 0.25 | 1.67 | 9.59 | 0.43 | 5 |
| One-to-one | 0.001 | 1.64 | 9.22 | 0.42 | 204 |

Chênh lệch lớn nhất là hậu xử lý ở conf 0.001: 1,54 so với 0,42 ms. Head one-to-one vẫn cần lọc confidence, nhưng bỏ NMS. Tăng ngưỡng IoU giữ được người đứng sát nhau tốt hơn, đồng thời có thể để lại nhiều box trùng.

## Phần 2 — Segmentation và SAM

- Semantic cho **8 vùng person**, trong khi Mask R-CNN đếm **4 người**; semantic có thể nhập hai người thành một vùng hoặc tách một người thành nhiều vùng. Instance ID trong panoptic giúp phân biệt từng người.
- Trong box xe buýt, semantic gán khoảng **63% pixel là train**, 23% person, 5% road và 3% vegetation. Box còn chứa nền và người, nên đây là phân bố trong box, không phải mọi pixel đều thuộc xe.
- SAM prompt A tách gần toàn người mặc áo sáng: **47.441 pixel**, IoU với mask-box khoảng **0,987**. B chỉ tách một vùng nhỏ gần chân: **2.714 pixel**, IoU khoảng **0,000**.
- Box prompt ít mơ hồ hơn một điểm, nhưng vẫn cần kiểm tra nhãn. Notebook đã xác nhận `autolabel/bus.txt` hợp lệ và polygon đọc lại khớp mask SAM.
- Hungarian ghép mask một-một. IoU giữa hai model đo mức đồng thuận, không phải độ chính xác so với nhãn thật.

## Phần 3 — Keypoint và luật ngã

- Khi bỏ ngưỡng Keypoint R-CNN, các điểm chân ngoài khung vẫn được đặt dọc mép dưới hoặc vùng thân; argmax luôn cho một cực đại nên cần confidence để quyết định có dùng điểm đó hay không.
- Đồ thị của lần chạy này: người 40×80 có OKS dưới 0,5 ở khoảng **7 px** độ lệch chuẩn nhiễu; người 250×500 chưa xuống dưới 0,5 trong khoảng **0–30 px** đã đo.
- Ở ảnh bus gốc, pose model phát hiện **5 người**, nhưng chỉ hai người có vai/hông đủ tin cậy để kết luận đứng (nghiêng khoảng 1°). Ba người còn lại không đủ tin cậy: không kết luận không đồng nghĩa với đứng.
- Ảnh xoay phát hiện **3 người**, góc thân khoảng **90°, 94°, 83°**, đều vượt ngưỡng 60°. Đây là phép thử hình chiếu và độ nhạy với hướng ảnh, chưa chứng minh hệ thống phát hiện ngã trên video.
- Cần tracking, diễn biến chuyển động, trạng thái theo thời gian, confidence và kiểm thử các hoạt động bình thường để giảm báo nhầm.

## Phần 4 — Tiger-pose và lỗi

Môi trường: **Tesla T4**, Ultralytics **8.4.171**; train **40 epoch**, `imgsz=640`, khoảng **5 phút**.

| Metric | Kết quả |
|---|---:|
| Box mAP50–95 | 0.914 |
| Pose mAP50 | 0.995 |
| Pose mAP50–95 | 0.436 |
| OKS trung bình trên 53 ảnh val | 0.726 |
| Không phát hiện được hổ | 0/53 |
| Nghi đảo trái/phải: hoán đổi làm OKS tăng >0.02 | 9/53 |

Pose mAP50 cao nhưng mAP50–95 thấp hơn nhiều cho thấy cần xem độ chính xác vị trí ở ngưỡng nghiêm ngặt, không chỉ một con số mAP50.

Hai kiểu lỗi quan sát để trả lời Q11:

1. **Co vị trí bàn chân về giữa hai chân** trong `Frame_17.jpg` (OKS 0.618), `Frame_18.jpg` (0.628). `right_front_paw` có sai số trung bình cao nhất, khoảng 0,26 lần căn diện tích object. Hướng sửa: thêm nhiều pha bước, kiểm tra nhãn đầu bàn chân và giữ đủ chi tiết ảnh.
2. **Chân phía xa bị che/chồng lên chân gần**, vị trí hock/wrist bị kéo về chân nhìn rõ trong `Frame_67.jpg` (0.621), `Frame_31.jpg` (0.633). Hướng sửa: thêm góc chụp/hướng quay và ảnh chân tách rõ, kiểm tra nhãn trái/phải, đánh giá riêng từng cặp. Không kết luận tất cả các ảnh đó đều đảo nhãn chỉ từ thống kê 9/53.

## Bonus 4C

| Model | Val gốc: Pose mAP50–95 | Val lật: Pose mAP50–95 | Mức giảm |
|---|---:|---:|---:|
| Flip giải phẫu | 0.436 | 0.425 | 0.011 |
| Flip đồng nhất | 0.415 | 0.275 | 0.140 |

Val gốc chỉ có hổ quay phải nên che một phần vấn đề trái/phải. Model đồng nhất tụt mạnh hơn trên ảnh lật. Val lật vẫn xuất phát từ val cũ; cần thêm tập kiểm thử độc lập nhiều hướng để kết luận rộng hơn.

## Bonus ONNX CPU

ONNX Runtime CPU, FP32, ảnh vào 640×640, 2 threads, warm-up 5 lần và đo 30 lần; không tính đọc ảnh từ đĩa hay export.

| Head | Conf | Inference ms | Postprocess ms | Pipeline ms | P95 pipeline ms | Box |
|---|---:|---:|---:|---:|---:|---:|
| One-to-many + NMS | 0.25 | 132.509 | 11.732 | 153.225 | 227.185 | 5 |
| One-to-many + NMS | 0.001 | 87.187 | 6.569 | 98.584 | 116.584 | 186 |
| One-to-one | 0.25 | 75.357 | 0.381 | 79.846 | 97.219 | 5 |
| One-to-one | 0.001 | 99.950 | 0.490 | 105.261 | 186.043 | 177 |

NMS-free giảm hậu xử lý ở cả hai mức conf, nhưng **không nhanh hơn về tổng pipeline ở conf 0.001 trong lần đo này**. Inference và P95 dao động khá nhiều trên CPU chia sẻ của Colab; không nên khẳng định NMS-free luôn nhanh hơn hay so trực tiếp với bảng GPU. Cần lặp phép đo, đổi thứ tự cấu hình hoặc xen kẽ chúng, và dùng nhiều ảnh khi đánh giá triển khai thực tế.

Bonus AP đã qua bộ kiểm tra; đường PR và giá trị AP nằm trong notebook.

## Khi nộp

Repo cần notebook còn output, `submission/ket_qua.json`, `submission/autolabel/bus.txt`, `submission/report.md` chứa link notebook, cùng báo cáo bonus. Không sửa tay trạng thái trong JSON. Chỉ nộp URL repo public vào VLearn và giữ public đến khi có điểm. Cần kiểm tra lại một lần `Restart session and run all` với bản đã bổ sung Q11 để chứng minh khả năng tái lập.
