# AI Usage Documentation — IIT414W Lab 01 | Group 19

---

## Entry 1 — 2026-03-15 — Repository scaffolding

**Context:** We needed to set up the required repository files (`.gitignore`, `environment.yml`, `README.md`, `PROMPTS.md`) before starting the lab work.

**Prompt(s):**
> "Your GitHub repository must contain exactly these files: PROMPTS.md, README.md, environment.yml, .gitignore — [followed by the format requirements]"

**Output:** Claude Code generated all four files with standard content for a Python/Jupyter EDA project: a `.gitignore` excluding cache and checkpoint files, a `conda` environment pinned to Python 3.11 with pandas, numpy, matplotlib, seaborn and scipy, a README runbook with setup instructions, and this PROMPTS.md file.

**Validation:** Files were inspected manually to confirm they matched the required format and contained sensible defaults for an EDA project.

**Adaptations:** Group member names and student IDs in `README.md` must be filled in. The `environment.yml` may need additional packages depending on the specific datasets or methods used in the lab.

**Final Decision:** Partially used — the structure was accepted as-is; package list and README details will be updated to reflect actual lab content.

---

---

## Entry 2 — 2026-03-15 — EDA notebook structure and decision-oriented analysis

**Context:** We needed to build `eda.ipynb` from scratch following the decision-oriented format (Question → Plot → Interpretation → Decision) with all required sections: class balance, temporal analysis, 5+ research questions, correlation analysis, trap awareness, temporal split, and 1-3-1 summary.

**Prompt(s):**
> "sigamos con eda nomas [...] dime que queda por hacer y despues te dire con cual de las tareas partimos" — followed by sharing the full lab brief and existing repo state.

**Output:** Claude Code generated the complete `eda.ipynb` (37 cells) covering: setup with `RANDOM_SEED = 414`, Jolpica API data loading for 2022–2024, target variable creation, data quality audit with MCAR/MAR/MNAR classification, 6 research questions (class balance, temporal stability, grid vs top-10, constructor dominance, driver consistency, constructor temporal shift), Spearman/Cramér's V/point-biserial correlation table for 6 features, survivorship bias trap check, programmatic leakage-verified temporal split (train=2022–2023, val=2024 H1, test=2024 H2), and the 1-3-1 summary cell.

**Validation:** We reviewed the cell-by-cell structure against the rubric checklist. Key checks:
- All 5+ required sections present (3.1–3.8 ✓)
- `RANDOM_SEED = 414` in first code cell ✓
- No post-race features (`points`, `laps`, `status`, `position_num`) used as features ✓
- Temporal split uses `assert` statements for programmatic leakage verification ✓
- 1-3-1 summary is decision-oriented (action-verb headline, 3 evidence points, 1 action) ✓

**Adaptations:** The survivorship bias example (Section 4) was chosen over spurious correlation because it directly connects to a real feature engineering risk in our dataset (driver historical rates). The correlation section uses three different measures (Spearman, Cramér's V, point-biserial) rather than only Pearson, because the features are of mixed types — Pearson would be inappropriate for categorical features.

**Final Decision:** Used — notebook accepted after structural review. Plots and interpretation will be validated when running on the actual environment with real API data.

---

## Entry 3 — 2026-03-15 — Baseline notebook with stretch metrics and sklearn

**Context:** We needed to build `baseline.ipynb` with: (1) a rule-based domain heuristic with no ML code, (2) accuracy on the validation set, (3) a reflection on accuracy limitations, (4) a lower bound statement, and optionally (5) stretch metrics (Precision, Recall, F1, ROC-AUC) and (6) a second sklearn baseline.

**Prompt(s):**
> "baseline" — after completing eda.ipynb, asked to proceed with the baseline notebook.

**Output:** Claude Code generated `baseline.ipynb` (26 cells) with: same API loading pipeline, temporal split with leakage assertions, an explicit feature audit (pre-race vs post-race), the heuristic rule `grid ≤ 10 → predict top-10`, accuracy computation with confusion matrix, comparison against trivial baselines (always predict top-10 ≈ 50%), a reflection section on what accuracy hides, a lower bound statement, and stretch work: `precision_score`, `recall_score`, `f1_score`, `roc_auc_score` from sklearn, a proper ROC curve using raw grid score (negated), a `DummyClassifier` comparison, and a `LogisticRegression` trained on the single `grid` feature.

**Validation:**
- Confirmed the heuristic uses only `grid` (pre-race) — no post-race features ✓
- Accuracy is computed on the **validation set** (2024 H1), not training set ✓
- `DummyClassifier` provides the trivial 50% baseline for comparison ✓
- ROC curve uses `-grid` as a continuous score (lower grid = higher probability of top-10), which is the correct way to draw a proper curve rather than a single point ✓

**Adaptations:** We added the ROC curve using the raw (negated) grid score as a continuous predictor, which gives a more informative curve than just plotting the single heuristic threshold point. This distinction — between a hard threshold and a probabilistic scorer — was not obvious from the Burkov Ch. 2 reading alone; the AI explanation clarified the difference between `predict()` and `predict_proba()` outputs.

**What we understood:** Precision = "of the ones we said were top-10, how many actually were?"; Recall = "of all actual top-10s, how many did we catch?"; F1 = harmonic mean balancing both; ROC-AUC = area under the receiver operating characteristic curve, measures ranking ability regardless of threshold. We understand that AUC=0.5 means random ranking and AUC=1.0 means perfect ranking.

**What we did not fully understand yet:** Why `roc_auc_score` with binary predictions (0/1) gives a different value than with probabilities — the AI explained this is because with binary predictions the curve has only 3 points (origin, one threshold point, corner), while probabilities trace a full curve. We will revisit this in Week 3–4.

**Final Decision:** Used — both required and stretch sections accepted. Metric justification cell written in our own words with explicit acknowledgment of uncertainty, as required by the rubric.

<!-- Add new entries below as you use AI tools during the lab -->
