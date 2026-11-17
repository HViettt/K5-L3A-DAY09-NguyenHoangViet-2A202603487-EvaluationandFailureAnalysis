# Judge audit

Chạy:
- `python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5`
- `python -m lab9.judge --csv data/eval/judge_helpful_public.csv --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Observed Fail Rate | Rogan-Gladen Corrected | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|---|---|
| `tone_ok` (`judge_tone_public.csv`) | 100 | 0.757 | 0.857 | 0.800 | 0.280 | 0.300 | **ĐẠT (PASS)** | **Dùng** (đủ tin cậy để kiểm tra giọng điệu CSKH) |
| `helpful` (`judge_helpful_public.csv`) | 100 | 0.375 | 1.000 | 0.300 | 0.090 | 0.300 | **KHÔNG ĐẠT (FAIL)** | **Loại bỏ** (bỏ lọt 70% lỗi, đặc biệt là lỗi sai số tiền FM-08) |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa Rubric / Grader |
|---|---|---|---|---|---|
| 1 | `H-008` (`tone`) — *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* | `PASS` | `FAIL` | **Người đúng** | Xưng `"em"` và dùng từ cảm thán `"nha"` quá suồng sã. Judge bị đánh lừa bởi giọng thân thiện. **Sửa rubric:** Bắt buộc xưng `"LogiX"` - `"anh/chị"`, cấm xưng `"em"`, `"bạn"` và cấm từ khẩu ngữ (`nha`, `nhé`). |
| 2 | `H-029` (`tone`) — *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* | `PASS` | `FAIL` | **Người đúng** | Vế sau *"không cần nhắn thêm"* mang tính mệnh lệnh, đuổi khách, nhưng Judge chỉ nhìn cụm từ lịch sự bề mặt *"Dạ, vui lòng"* ở đầu câu. **Sửa rubric:** Chấm toàn bộ câu; phạt nặng mọi mệnh lệnh chặn tương tác của khách dù có mở đầu bằng *"Dạ"*. |
| 3 | `H-042` (`tone`) — *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* | `FAIL` | `PASS` | **Người đúng** | Câu trả lời lịch sự, đầy đủ số tiền và thời gian nhận tiền; Judge phạt oan (`FP`) vì đòi phải có câu xin lỗi. **Sửa rubric:** Khi trạng thái là `resolved` (đã hoàn tiền thành công), chỉ cần lịch sự (`Dạ ... ạ`) và đủ thông tin, không bắt buộc câu xin lỗi rập khuôn. |
| 4 | `H-051` (`tone`) — *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* | `FAIL` | `PASS` | **Người đúng** | Quy chuẩn CSKH LogiX cho phép dùng thuật ngữ `"refund"`, nhưng Judge phạt vì chêm tiếng Anh (`FP`). **Sửa rubric:** Thêm danh sách thuật ngữ nghiệp vụ được phép (`refund`, `claim`, `voucher`, mã đơn `O-xxx`). |
| 5 | `H-063` (`tone`) — *"Dạ, LogiX đã hoàn 20.000đ và sẽ hoàn thêm cho anh/chị ạ."* | `PASS` | `FAIL` | **Người đúng** | Agent tự ý hứa hẹn *"sẽ hoàn thêm"* ngoài quyết định của hệ thống (`FN`). **Sửa rubric:** Cấm mọi cam kết hoàn thêm tiền trong tương lai không có trong kết quả xử lý. |
| 6 | `H-070` (`tone`) — *"Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂"* | `PASS` | `FAIL` | **Người đúng** | Sử dụng emoji `😂😂` phản cảm trong ngữ cảnh giải quyết khiếu nại (`FN`). **Sửa rubric:** Cấm toàn bộ emoji trong phản hồi CSKH. |
| 7 | `H-081` (`tone`) — *"Dạ, đơn O-107 chưa đủ điều kiện hoàn theo chính sách hiện hành ạ."* | `FAIL` | `PASS` | **Judge đúng** | Nhãn người gán sai (đọc nhầm ticket); phản hồi từ chối bị cụt, thiếu câu hỗ trợ chuẩn *"Anh/chị cần hỗ trợ thêm, LogiX luôn sẵn sàng ạ."* **Hành động:** Sửa lại nhãn gold của người (`human = FAIL`) trong bộ dữ liệu chuẩn. |
| 8 | `HL-016` (`helpful`) — *"Dạ, LogiX thành thật xin lỗi... LogiX đã hoàn 55.000đ cho đơn O-404..."* (Ledger ghi `35.000đ`) | `PASS` | `FAIL` | **Người đúng** | **Verbosity / Sycophancy Bias & Mù dữ liệu nền (FM-08):** Câu trả lời lặp lại lời xin lỗi rất dài dòng nhưng báo sai số tiền (`55.000đ` so với ledger `35.000đ`). Judge chấm `PASS` vì văn phong trôi chảy. **Hành động:** Loại bỏ `helpful` judge khỏi việc chấm tính đúng đắn; chuyển kiểm tra số tiền sang **code grader** (`RefundAmountCorrect` và `ReplyMatchesLedger`). |

## Kết luận
- **Tiêu chí được giữ lại:** `tone_ok` (`tp=24, fp=4, fn=6, tn=66`) đạt tất cả các ngưỡng sàn ($\kappa = 0.757 \ge 0.6$, $P = 0.857 \ge 0.8$, $R = 0.800 \ge 0.7$). Sau khi hiệu chỉnh Rogan-Gladen, tỷ lệ lỗi thực tế ước lượng là `0.300` (so với tỷ lệ quan sát `0.280`).
- **Tiêu chí bị loại và lý do:** `helpful` (`tp=9, fp=0, fn=21, tn=70`) bị **loại hoàn toàn khỏi CI gate** vì $\kappa = 0.375 < 0.6$ và $R = 0.300 < 0.7$. Trong 21 ca bỏ lọt (`FN`), có tới **18 ca là lỗi sai số tiền (FM-08)** được ngụy trang bằng lời xin lỗi dài dòng (verbosity bias) và 3 ca sai xưng hô. Tỷ lệ lỗi quan sát của `helpful` chỉ là `0.090` nhưng tỷ lệ lỗi thực sau hiệu chỉnh Rogan-Gladen lên tới `0.300`. Điều này chứng minh quy tắc cốt lõi: **tiền bạc, trạng thái phê duyệt và sự khớp nối với database/ledger bắt buộc phải chấm bằng code grader, tuyệt đối không dùng LLM Judge.**
- **Rủi ro còn lại:** Judge chỉ mới được kiểm định trên tập dữ liệu replay công khai (`n=100`); cần thu thập thêm nhãn con người độc lập trên tập held-out mới mỗi khi cập nhật prompt/rubric của judge để tránh overfit vào các mẫu câu cũ.
