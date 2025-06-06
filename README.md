# Gold, Kills, and Victory: Predicting Match Outcomes in League of Legends
This is a project for DSC 80 at UCSD
**Name:** Yao Ouyang

This project explores how early-game statistics in professional League of Legends (LoL) esports — especially kills and gold at 25 minutes — can help predict a team’s match outcome. We also examine fairness across competitive regions by testing whether the model performs equally well for major vs minor league teams.

---

## Introduction

Does early-game performance dictate final victory in League of Legends?  
Using data from Oracle’s Elixir, we analyze the predictive power of gold and kill stats at 25 minutes, and assess whether our model treats teams from different regions fairly.

---


## Data Cleaning and Exploratory Data Analysis

We cleaned the dataset by removing columns with over 50% missing values and focusing on rows where `gamelength ≥ 1500s` (25 minutes).  
Missing values in categorical columns were replaced with `'Unknown'`, and numeric ones filled with median values.

We focused our early analysis on the performance of Mid laners and ADCs at 25 minutes.

---

### 🔹 Mid laners only
<iframe src="assets/killsat25-mid.html" width="800" height="500" frameborder="0"></iframe>

### 🔹 ADCs only
<iframe src="assets/killsat25-adc.html" width="800" height="500" frameborder="0"></iframe>

### 🔹 Combined Mid vs ADC distribution
<iframe src="assets/killsat25-combine.html" width="800" height="500" frameborder="0"></iframe>

### 🔹 Kill distribution by roles (Mid vs ADC)
<iframe src="assets/killsat25-by-role.html" width="800" height="500" frameborder="0"></iframe>

### 🔹 Kills vs Gold at 25 Minutes by positions
<iframe src="assets/killsat25-goldat25-by-position.html" width="800" height="500" frameborder="0"></iframe>


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
