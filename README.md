# ml-assessment--bishakha-dey-

### Machine Learning Assignment | Supervised · Unsupervised · Feature Engineering · Business Analysis

---

## 👤 Student Details

| Field | Details |
|---|---|
| **Name** | [ Bishakha Dey ] |
| **Program** | Business Analytics |
| **Student Code** | [ bitsom_ba_2511726 ] 

## 📁 Repository Structure

```
├── data/
│   ├── q1_heart_disease.csv          # Heart disease classification dataset (800 rows, 12 cols)
│   ├── q2_customers.csv              # Customer segmentation dataset (500 rows, 6 cols)
│   └── q3_retail_promotions.csv      # Retail promotions regression dataset (1200 rows, 9 cols)
│
├── q1_supervised.ipynb               # Q1: Supervised Learning — Classification 
├── q2_unsupervised.ipynb             # Q2: Unsupervised Learning — K-Means + PCA 
├── q3_feature_engineering.ipynb      # Q3: Feature Engineering + Regression Pipeline 
│
├── part_b/
│   └── business_analysis.md          # Part B: Business Case Analysis
│
└── README.md                         
```

---

## 📊 Assignment Overview

| Question | Topic | File | 
|---|---|---|---|
| Q1 | Supervised Learning — Heart Disease Classification | `q1_supervised.ipynb` | 
| Q2 | Unsupervised Learning — Customer Segmentation (K-Means + PCA) | `q2_unsupervised.ipynb` | 
| Q3 | Feature Engineering & Regression Pipeline | `q3_feature_engineering.ipynb` | 
| Part B | Business Case Analysis — Promotion Effectiveness | `part_b/business_analysis.md` | 

---

## Q1. Supervised Learning — `q1_supervised.ipynb` — 

### 🎯 Objective
Build and evaluate classification models to predict whether a patient has heart disease.

### 📂 Dataset: `q1_heart_disease.csv`
- **Rows:** 800 | **Columns:** 12
- **Target Variable:** `heart_disease` (1 = disease present, 0 = absent)
- **Missing Values:** 56 (handled in preprocessing)

| Column | Type | Description |
|---|---|---|
| `age` | Numeric | Patient age in years |
| `sex` | Binary | 0 = Female, 1 = Male |
| `chest_pain_type` | Categorical | Type of chest pain (atypical_angina, non_anginal, asymptomatic, etc.) |
| `resting_bp` | Numeric | Resting blood pressure (mm Hg) |
| `cholesterol` | Numeric | Serum cholesterol (mg/dl) |
| `fasting_bs` | Binary | Fasting blood sugar > 120 mg/dl (1 = true, 0 = false) |
| `resting_ecg` | Categorical | Resting ECG results |
| `max_hr` | Numeric | Maximum heart rate achieved |
| `exercise_angina` | Binary | Exercise-induced angina (1 = yes, 0 = no) |
| `oldpeak` | Numeric | ST depression induced by exercise |
| `st_slope` | Categorical | Slope of peak exercise ST segment |
| `heart_disease` | Binary | **Target** — 1 = Disease, 0 = No Disease |

### ✅ Tasks Completed

**1. Data Loading and Inspection **
- Loaded `q1_heart_disease.csv` using `pd.read_csv('../data/q1_heart_disease.csv')`
- Displayed shape, data types, and missing value counts
- Showed first 5 rows using `.head()`

**2. Exploratory Data Analysis **
- Produced 3+ visualisations including:
  - Target class distribution plot (heart disease vs no disease)
  - Correlation heatmap of all numeric features
  - Age distribution by disease status (box plot)
- Each plot followed by a markdown cell interpreting findings

**3. Data Preprocessing **
- Handled 56 missing values using median imputation (justified in markdown)
- Applied one-hot encoding to all categorical variables (`chest_pain_type`, `resting_ecg`, `st_slope`)
- Scaled numerical features using `StandardScaler`
- Split data using `train_test_split` with `stratify=y` and `random_state=42`

**4. Model Training **
Trained three models using scikit-learn with `random_state=42`:
- Decision Tree Classifier
- Random Forest Classifier
- Gradient Boosting Classifier

