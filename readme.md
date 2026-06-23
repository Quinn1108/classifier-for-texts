# TakeMeter — Joe Rogan Instagram Content Classifier

A fine-tuned DistilBERT classifier that labels Joe Rogan Instagram captions as **Lifestyle / General Interest**, **Politics / Social Commentary**, or **Podcast Promotion**.

---

## Community & Motivation

Joe Rogan's Instagram (@joerogan, ~20M followers) produces a high volume of short-to-medium captions that span genuinely distinct registers: personal food and lifestyle observations, direct political opinions, and formulaic episode-announcement copy. The textual variation is stark enough that classification is meaningful — a fight-commentary post reads nothing like a podcast promo — yet the categories share a common narrator voice (first-person, casual, profanity-inclusive), which creates a real challenge for a model trained on tone rather than content.

A classifier that reliably separates these three roles answers a practical question: "Which hat is Joe wearing today?" This is useful for content-strategy analysis, sponsor-alignment research, and community filters that let followers opt into only the content type they care about.

---

## Label Definitions

| Label | Core Meaning | Key Signals |
|---|---|---|
| **Lifestyle / General Interest** | Personal observations, daily life, food, family, dogs, travel, casual reactions | No promotional CTA, no political stance; mentions meals, venues, family members, dogs, workouts |
| **Politics / Social Commentary** | Expressed political opinion, policy advocacy, commentary on social issues | References politicians, policies, government, elections, or social controversies; states a clear position or urgency |
| **Podcast Promotion** | Announcement of a new JRE episode; primary purpose is driving listens | Contains "available now on spotify," "episode available now," or "everywhere else"; introduces a guest; ends in a platform CTA |

**Hard edge cases:** Posts where guest-introduction warmth dominates before a trailing Spotify CTA were labeled **Podcast Promotion** (the CTA defines purpose). Posts referencing controversial figures (e.g., Terrence Howard) with a reflective editorial framing were labeled **Politics / Social Commentary** rather than Lifestyle, because the post stakes a position rather than describing an experience.

---

## Dataset

- **Source:** `joerogan_post_data_labeled.csv` — ~199 Instagram captions labeled manually
- **Label distribution:** Lifestyle/General Interest: 104 · Politics/Social Commentary: 42 · Podcast Promotion: 53
- **Split:** 70% train / 15% validation / 15% test (30 test examples: 15 Lifestyle, 7 Politics, 8 Podcast Promotion)

The class imbalance (Lifestyle is ~52% of the corpus) is mild enough that it doesn't require resampling, but it does make macro F1 the more honest primary metric — accuracy alone would be inflated by a majority-class guesser.

---

## Models

### Baseline: Llama-3.3-70B (Zero-Shot via Groq)

The baseline uses the Llama-3.3-70B instruction model prompted with the label definitions and asked to classify each caption zero-shot — no fine-tuning, no labeled examples in context. This is a strong baseline because the model has vast world knowledge (it knows who Joe Rogan is, what JRE is, what "available now on spotify" means) and can apply the label definitions directly.

### Fine-Tuned: DistilBERT-base-uncased

DistilBERT-base-uncased was fine-tuned on the 140-example training set for 3-class sequence classification. DistilBERT is a 66M-parameter distilled version of BERT, well-suited for short social-media text and fast iteration on small datasets.

---

## Evaluation Results

### Overall Accuracy

| Model | Accuracy (30-example test set) |
|---|---|
| Baseline (Llama-3.3-70B, zero-shot) | **0.80** |
| Fine-tuned (DistilBERT) | **0.50** |

The fine-tuned model performs 30 percentage points below the zero-shot baseline — a significant regression. The root cause is analyzed in detail below.

---

### Per-Class Metrics — Baseline (Llama-3.3-70B)

Per-class breakdowns were not individually logged for the baseline run; the aggregate accuracy of 0.80 (24/30 correct) represents its overall performance. Given that the fine-tuned model's failure is concentrated on Politics and Podcast Promotion (both recall = 0.00), the baseline's 6 additional correct answers almost certainly come from those two classes — the zero-shot model's world knowledge compensates for exactly the gap the fine-tuned model can't bridge.

---

### Per-Class Metrics — Fine-Tuned Model (DistilBERT)

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Lifestyle / General Interest | 0.50 | 1.00 | 0.67 | 15 |
| Politics / Social Commentary | 0.00 | 0.00 | 0.00 | 7 |
| Podcast Promotion | 0.00 | 0.00 | 0.00 | 8 |
| **Macro avg** | **0.17** | **0.33** | **0.22** | 30 |
| **Weighted avg** | **0.25** | **0.50** | **0.33** | 30 |

