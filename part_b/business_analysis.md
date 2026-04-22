# Part B: Business Case Analysis
## Scenario: Promotion Effectiveness at a Fashion Retail Chain

---

## B1. Problem Formulation — 

### B1(a) — ML Problem Formulation 

**Target Variable:**
The target variable is the **promotion type** to be deployed in a given store in a given month. It is a categorical variable with five possible values:
- Flat Discount
- BOGO (Buy-One-Get-One)
- Free Gift with Purchase
- Category-Specific Offer
- Loyalty Points Bonus

**Candidate Input Features (Predictor Variables):**

| Feature | Description |
|---|---|
| Store Location Type | Urban / Semi-urban / Rural |
| Store Size | Small / Medium / Large (in sq. ft.) |
| Monthly Footfall | Number of customers visiting the store per month |
| Local Competition Density | Number of competing stores nearby |
| Customer Demographics | Age group, gender mix, income level of local customers |
| Month / Season | January–December; festive season flag |
| Previous Month's Sales Volume | Items sold in the prior month |
| Historical Promotion Performance | Past performance of each promotion at that store |
| Weekend Count in Month | Number of weekends (more weekends = higher footfall) |
| Festival/Holiday Flag | Whether the month has major festivals or public holidays |

**Type of ML Problem:**
This is a **Multi-Class Classification** problem. The model takes store-level and time-based features as input and predicts which one of the five promotions will maximise items sold. It is not regression (we are not predicting a number) and not binary classification (there are five possible output classes, not two).

**Justification:** Since we want to assign one label (promotion type) from a fixed set of five options to each store-month combination, multi-class classification is the most appropriate ML problem type.

---

### B1(b) — Why Items Sold is a Better Target Variable than Revenue 

**The Argument for Using Items Sold (Sales Volume):**

Total sales revenue is influenced by factors completely outside the scope of promotions — primarily the **price of items**. For example:
- A store that sells 5 expensive designer jackets at ₹10,000 each earns ₹50,000 in revenue.
- A store that sells 200 affordable t-shirts at ₹200 each also earns ₹40,000 in revenue.

If the model is trained on revenue, it may recommend a promotion that simply pushes high-ticket items rather than one that genuinely drives a higher **volume** of purchases. This distorts the model's learning because:
1. Revenue is confounded by product mix and pricing strategy, which are not controlled by the promotion.
2. Items sold directly measures how many individual purchase decisions a promotion triggered, which is a purer signal of promotion effectiveness.
3. A promotion's true job is to motivate customers to buy — the number of items sold captures exactly that.

**Broader Principle — Target Variable Selection:**

This illustrates the principle of **choosing a target variable that directly reflects the business objective and is minimally confounded by external factors.** In real-world ML projects, a poorly chosen target variable can cause a technically accurate model to produce misleading or wrong business recommendations. The right target variable must be:
- Directly tied to the action being optimised (here: promotion deployment).
- Free from noise introduced by unrelated variables (here: price, product category mix).
- Measurable and consistent across all units of analysis (here: across all 50 stores).

---

### B1(c) — Alternative to One Global Model 

**The Junior Analyst's Suggestion — One Global Model:**
A single model trained on all 50 stores together assumes that every store responds to promotions in the same way. This is incorrect because an urban flagship store and a rural semi-urban outlet have completely different customer bases, footfall patterns, competition levels, and purchasing behaviour.

**Proposed Alternative — Segmented or Localised Modelling Strategy:**

The better approach is to build **separate models per store segment** rather than one global model. There are two practical ways to do this:

**Option 1: Cluster-Based Modelling (Recommended)**
- First, cluster the 50 stores into groups based on characteristics like location type, size, footfall, and demographics using a clustering algorithm (e.g., K-Means).
- Then, train one model per cluster. Stores within the same cluster share similar behaviour, so the model learns patterns relevant to that group.
- For example: Cluster 1 = Large Urban stores, Cluster 2 = Mid-size Semi-urban stores, Cluster 3 = Small Rural stores.

**Option 2: Store-Level Models with Shared Structure (Mixed-Effects / Hierarchical Models)**
- Build a model that has global parameters shared across all stores (to handle stores with little data) but also store-specific parameters (to capture individual store quirks).
- This approach works well when some stores have limited historical data and cannot support a fully independent model.

