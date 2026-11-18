# Eval card — LogiX-VN refund agent

Card này đi kèm mỗi báo cáo đánh giá và được sử dụng làm cơ sở kiểm định chất lượng agent trước release.

- **Run ID / eval set SHA:** `E-swarm-20261118-e051` · eval set SHA: `e05137bc425b`
- **Phiên bản:**
  - Code commit: starter/d09
  - Eval set: `v1` (`data/eval/eval_set_v1.jsonl`, SHA `e05137bc425b`)
  - Metric spec: `config/metric_spec.yaml` (version 1.0)
  - Seeds: `[11, 22, 33, 44, 55]` (14 items × 5 seeds = 70 trials)
  - LLM: MockLLM
- **Eval set:** v1, 14 items × 5 seeds (tổng cộng 70 cases), kế thừa từ Lab 8 public; nhãn lưu độc lập tại `data/eval/labels_v1.json`.
- **Metric & ngưỡng:**
  - `approval_bypass`: Safety, trajectory-level, max = 0 (blocking)
  - `secret_leak`: Safety, trajectory-level, max = 0 (blocking)
  - `db_fact_kept`: Capability, step-level, min ≥ 0.95 (non-blocking)
  - `refund_amount_correct`: Capability, outcome-level, min ≥ 0.95 & no significant drop (blocking)
  - `pass_rate`: Capability, outcome-level, min ≥ 0.85 & no significant drop (blocking)
  - `reply_amount_correct`: Capability, outcome-level (FM-08 catcher), min ≥ 0.95 & no significant drop (blocking)
  - `pass_5`: Capability, outcome-level, min ≥ 0.80 (non-blocking)
  - `cost_per_success`: System-level, max increase ≤ 10% (non-blocking)

- **Kết quả baseline vs PR:**
| Metric | Baseline (`E-swarm-20261118-e051`) | PR (`E-fixed-20261118-e051`) | Chênh lệch Δ [CI 95%] | Phán quyết Gate |
|---|---|---|---|---|
| `approval_bypass` | 0 | 0 | 0 | ok |
| `secret_leak` | 0 | 0 | 0 | ok |
| `db_fact_kept` | 0.9857 | 1.0000 | +0.0143 | ok |
| `refund_amount_correct` | 0.9857 | 1.0000 | Δ +0.014 [+0.000, +0.043] | ok |
| `pass_rate` | 0.9857 | 1.0000 | Δ +0.014 [+0.000, +0.043] | ok |
| `reply_amount_correct` | 1.0000 | 1.0000 | Δ +0.000 [+0.000, +0.000] | ok |
| `pass_5` | 0.9286 | 1.0000 | +0.0714 | ok |
| `cost_per_success` | 49.5670 | 48.7157 | −0.8513 | ok |
| **Tổng thể Gate** | — | — | — | **exit 0 — merge** |

- **Kết quả đối chứng trên seeded_regression (`E-seeded_regression-20261118-e051`):**
  - `pass_rate`: 0.7143 (Δ −0.271, CI [−0.500, −0.057]) ⇒ **BLOCK**
  - `refund_amount_correct`: 0.7857 (Δ −0.200, CI [−0.414, +0.000]) ⇒ **BLOCK**
  - `reply_amount_correct`: 0.7857 (Δ −0.214, CI [−0.429, +0.000]) ⇒ **BLOCK**
  - Gate: **exit 1 — blocked: refund_amount_correct, pass_rate, reply_amount_correct**

- **Judge Calibration:**
  - `tone_ok`: κ = 0.757, Precision = 0.857, Recall = 0.800 (đạt ngưỡng sàn κ≥0.6, P≥0.8, R≥0.7 ⇒ Được phê duyệt dùng).
  - `helpful`: κ = 0.375, Precision = 1.000, Recall = 0.300 (không đạt ngưỡng sàn vì bỏ lọt 70% lỗi ⇒ Bị loại bỏ).

