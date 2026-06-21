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