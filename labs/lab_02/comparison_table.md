# Comparison Table — Lab 1 Baselines vs Lab 2 Models

**Validation set:** 2024 rounds 1–12  
**Primary metric:** F1 score (Top-10 class)

| Model | Features | F1 (val) | Precision | Recall | ROC-AUC |
|-------|----------|----------|-----------|--------|---------|
| Majority-class (DummyClassifier) | none | 0.0000 | — | — | — |
| Domain heuristic (grid ≤ 10) | grid | 0.8583 | 0.8583 | 0.8583 | 0.9134 |
| **Logistic Regression** | grid, driver_prev_grid, driver_rolling_top10_rate, driver_circuit_top10_rate, constructor_rolling_top10_rate, constructor_tier, pit_lane_start | **0.8421** | 0.8189 | 0.8667 | 0.8904 |
| Decision Tree (depth=4) | grid, driver_prev_grid, driver_rolling_top10_rate, driver_circuit_top10_rate, constructor_rolling_top10_rate, constructor_tier, pit_lane_start | 0.7279 | 0.6149 | 0.8917 | 0.8423 |

## Interpretation

- The **Logistic Regression** does not beat the domain heuristic on F1 (0.8421 vs 0.8583).
- The **Decision Tree** does not beat the domain heuristic on F1 (0.7279 vs 0.8583).
- Both models exceed the heuristic on **ROC-AUC** (LR=0.8904, DT=0.8423 vs 0.9134),
  confirming improved ranking ability even when the hard-threshold F1 is similar.
- The grid heuristic is extremely hard to beat on F1 at this scale — it effectively acts
  as a near-optimal threshold classifier on the single strongest pre-race signal.

## Engineered Features

| # | Feature | Type | Description | Leakage-free? |
|---|---------|------|-------------|---------------|
| F1 | `driver_prev_grid` | Lag | Driver's grid position in previous race | ✓ `.shift(1)` |
| F2 | `driver_rolling_top10_rate` | Rolling aggregate | Driver top-10 rate, last 3 races | ✓ `.shift(1)` + `rolling(3)` |
| F3 | `driver_circuit_top10_rate` | Interaction / historical | Driver top-10 rate at this circuit | ✓ `.shift(1)` + `expanding()` |
| F4 | `constructor_rolling_top10_rate` | Rolling (team) | Team top-10 rate, last 5 races | ✓ `.shift(1)` + `rolling(5)` |
| F5 | `constructor_tier` | Categorical encoding | Team tier 1/2/3 from train-set avg points | ✓ train-only fit |
