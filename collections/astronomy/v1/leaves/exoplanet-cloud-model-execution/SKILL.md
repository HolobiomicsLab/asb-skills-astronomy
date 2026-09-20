---
name: exoplanet-cloud-model-execution
description: Use when you need to execute the ExoLyn codebase to compute an exoplanet
  atmospheric cloud structure (e.g., a hot-Jupiter layered Mg-silicate/Fe cloud) or
  benchmark its runtime, starting from its parameter and chemistry text files.
when_to_use_negative:
- You need radiative-transfer spectra generation itself rather than the cloud structure
  — ExoLyn is a standalone cloud model meant to be combined with optool (solid opacities)
  and petitRADTRANS (transmission/emission spectra), so chain those tools instead
  of expecting ExoLyn to produce spectra directly.
- Your problem requires a multi-dimensional or kinetically detailed cloud-formation
  model; ExoLyn is a deliberately simplified 1D model balancing physical consistency
  with computational efficiency.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ExoLyn
  role: The 1D multispecies cloud model being executed to solve cloud-particle and
    vapor transport equations
  repo: github.com/helonghuangastro/exolyn
- name: Fortran (sol.f90)
  role: Custom compiled linear-system solver that accelerates the relaxation solve
    relative to the numpy solver
  repo: github.com/helonghuangastro/exolyn
- name: Python 3
  role: 'Driver scripts: parameters.py (parameters.txt), read.py (reactions.txt, gibbs.txt),
    init.py (domain/equilibria), relaxation.py (solve), draw.py (plots)'
  repo: github.com/helonghuangastro/exolyn
- name: bash/build.sh
  role: Compilation entry point for the Fortran solver before any run
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Downstream tool ExoLyn output can be combined with to calculate solid opacities
- name: petitRADTRANS
  role: Downstream tool ExoLyn output can be combined with to generate transmission
    or emission spectra
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:a567b5a230860ea04df24c2d269e95ed085ecee50d5974ded672f454325c9530
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Fortran (sol.f90)
  - Python
  - Python 3
  - bash
  - bash/build.sh
  - optool
  - petitRADTRANS
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# exoplanet-cloud-model-execution

## Summary

Run the ExoLyn 1D multispecies cloud model (Huang, Ormel & Min 2024, A&A) end-to-end — Fortran linear-solver compilation, parameter/chemistry loading, initialization, and a relaxation solve — to reproduce cloud-structure results such as the layered hot-Jupiter magnesiumsilicate-over-iron clouds in seconds.

## When to use

Use when you need to execute the ExoLyn codebase to compute an exoplanet atmospheric cloud structure (e.g., a hot-Jupiter layered Mg-silicate/Fe cloud) or benchmark its runtime, starting from its parameter and chemistry text files.

## When NOT to use

- You need radiative-transfer spectra generation itself rather than the cloud structure — ExoLyn is a standalone cloud model meant to be combined with optool (solid opacities) and petitRADTRANS (transmission/emission spectra), so chain those tools instead of expecting ExoLyn to produce spectra directly.
- Your problem requires a multi-dimensional or kinetically detailed cloud-formation model; ExoLyn is a deliberately simplified 1D model balancing physical consistency with computational efficiency.

## Inputs

- parameters.txt (python-like configuration file defining planet/atmosphere parameters, reactions, and all gas and solid species)
- reactions.txt (chemical reaction data read via read.py)
- gibbs.txt (Gibbs formation energy data for all molecules, e.g., from janaf.nist.gov)

## Outputs

- Relaxed 1D cloud-structure solution (transport equation of cloud particles and vapor under self-consistently calculated cloud condensation rates)
- runtime. with measured wall-clock solve duration in seconds (when benchmarking)
- Plots of the cloud structure via draw.py (optional)

## How to apply

Clone the ExoLyn repository and first compile the custom Fortran linear-system solver (sol.f90) with `bash build.sh`, since it is much faster than the numpy solver. Load the configuration by having parameters.py read parameters.txt (a python-like .txt file where parameters can be defined via expressions referencing earlier lines) and load the chemical data (reactions.txt and gibbs.txt) via read.py. Initialize the atmospheric domain size and initial equilibrium concentrations with init.py, which finds an appropriate domain before computing initial equilibria. Then run one full relaxation solve with `python3 relaxation.py`, optionally wrapped in a wall-clock timer (e.g., Python time.perf_counter or /usr/bin/time) if measuring runtime. To model new chemistry, edit reactions in parameters.txt (# marks comments), add Gibbs formation energy data for all involved molecules in gibbs.txt (available from janaf.nist.gov), and list all gas and solid species in parameters.txt in any order.

## Related tools

- **ExoLyn** (The 1D multispecies cloud model being executed to solve cloud-particle and vapor transport equations) — github.com/helonghuangastro/exolyn
- **Fortran (sol.f90)** (Custom compiled linear-system solver that accelerates the relaxation solve relative to the numpy solver) — github.com/helonghuangastro/exolyn
- **Python 3** (Driver scripts: parameters.py (parameters.txt), read.py (reactions.txt, gibbs.txt), init.py (domain/equilibria), relaxation.py (solve), draw.py (plots)) — github.com/helonghuangastro/exolyn
- **bash/build.sh** (Compilation entry point for the Fortran solver before any run) — github.com/helonghuangastro/exolyn
- **optool** (Downstream tool ExoLyn output can be combined with to calculate solid opacities)
- **petitRADTRANS** (Downstream tool ExoLyn output can be combined with to generate transmission or emission spectra)

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- The relaxation solve completes without errors after `bash build.sh` compilation and `python3 relaxation.py` execution.
- Recorded runtime_seconds is on the order of a few seconds, consistent with the paper's claim that results 'can be obtained with ExoLyn in only a few seconds'.
- The resulting hot-Jupiter cloud structure is layered with a magnesiumsilicate-dominated cloud on top of an iron cloud, matching the paper's baseline finding.
- Every molecule in the chosen reactions has Gibbs formation energy data in gibbs.txt, and all gas and solid species appear in parameters.txt (order does not matter).
- Solutions agree with the more complex cloud-formation models cited in the paper (consistency check).

## Limitations

- The Fortran part (sol.f90) must be compiled via build.sh before running; skipping this step prevents execution (the numpy fallback exists but is slower).
- Custom reactions require manually sourcing Gibbs formation energies (e.g., from janaf.nist.gov) and listing all species in parameters.txt; missing entries will break the run.
- ExoLyn is a 1D model balancing physical consistency with efficiency — it is not a full multi-dimensional cloud microphysics model.

## Evidence

- [abstract] ExoLyn results in seconds: "This finding is consistent with more complex cloud formation models but can be obtained with ExoLyn in only a few seconds"
- [abstract] layered hot-Jupiter clouds: "the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud"
- [readme] compile and run: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [readme] run command: "After compilation, you can run the code by `python3 relaxation.py`"
- [readme] fast Fortran solver: "**sol.f90** is a custom fortran program to numerically solve linear systems. It's much faster than the numpy solver."
- [readme] add reactions workflow: "Please also implement gibbs formation energy data for all the molecules involved in the reaction in `gibbs.txt`. You may find these data on janaf.nist.gov."