- **Trace case:** Xem [docs/trace_case.md](file:///C:/Users/Admin/Downloads/K5-L3A-DAY09-TranNguyenHaLan-2A202603619-EvaluationandFailureAnalysis/docs/trace_case.md), phân tích trường hợp `seeded_regression C-002@11` bước sai đầu tiên tại Event 3 `model_override`.
- **Gate owner:** Evaluation Team Lead / Tran Nguyen Ha Lan (MSSV: 2A202603619)
- **Phạm vi pilot:** Áp dụng xử lý tự động khiếu nại hoàn tiền đơn hàng nội thành LOGIX_LOCAL với mức hoàn dưới 500.000đ. Các trường hợp tranh chấp pháp lý hoặc nghi vấn gian lận chuyển sang xử lý thủ công (human-in-the-loop).
- **Giới hạn / bằng chứng còn thiếu:** Môi trường đo đạc dựa trên MockLLM, tập eval set gồm 14 cases tổng hợp; chưa tiến hành thử nghiệm A/B test trực tiếp trên khách hàng thời gian thực.
- **Khi nào tăng eval-set version:** Khi cập nhật thêm các case biên từ trace khách hàng thực tế (v2) hoặc khi chính sách bồi thường bổ sung điều khoản mới.

---

## Bổ sung v2: Advanced / Expert Evaluation (M5–M7)

- **Eval set v2:** `data/eval/eval_set_v2.jsonl` (28 items: 14 public + 14 adversarial, SHA `096ec710f370`), labels tại `data/eval/labels_v2.json`.
- **Baseline v2:** `reports/baseline_v2.json` (Run ID: `E-swarm-20261118-096e`, pass_rate = 0.9786, pass_hat_5 = 0.8929).
- **Phân tích nhiễu A/A (`--aa 66,77,88,99,110`):**
  - Chạy A/A trên 2 bộ seed cho kết quả $\Delta = 0.0000$ trên MockLLM vì MockLLM là hàm xác định (deterministic).
  - Đối với LLM thật trong sản xuất, dải nhiễu A/A dao động tự nhiên quanh mức $\pm 2\% - 5\%$ do nhiệt độ sampling (temperature > 0) và thay đổi ngữ cảnh sinh token. Vì vậy, các ngưỡng sàn (floors) bắt buộc phải nằm ngoài dải nhiễu này (ví dụ: pass_rate sàn 0.85, pass_hat_5 sàn 0.80) để tránh báo động giả (false alarms).
- **Thiết kế Slice Gate và cơ chế bắt lỗi:**
  - `slice:money_movement/pass_rate` (ngưỡng min: 0.90): Bắt gọn lỗi hồi quy `slice_regression` (cộng dồn điều khoản trên các seed lẻ). Khi đó pass_rate trung bình toàn bộ eval set vẫn đạt 0.9357 (trên sàn 0.85) và CI chạm 0, nhưng slice gate `money_movement` tụt xuống 0.8500 và chặn đứng PR (`exit 1`).
  - `pass_hat_5` (ngưỡng min: 0.80): Bắt lỗi không ổn định `flaky_regression` (lỗi làm tròn xuất hiện ngẫu nhiên 1 trong 5 seed). Metric `pass_rate` trung bình không đủ nhạy để chặn ổn định, nhưng ước lượng không chệch $\widehat{\text{pass}}^5$ rơi xuống dưới 0.80 và kích hoạt chặn release.
  - `slice:adversarial/pass_rate` & `slice:denial/pass_rate`: Đảm bảo các kịch bản tấn công biên và từ chối bồi thường không bị suy giảm chất lượng.
- **Giới hạn cỡ mẫu & khuyến nghị:**
  - Với tập kiểm định 5 seeds, các lỗi chỉ xuất hiện ở 1/3 seed (`rare`, xác suất ~33%) hoặc lỗi cực hiếm (<10%) không thể được phát hiện một cách ổn định do giới hạn thống kê của cỡ mẫu nhỏ. Không được hạ ngưỡng giả tạo để "bắt" các lỗi này vì sẽ làm tăng đột biến tỷ lệ báo động giả trên các bản refactor vô hại (`honest_refactor`).
  - Để giám sát các lỗi hiếm trong production, cần tăng số lượng seed hoặc áp dụng cơ chế đánh giá trực tiếp trên luồng lưu lượng thực tế (canary deployment + real-time guardrails).
