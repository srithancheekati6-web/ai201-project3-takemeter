# TakeMeter — Planning Document
## Community: r/nba

### Why r/nba?
r/nba is one of the largest sports communities on Reddit, with millions of members posting constantly during the NBA season. The discourse is highly varied: some posts cite advanced statistics and historical context to make an argument, others are bold opinions stated with no evidence, and others are pure emotional reactions to a game that just ended. This range makes it an ideal community for a classification task — the distinctions between post types are real, meaningful to community members, and observable in the text itself.

---

## Label Taxonomy

### Label 1: `analysis`
**Definition:** The post makes a structured argument supported by specific statistics, historical comparisons, or tactical/strategic observations. Evidence is concrete and verifiable.

**Example 1:**
> "Jokic's assist-to-turnover ratio this season (4.8) is the best ever recorded for a center. Combined with his 26/12/9 line, this isn't just MVP-level — it's the greatest offensive season by a big man in NBA history."

**Example 2:**
> "The Celtics' defensive rating in the fourth quarter this postseason (98.2) is historically elite. They're holding teams to under 40% from three in crunch time. That's why their late-game collapses last year haven't repeated — the scheme is fundamentally different."

---

### Label 2: `hot_take`
**Definition:** A bold, confident opinion stated without supporting evidence. The claim may be provocative or contrarian, but the post asserts rather than argues. No stats, no comparisons — just a strong opinion.

**Example 1:**
> "LeBron is not and has never been better than Jordan. People who say otherwise have never watched pre-2010 basketball. It's not even a debate."

**Example 2:**
> "The Warriors dynasty is the most overrated era in NBA history. They caught lightning in a bottle with the three-point revolution and everyone acts like it was some genius achievement. Any team with Curry would have won those rings."

---

### Label 3: `reaction`
**Definition:** An immediate emotional response to a specific recent event (a game, a trade, an injury, a play). The post expresses a feeling in the moment with little to no argument or evidence. Time-sensitive and emotionally driven.

**Example 1:**
> "LUKA WITH THE BUZZER BEATER ARE YOU KIDDING ME. I can't breathe. This is the greatest game I've ever watched."

**Example 2:**
> "KD to the Suns??? I can't process this right now. The league is cooked. What even is the NBA anymore."

---

## Hard Edge Cases

### Anticipated Hard Case: Stats-citing hot take
**Post:**
> "LeBron is overrated — his playoff win rate against top-seeded opponents is below .500."

**Problem:** This post cites a specific stat (sounds like `analysis`) but uses it as ammunition for a one-sided attack rather than as part of a genuine argument (`hot_take`).

**Decision rule:** If the post provides one isolated statistic to support a pre-formed opinion — especially with accusatory or dismissive framing — label it `hot_take`. True `analysis` uses evidence as the foundation of an argument, not decoration for a predetermined conclusion. The key question: would removing the opinion framing leave a substantive argument? If no, it's `hot_take`.

### Anticipated Hard Case: Emotional reaction with a claim
**Post:**
> "That was the worst refereeing I've ever seen. The NBA is rigged, no other explanation."

**Problem:** It's a reaction (emotional, event-triggered) but also makes a claim ("the NBA is rigged").

**Decision rule:** If the claim is obviously hyperbolic and driven by frustration rather than reasoned argument, label it `reaction`. The tell is language like "no other explanation" — that's frustration, not analysis.

---

## Data Collection Plan

- **Source:** r/nba on Reddit — post titles and comment text from publicly accessible threads
- **Method:** Manual collection — read posts, copy text and label into CSV
- **Target distribution:** ~70 `analysis`, ~70 `hot_take`, ~70 `reaction` (approximately equal thirds — no label above 70%)
- **If a label is underrepresented after 150 examples:** Search r/nba specifically for that type (e.g., search "stat" or "numbers" threads for `analysis`; search game threads for `reaction`)
- **Minimum per label:** 60 examples

---

## Evaluation Metrics

**Why accuracy alone is not enough:**
This is a 3-class task with roughly balanced labels. Accuracy tells you overall correctness but hides per-class failures. A model that learns to predict `hot_take` 90% of the time would get decent accuracy on an imbalanced dataset while completely failing on `analysis` and `reaction`.

**Metrics I will use:**
- **Accuracy:** Overall benchmark, comparable to baseline
- **Per-class F1:** The most important metric — harmonic mean of precision and recall for each label. Catches cases where the model learns one label well at the expense of others.
- **Confusion matrix:** Shows directional errors — which specific pairs of labels are being confused and in which direction.
- **Macro F1:** Average F1 across classes (unweighted) — meaningful because classes are roughly balanced.

---

## Definition of Success

A classifier that achieves **macro F1 ≥ 0.70** across all three labels on the test set, with **no single label below F1 = 0.60**, would be genuinely useful as a community moderation or discourse-quality signal tool. I would accept this as "good enough for deployment." The fine-tuned model must also **meaningfully exceed the zero-shot baseline** (by at least 0.08 accuracy) to justify the fine-tuning effort.

---

## AI Tool Plan

### Label stress-testing
I will give Claude my label definitions and edge case description and ask it to generate 8–10 boundary posts that sit between `analysis` and `hot_take` — the hardest boundary. If I can't cleanly label those posts using my definitions, I will revise the definitions before annotating 200 examples.

### Annotation assistance
I plan to use an LLM to pre-label batches of 30–40 examples at a time by providing it my full label definitions and asking for one label per post. I will review and correct every pre-assigned label before accepting it into the dataset. All pre-labeled examples will be disclosed in the AI usage section.

### Failure analysis
After generating wrong predictions from the fine-tuned model, I will paste the full list of misclassified examples into Claude and ask it to identify systematic patterns (e.g., "does the model consistently confuse short posts?" or "is sarcasm a factor?"). I will then verify those patterns by re-reading the examples myself before writing up the evaluation report.

---

## Hard Annotation Decisions (updated during data collection)

*(Updated as I annotate — minimum 3 required)*

**Case 1:** "Tatum is good but he's not a guy you build around if you want a ring. History says so — look at the Celtics the last 5 years." — Labeled `hot_take`. Vague reference to "history" without actual data; "look at X" is not citing evidence.

**Case 2:** "Watching Wembanyama tonight was genuinely surreal. He blocked 5 shots in the third quarter alone against the Nuggets. How is this real." — Labeled `reaction`. The stat (5 blocks) is used to express awe, not to make an argument. The overall tone is immediate and emotional.

**Case 3:** "KD's scoring efficiency (62.1 TS%) actually holds up better than Durant stans admit when you look at shot difficulty. He creates his own shot at elite level." — Labeled `analysis`. This references a specific advanced metric (true shooting %) and contextualizes it with shot difficulty — a real analytical argument.
