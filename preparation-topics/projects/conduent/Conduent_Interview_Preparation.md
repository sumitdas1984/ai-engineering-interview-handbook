# Conduent Labs — Interview Preparation
## Hate Speech Classification + Dynamic Crime Hotspot Prediction

## 1. How to position your Conduent experience

At Conduent Labs, I worked on two machine-learning problems that gave me experience across both **NLP/deep learning** and **spatio-temporal predictive analytics**.

The first project was an **NLP-based hate-speech classification system**, where I developed and productionized an LSTM-based classifier and built both batch and real-time inference services using FastAPI, Docker, GitHub Actions, Amazon ECR, and Amazon ECS/Fargate.

The second project was **dynamic crime hotspot and crime-volume prediction**, where we used historical crime records and geospatial information to engineer spatio-temporal features and built a two-stage ML approach using algorithms including Random Forest and XGBoost.

The important story is not just the models. It is that I worked across the complete ML lifecycle: **problem formulation → data preparation → feature engineering → model development → evaluation → inference → deployment/productionization**.

---

# PROJECT 1 — HATE SPEECH CLASSIFICATION

## 2. 30-second interview answer

> At Conduent, I worked on an NLP classification problem for detecting potentially harmful content in customer/user reviews or textual content. The objective was to classify text into categories such as hate speech, offensive language, and neutral content.
>
> I developed an LSTM-based deep-learning model and took it beyond experimentation by building batch and real-time inference services using FastAPI and Docker. I also automated the deployment pipeline using GitHub Actions, Amazon ECR, and Amazon ECS/Fargate.
>
> The project gave me practical experience across NLP preprocessing, sequence modeling, model evaluation, and productionizing an ML model as a cloud-hosted service.

---

## 3. Business problem

The fundamental problem was **automated content classification/moderation**.

Manually reviewing large volumes of textual content does not scale. The system therefore needed to automatically determine whether a piece of text was:

1. Hate speech
2. Offensive language
3. Neutral/non-offensive content

A useful distinction to mention in an interview:

> Offensive language and hate speech are not necessarily the same thing. Offensive language may contain abusive language without explicitly targeting a group or individual in the same way as hate speech.

This made the problem a **multi-class text classification problem**.

---

# 4. End-to-end ML flow

```text
Raw Text
   ↓
Text Cleaning
   ↓
Stop-word Removal / Lemmatization
   ↓
Train / Validation Split
   ↓
Tokenization
   ↓
Sequence Padding
   ↓
Embedding
   ↓
Bidirectional LSTM
   ↓
Dense + Regularization
   ↓
Softmax
   ↓
Class Probabilities
   ↓
Hate / Offensive / Neutral
```

The reference implementation uses tokenization with a vocabulary limit and fixed sequence length, followed by an embedding layer and bidirectional LSTM. It also uses batch normalization, dropout and L1 regularization before the final softmax layer. The attached reference reports about 91% validation accuracy for that implementation.

**Interview caution:** do not present the reference article's 91% as your Conduent production metric unless you have an actual project metric confirming it.

---

# 5. Data and preprocessing

The text classification pipeline involved standard NLP preprocessing.

### Cleaning

Typical steps included:

- Convert text to lowercase
- Remove punctuation/noise
- Remove stop words
- Lemmatize words
- Convert text into a normalized representation

The purpose was to reduce irrelevant variation and make the textual input more suitable for sequence modeling.

### Tokenization

The cleaned text was converted into integer token sequences.

For example:

```text
"this product is really good"
        ↓
[21, 54, 87, 12, 36]
```

Because neural networks require consistent tensor shapes, sequences were padded/truncated to a fixed maximum length.

### Class imbalance

A key challenge in hate-speech classification is class imbalance.

The attached reference dataset is heavily dominated by offensive-language examples, while hate-speech examples are much smaller. The reference implementation addresses this using a combination of upsampling and downsampling.

In an interview, explain the general principle:

> Accuracy can be misleading when the classes are imbalanced, so I would pay particular attention to per-class precision, recall and F1, especially for the hate-speech class.

---

# 6. Why LSTM?

A likely question:

### Why did you choose LSTM?

Good answer:

> This is sequential text data, so the order and context of words matter. LSTM networks are designed to model dependencies across sequences and are better suited than simple feed-forward networks for this type of problem. A bidirectional LSTM can additionally use contextual information from both directions of the sequence.

Do not oversell LSTM as the only or best modern solution.

