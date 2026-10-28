# Option B: recalibration of the reconstructed rule base (pre-registered protocol)

Version 1, fixed 2026-10-24 before any full calibration run. Changes after the first full run are reported as deviations.

## 1. Rationale

The reconstructed Ulm rule base (Simon 2011 structure, Steiner 2013 parameters) passes all static checks (48 tests; skfuzzy agreement within quadrature error). It does not reproduce Steiner 2013 (ST13) dynamically:

- With ST13 design no. 706 as published, case B does not bridge by day 56 (IFM about 0.9 mm).
- The objective of this protocol is G = e_A + e_B = 11.5 (−3 % destructive threshold), dominated by case B (e_B = 10.8).

The likely causes are the unpublished chondrogenesis modification in ST13 and differences in implementation, for example the mesh, the 2D axisymmetric idealization and the fixator model. Ulm did not respond to the request for the rule base. The parameters are therefore recalibrated within ST13's bounds, using ST13's procedure and ST13's in vivo data. The recalibrated set is then tested on cases that are not used in the calibration.

## 2. Model and data (fixed)

- **Rule base:** Simon 2011 Table 2, 8 rules, with chondrogenesis as tabulated. No no-change rules. MATLAB Mamdani semantics.
- **Finite element model:**
  - axisymmetric CST;
  - callus outline cos²;
  - element area ≤ 0.16 mm² in the callus and ≤ 0.3 mm² in the cortex;
  - cortex E = 15750 MPa, ν = 0.325;
  - Steiner rule of mixtures with exponents for bone and cartilage.
- **Rate scaling:** A_ref = 1.2725 mm², the value matched to the Simon 2011 front speed at this mesh (out/front_calibration.json). ST13 Table 1 does not list rate parameters, so A_ref is not calibrated.
- **Load and fixator:** 840 N, with the fixator clearance set from the initial IFM of each case.
- **Data:** in vivo IFM means and SDs digitized from ST13 Fig. 2 (cases A and B) and Fig. 4 (V1–V4), at days 1, 7, …, 56 (calibration/data.py). The SD floor is 0.02 mm.

| Case | Gap (mm) | Initial IFM (mm) | Role |
|---|---|---|---|
| A | 2.25 | 0.25 | calibration |
| B | 3.3 | 1.30 | calibration |
| V1 | 1.44 | 0.32 | validation |
| V2 | 6.4 | 0.46 | validation |
| V3 | 6.97 | 1.43 | validation |
| V4 | 2.7 | 0.53 | validation |

## 3. Calibration procedure (ST13 Sect. 2.3)

**Parameters:** 12 parameters with ST13 Table 1 bounds:

- E and ν of bone, cartilage and connective tissue;
- the perfusion, bone and cartilage decrease thresholds and the cartilage increase threshold;
- the bone and cartilage mixture exponents.

All are sampled in normalized [0, 1] coordinates.

**Objective:** e_c is the mean of ((IFM_vivo − IFM_sim)/SD)² over the 9 time points of case c. The objective is G = e_A + e_B.

**Search:**

1. Latin hypercube sample of 150 designs, seed 20261024.
2. Particle swarm optimization initialized with the 32 best designs:
   - w = 0.729, c1 = c2 = 1.49445;
   - |v| ≤ 0.2;
   - at most 35 generations;
   - stop after 6 generations without a relative improvement of more than 0.1 %.
3. Destructive dilatational threshold −5 % in step 1. The best design is then re-evaluated at −3 % (ST13 step 2), and the threshold with the lower G is kept.

**Reference:** ST13 design 706 is evaluated in the same run at both thresholds.

## 4. Acceptance criteria (decided before the run)

| Criterion | Requirement | Purpose |
|---|---|---|
| K1 | (e_A + e_B)/2 ≤ 1.0, and each of e_A, e_B ≤ 1.5 | Calibration fit within about 1 SD on average |
| K2 | IFM of case B at day 56 ≤ 0.25 mm | Bridging of the large-IFM case (the failure mode) |
| K3 | Mean e over V1–V4 ≤ 2.0 | Out-of-sample fit |
| K4 | At most 3 of 12 parameters within 1 % of a bound | Calibration not driven against the search box |

## 5. Decisions

