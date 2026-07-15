# HSPD1: Known-Structural-State DSSO Compatibility Analysis

## Interpretation boundary

This analysis tests whether the A636T and WT run-level DSSO profiles are differentially compatible with the supplied candidate PDB states. It does not determine a unique brain structure, prove occupancy of a pure state, or establish population-level biological significance.

## Analysis status

- **Inferential testable:** False
- **Status:** selected structural links 2 < 3; WT strata with selected-link support 1 < 2
- Gene: `HSPD1`
- Conditions: `MS659_A636T` (A636T) versus `MS660_WT` (WT)
- Mapping mode: `sequence`
- Distance metric: `JWALK`
- Soft compatibility center/slope: 30 Å / 3 Å
- Homomer assignment policy: `best`
- Primary link set: `informative`
- Weighted support used: A636T=0.134199; WT=0.0113094
- Runs with selected-link support: A636T=28; WT=4
- Salt strata with selected-link support: A636T=4; WT=1
- Jwalk missing-path policy: `penalize`
- Jwalk maximum requested path: 60 Å
- Compatibility assigned to a missing/non-accessible path: 0.0001

Jwalk SASD is used as a solvent-accessible route estimate. A missing path can mean that one endpoint was not accessible, that no route was found, or that the route exceeded the configured Jwalk maximum. The chosen missing-path policy is therefore an explicit modeling assumption.

## Structural states

| State | File | Model | Selected chains | Description | Jwalk source |
|---|---|---:|---|---|---|
| APO | `/home/yizhi/XL_MS/Structures/HSPD1/HSPD1_APO_7AZP_assembly1.cif` | 0 | A,B,C,D,E,F,G |  | external_jwalk |
| ATPMIMIC_FOOT | `/home/yizhi/XL_MS/Structures/HSPD1/HSPD1_ATPMIMIC_FOOT_6HT7_assembly1.cif` | 0 | J,I,H,N,M,L,K,G,F,E,D,C,B,A |  | external_jwalk |
| ADP_FOOT | `/home/yizhi/XL_MS/Structures/HSPD1/HSPD1_ADP_FOOT_6MRC_assembly1.cif` | 0 | H,N,M,L,K,J,I,D,C,B,A,G,F,E |  | external_jwalk |
| ADP_HALF | `/home/yizhi/XL_MS/Structures/HSPD1/HSPD1_ADP_HALF_6MRD_assembly1.cif` | 0 | A,G,F,E,D,C,B |  | external_jwalk |

## Mapping and information content

- Observed XL features reported for the gene: 11
- Observed links mappable in every supplied state: 10
- Observed state-informative links: 2
- Observed links selected for the primary test: 2
- Total mapped rows including optional candidate opportunities: 11

A link is state-informative when it maps in every state and its max-minus-min soft compatibility is at least the configured threshold. The primary test always uses experimentally observed links only. Candidate-universe rows, when supplied, are isolated in a secondary exploratory analysis.

## State compatibility

| gene   | state         |   compatibility_A636T |   compatibility_WT |   difference_A636T_minus_WT | distance_metric   |   distance_cutoff_A |   sigmoid_slope_A | homomer_policy   |
|:-------|:--------------|----------------------:|-------------------:|----------------------------:|:------------------|--------------------:|------------------:|:-----------------|
| HSPD1  | APO           |               0.78778 |            0.78218 |                     0.00560 | JWALK             |            30.00000 |           3.00000 | best             |
| HSPD1  | ATPMIMIC_FOOT |               0.87922 |            0.89896 |                    -0.01974 | JWALK             |            30.00000 |           3.00000 | best             |
| HSPD1  | ADP_FOOT      |               0.86784 |            0.88987 |                    -0.02203 | JWALK             |            30.00000 |           3.00000 | best             |
| HSPD1  | ADP_HALF      |               0.82049 |            0.83979 |                    -0.01929 | JWALK             |            30.00000 |           3.00000 | best             |

## Global condition-by-state interaction

| gene   | test                                  |    effect | effect_definition                                                 | alternative   |   p_value |   exceedances | permutation_method   |
|:-------|:--------------------------------------|----------:|:------------------------------------------------------------------|:--------------|----------:|--------------:|:---------------------|
| HSPD1  | global_condition_by_state_interaction | 0.0112859 | RMS across states of centered (compatibility_A - compatibility_B) | greater       |       nan |             0 | not run              |

## Pairwise state contrasts

