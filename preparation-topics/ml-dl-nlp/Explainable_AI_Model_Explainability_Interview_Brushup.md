# Explainable AI (XAI) & Model Explainability — Interview Brush-Up

> **Goal:** High-level, interview-ready understanding for Senior AI Engineer / AI Architect interviews.

## 1. What is Explainable AI?

**Explainable AI (XAI)** helps humans understand **how a model behaves and why it produced a prediction**.

```text
                    ML Model
                       |
          +------------+------------+
          |                         |
     Prediction                Explanation
          |                         |
   "Risk = High"          "Features A, B, C
                           contributed most"
```

Why it matters:
- Trust and human adoption
- Debugging and model improvement
- Feature validation / leakage detection
- Governance and auditability
- Human-in-the-loop decisions
- Fairness investigation

**Important:** Explainability ≠ correctness, fairness, or causality.

---

## 2. Core Classifications

### Interpretability vs Explainability

- **Interpretability:** model is understandable by itself.
- **Explainability:** techniques explain a complex/black-box model.

Examples:

```text
Interpretable: Linear Regression, Logistic Regression,
               Small Decision Tree, Rules

Complex:       XGBoost, Random Forest, Neural Networks
               -> use post-hoc explanations
```

### Global vs Local

| Type | Question | Examples |
|---|---|---|
| **Global** | How does the model behave overall? | Feature importance, PDP, aggregated SHAP |
| **Local** | Why this specific prediction? | SHAP, LIME, Counterfactual |

### Intrinsic vs Post-hoc

```text
Intrinsic:  interpretable model itself explains behavior
Post-hoc:   complex model + separate explanation technique
```

### Model-specific vs Model-agnostic

- **Model-specific:** exploits model structure.
- **Model-agnostic:** treats the model approximately as a black box.

---

# 3. Main Explainability Techniques

## Feature Importance

Answers:

> Which features does the model rely on?

Useful for global understanding.

**Caution:** importance does not imply causality.

---

## Permutation Importance

Shuffle one feature and measure performance degradation.

```text
Baseline performance = 0.90
Shuffle Feature A    -> 0.72

Large drop => Feature A is important
```

**Pros:** model-agnostic, performance-based  
**Caveat:** correlated features can distort individual importance.

---

## PDP — Partial Dependence Plot

Shows the **average prediction change** as a feature varies.

```text
Feature value  --->  Model prediction
```

Good for global behavior.

**Caveat:** correlated features can create unrealistic feature combinations.

---

## ICE — Individual Conditional Expectation

Like PDP, but shows the relationship **for individual observations** rather than only the average.

```text
PDP = average effect
ICE = individual effects
```

Useful for discovering heterogeneous behavior hidden by averages.

---

# 4. LIME

**LIME = Local Interpretable Model-agnostic Explanations**

Core idea:

> Approximate the complex model with a simple model **around one specific prediction**.

```text
Original instance
       |
   Perturb inputs
       |
       v
Black-box model
       |
   Predictions
       |
       v
Simple local surrogate
       |
       v
Explanation
```

**Strengths**
- Model-agnostic
- Intuitive
- Good for local explanations

**Limitations**
- Sensitive to perturbation/neighborhood choice
- Can be unstable
- Local surrogate may not faithfully represent the original model

---

# 5. SHAP

**SHAP = SHapley Additive exPlanations**

Based on **Shapley values** from cooperative game theory.

Intuition:

> Treat each feature as a player contributing to the final prediction.

```text
Prediction
  =
Base / expected prediction
  + Feature A contribution
  + Feature B contribution
  + Feature C contribution
```

Example:

```text
Baseline ETA        2.0

Distance           +0.7
Traffic            +0.4
Lead Time          +0.2
Weather            -0.1
                    ----
Prediction          3.2
```

- Positive SHAP → pushes prediction higher
- Negative SHAP → pushes prediction lower

SHAP can support:

```text
Local:  Why this prediction?
Global: Which features matter across the dataset?
```

### SHAP vs LIME

| | SHAP | LIME |
|---|---|---|
| Core idea | Shapley-value attribution | Local surrogate |
| Main scope | Local + aggregated global | Mainly local |
| Foundation | Cooperative game theory | Local approximation |
| Key concern | Computational cost / attribution assumptions | Stability / neighborhood choice |

**Interview line:**

> "LIME approximates a black-box model locally, whereas SHAP provides feature attribution using a Shapley-value framework."

---

# 6. Counterfactual Explanations

Answers:

> **What would need to change for the prediction to change?**

Example:

```text
Current:
Loan = Rejected

Counterfactual:
Increase income by $8K
and reduce debt ratio by 5%
-> Loan = Approved
```

Good counterfactuals should be:
- Feasible
- Actionable
- Within domain constraints

```text
SHAP       -> Why did it happen?
Counterfactual -> What could change it?
```

---

# 7. Surrogate Models

A simpler model approximates a complex model's behavior.

```text
Complex model
     |
 Predictions
     |
Simple surrogate
     |
Explanation
```

Example:

```text
XGBoost -> Small Decision Tree
```

**Important:** a surrogate explains the **model's behavior**, not necessarily the true causal relationship.

---

# 8. Model-Specific Choices

| Model | Typical explanation |
|---|---|
| Linear / Logistic | Coefficients, odds ratios |
| Small Decision Tree | Tree structure, decision path |
| Random Forest | Permutation importance, SHAP |
| XGBoost / Boosting | SHAP, permutation importance, PDP/ICE |
| Neural Network | Integrated Gradients, saliency, SHAP variants, LIME |
| Vision model | Saliency / attribution maps |
| Text model | Token-level attribution, suitable attribution methods |

