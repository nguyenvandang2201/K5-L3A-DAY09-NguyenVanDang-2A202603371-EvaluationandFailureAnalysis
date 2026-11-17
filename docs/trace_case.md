# Trace case: dữ kiện ticket ghi đè DB

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, report `E-seeded_regression-20261117-affa` |
| Triệu chứng | `pass_rate`, `refund_amount_correct` và `db_fact_kept` fail; reply nói đã hoàn 20.000đ, ledger cũng ghi 20.000đ trong khi nhãn đúng là 0đ/`denied`. |
| Bước sai đầu tiên | Event 3 `model_override`: đổi `delay_days` thành 5. |
| Vì sao là bước đầu tiên | Event 1 chỉ tạo handoff đúng phạm vi. Event 2 `investigator.lookup_order` đọc DB đúng: đơn O-202 ở trạng thái delivered và `delay_days=0`. Sai lệch xuất hiện lần đầu khi event 3 lấy “5 ngày” từ ticket để đè DB. |
| Failure mode | FM-03 `ticket_trust` |
| Đối chứng | `fixed C-002@11`: không có `model_override`; event 3 quote 0 với `no_entitlement`, event 4 trả lời từ chối và ledger giữ 0đ. |
| Regression test | `tests/test_lab9_graders.py::test_override_event_breaks_db_fact` bắt event override; end-to-end gate trên `seeded_regression` chặn `pass_rate` với Δ −0.271, CI 95% [−0.500, −0.057]. |
| Chưa chắc chắn | MockLLM và một fixture không chứng minh tần suất lỗi trên traffic thật; trace chỉ chứng minh quan hệ nhân quả trong trial này. |

Lệnh tái hiện:

```powershell
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11
```

Quy tắc điều tra là đọc ledger và sự kiện tool trước reply. Bước sai đầu tiên phải là event cụ thể; triệu chứng cuối cùng không được dùng thay cho nguyên nhân.
