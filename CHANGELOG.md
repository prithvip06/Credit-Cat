# Changelog

## Portfolio cleanup — July 2026
Personal pass after the datathon, extending the original submission for a portfolio (see `PERSONAL_NOTE.md`).
- Filled in `MODEL_REPORT.md` Section 7 (Column Name Assignments), which shipped blank — the underlying analysis already existed in `notebooks/Variable_Justification.ipynb` but never made it into the report before the deadline.
- Found and fixed a labeling bug: the saved model bundle's `friendly` dict called A13 `"Zip Code (uncertain)"`, contradicting both `app.py`'s actual UI text and the Variable Justification analysis, both of which support "Credit Balance." Corrected in the model artifact and in `backend.py`.
- Corrected the PCA variance figure in `MODEL_REPORT.md` from an approximate "~35%" to the precise "~39%" (25.6% + 13.1%).
- Deduplicated four identical copy-pasted cells in `Variable_Justification.ipynb`.
- Made `Clustering_Story.ipynb` self-contained: it previously loaded data from a hardcoded URL pointing at the original repo; now reads the local file. Also synced the model-saving cell so it produces the exact same bundle keys as the shipped `models/cluster_model.joblib` (it was previously missing `friendly`, `pca`, `train_pca_coords`, `train_labels`, and `cluster_distances`), and guarded the Colab-only download cell so it doesn't error when run locally.
- Removed committed `__pycache__` and `.DS_Store`; added `.gitignore`.
- Removed personal phone numbers from the README; added this changelog (previously referenced but missing) and `PERSONAL_NOTE.md`.

## Datathon build — May 22–23, 2026
Built over two days for CSULB Datathon 2026, Pillar 2 (Unsupervised ML). Team: Emmet Gingerich & Prithvi Prasad.
- Initial data load, first K-Means clustering pass, and app scaffold.
- Full Streamlit UI, built across three iterations, plus branding and dashboard polish.
- Recommendation ("what moves the needle") algorithm.
- Full variable justification for all 14 anonymized columns and the cluster-naming/insights narrative (`notebooks/Variable_Justification.ipynb`, `notebooks/Clustering_Story.ipynb`).
- Final report, Responsible AI statement, and submission cleanup.
