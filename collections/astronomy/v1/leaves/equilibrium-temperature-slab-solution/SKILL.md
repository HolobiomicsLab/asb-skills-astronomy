---
name: equilibrium-temperature-slab-solution
description: Use when quantifying how thermal evolution, helium treatment, and recombination-radiation
  transfer alter H I / He I / He II column densities in Lyman-limit slabs exposed
  to an isotropic UV background.
when_to_use_negative:
- The problem geometry is a spherical distribution with a central point source or
  in an isotropic background rather than a semi-infinite slab with an isotropic background
- Non-equilibrium (time-dependent) ionization or metal line cooling is required —
  the paper states these are future improvements not yet included
- The gas is at densities far from Lyman-limit system densities, where the paper's
  order-of-magnitude column-density sensitivities may not hold
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rabacus
  role: Provides the slab analytic radiative-transfer solvers (grey_fixT_B, hm12_fixT_B,
    hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt) and the HM12 background table used to
    run the five-model comparison
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted wrapper language in which the slab models are invoked and results
    tabulated to CSV
  repo: https://github.com/galtay/rabacus
- name: Quantities
  role: Supplies units for all slab input/output variables (e.g. kpc slab thickness)
  repo: http://pythonhosted.org//rabacus
- name: Fortran 90
  role: Core solvers wrapped via Numpy f2py; must be compiled with a Fortran compiler
    (F90=gfortran or ifort) before the slab models can run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:a866e432fb086c68054a10d39ce0625deb3b4d25386c1baf3e031393506364cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# equilibrium-temperature-slab-solution

## Summary

Reproduce the five-model slab process-isolation study from the rabacus paper: solve radiative transfer through a semi-infinite slab in an isotropic HM12 background, comparing fixed vs self-consistently evolved equilibrium temperatures, case-A vs case-B recombination, and on-the-spot vs full recombination-radiation transfer.

## When to use

Use when quantifying how thermal evolution, helium treatment, and recombination-radiation transfer alter H I / He I / He II column densities in Lyman-limit slabs exposed to an isotropic UV background.

## When NOT to use

- The problem geometry is a spherical distribution with a central point source or in an isotropic background rather than a semi-infinite slab with an isotropic background
- Non-equilibrium (time-dependent) ionization or metal line cooling is required — the paper states these are future improvements not yet included
- The gas is at densities far from Lyman-limit system densities, where the paper's order-of-magnitude column-density sensitivities may not hold

## Inputs

- HM12 photorates table evaluated at z=3.0 (ra.HM12_Photorates_Table)
- Semi-infinite slab definition: overdensity Δ=1800, thickness L=4.61 kpc, exposed to isotropic HM12 ionizing background

## Outputs

- Per-model N_HI, N_HeI, N_HeII column densities and peak temperature for the five models (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt)
- Cross-model ratio table (e.g. slab_model_comparison.csv) with grey/polychromatic, evoT/fixT, and rt/case-B factors

## How to apply

Instantiate ra.HM12_Photorates_Table at z=3.0 and define a semi-infinite slab with overdensity Δ=1800 and thickness L=4.61 kpc. Run the grey fixT case-B model (grey_fixT_B), then the polychromatic hm12_fixT_B and hm12_evoT_B models, recording N_HI, N_HeI, N_HeII and peak temperature. The rationale is to isolate each physical process: polychromatic vs grey spectrum, temperature evolution, case-A vs case-B recombination (hm12_evoT_A), and on-the-spot vs recombination-radiation transfer (hm12_evoT_rt). Tabulate all five models, e.g. to slab_model_comparison.csv, and check the ratios against the paper's reported factors: grey/polychromatic N_HI ≈ 1.8, evoT/fixT N_HI ≈ 11, N_HeI ≈ 6.3, T_peak ≈ 1.6e4 K, rt/case-B ≈ 3.2, total column-density variation 3–11.

## Related tools

- **rabacus** (Provides the slab analytic radiative-transfer solvers (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt) and the HM12 background table used to run the five-model comparison) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted wrapper language in which the slab models are invoked and results tabulated to CSV) — https://github.com/galtay/rabacus
- **Quantities** (Supplies units for all slab input/output variables (e.g. kpc slab thickness)) — http://pythonhosted.org//rabacus
- **Fortran 90** (Core solvers wrapped via Numpy f2py; must be compiled with a Fortran compiler (F90=gfortran or ifort) before the slab models can run)

## Examples

```
import rabacus as ra; hm12 = ra.HM12_Photorates_Table(z=3.0); slab = ra.SlabGrey(raf=..., fixT=True); slab2 = ra.SlabHM12(raf=hm12, fixT=False)
```

## Evaluation signals

- Polychromatic/grey N_HI ratio ≈ 1.8
- evoT/fixT N_HI factor ≈ 11 and N_HeI factor ≈ 6.3, with T_peak ≈ 1.6e4 K at the slab edge
- rt-vs-case-B N_HI factor ≈ 3.2 (hm12_evoT_rt vs hm12_evoT_B)
- Total column-density variation across the five models lies between 3 and 11
- Tabulated values accepted when they reproduce the paper's factors to within differences in the last significant figure, written to slab_model_comparison.csv

## Limitations

- Metal line cooling is not yet included (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- No version number for the rabacus package is stated in the provided text

## Evidence