- **All criteria pass:** the recalibrated set becomes the Paper 2 baseline. It is reported as "Ulm-structured rule base, recalibrated following Steiner et al. 2013", with both parameter sets tabulated.
- **K1 or K2 fails:** the reconstruction cannot reproduce ST13 within the ST13 bounds. In that case:
  - ST13 is not claimed as a benchmark;
  - Paper 2 reports the failure;
  - any extension of the parameter space (for example chondrogenesis variants or A_ref) is labeled exploratory and validated only on V1–V4.
- **K3 fails while K1 and K2 pass:** the set is overfitted to A and B. This is reported as a limitation, and the dynamization comparisons in Paper 2 are restricted to the gap and IFM range of A and B.
- **K4 fails:** the parameters at their bounds are reported, together with a one-at-a-time sensitivity of G to each of them.

## 6. Outputs

All outputs go to results/:

- evaluations.csv: every design, its errors and its IFM at the in vivo days;
- calibration_summary.json;
- validation_summary.json: K1–K4 and the errors for all six cases, for both designs;
- validation_histories.npz;
- ifm_calibration_validation.png.

## 8. Addendum v1.1 (2026-10-25): LOO-CV and the revascularization-rate parameter

The addendum was written after the v1 results were known, so its hypothesis is post hoc in origin. Its criteria are fixed before the LOO runs.

**Observation behind it (v1 results):**

- In the low-IFM cases (A, V1, V2, V4), the simulated IFM falls below the in vivo mean between weeks 1 and 5. The model heals too fast.
- The high-IFM cases (B, V3) fit.
- ST13 reports the same trend for its own corroboration: "a trend towards too fast healing predictions"; "the gradient of IFM decrease for load cases V2 and V4 is larger than mean in vivo courses".
- The S11 sensitivity analysis (Sect. 2.5, Table 3) concluded that healing speed in the stable case depends mainly on the revascularization rate, whereas the less stable case is limited by the mechanical environment.
- In v1 the rate is fixed (A_ref set to the 0.22 mm/day maximum front speed of S11), so the calibration can only act through moduli and thresholds. Three stiffness parameters ended at or near their bounds (E_bone, exp_bone, E_conn).

**Hypothesis H-rate:** the too-fast healing of low-IFM cases reflects a mis-specified revascularization rate, not the strain thresholds.

**Variants:**

- baseline: the 12 parameters of Sect. 3.
- rate: the 12 parameters plus the S11 revascularization factor mu. Mu scales the predicted perfusion change, with bounds 0.5 to 2.0, the range tested in S11 Table 3 (maximum revascularization rate 0.15 to 0.34 mm/day in S11).

**Procedure:**

- Leave-one-out over A, B and V1 to V4. Each case is held out once: the parameters are calibrated on the other five and used to predict the held-out case.
- Fold "all" calibrates on all six cases and gives the parameter set for later use.
- The LHS, PSO settings, SD floor (0.02 mm) and objective are as in Sect. 3, with G equal to the sum of e_c over the training cases.
- The destructive threshold is fixed at −5 %, the value selected in v1.
- ST13 design 706 (mu = 1) is evaluated as the reference.

**Criteria:**

| Criterion | Requirement |
|---|---|
| L1 (generalization, each variant) | CV error, the mean held-out e over the 6 folds, ≤ 2.0 |
| P1 (rate vs baseline; kill criterion for H-rate) | CV error lower by ≥ 20 % and rate better in ≥ 4 of 6 folds |
| P2 | The summed held-out improvement in A, V1, V2, V4 exceeds that in B, V3 |
| P3 | In fold "all", mu < 1 and not at a bound |
| P4 | In fold "all", E_bone and exp_bone are no longer at a bound |

**Decisions:**

- If P1 fails, the rate parameter is not supported by these data and is not added to the Paper 2 model.
- If P1 holds, the rate variant becomes the Paper 2 model, with the parameters of fold "all".
- P2 to P4 test whether the improvement has the mechanistic signature that H-rate predicts. If P1 holds but P2 to P4 fail, the improvement is reported as a fitting gain, not as support for H-rate.
- Any parameter whose normalized range across the 6 folds exceeds 0.5 is reported as not identifiable from these data.

**Secondary analyses (reporting only):** held-out errors recomputed with an SD floor of 0.05 mm, and recomputed without day 1.

**Known limits:**

- Six folds give low power. Each held-out error comes from a single case, and the cases share one laboratory and one fixator system.
- ST13 also calibrated on torsion and shear cases (C and D), which the axisymmetric model cannot represent.

