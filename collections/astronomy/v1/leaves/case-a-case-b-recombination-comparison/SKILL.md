---
name: case-a-case-b-recombination-comparison
description: Use when comparing analytic radiative-transfer solutions for a semi-infinite
  slab (e.g. overdensity Δ=1800, L=4.61 kpc at z=3.0) where you must quantify how
  case-A/case-B recombination, temperature evolution, and spectral shape affect N_HI
  and N_HeI.
when_to_use_negative:
- The problem is not an analytic equilibrium slab/sphere geometry — rabacus is designed
  for well-posed simplified geometries (semi-infinite slab in isotropic background,
  spherical gas with central point source, spherical gas in isotropic background),
  not full 3D hydrodynamic radiative transfer.
- Time-dependent (non-equilibrium) ionization/thermal evolution is required — rabacus
  computes equilibrium states only; non-equilibrium solvers are a planned future improvement.
- Metal line cooling dominates the thermal balance — metal line cooling is not yet
  included in rabacus.
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: rabacus
  role: Runs the five slab solutions (case-A/case-B and recombination-radiation-transfer
    variants) and computes equilibrium ionization and temperature states
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: High-level driver language used to instantiate the HM12 table, configure the
    slab problem, and tabulate ratios into CSV
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Provides units for all rabacus variables (e.g. kpc slab thickness, K temperatures)
  repo: https://github.com/python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:075e84764c667dda163ee4eb598000e44daae015875d93029877953041a84de2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# case-a-case-b-recombination-comparison

## Summary

A rabacus-based skill for quantifying how the treatment of recombination radiation (case-A vs case-B, and full recombination-radiation transfer) plus fixed vs self-consistent equilibrium temperatures changes H I and He column densities in Lyman-limit slab models exposed to the HM12 UV background.

## When to use

Use when comparing analytic radiative-transfer solutions for a semi-infinite slab (e.g. overdensity Δ=1800, L=4.61 kpc at z=3.0) where you must quantify how case-A/case-B recombination, temperature evolution, and spectral shape affect N_HI and N_HeI.

## When NOT to use

- The problem is not an analytic equilibrium slab/sphere geometry — rabacus is designed for well-posed simplified geometries (semi-infinite slab in isotropic background, spherical gas with central point source, spherical gas in isotropic background), not full 3D hydrodynamic radiative transfer.
- Time-dependent (non-equilibrium) ionization/thermal evolution is required — rabacus computes equilibrium states only; non-equilibrium solvers are a planned future improvement.
- Metal line cooling dominates the thermal balance — metal line cooling is not yet included in rabacus.

## Inputs

- HM12 (Haardt & Madau 2012) meta-galactic radiation background photorates, evaluated at z=3.0 via rabacus
- Slab physical specification: overdensity Δ=1800, semi-infinite slab thickness L=4.61 kpc, isotropic incident ionizing background

## Outputs

- Per-model column densities N_HI, N_HeI, N_HeII and peak temperature for the five slab models (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt)
- Comparison ratios (grey/polychromatic, evoT/fixT, rt/case-B) and slab_model_comparison.csv summarizing all five models

## How to apply

Instantiate the HM12 photorates table at z=3.0 (ra.HM12_Photorates_Table) and set up the semi-infinite slab with overdensity Δ=1800 and thickness L=4.61 kpc illuminated by the isotropic HM12 ionizing background. Run the five-model series: grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, and hm12_evoT_rt, recording N_HI, N_HeI, N_HeII and peak temperature for each. Validate the polychromatic/grey N_HI ratio ≈1.8, the evoT/fixT factors (N_HI ≈11, N_HeI ≈6.3), the rt-vs-case-B N_HI factor ≈3.2, and T_peak ≈1.6e4 K, allowing differences only in the last significant figure. Tabulate all five models, confirming total column-density variation across them lies between 3 and 11, and write slab_model_comparison.csv. The rationale is that for Lyman-limit-system densities, temperature evolution, helium treatment, and recombination-radiation transfer can each shift column densities by up to an order of magnitude, so each physical approximation must be isolated by paired comparisons.

## Related tools

- **rabacus** (Runs the five slab solutions (case-A/case-B and recombination-radiation-transfer variants) and computes equilibrium ionization and temperature states) — https://bitbucket.org/galtay/rabacus
- **Python** (High-level driver language used to instantiate the HM12 table, configure the slab problem, and tabulate ratios into CSV) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Provides units for all rabacus variables (e.g. kpc slab thickness, K temperatures)) — https://github.com/python-quantities/python-quantities

## Examples

```
import rabacus as ra; hmr = ra.HM12_Photorates_Table(z=3.0); Slb = ra.SlabBG(Hm12=hmr, Delta=1800, L=4.61*2**0.5, fixT=1.0e4*2**0.5)
```

## Evaluation signals

- Polychromatic-to-grey N_HI ratio ≈ 1.8 (agreement to last significant figure).
- Self-consistent temperature (hm12_evoT_B vs hm12_fixT_B): N_HI factor ≈ 11 and N_HeI factor ≈ 6.3.
- Full recombination-radiation transfer vs case-B (hm12_evoT_rt vs hm12_evoT_B): N_HI factor ≈ 3.2.
- Peak equilibrium temperature of the evoT slab ≈ 1.6 × 10^4 K, elevated above the fixed 10^4 K case.
- Total column-density variation across all five models lies between 3 and 11, and slab_model_comparison.csv is produced with all five rows.

## Limitations

- Metal line cooling is not yet included (planned future improvement), which affects the equilibrium temperature solution.
- Non-equilibrium solvers are not yet included (planned future improvement); only equilibrium states are computed.
- Column-density factors are approximate validation targets; only differences in the last significant figure are tolerated.

## Evidence

- [discussion] For slabs exposed to isotropic backgrounds at Lyman limit system densities, temperature evolution, helium, and recombination radiation treatment can change hydrogen and helium column densities by an order of magnitude.: "For slabs exposed to isotropic backgrounds at Lyman limit system densities, these processes can change hydrogen and helium column densities by an order of magnitude."
- [abstract] Self-consistent equilibrium temperatures and recombination radiation transfer are included in the solutions.: "The solutions include a treatment of both hydrogen and helium, a self-consistent calculation of equilibrium temperatures, and the transfer of recombination radiation"
- [methods] Install test: import rabacus and compute He I photo-heating rate at z=3.0 from the meta-galactic background.: [evidence span withheld — anchor status: not_found]
- [abstract] rabacus supports the semi-infinite slab in isotropic background geometry used by this skill.: "1) a semi-infinite slab gas distribution in a homogeneous isotropic background, 2) a spherically symmetric gas distribution with a point source at the center"
- [intro] Quantities package provides units for all variables in rabacus.: "all variables in rabacus have units through use of the Quantities Python package"