### Attention is not automatically an explanation

For transformers:

```text
Attention weights ≠ guaranteed faithful explanation
```

Attention can be useful diagnostically, but attribution/explanation **faithfulness should be evaluated separately**.

---

# 9. Explainability in Production

Treat explainability as part of the ML system.

```text
Request
   |
   v
Model
   |
   +------> Prediction ------> Application
   |
   +------> Explanation
               |
        SHAP / LIME /
        Counterfactual
               |
               v
        User / Analyst
```

Consider logging:

- Model version
- Feature/model version
- Prediction
- Probability/confidence
- Explanation
- Timestamp
- Entity/request ID

### Latency

If:

```text
Prediction = 20 ms
Explanation = 300 ms
```

consider:

```text
Prediction -> synchronous
Explanation -> asynchronous
```

**only if the business workflow allows it.**

Also consider privacy and sensitive information in explanation logs.

---

# 10. Explainability ≠ Fairness ≠ Causality

These are separate concepts:

```text
Explainability -> Why did the model behave this way?
Fairness      -> Is the behavior equitable?
Causality     -> Does changing X cause Y?
```

Examples:

```text
High SHAP value
      ≠
Causal relationship

Explainable model
      ≠
Fair model
```

A feature may also be a proxy for a sensitive attribute, so fairness requires separate analysis.

---

# 11. Common Pitfalls

1. **Correlation ≠ causation**
2. Global importance does not explain every individual prediction.
3. Correlated features can make attribution ambiguous.
4. Post-hoc explanations are approximations; evaluate **faithfulness**.
5. Explanations should be understandable and actionable.
6. Explanation latency matters in production.
7. Do not assume attention weights are explanations.

### Key senior-level question

> **"How faithful and stable is the explanation to the actual model behavior?"**

---

# 12. Interview Decision Framework

If asked **"How would you explain this model?"**, don't immediately say SHAP.

```text
What do we need to understand?
          |
     Global / Local
          |
       What model?
          |
  +-------+--------+
  |                |
Simple           Complex
  |                |
Coefficients      Post-hoc
Tree path            |
                +---+---+---+
                |       |   |
               SHAP   LIME  CF
```

Then consider:

1. **Business question**
2. Global vs local
3. Model type
4. Appropriate technique
5. Faithfulness / stability
6. Human usability
7. Latency / privacy / governance

---

# 13. Practical Example — ETA Prediction

Model:

```text
Distance | Traffic | Weather | Vehicle Type
Historical Route Time | Lead Time
```

Prediction:

```text
ETA = 3.2 days
```

Local SHAP:

```text
Baseline              2.0
Distance             +0.7
Traffic              +0.4
Historical Route     +0.3
Weather              -0.1
Vehicle Type           0.0
Lead Time             -0.1
                      ----
Prediction             3.2
```

This answers:

> "Why did the model predict 3.2 days for this shipment?"

Global SHAP / feature importance answers:

> "Which features generally drive ETA predictions?"

Counterfactual answers:

> "What change could move this shipment into a different prediction category?"

---

# 14. Interview Questions

### Must Know

1. What is Explainable AI?
2. Interpretability vs explainability?
3. Global vs local explanation?
4. Intrinsic vs post-hoc?
5. What is SHAP?
6. How does LIME work?
7. SHAP vs LIME?
8. What is permutation importance?
9. PDP vs ICE?
10. What is a counterfactual explanation?

### Senior-Level

11. How would you explain an XGBoost model?
12. How would you explain a neural network?
13. How do correlated features affect SHAP/permutation importance?
14. How do you validate explanation faithfulness?
15. How would you add explainability to a production API?
16. Would explanations be synchronous or asynchronous?
17. Does explainability guarantee fairness?
18. Does SHAP establish causality?
19. Can attention weights be considered explanations?
20. How would you handle explanation latency?

---

# 15. 60-Second Interview Answer

> **"Explainable AI is about making model behavior understandable, particularly why a model produced a prediction. I distinguish global explanations, which describe overall model behavior, from local explanations, which explain an individual prediction. For inherently interpretable models such as linear models or small decision trees, the model itself provides much of the explanation. For complex models such as XGBoost or neural networks, techniques such as SHAP, LIME, permutation importance, PDP, ICE and counterfactuals can be used. SHAP provides feature-level attribution using a Shapley-value framework, while LIME builds a simple local surrogate around an instance. In production I would also consider explanation faithfulness, stability, latency, privacy, governance and whether the explanation is understandable and actionable. Finally, I would not confuse feature attribution with causality or assume that explainability automatically guarantees fairness."**

---

# 16. Final Quick Revision

```text
XAI
= understand model behavior / predictions

Global
= How does the model behave overall?

Local
= Why this prediction?

Intrinsic
= model is inherently understandable

Post-hoc
= separate technique explains complex model

Permutation Importance
= performance drop after shuffling feature

PDP
= average prediction vs feature

ICE
= individual prediction curves

LIME
= local surrogate around one instance

SHAP
= Shapley-based feature attribution

Counterfactual
= what change could alter prediction?

Surrogate
= simple model approximating complex model

Faithfulness
= explanation accurately reflects model behavior

Causality
= attribution does not establish cause

Attention
= useful signal, but not automatically an explanation
```

## Final Interview Principle

> **Don't start with SHAP. Start with the question the explanation needs to answer.**

```text
Question
   ↓
Global or Local?
   ↓
Model type
   ↓
Explanation method
   ↓
Faithfulness & stability
   ↓
Human usefulness
   ↓
Production constraints
```
