# 🏎️ F1 Race DNA Recommender System

## 1. Project Goal
We are building a **Content-Based Recommender System** for Formula 1 races. 

Unlike standard "Winner Prediction" models, our goal is to quantify the "Vibe" or "DNA" of every race from 2000–2023. We treat races as items with distinct statistical personalities (e.g., "Chaotic," "Strategic," "Boring") and recommend similar races to users.

**The User Story:** > *"I loved the 2021 Abu Dhabi GP because of the chaos and last-lap drama. What other historical races should I watch?"*

---

## 2. Data & Schema
We use the **Formula 1 World Championship** dataset (sourced from Kaggle/Ergast API). 
Ensure the following CSV files are placed in the `data/` directory:

| File Name | Description | Used For |
| :--- | :--- | :--- |
| **`races.csv`** | Metadata for every GP (Date, Circuit Name, Year). | **Indexing** (Grouping results by Race ID). |
| **`results.csv`** | Final finishing positions, points, and time gaps. | **Tension Metric** (Calculating the time gap between P1 and P2). |
| **`status.csv`** | Explains the `statusId` (e.g., "Collision", "Engine Failure"). | **Chaos Metric** (Identifying crashes vs. normal finishes). |
| **`lap_times.csv`** | Lap-by-lap position for every driver. | **Action Metric** (Counting overtakes/position changes). |
| **`pit_stops.csv`** | Timing and count of pit stops per driver. | **Strategy Metric** (Calculating avg. stops per race). |
| **`drivers.csv`** | Maps `driverId` to names (e.g., "Lewis Hamilton"). | **Visualization** (Showing user-friendly names). |

---

## 3. Architecture & Setup

### Tech Stack
* **Language:** Python 3.9+
* **Libraries:** `pandas` (Data Engineering), `scikit-learn` (Cosine Similarity), `matplotlib` (Plots)
