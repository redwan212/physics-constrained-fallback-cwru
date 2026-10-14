# A Physics-Constrained Fallback Framework for Safe Degraded Operation Under Unseen Industrial Faults

**CSE 4889: Machine Learning, Section F, Summer 2026. United International University, Department of CSE**
Md. Redwan Hossain Khan (0112330607) · Shajidul Islam Shajid (0112330905) · Fisul Islam Riad (0112330646)
Course Teacher: Kazi Fahim Tahmid (KFT)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/redwan212/physics-constrained-fallback-cwru/blob/main/ML4.ipynb)

## Summary

A neural fault detector trained on known bearing faults can give a confident but wrong diagnosis when it meets a fault it has never seen, and the machine keeps running at full speed. Stopping whenever the model is unsure avoids that risk, but it costs production. This project asks whether the system can **fail safely** when it does not know what is wrong. A 1D CNN is trained on seven known CWRU conditions at 1 hp, and two fault classes (IR021, B021) are held out as unseen. A logit-margin uncertainty score, averaged over five windows, decides whether a diagnosis is trusted. Machine capacity is set separately by a **physics envelope**: a continuous function of vibration velocity referenced to ISO 10816-1 Class I zones, which does not depend on the network. With the network alone, 100.0% of unseen-fault decision steps are silent misdiagnoses. Hard-stopping on uncertainty causes 61.5% downtime at 1 hp. The final design gives 0.0% silent failures and 0.0% downtime in the main run.

## Repository contents

| Path | What it is |
|---|---|
| `ML4.ipynb` | The complete notebook, **with all outputs saved**. You can read the results without rerunning it. |
| `Technical_Report.docx` | The final technical report. |
| `Technical_Report.Rmd` | R Markdown version of the report. Its Appendix B tables are read directly from `artifacts/*.csv` when knitted. |
| `figures/` | Every figure in the report, saved by the notebook (`A*` = Phase A, `B*` = Phase B, `Bplus_*` = Phase B+, `C*` = Phase C, `R_*` = seed robustness, `D*` = Phase D). |
| `artifacts/` | Saved results: CSV/JSON tables, trained models (`*.keras`), label mapping, normalisation parameters, the Phase A dataset (`phaseA_dataset.npz`). |
| `Documentation on  working phase.md` | Working notes on decisions and results for each phase. |
| `requirements.txt` | Python packages used. |

## Notebook map (report ↔ notebook)

| Report section | Notebook section in `ML4.ipynb` | Main outputs |
|---|---|---|
| §3 Dataset and preprocessing | Download cell + **Phase A** (Steps 1–14, 13b overlap check) | `raw_signal_stats.csv`, `normalization_params.json`, `phaseA_dataset.npz`, `A03`, `A09`, `A11` figures |
| §4.2–4.3, §5.1–5.2 CNN detector and uncertainty | **Phase B** (Steps 15–27, uncertainty gate) | `fault_cnn_1d.keras`, `label_mapping.json`, `uncertainty_thresholds.json`, `uncertainty_summary.csv`, `B20`, `B21`, `B26`, `B27` |
| §5.3, §5.5 Logit margin, multi-window aggregation, 2 hp control | **Phase B+** (B+.1–B+.8) | `uncertainty_thresholds_extended.json`, `multiwindow_aggregation.csv`, `extended_score_comparison.csv`, `Bplus_*` |
| §4.4–4.5, §5.6–5.8 Physics envelope, policies, fault injection | **Phase C** (Steps 28–31c) | `phaseC_results.json`, `phaseC_policy_comparison_{1hp,2hp}.csv`, `phaseC_velocity_rms_*.csv`, `phaseC_sensitivity_1hp.csv`, `phaseC_fault_injection_*.csv`, `C*` figures |
| §5.4 Seed variation and ensemble | **Robustness** (multi-seed repetition, deep ensemble) | `multiseed_*.csv/json`, `multiseed_model_seed*.keras`, `ensemble_results.json`, `ensemble_vs_single.csv`, `R_*` |
| §5.7, §6–7 Final tables, confusion matrices, limitations | **Phase D** (D1–D9) | `phaseD_final_policy_table.csv`, `phaseD_proposal_metrics.csv`, `phaseD_claims_traceability.csv`, `phaseD_confusion_*.csv`, `phaseD_limitations.md`, `D4`, `D5` |

**Traceability:** every number in the report is read from a file in `artifacts/`. `artifacts/phaseD_claims_traceability.csv` lists each claim with its source file, row or key, and column, and cell D9 re-reads and checks every entry.

## Dataset

Case Western Reserve University (CWRU) Bearing Data Center: <https://engineering.case.edu/bearingdatacenter>. We use the drive-end accelerometer at 12 kHz. The bearing is an SKF 6205-2RS.

The raw `.mat` files are **not** included in this repository. The first cell of the notebook downloads them from `https://engineering.case.edu/sites/default/files/<number>.mat` into `bearing_dataset/`. The normal-baseline files (98, 99) are 48 kHz and are resampled to 12 kHz.

| Class | Role | 1 hp file | 2 hp file |
|---|---|---|---|
| Normal | known | 98 | 99 |
| IR007 | known | 106 | 107 |
| IR014 | known | 170 | 171 |
| OR007@6 | known | 131 | 132 |
| OR014@6 | known | 198 | 199 |
| B007 | known | 119 | 120 |
| B014 | known | 186 | 187 |
| IR021 | **unseen** | 210 | 211 |
| B021 | **unseen** | 223 | 224 |

1 hp is used for training and testing. 2 hp is used only for evaluation, as a load the network never saw.

## How to run

**Google Colab (easiest):** click the badge above, then *Runtime → Run all*. A GPU runtime is faster because the robustness section retrains the network five times.

**Locally:**

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook ML4.ipynb
```

The last cell (D8) zips the outputs and uses `google.colab.files` to download them. Skip that cell when running outside Colab. Results were produced with TensorFlow 2.20.0 and seed 42. The other seeds are 7, 123, 2024 and 31337.

**Knitting the R Markdown report:** open `Technical_Report.Rmd` in RStudio from the repository root and click *Knit*. It needs the `rmarkdown` and `knitr` packages. Set the repository URL once in the YAML `params: repo_url`.

## Key results (main run, seed 42)

| Policy | Silent failure (unseen 1 hp) | Downtime 1 hp | Downtime 2 hp |
|---|---|---|---|
| NN only (no fallback) | 100.0% | 0.0% | 0.0% |
| Hard stop on uncertainty | 0.0% | 61.5% | 42.0% |
| Graceful (Simplex switch) | 0.0% | 0.0% | 0.0% |
| **Graceful + physics envelope (final)** | **0.0%** | **0.0%** | **0.0%** |

Source: `artifacts/phaseD_final_policy_table.csv`. Across five seeds, unseen-fault detection is 87.9 ± 12.7% (worst seed 65.7%). The five-network ensemble reaches 96.8%.

**Limitations.** The uncertainty score is also sensitive to load change: 27.9% of correctly classified 2 hp windows were flagged. All real recordings stayed in ISO Zone A, so derating and stopping are shown only by fault injection. All results come from one dataset and one test rig. See `artifacts/phaseD_limitations.md`.
