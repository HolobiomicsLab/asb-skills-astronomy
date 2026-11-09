---
name: jeans-equation-bin-free-fitting
description: Use when you have discrete stellar tracer positions and velocities (LOS-only
  or LOS+PM) for an equilibrium spherical system and need an unbiased recovery of
  the density profile, its logarithmic slope at R1/2, anisotropy, dispersion, or kurtosis
  without binning.
when_to_use_negative:
- The system is not a stellar-tracer system in equilibrium or is not well approximated
  as spherical — GravSphere2 solves the spherical higher-order Jeans equations
- You need binned kinematic profiles or Virial Shape Parameters — use the older GravSphere
  + binulator (justinread/gravsphere) instead
- You want a non-parametric MCMC Jeans analysis with pre-binned dispersion data —
  pyGravSphere (AnnaGenina/pyGravSphere) is the fitting alternative for that workflow
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: GravSphere2
  role: Core bin-free higher-order Jeans mass-modeling code; gravsphere2.py specifies
    likelihood and priors, initialise_ files set data and output directory
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used for posterior inference, run via run_multcpu_example.py
    (parallel) or run_example_notebook.ipynb
  repo: https://github.com/dadams42/GravSphere2
- name: gh_alternative
  role: Provides positive-definite alternative probability density functions (uniform/Laplace
    kernels) used for the non-Gaussian velocity PDF modeling with kurtosis
  repo: https://github.com/jls713/gh_alternative
- name: matplotlib
  role: Plotting density, slope, anisotropy, dispersion, and kurtosis profiles (verified
    version 3.9.0)
- name: numpy
  role: Numerical handling of tracer data and posterior statistics (verified version
    1.26.4)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:eba6b2257bbff74b64fbdaebe116cbc5725f143d5705b7c43be1e4598a1566ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - dynesty
  - gh_alternative
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# jeans-equation-bin-free-fitting

## Summary

Mass-model spherical stellar systems (dwarf galaxies, star clusters) with GravSphere2's bin-free spherical higher-order Jeans solver, fitting individual line-of-sight and proper-motion velocities up to fourth order with non-Gaussian velocity PDFs, sampled via dynesty.

## When to use

Use when you have discrete stellar tracer positions and velocities (LOS-only or LOS+PM) for an equilibrium spherical system and need an unbiased recovery of the density profile, its logarithmic slope at R1/2, anisotropy, dispersion, or kurtosis without binning.

## When NOT to use

- The system is not a stellar-tracer system in equilibrium or is not well approximated as spherical — GravSphere2 solves the spherical higher-order Jeans equations
- You need binned kinematic profiles or Virial Shape Parameters — use the older GravSphere + binulator (justinread/gravsphere) instead
- You want a non-parametric MCMC Jeans analysis with pre-binned dispersion data — pyGravSphere (AnnaGenina/pyGravSphere) is the fitting alternative for that workflow

## Inputs

- Discrete stellar tracer positions and line-of-sight velocities (optionally proper motions) for a galaxy or star cluster, e.g. a Fornax-like simulated dwarf galaxy dataset or Gaia Challenge mock data
- initialise_ file specifying galaxy data, fit configuration (LOS-only vs PM), priors, and Output directory

## Outputs

- Posterior samples for mass density profile, logarithmic slope at R1/2, stellar velocity anisotropy, dispersion, and kurtosis
- Checkpoint file in the Sampler Chains subfolder (resumable)
- Profile plots (density, slope, anisotropy, dispersion, kurtosis) from plots_example_notebook.ipynb

## How to apply

Create an initialise_ file per galaxy specifying tracer data, the no-proper-motion or with-PM configuration, and the Output directory; import that file at the top of gravsphere2.py, which sets the bin-free likelihood and priors including kurtosis terms from the fourth-order Jeans equations. Run the dynesty nested sampler (2.1.5) via run_multcpu_example.py for parallel execution or run_example_notebook.ipynb; checkpoint files are saved by default in per-galaxy Sampler Chains subfolders and can be reloaded to resume. Extract posteriors for the logarithmic density slope at R1/2 and report medians with 95% credible intervals; plot density, slope, anisotropy, dispersion, and kurtosis profiles with plots_example_notebook.ipynb. Expect ~12% (cuspy) / 25% (cored) slope errors at 1000 LOS-only tracers, ~30%/20% at 100, and graceful degradation (wider intervals, no bias) down to 10 tracers.

## Related tools

- **GravSphere2** (Core bin-free higher-order Jeans mass-modeling code; gravsphere2.py specifies likelihood and priors, initialise_ files set data and output directory) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used for posterior inference, run via run_multcpu_example.py (parallel) or run_example_notebook.ipynb) — https://github.com/dadams42/GravSphere2
- **gh_alternative** (Provides positive-definite alternative probability density functions (uniform/Laplace kernels) used for the non-Gaussian velocity PDF modeling with kurtosis) — https://github.com/jls713/gh_alternative
- **matplotlib** (Plotting density, slope, anisotropy, dispersion, and kurtosis profiles (verified version 3.9.0))
- **numpy** (Numerical handling of tracer data and posterior statistics (verified version 1.26.4))

## Examples

```
python run_multcpu_example.py  # after importing the initialise_ file in gravsphere2.py; checkpoint saved in Sampler Chains subfolder
```

## Evaluation signals

- Recovered mass density, anisotropy, and logarithmic slope at R1/2 fall within the quoted 95% confidence intervals across mocks over r/R1/2 ≈ 0.1–10
- With 1000 LOS-only tracers, slope statistical errors are ~12% (cuspy) and ~25% (cored); with 100 tracers, ~30% (cuspy) and ~20% (cored)
- Lowering tracer count to 10 widens uncertainties but does not bias the recovered slope — true slope stays inside the 95% credible interval at every N
- Checkpoint file is written to the Sampler Chains subfolder and can be reloaded to resume the dynesty run
- Dependencies match verified versions: python 3.10.12, numpy 1.26.4, scipy 1.15.1, matplotlib 3.9.0, dynesty 2.1.5, corner 2.2.2

## Limitations

- Spherical assumption and equilibrium requirement: designed for systems of stellar tracers in equilibrium, such as galaxies and star clusters
- LOS-only fits yield larger slope uncertainties (~12%/25% at 1000 tracers) than fits including proper motions (8%/12%)
- The public release is distinct from previous GravSphere versions that use higher-order Jeans or individual velocities — results are not directly comparable to binned GravSphere/binulator runs

## Evidence

- [abstract] It combines individual stellar velocities from LOS and proper motion (PM) measurements to solve the Jeans equations up to fourth order, without any data binning.: "It combines individual stellar velocities from LOS and proper motion (PM) measurements to solve the Jeans equations up to fourth order, without any data binning."
- [methods] The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] Checkpoint files stored in Sampler Chains subfolders: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [abstract] Dynesty nested sampler improved likelihood sampling: "we improved the likelihood sampling by using the nested sampler DYNESTY"
- [abstract] Graceful degradation with no bias down to ten tracers: "As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias"