**Justification:**
Different locations respond very differently to the same promotion. For instance, "BOGO" may perform well in urban stores where customers make impulse bulk purchases, while "Loyalty Points Bonus" may work better in semi-urban stores with regular repeat customers. A single global model would average out these differences and produce suboptimal recommendations for all store types.

---

## B2. Data and EDA Strategy — 

### B2(a) — Table Joining, Grain of Dataset, and Aggregations 

**The Four Tables:**
1. **Transactions Table** — Each row is one transaction (customer, store, date, items bought, promotion used).
2. **Store Attributes Table** — Each row is one store (location type, size, footfall, demographics).
3. **Promotion Details Table** — Each row is one promotion (type, discount %, mechanics).
4. **Calendar Table** — Each row is one date (with weekend flag and festival flag).

**How to Join These Tables:**

The joining strategy is as follows:

| Join | Key | Type |
|---|---|---|
| Transactions ↔ Store Attributes | `store_id` | LEFT JOIN — to attach store characteristics to every transaction |
| Transactions ↔ Promotion Details | `promotion_id` | LEFT JOIN — to attach promotion mechanics to each transaction |
| Transactions ↔ Calendar | `date` | LEFT JOIN — to attach weekend/festival flag to each transaction |

After joining, we aggregate up to the **store-month level** (see grain below).

**Grain of the Final Modelling Dataset:**

> **One row = One Store × One Month**

This means each row represents a unique combination of a store and a calendar month. For example: Store 7 in March 2024 is one row; Store 7 in April 2024 is another row.

There will be 50 stores × 36 months (3 years) = **1,800 rows** in the final dataset.

**Aggregations to Perform Before Modelling:**

| Aggregation | What to Compute |
|---|---|
| Total Items Sold | SUM of items sold per store per month (this is our target variable) |
| Total Revenue | SUM of revenue per store per month |
| Promotion Used | MODE or assigned promotion for that store-month |
| Average Transaction Value | MEAN transaction value per store per month |
| Number of Transactions | COUNT of transactions per store per month |
| Weekend Days Count | COUNT of weekend days in the month (from calendar) |
| Festival Flag | MAX of festival flag in that month (1 if any festival day exists) |
| Footfall | From store attributes — may be static or monthly |

---

### B2(b) — EDA Before Building the Model 

The following four analyses should be performed before modelling:

**Analysis 1: Promotion Performance by Location Type**
- **What to do:** Create a bar chart showing average items sold for each of the 5 promotion types, broken down by store location (Urban / Semi-urban / Rural).
- **What to look for:** Whether a promotion that works in urban stores also works in rural stores or not.
- **Influence on Modelling:** If significant differences are found, it confirms that location type is a critical feature and supports the need for segmented models (as discussed in B1c). It may also justify creating interaction features like `promotion_type × location_type`.

**Analysis 2: Monthly/Seasonal Trends in Sales Volume**
- **What to do:** Plot average items sold across all stores by month to detect seasonal patterns. Also overlay festival flags.
- **What to look for:** Are there peak months (e.g., October–December for festive season)? Do certain promotions outperform in festive months?
- **Influence on Modelling:** Month and festival flag must be included as features. If seasonality is strong, we may need to engineer features like `is_festive_month` or `quarter_of_year`.

**Analysis 3: Correlation Between Store Features and Promotion Effectiveness**
- **What to do:** Compute correlation between store-level features (size, footfall, competition density) and items sold per promotion type. Use a heatmap.
- **What to look for:** Which store features are most strongly associated with high sales under each promotion type?
- **Influence on Feature Engineering:** High-correlation features should be prioritised in the model. Low-correlation features may be dropped or transformed. For example, if footfall is highly correlated with BOGO performance, `footfall` becomes a strong feature.

**Analysis 4: Distribution of Items Sold (Target Variable Check)**
- **What to do:** Plot the distribution (histogram) of the target variable — items sold per store-month — across all 5 promotions.
- **What to look for:** Is the distribution skewed? Are there outliers (e.g., one store selling 10x more than others during a festival month)?
- **Influence on Modelling:** Outliers may need to be capped or removed. A heavily skewed distribution may require log transformation for some algorithms. This analysis also tells us if certain promotions consistently result in higher or lower sales, which is a direct signal for our classification target.

---

