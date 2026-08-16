# Class Imbalance & Anomaly Detection
## Senior AI Engineer Interview Brush-Up

> **Goal:** Build a high-level understanding of two important traditional ML topics frequently encountered in production ML and Senior/Staff AI Engineer interviews.

---

# 1. Class Imbalance — At a Glance

**Class imbalance** occurs when the classes in a classification dataset are distributed very unevenly.

Example:

```text
Fraud Detection

Normal     → 99,000
Fraud      →  1,000

Total      → 100,000
```

Class distribution:

```text
Normal → 99%
Fraud  →  1%
```

The minority class is usually the class we care most about.

Typical examples:

- Fraud detection
- Disease detection
- Spam detection
- Intrusion detection
- Equipment failure
- Defect detection
- Hate-speech detection

### Core problem

A model can achieve high **accuracy** while performing poorly on the minority class.

Example:

```text
1000 transactions
990 normal
10 fraud
```

A model predicting:

```text
Everything → Normal
```

gets:

```text
Accuracy = 99%
```

But:

```text
Fraud detected = 0 / 10
Recall = 0%
```

Therefore:

> **Accuracy can be highly misleading for imbalanced classification.**

---

# 2. The Most Important Mental Model

When classes are imbalanced, think about the problem in this order:

```text
Class Distribution
       ↓
What is the business cost of errors?
       ↓
Choose appropriate metrics
       ↓
Choose mitigation strategy
       ↓
Tune decision threshold
       ↓
Evaluate on realistic validation/test data
```

The important point is:

> There is no universally correct imbalance technique. The right approach depends on the business cost of false positives vs false negatives.

---

# 3. Confusion Matrix

For binary classification:

```text
                         Actual
                    Positive    Negative
                  +-----------+-----------+
Predicted Positive|    TP     |    FP     |
                  +-----------+-----------+
Predicted Negative|    FN     |    TN     |
                  +-----------+-----------+
```

Where:

### TP — True Positive

Model predicts positive and it is actually positive.

### TN — True Negative

Model predicts negative and it is actually negative.

### FP — False Positive

Model predicts positive but it is actually negative.

### FN — False Negative

Model predicts negative but it is actually positive.

---

# 4. Metrics for Imbalanced Classification

## Accuracy

```text
Accuracy = (TP + TN) / Total
```

Problem:

> It can be dominated by the majority class.

For a 99:1 dataset, a useless model can achieve 99% accuracy.

Therefore:

> **Do not rely on accuracy alone for highly imbalanced problems.**

---

# 5. Precision

```text
Precision = TP / (TP + FP)
```

Precision answers:

> "Of everything I predicted as positive, how many were actually positive?"

Example:

```text
100 alerts generated
30 are actual fraud
```

Precision:

```text
30 / 100 = 30%
```

High precision means:

> Fewer false alarms.

---

# 6. Recall

```text
Recall = TP / (TP + FN)
```

Recall answers:

> "Of all actual positive cases, how many did I detect?"

Example:

```text
100 actual fraud cases
80 detected
```

Recall:

```text
80%
```

High recall means:

> We miss fewer positive cases.

---

# 7. Precision vs Recall

This is one of the most important interview concepts.

```text
High Precision
      ↓
Fewer False Positives

High Recall
      ↓
Fewer False Negatives
```

Typical examples:

### Fraud detection

Missing fraud may be expensive.

```text
Prioritize Recall
```

But excessive false alarms also have operational cost, so precision matters too.

### Medical screening

Missing a disease can be very costly.

```text
Prioritize Recall
```

### Spam filtering

Sending legitimate email to spam can be costly.

```text
Prioritize Precision
```

### Interview answer

> "The precision-recall trade-off should be driven by the relative business cost of false positives and false negatives."

---

# 8. F1 Score

F1 combines precision and recall using the harmonic mean:

```text
F1 = 2 × Precision × Recall
     -------------------------
       Precision + Recall
```

It is useful when:

- Classes are imbalanced.
- Both precision and recall matter.
- You need a single metric balancing the two.

Important:

> F1 does not automatically solve the business trade-off. Sometimes Fβ is more appropriate.

---

# 9. Fβ Score

Fβ allows us to give more importance to either recall or precision.

