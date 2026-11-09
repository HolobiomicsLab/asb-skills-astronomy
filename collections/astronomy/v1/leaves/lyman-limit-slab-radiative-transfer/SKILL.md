---
name: lyman-limit-slab-radiative-transfer
description: Use when quantifying how spectral shape, equilibrium temperature evolution,
  helium treatment, and on-the-spot approximation change H I / He I / He II column
  densities in a slab exposed to an isotropic ionizing background at Lyman-limit densities.
when_to_use_negative:
- The geometry is a spherical distribution with a central point source or an isotropic
  background on a sphere — rabacus provides separate solvers for those cases, not
  the slab solver.
- The problem requires non-equilibrium chemistry or metal line cooling — the paper
  states these are future improvements not yet included in rabacus.
- Only a single ionization state is needed without radiative-transfer context — the
  five-model comparison machinery is overkill; use a simpler equilibrium calculation.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3945
tools:
- name: rabacus
  role: Provides the semi-infinite slab solvers (grey_fixT_B, hm12_fixT_B, hm12_evoT_B,
    hm12_evoT_A, hm12_evoT_rt) and the HM12 photorates table used for this comparison
    study
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted driver language; rabacus's Fortran 90 core is wrapped in Python
    via Numpy f2py and invoked from Python scripts
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Units package ensuring all slab and radiation variables carry physical units
    during the calculation
  repo: python-quantities/python-quantities
- name: Fortran 90
  role: Compiled core routines that execute the slab radiative-transfer solves thousands
    of times faster than pure Python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:adb6da0660b8097b0434b3f4e6c0cf3104a8c0dad1bcdb195aabc4f1874f3996
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3945
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# lyman-limit-slab-radiative-transfer

## Summary

Reproduce the rabacus five-model semi-infinite slab study (Fig. 3) for Lyman-limit systems, comparing grey vs. polychromatic HM12 spectra, fixed vs. self-consistently evolved equilibrium temperatures, and case-A/B vs. full recombination-radiation transfer to quantify how these approximations alter H I / He column densities.

## When to use

Use when quantifying how spectral shape, equilibrium temperature evolution, helium treatment, and on-the-spot approximation change H I / He I / He II column densities in a slab exposed to an isotropic ionizing background at Lyman-limit densities.

## When NOT to use

- The geometry is a spherical distribution with a central point source or an isotropic background on a sphere — rabacus provides separate solvers for those cases, not the slab solver.
- The problem requires non-equilibrium chemistry or metal line cooling — the paper states these are future improvements not yet included in rabacus.
- Only a single ionization state is needed without radiative-transfer context — the five-model comparison machinery is overkill; use a simpler equilibrium calculation.

## Inputs

- HM12 (Haardt & Madau 2012) meta-galactic ionizing background, instantiated via ra.HM12_Photorates_Table at redshift z=3.0
- Slab specification: semi-infinite geometry, overdensity Delta=1800, thickness L=4.61 kpc, exposed to isotropic background on both faces

## Outputs

- Per-model column densities N_HI, N_HeI, N_HeII and peak equilibrium temperature for the five slab models (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt)
- Comparison table slab_model_summary.csv (slab_model_comparison.csv) with cross-model ratios

## How to apply

Instantiate an ra.HM12_Photorates_Table at z=3.0 to define the isotropic meta-galactic background, then build the semi-infinite slab problem with overdensity Delta=1800 and thickness L=4.61 kpc using case-B recombination coefficients. Run the five rabacus slab models in order — grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, and hm12_evoT_rt — recording N_HI, N_HeI, N_HeII and the peak temperature for each. The rationale is that the paper shows these approximations (grey spectrum, fixed 10^4 K temperature, on-the-spot case-B) each shift columns by order-of-magnitude factors, so only the full polychromatic evolved-temperature recombination-transfer model captures the true thermal structure. Tabulate the ratios in slab_model_comparison.csv and accept the run only when computed ratios match the paper's factors within the last significant figure.

## Related tools

- **rabacus** (Provides the semi-infinite slab solvers (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt) and the HM12 photorates table used for this comparison study) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted driver language; rabacus's Fortran 90 core is wrapped in Python via Numpy f2py and invoked from Python scripts) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Units package ensuring all slab and radiation variables carry physical units during the calculation) — python-quantities/python-quantities
- **Fortran 90** (Compiled core routines that execute the slab radiative-transfer solves thousands of times faster than pure Python)

## Examples

```
import rabacus as ra; hm12 = ra.HM12_Photorates_Table(z=3.0); slab = ra.SlabHM12_evoT_B(Delta=1800, L=4.61*ra.u.kpc, bgnd=hm12)
```

## Evaluation signals

- Polychromatic-to-grey N_HI ratio ≈ 1.8 (hm12_fixT_B / grey_fixT_B)
- Evolved-temperature vs. fixed-temperature N_HI factor ≈ 11 and N_HeI factor ≈ 6.3 (hm12_evoT_B / hm12_fixT_B)
- Peak equilibrium temperature T_peak ≈ 1.6e4 K at the edge of the slab in the hm12_evoT_B model
- Recombination-radiation-transfer vs. case-B N_HI factor ≈ 3.2 (hm12_evoT_rt / hm12_evoT_B)
- Total column-density variation across all five models lies between 3 and 11, and all ratios match the paper's reported values to within the last significant figure

## Limitations

- Metal line cooling is not yet included; it is a planned future improvement.
- Non-equilibrium solvers are not yet included; only equilibrium ionization and temperature states are computed.
- Results are specific to Lyman-limit system densities; at other overdensities the order-of-magnitude column shifts may not hold.
- A Fortran 90 compiler (e.g. gfortran or ifort) must be in the executable path and the F90 environment variable set for installation to succeed.

## Evidence