A Principal-level follow-up:

> Today, depending on latency, data size and accuracy requirements, I would also benchmark transformer-based models such as BERT-family models or smaller distilled transformers. The choice would be based on accuracy, latency, cost, model size and operational constraints.

---

# 7. Model architecture

The reference implementation has this conceptual architecture:

```text
Token IDs
   ↓
Embedding
   ↓
Bidirectional LSTM
   ↓
Dense Layer
   ↓
Batch Normalization
   ↓
Dropout
   ↓
Softmax
   ↓
3 Classes
```

### Embedding

Maps token IDs into dense vectors.

Instead of representing a word as a single integer, the embedding represents it as a learned vector.

### Bidirectional LSTM

Processes the sequence in both directions and captures contextual patterns.

### Dense layer

Combines the learned sequence representation for classification.

### Batch normalization

Helps stabilize training.

### Dropout

Reduces overfitting by randomly dropping activations during training.

### L1 regularization

Encourages a more constrained model and can produce sparse weights.

### Softmax

Produces a probability distribution across the three classes.

---

# 8. Loss and optimization

For three mutually exclusive classes, categorical cross-entropy is appropriate.

Conceptually:

```text
Text → model → [P(hate), P(offensive), P(neutral)]
```

The model is trained to assign high probability to the correct class.

The reference implementation uses:

- Adam optimizer
- Categorical cross-entropy
- Accuracy as a training metric

Training also uses:

- EarlyStopping
- ReduceLROnPlateau

### Why EarlyStopping?

To stop training when validation performance stops improving and restore the best model weights.

### Why ReduceLROnPlateau?

When validation loss stops improving, reduce the learning rate so that optimization can make smaller updates and potentially converge to a better solution.

---

# 9. Evaluation

For this problem I would not rely only on accuracy.

Important metrics:

### Precision

Of the texts predicted as hate speech, how many were actually hate speech?

### Recall

Of all actual hate-speech examples, how many did we detect?

### F1

Balances precision and recall.

For moderation systems, the appropriate operating point depends on business risk.

For example:

- High recall → catch more potentially harmful content but may increase false positives.
- High precision → fewer false accusations but potentially more harmful content gets missed.

A strong answer:

> I would evaluate overall performance as well as class-level precision, recall and F1, and choose the production threshold based on the relative cost of false positives and false negatives.

---

# 10. Productionization

This is one of the strongest parts of your Conduent story.

The work was not limited to model training. You productionized the model through inference services.

## Real-time inference

```text
Client
  ↓
FastAPI
  ↓
Preprocessing
  ↓
Loaded LSTM Model
  ↓
Prediction
  ↓
JSON Response
```

FastAPI provides the HTTP API layer around the trained model.

A typical conceptual API:

```text
POST /predict
{
    "text": "..."
}
```

Response:

```text
{
    "label": "offensive",
    "probabilities": {
        "hate": 0.10,
        "offensive": 0.82,
        "neutral": 0.08
    }
}
```

## Batch inference

For large volumes of text, processing records individually through a synchronous API is inefficient.

Instead:

```text
Batch Input
   ↓
Preprocessing
   ↓
Model Inference
   ↓
Batch Predictions
   ↓
Output Dataset
```

This is useful when latency is not the primary requirement and throughput is more important.

---

# 11. Docker + AWS deployment

The productionization flow:

```text
Python ML Application
        ↓
FastAPI Service
        ↓
Docker Image
        ↓
Amazon ECR
        ↓
Amazon ECS / Fargate
        ↓
Running Inference Service
        ↓
CloudWatch
```

### Docker

Packages the application, dependencies, model-serving code and runtime environment into a reproducible container.

### Amazon ECR

Stores the Docker image.

### Amazon ECS/Fargate

Runs the containerized inference service without requiring you to manage EC2 servers directly.

### CloudWatch

Used for operational monitoring/logging.

---

# 12. CI/CD

You also automated deployment using GitHub Actions.

Conceptually:

```text
Developer Commit
      ↓
GitHub Actions
      ↓
Build / Test
      ↓
Build Docker Image
      ↓
Push Image → ECR
      ↓
Deploy / Update ECS Service
      ↓
Fargate Runs New Version
```

A good interview statement:

> The main benefit was repeatability. Instead of manually building and deploying the inference service, the pipeline made application changes reproducible and reduced deployment friction.

---

# 13. Batch vs real-time — likely interview question

### When would you use each?

