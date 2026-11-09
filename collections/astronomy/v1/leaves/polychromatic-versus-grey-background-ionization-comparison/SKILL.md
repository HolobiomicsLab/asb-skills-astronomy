---
name: polychromatic-versus-grey-background-ionization-comparison
description: Use when comparing ionization/temperature solutions of a Lyman-limit
  slab exposed to an isotropic background where spectral shape matters — i.e. testing
  grey vs. polychromatic (HM12) treatment of the meta-galactic background.
when_to_use_negative:
- The problem geometry is a point source or a spherical cloud rather than a semi-infinite
  slab in an isotropic background — rabacus provides separate solvers for those cases.
- Only an order-of-magnitude column-density estimate is needed and spectral shape,
  helium, and temperature treatment can all be treated approximately.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: rabacus
  role: Provides the grey_fixT_B and hm12_fixT_B case-B semi-infinite slab solvers
    and the HM12_Photorates_Table used to build the polychromatic background
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted language in which rabacus models are instantiated, run, and compared
  repo: https://github.com/galtay/rabacus
- name: Quantities
  role: Units handling for slab thickness (kpc), overdensity, and column-density variables
    in rabacus
  repo: https://github.com/python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:d47199cbd0b259e8a845f4f50fedbeb11212457e1803422608f331f4887f3c02
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# polychromatic-versus-grey-background-ionization-comparison

## Summary

Quantifies how the spectral shape of an ionizing background (polychromatic HM12 vs. a grey/hardening spectrum) alters equilibrium column densities and temperatures in a semi-infinite slab, using rabacus's grey_fixT_B and hm12_fixT_B case-B slab solvers.

## When to use

Use when comparing ionization/temperature solutions of a Lyman-limit slab exposed to an isotropic background where spectral shape matters — i.e. testing grey vs. polychromatic (HM12) treatment of the meta-galactic background.

## When NOT to use

- The problem geometry is a point source or a spherical cloud rather than a semi-infinite slab in an isotropic background — rabacus provides separate solvers for those cases.
- Only an order-of-magnitude column-density estimate is needed and spectral shape, helium, and temperature treatment can all be treated approximately.

## Inputs

- HM12 (Haardt & Madau 2012) meta-galactic radiation background evaluated at a chosen redshift (e.g. z=3.0) via ra.HM12_Photorates_Table
- Semi-infinite slab definition: overdensity Δ=1800, thickness L=4.61 kpc, isotropic incident background

## Outputs

- N_HI, N_HeI, N_HeII column densities for the grey (grey_fixT_B) and polychromatic (hm12_fixT_B) fixed-temperature case-B slab models
- Polychromatic/grey column-density ratios (target N_HI ratio ≈ 1.8)
- Rows appended to slab_model_comparison.csv in the five-model tabulation

## How to apply

Instantiate an ra.HM12_Photorates_Table at redshift 3.0 to obtain the Haardt & Madau 2012 meta-galactic background. Define the semi-infinite slab problem (overdensity Δ=1800, thickness L=4.61 kpc) and run the grey-spectrum fixed-temperature case-B model (grey_fixT_B), recording N_HI, N_HeI, N_HeII and peak temperature. Run the polychromatic fixed-temperature case-B model (hm12_fixT_B) on the identical slab and take the ratio of column densities. Accept the comparison when the polychromatic/grey N_HI ratio reproduces the reported value of ≈1.8 within the last significant figure; differences of this magnitude justify carrying full spectral shape in cosmological ionization calculations.

## Related tools

- **rabacus** (Provides the grey_fixT_B and hm12_fixT_B case-B semi-infinite slab solvers and the HM12_Photorates_Table used to build the polychromatic background) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted language in which rabacus models are instantiated, run, and compared) — https://github.com/galtay/rabacus
- **Quantities** (Units handling for slab thickness (kpc), overdensity, and column-density variables in rabacus) — https://github.com/python-quantities/python-quantities

## Examples

```
import rabacus as ra; hm12 = ra.HM12_Photorates_Table(z=3.0); slab_grey = ra.GreySlab(...); slab_poly = ra.Hm12Slab(...)
```

## Evaluation signals

- Polychromatic/grey N_HI ratio equals ≈1.8 to the last significant figure
- The fixed-temperature case-B models both run on the identical slab definition (Δ=1800, L=4.61 kpc) so the only differing variable is spectral shape
- Grey/polychromatic comparison rows appear in slab_model_comparison.csv alongside the other three models with total column-density variation of 3–11
- Import test succeeds: 'import rabacus as ra' in an ipython session after installation

## Limitations

- Metal line cooling is not included in rabacus, so equilibrium temperatures omit that channel.
- Grey spectrum results are approximate: spectral shape, helium, and temperature treatment can change slab column densities by an order of magnitude.
- Rabacus treats only equilibrium (not non-equilibrium) ionization and temperature states.

## Evidence
