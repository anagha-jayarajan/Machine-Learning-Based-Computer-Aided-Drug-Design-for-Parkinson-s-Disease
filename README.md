# Parkinson’s Disease CADD: LRRK2 Inhibitor Potency Prediction

A computational drug-discovery project that uses cheminformatics and machine learning to estimate the inhibitory potency (`pIC50`) of molecules against **LRRK2**, a kinase associated with Parkinson’s disease research. The notebook retrieves bioactivity records from BindingDB, converts molecular structures into fingerprints or molecular descriptors, compares regression models, and performs an in-silico screening of Parkinson’s-related drugs.

> **Important:** The notebook’s predictions are computational estimates, not experimental evidence that a compound inhibits LRRK2 or treats Parkinson’s disease. They must not be interpreted as clinical recommendations.

## Project Goals

- Prepare LRRK2 bioactivity data for quantitative structure–activity relationship (QSAR) modelling.
- Convert molecular structures represented as SMILES into numerical features.
- Compare machine-learning regressors using cross-validation and a held-out test set.
- Identify useful molecular descriptors using feature-importance analysis.
- Apply the selected PaDEL-based model to a panel of Parkinson’s-related drugs.

## Workflow

```text
BindingDB LRRK2 Bioactivity Data
              |
              v
 Select SMILES, Compound ID and IC50
              |
              v
 Data Cleaning and Structure Validation
              |
              v
       Convert IC50 to pIC50
              |
       +------+----------+
       |                 |
       v                 v
 Morgan Fingerprints   Molecular Descriptors
       |                 |
       +--------+--------+
                |
                v
     Train Regression Models
                |
                v
 5-Fold Cross-Validation and Test Evaluation
                |
                v
 Select Best Model and Analyze Features
                |
                v
 Virtual Screening of Parkinson’s Drugs
```

## Dataset

