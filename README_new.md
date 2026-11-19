# CPBSpred: Predicting Capsid-Protein Binding Sites in Viral RNA

CPBSpred is a machine-learning proof of concept for identifying **capsid-protein binding sites in single-stranded viral RNA** from local RNA backbone geometry. The project uses bacteriophage **Qβ** as a model system and represents a five-nucleotide RNA fragment with five geometric descriptors: three backbone angles and two dihedral angles. These features are used to train a small neural network to distinguish capsid-contacting from noncontacting fragments.

The study also investigates whether the classifier retains useful predictive information when RNA geometry is obtained from a computationally predicted structure instead of experimentally determined coordinates. Such predictions help study RNA–capsid recognition; they do **not** directly simulate capsid assembly or prove that a particular RNA will be packaged.

## Repository contents

| File | Description | How it is used |
| --- | --- | --- |
| [`RNA_NN_test_v3.ipynb`](RNA_NN_test_v3.ipynb) | Google Colab notebook containing data loading, neural network training, evaluation plots, testing on another CSV dataset, and decision-threshold analysis. | **Run this notebook** to reproduce the analyses implemented in the publicly available code. |
| [`7lhdRNA_LSMdataset.csv`](7lhdRNA_LSMdataset.csv) | Labeled RNA-fragment geometry dataset with columns `th1`, `th2`, `th3`, `fi1`, `fi2`, and `label`, as shown in the notebook. | **Upload at the first prompt**; the notebook explicitly loads this filename for training and an 80/20 train/test split. |
| [`qbeta_5nt_fragment_geometry_binary_only.csv`](qbeta_5nt_fragment_geometry_binary_only.csv) | Additional Qβ five-nucleotide-fragment geometry dataset. | Can be selected at the **second upload prompt** to evaluate the trained model, provided its columns follow the expected format. It is not automatically loaded by the first training section. |
| [`README.md`](README.md) | Overview, file descriptions, instructions, and interpretation notes. | Start here. |

> **Important:** The repository contains processed geometry tables and a classification notebook, not a complete pipeline that starts from an arbitrary raw RNA sequence or PDB file. The geometric-feature extraction workflow must be provided separately to process new raw structural data.

## Input data format

Each example represents a local **five-nucleotide RNA fragment**. The model uses five numerical features:

- `th1`, `th2`, `th3`: three consecutive RNA-backbone packing angles.
- `fi1`, `fi2`: two consecutive RNA-backbone dihedral angles.
- `label`: the binary ground-truth class (capsid-contacting versus noncontacting).

The notebook shows the following CSV column order:

```csv
th1,th2,th3,fi1,fi2,label
137.999,81.637,171.756,4.535,167.162,1
```

The notebook converts labels encoded as `-1`/`1` into `0`/`1`; binary `0`/`1` labels also remain unchanged. For its **additional-dataset evaluation section**, the code reads the **first five columns as features** and the **last column as the label**. Check the column order and class definitions before evaluating a new file. The CSV example above illustrates the expected *format*, not the complete dataset.

## Quick start: run in Google Colab (recommended)

The notebook was written for **Google Colab**, including interactive upload and download commands. No local installation is needed for the usual Colab environment.

