# Rule base: sources, reading choices and verification

## Sources

- **Rules:** Simon et al. (2011), Table 2: eight linguistic rules with seven inputs (dilatational strain, distortional strain, perfusion, perfusion of the neighborhood, cartilage fraction, bone fraction, bone fraction of the neighborhood) and three outputs (daily change of perfusion, cartilage and bone fraction).
- **Input membership functions:** Simon et al. (2011), Fig. 4. Strain invariants: Simon et al. (2011), Eqs. 9-10.
- **Output membership functions and parameters:**
  - published set Simon et al. (2011): output sets of Fig. 5, moduli of Table 1 (`published_set_simon2011.fis`);
  - published set Steiner et al. (2013): compressive destructive dilatational crossover at -3 %; generic output sets with 5 % transitions on a +/-50 % universe; thresholds -22/+22 % (perfusion), -28/+23 % (cartilage), -47.5/+22 % (bone); moduli 538/28/1.4 MPa; mixture exponents 4.5 (bone) and 3.1 (cartilage) (`published_set_steiner2013.fis`, design 706).
- **Designs of this study:** the same rule structure with the parameters of each design (`designs.json`; one `.fis` and one `.json` per design). Designs with the cartilage output universe +/-30 % (`cartilage_output_universe` 0.3) scale the cartilage thresholds by the same factor.

## Reading choices and assumptions (not stated in the sources)

- **Rules 1-2, dilatational range.** Simon et al. (2011), Table 2, reads "Neg. medium up to pos. high", but the dilatational variable has no "pos. high" set; read as up to "pos. medium".
- **Width of the moved destructive transition.** Steiner et al. (2013) moved the negative destructive crossover from -5 % to -3 % without stating the new transition width; kept at 2 % (-4 % to -2 %).
- **Chondrogenesis and perfusion.** The Simon et al. (2011) text states that chondrogenesis "was possible with higher mechanical stimuli and low perfusion"; Table 2, rule 4, has no perfusion condition. The table version is used.
- **"No change" rules** (Shefelbine et al. 2005, rules 4-5) are not part of the rule base.
- **Chondrogenesis modification of Steiner et al. (2013).** The article states that rules concerning chondrogenesis were modified but prints only the change of the output range; only that change is included.

## Encoding

- Mamdani inference with MATLAB Fuzzy Logic Toolbox semantics: AND = min, OR = max, NOT = 1 - membership, implication min, aggregation max, centroid defuzzification; the midpoint of the output range when no rule fires.
- Contiguous ranges such as "Neg. medium up to pos. medium" are encoded as one composite trapezoid. A linguistic rule with non-contiguous alternatives becomes one `.fis` rule per alternative, so the eight linguistic rules give eleven `.fis` rules.
- All membership functions are trapezoids [a, b, c, d]. Strains are absolute values (0.01 = 1 %).
- In the simulations, the centroid is computed exactly for the piecewise-linear aggregate (from protocol v2 on). With a 101-point grid, the default of the MATLAB toolbox, the centroid of the Steiner et al. (2013) bone output differs by up to 0.019 per day.
- Daily changes of perfusion and bone fraction are scaled by sqrt(A_ref / A_element) (Simon et al. 2011, Eq. 15); A_ref and the revascularization factor are set per mesh and sub-step count so that both fronts advance at 0.22 mm/day (`model_constants.json`, numerical settings).

## Verification of the rule base

| Check | Result |
|---|---|
| Partition of unity of all input sets | sum = 1 within 1e-12 on 40,000 points per variable |
| Published crossovers: +/-0.02, +/-0.85, +/-5 (-3 Steiner) % dilatational; 0.03, 7, 16 % distortional; 30, 70 % concentration | membership 0.5 for both neighboring sets at every crossover |
| Steiner et al. (2013) output thresholds | membership 0.5 at each threshold |
| Rule prototypes (one state per rule, both published sets) | the intended rule fires at 1.0, no other rule fires, output signs as Table 2 |
| `.fis` files against an independent evaluator, 20,000 random states | maximum difference < 1e-12 |
| Third implementation (scikit-fuzzy 0.5.0), 400 random states | maximum difference 0.002 (bone), 0.0004 (cartilage), 0.0001 (perfusion), equal to the quadrature error of a 1001-point centroid |

## References

- Simon U, Augat P, Utz M, Claes L (2011). Comput Methods Biomech Biomed Engin 14:79-93.
- Steiner M, Claes L, Ignatius A, Niemeyer F, Simon U, Wehner T (2013). J R Soc Interface 10:20130389.
- Shefelbine SJ, Augat P, Claes L, Simon U (2005). J Biomech 38:2440-2450.