- **Source:** [BindingDB](https://www.bindingdb.org/)
- **Target protein:** Leucine-rich repeat kinase 2 (LRRK2)
- **Molecular representation:** SMILES
- **Activity measurement:** IC50 in nanomolar (nM)
- **Prediction target:** pIC50

The notebook loads a tab-separated dataset named `LRRK2.tsv`, selects relevant compound and activity fields, removes missing values and duplicate SMILES, and filters out molecular structures that cannot be parsed by RDKit.

### pIC50 calculation

The activity values are transformed into a logarithmic scale:

\[
pIC50=-\log_{10}(IC50\text{ in molar})
\]

When IC50 is expressed in nanomolar units:

\[
pIC50=9-\log_{10}(IC50\text{ in nM})
\]

Higher pIC50 values correspond to lower IC50 values and greater measured potency in the relevant assay. Differences in experimental conditions can affect the comparability of bioactivity measurements.

## Molecular Feature Extraction

### 1. Morgan Fingerprints

RDKit generates Morgan circular fingerprints from SMILES. These binary vectors represent molecular substructures and are used in regression experiments to capture structural patterns associated with bioactivity.

### 2. RDKit Molecular Descriptors

RDKit calculates numerical molecular descriptors representing chemical and physicochemical properties. The notebook processes these features and removes zero-variance descriptors before modelling.

### 3. PaDEL Molecular Descriptors

The PaDEL workflow uses `padelpy` to calculate molecular descriptors in batches. The notebook cleans descriptor data, removes zero-variance features, and applies a correlation filter with an absolute correlation threshold of 0.8.

In the recorded notebook run, the PaDEL workflow reports **750 rows after descriptor cleaning** and **319 features after correlation filtering**.

## Machine Learning Models

The PaDEL workflow compares six regression algorithms:

- Ridge Regression
- Lasso Regression
- Support Vector Regression (SVR)
- Random Forest Regressor
- Extra Trees Regressor
- Gradient Boosting Regressor

### Model Evaluation

The notebook uses an 80:20 train–test split with `random_state=42` and five-fold cross-validation on the training data.

The following metrics are reported:

- **R² score:** Measures how much variation in the target is explained by the model.
- **RMSE:** Measures the typical magnitude of prediction errors, with larger errors penalized more heavily.

### Recorded PaDEL Model Results

| Model | CV R² Mean | CV R² Std | Test R² | Test RMSE |
|---|---:|---:|---:|---:|
| Random Forest | 0.4482 | 0.2575 | **0.5953** | **0.3596** |
| Gradient Boosting | 0.3973 | 0.2891 | 0.5756 | 0.3683 |
| Extra Trees | 0.5569 | 0.1765 | 0.5390 | 0.3838 |
| SVR | 0.4300 | 0.1852 | 0.5337 | 0.3860 |
| Lasso Regression | 0.4213 | 0.1229 | 0.4999 | 0.3998 |
| Ridge Regression | 0.0149 | 0.2435 | 0.4237 | 0.4292 |

**Best recorded model:** Random Forest using PaDEL descriptors, with a test R² of **0.5953** and an RMSE of **0.3596 pIC50 units**.

These results describe the particular split used in the notebook and do not guarantee performance on unseen chemical scaffolds or experimental assays.

## Feature Importance Analysis

The notebook extracts feature importance scores from the selected Random Forest model.

| Descriptor | Importance |
|---|---:|
| VC-5 | 0.2725 |
| VC-3 | 0.0772 |
| RDF60m | 0.0592 |
| nF10Ring | 0.0355 |
| apol | 0.0221 |
| IC3 | 0.0163 |
| ATSC1v | 0.0155 |
| RDF35u | 0.0138 |
| TDB1v | 0.0129 |
| SpMin3_Bhm | 0.0112 |

These scores indicate which descriptors contributed to the trained model’s predictions. They do not establish that any individual descriptor causes LRRK2 inhibition.

## Virtual Screening of Parkinson’s-Related Drugs

The notebook defines a panel of 14 Parkinson’s-related medicines or compounds and applies the trained model to their molecular descriptors.

The recorded prediction output contains 11 compounds:

| Compound | Predicted pIC50 | Predicted IC50 (nM) | Notebook Label |
|---|---:|---:|---|
| Safinamide | 9.0773 | 0.84 | Very Potent |
| Levodopa | 8.0012 | 9.97 | Very Potent |
| Entacapone | 7.9641 | 10.86 | Strong |
| Apomorphine | 7.9188 | 12.06 | Strong |
| Selegiline | 7.8843 | 13.05 | Strong |
| Rasagiline | 7.7996 | 15.86 | Strong |
| Rotigotine | 7.7988 | 15.89 | Strong |
| Amantadine | 7.6996 | 19.97 | Strong |
| Pramipexole | 7.6091 | 24.60 | Strong |
| Carbidopa | 7.4402 | 36.29 | Strong |
| Ropinirole | 7.4275 | 37.37 | Strong |

The predicted IC50 values are calculated from predicted pIC50 values and are **not experimentally measured results**. Many Parkinson’s medicines act through targets other than LRRK2. Therefore, these predictions should not be interpreted as evidence that the compounds directly inhibit LRRK2.

## Technology Stack

- **Python** — programming and analysis
- **Pandas and NumPy** — data processing
- **Matplotlib and Seaborn** — visualization
- **RDKit** — SMILES parsing, molecular fingerprints and descriptors
- **PaDEL / padelpy** — molecular descriptor generation
- **Scikit-learn** — preprocessing, regression, cross-validation and evaluation
- **Google Colab** — notebook execution environment

## Repository Structure

Suggested structure for publishing the project:

```text
parkinsons-cadd-lrrk2/
├── README.md
├── Parkinsons_Disease_project_CADD.ipynb
├── data/
│   └── README.md
├── results/
│   ├── fda_drug_predictions.csv
│   ├── feature_importance_padel.png
│   └── correlation_heatmap_filtered.png
└── requirements.txt
```

This is a suggested structure. Include only files and results that you have generated and verified. Check the dataset’s terms before redistributing it.

## How to Run the Project

The current notebook is designed primarily for Google Colab and expects a dataset at `/content/LRRK2.tsv`. Some cells also use Google Drive paths and Colab-specific download functionality.

1. Upload the notebook to Google Colab.
2. Obtain the LRRK2 BindingDB dataset and make `LRRK2.tsv` available at the path expected by the notebook.
3. Run the notebook cells in order, starting with the installation and data-loading cells.
4. Allow molecular descriptor generation to finish.
5. Review the model comparison, feature-importance analysis and virtual-screening output.
6. Save the generated plots and prediction tables for your repository.

### Dependencies

```text
numpy
pandas
matplotlib
seaborn
scikit-learn
rdkit
padelpy
```

PaDEL descriptor generation may require Java in the runtime. For reproducibility, confirm package versions and the expected file paths in the environment used for the final run.

## Limitations

- **Assay variability:** IC50 measurements can differ between experiments and laboratories.
- **Data size:** The recorded PaDEL pipeline uses 750 cleaned rows, which may limit generalization.
- **Cross-validation variability:** The recorded scores vary across folds.
- **Random splitting:** Structurally similar compounds may occur in both training and test sets.
- **Two-dimensional representations:** The pipeline does not directly model three-dimensional binding geometry or protein–ligand interactions.
- **No prospective experimental validation:** Predictions have not established actual LRRK2 inhibition by the screened medicines.
- **Reproducibility:** Some file paths and processing steps are specific to Google Colab or Google Drive.

## Future Improvements

- Use scaffold-based train–test splitting.
- Improve assay-level data curation and duplicate handling.
- Perform systematic hyperparameter tuning.
- Evaluate the model on an independent external dataset.
- Combine molecular fingerprints and physicochemical descriptors.
- Explore molecular docking, protein-structure features or graph neural networks.
- Estimate prediction uncertainty and experimentally test promising candidates.

## References and Resources

- [BindingDB](https://www.bindingdb.org/)
- [RDKit Documentation](https://www.rdkit.org/docs/)
- [PaDEL-Descriptor Publication](https://pubmed.ncbi.nlm.nih.gov/21425294/)
- [Scikit-learn Documentation](https://scikit-learn.org/stable/)
- LRRK2 and Parkinson’s disease references cited in the project report.

## Disclaimer

This project is intended for educational and research purposes only. The model produces computational estimates based on its training data and workflow. Predictions are not medical advice, clinical evidence, or confirmation of drug-target activity. Any proposed candidate requires independent computational and experimental validation.
