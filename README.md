# ViDimABSA: A Vietnamese Dataset and Benchmark for Dimensional Aspect-Based Sentiment Analysis

ViDimABSA is, to our knowledge, the first Vietnamese dataset for **dimensional aspect-based sentiment analysis (DimABSA)**. Most ABSA datasets give each opinion a polarity label (positive, negative or neutral). ViDimABSA instead scores each opinion on two continuous dimensions:

- **Valence**: from very negative (1) to very positive (9).
- **Arousal**: from calm (1) to highly excited (9).

The dataset contains **5,743 sentences** and **6,882 (aspect, category, opinion, valence, arousal) quadruplets**. The sentences come from Vietnamese reviews of consumer apps: e-commerce, food delivery, ride-hailing and digital banking. The dataset supports all three subtasks of SemEval-2026 Task 3 (DimABSA).


> 📄 **Paper:** *ViDimABSA: A Vietnamese Dataset and Benchmark for Dimensional Aspect-Based Sentiment Analysis*, Tam Khoi Tran and Phuoc Tran. Under review at IEEE Access.

---

## Highlights

- **Continuous VA scores per aspect.** Scores lie on a 1–9 scale in steps of 0.5. Sarcastic opinions are scored by their intended meaning, not their literal wording.
- **Implicit aspects.** 16.2% of quadruplets have no explicit aspect term (`NULL`).
- **10 balanced aspect categories.** Five entities (PRODUCT, SERVICE, STAFF, APP, PRICE), each with two attributes.
- **Informal Vietnamese.** The reviews keep teencode, intensifiers (e.g. *rất*, *lắm*, *vãi*, *xỉu*) and sarcasm. Opinion spans include their modifiers and negators.
- **High inter-annotator agreement** on 300 doubly annotated sentences:
  - Cohen's κ = 0.87 for categories.
  - Krippendorff's α = 0.95 for valence and 0.87 for arousal.

## Dataset

### Splits

| Split | File | Sentences | Quadruplets | Role in our experiments |
|---|---|---:|---:|---|
| Train | `data/train.jsonl` | 3,692 | 4,411 | training |
| Public test | `data/public_test_gold_subtask2.jsonl` | 1,408 | 1,691 | development (model selection) |
| Private test | `data/private_test_gold_subtask2.jsonl` | 643 | 780 | final evaluation |
| **Total** | | **5,743** | **6,882** | |

The public and private test sets match the public and private leaderboards of the competition. We follow the competition protocol:

- The public test set is used only for early stopping and checkpoint selection.
- All reported results are on the private test set.

### Format

Each line of a file is one JSON object:

```json
{
  "id": "PUB2_TEST_0001",
  "text": "Phí giao hàng dễ thương chưa, dễ thương bằng nửa giá món ăn.",
  "quadruplets": [
    {
      "aspect":   {"term": "Phí giao hàng", "start_idx": 0, "end_idx": 13},
      "category": "PRICE#RATES",
      "opinion":  {"term": "dễ thương chưa", "start_idx": 14, "end_idx": 28},
      "valence": 2.0,
      "arousal": 7.0
    }
  ]
}
```

- `start_idx` and `end_idx` are character offsets in `text`. `end_idx` is exclusive, so `text[start_idx:end_idx] == term`.
- An implicit aspect is written as `{"term": "NULL", "start_idx": -1, "end_idx": -1}`. Its `category` still names the target being evaluated.
- An aspect with several opinions gets one quadruplet per opinion, each with its own VA scores.
- In the example above, the opinion is sarcastic ("so *cute*" about a high delivery fee). It is therefore scored as negative (valence 2.0) with high arousal (7.0).

### Aspect categories

