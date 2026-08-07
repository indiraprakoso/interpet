# InterPET: PETase Prediction — Dataset Curation & Model Ablation

This repository contains the notebooks used to curate a PET-degrading enzyme (PETase) classification dataset from four public sources, acquire corresponding 3D structures, and train/compare eight model ablations (1D sequence-based, graph-based, and structure-aware) for predicting PET-degrading activity.

## Repository Structure

```
.
├── notebooks/
│   ├── 1D_Dataset_Curation_Final.ipynb        # Curate training set & benchmark from 4 public sources
│   ├── 3D_Dataset_Acquisition_Final.ipynb     # Acquire/predict 3D structures per sequence
│   ├── Model1_ESM2_XGB_Final.ipynb            # 1D, ESM-2 embedding + XGBoost
│   ├── Model2_ESM2_RF_Final.ipynb             # 1D, ESM-2 embedding + Random Forest
│   ├── Model3_ProtT5_XGB_Final.ipynb          # 1D, ProtT5 embedding + XGBoost
│   ├── Model4_ProtT5_RF_Final.ipynb           # 1D, ProtT5 embedding + Random Forest
│   ├── Model5_AACCTD_XGB_Final.ipynb          # 1D, AAC/CTD handcrafted features + XGBoost
│   ├── Model6_AACCTD_RF_Final.ipynb           # 1D, AAC/CTD handcrafted features + Random Forest
│   ├── Model7_Graph1D_Chain_Final.ipynb       # 1D Graph (ESM-2 node feat. + sequence-adjacency edges) + GraphSAGE
│   ├── Model8_Graph1D3D_Contact_Final.ipynb   # 1D+3D Graph (ESM-2 node feat. + structure contact-map edges) + GraphSAGE
│   └── Model_Comparison_All.ipynb             # Aggregates results from Model #1-8 into comparison tables/charts
├── data/
│   └── README.md                              # Instructions for obtaining raw source data
├── datasets/                                  # Curated output (not committed if large — see .gitignore)
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

## Pipeline Overview

1. **`1D_Dataset_Curation_Final.ipynb`** — Aggregates sequences from PlasticDB, PAZy, PlasticEnz, and PED; defines positive (PET-active) and negative (non-PET-active) labels; removes duplicates, label conflicts, and benchmark leakage; filters near-duplicate sequences with CD-HIT-2D; and computes ESM-2 embeddings for class-separation visualization. Outputs `train_final.*` and `benchmark_final.*`.
2. **`3D_Dataset_Acquisition_Final.ipynb`** — For each curated sequence, retrieves an experimental structure from RCSB PDB when a high-identity match exists, otherwise falls back to a predicted structure.
3. **`Model1`–`Model8`** — Ablation studies comparing input representation (ESM-2 embedding, ProtT5 embedding, AAC/CTD handcrafted features, sequence graph, structure-aware graph) and classifier (XGBoost, Random Forest, GraphSAGE).
4. **`Model_Comparison_All.ipynb`** — Does not train any model; reads the saved artifacts from Model #1–#8 and compiles them into consolidated comparison tables, charts, and an Excel workbook.

## Data Sources

Raw data is **not included** in this repository, as it originates from third-party public databases. Download it directly from the sources below and place it according to the paths configured in `1D_Dataset_Curation_Final.ipynb` (see the "Configure data source paths" cell):

| Source | Required format | Notes |
|---|---|---|
| PlasticDB | `.fasta` | Header: `>ID\|enzyme\|organism\|substrates\|accession` |
| PAZy | `.csv` (`;`-delimited) | Sequence & substrates columns |
| PlasticEnz | `train.fasta` + `train_labels.tsv`, `test.fasta` + `test_labels.tsv` | Test split is reserved as an independent benchmark |
| PED | `.xlsx` | `category` and `degradation` columns |

## How to Run

These notebooks are built for **Google Colab** (they mount Google Drive via `google.colab.drive`). To run:

1. Open a notebook in Google Colab.
2. Adjust `PROJECT_ROOT` in the configuration cell to your Google Drive location, or change it to a local path if running outside Colab.
3. Place the raw data files under `PROJECT_ROOT/input/` following the structure above.
4. Run all cells in order (`Runtime > Run all`).
5. Run notebooks in pipeline order: dataset curation → 3D structure acquisition → Model #1–8 → Model Comparison (the comparison notebook depends on artifacts saved by each model notebook).

### Running locally (non-Colab)

Remove/skip the `drive.mount(...)` cell and replace Google Drive paths with local paths. Make sure `cd-hit` is installed on your system (see Dependencies).

## Dependencies

See `requirements.txt` for the full list of Python packages. The dataset curation notebook additionally requires **CD-HIT** (specifically `cd-hit-2d`) for sequence similarity filtering:

```bash
# Ubuntu/Debian
apt-get install -y cd-hit

# or via conda
conda install -c bioconda cd-hit
```

If CD-HIT is not available, the notebook still runs, but the similarity-filtering steps (negative-vs-positive and train-vs-benchmark) will be skipped.

## Output

| File | Description |
|---|---|
| `datasets/train_final.fasta` | Final curated training sequences (FASTA) |
| `datasets/train_final_labels.csv` | Final training labels & metadata |
| `datasets/benchmark_final.fasta` | Independent benchmark sequences (FASTA) |
| `datasets/benchmark_final_labels.csv` | Benchmark labels & metadata |
| `datasets/X_train_embeddings.npy` | ESM-2 embeddings, training set (optional, large) |
| `datasets/X_benchmark_embeddings.npy` | ESM-2 embeddings, benchmark set (optional, large) |
| Model #1–8 artifacts | Trained model outputs, metrics, predictions (per-notebook) |
| Model comparison outputs | Consolidated comparison tables/charts/Excel workbook |

Large files (embeddings, trained model weights) should not be committed to plain git — use Git LFS or external hosting (e.g., Zenodo) if they need to be shared.

## Citation

If you use this dataset or code, please cite the original data sources:

1. Gambarini V, Pantos O, Kingsbury JM, Weaver L, Handley KM, Lear G. PlasticDB: a database of microorganisms and proteins linked to plastic biodegradation. *Database*. 2022;2022. https://doi.org/10.1093/DATABASE/BAAC008
2. Buchholz PCF, Feuerriegel G, Zhang H, Perez-Garcia P, Nover LL, Chow J, et al. Plastics degradation by hydrolytic enzymes: The plastics-active enzymes database—PAZy. *Proteins: Structure, Function and Bioinformatics*. 2022;90:1443–56. https://doi.org/10.1002/PROT.26325
3. Krzynowek A, Snoeks J, Faust K. PlasticEnz: An integrated database and screening tool combining homology and machine learning to identify plastic-degrading enzymes in meta-omics datasets. *PLoS Comput Biol*. 2026;22:e1013892. https://doi.org/10.1371/JOURNAL.PCBI.1013892
4. Jiang R, Shang L, Wang R, Wang D, Wei N. Machine Learning Based Prediction of Enzymatic Degradation of Plastics Using Encoded Protein Sequence and Effective Feature Representation. *Environ Sci Technol Lett*. 2023;10:557–64. https://doi.org/10.1021/ACS.ESTLETT.3C00293

## License

This project is licensed under **CC BY-NC 4.0** (Attribution-NonCommercial). You are free to share and adapt this work for non-commercial purposes, provided appropriate credit is given to the author(s). Commercial use is not permitted without explicit permission. See `LICENSE` for full terms.
