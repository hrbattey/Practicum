# Revision results summary v2 — primary operating point specificity ≥ 0.70 (WHO triage TPP)

Generated automatically from `Code/Practicum_REVISION.ipynb` (Restart & Run All, 0 errors) with the primary operating point changed from specificity ≥ 0.80 to ≥ 0.70. All values are read programmatically from the `rev_*.csv` files in `Output/csv` and compared with the previous run (specificity ≥ 0.80), archived in `Output/v1_spec080/`. Layout follows `REVISION_RESULTS_SUMMARY.md` (v1).

**Flags used throughout:**
- `† [was …]` — value changed from v1; the v1 value is shown in brackets.
- `(new)` — row not present in v1.
- *Verification* lines — columns that should not change were compared with v1 cell by cell (✓ = identical).
- Values without a flag are identical to v1.

## A. What changed in v2 and what did not

- **Operating point:** every threshold is now chosen to maximise sensitivity subject to specificity ≥ 0.70 on pooled 5-fold out-of-fold training predictions (highest-specificity point if none qualifies) and applied unchanged to the test set. Rationale: WHO target product profile for a TB triage test (sensitivity ≥ 90 %, specificity ≥ 70 %).
- **Changes:** sensitivity, specificity, PPV, NPV and thresholds of every model; the nested-CV full-feature comparison (Table 2 / Figure 2), which re-tunes hyperparameters for sensitivity at specificity ≥ 0.70; the specificity-target sweep (0.70 added; Supplementary Figure 2).
- **Does not change (verified):** data, split (636 / 159), seed (42), feature selection and every fitted AUC-tuned model (M1–M5, full-feature, top-k, resampled, sensitivity re-runs) — hence ROC-AUC, PR-AUC, Brier, recalibrated Brier, feature lists, SHAP, permutation importance, Table 1 and all descriptive tables. The notebook refits these models from the same seed; the refit is deterministic and reproduces v1 exactly (Section B).

## B. Verification that the fitted tiered models are unchanged

| Metric (M1–M5, both algorithms, test) | Expected | Max |v2 − v1| estimate | Max |v2 − v1| CI bound | Result |
|---|---|---|---|---|
| AUROC | unchanged | 0.00e+00 | 0.00e+00 | ✓ identical |
| AP | unchanged | 0.00e+00 | 0.00e+00 | ✓ identical |
| Brier | unchanged | 0.00e+00 | 0.00e+00 | ✓ identical |
| Sens | changes with threshold | 1.71e-01 | 1.78e-01 | changed |
| Spec | changes with threshold | 1.45e-01 | 1.59e-01 | changed |
| PPV | changes with threshold | 1.47e-01 | 1.58e-01 | changed |
| NPV | changes with threshold | 3.76e-02 | 4.01e-02 | changed |

- Feature selection logs and SHAP rankings (`rev_rfe_flow_*.csv`, `rev_topk_shap_ranking_*.csv`): identical ✓.
- Probability diagnostics of every fitted model (trees, prediction range, Brier, calibration; `rev_probability_diagnostics.csv`): identical ✓.

## C. WHO triage TPP check — M5 on the test set (n = 159, 35 TB)

| Algorithm | Model | Threshold | Sens (95% CI) | Spec (95% CI) | Meets TPP (point estimates) | Meets TPP (lower 95% bounds) | TPP within 95% CIs |
|---|---|---|---|---|---|---|---|
| XGBoost | Model 5 | 0.1303 | 0.800 (0.631, 0.916) | 0.710 (0.621, 0.788) | False | False | True |
| CatBoost | Model 5 | 0.1671 | 0.943 (0.808, 0.993) | 0.710 (0.621, 0.788) | True | False | True |
- **XGBoost M5:** sensitivity 0.800 (0.631, 0.916), specificity 0.710 (0.621, 0.788). Meets sensitivity ≥ 0.90 and specificity ≥ 0.70 on point estimates: **no**; on lower 95 % bounds: **no**; TPP values within the 95 % CIs: **yes**.
- **CatBoost M5:** sensitivity 0.943 (0.808, 0.993), specificity 0.710 (0.621, 0.788). Meets sensitivity ≥ 0.90 and specificity ≥ 0.70 on point estimates: **yes**; on lower 95 % bounds: **no**; TPP values within the 95 % CIs: **yes**.

Sens/spec CIs are exact Clopper–Pearson. Thresholds were fixed on training data before the test set was scored. File: `rev_who_tpp_check_M5.csv`.

## 0. How to read the numbers

- **Data / split (unchanged):** n = 795; stratified 80/20 split, seed 42 → 636 train (141 TB) / 159 test (35 TB).
- **Threshold (changed):** specificity ≥ **0.70** on pooled 5-fold out-of-fold training predictions, maximising sensitivity; applied unchanged to the test set (test specificity can differ from 0.70).
- **95 % CIs (test set, threshold fixed):** AUROC, AP and Brier — percentile bootstrap, 2,000 resamples; sensitivity, specificity, PPV, NPV — exact Clopper–Pearson.
- **Null Brier** = Brier of predicting the training TB prevalence for everyone (≈ 0.172). **Brier (recal.)** = after a logistic recalibration fitted on training out-of-fold predictions.
- Seed 42 everywhere. Model definitions as in v1 (Section 1 of v1).

## 1. Changes to the analysis since submission (Methods text)

All changes listed in v1 Section 1 still apply (TB-history leakage removed; XGBoost class-weighting bug fixed; one M1–M5 model set; single seed; CIs; Figure 5 labels). **New in v2:** primary operating point specificity ≥ 0.70 (WHO triage TPP) instead of ≥ 0.80; nested-CV comparison re-tuned at ≥ 0.70; mean fold threshold reported; specificity sweep extended to 0.70.

## 2. Study population — Table 1 (n = 795) — unchanged

*Verification:* Table 1, outcome composition, HIV and age-group tables identical to v1 ✓. Values: see v1 Section 2 (`rev_table1_analytic_sample.csv`; HIV 7/795 = 0.88 %; TB disease 162 bacteriologically confirmed + 14 clinician-diagnosed only).

## 3. Full-feature nested CV — Table 2 / Figure 2 (re-tuned at specificity ≥ 0.70)

Nested 5 × 3 CV within the 636 training rows; inner Optuna objective = mean sensitivity at specificity ≥ 0.70; per-fold thresholds at ≥ 0.70; pooled out-of-fold predictions; 2,000-bootstrap CIs. All values are expected to change (hyperparameters re-tuned). File: `rev_fig2_summary_metrics_XGB_vs_CAT_bootstrap_pooled.csv`; Figure 2 `rev_fig_ROC_PR_Calibration_Hists_XGB_left_CAT_right.png` (operating-point dots, threshold lines, histograms regenerated).

| Metric | CatBoost | XGBoost |
|---|---|---|
| Sens | 0.801 (0.734, 0.864) † [was 0.816 (0.752, 0.875)] | 0.844 (0.782, 0.900) † [was 0.780 (0.711, 0.844)] |
| Spec | 0.786 (0.750, 0.820) † [was 0.816 (0.781, 0.848)] | 0.760 (0.720, 0.796) † [was 0.848 (0.815, 0.878)] |
| PPV | 0.516 (0.449, 0.579) † [was 0.558 (0.488, 0.624)] | 0.500 (0.436, 0.562) † [was 0.595 (0.523, 0.667)] |
| NPV | 0.933 (0.909, 0.955) † [was 0.940 (0.917, 0.960)] | 0.945 (0.921, 0.965) † [was 0.931 (0.908, 0.953)] |
| AUC | 0.855 (0.817, 0.888) † [was 0.853 (0.817, 0.887)] | 0.894 (0.862, 0.922) † [was 0.892 (0.863, 0.919)] |
| PRAUC | 0.655 (0.571, 0.741) † [was 0.698 (0.624, 0.766)] | 0.745 (0.674, 0.815) † [was 0.725 (0.649, 0.796)] |
| Brier | 0.137 (0.123, 0.152) † [was 0.127 (0.115, 0.139)] | 0.103 (0.089, 0.119) † [was 0.103 (0.089, 0.118)] |
| Mean fold threshold (new) | 0.331 (0.087–0.589) | 0.294 (0.102–0.532) |

**Supplementary Figure 2 — sensitivity across specificity targets (nested CV)** (`rev_sensitivity_vs_spec_targets_cat_xgb.csv`; figure `rev_sensitivity_vs_spec_targets_cat_xgb.png`). The 0.70 rows are new; 0.75–0.90 rows re-run the same nested CV and should be identical to v1:

| Specificity_Target | Model | Sensitivity | Sensitivity_CI | Achieved_Specificity | PPV | NPV | AUROC | PRAUC | Brier |
|---|---|---|---|---|---|---|---|---|---|
| 0.7 (new) | CatBoost | 0.801 | (0.734, 0.864) | 0.786 | 0.516 | 0.933 | 0.855 | 0.655 | 0.137 |
| 0.7 (new) | XGBoost | 0.844 | (0.782, 0.900) | 0.76 | 0.5 | 0.945 | 0.894 | 0.745 | 0.103 |
| 0.75 | CatBoost | 0.837 | (0.776, 0.893) | 0.79 | 0.532 | 0.944 | 0.872 | 0.71 | 0.128 |
| 0.75 | XGBoost | 0.816 | (0.750, 0.874) | 0.804 | 0.542 | 0.939 | 0.9 | 0.755 | 0.104 |
| 0.8 | CatBoost | 0.816 | (0.752, 0.875) | 0.816 | 0.558 | 0.94 | 0.853 | 0.698 | 0.127 |
| 0.8 | XGBoost | 0.78 | (0.711, 0.844) | 0.848 | 0.595 | 0.931 | 0.892 | 0.725 | 0.103 |
| 0.85 | CatBoost | 0.745 | (0.674, 0.812) | 0.869 | 0.618 | 0.923 | 0.865 | 0.721 | 0.113 |
| 0.85 | XGBoost | 0.759 | (0.688, 0.823) | 0.867 | 0.618 | 0.927 | 0.886 | 0.717 | 0.102 |
| 0.9 | CatBoost | 0.66 | (0.583, 0.734) | 0.901 | 0.655 | 0.903 | 0.885 | 0.735 | 0.108 |
| 0.9 | XGBoost | 0.716 | (0.643, 0.790) | 0.885 | 0.639 | 0.916 | 0.892 | 0.739 | 0.1 |

*Verification:* rows at 0.75–0.90 identical to v1 ✓.

## 4. Staged models M1–M5 — held-out test set (Supplementary Tables S2 and S3)

Same fitted models; thresholds recomputed at specificity ≥ 0.70. ROC-AUC, PR-AUC, Brier (and Null / recalibrated Brier, trees, prediction range) must be unchanged — verified below each table.

**Supplementary Table S2 — XGBoost** (`rev_metrics_by_stage_XGB_test_bootCI.csv`):

| Model | n_feats | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr | Train OOF Sens | Train OOF Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Model 1 | 10 | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] | 0.667 † [was 0.482] | 0.701 † [was 0.8] |
| Model 2 | 15 | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] | 0.652 † [was 0.574] | 0.703 † [was 0.802] |
| Model 3 | 15 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.475 (0.343, 0.609) † [was 0.463 (0.326, 0.604)] | 0.930 (0.861, 0.971) † [was 0.905 (0.832, 0.953)] | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.179 † [was 0.2239] | 0.787 † [was 0.738] | 0.705 † [was 0.8] |
| Model 4 | 15 | 0.829 (0.664, 0.934) † [was 0.714 (0.537, 0.854)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.408 (0.293, 0.532) † [was 0.463 (0.326, 0.604)] | 0.932 (0.857, 0.975) † [was 0.905 (0.832, 0.953)] | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.1846 † [was 0.3091] | 0.872 † [was 0.801] | 0.705 † [was 0.804] |
| Model 5 | 15 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] | 0.908 † [was 0.837] | 0.713 † [was 0.828] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), n_feats, Trees, Test pred range — identical to v1 in every row ✓. 35 cell(s) changed (†).

**Supplementary Table S3 — CatBoost** (`rev_metrics_by_stage_CAT_test_bootCI.csv`):

| Model | n_feats | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr | Train OOF Sens | Train OOF Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Model 1 | 10 | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] | 0.482 † [was 0.383] | 0.709 † [was 0.818] |
| Model 2 | 15 | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] | 0.624 † [was 0.454] | 0.703 † [was 0.8] |
| Model 3 | 15 | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.441 (0.321, 0.567) † [was 0.490 (0.348, 0.634)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4355 † [was 0.4466] | 0.723 † [was 0.61] | 0.713 † [was 0.806] |
| Model 4 | 15 | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.422 (0.299, 0.552) † [was 0.553 (0.401, 0.698)] | 0.916 (0.841, 0.963) † [was 0.920 (0.853, 0.963)] | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.2483 † [was 0.3222] | 0.83 † [was 0.759] | 0.707 † [was 0.81] |
| Model 5 | 15 | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] | 0.915 † [was 0.865] | 0.703 † [was 0.802] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), n_feats, Trees, Test pred range — identical to v1 in every row ✓. 35 cell(s) changed (†).

M1–M5 feature sets: unchanged (Section B); see v1 Section 4.

## 5. Apparent vs cross-validated vs held-out test performance (R1.4)

