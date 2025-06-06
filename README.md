# Gold, Kills, and Victory: Predicting Match Outcomes in League of Legends

**Name:** Yao Ouyang

---

## Introduction

In professional League of Legends (LoL) esports, early-game performance is often seen as a key predictor of a team’s eventual victory.  
Metrics such as gold earned and kills secured in the first 25 minutes may offer meaningful insight into game momentum and team coordination.  
This project investigates whether these mid-game indicators — specifically, kills and gold at 25 minutes — can be used to predict match outcomes.

We also explore role-based performance dynamics, comparing **Mid laners** and **ADCs** (Bot lane carries) to test common beliefs about who “carries” more often.  
Using data from Oracle’s Elixir, which includes statistics from hundreds of professional matches, we analyze trends across positions, assess missing data patterns, and build predictive models to classify wins and losses.  
Finally, we evaluate whether the model performs fairly across different regions — a key consideration in ensuring equitable performance in esports analytics.

The dataset contains over **70,000 rows**, each representing an individual player or team’s performance in a single professional match.  
The columns most relevant to our analysis include:

- `killsat25`: Number of champion kills by a player or team at the 25-minute mark  
- `goldat25`: Total gold earned by that point in the match  
- `position`: The player's in-game role (e.g., Mid, Bot, Jungle, Top, Support, or team)  
- `role`: A simplified classification used to distinguish Mid laners from ADCs  
- `result`: Binary outcome indicating if the player’s team won (`1`) or lost (`0`) the match

These features allow us to explore two central questions:

1. **Can we predict whether a team will win a match based on their kills and gold at 25 minutes?**  
2. **Do Mid laners contribute more early-game action than ADCs, as commonly believed?**

---

## Data Cleaning and Exploratory Data Analysis

We began by cleaning the dataset to ensure our analysis was based on reliable and complete records.  
We removed columns with more than **50% missing values**, and filtered the dataset to include only matches that lasted **at least 1500 seconds (25 minutes)** — since we were investigating performance specifically at the 25-minute mark.

For missing values:
- **Categorical variables** (e.g., `position`) with missing entries were filled with `"Unknown"`
- **Numeric variables** (e.g., `killsat25`, `goldat25`) were filled using the **median** of each column

We also **excluded rows** where the `position` column was `"team"` to focus only on individual player roles. To simplify analysis, we created a new `role` column that consolidated positions: `"Mid"` for mid laners, `"ADC"` for bot laners, and excluded other roles from specific role-based questions.

Here is the head of our cleaned DataFrame:

| playername | position | role | killsat25 | goldat25 | result |
|------------|----------|------|-----------|----------|--------|
| Sayn       | mid      | Mid  | 1.0       | 8488.0   | 0      |
| Shiganari  | bot      | ADC  | 1.0       | 9097.0   | 0      |
| Nafkelah   | mid      | Mid  | 1.0       | 10040.0  | 1      |
| Soldier    | bot      | ADC  | 6.0       | 11710.0  | 1      |
| xKenzuke   | mid      | Mid  | 4.0       | 10940.0  | 1      |



---

### 🔹 Univariate Analysis

We first examined the **distribution of kills at 25 minutes** for Mid laners and ADCs to understand differences in early-game aggression.

#### Mid laners only  
<iframe src="assets/killsat25-mid.html" width="800" height="500" frameborder="0"></iframe>  
Mid laners tend to have a right-skewed distribution, with most players getting 1–4 kills at 25 minutes. A small portion reaches 5+ kills, showing occasional early-game carry potential.

#### ADCs only  
<iframe src="assets/killsat25-adc.html" width="800" height="500" frameborder="0"></iframe>  
ADCs show a similar but slightly lower kill distribution. Most have 0–3 kills by 25 minutes, reflecting a slower ramp-up or support-dependent lane.

#### Combined Mid vs ADC  
<iframe src="assets/killsat25-combine.html" width="800" height="500" frameborder="0"></iframe>  
This histogram shows that at 25 minutes, the mid laner had more low kills, while the ADC had more high kills. This may indicate that the mid laner is more stable, while ADCs have a higher potential.

---

### 🔹 Bivariate Analysis

Next, we explored the **relationship between kills and gold at 25 minutes**, broken down by position.

#### Kill vs Gold by Position  
<iframe src="assets/killsat25-goldat25-by-position.html" width="800" height="500" frameborder="0"></iframe>  
Gold at 25 minutes is strongly influenced by kills but also varies by position. The scatter plot supports the idea that not all roles convert kills into gold at the same efficiency.

#### Kill Distribution by Role  
<iframe src="assets/killsat25-by-role.html" width="800" height="500" frameborder="0"></iframe>  
Despite the perception that Mid laners are the primary “carry” role, Bot lane players (ADCs) may secure more kills on average, though their performance is more variable. This could challenge traditional assumptions in LoL strategy.

---

### 🔹 Interesting Aggregates

We also created grouped tables to summarize average performance by role:

| position | Avg Kills at 25 | Avg Gold at 25 |
|------|------------------|----------------   |
| bot  | 2.8              | 10055.7           |
| jng  | 2.3              | 8883.5            |
| mid  | 2.5              | 9739.9            |
| sup  | 0.7              | 6020.9            |
| top  | 1.9              | 9207.9            |

These aggregates show Bot lane (ADCs) have the highest average kills and gold at 25 minutes, suggesting that this position receives the most resources and has a strong early-game impact. Mid laners follow closely, with a solid average of kills and gold, showing that they are also central to early skirmishes and team success. This motivates the hypothesis test in the later section.

---


## Assessment of Missingness

