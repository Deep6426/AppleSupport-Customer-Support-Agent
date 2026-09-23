# AppleSupport Customer-Support Agent

An AI customer-support agent built using the **Customer Support on Twitter** dataset, focused on AppleSupport. The system combines traditional ML-based intent classification and semantic retrieval with **Generative AI** for grounded response generation.

The system:

- classifies customer messages into support intents
- predicts whether a message should be AUTO-handled or ESCALATED
- retrieves similar historical AppleSupport responses
- generates grounded support responses using Gemini
- evaluates classification and response quality
- analyzes common failure modes and proposes improvements

---

## Problem Framing

The goal is not to build a fully autonomous customer-support system. The project focuses on a conservative prototype where responses can be grounded in historical AppleSupport interactions.

"Good" means:

- relevant intent classification
- responses grounded in historical support behavior
- helpful and appropriate tone
- avoiding unsupported claims
- conservative escalation when confidence or intent coverage is insufficient

---

## Dataset

This project uses the **Customer Support on Twitter** dataset.

For this project, the `AppleSupport` brand was selected.

The paired dataset contains customer messages and corresponding historical AppleSupport responses.

---

## Intent Taxonomy

Customer messages are classified into 13 intents:

- `ios_update`
- `device_performance`
- `battery_power`
- `keyboard_input`
- `connectivity`
- `app_issues`
- `app_store_purchases`
- `itunes_media`
- `icloud_photos`
- `account_security`
- `calls_messaging`
- `hardware_accessories`
- `other_unclear`

---

## Golden Evaluation Set

A manually labelled Golden Evaluation Set of **200 examples** was created separately from the training data.

The set was used for:

- classification evaluation
- confusion analysis
- failure-mode analysis
- AUTO/ESCALATE evaluation

The sampling process used a candidate pool followed by manual labeling. The set should not be interpreted as statistically representative of all AppleSupport traffic.

The Golden Set was kept separate from the provisional weak labels used for classifier development.

---

## Classification Experiments

Multiple classification approaches were evaluated on the 200-example Golden Evaluation Set.

| Model | Accuracy | Macro F1 |
|---|---:|---:|
| Majority Baseline | 22.5% | 2.8% |
| TF-IDF + Logistic Regression | 55.0% | 0.48 |
| Sentence Embeddings + Logistic Regression | **57.5%** | **0.54** |

The sentence-embedding model performed best overall.

The classifier uses Sentence Transformers with `all-MiniLM-L6-v2` embeddings followed by balanced Logistic Regression.

The development training labels were refined using improved rule-based weak labeling. The manually labelled Golden Evaluation Set remained separate from training.

The updated classifier produced a modest improvement over the previous semantic classifier:

- Previous accuracy: **57.0%**
- Updated accuracy: **57.5%**
- Previous macro-F1: **0.527**
- Updated macro-F1: **approximately 0.54**

Performance still varies substantially between intents, showing that overall accuracy alone does not fully capture classifier weaknesses.

---

## AUTO / ESCALATE Policy

A conservative policy was used.

The following intents were included in the AUTO policy:

- `keyboard_input`
- `itunes_media`
- `connectivity`

Other intents were routed to ESCALATE.

The policy prioritizes safe handling over maximizing automation.

---

## Historical Response Retrieval

Historical AppleSupport responses are embedded using Sentence Transformers.

For an incoming customer message, the system retrieves the most similar historical support cases using cosine similarity.

The retrieved examples are then used as grounding context for response generation.

The retrieval component reuses historical customer-support interactions while keeping the Golden Evaluation Set excluded from the retrieval training pool.

---

## Response Generation

Response generation was tested with multiple approaches.

A local FLAN-T5-small experiment was conducted but produced weaker responses.

Gemini was subsequently used for grounded response generation, with historical AppleSupport responses provided as context.

Generation results:

- 30 examples were planned
- 17 responses were successfully generated before the Gemini API quota was exhausted
- all 17 generated responses were evaluated by the LLM judge

This limitation is documented rather than treating missing generations as model failures.

---

## Response Evaluation

Generated responses were evaluated using an LLM judge on five criteria:

- Relevance
- Groundedness
- Helpfulness
- Tone
- No unsupported claims

Mean scores across the 17 generated responses:

| Criterion | Mean |
|---|---:|
| Relevance | 4.00 / 5 |
| Groundedness | 4.94 / 5 |
| Helpfulness | 3.53 / 5 |
| Tone | 4.76 / 5 |
| No unsupported claims | 5.00 / 5 |
| Overall | 4.45 / 5 |

A small manual/assisted human evaluation of 10 examples produced an overall mean of **4.46 / 5**.

Human and LLM-judge overall scores had a Spearman correlation of **0.465** (`p = 0.175`, `n = 10`). This is preliminary because of the small sample size.

---

## Key Failure Modes

After refining the training-label rules, the remaining classification errors were concentrated around ambiguous intent boundaries.

### 1. `keyboard_input → other_unclear`

**Real example:**  
> "Seriously @AppleSupport do something bout these ugly ass question marks before I take my ass back to android..."

**Hypothesis:**  
The message refers to the keyboard/input bug indirectly and contains noisy language and unusual Unicode characters. This can make the semantic representation less specific and cause the classifier to fall back to `other_unclear`.

---

### 2. `other_unclear → ios_update`

**Real example:**  
> "@AppleSupport @115858"

**Hypothesis:**  
Very short or context-dependent messages contain too little information for reliable intent classification. Because many AppleSupport conversations in the dataset are related to software updates, the classifier can incorrectly assign these vague messages to `ios_update`.

---

### 3. `hardware_accessories → other_unclear`

**Real example:**  
> "poor support from @AppleSupport today trying to get my AirPods fixed 👎"

