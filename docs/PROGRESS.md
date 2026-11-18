# Progress: Lab 9 - Evaluation & Failure Analysis

- **Status**: M1 Completed, Moving to M2
- **Branch**: starter/d09
- **Done**:
  - M0 setup: clean Day 8 integration, initial evalset build.
  - M1: Defined 9 Failure Modes in `docs/failure_taxonomy.md` (all 5 anchors matched).
  - M1: Mapped `failure_modes` for 14 claims in `src/lab9/evalset.py`.
  - M1 verification: `python -m lab9.evalset --build --check` passed with 0 problems.
  - M1 tests: `pytest tests/test_lab9_evalset.py` passed 3/3.
- **In progress**:
  - M2: Implementing stats.py (confusion, precision_recall, kappa, rogan_gladen).
- **Next**:
  - Run judge calibration on `judge_tone_public.csv` & `judge_helpful_public.csv`.
  - Fill `docs/judge_audit.md` (≥5 disagreements audit).
  - M3: Trajectory graders & FM-08 custom metric.
- **Blockers**: None.