The fine-tuned model has collapsed into a single-class predictor: it assigns every test example to Lifestyle / General Interest, achieving exactly the Lifestyle base rate (50%) as its accuracy. Politics and Podcast Promotion recall are both 0.00.

---

### Confusion Matrix — Fine-Tuned Model

|  | **Predicted: Lifestyle** | **Predicted: Politics** | **Predicted: Podcast** |
|---|---|---|---|
| **True: Lifestyle / General Interest** | 15 | 0 | 0 |
| **True: Politics / Social Commentary** | 7 | 0 | 0 |
| **True: Podcast Promotion** | 8 | 0 | 0 |

The confusion matrix tells the whole story at a glance: every prediction lands in the Lifestyle column. There are no confusions *between* minority classes — the model never confuses Politics with Podcast Promotion, because it never predicts either. The entire error mass is a single directional failure: minority class → Lifestyle.

See also: [confusion_matrix.png](confusion_matrix.png)

---

## Error Analysis — 3 Specific Wrong Predictions

### Error 1: Short culturally-coded post read as Lifestyle

> **Text:** "timjdillon is a national treasure."
> **True label:** Politics / Social Commentary
> **Predicted:** Lifestyle / General Interest (confidence: 0.39)

This post is six words. Tim Dillon is a political comedian whose work is almost entirely political satire — calling him "a national treasure" is Rogan making a political-commentary-aligned endorsement. But the model has no world knowledge of who Tim Dillon is or what "national treasure" implies in context. To the model, this is indistinguishable from "marshallmaerogan is a special kid" (a Lifestyle post about Rogan's son). The failure is a **low-information-post problem**: the text alone provides no content signal, and the model correctly reflects that uncertainty — but defaults to the majority class rather than holding a reasonable distribution. To fix this, more short-form Politics examples (brief, culturally-loaded endorsements of political figures) would need to appear in training.

---

### Error 2: Podcast Promotion buried behind warm personal framing

> **Text:** "My man joshbrolin is a special human being. Fun and intelligent, friendly and soulful. I really enjoyed talking to him. Episode is out now on spotify and everywhere else."
> **True label:** Podcast Promotion
> **Predicted:** Lifestyle / General Interest (confidence: 0.42)

The "available on spotify" CTA is present and explicit — this is exactly the key signal defined in the taxonomy. Yet the model predicts Lifestyle. The reason is structural: three full sentences of warm, personal, first-person description ("special human being," "fun and intelligent," "really enjoyed talking to him") precede the single promotional sentence. The model appears to weight early context more heavily than the trailing CTA. Since Lifestyle posts are dominated by exactly this warm, personal, first-person tone, the opening registers as Lifestyle — and the final promotional clause doesn't override it. This is not a **labeling problem** (the post is unambiguously a promo by the spec's own definition); it is a **training-data distribution problem**: the training set likely underrepresents Podcast Promotion examples where promotional language comes last rather than first.

---

### Error 3: Politically-coded language diluted by casual phrasing

> **Text:** "We've got issues at the southern border. Our tax dollars are being used for unimaginable purposes. border jre wevegotissues"
> **True label:** Politics / Social Commentary
> **Predicted:** Lifestyle / General Interest (confidence: 0.46)

This is the most overtly political post among the errors — it explicitly names the southern border, references government spending, and uses the hashtag `wevegotissues` (a recurring political-commentary segment title). Yet the model still misfires. The likely cause: the hashtag string "wevegotissues" is not a natural-language phrase the model's vocabulary handles well, and the surrounding language ("We've got issues," "Our tax dollars") is casual-register political speech that doesn't match stereotypical political-text patterns (formal policy language, named politicians). The model has likely learned political content from examples with higher-density political signals (named politicians, policy terms) and hasn't generalized to the more colloquial register Rogan uses. The boundary here isn't inherently ambiguous — this post has a clear stance — but the model's decision boundary hasn't captured it. More casual-register political examples in training would help.

---

## Sample Classifications

The following table shows five posts passed through the fine-tuned model with their predicted labels and confidence scores. The first two are correctly predicted (true Lifestyle posts from the test set); the remaining three are errors.

| # | Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|---|
| 1 | "Elk steaks for dinner. Covered them in olive oil and traegergrills blackened Saskatchewan rub..." | Lifestyle / General Interest | **Lifestyle / General Interest** | 0.71 |
| 2 | "A beautiful day on the trails with my boy marshallmaerogan" | Lifestyle / General Interest | **Lifestyle / General Interest** | 0.68 |
| 3 | "timjdillon is a national treasure." | Politics / Social Commentary | Lifestyle / General Interest | 0.39 |
| 4 | "My man joshbrolin is a special human being... Episode is out now on spotify and everywhere else." | Podcast Promotion | Lifestyle / General Interest | 0.42 |
| 5 | "We've got issues at the southern border. Our tax dollars are being used for unimaginable purposes." | Politics / Social Commentary | Lifestyle / General Interest | 0.46 |

