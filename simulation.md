<h1 align="center">Simulation & Results Reproducibility — Guideline</h1>

---

> [!IMPORTANT]
> **The rule this document enforces:** every figure in the thesis and paper must be **regenerable by the next student, from the stored raw files, with one command**. A figure that cannot be regenerated from raw data does not pass the handover review (leaving-procedure item 5.7).

> [!NOTE]
> **Where this fits:** [research.md](./research.md) defines *what* each experiment scenario demonstrates (goal, constants, variables, figure axes). This document defines *how* the runs, raw files, and figures are organized so the chain below never breaks:
>
> **Scenario** (`research.md`) → **simulation run** → **raw files** (immutable) → **processing script** → **figure** (thesis/paper)

## Table of Contents

- [1. Directory Layout](#1-directory-layout)
- [2. Raw Files — the Ground Truth](#2-raw-files--the-ground-truth)
- [3. Dataset Registry](#3-dataset-registry)
- [4. Simulation Guide (`simulation/README.md`)](#4-simulation-guide-simulationreadmemd)
- [5. Figure Generation](#5-figure-generation)
- [6. Verification at Handover](#6-verification-at-handover)
- [7. Simulation Deliverables (video, live pipeline, AIML)](#7-simulation-deliverables-video-live-pipeline-aiml)

## 1. Directory Layout

Inside the main research repository:

```text
simulation/
├── scenarios/          # One config file per scenario (S1, S2, …): constants + swept variables
├── scripts/
│   ├── run/            # Scripts that execute a scenario and write raw files
│   └── plot/           # Scripts that turn raw/processed data into figures (one per figure)
├── raw/                # Raw outputs ≤ ~50 MB; larger files live in pCloud (Section 2)
├── processed/          # Intermediate data derived from raw/ — always regenerable, never hand-edited
├── figures/            # Generated figures — never hand-edited; named after the paper figures
├── Makefile            # `make figures` regenerates everything from raw data (Section 5)
└── README.md           # The Simulation Guide (Section 4)
```

## 2. Raw Files — the Ground Truth

- **Immutable.** Never edit a raw file; a corrected or repeated experiment produces a **new** file. Delete nothing until after graduation + handover review.
- **Naming:** `<scenario-id>_<yyyymmdd>_run<N>.<ext>` — e.g. `S1_20260615_run1.csv`. The scenario ID must match the [Experiment Scenarios table in research.md](./research.md#experiment-scenarios-and-results).
- **Metadata with every file** (sidecar `.meta.yaml`, CSV header comment, or InfluxDB tags): scenario ID, date, **7-digit commit hash of the code that produced it**, parameter values actually used, and the machine/testbed it ran on. Without the commit hash, a raw file cannot be traced back to the code version — it is unusable as evidence.
- **Storage:**
  - Small files (≤ ~50 MB total): commit them in `simulation/raw/`.
  - Large files: upload to the [Thesis pCloud folder](http://u.pc.cd/Dbd) under `Dataset/`, **mirroring the same file names**, and keep a pointer entry in the registry (Section 3). InfluxDB buckets are exported (`influx backup` or CSV export) into the same structure — a live database is not an archive.
- **Formats:** prefer open, tool-independent formats (CSV, JSON, pcap). Vendor/binary formats must ship with the conversion script in `scripts/run/`.

## 3. Dataset Registry

`simulation/README.md` keeps one registry table connecting every artifact in the chain. This is what the next student reads first:

| Scenario | Raw file(s) | Produced by (script @ commit) | Figure(s) | Used in |
| --- | --- | --- | --- | --- |
| S1: Low-traffic cell sleep | `S1_20260615_run{1..5}.csv` ([pCloud](pcloud-folder-link)) | `scripts/run/s1_sleep.sh` @ `a1b2c3d` | `figures/fig3_ues_vs_energy.pdf` | Paper Fig. 3, Thesis Fig. 4.2 |
| S2: Reactivation latency | `S2_20260618_run{1..3}.csv` | `scripts/run/s2_wakeup.sh` @ `a1b2c3d` | `figures/fig4_ramp_vs_latency.pdf` | Paper Fig. 4, Thesis Fig. 4.3 |

Every row must be complete before the oral exam: a paper figure with no registry row — or a raw file with no figure — is a red flag in the D-30 review.

## 4. Simulation Guide (`simulation/README.md`)

The Simulation Guide follows the same writing rules as [implementation.md](./implementation.md) — one command per block, expected output as `#` comments, only the critical path. Required sections:

1. **Environment** — exact software versions and testbed configuration (link to `implementation.md`; pipeline health verified per [oran-verification.md](./oran-verification.md)), plus **fixed random seeds**. A simulation with an undocumented seed is not reproducible.
2. **Per-scenario run steps** — for each scenario: the one command that runs it, expected runtime, and the raw file(s) it writes.
3. **Dataset registry** — the table from Section 3.
4. **Figure regeneration** — the one command from Section 5.
5. **Known issues** — flaky runs, hardware-dependent timing, configurations that always fail (same severity table as implementation.md).

## 5. Figure Generation

- **One script per figure** in `scripts/plot/`, deterministic: raw files in → figure out, with **no manual steps** (no spreadsheet edits, no GUI export, no hand-tuned annotations).
- **One command regenerates everything:**

  ```shell
  make figures        # or: python3 scripts/plot/plot_all.py

  # [S1] raw/S1_20260615_run*.csv -> processed/s1_agg.csv -> figures/fig3_ues_vs_energy.pdf
  # [S2] raw/S2_20260618_run*.csv -> processed/s2_agg.csv -> figures/fig4_ramp_vs_latency.pdf
  # done: 2 figures regenerated
  ```

  If raw files live in pCloud, the Makefile's first target prints the download instruction and verifies checksums before plotting.
- **Names match the paper:** `fig3_ues_vs_energy.pdf` is Fig. 3 in the paper. When figure numbers shift during revision, rename via the registry — never keep two figure sets.
- **Axes match the design:** X/Y labels and units must match the figure definition in the research.md Experiment Scenarios table; the plotting script is the single place where styling lives, so all figures stay consistent across thesis, paper, and slides.

## 6. Verification at Handover

The defense-slide experimental results are the **Minimum Reproducible Result (MRR)** — the minimum the incoming student must reproduce before the handover can be accepted ([Open Research Playbook — Knowledge Transfer](https://github.com/raycg/Open-Research-Playbook/blob/main/docs/10-knowledge-transfer.md)).

Performed by the **incoming student** on a clean machine, as part of handover review (leaving-procedure Section 5):

1. Clone the repo; fetch raw files from pCloud per the registry.
2. Run the one figure-regeneration command.
3. Diff each regenerated figure against the figure in the submitted paper/thesis — they must match (same data, same axes; cosmetic rendering differences are acceptable).
4. Run **one** scenario end-to-end from the Simulation Guide and confirm the fresh raw file is statistically consistent with the archived one.

Any step that fails goes into the Known Issues table with a workaround before the handover can be approved.

## 7. Simulation Deliverables (video, live pipeline, AIML)

Beyond the reproducible figures, every project archives these so results are independently verifiable:

- **Simulation video** — the full end-to-end run, uploaded to the [BMW Lab YouTube channel](https://www.youtube.com/@ntust.bmw-lab). Example: [OCUDU demo results](https://www.youtube.com/watch?v=ToUYOI_kL0Q).
- **Raw dataset** — all datasets used in the thesis/paper stored in the [Thesis pCloud folder](http://u.pc.cd/Dbd), following the Section 2 archival rules.
- **Live data pipeline (O-RAN projects)** — simulation data flows into **InfluxDB** and is processed by the rApp; **Grafana** visualizes it from InfluxDB. Verify both with the health checks in [oran-verification.md](./oran-verification.md).

### AIML projects — additional deliverables

- **Dataset specification:** source, format, size, and label description.
- **Trained model + training script:** model files plus the training script, with every hyperparameter documented.
- **Reproduce from scratch:** step-by-step instructions to retrain the model from the raw dataset.
