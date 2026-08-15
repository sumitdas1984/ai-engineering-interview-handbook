# Traditional ML Interview Brush-Up: Tree-Based Classification & Ensemble Methods

## 1. Decision Tree Classification

### What is a Decision Tree?

A Decision Tree is a supervised learning algorithm that makes predictions by asking a sequence of **if-else questions** about the features.

Example:

```text
             Age > 30?
             /       \
           Yes        No
           /           \
    Income > 50K?      No
      /      \
    Yes       No
    /          \
  Buy         No Buy
```

### Key concepts

1. **Root node**
   - The first question/split in the tree.
   - Example: `Age > 30?`

2. **Internal node**
   - A decision/question inside the tree.
   - Example: `Income > 50K?`

3. **Leaf node**
   - The final prediction.
   - Example: `Buy` or `No Buy`.

4. **Splitting**
   - The algorithm selects a feature and threshold that best separates the classes.

5. Common measures for classification splits:
   - **Gini Impurity**
   - **Entropy / Information Gain**

### Decision Tree Overfitting

A decision tree **overfits when it becomes too complex and starts learning the noise/details of the training data instead of the general pattern**.

Example:

```text
Training accuracy   = 99%
Validation accuracy = 75%
```

A common reason is that the tree grows too deep, creating many small branches and leaves.

### How to prevent overfitting

There are two main approaches:

#### A. Pre-pruning

Stop the tree from becoming too complex **while it is being built**.

Common parameters:

- `max_depth`
- `min_samples_split`
- `min_samples_leaf`
- `max_leaf_nodes`
- `min_impurity_decrease`

#### B. Post-pruning

First grow a large tree, then remove branches that do not contribute enough to generalization.

One common approach is **Cost Complexity Pruning**.

### Pre-pruning vs Post-pruning

| | Pre-pruning | Post-pruning |
|---|---|---|
| When? | During tree construction | After tree construction |
| Approach | Stop growth early | Grow first, then remove branches |
| Example | `max_depth=5` | Cost-complexity pruning |
| Advantage | Faster, simpler | Can find a better final tree |
| Risk | May stop too early | Initially creates a large tree |

### Important Decision Tree parameters for overfitting

#### `max_depth`

Controls the maximum depth of the tree.

- Lower `max_depth` → simpler tree → less overfitting
- Higher `max_depth` → more complex tree → higher overfitting risk

#### `min_samples_split`

Minimum number of samples required to split an internal node.

Higher value → fewer splits → simpler tree.

#### `min_samples_leaf`

Minimum number of samples allowed in a leaf.

Higher value → prevents very small leaves → simpler tree.

#### `max_leaf_nodes`

Limits the maximum number of leaf nodes.

Fewer leaves → simpler tree.

#### `max_features`

Controls how many features are considered when looking for a split.

This becomes particularly important in Random Forest.

### Bias-variance mental model

- Deep tree → high variance → overfitting
- Very shallow tree → potentially high bias → underfitting

### Interview answer

> A decision tree is a supervised learning algorithm that recursively splits the dataset based on feature values. For classification, it chooses splits that improve class purity, commonly using Gini impurity or entropy. The final leaf node gives the class prediction. A major issue is overfitting, especially with deep trees, which can be controlled using parameters like max_depth and minimum samples per leaf.

---

# 2. Ensemble Classification Techniques

## What is Ensemble Learning?

Ensemble learning means **combining multiple models to produce a better prediction than a single model**.

Conceptually:

```text
Model 1 ──┐
Model 2 ──┤
Model 3 ──┼──→ Combined Prediction
Model 4 ──┤
Model 5 ──┘
```

### Why use ensembles?

Main goals:

- Improve accuracy
- Improve generalization
- Improve stability
- Reduce overfitting

## Two important ensemble approaches

### 1. Bagging

Train multiple models **independently/in parallel** on different samples of the training data.

Main effect:

> **Reduce variance**

Example:

```text
Training Data
     ↓
 ┌───┼────┬────┐
 ↓   ↓    ↓    ↓
Tree Tree Tree Tree
 └───┴────┴────┘
       ↓
 Combine predictions
```

### 2. Boosting

Train models **sequentially**, where each new model tries to improve upon the mistakes of previous models.

Main effect:

> **Reduce bias / improve weak learners**

Example:

```text
Tree 1
  ↓
Focus on mistakes
  ↓
Tree 2
  ↓
Focus on remaining mistakes
  ↓
Tree 3
  ↓
Final prediction
```

