# Sentiment Analysis Study

This repository contains a small study comparing several sentiment analysis approaches on the same set of product reviews. It includes individual notebooks for TextBlob, AFINN, and VADER, as well as a consolidation notebook that visualizes how label distributions differ across methods and explores topic-specific slices of the data.

## Contents
- data/
  - reviews.xlsx — source reviews used by the method notebooks
  - sentiment-analysis-all.csv — a consolidated sheet combining manual and automated labels plus the review text
- textblob.ipynb — computes TextBlob polarity/subjectivity, derives labels, and plots distributions
- afinn.ipynb — computes AFINN scores, derives labels, and plots distributions
- vader.ipynb — computes VADER neg/neu/pos/compound, derives labels, and plots distributions
- distribution.ipynb — reads the consolidated CSV and compares label distributions across analyses; includes two small topical explorations

## Methods of Analysis
This study compares multiple ways to determine sentiment. Each method has distinct characteristics, strengths, and limitations. The distribution notebook references the following sources of labels:

- Manual ("Lawrence's Manual Label")
  - Description: A human-assigned categorical label per review (Positive, Neutral, Negative).
  - Notes: Serves as a reference/baseline. Human judgment can capture context, sarcasm, and domain nuances that automated methods miss, but can be subjective and time-consuming.

- TextBlob
  - How it works: Uses rule/lexicon-based analysis (PatternAnalyzer) to compute two continuous scores:
    - Polarity in [-1.0, 1.0] (negative to positive)
    - Subjectivity in [0.0, 1.0] (objective to subjective)
  - Labeling in this repo: We map polarity to a categorical label with thresholds: > 0.2 = Positive, < -0.2 = Negative, otherwise Neutral (see textblob.ipynb).
  - Strengths: Simple, fast, easy to interpret; provides subjectivity as well as polarity.
  - Limitations: Limited context handling; sarcasm/negation can be challenging; domain-specific language may require customization.

- AFINN
  - How it works: A lexicon assigning integer sentiment scores to words (roughly -5 to +5). A review’s score is the sum of word scores.
  - Labeling in this repo: Score > 0 = Positive, < 0 = Negative, 0 = Neutral (see afinn.ipynb). We also compute a normalized score per token to reduce text-length bias.
  - Strengths: Transparent and fast; easy to tune by editing word lists.
  - Limitations: Additive and context-agnostic; can be sensitive to length and miss compositional meaning, irony, or domain terms.

- VADER
  - How it works: A rule/lexicon method tuned for short, informal text. Returns proportions for neg/neu/pos and a normalized compound score in [-1, 1].
  - Labeling in this repo: Using common thresholds on the compound score: ≥ 0.05 = Positive, ≤ -0.05 = Negative, otherwise Neutral (see vader.ipynb).
  - Strengths: Handles punctuation emphasis, emoticons, slang; widely used for social-media style text.
  - Limitations: Still lexicon/rule-based; struggles with complex context, sarcasm, and domain-specific vocabulary.

- LLMs (ChatGPT, Claude)
  - How it fits here: The consolidated CSV references columns for ChatGPT and Claude labels. Where present, these are model-assigned categorical labels.
  - Strengths: Better at contextual understanding, handling negation, and nuanced phrasing than classic lexicon methods.
  - Limitations: Can be sensitive to prompt design; may be inconsistent across runs; closed-model outputs may be non-deterministic and require careful evaluation.

## Consolidated Distribution and Explorations
The distribution.ipynb notebook:
- Normalizes label text (e.g., different casing/spacing) into {Positive, Neutral, Negative} to compare methods fairly.
- Plots a grouped bar chart showing the count of Positive/Neutral/Negative labels for each analysis method (Manual, TextBlob, AFINN, VADER, ChatGPT, Claude when available).
- Saves figures into the data/ folder:
  - sentiment_distribution_across_analyses.png
  - positive_reviews_with_taste.png
  - negative_reviews_with_leak_or_crack.png
- Exploratory slices:
  - Positive reviews mentioning "taste" (case-insensitive)
  - Negative reviews mentioning potential defects: "leak" or "crack"

## Data
- Source file: data/reviews.xlsx — used by the method notebooks to compute scores and labels.
- Consolidated file: data/sentiment-analysis-all.csv — expected to contain the review text and columns for each labeling method. Column names referenced include:
  - "Review Text"
  - "Lawrence's Manual Label"
  - "TextBlob Label" or derived labels from textblob.ipynb
  - "AFINN Label" or derived labels from afinn.ipynb
  - "Vader" / "VADER_label" (note: naming can differ; distribution.ipynb includes normalization to handle minor differences)
  - "ChatGPT", "Claude" if available

## Running the Notebooks
1. Create and activate a virtual environment (optional but recommended).
2. Install dependencies, for example:
   - pandas, matplotlib, seaborn
   - textblob
   - afinn
   - vaderSentiment
   - openpyxl (for reading Excel)
3. Ensure the data files exist in data/ as listed above.
4. Open the notebooks in JupyterLab/VS Code and run cells top-to-bottom:
   - textblob.ipynb
   - afinn.ipynb
   - vader.ipynb
   - distribution.ipynb

Notes:
- Some notebooks derive label columns (e.g., AFINN Label, TextBlob_label, VADER_label) for visualization. The consolidated CSV used by distribution.ipynb may come from merging outputs or from an existing curated file.
- distribution.ipynb is defensive about missing columns: if a method’s column is absent, it fills with NAs and continues, so plots still render with available methods.

## Interpretation Guidelines
- Expect differences between methods, especially in borderline/neutral reviews.
- Lexicon-based methods (TextBlob, AFINN, VADER) can disagree due to scoring schemes and thresholds.
- Manual labels may capture context and intent better but can vary by annotator.
- LLM labels may align more closely with human judgment on nuanced text, but require careful prompt and consistency checks.

## Limitations and Next Steps
- Domain adaptation: product-review language may require tuned lexicons or thresholds.
- Class imbalance: distributions can be skewed; consider stratification or metrics beyond raw counts (precision/recall against manual labels).
- Reproducibility: record exact package versions if results need to be replicated precisely.
- Potential extensions:
  - Add a notebook comparing each method to manual labels (confusion matrices, agreement metrics like Cohen’s kappa).
  - Incorporate additional models (e.g., transformer-based classifiers) for a model-based baseline.
