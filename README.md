# Predicting In-Hospital Mortality Using Machine Learning

A healthcare data science project investigating whether demographic, diagnostic and laboratory information can be used to predict in-hospital mortality using the MIMIC-III Clinical Database Demo.

## Overview

In-hospital mortality is an important clinical outcome and a potential application of predictive analytics in healthcare. This project explores the use of supervised machine learning to classify hospital admissions according to whether they resulted in in-hospital death.

The project follows an end-to-end analytical workflow, covering clinical data preparation, exploratory data analysis (EDA), feature engineering, model development and evaluation.

**Research question:** Can machine learning models predict in-hospital mortality using information available during the first 24 hours of a hospital admission?

This is an exploratory proof-of-concept project, not a clinically validated prediction system.

## Dataset

The analysis uses the [MIMIC-III Clinical Database Demo](https://physionet.org/content/mimiciii-demo/1.4/), a demonstration dataset containing de-identified clinical information.

The final analytical dataset contains:

- **129 hospital admissions** from 100 unique patients.
- **40 admissions resulting in in-hospital death** (31.0%).
- **89 admissions not resulting in in-hospital death** (69.0%).

Each row represents a hospital admission. The target variable is `hospital_expire_flag`, where `1` represents in-hospital death and `0` represents survival to hospital discharge.

### Features

The models use demographic, diagnostic and laboratory information:

- **Demographic:** age at admission and gender.
- **Diagnostic:** grouped primary diagnosis category.
- **Laboratory:** bicarbonate, calcium, creatinine, glucose, magnesium, potassium, sodium, urea nitrogen, haematocrit, haemoglobin, platelet count, red cell distribution width (RDW) and white blood cell count.

Laboratory measurements were selected from the first 24 hours after admission. Where multiple measurements of the same test were available, the most recent measurement within that window was retained.

The dataset is very small, so its findings should not be assumed to generalise to the wider hospital population.

## Project Workflow

The analysis is organised into six Jupyter notebooks.

| Notebook | Purpose |
|---|---|
| `01_database_exploration.ipynb` | Inspect the source tables and understand the database structure. |
| `02_data_cleaning.ipynb` | Clean and combine clinical data into an admission-level dataset. |
| `03_eda.ipynb` | Explore outcome distributions, patient characteristics, laboratory measurements, missing values and correlations. |
| `04_feature_engineering.ipynb` | Select predictors and prepare numerical and categorical features for modelling. |
| `05_machine_learning.ipynb` | Train and compare classification models using cross-validation. |
| `06_model_evaluation.ipynb` | Evaluate the selected model on a held-out test set. |

The workflow uses a preprocessing pipeline to handle missing numerical values through median imputation, standardise numerical features and one-hot encode categorical features.

## Machine Learning Models

Three supervised classification algorithms were compared:

- **Logistic Regression** — a linear classification model used as a comparatively simple baseline approach.
- **Random Forest** — an ensemble of decision trees.
- **XGBoost** — a gradient-boosted tree model.

A majority-class baseline was also considered. Model comparison focused on both overall accuracy and mortality recall, since accuracy alone can obscure poor detection of fatal admissions.

Five-fold stratified cross-validation was used during model development, followed by a separate held-out test evaluation of Logistic Regression.

## Results

### Cross-validation

| Model | Mean accuracy | Mean mortality recall |
|---|---:|---:|
| Logistic Regression | 61.0% | 21.0% |
| Random Forest | 62.1% | 3.3% |
| XGBoost | 54.4% | 10.0% |

Logistic Regression achieved the highest mean mortality recall among the three models, although its performance remained limited. Random Forest had slightly higher mean accuracy but rarely identified admissions resulting in death.

### Held-out test evaluation

The selected Logistic Regression model was evaluated on 26 admissions, including eight deaths.

| Metric | Result |
|---|---:|
| Accuracy | 73.1% |
| Precision | 66.7% |
| Mortality recall | 25.0% |
| F1-score | 0.364 |
| Specificity | 94.4% |
| ROC-AUC | 0.507 |

The model correctly identified two of the eight fatal admissions and missed six. Its accuracy was only modestly higher than the majority-class baseline accuracy of 69.2%, while its ROC-AUC was close to 0.5, indicating little ability to distinguish between the two outcome classes in this test sample.

These results illustrate why classification models should be evaluated using multiple metrics, particularly when the outcome of interest is not the majority class.

## Limitations

Several limitations are important when interpreting the findings:

- **Small sample size:** 129 admissions provide insufficient evidence for reliable conclusions about performance in a wider patient population.
- **Limited test set:** only eight fatal admissions were included in the held-out test set, making the evaluation highly sensitive to individual predictions.
- **Potential temporal leakage:** although laboratory measurements were restricted to the first 24 hours, the project did not establish that all diagnosis-derived information was available within that period.
- **Repeated patients:** multiple admissions from the same patient may be represented in the dataset. The admission-level random split did not explicitly group admissions by patient, so the possibility of patient-level information crossing the training and test sets remains a limitation.
- **No external validation:** the models were not evaluated on an independent clinical dataset.

The models should therefore be treated as educational proof-of-concept implementations rather than clinically reliable mortality prediction tools.

## Technologies

The project uses:

- **Python** for data analysis and modelling.
- **Pandas and NumPy** for data manipulation.
- **Matplotlib** for data visualisation.
- **Scikit-learn** for preprocessing, cross-validation, classification and evaluation.
- **XGBoost** for gradient-boosted classification.
- **Jupyter Notebook** for interactive analysis and documentation.

## Repository Structure

```text
Predictive-Health-Analytics/
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── 01_database_exploration.ipynb
│   ├── 02_data_cleaning_and_feature_engineering.ipynb
│   ├── 03_eda.ipynb
│   ├── 04_feature_engineering.ipynb
│   ├── 05_machine_learning.ipynb
│   └── 06_model_evaluation.ipynb
├── report/
│   ├── Predicting In-Hospital Mortality - MIMIC-III Report.docx
│   ├── Predicting In-Hospital Mortality - MIMIC-III Report.pdf
├── README.md
└── requirements.txt
```

The structure separates raw and processed data, analysis notebooks, supporting code, visualisations and the project report.

## Getting Started

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd Predictive-Health-Analytics
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Activate it:

**macOS / Linux**
```bash
source .venv/bin/activate
```

**Windows**
```bash
.venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Obtain the dataset

Download the MIMIC-III Clinical Database Demo from [PhysioNet](https://physionet.org/content/mimiciii-demo/1.4/) and place the required source files in the appropriate directory under `data/raw/`.

The dataset is not included in this repository. Follow the dataset provider's access conditions and citation requirements.

### 5. Run the notebooks

Open Jupyter Notebook or JupyterLab and execute the notebooks in numerical order, starting with database exploration and proceeding through model evaluation.

The data paths in the notebooks must match your local directory structure. The project is intended to be run from the repository root, with notebook-relative paths used where appropriate.

## Future Improvements

Potential next steps include:

- Evaluate the workflow using a substantially larger clinical dataset.
- Enforce a strict prediction-time boundary for every predictor.
- Split data by patient to reduce the risk of information crossing between training and test sets.
- Use repeated validation and more systematic model selection.
- Investigate probability calibration and decision thresholds.
- Validate performance on an independent dataset.

## Author's Note

This project was developed as a portfolio project to strengthen practical skills in healthcare data science and machine learning. It demonstrates the process of preparing clinical data, investigating patterns, comparing predictive models and critically interpreting evaluation results.

The project is intended for educational and analytical purposes only. It does not provide medical advice and must not be used to make clinical decisions.