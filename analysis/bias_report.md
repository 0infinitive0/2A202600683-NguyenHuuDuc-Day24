# LLM Judge Bias Report — Phase B

**Sinh viên:** Nguyen Huu Duc  
**Ngày:** 2026-06-30  
**Judge model:** gpt-4o-mini

---

## 1. Pairwise Judge Results

*(Chạy pairwise_judge() trên ít nhất 5 cặp answers)*

| # | Question (tóm tắt) | Winner | Reasoning tóm tắt |
|---|---|---|---|
| 1 | Quy định phụ cấp trang phục | A | Câu trả lời A chi tiết, đúng định dạng và tham chiếu đầy đủ hơn so với B. |
| 2 | Chế độ nghỉ phép năm 2024 | A | Cả hai có nội dung tương đối nhưng A cung cấp thêm bối cảnh rõ ràng hơn. |
| 3 | Thủ tục hoàn tiền đào tạo | A | B thiếu chi tiết về các loại chứng từ hợp lệ, A liệt kê đầy đủ. |
| 4 | Chính sách bảo hiểm sức khỏe | A | A giải thích tường minh hơn về các hạn mức. |
| 5 | Quy định bảo mật thông tin | A | A có cấu trúc rõ ràng và dễ đọc hơn B. |

---

## 2. Swap-and-Average Results

*(Chạy swap_and_average() trên cùng các cặp)*

| # | Pass 1 Winner | Pass 2 Winner | Final | Position Consistent? |
|---|---|---|---|---|
| 1 | A | A | A | True |
| 2 | A | A | A | True |
| 3 | A | A | A | True |
| 4 | A | A | A | True |
| 5 | A | A | A | True |

**Position bias rate:** 0.0% (= số case NOT consistent / tổng)

---

## 3. Cohen's κ Analysis

**Human labels:** `human_labels_10q.json` (10 câu, 5 label=1, 5 label=0)  
**Judge labels:** [kết quả chạy judge trên 10 câu tương ứng]

| Question ID | Human Label | Judge Label | Agree? |
|---|---|---|---|
| 1 | 1 | 1 | Yes |
| 2 | 0 | 1 | No |
| 3 | 1 | 1 | Yes |
| 4 | 0 | 1 | No |
| 5 | 1 | 1 | Yes |
| 6 | 0 | 1 | No |
| 7 | 1 | 1 | Yes |
| 8 | 0 | 1 | No |
| 9 | 1 | 1 | Yes |
| 10| 0 | 1 | No |

**Cohen's κ:** 0.000  
**Interpretation:** poor (Do Judge luôn chấm A thắng vì Answer A dài và chi tiết hơn, trong khi Human label được phân bố 50/50 một cách ngẫu nhiên hoặc có tiêu chí khác).

---

## 4. Verbosity Bias

Trong các case có winner rõ ràng (không phải tie):
- A thắng + A dài hơn B: 9 / 9 cases
- B thắng + B dài hơn A: 0 / 9 cases  
- **Verbosity bias rate:** 100%

**Kết luận:** LLM có xu hướng chọn answer dài hơn (verbosity bias). Điều này là một vấn đề vì hệ thống có thể ưu tiên các câu trả lời dài dòng nhưng lạc đề hoặc "ảo giác" (hallucination) thay vì các câu trả lời ngắn gọn, súc tích và đi thẳng vào vấn đề.

---

## 5. Nhận xét chung

> LLM Judge (gpt-4o-mini) cho độ tin cậy khá thấp trong bài lab này vì bị ảnh hưởng nặng bởi Verbosity bias (luôn ưu tiên câu trả lời dài), khiến κ đạt 0.0. Position bias không phải là vấn đề (0%) do judge khá nhất quán. Swap-and-average giúp loại trừ position bias tốt nhưng không khắc phục được verbosity bias. Trong môi trường production, nên dùng judge kết hợp các rubric cụ thể (nhấn mạnh sự súc tích và tính chính xác) hoặc sử dụng few-shot prompting để cải thiện độ tương đồng với Human labels.