```text
β > 1
→ prioritize Recall

β < 1
→ prioritize Precision
```

For example:

```text
F2
→ recall is more important

F0.5
→ precision is more important
```

Interview takeaway:

> **F1 balances precision and recall; Fβ lets you explicitly weight one more heavily.**

---

# 10. ROC-AUC vs PR-AUC

This is an important Senior-level interview question.

## ROC-AUC

ROC curve plots:

```text
True Positive Rate
vs
False Positive Rate
```

It measures how well the model separates positive and negative classes across thresholds.

## PR-AUC

Precision-Recall curve plots:

```text
Precision
vs
Recall
```

For highly imbalanced classification:

> **PR-AUC is often more informative than ROC-AUC because it focuses on performance of the positive/minority class.**

Example:

```text
Fraud = 0.1%
```

ROC-AUC may still look very good because the enormous number of negative examples makes the false-positive rate appear small.

PR-AUC provides a more useful view of positive-class performance.

---

# 11. How Do You Handle Class Imbalance?

There are four major strategies:

```text
              Class Imbalance
                    |
       +------------+------------+
       |            |            |
       ↓            ↓            ↓
   Resampling   Class Weights   Threshold
       |            |            |
       ↓            ↓            ↓
 Over/Under     Cost-sensitive   Decision
 sampling       learning         tuning

                    +
              Better Metrics
```

---

# 12. Undersampling

Reduce the number of majority-class examples.

Example:

```text
Before:

Normal → 99,000
Fraud  →  1,000

After:

Normal → 10,000
Fraud  →  1,000
```

### Advantage

Smaller training dataset.

Can make the minority class more influential.

### Disadvantage

You may throw away useful information from the majority class.

### Interview answer

> "Undersampling reduces the majority class, but the main trade-off is potentially losing useful information."

---

# 13. Oversampling

Increase the number of minority-class examples.

Example:

```text
Before:

Normal → 99,000
Fraud  →  1,000

After:

Normal → 99,000
Fraud  → 20,000
```

The simplest approach is **random oversampling**, where minority examples are duplicated.

### Problem

Duplicating the same minority examples can increase overfitting.

---

# 14. SMOTE

**SMOTE = Synthetic Minority Over-sampling Technique**

Instead of simply duplicating minority examples, SMOTE creates synthetic minority samples.

Conceptually:

```text
Minority Point A
       \
        \  interpolation
         \
       Synthetic Point
         /
        /
Minority Point B
```

It generates new points between existing minority examples.

### Advantage

Can reduce simple duplication and help the model learn a broader minority-class region.

### Limitation

Synthetic examples are not always meaningful.

For example:

- Highly categorical data
- Very sparse data
- Noisy minority class
- Complex class boundaries

can make naive SMOTE less suitable.

### Interview answer

> "SMOTE creates synthetic minority examples by interpolating between existing minority samples. It can help when the minority class is underrepresented, but I would validate that the synthetic data makes sense for the feature space."

---

# 15. Class Weights

Instead of modifying the dataset, modify the **training objective**.

Conceptually:

```text
Majority error → lower cost

Minority error → higher cost
```

Example:

```text
Class 0 → weight = 1
Class 1 → weight = 10
```

The model is penalized more heavily for misclassifying minority examples.

Many ML algorithms support:

```text
class_weight="balanced"
```

or equivalent cost-sensitive learning.

### Why this is attractive

You keep the original data distribution while telling the model:

> "Minority-class mistakes matter more."

### Interview answer

> "Class weighting is often my first approach because it preserves the original dataset while making minority-class errors more expensive during training."

---

# 16. Threshold Tuning

A classifier often produces a probability:

```text
P(fraud) = 0.37
```

The default decision threshold might be:

```text
0.50
```

So:

```text
0.37 < 0.50
→ Normal
```

But if recall is important, we might choose:

```text
Threshold = 0.30
```

Then:

```text
0.37 > 0.30
→ Fraud
```

Lower threshold:

```text
Recall ↑
False Positives ↑
Precision may ↓
```

Higher threshold:

```text
Precision ↑
False Negatives ↑
Recall may ↓
```

This is a powerful production technique because:

> **You don't necessarily need to retrain the model to change the precision/recall trade-off.**

---

