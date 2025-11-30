# 🏎️ F1 Race DNA Recommender System

## 1. Project Goal
We are building a **Content-Based Recommender System** for Formula 1 races. 

Unlike standard "Winner Prediction" models, our goal is to quantify the "Vibe" or "DNA" of every race from 2000–2023. We will treat races as items with distinct statistical personalities (e.g., "Chaotic," "Strategic," "Boring") and recommend similar races to users.

**The User Story:** > *"I loved the 2021 Abu Dhabi GP because of the chaos and last-lap drama. What other historical races should I watch?"*

---

## 2. Architecture & Setup

### Tech Stack
* **Language:** Python 3.9+
* **Data Source:** [Ergast API](http://ergast.com/mrd/) (via Kaggle CSV dumps or `fastf1` library)
* **Key Libraries:** `pandas`, `scikit-learn` (for Cosine Similarity), `matplotlib`

### Installation
```bash
# Clone the repo
git clone <your-repo-url>

# Install dependencies
pip install pandas scikit-learn fastf1 matplotlib