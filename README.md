# WaferGuard — GDG Tech Wildcard AI/ML Submission

## Files
- `wafer_guard_fixed_completed.ipynb`: repaired end-to-end notebook.
- `AUDIT.md`: audit findings and measured results from the executed notebook.

## Run
1. Upload/open the notebook in Google Colab.
2. Upload `secom_telemetry_modified.csv` into the Colab session.
3. Run all cells from top to bottom.
4. Review the measured test metrics and generated `AUDIT.md`.

The notebook does not use the evaluator-only file. It uses a chronological split, training-only preprocessing, explicit PS-L firmware indicators, a corrected reconstruction loss, and a threshold selected on validation data.

## Important
The measured test results are poor (F1 around 0.10 on this run), so the audit recommends **NO-GO for production**. Do not claim the original reported F1 score as valid. Re-run the notebook in the submission environment and ensure the generated metrics match the final run before submitting.