## 7. Run

From the rulebase_v1 folder:

```
pip3 install -r calibration/requirements.txt
python3 -m calibration.run_calibration --smoke     # about 15 s; tests the setup
python3 -m calibration.run_calibration --workers 8 # full run; rerun the same command to resume
python3 -m calibration.validate --workers 8
```

## 9. Protocol version 2 (2026-10-25): numerics, mesh convergence, cartilage output

The v1 analyses (Sects. 3-8; frozen 2026-10-25, results/FREEZE.md) remain on record as version 1. Post-freeze diagnostics (diagnostics/early_stiffening.py) showed (i) strong mesh dependence of the healing time course (V2, design 706: e = 26.8 / 19.2 / 6.3 on coarse / standard / fine meshes) and (ii) that early callus stiffening is driven by fast chondrogenesis under the reconstructed ST13 cartilage output (+/-50 % universe). Version 2 is fixed here before any v2 run.

**9.1 Numerics.**
- Defuzzification: exact centroid of the piecewise-linear aggregated output (replaces the 1001-point discrete centroid; tested against a 20001-point grid, tolerance 1e-4).
- Sub-stepping: the daily fuzzy update is split into n sub-steps (mechanics held for the day; neighbor inputs updated each sub-step; changes divided by n), removing the limit of about one element of growth per day.
- A_ref is recalibrated for every mesh and n so that the bone front speed on a strip with the same mesher settings is 0.22 mm/day (S11).

**9.2 Mesh convergence (criterion).**
- Meshes M1-M4: maximum callus element area 0.32, 0.16, 0.08, 0.04 mm^2 (cortex twice the callus value).
- Sub-steps n in {1, 4, 8, 16}.
- Reference designs: ST13 design 706 with each cartilage variant (9.3), all six cases, 56 days.
- Criterion: for a mesh pair (Mk, Mk+1) at the same n, max |IFM(Mk) - IFM(Mk+1)| over the six cases and nine in vivo days <= 0.02 mm. The production setting is the coarsest Mk and smallest n satisfying the criterion with Mk+1 and with n doubled. If no setting satisfies it, the finest feasible setting is used and the residual difference is reported.

**9.3 Cartilage output (structural alternative).**
- cart50: ST13 reconstruction (cartilage output universe +/-50 %, thresholds calibrated within ST13 bounds -47.5 to -2.5 % and +2.5 to +47.5 %).
- cart30: S11 universe +/-30 % with the same calibrated generic sets; threshold bounds scaled to the universe (-28.5 to -1.5 % and +1.5 to +28.5 %; same fractions of the universe as ST13).
- Both variants are calibrated (A, B; Sect. 3 procedure) and cross-validated (Sect. 8 procedure, baseline parameters, no rate parameter).

**9.4 Decision rules.**
- K1-K4 (Sect. 4) and L1 (Sect. 8) apply unchanged to each variant.
- Variant selection for Paper 2b: lower CV error; a difference below 20 % is reported as no preference, and cart50 (the published structure) is kept.
- Paper 2a reports v1, the diagnostics and v2 side by side.

**9.5 Implementation notes (fixed before the v2 runs).**
- A_ref per (mesh, n) is calibrated with the bone and perfusion output sets of ST13 design 706 (front speed 0.22 mm/day on the 2D strip); it is held fixed during calibration and LOO.
- Exact centroid verified against a 200,001-point grid (max difference < 5e-5, ST13 and S11 sets; tests/test_rulebase.py). With substeps = 1 and the grid centroid the v1 loop is reproduced exactly (test).
- Observation from the setup checks (not a v2 result): the v1 setting (A_ref 1.2725 with the ST13 design 706 sets, 0.16 mm^2 mesh) gives a front speed of 0.236 mm/day, i.e. 7 % above the S11 target.
- Scripts: run_v2_stage1.sh (tests, diagnostics/mesh_convergence.py), run_v2_stage2.sh (calibration, validation and LOO for cart50 and cart30); outputs in results_v2/.

