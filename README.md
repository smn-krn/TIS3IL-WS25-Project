# Time Series Forecasting Project — Meteostat Daily Weather Data
TIS3IL – University Project (Deadline: 31.12.)

# Team
**Last Destination - this project will end us all**

Nr of Members: 3

**CB** - preprocessing, baseline, statsmodels

**AS** - ml-models, visualizations

**SK** - neural-models, errors, splitting

---

# Project Overview
This repository contains our full workflow for a university project on time series forecasting.  
The objective is to download, clean, and forecast daily weather indicators (temperature, precipitation, wind speed, etc.) using the Meteostat Daily Bulk API:

https://dev.meteostat.net/bulk/daily.html

The project includes:

- Data ingestion from Meteostat (daily CSV files)
- Preprocessing & feature engineering
- Multiple forecasting model families, which are not yet fixed. 
- Full evaluation on a proper train/validation/test split
- Visualizations & business-style interpretation
- Structured documentation

---

## Repository Structure

```
TIS3IL-WS25-PROJECT
│
├── .venv/
│
├── code/
│   ├── 01-data-prep.ipynb
│   ├── 02-baseline.ipynb
│   ├── 02-ml-models.ipynb
│   ├── 02-neural-models.ipynb
│   ├── 02-stats-models.ipynb
│   ├── 03-visualization.ipynb
│   ├── metrics.ipynb
│   ├── splitting.ipynb
│   └── test.ipynb
│
├── data/
│   ├── models/
│   │   ├── baseline_results.csv
│   │   ├── ml_models_results.csv
│   │   ├── neural_models_results.csv
│   │   └── statsmodels_results.csv
│   │
│   └── processed/
│       └── data.csv
│
├── docs/
│   ├── reports/
│   │   ├── presentation/
│   │   │   └── slides.pptx
│   │   ├── doc_models.md
│   │   ├── doc_preprocessing.md
│   │   ├── doc_visualizations.md
│   │   └── summary.md
│   │
│   └── visualizations/
│       └── placeholder.txt
│
├── .gitignore
├── .python-version
├── pyproject.toml
├── README.md
└── uv.lock
```


---

# Dataset — Meteostat Daily Weather Data

We use the Meteostat Daily Bulk dataset, which provides:

- Min / max / average temperature
- Precipitation
- Snow depth
- Wind speed
- Sun hours
- Atmospheric pressure

Advantages:

- Public and free  
- Provided in CSV format  
- Updates daily  
- Covers thousands of weather stations worldwide  
- Perfect for classical → ML → neural forecasting

We will select one or several stations for the project, depending on coverage and data quality.

---

# Setup Instructions

## Clone the repository
```bash
git clone https://github.com/smn-krn/TIS3IL-WS25-Project.git
cd TIS3IL-WS25-Project
```

## Project Dependencies & Environment Setup (using uv)

This project uses `uv` + `pyproject.toml` to manage Python dependencies.
Unlike pip or conda, uv supports dependency groups, which allow us to install only what we need for each part of the project (baseline models, ML models, neural models, visualizations, etc.).

This keeps environments small, fast, conflict-free, and makes collaboration easier.

------------------------------------------------------------
1. Installing Dependencies
------------------------------------------------------------

Install only the base dependencies (recommended for general users):

    uv sync

This installs only what is defined under:

    [project]
    dependencies = [...]

------------------------------------------------------------
Install dependencies for a specific model family
------------------------------------------------------------

ML models:

    uv sync --group ml

Neural forecasting models (PyTorch + NeuralForecast + TiReX):

    uv sync --group neural

Classical statistical models:

    uv sync --group stats

Visualizations:

    uv sync --group viz

------------------------------------------------------------
Install multiple dependency groups
------------------------------------------------------------

    uv sync --group ml --group neural

------------------------------------------------------------
Install everything (not recommended unless necessary)
------------------------------------------------------------

    uv sync --all-groups

------------------------------------------------------------
1. Adding New Dependencies
------------------------------------------------------------

Dependencies are tracked inside pyproject.toml.

Add a package to the base dependencies:

    uv add pandas

Add a package to a specific group:

Example: add Prophet to stats models:

    uv add prophet --group stats

Example: add TorchMetrics to neural models:

    uv add torchmetrics --group neural

Example: add seaborn extensions to viz tools:

    uv add seaborn ruff --group viz

------------------------------------------------------------
1. Checking Installed Packages
------------------------------------------------------------

List everything currently installed:

    uv pip list

See which groups are available:

    uv tree

------------------------------------------------------------
Summary Table
------------------------------------------------------------

Task                                 | Command
------------------------------------ | -----------------------------------------------
Install base deps                    | uv sync
Install ML deps                      | uv sync --group ml
Install neural deps                  | uv sync --group neural
Install stats deps                   | uv sync --group stats
Install viz tools                    | uv sync --group viz
Add package to base                  | uv add PACKAGE
Add package to a group               | uv add PACKAGE --group GROUPNAME
Install multiple groups              | uv sync --group GROUP1 --group GROUP2


------------------------------------------------------------


We are using Python **3.12.11**

So the nixtla libraries will hopefully not be an issue.

**That is all you need to do for the setup!**


# Branch Instructions

Make sure you are in the project folder.

## Check current branch
```bash
git branch
```
 should be *main and maybe more.

if you're not on main (so if * is on a different branch), switch via 
```bash 
git checkout main
```

## Pull the latest changes from origin
```bash
git pull origin main
```

## Create a new feature branch
```bash
git checkout -b <feature-name>
```

-b means: create and switch to the new branch.

**Naming convention:** 

[FirstLetterOfFirstName][FirstLetterOfLastName]-[FeatureAdded]


EXAMPLE: CB-Preprocessing or AS-Visualizations

## First push
```bash
git push -u origin <feature-name> 
```

Do this before any changes, because the very first push requires special handling using the "-u origin" feature. 

## All further pushes

After this, you can treat it normally using 
```bash 
git add . 
git commit -m "Message"
git push
```

## Merging

Merge via github website

1. Go to the repo on GitHub
2. Open the Pull Requests tab
3. GitHub usually suggests one automatically

If it doesn't 

1. Go to the repository, tab ```Pull Requests```
2. It should suggest one but if it doesn't click ```new pull request```
   
   ```base: main <- compare: [your-feature-branch]```

Should be able to switch without merging conflicts. If you have merging conflicts... pray.

## Switching to main

We first need to pull this from the repo so that we have this merge locally.

```bash
git fetch
git pull
```

now we can switch to the main

```bash
git checkout main
git pull origin main
```