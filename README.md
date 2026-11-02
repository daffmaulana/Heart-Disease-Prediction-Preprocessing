# Heart Disease Prediction: Data Preprocessing

<p align="center">
  <strong>From raw clinical records to a clean, model-ready dataset, automatically on every push.</strong>
</p>

<p align="center">
  The data preparation stage of a two-repository MLOps project: exploratory analysis, cleaning, and an automated preprocessing pipeline powered by GitHub Actions.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white" alt="pandas" />
  <img src="https://img.shields.io/badge/Matplotlib-11557C?style=flat&logo=plotly&logoColor=white" alt="Matplotlib" />
  <img src="https://img.shields.io/badge/Jupyter-F37626?style=flat&logo=jupyter&logoColor=white" alt="Jupyter" />
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white" alt="GitHub Actions" />
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#dataset">Dataset</a> •
  <a href="#preprocessing">Preprocessing</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#getting-started">Getting Started</a> •
</p>

---

## What is this project?

This repository takes the **raw Cleveland heart disease dataset**, explores it, fixes its data quality problems, and publishes a **clean dataset ready for model training**. The cleaning logic lives in a reusable Python script, and a GitHub Actions workflow reruns it on every push and commits the result back to the repository.

It is the **first stage** of a two-repository pipeline:

```text
Raw Data → EDA → Cleaning → Clean Dataset → Model Training & Docker (next repository)
```

| Stage | Repository | Role |
| --- | --- | --- |
| 1. Preprocessing | **this repository** | Explore the raw data, clean it, and publish the processed dataset |
| 2. Training and delivery | [heart-disease-prediction-ci-cd](https://github.com/daffmaulana/heart-disease-prediction-ci-cd) | Train a Random Forest with MLflow, build a Docker image, publish to Docker Hub |

> **Disclaimer:** : This is a learning and portfolio project by Dicoding Indonesia Bootcamp. It is not a medical device and must not be used for diagnosis.
---

## Features

| Feature                 | Description                                                              |
| ----------------------- | ------------------------------------------------------------------------ |
| Exploratory Analysis    | Inspect shape, data types, missing values, target balance and correlations |
| Data Validation         | Detect and remove values outside the valid range of `ca` and `thal`      |
| Readable Column Names   | Rename cryptic abbreviations (`cp`, `trestbps`, ...) into clear names    |
| Reusable Script         | Same cleaning logic as the notebook, packaged as a Python script         |
| Automated Preprocessing | GitHub Actions regenerates the clean dataset on every push to `main`     |
| Auto-commit             | The processed CSV is committed back to the repository automatically      |

---

## Dataset

| Property        | Raw data                          | After preprocessing               |
| --------------- | --------------------------------- | --------------------------------- |
| File            | `heartdisease_raw.csv`            | `heartdisease_preprocessing.csv`  |
| Rows            | 303                               | 296                               |
| Columns         | 14                                | 14                                |
| Missing values  | 0                                 | 0                                 |
| `target = 0`    | 138 (disease present)             | 136                               |
| `target = 1`    | 165 (no disease)                  | 160                               |

> In this dataset `target = 0` corresponds to patients **with** heart disease and `target = 1` to patients **without**. This is easy to get backwards, so be careful when interpreting model metrics.

<!-- TODO: cite the original dataset (e.g. UCI Heart Disease / Cleveland) and its license -->

### Column Reference

| Original   | Renamed                      | Type        |
| ---------- | ---------------------------- | ----------- |
| `age`      | `age`                        | Numerical   |
| `sex`      | `sex`                        | Binary      |
| `cp`       | `chest_pain_type`            | Categorical |
| `trestbps` | `resting_blood_pressure`     | Numerical   |
| `chol`     | `cholesterol`                | Numerical   |
| `fbs`      | `fasting_blood_sugar`        | Binary      |
| `restecg`  | `resting_electrocardiogram`  | Categorical |
| `thalach`  | `max_heart_rate_achieved`    | Numerical   |
| `exang`    | `exercise_induced_angina`    | Binary      |
| `oldpeak`  | `st_depression`              | Numerical   |
| `slope`    | `st_slope`                   | Categorical |
| `ca`       | `num_major_vessels`          | Numerical   |
| `thal`     | `thalassemia`                | Categorical |
| `target`   | `target`                     | Binary      |

### Exploratory Findings

- The dataset has **no missing values**.
- The target is fairly balanced: **54.5%** for `1` and **45.5%** for `0`.
- Pearson correlations between the numerical features and the target are **weak**.
- Column names are abbreviations that are hard to read.
- `ca` should only contain values 0-3 and `thal` only 1-3, but the raw data contains invalid values.

---

## Preprocessing

| Step | Action | Effect |
| --- | --- | --- |
| 1 | Remove rows with invalid `ca` (value 4; valid range is 0-3) | 5 rows removed |
| 2 | Remove rows with invalid `thal` (value 0; valid range is 1-3) | 2 rows removed |
| 3 | Rename columns to descriptive names | 11 columns renamed |
| 4 | Save the result as `heartdisease_preprocessing.csv` | 303 → 296 rows |

Scaling and encoding are **not** applied in this stage. Tree-based models such as the Random Forest used in the next repository do not require them.

---

## Architecture

```text
┌──────────────────────────────┐
│     Push to `main` branch    │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│      GitHub Actions          │
│      Python 3.10 · deps      │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│ automate_Daffa-Bagus-Maulana │
│ load → clean → rename → save │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│ git commit + push            │
│ heartdisease_preprocessing   │
│ .csv                         │
└──────────────┬───────────────┘
               ▼
        Used by the training
        repository (stage 2)
```

### CI Workflow

The workflow `Automated Preprocessing` runs on every push to `main`:

1. Check out the repository and set up Python 3.10
2. Install the dependencies from `requirements.txt`
3. Run `preprocessing/automate_Daffa-Bagus-Maulana.py`
4. Commit the regenerated `preprocessing/heartdisease_preprocessing.csv` and push it (skipped when nothing changed)

> The commit step needs write access. If the push fails with a 403 error, enable **Settings → Actions → General → Workflow permissions → Read and write permissions**, or add `permissions: contents: write` to the workflow. Pushes made with the default `GITHUB_TOKEN` do not retrigger the workflow, so there is no infinite loop.

### Repository Structure

<!-- TODO: verify this tree matches the actual layout of the repository -->

```text
heart-disease-prediction-preprocessing/
├── .github/workflows/
│   └── preprocessing.yml                  # Automated preprocessing workflow
├── preprocessing/
│   ├── Eksperimen_Daffa-Bagus-Maulana.ipynb   # EDA and preprocessing notebook
│   ├── automate_Daffa-Bagus-Maulana.py        # Reusable preprocessing script
│   └── heartdisease_preprocessing.csv         # Generated clean dataset
├── heartdisease_raw.csv                   # Raw dataset
├── requirements.txt
└── README.md
```

---

## Getting Started

```bash
# 1. Clone
git clone https://github.com/daffmaulana/heart-disease-prediction-preprocessing.git
cd heart-disease-prediction-preprocessing

# 2. Install
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3. Run the preprocessing script (from the repository root)
python preprocessing/automate_Daffa-Bagus-Maulana.py
```

To explore the notebook, install Jupyter as well (`pip install notebook`) and open `preprocessing/Eksperimen_Daffa-Bagus-Maulana.ipynb`.

> Run the script from the repository root, because it reads `./heartdisease_raw.csv` using a relative path.

---

<p align="center">
  <strong>Heart Disease Prediction: Data Preprocessing</strong><br>
  Clean data in, reliable models out.
</p>