**9.6 Amendment v2.1 (after stage 1, before any calibration).** Stage 1 (results_v2/mesh_v2.0) showed:
- at n = 1 the bone front cannot reach 0.22 mm/day on the finer meshes (M3, M4: 0.176 and 0.120 mm/day at the A_ref search limit), confirming the propagation limit of the v1 loop;
- with sub-steps, the perfusion front advances at about 0.63 mm/day (M2/M3, n = 16) although the bone front is 0.22 mm/day. S11 set both the maximum bone formation rate and the maximum revascularization rate to 0.22 mm/day; the v2.0 setting therefore violated the S11 revascularization constraint, and the healing of the low-IFM cases was dominated by this (V2 e about 48 for both cartilage variants on every mesh);
- the convergence table could not qualify M4 (no finer mesh) or n = 1 / n = 16 (no 2n level).
Amendment:
- the revascularization factor mu (S11 Sect. 2.5) is set per (mesh, n) so that the perfusion front on the 2D strip advances at 0.22 mm/day; this applies the published S11 constraint and is not a fit to IFM;
- sub-steps n in {4, 8, 16, 32}; meshes M1-M5 (M5: 0.02 mm^2); the n criterion compares consecutive levels;
- the 0.02 mm criterion and the production rule (Sect. 9.2) are unchanged. Per-case differences are reported, because the high-IFM cases (B, V3) lie near the union/non-union transition, where small numerical differences change the outcome.

**9.7 Amendment v2.2 (after stage 1 v2.1, before any calibration).** Stage 1 v2.1 (results_v2/mesh; 65.6 min):
- Both fronts are held at 0.22 mm/day on every mesh and n. Mu is mesh-independent (0.335-0.344).
- The low-IFM cases converge. Between M4 and M5 at n = 32, the maximum IFM difference is A 0.015, V1 0.005, V2 0.011 and V4 0.006 mm (cart50). Between M3 and M4 at n = 16 it is 0.016-0.034 mm.
- The high-IFM cases (B, V3) do not meet the 0.02 mm criterion on any setting (0.10-0.57 mm). Their outcome switches between union and non-union with small numerical changes. The criterion of Sect. 9.2 is therefore not met, and the rule selects M5, n = 32.
- Cost: M5, n = 32 takes 110-180 s per simulation on the Mac. Calibration would need about 20 h per variant, and LOO is infeasible (more than 6 days per variant).
- Feasibility probe (diagnostics/probe_v2.py, M3, n = 16, 52 LHS designs within the ST13 bounds; not a protocol result): the behavior is bimodal. Each design either heals V2 within 2-3 weeks (e 44-60) or not at all. B either heals (e 3.5-5) or never heals (e about 33). No design had e < 10 for both V2 and B.

Amendment:
- Two-fidelity procedure. Calibration (Sect. 3) runs at M3, n = 16 (setting_M3_16.json). The calibrated designs and ST13 design 706 are re-evaluated at M4, n = 32 (setting_M4_32.json), and the errors at M4, n = 32 are the reported results.
- LOO-CV (Sect. 8) is deferred. It is run only if at least one cartilage variant passes K1 at M4, n = 32; otherwise the structural failure is reported without LOO.
- Exploratory analysis (labelled as such in the paper): a time-scale scan. Both front speeds are set to v in {0.22, 0.15, 0.10, 0.07, 0.05, 0.035} mm/day at M3, n = 16, with ST13 design 706, six cases (diagnostics/timescale_scan.py). It asks which time scale the in vivo courses would require, compared with the S11 fluorochrome value of 0.22 mm/day.

**9.8 Stage 2 record and LOO decision (2026-10-26, before any LOO run).** Stage 2 (run_v2_stage2.sh; calibration 107.5 min for cart30) results, errors e per case:

| | A | B | V1 | V2 | V3 | V4 | mean V | K1 | K3 | K4 (at bounds) |
|---|---|---|---|---|---|---|---|---|---|---|
| cart50 calibrated, M3/n16 | 2.44 | 0.26 | 16.29 | 42.19 | 1.56 | 33.59 | 23.41 | fail (1.35) | fail | pass (1) |
| cart50 calibrated, M4/n32 | 3.02 | 1.42 | 16.46 | 43.52 | 3.57 | 37.08 | 25.16 | fail (2.22) | fail | |
| cart30 calibrated, M3/n16 | 0.43 | 0.12 | 1.20 | 21.03 | 0.85 | 6.55 | 7.41 | pass (0.28) | fail | fail (7) |
| cart30 calibrated, M4/n32 | 1.00 | 0.53 | 2.30 | 28.81 | 3.08 | 9.06 | 10.81 | pass (0.77) | fail | |
| ST13 design 706, M4/n32, cart50 | 2.67 | 4.92 | 14.35 | 44.16 | 2.91 | 38.34 | 24.94 | | | |
| ST13 design 706, M4/n32, cart30 | 2.72 | 4.81 | 15.19 | 44.43 | 3.14 | 39.01 | 25.44 | | | |