*Note: Confidence scores for correctly-predicted examples (#1, #2) are estimated from the model's softmax output; misclassified examples (#3–5) use logged confidence values.*

**Why prediction #1 is reasonable:** The post describes a specific cooking method (olive oil, cast iron skillet, internal temperature of 125°F), names a specific product (Traeger Grills), and ends with a sensory evaluation ("It was delicious"). This dense, first-person, activity-specific language with no promo CTA and no political stance is exactly what the Lifestyle definition targets — the model has the right intuition here even if it can't articulate the rule.

---

## Reflection: What the Model Captured vs. What I Intended

I intended the model to learn **content-based** distinctions: does this post describe a daily activity, express a political opinion, or announce an episode? What the model actually learned is closer to a **tone-based** heuristic: is this post first-person, casual, and conversational?

The problem is that all three categories share exactly that register — Joe Rogan writes all his posts in the same voice. The Lifestyle examples, the Politics examples, and the Podcast Promotion examples are all first-person, informal, and often warm or enthusiastic. The model's decision boundary captured the dominant surface-level feature of the corpus (casual tone) and correctly identified that Lifestyle is the most common casual-tone category. It then extended that to all inputs, because the minority-class distinguishing signals — political keywords, episode-announcement CTAs — weren't distributed consistently enough in training to override the tone prior.

What the model missed: **intent signals**. Lifestyle posts describe experiences. Politics posts state positions. Podcast posts drive action. These are structural-pragmatic differences, not purely lexical ones. A model trained on 140 examples without any explicit structural encoding (e.g., "does the post contain a call to action?") is unlikely to learn those distinctions reliably. The model also couldn't access world knowledge (who is Tim Dillon, what is a Joe Rogan Experience episode) that a zero-shot LLM uses fluently — which explains almost all of the baseline's advantage.

In short: the model overfit to **narrator tone** (casual, first-person) and missed the **communicative purpose** that actually defines the label boundaries.

---

## Spec Reflection

**One way the spec helped:** The spec's hard edge-case section — particularly the "UFC post with podcast promotion" and "political post that's also a podcast promotion" cases — forced me to commit to a priority rule (purpose over topic) before annotation. This made annotation faster and more consistent: whenever I encountered a mixed-signal post, I could apply the rule rather than re-litigating the definition from scratch.

**One way my implementation diverged:** The spec planned for four content categories (with Health/Biohacking as a fourth). After reviewing the actual data distribution, I collapsed Health into Lifestyle / General Interest rather than keeping it separate. The Ibogaine/wellness posts were too sparse to form a reliable training class (fewer than 10 examples), and the textual boundary between "health observation" and "lifestyle observation" was not consistent enough to annotate reliably at scale. Keeping a fourth underrepresented class would have hurt model performance without adding meaningful discriminative power — so the spec's four-class taxonomy became a three-class implementation.

---

## AI Usage

This project used AI assistance at two specific stages, with human oversight at both:

**1. Label stress-testing (label design phase)**
I prompted Claude with the label definitions and asked it to generate 10 synthetic boundary-case posts — examples that could plausibly fit two categories. It generated posts like a UFC recap that ended with a Spotify link, and a guest-focused post that read like a personal endorsement before revealing it was an episode announcement. I used these to refine the Podcast Promotion definition: the original definition said "template-like," but the stress tests showed that warm, personal guest introductions also count as promos when they include a platform CTA. I added that clarification to the key signals before annotating.

What Claude produced: 10 synthetic posts with label recommendations and reasoning.
What I changed: I rejected 2 of its recommended labels (it labeled a guest-intro-plus-CTA post as Lifestyle because of the personal tone, which contradicts my purpose-over-tone priority rule) and added one new edge case to the spec based on its output.

**2. Failure pattern identification (post-evaluation)**
After running evaluation, I provided Claude with the full list of 15 misclassified examples and asked it to identify patterns. It identified the directional collapse (all errors → Lifestyle), the structural "warm intro + buried CTA" pattern for Podcast Promotion errors, and the low-information-post problem for Politics errors. I verified each pattern against the actual predictions before including it in this report.

What Claude produced: five proposed failure patterns with brief explanations.
What I changed: I discarded one proposed pattern ("the model is confused by emoji/unicode") because the posts with encoding artifacts (â, ð) were no more likely to be wrong than clean posts — the correlation didn't hold up on manual inspection. The three patterns included in the error analysis above survived verification against the actual data.

**Annotation:** All 200 examples were labeled manually without AI pre-labeling. No LLM was used to generate or suggest labels during annotation.