# 17. Important: Don't Resample the Test Set

This is a common interview trap.

Suppose the real production distribution is:

```text
Normal → 99%
Fraud  → 1%
```

You may oversample the minority class **during training**.

But your final evaluation should generally represent the real-world distribution.

```text
Training
→ may use resampling

Validation
→ carefully designed

Test
→ realistic production distribution
```

Otherwise, your evaluation metrics may not represent real-world performance.

---

# 18. Data Leakage with Resampling

Another common mistake:

```text
Entire dataset
      ↓
SMOTE
      ↓
Train/Test split
```

This can cause information from the test set to influence synthetic training examples.

Better:

```text
Original dataset
      ↓
Train/Test split
      |
      +---------> Test untouched
      |
      v
Training data
      ↓
SMOTE / oversampling
      ↓
Model training
```

For cross-validation:

```text
Fold
 ↓
Resample training portion
 ↓
Train
 ↓
Evaluate on untouched validation portion
```

---

# 19. Which Technique Would You Choose?

A practical interview answer:

> "I would first understand the business cost of false positives versus false negatives and establish the right evaluation metrics. Then I would try class weighting as a simple baseline. If necessary, I would experiment with under/oversampling or SMOTE. Finally, I would tune the decision threshold against the business objective. I would evaluate on an untouched test set reflecting the production distribution."

That is a strong Senior AI Engineer answer.

---

# 20. Class Imbalance — End-to-End Mental Model

```text
                 Imbalanced Dataset
                        |
                        v
              Understand Business Cost
                        |
                        v
              Select Evaluation Metrics
                        |
            +-----------+-----------+
            |                       |
            v                       v
      Model Training          Data Strategy
            |                       |
      Class Weights         Under / Over Sampling
            |                       |
            +-----------+-----------+
                        |
                        v
                Probability Output
                        |
                        v
                Threshold Tuning
                        |
                        v
            Precision / Recall Trade-off
                        |
                        v
               Production Evaluation
```

---

# 21. Anomaly Detection — At a Glance

**Anomaly detection** is the problem of identifying observations that significantly deviate from expected/normal behavior.

Examples:

- Credit-card fraud
- Network intrusion
- Manufacturing defects
- Sensor failures
- Server incidents
- Unusual user behavior
- Financial transaction anomalies

Mental model:

```text
Normal behavior
     ↓
Learn what "normal" looks like
     ↓
New observation
     ↓
Deviation / anomaly score
     ↓
Threshold
     ↓
Normal / Anomaly
```

---

# 22. Why Is Anomaly Detection Different from Classification?

Traditional classification:

```text
Training data:

Normal → labeled
Fraud  → labeled

Model learns:
Normal vs Fraud
```

Anomaly detection often looks like:

```text
Training data:

Mostly normal
      ↓
Learn normal behavior

New data
      ↓
Is this sufficiently different?
```

The key distinction:

> **Classification learns known classes; anomaly detection often learns normality and identifies deviations from it.**

---

# 23. Types of Anomaly Detection

A useful classification:

```text
                 Anomaly Detection
                        |
        +---------------+---------------+
        |               |               |
        ↓               ↓               ↓
 Unsupervised     Semi-supervised    Supervised
        |               |               |
 No labels        Mostly normal      Labeled anomalies
```

### Unsupervised

No labels are available.

The algorithm tries to identify unusual observations based on the data distribution.

### Semi-supervised

Training data contains mostly/only normal observations.

The model learns:

> "This is normal."

Anything sufficiently different becomes suspicious.

### Supervised

Both normal and anomaly examples are labeled.

At that point, the problem starts looking more like an imbalanced classification problem.

---

# 24. Anomaly Detection vs Imbalanced Classification

This is a very important interview comparison.

| Aspect | Imbalanced Classification | Anomaly Detection |
|---|---|---|
| Labels | Usually available | Often unavailable |
| Minority class | Known | May be unknown |
| Objective | Predict known class | Detect unusual behavior |
| Training | Supervised | Often unsupervised/semi-supervised |
| Example | Spam vs legitimate | Unknown network attack |
| Output | Class probability | Anomaly score |

Key interview answer:

