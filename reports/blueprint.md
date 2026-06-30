# CI/CD Blueprint: RAG Eval + Guardrail Stack

**Sinh viên:** Nguyen Huu Duc  
**Ngày:** 2026-06-30

---

## Guard Stack Architecture

```
User Input
    │
    ▼ (~1854ms P95)
[Presidio PII Scan]
    │ block if: VN_CCCD / VN_PHONE / EMAIL detected
    │ action:   return 400 + "PII detected in query"
    ▼ (~5110ms P95)
[NeMo Input Rail]
    │ block if: off-topic / jailbreak / prompt injection
    │ action:   return 503 + refuse message
    ▼
[RAG Pipeline (Day 18)]
    │ M1 Chunk → M2 Search → M3 Rerank → GPT-4o-mini
    ▼
[NeMo Output Rail]
    │ flag if:  PII in response / sensitive content
    │ action:   replace with safe response
    ▼
User Response
```

---

## Latency Budget

*(Điền từ kết quả Task 12 — measure_p95_latency())*

| Layer | P50 (ms) | P95 (ms) | P99 (ms) | Budget |
|---|---|---|---|---|
| Presidio PII | ~1500 | 1854.35 | ~1900 | <10ms |
| NeMo Input Rail | ~4000 | 5110.97 | ~5500 | <300ms |
| RAG Pipeline | ~2000 | ~3000 | ~3500 | <2000ms |
| NeMo Output Rail | ~4000 | 5110.97 | ~5500 | <300ms |
| **Total Guard** | ~5500 | **6346.82** | ~7500 | **<500ms** |

**Budget OK?** [ ] Yes / [x] No  
**Comment:** Total Guard P95 là ~6346ms vượt rất xa mức 500ms. NeMo Guardrails sử dụng OpenAI API nên độ trễ rất cao (5s+). Cần chuyển sang mô hình local (e.g. Llama 3/Mistral chạy qua vLLM) hoặc rule-based rails thay vì LLM-based rails nếu muốn đạt yêu cầu <500ms.

---

## CI/CD Gates (phải pass trước khi merge to main)

```yaml
# .github/workflows/rag_eval.yml
- name: RAGAS Quality Gate
  run: python src/phase_a_ragas.py
  env:
    MIN_FAITHFULNESS: 0.75
    MIN_AVG_SCORE: 0.65

- name: Guardrail Gate
  run: pytest tests/test_phase_c.py -k "test_adversarial_suite_pass_rate"
  # phải ≥ 15/20 (75%)

- name: Latency Gate
  run: python -c "from src.phase_c_guard import measure_p95_latency; ..."
  # P95 total < 500ms
```

---

## Monitoring Dashboard (production)

| Metric | Alert Threshold | Action |
|---|---|---|
| RAGAS faithfulness (daily sample) | < 0.70 | Page on-call |
| Adversarial block rate | < 80% | Review new attack patterns |
| Guard P95 latency | > 600ms | Scale NeMo model |
| PII detected count | spike >10/hour | Security alert |

---

## Kết quả thực tế từ Lab

| | Kết quả |
|---|---|
| RAGAS avg_score (50q) | 0.75 |
| Worst metric | context_precision |
| Dominant failure distribution | adversarial |
| Cohen's κ | 0.000 |
| Adversarial pass rate | 4 / 20 |
| Guard P95 latency | 6346.82 ms |

---

## Nhận xét & Cải tiến

> Hệ thống guardrails có khả năng bắt PII khá tốt nhờ Presidio, nhưng độ trễ cực kỳ cao vì NeMo Guardrails phải gọi API LLM ở cả đầu vào lẫn đầu ra. Tỉ lệ adversarial pass thấp cho thấy cần điều chỉnh lại input rails.
> Trong môi trường production thực sự, mình sẽ: 1) Chuyển NeMo sang dùng local model nhỏ nhắn chuyên check an toàn (ví dụ Meta-Llama-Guard) để giảm latency, 2) Cache các rule thông thường thay vì gọi qua LLM, 3) Fine-tune RAG prompt để hạn chế tối đa sinh ra nội dung sai hoặc bị jailbreak.
