# TakeMeter — r/nba Discourse Classifier

Fine-tuned text classifier that evaluates discourse quality in the r/nba community. Built for AI201 Project 3.

---

## Community Choice

**r/nba** is one of the largest sports communities on Reddit, with millions of active members. Its discourse spans a wide spectrum: structured analytical arguments citing advanced statistics, confident opinions stated with zero evidence, and immediate emotional reactions to games and trades. This range makes it well-suited for a classification task — the distinctions are real, meaningful to community insiders, and observable in the text itself without requiring external context.

---

## Label Taxonomy

### `analysis`
A post that makes a structured argument supported by specific statistics, historical comparisons, or tactical observations. Evidence is concrete and verifiable.

**Example 1:**
> "Jokic's assist-to-turnover ratio this season (4.8) is the best ever recorded for a center. Combined with his 26/12/9 line, this is the greatest offensive season by a big man in NBA history."

**Example 2:**
> "The Celtics' defensive rating in the fourth quarter this postseason is historically elite. They're holding teams to under 40% from three in crunch time."

---

### `hot_take`
A bold, confident opinion stated without supporting evidence. The claim may be provocative or contrarian, but the post asserts rather than argues.

**Example 1:**
> "LeBron is not and has never been better than Jordan. People who say otherwise have never watched pre-2010 basketball. It's not even a debate."

**Example 2:**
> "The Warriors dynasty is the most overrated era in NBA history. They caught lightning in a bottle and everyone acts like it was genius."

---

### `reaction`
An immediate emotional response to a specific recent event — a game, trade, injury, or play. Expresses a feeling in the moment with little to no argument.

**Example 1:**
> "LUKA WITH THE BUZZER BEATER ARE YOU KIDDING ME. I cannot breathe. This is the greatest game I have ever watched."

**Example 2:**
> "KD to the Suns??? I cannot process this right now. The league is cooked. What even is the NBA anymore."

---

## Data Collection

**Source:** r/nba subreddit — post titles and comment text from publicly accessible threads.

**Method:** Manual collection. Posts were read and labeled directly into a CSV file. No scraping tools were used.

**Labeling process:** Each post was read in full and assigned exactly one label using the definitions above. Ambiguous cases were resolved using the decision rules documented in `planning.md`. An LLM was used to pre-label batches of 30–40 examples; every pre-assigned label was reviewed and corrected before inclusion (see AI Usage section).

**Label distribution:**

| Label | Count |
|-------|-------|
| analysis | 67 |
| hot_take | 67 |
| reaction | 67 |
| **Total** | **201** |

---

## Difficult-to-Label Examples

**Case 1:**
> "Tatum is good but he's not a guy you build around if you want a ring. History says so — look at the Celtics the last 5 years."

Could be `analysis` (references history) or `hot_take` (no actual data). **Decision: `hot_take`.** "Look at X" is not citing evidence — it is pointing the reader toward a vague conclusion. No specific statistics or comparisons are provided.

---

**Case 2:**
> "Watching Wembanyama tonight was genuinely surreal. He blocked 5 shots in the third quarter alone against the Nuggets. How is this real."

Could be `analysis` (mentions a specific stat) or `reaction` (obviously emotional). **Decision: `reaction`.** The statistic is used to express awe, not to construct an argument. The overall framing is wonder-in-the-moment, not a structured claim.

---

**Case 3:**
> "KD's scoring efficiency (62.1 TS%) actually holds up better than Durant stans admit when you look at shot difficulty. He creates his own shot at elite level."

Could be `hot_take` (defending a player against critics) or `analysis` (cites TS%). **Decision: `analysis`.** A specific advanced metric (true shooting %) is cited and contextualized with shot difficulty — this is a real analytical argument, not just a bold opinion with a decorative stat.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training setup:**

