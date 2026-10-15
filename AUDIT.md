# WaferGuard Audit

## Scope and measured evaluation

Chronological split: 60% training / 20% validation / 20% test.
The test set is not resampled and retains its observed prevalence.

- Test rows: 314
- Test defects: 17
- Test defect prevalence: 5.41%
- Validation-selected threshold: 0.000358
- Precision: 0.0617
- Recall: 0.2941
- F1: 0.1020
- ROC-AUC: 0.5260
- Average precision: 0.0681
- Confusion matrix counts: TN=221, FP=76, FN=12, TP=5

## Findings ranked by severity

1. **Critical — evaluation leakage in the original notebook.** The original code fitted mean imputation and standardisation on all rows, ran SMOTE on all rows, and only then created a random train/test split. Consequently, synthetic or related information from would-be test rows could influence training, and the test distribution was artificially balanced. The repaired pipeline splits chronologically first, fits preprocessing on training rows only, does not use SMOTE, and evaluates on original test wafers at natural prevalence.

2. **Critical — invalid reconstruction-loss broadcasting.** The original loader created inputs with shape `[batch, 1, features]`, while the decoder returned `[batch, features]`. PyTorch broadcasting makes their subtraction shape `[batch, batch, features]`, coupling each reconstruction to every sample in the batch. The repaired loader uses `[batch, features]`; explicit assertions verify reconstruction and input shapes match. The reproducible shape check in this notebook demonstrates the original broadcast shape.

3. **High — invariant blindness to the PS-L tool regime.** The original pipeline treated the legacy sentinel and firmware codes as ordinary continuous sensor values, without representing the station's categorical firmware state. This can make the learned representation impose an artificial numeric ordering on categories and obscure tool-regime-specific behaviour. The repaired preprocessing marks the two overloaded sensor values missing for legacy rows and appends three binary firmware indicators. The notebook reports counts for each category so this treatment is auditable.

4. **High — threshold and class balance did not reflect deployment evaluation.** The original fixed 0.5 threshold was evaluated on a SMOTE-balanced test set. The repaired threshold is selected using validation data only, and test metrics are calculated on the untouched chronological test period at its observed defect prevalence.

## Residual risks

- The dataset is small and contains few defects in the later validation/test periods; point estimates and the validation-selected threshold may be unstable.
- Chronological holdout is more consistent with forward deployment than a random split, but one historical period does not establish robustness to all future drift.
- The PS-L representation is a task-brief-driven repair for invariant blindness to the categorical tool regime; it should be validated against station metadata and process-integration expectations before production use.
- This is a challenge-level audit, not a substitute for prospective monitoring, calibration, drift alerts, and production acceptance testing.

## Release recommendation

**NO-GO for production pending further validation.** Use the measured test metrics above as the honest challenge evaluation. The corrected pipeline removes identified leakage and tensor-broadcasting defects, but the limited number of defects and single temporal holdout are insufficient to certify production readiness.