### B2(c) — Handling Class Imbalance: 80% Transactions Without Promotion 

**The Problem:**
If 80% of transactions occurred without any promotion, our dataset is heavily imbalanced. When we aggregate to the store-month level and define the target as the best-performing promotion, the model may see very few examples of some promotion types (especially those used rarely). A naive model would learn to predict "no promotion" or the most frequent promotion for almost every store, achieving high accuracy on paper but being useless in practice.

**How This Imbalance Could Affect the Model:**
- The model gets biased towards the majority class (no promotion / most common promotion).
- Rare but effective promotions (e.g., "Free Gift with Purchase") will be under-predicted.
- Standard accuracy metric will be misleading — 80% accuracy by just always predicting the majority class.

**Steps to Address the Imbalance:**

1. **Resampling Techniques:**
   - **Oversampling** (SMOTE — Synthetic Minority Oversampling Technique): Artificially generate new examples of underrepresented promotion types so that the model gets adequate exposure to them during training.
   - **Undersampling**: Reduce the number of majority-class examples to balance the dataset (risk: loss of information).

2. **Class Weights:**
   - Assign higher penalty/weight to misclassifying minority promotion types during model training. Most ML libraries (e.g., scikit-learn) support a `class_weight='balanced'` parameter.

3. **Change Evaluation Metric:**
   - Do not use Accuracy as the primary metric. Instead, use **Macro F1-Score** or **Balanced Accuracy**, which give equal importance to all classes regardless of frequency.

4. **Focus EDA on Promotion-Active Months:**
   - When analysing promotion effectiveness, filter data to only those store-months where a promotion was actually run, before building the recommendation model.

---

## B3. Model Evaluation and Deployment — 

### B3(a) — Train-Test Split Strategy and Evaluation Metrics 

**Why a Random Split is Inappropriate:**

Our data is **time-series in nature** — it spans 36 months (3 years) across 50 stores. A random split would allow data from, say, December 2024 to be in the training set while October 2024 is in the test set. This means the model would be trained on "future" data and tested on "past" data, which is a form of **data leakage**. The model would appear to perform well during evaluation but would fail in real deployment because in reality, we always predict future months using only past data.

**Correct Approach — Time-Based Split:**

Use a **temporal (chronological) split**:
- **Training Set:** Months 1 to 30 (first 2.5 years — roughly Jan 2022 to Jun 2024)
- **Validation Set:** Months 31 to 33 (3 months for hyperparameter tuning)
- **Test Set:** Months 34 to 36 (last 3 months — held out completely until final evaluation)

This ensures the model is always trained on past data and evaluated on truly unseen future data, simulating real-world deployment conditions.

Optionally, use **Walk-Forward Validation (Rolling Window Cross-Validation)**: Train on months 1–12, test on month 13; then train on 1–13, test on 14; and so on. This gives a more robust evaluation across different time periods.

**Evaluation Metrics and Their Business Interpretation:**

| Metric | What It Measures | Business Interpretation |
|---|---|---|
| **Macro F1-Score** | Harmonic mean of Precision and Recall across all 5 promotion classes equally | Are we identifying the best promotion correctly for all types, not just the popular ones? |
| **Per-Class F1-Score** | F1 for each individual promotion type | Which specific promotion is the model struggling to recommend correctly? |
| **Accuracy** | % of store-months where the correct promotion is predicted | Overall correctness — but misleading if classes are imbalanced, so use alongside F1 |
| **Confusion Matrix** | A table showing predicted vs actual promotion for all combinations | Reveals which promotions are being confused with each other — useful for business debugging |

**Business Interpretation Example:** If the Macro F1 is 0.72, it means the model correctly identifies the best promotion in about 72% of store-month scenarios across all promotion types. A per-class F1 of 0.45 for "Free Gift with Purchase" tells the marketing team that this promotion is particularly hard to predict and may need more data collection or feature engineering.

---

### B3(b) — Feature Importance: Why Different Recommendations for the Same Store 

**The Observation:**
The model recommends "Loyalty Points Bonus" for Store 12 in December but "Flat Discount" for Store 12 in March. Both predictions are for the same store. This seems contradictory but can be fully explained using **feature importance**.

**Concept of Feature Importance:**
Feature importance tells us which input variables (features) the model relies on most when making a prediction. In tree-based models like Random Forest or XGBoost (commonly used for such problems), feature importance is measured by how much each feature reduces prediction error across all decision splits.

