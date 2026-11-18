# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (0.757 ≥ 0.6, 0.857 ≥ 0.8, 0.800 ≥ 0.7) | DÙNG |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375 < 0.6, R=0.300 < 0.7) | LOẠI |

## ≥5 case audit thủ công (trích từ bất đồng của judge_tone và judge_helpful)

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | `H-008` (tone) | PASS | FAIL | Người đúng | Câu văn xưng hô suồng sã: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em"*. Judge bị đánh lừa bởi cảm giác thân mật. **Sửa rubric:** Cấm đại từ xưng hô tùy tiện (anh/em, nha), bắt buộc dùng ngôi chuẩn LogiX ("anh/chị", "Dạ"). |
| 2 | `H-029` (tone) | PASS | FAIL | Người đúng | Câu văn cộc lốc mang tính đuổi khách: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm"*. Judge chỉ thấy cụm "Dạ, vui lòng chờ". **Sửa rubric:** Thêm quy tắc cấm thái độ gạt phăng hoặc yêu cầu khách "không cần nhắn thêm". |
| 3 | `H-042` (tone) | FAIL | PASS | Người đúng | *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ"*. Câu trả lời đầy đủ, lịch sự. Judge phạt sai vì câu dài hơn template. **Sửa rubric:** Cho phép thông báo khung thời gian dự kiến (3–5 ngày làm việc) mà không bị tính lỗi tone. |
| 4 | `H-070` (tone) | PASS | FAIL | Người đúng | *"Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂"*. Chứa emoji cười cợt nhả không phù hợp giao tiếp doanh nghiệp. Judge bỏ qua emoji. **Sửa rubric:** Cấm tuyệt đối emoji suồng sã, thiếu nghiêm túc trong văn bản hỗ trợ tài chính. |
| 5 | `HL-016` (helpful) | PASS | FAIL | Người đúng | Khách khiếu nại nhưng agent chỉ xin lỗi sáo rỗng: *"Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này..."* mà không hề đưa ra giải pháp hay hướng dẫn tiếp theo. **Sửa rubric:** Lời xin lỗi sáo rỗng không đi kèm hành động/thông tin cụ thể phải bị đánh dấu FAIL tính hữu ích. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` bị loại hoàn toàn khỏi release gate.
  - **Lý do & Ma trận nhầm lẫn:** `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`. Độ nhạy (Recall) chỉ đạt $30\%$ ($R=0.300$), bỏ sót tới 21 lỗi nghiêm trọng ($FN=21$). Judge bị hiệu ứng thiên vị độ trôi chảy (fluency/politeness bias): cứ thấy câu chữ xin lỗi lịch sự là tự động cho PASS, dù không giúp ích gì cho khách hàng. Hệ số Cohen's $\kappa = 0.375$ rất thấp, không thể tin cậy làm cổng chặn.
- **Tiêu chí được giữ lại:** `tone_ok` được giữ lại vì vượt qua toàn bộ ngưỡng FLOORS ($\kappa = 0.757 \ge 0.6$, $P = 0.857 \ge 0.8$, $R = 0.800 \ge 0.7$).
- **Rủi ro còn lại:** Judge tone mới chỉ được kiểm định trên tập dữ liệu replay công khai; trong môi trường sản xuất thực tế, người dùng có thể sử dụng các biến thể từ lóng, teencode mới mà judge chưa từng gặp. Cần định kỳ gắn nhãn mẫu người thật (human labeled held-out set) để hiệu chỉnh lại prompt của judge.
