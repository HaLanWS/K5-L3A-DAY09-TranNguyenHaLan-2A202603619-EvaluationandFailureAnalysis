# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Chi tiết Case Audit: seeded_regression C-002@11

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, Run ID: `E-seeded_regression-20261118-e051` |
| Triệu chứng | Grader Lab 8 FAIL (`status resolved != denied`). Ledger ghi `refund_vnd=20000` (nhãn=0), reply báo `LogiX đã hoàn 20.000đ cho đơn O-202` dù đơn không đủ điều kiện hoàn tiền |
| Bước sai đầu tiên | Event số 3: `model_override` `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Bước 1 (`handoff`) và Bước 2 (`investigator`) hoàn toàn đúng: điều tra DB trả về `order.delay_days = 0`. Tuy nhiên sang Bước 3, agent tin lời khách khai trong ticket ("giao trễ 5 ngày") và sinh event `model_override` đè `delay_days=5`, từ đó kéo theo Bước 4 (`quote`) tính sai số tiền thành 20.000đ |
| Failure mode | `FM-03` (ticket_trust — tin ticket hơn DB) dẫn tới `FM-02` (ignore_denied) |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: Bước 2 trả về `delay_days=0`, agent giữ nguyên không override, Bước 3 `quote` trả về `amount_vnd: 0, reason: no_entitlement`, Bước 4 reply đúng: `Dạ, đơn O-202 chưa đủ điều kiện hoàn theo chính sách hiện hành.`, ledger không phát sinh hoàn tiền |
| Regression test | `tests/test_lab9_graders.py::test_override_event_breaks_db_fact` (bắt FM-03 qua `DbFactKept`) và `RefundAmountCorrect`. Trên `seeded_regression`, `DbFactKept` phát hiện 1 override event và `RefundAmountCorrect` báo lỗi ledger 20000 != 0. Trên `fixed`, tất cả grader PASS. |
| Chưa chắc chắn | Chưa thể xác định người dùng trong thực tế cố ý gian lận hay do nhớ nhầm ngày nhận hàng nếu chỉ nhìn vào văn bản ticket đơn lẻ. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: kiểm thử trong `tests/` hoặc mục dữ liệu trong eval set với provenance rõ ràng.
