# Data Quality Log — Lab 1

---

## Issue 1: `position_text` — Non-numeric / non-classified finish codes

- **What:** The `position_text` column returned by the Jolpica API contains non-numeric strings for drivers who did not finish the race: `"R"` (retired), `"D"` (disqualified), `"W"` (withdrawn), `"F"` (failed to qualify), `"N"` (not classified). These cannot be coerced to an integer position.
- **Classification:** **MNAR (Missing Not At Random)** — a driver retiring is directly caused by a race event (mechanical failure, crash, penalty). The "missingness" of a numeric finish position is causally related to the outcome of the race, not random.
- **Impact:** If treated as truly missing and dropped, we would lose all DNF/DSQ entries and bias the dataset toward classified finishers only. For the target variable, a driver who retires definitively did NOT finish in the top 10.
- **Decision:** Keep all rows. Apply `pd.to_numeric(errors='coerce')` to convert non-numeric codes to `NaN`, then derive `top_10 = 0` for all entries where `position_num` is NaN or > 10. No rows dropped.
- **Justification:** This correctly assigns `top_10 = 0` to all DNF/DSQ drivers without information loss. Dropping these rows would introduce survivorship bias in the training data (only finishing drivers would be modeled).

---

## Issue 2: `grid = 0` — Pit-lane starters

- **What:** A small number of entries (typically 1–5 per season) have `grid = 0`, indicating the driver started from the pit lane rather than a numbered grid position. This is not a data error — it is a deliberate race event caused by late setup changes, penalties, or strategic decisions.
- **Classification:** **MAR (Missing At Random)** — the grid=0 assignment is not random (it results from specific pre-race decisions), but it is not related to the outcome in the same way MNAR would be. It is a legitimate edge case in the data, not a recording error.
- **Impact:** Including `grid = 0` as a numeric value in grid-position analyses would distort the distribution (0 is not a valid qualifying position) and artificially inflate the "grid 1–10" top-10 rates. Spearman correlations including pit-lane starters would be misleading.
- **Decision:** Flag via a binary `pit_lane_start` column. Exclude grid=0 rows from grid-position analyses (Q3, correlation analysis). Retain all rows for overall class balance and split statistics.
- **Justification:** Pit-lane starts are a real phenomenon that a production model must handle. Flagging rather than dropping preserves the information and allows downstream models to treat this as a feature.

---

## Issue 3: `points` — Post-race outcome used as reference only

- **What:** The `points` column contains championship points awarded after the race (25 for win, 18 for P2, etc.). It is perfectly correlated with finish position and is therefore a direct post-race outcome.
- **Classification:** **Type error / Leakage risk** — not a data quality issue per se, but a feature engineering trap. Using `points` as a predictor would constitute data leakage since points are only known after the race ends.
- **Impact:** A model trained with `points` as a feature would achieve near-perfect accuracy on training data but would be completely useless at prediction time (you cannot know points before the race). This is one of the most common mistakes listed in Section 9 of the lab brief.
- **Decision:** Retain in DataFrame for reference but explicitly exclude from all feature sets. Added a comment in both notebooks marking it as a post-race column.
- **Justification:** Keeping it in the raw DataFrame allows manual inspection and cross-validation of the target variable, but it is never passed to any model or heuristic as input.

---

## Issue 4: `laps` — Post-race outcome, incomplete for DNF drivers

- **What:** The `laps` column records how many laps a driver completed. For drivers who finished the race, this is the full race distance. For DNF drivers, it is the lap on which they retired — meaning it is always lower than the race total and directly encodes their retirement lap.
- **Classification:** **MNAR** for DNF drivers (the number of laps completed is causally related to the reason for retirement) and a **post-race outcome** for all drivers.
- **Impact:** Same leakage risk as `points`. Additionally, using `laps` to predict finishing position would be circular: a driver who completes fewer laps necessarily finished lower (or DNF'd). Using it would produce a model that memorizes race outcomes rather than predicting them.
- **Decision:** Exclude from all feature sets. Retained in DataFrame for reference only.
- **Justification:** No pre-race information about how many laps a driver will complete is available. Any model using `laps` as a feature would fail at inference time.

---

## Issue 5: `status` — Post-race categorical outcome with high cardinality

- **What:** The `status` column contains a free-text description of the race outcome: `"Finished"`, `"Retired"`, `"+1 Lap"`, `"Collision"`, `"Engine"`, `"Gearbox"`, etc. There are 20–30 unique values across the dataset. It is a post-race outcome.
- **Classification:** **Post-race outcome / Leakage risk.** The status is MNAR for failure modes (an engine failure is not random — it is correlated with team, car reliability, and race conditions) but is only known after the race ends.
- **Impact:** If encoded and used as a feature, `status` would perfectly predict `top_10` for most entries (any non-"Finished" status maps to top_10 = 0 in almost all cases). This is a textbook data leakage scenario.
- **Decision:** Excluded from all feature sets. Used only to understand the `position_text` NaN pattern (cross-tabulation of `status` vs `position_num` NaN confirms all NaN positions are retirements or DSQs).
- **Justification:** Retaining `status` in the raw DataFrame helps audit the data quality but it must never be a model input.

---

## Issue 6: Outliers in `grid` — Positions above 20

- **What:** In rare cases (early-season practice events, driver substitutions, or regulatory changes), more than 20 cars may be listed in qualifying results. The Ergast/Jolpica API occasionally returns grid positions up to 22–23 for some seasons. These represent real race entries but fall outside the standard 20-car grid.
- **Classification:** **Outlier / Rare legitimate value** — not a recording error, but an edge case from expanded grids or late entries.
- **Impact:** Grid positions > 20 affect the x-axis scale of grid-position plots and slightly dilute the "grid 11–20" top-10 rate calculation. There are very few such rows (<0.5% of the dataset).
- **Decision:** Keep all rows. The grid position is valid information. Plots are adjusted to show up to the maximum grid position observed, and the grid ≤ 10 threshold remains unchanged regardless of maximum grid value.
- **Justification:** Dropping these rows would discard real race data. The heuristic threshold (grid ≤ 10) is unaffected since no driver in position > 20 would be predicted top-10.