Threshold-dependent columns change; AUROC/AP/Brier unchanged except the nested-CV row (re-tuned, Section 3). File: `rev_train_cv_test_metrics.csv`.

| Algorithm | Model | n_feats | Setting | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 40 | Apparent (train, n=636) | 0.972 † [was 0.929] | 0.747 † [was 0.824] | 0.523 † [was 0.601] | 0.989 † [was 0.976] | 0.967 | 0.912 | 0.061 |  |  | 0.1098 † [was 0.1710] |
| XGBoost | Full-feature | 40 | CV: 5-fold OOF, fixed params (train) | 0.901 † [was 0.844] | 0.715 † [was 0.800] | 0.474 † [was 0.546] | 0.962 † [was 0.947] | 0.905 | 0.783 | 0.092 |  |  | 0.1098 † [was 0.1710] |
| XGBoost | Full-feature | 40 | Held-out test (n=159) | 0.914 (0.769, 0.982) † [was 0.829 (0.664, 0.934)] | 0.734 (0.647, 0.809) † [was 0.847 (0.771, 0.905)] | 0.492 (0.366, 0.619) † [was 0.604 (0.453, 0.742)] | 0.968 (0.910, 0.993) † [was 0.946 (0.886, 0.980)] | 0.917 (0.856, 0.964) | 0.817 (0.681, 0.911) | 0.086 (0.057, 0.120) | 0.172 | 0.086 (0.056, 0.122) | 0.1098 † [was 0.1710] |
| XGBoost | Model 1 | 10 | Apparent (train, n=636) | 0.837 † [was 0.674] | 0.729 † [was 0.832] | 0.468 † [was 0.534] | 0.940 † [was 0.900] | 0.857 | 0.609 | 0.190 |  |  | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | 10 | CV: 5-fold OOF, fixed params (train) | 0.667 † [was 0.482] | 0.701 † [was 0.800] | 0.388 † [was 0.407] | 0.881 † [was 0.844] | 0.733 | 0.398 | 0.212 |  |  | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | 10 | Held-out test (n=159) | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 2 | 15 | Apparent (train, n=636) | 0.872 † [was 0.709] | 0.756 † [was 0.842] | 0.504 † [was 0.562] | 0.954 † [was 0.910] | 0.882 | 0.660 | 0.181 |  |  | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | 15 | CV: 5-fold OOF, fixed params (train) | 0.652 † [was 0.574] | 0.703 † [was 0.802] | 0.385 † [was 0.453] | 0.877 † [was 0.869] | 0.757 | 0.455 | 0.218 |  |  | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | 15 | Held-out test (n=159) | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 3 | 15 | Apparent (train, n=636) | 0.851 † [was 0.809] | 0.758 † [was 0.816] | 0.500 † [was 0.556] | 0.947 † [was 0.937] | 0.894 | 0.756 | 0.103 |  |  | 0.1790 † [was 0.2239] |
| XGBoost | Model 3 | 15 | CV: 5-fold OOF, fixed params (train) | 0.787 † [was 0.738] | 0.705 † [was 0.800] | 0.432 † [was 0.512] | 0.921 † [was 0.915] | 0.830 | 0.639 | 0.121 |  |  | 0.1790 † [was 0.2239] |
| XGBoost | Model 3 | 15 | Held-out test (n=159) | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.475 (0.343, 0.609) † [was 0.463 (0.326, 0.604)] | 0.930 (0.861, 0.971) † [was 0.905 (0.832, 0.953)] | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.1790 † [was 0.2239] |
| XGBoost | Model 4 | 15 | Apparent (train, n=636) | 1.000 † [was 0.979] | 0.756 † [was 0.857] | 0.538 † [was 0.660] | 1.000 † [was 0.993] | 0.980 | 0.924 | 0.057 |  |  | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | 15 | CV: 5-fold OOF, fixed params (train) | 0.872 † [was 0.801] | 0.705 † [was 0.804] | 0.457 † [was 0.538] | 0.951 † [was 0.934] | 0.876 | 0.672 | 0.115 |  |  | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | 15 | Held-out test (n=159) | 0.829 (0.664, 0.934) † [was 0.714 (0.537, 0.854)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.408 (0.293, 0.532) † [was 0.463 (0.326, 0.604)] | 0.932 (0.857, 0.975) † [was 0.905 (0.832, 0.953)] | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.1846 † [was 0.3091] |
| XGBoost | Model 5 | 15 | Apparent (train, n=636) | 1.000 | 0.762 † [was 0.883] | 0.544 † [was 0.709] | 1.000 | 0.991 | 0.954 | 0.038 |  |  | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | 15 | CV: 5-fold OOF, fixed params (train) | 0.908 † [was 0.837] | 0.713 † [was 0.828] | 0.474 † [was 0.581] | 0.964 † [was 0.947] | 0.914 | 0.770 | 0.097 |  |  | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | 15 | Held-out test (n=159) | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] |
| CatBoost | Full-feature | 40 | Apparent (train, n=636) | 0.979 † [was 0.950] | 0.770 † [was 0.842] | 0.548 † [was 0.632] | 0.992 † [was 0.983] | 0.980 | 0.951 | 0.053 |  |  | 0.1409 † [was 0.2031] |
| CatBoost | Full-feature | 40 | CV: 5-fold OOF, fixed params (train) | 0.908 † [was 0.851] | 0.717 † [was 0.804] | 0.478 † [was 0.553] | 0.965 † [was 0.950] | 0.909 | 0.779 | 0.094 |  |  | 0.1409 † [was 0.2031] |
| CatBoost | Full-feature | 40 | Held-out test (n=159) | 0.914 (0.769, 0.982) † [was 0.857 (0.697, 0.952)] | 0.750 (0.664, 0.823) † [was 0.823 (0.744, 0.885)] | 0.508 (0.379, 0.636) † [was 0.577 (0.432, 0.713)] | 0.969 (0.911, 0.994) † [was 0.953 (0.894, 0.985)] | 0.908 (0.836, 0.964) | 0.814 (0.681, 0.911) | 0.085 (0.057, 0.117) | 0.172 | 0.083 (0.053, 0.117) | 0.1409 † [was 0.2031] |
| CatBoost | Model 1 | 10 | Apparent (train, n=636) | 0.461 † [was 0.411] | 0.768 † [was 0.832] | 0.361 † [was 0.411] | 0.833 † [was 0.832] | 0.710 | 0.469 | 0.191 |  |  | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | 10 | CV: 5-fold OOF, fixed params (train) | 0.482 † [was 0.383] | 0.709 † [was 0.818] | 0.321 † [was 0.375] | 0.828 † [was 0.823] | 0.657 | 0.336 | 0.197 |  |  | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | 10 | Held-out test (n=159) | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 2 | 15 | Apparent (train, n=636) | 0.738 † [was 0.660] | 0.685 † [was 0.766] | 0.400 † [was 0.445] | 0.902 † [was 0.888] | 0.790 | 0.479 | 0.223 |  |  | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | 15 | CV: 5-fold OOF, fixed params (train) | 0.624 † [was 0.454] | 0.703 † [was 0.800] | 0.374 † [was 0.393] | 0.868 † [was 0.837] | 0.695 | 0.352 | 0.231 |  |  | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | 15 | Held-out test (n=159) | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 3 | 15 | Apparent (train, n=636) | 0.816 † [was 0.773] | 0.681 † [was 0.786] | 0.421 † [was 0.507] | 0.928 † [was 0.924] | 0.828 | 0.659 | 0.199 |  |  | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | 15 | CV: 5-fold OOF, fixed params (train) | 0.723 † [was 0.610] | 0.713 † [was 0.806] | 0.418 † [was 0.473] | 0.901 † [was 0.879] | 0.781 | 0.552 | 0.203 |  |  | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | 15 | Held-out test (n=159) | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.441 (0.321, 0.567) † [was 0.490 (0.348, 0.634)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4355 † [was 0.4466] |
| CatBoost | Model 4 | 15 | Apparent (train, n=636) | 0.887 † [was 0.801] | 0.747 † [was 0.851] | 0.500 † [was 0.604] | 0.959 † [was 0.938] | 0.912 | 0.781 | 0.107 |  |  | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | 15 | CV: 5-fold OOF, fixed params (train) | 0.830 † [was 0.759] | 0.707 † [was 0.810] | 0.447 † [was 0.532] | 0.936 † [was 0.922] | 0.849 | 0.600 | 0.127 |  |  | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | 15 | Held-out test (n=159) | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.422 (0.299, 0.552) † [was 0.553 (0.401, 0.698)] | 0.916 (0.841, 0.963) † [was 0.920 (0.853, 0.963)] | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.2483 † [was 0.3222] |
| CatBoost | Model 5 | 15 | Apparent (train, n=636) | 0.936 † [was 0.894] | 0.756 † [was 0.859] | 0.522 † [was 0.643] | 0.977 † [was 0.966] | 0.957 | 0.887 | 0.075 |  |  | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | 15 | CV: 5-fold OOF, fixed params (train) | 0.915 † [was 0.865] | 0.703 † [was 0.802] | 0.467 † [was 0.555] | 0.967 † [was 0.954] | 0.898 | 0.689 | 0.104 |  |  | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | 15 | Held-out test (n=159) | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.) — identical to v1 in every row ✓. 178 cell(s) changed (†).

Nested-CV rows (all values re-tuned at ≥ 0.70):

| Algorithm | Model | n_feats | Setting | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 40 | CV: nested-CV OOF (train; per-fold thr) | 0.844 † [was 0.780] | 0.760 † [was 0.848] | 0.500 † [was 0.595] | 0.945 † [was 0.931] | 0.894 † [was 0.892] | 0.745 † [was 0.725] | 0.103 |  |  | per fold |
| CatBoost | Full-feature | 40 | CV: nested-CV OOF (train; per-fold thr) | 0.801 † [was 0.816] | 0.786 † [was 0.816] | 0.516 † [was 0.558] | 0.933 † [was 0.940] | 0.855 † [was 0.853] | 0.655 † [was 0.698] | 0.137 † [was 0.127] |  |  | per fold |

13 cell(s) changed (†).

**Paired bootstrap ROC-AUC tests** — threshold-free, identical to v1 ✓ (`rev_paired_bootstrap_auc_tests.csv`):

| Comparison (A vs B) | AUC A | AUC B | ΔAUC (A−B) | p (paired bootstrap) | B used |
|---|---|---|---|---|---|
| XGBoost: Full-feature vs Model 5 | 0.917 | 0.885 | +0.031 (+0.003, +0.062) | 0.027 | 2000 |
| CatBoost: Full-feature vs Model 5 | 0.908 | 0.926 | -0.018 (-0.039, -0.002) | 0.028 | 2000 |
| Model 5: XGBoost vs CatBoost | 0.885 | 0.926 | -0.040 (-0.080, -0.004) | 0.025 | 2000 |

## 6. Operating points across specificity targets — test set (new; full-feature and M5)

Fixed fitted models; for each target the threshold maximises sensitivity subject to specificity ≥ target on pooled training out-of-fold predictions and is applied unchanged to the test set. Exact 95 % CIs. File: `rev_test_sens_spec_by_spec_target_fullfeature_M5.csv` (all rows new in v2).

| Algorithm | Model | Specificity target | Threshold | Train OOF Sens | Train OOF Spec | Test Sens (95% CI) | Test Spec (95% CI) | TP/FN/TN/FP |
|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 0.7 | 0.1098 | 0.901 | 0.715 | 0.914 (0.769, 0.982) | 0.734 (0.647, 0.809) | 32/3/91/33 |
| XGBoost | Full-feature | 0.75 | 0.1354 | 0.887 | 0.762 | 0.914 (0.769, 0.982) | 0.798 (0.717, 0.865) | 32/3/99/25 |
| XGBoost | Full-feature | 0.8 | 0.171 | 0.844 | 0.8 | 0.829 (0.664, 0.934) | 0.847 (0.771, 0.905) | 29/6/105/19 |
| XGBoost | Full-feature | 0.85 | 0.2229 | 0.809 | 0.853 | 0.743 (0.567, 0.875) | 0.871 (0.799, 0.924) | 26/9/108/16 |
| XGBoost | Full-feature | 0.9 | 0.363 | 0.73 | 0.907 | 0.686 (0.507, 0.831) | 0.919 (0.857, 0.961) | 24/11/114/10 |
| XGBoost | Model 5 | 0.7 | 0.1303 | 0.908 | 0.713 | 0.800 (0.631, 0.916) | 0.710 (0.621, 0.788) | 28/7/88/36 |
| XGBoost | Model 5 | 0.75 | 0.1966 | 0.887 | 0.78 | 0.714 (0.537, 0.854) | 0.774 (0.690, 0.844) | 25/10/96/28 |
| XGBoost | Model 5 | 0.8 | 0.2816 | 0.837 | 0.828 | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 25/10/102/22 |
| XGBoost | Model 5 | 0.85 | 0.3531 | 0.794 | 0.853 | 0.714 (0.537, 0.854) | 0.855 (0.780, 0.912) | 25/10/106/18 |
| XGBoost | Model 5 | 0.9 | 0.5023 | 0.745 | 0.901 | 0.657 (0.478, 0.809) | 0.911 (0.847, 0.955) | 23/12/113/11 |
| CatBoost | Full-feature | 0.7 | 0.1409 | 0.908 | 0.717 | 0.914 (0.769, 0.982) | 0.750 (0.664, 0.823) | 32/3/93/31 |
| CatBoost | Full-feature | 0.75 | 0.1817 | 0.879 | 0.774 | 0.857 (0.697, 0.952) | 0.798 (0.717, 0.865) | 30/5/99/25 |
| CatBoost | Full-feature | 0.8 | 0.2031 | 0.851 | 0.804 | 0.857 (0.697, 0.952) | 0.823 (0.744, 0.885) | 30/5/102/22 |
| CatBoost | Full-feature | 0.85 | 0.2686 | 0.809 | 0.853 | 0.771 (0.599, 0.896) | 0.871 (0.799, 0.924) | 27/8/108/16 |
| CatBoost | Full-feature | 0.9 | 0.3536 | 0.745 | 0.903 | 0.714 (0.537, 0.854) | 0.944 (0.887, 0.977) | 25/10/117/7 |
| CatBoost | Model 5 | 0.7 | 0.1671 | 0.915 | 0.703 | 0.943 (0.808, 0.993) | 0.710 (0.621, 0.788) | 33/2/88/36 |
| CatBoost | Model 5 | 0.75 | 0.1943 | 0.887 | 0.752 | 0.943 (0.808, 0.993) | 0.774 (0.690, 0.844) | 33/2/96/28 |
| CatBoost | Model 5 | 0.8 | 0.2493 | 0.865 | 0.802 | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 30/5/106/18 |
| CatBoost | Model 5 | 0.85 | 0.3212 | 0.794 | 0.857 | 0.743 (0.567, 0.875) | 0.911 (0.847, 0.955) | 26/9/113/11 |
| CatBoost | Model 5 | 0.9 | 0.4182 | 0.695 | 0.903 | 0.657 (0.478, 0.809) | 0.944 (0.887, 0.977) | 23/12/117/7 |

