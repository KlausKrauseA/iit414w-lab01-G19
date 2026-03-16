# IIT414W — Lab 01 | Group 19

## Overview

Exploratory Data Analysis (EDA) lab.
Course: IIT414W | Group: G19

---

## Requirements

- [Conda](https://docs.conda.io/en/latest/miniconda.html) (Miniconda or Anaconda)
- Git

---

## Reproduce results (< 10 minutes)

```bash
# 1. Clone the repository
git clone <repo-url>
cd iit414w-lab01-G19

# 2. Create and activate the environment
conda env create -f environment.yml
conda activate iit414w-lab01

# 3. Launch Jupyter
jupyter notebook
```

Open `eda.ipynb` and `baseline.ipynb` from the repository root and run all cells top to bottom (**Kernel → Restart & Run All**).

---

## Repository structure

```
iit414w-lab01-G19/
├── eda.ipynb           # Full decision-oriented EDA notebook
├── baseline.ipynb      # Domain heuristic baseline + optional stretch metrics/model
├── environment.yml     # Conda environment specification
├── DATA_QUALITY_LOG.md # Structured log of all data quality issues found and decisions made
├── README.md           # This file
├── PROMPTS.md          # AI usage documentation
└── .gitignore
```

---

## Group members

| Name |
|------|
| Daniela Ávila |
| Klaus Krause |

---

## Notes

- All notebooks were tested on the environment pinned in `environment.yml`.
- Data files are excluded from version control via `.gitignore`. Place raw data in `data/` before running notebooks.
