# Baseline Report - Daniela Avila and Klaus Krause

## 1. Prediction target
Predict whether a driver-race entry will finish in the Top-10, using only pre-race information available before the race starts.

## 2. Majority-class baseline
- Metric: F1 score (Top-10 class)
- Value: 0.0000
- Code: `DummyClassifier(strategy='most_frequent', random_state=414)`

## 3. Domain heuristic baseline
- Rule: Predict `top_10 = 1` if the driver's starting grid position is `<= 10`; otherwise predict `0`.
- Metric (same as above): 0.8583
- Code cell:

```python
def lab1_heuristic(df, threshold=10):
    """Predict Top-10 if the driver starts in grid position <= 10."""
    return (df["grid"] <= threshold).astype(int)
```

## 4. Metric choice + justification
I chose F1 score as my primary metric even though the dataset is balanced at about 50.0% positive class, because I still need a metric that forces both precision and recall to be good at the same time. A false positive means predicting a Top-10 finish for a driver who actually finishes outside the Top-10, which would overstate that driver's expected race result. A false negative means missing a real Top-10 finisher, which is also costly because the model fails to identify an actual contender. Given that both error types matter in this racing context, F1 is more informative than accuracy because it penalizes models that succeed only by favoring one side of the confusion matrix.

## 5. Leakage guard item
I checked item 2 from the leakage checklist: the train/validation/test split must be strictly temporal. In our notebook, training uses 2022-2023, validation uses only 2024 H1, and 2024 H2 stays held out as test data. No fix was needed, but verifying that split was important because a random split would leak future race patterns into training.

## 6. Baseline comparison & interpretation
(a) The domain heuristic is much harder to beat than the majority-class baseline. The majority baseline gets F1 = 0.0000 for the Top-10 class, while the grid heuristic reaches F1 = 0.8583, so starting position contains strong real signal. The remaining gap to perfect performance tells us the task is still not trivial: DNFs, overtakes, strategy, and team pace create errors that grid alone cannot explain.

(b) If a later ML model scores below the domain heuristic baseline, I would conclude that the model is not learning anything more useful than the simple expert rule. I would first re-check the validation design and feature pipeline, then try stronger pre-race features such as constructor strength, driver recent form, circuit effects, and rolling historical features before trusting a more complex model.
