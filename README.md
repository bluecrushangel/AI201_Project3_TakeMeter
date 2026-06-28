# TakeMeter

A text classifier that labels NBA Reddit posts by intent: Analysis, Hot Take, or Reaction. Fine-tuned on DistilBERT and evaluated against a Groq LLM baseline.

---

## Community

**Chosen community:** r/nba, r/nbatalk, r/nbadiscussion

NBA Reddit has high volume and natural variety.

---

## Label Taxonomy

Labels are based on **intent**, not content. The guiding annotation question was: _What is this post primarily trying to do?_

### Analysis

Build an argument with evidence the reader can verify.

> "He's averaging 35.4 points per 36 minutes (second only to Wilt all time) and 0.392 WS/48, which is the highest of all time. We may be witnessing the best regular season peak we've ever seen from a player, given the numbers."

> "Westbrook is Top 9 in 3FG% at 43.6% with 3PA > 80 (101 attempts) and Wide Open. His ranking is higher than Derrick White, LaMelo, Trey Murphy, Naz Reid, etc."

### Hot Take

Assert a claim confidently, with or without token stats.

> "JALEN BRUNSON IS THE BEST PG IN THE NBA…. SAY IT WITH US"

> "Anthony Davis has no weaknesses defensively, legitimately one of the most versatile defenders ever."

### Reaction

Express a feeling about something that just happened.

> "This is legit the most shocking trade I have ever seen and I've been watching for over 30 years. Completely out of nowhere."

> "Legit done as a Mavs fan. This is the dumbest trade of all time! Completely unforgivable."

---

## Data Collection

**Sources:** r/nba, r/nbatalk, r/nbadiscussion, manually scraped.

**Distribution:** ~67 posts per label, 200 total.

**Labeling process:** Each post was labeled by asking "What is this post primarily trying to do?" Evidence-backed reasoning goes to Analysis, bold assertion goes to Hot Take, and emotional response to a recent event goes to Reaction. A priority rule (Analysis > Hot Take > Reaction) resolved posts with features from multiple classes.

### Three difficult annotation decisions

**1. Stat-dressed hot take**

> "Westbrook is Top 9 in 3FG% at 43.6%... Call him Wetbrook - his shot is Wet this year."

Real stats, but the purpose is to assert Westbrook is underrated, not to build an argument. Labeled Analysis under the priority rule, but sits close to the Hot Take boundary. This pattern turned out to be the model's main failure mode.

**2. Opinion disguised as analysis**

> "While Leonard was amazing all series, no more so than on that shot, he doesn't get a chance to make it without the amazing work by Kyle Lowry throughout the game."

Multi-clause sentence that reads like a breakdown, but no verifiable evidence is cited. It's a subjective claim about credit. Labeled **Hot Take** under intent framing even though the structure is misleading.

**3. Reaction with a hot take opener**

> "All time dumbest trades. Got crumbs for a generational player that just made it to the finals. I literally will not watch the Mavericks anymore."

