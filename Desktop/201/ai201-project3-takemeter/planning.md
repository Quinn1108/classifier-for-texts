# ai201-project3-takemeter — planning.md

## Community: 

<!-- What community did you choose and why? Why is this community a good fit for a classification task — what makes the discourse varied enough to be interesting? -->

**Why This Community?**

**1. High content diversity**

Joe Rogan's Instagram covers distinctly different domains:

MMA/UFC (~30%): fight commentary, event reactions, athlete shoutouts\
Health/Biohacking (~20%): plasma exchange, diet, fitness, recovery protocols\
Podcast promotion (~25%): new episode announcements with formulaic language\
Hunting/Outdoor (~10%): bowhunting, wild game cooking\
Politics (~10%): election commentary, policy opinions\

**2. Textual variation is stark**

Different topics produce radically different text patterns:

UFC posts: short, exclamation-heavy, profanity ("What a fucking night!")\
Health posts: long-form narrative, technical terms ("plasma", "inflammation", "recovery")\
Promotion posts: fixed templates ("The great and powerful X... available now on spotify")\
This makes classification meaningful — the model isn't splitting hairs over subtle differences.

**3. Practical application**

Rogan has ~20M Instagram followers. A classifier that distinguishes his content roles answers: "Which hat is Joe wearing today?" — useful for content strategy, audience targeting, and sponsor alignment.

**4. Text-only viability**

Despite Instagram being visual, Rogan's captions are substantive, complete texts, not emoji-only filler. Pure text classification works.

**Why It Fits a Classification Task**

A good classification task requires clear, observable differences between categories. Rogan's posts deliver exactly that — the language of a fight commentator is nothing like the language of a biohacker, and both are worlds apart from a formulaic podcast promo.

200 labeled examples are sufficient because the signal is strong, not buried in ambiguity

# Project Plan: Joe Rogan Instagram Content Classifier

---

## Labels

### Label 1: Lifestyle / General Intereststyle / General Interest

A post about everyday Lifestyle / General Interest, personal experiences, or general observations — these are glimpses into Joe Rogan's daily world rather than announcements, reactions, or promotional content. They focus on food, family, dogs, travel, restaurants, personal routines, or casual thoughts. The tone is reflective, appreciative, or simply observational.

**Key signal:** Mentions daily activities — cooking, meals, specific restaurants, family members (marshallmaerogan), dogs, weather, travel, or personal routines. Uses descriptive language about food or experiences. No technical vocabulary. No promotional phrases ("available now," "new episode"). No fight terminology. Length varies but content is personal and relatable.

**Examples:**
> "Elk steaks for dinner. Covered them in olive oil and traegergrills blackened Saskatchewan rub. Cooked it 225 until it got an internal temperature of 125 then finished it on a cast iron skillet. Served it with Kimchi. It was delicious."

> "A beautiful day on the trails with my boy marshallmaerogan"

> "Torrisi in NYC. About as good as it's possible for Italian food to be. Fucking sensational."

> "Broke in my new traegergrills with some elk back straps."

---

### Label 2: Politics / Social Commentary

A post expressing a political opinion, commenting on current events, or advocating for a cause. These posts are usually medium-length, state a clear position, and may reference politicians, policies, or social issues. The tone is serious or urgent. Often involves advocacy or critique of government actions.

**Key signal:** Contains "Trump," "Biden," "RFK," "election," "president," "government," "public land," "wolf reintroduction," "vaccine," or "Ibogaine." Often includes a call to action or strong stance on a controversial topic.

**Examples:**
> "We can't let them sell public land. It's a short sighted solution with massive long term consequences. Public land is ours. All of ours, and they're trying to sell it without our consent."

> "Remote viewing is real. Aliens exist, and the USA has at least 10 crafts that are from non human intelligence. All according to renowned physicist Hal Puthoff."

---

### Label 3: Podcast Promotion

A post announcing a new JRE episode. These follow a predictable template: introduce a guest with superlatives, briefly describe the episode (often vaguely), and direct to streaming platforms. The primary purpose is driving listeners, not sharing substantive content. The same structure repeats across hundreds of posts.

**Key signal:** Contains "available now on spotify," "new episode," "episode available now," or "available on youtube." Follows the template: "The great and powerful X... Episode available now." Lacks detailed content about the actual conversation.

