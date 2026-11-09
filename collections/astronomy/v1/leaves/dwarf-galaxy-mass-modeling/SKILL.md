---
name: dwarf-galaxy-mass-modeling
description: Use when you have discrete stellar tracer positions and velocities (LOS-only
  or LOS+PM) for a dwarf galaxy or star cluster assumed to be in equilibrium, and
  you need an unbinned posterior for the log density slope at R1/2 (e.g. cuspy vs
  cored discrimination).
when_to_use_negative:
- The stellar system is not in dynamical equilibrium (the method assumes mass-modeling
  of systems of stellar tracers in equilibrium)
- The system is strongly non-spherical or you require binned dispersion/Virial Shape
  Parameter analysis — use the older binning-based GravSphere + binulator instead
- You need a quick mass estimate without a full posterior — the paper notes GravSphere2
  outperforms simple mass estimators, so simple estimators are the fallback, not this
  skill
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: GravSphere2
  role: 'Core solver: bin-free higher-order (fourth-order) Jeans fit of individual
    tracer velocities; configured via initialise_ files and gravsphere2.py likelihood/priors'
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used for likelihood sampling of the fit (version 2.1.5), run
    in parallel via run_multcpu_example.py with checkpointing
- name: numpy
  role: Numerical processing of posterior samples and tabulation of slope errors vs
    tracer count (e.g. subsample_slope_recovery.csv)
- name: gh_alternative (Sanders & Evans 2020 PDFs)
  role: Positive-definite near-Gaussian probability functions (uniform/Laplace kernels)
    used for the non-Gaussian velocity PDF modeling with kurtosis
  repo: https://github.com/jls713/gh_alternative
- name: matplotlib
  role: Plotting density, slope, anisotropy, dispersion, and kurtosis profiles (version
    3.9.0)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:540339fedbf187056cd7dc6ceeff5244a5681c2193c3f527d2fad93400a2285d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - dynesty
  - gh_alternative (Sanders & Evans 2020 PDFs)
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# dwarf-galaxy-mass-modeling

## Summary

Recover the mass density profile, anisotropy, and logarithmic density slope at R1/2 of a spherical stellar system by fitting individual (unbinned) line-of-sight and proper-motion velocities with GravSphere2, which solves the fourth-order Jeans equations and samples the posterior with dynesty.

## When to use

Use when you have discrete stellar tracer positions and velocities (LOS-only or LOS+PM) for a dwarf galaxy or star cluster assumed to be in equilibrium, and you need an unbinned posterior for the log density slope at R1/2 (e.g. cuspy vs cored discrimination).

## When NOT to use

- The stellar system is not in dynamical equilibrium (the method assumes mass-modeling of systems of stellar tracers in equilibrium)
- The system is strongly non-spherical or you require binned dispersion/Virial Shape Parameter analysis — use the older binning-based GravSphere + binulator instead
- You need a quick mass estimate without a full posterior — the paper notes GravSphere2 outperforms simple mass estimators, so simple estimators are the fallback, not this skill

## Inputs

- Discrete stellar tracer positions and line-of-sight velocities (optionally proper motions) for one galaxy, e.g. Gaia Challenge mock data or a Fornax-like simulated dwarf galaxy
- initialise_ file specifying the galaxy data, no-PM or PM configuration, output directory, likelihood and priors
- Optional custom tracer positions/velocities, tracer profiles, and priors in the initialise_ format

## Outputs

- Posterior samples for the mass density profile, stellar velocity anisotropy, and logarithmic density slope at R1/2
- Median slope, statistical error, and 95% credible-interval width at R1/2
- Checkpoint file saved by default in the Sampler Chains subfolder for later reload/resume
- Plots of density, slope, anisotropy, dispersion, and kurtosis profiles via plots_example_notebook.ipynb

## How to apply

Prepare an initialise_ file for the galaxy (same format as the Fornax-like simulated galaxy and Gaia Challenge mock examples), selecting the no-proper-motion or LOS+PM configuration and setting the Output directory so per-galaxy folders with Plots and Sampler Chains subfolders are auto-created. Import this initialisation file at the top of gravsphere2.py, where the bin-free likelihood (individual tracer velocities and positions fitted with flexible positive-definite PDFs including kurtosis) and priors are specified. Run the dynesty nested sampler (version 2.1.5) via run_multcpu_example.py for parallel execution; a checkpoint file is stored by default in the Sampler Chains subfolder and can be reloaded to resume or retrieve all data. Reload the checkpoint, extract the posterior for the logarithmic density slope at R1/2, and quote the median and 95% credible interval. Validation targets from the paper: with 1000 tracers without PMs expect ~12% (cuspy) / ~25% (cored) statistical errors, improving to ~8% / ~12% with PMs, and the truth should lie within the 95% interval at nearly all radii (~0.1–10 r/R1/2).

## Related tools

- **GravSphere2** (Core solver: bin-free higher-order (fourth-order) Jeans fit of individual tracer velocities; configured via initialise_ files and gravsphere2.py likelihood/priors) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used for likelihood sampling of the fit (version 2.1.5), run in parallel via run_multcpu_example.py with checkpointing)
- **numpy** (Numerical processing of posterior samples and tabulation of slope errors vs tracer count (e.g. subsample_slope_recovery.csv))
- **gh_alternative (Sanders & Evans 2020 PDFs)** (Positive-definite near-Gaussian probability functions (uniform/Laplace kernels) used for the non-Gaussian velocity PDF modeling with kurtosis) — https://github.com/jls713/gh_alternative
- **matplotlib** (Plotting density, slope, anisotropy, dispersion, and kurtosis profiles (version 3.9.0))

## Examples

```
python run_multcpu_example.py
```

## Evaluation signals

- With 1000 tracers and no PMs, the recovered log density slope at R1/2 has ~12% (cuspy) / ~25% (cored) statistical error; with PMs ~8% / ~12% — larger deviations indicate a misconfigured fit
- With 100 tracers and no PMs, slopes are still recovered with ~30% (cuspy) / ~20% (cored) errors
- Down to 10 tracers the method degrades gracefully: uncertainties grow but no bias is induced — the true slope must lie within the 95% credible interval at every tracer count
- Density, anisotropy, and slope profiles are recovered within 95% confidence intervals across ~0.1–10 r/R1/2 for almost all mocks
- Checkpoint file exists in the Sampler Chains subfolder and reloading it reproduces/resumes the dynesty run

## Limitations

- Requires spherical symmetry and dynamical equilibrium of the tracer system
- Statistical errors inflate substantially as tracer count drops (e.g. very large uncertainties at 10 tracers), even though no bias is induced
- LOS-only fits are less precise than LOS+PM fits (12%/25% vs 8%/12% at 1000 tracers); distinguishing cusps from cores may be harder without PMs
- Verified only with specific package versions (python 3.10.12, scipy 1.15.1, numpy 1.26.4, matplotlib 3.9.0, dynesty 2.1.5, corner 2.2.2)

## Evidence

- [abstract] Abstract finding: graceful degradation: "As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias"
- [abstract] Abstract finding: quantitative slope errors: "Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data"
- [methods] Methods: configuration of the fit: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] Methods: checkpointing: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [readme] README: verified dependency versions: "python 3.10.12
scipy version 1.15.1
numpy version 1.26.4
matplotlib version 3.9.0
dynesty version 2.1.5
corner version 2.2.2"
- [readme] README: scope of the code: "designed to mass-model systems of stellar tracers in equilibrium such as galaxies and star clusters with minimal bias and maximizing information content"