Examples:

- AdaBoost
- Gradient Boosting
- XGBoost

### Key distinction

> **Bagging:** models work independently → primarily reduce variance.

> **Boosting:** models work sequentially → primarily reduce bias and improve weak learners.

---

# 3. Bagging

## What is Bagging?

**Bagging = Bootstrap Aggregating.**

Bagging means training multiple models independently on different **bootstrap samples** of the training data, and then combining their predictions.

Example:

```text
Original Data
     ↓
 ┌───┼────┬────┬────┐
 ↓   ↓    ↓    ↓    ↓
S1  S2    S3   S4   S5
 ↓   ↓    ↓    ↓    ↓
Tree Tree Tree Tree Tree
 └───┴────┴────┴────┘
          ↓
    Combine predictions
```

### Bootstrap sampling

A bootstrap sample is created by **random sampling with replacement**.

Therefore:

- The same training record can appear multiple times.
- Some original records may not appear in a particular bootstrap sample.

### Why use Bagging?

The main purpose is to **reduce variance**.

A single deep decision tree can have high variance and overfit.

Instead:

```text
Tree 1 ──┐
Tree 2 ──┤
Tree 3 ──┼──→ Average / Majority Vote
Tree 4 ──┤
Tree 5 ──┘
```

The individual models may make different mistakes, but combining them makes the overall model more stable.

### Final prediction

For classification:

> **Majority voting**

Example:

```text
Tree 1 → Class A
Tree 2 → Class A
Tree 3 → Class B
Tree 4 → Class A
Tree 5 → Class B

Final → Class A
```

For regression:

> **Average of predictions**

### Important interview point

Bagging models are trained **independently/in parallel**.

This differs from Boosting, where models are trained **sequentially**, with later models focusing on previous errors.

### Interview answer

> Bagging, or Bootstrap Aggregating, is an ensemble technique where multiple models are trained independently on different bootstrap samples of the training data. Their predictions are then combined using majority voting for classification or averaging for regression. The primary benefit is reducing variance and improving model stability.

---

# 4. Random Forest

## What is Random Forest?

Think of:

> **Random Forest = Bagging + Decision Trees + Random Feature Selection**

Instead of building one decision tree, Random Forest builds many decision trees and combines their predictions.

```text
             Training Data
                  ↓
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     Tree 1     Tree 2     Tree 3   ... many trees
       ↓          ↓          ↓
       └──────────┼──────────┘
                  ↓
            Majority Vote
                  ↓
             Prediction
```

## Why is it called "Random" Forest?

There are two sources of randomness:

### 1. Random training samples

Each tree gets a bootstrap sample of the training data.

This comes from **Bagging**.

### 2. Random feature selection

When a tree is deciding which feature to use for a split, it considers only a **random subset of features**, rather than all features.

Example:

```text
20 features available
        ↓
Randomly select 5 features
        ↓
Find the best split among those 5
```

This happens repeatedly at different nodes.

## Why does this help?

Individual decision trees can be highly correlated and can overfit.

Random Forest makes the trees more different from each other.

Then their predictions are combined:

> Many diverse trees → majority voting → more stable prediction.

The main benefit is **reducing variance and overfitting compared with a single decision tree**.

## Random Forest vs Bagging

### Bagging with decision trees

> Different bootstrap samples → different trees → combine predictions.

### Random Forest

> Different bootstrap samples **AND** random subset of features at each split → different trees → combine predictions.

Therefore:

> **Random Forest = Bagging + Random Feature Selection**

### Interview answer

> Random Forest is an ensemble of decision trees. It uses bootstrap sampling to train different trees on different samples of the data, and at each split it considers a random subset of features. The predictions from all trees are then combined using majority voting for classification. This reduces variance and generally gives better generalization than a single decision tree.

---

# 5. Boosting

## What is Boosting?

Boosting is an ensemble technique where **multiple weak models are trained sequentially**, and each new model tries to correct the mistakes made by the previous models.

### Bagging vs Boosting

Bagging:

```text
Tree 1 ──┐
Tree 2 ──┤
Tree 3 ──┤  Independent
Tree 4 ──┘
    ↓
Combine
```

Boosting:

```text
Tree 1
  ↓
Find mistakes
  ↓
Tree 2 → focuses on mistakes
  ↓
Tree 3 → focuses on remaining mistakes
  ↓
...
  ↓
Final prediction
```

The key word is:

> **Sequential**

