# Machine Learning-Based Computer-Aided Drug Design for Parkinson’s Disease: LRRK2 Inhibitor Potency Prediction

## Overview

Parkinson’s disease (PD) is a progressive neurodegenerative disorder characterized by the degeneration of dopamine-producing neurons. Leucine-Rich Repeat Kinase 2 (LRRK2) is an important therapeutic target associated with Parkinson’s disease, particularly due to gain-of-function mutations that increase its kinase activity.

This project develops an end-to-end **Computer-Aided Drug Design (CADD) pipeline** using machine learning to predict the inhibitory potency of chemical compounds against LRRK2.

The workflow combines bioactivity data retrieval, molecular data preprocessing, chemical feature engineering, regression modeling, model evaluation, feature importance analysis, and virtual screening of Parkinson’s disease medications.

The primary objective is to identify a reliable quantitative structure–activity relationship (QSAR) model that can estimate the pIC50 values of compounds and help prioritize potential LRRK2 inhibitor candidates for further investigation.

## Objectives

- Retrieve LRRK2-specific bioactivity records from BindingDB.
- Clean and standardize molecular structures and IC50 measurements.
- Transform IC50 values into logarithmic pIC50 values for regression.
- Generate Morgan fingerprints and PaDEL molecular descriptors.
- Train and compare eight machine learning regression configurations.
- Evaluate model performance using five-fold cross-validation and an independent test set.
- Identify important molecular features associated with predicted LRRK2 inhibitory potency.
- Perform virtual screening of 14 Parkinson’s disease medications using the selected model.

## Methodology

The project follows a sequential computational drug discovery workflow.

```text
BindingDB LRRK2 Bioactivity Data
              |
              v
       Data Preprocessing
              |
              v
      IC50 to pIC50 Conversion
              |
              v
      Molecular Feature Engineering
          /             \
         v               v
 Morgan Fingerprints   PaDEL Descriptors
         |               |
         v               v
    ML Regression Models
              |
              v
   5-Fold Cross-Validation
              |
              v
 Independent Test Evaluation
              |
              v
     Best Model Selection
              |
              v
   Feature Importance Analysis
              |
              v
 Virtual Screening of 14 PD Drugs
              |
              v
  Predicted pIC50 and Potency Ranking
```

### 1. Data Collection

