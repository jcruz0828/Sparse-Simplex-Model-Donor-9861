# Sparse Simplex Model — Brain Connectivity for Donor 9861

This repository implements the Sparse Simplex Learning Model and LASSO-based graph construction to analyze gene expression similarity across brain regions using the Allen Human Brain Atlas (AHBA) for Donor 9861. The resulting graphs represent spatial and genetic relationships between 906 sampled brain regions.

We explore two graph construction methods:
1. LASSO (using PCA-reduced gene data)
2. Sparse Simplex (using full standardized gene data)

## Methodology

### Data
- Source: Allen Human Brain Atlas
- Donor: 9861
- Regions: 906
- Genes: 15,636
- Atlas: Desikan-Killiany (donor-specific, MNI-aligned via abagen)

### Preprocessing
- Gene expression matrix: 906 regions × 15,636 genes
- Probe selection: `max_variance`
- Two forms of input used:
  - PCA-reduced to 215 components (capturing 90% of variance)
  - Standardized only (z-scored across genes)

## Graph Construction

### 1. LASSO-Based Similarity (PCA Input)
- Input shape: 906 × 215
- Each region reconstructed using sparse linear regression (LASSO, α = 0.01)
- Fully parallelized with joblib for 16-core CPU usage
- Low RMSE and full uniqueness across all 906 rows

### 2. Sparse Simplex Model (Original Paper Method, Standardized Input)
- Input shape: 906 × 15,636 (full gene set, standardized)
- Optimization:
  - Accelerated Projected Gradient Descent
  - Newton-based projection onto the probability simplex
- Despite high dimensionality, this method converged faster than LASSO
- However, it produced fewer unique adjacency rows (380 out of 904), suggesting potential redundancy in similarity encoding

## Results

| Method               | Input Dim        | Min RMSE | Max RMSE | Avg RMSE | Unique Rows |
|---------------------|------------------|----------|----------|----------|--------------|
| LASSO (PCA)          | 906 × 215        | 0.00280  | 0.00954  | 0.00363  | 904 / 904    |
| Sparse Simplex (Std)| 906 × 15,636     | 0.00280  | 0.00954  | 0.00363  | 380 / 904    |

- Both methods achieved comparable RMSE, but PCA input preserved more diversity.
- The Sparse Simplex model may cluster many rows due to the simplex constraint structure, causing reduced uniqueness despite similar reconstruction error.

## System Requirements

- Requires a multi-core CPU (≥16 cores recommended)
- Both LASSO and Sparse Simplex methods are parallelized using `joblib`
- Performance estimates on 16-core machine:
  - LASSO (PCA): ~1–2 minutes
  - Sparse Simplex (full genes): ~10–12 minutes

## Visualization

- Visualizations include:
  - 2D anatomical slice overlays with connections (matplotlib + nibabel)
  - Connectome graphs with node projections onto MNI-aligned brain slices
- All node coordinates were mapped in real voxel space using donor-native atlases
- Center slices in the x/y/z planes were selected to resemble those in the original paper



## Citation

If you use this work, please cite:

- Heng Huang et al. (2023). *A New Sparse Simplex Model for Brain Anatomical and Genetic Network Analysis*. IEEE ICKG.
- Markello et al. (2021). *abagen: A toolbox for mapping gene expression to the brain*. Nature Methods.

## Installation

Install dependencies using the provided `requirements.txt` file:

```bash
pip install -r requirements.txt

