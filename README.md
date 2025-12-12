# Game Length and Team Performance: Analyzing Win Rates in Professional League of Legends

**Author:** Albert Zhang
**Website:** [League of Legends Game Analysis](https://seagull-angle.github.io/League_of_Legends_Game_Analysis/)

---

## Introduction

### Dataset Overview

This project analyzes professional League of Legends esports matches from the 2022 season, using data compiled by Oracle's Elixir. The dataset contains **150,588 rows** and **164 columns**, with each match generating 12 rows (10 players + 2 team summaries). We focus on **team-level data** (25,098 team records) with performance statistics including gold, kills, deaths, creep score, and experience at 10, 15, 20, and 25-minute intervals.

Key columns include: `teamname`, `result` (win/loss), `gamelength`, `teamkills`, `teamdeaths`, `totalgold`, and time-stamped metrics like `goldat20`, `killsat20`, `deathsat20`, etc.

### Research Question

**Do certain professional League of Legends teams perform significantly differently in longer games compared to their overall performance, and what in-game factors drive success in extended matches?**

### Why This Matters

**Strategic Value:** Teams excel at different game paces—some dominate through early-game aggression (favoring short games), others through late-game scaling (favoring long games). Understanding these patterns helps teams optimize champion selection, pacing strategies, and opponent preparation.

**Predictive Insights:** If teams show statistically different performance in long games, we can build better prediction models that account for team-specific patterns rather than treating all teams identically. This reveals whether game length is merely a consequence of team strength or an independent factor affecting outcomes.

**Professional Meta Understanding:** Analyzing what drives victory in long games versus short games reveals whether the competitive meta rewards early aggression or late-game execution, and which statistics (gold, kills, deaths) become disproportionately important as matches extend.

### What We Discover

Through exploratory analysis, hypothesis testing, and predictive modeling on the **top 10 most active teams**, we find:

- **Most elite teams perform worse in long games** (T1: 74% overall → 61% long games; RNG: 66% → 60%), suggesting reliance on early-game advantages
- **DRX is a notable exception**, performing better in long games (64% vs 55% overall), indicating superior late-game execution
- **Team deaths are highly significant** in long games (p ≈ 0.000), with losing teams averaging 6.5 more deaths
- **Prediction models achieve ~75% accuracy** but struggle with low recall for losses (~30%), revealing difficulty in reliably predicting defeats
- **Model fairness across game lengths** shows no significant bias (p = 0.45), meaning predictions work equally well for short and long games

We define "long games" as those exceeding the 75th percentile of game length (~35+ minutes), representing extended matches where late-game team fighting becomes decisive.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning Process

Our cleaning process involves several key steps:

1. **Filtering team-level data**: We filter to include only rows where `position == 'team'`, focusing on team-level statistics rather than individual player performance.

2. **Column selection**: We select essential columns:
   - Basic game information: `datacompleteness`, `teamname`, `gamelength`, `result`, `url`, `league`, `game`
   - Overall statistics: `teamkills`, `teamdeaths`, `totalgold`
   - Time-based statistics at 10, 15, 20, and 25 minutes

3. **Data type conversions**:
   - Convert `result` to boolean (True for wins, False for losses)
   - Convert `gamelength` from seconds to minutes

4. **Feature engineering - Creating incremental statistics**:
   - We create columns representing the **increase** in statistics between time intervals (e.g., `increased_goldat15` = gold gained between 10-15 minutes)
   - This captures team performance **momentum** during different game phases

5. **Handling missing values**:
   - Fill missing `increased_*` columns with 0 (indicating games ended before that time point)

The resulting `cleaned_df` contains 25,098 team-level game records with 50 columns.

### Univariate Analysis

**Game Length Distribution:**
- 24.9% of games exceed Q3 (long games)
- 1.6% are statistical outliers (extremely long)
- Distribution appears roughly normal with slight right skew

**Total Gold Distribution:**
- Shows expected right skew (longer games = more gold)
- Helps understand economic patterns across matches

### Bivariate Analysis

**Win Rate vs. Game Length for Popular Teams:**

Key trends observed:
- **Most teams perform worse in long games**: T1 (74% → 61%), Royal Never Give Up (66% → 60%), Evil Geniuses (61% → 55%)
- **DRX is an exception**: 64% long game win rate vs 55% overall
- **Saigon Buffalo shows largest decline**: 53% overall → 33% long games
<iframe
  src="{{ site.baseurl }}/assets/Univariate_Analysis_fig_1.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>

<iframe
  src="{{ site.baseurl }}/assets/Univariate_Analysis_fig_2.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
**Gold at 20 Minutes vs. Game Result:**

- Clear separation between winners and losers in mean gold
- Overlapping distributions indicate comebacks are possible
- Winning teams show tighter clustering (more consistent)
- Validates gold at 20 minutes as a useful predictive feature

<iframe
  src="{{ site.baseurl }}/assets/Univariate_Analysis_fig_3.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>

<iframe
  src="{{ site.baseurl }}/assets/Univariate_Analysis_fig_4.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
### Interesting Aggregates

We calculate average statistics at 20 minutes grouped by game result, revealing:
- Winning teams have substantially higher gold and kills
- Death counts are lower for winning teams
- Average game length is similar for wins and losses

---

## Assessment of Missingness

### NMAR Analysis: URL Column

The **`url`** column (VOD links) has **84.52%** missing values. We believe this is likely **NMAR (Not Missing At Random)** because missingness likely depends on unobserved match characteristics like "entertainment value" or "match importance." Boring stomps or lower-tier matches may be less likely to have VOD links preserved.

**Additional data that could make it MAR:** Tournament tier information, broadcast platform data, viewer counts, match importance flags, or official recording policies by league.

### Permutation Testing

**Test 1: URL Missingness vs. League (DOES DEPEND)**

- Test Statistic: Total Variation Distance (TVD)
- Observed TVD: **0.9702**
- P-value: **0.0000**
- **Conclusion:** URL missingness **strongly depends on league** (p < 0.05). Different leagues have vastly different broadcasting infrastructure and archival policies.
<iframe
  src="{{ site.baseurl }}/assets/Missingness_fig_1.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>

**Test 2: URL Missingness vs. Game Result (DOES NOT DEPEND)**

- Test Statistic: Total Variation Distance (TVD)
- Observed TVD: **~0.0000**
- P-value: **~1.0000**
- **Conclusion:** URL missingness **does not depend on result** (p ≥ 0.05). Every match has exactly one winner and one loser, so result distribution is always 50/50 regardless of URL missingness.
<iframe
  src="{{ site.baseurl }}/assets/Missingness_fig_2.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
<iframe
  src="{{ site.baseurl }}/assets/Missingness_fig_3.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
### Summary

URL missingness is **NMAR with MAR properties conditional on league**. Our prediction models use in-game statistics rather than URLs, so this missingness doesn't directly affect our core analysis.

---

## Hypothesis Testing

### Hypothesis Test 1: Do Specific Teams Perform Better in Long Games?

**Research Question:** Given a specific team, is it statistically significant that they perform better (or worse) in longer games?

**Hypotheses:**
- **H₀:** Team's win rate in long games equals overall win rate (difference due to random chance)
- **Hₐ:** Team's win rate in long games differs from overall win rate

**Test Statistic:** Difference in win rate = (long game win rate - overall win rate)

**Methodology:** Permutation test with 5,000 shuffles of game results while keeping game lengths fixed.

**Results for JD Gaming:**
- Observed difference: **4.7%**
- P-value: **0.4194**
- Long game win rate: 70.0% | Overall: 65.3%

**Conclusion:** We **fail to reject H₀** (p = 0.42 > 0.05). The 4.7% difference is **not statistically significant** and could easily occur by random chance.
<iframe
  src="{{ site.baseurl }}/assets/hp_test_fig_1.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
---

### Hypothesis Test 2: Are Team Deaths a Key Factor in Long Games?

**Research Question:** Is the number of team deaths a statistically significant factor in determining long-game outcomes?

**Hypotheses:**
- **H₀:** Team deaths not associated with outcome in long games
- **Hₐ:** Losing teams have significantly more deaths in long games

**Test Statistic:** Difference in mean deaths = (losing teams average - winning teams average)

**Methodology:** Permutation test on long games (length ≥ Q3) with 5,000 shuffles.

**Results:**
- Observed difference: **6.458 deaths**
- P-value: **0.0000**
- Losing teams: 19.823 deaths | Winning teams: 13.365 deaths

**Conclusion:** We **reject H₀** (p ≈ 0.00). Team deaths are **highly statistically significant** in long games. Losing teams average 6.5 more deaths, validating deaths as a critical predictive feature.
<iframe
  src="{{ site.baseurl }}/assets/hp_test_fig_2.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
---

## Framing a Prediction Problem

**Goal:** Predict match outcomes (win/loss) using in-game statistics at various time points.

**Problem Type:** Binary classification (response variable: `result`)

**Features:** Incremental statistics representing gains during 5-minute intervals:
- Gold, CS, kills, deaths at 10, 15, 20, 25 minutes
- 16 quantitative features total (excluding XP due to correlation with gold/CS)

**Why incremental statistics?** They capture **momentum** and **performance trends** better than absolute values. A team gaining 5,000 gold from 15-20 minutes shows strong mid-game performance regardless of early-game standing.

**Information available at prediction time:** All features would be available during live games. We do NOT use information only available after games end (final gold, total game length).

**Evaluation Metrics:**
- **Accuracy:** Overall proportion of correct predictions
- **Precision:** Of predicted wins, what proportion were actual wins
- **Recall:** Of actual wins, what proportion were correctly predicted
- **F1-score:** Harmonic mean of precision and recall

Using all four metrics gives a complete picture, as a model can have decent accuracy while struggling with recall on one class.

---

## Baseline Model

### Model Overview

**Algorithm:** Decision Tree Classifier
**Features (2, both quantitative):**
- `increased_goldat20`: Gold gained 15-20 minutes
- `increased_goldat25`: Gold gained 20-25 minutes

**Hyperparameter Tuning:** GridSearchCV with 5-fold CV
- `max_depth`: [2, 3, 4, 5, 7, 8, 10]
- `min_samples_split`: [2, 5, 20, 30, 40, 50, 60, 70, 100]
- `criterion`: ['gini', 'entropy']

### T1 Performance

**Best Parameters:** criterion='gini', max_depth=2, min_samples_split=20

**Training:** CV score = 79.6%

**Test Results:**
- Accuracy: **77.8%**
- Precision (False): 1.00 | (True): 0.76
- Recall (False): **0.20** | (True): 1.00
- F1-score: Weighted avg = 0.72

### Assessment

**Strengths:** Decent accuracy (77.8%), simple and interpretable

**Critical Weaknesses:**
- Extremely low recall for losses (0.20) - only identifies 20% of actual losses
- Achieves accuracy by predicting "Win" for almost everything
- Limited 2-feature set ignores early-game and other statistics

**Overall:** Moderately adequate but not satisfactory. The model exploits class imbalance rather than learning balanced decision boundaries.
---

## Final Model

### Model Overview

**Algorithm:** Decision Tree Classifier in Pipeline with StandardScaler

**Features (16, all quantitative):**
- Gold, CS, kills, deaths gained at 10, 15, 20, 25 minutes (4 features each)
- Excludes XP (correlated with gold/CS)

**Feature Engineering:**
- **StandardScaler:** Transforms features to mean=0, std=1
- **Why it helps:** Equalizes feature scales (gold in thousands, kills 0-10), improves stability, enables fair comparison across time intervals, reduces outlier sensitivity

**Hyperparameter Tuning:** Same GridSearchCV approach as baseline

### T1 Performance

**Best Parameters:** criterion='gini', max_depth=2, min_samples_split=2

**Training:** CV score = **81.6%** (improved from 79.6%)

**Test Results:**
- Accuracy: **75.0%**
- Precision (False): 0.60 | (True): 0.77
- Recall (False): **0.30** | (True): 0.92
- F1-score: Weighted avg = 0.72

### Comparison to Baseline

**Improvements:**
- Higher CV score (81.6% vs 79.6%) - better generalization
- Improved recall for losses (0.30 vs 0.20) - detects 50% more losses
- More balanced predictions (doesn't rely solely on predicting wins)
- Richer 16-feature set captures patterns across game phases

**Persistent Issues:**
- Similar test accuracy (75% vs 77.8%)
- Still poor recall for losses (0.30 = misses 70% of losses)
- Class imbalance bias remains

**Why it's better:** Demonstrates better learning behavior with improved CV scores and more balanced predictions, even though test accuracy is similar. Provides better foundation for future improvements.

### Confusion Matrix
<iframe
  src="{{ site.baseurl }}/assets/final_matrix_fig_1.html"
  width="650"
  height="500"
  frameborder="0"
></iframe>
The final model's confusion matrix for T1 shows:
- True Negatives: 3 | False Positives: 7
- False Negatives: 2 | True Positives: 24

This clearly visualizes the model's struggle with detecting losses - only 3 of 10 actual losses were correctly identified.

### Limitations and Future Work

**Why limitations exist:**
- Class imbalance (T1 wins 74% of games)
- Decision trees favor majority class without explicit balancing
- Max depth of 2 may be too simple

**Future improvements:**
- Class weighting for minority class
- Ensemble methods (Random Forest, Gradient Boosting)
- More data collection, especially for losses
- Interaction features (gold differentials, K/D ratios)

---

## Fairness Analysis

### Research Question

**Does our model perform equally well for short games versus long games?**

### Group Definitions

- **Group X (Short):** gamelength < 31.58 minutes (average)
- **Group Y (Long):** gamelength ≥ 31.58 minutes

### Hypotheses

- **H₀:** Model is fair - accuracy is the same for short and long games
- **Hₐ:** Model is unfair - accuracy differs between game types

**Test Statistic:** |accuracy_short - accuracy_long|

### Methodology

Permutation test with 5,000 shuffles of game type labels while keeping predictions/results fixed.

### Results for T1

**Test Set:** 18 short games, 18 long games

**Performance:**
- Short game accuracy: **83%**
- Long game accuracy: **67%**
- Observed difference: **16.7 percentage points**

**Statistical Test:**
- P-value: **0.4468**
- Significance level: α = 0.05

### Conclusion

We **fail to reject H₀** (p = 0.45 > 0.05). Although the model appears to perform better on short games (83% vs 67%), this difference is **not statistically significant**. The observed difference could easily occur by random chance, especially with small test set size (18 games per group).

**The model is FAIR with respect to game length.** The accuracy difference is likely due to random variation rather than systematic bias.

**Implications:**
- Model can be deployed across different game lengths without systematic performance degradation
- Small sample size (18 per group) limits statistical power
- Model still has fairness issues with class (low recall for losses)

---

## Conclusion

This project analyzed 2022 professional League of Legends matches to understand how game length affects team performance and to predict match outcomes.

**Key Findings:**

1. **Team-specific patterns exist but are often not statistically significant** - While most elite teams perform worse in long games, hypothesis testing shows these differences are often within random chance range.

2. **Team deaths are critical in long games** - Highly significant (p ≈ 0.000), with losing teams averaging 6.5 more deaths.

3. **Prediction is challenging** - Models achieve ~75% accuracy but struggle with detecting losses (recall ≈ 30%), suggesting professional outcomes depend on factors beyond basic statistics.

4. **Models are fair across game lengths** - No significant bias between short and long games (p = 0.45).

5. **Data quality varies by league** - URL missingness strongly depends on league (TVD = 0.97), requiring careful consideration in cross-league analyses.

**Implications:**

For teams and coaches, game length patterns should be interpreted cautiously - observed differences may not reflect true strategic advantages. The significance of team deaths in long games validates minimizing mistakes in extended matches.

For data scientists, this demonstrates both the potential and limitations of traditional ML approaches. Low recall for losses suggests capturing professional match outcomes may require more sophisticated feature engineering or ensemble methods.

---

## Repository Files

- `template.ipynb` - Main analysis notebook
- `2022_LoL_esports_match_data_from_OraclesElixir.csv` - Dataset
- `PROJECT_REPORT.md` - This report

---

## Author

**Albert Zhang**
Website: [League of Legends Game Analysis](https://seagull-angle.github.io/League_of_Legends_Game_Analysis/)

Project completed as part of DSC 80 at UC San Diego
Data sourced from Oracle's Elixir
