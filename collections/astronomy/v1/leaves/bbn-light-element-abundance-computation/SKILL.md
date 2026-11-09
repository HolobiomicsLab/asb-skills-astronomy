---
name: bbn-light-element-abundance-computation
description: Use when the analysis requires predicted BBN mass fractions (Y_P, D/H,
  He-3, Li-7) as a function of cosmic baryon density eta_B, or a PDG-style abundance
  plot to compare against deuterium quasar observations and CMB baryon-density bands.
when_to_use_negative:
- The goal is BSM/New Physics active during BBN rather than a Standard Model abundance
  scan — use PRyMordial's beyond-SM configurations (e.g. PRyMdemoNP) instead of the
  plain SM network workflow
- Only an existing precomputed tabulation of BBN likelihoods is needed — the repo
  already ships a dedicated numerical table (PRyM_Yp_DH_cosmoMC_2023.dat) for cosmological
  MC analyses, so no re-run is required
- The analysis targets late-universe observables unrelated to the BBN era; PRyMordial
  is focused specifically on the cosmological era of Big Bang Nucleosynthesis
edam_operation: http://edamontology.org/operation_0438
edam_topics: []
tools:
- name: PRyMordial
  role: 'Core package: computes BBN light-element abundances and N_eff from the stored
    thermal background and weak rates over the eta_B grid'
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which PRyMordial is written and runs entirely; used to drive the
    SM BBN network over the eta_B grid
- name: Julia
  role: Optional SciML-kit routines via PyJulia/diffeqpy for accelerated ordinary
    differential equation solving of the BBN network
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:293e7f7b0bea65b27d836cc032c45aa638ebf473424bda1966d8a332ee649eb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bbn-light-element-abundance-computation

## Summary

Compute Big Bang Nucleosynthesis primordial light-element abundances (helium-4, deuterium, helium-3, lithium-7) by evolving the thermal background and weak neutron-to-proton conversion rates with the PRyMordial package, typically over a grid of baryon-to-photon ratios eta_B for comparison with PDG review behaviour and CMB constraints.

## When to use

Use when the analysis requires predicted BBN mass fractions (Y_P, D/H, He-3, Li-7) as a function of cosmic baryon density eta_B, or a PDG-style abundance plot to compare against deuterium quasar observations and CMB baryon-density bands.

## When NOT to use

- The goal is BSM/New Physics active during BBN rather than a Standard Model abundance scan — use PRyMordial's beyond-SM configurations (e.g. PRyMdemoNP) instead of the plain SM network workflow
- Only an existing precomputed tabulation of BBN likelihoods is needed — the repo already ships a dedicated numerical table (PRyM_Yp_DH_cosmoMC_2023.dat) for cosmological MC analyses, so no re-run is required
- The analysis targets late-universe observables unrelated to the BBN era; PRyMordial is focused specifically on the cosmological era of Big Bang Nucleosynthesis

## Inputs

- Stored thermal background evolution and weak neutron-to-proton conversion rates shipped with the PRyMordial package
- Grid of baryon-to-photon ratio eta_B values spanning the observationally allowed region
- Standard Model BBN nuclear reaction network configuration (Python 3; optional Julia SciML routines)
- PDG observational data points and CMB baryon-density constraint band for overlay

## Outputs

- Final predicted mass fractions of helium-4, deuterium, helium-3 and lithium-7 for each eta_B value
- Abundance-versus-eta_B table (e.g. abundance_curves.csv)
- PDG Figure 24.1-style reproduction plot (e.g. fig2_abundances_vs_etaB.png)
- Effective number of relativistic degrees of freedom (N_eff) from the same solver run

## How to apply

Load the stored thermal background evolution and weak neutron-to-proton conversion rates shipped with the PRyMordial package rather than recomputing them from scratch. Configure the Python 3 code to run the Standard Model BBN network over a grid of baryon-to-photon ratio eta_B values spanning the observationally allowed region. Extract the final predicted mass fractions of helium-4, deuterium, helium-3 and lithium-7 for each eta_B value from the solver output, then plot the four abundance curves against eta_B with matplotlib, overlaying PDG observational data points and the CMB baryon-density constraint band (as in PDG Figure 24.1 / the paper's Fig. 2). PRyMordial additionally evaluates the effective number of relativistic degrees of freedom including non-instantaneous decoupling effects, so N_eff output can be checked alongside abundances; optional Julia/SciML routines (via PyJulia/diffeqpy) can accelerate the solver, and NumPy, SciPy, Numba, Numdifftools and Vegas are the recommended dependencies.

## Related tools

- **PRyMordial** (Core package: computes BBN light-element abundances and N_eff from the stored thermal background and weak rates over the eta_B grid) — github.com/vallima/PRyMordial
- **Python 3** (Language in which PRyMordial is written and runs entirely; used to drive the SM BBN network over the eta_B grid)
- **Julia** (Optional SciML-kit routines via PyJulia/diffeqpy for accelerated ordinary differential equation solving of the BBN network)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The four abundance curves versus eta_B reproduce the qualitative behaviour of PDG Figure 24.1 (Y_P rising, D/H and He-3 falling steeply, Li-7 showing its characteristic plateau-plus-drop shape)
- The deuterium prediction matches quasar absorption observations and the CMB baryon-density constraint band at the observed eta_B
- Final mass fractions for helium-4, deuterium, helium-3 and lithium-7 are produced for every eta_B value in the grid with no solver failures across the observationally allowed region
- A written abundance-versus-eta_B table (abundance_curves.csv) exists with columns for all four species, and the plot file (fig2_abundances_vs_etaB.png) renders with observational overlays

## Limitations

- The code depends on NumPy and SciPy (mandatory); Numba, Numdifftools and Vegas are recommended for the best possible usage, and the optional Julia path additionally requires the Julia language and SciML, so a minimal install may run slower
- The article's discussion section contains no changelog, and no explicit limitations, artifacts, or reproducibility gaps were extractable from the provided document text
- The Standard Model workflow described here covers within-SM predictions; BSM-era New Physics investigations require the package's separate beyond-SM configuration

## Evidence

- [abstract] A package dedicated to efficient computations of observables in the Early Universe with the focus on the cosmological era of Big Bang Nucleosynthesis (BBN).: "In this work we present PRyMordial: A package dedicated to efficient computations of observables in the Early Universe with the focus on the cosmological era of Big Bang Nucleosynthesis (BBN)."
- [intro] The package is written and runs entirely with Python 3.: "The package is written and runs entirely with Python 3"
- [other] Extract the final predicted mass fractions of helium-4, deuterium, helium-3 and lithium-7 for each eta_B value from the solver output.: [evidence span withheld — anchor status: not_found]
- [readme] A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo.: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
