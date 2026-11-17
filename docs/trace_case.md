# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run_id: `run-seeded_regression-C-002-11` |
| Triệu chứng | Grader Lab 8 fail: `status resolved != denied`. Ledger ghi hoàn 20.000đ (nhãn kỳ vọng: 0đ, denied). Reply nói: "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." |
| Bước sai đầu tiên | Event 3 — kind: `model_override` — payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) và Event 2 (`investigator`) hoàn toàn đúng: `lookup_order` tra cứu DB trả về `order_id: O-202` có `delay_days: 0`. Đến Event 3, agent mới bắt đầu ghi đè dữ liệu DB bằng số ngày trong ticket. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Trên `fixed C-002@11`: hoàn toàn không có event `model_override`. Bước 3 là `quote {"amount_vnd": 0}`, status trả về `denied`, ledger hoàn 0đ, khớp 100% với nhãn. |
| Regression test | Grader `DbFactKept` (trong `src/lab9/graders.py`) bắt event `model_override`; kiểm thử qua `test_override_event_breaks_db_fact` trong `src/tests/test_lab9_graders.py`. Đã xác nhận FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi đè giá trị `delay_days: 5`, nhưng không ghi log chuỗi suy luận (chain-of-thought) bên trong của LLM về việc vì sao tin lời khách hàng hơn DB. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
