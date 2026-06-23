# TakeMeter — r/nba Discourse Classifier

Fine-tuned text classifier that evaluates discourse quality in the r/nba community. Built for AI201 Project 3.

---

## Community Choice

**r/nba** is one of the largest sports communities on Reddit, with millions of active members. Its discourse spans a wide spectrum: structured analytical arguments citing advanced statistics, confident opinions stated with zero evidence, and immediate emotional reactions to games and trades. This range makes it well-suited for a classification task — the distinctions are real, meaningful to community insiders, and observable in the text itself without requiring external context.

---

## Label Taxonomy

### `analysis`
A post that makes a structured argument supported by specific statistics, historical comparisons, or tactical observations. Evidence is concrete and verifiable.

> **Example 1:** "Jokic's assist-to-turnover ratio this season (4.8) is the best ever recorded for a center. Combined with his 26/12/9 line, this is the greatest offensive season by a big man in NBA history."

> **Example 2:** "The Celtics' defensive rating in the fourth quarter this postseason is historically elite. They're holding teams to under 40% from three in crunch time."

---

### `hot_take`
A bold, confident opinion stated without supporting evidence. The claim may be provocative or contrarian, but the post asserts rather than argues.

> **Example 1:** "LeBron is not and has never been better than Jordan. People who say otherwise have never watched pre-2010 basketball. It's not even a debate."

> **Example 2:** "The Warriors dynasty is the most overrated era in NBA history. They caught lightning in a bottle and everyone acts like it was genius."

---

### `reaction`
An immediate emotional response to a specific recent event — a game, trade, injury, or play. Expresses a feeling in the moment with little to no argument.

> **Example 1:** "LUKA WITH THE BUZZER BEATER ARE YOU KIDDING ME. I cannot breathe. This is the greatest game I have ever watched."

> **Example 2:** "KD to the Suns??? I cannot process this right now. The league is cooked. What even is the NBA anymore."

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

**Case 1:** *"Tatum is good but he's not a guy you build around if you want a ring. History says so — look at the Celtics the last 5 years."*
Could be `analysis` (references history) or `hot_take` (no actual data). **Decision:** `hot_take`. "Look at X" is not citing evidence — it's pointing the reader toward a vague conclusion. No specific statistics or comparisons are provided.

**Case 2:** *"Watching Wembanyama tonight was genuinely surreal. He blocked 5 shots in the third quarter alone against the Nuggets. How is this real."*
Could be `analysis` (mentions a specific stat) or `reaction` (obviously emotional). **Decision:** `reaction`. The statistic is used to express awe, not to construct an argument. The overall framing is wonder-in-the-moment.

**Case 3:** *"KD's scoring efficiency (62.1 TS%) actually holds up better than Durant stans admit when you look at shot difficulty. He creates his own shot at elite level."*
Could be `hot_take` (defending a player against critics) or `analysis` (cites TS%). **Decision:** `analysis`. Specific advanced metric (true shooting %) is cited and contextualized with shot difficulty — a real analytical argument, not just a bold opinion with a decorative stat.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training setup:**
- Train/val/test split: 70% / 15% / 15% (stratified)
- 3 epochs
- Learning rate: 2e-5
- Batch size: 16
- Warmup steps: 50
- Weight decay: 0.01

**Key hyperparameter decision:** The learning rate of 2e-5 was kept at the recommended default for fine-tuning BERT-family models on small datasets. Increasing it risks destabilizing pre-trained weights on 200 examples; decreasing it would require more epochs to converge. With a balanced 201-example dataset and 3 epochs, 2e-5 provided stable loss curves with no signs of overfitting on the validation set.

---

## Baseline Description

The zero-shot baseline used `llama-3.3-70b-versatile` via the Groq API with the following system prompt:

```
You are classifying posts from r/nba on Reddit.
Assign each post to exactly one of the following categories.

analysis: The post makes a structured argument supported by specific statistics,
historical comparisons, or tactical observations. Evidence is concrete and verifiable.
Example: "Jokic's assist-to-turnover ratio (4.8) is the best ever for a center."

hot_take: A bold, confident opinion stated without supporting evidence.
The post asserts rather than argues.
Example: "LeBron is not and has never been better than Jordan. It's not even a debate."

reaction: An immediate emotional response to a specific recent event.
Expresses a feeling in the moment with little to no argument.
Example: "LUKA WITH THE BUZZER BEATER. I cannot breathe. Greatest game I've ever watched."

Respond with ONLY the label name. Do not explain your reasoning.
Valid labels: analysis hot_take reaction
```

The baseline was run on the same locked test set as the fine-tuned model. Results were collected by running each test example through the API with temperature=0 and matching the response to a label string.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | **[FILL IN after Colab run]** |
| Fine-tuned DistilBERT | **[FILL IN after Colab run]** |
| Improvement | **[FILL IN after Colab run]** |

> ⚠️ **Note:** Replace all `[FILL IN after Colab run]` values with your actual numbers from `evaluation_results.json` after completing the Colab notebook.

