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

## Labels

### Label 1: Depth Content

A post containing substantive narrative (typically 2+ sentences) that shares a personal experience, detailed opinion, event explanation, or knowledge-based content — the post has genuine information value beyond promotion or reaction.

**Examples:**
> "Went down to Ways2Well and did plasmapheresis— this is the stuff they pulled out of my blood. That yellow-orange liquid is plasma — it carries a lot of the inflammatory proteins, toxins, and byproducts that build up over time. They separate it out, remove what they don't want, and replace it so your system can function cleaner."

> "Update on wolves in Aspen, Colorado. Back in February I posted about wolves being released on my friend's ranch... since then just on his neighbor's property they've had 7 calves and one cow killed by wolves."

### Label 2: Podcast Promotion

A post whose primary purpose is announcing a new JRE episode — it follows a predictable template, introduces a guest with superlatives, and directs audiences to streaming platforms with minimal substantive content about the conversation itself.

**Examples:**
> "The great and powerful harlandwilliams is a one of a kind awesome human being. New episode available now on spotify and everywhere else."

> "My good friend of many years therealjeffreyross has a new special out on netflix. It's an amazing one man show that he's been working on for years. Our podcast is available now on spotify and everywhere else."

### Label 3: Lightweight Reaction

A post consisting of short phrases, exclamations, or emotional reactions with virtually no informational content — these are impulse posts that convey excitement, disbelief, or general sentiment without substantive text.

**Examples:**
> "Let's fucking go"

> "What a fucking night! ufcfreedom250"

---

## Hard Edge Cases

### The Ambiguous Case: "Hybrid Posts"

The most challenging edge case is posts that **mix promotional language with substantive content**.

**Example:**
> "I had a great time talking to zuck, and marshallmaerogan and I got to try out some awesome new AR devices that Meta is working on. Episode is out now on Spotify!"

**Why it's ambiguous:**
- Contains promotional phrase → signals Label 2
- Contains substantive narrative → signals Label 1

### How to Handle It During Annotation

Apply the **"Information Value Test"** :

> *If I removed the promotional phrase, would this post still contain something worth reading?*

- If **yes** → Label 1 (Depth Content) — the post stands on its own as interesting content
- If **no** → Label 2 (Podcast Promotion) — the promotion is the only reason this post exists

For the example above: Removing the promotional phrase still leaves a description of testing AR devices → **Label 1 (Depth Content)** .

### Documentation Practice

All ambiguous cases will be logged in `error_analysis.md` with:
- The original text
- The label chosen and why
- The runner-up label and why it was rejected

---

## Data Collection Plan

### Data Source

All data comes from the provided CSV file `joerogan_post_metadata.csv`, which contains ~200 Instagram captions from @joerogan's public posts.

### Sampling Strategy

| Label | Target Count | Justification |
|-------|-------------|---------------|
| Depth Content | 70-80 | Rich, substantive posts are the "signal" in the noise |
| Podcast Promotion | 50-60 | The most frequent type of post based on data inspection |
| Lightweight Reaction | 50-60 | Short, emotional posts with minimal content |

**Total: ~200 examples**

### If a Label is Underrepresented

If after reviewing the data I find fewer than 40 examples of any label:

1. **First check:** Review borderline cases to see if misclassification is artificially reducing count
2. **If still low:** Consider merging the underrepresented label with a related label (e.g., if Lightweight Reaction is too small, merge with Podcast Promotion into a "Shallow Content" category)
3. **Document the decision:** Explain why the label was rare and how handling it addresses the "mutually exclusive and exhaustive" requirement

Based on review of the data, all three labels appear sufficiently represented.

### Train/Validation/Test Split

- **Training:** 140 examples (70%)
- **Validation:** 30 examples (15%)
- **Test:** 30 examples (15%)

---

## Evaluation Metrics

### Primary Metric: **Macro F1-Score**

**Why this is the right choice:**

The dataset has potentially **unbalanced labels** (Depth Content may appear less frequently than Podcast Promotion). Accuracy would be misleading — if the model simply predicts "Podcast Promotion" for everything, it might still achieve ~40% accuracy, which looks acceptable but is actually useless.

Macro F1-score calculates F1 per label independently and averages them **without weighting by frequency**. This means:
- The model cannot "cheat" by ignoring rare labels
- Poor performance on Depth Content will visibly hurt the score
- True performance across all categories is transparent

### Secondary Metrics:

| Metric | Purpose |
|--------|---------|
| **Per-class Precision** | Tells me if labels are being over-predicted (false positives) |
| **Per-class Recall** | Tells me if labels are being missed (false negatives) |
| **Confusion Matrix** | Reveals which labels are being confused with each other (e.g., Depth Content vs. Podcast Promotion) |

