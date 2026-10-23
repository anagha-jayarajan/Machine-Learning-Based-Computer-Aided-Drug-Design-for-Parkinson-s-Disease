#Parkinson’s Disease CADD: LRRK2 Inhibitor Potency Prediction
A computational drug-discovery project that uses cheminformatics and machine learning to estimate the inhibitory potency (pIC50) of molecules against LRRK2, a kinase associated with Parkinson’s disease research. The notebook retrieves bioactivity records from BindingDB, converts molecular structures into fingerprints or molecular descriptors, compares regression models, and performs an in-silico screening of Parkinson’s-related drugs.
Important: The notebook’s predictions are computational estimates, not experimental evidence that a compound inhibits LRRK2 or treats Parkinson’s disease. They must not be interpreted as clinical recommendations.

Project goals
- Prepare LRRK2 bioactivity data for quantitative structure–activity relationship (QSAR) modelling.
- Convert molecular structures represented as SMILES into numerical features.
- Compare machine-learning regressors using cross-validation and a held-out test set.
- Identify useful molecular descriptors using feature-importance analysis.
- Apply the selected PaDEL-based model to a small panel of Parkinson’s-related drugs.
Workflow
BindingDB LRRK2 bioactivity data
             │
             ▼
Select SMILES, compound ID and IC50
             │
             ▼
Clean records and validate molecular structures
             │
             ▼
Convert IC50 (nM) to pIC50
             │
       ┌─────┴───────────┐
       ▼                 ▼
Morgan fingerprints   Molecular descriptors
(RDKit)               (RDKit / PaDEL)
       │                 │
       ▼                 ▼
Train and compare regression models
             │
             ▼
5-fold cross-validation + held-out test evaluation
             │
             ▼
Select best model and inspect feature importance
             │
             ▼
Virtual screening of Parkinson’s-related drugs
Data and target
- Bioactivity source: BindingDB
- Protein target: Leucine-rich repeat kinase 2 (LRRK2)
- Structure format: SMILES
- Measured activity field: IC50 (nM)
- Prediction target: pIC50
The notebook loads a tab-separated file named LRRK2.tsv, selects the compound ID, ligand SMILES, and IC50 columns, removes rows with missing values and duplicate SMILES, and removes structures that RDKit cannot parse.

The target is calculated as:
\[
pIC50 = -\log_{10}(IC50\text{ in molar})
\]
When IC50 is supplied in nanomolar units, this is equivalent to:
\[
pIC50 = 9 - \log_{10}(IC50\text{ in nM})
\]
Higher pIC50 values correspond to lower IC50 values and therefore greater measured potency in the relevant assay. Assay conditions can vary, so values from different experiments may not be perfectly comparable.
Molecular representations
The notebook explores more than one representation:
1. Morgan fingerprints
RDKit is used to parse SMILES and generate Morgan circular fingerprints. These encode local molecular substructures as a binary feature vector and are used in an initial set of regression experiments.
2. RDKit molecular descriptors
The notebook calculates numerical molecular descriptors that represent properties of the molecules. It cleans descriptor data, removes zero-variance features, and evaluates models on the resulting feature matrix.
3. PaDEL descriptors
The later pipeline uses padelpy to calculate PaDEL descriptors from SMILES in batches. It cleans descriptor values, attaches pIC50 labels, removes zero-variance features, and applies a correlation filter (absolute correlation threshold of 0.8) to reduce redundant features.

In the recorded notebook run, the PaDEL workflow reports 750 rows after descriptor cleaning and 319 features after correlation filtering. These counts are specific to the saved notebook run and may change if the input data or preprocessing changes.
Models and evaluation
The PaDEL workflow compares six regression models:
- Ridge Regression
- Lasso Regression
- Support Vector Regression (SVR)
- Random Forest Regressor
- Extra Trees Regressor
- Gradient Boosting Regressor
The notebook uses an 80/20 train–test split with random_state=42 and five-fold cross-validation on the training data. It reports the mean and standard deviation of cross-validation \(R^2\), test-set \(R^2\), and test RMSE.
Recorded PaDEL model results
Model	CV R² mean	CV R² std	Test R²	Test RMSE
Random Forest	0.4482	0.2575	0.5953	0.3596
Gradient Boosting	0.3973	0.2891	0.5756	0.3683
Extra Trees	0.5569	0.1765	0.5390	0.3838
SVR	0.4300	0.1852	0.5337	0.3860
Lasso Regression	0.4213	0.1229	0.4999	0.3998
Ridge Regression	0.0149	0.2435	0.4237	0.4292


Best recorded PaDEL model: Random Forest, selected by the notebook based on test-set \(R^2\), with a test \(R^2\) of 0.5953 and RMSE of 0.3596 pIC50 units.

These metrics describe performance on the notebook’s particular split. They do not establish prospective performance on new chemical scaffolds or experimental assays. The variation across cross-validation folds also suggests that performance should be assessed cautiously.
Feature importance
The notebook extracts feature importance from the selected Random Forest model. The leading descriptors in the recorded run include:
Descriptor	Importance
VC-5	0.2725
VC-3	0.0772
RDF60m	0.0592
nF10Ring	0.0355
apol	0.0221
IC3	0.0163
ATSC1v	0.0155
RDF35u	0.0138
TDB1v	0.0129
SpMin3_Bhm	0.0112