Bioactivity data were obtained from [BindingDB](https://www.bindingdb.org/), a public database of experimentally measured protein–ligand binding affinities.

- Target protein: LRRK2 (Leucine-Rich Repeat Kinase 2)
- Initial dataset: Approximately 23,000 records
- Selected fields: Compound identifier, ligand SMILES, and IC50 (nM)
- Final curated dataset: 6,640 distinct molecular entries

### 2. Data Preprocessing

The raw dataset was processed to improve data quality and consistency.

The preprocessing steps included:

1. Selecting relevant compound and bioactivity columns.
2. Removing records with missing IC50 values or SMILES strings.
3. Removing duplicate molecular structures based on SMILES.
4. Cleaning IC50 strings and converting valid measurements into numeric values.
5. Removing invalid, infinite, and missing target values.
6. Converting IC50 measurements to pIC50.

The target variable was calculated as:

\[
pIC_{50}=-\log_{10}(IC_{50}\text{ in molar})
\]

Higher pIC50 values indicate greater inhibitory potency.

### 3. Molecular Feature Engineering

Two complementary molecular representations were generated to capture different aspects of chemical structure.

**Morgan Fingerprints**

- 2,048-bit binary molecular representation.
- Radius of 2.
- Captures local molecular substructures and chemical environments.
- Zero-variance fingerprint columns were removed before modeling.

**PaDEL Molecular Descriptors**

- Physicochemical and topological molecular properties.
- 337 retained features in the final descriptor matrix.
- Includes properties such as molecular weight, lipophilicity (logP/SlogP), topological polar surface area (TPSA), hydrogen-bond donors and acceptors, and ring counts.
- Descriptor filtering included a variance threshold of 0.01 and removal of columns with more than 10% missing values.

### 4. Machine Learning Models

The regression models were trained using both Morgan fingerprints and PaDEL descriptors.

The evaluated model configurations included:

- Linear Regression
- Ridge Regression
- Lasso Regression
- K-Nearest Neighbors (KNN) Regression
- Support Vector Regression (SVR)
- Random Forest Regression
- Gradient Boosting Regression

The report evaluates eight model configurations for each feature representation.

### 5. Model Evaluation

The dataset was divided into an 80% training set and a 20% independent test set.

Five-fold cross-validation was performed on the training data using shuffled KFold splits with `random_state=42`.

The following metrics were used:

| Metric | Purpose | Preferred value |
|---|---|---|
| R² | Measures explained variance in pIC50 | Higher |
| RMSE | Measures prediction error in pIC50 units | Lower |

The independent test set was reserved for evaluating model generalization.

### 6. Feature Importance Analysis

The selected Random Forest model trained on PaDEL descriptors was examined to identify influential molecular features.

The analysis highlighted descriptors associated with:

- Molecular lipophilicity
- Molecular size and weight
- Topological polar surface area
- Aromatic ring count
- Overall ring-system complexity

These findings provide insight into the structural properties associated with the model's predicted inhibitory potency. Feature importance indicates predictive association, not proof of a causal biochemical mechanism.

### 7. Virtual Screening

The best-performing model was applied to a panel of 14 Parkinson’s disease medications and clinically used compounds.

The workflow included:

1. Obtaining and validating compound SMILES representations.
2. Calculating PaDEL molecular descriptors.
3. Aligning the descriptors with the 337-feature training schema.
4. Generating predicted pIC50 values using the trained Random Forest model.
5. Converting predicted pIC50 values to estimated IC50 values.
6. Categorizing compounds into potency tiers.

The potency categories used in the report were:

| Category | pIC50 range |
|---|---|
| Very Potent | pIC50 ≥ 8.0 |
| Strong | 7.0 ≤ pIC50 < 8.0 |
| Moderate | 6.0 ≤ pIC50 < 7.0 |
| Weak | 5.0 ≤ pIC50 < 6.0 |
| Inactive | pIC50 < 5.0 |

These are model-based screening categories and do not establish measured LRRK2 inhibition.

## Results

### Best-Performing Model

The Random Forest regressor trained on PaDEL molecular descriptors achieved the best reported performance.

| Evaluation metric | Result |
|---|---:|
| Five-fold CV R² mean | 0.6310 |
| Five-fold CV R² standard deviation | 0.0141 |
| Test R² | 0.6380 |
| Test RMSE | 0.7601 |

The model explains approximately 63.8% of the variance in the held-out pIC50 values in this evaluation.

### Comparison of Feature Representations

| Model | Feature representation | Test R² | Test RMSE |
|---|---|---:|---:|
| Random Forest | PaDEL descriptors | 0.6380 | 0.7601 |
| Gradient Boosting | PaDEL descriptors | 0.6250 | 0.7720 |
| Random Forest | Morgan fingerprints | 0.5901 | 0.8103 |
| Gradient Boosting | Morgan fingerprints | 0.5804 | 0.8211 |

PaDEL descriptors outperformed Morgan fingerprints for the leading ensemble models in the reported experiments.

### Virtual Screening Findings

The report's virtual screening produced the following leading predictions:

| Compound | Predicted pIC50 | Predicted potency tier |
|---|---:|---|
| Bromocriptine | 8.14 | Very Potent |
| Cabergoline | 8.02 | Very Potent |
| Apomorphine | 7.55 | Strong |
| Pramipexole | 7.44 | Strong |
| Rotigotine | 7.38 | Strong |
| Ropinirole | 7.21 | Strong |
| Amantadine | 5.91 | Weak |

These are predictions from the trained model, not experimentally confirmed LRRK2 IC50 measurements. Many Parkinson’s medications act on targets other than LRRK2, so predicted potency should not be interpreted as evidence of direct LRRK2 inhibition or clinical efficacy against the target.

## Key Findings

- Random Forest with PaDEL descriptors was the best-performing model in the reported evaluation.
- PaDEL descriptors provided better predictive performance than Morgan fingerprints for the leading models.
- Tree-based ensemble methods outperformed the evaluated linear and distance-based approaches.
- Feature importance analysis identified molecular properties associated with the model's predictions.
- Virtual screening demonstrated how the trained model can prioritize compounds computationally for further investigation.

## Technology Stack

The project uses or is designed around the following tools and technologies:

- **Python** — data processing and machine learning.
- **Pandas and NumPy** — data manipulation and numerical computation.
- **RDKit** — molecular structure parsing and chemical validation.
- **PaDEL-Descriptor** — molecular descriptor calculation.
- **Scikit-learn** — regression models, preprocessing, cross-validation, and evaluation.
- **Matplotlib** — visualization of distributions and model performance.
- **BindingDB** — source of experimentally measured bioactivity data.

## Project Structure

A suggested repository structure is shown below. Adapt the filenames to match the files in your actual repository.

```text
LRRK2-CADD-Parkinsons/
├── data/
│   └── README.md
├── notebooks/
│   └── CADD_LRRK2_QSAR.ipynb
├── models/
│   └── README.md
├── results/
│   ├── model_comparison.csv
│   └── virtual_screening_results.csv
├── figures/
│   ├── pic50_distribution.png
│   ├── model_comparison.png
│   └── feature_importance.png
├── requirements.txt
└── README.md
```

This is a suggested structure, not a claim that all these files are already present.

## Getting Started

### Prerequisites

Install Python and the required libraries. RDKit and PaDEL-Descriptor may require separate installation or configuration depending on your operating system.

### Installation

Clone the repository:

```bash
git clone <YOUR_REPOSITORY_URL>
cd LRRK2-CADD-Parkinsons
```

Install the Python dependencies listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

### Running the Project

1. Open the Jupyter notebook in the `notebooks/` directory.
2. Configure the paths to the BindingDB data and any required descriptor files.
3. Run the preprocessing and feature-generation steps.
4. Train and evaluate the regression models.
5. Compare the models using cross-validation and held-out test metrics.
6. Run virtual screening with the selected model.

The repository must include the corresponding notebook, input data or data-access instructions, and required model artifacts for these steps to run as described.

## Limitations

- BindingDB measurements can vary across assay conditions and experimental sources.
- The model uses two-dimensional molecular representations and does not directly model three-dimensional protein–ligand interactions.
- Test performance does not guarantee accurate predictions for chemically dissimilar compounds.
- Virtual screening predictions have not been confirmed by prospective biochemical experiments.
- Parkinson’s disease medications may act through different molecular targets; their predicted scores do not establish LRRK2-specific activity.

## Future Work

Potential extensions include:

- Combining Morgan fingerprints and PaDEL descriptors in a unified model.
- Exploring graph neural networks for molecular representation learning.
- Incorporating molecular docking and three-dimensional structural information.
- Applying scaffold-based and external validation to test generalization to new chemical series.
- Using explainability techniques to investigate molecular features and substructures.
- Experimentally validating promising predicted compounds against LRRK2.

## References

1. Gilks, W. P. et al. (2005). A common LRRK2 mutation in idiopathic Parkinson's disease. *The Lancet*, 365(9457), 415–416.
2. Liu, T., Lin, Y., Wen, X., Jorissen, R. N., & Gilson, M. K. (2007). BindingDB: a web-accessible database of experimentally determined protein–ligand binding affinities. *Nucleic Acids Research*, 35, D198–D201.
3. Landrum, G. (2023). RDKit: Open-source cheminformatics software. [https://www.rdkit.org](https://www.rdkit.org)
4. Yap, C. W. (2011). PaDEL-descriptor: An open source software to calculate molecular descriptors and fingerprints. *Journal of Computational Chemistry*, 32(7), 1466–1474.
5. Breiman, L. (2001). Random Forests. *Machine Learning*, 45, 5–32.
6. Friedman, J. H. (2001). Greedy function approximation: A gradient boosting machine. *The Annals of Statistics*, 29, 1189–1232.
7. Pedregosa, F. et al. (2011). Scikit-learn: Machine Learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.
8. Morgan, H. L. (1965). The generation of a unique machine description for chemical structures. *Journal of Chemical Documentation*, 5(2), 107–113.
9. Alessi, D. R., & Sammler, E. (2018). LRRK2 kinase in Parkinson's disease. *Science*, 360(6384), 36–37.

## Disclaimer

This project is an academic computational drug discovery study. Its predictions are intended for research and candidate prioritization only. Predicted potency does not establish experimental LRRK2 inhibition, therapeutic efficacy, safety, or suitability for clinical use.

