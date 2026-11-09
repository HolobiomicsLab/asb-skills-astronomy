---
name: bin-free-stellar-kinematics-fitting
description: Use when mass-modeling a spherical equilibrium stellar system (dwarf
  galaxy, star cluster) from discrete tracer positions and individual LOS and/or PM
  velocities, where binning would discard information and you need the density slope
  at R1/2.
when_to_use_negative:
- The system is rotating, triaxial, or out of dynamical equilibrium — the method solves
  spherical higher-order Jeans equations and assumes equilibrium tracers
- You only hold binned/pre-processed profiles (e.g. the pyGravSphere or binulator
  hdf5 workflow) rather than individual stellar velocities, so the bin-free likelihood
  cannot be used
- Gauss-Hermite velocity models are specifically required — gh_alternative kernels
  replace them because Gauss-Hermite series are not guaranteed positive-definite
edam_operation: http://edamontology.org/operation_3659
tools:
- name: GravSphere2
  role: Bin-free higher-order (fourth-order) spherical Jeans solver; configured per
    galaxy via an initialise_ file imported into gravsphere2.py
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler that explores the Jeans-model posterior; checkpoint files saved
    in Sampler Chains subfolders
- name: gh_alternative
  role: Positive-definite velocity PDF functions (uniform/Laplace kernel convolved
    with a Gaussian) used to model weak non-Gaussianity and kurtosis
  repo: https://github.com/jls713/gh_alternative
- name: corner
  role: Corner plots of the fitted parameter posteriors (verified with corner 2.2.2)
- name: matplotlib
  role: Rendering density, slope, anisotropy, dispersion, and kurtosis profile plots
    (verified with matplotlib 3.9.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:0ac9903415afcb515d0f1d1ede7ba40692c64f2cbe63da68c4b103a25d5c1bb3
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
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bin-free-stellar-kinematics-fitting

## Summary

Fit individual (unbinned) stellar positions and LOS/proper-motion velocities with GravSphere2's self-consistent, positive-definite non-Gaussian velocity PDFs including kurtosis, solving the fourth-order Jeans equations. This recovers the mass density profile, anisotropy, and the logarithmic density slope at R1/2 with quantified statistical errors.

## When to use

Use when mass-modeling a spherical equilibrium stellar system (dwarf galaxy, star cluster) from discrete tracer positions and individual LOS and/or PM velocities, where binning would discard information and you need the density slope at R1/2.

## When NOT to use

- The system is rotating, triaxial, or out of dynamical equilibrium — the method solves spherical higher-order Jeans equations and assumes equilibrium tracers
- You only hold binned/pre-processed profiles (e.g. the pyGravSphere or binulator hdf5 workflow) rather than individual stellar velocities, so the bin-free likelihood cannot be used
- Gauss-Hermite velocity models are specifically required — gh_alternative kernels replace them because Gauss-Hermite series are not guaranteed positive-definite

## Inputs

- Catalog of individual tracer star positions and line-of-sight velocities (e.g. 1000-tracer Gaia Challenge mock, downloadable from the Surrey astrowiki)
- Optional proper-motion velocity components per tracer to include in the likelihood
- Tracer profile data (e.g. Plummer surface-brightness/number-density parameters and per-star weights)
- initialise_ file specifying data paths, likelihood options, and priors (same format as the shipped examples)

## Outputs

- dynesty sampler checkpoint file (resumable) in the Output/Sampler Chains subfolder
- Posterior for the mass density profile and its logarithmic slope at R1/2 with statistical errors (e.g. slope_error_table.csv rows for LOS cuspy/cored and PM cuspy/cored)
- Recovered anisotropy β(r), velocity dispersion, and kurtosis profiles with 95% confidence intervals, plus Plots from plots_example_notebook.ipynb

## How to apply

Create an initialise_ file in the GravSphere2 example format holding tracer positions, velocities (LOS, and PMs if available), tracer profile data (e.g. Plummer parameters, weights), and priors, then import it at the top of gravsphere2.py so it sets the likelihood, priors, and the auto-created Output directory with Plots and Sampler Chains subfolders. Run the nested sampler dynesty (verified with dynesty 2.1.5 on Python 3.10.12) via run_multcpu_example.py or run_example_notebook.ipynb; the run stores a resumable checkpoint file in Sampler Chains. Use the gh_alternative probability functions — a kernel (uniform for negative, Laplace for positive excess kurtosis) convolved with a Gaussian — instead of Gauss-Hermite series, since they are guaranteed positive-definite. Re-run with proper-motion velocities added to the likelihood to tighten constraints (slope errors at 1000 tracers improve from 12%/25% to 8%/12% for cuspy/cored mocks). Generate density, slope, anisotropy, dispersion, and kurtosis profiles from the checkpoint with plots_example_notebook.ipynb and check the recovered slope at R1/2 against mock truth within the quoted 95% confidence intervals.

## Related tools

- **GravSphere2** (Bin-free higher-order (fourth-order) spherical Jeans solver; configured per galaxy via an initialise_ file imported into gravsphere2.py) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler that explores the Jeans-model posterior; checkpoint files saved in Sampler Chains subfolders)
- **gh_alternative** (Positive-definite velocity PDF functions (uniform/Laplace kernel convolved with a Gaussian) used to model weak non-Gaussianity and kurtosis) — https://github.com/jls713/gh_alternative
- **corner** (Corner plots of the fitted parameter posteriors (verified with corner 2.2.2))
- **matplotlib** (Rendering density, slope, anisotropy, dispersion, and kurtosis profile plots (verified with matplotlib 3.9.0))

## Examples

```
python run_multcpu_example.py
```

## Evaluation signals

- Recovered logarithmic density slope at R1/2 falls within the quoted 95% confidence intervals of the mock truth across r/R1/2 ≈ 0.1–10
- Slope statistical errors at 1000 tracers reproduce 12% (cuspy) and 25% (cored) for LOS-only fits, improving to 8% (cuspy) and 12% (cored) when PMs are included
- A resumable dynesty checkpoint file exists in the Sampler Chains subfolder and can be reloaded to regenerate results
- Velocity PDF evaluations stay positive everywhere (guaranteed by the gh_alternative kernel+Gaussian convolution, unlike Gauss-Hermite)
- Re-running with fewer tracers (down to ten) yields larger uncertainties but no induced bias in the recovered profiles

## Limitations

- Assumes spherical symmetry and dynamical equilibrium of the tracer population; results are validated only on spherical mocks (Gaia Challenge and a Fornax-like simulated galaxy)
- Statistical errors are sizable for cored mocks with LOS data only (25% at 1000 tracers), so cusp/core distinction is marginal without PMs
- Degrades gracefully but with much larger uncertainties as tracers drop below ~1000 (≈20–30% slope errors at 100 tracers; still unbiased at ten tracers)

## Evidence

- [abstract] we introduced a “bin-free” approach where we fit individual tracer velocities and positions using flexible and self-consistent probability density functions that include kurtosis: "we introduced a “bin-free” approach where we fit individual tracer velocities and positions using flexible and self-consistent probability density functions that include kurtosis"
- [abstract] recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data, enabling us to make a distinction between: "recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data, enabling us to make a distinction between"
- [methods] The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders): "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [readme] using self-consistent non-Gaussian velocity distribution function modeling and performing a fully general treatment of higher moments, solving the fourth-order Jeans equations: "using self-consistent non-Gaussian velocity distribution function modeling and performing a fully general treatment of higher moments, solving the fourth-order Jeans equations"
