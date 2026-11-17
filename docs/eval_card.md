# Eval card — LogiX-VN refund agent

## Phiên bản và phạm vi

- Code: commit `bd7ea9c`; MockLLM; Python 3.12.10.
- v1: run `E-swarm-20261117-21e9`, eval SHA `21e91420f3dc`, metric spec 1.0, 14 item × 5 seed.
- v2: run `E-swarm-20261117-1ffb`, eval SHA `1ffbf2775eca`, metric spec 2.0, 28 item × 5 seed.
- Nguồn v1: `lab8_public`; v2 thêm 14 adversarial item. Nhãn nằm riêng trong `labels_v1.json`/`labels_v2.json`.

## Kết quả Core

| Metric | Baseline swarm | `fixed` | `seeded_regression` | Ngưỡng / quyết định |
|---|---:|---:|---:|---|
| `approval_bypass` | 0 | 0 | 0 | max 0, blocking |
| `secret_leak` | 0 | 0 | 0 | max 0, blocking |
| `db_fact_kept` | 0.9857 | 1.0000 | 0.9286 | min 0.95, theo dõi |
| `refund_amount_correct` | 0.9857 | 1.0000 | 0.8571 | min 0.95 + không giảm có ý nghĩa |
| `pass_rate` | 0.9857 | 1.0000 | 0.7143 | min 0.85 + không giảm có ý nghĩa |
| `reply_ledger_consistent` | 1.0000 | 1.0000 | 0.7857 | min 0.95 + không giảm có ý nghĩa |
| `proposal_amount_correct` | 0.9857 | 1.0000 | 0.7857 | min 0.95 + không giảm có ý nghĩa |
| `pass_5` | 0.9286 | 1.0000 | 0.7143 | min 0.80, không blocking |
| `cost_per_success` | 49.5670 | 48.7157 | 81.2020 | cảnh báo nếu tăng >10% |

`fixed` được merge. Với `seeded_regression`, `pass_rate` có Δ −0.271 và CI 95% [−0.500, −0.057], nên gate exit 1. Grader bổ sung giải thích 100% các row regression fail và báo nhầm 0% trên `fixed`.

## Judge

`tone_ok`: κ=0.757, precision=0.857, recall=0.800, đạt floors. `helpful`: κ=0.375, precision=1.000, recall=0.300, bị loại. Chi tiết và hành động sửa rubric ở `docs/judge_audit.md`.

## Advanced / Expert v2

- A/A giữa seeds 11/22/33/44/55 và 66/77/88/99/110 cho Δ=0 trên mọi metric của MockLLM. Floor blocking `pass_hat_5=0.80` nằm dưới baseline 0.8929.
- `slice_regression` có `pass_rate=0.9357`, CI toàn cục chạm 0 nên gate toàn cục không chặn; slice `money_movement=0.85` thấp hơn floor 0.90 và chặn đúng regression.
- Slice baseline: `denial=0.9455`, `money_movement=1.0000`, `adversarial=0.9714`.
- `flaky_regression` bị chặn bởi `pass_rate`, `pass_hat_5`, `proposal_amount_correct` và slice `money_movement`.
- Mutation score 3/3, false alarm 0: `fixed` và `honest_refactor` merge; `seeded_regression`, `slice_regression`, `flaky_regression` block.
- Integrity gate chặn `evaluator_errors_as_nan`, `dropped_failing_rows`, `cherry_picked_seeds`, `loosened_spec`; đồng thời kiểm dòng trùng, grader failure, aggregate bị sửa, tập `(item, seed)` và slice tags.

## Trace, ownership và giới hạn

- Trace case: `docs/trace_case.md`, FM-03 tại event 3; regression test và đối chứng `fixed` đã chạy.
- Gate owner: Nguyễn Văn Đăng. Gate exit 1 được xử lý bằng trace + grader; không waiver bằng cách sửa report hoặc nới ngưỡng.
- Pilot: yêu cầu hoàn phí/bồi thường tiếng Việt trên fixture công khai, một principal và một order/claim theo contract hiện tại.
- Giới hạn: MockLLM không đại diện model thật; v1 chỉ 14 item; judge là replay công khai; chưa có traffic hay nhãn người dùng thật. Một bug xuất hiện 1/3 seed không được phát hiện ổn định với chỉ 5 seed, nên cần tăng sample trước khi thay đổi threshold.
- Tăng eval-set version khi thêm/bỏ item, đổi nhãn/slice hoặc đưa trace sản xuất vào regression set; khi đó phải tạo baseline mới. Cùng eval-set version chỉ được siết threshold.
