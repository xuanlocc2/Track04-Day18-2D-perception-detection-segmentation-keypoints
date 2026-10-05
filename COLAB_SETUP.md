# Chuẩn bị notebook và môi trường Colab

## 1. Tạo bản notebook cá nhân

Mở [notebook chính thức trên Colab](https://colab.research.google.com/github/VinUni-AI20k/Track04-Day18-2D-perception-detection-segmentation-keypoints/blob/main/lab_2d_perception_student.ipynb), chọn **File → Save a copy in Drive** và làm trên bản sao.

Nếu link không mở được, vào https://colab.research.google.com, chọn **File → Upload notebook** và tải lên `lab_2d_perception_student.ipynb` trong thư mục này. Lưu notebook vào Drive trước khi làm bài.

Chọn **Runtime → Change runtime type → T4 GPU → Save**. Metadata của notebook đã đặt GPU T4, nhưng bạn vẫn cần kiểm tra GPU thực tế được Colab cấp.

## 2. Chạy Phần 0 — Setup (10 phút)

Chạy lần lượt năm ô code bằng **Shift + Enter**:

1. Cài thư viện: `%pip install -q "ultralytics==8.4.171"`.
2. Import và kiểm tra thiết bị.
3. Định nghĩa hàm tiện ích và bộ kiểm tra.
4. Tải weights và tiger-pose, tổng khoảng 0,8 GB.
5. Hiển thị hai ảnh mẫu.

Chỉ chuyển sang Phần 1 khi output có đủ:

- `device = cuda`.
- Dòng `🔧 Sẵn sàng: ...`.
- Chín dấu `✓`: ba weights torchvision, năm weights Ultralytics, một dataset.
- Hai ảnh `bus.jpg` và `zidane.jpg`.

Nếu `device = cpu`, chọn lại T4 và chạy lại từ đầu. CPU chỉ chạy fine-tune 3 epoch, không đáp ứng yêu cầu nộp. Nếu tải tài nguyên lỗi mạng, chạy lại ô tải; các file tải xong được giữ lại. Nếu Colab yêu cầu restart sau cài thư viện, restart rồi chạy lại từ ô import.

Không cần API key, cài môi trường Python trên máy cá nhân hay tải dataset thủ công.

## 3. Làm bài theo thứ tự

| Phần | Thời gian | Công việc |
|---|---:|---|
| 0 | 10 phút | Setup và kiểm tra output |
| 1 | 35 phút | Detection, IoU, NMS, latency |
| 2 | 30 phút | Segmentation, polygon, SAM 2.1 auto-label |
| 3 | 25 phút | Keypoints, OKS, góc khớp |
| 4 | 20 phút | Sửa `FLIP_IDX`, fine-tune và phân tích lỗi |

Điền các TODO và Q1–Q12 trong notebook. Các phần sau dùng lại hàm của phần trước. Mục ⭐ là tùy chọn. Sau khi sửa hàm, chạy lại chính ô TODO để định nghĩa mới có hiệu lực. Đối chiếu [README](README.md) và [rubric](rubric.md).

Fine-tune bắt buộc ở 4B: GPU, **40 epoch**, **imgsz 640**. Nếu hết bộ nhớ GPU, restart và chạy lại; nếu vẫn lỗi, giảm `batch=16` xuống `batch=8` theo README. Q11 cần mô tả hai kiểu lỗi quan sát được trên ảnh val của chính lần chạy, kèm cách khắc phục cụ thể.

## 4. Lưu kết quả và link notebook

Sau khi hoàn thành:

1. Chọn **Runtime → Restart session and run all**; chờ chạy hết và kiểm tra không ô nào lỗi.
2. Ô cuối `final_report()` tạo `submission/ket_qua.json`, `submission/autolabel/bus.txt` và `submission.zip`. Mọi dòng bắt buộc trong checklist cần là ✅.
3. Tải notebook đã chạy và `submission.zip` về máy, giải nén ZIP.
4. Sau khi đưa notebook lên repo GitHub public của bạn, mở trang file notebook và sao chép URL (không dùng link repo giảng viên).
5. Tạo `submission/report.md`, thêm dòng `Link notebook đã chạy: <URL trang notebook trong repo của bạn>`, rồi đưa báo cáo lên repo. Không sửa `ket_qua.json`.

Ô thêm cuối notebook cũng có thể tạo `report.md` và cập nhật ZIP nếu đã biết URL GitHub đích: điền `NOTEBOOK_URL`, chạy ô rồi tải lại ZIP. Nếu chưa biết URL, làm thủ công sau khi upload như trên.

Bonus đã chọn: AP 1D, 4C với cả hai model, ONNX CPU. Chạy tất cả các ô từ trên xuống. Các báo cáo `bonus_4c.md`, `bonus_onnx.md`, `onnx_latency.json` được tạo trong `submission/` và đưa vào ZIP bởi ô cuối. Q11 cần xem hình và điền trước lần Restart & Run All cuối cùng.

## 5. Nộp bài

Tải notebook bằng **File → Download → Download .ipynb**, giữ nguyên output. Tải `submission.zip` từ bảng Files ở bên trái Colab và giải nén. Đưa notebook đã chạy và thư mục `submission/` lên repo GitHub **public** của bạn (fork hoặc repo mới).

```text
lab_2d_perception_student.ipynb   # Đã chạy, còn nguyên output
submission/
    ket_qua.json
    report.md                  # Có link notebook đã chạy
    autolabel/
        bus.txt
```

Chỉ nộp **URL repo GitHub public vào VLearn**. Không mở PR vào repo giảng viên. Giữ repo public cho đến khi có điểm. Các file trong `submission/` được tạo sau khi chạy bài; bản chuẩn bị này chưa có kết quả thực nghiệm.

Nguồn: [repo giảng viên, nhánh main](https://github.com/VinUni-AI20k/Track04-Day18-2D-perception-detection-segmentation-keypoints).
