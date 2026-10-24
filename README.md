# Genotype-Phenotype Mapping in Cystic Fibrosis

MSc dissertation project, Queen Mary University of London.

Cystic fibrosis (CF) comes from variants in a single gene, CFTR, yet severity varies a lot. This project asks how much of a patient's clinical picture can be predicted from variant-level lab and computational features, and how much of that skill survives a strict, leakage-controlled evaluation.

## Data

The Sosnay et al. (2013) phenotype summary table: 159 CFTR variants and about 39,696 patients. Each row is one variant, with averaged clinical outcomes (sweat chloride, pancreatic insufficiency, FEV1, Pseudomonas infection), lab measurements (chloride conductance, protein processing) and in-silico scores (POSE, PolyPhen-2, SIFT, PANTHER). 156 variants have complete clinical targets and are used for modelling.

The table is not included in this repo. Get it from the supplementary material of Sosnay et al., *Nature Genetics* 45(10), 1160-1167, export it to CSV, and upload it when the notebook asks.

## What the notebook does

1. **Cleaning:** merged two-row header, hidden non-breaking spaces in variant names, censored values such as "<10" stored as text, one-hot variant type, ordinal PolyPhen-2.
2. **Exploration:** outcome distributions by clinical call, correlations, functional features for missense variants.
3. **Unsupervised stage:** PCA, UMAP, K-means (k = 2 to 8) and Ward hierarchical clustering, compared with the known clinical calls.
4. **Single-allele prediction:** Random Forest for sweat chloride, pancreatic insufficiency and FEV1 (one model per target), leave-one-out cross-validation, permutation importance.
5. **Model families:** the same task repeated with CatBoost and TabPFN (v2).
6. **Genotype pairs:** every unordered pair of the 156 variants (12,246 pairs). Alleles are described by their minimum and maximum feature values, so a pair reads the same in either order. Pair targets follow a minimum-function rule (the milder allele sets residual function).
7. **Leakage control:** variants, not pairs, are split into 5 folds. A pair is scored only when both alleles sit in the held-out fold, and it is trained on pairs containing neither. This scores 2,512 of the 12,246 pairs.
8. **Held-out genotypes:** F508del, G551D and R117H are removed from training and their genotypes are predicted.
9. **Severity heatmap:** predicted sweat chloride for pairs of the 20 most frequent variants.

## Results (Random Forest, R²)

| Target | Single allele (LOOCV) | Genotype pair (variant split) |
|---|---|---|
| Sweat chloride | 0.506 | 0.328 |
| Pancreatic insufficiency | 0.545 | 0.271 |
| FEV1 | -0.037 | -0.495 |

- Sweat chloride and pancreatic insufficiency are moderately predictable. FEV1 is not, which fits what is known about lung disease: it depends on modifier genes, infection and treatment, not on the variant alone.
- An earlier version that split pairs at random reported a sweat chloride R² of about 0.85. That was allele memorisation, because the same alleles appeared in both training and test pairs. That run is not in this notebook.
- CatBoost and TabPFN show the same pattern, so it comes from the data and not from one algorithm.
- Predictions are pulled toward the middle of the range, so very severe genotypes are under-predicted and mild ones over-predicted.

## Limitations

- Only 156 variants, with few mild ones.
- Every value is a variant average, not a patient reading, so the models describe groups and not individuals.
- Lab features exist only for missense variants and are filled with the median for the rest.
- The pair targets are computed from the single-allele values, so pairing adds no new biological information.
- The predicted severity table is an illustration, not a validated clinical tool.

## Running it

The notebook was written for Google Colab and loads the data with `files.upload()`. Run the cells in order in Colab and upload the CSV when asked. Figures are saved by the notebook as PNG files.

To run it locally:

```bash
pip install -r requirements.txt
jupyter notebook Dissertation.ipynb
```

and replace the upload cell with `pd.read_csv("your_file.csv", header=None)`.

## Tech

Python, pandas, NumPy, scikit-learn, CatBoost, TabPFN, UMAP, SciPy, Matplotlib, Seaborn

## Author

Elidian Terziu
[LinkedIn](https://linkedin.com/in/elidian-terziu)