> "If I have representative labeled examples of the positive class, I would generally formulate the problem as imbalanced classification. If positive examples are rare, unknown, or unavailable, anomaly detection becomes more appropriate."

---

# 25. Isolation Forest

One of the most important classical anomaly detection algorithms.

**Isolation Forest** detects anomalies by trying to isolate observations.

Key intuition:

> Anomalies are easier to isolate than normal observations.

Imagine:

```text
Normal points
      ● ● ●
    ● ● ● ●
      ● ●

Anomaly

                         X
```

The anomaly is far away.

Random partitioning tends to isolate `X` quickly.

Normal observations require more splits.

```text
Anomaly
   ↓
Few splits required
   ↓
Short path length
   ↓
Likely anomaly
```

Normal point:

```text
Many splits required
   ↓
Long path length
   ↓
Likely normal
```

---

# 26. Isolation Forest Mental Model

```text
Dataset
   |
   v
Randomly select feature
   |
   v
Randomly select split value
   |
   v
Partition data
   |
   v
Repeat recursively
   |
   v
Measure path length
   |
   v
Short path → anomaly
Long path  → normal
```

### Why it is useful

- Does not require anomaly labels.
- Relatively efficient.
- Works well for many tabular anomaly-detection problems.
- Captures anomalies based on isolation rather than distance alone.

### Interview one-liner

> "Isolation Forest identifies anomalies because unusual points tend to be isolated with fewer random partitioning steps than normal points."

---

# 27. One-Class SVM

**One-Class SVM** learns a boundary around normal observations.

Conceptually:

```text
       Normal region
      +-------------+
     /               \
    |   ● ● ● ● ●     |
    |  ● ● ● ● ● ●    |
     \               /
      +-------------+

              X
          anomaly
```

The model learns a decision boundary around the normal data.

New points outside the learned region can be classified as anomalies.

### Interview one-liner

> "One-Class SVM learns a boundary around normal observations and flags points outside that learned region as anomalies."

### Important trade-off

One-Class SVM can become computationally expensive at large scale and is sensitive to feature scaling and hyperparameters such as the kernel and `ν`.

---

# 28. Autoencoder-Based Anomaly Detection

A neural-network approach.

Train an autoencoder primarily on normal data.

```text
Normal Input
     |
     v
 Encoder
     |
     v
 Latent Representation
     |
     v
 Decoder
     |
     v
Reconstructed Input
```

The model learns to reconstruct normal examples well.

For an anomaly:

```text
Anomaly
   |
   v
Encoder
   |
   v
Decoder
   |
   v
Poor Reconstruction
```

Therefore:

```text
Reconstruction Error ↑
        ↓
Anomaly Score ↑
```

### Mental model

```text
Input
  |
  v
Autoencoder
  |
  v
Reconstruction
  |
  v
Reconstruction Error
  |
  v
Threshold
  |
  +--> Normal
  |
  +--> Anomaly
```

---

# 29. Statistical / Distance-Based Methods

Some anomaly detection methods rely on statistical assumptions or distance.

Examples:

- Z-score
- IQR
- Mahalanobis distance
- DBSCAN-based outlier detection

These can work well when the data structure is relatively simple.

Example:

```text
Mean = 100
Std = 10

Observation = 170

Z-score = (170 - 100) / 10
        = 7
```

A very large deviation from the expected distribution may indicate an anomaly.

### Interview point

> Simple statistical methods can be excellent baselines when the data distribution is well understood.

---

# 30. Local Outlier Factor — LOF

LOF identifies observations that have substantially lower local density than their neighbors.

Mental model:

```text
Dense normal region:

● ● ● ●
 ● ● ●
● ● ● ●


Sparse point:

                 X
```

If a point is much less dense relative to its local neighborhood, it may be an anomaly.

### Interview one-liner

> "LOF detects anomalies by comparing the local density of a point with the density of its neighboring points."

---

# 31. Choosing an Anomaly Detection Algorithm

A practical high-level approach:

| Situation | Good starting point |
|---|---|
| Simple statistical data | Z-score / IQR |
| General tabular data | Isolation Forest |
| Local-density anomalies | LOF |
| Learn boundary around normal data | One-Class SVM |
| Complex/high-dimensional patterns | Autoencoder |
| Labeled anomalies available | Supervised classifier |