| gene   | state2        | state1        |   state2_minus_state1_A636T |   state2_minus_state1_WT |   condition_by_state_interaction | direction                                         | alternative   |   p_value |   exceedances | permutation_method   |   FDR_BH_within_gene_contrasts |
|:-------|:--------------|:--------------|----------------------------:|-------------------------:|---------------------------------:|:--------------------------------------------------|:--------------|----------:|--------------:|:---------------------|-------------------------------:|
| HSPD1  | ATPMIMIC_FOOT | APO           |                   0.0914447 |               0.116779   |                      -0.0253344  | A636T shifted toward APO relative to WT           | two-sided     |       nan |             0 | not run              |                            nan |
| HSPD1  | ADP_FOOT      | ATPMIMIC_FOOT |                  -0.0113832 |              -0.00908524 |                      -0.00229798 | A636T shifted toward ATPMIMIC_FOOT relative to WT | two-sided     |       nan |             0 | not run              |                            nan |
| HSPD1  | ADP_HALF      | ADP_FOOT      |                  -0.0473458 |              -0.0500866  |                       0.00274086 | A636T shifted toward ADP_HALF relative to WT      | two-sided     |       nan |             0 | not run              |                            nan |
| HSPD1  | ADP_HALF      | APO           |                   0.0327157 |               0.0576072  |                      -0.0248915  | A636T shifted toward APO relative to WT           | two-sided     |       nan |             0 | not run              |                            nan |

## Observed-restraint categories

Matched, violating, and non-accessible fractions are descriptive decompositions of salt-balanced observed-link support. They complement—but do not replace—the quantitative condition-by-state interaction test.

| condition   | state         |   total_salt_balanced_support |   weighted_mean_compatibility |   fraction_matched |   fraction_violating |   fraction_nonaccessible_or_over_jwalk_limit |   fraction_unmapped_or_excluded |
|:------------|:--------------|------------------------------:|------------------------------:|-------------------:|---------------------:|---------------------------------------------:|--------------------------------:|
| A636T       | APO           |                       0.20032 |                       0.84790 |            0.93838 |              0.00000 |                                      0.00000 |                         0.06162 |
| WT          | APO           |                       0.06400 |                       0.93163 |            0.66222 |              0.00000 |                                      0.00000 |                         0.33778 |
| A636T       | ATPMIMIC_FOOT |                       0.20032 |                       0.91332 |            0.91741 |              0.02097 |                                      0.00000 |                         0.06162 |
| WT          | ATPMIMIC_FOOT |                       0.06400 |                       0.96434 |            0.66222 |              0.00000 |                                      0.00000 |                         0.33778 |
| A636T       | ADP_FOOT      |                       0.20032 |                       0.90519 |            0.91741 |              0.02097 |                                      0.00000 |                         0.06162 |
| WT          | ADP_FOOT      |                       0.06400 |                       0.96192 |            0.66222 |              0.00000 |                                      0.00000 |                         0.33778 |
| A636T       | ADP_HALF      |                       0.20032 |                       0.87049 |            0.91741 |              0.02097 |                                      0.00000 |                         0.06162 |
| WT          | ADP_HALF      |                       0.06400 |                       0.93735 |            0.66222 |              0.00000 |                                      0.00000 |                         0.33778 |

## Statistical definition

For condition *c* and state *s*, the primary score is the salt-balanced weighted mean of observed-link soft distance compatibilities. For state2 versus state1:

`Δ = (C[A,state2] − C[A,state1]) − (C[B,state2] − C[B,state1])`

Pairwise p-values use the `two-sided` alternative. Condition labels are exchanged only within upstream matched salt strata. The BH column corrects only contrasts in this one-gene run. Multiple proteins require an additional across-protein/contrast correction.

## Assignment rules

- Looplinks are constrained to same-chain assignments because both linked residues occur in one peptide molecule.
- Same-protein intralinks in multi-copy assemblies may be intra- or inter-subunit. All minimum distances and assignment counts are retained for sensitivity analysis.
- `best` is an optimistic minimum-distance policy; repeat homomer analyses with `intra`, `inter`, and `mean` where biologically plausible.
- Jwalk is executed on a canonical PDB that retains all amino-acid chains as steric obstacles while renumbering target chains into the exact protein-coordinate system.

## Run/stratum design

| stratum   |   MS659_A636T |   MS660_WT |
|:----------|--------------:|-----------:|
| 100mM     |            23 |          9 |
| 200mM     |             8 |         15 |
| 350mM     |            14 |         22 |
| 500mM     |            20 |         22 |
| 50mM      |            10 |          9 |
| 750mM     |             9 |         20 |

## Required cautions

1. Known PDB states are hypotheses, not an exhaustive conformational ensemble.
2. Differential XL support can reflect conformation, oligomerization, partner occupancy, accessibility, PTMs, abundance, extraction, digestion, or fraction recovery.
3. Use the exact Scout sequence for mapping; direct PDB numbering is acceptable only after manual verification.
4. Use comparable biological assemblies and evaluate homomeric assignment sensitivity.
5. Technical raw-file permutations do not provide population inference; independent mice are required.
6. Opportunity-universe analyses require digestion/search-space matching and external calibration before their relative link propensities can be called probabilities.

## Principal output files

- `state_compatibility_summary.csv`
- `global_state_interaction_test.csv`
- `pairwise_state_contrasts.csv`
- `link_state_distance_compatibility.csv`
- `observed_link_constraint_summary.csv`
- `chain_mapping_qc.csv`
- `jwalk_run_qc.csv` and `jwalk_work/` when Jwalk is used
- `opportunity_*` files when a candidate universe is supplied
- PNG/PDF state, distance, category, heatmap, driver, and null-distribution figures