Opens with a bold superlative claim, but the rest of the post is a raw emotional reaction to a specific trade. Labeled **Reaction** because the dominant intent is expressing a feeling, not asserting a ranking.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased`

**Training setup:** Fine-tuned on Google Colab using a free T4 GPU. Training took roughly 5-10 minutes on 200 examples.

**Hyperparameter decision:** Used a batch size of 8 to reduce overfitting risk. With only ~67 examples per class, larger batches would have produced noisier gradient updates.

---

## Baseline

**Model:** Groq LLM

**Approach:** Each post was passed to the Groq API with a prompt using the same intent-based label definitions as annotation. Results were collected over the full 31-post test set.

---

## Evaluation Report

### Overall Accuracy

| Model                 | Accuracy |
| --------------------- | -------- |
| Fine-tuned DistilBERT | 74.2%    |
| Groq LLM baseline     | 93.5%    |

### Per-Class Metrics: Fine-Tuned Model

| Class         | Precision | Recall   | F1       | Support |
| ------------- | --------- | -------- | -------- | ------- |
| Reaction      | 1.00      | 0.90     | 0.95     | 10      |
| Analysis      | 0.59      | 1.00     | 0.74     | 10      |
| Hot Take      | 0.80      | 0.36     | 0.50     | 11      |
| **Macro avg** | **0.80**  | **0.75** | **0.73** | **31**  |

### Per-Class Metrics: Groq Baseline

| Class         | Precision | Recall   | F1       | Support |
| ------------- | --------- | -------- | -------- | ------- |
| Reaction      | 1.00      | 0.90     | 0.95     | 10      |
| Analysis      | 0.91      | 1.00     | 0.95     | 10      |
| Hot Take      | 0.91      | 0.91     | 0.91     | 11      |
| **Macro avg** | **0.94**  | **0.94** | **0.94** | **31**  |

### Confusion Matrix: Fine-Tuned Model

|                    | Predicted: Reaction | Predicted: Analysis | Predicted: Hot Take |
| ------------------ | ------------------- | ------------------- | ------------------- |
| **True: Reaction** | 9                   | 0                   | 1                   |
| **True: Analysis** | 0                   | 10                  | 0                   |
| **True: Hot Take** | 0                   | 7                   | 4                   |

The confusion is almost entirely one-directional: **7 of 11 Hot Take posts were predicted as Analysis**. Analysis was never misclassified. The model learned Reaction and Analysis cleanly but never found the Hot Take / Analysis boundary.

### Wrong Predictions

**Example 1**

> "Despite 3 MVPs and a chip Jokic is still underrated by the general public"
> True: Hot Take | Predicted: Analysis (confidence: 0.35)

"3 MVPs and a chip" looks like evidence to the model, but the post is asserting an opinion about public perception. Any reference to accolades or stats gets pulled toward Analysis, even when those references are just decoration.

**Example 2**

> "2016 LeBron is EASILY better than both Giannis and Jokic at their MVP peak"
> True: Hot Take | Predicted: Analysis (confidence: 0.38)

Bold comparative claim, but the model latched onto the comparative structure ("better than... at their MVP peak") as resembling an argument. Comparative framing does not equal analysis.

**Example 3**

> "While Leonard was amazing all series, no more so than on that shot, he doesn't get a chance to make it without the amazing work by Kyle Lowry throughout the game."
> True: Hot Take | Predicted: Analysis (confidence: 0.37)

Multi-clause subordinate sentence structure ("while... he doesn't... without...") that reads like a breakdown. The model pattern-matched on sentence complexity, not on whether evidence was actually cited.

**Root cause:** All wrong predictions have confidence scores between 0.34 and 0.38, barely above the 0.33 random baseline. The model is not confidently wrong; it is genuinely uncertain. Understanding the Hot Take / Analysis boundary requires knowing _why_ a stat or structured sentence is present, which DistilBERT on 200 examples can't reliably infer.

**What would fix it:** More Hot Take training examples that use complex sentence structure and reference stats as decoration, so the model learns that organized language does not automatically mean Analysis.

### Sample Classifications

| Post (truncated)                                                                        | True Label | Predicted | Confidence | Notes                                                           |
| --------------------------------------------------------------------------------------- | ---------- | --------- | ---------- | --------------------------------------------------------------- |
| "JALEN BRUNSON IS THE BEST PG IN THE NBA…. SAY IT WITH US"                              | Hot Take   | Hot Take  | N/A        | Short, loud, all-caps — clear Hot Take signal                   |
| "He's averaging 35.4 pts/36min, second only to Wilt all time..."                        | Analysis   | Analysis  | N/A        | Multiple verifiable stats and comparative reasoning             |
| "This is legit the most shocking trade I have ever seen in 30 years"                    | Reaction   | Reaction  | N/A        | Time-anchored, emotional, no argument                           |
| "Despite 3 MVPs and a chip Jokic is still underrated"                                   | Hot Take   | Analysis  | 0.35       | Accolade-as-decoration; model reads "3 MVPs" as evidence        |
| "Anthony Davis has no weaknesses defensively, one of the most versatile defenders ever" | Hot Take   | Analysis  | 0.35       | Declarative confidence without evidence; absorbed into Analysis |

---

## Reflection

I designed the labels around intent (what a post is primarily trying to do), but the model learned surface features instead: sentence complexity, presence of numbers, comparative structure. For Reaction posts this was fine because Reaction has obvious surface signals. For Hot Take vs. Analysis, surface features are unreliable because NBA hot takes are often phrased in complete, structured sentences.

The Groq baseline hitting 0.94 macro F1 confirms the labels are sound. The gap between 0.73 and 0.94 is a data size and model capacity problem, not a labeling problem.

---

## Spec Reflection

**One way the spec helped:** Defining labels around intent before collecting data gave me a clear tiebreaker for ambiguous posts from the start. It kept annotation consistent even when the surface content was mixed.

**One way implementation diverged:** The spec used a priority rule (Analysis > Hot Take > Reaction) to handle edge cases. In practice, the priority rule and the intent question didn't always agree. A few posts with stats were labeled Analysis under the priority rule but clearly read as Hot Takes under intent framing. If I were doing this again I'd drop the priority rule entirely, since it introduced the exact "stat = Analysis" association that became the model's blind spot.

---

## AI Usage

**Instance 1: Label definition refinement**
I initially defined labels by content presence (e.g., "post contains stats = Analysis"). Claude flagged that this would cause stat-dressed hot takes to be mislabeled and suggested reframing around intent instead. I adopted the intent framing and updated all label definitions before annotating. This was a genuine change to my approach, not just wording.

**Instance 2: Error analysis**
After seeing the confusion matrix, I shared the 8 wrong predictions with Claude to identify the failure pattern. Claude identified two sub-patterns (structured-sentence hot takes and accolade-as-decoration hot takes) and noted that all confidence scores were near the 0.33 random baseline, indicating genuine model uncertainty. I used this analysis in the evaluation write-up; the wrong predictions and metrics came from my own training run.

**Instance 3: README structure and data section**
I asked Claude to help structure the README and draft the data collection section based on my planning.md and evaluation results. I then edited for conciseness and adjusted the tone throughout.

**Annotation assistance:** All 200 posts were labeled manually. Claude nor any other AI/LLM were used for annotation decisions.

---