### Why These Metrics Are the Right Ones for This Task

| Scenario | What It Would Look Like | Why It Matters |
|----------|------------------------|----------------|
| Model predicts "Promotion" for everything | Accuracy ~40%, Macro F1 ~0.27 | Looks decent on accuracy, but fails entirely on two labels — a deployed model would flag substantive posts as promotion |
| Model performs well on 2 labels, fails on 1 | Accuracy ~70%, Macro F1 ~0.55 | Accuracy hides the failure on Depth Content — users would miss the most valuable posts |
| Balanced performance across all 3 | Accuracy ~65%, Macro F1 ~0.65 | True balanced performance — the goal for a usable tool |

---

## Definition of Success

### "Good Enough" for Deployment

| Metric | Target | Justification |
|--------|--------|---------------|
| Macro F1 | **≥ 0.70** | Indicates balanced performance across all three categories |
| Per-class Recall | **≥ 0.65** for each | No label is systematically missed — users won't lose valuable content |
| Per-class Precision | **≥ 0.65** for each | No label is systematically over-predicted — users won't be flooded with misclassified content |

### What This Means in Practice

If these targets are hit:
- The model can reliably identify Depth Content posts for a "what's worth reading" filter
- The model can filter out promotional noise without flagging substantive content
- A community tool could trust the classification to surface interesting posts to users

### What I Would Accept as "Good Enough"

Even with Macro F1 around **0.55–0.65**, the model could still be useful as:

1. **A pre-filter** — labeling obvious promotion posts for removal, leaving the more ambiguous cases to human review
2. **A trend monitor** — tracking shifts in content mix over time (what percentage of posts are Depth Content vs. Promotion vs. Reaction) even if individual labels aren't perfect
3. **A supportive tool** — not replacing human judgment, but helping users triage content more efficiently

### Minimum Viable Performance

| Metric | Minimum | Why |
|--------|---------|-----|
| Macro F1 | ≥ 0.55 | Below this, the model adds no value over random guessing with label distribution |
| Per-class Recall | ≥ 0.50 for each | Below 0.50, the model is missing more than half of the instances for a category, making it unreliable for surfacing content |
| Per-class Precision | ≥ 0.50 for each | Below 0.50, more than half of predictions are wrong, causing user frustration |

**Final judgment:** Macro F1 ≥ 0.65 with all per-class metrics ≥ 0.60 is the threshold where I would consider this classifier genuinely useful for real-world application.

## AI Tool Plan

This project uses AI tools at three specific points in the workflow. Each is a tool-assisted step where AI generates suggestions, but human judgment makes the final call.

---

### 1. Label Stress-Testing

**Before annotating 200 examples**, I will use an LLM to stress-test the label definitions by generating synthetic boundary cases.

**Process:**

1. Take the label definitions, key signals, and hard edge cases from this document
2. Prompt the LLM (Claude or ChatGPT) with:
   > "Here are three label definitions for classifying Joe Rogan Instagram posts. Generate 10 posts that sit at the boundary between two labels — cases where the definition alone doesn't make the classification obvious. For each, explain which label you'd choose and why."
3. Review each generated example and attempt to classify it using my definitions
4. If any generated example is genuinely unclassifiable or fits two labels equally well:
   - The label definitions need tightening
   - Adjust the definitions or add a new key signal before annotation begins

**Why this matters:**

The edge cases I've already identified (Substantive Promotion, Reaction with a Guest Name, etc.) come from my reading of the data. But there may be edge cases I haven't thought of. Stress-testing with synthetic examples forces me to confront weaknesses in the taxonomy before I've invested hours in annotating 200 real examples. It's a low-cost way to catch definitional problems early.

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
   > "Here are misclassified examples from a text classifier trained to label Joe Rogan Instagram posts as Depth Content, Podcast Promotion, or Lightweight Reaction. For each, the model's prediction and the true label are shown. Identify 3–5 patterns or failure modes. Are certain labels consistently confused? Does the model rely on the wrong signals?"
4. Use the LLM's identified patterns as a starting point for my own analysis
5. Verify each pattern against the actual data:
   - Manually review 3–5 examples per pattern to confirm it exists
   - If the pattern is real, include it in the evaluation report with specific examples
   - If the pattern is an LLM hallucination, discard it

**What I'll look for (verification checklist):**

- Are Depth Content posts with short length consistently misclassified as Lightweight Reaction?
- Are Podcast Promotion posts missing the "available now" template (e.g., live event announcements) being miscategorized?
- Are substantive posts ending with promotional phrases being misclassified as Promotion?
- Is the model over-relying on a specific key signal (length, specific words) and missing context?

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
