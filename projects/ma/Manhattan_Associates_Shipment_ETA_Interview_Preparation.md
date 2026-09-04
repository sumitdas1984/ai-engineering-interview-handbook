# Manhattan Associates — Interview Preparation
## Shipment ETA / Delay Prediction ML Project

> **Goal:** Explain the MA work confidently end-to-end: business problem → ML → productionization → AWS → CI/CD → your contribution.

## 1. 30-Second Introduction

> “Manhattan Associates is a leader in supply-chain and warehouse-management software. Around 2020, the Bangalore team was building ML capabilities on top of its data-driven supply-chain platform, and I was part of that ML team.
>
> One of the key projects I worked on was shipment ETA prediction. The objective was to predict the expected delay category of a shipment using historical shipment information, route characteristics, carrier performance, weather and temporal features. I worked across the ML lifecycle—from feature engineering and model development using Random Forest and XGBoost to productionizing the models for both batch and real-time inference using FastAPI, Docker and AWS ECS/Fargate, with GitHub Actions for CI/CD.”

## 2. 2–3 Minute End-to-End Story

> “The business problem was around shipment delays. In supply-chain operations, knowing whether a shipment is likely to arrive on time or experience a delay can help logistics teams with customer communication, warehouse planning and operational decisions.
>
> We built an ML-based shipment delay prediction capability. Instead of predicting an exact ETA initially, we formulated the problem as a classification problem with four delay buckets: on time, minor delay, moderate delay and major delay.
>
> We used shipment-level information such as origin and destination, shipment date and time, route information such as distance and route type, carrier performance metrics such as historical delay rate and average delay, weather conditions at origin and destination, and temporal features such as day of week, weekend, holiday and peak-season indicators.
>
> I worked on data preparation and feature engineering and experimented with tree-based ML models, particularly Random Forest and XGBoost. These models were a good fit because the problem contains heterogeneous features and potentially nonlinear relationships.
>
> After training, we evaluated the models using classification metrics such as accuracy and F1-score, and also considered ROC-AUC.
>
> My work wasn't limited to model development. A major part was taking the model toward production. We exposed the trained model through a FastAPI inference service and containerized it using Docker. We supported both batch inference for processing larger sets of shipments and real-time inference through an API.
>
> For deployment, we used Amazon ECR for container images and ECS with Fargate for running the containerized inference service. GitHub Actions was used for CI/CD, and CloudWatch was used for operational monitoring.
>
> So the project gave me experience across the complete ML lifecycle—from problem formulation and feature engineering through model development, evaluation, API development, containerization, cloud deployment and CI/CD.”

The README documents this workflow, feature set, model choices and deployment stack. fileciteturn1file0

## 3. Business Problem

**Problem:** Shipment delays create operational uncertainty in supply-chain and warehouse operations.

**Business value:**
- Customer communication
- Warehouse planning
- Operational decision-making
- Better planning around potentially delayed shipments
- Reduction of costs caused by unforeseen delays

**Key framing:**

> “The objective wasn't simply to build an ML model; it was to build and productionize an ML capability that could provide shipment-delay predictions for supply-chain operations.”

## 4. ML Problem Formulation

**Target:** `delay_bucket`

| Value | Meaning |
|---|---|
| 0 | On Time |
| 1 | Minor Delay |
| 2 | Moderate Delay |
| 3 | Major Delay |

**Classification or regression?**

> “We formulated the initial problem as a multi-class classification problem because the business requirement was to identify the delay category rather than predict an exact number of delay hours.”

## 5. Input Features

**Shipment:** `origin`, `destination`, `ship_date`, `ship_time`

**Route:** `distance_km`, `route_type`

**Carrier:** `carrier_name`, `carrier_delay_rate`, `carrier_avg_delay_hrs`

**Weather:** `origin_weather`, `destination_weather`

**Temporal:** `ship_day_of_week`, `is_weekend`, `is_holiday`, `is_peak_season`

These are the feature groups documented in the project README. fileciteturn1file0

## 6. End-to-End ML Workflow

```text
Shipment / Historical Data
          |
          v
Data Preparation
          |
          v
Feature Engineering
          |
          v
EDA
          |
          v
Model Training
(Random Forest / XGBoost)
          |
          v
Model Evaluation
(Accuracy / F1 / ROC-AUC)
          |
          v
Trained Model
          |
          +--------------------+
          |                    |
          v                    v
     Batch Inference      Real-time API
                               |
                               v
                            FastAPI
                               |
                               v
                             Docker
                               |
                               v
                         AWS ECS/Fargate
                               |
                               v
                          CloudWatch
```

## 7. Model Choice

### Random Forest

> “Random Forest is an ensemble of decision trees. It is robust for structured/tabular data and can capture nonlinear relationships without requiring us to explicitly define those interactions.”

### XGBoost

> “XGBoost is a gradient-boosted-tree approach where trees are built sequentially to improve on previous errors. It is often strong on structured/tabular datasets.”