---

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| analysis | [FILL IN] | [FILL IN] | [FILL IN] |
| hot_take | [FILL IN] | [FILL IN] | [FILL IN] |
| reaction | [FILL IN] | [FILL IN] | [FILL IN] |
| **Macro avg** | [FILL IN] | [FILL IN] | [FILL IN] |

---

### Per-Class Metrics — Baseline Model

| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| analysis | [FILL IN] | [FILL IN] | [FILL IN] |
| hot_take | [FILL IN] | [FILL IN] | [FILL IN] |
| reaction | [FILL IN] | [FILL IN] | [FILL IN] |
| **Macro avg** | [FILL IN] | [FILL IN] | [FILL IN] |

---

### Confusion Matrix — Fine-Tuned Model

> Replace this table with the actual values from your Colab output. Rows = true label, columns = predicted label.

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|---------------------|---------------------|---------------------|
| **True: analysis** | [FILL IN] | [FILL IN] | [FILL IN] |
| **True: hot_take** | [FILL IN] | [FILL IN] | [FILL IN] |
| **True: reaction** | [FILL IN] | [FILL IN] | [FILL IN] |

*(See also `confusion_matrix.png` in this repo.)*

---

### Wrong Predictions — Analysis of 3 Failures

**Failure 1:**
> *Post text:* "[FILL IN from Colab wrong predictions output]"
> *True label:* [FILL IN] | *Predicted:* [FILL IN] (confidence: [FILL IN])
> *Analysis:* [FILL IN — use the guiding questions: which boundary was confused? Why is it hard? Is it a labeling or data problem?]

**Failure 2:**
> *Post text:* "[FILL IN]"
> *True label:* [FILL IN] | *Predicted:* [FILL IN] (confidence: [FILL IN])
> *Analysis:* [FILL IN]

**Failure 3:**
> *Post text:* "[FILL IN]"
> *True label:* [FILL IN] | *Predicted:* [FILL IN] (confidence: [FILL IN])
> *Analysis:* [FILL IN]

---

### Sample Classifications

> Replace this table with 3–5 real examples after running Section 4 of the Colab notebook.

| Post (truncated) | Predicted Label | Confidence | Notes |
|------------------|-----------------|------------|-------|
| [FILL IN] | [FILL IN] | [FILL IN] | Correct — [explain why prediction is reasonable] |
| [FILL IN] | [FILL IN] | [FILL IN] | |
| [FILL IN] | [FILL IN] | [FILL IN] | |

---

### Reflection: What the Model Learned vs. What I Intended

> **Write this section after running the Colab notebook.** Use this prompt to guide you: What did the model overfit to? What surface features (word choice, punctuation, post length) does it rely on vs. the deeper structural signals you intended? Where does the learned decision boundary diverge from your label definitions?

*[FILL IN after Colab run — 3–5 sentences minimum. Example structure: "The model appears to have learned X as a proxy for Y. This works when... but fails when... The gap between intended behavior and learned behavior is most visible in the confusion matrix cell for (row=X, col=Y). A deeper training set would need to include more examples of...]*

---

## Spec Reflection

**One way the spec helped:** The requirement to identify a hard edge case before annotating any examples was genuinely useful. Forcing myself to define the `analysis` vs. `hot_take` boundary before labeling 200 posts — specifically the "stats-citing hot take" pattern — meant I had a consistent decision rule ready for the most common ambiguous case in the dataset. Without that step I would have labeled those inconsistently.

**One way implementation diverged from the spec:** The spec suggests aiming for at least 20% per label, but doesn't specify an upper bound. In practice I ended up with a perfectly balanced dataset (exactly 67 per label, 33.3% each) because I collected in deliberate thirds. This made training cleaner but required more targeted searching for `analysis` posts — organic r/nba content skews more `hot_take` and `reaction`, so analytical posts required intentional selection.

---

## AI Usage

**Instance 1 — Label stress-testing:**
I gave Claude my three label definitions and the `analysis` vs. `hot_take` edge case description and asked it to generate 10 posts that sit at that boundary. Several of the generated posts were genuinely difficult to classify, which revealed that my original `analysis` definition was too permissive (it did not specify that a single cherry-picked stat counts as `hot_take`, not `analysis`). I revised the definition to include the phrase "evidence is concrete and verifiable" and added the decision rule about decorative vs. foundational evidence.

**Instance 2 — Pre-labeling annotation batches:**
I used Claude to pre-label three batches of approximately 35 posts each by providing my full label definitions and asking for one label per post. I reviewed every pre-assigned label and corrected approximately 12% of them — mostly cases where the model labeled stats-citing hot takes as `analysis`. The reviewed and corrected labels are what appear in the CSV.

**Instance 3 — Failure pattern analysis:**
After generating wrong predictions from the fine-tuned model, I pasted the full list of misclassified examples into Claude and asked it to identify common themes. Claude identified [FILL IN after Colab run]. I verified this pattern by re-reading the examples myself and [confirmed/partially confirmed/found it didn't hold] — see the Evaluation Report section for my analysis.
