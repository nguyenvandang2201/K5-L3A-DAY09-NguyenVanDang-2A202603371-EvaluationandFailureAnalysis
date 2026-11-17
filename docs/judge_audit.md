# Judge audit

Kết quả replay ngày 2026-11-17, với FAIL là positive class:

| Tiêu chí | n | κ | Precision | Recall | Confusion `tp/fp/fn/tn` | Đạt floors? | Quyết định |
|---|---:|---:|---:|---:|---|---|---|
| `tone_ok` | 100 | 0.757 | 0.857 | 0.800 | 24 / 4 / 6 / 66 | Có | Dùng được cho tiêu chí tone trong phạm vi replay |
| `helpful` | 100 | 0.375 | 1.000 | 0.300 | 9 / 0 / 21 / 70 | Không | Loại khỏi gate |

Tone judge quan sát failure rate 0.28; Rogan–Gladen hiệu chỉnh thành 0.30. Helpful judge bỏ sót 21/30 lỗi và không đạt cả κ lẫn recall, nên precision 1.0 không đủ để đưa judge này vào gate.

## Audit thủ công các bất đồng

| # | Case | Judge | Người | Ai đúng? | Sửa rubric / hành động |
|---:|---|---|---|---|---|
| 1 | H-008, “Em đã hoàn tiền… nha” | PASS | FAIL | Người | Nêu rõ xưng “em/anh” và “nha” là quá suồng sã trong khiếu nại. |
| 2 | H-042, reply ngắn nhưng đủ đơn, tiền và thời gian | FAIL | PASS | Người | Không bắt buộc câu xin lỗi khi câu trả lời đã lịch sự và đủ thông tin. |
| 3 | H-063, hứa “sẽ hoàn thêm” | PASS | FAIL | Người | Cấm lời hứa ngoài trạng thái/ledger hiện có. |
| 4 | H-070, dùng 😂😂 trong khiếu nại | PASS | FAIL | Người | Ghi rõ emoji vui nhộn là không phù hợp ngữ cảnh khiếu nại. |
| 5 | H-094, tiếng Việt mất dấu kiểu dịch máy | PASS | FAIL | Người | Yêu cầu tiếng Việt tự nhiên, có dấu; không chỉ kiểm tra từ khóa lịch sự. |
| 6 | HL-015, nói hoàn 55.000đ nhưng ledger 40.000đ | PASS | FAIL | Người | Helpful rubric phải đối chiếu ledger; chuyển phép kiểm tiền sang code grader. |
| 7 | HL-047, “Ok bạn… xử lý rồi nhé” | PASS | FAIL | Người | Tách sự thân thiện khỏi tính hữu ích; yêu cầu trạng thái và hành động cụ thể. |

## Kết luận

Chỉ `tone_ok` đạt κ≥0.6, precision≥0.8 và recall≥0.7. Judge này chỉ hỗ trợ tiêu chí ngôn ngữ; tiền, trạng thái, approval và ledger vẫn dùng code grader. `helpful` bị loại vì recall 0.3: một gate dựa trên nó sẽ cho lọt 70% lỗi thật. Cả hai bộ đều là replay công khai, chưa phải held-out; trước khi dùng với model thật cần một tập nhãn người mới và audit theo slice/ngôn ngữ.
