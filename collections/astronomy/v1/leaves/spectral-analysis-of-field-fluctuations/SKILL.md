---
name: spectral-analysis-of-field-fluctuations
description: Use when time-resolved electromagnetic field fluctuations on a spatial
  grid (e.g. from 1D Vlasiator wave-test runs) must be decomposed into ω(k) dispersion
  curves per propagation angle and validated against analytical linear-theory solutions.
when_to_use_negative:
- The input is already a dispersion curve or linear-theory solution and only needs
  plotting rather than spectral decomposition
- The goal is a global, multi-dimensional magnetospheric simulation result rather
  than a controlled 1D wave-test verification of the solver
- Fluctuations are non-stationary or strongly non-linear, where a spatiotemporal Fourier
  decomposition into ω(k) is not meaningful
tools:
- name: Vlasiator
  role: Hybrid-Vlasov solver used to produce the simulated electromagnetic field fluctuations
    in 1D low-β wave tests
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python toolset for reading and analysing Vlasiator VLSV output files prior
    to the Fourier decomposition
  repo: https://github.com/fmihpc/analysator
- name: WHAMP
  role: Independent reference solver computing linear hot-plasma dispersion solutions
    for the same parameters and angles
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:34ff1e152c451828f00806981f6b3c3434cfc6d84b3b8c92213bbddc5d8e6dad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  - WHAMP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-analysis-of-field-fluctuations

## Summary

Spatiotemporal Fourier/spectral decomposition of simulated electromagnetic field fluctuations to extract numerical ω(k) dispersion curves, used to verify a plasma solver (Vlasiator 1D low-β wave tests) against linear hot-plasma theory (WHAMP).

## When to use

Use when time-resolved electromagnetic field fluctuations on a spatial grid (e.g. from 1D Vlasiator wave-test runs) must be decomposed into ω(k) dispersion curves per propagation angle and validated against analytical linear-theory solutions.

## When NOT to use

- The input is already a dispersion curve or linear-theory solution and only needs plotting rather than spectral decomposition
- The goal is a global, multi-dimensional magnetospheric simulation result rather than a controlled 1D wave-test verification of the solver
- Fluctuations are non-stationary or strongly non-linear, where a spatiotemporal Fourier decomposition into ω(k) is not meaningful

## Inputs

- Time series of electromagnetic field fluctuations on a spatial grid from 1D low-β Vlasiator wave-test runs (VLSV output files)
- Propagation geometry specification (parallel, perpendicular, oblique angles)
- Plasma parameters used in the simulation, for the matching WHAMP reference run

## Outputs

- dispersion_curves.csv with ω vs k data per propagation angle
- Maximum relative frequency error score versus WHAMP linear theory
- Comparison of simulated vs analytical dispersion curves per angle

## How to apply

Configure 1D low-β wave-test runs of the solver for the chosen propagation geometries (parallel, perpendicular, oblique) and extract the simulated electromagnetic field fluctuations, e.g. with Analysator for VLSV output files. Apply a spatiotemporal Fourier decomposition to each geometry to obtain the numerical ω(k) dispersion curves. Independently compute reference linear hot-plasma dispersion solutions with WHAMP (Rönnmark 1982) for the same plasma parameters and angles. Compare simulated and theory curves and compute the maximum relative frequency error; for the Vlasiator low-β verification, the expected outcome is excellent agreement at parallel, perpendicular and oblique angles, with noticeable deviation only at high frequencies and wave numbers. Write the per-angle ω vs k data to dispersion_curves.csv and record the error score in structured output.

## Related tools

- **Vlasiator** (Hybrid-Vlasov solver used to produce the simulated electromagnetic field fluctuations in 1D low-β wave tests) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python toolset for reading and analysing Vlasiator VLSV output files prior to the Fourier decomposition) — https://github.com/fmihpc/analysator
- **WHAMP** (Independent reference solver computing linear hot-plasma dispersion solutions for the same parameters and angles)

## Examples

```
pip install --editable ./analysator; export PYTHONPATH=$HOME/analysator; export PTNONINTERACTIVE=1
```

## Evaluation signals

- Simulated dispersion curves overlap WHAMP solutions for parallel, perpendicular and oblique propagation angles
- Maximum relative frequency error is small, with any deviation concentrated at high frequencies and high wave numbers (as reported in Kempf et al. 2013)
- dispersion_curves.csv contains one ω vs k series per propagation angle with consistent plasma parameters across simulated and reference runs
- Each angle's extracted ω(k) is traceable to a distinct 1D wave-test configuration of the solver
- Reference WHAMP runs use identical plasma parameters to the simulation, so differences are attributable to the numerics

## Limitations

- Agreement with linear theory degrades noticeably at high frequencies and high wave numbers, so conclusions should not be drawn from that regime
- Verification applies to 1D low-β wave simulations; extending to other geometries or plasma conditions requires re-running both the simulation and WHAMP with matched parameters
- The spectral decomposition assumes the solver's field fluctuations can be meaningfully decomposed into single ω(k) modes per geometry

## Evidence

- [abstract] verification review in article: "A short review of verification, validation and physical results is included"
- [intro] models vs theories: "When tested against observations, numerical models can strengthen or invalidate existing theories and quantify the degree to which the theories have to be improved."
- [readme] analysator purpose: [evidence span withheld — anchor status: not_found]
- [readme] vlasiator description: [evidence span withheld — anchor status: external_doc]
