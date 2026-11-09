---
name: omega-k-dispersion-curve-extraction
description: Use when you have time series of simulated electromagnetic field fluctuations
  from plasma wave runs and need the numerical ω(k) dispersion relation per propagation
  angle to verify against linear plasma-wave theory.
when_to_use_negative:
- The input is not field-fluctuation data from a wave simulation but e.g. in situ
  spacecraft time series with gaps — Fourier decomposition over a simulation grid
  is not applicable.
- The goal is global magnetospheric dynamics (foreshock, magnetosheath) rather than
  verification of linear wave dispersion; use full Vlasiator global runs instead of
  1D wave tests.
- 'You need the physical results of Vlasiator rather than its verification: this skill
  addresses the ''verification and validation'' layer only.'
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Vlasiator
  role: Hybrid-Vlasov solver used to run the 1D low-β wave simulations whose electromagnetic
    field fluctuations are decomposed into ω(k) dispersion curves
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python toolset used to read and analyse the VLSV output files produced by
    Vlasiator, from which the spectral decomposition is performed
  repo: https://github.com/fmihpc/analysator
- name: WHAMP
  role: Reference solver (Rönnmark 1982) that independently computes the linear hot-plasma
    dispersion solutions for comparison against the simulated curves
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:dfe9477fd475d58d8fec34a87e4d7c74bc539e83945ae250bd1e1962efde764e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  - WHAMP
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# omega-k-dispersion-curve-extraction

## Summary

Extracts numerical ω(k) dispersion curves from low-β hybrid-Vlasov wave simulations by spatiotemporal Fourier decomposition of electromagnetic field fluctuations, and validates them against linear hot-plasma dispersion solutions from WHAMP.

## When to use

Use when you have time series of simulated electromagnetic field fluctuations from plasma wave runs and need the numerical ω(k) dispersion relation per propagation angle to verify against linear plasma-wave theory.

## When NOT to use

- The input is not field-fluctuation data from a wave simulation but e.g. in situ spacecraft time series with gaps — Fourier decomposition over a simulation grid is not applicable.
- The goal is global magnetospheric dynamics (foreshock, magnetosheath) rather than verification of linear wave dispersion; use full Vlasiator global runs instead of 1D wave tests.
- You need the physical results of Vlasiator rather than its verification: this skill addresses the 'verification and validation' layer only.

## Inputs

- 1D low-β Vlasiator wave-test simulation output (VLSV files) for parallel, perpendicular, and oblique propagation geometries
- Time series of simulated electromagnetic field fluctuations
- Plasma parameters (densities, magnetic field, temperatures) for the matching WHAMP run at each propagation angle

## Outputs

- dispersion_curves.csv containing ω vs k per propagation angle
- Numerical ω(k) dispersion curves from the Fourier/spectral decomposition
- WHAMP reference linear dispersion solutions for the same parameters and angles
- Maximum relative frequency error score in a structured output

## How to apply

Configure and launch 1D low-β Vlasiator wave-test runs for three propagation geometries — parallel, perpendicular, and oblique angles — using the hybrid-Vlasov solver. Extract the simulated electromagnetic field fluctuations and apply a spatiotemporal Fourier/spectral decomposition to obtain the numerical ω(k) dispersion curves for each propagation angle; Vlasiator writes VLSV output files, which can be read and analysed with the Analysator Python tools. Independently compute the reference linear hot-plasma dispersion solutions with WHAMP (Rönnmark 1982) for identical plasma parameters and angles. Compare simulated and linear-theory dispersion curves and compute the maximum relative frequency error, targeting the excellent agreement reported in Kempf et al. 2013, with the only expected deviations at high frequencies and wave numbers. Write the per-angle ω vs k data to dispersion_curves.csv and record the error score in a structured output.

## Related tools

- **Vlasiator** (Hybrid-Vlasov solver used to run the 1D low-β wave simulations whose electromagnetic field fluctuations are decomposed into ω(k) dispersion curves) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python toolset used to read and analyse the VLSV output files produced by Vlasiator, from which the spectral decomposition is performed) — https://github.com/fmihpc/analysator
- **WHAMP** (Reference solver (Rönnmark 1982) that independently computes the linear hot-plasma dispersion solutions for comparison against the simulated curves)

## Examples

```
export PTNONINTERACTIVE=1; pip install --editable ./analysator; python -c "import analysator as pt; f=pt.vlsvfile.VlsvReader('bulk.0000000.vlsv')"
```

## Evaluation signals

- dispersion_curves.csv contains ω vs k entries for all three propagation geometries (parallel, perpendicular, oblique)
- Maximum relative frequency error between simulated and WHAMP curves is small enough to count as 'excellent agreement' as in Kempf et al. 2013
- Deviations are confined to high frequencies and wave numbers; low/intermediate k curves overlap with linear theory
- WHAMP solutions were computed independently for the same plasma parameters and angles as the simulation runs
- Error score is recorded in the structured output alongside the CSV

## Limitations

- Agreement degrades at high frequencies and wave numbers, so discrepancies there are expected rather than a failure signal.
- 1D wave-test configurations limit the geometry to a single prescribed propagation angle per run (parallel, perpendicular, oblique), unlike the full multi-dimensional magnetospheric problem.
- Vlasiator's multi-dimensional, ion-kinetic approach has immense computational demands, requiring massively parallel HPC resources.

## Evidence

- [abstract] Vlasiator is the world's first global hybrid-Vlasov simulation: "Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [abstract] Verification and validation review included: "A short review of verification, validation and physical results is included"
- [readme] Analysator reads VLSV output: [evidence span withheld — anchor status: not_found]
- [other] Source card finding: agreement with WHAMP: [evidence span withheld — anchor status: not_found]
