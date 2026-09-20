---
name: jeans-equation-mass-modeling
description: Use when you have discrete (unbinned) stellar tracer positions and line-of-sight
  and/or proper-motion velocities for a system assumed to be in equilibrium and approximately
  spherical, and you need posterior mass density, anisotropy, or log-slope profiles
  with credible intervals.
when_to_use_negative:
- The stellar system is not in dynamical equilibrium (e.g. strongly tidally perturbed
  dwarfs), violating the Jeans-equation assumption
- You require axisymmetric or triaxial mass modeling — GravSphere2 solves the spherical
  higher-order Jeans equations only
- You only need binned kinematic profiles from a prior GravSphere/binulator or pyGravSphere
  workflow — those binning-based codes are a separate (older) approach
edam_operation: http://edamontology.org/operation_2945
tools:
- name: GravSphere2
  role: 'Main solver: bin-free fourth-order spherical Jeans fitting of individual
    tracer velocities, configured via initialise_ files and run with dynesty'
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used for the posterior inference of density, anisotropy, and
    slope profiles; version 2.1.5 verified
- name: gh_alternative
  role: Provides the positive-definite velocity PDF functions (uniform/Laplace kernel
    convolutions) used for the non-Gaussian, kurtosis-including likelihood
  repo: https://github.com/jls713/gh_alternative
- name: justinread/gravsphere
  role: Earlier binned GravSphere + binulator implementation; contrast/reference for
    the bin-free method
  repo: https://github.com/justinread/gravsphere
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:3661a03daa67667831cf75799130d3ae5eb434636254014f9d3b96f0edf8f517
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - corner
  - dynesty
  - gh_alternative
  - justinread/gravsphere
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# jeans-equation-mass-modeling

## Summary

Mass-model spherical stellar systems (dwarf galaxies, star clusters) by solving the fourth-order spherical Jeans equations bin-free: GravSphere2 fits individual LOS and proper-motion velocities with self-consistent non-Gaussian velocity PDFs (including kurtosis) and dynesty nested sampling to recover density, anisotropy, and density-slope posteriors.

## When to use

Use when you have discrete (unbinned) stellar tracer positions and line-of-sight and/or proper-motion velocities for a system assumed to be in equilibrium and approximately spherical, and you need posterior mass density, anisotropy, or log-slope profiles with credible intervals.

## When NOT to use

- The stellar system is not in dynamical equilibrium (e.g. strongly tidally perturbed dwarfs), violating the Jeans-equation assumption
- You require axisymmetric or triaxial mass modeling — GravSphere2 solves the spherical higher-order Jeans equations only
- You only need binned kinematic profiles from a prior GravSphere/binulator or pyGravSphere workflow — those binning-based codes are a separate (older) approach

## Inputs

- Discrete stellar tracer positions (projected radii) and individual line-of-sight velocities
- Optional individual stellar proper-motion velocity components
- Tracer/light profile data (e.g. surface-brightness or number-density profile, half-light radius R1/2)
- initialise_ file defining likelihood, priors, and fit options in the GravSphere2 format
- Gaia Challenge mock data (cuspy/cored) from the Surrey astrowiki, or own data in the same format

## Outputs

- dynesty checkpoint files in the Output/Sampler Chains subfolder (resumable posterior samples)
- Posterior profiles of mass density, logarithmic density slope, stellar velocity anisotropy, velocity dispersion, and kurtosis with 95% credible intervals as a function of r/R1/2
- Profile plots in the Output/Plots subfolder (via plots_example_notebook.ipynb)
- Derived summary quantities such as the log density slope at R1/2 with statistical error, or mass within R1/2

## How to apply

Create an initialise_ file per galaxy specifying tracer positions, velocities (LOS and optionally PMs), tracer number-density/light profile data, and priors; import it at the top of gravsphere2.py to set the likelihood and fit arguments and auto-create an Output directory with Plots and Sampler Chains subfolders. Run nested sampling with dynesty 2.1.5 via run_example_notebook.ipynb or run_multcpu_example.py for multi-CPU parallel execution; the run stores a resumable checkpoint file in the Sampler Chains subfolder from which all posterior data can later be retrieved. Reload checkpoints and use plots_example_notebook.ipynb to generate density, slope, anisotropy, dispersion, and kurtosis profiles with 95% credible intervals over the radial range 0.1 <= r/R1/2 <= 10. The bin-free likelihood uses positive-definite velocity PDFs (uniform kernel for negative, Laplace kernel for positive excess kurtosis) rather than Gauss-Hermite series, so higher moments are modeled without binning-induced bias; with fewer tracers the fits degrade gracefully with larger uncertainties rather than induced bias.

## Related tools

- **GravSphere2** (Main solver: bin-free fourth-order spherical Jeans fitting of individual tracer velocities, configured via initialise_ files and run with dynesty) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used for the posterior inference of density, anisotropy, and slope profiles; version 2.1.5 verified)
- **gh_alternative** (Provides the positive-definite velocity PDF functions (uniform/Laplace kernel convolutions) used for the non-Gaussian, kurtosis-including likelihood) — https://github.com/jls713/gh_alternative
- **justinread/gravsphere** (Earlier binned GravSphere + binulator implementation; contrast/reference for the bin-free method) — https://github.com/justinread/gravsphere

## Examples

```
python run_multcpu_example.py
```

## Evaluation signals

- With 1000 tracers, recovered log density slope at R1/2 should carry ~12% (cuspy) and ~25% (cored) statistical errors with LOS velocities only, improving to ~8% (cuspy) and ~12% (cored) when proper motions are included
- True mass density, anisotropy, and log slope should fall inside the quoted 95% credible intervals for almost all mocks over 0.1 <= r/R1/2 <= 10
- Downsampling tracers (even to ten) should enlarge posterior uncertainties without shifting the recovered profiles (no induced bias)
- GravSphere2 mass within R1/2 should show smaller fractional recovery error (M_inferred - M_true)/M_true than Wolf et al.-type simple mass estimators on the same mocks, including few-LOS-velocity cases
- The run must produce a resumable dynesty checkpoint in the Sampler Chains subfolder from which all posterior data can be retrieved

## Limitations

- Assumes spherical symmetry and dynamical equilibrium of the tracer system; non-equilibrium or highly flattened systems are outside the method's scope
- With few tracers the uncertainties grow substantially (e.g. ~30% (20%) slope errors at R1/2 with only 100 tracers and no PMs), though no bias is induced
- Verified dependency versions are specific (python 3.10.12, scipy 1.15.1, numpy 1.26.4, matplotlib 3.9.0, dynesty 2.1.5, corner 2.2.2); other versions are unverified

## Evidence

- [abstract] It combines individual stellar velocities from LOS and proper motion (PM) measurements to solve the Jeans equations up to fourth order, without any data binning.: "It combines individual stellar velocities from LOS and proper motion (PM) measurements to solve the Jeans equations up to fourth order, without any data binning."
- [methods] The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming: "We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming"
- [readme] It improves on previous versions of GravSphere and other methods by removing the need for binning, using self-consistent non-Gaussian velocity distribution function modeling: "It improves on previous versions of GravSphere and other methods by removing the need for binning, using self-consistent non-Gaussian velocity distribution function modeling"
- [abstract] we introduced a “bin-free” approach where we fit individual tracer velocities and positions using flexible and self-consistent probability density functions that include kurtosis: "we introduced a “bin-free” approach where we fit individual tracer velocities and positions using flexible and self-consistent probability density functions that include kurtosis"