We hypothesized that the missingness of firstbloodvictim depends on whether the game recorded the first blood event. The presence of such an event is indicated by the first blood column (1 if a player secured first blood, 0 otherwise). If a match did not record the first blood, then logically no player can be recorded as the victim — making it plausible that this missingness is Missing At Random (MAR) rather than Missing Not At Random (NMAR). 

Based on the context of League of Legends (LoL), we suspect the missingness in `firstbloodvictim` is **MAR**, as the variable is likely to be missing in cases where first blood does not occur or is not recorded, which may relate to `firstblood`. If more game events are documented when first blood is secured, missingness would be related to observed game outcomes — making it MAR.

We conducted a **permutation test** to evaluate the dependency between the missingness of `firstbloodvictim` and the `firstblood` column. Our null hypothesis was:

> **Null Hypothesis:** The missingness of `firstbloodvictim` is independent of the `firstblood` value.

> **Alternative Hypothesis:** The missingness of `firstbloodvictim` depends on the `firstblood` value.

### Permutation Test Result

We calculated an observed difference in missingness rates and compared it to the distribution of permuted differences generated by randomly shuffling the `firstblood` labels.

<iframe src="assets/permutation_missingness_test.html" width="800" height="500" frameborder="0"></iframe>

- **Observed Difference:** 0.186  
- **P-value:** 0.0  

This result provides strong evidence that the missingness in `firstbloodvictim` is not independent of the `firstblood` column, supporting the idea that it is **MAR**.

### Conditional Distribution of Firstblood

To further illustrate this dependency, we visualized the distribution of `firstblood` when `firstbloodvictim` is missing versus not missing:

<iframe src="assets/Distribution_Missingness.html" width="800" height="500" frameborder="0"></iframe>

This histogram shows clear differences in `firstblood` outcomes between rows where `firstbloodvictim` is missing and not missing, reinforcing the results of our permutation test.

---


## Hypothesis Testing

To investigate role-based performance, we tested whether **ADCs** (Attack Damage Carries) have a higher number of kills at the 25-minute mark than **Mid laners**.

### Hypotheses

- **Null Hypothesis (H₀):**  
There is no difference in the mean number of kills at 25 minutes between Mid laners and ADCs.

- **Alternative Hypothesis (H₁):**  
ADCs have a higher mean number of kills at 25 minutes than Mid laners.

### Test Details

- **Test Statistic:** Mean difference in kills between Mid and ADC roles  
- **Significance Level (α):** 0.05  
- **Observed Mean Difference:** ~-0.3695
- **P-value (from permutation test):** 0.0  


### Interpretation

We chose a permutation test with the mean difference in kills at 25 minutes as the test statistic, because our goal was to compare the average performance of Mid laners and ADCs. This choice offers a statistically valid and interpretable way to assess whether role impacts in-game performance and are appropriate given the exploratory nature of our dataset and question.

The negative mean difference indicates that ADCs have, on average, more kills at 25 minutes than Mid laners. A p-value of 0.0 suggests this result is statistically significant at the 0.05 level. Thus, we reject the null hypothesis in favor of the alternative: **ADCs statistically outperform Mid laners** in kill count at the 25-minute mark, contrary to initial expectations.

---

## Framing a Prediction Problem

Our prediction task is to **predict whether a team will win a match** based on their in-game performance at the 25-minute mark. This is a **binary classification** problem because the outcome variable is either a win (1) or a loss (0).

### Response Variable:
- `result`: 1 if the team won the match, 0 if they lost.

### Features:
- `killsat25`: Number of kills the team secured by 25 minutes.
- `goldat25`: Total gold the team accumulated by 25 minutes.

These features were chosen because they reflect team performance during the early and mid-game phases, which are available at the **time of prediction** (25-minute mark). This ensures we are not using information that would only be known after the match is over, preventing data leakage.

### Type of Prediction:
- **Binary Classification**

### Evaluation Metric:
- **Accuracy**: We use accuracy to evaluate model performance because the dataset has a relatively balanced distribution of wins and losses, and accuracy provides a straightforward interpretation of predictive success.

We only included features that are known during the game and excluded any post-match statistics to ensure the model mimics a real-time prediction setting.

---

## Baseline Model

We built a logistic regression using `killsat25` and `goldat25`, scaled using `StandardScaler`, and trained with `train_test_split(random_state=42)`.

**Test Accuracy:** ~0.61  
**Precision (win):** 0.63  
**Recall (win):** 0.49  

The model shows decent ability to use early-game metrics to predict victory.

---

## Final Model

We added 3 engineered features:
- `kpg` = kills per minute  
- `gold_per_kill` = gold efficiency  
- `fb_indicator` = early-game momentum

We used `GridSearchCV` to tune `C` and `penalty` for logistic regression.

**Best parameters:**  
- C = 0.1  
- penalty = 'l2'

**Test Accuracy:** 0.605 (similar to baseline)  
Though accuracy didn’t improve, the model now accounts for game pace and early momentum.

---

## Fairness Analysis

**Question:** Does the model perform worse for minor-region teams?

- **Group X:** Major regions (LCK, LPL, LCS, LEC)  
- **Group Y:** Minor regions (all others)  
- **Metric:** Precision  
- **Observed Difference:** −0.0347 (minor < major)  
- **p-value:** 0.062

<iframe src="assets/fairness-precision.html" width="800" height="600" frameborder="0"></iframe>

✅ **Conclusion:** The model performs slightly worse for minor-region teams, but not significantly so at p < 0.05.

---

### Thanks for reading! 🧠🏆  
This project highlights how simple metrics like gold and kills can explain — but not fully determine — match outcomes in esports. Even in pro games, the rift remains unpredictable.
