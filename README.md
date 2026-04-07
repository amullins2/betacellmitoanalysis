# Pancreatic Islet Single-Cell Segmentation Pipeline

## Overview
A Python pipeline for automated single-cell segmentation and mitochondrial 
protein quantification in human pancreatic tissue from large-scale confocal 
images. Compares VDAC1 (mitochondrial mass) and MTCO1 (Complex IV) expression 
between islet beta cells and extra-islet beta cells at single-cell resolution.


## Pipeline Overview

## Requirements
```bash
conda create -n islet_pipeline python=3.11
conda activate islet_pipeline
pip install cellpose tifffile scikit-image scikit-learn scipy 
pip install pandas numpy matplotlib seaborn tifffile
```

## Input Data
- **Format**: Multi-channel TIFF (4 channels)
- **Channels**: CH1=VDAC1 (AF488), CH2=MTCO1 (AF647), 
                CH3=Insulin (AF405), CH4=E-cadherin (AF568)
- **Acquisition**: Zeiss LSM980, 20×, pixel size 0.1235 µm/px

## Pipeline Steps

### 1. Cell Segmentation
```python
# Tiled Cellpose segmentation
# TILE_SIZE=4000px, OVERLAP=400px, DS=1 (full resolution)
# Model: cyto3, diameter=100px, flow_threshold=0.8, cellprob_threshold=-1.0
# Input: E-cadherin (primary) + Insulin (secondary)
# Output: all_cells_raw.csv (76,628 cells)
# Runtime: ~34 minutes on Apple M4 Max (Metal GPU)
```

### 2. Beta Cell Classification
```python
# Otsu threshold on insulin_mean distribution
# Threshold: 14.9 AU
# Beta cells: 3,716 / 76,628 total cells
```

### 3. Islet Detection
```python
# DBSCAN on insulin+ cell coordinates
# eps=200px (~24.7µm), min_samples=10
# 51 clusters detected → 7 ducts excluded (visual inspection)
# Valid islets: 44
```

### 4. Extra-Islet Beta Cell Identification
```python
# Multi-criteria filter:
# - insulin_mean > 27.9 AU (islet 75th percentile)
# - distance from nearest islet > 150 µm
# - area: 1,232-7,787 px² (islet beta cell 10th-95th percentile)
# - insulin contrast ratio > 4.0
# - spatially isolated (no neighbour within 150px)
# Candidates: 38 → 20 after manual validation
```

### 5. Statistical Analysis
```python
# Comparison: 49 per-islet means vs 20 individual extra-islet cells
# Test: two-sided Mann-Whitney U

## Output Files

| File | Description |
|------|-------------|
| `all_cells_raw.csv` | All 76,628 segmented cells with intensity measurements |
| `all_cells_final.csv` | Cells with classification labels |
| `FINAL_tissue_map.png` | Tissue overview with islet and extra-islet annotations |
| `FINAL_violin_plots.png` | VDAC1/MTCO1/Insulin comparison figures |
| `exemplar_extraislet_final_v2.png` | Exemplar extra-islet beta cell |
| `manual_inspection_5ch.png` | Manual validation gallery |
| `computational_methods.txt` | Full methods text for publication |

## Key Parameters

| Parameter | Value | Justification |
|-----------|-------|---------------|
| Cellpose diameter | 100 px | ~12.4 µm, appropriate for pancreatic cells at 20× |
| DBSCAN eps | 200 px | ~24.7 µm max gap between adjacent islet beta cells |
| DBSCAN min_samples | 10 | Minimum beta cells to define an islet |
| Insulin threshold | 27.9 AU | 75th percentile of islet beta cell insulin (conservative) |
| Min distance from islet | 150 µm | Excludes peri-islet beta cells |
| Extra-islet isolation radius | 150 px | ~18.5 µm, excludes micro-clusters |

## Citation
If you use this pipeline, please cite:
- Cellpose: Stringer et al., Nature Methods 2021
- Granlund et al., Acta Diabetol 2024 (extra-islet beta cell density reference)

## Author
Alana Mullins


readme_path = OUTPUT_DIR / "README.md"
with open(readme_path, 'w') as f:
    f.write(readme_text)
print(f"Saved: {readme_path}")
print(f"Saved: {methods_path}")
print("\nBoth documents ready for use!")