Do not treat this as a rigid rule.

> Start with the simplest method that captures the problem, establish a baseline, then evaluate more sophisticated approaches.

---

# 32. The Hardest Part: Defining the Threshold

Most anomaly detectors produce:

```text
Anomaly Score
```

Example:

```text
Transaction A → 0.02
Transaction B → 0.08
Transaction C → 0.91
Transaction D → 0.12
```

We need a threshold:

```text
score > threshold
→ anomaly
```

Suppose:

```text
threshold = 0.80
```

Then:

```text
C → anomaly
```

But the threshold determines:

```text
Lower threshold
→ more anomalies detected
→ more false positives

Higher threshold
→ fewer alerts
→ potentially more missed anomalies
```

This is essentially another precision/recall trade-off.

---

# 33. How Do You Evaluate Anomaly Detection?

If labels are available:

Use standard classification metrics:

```text
Precision
Recall
F1
PR-AUC
ROC-AUC
```

If labels are unavailable:

Evaluation becomes harder.

Possible approaches:

- Expert review
- Historical incident data
- Synthetic anomalies
- Business-rule validation
- Monitoring alert quality
- False-positive rate
- Human feedback

Important:

> **Anomaly detection without ground-truth labels is fundamentally harder to evaluate.**

---

# 34. Precision vs Recall for Anomaly Detection

Suppose an anomaly detector monitors production servers.

```text
1000 alerts/day
```

If almost all alerts are false:

```text
Engineers → alert fatigue
```

So maximizing recall blindly may not be appropriate.

You need to balance:

```text
Detect real incidents
        vs
Avoid excessive false alarms
```

A strong production answer:

> "I would not optimize anomaly detection purely for maximum recall. I would consider alert volume, investigation capacity, business impact, and the cost of missed anomalies versus false alarms."

---

# 35. Point Anomalies vs Contextual Anomalies

Not every unusual value is anomalous.

### Point anomaly

A single observation is unusual.

Example:

```text
Transactions:
₹100
₹120
₹95
₹110
₹1,000,000  ← anomaly
```

### Contextual anomaly

A value is anomalous only in context.

Example:

```text
Temperature = 30°C
```

This may be normal in summer.

But:

```text
30°C at a freezer sensor
```

may be highly anomalous.

Context can include:

- Time
- Location
- User
- Device
- Season
- Business state

---

# 36. Collective Anomalies

A group of observations may collectively represent an anomaly even if individual observations appear normal.

Example:

```text
Login events:

09:01 → India
09:03 → India
09:05 → India
09:06 → US
09:07 → Germany
09:08 → Singapore
09:09 → Brazil
```

Each login might individually look reasonable.

Together:

> The sequence may represent suspicious behavior.

This is a **collective anomaly**.

---

# 37. Time-Series Anomaly Detection

For time-series data, anomaly detection often considers:

```text
Trend
Seasonality
Noise
Change points
Residuals
```

A common conceptual approach:

```text
Observed signal
      |
      v
Expected signal
      |
      v
Residual
      |
      v
Large residual
      ↓
Possible anomaly
```

Example:

```text
Actual
   |
   |       X
   |      /
   | ----/------ Expected
   |
   +---------------- Time
```

A sudden deviation from expected behavior may indicate an anomaly.

---

# 38. Production Anomaly Detection Architecture

A typical production architecture:

```text
                    Data Sources
                         |
          +--------------+--------------+
          |              |              |
       Logs           Metrics        Events
          |              |              |
          +--------------+--------------+
                         |
                         v
                  Feature Pipeline
                         |
                         v
                Anomaly Detection Model
                         |
                         v
                  Anomaly Score
                         |
                         v
                  Threshold Layer
                         |
              +----------+----------+
              |                     |
          Normal                 Anomaly
                                    |
                                    v
                              Alert / Case
                                    |
                                    v
                             Human Review
                                    |
                                    v
                             Feedback Loop
```

---

# 39. Production Challenges

A Senior AI Engineer should think beyond the algorithm.

Important concerns:

### Data drift

Normal behavior changes over time.

```text
Old normal ≠ New normal
```

### Concept drift

The meaning of normal/anomalous behavior changes.

### False-positive explosion

Too many alerts make the system operationally useless.