### Why boosting?

Individual models are usually **weak learners**, often shallow decision trees.

By combining many weak learners sequentially:

> **Weak learners → strong learner**

### Main objective

Boosting primarily tries to **reduce bias** and improve predictive performance.

### Interview answer

> Boosting is an ensemble technique where weak learners are trained sequentially. Each new learner focuses on improving the errors of the previous ensemble. The predictions of all learners are then combined to produce a strong model. Unlike bagging, which primarily reduces variance, boosting primarily helps reduce bias.

---

# 6. AdaBoost

## What is AdaBoost?

**AdaBoost = Adaptive Boosting.**

The easiest way to remember it:

> **Train a weak learner → identify wrongly classified samples → give those samples more importance → train the next learner → repeat.**

### Step 1: Start with equal weights

Initially, all training samples have equal importance.

```text
All samples have equal importance
        ↓
Train Tree 1
```

AdaBoost commonly uses small/shallow decision trees as weak learners.

### Step 2: Identify mistakes

Suppose:

```text
100 samples
    ↓
80 correct
20 incorrect
```

AdaBoost gives higher weight to the incorrectly classified samples.

```text
Incorrect samples → higher weight
Correct samples   → lower relative weight
```

### Step 3: Train the next learner

The next learner focuses more on the difficult/misclassified samples.

```text
Tree 1
  ↓
Find mistakes
  ↓
Increase weight of mistakes
  ↓
Tree 2
  ↓
Find mistakes
  ↓
Increase weight of mistakes
  ↓
Tree 3
```

This is why it is called **Adaptive** Boosting.

### Final prediction

All weak learners contribute to the final prediction, but better-performing learners get more influence.

```text
Tree 1 ──┐
Tree 2 ──┤
Tree 3 ──┼──→ Weighted Voting → Final Class
Tree 4 ──┤
Tree 5 ──┘
```

### AdaBoost vs Random Forest

**Random Forest:**

> Trees are trained independently. Each tree gets a different sample/features. Final prediction = majority vote.

**AdaBoost:**

> Trees are trained sequentially. Each new tree focuses more on the mistakes of previous trees. Final prediction = weighted vote.

### Interview answer

> AdaBoost is a boosting algorithm that combines multiple weak learners sequentially. Initially, all training samples have equal weights. After each learner, incorrectly classified samples receive higher weights, so the next learner focuses more on those difficult examples. Finally, the predictions of all learners are combined using weighted voting.

### Mental model

> **AdaBoost → sample weights → focus on mistakes → sequential weak learners**

---

# 7. Gradient Boosting

## What is Gradient Boosting?

Think of Gradient Boosting as:

> **Build trees sequentially, where each new tree tries to reduce the errors/loss made by the existing ensemble.**

### Basic flow

```text
Training data
     ↓
Initial model
     ↓
Predictions
     ↓
Calculate loss/errors
     ↓
New tree learns to reduce the loss
     ↓
Updated ensemble
     ↓
Calculate remaining loss
     ↓
Another tree
     ↓
...
```

The important idea is:

> We add trees gradually to minimize a loss function.

### Why "Gradient"?

The algorithm uses the **gradient of the loss function** to determine the direction in which the model should improve.

For an interview-level understanding:

> **Gradient = direction to reduce the loss.**

### Gradient Boosting vs AdaBoost

**AdaBoost:**

> Which training samples did I classify incorrectly? → increase their weights.

**Gradient Boosting:**

> What error/loss is remaining? → train the next tree to reduce that loss.

### Important parameters

- `n_estimators` → number of trees
- `learning_rate` → contribution of each tree
- `max_depth` → complexity of each tree

Generally:

> Smaller `learning_rate` → more trees may be needed.

### Interview answer

> Gradient Boosting is an ensemble technique where weak decision trees are trained sequentially. Each new tree is added to the existing ensemble to minimize the loss function, using the gradient of the loss to determine how the model should improve.

### Mental model

> **AdaBoost → focus on misclassified samples**

> **Gradient Boosting → minimize loss using gradients**

---

# 8. XGBoost

## What is XGBoost?

**XGBoost = Extreme Gradient Boosting.**

For interview purposes:

> **XGBoost is a highly optimized and regularized implementation of Gradient Boosting, mainly using decision trees as weak learners.**

### Basic idea

It follows the sequential boosting approach:

```text
Tree 1
  ↓
Calculate remaining error/loss
  ↓
Tree 2 improves it
  ↓
Tree 3 improves further
  ↓
...
  ↓
Final prediction
```