| Parameter | Value |
|-----------|-------|
| Train / val / test split | 70% / 15% / 15% (stratified) |
| Epochs | 3 |
| Learning rate | 2e-5 |
| Batch size | 16 |
| Warmup steps | 50 |
| Weight decay | 0.01 |

**Key hyperparameter decision:** The learning rate of 2e-5 was kept at the recommended default for fine-tuning BERT-family models on small datasets. Increasing it risks destabilizing pre-trained weights on 200 examples; decreasing it would require more epochs to converge. With a balanced 201-example dataset and 3 epochs, 2e-5 provided stable loss curves with no signs of overfitting on the validation set.

---

## Baseline Description

The zero-shot baseline used `llama-3.3-70b-versatile` via the Groq API with the following system prompt:

```
You are classifying posts from r/nba on Reddit.
Assign each post to exactly one of the following categories.

analysis: The post makes a structured argument supported by specific
statistics, historical comparisons, or tactical observations.
Evidence is concrete and verifiable.
Example: "Jokic's assist-to-turnover ratio (4.8) is the best ever
for a center. Combined with his 26/12/9 line, this is the greatest
offensive season by a big man in NBA history."

hot_take: A bold, confident opinion stated without supporting evidence.
The post asserts rather than argues.
Example: "LeBron is not and has never been better than Jordan.
People who say otherwise have never watched pre-2010 basketball.
It's not even a debate."

reaction: An immediate emotional response to a specific recent event.
Expresses a feeling in the moment with little to no argument.
Example: "LUKA WITH THE BUZZER BEATER ARE YOU KIDDING ME.
I cannot breathe. This is the greatest game I have ever watched."

Respond with ONLY the label name. Do not explain your reasoning.
Valid labels: analysis hot_take reaction
```

The baseline was run on the same locked 31-example test set as the fine-tuned model. Results were collected by running each test example through the API with temperature=0 and matching the response to a label string.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | 1.000 |
| Fine-tuned DistilBERT | 0.968 |
| Difference | -0.032 |

The baseline achieved perfect accuracy on the 31-example test set, while the fine-tuned model made 1 error. A 100% baseline result on a 3-class task indicates that the label taxonomy is learnable from surface features alone — capitalization and punctuation for `reaction`, statistical language for `analysis`, and assertive opinion phrasing for `hot_take`. The large llama-3.3-70b-versatile model handles these surface distinctions perfectly zero-shot. The fine-tuned DistilBERT still achieved 96.8% accuracy, which is strong performance for a 201-example training set.

---

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 1.00 | 1.00 | 10 |
| hot_take | 1.00 | 0.90 | 0.95 | 10 |
| reaction | 0.92 | 1.00 | 0.96 | 11 |
| **Macro avg** | **0.97** | **0.97** | **0.97** | **31** |

---

### Per-Class Metrics — Baseline Model

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 1.00 | 1.00 | 10 |
| hot_take | 1.00 | 1.00 | 1.00 | 10 |
| reaction | 1.00 | 1.00 | 1.00 | 11 |
| **Macro avg** | **1.00** | **1.00** | **1.00** | **31** |

---

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|:-------------------:|:-------------------:|:-------------------:|
| **True: analysis** | 10 | 0 | 0 |
| **True: hot_take** | 0 | 9 | 1 |
| **True: reaction** | 0 | 0 | 11 |

*(See also `confusion_matrix.png` in this repo.)*

The only error was one `hot_take` post predicted as `reaction`. Every other example was classified correctly. The `hot_take` → `reaction` direction is the only failure, which points to a specific boundary weakness documented in the failure analysis below.

---

### Wrong Predictions — Analysis of Failures

The fine-tuned model made only 1 hard error on the 31-example test set. Because the spec requires analysis of at least 3 failures, the section below covers the 1 true error plus 2 low-confidence correct predictions (confidence below 0.62) that reveal the same boundary weaknesses a larger test set would likely surface as additional errors.

---

**Failure 1 — True error:**

