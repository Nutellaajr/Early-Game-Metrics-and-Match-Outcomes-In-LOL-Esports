# Gold, Kills, and Victory: Predicting Match Outcomes in League of Legends

**Name:** Yao Ouyang

---

## Introduction

In professional League of Legends (LoL) esports, early-game performance is often seen as a key predictor of a team’s eventual victory.  
Metrics such as gold earned and kills secured in the first 25 minutes may offer meaningful insight into game momentum and team coordination.  
This project investigates whether these mid-game indicators — specifically, kills and gold at 25 minutes — can be used to predict match outcomes.

I also explore role-based performance dynamics, comparing **Mid laners** and **ADCs** (Bot lane carries) to test common beliefs about who “carries” more often.  
Using data from Oracle’s Elixir, which includes statistics from hundreds of professional matches, I analyze trends across positions, assess missing data patterns, and build predictive models to classify wins and losses.  
Finally, I evaluate whether the model performs fairly across different regions — a key consideration in ensuring equitable performance in esports analytics.

The dataset contains over **70,000 rows**, each representing an individual player or team’s performance in a single professional match.  
The columns most relevant to our analysis include:

- `killsat25`: Number of champion kills by a player or team at the 25-minute mark  
- `goldat25`: Total gold earned by that point in the match  
- `position`: The player's in-game role (e.g., Mid, Bot, Jungle, Top, Support, or team)  
- `role`: A simplified classification used to distinguish Mid laners from ADCs  
- `result`: Binary outcome indicating if the player’s team won (`1`) or lost (`0`) the match

These features allow us to explore two central questions:

1. **Can I predict whether a team will win a match based on their kills and gold at 25 minutes?**  
2. **Do Mid laners contribute more early-game action than ADCs, as commonly believed?**

---

## Data Cleaning and Exploratory Data Analysis

I began by cleaning the dataset to ensure our analysis was based on reliable and complete records.  
I removed columns with more than **50% missing values**, and filtered the dataset to include only matches that lasted **at least 1500 seconds (25 minutes)** — since I was investigating performance specifically at the 25-minute mark.

For missing values:
- **Categorical variables** (e.g., `position`) with missing entries were filled with `"Unknown"`
- **Numeric variables** (e.g., `killsat25`, `goldat25`) were filled using the **median** of each column

I also **excluded rows** where the `position` column was `"team"` to focus only on individual player roles. To simplify analysis, I created a new `role` column that consolidated positions: `"Mid"` for mid laners, `"ADC"` for bot laners, and excluded other roles from specific role-based questions.

Here is the head of our cleaned DataFrame:

| playername | position | role | killsat25 | goldat25 | result |
|------------|----------|------|-----------|----------|--------|
| Sayn       | mid      | Mid  | 1.0       | 8488.0   | 0      |
| Shiganari  | bot      | ADC  | 1.0       | 9097.0   | 0      |
| Nafkelah   | mid      | Mid  | 1.0       | 10040.0  | 1      |
| Soldier    | bot      | ADC  | 6.0       | 11710.0  | 1      |
| xKenzuke   | mid      | Mid  | 4.0       | 10940.0  | 1      |



---

### Univariate Analysis

I first examined the **distribution of kills at 25 minutes** for Mid laners and ADCs to understand differences in early-game aggression.

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

###  Bivariate Analysis

Next, I explored the **relationship between kills and gold at 25 minutes**, broken down by position.

#### Kill vs Gold by Position  
<iframe src="assets/killsat25-goldat25-by-position.html" width="800" height="500" frameborder="0"></iframe>  
Gold at 25 minutes is strongly influenced by kills but also varies by position. The scatter plot supports the idea that not all roles convert kills into gold at the same efficiency.

#### Kill Distribution by Role  
<iframe src="assets/killsat25-by-role.html" width="800" height="500" frameborder="0"></iframe>  
Despite the perception that Mid laners are the primary “carry” role, Bot lane players (ADCs) may secure more kills on average, though their performance is more variable. This could challenge traditional assumptions in LoL strategy.

---

###  Interesting Aggregates

I also created grouped tables to summarize average performance by role:

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