XGBoost is **not bagging**. Its boosting trees are built sequentially.

## What makes XGBoost special?

Remember three things:

### 1. Gradient boosting

Each new tree tries to improve the existing model by minimizing the loss.

### 2. Regularization

XGBoost includes regularization to control model complexity and reduce overfitting.

It can penalize:

- Number of leaves
- Complexity of trees
- Large leaf weights

### 3. Engineering/optimization

XGBoost was designed to be efficient and scalable, with optimizations such as parallelized computation and efficient tree construction.

## Important XGBoost parameters

| Parameter | Meaning |
|---|---|
| `n_estimators` | Number of trees |
| `learning_rate` | Contribution of each tree |
| `max_depth` | Maximum tree depth |
| `subsample` | Fraction of training samples used per tree |
| `colsample_bytree` | Fraction of features used per tree |
| `reg_alpha` | L1 regularization |
| `reg_lambda` | L2 regularization |

For overfitting, particularly remember:

> `max_depth`, `learning_rate`, `subsample`, `colsample_bytree`, `reg_alpha`, `reg_lambda`

## Gradient Boosting vs XGBoost

A good interview answer:

> **Gradient Boosting is the general algorithmic approach. XGBoost is a highly optimized implementation of gradient boosting with additional regularization and engineering improvements.**

---

# 9. Overall Mental Map

Keep this hierarchy in mind:

```text
                 Ensemble
                    │
          ┌─────────┴─────────┐
          │                   │
       Bagging             Boosting
          │                   │
    Random Forest       ┌──────┼──────┐
                        │      │      │
                    AdaBoost  GB   XGBoost
```

## Quick Comparison

| Technique | How models are trained | Main idea | Main benefit |
|---|---|---|---|
| Decision Tree | One tree | Recursive splits | Simple/interpretable |
| Bagging | Independently | Bootstrap samples | Reduce variance |
| Random Forest | Independently | Bootstrap + random features | Reduce variance / improve generalization |
| Boosting | Sequentially | Correct/improve previous model | Reduce bias |
| AdaBoost | Sequentially | Increase weight of mistakes | Focus on difficult samples |
| Gradient Boosting | Sequentially | Minimize loss using gradients | Strong predictive performance |
| XGBoost | Sequentially | Optimized + regularized gradient boosting | Performance + regularization + scalability |

---

# 10. Interview One-Liners

### Decision Tree
> A tree-based supervised model that recursively splits data to make predictions.

### Overfitting in Decision Tree
> A very complex/deep tree can memorize training data and fail to generalize to unseen data.

### Pre-pruning
> Control tree growth during construction using parameters such as `max_depth` and `min_samples_leaf`.

### Post-pruning
> Grow the tree and then remove unnecessary branches, such as through cost-complexity pruning.

### Bagging
> Train models independently on bootstrap samples and combine their predictions to reduce variance.

### Random Forest
> Bagging of decision trees with additional random feature selection at each split.

### Boosting
> Train weak learners sequentially so that each new learner improves the existing ensemble.

### AdaBoost
> Sequentially increase the importance of misclassified samples so subsequent learners focus on them.

### Gradient Boosting
> Sequentially add trees that move the model toward lower loss using gradient information.

### XGBoost
> An optimized and regularized implementation of gradient boosting designed for strong predictive performance and scalability.

---

## Most Important Comparisons to Revise Before the Interview

```text
Decision Tree
     ↓
Can overfit → high variance

Bagging
     ↓
Independent models
     ↓
Reduce variance

Random Forest
     ↓
Bagging + random features
     ↓
Reduce variance

Boosting
     ↓
Sequential models
     ↓
Reduce bias

AdaBoost
     ↓
Increase weights of mistakes

Gradient Boosting
     ↓
Minimize loss using gradients

XGBoost
     ↓
Gradient Boosting
+ Regularization
+ Engineering/optimization
```

## 30-Second Overall Explanation

> "Decision trees are individual models that can have high variance and overfit. Ensemble methods address this by combining multiple models. Bagging trains models independently on bootstrap samples and mainly reduces variance. Random Forest extends bagging by also randomly selecting features at each split. Boosting works sequentially, where each new weak learner improves the existing ensemble. AdaBoost focuses on misclassified samples by increasing their weights, while Gradient Boosting trains new trees to reduce the loss using gradient information. XGBoost is an optimized and regularized implementation of gradient boosting."