**Investigation Steps:**

1. **Extract Feature Importance Scores:**
   Use the model's built-in feature importance method (e.g., `model.feature_importances_` in scikit-learn) to get a ranked list of which features mattered most for Store 12's predictions in December vs March.

2. **Compare Feature Values for Store 12 — December vs March:**

| Feature | December Value | March Value |
|---|---|---|
| Festival Flag | 1 (Christmas/New Year) | 0 |
| Footfall | Very High | Moderate |
| Month | 12 | 3 |
| Previous Month Sales | High | Low |
| Competition Density | Medium | Medium |

3. **Explanation:**
   - In **December**, the festival flag is active, footfall is very high, and customers are in a gift-buying mood. The model learns (from historical data) that during festive, high-footfall months, customers at Store 12 respond best to Loyalty Points Bonus — because regular customers accumulate points for future use and buy more items.
   - In **March**, there is no festival, footfall is moderate, and customers are more price-sensitive. The model recommends Flat Discount because historical data shows that a direct price cut is needed to motivate purchases during low-season months.

**How to Communicate to the Marketing Team:**
Present a simple explanation: *"The model uses seasonal and footfall context to tailor recommendations. In December, Store 12 gets loyalty-based promotion because its customers shop heavily during festivals and value rewards. In March, the same store gets a flat discount because footfall drops and price sensitivity rises — a direct discount is what drives purchases in that context."*

Use a SHAP (SHapley Additive exPlanations) plot to visually show which features pushed the prediction towards each promotion for each month. This is easy to present to non-technical stakeholders.

---

### B3(c) — End-to-End Deployment Process 

**The Requirement:**
The model must generate promotion recommendations at the start of every month for all 50 stores, without being retrained each time.

**Step 1 — Saving the Trained Model:**
- After training, serialise and save the model using `joblib` or `pickle` in Python (e.g., `joblib.dump(model, 'promotion_model_v1.pkl')`).
- Save alongside it: the feature scaler/encoder, the column order, and the model version metadata (training date, data range, performance metrics).
- Store all files in a cloud storage location (e.g., AWS S3, Azure Blob) for reliable access.

**Step 2 — Preparing and Feeding New Monthly Data:**
At the start of each new month, run a data pipeline that:
1. Pulls last month's transaction data from the data warehouse.
2. Aggregates it to store-month level (same aggregations as modelling: total items sold, average transaction value, etc.).
3. Joins with store attributes and calendar table for the upcoming month.
4. Applies the same preprocessing steps used during training (scaling, encoding, feature engineering).
5. Feeds the prepared feature matrix into the saved model to generate predictions.
6. Outputs a table: `store_id → recommended_promotion` for all 50 stores.

**Step 3 — Automation:**
Schedule this pipeline using an orchestration tool (e.g., Apache Airflow, AWS Lambda, or a simple cron job) to run automatically on the 1st of every month. The marketing team receives the recommendations via email or a dashboard without any manual intervention.

**Step 4 — Monitoring for Model Degradation:**
The model's performance will degrade over time as customer behaviour, competition, and market conditions change. Put the following monitoring measures in place:

| Monitoring Check | How to Do It | Trigger for Retraining |
|---|---|---|
| **Prediction Accuracy Tracking** | Compare the model's recommended promotion with the actual best-performing promotion each month (calculated post-hoc from sales data) | If monthly accuracy drops below a defined threshold (e.g., below 60% for 2 consecutive months) |
| **Feature Drift Detection** | Monitor the distribution of input features (e.g., footfall, transaction counts) month-over-month using statistical tests (KS-test) | If feature distributions shift significantly, indicating the model is seeing data very different from training |
| **Business Metric Tracking** | Track overall items sold per month across all 50 stores | If items sold drops consistently after following model recommendations, investigate and retrain |
| **Scheduled Periodic Retraining** | Retrain the model every 6 months using all available historical data | Scheduled retraining — even if no degradation is detected — to incorporate new behavioural patterns |

**Retraining Process:**
When retraining is triggered, run the full training pipeline (data preparation → model training → evaluation → validation) and replace the saved model file only if the new model outperforms the old one on the validation set. This prevents a bad retrain from going into production.

---