I hypothesized that the missingness of firstbloodvictim depends on whether the game recorded the first blood event. The presence of such an event is indicated by the first blood column (1 if a player secured first blood, 0 otherwise). If a match did not record the first blood, then logically no player can be recorded as the victim — making it plausible that this missingness is Missing At Random (MAR) rather than Missing Not At Random (NMAR). 

Based on the context of League of Legends (LoL), I suspect the missingness in `firstbloodvictim` is **MAR**, as the variable is likely to be missing in cases where first blood does not occur or is not recorded, which may relate to `firstblood`. If more game events are documented when first blood is secured, missingness would be related to observed game outcomes — making it MAR.

I conducted a **permutation test** to evaluate the dependency between the missingness of `firstbloodvictim` and the `firstblood` column. Our null hypothesis was:

> **Null Hypothesis:** The missingness of `firstbloodvictim` is independent of the `firstblood` value.

> **Alternative Hypothesis:** The missingness of `firstbloodvictim` depends on the `firstblood` value.

### Permutation Test Result

I calculated an observed difference in missingness rates and compared it to the distribution of permuted differences generated by randomly shuffling the `firstblood` labels.

<iframe src="assets/permutation_missingness_test.html" width="800" height="500" frameborder="0"></iframe>

- **Observed Difference:** 0.186  
- **P-value:** 0.0  

This result provides strong evidence that the missingness in `firstbloodvictim` is not independent of the `firstblood` column, supporting the idea that it is **MAR**.

### Conditional Distribution of Firstblood

To further illustrate this dependency, I visualized the distribution of `firstblood` when `firstbloodvictim` is missing versus not missing:

<iframe src="assets/Distribution_Missingness.html" width="800" height="500" frameborder="0"></iframe>

This histogram shows clear differences in `firstblood` outcomes between rows where `firstbloodvictim` is missing and not missing, reinforcing the results of our permutation test.

---


## Hypothesis Testing

To investigate role-based performance, I tested whether **ADCs** (Attack Damage Carries) have a higher number of kills at the 25-minute mark than **Mid laners**.

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

I chose a permutation test with the mean difference in kills at 25 minutes as the test statistic, because our goal was to compare the average performance of Mid laners and ADCs. This choice offers a statistically valid and interpretable way to assess whether role impacts in-game performance and are appropriate given the exploratory nature of our dataset and question.

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
- **Accuracy**: I use accuracy to evaluate model performance because the dataset has a relatively balanced distribution of wins and losses, and accuracy provides a straightforward interpretation of predictive success.

I only included features that are known during the game and excluded any post-match statistics to ensure the model mimics a real-time prediction setting.

---

## Baseline Model


I developed a baseline classification model to predict whether a team would win a match based on their **kills** and **gold** at the 25-minute mark. Both features, `killsat25` and `goldat25`, are **quantitative** numerical variables and required no encoding. The target variable, `result`, is a binary categorical variable where `1` indicates a win and `0` indicates a loss.

I used a **Logistic Regression** classifier implemented in a `scikit-learn` pipeline. The pipeline includes a `StandardScaler` to normalize the input features and a `LogisticRegression()` model to perform binary classification. The dataset was split into training and testing sets using an 80-20 ratio.

I evaluated the performance of our baseline model using **accuracy** and the **classification report** (which includes precision, recall, and F1-score). The model achieved an accuracy of approximately **0.605** on the test set. Below is the breakdown of performance:

- **Class 0 (loss)**:
  - Precision: 0.59
  - Recall: 0.71
  - F1-score: 0.65
    
- **Class 1 (win)**:
  - Precision: 0.63
  - Recall: 0.49
  - F1-score: 0.55

- **Macro Average**:
  - Precision: 0.61
  - Recall: 0.60
  - F1-score: 0.60

- **Weighted Average**:
  - Precision: 0.61
  - Recall: 0.61
  - F1-score: 0.60

This baseline model offers moderate predictive power, especially favoring the correct identification of losses over wins. The imbalance in recall suggests potential room for improvement, particularly in detecting wins. In future steps, I will experiment with additional features and model tuning to improve fairness and accuracy across both classes.