K2 passes for both calibrated variants (B, day 56: 0.098 and 0.070 mm at M4/n32). cart30 parameters at bounds: E_bone, E_conn, nu_bone, nu_conn, dec_perf, dec_cart, exp_bone. cart30 PSO stopped at the generation limit (35).

Decision under 9.7: cart30 passes K1 at M4/n32, so LOO-CV is triggered for cart30 only. It runs with the baseline parameter set at M3/n16 and the held-out errors are re-evaluated at M4/n32 (calibration/loo_reevaluate.py; run_v2_stage3.sh). L1 is evaluated on the M4/n32 held-out errors. Deviation from Sect. 8: the rate variant is not run in v2, because mu is now fixed by the S11 perfusion-front constraint (9.6) and H-rate was rejected in v1; the time-scale dependence is reported through the exploratory scan instead.

Time-scale scan (exploratory; design 706, cart50, M3/n16; mu 0.337-0.349 at every v, A_ref 0.247 to 0.007 mm^2):

| v (mm/day) | A | B | V1 | V2 | V3 | V4 | mean V |
|---|---|---|---|---|---|---|---|
| 0.220 | 2.40 | 4.45 | 12.65 | 42.62 | 2.73 | 37.17 | 23.79 |
| 0.150 | 1.26 | 2.81 | 4.88 | 31.85 | 2.56 | 17.01 | 14.08 |
| 0.100 | 0.22 | 29.18 | 1.17 | 9.94 | 2.79 | 5.70 | 4.90 |
| 0.070 | 0.87 | 30.48 | 1.92 | 1.18 | 2.95 | 6.51 | 3.14 |
| 0.050 | 1.37 | 31.62 | 2.65 | 0.75 | 2.98 | 7.22 | 3.40 |
| 0.035 | 2.15 | 30.93 | 3.20 | 0.82 | 3.09 | 7.62 | 3.69 |

At v <= 0.10 both high-IFM cases (B, IFM0 1.3 mm; V3, IFM0 1.43 mm) stay non-united (IFM at day 56: B 1.22, V3 1.36 mm at v = 0.07; in vivo 0.14 and 0.35 mm). The V3 error remains near 3 only because of its large in vivo SD.

**9.9 Stage 3 record (LOO-CV, cart30, baseline; 2026-10-27).** run_v2_stage3.sh, exit 0. Held-out errors (M3/n16 -> M4/n32): A 3.63 -> 3.48, B 33.81 -> 33.79, V1 2.15 -> 1.61, V2 21.78 -> 29.42, V3 3.39 -> 3.41, V4 7.09 -> 7.31. CV mean at M4/n32 13.17, median 5.40: L1 fails. Fold "all": G_train 8.131 per case. dec_perf at its bound (-0.025) in all seven folds; E_bone and nu_cart span their full ranges across folds. Reference (design 706, nd -0.05, cart30, M4/n32): A 3.22, B 4.08, V1 17.92, V2 45.43, V3 3.16, V4 41.06.

Post hoc analysis of the 4,124 designs in results_v2/loo_cart30_baseline/simulations.csv (M3/n16; optimizer-sampled, not exhaustive):
- Of 2,203 designs evaluated on both B and V2, 58 unite B (IFM day 56 <= 0.25 mm); the lowest V2 error among them is 21.78. Among the 2,145 that do not unite B, the lowest V2 error is 0.52. The B-V2 Pareto front has no point with both errors below 21.78.
- Of 2,927 designs evaluated on B and V3, 9 unite V3 (IFM day 56 <= 0.5 mm), all of which also unite B.
- Held-out B fails because without B the only high-IFM training case is V3, whose non-union costs about 3.4 (large late in vivo SDs), so the optimizer settles in the non-union basin. Held-out V2 fails because with B in training the optimizer settles in the union basin, which stiffens V2 by week 2-3.
- The fold "all" PSO ended at 8.13 per case although the fold V2 design gives 5.52 per case on all six cases (A 0.82, B 0.59, V1 2.05, V2 21.78, V3 0.98, V4 6.88). The objective is multimodal and the pre-registered PSO does not reliably find the better basin.

