---
name: baryonic-dark-matter-mass-decomposition
description: Use when hydrostatic mass profiles exist from X-ray (surface brightness
  + spectroscopic temperature) and SZ pressure data and the goal is radial gravitational-field
  contributions of baryons vs dark matter, e.g. stellar dominance radii or baryon
  fraction profiles out to R200.
when_to_use_negative:
- The analysis goal is the shape/comparison of total mass profiles (NFW vs Einasto)
  or the radial acceleration relation rather than the baryonic/dark-matter split of
  the field.
- No spectroscopic temperature or SZ pressure data are available, so hydrostatic equilibrium
  cannot be jointly constrained over the radial range needed.
- The system lacks BCG stellar mass and satellite galaxy information, so the stellar
  contribution to the gravitational field cannot be separated from the gas component.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: hydromass
  role: Performs the hydrostatic mass reconstruction and the decomposition of the
    hydrostatic mass profile into baryonic and dark matter components, including non-thermal
    pressure modeling and marginalization and plotting utilities
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Backend for the efficient Bayesian optimization based on Hamiltonian Monte
    Carlo used in the reconstruction
- name: pyproffit
  role: Dependency of hydromass for the X-ray/SZ profile analysis pipeline
- name: numpy
  role: Numerical dependency of hydromass used when loading data products and evaluating
    radial fractions in the notebook
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:7613828d07a6c8660286f7fa09db6084fe8bba814099aa75c3ae729159f5771c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/domeckert/hydromass
  tools:
  - Python
  - astropy
  - hydromass
  - matplotlib
  - numpy
  - pymc3
  - pyproffit
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# baryonic-dark-matter-mass-decomposition

## Summary

Disentangles a cluster's hydrostatic mass profile into baryonic components (gas, brightest cluster galaxy, satellite galaxies) and dark matter as a function of radius, using hydromass's decomposition of hydrostatic mass profiles reconstructed jointly from X-ray and SZ data.

## When to use

Use when hydrostatic mass profiles exist from X-ray (surface brightness + spectroscopic temperature) and SZ pressure data and the goal is radial gravitational-field contributions of baryons vs dark matter, e.g. stellar dominance radii or baryon fraction profiles out to R200.

## When NOT to use

- The analysis goal is the shape/comparison of total mass profiles (NFW vs Einasto) or the radial acceleration relation rather than the baryonic/dark-matter split of the field.
- No spectroscopic temperature or SZ pressure data are available, so hydrostatic equilibrium cannot be jointly constrained over the radial range needed.
- The system lacks BCG stellar mass and satellite galaxy information, so the stellar contribution to the gravitational field cannot be separated from the gas component.

## Inputs

- X-ray surface brightness profiles for the cluster
- X-ray spectroscopic temperature profiles
- Sunyaev-Zeldovich pressure profiles
- BCG stellar mass profile and satellite galaxy (member galaxy) distribution for systems where the field is decomposed
- hydromass package installed (depends on numpy, scipy, astropy, matplotlib, pymc3, pyproffit)

## Outputs

- Decomposed hydrostatic mass profiles giving radial contributions of gas, BCG, satellite galaxies, and dark matter
- Stellar mass fraction of the gravitational field vs radius (dominance radius ~0.02 R500; 1-2% inside R200)
- Total baryon fraction profile compared to the cosmic value at R200 and beyond
- Nonthermal pressure support estimate (10-20%) in cluster outskirts
- baryon_decomposition_profiles.csv

## How to apply

Load the public X-COP data products (X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles) within a Jupyter notebook and run the hydromass hydrostatic mass reconstruction, which performs efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3. For the subset of systems with stellar data, apply hydromass's decomposition of the hydrostatic mass profile into baryonic components (gas, brightest cluster galaxy, satellite galaxies) and dark matter, then evaluate radial fractions: find the radius (~0.02 R500) inside which stars dominate the field and the stellar contribution inside R200 (expected 1–2%). Compute the total baryon fraction profile and compare it to the cosmic value at and beyond R200, attributing any excess to mild nonthermal pressure support (10–20%) via hydromass's non-thermal pressure modeling and marginalization. Visualize component contributions with hydromass's plotting utilities and write decomposition profiles to CSV.

## Related tools

- **hydromass** (Performs the hydrostatic mass reconstruction and the decomposition of the hydrostatic mass profile into baryonic and dark matter components, including non-thermal pressure modeling and marginalization and plotting utilities) — https://github.com/domeckert/hydromass
- **pymc3** (Backend for the efficient Bayesian optimization based on Hamiltonian Monte Carlo used in the reconstruction)
- **pyproffit** (Dependency of hydromass for the X-ray/SZ profile analysis pipeline)
- **numpy** (Numerical dependency of hydromass used when loading data products and evaluating radial fractions in the notebook)

## Examples

```
from hydromass import decompose; mdec = decompose(myt, mbcg=bcg_profile, msat=sat_profile)  # baryonic/DM split of the hydrostatic mass profile
```

## Evaluation signals

- The stellar component must dominate the gravitational field inside ~0.02 R500.
- The stellar contribution to the total gravitational field inside R200 must fall in the 1-2% range.
- The total baryon fraction must reach the cosmic value at R200.
- Any excess of the baryon fraction beyond R200 must correspond to a mild 10-20% nonthermal pressure support level when marginalizing over the non-thermal pressure model.
- Posterior diagnostics via hydromass's posterior predictive checks and WAIC should indicate acceptable goodness-of-fit before trusting the decomposition.

## Limitations

- The PyPI release may not be the latest version, so some features may be missing; the stable release is judged by the developer.
- Decomposition into baryonic components was applied only to a subset of X-COP systems with the required stellar data (BCG and satellite galaxy profiles), not the full sample.
- The excess baryon fraction beyond R200 only 'possibly indicates' nonthermal pressure support (10-20%); the interpretation is not a direct measurement.

## Evidence

- [abstract] Stellar dominance radius and R200 fraction: "The stellar content dominates the gravitational field inside ∼0.02 R 500 but is responsible for only 1–2% of the total gravitational field inside R 200"
- [abstract] Baryon fraction vs cosmic value and nonthermal support: "The total baryon fraction reaches the cosmic value at R 200 and slightly exceeds it beyond this point, possibly indicating a mild level of nonthermal pressure support (10 − 20%) in cluster outskirts"
- [methods] Decomposition capability in hydromass: "Decomposition of the hydrostatic mass profile into baryonic and dark matter components"
- [methods] HMC-based Bayesian reconstruction: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [methods] Loading public X-COP data products in a notebook: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
- [readme] Non-thermal pressure modeling and plotting in README: "Non-thermal pressure modeling and marginalization"
