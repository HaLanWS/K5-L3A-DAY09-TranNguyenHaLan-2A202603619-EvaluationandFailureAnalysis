# Judge audit

Chạy kiểm chuẩn: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | DÙNG |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375 < 0.6, R=0.300 < 0.7) | LOẠI |

## ≥5 case audit thủ công

### Chi tiết bất đồng `tone_ok` (n=100: TP=24, FP=4, FN=6, TN=66)
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Câu trả lời xưng hô "Em ... anh" không đúng quy chuẩn giọng văn dịch vụ khách hàng chuyên nghiệp của LogiX. |
| 2 | H-017 | PASS | FAIL | Người đúng | Dùng từ ngữ quá suồng sã ("bạn ơi ... nha") vi phạm tone giọng trang trọng bắt buộc. |
| 3 | H-029 | PASS | FAIL | Người đúng | Thái độ cộc lốc, mệnh lệnh ("không cần nhắn thêm") gây ức chế cho khách hàng. |
| 4 | H-042 | FAIL | PASS | Judge sai | Trả lời đầy đủ, lịch sự và nêu rõ mốc thời gian 3–5 ngày, không có lỗi ngữ nghĩa hay thái độ. |
| 5 | H-051 | FAIL | PASS | Judge sai | Sử dụng từ mượn thông dụng ("refund") nhưng vẫn giữ thái độ phục vụ lịch thiệp, dễ hiểu. |

### Chi tiết bất đồng `helpful` (n=100: TP=9, FP=0, FN=21, TN=70)
Judge có độ nhạy (Recall) cực thấp (R=0.300), bỏ lọt tới 70% các phản hồi vô bổ hoặc thiếu thông tin thực tế.
- **HL-015**: Người gắn nhãn FAIL (khách khiếu nại nhưng agent trả lời generic), Judge đánh giá PASS.
- **HL-016**: Agent chỉ xin lỗi suông mà không đưa ra hướng dẫn tiếp theo, Người gắn FAIL, Judge gắn PASS.
- **HL-020**: Hoàn thiếu thông tin tra cứu mã giao dịch, Người gắn FAIL, Judge gắn PASS.
- **HL-028**: Xin lỗi chung chung, không giải quyết vấn đề, Người gắn FAIL, Judge gắn PASS.
- **HL-035**: Lặp lại câu chào xin lỗi không có giải pháp cụ thể, Người gắn FAIL, Judge gắn PASS.

## Kết luận
- **Tiêu chí bị loại**: `helpful` bị loại hoàn toàn khỏi bộ đo blocking vì chỉ số Cohen's Kappa chỉ đạt 0.375 (dưới ngưỡng 0.6) và Recall chỉ đạt 0.3 (bỏ lọt 21/30 ca lỗi theo ma trận nhầm lẫn: TP=9, FP=0, FN=21, TN=70). Tỷ lệ lỗi quan sát thấy (0.09) sau khi hiệu chỉnh Rogan-Gladen là 0.30.
- **Tiêu chí được giữ**: `tone_ok` đạt chuẩn với κ=0.757, Precision 85.7%, Recall 80.0%, đảm bảo độ tin cậy để giám sát chất lượng hội thoại.
- **Rủi ro còn lại**: Bộ dữ liệu judge chỉ được kiểm nghiệm trên tập replay public đã biết trước; khi đưa vào môi trường thực tế cần thu thập nhãn đối chứng ngẫu nhiên định kỳ để tránh overfit và trôi dạt phân phối (distribution drift).