**5. Model Evaluation **
For each model:
- Printed confusion matrix
- Reported Precision, Recall, and F1-score on the test set
- Wrote markdown cell identifying the best model and justifying conclusion using all metrics (not just accuracy)

**6. Hyperparameter Tuning **
- Used `GridSearchCV` to tune the best-performing model
- Reported best parameters found
- Compared tuned model performance against untuned baseline

---

## Q2. Unsupervised Learning — `q2_unsupervised.ipynb` — 

### 🎯 Objective
Perform customer segmentation using K-Means clustering and visualise results using PCA.

### 📂 Dataset: `q2_customers.csv`
- **Rows:** 500 | **Columns:** 6
- **No missing values**

| Column | Type | Description |
|---|---|---|
| `age` | Numeric | Customer age |
| `annual_spend` | Numeric | Total annual spending (₹) |
| `visits_per_month` | Numeric | Average store visits per month |
| `basket_size` | Numeric | Average number of items per visit |
| `days_since_last_visit` | Numeric | Days elapsed since last purchase |
| `num_categories_purchased` | Numeric | Number of distinct product categories bought |

### ✅ Tasks Completed

**1. Data Preparation **
- Loaded `q2_customers.csv` using `pd.read_csv('../data/q2_customers.csv')`
- Scaled all 6 features using `StandardScaler`
- Explained in markdown why scaling is essential before K-Means:
  > K-Means uses Euclidean distance to measure similarity between points. Without scaling, features with large numeric ranges (e.g., `annual_spend` in thousands) would dominate the distance calculation and overshadow features like `visits_per_month`, making clusters meaningless.

**2. Choosing K — Elbow Method **
- Computed WCSS (Within-Cluster Sum of Squares) for K = 1 through 10
- Plotted the Elbow Curve
- Identified the optimal K from the elbow point and justified the selection in a markdown cell

**3. K-Means Clustering **
- Fit K-Means with the chosen K value
- Added a `cluster` column to the dataframe
- Printed cluster centroids as a readable dataframe
- Wrote markdown interpreting each cluster in business terms (e.g., "Cluster 0 represents young, frequent low-spenders who visit often but spend less per visit")

**4. Dimensionality Reduction with PCA **
- Applied PCA to reduce data to 2 principal components
- Printed explained variance ratio for each component
- Printed feature loadings (components) as a readable dataframe
- Interpreted what PC1 and PC2 capture based on the loadings in a markdown cell

**5. Cluster Visualisation **
- Created scatter plot of PC1 vs PC2 with points coloured by cluster label
- Included title, axis labels, and legend

---

## Q3. Feature Engineering and Regression Pipeline — `q3_feature_engineering.ipynb` — 

### 🎯 Objective
Build a reproducible scikit-learn regression pipeline to predict `items_sold` at a retail store.

### 📂 Dataset: `q3_retail_promotions.csv`
- **Rows:** 1200 | **Columns:** 9
- **No missing values**

| Column | Type | Description |
|---|---|---|
| `transaction_date` | Date | Date of the store transaction |
| `store_id` | Numeric | Unique identifier for each store |
| `store_size` | Categorical | Size of store (small / medium / large) |
| `location_type` | Categorical | Store location (urban / semi-urban / rural) |
| `promotion_type` | Categorical | Active promotion (flat_discount / bogo / free_gift / category_offer / loyalty_points) |
| `is_weekend` | Binary | 1 = Weekend, 0 = Weekday |
| `is_festival` | Binary | 1 = Festival day, 0 = Regular day |
| `competition_density` | Numeric | Number of competing stores nearby |
| `items_sold` | Numeric | **Target** — Number of items sold |

### ✅ Tasks Completed

**1. Date Feature Engineering **
- Extracted `year`, `month`, `day_of_week` from `transaction_date`
- Created binary feature `is_month_end` (1 if day of month ≥ 25, else 0)
- Displayed sample of resulting dataframe to confirm new columns

