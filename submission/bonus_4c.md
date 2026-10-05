# Bonus 4C: val gốc và val lật gương

Metric: Pose mAP50-95

- flip_idx giải phẫu: val gốc 0.4357, val lật 0.4253, mức giảm 0.0104.
- flip_idx đồng nhất: val gốc 0.4147, val lật 0.2751, mức giảm 0.1396.

Val gốc chỉ có hổ quay phải nên metric tổng hợp trên tập này có thể che lỗi đặt tên trái/phải khi quay trái; phép so sánh val lật kiểm tra thêm tình huống đó. Nếu bảng không cho thấy model đồng nhất tụt nhiều hơn, chưa thể kết luận đã chứng minh lỗi từ mAP: cần kiểm tra từng cặp keypoint, OKS trước/sau hoán đổi và nhiều seed; sigma đồng nhất và mAP tổng hợp có thể làm lỗi khó thấy. Val lật cũng là ảnh biến đổi từ val gốc, chưa thay thế tập kiểm thử độc lập có nhiều hướng, che khuất và góc chụp.