> *Post:* "Clippers fans deserve to suffer. I said what I said. Years of choke jobs. I have no sympathy. Not one drop."
>
> *True label:* `hot_take` | *Predicted:* `reaction` | *Confidence:* 0.40

**Analysis:** This post sits exactly at the `hot_take` vs. `reaction` boundary. It has the emotional intensity and punchy short-sentence structure the model learned to associate with `reaction`. However it makes no reference to a specific recent event — "years of choke jobs" is a general opinion claim with no time-anchored trigger, which by definition makes it `hot_take`. The model learned emotional tone as a proxy for `reaction` rather than the deeper structural rule: `reaction` requires a specific triggering event. The low confidence (0.40) shows the model itself was uncertain — this is the hardest case in the dataset. To fix this, the training set needs more examples of emotionally-worded hot takes that lack a specific triggering event, so the model learns to look for the event anchor rather than just the emotional register.

---

**Failure 2 — Low-confidence correct prediction (near-miss):**

> *Post:* "I've never seen two thousand people lose their minds simultaneously before tonight. The whole section I was sitting in was just completely unhinged."
>
> *True label:* `reaction` | *Predicted:* `reaction` | *Confidence:* 0.54

**Analysis:** The model got this right but barely. The post describes an in-person game experience without using the typical surface markers the model learned to associate with `reaction` — no ALL CAPS, no exclamation marks, no explosive punctuation. The calm measured tone despite clearly being an emotional reaction confused the model. This reveals that the model partially relies on punctuation and capitalization as signals for `reaction`. When those features are absent, confidence drops significantly even when the post is clearly a time-anchored emotional experience. A more robust model would learn that first-person present-tense experiential language ("I've never seen," "I was sitting in") is a `reaction` signal regardless of punctuation style.

---

**Failure 3 — Low-confidence correct prediction (near-miss):**

> *Post:* "Playoff teams with a usage rate disparity of more than 8 points between their best and second-best players win 43% of series, while balanced teams win 58% — suggesting over-reliance is a structural vulnerability."
>
> *True label:* `analysis` | *Predicted:* `analysis` | *Confidence:* 0.61

**Analysis:** The model correctly labeled this as `analysis` but with lower confidence than typical analysis posts, which mostly scored 0.95 or above. The post is dense with numbers but the argumentative framing is more subtle — it uses "suggesting" rather than the more direct "this proves" or "this shows" language present in most training examples. This indicates the model partially learned rhetorical framing words as signals for `analysis` in addition to the presence of statistics. When the connective argumentative language is understated, confidence drops even when the statistical content is strong. Fix: include more training examples of data-dense posts with understated framing to teach the model that statistics alone are sufficient evidence for the `analysis` label.

---

### Sample Classifications

| Post (truncated to 80 chars) | Predicted Label | Confidence |
|------------------------------|-----------------|------------|
| "Jokic's assist-to-turnover ratio this season (4.8) is the best ever record..." | `analysis` | 0.98 |
| "LeBron is not and has never been better than Jordan. It's not even a debate." | `hot_take` | 0.97 |
| "LUKA WITH THE BUZZER BEATER ARE YOU KIDDING ME. I cannot breathe." | `reaction` | 0.99 |
| "Clippers fans deserve to suffer. I said what I said. Years of choke jobs." | `reaction` | 0.40 |
| "The Celtics defensive rating in the fourth quarter this postseason is histo..." | `analysis` | 0.96 |

**Correct prediction explained:** The model correctly labeled the Jokic post as `analysis` with 0.98 confidence. This is the clearest possible example of the label — it names a specific metric (4.8 assist-to-turnover ratio), makes an explicit superlative claim grounded in that stat ("best ever recorded for a center"), and draws a historical comparison. These are exactly the structural features that define `analysis`: verifiable evidence used as the foundation of an argument rather than decoration for a pre-formed opinion.

---

### Reflection: What the Model Learned vs. What I Intended

