# Time Series Forecasting Project — Meteostat Daily Weather Data
TIS3IL – University Project (Deadline: 31.12.)

# Team
**Last Destination - this project will end us all**

Nr of Members: 3

---

## Project Overview
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

TIS3IL-WS25-PROJECT
│
├── .venv/
│
├── code/
│   ├── test.ipynb
│   ├── 01-data-prep.ipynb
│   ├── 02-baseline.ipynb
│   ├── 03-ml-models.ipynb
│   ├── 04-neural-models.ipynb
│   └── 05-evaluation-visualization.ipynb
│
├── data/
│   ├── raw/
│   │   ├── placeholder.txt
│   │
│   ├── processed/
│   │   ├── placeholder.txt
│
├── docs/
│   ├── visualizations/
│   │   ├── placeholder.png
│   │
│   ├── reports/
│   │   ├── doc_preprocessing.md
│   │   ├── doc_models.md
│   │   ├── doc_visualizations.md
│   │   ├── summary.md
│   │   └── presentation/
│   │       └── slides.pptx
│   │
│   ├── doc_preprocessing.md
│   └── README.md
│
├── .gitignore
│
├── .python-version
│
├── pyproject.toml
│
├── README.md
│
└── uv.lock


---

## Dataset — Meteostat Daily Weather Data

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

## Setup Instructions

### Clone the repository
```bash
git clone https://github.com/smn-krn/TIS3IL-WS25-Project.git
cd TIS3IL-WS25-Project

### Install dependencies using UV
```bash
uv sync
```

We are using Python **3.12.11**
So the nixtla libraries will hopefully not be an issue.

that is all you need to do! Cheat-Sheet is just in case.

### uv cheat-sheet
| Command                 | Purpose                             |
| ----------------------- | ----------------------------------- |
| `uv --version`          | Check if uv is installed            |
| `uv init`               | Create a new Python project with uv **DON'T DO THIS WHEN CLONING THE REPOSITORY** |
| `uv venv`               | Create a .venv manually             |
| `uv add <pkg>`          | Install a package                   |
| `uv run python file.py` | Run inside the environment          |