*Verification:* the 0.80 rows reproduce the v1 (specificity ≥ 0.80) M5 test sensitivity/specificity ✓; the 0.70 rows equal the primary results in Sections 4 and C.

## 7. TB-history leakage check and original-variable comparison (supplementary)

Leakage cross-tab identical to v1 ✓ (`rev_tbhistory_leakage_check.csv`; see v1 Section 6). M1–M5 with the original composite (`rev_tbhistory_original_composite_sensitivity.csv`):

| Algorithm | Model | Analysis | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main: interview-only prior TB | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | Supplementary: original composite | 0.686 (0.507, 0.831) | 0.839 (0.762, 0.899) † [was 0.871 (0.799, 0.924)] | 0.545 (0.388, 0.696) † [was 0.600 (0.433, 0.751)] | 0.904 (0.835, 0.951) † [was 0.908 (0.841, 0.953)] | 0.806 (0.720, 0.882) | 0.530 (0.370, 0.695) | 0.142 (0.111, 0.173) | 0.172 | 0.135 (0.100, 0.173) | 0.2028 † [was 0.2674] |
| XGBoost | Model 2 | Main: interview-only prior TB | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | Supplementary: original composite | 0.743 (0.567, 0.875) † [was 0.657 (0.478, 0.809)] | 0.782 (0.699, 0.851) † [was 0.879 (0.808, 0.931)] | 0.491 (0.351, 0.632) † [was 0.605 (0.434, 0.760)] | 0.915 (0.845, 0.960) † [was 0.901 (0.833, 0.948)] | 0.817 (0.730, 0.893) | 0.589 (0.435, 0.748) | 0.133 (0.102, 0.166) | 0.172 | 0.132 (0.097, 0.170) | 0.1825 † [was 0.2633] |
| XGBoost | Model 3 | Main: interview-only prior TB | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.475 (0.343, 0.609) † [was 0.463 (0.326, 0.604)] | 0.930 (0.861, 0.971) † [was 0.905 (0.832, 0.953)] | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.179 † [was 0.2239] |
| XGBoost | Model 3 | Supplementary: original composite | 0.886 (0.733, 0.968) † [was 0.771 (0.599, 0.896)] | 0.766 (0.682, 0.837) † [was 0.847 (0.771, 0.905)] | 0.517 (0.384, 0.648) † [was 0.587 (0.432, 0.730)] | 0.960 (0.900, 0.989) † [was 0.929 (0.865, 0.969)] | 0.891 (0.830, 0.943) | 0.683 (0.530, 0.840) | 0.114 (0.088, 0.142) | 0.172 | 0.105 (0.075, 0.140) | 0.3247 † [was 0.416] |
| XGBoost | Model 4 | Main: interview-only prior TB | 0.829 (0.664, 0.934) † [was 0.714 (0.537, 0.854)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.408 (0.293, 0.532) † [was 0.463 (0.326, 0.604)] | 0.932 (0.857, 0.975) † [was 0.905 (0.832, 0.953)] | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | Supplementary: original composite | 0.829 (0.664, 0.934) † [was 0.743 (0.567, 0.875)] | 0.734 (0.647, 0.809) † [was 0.839 (0.762, 0.899)] | 0.468 (0.340, 0.599) † [was 0.565 (0.411, 0.711)] | 0.938 (0.870, 0.977) † [was 0.920 (0.854, 0.963)] | 0.896 (0.835, 0.945) | 0.763 (0.618, 0.875) | 0.104 (0.073, 0.140) | 0.172 | 0.098 (0.066, 0.134) | 0.1728 † [was 0.3302] |
| XGBoost | Model 5 | Main: interview-only prior TB | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | Supplementary: original composite | 0.886 (0.733, 0.968) † [was 0.829 (0.664, 0.934)] | 0.774 (0.690, 0.844) † [was 0.871 (0.799, 0.924)] | 0.525 (0.391, 0.657) † [was 0.644 (0.488, 0.781)] | 0.960 (0.901, 0.989) † [was 0.947 (0.889, 0.980)] | 0.919 (0.857, 0.965) | 0.821 (0.691, 0.915) | 0.087 (0.058, 0.120) | 0.172 | 0.084 (0.054, 0.119) | 0.1552 † [was 0.2898] |
| CatBoost | Model 1 | Main: interview-only prior TB | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | Supplementary: original composite | 0.714 (0.537, 0.854) † [was 0.600 (0.421, 0.761)] | 0.823 (0.744, 0.885) † [was 0.855 (0.780, 0.912)] | 0.532 (0.381, 0.679) † [was 0.538 (0.372, 0.699)] | 0.911 (0.842, 0.956) † [was 0.883 (0.812, 0.935)] | 0.821 (0.724, 0.900) | 0.653 (0.495, 0.797) | 0.132 (0.107, 0.158) | 0.172 | 0.123 (0.090, 0.159) | 0.2665 † [was 0.4089] |
| CatBoost | Model 2 | Main: interview-only prior TB | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | Supplementary: original composite | 0.714 (0.537, 0.854) † [was 0.629 (0.449, 0.785)] | 0.823 (0.744, 0.885) † [was 0.871 (0.799, 0.924)] | 0.532 (0.381, 0.679) † [was 0.579 (0.408, 0.737)] | 0.911 (0.842, 0.956) † [was 0.893 (0.823, 0.942)] | 0.793 (0.693, 0.878) | 0.641 (0.485, 0.782) | 0.171 (0.157, 0.185) | 0.172 | 0.124 (0.091, 0.162) | 0.3733 † [was 0.4554] |
| CatBoost | Model 3 | Main: interview-only prior TB | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.441 (0.321, 0.567) † [was 0.490 (0.348, 0.634)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | Supplementary: original composite | 0.857 (0.697, 0.952) † [was 0.743 (0.567, 0.875)] | 0.774 (0.690, 0.844) † [was 0.863 (0.790, 0.918)] | 0.517 (0.382, 0.650) † [was 0.605 (0.444, 0.750)] | 0.950 (0.888, 0.984) † [was 0.922 (0.858, 0.964)] | 0.887 (0.821, 0.941) | 0.666 (0.510, 0.833) | 0.130 (0.109, 0.154) | 0.172 | 0.104 (0.075, 0.137) | 0.3945 † [was 0.5099] |
| CatBoost | Model 4 | Main: interview-only prior TB | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.422 (0.299, 0.552) † [was 0.553 (0.401, 0.698)] | 0.916 (0.841, 0.963) † [was 0.920 (0.853, 0.963)] | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | Supplementary: original composite | 0.886 (0.733, 0.968) † [was 0.686 (0.507, 0.831)] | 0.726 (0.638, 0.802) † [was 0.855 (0.780, 0.912)] | 0.477 (0.351, 0.605) † [was 0.571 (0.410, 0.723)] | 0.957 (0.895, 0.988) † [was 0.906 (0.838, 0.952)] | 0.895 (0.835, 0.944) | 0.728 (0.574, 0.863) | 0.103 (0.071, 0.138) | 0.172 | 0.104 (0.070, 0.140) | 0.1234 † [was 0.1886] |
| CatBoost | Model 5 | Main: interview-only prior TB | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | Supplementary: original composite | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.726 (0.638, 0.802) † [was 0.863 (0.790, 0.918)] | 0.493 (0.368, 0.618) † [was 0.638 (0.485, 0.773)] | 0.978 (0.924, 0.997) † [was 0.955 (0.899, 0.985)] | 0.943 (0.895, 0.977) | 0.838 (0.702, 0.934) | 0.096 (0.075, 0.121) | 0.172 | 0.080 (0.052, 0.112) | 0.3028 † [was 0.4548] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Features — identical to v1 in every row ✓. 99 cell(s) changed (†).

## 8. Full-feature and top-10 / top-15 models (Supplementary Table S5; R1.2)

File: `rev_topk_models.csv`. Selected features unchanged.

| Algorithm | Model | n_feats | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 40 | 0.914 (0.769, 0.982) † [was 0.829 (0.664, 0.934)] | 0.734 (0.647, 0.809) † [was 0.847 (0.771, 0.905)] | 0.492 (0.366, 0.619) † [was 0.604 (0.453, 0.742)] | 0.968 (0.910, 0.993) † [was 0.946 (0.886, 0.980)] | 0.917 (0.856, 0.964) | 0.817 (0.681, 0.911) | 0.086 (0.057, 0.120) | 0.172 | 0.086 (0.056, 0.122) | 0.1098 † [was 0.171] |
| XGBoost | Top-15 | 15 | 0.886 (0.733, 0.968) † [was 0.800 (0.631, 0.916)] | 0.718 (0.630, 0.795) † [was 0.815 (0.735, 0.879)] | 0.470 (0.346, 0.597) † [was 0.549 (0.403, 0.689)] | 0.957 (0.894, 0.988) † [was 0.935 (0.871, 0.974)] | 0.901 (0.839, 0.951) | 0.776 (0.626, 0.889) | 0.113 (0.081, 0.149) | 0.172 | 0.095 (0.065, 0.132) | 0.2232 † [was 0.3565] |
| XGBoost | Top-10 | 10 | 0.886 (0.733, 0.968) † [was 0.771 (0.599, 0.896)] | 0.750 (0.664, 0.823) † [was 0.806 (0.726, 0.872)] | 0.500 (0.370, 0.630) † [was 0.529 (0.385, 0.671)] | 0.959 (0.898, 0.989) † [was 0.926 (0.859, 0.967)] | 0.903 (0.839, 0.954) | 0.798 (0.665, 0.897) | 0.102 (0.071, 0.137) | 0.172 | 0.093 (0.061, 0.129) | 0.1614 † [was 0.2832] |
| XGBoost | Model 5 | 15 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] |
| CatBoost | Full-feature | 40 | 0.914 (0.769, 0.982) † [was 0.857 (0.697, 0.952)] | 0.750 (0.664, 0.823) † [was 0.823 (0.744, 0.885)] | 0.508 (0.379, 0.636) † [was 0.577 (0.432, 0.713)] | 0.969 (0.911, 0.994) † [was 0.953 (0.894, 0.985)] | 0.908 (0.836, 0.964) | 0.814 (0.681, 0.911) | 0.085 (0.057, 0.117) | 0.172 | 0.083 (0.053, 0.117) | 0.1409 † [was 0.2031] |
| CatBoost | Top-15 | 15 | 0.914 (0.769, 0.982) † [was 0.857 (0.697, 0.952)] | 0.742 (0.656, 0.816) † [was 0.831 (0.753, 0.892)] | 0.500 (0.372, 0.628) † [was 0.588 (0.442, 0.724)] | 0.968 (0.910, 0.993) † [was 0.954 (0.895, 0.985)] | 0.912 (0.840, 0.966) | 0.822 (0.705, 0.915) | 0.100 (0.079, 0.124) | 0.172 | 0.085 (0.055, 0.120) | 0.244 † [was 0.3033] |
| CatBoost | Top-10 | 10 | 0.914 (0.769, 0.982) | 0.758 (0.673, 0.830) † [was 0.806 (0.726, 0.872)] | 0.516 (0.386, 0.645) † [was 0.571 (0.432, 0.703)] | 0.969 (0.912, 0.994) † [was 0.971 (0.917, 0.994)] | 0.920 (0.860, 0.967) | 0.818 (0.686, 0.917) | 0.085 (0.058, 0.115) | 0.172 | 0.082 (0.052, 0.117) | 0.147 † [was 0.2013] |
| CatBoost | Model 5 | 15 | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), n_feats, Selected features — identical to v1 in every row ✓. 39 cell(s) changed (†).

