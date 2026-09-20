---
name: dwarf-galaxy-density-slope-recovery
description: Use when you have discrete stellar tracer positions and LOS velocities
  (optionally proper motions) for a spherical, equilibrium stellar system and need
  the inner dark matter density slope at R1/2 with a statistical error, e.g. to distinguish
  cuspy from cored profiles.
when_to_use_negative:
- The stellar system is not spherical or not in equilibrium — GravSphere2 is designed
  to mass-model systems of stellar tracers in equilibrium using spherical higher-order
  Jeans equations
- You only have binned/binned-summary kinematic data and need the older GravSphere
  + binulator workflow (justinread/gravsphere), not the bin-free individual-velocity
  fit
- You have very few tracers and need tight constraints — with only 100 tracers and
  no PMs, slope errors grow to ~30% (20%), so the 1000-tracer benchmark expectations
  do not apply
edam_operation: http://edamontology.org/operation_2238
tools:
- name: GravSphere2
  role: 'Core solver: bin-free fourth-order spherical Jeans mass-modeling of individual
    tracer velocities; configured via initialise_ files and gravsphere2.py'
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used for the improved likelihood sampling; stores resumable
    checkpoint chains
  repo: https://github.com/dadams42/GravSphere2
- name: Python
  role: Runtime environment (verified python 3.10.12) for running the fit and plotting
    scripts
  repo: https://github.com/dadams42/GravSphere2
- name: numpy
  role: Numerical dependency of the GravSphere2 code (verified numpy version 1.26.4)
  repo: https://github.com/dadams42/GravSphere2
- name: scipy
  role: Numerical dependency for the Jeans-equation solver (verified scipy version
    1.15.1)
  repo: https://github.com/dadams42/GravSphere2
- name: matplotlib
  role: Dependency used by plots_example_notebook.ipynb to render density, slope,
    anisotropy, dispersion, and kurtosis profiles
  repo: https://github.com/dadams42/GravSphere2
- name: corner
  role: Dependency for posterior corner plots of the recovered parameters (verified
    corner version 2.2.2)
  repo: https://github.com/dadams42/GravSphere2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:506a988ac8a5726cb34d21aa0bdd5c6fbe2a9161ba2af4c93b524712e8b1cebf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - corner
  - dynesty
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# dwarf-galaxy-density-slope-recovery

## Summary

Recover the logarithmic density slope at the half-light radius (R1/2) of a spherical dwarf galaxy mock using GravSphere2, a bin-free fourth-order spherical Jeans solver, and quantify the statistical slope error with and without proper motions. This skill operationalizes the paper's benchmark that 1000-tracer cuspy (cored) mocks yield 12% (25%) LOS-only slope errors, improving to 8% (12%) with PMs.

## When to use

Use when you have discrete stellar tracer positions and LOS velocities (optionally proper motions) for a spherical, equilibrium stellar system and need the inner dark matter density slope at R1/2 with a statistical error, e.g. to distinguish cuspy from cored profiles.

## When NOT to use

- The stellar system is not spherical or not in equilibrium — GravSphere2 is designed to mass-model systems of stellar tracers in equilibrium using spherical higher-order Jeans equations
- You only have binned/binned-summary kinematic data and need the older GravSphere + binulator workflow (justinread/gravsphere), not the bin-free individual-velocity fit
- You have very few tracers and need tight constraints — with only 100 tracers and no PMs, slope errors grow to ~30% (20%), so the 1000-tracer benchmark expectations do not apply

## Inputs

- Gaia Challenge mock dataset with 1000 tracer stars (tracer positions, line-of-sight velocities, proper motions) from the Surrey astrowiki
- initialise_ file (GravSphere2 format) specifying tracer data, likelihood, priors, and Output directory

## Outputs

- dynesty sampler checkpoint file in the Output/Sampler Chains subfolder (resumable)
- Recovered density, slope, anisotropy, dispersion, and kurtosis profile plots from plots_example_notebook.ipynb
- Four-row slope-error table (LOS cuspy, LOS cored, PM cuspy, PM cored) of the logarithmic density slope at R1/2 with statistical errors