The fine-tuned model achieved 96.8% accuracy and macro F1 of 0.97, which is strong performance for a 201-example dataset. However the baseline also achieving 100% accuracy reveals something important: all three labels are separable by surface features alone. The model learned punctuation and capitalization as proxies for `reaction` (ALL CAPS, exclamation marks, short explosive sentences), statistical vocabulary as a proxy for `analysis` (percentages, named metrics, comparisons), and assertive opinion language as a proxy for `hot_take` ("never," "always," "it's not even a debate"). These surface shortcuts work almost perfectly because the training examples were written with those features naturally present.

The gap between intended and learned behavior shows up in the one true failure and the two near-misses. In all three cases, the post had surface features that pointed toward one label but structural features that pointed toward another. The model learned the surface signal but not the deeper rule. For `reaction` specifically, the intended rule was "a time-anchored emotional response to a specific event" — but the model learned "emotionally-toned short text with emphatic punctuation." Those two definitions agree 96% of the time, but diverge on emotionally-written hot takes (no event anchor) and calmly-written reactions (no emphatic punctuation). A more robust classifier would require a larger and more deliberately adversarial training set — one that explicitly includes emotionally-worded hot takes and calmly-worded reactions to force the model to learn the structural rule rather than the surface shortcut.

---

## Spec Reflection

**One way the spec helped:** The requirement to identify a hard edge case before annotating any examples was genuinely valuable. Defining the `analysis` vs. `hot_take` boundary — specifically the "stats-citing hot take" decision rule — before labeling 200 posts meant every ambiguous case during annotation had a consistent resolution. Without that pre-work the dataset would have had inconsistent labels for the most common hard case, which would have directly degraded model performance on the most interesting boundary.

**One way implementation diverged from the spec:** The spec warns that a model performing above 95% accuracy on a hard subjective task is suspicious and suggests checking for test set leakage or labels that are too easy. The fine-tuned model hit 96.8% and the baseline hit 100%. There was no leakage — the split was clean and stratified. The labels genuinely are learnable from surface features, which is a real finding about the taxonomy rather than a bug in the pipeline. The honest conclusion is that the three labels are more surface-distinguishable than expected, and a harder version of this task would require labels that share more surface features while differing structurally.

---

## AI Usage

**Instance 1 — Label stress-testing before annotation:**
I gave Claude my three label definitions and the `analysis` vs. `hot_take` edge case description and asked it to generate 10 posts that sit at the boundary between those two labels. Several generated posts were genuinely difficult to classify, which revealed that my original `analysis` definition was too permissive — it did not explicitly state that a single cherry-picked stat used as rhetorical ammunition counts as `hot_take`, not `analysis`. I revised the definition to add the phrase "evidence is concrete and verifiable" and added the explicit decision rule: if removing the opinion framing would leave no substantive argument, it is `hot_take`. This change was made before annotating any examples.

**Instance 2 — Pre-labeling annotation batches:**
I used Claude to pre-label three batches of approximately 35 posts each by providing the full label definitions from `planning.md` and asking for one label per post with no explanation. I reviewed every pre-assigned label individually and corrected approximately 12% of them — the errors were concentrated in stats-citing hot takes being labeled `analysis` and calmly-written reactions being labeled `analysis` due to descriptive language. All corrected labels are what appear in the final CSV. The pre-labeling accelerated the annotation process but the correction pass was where the actual annotation judgment happened.

**Instance 3 — Failure pattern analysis:**
After generating wrong predictions from the fine-tuned model, I gave Claude the full list of misclassified and low-confidence examples and asked it to identify common themes. Claude identified that all three problematic cases involved a mismatch between surface emotional register and structural event-anchoring — the model was treating emotional tone as the primary signal for `reaction` rather than the presence of a specific triggering event. I verified this by re-reading each example and confirmed the pattern held across all three cases. This analysis is documented in the failure analysis section above.
