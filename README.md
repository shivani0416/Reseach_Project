# Analysis of Concept Drift in Machine Learning-Based Network Intrusion Detection

A research project investigating how changes in network traffic over
time affect a machine learning-based Network Intrusion Detection System
(NIDS), and whether sequential model retraining can reduce performance
degradation.

> **Notebook note:** The notebook in this repository is the original
> Google Colab notebook supplied for this project. It is intentionally
> preserved as-is. This README explains the project and provides setup
> guidance; it does not imply that the notebook's code has been modified
> or independently rerun from a clean environment.

## Table of Contents

-   [Project Overview](#project-overview)
-   [Research Question](#research-question)
-   [Objectives](#objectives)
-   [Dataset](#dataset)
-   [Tools and Technologies](#tools-and-technologies)
-   [Experimental Methodology](#experimental-methodology)
-   [Reported Results](#reported-results)
-   [How to Run the Notebook](#how-to-run-the-notebook)
-   [Repository Contents](#repository-contents)
-   [Important Notes and Limitations](#important-notes-and-limitations)
-   [Future Scope](#future-scope)
-   [References](#references)

## Project Overview

Network Intrusion Detection Systems identify suspicious or malicious
activity in network traffic. Machine learning models can learn patterns
from historical traffic, but real network environments change as users,
applications, network behaviour, and attack techniques evolve.

A model that performs well when training and test records are randomly
mixed may not perform equally well on traffic collected later. This
project examines that issue through chronological evaluation,
feature-distribution analysis, and sequential model retraining.

The study uses the CICIDS2017 dataset and a Random Forest classifier to
distinguish **Benign** traffic from **Attack** traffic. It is an offline
research experiment, not a production or real-time intrusion detection
system.

## Research Question

**How does temporal change in network traffic affect the performance of
a machine learning-based intrusion detection system, and can model
adaptation reduce this performance degradation?**

## Objectives

1.  Develop a machine learning-based network intrusion detection model
    using CICIDS2017.
2.  Evaluate the model using a chronological train-test setup, in
    addition to a conventional random-split baseline.
3.  Measure changes in network traffic feature distributions using the
    Population Stability Index (PSI).
4.  Evaluate detection performance using Accuracy, Precision, Recall,
    F1-score, and confusion matrices.
5.  Examine whether sequential retraining using previously observed
    labelled traffic improves performance on later traffic periods.

## Dataset

This project uses **CICIDS2017**, published by the Canadian Institute
for Cybersecurity at the University of New Brunswick.

-   Dataset information: https://www.unb.ca/cic/datasets/ids-2017.html
-   Data type: labelled network-flow records
-   Target task: binary classification
-   Target labels used in the project:
    -   `0` --- Benign
    -   `1` --- Attack
-   Model input: 78 network traffic features, as described in the
    research report

The research report describes the combined Tuesday--Friday data used in
the main experiments as containing **2,300,825 records**: 1,743,179
benign and 557,646 attack records. These are figures reported by the
project documentation; verify them against the dataset and notebook
outputs if you reproduce the experiments.

Monday is excluded from model training in the documented methodology
because it contains benign traffic only. The report describes traffic
periods including Tuesday and Wednesday training traffic, followed by
Thursday Web Attacks, Thursday Infiltration, Friday Morning, Friday
PortScan, and Friday DDoS evaluation periods.

### Obtaining the data

1.  Visit the official CICIDS2017 dataset page linked above.
2.  Download the relevant Machine Learning CSV files required by the
    notebook.
3.  Store the files in Google Drive or another location that you can
    access from Colab.
4.  Check the notebook's data-loading cells for the expected file names
    and paths. Use the same paths or update only the path configuration
    in your own working copy if necessary.
5.  Do not upload the full dataset to this repository unless you have
    checked the dataset's applicable terms and confirmed that
    redistribution is permitted. The dataset is not included here.

## Tools and Technologies

-   **Python** --- programming language
-   **Google Colab** --- notebook execution environment
-   **Pandas and NumPy** --- data handling and numerical operations
-   **Scikit-learn** --- preprocessing, Random Forest classification,
    and evaluation metrics
-   **Matplotlib** --- data visualization
-   **CICIDS2017** --- network intrusion detection dataset
-   **Population Stability Index (PSI)** --- feature-distribution shift
    indicator

The exact library versions used in the original Colab session may differ
from the latest versions. No fixed environment or dependency lockfile is
supplied with the original notebook.

## Experimental Methodology

### 1. Data preprocessing

The documented workflow standardizes column names and labels, converts
infinite values to missing values, imputes missing numerical values
using median imputation, preserves duplicate records, retains the
original attack labels for reference, and uses 78 network traffic
features for training.

### 2. Random-split baseline

A stratified random sample of 500,000 records is used for a conventional
baseline experiment with an 80% training split and a 20% testing split.
This provides a reference for standard classification performance, but
it does not preserve a strict future-only evaluation boundary.

### 3. Chronological evaluation

The documented approach trains the model using Tuesday and Wednesday
traffic and evaluates it on later traffic periods without retraining the
model between evaluation periods. This is intended to approximate the
situation where a deployed model encounters later traffic.

The evaluation sequence described in the report is:

1.  Thursday Web Attacks
2.  Thursday Infiltration
3.  Friday Morning
4.  Friday PortScan
5.  Friday DDoS

### 4. Distribution-shift analysis with PSI

The Population Stability Index is used to measure differences between
feature distributions in the initial training period and later
evaluation periods. PSI is treated as an indicator of distributional
change; a high PSI value alone does **not** prove concept drift or
establish the cause of a performance change.

### 5. Sequential model adaptation

A second experiment periodically retrains the model using labelled data
from a completed evaluation period before moving to the next period. The
intent is to avoid using a future evaluation period for model updating
before that period has been evaluated.

### 6. Evaluation metrics

-   **Accuracy:** proportion of all predictions that are correct.
-   **Precision:** proportion of predicted attacks that are actually
    attacks.
-   **Recall:** proportion of actual attacks that are detected.
-   **F1-score:** harmonic mean of precision and recall.
-   **Confusion matrix:** breakdown of correct and incorrect
    classifications by class.

For intrusion detection, Recall and F1-score are especially important to
inspect alongside Accuracy because a model can achieve high overall
accuracy while missing many attacks.

## Reported Results

The following values are transcribed from the accompanying research
report. They are presented as **reported project results**, not as an
independent verification or guarantee that a fresh run will produce
identical values.

### Random-split baseline

  Metric        Reported score
  ----------- ----------------
  Accuracy              99.84%
  Precision             99.69%
  Recall                99.63%
  F1-score              99.66%

### Chronological evaluation of the static model

  Evaluation period         Accuracy   Precision   Recall   F1-score
  ----------------------- ---------- ----------- -------- ----------
  Thursday Web Attacks        98.71%      42.86%    2.06%      3.94%
  Thursday Infiltration       99.94%          0%       0%         0%
  Friday Morning              98.94%          0%       0%         0%
  Friday PortScan             44.56%      76.19%    0.09%      0.18%
  Friday DDoS                 79.10%      99.96%   63.18%     77.43%

These results illustrate why Accuracy should not be considered in
isolation. For example, the report records 99.94% Accuracy but 0% Recall
for Thursday Infiltration, meaning the model did not detect the attack
instances in that evaluation period according to the reported metrics.

### PSI analysis

  Evaluation period         Mean PSI   Features with substantial shift
  ----------------------- ---------- ---------------------------------
  Thursday Web Attacks        0.1633                                19
  Thursday Infiltration       0.2273                                33
  Friday Morning              0.1638                                21
  Friday PortScan             0.6553                                51
  Friday DDoS                 0.4465                                47

The report identifies Friday PortScan as the period with the largest
mean PSI and the most features showing substantial shift under the
project's analysis.

### Sequential retraining

  Evaluation period         Static F1-score   Adapted F1-score
  ----------------------- ----------------- ------------------
  Thursday Web Attacks                3.94%              3.94%
  Thursday Infiltration                  0%                 0%
  Friday Morning                         0%                 0%
  Friday PortScan                     0.18%              0.19%
  Friday DDoS                        77.43%             77.33%

According to the report, sequential retraining produced only small
changes in this experimental setup and did not substantially recover the
low detection performance observed in several periods.

## How to Run the Notebook

The notebook was developed for Google Colab and expects access to the
CICIDS2017 CSV data. Follow these steps to run it.

### Step 1 --- Open the repository

Open the GitHub repository and select the notebook file (for example,
`Concept_Drift_IDS.ipynb`).

### Step 2 --- Open it in Google Colab

In Colab, use **File → Open notebook → GitHub**, then paste the URL of
this repository and select the notebook. Alternatively, download the
`.ipynb` file from GitHub and upload it through Colab's notebook picker.

### Step 3 --- Provide dataset access

If the notebook uses Google Drive paths, mount Google Drive in Colab
when prompted and place the required CICIDS2017 files at the locations
expected by the notebook's data-loading cells. The notebook may not run
successfully until those paths and files are available.

### Step 4 --- Check the runtime

The project uses Pandas, NumPy, Scikit-learn, and Matplotlib. Colab
commonly provides these libraries, but versions can change. If an import
fails, install the missing package in the Colab runtime and restart the
relevant cells if required. Because the original notebook is preserved
unchanged, its exact setup may need to be recreated manually.

### Step 5 --- Run cells in order

Use **Runtime → Run all** only after confirming that the dataset paths
are correct. For a large dataset, execution may take time and require
substantial RAM. If a run stops, inspect the first error and verify the
file path, available memory, and expected CSV columns before continuing.

### Step 6 --- Review outputs

Review the baseline metrics, chronological evaluation tables, confusion
matrices, PSI analysis, sequential-retraining results, and any plots
produced by the notebook. Compare the output with the reported results
above; differences can occur if the data, sampling, library versions, or
execution state differs.

> **Reproducibility note:** The README describes the documented
> workflow, but the original notebook may rely on the author's Google
> Drive structure, pre-existing files, execution order, or saved cell
> outputs. A fresh run has not been certified by this README.

## Repository Contents

A simple repository layout for this project is:

``` text
Reserach_Project/
├── README.md
└── Concept_Drift_IDS.ipynb
```

The notebook filename can remain exactly as it is in your repository.
The layout above is illustrative; it does not indicate that files have
already been uploaded.

If you later decide to include selected result charts or exported metric
tables, add them in clearly named `results/` folders. Keep large raw
datasets, temporary files, and unnecessary model artifacts out of Git
unless there is a clear reason and permission to share them.

## Important Notes and Limitations

The following limitations are described in the research report:

1.  **No row-level timestamps:** the Machine Learning CSV files used in
    the project do not contain a Timestamp column. Temporal order is
    based on documented capture days and traffic-session sequence.
2.  **One primary model:** the final comparative analysis focuses on
    Random Forest rather than comparing multiple algorithms.
3.  **Previously unseen attack types:** later evaluation periods may
    contain attack types absent from the initial training data.
    Performance changes may therefore reflect both distributional
    changes and unfamiliar attack patterns.
4.  **PSI interpretation:** PSI measures feature-distribution change but
    does not independently establish concept drift.
5.  **Label availability:** the sequential retraining experiment assumes
    labelled data from a completed period becomes available before
    updating the model.
6.  **Dataset-specific findings:** the results are based on CICIDS2017
    and may not generalize to all network environments.
7.  **Offline experiment:** this project does not implement
    production-grade, real-time traffic monitoring or alerting.

## Future Scope

Potential extensions identified in the research report include:

-   Automated monitoring for distribution changes and performance
    degradation.
-   Online or incremental learning approaches for model adaptation.
-   A real-time intrusion detection pipeline.
-   Automated retraining triggered by drift or performance alerts.
-   Comparisons with algorithms such as XGBoost, SVM, and neural
    networks.
-   Evaluation on newer intrusion detection datasets.
-   An interactive dashboard for traffic statistics, PSI values, model
    metrics, detected attacks, and alerts.

## References

1.  Gama, J., Žliobaitė, I., Bifet, A., Pechenizkiy, M., &
    Bouchachia, A. (2014). A Survey on Concept Drift Adaptation. *ACM
    Computing Surveys, 46*(4), Article 44.
    https://doi.org/10.1145/2523813
2.  Lu, J., Liu, A., Dong, F., Gu, F., Gama, J., & Zhang, G. (2019).
    Learning under Concept Drift: A Review. *IEEE Transactions on
    Knowledge and Data Engineering, 31*(12), 2346--2363.
    https://doi.org/10.1109/TKDE.2018.2876857
3.  Breiman, L. (2001). Random Forests. *Machine Learning, 45*(1),
    5--32. https://doi.org/10.1023/A:1010933404324
4.  Ring, M., Wunderlich, S., Scheuring, D., Landes, D., & Hotho, A.
    (2019). A Survey of Network-based Intrusion Detection Data Sets.
    *Computers & Security, 86*, 147--167.
    https://doi.org/10.1016/j.cose.2019.06.005
5.  Sharafaldin, I., Lashkari, A. H., & Ghorbani, A. A. (2018). Toward
    Generating a New Intrusion Detection Dataset and Intrusion Traffic
    Characterization. In *Proceedings of the 4th International
    Conference on Information Systems Security and Privacy (ICISSP)*
    (pp. 108--116). https://doi.org/10.5220/0006639801080116
6.  Canadian Institute for Cybersecurity, University of New Brunswick.
    CICIDS2017 dataset. https://www.unb.ca/cic/datasets/ids-2017.html

------------------------------------------------------------------------

**Academic project:** MSc Information Technology, Pillai College of
Arts, Commerce & Science (Autonomous), New Panvel.\
**Project title:** Analysis of Concept Drift in Machine Learning-Based
Network Intrusion Detection.
