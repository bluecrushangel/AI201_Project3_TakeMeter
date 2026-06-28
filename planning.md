# TakeMeter — planning.md

> Complete this document before writing any implementation code.
> Your spec and annotation decisions are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Community

**What community did you choose?**

I chose the nba community for this project. Mainly: r/nba, r/nbatalk & r/nbadiscussion

**Why this community?**

I chose this community because I'm a big basketall fan and more importantly, a big NBA discourse fan.

**Why is it a good fit for a classification task?**

I believe this is a good fit for a classification task because NBA discourse has alot of content with alot of variety in what people talk about. I also intend to use the labels given in the example (hot_take, analysis, reaction), which suits the topic.

---

## Labels

**IMPORTANT NOTE**

I'm using the labels given as an example in the project description, however I'm reframing them around intent, rather than content presence.

Question that frames my label: What is this post primarily trying to do?

**Label 1 — : Analysis**

Build an argument with evidence the reader can verify

> Example post 1: Agreed. he's having an insane season and watching him play justifies his stats. He pretty much wraps up games midway through the 3rd quarter. I just checked the stats and this season he's averaging 35.4 points per 36 minutes (second only to Wilt all time) and 0.392 WS / 48 which is again the highest of all time. We may be witnessing probably the best regular season peak we've ever seen from a player, given the numbers. If he can keep this pace up, he'd have a comfortably better season than 2016 steph in terms of team wins and individual stats, and I'm saying this as a warriors fan.

> Example post 2: Westbrook is Top 9 in 3FG % at 43.6% with 3PA > 80 (Westbrook at 101 attempts) and Wide Open. His ranking is higher than Derrick White, Lamelo, Trey Murphy, Naz Reid, etc. Call him Wetbrook - his shot is Wet this year.

**Label 2 — : Hot take**

Assert a claim confidently, with or without token stats.

> Example post 1: JALEN BRUNSON IS THE BEST PG IN THE NBA…. SAY IT WITH US 

> Example post 2: Anthony Davis has no weaknesses defensively, legitimately one of the most versatile defenders ever. 

**Label 3 — : Reaction**

Express a feeling about something that just happened.

> Example post 1: This is legit the most shocking trade I have ever seen and I've been watching for over 30 years. Completely out of nowhere

> Example post 2: Legit done as a Mavs fan. This is the dumbest trade of all time! Completely unforgivable

---

## Hard Edge Cases

**What type of post will be genuinely ambiguous?**

Posts that has elements of multiple labels. For example, a post that makes a bold claim and cites one or two stats without building a full argument. This mainly makes it hard to distinguish between hot takes and analysis, which I predit the model would struggle with the most.

Example:
Westbrook is Top 9 in 3FG % at 43.6% with 3PA > 80 (Westbrook at 101 attempts) and Wide Open. His ranking is higher than Derrick White, Lamelo, Trey Murphy, Naz Reid, etc. Call him Wetbrook - his shot is Wet this year.

Why:
User is heavily asserting that Westbrook is much better than people think he is, but is doing it with an analytical edge. 

**How will you handle these during annotation?**

Add a priority system:

Analysis > Hot take > Reaction

Where posts featuring analysis features would be labeled analysis and so on.

---

## Data Collection Plan

**Where will you collect examples?**

3 subreddits: r/nba, r/nbatalk, r/nbadiscussion

**How many per label?**

~67 posts per label

**What will you do if a label is underrepresented after 200 examples?**

Since I'm manually scrapping label data, I'd manually add more examples.

---

## Evaluation Metrics

**Primary metric:**

Per-class F1 for each label

**Secondary metric:**

Confusion matrix

**Why accuracy alone is not enough:**

On a 3-class task, a random guesser already hits 33% accuracy, so a model can beat baseline while still completely failing on one class, per-class F1 catches things which accuracy won't.

---

## Definition of Success

**What performance would make this classifier genuinely useful?**

All three per-class F1 scores at 0.70 or higher, which would mean that the model is learning all three distinctions well.

**What would you accept as "good enough" for deployment?**

No class with F1 near 0, and overall accuracy meaningfully above the 33% random baseline. At minimum the model is doing something useful even if not all classes are at 0.70.

---