---


## Final Model

To improve upon our Baseline Model, I added two new features: `firstblood` and `firstdragon`. These features are categorical indicators of whether the team secured the first kill or the first dragon in a match, respectively. From a game dynamics perspective, these are strong early-game objectives that often reflect early momentum and map control — factors that can significantly influence a team's chance of winning. Including them captures more of the team’s performance in the first 25 minutes and complements our initial features, `killsat25` and `goldat25`.

Since our target variable `result` is binary (win/loss), I used a classification approach. For our final model, I chose a `RandomForestClassifier`, which is well-suited to handle both numerical and categorical inputs, and captures nonlinear interactions between features without requiring extensive preprocessing. I used a pipeline to preprocess the data: numerical features were standardized using `StandardScaler`, and categorical features were one-hot encoded with `OneHotEncoder`.

I did not tune hyperparameters in this iteration, but I used `random_state=42` to ensure reproducibility. This model was trained using an 80/20 train-test split.

### Model Performance

Our Final Model achieved a test accuracy of **0.707**, improving over the Baseline Model's accuracy of **0.605**. It also achieved a **macro F1-score of 0.71**, showing better balance in performance across both classes. In contrast, the baseline logistic regression model struggled with class imbalance and had an F1-score of **0.60**. Below is the breakdown of performance:

- **Class 0 (loss)**:
  - Precision: 0.74
  - Recall: 0.67
  - F1-score: 0.70
    
- **Class 1 (win)**:
  - Precision: 0.68
  - Recall: 0.75
  - F1-score: 0.71

- **Macro Average**:
  - Precision: 0.71
  - Recall: 0.71
  - F1-score: 0.71

- **Weighted Average**:
  - Precision: 0.71
  - Recall: 0.71
  - F1-score: 0.71

The improvement suggests that `firstblood` and `firstdragon` provide valuable early-game signals not captured by kills and gold alone. This confirms our hypothesis that capturing early strategic objectives is important for predicting match outcomes in professional League of Legends esports games.

I consider this a meaningful performance gain and evidence that the Final Model captures the game’s dynamics more comprehensively than the baseline.


---

## Fairness Analysis

To assess whether our final model performs equitably across different groups, I conducted a **fairness permutation test**. Specifically, I investigated whether the model’s precision differs between players on the **Blue** side and those on the **Red** side in League of Legends matches.

### Group Definition

- **Group X (Blue side):** Players whose `side` value is `"Blue"`.
- **Group Y (Red side):** Players whose `side` value is `"Red"`.

I used the `side` column from Oracle’s dataset to define these groups, aligning each row in the test set with its original metadata by index.

### Evaluation Metric and Hypotheses

I used **precision** as our evaluation metric because our task is a binary classification (win vs. loss), and I wanted to understand how often our model's predicted wins were correct for each side.

- **Null Hypothesis (H₀):** The model is fair with respect to team side. That is, precision is the same for Blue and Red sides.  
  H₀: Δ = 0  
- **Alternative Hypothesis (H₁):** The model is less precise for the Red side than the Blue side.  
  H₁: Δ < 0 (where Δ = precision_Red − precision_Blue)

I used a **one-sided permutation test** with 10,000 permutations and set the significance level α = 0.05.

### Observed Metrics

- **Precision (Blue side):** 0.713  
- **Precision (Red side):** 0.640  
- **Observed Δ (Red − Blue):** -0.074  
- **p-value (one-sided):** 0.0076

### Conclusion

Since the p-value (0.0076) is below our significance level (α = 0.05), I reject the null hypothesis. This suggests that our final model is **unfair** with respect to team side: it is **less precise** when predicting wins for players on the Red side than for those on the Blue side.

### Visualization

I include a permutation test histogram below to illustrate the distribution of Δ under the null hypothesis, with the observed statistic marked for reference:

<iframe src="assets/Precision_Difference.html" width="800" height="500" frameborder="0"></iframe>


---

## Thanks for reading! 🧠🏆  
This project highlights how simple metrics like gold and kills can explain — but not fully determine — match outcomes in esports. Even in pro games, the rift remains unpredictable.
