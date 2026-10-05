# Reflection — Lab 19

**Tên:** Lê Minh Hiếu — 2A202602848
**Cohort:** A20-K4
**Path đã chạy:** lite

---

## Câu hỏi (≤ 200 chữ)

> Trên golden set 50 queries, mode nào thắng ở loại query nào (`exact` /
> `paraphrase` / `mixed`), và tại sao? Khi nào bạn **không** dùng hybrid
> (i.e. khi nào pure BM25 hoặc pure vector là lựa chọn đúng)?

Precision@10 trung bình: hybrid 78.6% > BM25 77.8% > vector 73.2%.

- **`exact`**: BM25 = hybrid (96.7%) > vector (88.7%). Query chứa đúng thuật
  ngữ trong doc nên khớp từ khoá là đủ.
- **`paraphrase`**: BM25 33.3% ≈ hybrid 32.0% > vector 24.0%. Khác lý thuyết,
  vector không thắng vì `bge-small-en` huấn luyện cho tiếng Anh, hiểu kém câu
  tiếng Việt diễn đạt lại. Đổi sang `bge-m3` mới kỳ vọng vector thắng.
- **`mixed`**: hybrid thắng rõ (100% vs 97.0% / 98.5%). RRF cộng điểm cho doc
  được cả hai retriever xếp cao, nên lỗi riêng của từng bên bị triệt tiêu.

**Khi không dùng hybrid:** tra mã/ID/tên riêng (mã lỗi, SKU, điều luật) thì
pure BM25 đủ, nhanh hơn (P99 8.2ms vs 16.7ms) và dễ giải thích. Khi query
ngắn, đa ngôn ngữ hoặc thuần ngữ nghĩa với embedding đa ngữ tốt, và corpus
không có từ vựng chung với query, pure vector là đủ — thêm BM25 chỉ thêm nhiễu
và latency.

---

## Khối nâng cao (NB5–NB7)

- **NB5 — Filtered search:** post-filter sập khi filter chặt (recall 0.20 ở
  sel 13–32%, 0.00 ở `acme AND ≥2026` = 3.8%), filtered-ANN giữ 1.00. Over-fetch
  chỉ cứu được recall khi `fetch_k = 500` (50% corpus) — mất hết lợi ích ANN.
- **NB6 — Agentic retrieval** (cùng ngân sách 16 doc): agentic (no filter)
  recall 0.906 / balance 0.93 so với single-shot 0.526 / 0.08. `agentic (+filter)`
  thấp hơn (0.823 / 0.76) vì topic do planner **đoán** từ keyword: đoán sai hoặc
  quá hẹp là loại bỏ luôn doc liên quan nằm ở cụm bên cạnh, không có cách lấy
  lại. Filter rẻ về số call nhưng không miễn phí về recall — phải đo, không đoán.
- **NB7 — Semantic cache:** chọn ngưỡng **0.85** — tiết kiệm 100%, trả lời sai
  0%. Ngưỡng 0.75 (con số AWS) chưa đủ: vẫn trả lời sai 36% vì câu tiếng Việt
  khác chủ đề vẫn có cosine cao với `bge-small-en`. 0.90 an toàn hơn nếu corpus
  thay đổi. Demo rò tenant: `namespaced=False` GLOBEX đọc được doanh thu ACME;
  `namespaced=True` → MISS.
- **NB8:** chưa hoàn thành (lỗi Feast khi materialize on-demand feature view).

---

## Điều ngạc nhiên nhất khi làm lab này

Trên Windows, gọi API qua `localhost` chậm ~2s/request do thử IPv6 trước;
đổi sang `127.0.0.1` thì latency về đúng ~10ms. Đo latency phải biết mình đang
đo cái gì.

---

## Bonus challenge

- [ ] Đã làm bonus (xem `bonus/`)
- [ ] Pair work với: _<tên đồng đội nếu có>_