### Threshold maintenance

A threshold that works today may not work next month.

### Feedback loop

Human-confirmed anomalies can become future training/evaluation data.

### Latency

Some applications require real-time detection.

### Explainability

Operations teams often need to know:

> "Why was this flagged?"

---

# 40. Class Imbalance vs Anomaly Detection — Interview Scenario

### Question

> "You are building a fraud detection system. Would you use anomaly detection or classification?"

Strong answer:

> "It depends on the availability and quality of fraud labels. If I have enough representative labeled fraud examples, I would start with supervised imbalanced classification because the target class is known. If fraud patterns are evolving and many attack types are unknown, I could complement the classifier with anomaly detection to identify previously unseen behavior."

This is a strong **Senior-level answer** because the two approaches don't have to be mutually exclusive.

---

# 41. Hybrid Fraud Detection Architecture

A production system could use both:

```text
Transaction
     |
     +----------------------+
     |                      |
     v                      v
Supervised Model      Anomaly Detector
     |                      |
Fraud Probability      Anomaly Score
     |                      |
     +----------+-----------+
                |
                v
          Decision Engine
                |
        +-------+-------+
        |               |
     Approve          Review
                        |
                        v
                     Reject
```

For example:

```text
Fraud probability = 0.85
Anomaly score     = 0.92
```

→ High-risk transaction.

Whereas:

```text
Fraud probability = 0.10
Anomaly score     = 0.95
```

might indicate:

> "The transaction doesn't resemble known fraud, but its behavior is unusual."

That can be valuable for discovering new fraud patterns.

---

# 42. Common Interview Questions

## Class Imbalance

### Q1. Why is accuracy bad for imbalanced classification?

> Because the majority class can dominate the metric. A model can achieve high accuracy while completely failing to detect the minority class.

### Q2. Precision vs recall?

> Precision measures how many predicted positives are actually positive. Recall measures how many actual positives we successfully detect.

### Q3. When would you prioritize recall?

> When false negatives are more costly, such as fraud, disease screening, or safety-critical failure detection.

### Q4. How would you handle class imbalance?

> Start with appropriate metrics, understand FP/FN business costs, try class weighting, consider resampling such as SMOTE, and tune the decision threshold. Evaluate on an untouched test set representing production distribution.

### Q5. SMOTE vs class weighting?

> SMOTE changes the training data by creating synthetic minority examples. Class weighting keeps the data distribution but increases the cost of minority-class errors during training.

### Q6. Why shouldn't you SMOTE before train/test split?

> Because synthetic samples can incorporate information derived from the future test set, causing data leakage.

### Q7. ROC-AUC vs PR-AUC for imbalance?

> PR-AUC is often more informative when the positive class is extremely rare because it focuses on precision and recall for the positive class.

---

# 43. Anomaly Detection Interview Questions

### Q1. What is anomaly detection?

> Identifying observations that significantly deviate from expected or normal behavior.

### Q2. Classification vs anomaly detection?

> Classification learns known labeled classes. Anomaly detection often learns normal behavior and identifies deviations, particularly when anomaly labels are scarce or unknown.

### Q3. How does Isolation Forest work?

> It randomly partitions the data. Anomalies tend to be isolated with fewer splits, resulting in shorter path lengths.

### Q4. What is One-Class SVM?

> It learns a boundary around normal observations and flags points outside that boundary as anomalies.

### Q5. How does an autoencoder detect anomalies?

> Train it to reconstruct normal examples. Anomalies generally produce higher reconstruction error, which can be used as an anomaly score.

### Q6. How do you choose the anomaly threshold?

> Based on validation data, business costs, acceptable alert volume, precision/recall requirements, and operational capacity.

### Q7. How do you evaluate anomaly detection without labels?

> Use expert review, historical incidents, synthetic anomalies, business rules, alert quality, and human feedback. But evaluation is inherently harder without ground truth.

---

# 44. Scenario Questions

## Scenario 1 — 99.9% Normal, 0.1% Fraud

**Question:**

> "Your model has 99.9% accuracy. Is it good?"

Answer:

> "Not necessarily. Accuracy is misleading here because the majority class dominates. I would look at precision, recall, F1 and especially PR-AUC, then evaluate the model against the business cost of false positives and false negatives."