These are model-derived importance scores, not proof that a descriptor causes LRRK2 inhibition. Consult the descriptor definitions and preprocessing used by PaDEL before assigning a chemical interpretation.
Virtual screening
The notebook defines a panel of 14 Parkinson’s-related medicines or compounds and attempts to generate aligned PaDEL features for prediction with the selected model. In the saved run, the prediction output shown by the notebook contains 11 compounds; not all 14 listed compounds appear in the displayed prediction table. Check the output and descriptor-generation cells when reproducing the screening.

The recorded prediction table begins with the following results:
Compound	Predicted pIC50	Predicted IC50 (nM)	Notebook potency label
Safinamide	9.0773	0.84	Very Potent
Levodopa	8.0012	9.97	Very Potent
Entacapone	7.9641	10.86	Strong
Apomorphine	7.9188	12.06	Strong
Selegiline	7.8843	13.05	Strong
Rasagiline	7.7996	15.86	Strong
Rotigotine	7.7988	15.89	Strong
Amantadine	7.6996	19.97	Strong
Pramipexole	7.6091	24.60	Strong
Carbidopa	7.4402	36.29	Strong
Ropinirole	7.4275	37.37	Strong


Predicted IC50 values are derived from predicted pIC50 values; they are not measured assay results. Many Parkinson’s medicines act through targets other than LRRK2, so their predicted values should not be taken as evidence of direct LRRK2 inhibition. The screening is best treated as an illustration of model application, not as biological validation.
Technology stack
- Python — analysis and modelling
- Pandas / NumPy — data processing
- Matplotlib / Seaborn — plots and visualizations
- RDKit — SMILES parsing, molecular fingerprints and descriptors
- PaDEL / padelpy — molecular descriptor calculation
- scikit-learn — preprocessing, regression models, cross-validation and evaluation
- Google Colab — notebook execution in the current implementation
Repository structure
A suggested structure for publishing the project is:
parkinsons-cadd-lrrk2/
├── README.md
├── Parkinsons_Disease_project_CADD.ipynb
├── data/
│   └── README.md              # Explain how to obtain LRRK2.tsv
├── results/
│   ├── fda_drug_predictions.csv
│   ├── feature_importance_padel.png
│   └── correlation_heatmap_filtered.png
└── requirements.txt
This is a recommended repository layout, not a claim that all these files are already included in the uploaded notebook. Add only result files that you have generated and verified. Do not commit large datasets unless their terms of use permit redistribution.
Running the notebook
The notebook is currently written for Google Colab and expects a local runtime file at /content/LRRK2.tsv. Some cells also use Google Drive paths and Colab’s file-download utility. It is not yet a self-contained command-line application.
1. Upload the notebook to Google Colab.
2. Obtain the LRRK2 BindingDB data and make the tab-separated file available as LRRK2.tsv at the path expected by the notebook.
3. Run the notebook cells in order, starting with library installation and data loading.
4. Allow the descriptor-generation batches to finish. If using checkpoint files, verify that they correspond to the current input data.
5. Review the printed model comparison, feature-importance plot and virtual-screening output.
6. Download or save the generated CSV and plots, then place the verified outputs in your repository’s results/ folder.
Dependencies
The notebook installs or imports the following packages in its cells:
numpy
pandas
matplotlib
seaborn
scikit-learn
rdkit
padelpy
PaDEL descriptor generation may also require Java to be available in the runtime. The most reliable way to reproduce the current workflow is to use the notebook’s Colab environment and installation cells. A pinned requirements.txt should be created after confirming the package versions in the environment used for the final run.
Limitations and next steps
- Assay variability: BindingDB IC50 values may originate from different experimental conditions and are not necessarily directly comparable.
- Small descriptor subset in the PaDEL run: The recorded PaDEL workflow proceeds with 750 cleaned rows; results may depend strongly on data selection and cleaning.
- Cross-validation variability: The recorded fold-to-fold scores vary, so a single test split should not be treated as definitive.
- Potential data leakage / split strategy: Random splitting can place structurally similar molecules in both train and test sets. A scaffold-based split and external test set would provide a stronger test of generalization.
- 2D features only: The workflow does not directly model the 3D binding pose or protein–ligand interactions.
- No prospective experimental validation: The notebook does not establish that screened medicines bind to or inhibit LRRK2.
- Reproducibility: Several paths are specific to Colab or Google Drive, and descriptor batches/checkpoints must be checked before rerunning.
Potential improvements include scaffold-based validation, duplicate and assay-level curation, hyperparameter tuning within cross-validation, uncertainty estimation, an external test set, molecular docking or protein-structure features, and experimental testing of any future candidate compounds.
References and resources
- BindingDB — https://www.bindingdb.org/
- RDKit documentation — https://www.rdkit.org/docs/
- PaDEL-Descriptor — https://pubmed.ncbi.nlm.nih.gov/21425294/
- scikit-learn documentation — https://scikit-learn.org/stable/
- LRRK2 and Parkinson’s disease background — see the primary literature cited in the project report.
Disclaimer
This repository is for educational and research purposes only. Model outputs are estimates based on the dataset and workflow used; they are not medical advice, clinical evidence, or confirmation of drug-target activity. Any candidate requires independent computational and experimental validation.