**QFT and TB history (R1.2):** threshold-free; identical to v1 ✓ — see v1 Section 8.

## 9. Resampling sensitivity (Supplementary Table S6; R1.3)

File: `rev_resampling_sensitivity.csv`. Fitted resampled models, AUROC/AP/Brier and calibration unchanged; sens/spec at the ≥ 0.70 threshold recomputed.

| Algorithm | Model | Resampling | AUROC | AP | Brier | Null Brier | Brier (recal.) | Trees | Calib intercept | Calib slope | Sens | Spec | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | Unresampled | 0.917 (0.856, 0.964) | 0.817 (0.681, 0.911) | 0.086 (0.057, 0.120) | 0.172 | 0.086 (0.056, 0.122) | 62 | 0.102 (-0.409, 0.583) | 1.155 (0.885, 1.571) | 0.914 (0.769, 0.982) † [was 0.829 (0.664, 0.934)] | 0.734 (0.647, 0.809) † [was 0.847 (0.771, 0.905)] | 0.1098 † [was 0.171] |
| XGBoost | Full-feature | Oversampled 1:1 | 0.914 (0.849, 0.964) | 0.809 (0.664, 0.908) | 0.112 (0.083, 0.146) | 0.172 | 0.086 (0.057, 0.121) | 55 | -1.156 (-1.657, -0.688) | 1.148 (0.858, 1.648) | 0.886 (0.733, 0.968) † [was 0.829 (0.664, 0.934)] | 0.734 (0.647, 0.809) † [was 0.855 (0.780, 0.912)] | 0.3196 † [was 0.4783] |
| XGBoost | Full-feature | Undersampled 1:1 | 0.919 (0.854, 0.970) | 0.800 (0.654, 0.914) | 0.122 (0.089, 0.158) | 0.172 | 0.085 (0.056, 0.118) | 105 | -1.516 (-2.075, -1.019) | 1.025 (0.743, 1.548) | 0.943 (0.808, 0.993) | 0.734 (0.647, 0.809) † [was 0.766 (0.682, 0.837)] | 0.3627 † [was 0.4673] |
| XGBoost | Model 5 | Unresampled | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 427 | -0.490 (-1.242, 0.276) | 0.676 (0.504, 0.934) | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | Oversampled 1:1 | 0.889 (0.821, 0.944) | 0.763 (0.608, 0.883) | 0.123 (0.089, 0.163) | 0.172 | 0.096 (0.063, 0.134) | 229 | -1.109 (-1.780, -0.435) | 0.744 (0.549, 1.057) | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.806 (0.726, 0.872)] | 0.2077 † [was 0.4139] |
| XGBoost | Model 5 | Undersampled 1:1 | 0.909 (0.844, 0.960) | 0.796 (0.653, 0.899) | 0.124 (0.090, 0.163) | 0.172 | 0.091 (0.062, 0.125) | 184 | -1.488 (-2.141, -0.879) | 0.847 (0.630, 1.205) | 0.943 (0.808, 0.993) † [was 0.886 (0.733, 0.968)] | 0.726 (0.638, 0.802) † [was 0.790 (0.708, 0.858)] | 0.2928 † [was 0.4364] |
| CatBoost | Full-feature | Unresampled | 0.908 (0.836, 0.964) | 0.814 (0.681, 0.911) | 0.085 (0.057, 0.117) | 0.172 | 0.083 (0.053, 0.117) | 34 | 0.025 (-0.443, 0.445) | 1.326 (0.976, 1.895) | 0.914 (0.769, 0.982) † [was 0.857 (0.697, 0.952)] | 0.750 (0.664, 0.823) † [was 0.823 (0.744, 0.885)] | 0.1409 † [was 0.2031] |
| CatBoost | Full-feature | Oversampled 1:1 | 0.916 (0.861, 0.960) | 0.788 (0.640, 0.900) | 0.093 (0.061, 0.130) | 0.172 | 0.090 (0.060, 0.124) | 106 | -0.181 (-0.854, 0.448) | 0.833 (0.630, 1.166) | 0.914 (0.769, 0.982) † [was 0.743 (0.567, 0.875)] | 0.710 (0.621, 0.788) † [was 0.871 (0.799, 0.924)] | 0.108 † [was 0.2579] |
| CatBoost | Full-feature | Undersampled 1:1 | 0.905 (0.827, 0.964) | 0.791 (0.638, 0.906) | 0.116 (0.084, 0.154) | 0.172 | 0.086 (0.058, 0.119) | 70 | -1.442 (-2.061, -0.874) | 0.881 (0.631, 1.315) | 0.943 (0.808, 0.993) † [was 0.886 (0.733, 0.968)] | 0.702 (0.613, 0.780) † [was 0.815 (0.735, 0.879)] | 0.2862 † [was 0.4514] |
| CatBoost | Model 5 | Unresampled | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 89 | -0.110 (-0.529, 0.261) | 1.703 (1.291, 2.427) | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | Oversampled 1:1 | 0.926 (0.865, 0.972) | 0.844 (0.726, 0.929) | 0.107 (0.083, 0.133) | 0.172 | 0.083 (0.055, 0.113) | 60 | -0.976 (-1.397, -0.601) | 1.689 (1.244, 2.528) | 0.914 (0.769, 0.982) | 0.742 (0.656, 0.816) † [was 0.823 (0.744, 0.885)] | 0.3223 † [was 0.4368] |
| CatBoost | Model 5 | Undersampled 1:1 | 0.921 (0.851, 0.976) | 0.820 (0.688, 0.925) | 0.118 (0.093, 0.146) | 0.172 | 0.082 (0.054, 0.114) | 84 | -1.160 (-1.587, -0.790) | 1.641 (1.175, 2.507) | 0.943 (0.808, 0.993) † [was 0.914 (0.769, 0.982)] | 0.782 (0.699, 0.851) † [was 0.839 (0.762, 0.899)] | 0.4125 † [was 0.4982] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Trees, Calib intercept, Calib slope — identical to v1 in every row ✓. 34 cell(s) changed (†).

## 10. Age, QFT and subgroups (R2.5–R2.7)