1. Open the notebook in Colab: **[Open `RNA_NN_test_v3.ipynb` in Google Colab](https://colab.research.google.com/github/melody144/CPBSpred/blob/main/RNA_NN_test_v3.ipynb)**.
2. Connect to a Python runtime. A standard CPU runtime is sufficient for this small feedforward network.
3. Start running the cells **from top to bottom**. At the **first** `files.upload()` prompt, choose `7lhdRNA_LSMdataset.csv` from this repository.
4. Continue through the data preparation and training cells. The notebook uses a five-input, ten-hidden-unit, one-output neural network implemented in PyTorch, with binary cross-entropy loss and the Adam optimizer. Its current training section makes an **80/20 train/test split** (`random_state=42`) and runs up to **1,000 training epochs**.
5. Review the training and test-set results, including classification accuracy, an ROC curve, and a confusion matrix.
6. When you reach the **second** `files.upload()` prompt, upload an additional labeled CSV such as `qbeta_5nt_fragment_geometry_binary_only.csv` (after checking that its first five columns contain the expected features and its final column is a compatible binary label). This section evaluates the **already trained** model; it does not retrain it on the uploaded file.
7. Continue to view the additional-dataset classification metrics and plots. The notebook saves the predictions as **`qbeta_model_predictions.csv`** and triggers a Colab download.
8. Run the optional threshold-analysis cells to compare metrics across decision thresholds. These save **`threshold_optimization_results.csv`** and trigger another download.

**Practical tip:** Running **Runtime → Run all** may pause at each of the two file-upload prompts. If the notebook loses its runtime session, run the earlier training cells again before using the later evaluation cells because they reference the in-memory `model` object.

### Files produced by the notebook

| Output | Contents |
| --- | --- |
| Notebook plots and printed metrics | Training loss, accuracy, ROC analysis, confusion matrices, precision–recall analysis, and related classification summaries. |
| `qbeta_model_predictions.csv` | Rows from the second uploaded dataset plus columns for the true class, predicted probability, and predicted class. The output name is hard-coded in the notebook. |
| `threshold_optimization_results.csv` | Classification metrics evaluated over different probability cutoffs, used to examine the precision/recall tradeoff. |

These two output CSV files are **generated when the notebook runs**; they are not inputs included in the current repository.

## Running outside Google Colab

The analysis uses **Python 3**, **PyTorch**, **NumPy**, **pandas**, **scikit-learn**, and **Matplotlib**. For a local Jupyter installation, install the dependencies in your Python environment:

```bash
python -m pip install notebook torch numpy pandas scikit-learn matplotlib
```

Then launch Jupyter:

```bash
jupyter notebook RNA_NN_test_v3.ipynb
```

**Local adjustment required:** The published notebook uses Colab-only commands (`from google.colab import files`, `files.upload()`, and `files.download()`). To run outside Colab, replace each upload step with a normal local CSV read, for example:

```python
import pandas as pd

df = pd.read_csv("7lhdRNA_LSMdataset.csv")
# At the later evaluation step:
df_test = pd.read_csv("qbeta_5nt_fragment_geometry_binary_only.csv")
```

The `files.download(...)` calls can be removed locally; generated CSV files will remain in the notebook's working directory. Other notebook cells assume that earlier cells have defined `model`, `X_test`, and related variables, so maintain execution order. These local modifications have not been packaged as a standalone script.

## Published findings and reproducibility notes

The accompanying study reports an **ROC AUC of 0.88** for RNA–capsid contact-site classification using experimentally determined structure-based geometric features with **five-fold cross-validation**, and an **ROC AUC of 0.75** when evaluating features obtained from a computationally predicted RNA structure.

**Distinguish the reported study from the current public notebook:**

- The notebook's visible primary training workflow performs a **single 80/20 holdout split**, not the five-fold cross-validation described in the study. Consequently, running this notebook as written should **not** be presented as an exact reproduction of the reported cross-validation AUC.
- The notebook uses precomputed five-feature CSVs; it does not independently regenerate those descriptors from cryo-EM or predicted atomic structures.
- The second upload section depends on the fitted model remaining in memory; the repository does not currently provide a separate saved trained-model checkpoint.
- Choosing a probability threshold based on the same dataset used to report its final performance can bias that estimate. For an unbiased assessment, select the threshold on separate validation data before testing on a held-out dataset.
- Experimental contact labels and a computational prediction of contacts are not the same as direct evidence of RNA packaging or viral assembly.

## Research reference

*Predicting Capsid Protein Binding Sites in Single-Stranded RNA Viruses Using Machine Learning from Local Geometric Features.* bioRxiv preprint (2026). DOI: [10.64898/2026.09.10.750674](https://doi.org/10.64898/2026.09.10.750674).

When using this repository, please cite the associated preprint. For further reproducibility, future repository updates could include the exact five-fold cross-validation code, a requirements file with tested package versions, and a script to extract geometric descriptors directly from structural coordinates.