## 10. Protocol v3: structural extension (separate from the v2 confirmatory analysis)

v2 (Sect. 9) is the confirmatory result and is reported in full. v3 is designed after the six cases were seen; this is stated in the paper. The v3 hypothesis, parameters, criteria and LOO settings will be fixed in Sect. 10.2 before any v3 calibration. ST13 case V5 (axial, gap 9.9 mm, IFM0 3.99 mm), not used in v1 or v2, is reserved as an external test and is digitized and sealed before Sect. 10.2 is written.

**10.1 Exploratory screen (2026-10-27, before any v3 registration).** Candidate 1: front speed below the S11 fluorochrome maximum (v <= 0.22 mm/day) together with a larger healing region (maximal callus index mCI; ST13 Sect. 2.5, V5). Implementation: calibration/model.py SETTINGS["mci"] (outer callus radius R = 8 mCI mm; callus height unchanged; mCI 2.0 reproduces v1/v2). Script diagnostics/v3_screen.py, run_v3_screen.sh: v in {0.22, 0.15, 0.10, 0.07} (A_ref, mu from results_v2/timescale_scan.json), mCI in {2.0, 2.5, 3.0, 3.5}, six cases, two fixed designs (ST13 design 706, cart50, -3 %; LOO fold-V2 design, cart30, -5 %), M3/n16, no fitting. Decision rule, fixed before the run: candidate 1 proceeds only if, for at least one design, at some v <= 0.10 and mCI <= 3.5, B unites (IFM day 56 <= 0.25 mm), V3 unites (IFM day 56 <= 0.50 mm) and the mean error of A, V1, V2, V4 is <= 5. Otherwise candidate 1 is dropped and candidates 2-3 (cartilage stiffness bounds, onset delay) are screened.

10.1 result (results_v3/screen.json, 25.6 min): candidate 1 dropped (0 of 16 settings with v <= 0.10 pass). At v <= 0.10, B and V3 stay non-united for every mCI and both designs (IFM day 56: B 0.65-1.25, V3 1.33-1.39 mm; the trajectories are nearly flat). B and V3 unite together only at v = 0.22 with mCI 2.5-3.5 (design 706: low-IFM mean error 22.1-22.4; fold-V2 design, mCI 3.5: 8.7, V2 22.4). The effect of mCI on V3 is non-monotonic (design 706, v = 0.22: V3 e 2.73/13.10/33.32/18.32 for mCI 2.0/2.5/3.0/3.5).

**10.1b Exploratory screen 2 (2026-10-27, before any v3 registration).** Candidate 2: callus tissue moduli below the ST13 lower bounds, anchored in Leong & Morgan 2008 (Acta Biomater 4:1569-1575; rat callus, day 35, nanoindentation: chondroid 1.39-4.42 MPa, woven bone 26.9-1010 MPa, granulation 0.61-1.27 MPa). Script diagnostics/v3_screen2.py, run_v3_screen2.sh: v in {0.22, 0.15}, mCI in {2.0, 3.5}, E_cart in {design, 3.0, 1.4} MPa, E_bone in {design, 200, 100} MPa, six cases, designs 706 and fold-V2, M3/n16, no fitting. Decision rule, fixed before the run: proceeds only if, for at least one design and setting, B unites (<= 0.25 mm), V3 unites (<= 0.50 mm) and the mean error of A, V1, V2, V4 is <= 5. Caveat for any later registration: the anchor is a single rat at one time point; sheep callus moduli must be sought before bounds are fixed.

10.1b result (results_v3/screen2.json, 45.4 min): candidate 2 dropped (0 of 72 settings pass). Softer callus tissue (E_cart 1.4-3 MPa, E_bone 100-200 MPa) removes the early low-IFM stiffening even at v = 0.22 (low-IFM mean error 2.4-3.2; V2 0.48-1.38) but turns B and V3 into non-unions in all but one setting (B e 9.2-34, V3 not united). Only the two settings with ST13-range moduli unite both.