**2. Temporal Train-Test Split **
- Sorted data by `transaction_date`
- Used the most recent 20% of records as the test set, remaining 80% as training set
- Did **not** use random split
- Explained in markdown why random split is inappropriate for time-ordered data:
  > A random split would allow future dates into the training set and past dates into the test set, causing data leakage. The model would learn from the "future" which is impossible in real deployment. Temporal split ensures the model is always trained on past data and evaluated on truly unseen future data.

**3. Preprocessing Pipeline **
- Built a scikit-learn `Pipeline` using `ColumnTransformer` that applies:
  - One-hot encoding to `promotion_type`, `location_type`, and `store_size`
  - `StandardScaler` to all numerical features
- Pipeline was fit only on the training set and applied to both train and test sets

**4. Model Training and Evaluation **
- Trained **Linear Regression** and **Random Forest Regressor** (both inside the pipeline)
- Reported **RMSE** and **MAE** on the test set for each model
- Produced parity plots (predicted vs actual `items_sold`) for each model with a diagonal reference line
- Printed feature importances from Random Forest and identified the **top 5 most influential features**

---

## Part B. Business Case Analysis — `part_b/business_analysis.md` — 

### 🎯 Scenario
A fashion retailer with 50 stores (urban, semi-urban, rural) runs one of 5 promotions each month. The goal is to determine which promotion should be deployed in each store each month to maximise items sold.

### ✅ Sections Covered

**B1. Problem Formulation **
- B1(a): Target variable, candidate input features, ML problem type (Multi-Class Classification) with full justification
- B1(b): Why items sold is a better target than revenue — business and statistical reasoning
- B1(c): Why one global model fails; proposed cluster-based segmented modelling strategy

**B2. Data and EDA Strategy **
- B2(a): How to join 4 tables (transactions, store attributes, promotions, calendar); dataset grain = 1 store × 1 month; all required aggregations listed
- B2(b): Four EDA analyses — promotion vs location bar chart, seasonal trend analysis, correlation heatmap, target variable distribution
- B2(c): Handling 80% class imbalance — SMOTE, class weighting, Macro F1-Score metric

**B3. Model Evaluation and Deployment **
- B3(a): Temporal train-test split strategy; why random split causes data leakage; 4 evaluation metrics with business interpretation
- B3(b): Feature importance + SHAP explanation for why the same store gets different recommendations in different months
- B3(c): End-to-end deployment pipeline — model saving, monthly data pipeline, automation, monitoring, and retraining strategy

---

## 🛠️ Tools & Libraries Used

| Category | Tools |
|---|---|
| **Data Manipulation** | pandas, numpy |
| **Visualisation** | matplotlib, seaborn |
| **Machine Learning** | scikit-learn (Pipeline, ColumnTransformer, GridSearchCV) |
| **Models Used** | Decision Tree, Random Forest, Gradient Boosting, Linear Regression, K-Means, PCA |
| **Preprocessing** | StandardScaler, OneHotEncoder, SimpleImputer |
| **Explainability** | Feature Importance, SHAP (referenced in Part B) |

---

## ⚙️ How to Run the Notebooks

1. Clone this repository:
   ```bash
   git clone https://github.com/bishakhadey882-byte/ml-assessment--bishakha-dey-.git
   cd ml-assessment--bishakha-dey-
   ```

2. Install required libraries:
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn jupyter
   ```

3. Launch Jupyter:
   ```bash
   jupyter notebook
   ```

4. Open notebooks in this order:
   - `q1_supervised.ipynb`
   - `q2_unsupervised.ipynb`
   - `q3_feature_engineering.ipynb`

---

## 📌 Key Concepts Demonstrated

- Binary and multi-class classification
- Handling missing values with documented strategy
- One-hot encoding and feature scaling
- Stratified train-test split
- K-Means clustering and elbow method
- Principal Component Analysis (PCA) for dimensionality reduction
- Temporal train-test split for time-ordered data
- Scikit-learn Pipeline with ColumnTransformer
- Model evaluation using Precision, Recall, F1-Score, RMSE, MAE
- Hyperparameter tuning with GridSearchCV
- Feature importance analysis
- Business interpretation of all ML results

---

