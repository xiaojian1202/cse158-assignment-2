# 🏎️ F1 Pre-Race Excitement Forecaster

## 1. Project Goal
We are building a **Binary Classifier** to predict the "Entertainment Value" of a Formula 1 race *before it starts*.

**The Problem:** Casual fans often skip races because they fear they will be boring processions.
**The Solution:** A machine learning model that analyzes **Pre-Race Data** (Qualifying, Standings, Track Layout) to predict if the upcoming race will be **"Exciting" (1)** or **"Boring" (0)**.

---

## 2. Dataset
**Source:** 

[Formula 1 World Championship (1950 - 2024)](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020?resource=download)

[Weather Dataset](https://www.kaggle.com/datasets/mariyakostyrya/formula-1-weather-info-1950-2024)

**Key Files Used:**
* `qualifying.csv` (Grid positions)
* `driver_standings.csv` (Championship context)
* `circuits.csv` (Track characteristics)
* `lap_times.csv` & `status.csv` (Used *only* to generate training labels)

---

## 3. Methodology: The Supervised Pipeline

Our project is divided into two distinct engineering phases: **Label Generation** (determining the ground truth) and **Feature Engineering** (creating the inputs).

### Phase A: Defining "Excitement" (The Target Variable $Y$)
Since "Excitement" is subjective, we calculate it mathematically using post-race metrics.
* **Logic:** A race is labeled **1 (Exciting)** if it meets *either* of these criteria:
    * **High Chaos:** Top 25% of races with the most DNFs/Crashes.
    * **High Action:** Top 25% of races with the most Overtakes.
* **Otherwise:** It is labeled **0 (Boring)**.

### Phase B: Pre-Race Features (The Inputs $X$)
We train the model using **only** data available on Kaggle sets (Pre-Race). Pre-Race is defined as any type of data that can be gathered before a race starts such point difference between two drivers. We strictly avoid data leakage (e.g., we cannot use "Number of Pit Stops" because that happens *during* the race).

| Feature Name | Source File | Logic | Why it matters? |
| :--- | :--- | :--- | :--- |
| **Grid Spread** | `qualifying.csv` | Std. Dev of Q3 lap times (Top 10). | Low spread = Cars are equal speed = Close racing. |
| **Title Tension** | `driver_standings.csv` | Std. Dev of points among Top 3 drivers. | Low deviation = Close title fight = Drivers take more risks. |
| **Track Altitude** | `circuits.csv` | Numeric altitude (meters). | High altitude mean thinner air, impacting performance(Chaos). |
| **Grid Mix-Up** | `qualifying.csv` | Difference between Driver's Grid Position and Season Avg. | Fast cars starting at the back (e.g., penalties) likely suggest overtakes. |

---

## 4. Models & Evaluation
We treat this as a **Category Prediction** task (Binary Classification).

### Models
1.  **Logistic Regression:** Baseline model to check linear relationships.
2.  **Random Forest Classifier:** Main model. Handles non-linear interactions.

### Evaluation Metrics
Since "Exciting" races are the minority class (imbalanced data), **Accuracy** is a bad metric. We focus on:
* **Precision:** If we predict "Exciting," are we right? (Avoids false hype).
* **Recall:** Did we catch all the truly chaotic races?
* **F1-Score:** The harmonic mean of Precision and Recall.