---

## Scenario 2 — Recall Is 99%, But Operations Are Overwhelmed

Answer:

> "The threshold may be too aggressive. I would analyze the precision-recall trade-off and increase the threshold if necessary to reduce false positives while maintaining an acceptable recall."

---

## Scenario 3 — No Fraud Labels Exist

Answer:

> "I would initially consider an anomaly-detection approach such as Isolation Forest. But I would also establish a human-review process to generate labels, because eventually labeled feedback can enable a supervised classifier and better evaluation."

---

## Scenario 4 — Known Fraud + New Fraud Patterns

Best approach:

```text
Known fraud
    ↓
Supervised classifier

Unknown behavior
    ↓
Anomaly detector

Both
    ↓
Risk / Decision Engine
```

Answer:

> "I would consider a hybrid system. The supervised model handles known fraud patterns while an anomaly detector provides a signal for previously unseen behavior."

---

## Scenario 5 — Anomaly Detector Generates Too Many Alerts

Investigate:

```text
Threshold
   ↓
Data drift
   ↓
Feature quality
   ↓
Seasonality/context
   ↓
Model assumptions
   ↓
Duplicate events
```

Then tune the threshold or retrain/recalibrate the system based on the root cause.

---

# 45. One-Line Definitions for Fast Revision

**Class Imbalance:** A classification problem where one or more classes have substantially fewer examples than others.

**Precision:** Of predicted positives, how many are actually positive?

**Recall:** Of actual positives, how many did we detect?

**F1:** Harmonic mean of precision and recall.

**PR-AUC:** Area under the precision-recall curve; particularly useful for evaluating rare positive classes.

**Class Weighting:** Increase the training penalty for errors on selected classes.

**Undersampling:** Reduce majority-class examples.

**Oversampling:** Increase minority-class examples.

**SMOTE:** Generate synthetic minority-class examples through interpolation.

**Threshold Tuning:** Adjust the probability cutoff to control the precision-recall trade-off.

**Anomaly Detection:** Identify observations that deviate significantly from expected behavior.

**Isolation Forest:** Detect anomalies based on how quickly observations can be isolated through random partitioning.

**One-Class SVM:** Learn a boundary around normal observations.

**Autoencoder Anomaly Detection:** Detect anomalies using high reconstruction error.

**LOF:** Detect points whose local density is substantially lower than that of their neighbors.

**Point Anomaly:** A single observation is unusual.

**Contextual Anomaly:** An observation is anomalous only given its context.

**Collective Anomaly:** A group or sequence of observations is anomalous collectively.

**Concept Drift:** The underlying relationship or definition of normal behavior changes over time.

---

# 46. Final Interview Summary

The most important mental model is:

```text
             Rare Positive Class?
                    |
          +---------+---------+
          |                   |
         YES                  NO
          |                   |
          v                   v
  Understand FP/FN       Standard Classification
        cost
          |
          v
  Precision / Recall /
       PR-AUC
          |
          v
  Class Weighting /
  Resampling
          |
          v
  Threshold Tuning
```

For anomaly detection:

```text
       Do we have reliable anomaly labels?
                    |
          +---------+---------+
          |                   |
         YES                  NO
          |                   |
          v                   v
   Imbalanced            Anomaly Detection
   Classification              |
          |             +------+------+
          |             |             |
          |       Isolation Forest  Autoencoder
          |       One-Class SVM     LOF / Statistical
          |             |             |
          +-------------+-------------+
                        |
                        v
                 Anomaly Score
                        |
                        v
                    Threshold
                        |
                        v
                 Human / System Action
```

### The Senior AI Engineer perspective

Don't answer these questions only with algorithms.

Think in terms of:

```text
Business Cost
     ↓
Data / Labels
     ↓
Model Choice
     ↓
Metric
     ↓
Threshold
     ↓
Operational Impact
     ↓
Monitoring & Feedback
```

The strongest interview answer is rarely:

> "Use SMOTE."

or:

> "Use Isolation Forest."

Instead:

> **"I would first understand the data, label availability, business cost of false positives and false negatives, and operational constraints. Then I would choose the modeling strategy and evaluation metrics accordingly."**

This is the level of reasoning expected from a Senior/Staff AI Engineer.