### Why tree-based models?

> “The data contains numerical, categorical and temporal characteristics, and shipment delays can have nonlinear relationships with factors such as distance, carrier performance and weather. Tree-based models are therefore a natural choice.”

**If asked: Random Forest vs XGBoost?**

| Random Forest | XGBoost |
|---|---|
| Bagging | Boosting |
| Trees are built more independently | Trees are built sequentially |
| Robust baseline | Often strong predictive performance on tabular data |
| Relatively straightforward | More tuning/complexity |

**Do not claim which model was finally selected for production unless you are certain.**

## 8. Evaluation

The README gives these as **example** values:

| Metric | Example |
|---|---:|
| Accuracy | 87% |
| F1-score | 0.84 |
| ROC-AUC | 0.90 |

fileciteturn1file0

> **Important:** Because the README labels these as example values, don't present them as actual production metrics unless you independently remember that they were the real project numbers.

### Why not only accuracy?

> “Accuracy can be misleading when classes are imbalanced. F1 gives us a balance between precision and recall, so I would use it together with class-level metrics and a confusion matrix.”

### Principal-level point

> “I would pay particular attention to the confusion matrix because the business cost of misclassifying a major delay as on-time may be much higher than confusing two neighboring delay categories.”

## 9. Productionization

### Real-time inference

```text
Client / Application
        |
        v
    FastAPI
        |
        v
Input Validation
        |
        v
Feature Preparation
        |
        v
ML Model
        |
        v
Prediction
        |
        v
JSON Response
```

> “For real-time use cases, we exposed the trained model through a FastAPI service. The service accepts shipment information, prepares the model input, performs inference and returns the predicted delay category.”

### Batch inference

> “For larger volumes of shipment records, batch inference allows many shipments to be scored together rather than making an individual synchronous API request for every record.”

The README explicitly documents both batch and real-time inference. fileciteturn1file0

## 10. Docker

> “Docker gave us a reproducible runtime containing the application, dependencies and model-serving code. It also made the inference service portable and easier to deploy consistently across environments.”

## 11. AWS Architecture

```text
                    GitHub
                       |
                       v
                GitHub Actions
                       |
                 Docker Build
                       |
                       v
                     ECR
                       |
                       v
                ECS / Fargate
                       |
                       v
                ML Inference API
                       |
                       v
                  CloudWatch
```

**ECR**

> “Amazon ECR served as the container image registry. The CI/CD pipeline could build the Docker image and push it to ECR.”

**ECS/Fargate**

> “ECS/Fargate allowed us to run the containerized inference service without managing the underlying EC2 infrastructure.”

**CloudWatch**

> “CloudWatch was used for operational monitoring of the deployed service.”

## 12. CI/CD

```text
Code Change
    |
    v
GitHub
    |
    v
GitHub Actions
    |
    +--> Build / Test
    |
    +--> Docker Build
    |
    +--> Push Image to ECR
    |
    +--> Deploy to ECS/Fargate
```

> “GitHub Actions automated the deployment process. The pipeline could build the application and Docker image, push the image to ECR and deploy the updated container to ECS/Fargate. This provided a repeatable deployment process instead of relying on manual deployment.”

If asked what you would add today:

> “Today I would additionally include automated unit/integration tests, model validation, security scanning, model/version tracking, deployment approval gates and potentially canary or blue-green deployment.”

## 13. Your Contribution

> “My contribution spanned both the ML and engineering sides. On the ML side, I worked on data preparation, feature engineering, model development and evaluation, particularly Random Forest and XGBoost. On the engineering side, I worked on productionizing the inference layer, including FastAPI services, Dockerization, AWS deployment using ECR and ECS/Fargate, and the GitHub Actions CI/CD pipeline.”

**Key positioning:**

> **ML + Python + API engineering + Docker + AWS + CI/CD + productionization**

## 14. Likely ML Questions

**Why classification instead of regression?**

> “The initial business requirement was to categorize shipments by severity of delay. If the business later required an exact delay duration, I would formulate it as regression or potentially use a two-stage approach.”

**Why Random Forest?**

> “It is robust on structured data, captures nonlinear relationships and interactions, and provides a strong baseline with relatively little preprocessing.”

**Why XGBoost?**

> “XGBoost is a strong gradient-boosted-tree algorithm for structured data and can capture complex nonlinear relationships while often delivering strong predictive performance.”

**How would you handle class imbalance?**

> “First I would inspect class distribution and per-class precision/recall. Depending on the problem, I could use class weights, resampling, and evaluate with F1 or other class-sensitive metrics rather than relying only on accuracy.”

**How would you prevent data leakage?**

> “I would make sure that features available only after the prediction point are not used during training. Future shipment outcomes or post-delivery information must not leak into the feature set.”

**How would you validate the model?**

> “I would use a proper train/validation/test strategy. For time-dependent shipment data, I would also consider a time-based split to avoid training on future information.”

## 15. Likely Production Questions

**Why FastAPI?**

