---
name: nested-sampling-posterior-inference
description: Use when you need posterior inference (credible intervals, not point
  estimates) for GravSphere2 mass modeling of individual stellar LOS/PM velocities
  — e.g. recovering the log density slope at R1/2 from Gaia Challenge or Fornax-like
  mocks with quantified uncertainty.
when_to_use_negative:
- The system is not a spherical stellar system in equilibrium — GravSphere2 solves
  the spherical higher-order (fourth-order) Jeans equations, so rotating, triaxial,
  or tidally perturbed objects fall outside its assumptions
- Only binned dispersion/surface-brightness profiles are available (no individual
  stellar velocities) — the binning-based GravSphere + binulator or pyGravSphere (emcee)
  codes are the intended tools in that case
edam_operation: http://edamontology.org/operation_3659
tools:
- name: dynesty
  role: Nested sampler that performs the posterior inference on the bin-free higher-order
    Jeans likelihood; run at version 2.1.5 with checkpointing via run_multcpu_example.py
    or run_example_notebook.ipynb
- name: GravSphere2
  role: Framework whose gravsphere2.py imports the initialise_ file defining likelihood
    and priors, and whose Output directory stores Sampler Chains checkpoints and Plots
  repo: https://github.com/dadams42/GravSphere2
- name: corner
  role: Posterior visualization (corner plots) of nested-sampling chains; verified
    at version 2.2.2
- name: numpy
  role: Array/posterior-sample manipulation when extracting credible intervals from
    checkpoint chains; verified at version 1.26.4
- name: matplotlib
  role: Rendering density, slope, anisotropy, dispersion, and kurtosis profiles in
    plots_example_notebook.ipynb; verified at version 3.9.0
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:8c33ac91bbe91934953bc7c3dab420b0de70569827f9d2466a7f5aaa8e93841c
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# nested-sampling-posterior-inference

## Summary

Runs GravSphere2's Bayesian mass-model fits by driving the dynesty nested sampler (dynesty 2.1.5) from initialise_ files that specify likelihood and priors, yielding resumable checkpoint chains and posterior samples for density, slope, anisotropy, dispersion, and kurtosis profiles of spherical stellar systems.

## When to use

Use when you need posterior inference (credible intervals, not point estimates) for GravSphere2 mass modeling of individual stellar LOS/PM velocities — e.g. recovering the log density slope at R1/2 from Gaia Challenge or Fornax-like mocks with quantified uncertainty.

## When NOT to use

- The system is not a spherical stellar system in equilibrium — GravSphere2 solves the spherical higher-order (fourth-order) Jeans equations, so rotating, triaxial, or tidally perturbed objects fall outside its assumptions
- Only binned dispersion/surface-brightness profiles are available (no individual stellar velocities) — the binning-based GravSphere + binulator or pyGravSphere (emcee) codes are the intended tools in that case

## Inputs

- initialise_ file per galaxy (tracer positions, individual LOS/PM velocities, tracer profile data, priors, proper-motion fitting flag, Output directory path)
- Gaia Challenge mock datasets with 1000 tracer stars (cuspy and cored) from the Surrey astrowiki
- Fornax-like simulated dwarf-galaxy dataset with subsamples of 1000, 100, and 10 tracers
- Python environment: python 3.10.12, numpy 1.26.4, scipy 1.15.1, matplotlib 3.9.0, dynesty 2.1.5, corner 2.2.2

## Outputs

- Resumable dynesty checkpoint files stored in the Sampler Chains subfolders of the auto-created Output directory
- Posterior samples for the mass density, logarithmic density slope, anisotropy, dispersion, and kurtosis profiles
- 95% credible-interval bounds as a function of radius (e.g. 0.1 ≲ r/R1/2 ≲ 10) and posterior slope at R1/2 with statistical errors
- Profile plots from plots_example_notebook.ipynb saved in the Plots subfolder

## How to apply

