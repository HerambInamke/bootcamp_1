# AI Feature Specification: Waitlist Confirmation Probability Predictor

This specification details a practically deployable, machine-learning-based feature to predict the probability of a waitlisted (WL) ticket being confirmed before train departure. It integrates directly with the search results and the PNR status details.

---

## Problem It Solves
This feature directly addresses **Problem 5 (Incomplete PNR Journey Information)** and the broader user anxiety associated with booking waitlisted tickets. Approximately 35% of all daily ticket bookings on IRCTC result in a waitlisted status. Passengers have no official way to know if their waitlist position (e.g., `WL 18`) has a realistic chance of confirmation before travel, leading to booking redundancy (buying tickets on multiple trains), travel plan cancellations, and reliance on unverified third-party prediction websites.

## Proposed Feature — User Perspective
From the user’s perspective, the feature operates as an intelligent decision aid embedded inside the train search card and the PNR dashboard:
*   When a user clicks on a train class that has waitlisted seats (e.g., "Sleeper - WL 14"), a small, neat badge appears showing: **"84% Confirmation Probability"** accompanied by a green status indicator.
*   For high probabilities (>75%), a message displays: *"High chance of confirmation based on historical trends. Recommended to book."*
*   For low probabilities (<40%), a red badge appears showing: *"32% Confirmation Probability. We recommend looking for alternative trains."*
*   On the PNR status page, the user sees a dynamic dial that updates as the chart preparation date approaches.

## Model or API Choice
We select a custom **XGBoost (Extreme Gradient Boosting) Classifier** deployed as a serverless containerized endpoint on **Google Vertex AI** or **AWS SageMaker**. 
*   **Why XGBoost?** The prediction task is a tabular classification/regression problem based on structured numerical and categorical features (dates, stations, quotas, classes). Large Language Models (like GPT-4) are unsuitable, slow, and cost-prohibitive for this volume. XGBoost provides sub-10ms inference latency, works reliably over slow 2G mobile data connections (payload size is <50 bytes), and consumes negligible computing resources.

## Training or Input Data
The model requires a structured tabular dataset of historical train bookings.
*   **Data Sourced From:** CRIS (Centre for Railway Information Systems) database archives.
*   **Historical Volume:** 5 years of booking records (approx. 1.8 billion rows).
*   **Model Features (Inputs):**
    1.  `train_number`: Categorical
    2.  `source_station` & `destination_station`: Categorical
    3.  `class_code`: Categorical (`SL`, `3A`, `2A`, `1A`)
    4.  `quota_code`: Categorical (`GN`, `TQ`, `DF`)
    5.  `journey_date`: Date components (month, day of week, day of year)
    6.  `booking_waitlist_number`: Integer (the waitlist position when booked)
    7.  `current_waitlist_number`: Integer (the active position at inquiry time)
    8.  `days_to_departure`: Integer (difference between journey date and current date)
    9.  `seasonality_index`: Float (calculated based on major festivals like Diwali, Holi, Puja, and school holidays)

This data is fully available within Indian Railways' internal database systems and only requires an offline ETL pipeline to clean and train model weights.

## How Output Is Shown to the User
The probability badge is integrated directly into the train availability card on the search results page:

```ascii
┌─────────────────────────────────────────────────────────────────┐
│  12622 - TAMIL NADU EXPRESS | 22:00 MAS ───> 07:10 NDLS         │
│  Duration: 21h 10m | Runs: M T W T F S S                        │
├─────────────────────────────────────────────────────────────────┤
│  [ SL ]                [ 3A ]                [ 2A ]             │
│  WL 14                 AVAILABLE 8           AVAILABLE 2        │
│  ┌───────────────────┐                                          │
│  │ 🟢 84% Confirmation│                                          │
│  │    Probability     │                                          │
│  └───────────────────┘                                          │
└─────────────────────────────────────────────────────────────────┘
```

On the mobile PNR page, this is rendered as a clean circular gauge widget in the [PNR Travel Assistant Dashboard](file:///c:/Projects/bootcamp_1/part-b/SPECS.md#feature-spec-5-consolidated-pnr-travel-assistant).

## Confidence Threshold and Fallback
The model calculates a confidence interval (CI) for its probability score.
*   **Confidence Threshold:** If the confidence interval is wider than ±10% (indicating high variance due to sparse historical data, such as a newly introduced train route), the UI suppresses the percentage score.
*   **UI Fallback:** Instead of a percentage, the UI displays: **"Prediction Unavailable"** with a sub-label: *"Insufficient data to calculate confirmation probability for this new route."*
*   **Technical Fallback:** If the Vertex AI endpoint fails or times out (exceeding 200ms response window), the frontend gateway catches the error and degrades gracefully by displaying the basic historical average confirmation rate stored in a local Redis cache.

## Success Metrics
*   **Model Accuracy:** Area Under the ROC Curve (AUC-ROC) >= 0.90 on test datasets.
*   **Business Impact:** Redundant waitlisted bookings (passengers booking multiple trains for the same journey) decrease by 40%.
*   **User Retention:** Net Promoter Score (NPS) for waitlist bookings rises by 25 points.

## Limitations and Risks
*   **Sudden Trend Shifts:** The model cannot anticipate sudden external factors like flash floods, sudden train cancellations, or last-minute extra coach additions.
*   **User Reliance Risk:** If the AI predicts an 85% chance of confirmation and the ticket fails to confirm, the passenger may miss their travel date.
*   **Mitigation:** The interface must feature a clear legal disclaimer: *"This prediction is an estimate based on historical trends and does not guarantee confirmation. Please make travel arrangements accordingly."*
