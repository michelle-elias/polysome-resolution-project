# Polysome Resolution Project

**Investigating Effects of Polysome Profiling Resolution for a Translation Reporter Assay with Deep Learning**

Systems Biology: Systems Biology: Computational Analysis and Interpretation of High-throughput Data

Final Project (2025/2026)

Humboldt-Universität zu Berlin

---

## Overview

This project investigates how reducing the resolution of polysome profiling affects the ability of a convolutional neural network (CNN) to predict mean ribosome load (MRL) from 5′ UTR sequences.

Polysome profiling separates mRNAs by ribosome occupancy via sucrose density gradient centrifugation. The original dataset uses 14 fractions. We simulate lower-resolution experiments by merging adjacent fractions into coarser schemes (7, 5, and 3 bins) and ask: does reducing experimental resolution hurt model performance?

**Reference:** Sample, P.J. et al. Human 5′ UTR design and variant effect prediction from a massively parallel translation assay. *Nature Biotechnology* 37, 803–809 (2019). [GSM3130435](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM3130435)

---

## Key Findings

- **Resolution matters only little for prediction performance.** A CNN trained on 3-bin labels achieves nearly identical test PCC (~0.945) as one trained on 14-bin labels.
- **The label ceiling is ~1.0 across all schemes.** Merging bins preserves sequence rankings almost perfectly. The information loss from coarser binning is negligible.
- **Resolution changes what the model learns.** Attribution analysis via Integrated Gradients shows that 14-bin models learn fine-grained positional motifs, while 3-bin models rely more on global nucleotide composition. Cross-resolution profile correlation drops to r = 0.587 (14 vs 3 bins).
- **Biological signal is recoverable.** Attribution logos show consistent patterns across resolutions: G and C suppress translation (likely via secondary structure), A promotes it, and the Kozak-proximal region (positions ~45–49) shows the strongest positional signal.

---

## Repository Structure

```
polysome-resolution-project/
├── code/
│   ├── polysome_resolution_project.ipynb   # Main notebook (Google Colab, exported)
│   └── polysome_resolution_project.py      # Exported Python script
├── results/
│   ├── arch_search_results.png             # Architecture search: val PCC per hyperparameter
│   ├── loss_curve.png                      # Training loss curve (final retrain)
│   ├── mrl_distribution.png                # MRL distribution (14-bin, n=280,000)
│   ├── mrl_distributions_per_scheme.png    # MRL distributions across resolution schemes
│   ├── evaluation_pcc_vs_resolution.png    # Main result: PCC vs resolution + label ceiling
│   ├── attribution_comparison.png          # IG positional profiles + cross-resolution correlation
│   ├── attribution_logo_all_schemes.png    # Sequence logos per resolution scheme
│   └── per_nucleotide_importance_all_schemes.png  # Signed IG attributions per nucleotide
└── README.md
```

---

## Methods Summary

### Data
- 280,000 synthetic 50-nt 5′ UTR sequences (top by read depth) from Sample et al. (2019)
- MRL computed via two-step normalization following Supplementary Note 1 of the original paper
- Resolution schemes: 14 bins (full), 7 bins (paired merge), 5 bins, 3 bins (coarse)

### Model
- 1D CNN: 3 × (Conv1d + ReLU) → Flatten → FC head → scalar MRL output
- Architecture selected via search over 15 configurations; fixed across all resolution experiments
- Trained on low-resolution MRL labels; evaluated against high-resolution 14-bin MRL on held-out test set
- 5-fold cross-validation on 224,000 sequences; 56,000 held-out test sequences

### Evaluation
- **Label ceiling:** PCC(low-res MRL, high-res MRL) on test set. Upper bound imposed by bin merging, independent of any model
- **Model gap:** ceiling − CNN PCC; indicates whether the bottleneck is resolution or model capacity
- **Attribution analysis:** Integrated Gradients (Captum) on best-fold models; positional importance profiles compared across resolutions

---

## Requirements

The notebook runs on **Google Colab** (GPU recommended; ~60 min on NVIDIA A100).

```
torch
numpy
pandas
scipy
scikit-learn
matplotlib
captum
logomaker
```

The raw data CSV (`GSM3130435_egfp_unmod_1.csv`) must be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM3130435) and placed in the project data directory.