**Real-time:**

- User-facing application
- Low-latency decision
- Individual request
- Immediate response required

**Batch:**

- Large historical dataset
- Periodic processing
- Throughput more important than latency
- Offline scoring

Principal-level answer:

> I would avoid forcing everything through real-time APIs. The serving mode should follow the business SLA. For low-latency decisions I would use an online service, while periodic large-scale scoring is better handled through batch processing.

---

# 14. Likely cross-questions — Hate Speech

### Why not traditional ML such as TF-IDF + Logistic Regression?

> I would absolutely establish a classical ML baseline such as TF-IDF plus Logistic Regression or Linear SVM. It is inexpensive, interpretable and often surprisingly strong for text classification. LSTM becomes attractive when sequence/context modeling provides meaningful improvement.

### How would you handle class imbalance?

> Class weights, controlled oversampling/undersampling, threshold tuning and evaluation using class-level precision, recall and F1 rather than accuracy alone.

### How would you reduce false positives?

> Analyze the confusion matrix, inspect misclassified examples, improve training data, tune thresholds, consider class weighting, and potentially introduce a human-review path for uncertain or high-impact cases.

### What would you do today?

> I would benchmark the LSTM against transformer-based models and a strong classical baseline, then select based on accuracy, latency, cost, explainability and operational complexity.

### How would you monitor the model in production?

Monitor two dimensions:

**System:**
- latency
- throughput
- error rate
- CPU/memory
- container health

**ML:**
- prediction distribution
- class distribution
- input drift
- data quality
- confidence distribution
- eventual precision/recall when labels become available

---

# PROJECT 2 — DYNAMIC CRIME HOTSPOT & CRIME VOLUME PREDICTION

## 15. 30-second interview answer

> The second project was a predictive analytics problem around dynamic crime hotspots. The objective was not simply to identify historical hotspots, but to predict where crime activity would be higher in the next month so that resources could potentially be deployed more effectively.
>
> We used historical crime records containing timestamps and geographic coordinates. We divided the geographical region into configurable square grids, mapped crime events to those grids, aggregated them monthly, and engineered spatio-temporal features such as recent crime volume, grid rank, hotspot history and neighboring-grid activity.
>
> We formulated the problem as a two-stage classification system. First, we predicted whether a grid would have zero or non-zero crime volume. For non-zero grids, a second classifier predicted the crime-volume bucket. We then used the predicted volume and its ordinal information to identify the top N future hotspots.
>
> We evaluated Logistic Regression, CART, Random Forest and XGBoost, with XGBoost performing best in the experiments.

---

# 16. Business problem

Traditional hotspot maps are largely descriptive:

> "Where has crime historically occurred?"

The more useful predictive question is:

> "Which areas are likely to become hotspots in the next time window?"

The objective was therefore to predict **future crime intensity at a geographic grid level**.

Potential operational applications include:

- police resource allocation
- patrol planning
- proactive deployment
- understanding future crime risk

The paper also discusses potential use in travel-related applications.

---

# 17. Why grid-based modeling?

Raw latitude/longitude points are difficult to model directly as operational regions.

So the geographic area was divided into square grids.

Conceptually:

```text
+----+----+----+----+
| G1 | G2 | G3 | G4 |
+----+----+----+----+
| G5 | G6 | G7 | G8 |
+----+----+----+----+
| G9 |... |... |... |
+----+----+----+----+
```

Each crime event:

```text
(latitude, longitude)
        ↓
     Grid ID
```

The study experimented with grid sizes and found **1 sq-km** to be a reasonable configuration for the Denver experiment.

---

# 18. Data

The experiment used five years of Denver crime data covering **2013–2018**.

The dataset included:

- timestamp
- longitude
- latitude
- address
- neighborhood
- precinct ID
- crime category

Traffic accident instances were excluded.

Importantly, the approach intentionally relied only on **historical crime data**, rather than demographic or socioeconomic attributes.

This is worth mentioning because the paper discusses concerns around:

- stale socioeconomic data
- geographic granularity mismatch
- privacy/civil-rights considerations

---

# 19. Why monthly prediction?

A very important design decision.

Crime events become sparse when the geographic area is divided into small grids.

Trying to predict at daily or weekly granularity can produce too few observations per grid to build a robust generalized model.

Therefore the approach used a **one-month future prediction window**.

Strong interview answer:

> The temporal granularity was a modeling trade-off. Finer granularity sounds attractive, but after spatial partitioning the data becomes sparse. Monthly aggregation provided a better balance between prediction usefulness and statistical robustness.

---

# 20. Feature engineering

This was one of the core technical contributions.

### Historical grid features

For the previous 1, 2 and 3 months:

- mean crime count
- normalized crime count
- grid rank
- hotspot frequency
- hotspot history

### Neighborhood features

- total crime in neighboring grids over the previous 1, 2 and 3 months

So the model captures both:

**Temporal behavior**

```text
What happened recently in this grid?
```

and

**Spatial behavior**

```text
What happened recently around this grid?
```

This is why the problem is naturally described as **spatio-temporal prediction**.

---

# 21. Why normalize crime count?

Crime count depends partly on the geographic area.

The approach calculated:

```text
normalized crime volume
=
crime count / grid area
```

This gives a density/intensity-like measure rather than relying purely on raw counts.

Since the experimental grids were equal-sized, this also provides a consistent interpretation of crime intensity.

---

# 22. Why predict buckets instead of exact crime count?

This is a very good interview question.

The raw crime-count distribution was highly skewed:

```text
Many grids → very few crimes
Few grids  → very high crime
```

Predicting an exact count can therefore be difficult.

The solution discretized crime volume into buckets.

For Denver:

```text
0
1–7
8–15
16–41
42–103
>103
```

This converted the problem from regression into classification.

Benefits:

1. Easier interpretation
2. Better handling of highly skewed count distribution
3. Natural ranking of areas by predicted volume bucket

---

# 23. Why two-stage classification?

This is probably the most important technical idea in the project.

Instead of directly predicting all six buckets:

```text
Input
 ↓
6-class classifier
```

the solution used:

```text
                 ┌→ Zero
Input → Stage 1 ┤
                 └→ Non-zero
                       ↓
                 Stage 2
                       ↓
              Bucket 1...5
```

### Stage 1

Predict:

```text
zero vs non-zero
```

### Stage 2

Only for non-zero grids:

```text
1–7
8–15
16–41
42–103
>103
```

### Why?

The zero/non-zero distinction is much easier and very important because the data is sparse.

The second classifier can then focus on distinguishing different levels of activity among grids that are actually expected to have crime.

---

# 24. Model selection

The experiments compared:

- Logistic Regression
- CART
- Random Forest
- XGBoost

XGBoost performed best.

### Stage 1

XGBoost:

- Precision ≈ 0.99
- Recall ≈ 0.98
- F1 ≈ 0.99

### Stage 2

XGBoost:

- Precision ≈ 0.69
- Recall ≈ 0.69
- F1 ≈ 0.69

The paper therefore selected XGBoost for crime-volume prediction.

---

# 25. Why XGBoost?

A strong answer:

> The engineered features were primarily tabular spatio-temporal features, with nonlinear relationships and interactions. Tree-based boosting methods are well suited to this kind of structured data. We benchmarked several algorithms rather than assuming one model would be best, and XGBoost performed best in the experiments.

Avoid saying XGBoost is always better.

---

# 26. Hotspot prediction

Once the next month's volume bucket was predicted, the system used that output to determine hotspots.

The number of desired hotspots, **N**, was supplied as an input.

Conceptually:

```text
Predicted Volume Bucket
        ↓
Rank grids by predicted crime intensity
        ↓
Select top N
        ↓
Future Hotspots
```

The paper's algorithm uses the bucket ordering and model probabilities to construct the hotspot selection.

This is important:

> The system does not simply predict a binary hotspot label independently. It uses the predicted crime volume information to derive the future hotspot ranking.

---

# 27. Results

The reported hotspot results were:

| Approach | Precision | Recall | F1 |
|---|---:|---:|---:|
| Baseline | 0.75 | 0.73 | 0.74 |
| Proposed approach | 0.80 | 0.77 | 0.79 |

So the proposed approach achieved approximately a **5 percentage-point improvement in F1** over the baseline.

The reported crime-volume F1 for the selected XGBoost models was:

- Stage 1: **0.99**
- Stage 2: **0.69**
- Hotspot prediction: **0.79**

The Stage 2 results also showed that the model performed better for the lowest and highest volume buckets than for some middle buckets.

---

# 28. Evaluation methodology

The data was split **80:20 based on months over the years**.

This temporal split is important because random row-level splitting can create leakage in time-dependent problems.

A good interview statement:

> For a temporal prediction problem, I would preserve chronological separation between training and test data so that the model is evaluated on future-like observations rather than randomly mixed historical observations.

---

# 29. End-to-end architecture

```text
              HISTORICAL CRIME DATA
                       |
                       v
              +------------------+
              |   GIS / Data     |
              |    Management    |
              +------------------+
                       |
             Create geographic grids
                       |
             Map lat/long → grid
                       |
             Aggregate grid/month
                       |
                       v
             +------------------+
             | Feature Engineer |
             | Spatio-Temporal  |
             +------------------+
                       |
                       v
             +------------------+
             | Stage 1 XGBoost  |
             | Zero / Non-Zero   |
             +------------------+
                       |
                 Non-zero grids
                       |
                       v
             +------------------+
             | Stage 2 XGBoost  |
             | Volume Buckets   |
             +------------------+
                       |
                       v
             Predicted Grid Volume
                       |
                       v
             +------------------+
             | Hotspot Ranking  |
             | Select Top N     |
             +------------------+
                       |
                       v
               Future Hotspots
                       |
                       v
                 GIS Visualization
```

The attached paper's system architecture on page 3 presents the same high-level separation between a **GIS module** and an **ML module**, with grid creation/aggregation feeding spatial-temporal feature engineering and ML prediction, followed by visualization.

---

# 30. Important modeling challenges

## Challenge 1 — Sparsity

Most grids had relatively few crime events.

**Solution:**

- monthly aggregation
- volume buckets
- two-stage modeling

## Challenge 2 — Highly skewed distribution

A small number of grids had very high crime volumes.

**Solution:**

- discretize volume into buckets
- hierarchical classification

## Challenge 3 — Spatial dependency

Crime in neighboring areas can provide useful context.

**Solution:**

Include neighboring-grid historical crime features.

## Challenge 4 — Temporal dependency

Recent crime behavior matters.

**Solution:**

Use rolling historical features from the previous 1, 2 and 3 months.

---

# 31. Likely cross-questions — Crime project

### Why not use LSTM or another deep-learning model?

> We considered the nature of the data and the enterprise setting. At monthly grid level the data becomes sparse, and the engineered feature set was primarily structured tabular data. Tree-based models were therefore a practical choice. The paper also explicitly discusses the sparsity and performance considerations for generalized enterprise use.

### Why not predict exact crime count using regression?

> The count distribution was highly skewed and difficult to model robustly. Bucketization made the output easier to interpret and allowed us to formulate the problem as classification.

### Why not directly predict hotspots?

> Direct binary hotspot prediction loses information about the degree of expected crime activity. The hierarchical approach first predicts crime volume and then uses the ordinal volume information to select the top N hotspots.

### Why XGBoost over Random Forest?

> Both are strong tree-based models, but in our experiments XGBoost gave better performance for both stages. Therefore we selected it based on empirical evaluation.

### What is a risk with this model?

> A model trained on historical crime records can reproduce historical reporting or enforcement biases. Even if the model does not use demographic attributes, historical data itself can contain bias. Therefore I would treat predictions as decision support rather than fully autonomous decisions and include governance and human oversight.

### How would you prevent temporal leakage?

> All features for a prediction month must be derived only from data available before that month. I would use chronological train/validation/test splits and carefully construct rolling features.

### What happens when a new area has no history?

> This is a cold-start problem. I would need a fallback strategy, such as neighborhood-level or regional aggregates, or a minimum-history rule, rather than pretending the grid has reliable historical evidence.

---

# 32. Principal-level "What would you improve today?"

A strong answer covering both projects:

> Looking back, I would strengthen both the modeling and production layers.
>
> For the NLP system, I would benchmark a strong classical baseline and modern transformer-based models, introduce better model evaluation and calibration, and establish production monitoring for both system health and model/data drift.
>
> For the crime model, I would strengthen temporal validation, explicitly test for spatial and temporal leakage, evaluate additional forecasting formulations, and quantify uncertainty. I would also treat the output as decision support because historical crime data can contain reporting and enforcement bias.
>
> From an engineering perspective, I would formalize model versioning, experiment tracking, automated testing, deployment gates, monitoring and rollback.

---

# 33. How Conduent connects to your later career

This is a useful career-evolution story.

