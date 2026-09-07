# Data Preprocessing — Study Notes

## Table of Contents
1. [Class Imbalance](#1-class-imbalance)
   - [Undersampling](#undersampling)
   - [Oversampling](#oversampling)
   - [SMOTE — Interview Q&A](#smote--interview-qa)
2. [Data Interpolation](#2-data-interpolation)
3. [Outliers](#3-outliers)
   - [Detecting Outliers (IQR Method)](#detecting-outliers-iqr-method)
   - [Techniques to Handle Outliers](#techniques-to-handle-outliers)
4. [Feature Extraction, Feature Selection & Feature Scaling](#4-feature-extraction-feature-selection--feature-scaling)
   - [Why we need them — Curse of Dimensionality](#why-we-need-them--curse-of-dimensionality)
   - [Feature Extraction vs Feature Selection](#feature-extraction-vs-feature-selection)
   - [Feature Selection Methods: Filter, Wrapper, Embedded](#feature-selection-methods-filter-wrapper-embedded)
   - [Feature Scaling — Types, Formulas, Pros & Cons](#feature-scaling--types-formulas-pros--cons)
5. [Data Encoding](#5-data-encoding)
   - [Nominal / One-Hot Encoding](#nominal--one-hot-encoding-ohe)
   - [Label & Ordinal Encoding](#label--ordinal-encoding)
   - [Target-Guided Ordinal Encoding](#target-guided-ordinal-encoding)
   - [Encoding — Interview Q&A](#encoding--interview-qa-how-do-you-decide-which-encoding-technique-to-use)

---

## 1. Class Imbalance

**Class imbalance** occurs when the classes in a classification problem are not represented equally — e.g., in fraud detection, 99% of transactions are "genuine" and only 1% are "fraud." A model trained naively on such data tends to become biased toward the majority class and performs poorly at detecting the minority class (which is usually the class we care about most).

### Undersampling
Reduce the number of samples in the **majority class** to match the minority class.
- **Pros:** Faster training, less memory.
- **Cons:** Can throw away potentially useful data, risk of underfitting.
- **When to use:** When you have a very large dataset and can afford to lose some majority-class data.

### Oversampling
Increase the number of samples in the **minority class** (e.g., by duplicating existing samples or generating synthetic ones).
- **Pros:** No data loss.
- **Cons:** Simple duplication can cause overfitting (model memorizes repeated minority samples).
- **When to use:** When the dataset is small and you can't afford to lose majority-class data.

### SMOTE — Interview Q&A

> **Q: What is SMOTE, when should you use it, and how does it work?**

**A:**
**SMOTE (Synthetic Minority Over-sampling Technique)** is an oversampling method that addresses class imbalance by generating **new synthetic samples** for the minority class, rather than simply duplicating existing ones.

**When to use it:**
- When the dataset has significant class imbalance (e.g., fraud detection, disease diagnosis, churn prediction).
- When simple random oversampling would cause overfitting due to exact duplicate records.
- Best suited for **continuous/numerical features**; not ideal for purely categorical data (SMOTE-NC is a variant for mixed data).

**How it works (step-by-step):**
1. For each minority-class sample, find its **k-nearest neighbors** (also from the minority class) — typically using Euclidean distance.
2. Randomly select one (or more) of these neighbors.
3. Create a **synthetic sample** along the line segment joining the original point and the chosen neighbor:
   $$x_{new} = x_i + \lambda \times (x_{neighbor} - x_i), \quad \lambda \in [0, 1] \text{ (random)}$$
4. Repeat this process until the minority class is sufficiently balanced with the majority class.

**Key point for interviews:** SMOTE doesn't just copy-paste minority samples — it interpolates **between** real minority samples to create plausible new ones, which reduces overfitting compared to naive oversampling.

```python
from imblearn.over_sampling import SMOTE

smote = SMOTE(sampling_strategy='auto', k_neighbors=5, random_state=42)
X_resampled, y_resampled = smote.fit_resample(X_train, y_train)
```

**Limitation to mention in interviews:** SMOTE can generate noisy samples if minority-class points are near the decision boundary or overlap with majority-class points — variants like **Borderline-SMOTE** or **ADASYN** address this.

---

## 2. Data Interpolation

**Data interpolation** is a method of estimating unknown values that fall **between** known data points. It's used when we have a set of discrete data points and want to predict/fill in intermediate values (commonly for time-series or missing-value imputation).

### Types of Interpolation

| Type | Definition | Example |
|---|---|---|
| **Linear Interpolation** | Assumes a straight line between two known points and estimates the value on that line. | Temperature at 9 AM = 20°C and at 11 AM = 24°C. Missing value at 10 AM is estimated as the midpoint: **22°C**. |
| **Polynomial Interpolation** | Fits a polynomial curve through multiple known points for a smoother, non-linear estimate. | Given stock prices on Day 1, 3, and 5 that follow a curved trend, a 2nd-degree polynomial is fit through all three points to estimate Day 2 and Day 4. |
| **Spline Interpolation** | Fits piecewise polynomials (splines) between each pair of points, ensuring smooth transitions — avoids the erratic swings polynomial interpolation can have. | Smoothly estimating missing daily temperature readings across a month, where a single high-degree polynomial would oscillate wildly but a spline stays smooth segment-by-segment. |
| **Time-based Interpolation** | Interpolates based on the actual time index/gaps rather than row position — useful for irregularly spaced time-series data. | Sensor readings logged at 9:00, 9:10, and 9:45 (uneven gaps) — time-based interpolation correctly weights the estimate for a missing 9:20 reading closer to the 9:10 value. |
| **Nearest Neighbor Interpolation** | Fills in the value of the nearest known data point (by index or distance), instead of computing an intermediate value. | Missing category-like numeric code between two known points is simply replaced with whichever known point is closer, rather than an averaged value. |

```python
import pandas as pd

df['value'] = df['value'].interpolate(method='linear')      # straight-line
df['value'] = df['value'].interpolate(method='polynomial', order=2)
df['value'] = df['value'].interpolate(method='time')        # time-indexed series
```

**When to use interpolation vs. simple imputation:** Interpolation is preferred over mean/median imputation when data has a natural **order or trend** (e.g., time series, sensor readings) — since it preserves the trend, unlike a static mean fill.

---

## 3. Outliers

**Outliers** are data points that differ significantly from the rest of the dataset — either unusually high or low. They can distort statistical measures (mean, standard deviation) and negatively affect model training, especially for distance-based or linear models.

### Detecting Outliers (IQR Method)

The **Interquartile Range (IQR)** method is one of the most common statistical techniques:

$$IQR = Q3 - Q1$$
$$\text{Lower Fence} = Q1 - 1.5 \times IQR$$
$$\text{Upper Fence} = Q3 + 1.5 \times IQR$$

Any data point **below the Lower Fence** or **above the Upper Fence** is flagged as an outlier.

**Worked example:** Suppose `Q1 = 42.9` and `Q3 = 55.2`, so `IQR = 12.3`.
- Lower Fence = 42.9 − (1.5 × 12.3) = **24.3**
- Upper Fence = 55.2 + (1.5 × 12.3) = **73.8**

Any value below 24.3 or above 73.8 is an outlier — as visualized below:

![Outlier detection using IQR method](outlier_iqr_diagram.png)

```python
import numpy as np

Q1 = df['value'].quantile(0.25)
Q3 = df['value'].quantile(0.75)
IQR = Q3 - Q1

lower_fence = Q1 - 1.5 * IQR
upper_fence = Q3 + 1.5 * IQR

# Flagging outliers
outliers = df[(df['value'] < lower_fence) | (df['value'] > upper_fence)]

# Using np.where to cap (winsorize) outliers instead of dropping
df['value'] = np.where(df['value'] > upper_fence, upper_fence,
                np.where(df['value'] < lower_fence, lower_fence, df['value']))
```
<img width="1350" height="675" alt="image" src="https://github.com/user-attachments/assets/aff99068-0656-4edb-aaac-1ead307094a0" />

### Techniques to Handle Outliers

> **Refined interview question: "What techniques would you use to handle outliers, and how do you decide which one to use?"**

| Technique | What it does | When to use |
|---|---|---|
| **Dropping/Removing** | Delete rows containing outliers entirely | When outliers are clearly **data errors** (e.g., age = 300) and the dataset is large enough that losing rows won't hurt |
| **Capping / Winsorization** (using IQR fences) | Replace outlier values with the nearest fence value (upper/lower bound) | When outliers are **genuine but extreme** values and you don't want to lose the row, only tame its effect |
| **Imputation with Mean/Median** | Replace outlier value with the column's mean or median | When you want to preserve row count and the outlier is likely a data-entry mistake; **median preferred** over mean since mean itself is sensitive to outliers |
| **Log/Box-Cox Transformation** | Compress the scale of the data to reduce the influence of extreme values | When data is highly right-skewed (e.g., income, prices) |
| **Binning** | Convert continuous values into discrete bins/ranges | When outlier impact needs to be reduced without deleting information, and exact values aren't critical |
| **Z-score method** | Flag points where `(x - mean) / std_dev` exceeds a threshold (commonly ±3) | Best for data that is roughly **normally distributed**; unlike IQR, it's sensitive to the mean/std themselves being skewed by outliers |
| **Using robust models** | Use tree-based models (Random Forest, XGBoost) which are naturally less sensitive to outliers | When you don't want to modify the data at all |

**Example:** Capping outliers using `np.where` (as shown in code above) — this is often preferred over dropping because it retains all data points while pulling extreme values back to a reasonable range (also called **winsorizing**).

---

## 4. Feature Extraction, Feature Selection & Feature Scaling

### Why we need them — Curse of Dimensionality

As the number of features (dimensions) in a dataset grows, the volume of the feature space increases so fast that the available data becomes sparse. This is called the **curse of dimensionality**, and it causes:
- Models needing exponentially more data to generalize well.
- Increased risk of overfitting.
- Increased computation cost.
- Distance-based algorithms (KNN, K-Means) becoming less meaningful, since all points start looking equally "far apart."

**Feature extraction and feature selection both fight the curse of dimensionality** — by reducing the number of features the model has to deal with, while trying to retain (or even improve) predictive power.

**Feature scaling**, on the other hand, doesn't reduce dimensions — it addresses a different problem: **without scaling, features with larger numeric ranges can dominate the model's learning process**, making the model biased toward those features even if they aren't more important (this mainly affects distance-based and gradient-based algorithms, not tree-based ones).

### Feature Extraction vs Feature Selection

| | **Feature Extraction** | **Feature Selection** |
|---|---|---|
| **Definition** | Creates **new features** by transforming/combining original features into a lower-dimensional space | Selects a **subset of existing features**, discarding the rest |
| **Example techniques** | PCA, LDA, Autoencoders, t-SNE | Filter, Wrapper, Embedded methods |
| **Interpretability** | Lower — new features are combinations, harder to interpret | Higher — original, meaningful features are kept |
| **When to use** | When features are highly correlated and you want compressed representations (e.g., image/text data) | When you want to keep original features for interpretability, or remove irrelevant/redundant ones |

### Feature Selection Methods: Filter, Wrapper, Embedded

| Method | How it works | Example techniques | Pros | Cons |
|---|---|---|---|---|
| **Filter Method** | Selects features based on their statistical relationship with the target, independent of any ML model | Correlation coefficient, Chi-square test, ANOVA, Mutual Information | Fast, model-agnostic, scales well | Ignores feature interactions and doesn't consider the model being used |
| **Wrapper Method** | Uses a specific ML model to evaluate subsets of features by actually training and testing the model on them | Forward Selection, Backward Elimination, Recursive Feature Elimination (RFE) | Considers feature interactions, usually gives better-performing subsets | Computationally expensive, risk of overfitting to that specific model |
| **Embedded Method** | Feature selection happens **during model training** itself, as part of the algorithm | Lasso (L1) Regression, Decision Tree feature importances, Random Forest feature importances | Efficient (no separate search process), captures interactions | Tied to a specific model's assumptions |

### Feature Scaling — Types, Formulas, Pros & Cons

Feature scaling brings all numerical features onto a **similar scale**, so no single feature dominates the model just because of its magnitude. This is crucial for algorithms like KNN, K-Means, SVM, PCA, and gradient-descent-based models (linear/logistic regression, neural networks). It is **not required** for tree-based models (Decision Tree, Random Forest, XGBoost), since they split on thresholds rather than distances.

| Type | Formula | Pros | Cons | When to use |
|---|---|---|---|---|
| **Min-Max Scaling (Normalization)** | $$x' = \frac{x - x_{min}}{x_{max} - x_{min}}$$ | Bounds data to a fixed range [0,1]; preserves relationships between values | Very sensitive to outliers (a single extreme value compresses everything else) | When data doesn't have significant outliers and you need bounded input (e.g., neural networks, image pixel values) |
| **Standardization (Z-score Scaling)** | $$x' = \frac{x - \mu}{\sigma}$$ | Less sensitive to outliers than Min-Max; centers data around 0 with unit variance | Doesn't produce a bounded range; assumes data roughly follows a Gaussian-like spread | When data may have outliers, or algorithm assumes normally distributed features (e.g., logistic regression, SVM, PCA) |
| **Robust Scaling** | $$x' = \frac{x - \text{median}}{IQR}$$ | Explicitly designed to be robust to outliers, since it uses median & IQR instead of mean & std | Doesn't produce a fixed bounded range | When the dataset has many outliers you don't want to remove |
| **Max-Abs Scaling** | $$x' = \frac{x}{|x_{max}|}$$ | Preserves sparsity (good for sparse data), doesn't shift/center data | Still sensitive to outliers since it relies on max value | When working with sparse data (e.g., text/TF-IDF features) |
| **Log Transformation** | $$x' = \log(x + 1)$$ | Reduces skewness and impact of large outliers | Only works on positive values; doesn't literally "scale" to a fixed range | When data is highly right-skewed (income, population, prices) |

```python
from sklearn.preprocessing import MinMaxScaler, StandardScaler, RobustScaler, MaxAbsScaler
import numpy as np

# Min-Max
mm_scaler = MinMaxScaler()
df_scaled = mm_scaler.fit_transform(df[['feature']])

# Standardization
std_scaler = StandardScaler()
df_scaled = std_scaler.fit_transform(df[['feature']])

# Robust Scaling (best when outliers are present)
rb_scaler = RobustScaler()
df_scaled = rb_scaler.fit_transform(df[['feature']])

# Max-Abs Scaling (best for sparse data, e.g. TF-IDF matrices)
ma_scaler = MaxAbsScaler()
df_scaled = ma_scaler.fit_transform(df[['feature']])

# Log Transformation (best for right-skewed data like income/price)
df['feature_log'] = np.log1p(df['feature'])   # log1p = log(x + 1), handles zeros safely
```

---
<img width="885" height="557" alt="image" src="https://github.com/user-attachments/assets/5b20e31f-713b-4593-b395-79d322f7af7e" />

## 5. Data Encoding

**Why we need encoding:** Most ML algorithms work with numbers, not text/categories. Encoding converts categorical (non-numeric) data into a numeric form the model can process, **without introducing incorrect relationships between categories** where none exist.

### Nominal / One-Hot Encoding (OHE)

**Definition:** Used for **nominal** categorical data — categories with **no inherent order** (e.g., "Red," "Blue," "Green"). Creates a new binary (0/1) column for each category.

*Example:*
| Color | Red | Blue | Green |
|---|---|---|---|
| Red | 1 | 0 | 0 |
| Blue | 0 | 1 | 0 |
| Green | 0 | 0 | 1 |

```python
import pandas as pd
df_encoded = pd.get_dummies(df, columns=['Color'])
```

| Advantages | Disadvantages |
|---|---|
| No false ordinal relationship implied between categories | Leads to **high dimensionality** with many unique categories (curse of dimensionality) |
| Works well with linear models and neural networks | Can create sparse matrices, increasing memory usage |

**When to use:** Nominal categories with a **small-to-moderate number of unique values**, and when using models sensitive to numeric ordering (e.g., linear/logistic regression).

### Label & Ordinal Encoding

**Label Encoding:** Assigns a unique integer to each category. Best suited for **target/label columns**, or nominal features being fed into **tree-based models** (which don't assume order).

*Example:* Red → 0, Blue → 1, Green → 2

**Ordinal Encoding:** Same mechanism as label encoding, but used specifically when categories **do have a natural order** — the assigned integers should reflect that order.

*Example:* Education level → High School: 0, Bachelor's: 1, Master's: 2, PhD: 3

```python
from sklearn.preprocessing import LabelEncoder, OrdinalEncoder

# Label Encoding (for target variable or nominal + tree models)
le = LabelEncoder()
df['Color_encoded'] = le.fit_transform(df['Color'])

# Ordinal Encoding (for ordered categories, preserving rank)
oe = OrdinalEncoder(categories=[['High School', "Bachelor's", "Master's", 'PhD']])
df['Education_encoded'] = oe.fit_transform(df[['Education']])
```

| Advantages | Disadvantages |
|---|---|
| Simple, memory-efficient (single column, no dimensionality explosion) | If used on **nominal** data with non-tree models, it wrongly implies order/magnitude relationships between categories |
| Preserves natural order for ordinal data, helping the model learn rank-based patterns | Ordinal encoding requires **domain knowledge** to correctly define the order |

- **Why do tree-based machine learning models work well with Label Encoded nominal features, even though the encoding introduces an arbitrary numerical order?**

<img width="699" height="675" alt="image" src="https://github.com/user-attachments/assets/161e2dc7-4945-4186-bc55-9f6f3b0a58ed" />



### Target-Guided Ordinal Encoding

**Definition:** Categories are encoded based on their relationship with the **target variable** — typically, categories are ranked/ordered according to the mean (or another aggregate) of the target variable for each category, then assigned ordinal ranks (or the mean value itself).

*Example:* Predicting house price based on "City" —
| City | Mean House Price | Encoded Rank |
|---|---|---|
| CityA | 50,000 | 1 |
| CityB | 80,000 | 2 |
| CityC | 120,000 | 3 |

```python
mean_price = df.groupby('City')['Price'].mean().to_dict()
df['City_encoded'] = df['City'].map(mean_price)
```

| Advantages | Disadvantages |
|---|---|
| Captures the relationship between category and target directly, often improving model performance | Risk of **data leakage/overfitting** if computed using the full dataset (must be computed only on training data, and ideally with cross-validation/smoothing) |
| Handles high-cardinality categorical features well without exploding dimensionality (unlike OHE) | Sensitive to rare categories with few samples — their target-mean estimate can be noisy/unreliable |

**When to use:** High-cardinality nominal features (e.g., "City," "Zip Code") where OHE would create too many columns, and you have a supervised target to guide the encoding — commonly used in Kaggle competitions and gradient-boosting pipelines (XGBoost/LightGBM/CatBoost).

### Encoding — Interview Q&A: "How do you decide which encoding technique to use?"

> **Q: You have a categorical column. How do you decide whether to use One-Hot, Label/Ordinal, or Target-Guided Encoding?**

**A:** The decision comes down to three questions:

1. **Does the category have a natural order?**
   - No natural order (nominal, e.g., "Color," "City" with few values) → **One-Hot Encoding**
   - Natural order exists (e.g., "Education Level," "Rating: Low/Medium/High") → **Ordinal Encoding**

2. **How many unique categories does it have (cardinality)?**
   - Low cardinality (a handful of categories) → OHE is fine.
   - High cardinality (hundreds of categories, e.g., "Zip Code," "Product ID") → OHE would blow up dimensionality → prefer **Target-Guided Encoding** or **Label Encoding** (if using tree-based models).

3. **What model are you using?**
   - Linear/logistic regression, SVM, neural nets → these assume a numeric relationship, so **never** use plain Label Encoding on nominal data (it implies false ordering) — use OHE instead.
   - Tree-based models (Decision Tree, Random Forest, XGBoost) → can safely use **Label Encoding** even on nominal data, since trees split on thresholds and don't assume numeric order/magnitude.
   - Gradient boosting with high-cardinality categorical features and a clear target relationship → **Target-Guided Encoding** is a strong choice (used heavily in Kaggle-style tabular pipelines).

| Scenario | Best Choice |
|---|---|
| Nominal, few categories, linear model | One-Hot Encoding |
| Nominal, few categories, tree-based model | Label Encoding (or OHE, either works) |
| Ordinal (has rank), any model | Ordinal Encoding |
| Nominal, high cardinality, any model | Target-Guided Encoding (watch for leakage — fit only on train fold) |

---

## Quick Cheat-Sheet

| Concept | One-liner |
|---|---|
| Undersampling | Reduce majority class |
| Oversampling | Duplicate/increase minority class |
| SMOTE | Creates *synthetic* minority samples via interpolation between neighbors |
| Interpolation | Estimate unknown values between known data points (good for time-series/trend data) |
| IQR outlier rule | Outlier if `x < Q1 - 1.5*IQR` or `x > Q3 + 1.5*IQR` |
| Feature Extraction | Create new compressed features (PCA, LDA) |
| Feature Selection | Keep subset of original features (Filter/Wrapper/Embedded) |
| Min-Max Scaling | Scales to [0,1], sensitive to outliers |
| Standardization | Mean 0, std 1, more outlier-robust than Min-Max |
| Robust Scaling | Uses median/IQR — best when outliers are present |
| One-Hot Encoding | For nominal data, no order, high-cardinality risk |
| Label/Ordinal Encoding | For ordered categories or tree models |
| Target-Guided Encoding | Encodes using target mean — great for high-cardinality features |





