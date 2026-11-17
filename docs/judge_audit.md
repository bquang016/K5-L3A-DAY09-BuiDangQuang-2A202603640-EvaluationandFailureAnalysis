# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (κ=0.757≥0.6, P=0.857≥0.8, R=0.8≥0.7) | DÙNG |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375<0.6, R=0.3<0.7) | LOẠI |

## ≥5 case audit thủ công (trích xuất từ judge_tone_public.csv)
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Đề xuất sửa rubric |
|---|---|---|---|---|---|
| 1 | `H-008`: "Em đã hoàn tiền cho anh rồi nha, anh check giúp em." | PASS | FAIL | Người đúng | Phản hồi xưng hô tùy tiện ("em - anh", "nha", "check giúp em"), thiếu tính chuyên nghiệp của thương hiệu LogiX. **Sửa rubric:** Bắt buộc đại từ chuẩn ("LogiX - anh/chị"), cấm xưng hô thân mật tự phát. |
| 2 | `H-029`: "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm." | PASS | FAIL | Người đúng | Cụm từ "không cần nhắn thêm" mang sắc thái cộc cằn, xua đuổi khách hàng dù có mở đầu bằng "Dạ, vui lòng chờ". **Sửa rubric:** Cấm các cụm từ thể hiện thái độ gắt gỏng, mất kiên nhẫn hoặc ngăn cản khách hỏi thêm. |
| 3 | `H-051`: "Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ." | FAIL | PASS | Người đúng | Khách hàng hiểu rõ từ mượn thông dụng "refund", câu văn đầy đủ kính ngữ "Dạ... ạ" lịch sự. Judge phạt quá máy móc. **Sửa rubric:** Cho phép các từ mượn TMĐT phổ biến (refund, order) nếu ngữ cảnh phù hợp, không trừ điểm tone. |
| 4 | `H-070`: "Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂" | PASS | FAIL | Người đúng | Chèn emoji cợt nhả ("😂😂") vào tin nhắn xử lý tài chính/khiếu nại là thiếu nghiêm túc. Judge bỏ qua emoji và chỉ đọc text. **Sửa rubric:** Quy định cấm emoji cợt nhả/thiếu đứng đắn trong văn cảnh bồi thường. |
| 5 | `H-094`: "Da, LogiX da hoan 40.000đ cho don O-341, xin loi vi su bat tien." | PASS | FAIL | Người đúng | Gõ tiếng Việt không dấu vi phạm nghiêm trọng tiêu chuẩn chăm sóc khách hàng doanh nghiệp. Judge hiểu nghĩa nên vẫn cho qua. **Sửa rubric:** Thêm tiêu chí bắt buộc tiếng Việt có dấu chuẩn chỉnh. |

## Kết luận
- **Tiêu chí bị loại và lý do**: 
  - Tiêu chí `helpful` bị **LOẠI BỎ HOÀN TOÀN** khỏi CI gate vì không đạt sàn hiệu chuẩn: độ đồng thuận rất kém ($\kappa = 0.375 < 0.6$) và độ bao phủ lỗi quá thấp ($Recall = 0.3 < 0.7$).
  - Dẫn chứng ma trận nhầm lẫn: `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`. Trong 30 ca không hữu ích (FAIL), Judge chỉ phát hiện được 9 ca và **bỏ sót tới 21 ca lỗi** ($FN = 21$). Judge bị đánh lừa bởi những câu xin lỗi sáo rỗng nhưng không đem lại giải pháp thực tế cho khách hàng. Tỉ lệ lỗi thực tế là $30\%$ nhưng Judge chỉ báo $9\%$. Nếu dùng làm gate, hệ thống sẽ để lọt $70\%$ phản hồi vô dụng.
- **Tiêu chí được chấp nhận**:
  - `tone_ok` đạt chuẩn sàn ($\kappa = 0.757 \ge 0.6, P = 0.857 \ge 0.8, R = 0.8 \ge 0.7$, ma trận: `{'tp': 24, 'fp': 4, 'fn': 6, 'tn': 66}`). Tỉ lệ lỗi quan sát $28\%$ được hiệu chỉnh qua Rogan-Gladen về $30\%$ (sát thực tế). Tiêu chí này đủ tin cậy để làm gate đo lường giọng điệu CSKH.
- **Rủi ro còn lại**:
  - Cả hai judge đều chỉ mới được kiểm nghiệm trên tập dữ liệu replay công khai (100 mẫu). Cần bổ sung tập nhãn người ẩn (held-out) với các mẫu câu mới liên tục để tránh hiện tượng prompt/rubric bị overfit trên tập public.
