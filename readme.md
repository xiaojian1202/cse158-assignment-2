# 🏎️ F1 Race DNA Recommender System

## 1. Project Goal
We are building a **Content-Based Recommender System** for Formula 1 races.

Unlike standard "Winner Prediction" models, our goal is to quantify the "Vibe" or "DNA" of every race from 2000–2023. We treat races as items with distinct statistical personalities (e.g., "Chaotic," "Strategic," "Boring") and recommend similar races to users.

**The User Story:**
> *"I loved the 2021 Abu Dhabi GP because of the chaos and last-lap drama. What other historical races should I watch?"*

---

## 2. Data Sources
We use the **Formula 1 World Championship** dataset (sourced from Kaggle/Ergast API).
Ensure these 5 CSV files are in your `data/` folder:

| File Name | Description | Used For |
| :--- | :--- | :--- |
| **`races.csv`** | Metadata (Date, Year, Circuit). | **Indexing** (Grouping results by Race ID). |
| **`results.csv`** | Finishing positions & time gaps. | **Tension Metric** (Gap between P1 & P2). |
| **`status.csv`** | Explains why a driver stopped. | **Chaos Metric** (Identifying crashes). |
| **`lap_times.csv`** | Lap-by-lap positions. | **Action Metric** (Counting overtakes). |
| **`pit_stops.csv`** | Pit stop timings. | **Strategy Metric** (Avg stops per driver). |

---

## 3. Metric Implementation Guide (Formulas & Logic)

### 🔴 Metric A: The Chaos Score
**Definition:** A measure of disruption. High score = Demolition Derby. Low score = Clean race.
* **Data Source:** `results.csv` merged with `status.csv`.
* **The Logic:** We count **DNFs** (Did Not Finish). A "Normal" finish is status ID `1` (Finished) or `11-19` (+1 Lap, +2 Laps, etc.). Anything else is a DNF (Crash, Engine Failure).
* **Formula:**
    $$\text{Chaos Score} = \frac{\text{Count of Non-Finishers}}{\text{Total Drivers}}$$
* **Pandas Implementation:**
    ```python
    # 1. Filter out normal finishes (IDs 1, 11, 12... usually represent finishing)
    # Note: Check status.csv to confirm IDs. Usually 1=Finished.
    dnf_df = df[~df['statusId'].isin([1, 11, 12, 13, 14, 15])]
    chaos_score = dnf_df.groupby('raceId')['driverId'].count() / total_drivers
    ```

### 🟢 Metric B: The Action Score
**Definition:** A measure of on-track volatility. High score = Constant position changes.
* **Data Source:** `lap_times.csv`.
* **The Logic:** We calculate how many times drivers changed position compared to the previous lap.
* **Formula:**
    $$ \text{Action} = \sum_{\text{lap}=2}^{\text{last}} \sum_{\text{driver}} | \text{Pos}_{\text{lap}} - \text{Pos}_{\text{lap}-1} | $$
* **Pandas Implementation:**
    ```python
    # 1. Sort by Driver and Lap
    df = df.sort_values(['raceId', 'driverId', 'lap'])
    # 2. Calculate difference from previous row
    df['pos_change'] = df.groupby(['raceId', 'driverId'])['position'].diff().abs()
    # 3. Sum all changes per race
    action_score = df.groupby('raceId')['pos_change'].sum()
    ```

### 🔵 Metric C: The Tension Score
**Definition:** A measure of the winning margin. High score = Photo finish.
* **Data Source:** `results.csv`.
* **The Logic:** Find the time gap between 1st and 2nd place. We take the inverse (1/gap) so that small gaps equal high scores.
* **Formula:**
    $$ \text{Tension} = \frac{1000}{\text{Time Gap (ms)}} $$
* **Pandas Implementation:**
    ```python
    # 1. Filter for P1 and P2 only
    top2 = df[df['positionOrder'].isin([1, 2])]
    # 2. Pivot so P1 and P2 are in columns, then subtract milliseconds
    # Note: Handle NaN if P2 was lapped (no time gap)
    ```

### 🟣 Metric D: The Strategy Score
**Definition:** A measure of strategic activity. High score = 3-stop race. Low score = 1-stop race.
* **Data Source:** `pit_stops.csv`.
* **The Logic:** Calculate the average number of stops per driver.
* **Formula:**
    $$ \text{Strategy} = \frac{\text{Total Pit Stops in Race}}{\text{Total Drivers in Race}} $$
* **Pandas Implementation:**
    ```python
    total_stops = pit_stops.groupby('raceId')['stop'].count()
    strategy_score = total_stops / num_drivers
    ```

---

## 4. Modeling Strategy
Once we generate these 4 numbers for every race, we will use **Cosine Similarity**.

1.  **Normalize:** Scale all metrics to 0-1 (so "Chaos" doesn't overpower "Tension").
2.  **Vectorize:** Every race becomes a vector: `[0.8, 0.2, 0.9, 0.1]`.
3.  **Recommend:** Calculate the distance between the User's Input Race and all other races. Return the top 5 closest matches.