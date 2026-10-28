# Data for: open reconstruction and pre-registered recalibration of a fuzzy-logic fracture-healing model

Version 1.0.0 (2027-01-05). Repository: https://github.com/MedEngLab/Healing_Latency_Calibration

This repository contains the data that support the findings of the associated article: the simulation output of every stage of the analysis, the in vivo and digitized reference data used for calibration and validation, and the fuzzy-logic rule base of each model design in machine-readable form. The simulation code is not part of this release.

The study reconstructs the fuzzy-logic fracture-healing model of Simon et al. (2011) as calibrated by Steiner et al. (2013) for the ovine metatarsal osteotomy, verifies its numerics, recalibrates it under a pre-registered protocol with leave-one-out cross-validation (LOO), and tests a registered structural extension (onset delay of the healing response, tau, and maximal callus index, mCI). The protocol and every dated amendment are in `docs/PROTOCOL.md`; the amendment log is in `docs/AMENDMENT_LOG.md`.

## Contents

| Path | Content |
|---|---|
| `rulebase/<design>.fis` | Rule base of each design in MATLAB Fuzzy Logic Toolbox text format (Mamdani; min, max, centroid) |
| `rulebase/<design>.json` | The same rule base as JSON: trapezoidal membership functions of all inputs and outputs, rules, material parameters, onset delay and mCI |
| `rulebase/designs.json` | Parameter values of all designs: `ST13_design706`, `v1_recalibrated`, `v2_cart50`, `v2_cart30`, `v3_fit_all_six`, `v3_calibration_AB`, `v3_secondary_a_14par` |
| `rulebase/published_set_*.fis` | Rule base with the published parameter sets of Simon et al. (2011) and Steiner et al. (2013) |
| `rulebase/model_constants.json` | Fixator law, load, cortex material, geometry, numerical settings, objective |
| `data/invivo_ifm.csv` | In vivo interfragmentary movement (IFM), cases A, B (calibration) and V1-V4 (validation): mean and SD per time point |
| `data/st13_simulated_ifm.csv`, `data/trace_*.csv` | Simulated IFM curves published by Steiner et al. (2013, Fig. 4), digitized |
| `results/` | Stage 1: recalibration with the original numerics (v1), validation, LOO |
| `results_v2/` | Stages 2-3: mesh and sub-step convergence, recalibration (two cartilage outputs), time-scale scan, LOO |
| `results_v3/` | Stages 4-5: structural screens, registered extension (exhaustive tau x mCI grid, LOO), secondary analyses, registration and freeze records |
| `figure_source_data/collected.json` | Every number plotted in the figures of the article |
| `figure_source_data/amendment_log.csv` | Amendment log (machine-readable) |
| `docs/` | Protocol with all amendments, amendment log, rule-base notes (sources, reading choices, verification) |
| `checksums_frozen.sha256` | SHA-256 of the frozen files; identical to the freeze records in `results/FREEZE.md` and `results_v3/FREEZE_v2_v3.md` |
| `MANIFEST.sha256` | SHA-256 of every file in this release |

### Main result files

| File | Content |
|---|---|
| `*/calibration_summary.json` | Calibration settings, best parameter set, objective history, parameters at bounds |
| `*/evaluations.csv` | Every design evaluated during calibration: parameters, errors, IFM at the in vivo days |
| `*/validation_summary.json` | Criteria K1-K4, errors and IFM at the in vivo days for the calibrated design and ST13 design 706 |
| `*/loo_summary.json`, `*/simulations.csv` | LOO folds (estimates, held-out errors) and the cache of all simulated (design, case) pairs |
| `*/reevaluation_M4_32.json`, `*/loo_reevaluation_M4_32.json` | Re-evaluation at the reporting fidelity (mesh M4, 32 sub-steps), daily IFM |
| `results_v2/mesh/summary.json` | Front-speed calibration and convergence per mesh (M1-M5) and sub-step count, daily IFM per run |
| `results_v2/timescale_scan.json` | Errors and IFM per front speed and case |
| `results_v3/screen*.json`, `delay_shift.json` | Structural screens with their decision outcomes |
| `results_v3/primary_M4_32/v3_grid.npz` | Registered extension: error `E[mCI, tau, case]`, IFM at the in vivo days `U9[mCI, tau, case, day]`, daily IFM without delay `ifm_daily[mCI, case, day]` |
| `results_v3/*/v3_summary.json` | Registered criteria, calibration on A + B, LOO folds |

Units: IFM in mm; time in days; moduli in MPa; strains as absolute values (0.01 = 1 %); tissue fractions and perfusion in [0, 1]. Case error `e` = mean over the nine in vivo time points of ((mean - simulated) / max(SD, 0.02 mm))^2.

## Provenance of the reference data

- In vivo IFM (cases A, B, V1-V4) and the published simulated curves were digitized from Steiner et al. (2013), Figs. 2 and 4, which report experiments of Claes and co-workers on the ovine metatarsus. The first in vivo point lies a few days after surgery and is assigned to day 1. Figures of the article are not redistributed.
- The rule base follows Simon et al. (2011), Table 2, with membership functions from Simon et al. (2011), Figs. 4-5, and the parameters of Steiner et al. (2013); reading choices, assumptions and the verification of the rule base are listed in `docs/RULEBASE_NOTES.md`.

## Integrity

```
shasum -a 256 -c checksums_frozen.sha256
shasum -a 256 -c MANIFEST.sha256
```

## References

- Simon U, Augat P, Utz M, Claes L (2011). A numerical model of the fracture healing process that describes tissue development and revascularisation. Comput Methods Biomech Biomed Engin 14:79-93. doi:10.1080/10255842.2010.499865
- Steiner M, Claes L, Ignatius A, Niemeyer F, Simon U, Wehner T (2013). Prediction of fracture healing under axial loading, shear loading and bending is possible using distortional and dilatational strains as determining mechanical stimuli. J R Soc Interface 10:20130389.
- Claes LE, Heigele CA (1999). Magnitudes of local stress and strain along bony surfaces predict the course and type of fracture healing. J Biomech 32:255-266.

## License and citation

Data released under the Creative Commons Attribution 4.0 International license (CC BY 4.0; `LICENSE`). Cite the Zenodo record of this release and the associated article; see `CITATION.cff`.
