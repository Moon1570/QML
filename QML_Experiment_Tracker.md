# QML Segmentation Study: Experiment Plan and Tracker

_Started 5 Oct 2026. Status: ⬜ not started · 🟨 running · ✅ done · ⏸ on hold · ❌ failed/dropped_

## Repo layout

```
QML/
├── QML_Experiment_Tracker.md     ← this file
├── Notebooks/                    ← one notebook per run
└── results/<slug>/               ← copy of each run's Kaggle outputs/<slug>/ folder
                                    (training_log.csv, per_image_test*.csv, results_record.json, config.json, PNGs)
```
Weights (`*.pth`) are gitignored. Kaggle versions keep the checkpoints.

---

## 1. Fixed protocol (same for every EMCAD notebook)

| Item | Value |
|---|---|
| Model | EMCADNet, encoder PVTv2-B0 (ImageNet-pretrained), official EMCAD `lib/` code |
| Dataset / split | CVC-ClinicDB, EMCAD official folders: train 489 / val 61 / test 62 |
| Recipe | 352 input, batch 8, **200 epochs**, AdamW lr 5e-4 wd 1e-4, cosine (T_max 200, eta_min 1e-6), grad clip 0.5 (value), multi-scale {0.75, 1, 1.25}, augmentation (rotate, flips) |
| Loss | structure loss on p4, p3, p2, p1 and their sum |
| Eval | official EMCAD test (head p1, min-max, pred ≥ 0.5, GT ≥ 0.2) |
| Selection | best val Dice → test once. Also report last-epoch test |
| Seed | 42 for all single runs. Repeats: RUN_NUMBER 1–5 |
| Flags | `MULTISCALE_MODE="official"`, `USE_AMP=False` (change for ALL notebooks or none) |
| Saved per run | `training_log.csv`, `per_image_test.csv`, `results_record.json`, curves, Kaggle version |

**Rules:** one notebook = one run. Only the component under test changes. **200 epochs is the default for every notebook** (EMCAD and WACV).
**Notebook structure (from Baseline 1 on):** single CONFIG cell, smoke test before training, resumable (`RESUME` + `KAGGLE_MODEL_INPUT_DIR`), per-epoch summary block. Baseline 0 stays as built.

---

## 2. Baselines

| ID | Notebook | Description | Status | Kaggle ver. | Test Dice (best) | Test Dice (last) | IoU | Params | Time | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| Baseline 0 | `NB1_EMCAD_B0_Baseline_ClinicDB` | Official EMCAD-B0, original LGAG | ✅ ×3 | | **0.9486 ± 0.0008** | 0.9491 ± 0.0024 | 0.9052 | 3,916,795 | 111 min | Corrected (H,W) eval 0.9498. `results/baseline0_emcad_b0/` |
| Baseline 1 | `litecm-unet-qml` (gag) | WACV hybrid, original gate, WACV recipe, 200 ep | ✅ ×3 | | **0.9158 ± 0.0048** | — | 0.8593 | 408,702 | 83 min | 150-ep run gave 0.8993 (reference only). `results/baseline1_wacv_gag_AG1/` |

---

## 3. Experiments

### Phase A: Quantum skip gating (LGAG). Compare against Baseline 0

All 3 LGAGs swapped (`lgag3` 22×22, `lgag2` 44×44, `lgag1` 88×88 @352). Gate swap uses a forked RNG, so init and data order match Baseline 0.

| ID | Notebook | GATE_MODE | What changes | Status | Kaggle ver. | Test Dice | IoU | Gate params | Time | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| NB2a | `NB2a_EMCAD_B0_QAG_Quantum_ClinicDB` | `replace` | LGAG fusion → 4-qubit entangling circuit | 🟨 | | | | | | 1st try ran on CPU (no GPU), stopped at ep 39 after 12 h. Relaunched on GPU |
| NB2b | `NB2b_EMCAD_B0_QAG_Classical_ClinicDB` | `classical` | circuit → same-param MLP (22 vs 24) | 🟨 | | | | | | Key control. Relaunched on GPU |
| NB2c | `NB2c_EMCAD_B0_NoGate_ClinicDB` | `none` | gates removed | 🟨 | | | | 0 | | Does gating matter? Relaunched on GPU |
| NB2d | `NB2d_EMCAD_B0_QAG_NoEntangle_ClinicDB` | `noent` | quantum circuit without CNOTs | ⬜ | | | | | | Isolates entanglement |

### Phase B: Quantum multi-scale fusion (MSDC). Compare against Baseline 0

