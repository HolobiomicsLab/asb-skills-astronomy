---
name: galaxy-cluster-stellar-mass-fraction-estimation
description: Use when disentangling the gravitational field of X-ray/SZ galaxy clusters
  into gas, stellar (BCG + satellite), and dark-matter contributions, and you need
  the radial stellar mass fraction or the radius where stars dominate the field (e.g.
  inside ~0.02 R500).
when_to_use_negative:
- When the goal is total mass or NFW/Einasto concentration fitting without component
  separation — the nonparametric/parametric mass reconstruction skill covers that
- When the relation between observed and baryonic acceleration (radial acceleration
  relation) is the target — this analysis explicitly shows the RAR does not hold at
  galaxy-cluster scale
- When only X-ray data without SZ pressure profiles are available and joint constraints
  are required for the outskirts
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hydromass
  role: Performs hydrostatic mass reconstruction and decomposition of the mass profile
    into baryonic and dark matter components, including non-thermal pressure modeling
    and marginalization
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Hamiltonian Monte Carlo backend for the efficient Bayesian optimization of
    the mass/thermodynamic reconstruction
- name: pyproffit
  role: Dependency of hydromass used for X-ray surface brightness profile analysis
    feeding the decomposition
- name: matplotlib
  role: Visualization of component contributions to the output mass profiles via hydromass
    plotting utilities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:47c000bd912ef79669576ea0c1aaac0b81b371969076614803bb1050f845df92
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# galaxy-cluster-stellar-mass-fraction-estimation

## Summary

Estimates the stellar share of the total gravitational field in galaxy clusters by decomposing the hydrostatic mass profile into baryonic components (gas, BCG, satellite galaxies) and dark matter using the hydromass package, which joint-fits X-ray surface brightness, spectroscopic temperature, and SZ pressure data.

## When to use

Use when disentangling the gravitational field of X-ray/SZ galaxy clusters into gas, stellar (BCG + satellite), and dark-matter contributions, and you need the radial stellar mass fraction or the radius where stars dominate the field (e.g. inside ~0.02 R500).

## When NOT to use

- When the goal is total mass or NFW/Einasto concentration fitting without component separation — the nonparametric/parametric mass reconstruction skill covers that
- When the relation between observed and baryonic acceleration (radial acceleration relation) is the target — this analysis explicitly shows the RAR does not hold at galaxy-cluster scale
- When only X-ray data without SZ pressure profiles are available and joint constraints are required for the outskirts

## Inputs

- Public X-COP data products: X-ray surface brightness profiles
- X-ray spectroscopic temperature profiles
- Sunyaev-Zeldovich (SZ) pressure profiles
- Stellar mass profiles for the BCG and satellite galaxies (subset of systems)

## Outputs

- Decomposed hydrostatic mass profiles with gas, BCG, satellite galaxy, and dark matter components
- Stellar contribution fraction as a function of radius (expected 1–2% inside R200)
- Total baryon fraction profile compared to the cosmic value at R200
- Component contribution plots and decomposition profile tables (e.g. baryon_decomposition_profiles.csv)

## How to apply

Load the public X-COP data products (X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles) within a Jupyter notebook via hydromass, then run the hydrostatic mass reconstruction with Bayesian optimization based on Hamiltonian Monte Carlo (PyMC3). For the subset of systems with stellar mass data, apply hydromass's decomposition of the hydrostatic mass profile into baryonic (gas, BCG, satellite galaxies) and dark matter components. Evaluate radial fractions to identify the radius (~0.02 R500) inside which the stellar content dominates the gravitational field, and quantify that stars account for only 1–2% of the total gravitational field inside R200. Separately compute the total baryon fraction profile, comparing it to the cosmic value at R200 and attributing any excess beyond R200 to mild nonthermal pressure support (10–20%) using hydromass's non-thermal pressure modeling and marginalization. Validate with posterior predictive checks and WAIC diagnostics, and visualize component contributions with hydromass plotting utilities.

## Related tools

- **hydromass** (Performs hydrostatic mass reconstruction and decomposition of the mass profile into baryonic and dark matter components, including non-thermal pressure modeling and marginalization) — https://github.com/domeckert/hydromass
- **pymc3** (Hamiltonian Monte Carlo backend for the efficient Bayesian optimization of the mass/thermodynamic reconstruction)
- **pyproffit** (Dependency of hydromass used for X-ray surface brightness profile analysis feeding the decomposition)
- **matplotlib** (Visualization of component contributions to the output mass profiles via hydromass plotting utilities)

## Examples

```
from hydromass import decompose; mo = decompose(data=sz_xray_data, moprofile=nfw_profile, components={'gas': gas_prof, 'stars': bcg_sat_prof})
```

## Evaluation signals

- The stellar component must dominate the gravitational field inside ~0.02 R500
- Stars must account for only 1–2% of the total gravitational field inside R200
- The total baryon fraction must reach the cosmic value at R200
- Any baryon fraction excess beyond R200 should correspond to ~10–20% nonthermal pressure support
- Posterior predictive checks and WAIC from hydromass diagnostics indicate acceptable goodness-of-fit

## Limitations

- The hydromass version was not stated; the PyPI release may lag the latest features
- Baryonic decomposition applies only to the subset of systems with stellar mass data, not all 12 X-COP clusters
- The baryon fraction excess beyond R200 is only a possible indicator of nonthermal support (10–20%), not a definitive measurement
- No discussion-section limitations or changelog were available in the source text

## Evidence

- [abstract] Stellar dominance inside ~0.02 R500: "The stellar content dominates the gravitational field inside ∼0.02 R 500 but is responsible for only 1–2% of the total gravitational field inside R 200"
- [abstract] Baryon fraction and nonthermal support: "The total baryon fraction reaches the cosmic value at R 200 and slightly exceeds it beyond this point, possibly indicating a mild level of nonthermal pressure support (10 − 20%) in cluster outskirts"
- [methods] Mass decomposition capability: "Decomposition of the hydrostatic mass profile into baryonic and dark matter components"
- [methods] Bayesian HMC optimization: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [methods] Loading X-COP data products: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
- [readme] Non-thermal pressure feature from README: "Non-thermal pressure modeling and marginalization"