> “It is a lightweight Python framework suitable for exposing ML inference through REST APIs, with request validation, automatic API documentation and straightforward containerization.”

**Why Docker?**

> “For reproducible environments and portable deployment.”

**Why ECS/Fargate?**

> “To run containerized services while avoiding direct management of EC2 infrastructure.”

**How would you scale the service?**

> “I would run multiple stateless inference tasks behind a load balancer and use autoscaling based on appropriate signals such as request volume, CPU/memory utilization or latency.”

**What if the model service goes down?**

> “Use multiple service instances, health checks and automated task replacement. Depending on business requirements, I would also consider retries, timeouts and a fallback strategy.”

**How would you reduce inference latency?**

> “Profile the complete request path first, then optimize feature preparation, model loading, serialization and network calls. The model should be loaded once per process rather than for every request.”

## 16. Monitoring — Service vs ML

### Service monitoring
- Request volume
- Latency
- Error rate
- Container health
- CPU/memory
- Availability

### ML monitoring
- Prediction distribution
- Feature distribution
- Data drift
- Model performance when ground truth becomes available
- Class distribution changes
- Model/version tracking

Principal-level answer:

> “Operational monitoring and ML monitoring are different concerns. CloudWatch can help with service health, but a mature ML platform should also monitor input drift, prediction drift and eventually model performance against ground truth.”

## 17. “What Would You Do Differently Today?”

> “The original solution was focused on solving the shipment prediction problem and getting it into production. If I were designing it today at platform scale, I would make the ML lifecycle more standardized and reusable.
>
> I would introduce stronger model versioning and model registry capabilities, automated model validation, centralized observability, drift monitoring and controlled deployment strategies. I would also separate the model-serving infrastructure from individual models so that multiple ML use cases could use a common serving platform.”

Possible additions:
- Automated testing
- Security scanning
- Model approval gates
- Canary/blue-green deployment
- Model rollback
- Feature/data validation
- Standard inference API
- Cost and latency monitoring

## 18. “What If Traffic Increases 10×?”

> “I would keep the inference service stateless so it can scale horizontally. For real-time traffic, ECS could run additional Fargate tasks behind a load balancer with autoscaling. For batch workloads, I would decouple ingestion and processing and scale workers independently. I would also monitor latency, throughput and resource utilization to determine the scaling policy.”

## 19. Architecture Trade-offs

### Real-time vs batch

**Real-time**
- Low latency
- Immediate prediction
- More operational complexity

**Batch**
- Efficient for large volumes
- Good for periodic planning
- Not appropriate when an immediate response is required

### ECS/Fargate vs EC2

**Fargate**
- Less infrastructure management
- Good for containerized services

**EC2**
- More infrastructure control
- Potentially useful for specialized workloads

### Random Forest vs XGBoost

Consider:
- Robust baseline vs potentially stronger tabular performance
- Training characteristics
- Interpretability/complexity
- Inference and operational requirements

## 20. Important “Don't Overclaim” Rules

The README supports:
- Synthetic dataset
- Four-class delay prediction
- Listed shipment/route/carrier/weather/time features
- Random Forest and XGBoost
- Accuracy/F1/ROC-AUC evaluation
- Batch and real-time inference
- FastAPI
- Docker
- GitHub Actions
- ECR
- ECS/Fargate
- CloudWatch

The README does **not** establish:
- Exact production dataset
- Exact production metrics
- Exact final production model
- Exact database/storage architecture
- Exact autoscaling configuration
- Exact model registry
- Exact drift-monitoring implementation

If asked about these, answer from your actual experience rather than inventing details.

## 21. Final 2-Minute Revision Card

```text
MANHATTAN ASSOCIATES
        |
        v
Supply Chain / Warehouse Management
        |
        v
Shipment Delay Problem
        |
        v
4-Class Classification
On-Time / Minor / Moderate / Major
        |
        v
Features
Shipment + Route + Carrier + Weather + Time
        |
        v
Random Forest / XGBoost
        |
        v
Accuracy / F1 / ROC-AUC
        |
        v
Productionization
        |
        +----------------+
        |                |
        v                v
     Batch          Real-time
                         |
                         v
                      FastAPI
                         |
                         v
                       Docker
                         |
                         v
                   AWS ECR
                         |
                         v
                  ECS / Fargate
                         |
                         v
                    CloudWatch

             GitHub Actions
                  CI/CD
```

### One-line positioning

> **“At Manhattan Associates, I worked across the complete lifecycle of a production ML capability—from shipment-delay prediction and feature engineering to model development, API-based inference, containerization, AWS deployment and CI/CD.”**

## 22. Career Evolution Connection

If asked how the experience connects to your current profile:

> “My work at Manhattan Associates was an important step from traditional ML model development into production AI engineering. I was working not only on predictive modeling but also on APIs, containers, AWS and CI/CD, which later helped me move toward larger AI/ML platforms and eventually GenAI and agentic AI architecture.”
