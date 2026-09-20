---
name: new-physics-energy-injection-modeling
description: Use when scanning a new-physics particle species thermalized with the
  plasma or neutrino bath over a mass grid and predicting N_eff and light-element
  abundances for BBN exclusion/allowance regions.
when_to_use_negative:
- The goal is a pure Standard Model BBN analysis with no thermalized new species —
  PRyMordial's SM workflow suffices and the NP flags stay off
- The new-physics scenario involves decaying particles or non-thermal injection rather
  than a species thermalized with the plasma or neutrino bath, which is the setup
  PRyMordial's Fig. 4 scan models
tools:
- name: PRyMordial
  role: 'Core tool: coupled thermal-history ODE evolution with NP_nu_flag/NP_e_flag,
    non-instantaneous decoupling treatment, BBN abundance computation, and Fig. 4-style
    band rendering'
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the package is written and runs entirely, used to configure
    and drive the NP mass-grid scan
- name: Julia
  role: Optional SciML/diffeqpy routines for ODE solving during the coupled thermal-history
    evolution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:ddc2565a9034a9596b457665d77f906dfab2c467097e7aa8e237cec0afbc8c6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - PRyMordial
  - Python 3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# new-physics-energy-injection-modeling

## Summary

Models the impact of a new relativistic species X (real/complex scalar, massive vector, Majorana or Dirac fermion) thermalized with the electromagnetic plasma or the neutrino bath on BBN observables using PRyMordial, capturing expansion-rate changes, neutrino-to-photon temperature-ratio evolution, and non-instantaneous decoupling effects.

## When to use

Use when scanning a new-physics particle species thermalized with the plasma or neutrino bath over a mass grid and predicting N_eff and light-element abundances for BBN exclusion/allowance regions.

## When NOT to use

- The goal is a pure Standard Model BBN analysis with no thermalized new species — PRyMordial's SM workflow suffices and the NP flags stay off
- The new-physics scenario involves decaying particles or non-thermal injection rather than a species thermalized with the plasma or neutrino bath, which is the setup PRyMordial's Fig. 4 scan models

## Inputs

- PRyMordial run configuration with NP_nu_flag and NP_e_flag enabled (Python 3, optionally Julia/SciML ODE backends)
- Species X specification: quantum structure (real/complex scalar, massive vector, Majorana or Dirac fermion) and mass grid
- Coupled thermal-history settings specifying which bath (plasma vs. neutrino) X thermalizes with

## Outputs

- N_eff and neutron-to-proton conversion history per mass point
- Light-element abundance bands versus species mass across the full scan
- fig4_np_abundance_bands.png — abundance-band plot with overlaid observational exclusion region

## How to apply

Enable NP_nu_flag (energy exchange with the neutrino bath) and NP_e_flag (energy exchange with the electromagnetic plasma) in the PRyMordial run configuration, then select the spin/quantum structure of species X — scalar, vector, Majorana, or Dirac — and define the mass grid over which X thermalizes. Evolve the coupled thermal history with PRyMordial's ODE solver, deliberately retaining non-instantaneous decoupling effects rather than assuming instantaneous neutrino decoupling, to obtain N_eff and the neutron-to-proton conversion history at each mass point. Run the BBN abundance computation per mass point to assemble light-element yields, then render the Fig. 4 style abundance-band plot with the observational exclusion region overlaid on the same mass axis (fig4_np_abundance_bands.png). The primary NP effects to track are the change in the cosmological expansion rate via the time–temperature relation and the impact on the neutrino-to-photon temperature ratio.

## Related tools

- **PRyMordial** (Core tool: coupled thermal-history ODE evolution with NP_nu_flag/NP_e_flag, non-instantaneous decoupling treatment, BBN abundance computation, and Fig. 4-style band rendering) — github.com/vallima/PRyMordial
- **Python 3** (Language in which the package is written and runs entirely, used to configure and drive the NP mass-grid scan)
- **Julia** (Optional SciML/diffeqpy routines for ODE solving during the coupled thermal-history evolution)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Reproduced abundance bands and overlaid exclusion boundary show visual agreement with the corresponding Fig. 4 panel of the published paper (band positions and exclusion boundary)
- N_eff values reflect non-instantaneous decoupling effects rather than an instantaneous-decoupling approximation
- Abundance bands vary smoothly and monotonically as expected across the species mass grid
- Predicted light-element yields remain within the range of the published percent-level deuterium and helium-4 measurements in allowed regions

## Limitations

- Best usage requires optional dependencies: Numba and Numdifftools are recommended, and the Julia/sciML (diffeqpy) path is optional — the code can avoid Numba and Numdifftools but the defaults use Vegas only sparingly
- SciPy and NumPy are mandatory dependencies; the quick example runPRyM_julia.py is the entry point rather than a turnkey NP scan
- The article's discussion section was unavailable in the source index, so discussion-level caveats on reproducibility gaps could not be extracted

## Evidence

- [abstract] PRyMordial offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: [evidence span withheld — anchor status: not_found]
- [intro] PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN.: "PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN."
- [methods] PRyMordial's Fig. 4 analysis of a new species thermalized with the plasma or neutrino bath identifies as primary NP effects a change in the cosmological expansion rate.: [evidence span withheld — anchor status: not_found]
- [readme] README examples and dependencies: PRyMdemoSM/PRyMdemoNP notebooks and mandatory NumPy/SciPy with recommended Numba.: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
- [readme] README quick-start entry point.: "To run a quick example, open the terminal and once in PRyMordial folder just type:"