## How to apply

Download the cuspy and cored Gaia Challenge mocks (1000 tracers, positions, LOS velocities, PMs) from the Surrey astrowiki, then write an initialise_ file per mock in the GravSphere2 format specifying tracer positions, velocities, tracer profile data, and priors. Select the galaxy by importing that initialisation file at the top of gravsphere2.py, which sets the likelihood and priors and auto-creates an Output directory with Plots and Sampler Chains subfolders. Run the nested sampling with dynesty via run_multcpu_example.py (or run_example_notebook.ipynb); dynesty writes a resumable checkpoint file into the Sampler Chains subfolder. Fit each mock twice — LOS velocities only, then re-run including proper motion velocities in the likelihood — and use plots_example_notebook.ipynb to generate density, slope, anisotropy, dispersion, and kurtosis profiles from each checkpoint. Finally, extract the logarithmic density slope at R1/2 and its posterior statistical error for each fit and tabulate the four cases (LOS cuspy, LOS cored, PM cuspy, PM cored).

## Related tools

- **GravSphere2** (Core solver: bin-free fourth-order spherical Jeans mass-modeling of individual tracer velocities; configured via initialise_ files and gravsphere2.py) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used for the improved likelihood sampling; stores resumable checkpoint chains) — https://github.com/dadams42/GravSphere2
- **Python** (Runtime environment (verified python 3.10.12) for running the fit and plotting scripts) — https://github.com/dadams42/GravSphere2
- **numpy** (Numerical dependency of the GravSphere2 code (verified numpy version 1.26.4)) — https://github.com/dadams42/GravSphere2
- **scipy** (Numerical dependency for the Jeans-equation solver (verified scipy version 1.15.1)) — https://github.com/dadams42/GravSphere2
- **matplotlib** (Dependency used by plots_example_notebook.ipynb to render density, slope, anisotropy, dispersion, and kurtosis profiles) — https://github.com/dadams42/GravSphere2
- **corner** (Dependency for posterior corner plots of the recovered parameters (verified corner version 2.2.2)) — https://github.com/dadams42/GravSphere2

## Examples

```
python run_multcpu_example.py  # dynesty fit of the initialise_ mock; checkpoint saved in Output/Sampler Chains
```

## Evaluation signals

- LOS-only fits reproduce the reported statistical slope errors: 12% for cuspy and 25% for cored mocks with 1000 tracers
- Fits including proper motions improve the slope errors to 8% (cuspy) and 12% (cored)
- Recovered logarithmic density slope at R1/2 falls within the quoted 95% confidence intervals
- Recovered mass density, anisotropy, and slope profiles are consistent within 95% confidence intervals over roughly r/R1/2 ~ 0.1 to 10
- A resumable dynesty checkpoint file exists in the Sampler Chains subfolder and reloading it reproduces the posterior

## Limitations

- Method degrades gracefully with fewer tracers (down to ten) — larger uncertainties but no induced bias; the quoted 12%/25% and 8%/12% errors apply to 1000 tracers
- With only 100 tracers and no PMs, slope errors are only ~30% (20%), weak for cuspy-vs-cored distinction
- Validation against the published errors requires the specific Gaia Challenge cuspy/cored mocks; other inputs (e.g. Fornax-like simulated galaxy) are separate examples
- The code is distinct from previous GravSphere versions that use binning and individual velocities, so legacy binulator workflows and outputs are incompatible

## Evidence

- [abstract] 1000-tracer slope errors with and without PMs: "Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data"
- [methods] Selecting the galaxy via the initialisation file: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] Resumable dynesty checkpoints: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [methods] Running scripts for sampling: "We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming"
- [methods] Plotting notebook: "We have also created a notebook (plots_example_notebook.ipynb) where we show examples of plotting density, slope, anisotropy, dispersion, and kurtosis profiles"
- [readme] What GravSphere2 solves (README): "GravSphere2 is an improved solver of the spherical higher-order Jeans equations, designed to mass-model systems of stellar tracers in equilibrium"
