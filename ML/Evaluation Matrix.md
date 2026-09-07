# Machine Learning Evaluation Metrics & Multi-Class Classification — Study Notes

## Table of Contents
1. [Regression Evaluation Metrics](#1-regression-evaluation-metrics)
   - [R² Score (Coefficient of Determination)](#r²-score-coefficient-of-determination)
   - [Adjusted R²](#adjusted-r²)
   - [MSE vs MAE vs RMSE](#mse-vs-mae-vs-rmse)
2. [Classification Evaluation Metrics](#2-classification-evaluation-metrics)
   - [The Confusion Matrix](#the-confusion-matrix)
   - [Core Classification Metrics](#core-classification-metrics)
   - [Precision vs Recall Trade-off](#precision-vs-recall-trade-off-domain-examples)
   - [ROC Curve & Precision-Recall Curve](#roc-curve--precision-recall-curve)
3. [Multi-Class Logistic Regression](#3-multi-class-logistic-regression)
   - [One-vs-Rest & One-vs-One](#strategies-for-multi-class-classification)
   - [Softmax Function](#softmax-function-multinomial-logistic-regression)
   - [Cross-Entropy Loss for Multi-Class](#cross-entropy-loss-for-multi-class)
   - [Micro vs Macro vs Weighted Averaging](#multi-class-metric-aggregation-micro-vs-macro-vs-weighted)
4. [Real Industry Interview Questions](#4-real-industry-interview-questions)

---

## 1. Regression Evaluation Metrics

Regression models predict continuous values. To evaluate them, we measure the distance between actual values ($y_i$) and predicted values ($\hat{y}_i$).

### R² Score (Coefficient of Determination)

$R^2$ represents the proportion of variance in the dependent variable ($y$) that is predictable from the independent variables ($X$).

**Core Sum of Squares Terms:**

1. **Residual Sum of Squares (RSS / SSE):** Unexplained variance by the model.
$$\text{RSS} = \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$

2. **Total Sum of Squares (TSS):** Total variance present in the actual target values relative to the mean ($\bar{y}$).
$$\text{TSS} = \sum_{i=1}^{n} (y_i - \bar{y})^2$$

3. **Explained Sum of Squares (ESS / SSR):** Variance captured by the model.
$$\text{ESS} = \sum_{i=1}^{n} (\hat{y}_i - \bar{y})^2$$

$$\text{TSS} = \text{ESS} + \text{RSS}$$

> ⚠️ **Important caveat:** This identity ($\text{TSS} = \text{ESS} + \text{RSS}$) **only holds for OLS linear regression models that include an intercept term**. For models without an intercept, or non-linear models, this equality can break down — meaning $R^2 = \text{ESS}/\text{TSS}$ can give an incorrect (even nonsensical, e.g. >1 or negative in unexpected ways) result. For that reason, always compute R² using the **canonical, universally valid formula**:
> $$R^2 = 1 - \frac{\text{RSS}}{\text{TSS}}$$
> and treat $R^2 = \text{ESS}/\text{TSS}$ as a convenient shortcut that's only reliable for standard OLS-with-intercept models.

**R² Formula:**
$$R^2 = 1 - \frac{\text{RSS}}{\text{TSS}}$$

![R² decomposition: TSS vs RSS vs ESS](r2_decomposition.png)

**Interpretation & Range:**
- **Range:** Typically $[0, 1]$, but can be negative $(-\infty, 1]$ if the model performs worse than a baseline horizontal mean line ($\bar{y}$).
- **Intuition (Example):** If predicting house prices yields an $R^2 = 0.80$, then 80% of the price variability is explained by features like size, location, and bedrooms. The remaining 20% is unexplained noise or missing variables.

### Adjusted R²

**Why R² is Flawed:**
$R^2$ **never decreases** when you add new features, even if those features are completely irrelevant (e.g., adding "owner's shoe size" to a real estate model). The model uses noise to marginally fit training data, causing $R^2$ to artificially inflate and disguise overfitting.

**Formula:**
$$\text{Adjusted } R^2 = 1 - \left[ \frac{(1 - R^2)(n - 1)}{n - p - 1} \right]$$

Where:
- $n$ = Total sample size
- $p$ = Number of feature predictors

**How It Fixes R²:** It penalizes the model for adding non-informative parameters $p$. Adjusted R² increases **only** if the new feature improves R² more than expected by random chance.

> **Interview tip:** If you add a useless feature and R² barely moves (e.g., 0.801 vs 0.800), Adjusted R² will actually **decrease** — this is the tell-tale sign to check when deciding whether a new feature is genuinely useful.

### MSE vs MAE vs RMSE

| Metric | Formula | Advantages | Disadvantages |
|---|---|---|---|
| **MSE** (Mean Squared Error) | $\frac{1}{n} \sum (y_i - \hat{y}_i)^2$ | Continuous and smooth everywhere; easily differentiable for gradient descent. | Units are squared ($\text{units}^2$), making direct business interpretation difficult; highly sensitive to outliers. |
| **MAE** (Mean Absolute Error) | $\frac{1}{n} \sum \vert y_i - \hat{y}_i\vert$ | Robust to outliers; errors are measured in original target units. | Non-differentiable at $y - \hat{y} = 0$ (sharp V-shape corner); constant gradient ($\pm 1$) causes overshoot/zigzag behavior during optimization. |
| **RMSE** (Root Mean Squared Error) | $\sqrt{\text{MSE}}$ | Returns error to target units; penalizes larger errors more than MAE. | Still sensitive to extreme outliers compared to MAE. |

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
import numpy as np

mse = mean_squared_error(y_true, y_pred)
mae = mean_absolute_error(y_true, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_true, y_pred)

n, p = len(y_true), X_train.shape[1]
adj_r2 = 1 - ((1 - r2) * (n - 1) / (n - p - 1))
```

---

## 2. Classification Evaluation Metrics

Classification models predict categorical class labels or probabilities.

### The Confusion Matrix

A 2×2 grid comparing predicted outcomes against true labels:

|  | Predicted Positive ($\hat{Y}=1$) | Predicted Negative ($\hat{Y}=0$) |
|---|---|---|
| **Actual Positive ($Y=1$)** | **True Positive (TP)** | **False Negative (FN)** *(Type II Error)* |
| **Actual Negative ($Y=0$)** | **False Positive (FP)** *(Type I Error)* | **True Negative (TN)** |

![Confusion matrix example](confusion_matrix.png)

### Core Classification Metrics

**1. Accuracy** — Overall proportion of correctly classified samples.
$$\text{Accuracy} = \frac{\text{TP} + \text{TN}}{\text{TP} + \text{TN} + \text{FP} + \text{FN}}$$
> ⚠️ **Warning:** Highly misleading on imbalanced datasets (e.g., if 99% of transactions are legitimate, a model predicting "legitimate" for every case achieves 99% accuracy while failing entirely at fraud detection).

**2. Misclassification Rate**
$$\text{Error Rate} = 1 - \text{Accuracy} = \frac{\text{FP} + \text{FN}}{\text{TP} + \text{TN} + \text{FP} + \text{FN}}$$

**3. Precision (Positive Predictive Value)** — Out of all instances predicted as Positive, how many were actually Positive?
$$\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}}$$

**4. Recall (Sensitivity / True Positive Rate)** — Out of all actual Positive cases, how many did the model correctly identify?
$$\text{Recall / TPR} = \frac{\text{TP}}{\text{TP} + \text{FN}}$$

**5. Specificity (True Negative Rate)** — Out of all actual Negative cases, how many were correctly identified?
$$\text{Specificity} = \frac{\text{TN}}{\text{TN} + \text{FP}}$$

**6. False Positive Rate (FPR)**
$$\text{FPR} = 1 - \text{Specificity} = \frac{\text{FP}}{\text{TN} + \text{FP}}$$

**7. F1-Score** — Harmonic mean of Precision and Recall. Used when you need a balance between Precision and Recall on imbalanced datasets.
$$F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}} = \frac{2\text{TP}}{2\text{TP} + \text{FP} + \text{FN}}$$

> **Why harmonic mean and not simple average?** The harmonic mean penalizes extreme imbalance between Precision and Recall much more than a simple average would. If Precision = 1.0 and Recall = 0.0, a simple average gives 0.5 (misleadingly decent), but the harmonic mean (F1) gives 0 — correctly reflecting that the model is useless at actually catching positives.

```python
from sklearn.metrics import confusion_matrix, accuracy_score, precision_score, recall_score, f1_score

cm = confusion_matrix(y_true, y_pred)
acc = accuracy_score(y_true, y_pred)
prec = precision_score(y_true, y_pred, zero_division=0)
rec = recall_score(y_true, y_pred, zero_division=0)
f1 = f1_score(y_true, y_pred, zero_division=0)
```
> **Edge case:** `precision_score`, `recall_score`, and `f1_score` raise a warning (and default to 0) when a class has **zero predicted positives** (i.e., `TP + FP = 0`) — this commonly happens with a rare class or a heavily skewed threshold. Passing `zero_division=0` (or `1`, depending on how you want that edge case scored) silences the warning and makes the behavior explicit rather than relying on sklearn's default.

### Precision vs Recall Trade-off (Domain Examples)

Adjusting the decision threshold changes the balance between Precision and Recall.

> **In plain words:** Precision matters when you **can't afford a false positive** — like an email classifier, where you can't afford to let an important mail get sent to spam/trash. Recall matters when you **can't afford to miss an actual positive case** — like cancer detection, where if the model says "yes," the person can always be re-examined and cleared, but if it wrongly says "no," that person may never get the treatment they actually needed.

**Scenario A: High Precision Focus (Minimize False Positives)**
- **Example:** Email Spam Filter.
- **Why:** Marking an important work email as Spam (False Positive) is far worse than letting an actual spam email into the inbox (False Negative).

**Scenario B: High Recall Focus (Minimize False Negatives)**
- **Example:** Medical Cancer Detection.
- **Why:** Missing an actual cancer diagnosis (False Negative) can be fatal, whereas sending a healthy patient for follow-up testing (False Positive) is acceptable.

### ROC Curve & Precision-Recall Curve

**Receiver Operating Characteristic (ROC) Curve** — Plots **True Positive Rate (TPR)** vs. **False Positive Rate (FPR)** across all decision thresholds.

![ROC Curve](roc_curve.png)

- **ROC-AUC (Area Under Curve):** Ranges from 0.5 (random guessing) to 1.0 (perfect classification).
- **When to use ROC:** Balanced datasets where both positive and negative classes matter equally.

**Precision-Recall Curve** — plots Precision vs. Recall across thresholds.

![Precision-Recall Curve](pr_curve.png)

- **When to use PR Curve:** Highly imbalanced datasets (e.g., 0.1% fraud rate), as ROC-AUC can present an overly optimistic picture due to a large True Negative count suppressing FPR.

```python
from sklearn.metrics import roc_curve, roc_auc_score, precision_recall_curve, auc

fpr, tpr, thresholds = roc_curve(y_true, y_pred_proba)
roc_auc = roc_auc_score(y_true, y_pred_proba)

precision, recall, thresholds = precision_recall_curve(y_true, y_pred_proba)
pr_auc = auc(recall, precision)
```

---

## 3. Multi-Class Logistic Regression

Standard Logistic Regression is a binary classifier ($y \in \{0, 1\}$). To handle $K > 2$ classes, we use multi-class extensions.

### Strategies for Multi-Class Classification

**1. One-vs-Rest (OvR / One-vs-All):**
- Trains $K$ distinct binary logistic regression classifiers.
- Classifier $k$ trains to separate Class $k$ from all remaining $K-1$ classes.
- Final prediction: $\hat{y} = \arg\max_{k}(P_k)$.

**2. One-vs-One (OvO):**
- Trains $\frac{K(K-1)}{2}$ binary classifiers for every pair of classes.
- Final prediction determined by majority voting across all pair classifiers.

| | One-vs-Rest | One-vs-One |
|---|---|---|
| **Number of classifiers** | $K$ | $K(K-1)/2$ |
| **Training cost** | Lower (fewer models) | Higher (many more models for large $K$) |
| **Data per classifier** | Uses full dataset each time (imbalanced: 1 class vs rest) | Uses only the two relevant classes' data each time (more balanced) |
| **Best for** | Large number of classes, faster training needed | Smaller number of classes, or when classes are easier to separate pairwise |

### Softmax Function (Multinomial Logistic Regression)

Rather than running multiple binary models, **Multinomial Logistic Regression** computes probabilities for all $K$ classes simultaneously using the **Softmax Function**.

**Linear Output (Logits):** For each class $k \in \{1, 2, \dots, K\}$:
$$z_k = w_k^T x + b_k$$

**Softmax Formula:** Converts raw real-valued scores ($z_k$) into normalized probabilities that sum to 1:
$$P(y = k \mid x) = \frac{e^{z_k}}{\sum_{j=1}^{K} e^{z_j}}$$

**Intuition:** Softmax is a generalization of the sigmoid function to more than 2 classes — instead of squashing one value into (0,1), it takes a vector of raw scores and turns them into a probability distribution across all classes that sums to exactly 1.

**Shift-invariance property (numerical stability trick):**
$$\text{Softmax}(z) = \text{Softmax}(z + c)$$
Adding the *same* scalar constant $c$ to every logit does **not** change the output probabilities — the softmax function is invariant to a uniform shift. This is more than a mathematical curiosity: it's the standard trick used in practice to avoid numerical overflow. Since softmax involves $e^{z_k}$, very large logits can cause `exp()` to overflow to `inf`. The fix is to subtract $\max(z)$ from every logit before exponentiating:
$$\text{Softmax}(z)_k = \frac{e^{z_k - \max(z)}}{\sum_j e^{z_j - \max(z)}}$$
This produces mathematically identical probabilities while keeping all exponents ≤ 0, which is numerically safe. Most frameworks (PyTorch, TensorFlow, sklearn) implement this internally by default.

```python
from sklearn.linear_model import LogisticRegression

# Softmax / multinomial approach (computes all class probabilities jointly)
model_softmax = LogisticRegression(multi_class='multinomial', solver='lbfgs')

# One-vs-Rest approach — pair with 'liblinear' explicitly to avoid
# deprecation warnings / inconsistent behavior across sklearn versions
model_ovr = LogisticRegression(multi_class='ovr', solver='liblinear')

model_softmax.fit(X_train, y_train)
probabilities = model_softmax.predict_proba(X_test)  # returns probability per class
```

### Cross-Entropy Loss for Multi-Class

The loss function for Softmax Regression (Categorical Cross-Entropy) is:

$$\mathcal{L}(\theta) = -\frac{1}{n} \sum_{i=1}^{n} \sum_{k=1}^{K} y_{i,k} \ln(\hat{y}_{i,k})$$

Where $y_{i,k}$ is an indicator (1 if sample $i$ belongs to class $k$, 0 otherwise) from one-hot encoded ground truth vectors.

**Intuition:** Since $y_{i,k}$ is 0 for all classes except the true class, this simplifies to just $-\ln(\hat{y}_{i,\text{true class}})$ for each sample — the loss only cares about the predicted probability assigned to the *correct* class. The closer that probability is to 1, the closer the loss is to 0.

### Multi-Class Metric Aggregation: Micro vs Macro vs Weighted

When you compute Precision, Recall, or F1 for a multi-class problem, you first get **one score per class**, then need to combine them into a single number. There are three standard ways to do this, and they can tell very different stories:

| Averaging | How it's computed | Focus | When to use |
|---|---|---|---|
| **Micro** | Pool all TP, FP, FN across *all* classes globally, then compute one overall Precision/Recall/F1 | Every individual **instance/prediction** counts equally | When classes are roughly balanced, or you care about overall correctness regardless of which class it came from |
| **Macro** | Compute the metric independently for each class, then take the simple (unweighted) average | Every **class** counts equally, regardless of how many samples it has | When you specifically care about performance on **rare/minority classes** — macro won't let a large class hide a poorly-performing small one |
| **Weighted** | Compute the metric per class, then average weighted by each class's support (number of true instances) | A blend — rewards overall correctness, but adjusted for class imbalance | When you want a single "fair" summary number for an imbalanced dataset without letting rare classes dominate the average as much as macro would |

**Example:** In a 3-class problem where Class A has 950 samples and Classes B & C have 25 each — if the model is great at A but terrible at B and C:
- **Micro F1** will look high (dominated by the large Class A).
- **Macro F1** will look much lower (B and C's poor scores are weighted equally with A).
- **Weighted F1** sits in between, leaning toward Class A's performance since it has more support.

> **Interview tip:** If asked "which do you report?" — the honest answer is "it depends on the goal": use **macro** when detecting rare classes matters (e.g., rare disease diagnosis, fraud types), use **micro/weighted** when overall throughput correctness matters more.

```python
from sklearn.metrics import f1_score, classification_report

f1_micro = f1_score(y_true, y_pred, average='micro')
f1_macro = f1_score(y_true, y_pred, average='macro')
f1_weighted = f1_score(y_true, y_pred, average='weighted')

# Or get everything at once, per class + aggregated
print(classification_report(y_true, y_pred, zero_division=0))
```

---

## 4. Real Industry Interview Questions

### Regression Metrics
1. What does R² actually measure? Can you explain it without using the formula?
2. Can R² be negative? Under what circumstances?
3. Why does R² increase even when you add irrelevant features? How does Adjusted R² fix this?
4. If you add 5 new features and Adjusted R² decreases, what does that tell you?
5. What's the difference between RSS, TSS, and ESS?
6. Why would you choose RMSE over MAE, or vice versa, for a business reporting use case?
7. Your model has a very low MSE but a low R² too — is that possible? Explain.

### Classification Metrics
8. Walk me through a confusion matrix and define TP, TN, FP, FN with a real example.
9. Why is accuracy a poor metric for imbalanced datasets? Give an example.
10. What's the difference between Precision and Recall? Give a business scenario for each.
11. Why is F1-score the harmonic mean and not the simple average of Precision and Recall?
12. In a medical diagnosis model, would you optimize for Precision or Recall? Why?
13. In a spam detection model, would you optimize for Precision or Recall? Why?
14. What is Specificity, and how is it different from Recall?
15. Explain the ROC curve and what ROC-AUC represents.
16. When would you prefer a Precision-Recall curve over an ROC curve?
17. Your model has a ROC-AUC of 0.95 but performs poorly in production on a highly imbalanced dataset — what might be going on?
18. How does changing the classification threshold affect Precision and Recall?

### Multi-Class Classification
19. How does Logistic Regression handle more than 2 classes?
20. What's the difference between One-vs-Rest and One-vs-One? When would you choose one over the other?
21. Explain the Softmax function and how it relates to the Sigmoid function.
22. What is Categorical Cross-Entropy loss, and how is it different from binary log loss?
23. If you have 10 classes, how many classifiers does One-vs-One need to train?
24. Can you use ROC-AUC for multi-class problems? How would you adapt it?

### Applied / Scenario-Based
25. You're evaluating a credit card fraud model with 0.5% fraud rate. Which metrics would you report to stakeholders, and why not just accuracy?
26. A stakeholder asks "what's a good R² value?" — how do you respond, and does it depend on the domain?
27. You're comparing two classification models: one has higher Precision, the other has higher Recall. How do you decide which to deploy?
28. Explain how you would choose the optimal decision threshold for a binary classifier in a business context.
29. You've been asked to evaluate a multi-class product-category classifier. Which metrics would you compute per class, and which would you aggregate — and how (macro vs micro vs weighted average)?
