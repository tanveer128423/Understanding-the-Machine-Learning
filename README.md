# Learning Milestone: Understanding the ML Workflow

## Objective

This milestone explains the full machine learning workflow from raw data to reliable predictions in production:

Data -> Features -> Model -> Prediction

It also includes the two operational stages that make ML systems trustworthy in practice:

Evaluation -> Monitoring

The goal is to understand the full system, not just model training.

---

## 1) Complete ML Workflow (In My Own Words)

### 1. Business Problem Definition

Every ML project starts by defining a decision problem.

Examples:

- Will this customer churn?
- Is this transaction fraudulent?
- What is the likely sales value next month?

Why this stage matters:

- It decides what target to predict.
- It defines success metrics and acceptable failure tradeoffs.
- A vague problem definition creates confusion in every later stage.

Connection to next stage:

- Once the problem is clear, we know what data to collect.

### 2. Raw Data Collection

Raw data is gathered from databases, logs, forms, sensors, or APIs.

Typical issues in raw data:

- Missing values
- Duplicates
- Inconsistent formats
- Noise and outliers
- Irrelevant columns

Why this stage matters:

- The model can only learn patterns present in the data.
- Biased or incomplete data creates biased or weak models.

Connection to next stage:

- Raw data is usually not model-ready, so it must be cleaned and prepared.

### 3. Data Cleaning and Preprocessing

This stage fixes quality issues so data becomes reliable input.

Common preprocessing actions:

- Handle null values (impute or drop)
- Remove duplicates
- Standardize formats (dates, categories, units)
- Detect and treat outliers
- Split data into train, validation, and test sets

Why this stage matters:

- Garbage in, garbage out. Poor data quality creates poor predictions.
- Prevents silent errors before feature engineering starts.

Connection to next stage:

- Clean data can now be transformed into meaningful features.

### 4. Feature Engineering

Feature engineering turns cleaned raw columns into numerical signals the model can learn from.

Examples:

- JoinDate -> DaysSinceJoin
- PlanType -> One-hot encoded indicators
- MonthlySpend -> Normalized value
- Timestamp -> Hour-of-day cyclic encoding

Why this stage matters:

- Models do not understand business meaning directly. They learn from numeric representations.
- Feature quality usually has more impact than algorithm choice.

Connection to next stage:

- These engineered features become the actual input X for training.

### 5. Model Training

A model learns a mapping from features X to target y.

Learning process (high level):

- Predict on training examples
- Measure error using a loss function
- Update internal parameters to reduce error
- Repeat until performance stabilizes

Why this stage matters:

- Produces a trained model artifact that can make future predictions.

Connection to next stage:

- The trained model must be evaluated on unseen data before deployment.

### 6. Evaluation (Hidden but Essential Stage)

Evaluation checks whether the model generalizes to new data.

Important rule:

- Test only on data not used during training.

Common metrics:

- Classification: Precision, Recall, F1, ROC-AUC
- Regression: MAE, RMSE

Why this stage matters:

- Prevents false confidence from training-only results.
- Detects overfitting and weak generalization.

Connection to next stage:

- If evaluation is acceptable, the model can be deployed for real predictions.

### 7. Deployment and Prediction

The deployed model receives new incoming raw data and returns predictions.

Inference flow:

- New Raw Data -> Same Feature Transformations -> Model -> Prediction

Critical detail:

- Feature transformations at prediction time must match training-time logic exactly.

Why this stage matters:

- This is where business value is created.
- Predictions are probabilistic estimates, not guaranteed truths.

Connection to next stage:

- Once live, the system must be monitored continuously.

### 8. Monitoring and Retraining (Ongoing)

Model quality can degrade after deployment because the world changes.

What to monitor:

- Input feature distribution changes (data drift)
- Relationship changes between inputs and target (concept drift)
- Live performance metrics over time

Why this stage matters:

- A model that was good at launch can become wrong or harmful later.
- Monitoring triggers retraining when performance drops.

Connection back into pipeline:

- Monitoring feedback leads to new data collection, updated features, and retraining.

---

## 2) Why Each Stage Matters and How Stages Connect

- Problem definition guides what data is needed.
- Data quality controls what signal can exist.
- Feature engineering decides what signal the model can access.
- Model training fits patterns in those features.
- Evaluation validates whether patterns generalize.
- Deployment applies the trained system to new cases.
- Monitoring ensures reliability as conditions change.

This is a chain. Weakness at one stage propagates to later stages.

---

## 3) Real-World Example Traced Through the Full Pipeline (Fraud Detection)

### Problem

Predict whether a payment transaction is fraudulent in real time.

### Raw Data

- Transaction amount
- Merchant country
- Device ID
- Timestamp
- Account age
- Previous transaction location and time

### Cleaning and Preprocessing

- Remove duplicate transaction records
- Handle missing locations
- Standardize timezones and currency units

### Feature Engineering

- Amount deviation from user 90-day average
- Distance from previous transaction location
- Number of transactions in last 24 hours
- Is new device for this user (0/1)
- Hour-of-day cyclic encoding using sine and cosine

### Model Training

- Train a gradient boosting classifier on labeled historical transactions.

### Evaluation

- Use a held-out test set.
- Prioritize precision/recall tradeoff because class imbalance is high.
- Validate ROC-AUC and confusion matrix at business thresholds.

### Deployment and Prediction

- Model outputs fraud probability per transaction.
- Example action policy:
  - Probability < 0.20: approve
  - 0.20 to 0.70: manual review
  - > 0.70: block and alert

### Monitoring and Retraining

- Track false positives and false negatives weekly.
- Watch shifts in feature distributions by region and payment channel.
- Retrain when performance drops below threshold.

---

## 4) Failure Scenario (What Goes Wrong, Where, and Why)

### Scenario: Training-Serving Skew in Feature Scaling

What goes wrong:

- During training, MonthlySpend is standardized using training mean and standard deviation.
- In production, the service accidentally recomputes mean and standard deviation from each incoming batch.

Where it fails:

- Prediction pipeline stage (feature transformation mismatch).

Why it fails:

- The model learned parameter weights under one feature scale.
- Different scaling at inference changes feature magnitudes and breaks learned relationships.

Symptoms:

- Offline test metrics look strong.
- Live predictions become unstable.
- Sudden increase in wrong classifications with no code change in model weights.

How to diagnose:

- Compare training vs serving feature statistics.
- Log transformed feature values in both pipelines for same sample records.
- Verify shared preprocessing artifact is versioned and reused.

Fix:

- Serialize and version the exact training preprocessors.
- Reuse identical pipeline object in both training and inference.
- Add tests that assert transformation parity.

---

## 5) Key Principles I Will Carry Forward

- Models do not understand raw business context; they operate on numeric features.
- Better features usually beat more complex algorithms.
- Evaluation on unseen data is non-negotiable.
- Prediction is probabilistic; decisions require threshold and business logic.
- Monitoring is mandatory because data and behavior change over time.
- Most real ML failures are pipeline failures, not just model failures.

---

## 6) 2-Minute Video Talking Points (Submission Aid)

Use this structure for your short video:

1. Start with the full pipeline in one sentence.
2. Briefly explain each stage from raw data to monitoring.
3. Walk through one real example (fraud detection) end to end.
4. Emphasize why features matter more than algorithm choice.
5. Explain one failure point and how you would diagnose it.
6. Close by stating that ML is an iterative system, not a one-time training step.

Suggested short opening line:
Machine learning is a pipeline where raw data is transformed into features, features are used to train a model, predictions are evaluated, and the system is continuously monitored and improved.