**Examples:**
> "The great and powerful harlandwilliams is a one of a kind awesome human being. New episode available now on spotify and everywhere else."

> "Episode 2509 calebhammercomposer drops financial science joeroganexperience available now on spotify"

---

## Hard Edge Cases

### Ambiguous Case 1: "UFC Post with Podcast Promotion"

**Example:**
> "Fun times on the Fight Companion! With mattserrabjj brendanschaub bryancallen and jamievernon. Available now on spotify."

**Why it's ambiguous:**
- "Fight Companion" and fighter names → signals UFC
- "Available now on spotify" → signals Podcast Promotion
- Short length, no episode number or guest introduction

**Resolution:** The primary purpose is promoting the Fight Companion podcast episode. The UFC content is the *topic*, not the *purpose*. The post exists to drive listeners, not to discuss a fight → **Label 3 (Podcast Promotion)** .

---

### Ambiguous Case 2: "Political Post That Happens to Be a Podcast Promotion"

**Example:**
> "I had a great time talking to zuck, and marshallmaerogan and I got to try out some awesome new AR devices that Meta is working on. Episode is out now on Spotify!"

**Why it's ambiguous:**
- Describes AR devices and Meta → could be Politics (tech policy) or just tech chat
- Ends with "Episode out now" → signals Podcast Promotion
- No explicit political stance

