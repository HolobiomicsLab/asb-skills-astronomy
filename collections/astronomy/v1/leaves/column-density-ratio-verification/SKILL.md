---
name: column-density-ratio-verification
description: Use when replicating or validating the five-model Slab2Bgnd study with
  Slab2Bgnd, or when any rabacus slab calculation must be checked against published
  column-density ratio benchmarks (e.g. grey/polychromatic N_HI ≈ 1.8, evoT/fixT ≈
  11).
when_to_use_negative:
- The geometry is a spherical distribution with a central point source or an isotropic
  background on a sphere rather than the semi-infinite slab in an isotropic background
  — the benchmark ratios (1.8, 11, 6.3, 3.2) are specific to the Slab2Bgnd configuration.
- The problem requires non-equilibrium solvers or metal line cooling, which rabacus
  does not yet include.
- The slab density is far outside the Lyman-limit regime (Δ=1800); the order-of-magnitude
  column-density sensitivity to thermal treatment was established for Lyman-limit
  system densities.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rabacus
  role: Python package whose Slab2Bgnd class produces the five slab solutions whose
    column densities are cross-checked against the published ratios
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted host language used to instantiate the HM12 table, run the five
    models, compute the ratios, and write slab_model_comparison.csv
- name: Quantities
  role: Provides units on all rabacus variables so column densities and temperatures
    are compared with consistent dimensions
  repo: https://www.github.com/python-quantities/python-quantities.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:f143c10e939c8e25a0632a7ac1781bcfdd6c4bdb365065b8d049ca430877f364
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Column-density ratio verification

## Summary

A verification skill for rabacus slab radiative-transfer studies: cross-check computed hydrogen and helium column-density ratios (and peak temperatures) across grey, polychromatic, fixed-temperature, evolved-temperature, case-A/B, and recombination-radiation-transfer models against the paper's reported benchmark factors.

## When to use

Use when replicating or validating the five-model Slab2Bgnd study with Slab2Bgnd, or when any rabacus slab calculation must be checked against published column-density ratio benchmarks (e.g. grey/polychromatic N_HI ≈ 1.8, evoT/fixT ≈ 11).

## When NOT to use

- The geometry is a spherical distribution with a central point source or an isotropic background on a sphere rather than the semi-infinite slab in an isotropic background — the benchmark ratios (1.8, 11, 6.3, 3.2) are specific to the Slab2Bgnd configuration.
- The problem requires non-equilibrium solvers or metal line cooling, which rabacus does not yet include.
- The slab density is far outside the Lyman-limit regime (Δ=1800); the order-of-magnitude column-density sensitivity to thermal treatment was established for Lyman-limit system densities.

## Inputs

- rabacus HM12_Photorates_Table instance evaluated at z=3.0
- Slab2Bgnd problem definition: semi-infinite slab, overdensity Δ=1800, thickness L=4.61 kpc, isotropic HM12 ionizing background
- Solution objects from the five slab runs (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt)

## Outputs

- Diagnostic column-density ratios (grey/polychromatic N_HI ≈ 1.8; evoT/fixT N_HI ≈ 11 and N_HeI ≈ 6.3; rt/case-B N_HI ≈ 3.2)
- Peak equilibrium temperature T_peak ≈ 1.6e4 K for the evolved-temperature case-B slab
- slab_model_comparison.csv tabulating N_HI, N_HeI, N_HeII and T_peak for all five models

## How to apply

Run the five slab cases in rabacus at z=3.0 with the HM12 photorates table: grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, and hm12_evoT_rt, for a semi-infinite slab with overdensity Δ=1800 and thickness L=4.61 kpc exposed to the isotropic HM12 background. From each solution record N_HI, N_HeI, N_HeII columns and peak temperature, then form the diagnostic ratios: polychromatic/grey N_HI ≈ 1.8, hm12_evoT_B/hm12_fixT_B N_HI ≈ 11 and N_HeI ≈ 6.3, T_peak ≈ 1.6×10^4 K for the evolved-temperature case, and hm12_evoT_rt/hm12_evoT_B N_HI ≈ 3.2. Confirm the total column-density variation across all five models lies between 3 and 11 (i.e. the physical processes — spectral shape, helium, temperature evolution, recombination radiation — matter at the order-of-magnitude level for Lyman-limit slabs). Tabulate all five models in slab_model_comparison.csv and accept ratios that agree with the paper's factors to within the last significant figure.

## Related tools

- **rabacus** (Python package whose Slab2Bgnd class produces the five slab solutions whose column densities are cross-checked against the published ratios) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted host language used to instantiate the HM12 table, run the five models, compute the ratios, and write slab_model_comparison.csv)
- **Quantities** (Provides units on all rabacus variables so column densities and temperatures are compared with consistent dimensions) — https://www.github.com/python-quantities/python-quantities.git

## Examples

```
import rabacus as ra; pt = ra.HM12_Photorates_Table(z=3.0); slab = ra.Slab2Bgnd(...)  # compare N_HI across grey_fixT_B, hm12_fixT_B, hm12_evoT_B
```

## Evaluation signals

- Grey/polychromatic N_HI ratio reproduces the reported value 1.8 to within the last significant figure
- hm12_evoT_B / hm12_fixT_B gives N_HI factor ≈ 11 and N_HeI factor ≈ 6.3
- Evolved-temperature case-B peak temperature reaches T_peak ≈ 1.6×10^4 K at the slab edge (elevated over the fixed 10^4 K case)
- Recombination-radiation-transfer vs case-B N_HI factor ≈ 3.2 (hm12_evoT_rt vs hm12_evoT_B)
- Total column-density variation across all five models falls between 3 and 11, and slab_model_comparison.csv contains rows for all five models

## Limitations

- Metal line cooling is not included in rabacus; the equilibrium temperatures (and hence the benchmark ratios) may shift once it is added as a planned future improvement.
- Non-equilibrium solvers are not yet included, so all comparisons assume ionization/thermal equilibrium states.
- Ratios are sensitive to the last significant figure tolerance — acceptance allows differences only in the last significant digit, so coarser agreement must be flagged rather than silently accepted.

## Evidence