Create an initialise_ file per galaxy containing tracer positions, individual velocities, tracer profile data, the proper-motion fitting option, and priors, then import it at the beginning of gravsphere2.py so it sets the likelihood and other fit arguments; the Output directory (with Plots and Sampler Chains subfolders) is auto-created. Launch the dynesty 2.1.5 nested sampler via run_multcpu_example.py for parallel execution on a multi-CPU machine, or run_example_notebook.ipynb for a single run. Dynesty stores a resumable checkpoint file by default in the Sampler Chains subfolder, from which all data can be retrieved and the run continued if needed. Reload checkpoints, extract posterior samples, and compute 95% credible intervals for the density, logarithmic slope, and anisotropy profiles over 0.1 ≲ r/R1/2 ≲ 10 with numpy/scipy; render profiles with plots_example_notebook.ipynb. Rationale: nested sampling was adopted specifically to improve the likelihood sampling of the bin-free higher-order Jeans fit (the code replaces the emcee-based, binned GravSphere), so unbiased posteriors even at 10 tracers depend on running the sampler un-binned over individual velocities in the verified environment (python 3.10.12, numpy 1.26.4, scipy 1.15.1, matplotlib 3.9.0, corner 2.2.2).

## Related tools

- **dynesty** (Nested sampler that performs the posterior inference on the bin-free higher-order Jeans likelihood; run at version 2.1.5 with checkpointing via run_multcpu_example.py or run_example_notebook.ipynb)
- **GravSphere2** (Framework whose gravsphere2.py imports the initialise_ file defining likelihood and priors, and whose Output directory stores Sampler Chains checkpoints and Plots) — https://github.com/dadams42/GravSphere2
- **corner** (Posterior visualization (corner plots) of nested-sampling chains; verified at version 2.2.2)
- **numpy** (Array/posterior-sample manipulation when extracting credible intervals from checkpoint chains; verified at version 1.26.4)
- **matplotlib** (Rendering density, slope, anisotropy, dispersion, and kurtosis profiles in plots_example_notebook.ipynb; verified at version 3.9.0)

## Examples

```
python run_multcpu_example.py
```

## Evaluation signals

- Recovered logarithmic density slope at R1/2 with 1000 tracers shows ~12% (cuspy) / ~25% (cored) statistical errors for LOS-only fits, improving to ~8% / ~12% when proper motions are included
- True mass density, stellar velocity anisotropy, and log slope lie inside the quoted 95% credible intervals for almost all mocks over 0.1 ≲ r/R1/2 ≲ 10
- As tracers drop to 100 (and even 10), uncertainties grow (~20–30% at 100 tracers, no PMs) but the true slope stays within the 95% credible interval — no induced bias
- A resumable checkpoint file exists in the Sampler Chains subfolder after each fit and can be reloaded to reproduce the posterior
- The run was executed under the pinned environment (dynesty 2.1.5, python 3.10.12) via run_multcpu_example.py or run_example_notebook.ipynb

## Limitations

- Posterior validity is demonstrated on mock data (Gaia Challenge cuspy/cored mocks and a Fornax-like simulated galaxy) assuming spherical, equilibrium systems; real-system coverage may differ
- The verified toolchain is version-specific (dynesty 2.1.5, numpy 1.26.4, scipy 1.15.1, python 3.10.12); other dynesty versions may change checkpoint or sampler behavior
- With very few tracers (e.g. 10) the posterior is unbiased but uncertainty is large, so cuspy-vs-cored distinctions are no longer resolved

## Evidence

- [abstract] we improved the likelihood sampling by using the nested sampler DYNESTY: "we improved the likelihood sampling by using the nested sampler DYNESTY"
- [methods] The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] checkpoint file from which all data can be retrieved: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [methods] We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming: "We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming"
- [readme] dynesty version 2.1.5: "python 3.10.12
scipy version 1.15.1
numpy version 1.26.4
matplotlib version 3.9.0
dynesty version 2.1.5"