```text
Conduent
  |
  | ML models + NLP + Deep Learning
  | FastAPI + Docker + AWS
  | CI/CD + production inference
  |
  ↓
Manhattan Associates
  |
  | ML + real-time/batch inference
  | AWS production systems
  |
  ↓
Thomson Reuters
  |
  | NLP + RAG + LLMs
  | Agentic QA
  | Production AI
  |
  ↓
Bosch
  |
  | GenAI platform
  | LLMOps
  | Prompt management
  | Reusable AI capabilities
  |
  ↓
Target
  |
  | Staff/Principal AI Engineer
  | AI Architect
  | Agentic AI Architect
```

The important narrative:

> My early work gave me strong fundamentals in ML modeling and production ML systems. At Conduent I learned how to take ML models into deployable services. At Manhattan I further developed production ML and cloud capabilities. At Thomson Reuters I moved into enterprise NLP, RAG and GenAI. At Bosch I moved further toward AI platform architecture and LLMOps. That progression is why I am now targeting Principal/Architect-level AI roles.

---

# 34. 2-minute combined Conduent story

> At Conduent Labs, I worked on two main ML initiatives.
>
> The first was an NLP classification problem for detecting hate speech and offensive content. I developed an LSTM-based sequence classification model and worked through the complete pipeline from text preprocessing, tokenization and sequence modeling through evaluation. More importantly, I productionized the model by exposing inference through FastAPI, containerizing the service with Docker, and automating deployment using GitHub Actions, Amazon ECR and ECS/Fargate. I also supported both batch and real-time inference patterns.
>
> The second project was dynamic crime hotspot prediction. The goal was to predict where crime activity would be higher in the following month. We used historical crime records with geographic coordinates, divided the area into grids, aggregated crime events by grid and month, and engineered spatio-temporal features using recent crime history and neighboring-grid activity.
>
> One of the interesting challenges was that crime counts were highly skewed and sparse. Instead of predicting an exact count, we converted the problem into volume buckets and used a two-stage classifier: first zero versus non-zero crime volume, and then the volume bucket for non-zero grids. We evaluated Logistic Regression, CART, Random Forest and XGBoost, with XGBoost performing best. The predicted volume was then used to rank and select the top N future hotspots. In the reported experiment, the approach achieved an F1 of 0.79 for hotspot prediction versus 0.74 for the baseline.
>
> Overall, Conduent gave me strong experience across both ML/NLP modeling and production engineering, which became the foundation for my later work in enterprise AI, GenAI and AI platforms.

---

# 35. Rapid revision card

## Hate Speech

**Problem:** 3-class NLP classification

**Classes:** Hate / Offensive / Neutral

**Model:** Embedding → BiLSTM → Dense → BN → Dropout → Softmax

**Training:** Adam + categorical cross-entropy

**Data issues:** class imbalance

**Evaluation:** precision / recall / F1 + accuracy

**Serving:** FastAPI

**Packaging:** Docker

**AWS:** ECR → ECS/Fargate

**CI/CD:** GitHub Actions

**Modes:** batch + real-time

**Principal angle:** classical baseline + transformers + drift/quality monitoring + human review for uncertain/high-impact cases

---

## Crime Prediction

**Problem:** Predict future monthly crime intensity/hotspots

**Data:** Denver crime records, 2013–2018

**Spatial:** geographic square grids

**Temporal:** monthly

**Features:** previous 1/2/3 month statistics + neighboring grids

**Problem formulation:** volume bucket classification

**Stage 1:** zero vs non-zero

**Stage 2:** volume bucket 1–5

**Models:** LR / CART / RF / XGBoost

**Winner:** XGBoost

**Stage 1 F1:** 0.99

**Stage 2 F1:** 0.69

**Hotspot F1:** 0.79

**Baseline hotspot F1:** 0.74

**Key challenge:** sparse + highly skewed data

**Key insight:** predict volume first → derive top N hotspots

**Principal angle:** temporal leakage, spatial leakage, uncertainty, bias, governance, human decision support

---

# 36. Things NOT to overclaim

1. Do not claim the 91% validation accuracy from the attached hate-speech reference as your Conduent production metric unless you have the actual project result.
2. Do not say the crime model predicts the exact number of future crimes; it predicts volume buckets.
3. Do not say the crime model independently predicts a binary hotspot first; hotspot selection is derived from predicted volume.
4. Do not claim deep learning was used for the crime model unless you have a separate project implementation proving it.
5. Do not claim a production crime system if your contribution was research/prototype work.
6. Keep your strongest productionization claims around the hate-speech project where you explicitly worked with FastAPI, Docker, GitHub Actions, ECR and ECS/Fargate.
