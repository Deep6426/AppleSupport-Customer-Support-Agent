# AppleSupport Customer-Support Agent

An AI customer-support agent built using the Customer Support on Twitter dataset, focused on AppleSupport.

The system:
- classifies customer messages into support intents
- predicts whether a message should be AUTO-handled or ESCALATED
- retrieves similar historical AppleSupport responses
- generates grounded support responses
- evaluates classification and response quality

## Problem Framing

The goal is not to build a fully autonomous customer-support system. The project focuses on a conservative prototype where responses can be grounded in historical AppleSupport interactions.

"Good" means:
- relevant intent classification
- responses grounded in historical support behavior
- helpful and appropriate tone
- avoiding unsupported claims
- conservative escalation when confidence or intent coverage is insufficient

## Dataset

This project uses the **Customer Support on Twitter** dataset.

For this project, the `AppleSupport` brand was selected.

The paired dataset contains customer messages and corresponding historical AppleSupport responses.

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

## Golden Evaluation Set

A manually labelled Golden Evaluation Set of 200 examples was created separately from the training data.

The set was used for:
- classification evaluation
- confusion analysis
- failure-mode analysis
- AUTO/ESCALATE evaluation

The sampling process used a candidate pool followed by manual labeling. The set should not be interpreted as statistically representative of all AppleSupport traffic.

## Classification Experiments

Three classification baselines were evaluated.

| Model | Accuracy | Macro F1 |
|---|---:|---:|
| Majority Baseline | 22.5% | 2.8% |
| TF-IDF + Logistic Regression | 52.5% | 50.2% |
| Sentence Embeddings + Logistic Regression | 57.0% | 52.7% |

The sentence-embedding model performed best overall, although performance varied substantially between intents.

Per-intent F1 ranged from 0.167 to 0.800, showing that overall accuracy alone does not capture the weaknesses of the classifier.

## AUTO / ESCALATE Policy

A conservative policy was used.

The following intents were included in the AUTO policy:

- `keyboard_input`
- `itunes_media`
- `connectivity`

Other intents were routed to ESCALATE.

The policy prioritizes safe handling over maximizing automation.

## Historical Response Retrieval

Historical AppleSupport responses are embedded using Sentence Transformers.

For an incoming customer message, the system retrieves the most similar historical support cases using cosine similarity.

The retrieved examples are then used as grounding context for response generation.

## Response Generation

Response generation was tested with multiple approaches.

A local FLAN-T5-small experiment was conducted but produced weaker responses.

Gemini was subsequently used for grounded response generation, with historical AppleSupport responses provided as context.

Generation results:

- 30 examples were planned
- 17 responses were successfully generated before the Gemini API quota was exhausted
- all 17 generated responses were evaluated by the LLM judge

This limitation is documented rather than treating missing generations as model failures.

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

A small manual/assisted human evaluation of 10 examples produced an overall mean of 4.46 / 5.

Human and LLM-judge overall scores had a Spearman correlation of 0.465 (`p = 0.175`, `n = 10`). This is preliminary because of the small sample size.

## Key Failure Modes

The most common classification errors included:

1. `keyboard_input → other_unclear`
2. `device_performance → ios_update`
3. `app_store_purchases → other_unclear`
4. `hardware_accessories → other_unclear`
5. `other_unclear → ios_update`

These errors suggest that some intents overlap strongly in language, while vague customer messages remain difficult to classify reliably.

## One-Week Improvement Plan

If given another week, the main priorities would be:

1. Expand and rebalance the Golden Evaluation Set.
2. Improve intent definitions and annotation guidelines.
3. Add hard-negative examples for commonly confused intents.
4. Improve retrieval quality with better filtering and reranking.
5. Evaluate additional classification models.
6. Improve response-generation grounding.
7. Expand human evaluation and judge-agreement analysis.

## Reproducibility

The main implementation is contained in:

`AppleSupport_Customer_Support_Agent_FINAL.ipynb`

The notebook contains the complete workflow from dataset preparation through evaluation.

### Setup

The notebook is designed to run in Google Colab.

Required Python packages include:

- pandas
- numpy
- scikit-learn
- sentence-transformers
- google-genai
- matplotlib

A Gemini API key is required only for the Gemini-based generation and judging experiments.

## Project Structure

```text
AppleSupport-Customer-Support-Agent/
│
├── AppleSupport_Customer_Support_Agent_FINAL.ipynb
└── README.md
