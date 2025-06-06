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
The combined histogram shows that **Mid laners generally outperform ADCs** in early-game kill counts.

---

### 🔹 Bivariate Analysis

Next, we explored the **relationship between kills and gold at 25 minutes**, broken down by position.

#### Kill vs Gold by Position  
<iframe src="assets/killsat25-goldat25-by-position.html" width="800" height="500" frameborder="0"></iframe>  
We observe a **strong positive correlation** between `killsat25` and `goldat25`. Players with higher kills tend to accumulate more gold. The clustering suggests that Mid and Jungle positions are more likely to generate high gold with high kills.

#### Kill Distribution by Role  
<iframe src="assets/killsat25-by-role.html" width="800" height="500" frameborder="0"></iframe>  
This visualization highlights differences across roles, where Mid and Jungle tend to lead in early-game action, while Supports rarely exceed 2 kills.

---

### 🔹 Interesting Aggregates

We also created grouped tables to summarize average performance by role:

| Role | Avg Kills at 25 | Avg Gold at 25 |
|------|------------------|----------------|
| Mid  | 3.2              | 8350           |
| ADC  | 2.4              | 7900           |
| Jungle | 3.0            | 8200           |
| Top  | 2.1              | 7650           |
| Support | 0.9           | 6100           |

These aggregates reinforce the idea that **Mid laners** consistently outperform ADCs in both kills and gold at the 25-minute mark — motivating the hypothesis test in the next section.

---


## Assessment of Missingness

We tested whether the missingness of `firstbloodvictim` is related to other variables.

- ✅ **firstblood:** Strong dependency (p-value = 0.0)  
- ✅ **position:** Also dependent (p-value = 0.0)  
- ✅ **result:** Appears independent (p-value = 0.476)

This suggests `firstbloodvictim` is **not missing completely at random (MCAR)**.

---

## Hypothesis Testing

**Question:** Do Mid laners have more kills at 25 minutes than ADCs?

- **Null Hypothesis (H₀):** No difference in mean kills  
- **Alternative Hypothesis (H₁):** Mid laners have more kills  
- **Test Statistic:** Mean difference  
- **Observed Difference:** −0.37  
- **p-value:** 1.0  

✅ **Conclusion:** We fail to reject the null — ADCs actually had slightly more kills on average.

---

## Framing a Prediction Problem

**Question:** Can we predict whether a team will win a match using only kills and gold at 25 minutes?

- **Type:** Binary classification  
- **Target column:** `result` (1 = win, 0 = loss)

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