**10.1c Exploratory screen 3 (2026-10-27; no new simulations).** Candidate 3: onset delay tau, implemented as suspension of all fuzzy updates until day tau under constant load, which makes the delayed run the undelayed run shifted by tau (exact for tau in {7, 14, 21} with the stored IFM days; IFM(day 1) = IFM0 in all runs). Script diagnostics/v3_delay_shift.py; sources: LOO cache (4,124 designs), screens 1 and 2. Same decision rule. Result (results_v3/delay_shift.json): candidate 3 proceeds (3 passing combinations). Best: ST13 design 706 unchanged (cart50, -3 %), v = 0.22 (S11), mCI 3.5, tau = 21 d: A 0.40, B 1.36, V1 0.76, V2 3.38, V3 1.02, V4 6.11 (mean of six 2.17; V1-V4 mean 2.82; B and V3 united, IFM day 56 0.15 and 0.19 mm). Same design and v with mCI 2.0-3.0 at tau 21: mean 2.43-3.17. Residual misfit: low-IFM IFM falls below in vivo after day 42 (V2 0.06 vs 0.23 mm; V4 0.04 vs 0.14 mm at day 56). Limitations: M3/n16 only; tau resolved at 7 d; the freeze form implies no tissue change before tau, which must be checked against sheep histology; the candidate was chosen after three screens on the same six cases.

**10.2 Registration of v3 (2026-10-27; written before any v3 calibration).**

Decision (user, 2026-10-27): register the freeze form of the onset delay. Case V5 is excluded from Paper 2a; its gap (9.9 mm; half-gap 4.95 mm > endosteal callus height 4.5 mm) lies outside the model geometry, and large-gap cases will be addressed by a model extension before Paper 2b. Disclosure: the v3 structure was selected after the exploratory screens 10.1-10.1c on the same six cases; in Paper 2a the LOO-CV is the only out-of-sample test of v3.

Model. ST13 rule base with the ST13 cartilage output (cart50) and ST13 design 706 at its published values (destructive threshold -3 %). Exact centroid defuzzification. Front speeds at the S11 value of 0.22 mm/day (A_ref and mu from the v2 setting files). Load, fixator and geometry as in v2, except: (i) onset delay tau: no fuzzy update before day tau, after which the full healing program starts, including the S11 day-10 marrow perfusion onset counted from tau (verification/healing.py onset_delay; equivalent to a shift of the undelayed run by tau, tests/test_rulebase.py::test_onset_delay_equals_time_shift); (ii) global maximal callus index mCI (outer callus radius 8 mCI mm; callus height unchanged), the same value for all cases.

Free parameters and grid. tau in {0, 1, ..., 35} days; mCI in {2.0, 2.1, ..., 3.5} (bounds: the two values used by ST13). All 12 ST13 parameters fixed. Exhaustive grid evaluation, no optimizer. Ties are broken by the lowest mCI, then the lowest tau.

Fidelity. Primary analysis at M4, n = 32 (results_v2/mesh/setting_M4_32.json), the v2 reporting setting. The same analysis at M3, n = 16 is reported as a sensitivity result.

Step 1, calibration. Minimize G = e_A + e_B over the grid (e as in Sect. 2).
Step 2, criteria at the step-1 estimate: K1 (mean of e_A, e_B <= 1.0, each <= 1.5), K2 (IFM of B at day 56 <= 0.25 mm), K3 (mean e over V1-V4 <= 2.0). K4 is replaced by a report of any estimate at a grid bound (not a kill criterion; with two parameters the v1 rule is vacuous).
Step 3, LOO-CV over the six cases: for each held-out case, minimize the summed error of the other five over the grid and evaluate the held-out case. L1: CV mean <= 2.0. Also reported: median, fold estimates, and fold "all".
Kill criteria: v3 is supported in Paper 2a only if K1, K2, K3 and L1 all pass at M4, n = 32. Otherwise v3 is reported as failed, together with the v2 results.

Also reported (not criteria): error surfaces over (tau, mCI) for identifiability; ST13 design 706 at tau = 0, mCI = 2.0 as reference.

Secondary analyses, labelled exploratory and run after the primary analysis: (a) calibration of the 12 ST13 parameters together with tau and mCI (LHS + PSO as Sect. 3, at M3, n = 16, re-evaluated at M4, n = 32); (b) qualitative comparison of the simulated tissue patterns with sheep callus histology (for example Vetter et al. 2010), to test whether a freeze before tau is compatible with observed early tissue formation; (c) M3/n16 against M4/n32.

Scripts: calibration/v3_grid.py; run_v3_primary.sh. Registration hashes: results_v3/REGISTRATION_v3.md.