**Hypothesis:**  
Hardware and accessory-related issues are expressed using broad support language rather than specific hardware terminology. Short complaints such as this can therefore be absorbed by the broad `other_unclear` category.

---

### 4. `itunes_media → app_store_purchases`

**Real example:**  
> "@AppleSupport where do i find boost code from taylors album that i purchased from itunes?"

**Hypothesis:**  
iTunes/media purchases and App Store purchases share vocabulary such as "purchased", "download", and "account". Without stronger media-specific cues, the classifier can confuse the two categories.

---

### 5. `app_issues → other_unclear`

**Real example:**  
> "Y’all better fix both of my phones I’m tired of my fucking apps crashing fix that shit"

**Hypothesis:**  
Although the message explicitly mentions crashing apps, it contains multiple devices and highly informal language. The classifier may therefore represent it as a general device complaint rather than a specific third-party application issue.

---

These failure modes show that the remaining errors are primarily caused by short, noisy, ambiguous, or overlapping customer messages. In particular, `other_unclear` acts as a broad fallback category and absorbs several difficult examples.

---

## Misleading Headline Metric

The overall accuracy of **57.5%** is useful but does not tell the complete story.

Performance varies substantially across individual intents. Therefore, reporting only accuracy could hide poor performance on minority or difficult classes.

For this reason, the project also reports:

- macro precision
- macro recall
- macro F1
- weighted F1
- per-intent F1
- confusion matrices
- concrete failure examples

Macro-F1 is particularly useful here because it gives each intent equal importance rather than being dominated by the most frequent classes.

---

## One-Week Improvement Plan

The first improvement iteration refined the weak-labeling rules used to construct the development training labels. This produced a modest improvement from **57.0% to 57.5% accuracy** and from **0.527 to approximately 0.54 macro-F1**.

If given another week, the main priorities would be:

1. Use LLM-assisted weak labeling to improve training-label quality.
2. Expand and rebalance the Golden Evaluation Set.
3. Improve intent definitions and annotation guidelines.
4. Add hard-negative examples for commonly confused intents.
5. Evaluate stronger embedding models and lexical-semantic ensembles.
6. Reconsider the broad `other_unclear` category and split it if sufficient data supports doing so.
7. Improve retrieval quality with filtering and reranking.
8. Expand human evaluation and judge-agreement analysis.

---

## Limitations

- The Golden Evaluation Set contains 200 manually labelled examples and should not be treated as statistically representative of all AppleSupport traffic.
- Training labels are provisional weak labels rather than fully human-labelled data.
- The classifier still has difficulty with overlapping and ambiguous intents.
- `other_unclear` combines several types of difficult messages, including vague, multi-issue, image-only, and out-of-scope cases.
- Only 17 of 30 planned response-generation examples were successfully generated because the Gemini API quota was exhausted during the experiment.
- Human evaluation covered only 10 examples, so human–LLM judge agreement should be interpreted as preliminary.
- The project is a prototype and is not intended to autonomously handle real customer-support traffic.

---

## Decision Log

Key engineering decisions included:

- Selecting `AppleSupport` as the target brand.
- Using a 13-intent taxonomy.
- Keeping the Golden Evaluation Set separate from classifier development.
- Using transparent weak-label rules for development training labels.
- Using Sentence Transformers for semantic representations.
- Using Logistic Regression as a lightweight classifier.
- Using a conservative AUTO/ESCALATE policy.
- Using historical AppleSupport responses as retrieval context.
- Testing FLAN-T5-small before moving to Gemini for response generation.
- Evaluating generated responses with an LLM judge.
- Retaining human evaluation as a small preliminary validation.
- Refining weak-label rules after analyzing classification errors.
- Retaining `all-MiniLM-L6-v2` because the stronger embedding experiment was substantially more expensive computationally.
- Treating stronger models, LLM-assisted labeling, and hybrid classifiers as future improvements rather than adding unnecessary complexity to the current prototype.

---

## Reproducibility

The main implementation is contained in:

`AppleSupport_Customer_Support_Agent_FINAL_UPDATED.ipynb`

The notebook contains the complete workflow from dataset preparation through evaluation.

### Reproducing the Headline Results

The headline classification results can be reproduced in **under 15 minutes** using the development sample and the 200-example Golden Evaluation Set.

1. Open `AppleSupport_Customer_Support_Agent_FINAL_UPDATED.ipynb` in Google Colab.
2. Load the Customer Support on Twitter dataset.
3. Run the dataset preparation and AppleSupport filtering cells.
4. Run the intent-label preparation and classification cells.
5. Run the evaluation cells for the 200-example Golden Evaluation Set.

The main classification results are:

| Model | Accuracy | Macro F1 |
|---|---:|---:|
| Majority Baseline | 22.5% | 2.8% |
| TF-IDF + Logistic Regression | 55.0% | 0.48 |
| Sentence Embeddings + Logistic Regression | **57.5%** | **0.54** |

The notebook uses a 20,000-example development sample rather than processing the full dataset, as allowed by the assignment.

Gemini-based response generation and LLM judging are optional evaluation components and require a Gemini API key. They are not required to reproduce the headline classification results.

### Setup

The notebook is designed to run in **Google Colab**.

Required Python packages include:

- pandas
- numpy
- scikit-learn
- sentence-transformers
- google-genai
- matplotlib

A Gemini API key is required only for the Gemini-based generation and judging experiments.

The API key should be provided through the Colab Secrets mechanism rather than hard-coded in the notebook.
## Project Structure

```text
AppleSupport-Customer-Support-Agent/
│
├── AppleSupport_Customer_Support_Agent_FINAL_UPDATED.ipynb
└── README.md
