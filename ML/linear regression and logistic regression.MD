# Regression — Study Notes

## Table of Contents
1. [Simple Linear Regression](#1-simple-linear-regression)
   - [The Equation](#the-equation)
   - [Error / Residuals](#error--residuals)
   - [Why Squared Error, not Absolute Difference?](#why-squared-error-not-absolute-difference)
2. [Optimization Methods & Cost Function](#2-optimization-methods--cost-function)
   - [Ordinary Least Squares (OLS)](#ordinary-least-squares-ols)
   - [Cost Function](#cost-function)
   - [MSE vs MAE](#mse-vs-mae)
3. [Gradient Descent](#3-gradient-descent)
   - [How it Works](#how-it-works)
   - [Types of Gradient Descent](#types-of-gradient-descent)
   - [Edge Cases: Vanishing & Exploding Gradients](#edge-cases-vanishing--exploding-gradients)
4. [Multiple Linear Regression (MLR)](#4-multiple-linear-regression-mlr)
   - [Interpreting Coefficients & Intercept](#interpreting-coefficients--intercept)
   - [Feature Importance](#feature-importance)
5. [Polynomial Regression](#5-polynomial-regression)
6. [Assumptions of Linear Regression](#6-assumptions-of-linear-regression)
7. [Correlation, Collinearity & Multicollinearity](#7-correlation-collinearity--multicollinearity)
   - [Detecting Multicollinearity (VIF)](#detecting-multicollinearity-vif)
   - [Solutions to Multicollinearity](#solutions-to-multicollinearity)
8. [Regularization](#8-regularization)
   - [Why We Penalize](#why-we-penalize)
   - [Ridge Regression (L2)](#ridge-regression-l2)
   - [Lasso Regression (L1)](#lasso-regression-l1)
   - [Elastic Net](#elastic-net)
   - [Comparison Table](#comparison-table)
9. [Logistic Regression](#9-logistic-regression)
   - [Why Not Use Linear Regression for Classification?](#why-not-use-linear-regression-for-classification)
   - [The Sigmoid Function](#the-sigmoid-function)
   - [Why is it Called "Regression" if it Classifies?](#why-is-it-called-regression-if-it-classifies)
   - [Convex vs Non-Convex Cost Functions](#convex-vs-non-convex-cost-functions)
   - [Log Loss / Cross-Entropy](#log-loss--cross-entropy)
   - [Gradient Descent for Logistic Regression](#gradient-descent-for-logistic-regression)
   - [Regularized Logistic Regression](#regularized-logistic-regression)
10. [Real Industry Interview Questions](#10-real-industry-interview-questions)

---

## 1. Simple Linear Regression

Simple Linear Regression models the relationship between **one independent variable (X)** and **one dependent variable (Y)** by fitting a straight line through the data.

### The Equation

You'll see this written in a few equivalent notations across textbooks:

$$Y = mX + c$$
$$Y = \beta_1 X + \beta_0$$
$$Y = \theta_1 X + \theta_0$$

All three mean the same thing:
- $m$ / $\beta_1$ / $\theta_1$ → the **slope (coefficient)** — how much Y changes for a one-unit change in X.
- $c$ / $\beta_0$ / $\theta_0$ → the **intercept** — the value of Y when X = 0.

**Example:** Predicting salary (Y) based on years of experience (X):
$$\text{Salary} = 8000 \times \text{Experience} + 25000$$
Here, every additional year of experience adds ₹8,000 to salary, and a fresh graduate (0 years) starts around ₹25,000.

### Error / Residuals

There are infinitely many possible lines you could draw through a scatter of points — but only **one "best fit" line** minimizes the total error. This error (also called a **residual**) is the difference between the actual value and the predicted value:

$$\text{Error} = y_i - \hat{y}_i$$

The goal of training a linear regression model is to find the **optimal parameters** ($\theta_0$, $\theta_1$) that minimize this total error across all data points.



### Why Squared Error, not Absolute Difference?

> **Interview-refined Q: Why do we square the error instead of just taking the raw difference (or even just summing the differences)?**

**A:** Two reasons:

1. **Cancellation problem:** If we simply summed `(actual − predicted)` across all points, positive and negative errors would **cancel each other out**, making a genuinely bad model look like it has near-zero total error. For example, an error of `+5` and an error of `-5` would sum to 0, hiding the fact that the model is off by 5 units on each prediction.

2. **Squaring fixes cancellation, but the deeper reason we use squaring (not absolute value) is mathematical convenience + penalty behavior:**
   - Squaring makes all errors positive without needing a conditional (`abs()`), and produces a **smooth, differentiable function** — critical because we need to take derivatives to run gradient descent. `|x|` has a sharp corner at 0 and isn't differentiable there, which complicates optimization.
   - Squaring **penalizes larger errors disproportionately more** than smaller ones (an error of 10 contributes 100, not just 10), which is often desirable — we want to punish big mistakes harder than small ones.

---

## 2. Optimization Methods & Cost Function

To find the best-fit parameters, we need an **optimization method** — a systematic way to search for the parameter values that minimize error. Common methods:

| Method | Description |
|---|---|
| **Ordinary Least Squares (OLS)** | A closed-form (direct, analytical) solution that computes optimal parameters using matrix algebra — no iteration needed. Fast for small/medium datasets. |
| **Gradient Descent** | An iterative optimization method that adjusts parameters step-by-step in the direction that reduces error. Needed when OLS is computationally infeasible (very large datasets/many features). |
| **Mean Squared Error (MSE) minimization** | The actual objective being minimized — both OLS and Gradient Descent are just different *methods* to minimize this same cost function. |

### Ordinary Least Squares (OLS)
OLS directly solves for the coefficients that minimize the sum of squared residuals using a closed-form formula:
$$\theta = (X^T X)^{-1} X^T y$$
This gives the exact optimal solution in one step, but becomes computationally expensive as the number of features grows (matrix inversion is costly).

### Cost Function
The **cost function** (also called the loss function when referring to a single data point) is the formula we're trying to minimize during training. For linear regression, it's typically the **Mean Squared Error**:

$$J(\theta) = \frac{1}{n}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2$$

Training a model = finding the $\theta$ values that **minimize $J(\theta)$**.

### MSE vs MAE

**Mean Squared Error (MSE):**
$$\text{MSE} = \frac{1}{n}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2$$
- No cancellation of positive/negative errors.
- Heavily penalizes large errors (squaring amplifies them).
- Very sensitive to outliers.
- Differentiable everywhere → ideal for gradient-based optimization.

**Mean Absolute Error (MAE):**
$$\text{MAE} = \frac{1}{n}\sum_{i=1}^{n}|y_i - \hat{y}_i|$$
- Treats all errors linearly (an error of 10 contributes 10, not 100).
- More robust to outliers than MSE.
- Not smoothly differentiable at zero (has a sharp corner), making optimization slightly harder.

| | MSE | MAE |
|---|---|---|
| Sensitivity to outliers | High | Low |
| Penalizes large errors | Heavily | Proportionally |
| Differentiability | Smooth everywhere | Corner at 0 |
| Interpretability | Squared units (harder to interpret) | Same unit as target (easy to interpret) |
| When to use | When large errors are especially costly, and outliers are rare/handled | When your data has many outliers and you want a robust metric |

---

## 3. Gradient Descent

**Gradient Descent** is an iterative optimization algorithm used to find the parameter values that minimize the cost function, by repeatedly moving in the direction opposite to the gradient (slope) of the cost function.

### How it Works

Update rule:
$$\theta_j := \theta_j - \alpha \frac{\partial J(\theta)}{\partial \theta_j}$$

Where:
- $\theta_j$ = the parameter being updated
- $\alpha$ = **learning rate** (step size)
- $\frac{\partial J(\theta)}{\partial \theta_j}$ = gradient (slope) of the cost function w.r.t. $\theta_j$

**Intuition:** Imagine standing on a hilly cost surface (bowl-shaped for linear regression's MSE). The gradient tells you the direction of steepest ascent; you move in the *opposite* direction (steepest descent) to get closer to the minimum, taking small steps sized by the learning rate.

![Gradient Descent convergence](gradient_descent.png)

**Learning rate matters a lot:**
- **Too small** → convergence is extremely slow, may need thousands of iterations.
- **Too large** → the algorithm can overshoot the minimum and **diverge** (bounce around or move further away) instead of converging.

### Types of Gradient Descent

| Type | Description | Pros | Cons |
|---|---|---|---|
| **Batch Gradient Descent** | Uses the **entire dataset** to compute the gradient at each step | Stable, accurate gradient | Slow for large datasets, high memory usage |
| **Stochastic Gradient Descent (SGD)** | Uses **one random sample** at a time to update parameters | Fast, can escape shallow local minima | Noisy updates, doesn't converge smoothly |
| **Mini-Batch Gradient Descent** | Uses a **small batch** (e.g., 32, 64 samples) per update | Balance of speed and stability — most commonly used in practice | Requires tuning batch size |

### Edge Cases: Vanishing & Exploding Gradients

These issues are most prominent in **deep neural networks** (many layers) but the same principles apply conceptually wherever gradient-based optimization is used:

**Vanishing Gradient:**
- As gradients are propagated backward through many layers (or many iterations), they can become **exceedingly small**, approaching zero.
- This causes the parameters in earlier layers to update extremely slowly (or stop updating altogether) — the model effectively stops learning.
- Common with activation functions like sigmoid/tanh, whose derivatives are small (max 0.25 for sigmoid) and shrink further when multiplied across many layers.
- **Fixes:** Use ReLU-family activations, batch normalization, residual/skip connections, careful weight initialization (Xavier/He initialization).

**Exploding Gradient:**
- The opposite problem — gradients grow **exponentially large** as they propagate, causing huge, unstable parameter updates.
- Leads to the model diverging, weights becoming `NaN`, or wildly oscillating loss.
- **Fixes:** Gradient clipping (cap the gradient's max value), lower learning rate, proper weight initialization, batch normalization.

**Gradient "acceleration" (Momentum):**
- To speed up convergence and dampen oscillations, optimizers add **momentum** — accumulating a moving average of past gradients so the update also considers previous direction, helping it push through flat regions (plateaus) and small local bumps faster.
- Formula (simplified): $v_t = \gamma v_{t-1} + \alpha \nabla J(\theta)$, then $\theta := \theta - v_t$
- This is the foundation for optimizers like **Momentum, RMSProp, and Adam**, which are widely used in practice instead of vanilla gradient descent.

---

## 4. Multiple Linear Regression (MLR)

**Multiple Linear Regression** extends simple linear regression to use **multiple independent variables** to predict a single dependent variable:

$$Y = \theta_0 + \theta_1 X_1 + \theta_2 X_2 + \dots + \theta_p X_p$$

> **Notation note:** From here on, $p$ denotes the **number of features/predictors**, while $n$ (used earlier in MSE/cost function) denotes the **number of samples/observations**. Keeping these separate matters — several textbooks sloppily reuse $n$ for both, which gets confusing once you reach the regularization sums below.

**Example:** Predicting house price using multiple features:
$$\text{Price} = 50000 + (200 \times \text{SqFt}) + (10000 \times \text{Bedrooms}) - (5000 \times \text{Age})$$

### Interpreting Coefficients & Intercept

- **Intercept ($\theta_0$):** The predicted value of Y when **all** independent variables are zero. (Sometimes not practically meaningful — e.g., a house with 0 sq ft doesn't exist — but it's mathematically necessary to anchor the line/plane.)
- **Coefficient ($\theta_i$):** The expected change in Y for a **one-unit increase** in $X_i$, **holding all other variables constant** ("all else equal"). In the example above, each additional square foot adds ₹200 to the predicted price, holding bedrooms and age fixed.
- **Sign of coefficient:** Positive → that feature increases Y; Negative → that feature decreases Y (like "Age" reducing price above).

### Feature Importance

In MLR, the **magnitude** of a coefficient can hint at feature importance — but only after features are on comparable scales (since raw coefficient sizes are affected by the unit/scale of each feature). Generally: **higher (absolute) standardized coefficient → more influence on the prediction**, all else equal.

> **Interview tip:** Never compare raw (unscaled) coefficients directly to judge importance — a feature measured in "thousands" will naturally have a smaller coefficient than one measured in "single units," even if it's more important. Standardize features first (Z-score) before comparing coefficient magnitudes.

---

## 5. Polynomial Regression

When the relationship between X and Y is **not linear** (i.e., a straight line underfits the data), we can add polynomial terms of X to capture curvature:

$$Y = \theta_0 + \theta_1 X + \theta_2 X^2 + \theta_3 X^3 + \dots + \theta_d X^d$$

*(Here $d$ is the degree of the polynomial — again, a different quantity from $n$ the sample count or $p$ the feature count.)*

**Multiple Polynomial Regression** extends this further to multiple variables with polynomial and interaction terms:
$$Y = \theta_0 + \theta_1 X_1 + \theta_2 X_2 + \theta_3 X_1^2 + \theta_4 X_2^2 + \theta_5 X_1 X_2$$

**Example:** Modeling the relationship between speed and fuel efficiency, which typically rises then falls (a curve) rather than following a straight line — a degree-2 polynomial captures this better than a straight line.

**Important nuance for interviews:** Despite fitting a curve, polynomial regression is still considered a form of **linear regression** — because it's linear *in the parameters* ($\theta$), even though it's non-linear in X. This is a very common interview trick question.

**Caution:** Higher-degree polynomials fit training data increasingly well but risk severe **overfitting** — the curve starts chasing noise instead of the true trend (ties directly back to the bias-variance tradeoff).

---

## 6. Assumptions of Linear Regression

For a linear regression model's coefficients and inference (p-values, confidence intervals) to be valid, several assumptions should approximately hold:

1. **Linearity** — the relationship between independent variables and the target is linear.
2. **Independence of errors** — residuals should not be correlated with each other (important for time-series data, checked via Durbin-Watson test).
3. **Homoscedasticity** — the variance of residuals should be constant across all levels of X (no "funnel" shape when plotting residuals vs predicted values).
4. **Normality of residuals** — residuals should be approximately normally distributed (checked via Q-Q plot).
5. **No (or low) multicollinearity** — independent variables shouldn't be highly correlated with each other (covered in detail next).
6. **No significant outliers/high-leverage points** — since OLS is very sensitive to them.

---

## 7. Correlation, Collinearity & Multicollinearity

These three terms are often confused — here's the precise distinction:

- **Correlation:** A statistical measure (ranging from -1 to +1) of the **linear relationship strength** between any two variables (could be feature-feature, or feature-target).
- **Collinearity:** A special case where **two independent variables** are highly correlated with each other.
- **Multicollinearity:** When **three or more independent variables** are correlated with each other (or a variable can be linearly predicted from a combination of others) — a broader, more general version of collinearity.

**Why it's a problem:** When independent variables are highly correlated, the model can't tell which variable is actually driving the change in Y — the coefficient estimates become unstable, their signs can flip unexpectedly, and standard errors inflate, making the coefficients unreliable/hard to interpret (even though overall prediction accuracy may not suffer much).

**Example:** Predicting salary using both "Years of Experience" and "Age" — these two are usually highly correlated, so the model struggles to isolate the individual effect of each.

### Detecting Multicollinearity (VIF)

**Variance Inflation Factor (VIF)** quantifies how much a coefficient's variance is inflated due to correlation with other predictors:

$$VIF_i = \frac{1}{1 - R_i^2}$$

Where $R_i^2$ is the R² obtained by regressing the $i^{th}$ independent variable against all other independent variables.

**Rule of thumb:**
- VIF = 1 → No correlation with other features.
- VIF between 1–5 → Moderate correlation, generally acceptable.
- VIF > 5 (some use >10) → High multicollinearity, should be addressed.

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
import pandas as pd

vif_data = pd.DataFrame()
vif_data["feature"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i) for i in range(X.shape[1])]
```

### Solutions to Multicollinearity

| Solution | How it helps |
|---|---|
| **Drop one of the correlated variables** | Simplest fix — remove the redundant feature |
| **Combine correlated features** | E.g., create a single ratio/index feature instead of keeping both |
| **PCA (Principal Component Analysis)** | Transforms correlated features into a smaller set of **uncorrelated** principal components |
| **Ridge Regression (L2 regularization)** | Shrinks correlated coefficients together, stabilizing estimates without dropping features |
| **Lasso Regression (L1 regularization)** | Can shrink some correlated coefficients to exactly zero, effectively performing feature selection |
| **Increase sample size** | More data can sometimes reduce the instability caused by multicollinearity |

---

## 8. Regularization

### Why We Penalize

As models become more complex (more features, higher-degree polynomials), they risk **overfitting** — fitting noise in the training data rather than the true underlying pattern (tying back to the bias-variance tradeoff). 

**Regularization** combats this by **adding a penalty term to the cost function** that discourages large coefficient values. This forces the model to keep coefficients small/simple unless they meaningfully reduce error, effectively trading a small increase in bias for a large reduction in variance.

### Ridge Regression (L2)

Adds the **sum of squared coefficients** as a penalty:

$$J(\theta) = \text{MSE} + \lambda \sum_{j=1}^{p}\theta_j^2$$

- $\lambda$ (also called $\alpha$ in sklearn) controls the strength of the penalty — higher $\lambda$ → more shrinkage.
- Shrinks coefficients **toward zero but rarely exactly to zero**.
- Handles multicollinearity well by distributing weight among correlated features rather than picking one.

### Lasso Regression (L1)

Adds the **sum of absolute coefficients** as a penalty:

$$J(\theta) = \text{MSE} + \lambda \sum_{j=1}^{p}|\theta_j|$$

- Can shrink some coefficients to **exactly zero**, effectively performing automatic **feature selection**.
- Useful when you suspect many features are irrelevant.

**Why does L1 produce exact zeros but L2 doesn't?** Geometrically, the L1 constraint region is a **diamond** (has sharp corners), and the optimal solution (where the elliptical cost contours touch the constraint region) often lands exactly on a corner — where one coefficient is zero. The L2 constraint region is a **smooth circle**, so the optimal touching point rarely lands exactly on an axis.

![L1 vs L2 regularization geometry](l1_vs_l2.png)

### Elastic Net

Combines both L1 and L2 penalties:

$$J(\theta) = \text{MSE} + \lambda_1 \sum_{j=1}^{p}|\theta_j| + \lambda_2 \sum_{j=1}^{p}\theta_j^2$$

- Gets the benefits of both: some automatic feature selection (from L1) plus the stability of coefficient shrinkage for correlated features (from L2).
- Has two hyperparameters to tune ($\lambda_1$, $\lambda_2$, or equivalently `alpha` and `l1_ratio` in sklearn).

### Comparison Table

| | Ridge (L2) | Lasso (L1) | Elastic Net |
|---|---|---|---|
| **Penalty** | Sum of squared coefficients | Sum of absolute coefficients | Combination of both |
| **Feature selection** | No (keeps all features, shrinks them) | Yes (can zero out coefficients) | Yes (partial) |
| **Best for** | Many small/medium correlated features | Few features truly matter, want sparsity | High-dimensional data with correlated groups of features |
| **Advantage** | Handles multicollinearity gracefully | Simplifies model, improves interpretability | Balances both worlds |
| **Disadvantage** | Doesn't simplify the model (all features retained) | Can arbitrarily pick one feature among correlated ones and zero the rest | More hyperparameters to tune |
| **When to use** | Multicollinearity present, all features believed relevant | Suspect many irrelevant/redundant features | Both correlation and irrelevant features are present |

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet

ridge = Ridge(alpha=1.0)
lasso = Lasso(alpha=1.0)
elastic = ElasticNet(alpha=1.0, l1_ratio=0.5)  # l1_ratio: mix of L1 vs L2
```

---

## 9. Logistic Regression

### Why Not Use Linear Regression for Classification?

For a binary classification problem (e.g., spam vs. not spam), you might think of just fitting a straight regression line and applying a threshold. This fails for several reasons:

1. Linear regression can predict values **outside the [0,1] range** (e.g., -0.5 or 1.8), which makes no sense as a probability.
2. A straight line is heavily influenced by outliers — a few extreme points can shift the decision threshold significantly, misclassifying many points.
3. Linear regression assumes a constant linear relationship, but the relationship between X and the **probability** of belonging to a class is typically **S-shaped**, not a straight line.

So instead of predicting Y directly, **Logistic Regression predicts the probability that Y belongs to a class**, using a function that's bounded between 0 and 1.

### The Sigmoid Function

The **sigmoid (logistic) function** squashes any real-valued number into the range (0, 1):

$$\sigma(z) = \frac{1}{1 + e^{-z}}, \quad \text{where } z = \theta_0 + \theta_1 X_1 + \dots + \theta_p X_p$$

![Sigmoid function](sigmoid.png)

**How classification happens:**
1. Compute the linear combination $z = \theta^T X$ (same as linear regression).
2. Pass $z$ through the sigmoid function to get a probability $\sigma(z)$ between 0 and 1.
3. Apply a **threshold** (commonly 0.5): if $\sigma(z) \geq 0.5$ → class 1, else → class 0.

**Example:** Predicting whether a customer will churn (1) or not (0) based on usage patterns. The model outputs a probability like 0.82 → since it's ≥ 0.5, predict "will churn."

### Why is it Called "Regression" if it Classifies?

> **Interview Q: Logistic Regression is used for classification — so why is "Regression" in the name?**

**A:** Because under the hood, it's still fundamentally **regressing/estimating a continuous quantity** — specifically, the **probability** (a continuous value between 0 and 1) that an observation belongs to a class, using a linear combination of inputs passed through the sigmoid function. The *classification* happens only as a final extra step (thresholding that continuous probability). The core mechanism — fitting coefficients to a linear function of the inputs — is a regression technique, historically named before the classification use case was emphasized.

### Convex vs Non-Convex Cost Functions

If we naively used the same MSE cost function from linear regression but plugged in the sigmoid-based predictions, the resulting cost function becomes **non-convex** — full of many local minima — because the sigmoid function introduces non-linearity into the cost surface.

![Convex vs Non-convex functions](convex_vs_nonconvex.png)

**Why this matters:** Gradient Descent is only guaranteed to find the **global minimum** on a convex function (single bowl shape). On a non-convex function, gradient descent can get stuck in a **local minimum**, producing a suboptimal model.

**The solution:** Instead of MSE, logistic regression uses a specially designed cost function — **Log Loss (Cross-Entropy Loss)** — which, when combined with the sigmoid function, produces a **convex** cost surface, guaranteeing gradient descent converges to the global minimum.

### Log Loss / Cross-Entropy

For a single training example:
$$\text{Cost}(\hat{y}, y) = -\big[y \log(\hat{y}) + (1-y)\log(1-\hat{y})\big]$$

For the full dataset (averaged over $n$ examples):
$$J(\theta) = -\frac{1}{n}\sum_{i=1}^{n}\big[y_i \log(\hat{y}_i) + (1-y_i)\log(1-\hat{y}_i)\big]$$

**Intuition:**
- If the true label $y=1$ and the model predicts $\hat{y}$ close to 1 → $\log(\hat{y})$ is close to 0 → **low cost** (good).
- If $y=1$ but the model predicts $\hat{y}$ close to 0 (confidently wrong) → $\log(\hat{y})$ approaches $-\infty$ → **very high cost** (heavily penalized).
- This asymmetric, heavy penalty for confident-but-wrong predictions is exactly what makes the function convex and well-suited for probability-based classification.

### Gradient Descent for Logistic Regression

Despite the different cost function, the parameter update rule looks remarkably similar to linear regression:

$$\theta_j := \theta_j - \alpha \frac{\partial J(\theta)}{\partial \theta_j} = \theta_j - \alpha \frac{1}{n}\sum_{i=1}^{n}(\hat{y}_i - y_i)x_{ij}$$

This is iterated until the cost function converges to a minimum (or a set number of iterations is reached).

### Regularized Logistic Regression

Just like linear regression, logistic regression can overfit (especially with many features), so the same regularization techniques apply — added directly to the log loss cost function:

**L2 (Ridge) regularized logistic regression:**
$$J(\theta) = -\frac{1}{n}\sum_{i=1}^{n}\big[y_i \log(\hat{y}_i) + (1-y_i)\log(1-\hat{y}_i)\big] + \lambda\sum_{j=1}^{p}\theta_j^2$$

*(Here $n$ = number of samples in the outer sum, $p$ = number of features in the penalty sum — not the same quantity.)*

**L1 (Lasso) regularized logistic regression:** same idea, but with $\sum|\theta_j|$ instead — useful for automatic feature selection in high-dimensional classification problems.

```python
from sklearn.linear_model import LogisticRegression

# L2 regularization (default in sklearn)
model = LogisticRegression(penalty='l2', C=1.0)  # C = 1/λ (inverse of regularization strength)

# L1 regularization
model_l1 = LogisticRegression(penalty='l1', solver='liblinear', C=1.0)

model.fit(X_train, y_train)
```

---

## 10. Real Industry Interview Questions

### Linear Regression Fundamentals
1. Explain simple linear regression and its equation. What do the coefficient and intercept represent?
2. Why do we square the error instead of using absolute error in OLS-based linear regression?
3. What's the difference between MSE and MAE? When would you prefer one over the other?
4. Walk me through how Ordinary Least Squares finds the optimal coefficients.
5. What is a cost function, and how is it different from a loss function?

### Gradient Descent
6. Explain gradient descent in your own words. What role does the learning rate play?
7. What happens if the learning rate is too high? Too low?
8. What's the difference between Batch, Stochastic, and Mini-Batch Gradient Descent?
9. What are vanishing and exploding gradients? How would you fix each?
10. What is momentum in gradient descent, and why is it useful?

### Multiple/Polynomial Regression
11. How do you interpret a coefficient in multiple linear regression?
12. Why can't you compare raw coefficient magnitudes directly to judge feature importance?
13. Is polynomial regression still a "linear" model? Why or why not?
14. What risk do you run by using a very high-degree polynomial?

### Assumptions & Multicollinearity
15. What are the key assumptions of linear regression?
16. What's the difference between correlation, collinearity, and multicollinearity?
17. How do you detect multicollinearity? What does a VIF of 8 tell you?
18. Your model has high multicollinearity — what are your options to fix it?
19. Does multicollinearity affect a model's predictive accuracy, or just interpretability? Explain.

### Regularization
20. What is regularization, and why do we need it?
21. What's the mathematical difference between L1 and L2 regularization?
22. Why does Lasso produce sparse (zero) coefficients, but Ridge doesn't?
23. When would you choose Elastic Net over plain Lasso or Ridge?
24. How does increasing the regularization parameter (λ) affect bias and variance?

### Logistic Regression
25. Why can't you use linear regression directly for a classification problem?
26. Explain the sigmoid function and why it's used in logistic regression.
27. Logistic Regression is used for classification — why does it have "regression" in its name?
28. Why can't we use MSE as the cost function for logistic regression?
29. What's the difference between a convex and non-convex cost function, and why does it matter for optimization?
30. Derive or explain the log loss (cross-entropy) formula and its intuition.
31. How would you handle a multi-class classification problem using logistic regression? (Hint: One-vs-Rest / Softmax)
32. What does the regularization parameter `C` mean in sklearn's LogisticRegression, and how is it related to λ?

### Applied / Scenario-Based
33. Your linear regression model has a very high R² on training data but performs poorly on test data. What's happening, and how do you fix it?
34. You suspect two features are highly correlated. How would you confirm this, and what would you do about it?
35. You're building a churn prediction model and want interpretable coefficients as well as automatic feature selection — which regression variant would you pick, and why?
36. Walk me through how you'd debug a logistic regression model that outputs probabilities close to 0.5 for almost every input.