| ID | Notebook | What changes | Status | Kaggle ver. | Test Dice | IoU | Added params | Time | Notes |
|---|---|---|---|---|---|---|---|---|---|
| NB3a | `NB3a_EMCAD_B0_QMSF_Quantum` | Sum of 1/3/5-kernel outputs → per-pixel quantum mixer (deepest stage) | ⬜ | | | | | | Best novelty bet |
| NB3b | `NB3b_EMCAD_B0_QMSF_Classical` | Same, classical same-param mixer | ⬜ | | | | | | Control |
| NB3c | `NB3c_EMCAD_B0_QMSF_NoEntangle` | Quantum mixer without CNOTs | ⬜ | | | | | | Control |

### Phase C: Quantum head aggregation. Compare against Baseline 0

| ID | Notebook | What changes | Status | Kaggle ver. | Test Dice | IoU | Added params | Time | Notes |
|---|---|---|---|---|---|---|---|---|---|
| NB4a | `NB4a_EMCAD_B0_QHA_Quantum` | p4+p3+p2+p1 → 4-qubit circuit (1 qubit per head), ¼ resolution | ⬜ | | | | | | |
| NB4b | `NB4b_EMCAD_B0_QHA_Classical` | Same, classical same-param fusion | ⬜ | | | | | | Control |

### Phase D: Consolidation (only for variants that beat their classical control)

| ID | What | Status | Notes |
|---|---|---|---|
| D1 | 5 runs (RUN_NUMBER 1–5) of Baseline 0 + winning variant + its classical control | ⬜ | EMCAD reports mean of 5 |
| D2 | Paired Wilcoxon on `per_image_test.csv`: quantum vs Baseline 0, quantum vs classical control | ⬜ | |
| D3 | Noise robustness: evaluate trained quantum gate under shot / depolarizing noise | ⬜ | Analysis cell, no retraining |
| D4 | Confirm winner on EMCAD-B2 | ⬜ | Slower; only once B0 result is clear |
| D5 | Cross-dataset: ColonDB, ETIS, Kvasir (EMCAD splits) | ⬜ | |
| D6 | Ablations: qubits, circuit depth, re-uploading on/off, which LGAGs | ⬜ | |

### WACV hybrid track (Baseline 1 host)

| ID | GATE_MODE | Status | Notes |
|---|---|---|---|
| W-E0 | gag, AG1 (= Baseline 1) | ✅ ×3 | 0.9158 ± 0.0048 |
| W-E0b | gag, all 4 gates in fast-LR group | ✅ ×3 | 0.9118 ± 0.0076 — LR group of gates does not matter. `results/wacv_gag_AG1-AG4_fastlr/` |
| W-E1 | none | ⏸ | |
| W-E2 | replace (quantum), AG1 | ✅ ×3 | 0.9123 ± 0.0147 (range 0.8954–0.9225). Ties Baseline 1. `results/wacv_qag_replace_AG1/` |
| W-E3 | classical | ⏸ | |
| W-E4 | augment | ⏸ | |

---

## 4. Decision rules

- **A variant "works"** if it beats its **classical same-param control** (not just Baseline 0), with paired Wilcoxon p < 0.05 over the mean of 5 runs.
- **Entanglement matters** only if the quantum version beats its no-CNOT version.
- **Single-run differences < ~0.01 Dice are noise** (val swings ±0.02 per epoch on 61 images).
- **Any protocol change** (AMP, multi-scale mode, eval, epochs) → rerun the baselines before comparing.

---

## 5. Open questions for Sir

| # | Question | Status | Answer |
|---|---|---|---|
| 1 | Recipe: WACV differential LR + EMCAD polyp settings? | ⬜ asked? | |
| 2 | Host: WACV main + EMCAD second, or EMCAD only? | ⬜ asked? | |
| 3 | EMCAD official code: multi-scale overwrite bug, keep or fix? | ⬜ | |
| 4 | EMCAD official eval resizes to (W,H): keep official or report corrected? | ⬜ | |
| 5 | Target venue and deadline | ⬜ | |
| 6 | Is "quantum matches classical with fewer params" an acceptable outcome? | ⬜ | |

---

## 6. Log

| Date | What happened |
|---|---|
| 24 Sep | WACV E0 (gag) done at 150 epochs: test Dice 0.8993, IoU 0.8417, 63 min |
| 5 Oct | Decided EMCAD-B0 study. NB1 built and CPU-tested (params 3,916,795 match official) |
| 10 Oct | 200 epochs made the default for all notebooks (EMCAD and WACV) |
| 11 Oct | Repo set up. Phase A notebooks (NB2a–d) built in the WACV structure and CPU-tested |
| 12 Oct | Baseline 0 ×3: 0.9486 ± 0.0008. Baseline 1 (WACV gag) ×3: 0.9158 ± 0.0048. WACV quantum AG1 ×3: 0.9123 ± 0.0147 (tie). NB2a–c first attempt ran on CPU; added a GPU check, relaunched. Summary in `results/SUMMARY.md` |