| Category | Covers | Example (aspect, opinion) |
|---|---|---|
| `PRODUCT#QUALITY` | quality of the goods or food | (*gà rán*, *giòn tan*) |
| `PRODUCT#DESIGN` | appearance and packaging | (*bao bì*, *rách tả tơi*) |
| `SERVICE#TIME` | speed and punctuality | (*giao hàng*, *chậm quá*) |
| `SERVICE#GENERAL` | overall service experience | (*dịch vụ giao hàng*, *ổn*) |
| `STAFF#ATTITUDE` | politeness and responsiveness | (*tài xế*, *nói chuyện thì cộc cằn*) |
| `STAFF#PROFESSIONALISM` | competence and conduct | (*bác tài*, *chạy ẩu*) |
| `APP#PERFORMANCE` | speed, stability, reliability | (*app*, *đơ vãi*) |
| `APP#UI_UX` | interface and usability | (*thanh menu*, *quá nhỏ*) |
| `PRICE#PROMO` | discounts, vouchers, promotions | (*mã giảm giá*, *cái nào cũng hết lượt*) |
| `PRICE#RATES` | prices, fees, charges | (*giá ship*, *chát khủng khiếp*) |

## Subtasks

All three subtasks use the same sentences. The DimASR inputs are built from the gold aspects of each sentence.

| Subtask | Input | Output | Metric |
|---|---|---|---|
| **DimASR**: aspect sentiment regression | sentence + gold aspects | (valence, arousal) for each aspect | RMSE<sub>VA</sub> (also PCC<sub>V</sub>, PCC<sub>A</sub>) |
| **DimASTE**: aspect sentiment triplet extraction | sentence | {(aspect, opinion, V, A)} | continuous F1 (cF1) |
| **DimASQP**: aspect sentiment quad prediction | sentence | {(aspect, category, opinion, V, A)} | continuous F1 (cF1) |

**How the metrics work:**

- **RMSE<sub>VA</sub>:** `sqrt(mean((v̂−v)² + (â−a)²))`. An aspect with no prediction is scored as (5, 5).
- **cF1, matching:** a predicted tuple matches a gold tuple only if all of its categorical elements are identical after lowercasing.
- **cF1, VA discount:** each match is discounted by its VA error, `cTP = max(0, 1 − ‖(v̂,â) − (v,a)‖ / √128)`.
- **Duplicates:** a gold tuple matched by more than one prediction counts as 0.

## Benchmark results (private test set)

Multi-seed results are given as mean ± standard deviation over 3 seeds. Full results, dev scores and significance tests are in the paper.

| Model | Setting | DimASR RMSE ↓ | DimASTE cF1 ↑ | DimASQP cF1 ↑ |
|---|---|---:|---:|---:|
| Constant (5, 5) | baseline | 3.13 | – | – |
| PhoBERT-base | fine-tuned | 1.16 ± 0.04 | – | – |
| PhoBERT-large | fine-tuned | 1.16 ± 0.13 | – | – |
| XLM-R-large | fine-tuned | 1.17 ± 0.03 | – | – |
| ViT5-base | fine-tuned | 1.65 ± 0.08 | 59.49 ± 1.01 | 55.25 ± 2.40 |
| mT5-base | fine-tuned | 2.39 ± 0.66 | 60.63 ± 1.07 | 54.82 ± 1.40 |
| Qwen2.5-7B-Instruct | zero-shot | 2.94 | 25.67 | 18.27 |
| Qwen2.5-7B-Instruct | 10-shot | 1.91 | 39.19 | 33.72 |
| Qwen2.5-7B-Instruct | QLoRA | 1.02 | 62.78 | 58.63 ± 0.34 |
| Llama-3.1-8B-Instruct | QLoRA | **0.98** | 60.53 | 58.42 ± 0.48 |
| Aya Expanse 8B | QLoRA | 1.01 | **67.83** | **60.95 ± 0.80** |
| *Inter-annotator agreement* | *300 train sentences* | – | *77.61* | *68.71* |

On DimASTE and DimASR, QLoRA models were trained with a single seed.


## Contact

**Tam Khoi Tran**: Faculty of Information Technology, Ton Duc Thang University, Ho Chi Minh City, Vietnam

**Phuoc Tran** (corresponding author): Natural Language Processing and Knowledge Discovery Research Group, Faculty of Information Technology, Ton Duc Thang University. Email: tranhthanhphuoc@tdtu.edu.vn

Questions and issues are welcome via [GitHub Issues](https://github.com/tamkhojj/ViDimABSA/issues).
