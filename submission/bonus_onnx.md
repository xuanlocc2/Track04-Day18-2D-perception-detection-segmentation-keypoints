# Bonus: ONNX và latency CPU

{"python": "3.12.13", "platform": "Linux-6.6.122+-x86_64-with-glibc2.35", "processor": "x86_64", "onnxruntime": "1.30.0", "provider": "CPUExecutionProvider", "threads": 2, "warmup": 5, "repeats": 30, "imgsz": 640, "precision": "FP32", "iou": 0.7}

| head | conf | preprocess ms | inference ms | postprocess ms | pipeline ms | box |
|---|---:|---:|---:|---:|---:|---:|
| one_to_many | 0.25 | 8.984 | 132.509 | 11.732 | 153.225 | 5 |
| one_to_many | 0.001 | 4.828 | 87.187 | 6.569 | 98.584 | 186 |
| one_to_one | 0.25 | 4.108 | 75.357 | 0.381 | 79.846 | 5 |
| one_to_one | 0.001 | 4.82 | 99.95 | 0.49 | 105.261 | 177 |

Conf 0.25: pipeline one-to-many 153.225 ms, one-to-one 79.846 ms; chênh lệch (many − one) 73.379 ms.

Conf 0.001: pipeline one-to-many 98.584 ms, one-to-one 105.261 ms; chênh lệch (many − one) -6.677 ms.

NMS-free bỏ bước loại box trùng nhưng vẫn có lọc confidence; tốc độ tổng phụ thuộc cả graph và CPU. Đo trên một ảnh, không đo đọc ảnh từ đĩa, camera hoặc tạo Results, nên chưa đại diện mọi cảnh và chưa đánh giá chênh lệch độ chính xác hai head.