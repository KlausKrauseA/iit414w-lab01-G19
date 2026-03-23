# AI Usage Documentation — IIT414W Lab 02 | Group 19

---

## Entry 1 — 2026-03-23 — Lab 02 notebook and supporting files

**Context:** After completing Lab 1 (EDA + Baseline), we needed to build Lab 2: a feature engineering notebook with ≥3 new pre-race features, a trained model (LogisticRegression and DecisionTree), a leakage guard checklist for each feature, error analysis, and a comparison_table.md against Lab 1 baselines.

**Prompt(s):**
> "Ayúdame a hacer la parte de Lab 2, analiza bien las instrucciones de las imágenes que son lo que indica que hacer ahora con lo que tengo que es el Lab 2, por que ahora tenemos la base de lo que hicimos en lab 1. Considera la rúbrica y que se cumplan todos los puntos como se debe."

**Output:** Claude Code generated:
- `lab02_feature_engineering.ipynb` (10 sections, ~35 cells) with:
  - Same paginated Jolpica API loader and temporal split as Lab 1
  - 5 engineered features: `driver_prev_grid` (lag), `driver_rolling_top10_rate` (rolling), `driver_circuit_top10_rate` (interaction), `constructor_rolling_top10_rate` (rolling team), `constructor_tier` (categorical encoding)
  - Per-feature leakage guard checklist (timing, computation, target isolation)
  - Logistic Regression + Decision Tree trained on engineered features
  - Comparison against Lab 1 majority-class and heuristic baselines (F1, Precision, Recall, ROC-AUC)
  - Confusion matrices for heuristic vs best model
  - Error analysis: 3 failure modes (FP pattern, FN pattern, circuit-level concentration)
  - Feature importance via LR coefficients
  - Programmatic write of `comparison_table.md`
  - 1-3-1 summary cell
- `comparison_table.md`: standalone comparison table with feature inventory
- `PROMPTS.md` (this file): AI usage documentation
- `README.md`: reproduction instructions

**Validation:**
- All features use `.shift(1)` before any aggregation — current-race `top_10` never used ✓
- Constructor tier fitted on train set only, mapped to val/test without future leakage ✓
- NaN imputation uses training-set median for lag feature, 0.5 prior for rate features ✓
- Same temporal split assertions as Lab 1 ✓
- `RANDOM_SEED = 414` in setup cell ✓
- Primary metric = F1 (consistent with `baseline_report.md`) ✓
- ≥3 feature types covered: lag (F1), rolling (F2, F4), interaction (F3), categorical encoding (F5) ✓
- ≥2 Lab 1 baselines in comparison table ✓

**Adaptations:**
- Used `expanding().mean()` for the driver×circuit feature (not a fixed window) because some circuit/driver combinations have very few historical visits, so a fixed rolling window would produce too many NaNs.
- Constructor tier uses training-set average points as a proxy for team budget/car quality — this is a common approach in F1 prediction but relies on the assumption that team hierarchy is stable across seasons. The tier assignment is frozen from training data to avoid leakage.
- NaN imputation for rolling/rate features defaults to 0.5 (prior probability near class balance) rather than 0 or 1, which would bias the model toward one class for new drivers.

**What we understood:** Lag features capture momentum (a driver who qualified well recently is likely to do so again). Rolling aggregates are a form of exponential smoothing over a fixed window. Interaction features (driver×circuit) capture track specialisation. Categorical encoding converts a nominal variable (constructor name) into an ordinal integer by learning the mapping from training data, which is valid as long as the mapping is never updated using validation or test data.

**What we did not fully understand yet:** Why Decision Tree sometimes outperforms Logistic Regression on F1 despite having a smaller ROC-AUC — the AI clarified that LR optimises log-loss (a probability objective) while DT optimises a threshold-based split criterion, so F1 (a threshold metric) can favor DT while AUC (a ranking metric) favours LR. We will revisit tree-based ensemble methods in later weeks.

**Final Decision:** Used — all sections accepted after structural review against the lab rubric. Metric values will be validated when running on the live environment.

---

<!-- Add new entries below as you continue to use AI tools in Lab 02 -->
