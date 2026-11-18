# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

## Báo cáo phân tích ca lỗi

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11` (run ID: `seeded_regression-C-002-11`) |
| Triệu chứng | Lab 8 grader `FAIL` (`status resolved != denied`); Grader `RefundAmountCorrect` thất bại (ledger ghi 20.000đ nhưng label yêu cầu 0đ); Reply công bố *"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202"* trong khi đơn không đủ điều kiện bồi thường. |
| Bước sai đầu tiên | **Event 3: `model_override`** với payload `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) khởi tạo đúng contract. Event 2 (`investigator`) tra cứu DB chính xác: đơn hàng O-202 có `delay_days: 2` (dưới ngưỡng tối thiểu 3 ngày để được hoàn ship). Bước 3 là nơi agent bắt đầu sai lầm: agent tin vào văn bản ticket của khách (*"Đơn giao trễ 5 ngày..."*) và ghi đè sự thật DB thành `delay_days: 5`, trực tiếp khiến bước 4 (`quote`) tính sai số tiền thành 20.000đ và bước 6 phát lệnh hoàn tiền sai trái. |
| Failure mode | **`FM-03`** (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Cùng claim `C-002@11` trên shape `fixed`: Sau Event 2 (`investigator`), agent giữ nguyên dữ liệu DB và chuyển thẳng sang Event 3 (`quote`) ra kết quả `amount_vnd: 0, reason: "no_entitlement"`, **hoàn toàn không có event `model_override`**. Kết quả status là `denied`, ledger ghi nhận 0đ, khớp 100% với nhãn kỳ vọng. |
| Regression test | `tests/test_lab9_graders.py::test_override_event_breaks_db_fact` với Grader `DbFactKept`. Test kiểm tra bắt buộc không có event `model_override`; bắt trúng lỗi trên `seeded_regression` (`value=False`) và pass trên `fixed` (`value=True`). Ngoài ra Grader `RefundAmountCorrect` bắt được ở mức outcome. |
| Chưa chắc chắn | Trace case này bắt được vì hệ thống có ghi log event `model_override`. Nếu agent gặp lỗi `quiet_override` (ghi đè trực tiếp trong bộ nhớ nhưng che giấu event khỏi trace log) thì `DbFactKept` tại mức bước sẽ không phát hiện được, mà phải phụ thuộc hoàn toàn vào code grader mức outcome `RefundAmountCorrect`. |