**Age ≥ 15 re-run** (`rev_age15plus_sensitivity.csv`):

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main analysis (all ages) | 159 | 35 | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | Main models, age>=15 test rows | 151 | 34 | 0.559 (0.379, 0.728) † [was 0.441 (0.272, 0.621)] | 0.675 (0.582, 0.759) † [was 0.778 (0.692, 0.849)] | 0.333 (0.214, 0.471) † [was 0.366 (0.221, 0.531)] | 0.840 (0.750, 0.908) † [was 0.827 (0.743, 0.893)] | 0.624 (0.514, 0.727) | 0.330 (0.219, 0.495) | 0.242 (0.216, 0.268) | 0.174 | 0.174 (0.138, 0.212) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | Re-run on age>=15 | 151 | 34 | 0.529 (0.351, 0.702) † [was 0.294 (0.151, 0.475)] | 0.650 (0.556, 0.735) † [was 0.786 (0.701, 0.857)] | 0.305 (0.192, 0.439) † [was 0.286 (0.146, 0.463)] | 0.826 (0.733, 0.897) † [was 0.793 (0.708, 0.863)] | 0.603 (0.489, 0.712) | 0.312 (0.199, 0.461) | 0.172 (0.136, 0.210) | 0.174 | 0.177 (0.139, 0.218) | 0.2321 † [was 0.2483] |
| XGBoost | Model 2 | Main analysis (all ages) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | Main models, age>=15 test rows | 151 | 34 | 0.588 (0.407, 0.754) † [was 0.500 (0.324, 0.676)] | 0.684 (0.591, 0.767) † [was 0.778 (0.692, 0.849)] | 0.351 (0.229, 0.489) † [was 0.395 (0.250, 0.556)] | 0.851 (0.763, 0.916) † [was 0.843 (0.760, 0.906)] | 0.660 (0.543, 0.767) | 0.391 (0.251, 0.561) | 0.249 (0.213, 0.286) | 0.174 | 0.166 (0.130, 0.207) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | Re-run on age>=15 | 151 | 34 | 0.588 (0.407, 0.754) † [was 0.471 (0.298, 0.649)] | 0.658 (0.565, 0.743) † [was 0.812 (0.729, 0.878)] | 0.333 (0.217, 0.467) † [was 0.421 (0.263, 0.592)] | 0.846 (0.755, 0.913) † [was 0.841 (0.760, 0.903)] | 0.648 (0.529, 0.759) | 0.396 (0.250, 0.571) | 0.223 (0.194, 0.252) | 0.174 | 0.169 (0.133, 0.211) | 0.5015 † [was 0.593] |
| XGBoost | Model 3 | Main analysis (all ages) | 159 | 35 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.475 (0.343, 0.609) † [was 0.463 (0.326, 0.604)] | 0.930 (0.861, 0.971) † [was 0.905 (0.832, 0.953)] | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.179 † [was 0.2239] |
| XGBoost | Model 3 | Main models, age>=15 test rows | 151 | 34 | 0.794 (0.621, 0.913) † [was 0.706 (0.525, 0.849)] | 0.752 (0.664, 0.827) † [was 0.761 (0.673, 0.835)] | 0.482 (0.347, 0.620) † [was 0.462 (0.322, 0.605)] | 0.926 (0.854, 0.970) † [was 0.899 (0.822, 0.950)] | 0.845 (0.769, 0.910) | 0.600 (0.434, 0.769) | 0.126 (0.094, 0.159) | 0.174 | 0.126 (0.092, 0.162) | 0.179 † [was 0.2239] |
| XGBoost | Model 3 | Re-run on age>=15 | 151 | 34 | 0.765 (0.588, 0.893) † [was 0.735 (0.556, 0.871)] | 0.744 (0.655, 0.820) † [was 0.778 (0.692, 0.849)] | 0.464 (0.330, 0.603) † [was 0.490 (0.348, 0.634)] | 0.916 (0.841, 0.963) † [was 0.910 (0.836, 0.958)] | 0.843 (0.765, 0.910) | 0.621 (0.454, 0.786) | 0.141 (0.111, 0.175) | 0.174 | 0.126 (0.093, 0.163) | 0.3178 † [was 0.3958] |
| XGBoost | Model 4 | Main analysis (all ages) | 159 | 35 | 0.829 (0.664, 0.934) † [was 0.714 (0.537, 0.854)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.408 (0.293, 0.532) † [was 0.463 (0.326, 0.604)] | 0.932 (0.857, 0.975) † [was 0.905 (0.832, 0.953)] | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | Main models, age>=15 test rows | 151 | 34 | 0.824 (0.655, 0.932) † [was 0.706 (0.525, 0.849)] | 0.658 (0.565, 0.743) † [was 0.769 (0.682, 0.842)] | 0.412 (0.294, 0.538) † [was 0.471 (0.329, 0.615)] | 0.928 (0.849, 0.973) † [was 0.900 (0.824, 0.951)] | 0.840 (0.750, 0.908) | 0.633 (0.456, 0.801) | 0.131 (0.096, 0.171) | 0.174 | 0.122 (0.087, 0.162) | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | Re-run on age>=15 | 151 | 34 | 0.794 (0.621, 0.913) † [was 0.706 (0.525, 0.849)] | 0.684 (0.591, 0.767) † [was 0.838 (0.758, 0.899)] | 0.422 (0.299, 0.552) † [was 0.558 (0.399, 0.709)] | 0.920 (0.841, 0.967) † [was 0.907 (0.836, 0.955)] | 0.860 (0.783, 0.923) | 0.693 (0.522, 0.832) | 0.135 (0.101, 0.172) | 0.174 | 0.114 (0.081, 0.150) | 0.2885 † [was 0.4739] |
| XGBoost | Model 5 | Main analysis (all ages) | 159 | 35 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | Main models, age>=15 test rows | 151 | 34 | 0.794 (0.621, 0.913) † [was 0.706 (0.525, 0.849)] | 0.709 (0.618, 0.790) † [was 0.812 (0.729, 0.878)] | 0.443 (0.315, 0.576) † [was 0.522 (0.369, 0.671)] | 0.922 (0.846, 0.968) † [was 0.905 (0.832, 0.953)] | 0.882 (0.815, 0.936) | 0.746 (0.592, 0.863) | 0.118 (0.082, 0.158) | 0.174 | 0.106 (0.071, 0.146) | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | Re-run on age>=15 | 151 | 34 | 0.882 (0.725, 0.967) † [was 0.765 (0.588, 0.893)] | 0.752 (0.664, 0.827) † [was 0.846 (0.768, 0.906)] | 0.508 (0.375, 0.641) † [was 0.591 (0.432, 0.737)] | 0.957 (0.892, 0.988) † [was 0.925 (0.858, 0.967)] | 0.903 (0.835, 0.954) | 0.789 (0.650, 0.901) | 0.103 (0.073, 0.138) | 0.174 | 0.091 (0.061, 0.127) | 0.2256 † [was 0.417] |
| CatBoost | Model 1 | Main analysis (all ages) | 159 | 35 | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | Main models, age>=15 test rows | 151 | 34 | 0.382 (0.222, 0.564) † [was 0.206 (0.087, 0.379)] | 0.786 (0.701, 0.857) † [was 0.829 (0.748, 0.892)] | 0.342 (0.196, 0.514) † [was 0.259 (0.111, 0.463)] | 0.814 (0.730, 0.881) † [was 0.782 (0.699, 0.851)] | 0.592 (0.488, 0.702) | 0.293 (0.202, 0.444) | 0.197 (0.181, 0.214) | 0.174 | 0.175 (0.134, 0.218) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | Re-run on age>=15 | 151 | 34 | 0.529 (0.351, 0.702) † [was 0.382 (0.222, 0.564)] | 0.726 (0.636, 0.805) † [was 0.795 (0.710, 0.864)] | 0.360 (0.229, 0.508) † [was 0.351 (0.202, 0.525)] | 0.842 (0.756, 0.907) † [was 0.816 (0.732, 0.882)] | 0.650 (0.539, 0.755) | 0.385 (0.251, 0.553) | 0.167 (0.136, 0.199) | 0.174 | 0.167 (0.130, 0.207) | 0.2914 † [was 0.3337] |
| CatBoost | Model 2 | Main analysis (all ages) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | Main models, age>=15 test rows | 151 | 34 | 0.588 (0.407, 0.754) † [was 0.529 (0.351, 0.702)] | 0.607 (0.512, 0.696) † [was 0.726 (0.636, 0.805)] | 0.303 (0.196, 0.429) † [was 0.360 (0.229, 0.508)] | 0.835 (0.739, 0.907) † [was 0.842 (0.756, 0.907)] | 0.640 (0.526, 0.743) | 0.363 (0.233, 0.526) | 0.231 (0.224, 0.238) | 0.174 | 0.169 (0.134, 0.208) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | Re-run on age>=15 | 151 | 34 | 0.559 (0.379, 0.728) † [was 0.382 (0.222, 0.564)] | 0.650 (0.556, 0.735) † [was 0.752 (0.664, 0.827)] | 0.317 (0.203, 0.450) † [was 0.310 (0.176, 0.471)] | 0.835 (0.743, 0.905) † [was 0.807 (0.721, 0.877)] | 0.658 (0.543, 0.760) | 0.383 (0.247, 0.548) | 0.169 (0.143, 0.198) | 0.174 | 0.168 (0.133, 0.209) | 0.3072 † [was 0.3351] |
| CatBoost | Model 3 | Main analysis (all ages) | 159 | 35 | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.441 (0.321, 0.567) † [was 0.490 (0.348, 0.634)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | Main models, age>=15 test rows | 151 | 34 | 0.853 (0.689, 0.950) † [was 0.706 (0.525, 0.849)] | 0.684 (0.591, 0.767) † [was 0.786 (0.701, 0.857)] | 0.439 (0.317, 0.567) † [was 0.490 (0.344, 0.637)] | 0.941 (0.868, 0.981) † [was 0.902 (0.827, 0.952)] | 0.821 (0.741, 0.891) | 0.586 (0.425, 0.739) | 0.200 (0.190, 0.210) | 0.174 | 0.133 (0.098, 0.170) | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | Re-run on age>=15 | 151 | 34 | 0.824 (0.655, 0.932) † [was 0.735 (0.556, 0.871)] | 0.684 (0.591, 0.767) † [was 0.761 (0.673, 0.835)] | 0.431 (0.308, 0.560) † [was 0.472 (0.333, 0.614)] | 0.930 (0.854, 0.974) † [was 0.908 (0.833, 0.957)] | 0.833 (0.756, 0.900) | 0.597 (0.437, 0.773) | 0.159 (0.126, 0.195) | 0.174 | 0.129 (0.097, 0.165) | 0.3631 † [was 0.4718] |
| CatBoost | Model 4 | Main analysis (all ages) | 159 | 35 | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.422 (0.299, 0.552) † [was 0.553 (0.401, 0.698)] | 0.916 (0.841, 0.963) † [was 0.920 (0.853, 0.963)] | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | Main models, age>=15 test rows | 151 | 34 | 0.765 (0.588, 0.893) † [was 0.735 (0.556, 0.871)] | 0.701 (0.609, 0.782) † [was 0.838 (0.758, 0.899)] | 0.426 (0.300, 0.559) † [was 0.568 (0.410, 0.717)] | 0.911 (0.832, 0.961) † [was 0.916 (0.846, 0.961)] | 0.832 (0.740, 0.908) | 0.618 (0.436, 0.798) | 0.128 (0.101, 0.156) | 0.174 | 0.121 (0.088, 0.157) | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | Re-run on age>=15 | 151 | 34 | 0.824 (0.655, 0.932) † [was 0.735 (0.556, 0.871)] | 0.684 (0.591, 0.767) † [was 0.829 (0.748, 0.892)] | 0.431 (0.308, 0.560) † [was 0.556 (0.400, 0.704)] | 0.930 (0.854, 0.974) † [was 0.915 (0.845, 0.960)] | 0.862 (0.784, 0.925) | 0.659 (0.484, 0.824) | 0.116 (0.088, 0.145) | 0.174 | 0.113 (0.081, 0.147) | 0.2002 † [was 0.285] |
| CatBoost | Model 5 | Main analysis (all ages) | 159 | 35 | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | Main models, age>=15 test rows | 151 | 34 | 0.941 (0.803, 0.993) † [was 0.853 (0.689, 0.950)] | 0.709 (0.618, 0.790) † [was 0.846 (0.768, 0.906)] | 0.485 (0.360, 0.611) † [was 0.617 (0.464, 0.755)] | 0.976 (0.918, 0.997) † [was 0.952 (0.891, 0.984)] | 0.922 (0.858, 0.970) | 0.843 (0.729, 0.925) | 0.087 (0.063, 0.115) | 0.174 | 0.084 (0.056, 0.116) | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | Re-run on age>=15 | 151 | 34 | 0.941 (0.803, 0.993) † [was 0.882 (0.725, 0.967)] | 0.726 (0.636, 0.805) † [was 0.829 (0.748, 0.892)] | 0.500 (0.372, 0.628) † [was 0.600 (0.452, 0.736)] | 0.977 (0.919, 0.997) † [was 0.960 (0.902, 0.989)] | 0.917 (0.846, 0.971) | 0.842 (0.729, 0.927) | 0.105 (0.085, 0.127) | 0.174 | 0.084 (0.056, 0.116) | 0.2493 † [was 0.309] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Features — identical to v1 in every row ✓. 150 cell(s) changed (†).

**QFT-excluded re-run** (`rev_qft_excluded_sensitivity.csv`):

| Algorithm | Model | Analysis | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main analysis | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | QFT excluded | 0.514 (0.340, 0.686) † [was 0.400 (0.239, 0.579)] | 0.694 (0.604, 0.773) † [was 0.823 (0.744, 0.885)] | 0.321 (0.203, 0.460) † [was 0.389 (0.231, 0.565)] | 0.835 (0.749, 0.901) † [was 0.829 (0.751, 0.891)] | 0.642 (0.532, 0.748) | 0.319 (0.218, 0.483) | 0.231 (0.209, 0.254) | 0.172 | 0.166 (0.133, 0.200) | 0.5229 † [was 0.5792] |
| XGBoost | Model 2 | Main analysis | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | QFT excluded | 0.600 (0.421, 0.761) † [was 0.457 (0.288, 0.634)] | 0.694 (0.604, 0.773) † [was 0.823 (0.744, 0.885)] | 0.356 (0.236, 0.491) † [was 0.421 (0.263, 0.592)] | 0.860 (0.776, 0.921) † [was 0.843 (0.766, 0.903)] | 0.660 (0.546, 0.768) | 0.353 (0.242, 0.526) | 0.164 (0.130, 0.199) | 0.172 | 0.165 (0.131, 0.201) | 0.2293 † [was 0.2742] |
| XGBoost | Model 3 | Main analysis | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.475 (0.343, 0.609) † [was 0.463 (0.326, 0.604)] | 0.930 (0.861, 0.971) † [was 0.905 (0.832, 0.953)] | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.179 † [was 0.2239] |
| XGBoost | Model 3 | QFT excluded | 0.771 (0.599, 0.896) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.815 (0.735, 0.879)] | 0.429 (0.305, 0.560) † [was 0.521 (0.372, 0.667)] | 0.917 (0.842, 0.963) † [was 0.910 (0.841, 0.956)] | 0.845 (0.767, 0.911) | 0.610 (0.452, 0.789) | 0.124 (0.095, 0.157) | 0.172 | 0.123 (0.092, 0.157) | 0.185 † [was 0.2591] |
| XGBoost | Model 4 | Main analysis | 0.829 (0.664, 0.934) † [was 0.714 (0.537, 0.854)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.408 (0.293, 0.532) † [was 0.463 (0.326, 0.604)] | 0.932 (0.857, 0.975) † [was 0.905 (0.832, 0.953)] | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | QFT excluded | 0.771 (0.599, 0.896) | 0.718 (0.630, 0.795) † [was 0.831 (0.753, 0.892)] | 0.435 (0.310, 0.567) † [was 0.562 (0.412, 0.705)] | 0.918 (0.844, 0.964) † [was 0.928 (0.863, 0.968)] | 0.850 (0.767, 0.922) | 0.692 (0.528, 0.836) | 0.138 (0.108, 0.172) | 0.172 | 0.109 (0.078, 0.142) | 0.3536 † [was 0.5093] |
| XGBoost | Model 5 | Main analysis | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | QFT excluded | 0.914 (0.769, 0.982) † [was 0.857 (0.697, 0.952)] | 0.782 (0.699, 0.851) † [was 0.847 (0.771, 0.905)] | 0.542 (0.408, 0.673) † [was 0.612 (0.462, 0.748)] | 0.970 (0.915, 0.994) † [was 0.955 (0.897, 0.985)] | 0.913 (0.846, 0.964) | 0.814 (0.691, 0.909) | 0.097 (0.068, 0.130) | 0.172 | 0.088 (0.059, 0.122) | 0.233 † [was 0.3405] |
| CatBoost | Model 1 | Main analysis | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | QFT excluded | 0.514 (0.340, 0.686) | 0.710 (0.621, 0.788) | 0.333 (0.211, 0.475) | 0.838 (0.753, 0.903) | 0.635 (0.532, 0.728) | 0.295 (0.206, 0.417) | 0.238 (0.235, 0.241) | 0.172 | 0.167 (0.131, 0.205) | 0.483 † [was 0.4851] |
| CatBoost | Model 2 | Main analysis | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | QFT excluded | 0.543 (0.366, 0.712) † [was 0.486 (0.314, 0.660)] | 0.661 (0.571, 0.744) † [was 0.758 (0.673, 0.830)] | 0.311 (0.199, 0.443) † [was 0.362 (0.227, 0.515)] | 0.837 (0.748, 0.904) † [was 0.839 (0.758, 0.902)] | 0.656 (0.541, 0.759) | 0.355 (0.235, 0.507) | 0.237 (0.234, 0.240) | 0.172 | 0.164 (0.131, 0.197) | 0.4868 † [was 0.4908] |
| CatBoost | Model 3 | Main analysis | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.441 (0.321, 0.567) † [was 0.490 (0.348, 0.634)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | QFT excluded | 0.886 (0.733, 0.968) † [was 0.800 (0.631, 0.916)] | 0.685 (0.596, 0.766) † [was 0.823 (0.744, 0.885)] | 0.443 (0.324, 0.567) † [was 0.560 (0.413, 0.700)] | 0.955 (0.889, 0.988) † [was 0.936 (0.872, 0.974)] | 0.865 (0.793, 0.926) | 0.665 (0.515, 0.805) | 0.201 (0.189, 0.213) | 0.172 | 0.117 (0.091, 0.145) | 0.5038 † [was 0.5459] |
| CatBoost | Model 4 | Main analysis | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.422 (0.299, 0.552) † [was 0.553 (0.401, 0.698)] | 0.916 (0.841, 0.963) † [was 0.920 (0.853, 0.963)] | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | QFT excluded | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.815 (0.735, 0.879)] | 0.422 (0.299, 0.552) † [was 0.531 (0.383, 0.675)] | 0.916 (0.841, 0.963) † [was 0.918 (0.850, 0.962)] | 0.841 (0.758, 0.911) | 0.650 (0.486, 0.801) | 0.139 (0.118, 0.161) | 0.172 | 0.118 (0.086, 0.152) | 0.3137 † [was 0.3613] |
| CatBoost | Model 5 | Main analysis | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | QFT excluded | 0.886 (0.733, 0.968) † [was 0.829 (0.664, 0.934)] | 0.790 (0.708, 0.858) † [was 0.879 (0.808, 0.931)] | 0.544 (0.407, 0.676) † [was 0.659 (0.501, 0.795)] | 0.961 (0.903, 0.989) † [was 0.948 (0.890, 0.981)] | 0.909 (0.835, 0.968) | 0.825 (0.698, 0.919) | 0.160 (0.149, 0.172) | 0.172 | 0.082 (0.054, 0.116) | 0.3964 † [was 0.4307] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Features — identical to v1 in every row ✓. 95 cell(s) changed (†).

**Age and diabetes subgroups (Supplementary Table S11)** — 5-fold OOF over all 795 with fixed M5 features and hyperparameters; sens/spec at the fixed M5 threshold (now the ≥ 0.70 threshold). Exploratory; < 10 TB cases flagged. File: `rev_subgroup_performance_oof795.csv`.

| Algorithm | Subgroup | n | TB cases | AUROC | AP | Sens | Spec | Brier | Null Brier | Thr (fixed M5) | Flag |
|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | All participants | 795 | 176 | 0.905 (0.879, 0.930) | 0.765 (0.702, 0.832) | 0.898 (0.843, 0.938) † [was 0.812 (0.747, 0.867)] | 0.722 (0.685, 0.757) † [was 0.840 (0.809, 0.868)] | 0.096 (0.082, 0.111) | 0.172 | 0.1303 † [was 0.2816] |  |
| XGBoost | Age 5–14 | 51 | 7 | 0.886 (0.612, 1.000) | 0.838 (0.501, 1.000) | 0.857 (0.421, 0.996) | 0.727 (0.572, 0.850) † [was 0.955 (0.845, 0.994)] | 0.053 (0.012, 0.112) | 0.126 | 0.1303 † [was 0.2816] | <10 TB cases — interpret with caution |
| XGBoost | Age 15–34 | 339 | 85 | 0.904 (0.862, 0.940) | 0.776 (0.676, 0.875) | 0.871 (0.780, 0.934) † [was 0.824 (0.726, 0.898)] | 0.748 (0.690, 0.800) † [was 0.862 (0.814, 0.902)] | 0.097 (0.073, 0.122) | 0.189 | 0.1303 † [was 0.2816] |  |
| XGBoost | Age 35–54 | 246 | 47 | 0.915 (0.872, 0.952) | 0.786 (0.681, 0.877) | 0.936 (0.825, 0.987) † [was 0.787 (0.643, 0.893)] | 0.749 (0.683, 0.807) † [was 0.824 (0.764, 0.874)] | 0.094 (0.069, 0.119) | 0.155 | 0.1303 † [was 0.2816] |  |
| XGBoost | Age ≥55 | 159 | 37 | 0.887 (0.823, 0.940) | 0.728 (0.579, 0.862) | 0.919 (0.781, 0.983) † [was 0.811 (0.648, 0.920)] | 0.623 (0.531, 0.709) † [was 0.779 (0.695, 0.849)] | 0.112 (0.081, 0.147) | 0.179 | 0.1303 † [was 0.2816] |  |
| XGBoost | Diabetes: yes | 50 | 19 | 0.973 (0.928, 0.998) | 0.961 (0.889, 0.998) | 1.000 (0.824, 1.000) † [was 0.947 (0.740, 0.999)] | 0.742 (0.554, 0.881) † [was 0.871 (0.702, 0.964)] | 0.074 (0.035, 0.119) | 0.261 | 0.1303 † [was 0.2816] |  |
| XGBoost | Diabetes: no | 745 | 157 | 0.898 (0.868, 0.924) | 0.741 (0.665, 0.811) | 0.885 (0.825, 0.931) † [was 0.796 (0.725, 0.856)] | 0.721 (0.683, 0.757) † [was 0.838 (0.806, 0.867)] | 0.098 (0.083, 0.114) | 0.166 | 0.1303 † [was 0.2816] |  |
| CatBoost | All participants | 795 | 176 | 0.901 (0.875, 0.925) | 0.761 (0.700, 0.818) | 0.892 (0.837, 0.934) † [was 0.818 (0.753, 0.872)] | 0.735 (0.698, 0.769) † [was 0.827 (0.795, 0.856)] | 0.098 (0.086, 0.110) | 0.172 | 0.1671 † [was 0.2493] |  |
| CatBoost | Age 5–14 | 51 | 7 | 0.870 (0.574, 1.000) | 0.792 (0.414, 1.000) | 0.857 (0.421, 0.996) | 0.864 (0.726, 0.948) † [was 0.955 (0.845, 0.994)] | 0.053 (0.018, 0.102) | 0.126 | 0.1671 † [was 0.2493] | <10 TB cases — interpret with caution |
| CatBoost | Age 15–34 | 339 | 85 | 0.901 (0.861, 0.938) | 0.820 (0.746, 0.882) | 0.847 (0.753, 0.916) † [was 0.800 (0.699, 0.879)] | 0.776 (0.719, 0.825) † [was 0.854 (0.805, 0.895)] | 0.100 (0.079, 0.120) | 0.189 | 0.1671 † [was 0.2493] |  |
| CatBoost | Age 35–54 | 246 | 47 | 0.918 (0.877, 0.955) | 0.723 (0.598, 0.839) | 0.957 (0.855, 0.995) † [was 0.851 (0.717, 0.938)] | 0.749 (0.683, 0.807) † [was 0.839 (0.781, 0.887)] | 0.090 (0.070, 0.111) | 0.155 | 0.1671 † [was 0.2493] |  |
| CatBoost | Age ≥55 | 159 | 37 | 0.868 (0.806, 0.921) | 0.660 (0.510, 0.813) | 0.919 (0.781, 0.983) † [was 0.811 (0.648, 0.920)] | 0.582 (0.489, 0.671) † [was 0.705 (0.616, 0.784)] | 0.122 (0.096, 0.149) | 0.179 | 0.1671 † [was 0.2493] |  |
| CatBoost | Diabetes: yes | 50 | 19 | 0.959 (0.896, 1.000) | 0.915 (0.772, 1.000) | 1.000 (0.824, 1.000) | 0.710 (0.520, 0.858) † [was 0.871 (0.702, 0.964)] | 0.104 (0.067, 0.146) | 0.261 | 0.1671 † [was 0.2493] |  |
| CatBoost | Diabetes: no | 745 | 157 | 0.895 (0.865, 0.921) | 0.745 (0.675, 0.807) | 0.879 (0.817, 0.926) † [was 0.796 (0.725, 0.856)] | 0.736 (0.699, 0.772) † [was 0.825 (0.792, 0.855)] | 0.098 (0.085, 0.111) | 0.166 | 0.1671 † [was 0.2493] |  |

*Verification:* n, TB cases, AUROC, AP, Brier, Null Brier — identical to v1 in every row ✓. 39 cell(s) changed (†).

## 11. Bacteriologically confirmed TB only (R3.1)

Exclusions identical to v1 ✓ (14 clinician-diagnosed-only cases excluded; 781 analysed). Results (`rev_bac_confirmed_sensitivity.csv`):

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main analysis (all TB vs no TB) | 159 | 35 | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | Main models, bac-confirmed test rows | 156 | 32 | 0.625 (0.437, 0.789) † [was 0.469 (0.291, 0.653)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.876 (0.794, 0.934) † [was 0.852 (0.774, 0.911)] | 0.661 (0.547, 0.772) | 0.334 (0.218, 0.506) | 0.235 (0.211, 0.260) | 0.163 | 0.157 (0.123, 0.191) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.719 (0.533, 0.863) † [was 0.531 (0.347, 0.709)] | 0.629 (0.538, 0.714) † [was 0.815 (0.735, 0.879)] | 0.333 (0.224, 0.457) † [was 0.425 (0.270, 0.591)] | 0.897 (0.813, 0.952) † [was 0.871 (0.796, 0.926)] | 0.695 (0.582, 0.803) | 0.380 (0.251, 0.562) | 0.213 (0.200, 0.226) | 0.163 | 0.151 (0.116, 0.185) | 0.4807 † [was 0.5255] |
| XGBoost | Model 2 | Main analysis (all TB vs no TB) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | Main models, bac-confirmed test rows | 156 | 32 | 0.656 (0.468, 0.814) † [was 0.562 (0.377, 0.736)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.884 (0.802, 0.941) † [was 0.873 (0.796, 0.929)] | 0.700 (0.584, 0.806) | 0.402 (0.260, 0.574) | 0.245 (0.211, 0.280) | 0.163 | 0.150 (0.116, 0.185) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.625 (0.437, 0.789) † [was 0.562 (0.377, 0.736)] | 0.669 (0.579, 0.751) † [was 0.766 (0.682, 0.837)] | 0.328 (0.213, 0.460) † [was 0.383 (0.245, 0.536)] | 0.874 (0.790, 0.933) † [was 0.872 (0.794, 0.928)] | 0.709 (0.602, 0.811) | 0.405 (0.264, 0.579) | 0.231 (0.196, 0.267) | 0.163 | 0.150 (0.114, 0.187) | 0.541 † [was 0.634] |
| XGBoost | Model 3 | Main analysis (all TB vs no TB) | 159 | 35 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.475 (0.343, 0.609) † [was 0.463 (0.326, 0.604)] | 0.930 (0.861, 0.971) † [was 0.905 (0.832, 0.953)] | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.179 † [was 0.2239] |
| XGBoost | Model 3 | Main models, bac-confirmed test rows | 156 | 32 | 0.812 (0.636, 0.928) † [was 0.750 (0.566, 0.885)] | 0.750 (0.664, 0.823) † [was 0.766 (0.682, 0.837)] | 0.456 (0.324, 0.593) † [was 0.453 (0.316, 0.596)] | 0.939 (0.873, 0.977) † [was 0.922 (0.853, 0.966)] | 0.859 (0.789, 0.918) | 0.592 (0.423, 0.765) | 0.115 (0.087, 0.144) | 0.163 | 0.115 (0.085, 0.146) | 0.179 † [was 0.2239] |
| XGBoost | Model 3 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.812 (0.636, 0.928) † [was 0.750 (0.566, 0.885)] | 0.742 (0.656, 0.816) † [was 0.774 (0.690, 0.844)] | 0.448 (0.317, 0.585) † [was 0.462 (0.322, 0.605)] | 0.939 (0.871, 0.977) † [was 0.923 (0.854, 0.966)] | 0.862 (0.788, 0.923) | 0.641 (0.470, 0.810) | 0.135 (0.106, 0.167) | 0.163 | 0.112 (0.081, 0.143) | 0.3235 † [was 0.4144] |
| XGBoost | Model 4 | Main analysis (all TB vs no TB) | 159 | 35 | 0.829 (0.664, 0.934) † [was 0.714 (0.537, 0.854)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.408 (0.293, 0.532) † [was 0.463 (0.326, 0.604)] | 0.932 (0.857, 0.975) † [was 0.905 (0.832, 0.953)] | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | Main models, bac-confirmed test rows | 156 | 32 | 0.875 (0.710, 0.965) † [was 0.750 (0.566, 0.885)] | 0.661 (0.571, 0.744) † [was 0.766 (0.682, 0.837)] | 0.400 (0.285, 0.524) † [was 0.453 (0.316, 0.596)] | 0.953 (0.885, 0.987) † [was 0.922 (0.853, 0.966)] | 0.868 (0.793, 0.933) | 0.659 (0.487, 0.831) | 0.116 (0.085, 0.150) | 0.163 | 0.106 (0.075, 0.141) | 0.1846 † [was 0.3091] |
| XGBoost | Model 4 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.812 (0.636, 0.928) † [was 0.750 (0.566, 0.885)] | 0.726 (0.638, 0.802) † [was 0.782 (0.699, 0.851)] | 0.433 (0.306, 0.568) † [was 0.471 (0.329, 0.615)] | 0.938 (0.869, 0.977) † [was 0.924 (0.855, 0.967)] | 0.868 (0.792, 0.932) | 0.707 (0.556, 0.844) | 0.129 (0.094, 0.165) | 0.163 | 0.104 (0.073, 0.139) | 0.3076 † [was 0.3927] |
| XGBoost | Model 5 | Main analysis (all TB vs no TB) | 159 | 35 | 0.800 (0.631, 0.916) † [was 0.714 (0.537, 0.854)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.438 (0.314, 0.567) † [was 0.532 (0.381, 0.679)] | 0.926 (0.854, 0.970) † [was 0.911 (0.842, 0.956)] | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | Main models, bac-confirmed test rows | 156 | 32 | 0.844 (0.672, 0.947) † [was 0.781 (0.600, 0.907)] | 0.710 (0.621, 0.788) † [was 0.823 (0.744, 0.885)] | 0.429 (0.305, 0.560) † [was 0.532 (0.381, 0.679)] | 0.946 (0.879, 0.982) † [was 0.936 (0.872, 0.974)] | 0.908 (0.845, 0.957) | 0.784 (0.641, 0.899) | 0.099 (0.067, 0.134) | 0.163 | 0.087 (0.057, 0.121) | 0.1303 † [was 0.2816] |
| XGBoost | Model 5 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.875 (0.710, 0.965) † [was 0.812 (0.636, 0.928)] | 0.774 (0.690, 0.844) † [was 0.815 (0.735, 0.879)] | 0.500 (0.363, 0.637) † [was 0.531 (0.383, 0.675)] | 0.960 (0.901, 0.989) † [was 0.944 (0.882, 0.979)] | 0.907 (0.832, 0.963) | 0.799 (0.655, 0.913) | 0.078 (0.046, 0.114) | 0.163 | 0.078 (0.047, 0.113) | 0.0768 † [was 0.1153] |
| CatBoost | Model 1 | Main analysis (all TB vs no TB) | 159 | 35 | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | Main models, bac-confirmed test rows | 156 | 32 | 0.406 (0.237, 0.594) † [was 0.250 (0.115, 0.434)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.295 (0.168, 0.452) † [was 0.235 (0.107, 0.412)] | 0.830 (0.748, 0.895) † [was 0.803 (0.722, 0.870)] | 0.592 (0.476, 0.698) | 0.279 (0.183, 0.440) | 0.194 (0.179, 0.209) | 0.163 | 0.164 (0.127, 0.201) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.625 (0.437, 0.789) † [was 0.375 (0.211, 0.563)] | 0.702 (0.613, 0.780) † [was 0.790 (0.708, 0.858)] | 0.351 (0.229, 0.489) † [was 0.316 (0.175, 0.487)] | 0.879 (0.798, 0.936) † [was 0.831 (0.750, 0.893)] | 0.683 (0.575, 0.787) | 0.383 (0.239, 0.550) | 0.265 (0.237, 0.295) | 0.163 | 0.153 (0.118, 0.188) | 0.6372 † [was 0.7077] |
| CatBoost | Model 2 | Main analysis (all TB vs no TB) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | Main models, bac-confirmed test rows | 156 | 32 | 0.656 (0.468, 0.814) † [was 0.594 (0.406, 0.763)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.874 (0.785, 0.935) † [was 0.875 (0.796, 0.932)] | 0.690 (0.578, 0.796) | 0.404 (0.256, 0.572) | 0.229 (0.222, 0.235) | 0.163 | 0.152 (0.118, 0.184) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.594 (0.406, 0.763) † [was 0.406 (0.237, 0.594)] | 0.734 (0.647, 0.809) † [was 0.879 (0.808, 0.931)] | 0.365 (0.236, 0.510) † [was 0.464 (0.275, 0.661)] | 0.875 (0.796, 0.932) † [was 0.852 (0.778, 0.908)] | 0.729 (0.623, 0.828) | 0.439 (0.287, 0.626) | 0.158 (0.130, 0.185) | 0.163 | 0.146 (0.108, 0.183) | 0.3809 † [was 0.487] |
| CatBoost | Model 3 | Main analysis (all TB vs no TB) | 159 | 35 | 0.857 (0.697, 0.952) † [was 0.714 (0.537, 0.854)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.441 (0.321, 0.567) † [was 0.490 (0.348, 0.634)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | Main models, bac-confirmed test rows | 156 | 32 | 0.844 (0.672, 0.947) † [was 0.688 (0.500, 0.839)] | 0.694 (0.604, 0.773) † [was 0.790 (0.708, 0.858)] | 0.415 (0.294, 0.544) † [was 0.458 (0.314, 0.608)] | 0.945 (0.876, 0.982) † [was 0.907 (0.836, 0.955)] | 0.821 (0.739, 0.894) | 0.564 (0.397, 0.720) | 0.198 (0.189, 0.207) | 0.163 | 0.127 (0.095, 0.162) | 0.4355 † [was 0.4466] |
| CatBoost | Model 3 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.844 (0.672, 0.947) † [was 0.750 (0.566, 0.885)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.466 (0.333, 0.601) † [was 0.480 (0.337, 0.626)] | 0.949 (0.885, 0.983) † [was 0.925 (0.857, 0.967)] | 0.861 (0.793, 0.920) | 0.600 (0.434, 0.767) | 0.119 (0.091, 0.149) | 0.163 | 0.116 (0.084, 0.150) | 0.1997 † [was 0.2489] |
| CatBoost | Model 4 | Main analysis (all TB vs no TB) | 159 | 35 | 0.771 (0.599, 0.896) † [was 0.743 (0.567, 0.875)] | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.422 (0.299, 0.552) † [was 0.553 (0.401, 0.698)] | 0.916 (0.841, 0.963) † [was 0.920 (0.853, 0.963)] | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | Main models, bac-confirmed test rows | 156 | 32 | 0.781 (0.600, 0.907) | 0.702 (0.613, 0.780) † [was 0.831 (0.753, 0.892)] | 0.403 (0.281, 0.536) † [was 0.543 (0.390, 0.691)] | 0.926 (0.853, 0.970) † [was 0.936 (0.873, 0.974)] | 0.852 (0.767, 0.927) | 0.634 (0.473, 0.824) | 0.118 (0.093, 0.144) | 0.163 | 0.107 (0.077, 0.139) | 0.2483 † [was 0.3222] |
| CatBoost | Model 4 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.844 (0.672, 0.947) † [was 0.812 (0.636, 0.928)] | 0.702 (0.613, 0.780) † [was 0.806 (0.726, 0.872)] | 0.422 (0.299, 0.552) † [was 0.520 (0.374, 0.663)] | 0.946 (0.878, 0.982) † [was 0.943 (0.881, 0.979)] | 0.901 (0.835, 0.957) | 0.797 (0.665, 0.904) | 0.157 (0.130, 0.187) | 0.163 | 0.092 (0.065, 0.122) | 0.4444 † [was 0.5593] |
| CatBoost | Model 5 | Main analysis (all TB vs no TB) | 159 | 35 | 0.943 (0.808, 0.993) † [was 0.857 (0.697, 0.952)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.478 (0.356, 0.602) † [was 0.625 (0.474, 0.760)] | 0.978 (0.922, 0.997) † [was 0.955 (0.898, 0.985)] | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | Main models, bac-confirmed test rows | 156 | 32 | 0.938 (0.792, 0.992) † [was 0.906 (0.750, 0.980)] | 0.710 (0.621, 0.788) † [was 0.855 (0.780, 0.912)] | 0.455 (0.331, 0.582) † [was 0.617 (0.464, 0.755)] | 0.978 (0.922, 0.997) † [was 0.972 (0.922, 0.994)] | 0.934 (0.873, 0.981) | 0.870 (0.771, 0.949) | 0.075 (0.054, 0.099) | 0.163 | 0.069 (0.045, 0.097) | 0.1671 † [was 0.2493] |
| CatBoost | Model 5 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.938 (0.792, 0.992) † [was 0.875 (0.710, 0.965)] | 0.694 (0.604, 0.773) † [was 0.855 (0.780, 0.912)] | 0.441 (0.321, 0.567) † [was 0.609 (0.454, 0.749)] | 0.977 (0.920, 0.997) † [was 0.964 (0.910, 0.990)] | 0.931 (0.874, 0.977) | 0.858 (0.747, 0.943) | 0.075 (0.052, 0.103) | 0.163 | 0.070 (0.044, 0.101) | 0.1255 † [was 0.1782] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Features — identical to v1 in every row ✓. 149 cell(s) changed (†).

## 12. Participants with and without a chest X-ray (Supplementary Table S9; R3.2)

Included (n = 795) vs excluded (n = 316) comparison — threshold-free, identical to v1 ✓:

| Variable | Level | Included: CXR available (N=795) | Excluded: no CXR (N=316) | of which No scan (N=289) | of which Not in Qure (N=27) | p-value (included vs excluded) | Test |
|---|---|---|---|---|---|---|---|
| Age, mean (SD) |  | 37.0 (17.3) | 39.0 (16.9) | 39.1 (17.2) | 37.7 (13.6) | 0.091 | t-test |
| Sex, n (%) | Female | 416 (52.3) | 150 (47.5) | 136 (47.1) | 14 (51.9) | 0.144 | Chi-square |
|  | Male | 379 (47.7) | 166 (52.5) | 153 (52.9) | 13 (48.1) |  |  |
| BMI, mean (SD) |  | 20.9 (4.8) | 21.8 (5.1) | 21.8 (5.1) | 22.2 (5.6) | 0.006 | t-test |
| Persistent cough, n (%) | No | 151 (19.0) | 48 (15.2) | 40 (13.8) | 8 (29.6) | 0.136 | Chi-square |
|  | Yes | 644 (81.0) | 268 (84.8) | 249 (86.2) | 19 (70.4) |  |  |
| Shortness breath, n (%) | No | 402 (50.6) | 133 (42.1) | 123 (42.6) | 10 (37.0) | 0.011 | Chi-square |
|  | Yes | 393 (49.4) | 183 (57.9) | 166 (57.4) | 17 (63.0) |  |  |
| Fever, n (%) | No | 43 (5.4) | 44 (13.9) | 39 (13.5) | 5 (18.5) | <0.001 | Chi-square |
|  | Yes | 752 (94.6) | 272 (86.1) | 250 (86.5) | 22 (81.5) |  |  |
| Fatigue, n (%) | No | 90 (11.3) | 57 (18.0) | 50 (17.3) | 7 (25.9) | 0.003 | Chi-square |
|  | Yes | 705 (88.7) | 259 (82.0) | 239 (82.7) | 20 (74.1) |  |  |
| Hemoptysis, n (%) | No | 707 (88.9) | 274 (86.7) | 248 (85.8) | 26 (96.3) | 0.299 | Chi-square |
|  | Yes | 88 (11.1) | 42 (13.3) | 41 (14.2) | 1 (3.7) |  |  |
| Weight loss, n (%) | No | 272 (34.2) | 118 (37.3) | 110 (38.1) | 8 (29.6) | 0.324 | Chi-square |
|  | Yes | 523 (65.8) | 198 (62.7) | 179 (61.9) | 19 (70.4) |  |  |
| Night sweats, n (%) | No | 541 (68.1) | 223 (70.6) | 212 (73.4) | 11 (40.7) | 0.414 | Chi-square |
|  | Yes | 254 (31.9) | 93 (29.4) | 77 (26.6) | 16 (59.3) |  |  |
| Prior TB disease (self-report), n (%) | No | 626 (78.7) | 254 (80.4) | 231 (79.9) | 23 (85.2) | 0.544 | Chi-square |
|  | Yes | 169 (21.3) | 62 (19.6) | 58 (20.1) | 4 (14.8) |  |  |
| QFT result, n (%) | Indeterminate | 37 (4.7) | 20 (6.3) | 20 (6.9) | 0 (0.0) | 0.149 | Chi-square |
|  | Negative | 300 (37.7) | 133 (42.1) | 124 (42.9) | 9 (33.3) |  |  |
|  | Positive | 458 (57.6) | 163 (51.6) | 145 (50.2) | 18 (66.7) |  |  |
| TB disease, n (%) | No | 619 (77.9) | 228 (72.2) | 205 (70.9) | 23 (85.2) | 0.044 | Chi-square |
|  | Yes | 176 (22.1) | 88 (27.8) | 84 (29.1) | 4 (14.8) |  |  |

**M1/M2 on all eligible participants** (new split of 1,111; `rev_all_eligible_M1M2_vs_cxr_restricted.csv`):

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | CXR-restricted (main, n=795) | 159 | 35 | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | All eligible (n=1111) | 223 | 53 | 0.679 (0.537, 0.801) † [was 0.491 (0.351, 0.632)] | 0.653 (0.576, 0.724) † [was 0.776 (0.706, 0.837)] | 0.379 (0.281, 0.484) † [was 0.406 (0.285, 0.536)] | 0.867 (0.796, 0.921) † [was 0.830 (0.763, 0.885)] | 0.700 (0.626, 0.775) | 0.369 (0.281, 0.503) | 0.206 (0.184, 0.229) | 0.181 | 0.168 (0.144, 0.194) | 0.4754 † [was 0.568] |
| XGBoost | Model 2 | CXR-restricted (main, n=795) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | All eligible (n=1111) | 223 | 53 | 0.660 (0.517, 0.785) † [was 0.604 (0.460, 0.735)] | 0.612 (0.534, 0.685) † [was 0.776 (0.706, 0.837)] | 0.347 (0.255, 0.448) † [was 0.457 (0.337, 0.581)] | 0.852 (0.777, 0.910) † [was 0.863 (0.798, 0.913)] | 0.707 (0.627, 0.788) | 0.392 (0.297, 0.533) | 0.243 (0.212, 0.275) | 0.181 | 0.168 (0.142, 0.197) | 0.5496 † [was 0.6701] |
| CatBoost | Model 1 | CXR-restricted (main, n=795) | 159 | 35 | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | All eligible (n=1111) | 223 | 53 | 0.660 (0.517, 0.785) † [was 0.509 (0.368, 0.649)] | 0.671 (0.594, 0.741) † [was 0.771 (0.700, 0.831)] | 0.385 (0.284, 0.492) † [was 0.409 (0.290, 0.537)] | 0.864 (0.793, 0.917) † [was 0.834 (0.767, 0.889)] | 0.724 (0.655, 0.793) | 0.400 (0.304, 0.535) | 0.163 (0.139, 0.189) | 0.181 | 0.163 (0.138, 0.189) | 0.2698 † [was 0.327] |
| CatBoost | Model 2 | CXR-restricted (main, n=795) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | All eligible (n=1111) | 223 | 53 | 0.642 (0.498, 0.769) † [was 0.528 (0.386, 0.667)] | 0.671 (0.594, 0.741) † [was 0.765 (0.694, 0.826)] | 0.378 (0.278, 0.486) † [was 0.412 (0.294, 0.538)] | 0.857 (0.786, 0.912) † [was 0.839 (0.771, 0.893)] | 0.678 (0.587, 0.764) | 0.437 (0.314, 0.571) | 0.188 (0.173, 0.203) | 0.181 | 0.166 (0.140, 0.194) | 0.3631 † [was 0.374] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Features — identical to v1 in every row ✓. 40 cell(s) changed (†).

**Main CXR-trained M1/M2 scored on the 316 participants without a CXR** (`rev_noCXR_holdout_M1M2.csv`):

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main model, CXR test set (n=159) | 159 | 35 | 0.571 (0.394, 0.737) † [was 0.429 (0.263, 0.606)] | 0.685 (0.596, 0.766) † [was 0.790 (0.708, 0.858)] | 0.339 (0.221, 0.474) † [was 0.366 (0.221, 0.531)] | 0.850 (0.765, 0.914) † [was 0.831 (0.750, 0.893)] | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.5532 † [was 0.6183] |
| XGBoost | Model 1 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.580 (0.470, 0.684) † [was 0.386 (0.284, 0.496)] | 0.737 (0.675, 0.793) † [was 0.829 (0.774, 0.875)] | 0.459 (0.364, 0.557) † [was 0.466 (0.348, 0.586)] | 0.820 (0.760, 0.870) † [was 0.778 (0.720, 0.828)] | 0.706 (0.637, 0.769) | 0.482 (0.387, 0.590) | 0.206 (0.188, 0.224) | 0.204 | 0.185 (0.157, 0.212) | 0.5532 † [was 0.6183] |
| XGBoost | Model 2 | Main model, CXR test set (n=159) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.514 (0.340, 0.686)] | 0.677 (0.588, 0.759) † [was 0.774 (0.690, 0.844)] | 0.344 (0.227, 0.477) † [was 0.391 (0.251, 0.546)] | 0.857 (0.772, 0.920) † [was 0.850 (0.770, 0.910)] | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.5771 † [was 0.6506] |
| XGBoost | Model 2 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.568 (0.458, 0.673) † [was 0.455 (0.348, 0.564)] | 0.798 (0.740, 0.848) † [was 0.851 (0.798, 0.894)] | 0.521 (0.416, 0.624) † [was 0.541 (0.421, 0.657)] | 0.827 (0.771, 0.875) † [was 0.802 (0.746, 0.850)] | 0.749 (0.686, 0.806) | 0.512 (0.415, 0.628) | 0.198 (0.177, 0.221) | 0.204 | 0.179 (0.152, 0.207) | 0.5771 † [was 0.6506] |
| CatBoost | Model 1 | Main model, CXR test set (n=159) | 159 | 35 | 0.400 (0.239, 0.579) † [was 0.229 (0.104, 0.401)] | 0.750 (0.664, 0.823) † [was 0.790 (0.708, 0.858)] | 0.311 (0.182, 0.466) † [was 0.235 (0.107, 0.412)] | 0.816 (0.732, 0.882) † [was 0.784 (0.702, 0.853)] | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4044 † [was 0.4181] |
| CatBoost | Model 1 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.307 (0.213, 0.414) † [was 0.239 (0.154, 0.341)] | 0.776 (0.717, 0.829) † [was 0.825 (0.769, 0.872)] | 0.346 (0.242, 0.462) † [was 0.344 (0.227, 0.477)] | 0.744 (0.683, 0.798) † [was 0.737 (0.679, 0.790)] | 0.598 (0.527, 0.670) | 0.349 (0.279, 0.454) | 0.208 (0.197, 0.220) | 0.204 | 0.206 (0.176, 0.236) | 0.4044 † [was 0.4181] |
| CatBoost | Model 2 | Main model, CXR test set (n=159) | 159 | 35 | 0.600 (0.421, 0.761) † [was 0.543 (0.366, 0.712)] | 0.613 (0.521, 0.699) † [was 0.734 (0.647, 0.809)] | 0.304 (0.199, 0.427) † [was 0.365 (0.236, 0.510)] | 0.844 (0.753, 0.912) † [was 0.850 (0.769, 0.912)] | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.4875 † [was 0.5033] |
| CatBoost | Model 2 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.580 (0.470, 0.684) † [was 0.443 (0.337, 0.553)] | 0.750 (0.689, 0.805) † [was 0.833 (0.778, 0.879)] | 0.472 (0.375, 0.571) † [was 0.506 (0.390, 0.622)] | 0.822 (0.763, 0.872) † [was 0.795 (0.738, 0.844)] | 0.714 (0.648, 0.776) | 0.500 (0.394, 0.614) | 0.226 (0.221, 0.231) | 0.204 | 0.184 (0.157, 0.212) | 0.4875 † [was 0.5033] |

*Verification:* AUROC, AP, Brier, Null Brier, Brier (recal.), Features — identical to v1 in every row ✓. 40 cell(s) changed (†).

## 13. Class weighting actually used

Final M1–M5 weighting identical to v1 ✓: XGBoost M1, M2, M4, M5 and CatBoost M2 weighted; full-feature models unweighted. The nested-CV fold models are re-tuned at ≥ 0.70, so their weighting can differ (`rev_class_weighting_audit.csv`):

| Algo | Source | Weighted | Detail |
|---|---|---|---|
| CAT | Nested-CV OOF spec=0.70 fold 1 | True | class_weights=[1.000, 1.903] |
| CAT | Nested-CV OOF spec=0.70 fold 2 | True | class_weights=[1.000, 5.466] |
| CAT | Nested-CV OOF spec=0.70 fold 3 | False | class_weights=None |
| CAT | Nested-CV OOF spec=0.70 fold 4 | True | class_weights=[1.000, 6.532] |
| CAT | Nested-CV OOF spec=0.70 fold 5 | True | class_weights=[1.000, 2.157] |
| XGB | Nested-CV OOF spec=0.70 fold 1 | True | scale_pos_weight=5.143 |
| XGB | Nested-CV OOF spec=0.70 fold 2 | False | scale_pos_weight=1.000 |
| XGB | Nested-CV OOF spec=0.70 fold 3 | True | scale_pos_weight=5.534 |
| XGB | Nested-CV OOF spec=0.70 fold 4 | True | scale_pos_weight=3.392 |
| XGB | Nested-CV OOF spec=0.70 fold 5 | False | scale_pos_weight=1.000 |

## 14. Hyperparameters (full-feature models)

`rev_tuning_fairness_table.csv` identical to v1 ✓ (AUC-tuned full-feature models); see v1 Section 14.

## 15. Permutation importance (Figure 3)

Threshold-free; identical to v1 ✓ — see v1 Section 15 (including the caution on q-values).

## 16. Figures — regenerated vs unchanged (byte comparison with v1)

| File | Figure | Expected | Result |
|---|---|---|---|
| rev_fig_ROC_PR_Calibration_Hists_XGB_left_CAT_right.png | Figure 2 (nested CV; dots, threshold lines, histograms) | changes | regenerated † |
| rev_sensitivity_vs_spec_targets_cat_xgb.png | Supplementary Figure 2 (specificity sweep, 0.70 added) | changes | regenerated † |
| rev_perm_importance_PR-AUCprimary_with_corr_XGB_left_CAT_right.png | Figure 3 permutation importance | unchanged | identical ✓ |
| rev_perm_importance_AUROC_sensitivity_with_corr_XGB_left_CAT_right.png | Permutation importance (ΔAUROC) | unchanged | identical ✓ |
| rev_fig_rfe_carryforward_xgb_cat_plus_grouped.png | Figure 5 SHAP grid (A–C) | unchanged | identical ✓ |
| rev_fig_staged_PR_left_ROC_right_xgb_cat.png | Staged PR/ROC curves | unchanged | identical ✓ |
| rev_fig_bar_AP_left_AUC_right_stages_xgb_cat.png | AP/AUC bars | unchanged | identical ✓ |
| rev_fig5_shap_beeswarm_model5_xgb_vs_cat_wide_annotated.png | M5 beeswarms (annotated) | unchanged | pixels differ (content check below) |
| rev_fig5_shap_beeswarm_model5_xgb_vs_cat_wide.png | M5 beeswarms | unchanged | pixels differ (content check below) |
| rev_model5_score_split_dists_xgb_cat.png | qXR split values | unchanged | identical ✓ |
| rev_train_test_distribution_horizontal.png | Supplementary Figure 1 (train/test outcome) | unchanged | identical ✓ |
| rev_fig_staged_PR_left_ROC_right_xgb_cat_bacteriologically_confirmed.png | PR/ROC, bacteriologically confirmed | unchanged | identical ✓ |
| rev_calibration_resampling_sensitivity.png | Calibration, resampling | unchanged | identical ✓ |

A byte difference in an 'unchanged' figure can come from non-deterministic rendering (e.g. SHAP beeswarm jitter or embedded metadata); the underlying values are verified identical through the CSV checks above.

## 17. Points for Methods / Limitations (additions for v2)

- State the operating point as specificity ≥ 0.70 chosen on training out-of-fold predictions to match the WHO triage TPP (sensitivity ≥ 90 %, specificity ≥ 70 %); report test specificity as achieved, not targeted.
- Section C gives the TPP verdict for M5 with exact CIs; with 35 test TB cases the sensitivity CI is wide (± ≈ 0.1), so 'meets TPP' should be stated for point estimates and qualified by the lower confidence bound.
- The nested-CV comparison (Table 2 / Figure 2) was re-tuned at specificity ≥ 0.70; its hyperparameters and class weighting per fold therefore differ from v1.
- All v1 limitations still apply (v1 Section 17).

## Verification summary

| Section | Check | OK |
|---|---|---|
| B | M1–M5 test AUROC (estimate and CI) identical to v1 | ✓ |
| B | M1–M5 test AP (estimate and CI) identical to v1 | ✓ |
| B | M1–M5 test Brier (estimate and CI) identical to v1 | ✓ |
| B | rev_rfe_flow_XGB.csv identical to v1 | ✓ |
| B | rev_rfe_flow_CatBoost.csv identical to v1 | ✓ |
| B | rev_topk_shap_ranking_XGB.csv identical to v1 | ✓ |
| B | rev_topk_shap_ranking_CAT.csv identical to v1 | ✓ |
| B | rev_probability_diagnostics.csv identical to v1 | ✓ |
| 2 | rev_table1_analytic_sample.csv identical to v1 | ✓ |
| 2 | rev_outcome_detail_counts.csv identical to v1 | ✓ |
| 2 | rev_hiv_counts.csv identical to v1 | ✓ |
| 2 | rev_age_group_counts.csv identical to v1 | ✓ |
| 3 | Specificity sweep 0.75–0.90 rows identical to v1 | ✓ |
| 4 | Table S2: AUROC, AP, Brier, Null Brier, Brier (recal.), n_feats, Trees, Test pred range identical to v1 | ✓ |
| 4 | Table S3: AUROC, AP, Brier, Null Brier, Brier (recal.), n_feats, Trees, Test pred range identical to v1 | ✓ |
| 5 | R1.4 (non-nested rows): AUROC, AP, Brier, Null Brier, Brier (recal.) identical to v1 | ✓ |
| 5 | rev_paired_bootstrap_auc_tests.csv identical to v1 | ✓ |
| 6 | Spec-target 0.80 rows reproduce v1 M5 test sens/spec | ✓ |
| 7 | rev_tbhistory_leakage_check.csv identical to v1 | ✓ |
| 7 | rev_tbhistory_original_composite_sensitivity.csv: AUROC, AP, Brier, Null Brier, Brier (recal.), Features identical to v1 | ✓ |
| 8 | Table S5: AUROC, AP, Brier, Null Brier, Brier (recal.), n_feats, Selected features identical to v1 | ✓ |
| 8 | rev_qft_tbhistory_crosstab_train.csv identical to v1 | ✓ |
| 8 | rev_qft_shap_drop_tbhistory_xgb_m5.csv identical to v1 | ✓ |
| 9 | Table S6: AUROC, AP, Brier, Null Brier, Brier (recal.), Trees, Calib intercept, Calib slope identical to v1 | ✓ |
| 10 | rev_age15plus_sensitivity.csv: AUROC, AP, Brier, Null Brier, Brier (recal.), Features identical to v1 | ✓ |
| 10 | rev_qft_excluded_sensitivity.csv: AUROC, AP, Brier, Null Brier, Brier (recal.), Features identical to v1 | ✓ |
| 10 | Table S11: n, TB cases, AUROC, AP, Brier, Null Brier identical to v1 | ✓ |
| 11 | rev_bac_confirmed_exclusions.csv identical to v1 | ✓ |
| 11 | rev_bac_confirmed_sensitivity.csv: AUROC, AP, Brier, Null Brier, Brier (recal.), Features identical to v1 | ✓ |
| 12 | rev_cxr_included_vs_excluded.csv identical to v1 | ✓ |
| 12 | Table S9 (all eligible): AUROC, AP, Brier, Null Brier, Brier (recal.), Features identical to v1 | ✓ |
| 12 | Table S9 (no-CXR): AUROC, AP, Brier, Null Brier, Brier (recal.), Features identical to v1 | ✓ |
| 13 | Final M1–M5 class weighting identical to v1 | ✓ |
| 14 | rev_tuning_fairness_table.csv identical to v1 | ✓ |
| 15 | rev_perm_importance_XGB_PR-AUC_primary.csv identical to v1 | ✓ |
| 15 | rev_perm_importance_CAT_PR-AUC_primary.csv identical to v1 | ✓ |
| 15 | rev_perm_importance_XGB_AUROC_sensitivity.csv identical to v1 | ✓ |
| 15 | rev_perm_importance_CAT_AUROC_sensitivity.csv identical to v1 | ✓ |

38 checks, 0 failed.

## Appendix. Reproducibility

- Notebook `Code/Practicum_REVISION.ipynb` (`TARGET_SPEC = 0.70`, tag `[REV-1.0]`); v1 notebook and outputs archived in `Output/v1_spec080/` (`Practicum_REVISION_spec080.ipynb`, `csv/`, `Images/`, `REVISION_RESULTS_SUMMARY.md`).
- Same environment and inputs as v1 (v1 Appendix B).
