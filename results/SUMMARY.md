# Results summary

_ClinicDB, EMCAD official split (train 489 / val 61 / test 62). 200 epochs. Tesla T4._
_All repeats use the same seed (42) and RUN_NUMBER 1, so the spread comes only from GPU nondeterminism. Values are mean ± std over 3 runs._

## Test metrics (best-val checkpoint)

| Model | Runs | Test Dice | Test IoU | Sensitivity | Precision | Best val Dice | Params | Train min |
|---|---|---|---|---|---|---|---|---|
| Baseline 0 — EMCAD-B0, original LGAG | 3 | **0.9486 ± 0.0008** | 0.9052 ± 0.0010 | 0.9556 ± 0.0030 | 0.9454 ± 0.0019 | 0.9348 ± 0.0034 | 3,916,795 | 111 |
| Baseline 1 — WACV hybrid, original gate (GAG) | 3 | **0.9158 ± 0.0048** | 0.8593 ± 0.0028 | 0.9248 ± 0.0081 | 0.9184 ± 0.0019 | 0.9015 ± 0.0025 | 408,702 | 83 |
| WACV hybrid, quantum gate on AG1 | 3 | **0.9123 ± 0.0147** | 0.8582 ± 0.0158 | 0.9279 ± 0.0107 | 0.9128 ± 0.0130 | 0.9041 ± 0.0046 | 408,878 | 92 |
| WACV hybrid, original gate, all 4 gates in fast-LR group | 3 | **0.9118 ± 0.0076** | 0.8577 ± 0.0090 | 0.9140 ± 0.0093 | 0.9196 ± 0.0095 | 0.9073 ± 0.0069 | 408,702 | 83 |

## Baseline 0 extra evaluations

| Evaluation | Test Dice | Test IoU |
|---|---|---|
| Best-val checkpoint, official eval | 0.9486 ± 0.0008 | 0.9052 ± 0.0010 |
| Best-val checkpoint, corrected (H,W) eval | 0.9498 ± 0.0005 | 0.9073 ± 0.0005 |
| Last epoch (200), official eval | 0.9491 ± 0.0024 | 0.9060 ± 0.0036 |

## Per-run test Dice

| Model | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| Baseline 0 — EMCAD-B0, original LGAG | 0.9495 | 0.9481 | 0.9483 |
| Baseline 1 — WACV hybrid, original gate (GAG) | 0.9109 | 0.9161 | 0.9205 |
| WACV hybrid, quantum gate on AG1 | 0.8954 | 0.9191 | 0.9225 |
| WACV hybrid, original gate, all 4 gates in fast-LR group | 0.9118 | 0.9194 | 0.9043 |

## Findings

- **EMCAD-B0 (Baseline 0) is stable and strong:** 0.9486 ± 0.0008 test Dice, about 3.3 points above the WACV hybrid on ClinicDB.
- **On the WACV hybrid, the quantum gate on AG1 matches the classical gate:** test Dice 0.9123 vs 0.9158 (−0.0035), best val Dice 0.9041 vs 0.9015 (+0.0026). Both gaps are well inside run-to-run noise. The quantum gate adds 176 parameters and about 10% training time.
- **WACV runs are noisy even with a fixed seed:** test Dice range up to 0.027 (quantum: 0.8954–0.9225). Single WACV runs cannot separate gates; comparisons need multi-run means and a paired per-image test.
- **Gate learning-rate group does not matter:** putting all 4 gates in the fast-LR group (0.9118) matches the AG1-only setting (0.9158).
- **200 epochs helps WACV:** 0.9158 mean vs 0.8993 for the earlier single 150-epoch run.
- **EMCAD eval-orientation quirk is negligible:** corrected (H,W) resize adds about +0.001 Dice.

## Not yet available

- EMCAD + quantum gate runs (NB2a–d): first attempt ran on CPU by mistake (~1,040 s/epoch, stopped at epoch 39 by Kaggle's 12 h limit). Relaunched on GPU.
- Per-image test CSVs for the paired Wilcoxon test.
- WACV same-param classical control (E3).
