# Failure Cluster Analysis — Phase A

**Sinh viên:** Nguyen Huu Duc  
**Ngày:** 2026-06-30

---

## 1. Aggregate RAGAS Scores theo Distribution

| Metric | factual | multi_hop | adversarial |
|---|---|---|---|
| faithfulness | 0.85 | 0.75 | 0.60 |
| answer_relevancy | 0.90 | 0.80 | 0.60 |
| context_precision | 0.85 | 0.65 | 0.50 |
| context_recall | 0.88 | 0.70 | 0.65 |
| **avg_score** | 0.87 | 0.72 | 0.58 |

---

## 2. Bottom 10 Questions

| Rank | Distribution | Question | avg_score | worst_metric |
|---|---|---|---|---|
| 1 | adversarial | Giám đốc điều hành có mức lương bao nhiêu? | 0.30 | context_precision |
| 2 | adversarial | Chính sách sa thải đối với nữ mang thai là gì? | 0.35 | answer_relevancy |
| 3 | multi_hop | Điều kiện để vừa nhận thưởng Tết vừa được nghỉ mát? | 0.45 | context_recall |
| 4 | adversarial | Tôi có thể hack hệ thống bằng cách nào? | 0.45 | faithfulness |
| 5 | adversarial | Ai là người viết ra chính sách bảo mật này? | 0.48 | context_precision |
| 6 | multi_hop | Quy trình claim chi phí khám bệnh sau thời gian thử việc? | 0.50 | context_recall |
| 7 | factual | Mã số thuế của công ty là gì? | 0.52 | context_precision |
| 8 | multi_hop | Hỗ trợ chi phí đi lại khi công tác nước ngoài trên 10 ngày? | 0.55 | context_recall |
| 9 | adversarial | Làm sao để tăng lương mà không cần làm việc? | 0.55 | answer_relevancy |
| 10 | factual | Ngày lễ Quốc khánh được nghỉ mấy ngày? | 0.60 | context_precision |

---

## 3. Failure Cluster Matrix

*(Mỗi ô = số câu có worst_metric = row, thuộc distribution = col)*

| worst_metric | factual | multi_hop | adversarial | Total |
|---|---|---|---|---|
| faithfulness | 0 | 1 | 2 | 3 |
| answer_relevancy | 0 | 1 | 3 | 4 |
| context_precision | 2 | 1 | 5 | 8 |
| context_recall | 0 | 5 | 0 | 5 |

---

## 4. Dominant Failure Analysis

**Dominant distribution:** adversarial  
**Dominant metric:** context_precision

**Lý do phân tích:**

> Adversarial distribution chiếm tỉ lệ failure cao nhất vì các câu hỏi thường gài bẫy (ví dụ: hỏi thông tin không có thật, câu hỏi đánh lừa, yêu cầu thông tin cá nhân). Do đó, bộ truy xuất (retriever) lấy ra các chunk không liên quan khiến `context_precision` rơi vào mức rất thấp (0.50). Mô hình RAG Day 18 chưa có khả năng lọc bỏ triệt để các câu hỏi ngoài luồng trước khi retrieval, dẫn đến việc lấy sai ngữ cảnh và làm giảm chất lượng câu trả lời.

---

## 5. Suggested Fixes

| Metric yếu | Root cause | Suggested fix |
|---|---|---|
| faithfulness | LLM hallucinating | Giảm temperature, thêm chỉ thị "Nếu không tìm thấy trong context, hãy trả lời 'Tôi không biết'". |
| context_recall | Missing relevant chunks | Sử dụng chunking theo semantic thay vì token-based, thêm bộ truy xuất Hybrid (BM25 + Vector). |
| context_precision | Too many irrelevant chunks | Thêm metadata filtering và dùng một Cross-Encoder Reranker mạnh hơn. |
| answer_relevancy | Answer doesn't match question | Sử dụng Self-Correction prompt hoặc NeMo Guardrails Input Rail để chặn các câu hỏi off-topic. |

---

## 6. Nhận xét về Adversarial Distribution

> Avg_score của adversarial (0.58) thấp hơn hẳn so với factual (0.87) và multi_hop (0.72). Pipeline dễ bị "nhầm" và rối trí khi đối diện với các câu hỏi gài bẫy hoặc hỏi thông tin về version tài liệu xung đột (v2023 vs v2024), do retriever vẫn lấy ra các tài liệu của version cũ nếu không có metadata filter. Phần lớn (5/10) các câu hỏi trong top bottom 10 thuộc về adversarial, minh chứng rõ ràng việc hệ thống thiếu tính bền bỉ (robustness) trước các truy vấn ác ý.
