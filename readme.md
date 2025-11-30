# 🏎️ F1 Race DNA Recommender System

## 1. Project Goal
We are building a **Content-Based Recommender System** for Formula 1 races. 

Unlike standard "Winner Prediction" models, our goal is to quantify the "Vibe" or "DNA" of every race. We treat races as items with distinct statistical personalities (e.g., "Chaotic," "Strategic," "Boring") and recommend similar races to users based on these features.

**The User Story:**
> *"I loved the 2021 Abu Dhabi GP because of the chaos and last-lap drama. What other historical races should I watch?"*

---

## 2. Data Source
**Dataset:** [Formula 1 World Championship (1950 - 2024)](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020?resource=download)

**Setup Instructions:**
1. Download the dataset from the link above.
2. Unzip the files.
3. Place the `.csv` files into a folder named `data/` in your project root.

### Schema & Usage
We only need 5 specific files from the dataset to build our metrics:

| File Name | Description | Metric Usage |
| :--- | :--- | :--- |
| **`races.csv`** | Race metadata (Date, Year, Circuit). | **Indexing:** Groups results by `raceId`. |
| **`results.csv`** | Finishing positions & time gaps. | **Tension Metric:** Calculates gap between P1 & P2. |
| **`status.csv`** | Explains `statusId` (e.g., Collision, Engine). | **Chaos Metric:** Identifies crashes vs. normal finishes. |
| **`lap_times.csv`** | Lap-by-lap positions for drivers. | **Action Metric:** Counts total overtakes. |
| **`pit_stops.csv`** | Pit stop timings and counts. | **Strategy Metric:** Calculates avg. stops per driver. |

---

## 3. Metric Implementation Guide (Formulas & Logic)

To capture the "DNA" of a race, we will engineer 4 custom scores.

### 🔴 Metric A: The Chaos Score
**Definition:** A measure of disruption. High score = Demolition Derby. Low score = Clean race.
* **Logic:** We count **DNFs** (Did Not Finish). A "Normal" finish usually has a `statusId` of 1 (Finished) or 11-19 (+1 Lap). Any other status implies a crash or mechanical failure.
* **Formula:**
    $$\text{Chaos Score} = \frac{\text{Count of Non-Finishers}}{\text{Total Drivers}}$$

### 🟢 Metric B: The Action Score
**Definition:** A measure of on-track volatility. High score = Constant position changes.
* **Logic:** We calculate the absolute difference in position for every driver between Lap $N$ and Lap $N-1$.
* **Formula:**
    $$\text{Action} = \sum_{\text{lap}=2}^{\text{last}} \sum_{\text{driver}} | \text{Pos}_{\text{lap}} - \text{Pos}_{\text{lap}-1} |$$
* **Note:** The `lap_times.csv` file is large (~500k rows). Filter by `raceId` before processing to save memory.

### 🔵 Metric C: The Tension Score
**Definition:** A measure of the winning margin. High score = Photo finish.
* **Logic:** Find the time gap (in milliseconds) between 1st Place and 2nd Place. We take the inverse (1000 / gap) so that smaller gaps result in higher scores.
* **Formula:**
    $$\text{Tension} = \frac{1000}{\text{Time Gap (ms)}}$$

### 🟣 Metric D: The Strategy Score
**Definition:** A measure of strategic activity. High score = Multi-stop race.
* **Logic:** Calculate the average number of pit stops per driver.
* **Formula:**
    $$\text{Strategy} = \frac{\text{Total Pit Stops in Race}}{\text{Total Drivers}}$$

---

## 4. Modeling Strategy
Once we generate these 4 numbers for every race, we will use **Cosine Similarity**.

1.  **Normalize:** Scale all metrics to 0-1 (so "Chaos" doesn't overpower "Tension") using `MinMaxScaler`.
2.  **Vectorize:** Every race becomes a vector: `[0.8, 0.2, 0.9, 0.1]`.
3.  **Recommend:** Calculate the distance between the User's Input Race and all other races. Return the top 5 closest matches.
