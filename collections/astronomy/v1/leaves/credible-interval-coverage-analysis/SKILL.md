---
name: credible-interval-coverage-analysis
description: Use when you need to validate a Bayesian mass-modeling fit (e.g. GravSphere2
  + dynesty) by measuring credible-interval coverage of known truth profiles from
  mock galaxy data across tracer counts.
when_to_use_negative:
- The fit uses binned data and virial shape parameters (e.g. the older justinread/gravsphere
  + binulator or AnnaGenina/pyGravSphere codes) rather than GravSphere2's bin-free
  posterior samples
- No ground-truth/mock profiles are available, so coverage of known truth cannot be
  assessed — use the intervals directly for inference instead
tools:
- name: GravSphere2
  role: Produces the posterior samples and checkpoint files whose credible intervals
    are assessed
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler (v2.1.5) generating the posteriors; run via run_multcpu_example.py
  repo: https://github.com/dadams42/GravSphere2
- name: numpy
  role: Computes 95% credible-interval bounds from posterior samples and builds the
    coverage tables (v1.26.4 verified)
- name: scipy
  role: Supports interval/percentile computation from posterior samples (v1.15.1 verified)
- name: matplotlib
  role: Plots density, slope, and anisotropy profiles as in plots_example_notebook.ipynb
    (v3.9.0 verified)
- name: corner
  role: Posterior/parameter visualization supporting the coverage checks (v2.2.2 verified)
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:02781f623e962075e29198ec431f93be7012b121afdb4c82abd833df2368a9ae
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# credible-interval-coverage-analysis

## Summary

A validation workflow for GravSphere2 mock-galaxy tests that extracts posterior samples for density, anisotropy, and log-slope profiles, computes 95% credible intervals per radius bin over 0.1 ≲ r/R1/2 ≲ 10, and checks whether known true values fall inside those intervals.

## When to use

Use when you need to validate a Bayesian mass-modeling fit (e.g. GravSphere2 + dynesty) by measuring credible-interval coverage of known truth profiles from mock galaxy data across tracer counts.

## When NOT to use

- The fit uses binned data and virial shape parameters (e.g. the older justinread/gravsphere + binulator or AnnaGenina/pyGravSphere codes) rather than GravSphere2's bin-free posterior samples
- No ground-truth/mock profiles are available, so coverage of known truth cannot be assessed — use the intervals directly for inference instead

## Inputs

- GravSphere2 initialise_ files per mock galaxy (tracer positions, velocities, tracer profile data, fitting options such as proper-motion fitting and priors)
- dynesty checkpoint files in the Sampler Chains subfolders
- Known true mass density, anisotropy, and log-slope profiles for each mock (e.g. Fornax-like simulated dwarf galaxy or Gaia Challenge mocks)

## Outputs

- mock_coverage_table.csv with columns mock ID, quantity, covered, and interval bounds
- subsampample_slope_recovery.csv-style table of slope error and 95% credible-interval width versus tracer count N = 1000, 100, 10
- Profile plots (density, slope, anisotropy) produced with plots_example_notebook.ipynb and stored in the Plots subfolder

## How to apply

Run the GravSphere2 nested-sampling fit (dynesty 2.1.5) on each mock's initialise_ file using run_multcpu_example.py for parallel execution, letting the code auto-create the Output directory with Plots and Sampler Chains subfolders. Reload the checkpoint files from the Sampler Chains subfolders and extract posterior samples for the density, stellar velocity anisotropy, and logarithmic slope profiles. For each quantity, compute the 95% credible-interval bounds as a function of radius over 0.1 ≲ r/R1/2 ≲ 10 using numpy/scipy, then mark each (mock ID, quantity, radius) entry as covered=true/false depending on whether the known true profile value lies inside the bounds. Assemble a coverage table with columns mock ID, quantity, covered, and interval bounds, saved as mock_coverage_table.csv; the same procedure applied to subsampled Fornax-like datasets (1000, 100, 10 tracers) verifies graceful degradation — errors grow (~12% cuspy / ~25–30% at 1000–100 tracers, much larger at 10) but the true slope must remain inside the 95% interval at every N.

## Related tools

- **GravSphere2** (Produces the posterior samples and checkpoint files whose credible intervals are assessed) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler (v2.1.5) generating the posteriors; run via run_multcpu_example.py) — https://github.com/dadams42/GravSphere2
- **numpy** (Computes 95% credible-interval bounds from posterior samples and builds the coverage tables (v1.26.4 verified))
- **scipy** (Supports interval/percentile computation from posterior samples (v1.15.1 verified))
- **matplotlib** (Plots density, slope, and anisotropy profiles as in plots_example_notebook.ipynb (v3.9.0 verified))
- **corner** (Posterior/parameter visualization supporting the coverage checks (v2.2.2 verified))

## Examples

```
python run_multcpu_example.py  # after importing the mock's initialise_ file at the top of gravsphere2.py; reload checkpoints from Sampler Chains to extract posteriors
```

## Evaluation signals

- mock_coverage_table.csv shows covered=true for nearly every (mock ID, quantity, radius) entry over 0.1 ≲ r/R1/2 ≲ 10
- True mass density, anisotropy, and log-slope values lie inside the 95% credible intervals for almost all mocks, matching the paper's reported recovery
- Slope error vs tracer count shows degradation without bias: ~12% (cuspy) / ~25–30% errors at 1000–100 tracers, larger at 10, with the true slope inside the 95% interval at every N
- Checkpoint files are reloadable from Sampler Chains subfolders and posteriors reproduce the same intervals on re-extraction
- Interval bounds widen monotonically as tracer count drops from 1000 → 100 → 10 in the no-proper-motion configuration

## Limitations

- Coverage is validated only for systems of stellar tracers in equilibrium (galaxies/star clusters); GravSphere2 assumes spherical symmetry via the spherical higher-order Jeans equations
- With only ~10 tracers, uncertainties become large even though no bias is induced — coverage statements are correspondingly loose
- Results depend on the specified likelihood and priors set in the initialise_ file; changing these alters the credible intervals being tested

## Evidence

- [abstract] GravSphere 2 recovered mass density, stellar velocity anisotropy, and the logarithmic slope of the mass density profile within 95% confidence intervals across almost all mocks over r/R1/2 of about 0.1 to 10: "was able to recover the mass density, stellar velocity anisotropy, and the logarithmic slope of the mass density profile within its quoted 95% confidence intervals across almost all mocks over a wide"
- [abstract] Graceful degradation finding: "As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias"
- [methods] Checkpoint storage in Sampler Chains subfolders: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [methods] Dynesty run scripts: "We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming"
- [readme] Verified dependency versions: "The code has been verified to work with: python 3.10.12 scipy version 1.15.1 numpy version 1.26.4"
