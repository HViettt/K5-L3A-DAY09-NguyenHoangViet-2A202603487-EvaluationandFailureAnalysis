# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Case thực tế: `seeded_regression C-002@11` (Đối chứng với `fixed C-002@11`)

| Trường | Nội dung |
|---|---|
| **Trial** | `seeded_regression C-002@11` (run ID: `seeded_regression-C-002-s11`, report `E-seeded_regression-20261117-d421`) |
| **Triệu chứng** | - `TrialPass` (`pass_rate`): **FAIL** (`status resolved != denied`)<br>- `RefundAmountCorrect` (`refund_amount_correct`): **FAIL** (`ledger=20000 != expected=0`)<br>- `DbFactKept` (`db_fact_kept`): **FAIL** (`1 override event(s)`)<br>- **Ledger vs Reply:** Ledger ghi `refund_vnd = 20000`, `refund_count = 1`; event `reply` nói `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."` trong khi nhãn đúng là `status = "denied"`, `refund_vnd = 0`, `refund_count = 0`. |
| **Bước sai đầu tiên** | **Step 3 · `model_override` · `{"delay_days": 5}`** |
| **Vì sao là bước *đầu tiên*** | - **Step 1 (`handoff`)** đúng: chuyển đúng `claim_id="C-002"`, `allowed_tools=["lookup_order", "lookup_policy", "compute_compensation"]` và `facts={}` rỗng.<br>- **Step 2 (`investigator`)** đúng: đọc DB chuẩn xác cho đơn `O-202` (`"delay_days": 2`, `"shipping_fee_vnd": 20000`, `"status": "delivered"`, `policy_count: 1`). Với `delay_days = 2 < 3`, đơn không đủ điều kiện hoàn phí ship.<br>- Sang **Step 3 (`model_override`)**, agent tin lời khai trong ticket (*"Đơn giao trễ 5 ngày"*) và ghi đè `order["delay_days"] = 5` đè lên sự thật từ DB (`delay_days = 2`). Từ bước sai này mới dẫn tới Step 4 (`quote` tính sai thành `amount_vnd = 20000`) và Step 5 (`reply` + hoàn tiền sai `20.000đ` vào ledger). |
| **Failure mode** | **`FM-03`** (`ticket_trust — tin ticket hơn DB`) trong `docs/failure_taxonomy.md` |
| **Đối chứng** | Chạy cùng `C-002@11` trên shape `fixed` (`python -m lab9.trace_case --shape fixed --claim C-002 --seed 11`):<br>- Step 1 (`handoff`) và Step 2 (`investigator`) giống hệt.<br>- **Không có Step `model_override`**: shape `fixed` giữ nguyên `delay_days = 2` của DB.<br>- Step 3 (`quote`) trả về `{"amount_vnd": 0, "reason": "not_eligible", "clause_ids": [], "parts": {}}`.<br>- Step 4 (`reply`) trả về `"Dạ, đơn O-202 chưa đủ điều kiện hoàn theo chính sách hiện hành."`, `status = "denied"`, `ledger.refund_vnd = 0` $\rightarrow$ **PASS**. |
| **Regression test** | - Test hàm: `test_trace_case_c002_seed11_regression` trong `src/tests/test_lab9_graders.py` (kiểm tra cả mức bước bằng `DbFactKept` và mức kết quả ledger bằng `RefundAmountCorrect` trên `C-002@11`).<br>- Đã xác nhận: **FAIL** (`value=False`) trên `seeded_regression` và **PASS** (`value=True`) trên `fixed`. |
| **Chưa chắc chắn** | Trace chỉ ghi nhận sự kiện `model_override: {"delay_days": 5}` mà không lưu toàn bộ prompt/response thô của bước `parse_delay_days`; nếu một biến thể ẩn (`quiet_override`) ghi đè `delay_days` mà không phát sự kiện `model_override` vào trace, grader mức bước (`DbFactKept`) sẽ không thấy, nên bắt buộc phải kết hợp với grader mức outcome trên ledger (`RefundAmountCorrect`). |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
