# ⚽ FIFA World Cup 2026 — Match Prediction System

> **Dixon-Coles Poisson model with domain adjustment and deterministic bracket resolution**  
> Predicting all 104 matches across the group stage and knockout rounds before a ball is kicked.

---

## 📌 Overview

This project builds an end-to-end football match prediction system for the **2026 FIFA World Cup** — the first edition to feature 48 teams and 104 matches. Built for the [DataCamp WC2026 Prediction Competition](https://www.datacamp.com), the system predicts scorelines, corners, cards, match winners, and penalty shootout outcomes for every fixture.

The competition scoring system rewards precision (exact scorelines, correct winners) with multipliers increasing in later rounds — making a robust model that holds up through the knockout stages critical to a strong final score.

**Final Prediction: 🏆 Argentina wins the World Cup, defeating Spain 2–1 in the Final.**

---

## 🧠 Methodology

### 1. Dixon-Coles Poisson Model
The foundation is the **Dixon-Coles (1997)** model — a well-established framework in football analytics. It models goals scored by each team as independent Poisson processes parameterised by:

- **Attack strength** — how many goals a team scores relative to average
- **Defense strength** — how many goals a team concedes relative to average  
- **Home advantage** — estimated from historical data
- **ρ (rho) correction** — adjusts for the real-world over-occurrence of low-scoring draws (0-0, 1-0, 0-1, 1-1)

Parameters are fitted using **maximum likelihood estimation** on historical international match data, with exponential time-weighting to prioritise recent fixtures.

### 2. Elo Blend
Raw Dixon-Coles attack/defense parameters are blended with **Elo ratings** to capture tournament-level quality signals that goal-count data alone may miss — particularly for teams whose recent results come predominantly against weak opposition.

### 3. Domain Adjustment Layer
A manual override layer applies confederation-aware corrections:

- **Elo inflation correction** — teams rated highly from AFCON, AFC, or CONCACAF regional competition receive attack/defense debuffs reflecting the quality gap at World Cup level
- **Tournament debutant penalty** — first-time World Cup participants with no major tournament pedigree
- **Host nation boost** — USA receive a modest uplift reflecting home advantage and squad quality underrepresented in the data
- **Elite team calibration** — Spain, Argentina, France, Brazil, Germany tuned against recent tournament results

### 4. Score-Point-Optimised Predictions
Scorelines are not chosen as the **most likely** outcome but as the scoreline that **maximises expected competition points** given the full probability distribution:

| Correct prediction | Points |
|---|---|
| Exact scoreline | 25 pts |
| Correct goal difference | 10 pts |
| Correct total goals | 10 pts |
| Correct match winner (group) | 40 pts |
| Correct match winner (knockout) | 20 pts |

This means a slightly less probable scoreline that strongly signals the correct winner can outperform a "safe" 1-1 draw in expected points — the optimizer accounts for this across the full score matrix.

### 5. Deterministic Bracket Resolution
Group standings, best third-place qualifiers, and the full knockout bracket are resolved **deterministically from predicted scorelines** rather than Monte Carlo averages — ensuring every team that advances does so based on exactly the scores being submitted. Best third-place assignments use **bipartite matching** (scipy `linear_sum_assignment`) to guarantee all 8 slots are filled without greedy pool exhaustion.

---

## 📊 Results

### Predicted Tournament Path

| Round | Match | Score | Winner |
|---|---|---|---|
| Final | Spain vs Argentina | 1 – 2 | 🏆 Argentina |
| Semi-final | Brazil vs Spain | 2 – 2 (aet) | Spain |
| Semi-final | France vs Argentina | 1 – 1 (aet) | Argentina |
| Quarter-final | Germany vs Brazil | 2 – 2 (aet) | Brazil |
| Quarter-final | Spain vs Belgium | 2 – 2 (aet) | Spain |
| Quarter-final | France vs Mexico | 1 – 1 (aet) | France |
| Quarter-final | Portugal vs Argentina | 1 – 1 (aet) | Argentina |

### Notable Group Stage Predictions
- **Group H**: Spain top the group ahead of Uruguay
- **Group J**: Argentina win comfortably; Algeria exit in the group stage
- **Group D**: Türkiye and USA advance ahead of Australia
- **Group K**: Portugal and Colombia qualify; Uzbekistan eliminated (1pt)

---

## 🛠️ Setup & Usage

### Requirements
```bash
pip install pandas numpy scipy scikit-learn
```

### Data
Historical international match data is fetched directly in the notebook. No external files needed beyond the competition-provided fixture lists.

### Running the Notebook
Open `notebook.ipynb` and run cells **top to bottom** in order:

1. **Data ingestion** — fetch and clean historical match data
2. **Elo computation** — build team Elo ratings from match history
3. **Dixon-Coles fitting** — MLE on attack/defense/home parameters
4. **Domain adjustments** — apply manual override layer
5. **Group stage predictions** — score-optimised scorelines for all 72 group matches
6. **Standings resolution** — derive group tables and best third-place qualifiers from predicted scorelines
7. **Knockout bracket** — resolve bracket slots and predict all 32 KO matches
8. **Submission export** — generate final CSV

> ⚠️ The Monte Carlo simulation cell (50,000 runs) is commented out — it is no longer needed for bracket resolution and significantly increases runtime. Re-enable if you want probabilistic finish percentages for analysis.

---

## 📁 Project Structure

```
├── notebook.ipynb          # Main prediction notebook
├── README.md               # This file
```

---

## ⚠️ Limitations

- **No in-tournament data** — all predictions are locked in before kick-off; injuries, suspensions, and team news are not factored in
- **Historical data bias** — teams with limited international fixture history (debutants, smaller confederations) have noisier parameter estimates; partially corrected by the domain adjustment layer but not eliminated
- **Poisson independence assumption** — Dixon-Coles assumes goals within a match are independent after the low-score correction; in reality, game state (score, time, red cards) affects goal rates in ways the model cannot capture
- **Deterministic bracket** — using a single predicted scoreline per match means upsets are not propagated; a probabilistic bracket would better reflect uncertainty but would be inconsistent with the submitted scorelines
- **Manual overrides are subjective** — the domain adjustment layer introduces domain knowledge that improves intuitive validity but is not cross-validated; different assumptions would produce different brackets

---

## 👤 Author

**Israel Aiyegbeni (Eazi)**  
Data Scientist | Lagos, Nigeria  
B.Sc. Agricultural & Environmental Engineering, University of Ibadan  

[LinkedIn](https://linkedin.com/in/) · [GitHub](https://github.com/)
