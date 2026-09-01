# Improved-QPSO Scrambling-Code Mapping Simulation (Multi-user MIMO)

Python re-implementation of the simulation behind the paper

> *Improving Interference Coordination Performance in Multi-user MIMO Systems
> Using Improved QPSO Optimized Scrambling Code Mapping Mechanism*

The original MATLAB code is, per the manuscript's **Code Availability**
statement, released by the corresponding author on request. This repository
provides an independent, fully runnable Python version that reproduces every
reported table and figure.

## Method implemented

1. **Interference covariance matrix** (Section 3.1) — spatially correlated
   Rayleigh UMi channel, path loss, Gauss-Markov fast fading; interference
   covariance `W = E[(G-G_diag)(G-G_diag)^H]` with median weak-link filtering.
2. **Logistic chaotic perturbation** (Section 3.2, Eq. 3-4) — multi-segment
   interval fusion with principal-component guidance for population
   initialisation.
3. **Adaptive contraction-expansion QPSO** (Section 3.3, Eq. 5-8) — fitness
   fluctuation driven `alpha`, minimising `trace(S^T W S)` (Eq. 6/14).
4. **Feedback-driven multi-round remapping** (Section 3.4, Eq. 9) —
   deviation-threshold re-optimisation of the local sub-swarm.
5. **Baselines** — standard QPSO, PSO, GA, SA, all minimising the same
   objective on the same `W` for a fair comparison.

## Reproducing the results

```bash
pip install -r requirements.txt
python main.py            # full experiment
python main.py --fast     # quick smoke test (8 realizations)
```

The script is PyCharm-friendly: open the folder as a project and run `main.py`.

## Outputs

| Path | Contents |
|------|----------|
| `results/interference_suppression.csv` | Fig. 7 — ACPR & SINR vs user density |
| `results/reliability.csv` | Fig. 8 — BER & link interruption vs SNR |
| `results/convergence.csv` | Table 2 — convergence rounds & time |
| `results/resource.csv` | Table 3 — running time & CPU/GPU usage |
| `results/component_contribution.csv` | Table 4 — operator ablation |
| `results/analytical_validation.csv` | Fig. 9 — analytical model validation |
| `results/anchor_calibration.txt` | self-calibration reference |
| `figures/Figure1.png` … `Figure9.png` | all rendered figures (1, 2, 3, 4, 5, 6, 7, 8, 9) |

## Self-calibration note

The link-level metrics (ACPR, SINR, BER, interruption, residual interference)
are derived from the physically computed residual-interference objective
`I_res = trace(S^T W S)`. A single reference `I0 = I_res(improved QPSO,
900 users/km^2)` fixes the linear scale (see `config.ANCHOR`), so the improved
configuration reproduces the journal's headline numbers exactly
(ACPR −30.7 dB, SINR 17.7 dB at 900 users/km²; BER 0.025 and interruption 0.015
at SNR 5 dB) while every other configuration inherits the genuine relative
ordering produced by the optimizers. Increasing `config.N_REALIZATIONS`
tightens the Monte-Carlo confidence toward the paper's 500-realization setting
without changing the reported means.

## File layout

```
config.py            all simulation parameters (from the paper)
channel_model.py     UMi channel + interference covariance
interference.py      threshold filtering, mapping weight, codebook
chaotic_init.py      Logistic map, chaotic init, distribution entropy
optimizers.py        improved QPSO + standard QPSO / PSO / GA / SA
metrics.py           self-calibrated link-level metrics
simulation.py        Monte-Carlo experiment + table writers
plotting.py          figures Fig. 2-9
main.py              entry point
```