**Resolution:** The post is primarily a description of a fun experience (trying AR devices). The podcast promotion is secondary. There is no political stance being expressed → **Label 3 (Podcast Promotion)** (it's promoting an episode that happens to discuss tech).

---

### Ambiguous Case 3: "Health Post That Looks Like Politics"

**Example:**
> "One dose of Ibogaine has an 80% rate of freeing people from addiction... A 2024 Stanford-led trial found magnesium-ibogaine treatment led to 88% reduction in PTSD symptoms."

**Why it's ambiguous:**
- Discusses Ibogaine therapy → could be health/wellness content (which I'm not explicitly labeling)
- Could be seen as political advocacy for drug policy reform
- No explicit call to action or political figure

**Resolution:** This post is about vaccine, thus controversial topics, so it would be **Label 2: Politics / Social Commentary**
---

### How to Handle Ambiguous Cases During Annotation

**Step 1:** Log the case in `error_analysis.md` with:
- The original text
- The label chosen and why
- The runner-up label and why it was rejected

**Step 2:** For each ambiguous case, note which signal dominated the decision. This provides material for the failure mode analysis section of the final report.

**Step 3:** If more than 10% of posts are ambiguous, revisit the taxonomy definition. A good taxonomy has clear boundaries; if too many posts are borderline, the labels themselves may need adjustment.

---

## Data Collection Plan

### Data Source

All data comes from the provided CSV file `joe_roegan_text_labeled - Sheet1.csv`, which contains ~200 Instagram captions from @joerogan's public posts.

### Sampling Strategy

| Label | Target Count | Justification |
|-------|-------------|---------------|
| Lifestyle / General Intereststyle / General Interest | 70-80 | Most common type — food, family, daily Lifestyle / General Interest, travel |
| Politics / Social Commentary | 40-50 | Less frequent but significant — controversial topics get high engagement |
| Podcast Promotion | 70-80 | Extremely frequent — formulaic episode announcements |

**Total: ~200 examples**

### If a Label is Underrepresented

If after reviewing the data I find fewer than 30 examples of any label:

1. **First check:** Review borderline cases to see if misclassification is artificially reducing count
2. **If still low:** Consider whether the label can be split/merged, or whether the data simply doesn't contain enough examples for a fair evaluation
3. **Document the decision:** Explain what adjustments were made and why

Based on review of the data, all labels appear sufficiently represented.

### Train/Validation/Test Split

- **Training:** 140 examples (70%)
- **Validation:** 30 examples (15%)
- **Test:** 30 examples (15%)

---

## Evaluation Metrics

### Primary Metric: **Macro F1-Score**

**Why this is the right choice:**

The dataset has potentially **unbalanced labels** (some topics appear more frequently than others). Accuracy would be misleading — if the model simply predicts "UFC" for everything, it might still achieve ~35-40% accuracy, which looks acceptable but is actually useless.

Macro F1-score calculates F1 per label independently and averages them **without weighting by frequency**. This means:
- The model cannot "cheat" by ignoring rare labels
- Poor performance on Politics posts will visibly hurt the score
- True performance across all categories is transparent

### Secondary Metrics:

| Metric | Purpose |
|--------|---------|
| **Per-class Precision** | Tells me if labels are being over-predicted (false positives) |
| **Per-class Recall** | Tells me if labels are being missed (false negatives) |
| **Confusion Matrix** | Reveals which labels are being confused with each other (e.g., UFC vs. Podcast Promotion, Politics vs. Health) |

### Why These Metrics Are the Right Ones for This Task

| Scenario | What It Would Look Like | Why It Matters |
|----------|------------------------|----------------|
| Model predicts "UFC" for everything | Accuracy ~35%, Macro F1 ~0.20 | Looks terrible on F1 — the metric exposes the failure immediately |
| Model performs well on 2 labels, fails on 1 | Accuracy ~75%, Macro F1 ~0.58 | Accuracy hides the failure on the rare label — users would miss important Politics content |
| Balanced performance across all 3 | Accuracy ~70%, Macro F1 ~0.70 | True balanced performance — the goal for a usable tool |

---

## Definition of Success

### "Good Enough" for Deployment

| Metric | Target | Justification |
|--------|--------|---------------|
| Macro F1 | **≥ 0.70** | Indicates balanced performance across all four categories |
| Per-class Recall | **≥ 0.65** for each | No label is systematically missed — users won't lose valuable content |
| Per-class Precision | **≥ 0.65** for each | No label is systematically over-predicted — users won't be flooded with misclassified content |

### What This Means in Practice

If these targets are hit:
- The model can reliably filter Joe Rogan's posts by topic
- Users can follow only UFC content, or only Politics content, or skip Podcast Promotions
- A community tool could present a "topic filter" to help fans find what they care about

### What I Would Accept as "Good Enough"

Even with Macro F1 around **0.55–0.65**, the model could still be useful as:

1. **A pre-filter** — labeling obvious UFC or Podcast Promotion posts, leaving ambiguous cases to human review
2. **A trend monitor** — tracking shifts in content mix over time (what percentage of posts are UFC vs. Politics vs. Promotion)
3. **A supportive tool** — not replacing human judgment, but helping users triage content more efficiently

### Minimum Viable Performance

| Metric | Minimum | Why |
|--------|---------|-----|
| Macro F1 | ≥ 0.55 | Below this, the model adds no value over random guessing with label distribution |
| Per-class Recall | ≥ 0.50 for each | Below 0.50, the model is missing more than half of the instances for a category, making it unreliable for surfacing content |
| Per-class Precision | ≥ 0.50 for each | Below 0.50, more than half of predictions are wrong, causing user frustration |

**Final judgment:** Macro F1 ≥ 0.65 with all per-class metrics ≥ 0.60 is the threshold where I would consider this classifier genuinely useful for real-world application.

---

## AI Tool Plan

This project uses AI tools at three specific points in the workflow. Each is a tool-assisted step where AI generates suggestions, but human judgment makes the final call.

---

### 1. Label Stress-Testing

**Before annotating 200 examples**, I will use an LLM to stress-test the label definitions by generating synthetic boundary cases.

**Process:**

1. Take the label definitions, key signals, and hard edge cases from this document
2. Prompt the LLM (Claude or ChatGPT) with:
   > "Here are four label definitions for classifying Joe Rogan Instagram posts: UFC/MMA, Politics/Social Commentary, and Podcast Promotion. Generate 10 posts that sit at the boundary between two labels — cases where the definition alone doesn't make the classification obvious. For each, explain which label you'd choose and why."
3. Review each generated example and attempt to classify it using my definitions
4. If any generated example is genuinely unclassifiable or fits two labels equally well:
   - The label definitions need tightening
   - Adjust the definitions or add a new key signal before annotation begins

**Why this matters:**

The edge cases I've already identified come from my reading of the data. But there may be edge cases I haven't thought of. Stress-testing with synthetic examples forces me to confront weaknesses in the taxonomy before I've invested hours in annotating 200 real examples. It's a low-cost way to catch definitional problems early.

**What I'll do with problematic generated examples:**

- If an example is genuinely unclassifiable, I'll add a new hard edge case section for it
- If a generated example reveals that two labels overlap, I'll revise the definitions to clarify the boundary
- I'll document which label definitions were revised based on stress-testing

**Output:** A section in `planning.md` titled "Label Stress-Test Results" listing the generated examples and which definitions were tightened as a result.

---

### 2. Annotation Assistance

**Decision:** I will **not** use an LLM to pre-label examples.

**Reasoning:**

This is a small-scale annotation task (200 examples) with labels that are designed to be simple enough for consistent human judgment. Introducing LLM pre-labeling adds overhead (tracking which examples were pre-labeled, verifying correctness) without a clear benefit.

The key risk is that pre-labeling creates a bias in the final dataset. If I review LLM-generated labels, I'm more likely to accept or reject them in ways that align with the LLM's error patterns. This is the same problem that affects "AI-assisted annotation" in academic settings — the human reviewer becomes less critical.

**Alternative approach:**

I will annotate all 200 examples manually, using the taxonomy and edge case resolutions defined above. This ensures:
- Consistent application of the taxonomy
- No hidden bias from pre-labeling
- Clear documentation of my own annotation process

**If I change my mind later (pre-labeling for pilot batch):**

I'll use the `classifier.py` pipeline with the zero-shot LLM (Llama-3.3-70B via Groq) to generate initial predictions on 20 pilot examples. I'll then review these predictions against my own manual labels, using the agreement/disagreement to refine the taxonomy further. This would be disclosed as:

> "20 pilot examples were pre-labeled by the zero-shot classifier (Llama-3.3-70B) and reviewed by the annotator. The final taxonomy was adjusted based on disagreements. The remaining 180 examples were annotated without AI assistance."

---

### 3. Failure Analysis

**After the model evaluation**, I will use an LLM to help identify patterns in the wrong predictions.

**Process:**

1. Run the fine-tuned DistilBERT model on the test set (30 examples)
2. Collect all misclassified examples (the model's label + the true label)
3. Feed the list of misclassifications to an LLM (Claude or ChatGPT) with the prompt:
   > "Here are misclassified examples from a text classifier trained to label Joe Rogan Instagram posts as UFC/MMA, Politics/Social Commentary, Podcast Promotion, or Health/Biohacking/Lifestyle / General Intereststyle. For each, the model's prediction and the true label are shown. Identify 3–5 patterns or failure modes. Are certain labels consistently confused? Does the model rely on the wrong signals?"
4. Use the LLM's identified patterns as a starting point for my own analysis
5. Verify each pattern against the actual data:
   - Manually review 3–5 examples per pattern to confirm it exists
   - If the pattern is real, include it in the evaluation report with specific examples
   - If the pattern is an LLM hallucination, discard it

**What I'll look for (verification checklist):**

- Are UFC posts with short exclamations being misclassified as Podcast Promotion?
- Are Politics posts being confused with Health posts (e.g., Ibogaine advocacy as either)?
- Are Podcast Promotion posts being misclassified as UFC because they mention fighters?
- Does the model over-rely on single keywords and miss context?

**Why this matters:**

The LLM can quickly spot patterns across 10+ examples that I might miss reviewing one by one. It's a pattern-finding aid, not a replacement for my own judgment. The final analysis and conclusions are still mine — the LLM just helps me see the data from a different angle.

**Output:** A section in `evaluation_report.md` titled "Failure Mode Analysis" that includes:
- 3–5 failure patterns identified (with LLM's suggested patterns noted)
- Verification notes confirming each pattern against the actual test data
- 3+ specific examples of wrong predictions with my own analysis of why they went wrong
- A reflection on what the model learned vs. what I intended it to learn

---

### Disclosure Statement

AI tools will be used in this project as follows:

| Phase | Tool | Purpose | Human Oversight |
|-------|------|---------|-----------------|
| Label design | Claude / ChatGPT | Stress-testing label definitions with synthetic boundary cases | Full — I review each generated example and decide if definitions need revision |
| Annotation | None | Manual annotation of all 200 examples | N/A — no AI assistance used |
| Failure analysis | Claude / ChatGPT | Identifying patterns in misclassified predictions | Full — I verify all patterns against actual data and make final conclusions |
| Code generation | None | All code written manually | N/A |