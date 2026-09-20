---
name: cluster-baryon-fraction-cosmic-value-comparison
description: Use when a cluster sample has joint X-ray (surface brightness, spectroscopic
  temperature) and SZ pressure profiles plus a hydrostatic mass reconstruction, and
  the goal is to evaluate the baryon fraction profile against the cosmic value at
  R200.
when_to_use_negative:
- Systems lacking joint X-ray and SZ pressure data, where hydromass's joint reconstruction
  and outskirt baryon fraction constraints are not supported.
- Analyses where the hydrostatic mass profile has not yet been reconstructed — the
  baryon fraction requires a total gravitating mass profile, not just gas density/temperature
  data.
- Questions about cluster mass profile shape (NFW vs. Einasto concentration) rather
  than the baryonic mass budget relative to the cosmic value.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hydromass
  role: Reconstructs hydrostatic mass profiles from joint X-ray/SZ data, decomposes
    the mass profile into baryonic and dark-matter components, and performs non-thermal
    pressure modeling and marginalization used to build the baryon fraction profile
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Backend for the efficient Bayesian Hamiltonian Monte Carlo optimization underlying
    the hydromass reconstruction and marginalization
- name: numpy
  role: Numerical computation dependency of hydromass used for profile arithmetic
    (radial fractions, baryon-to-total-mass ratios)
- name: astropy
  role: Dependency of hydromass for cosmological units and physical constants when
    evaluating radii (R500, R200) and the cosmic baryon fraction
- name: matplotlib
  role: Dependency of hydromass used for visualizing the baryon fraction and decomposition
    profiles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:2b4667303f1eae677a923048c4ea9a03ece26af4dda433f2e881127dd9c72539
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
  - scipy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cluster-baryon-fraction-cosmic-value-comparison

## Summary

Computes the total baryon fraction profile of X-ray/SZ galaxy clusters from hydromass hydrostatic mass reconstructions and compares it to the cosmic baryon fraction, testing whether the baryon budget reaches the cosmic value at R200 and attributing any excess beyond R200 to mild nonthermal pressure support.

## When to use

Use when a cluster sample has joint X-ray (surface brightness, spectroscopic temperature) and SZ pressure profiles plus a hydrostatic mass reconstruction, and the goal is to evaluate the baryon fraction profile against the cosmic value at R200.

## When NOT to use

- Systems lacking joint X-ray and SZ pressure data, where hydromass's joint reconstruction and outskirt baryon fraction constraints are not supported.
- Analyses where the hydrostatic mass profile has not yet been reconstructed — the baryon fraction requires a total gravitating mass profile, not just gas density/temperature data.
- Questions about cluster mass profile shape (NFW vs. Einasto concentration) rather than the baryonic mass budget relative to the cosmic value.

## Inputs

- X-ray surface brightness profiles of galaxy clusters
- X-ray spectroscopic temperature profiles
- Sunyaev-Zeldovich pressure profiles
- Hydrostatic mass profiles reconstructed with hydromass
- Cosmic (fiducial ΛCDM) baryon fraction reference value

## Outputs

- Radial baryon fraction profile f_b(r) out to and beyond R200
- Assessment of the radius where f_b reaches the cosmic value (expected at R200)
- Estimated 10-20% nonthermal pressure support contribution in cluster outskirts
- baryon_decomposition_profiles.csv with component decomposition profiles
- Plots of baryon fraction vs. radius with hydromass plotting utilities

## How to apply

Load the public X-COP data products (X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles) within a Jupyter notebook and run the hydrostatic mass reconstruction with hydromass, which performs Bayesian inference via Hamiltonian Monte Carlo through PyMC3. Use hydromass's decomposition of the hydrostatic mass profile into baryonic components (gas, BCG, satellite galaxies) and dark matter to build the cumulative baryon-to-total-mass ratio as a function of radius. Compare the resulting baryon fraction profile to the cosmic value: the expectation for X-COP-like systems is that the total baryon fraction reaches the cosmic value at R200. If the fraction slightly exceeds the cosmic value beyond R200, interpret the excess as evidence for mild nonthermal pressure support of order 10-20% in cluster outskirts, using hydromass's non-thermal pressure modeling and marginalization to quantify it. Write the resulting profiles (e.g. to baryon_decomposition_profiles.csv) and visualize them with hydromass plotting utilities.

## Related tools

- **hydromass** (Reconstructs hydrostatic mass profiles from joint X-ray/SZ data, decomposes the mass profile into baryonic and dark-matter components, and performs non-thermal pressure modeling and marginalization used to build the baryon fraction profile) — https://github.com/domeckert/hydromass
- **pymc3** (Backend for the efficient Bayesian Hamiltonian Monte Carlo optimization underlying the hydromass reconstruction and marginalization)
- **numpy** (Numerical computation dependency of hydromass used for profile arithmetic (radial fractions, baryon-to-total-mass ratios))
- **astropy** (Dependency of hydromass for cosmological units and physical constants when evaluating radii (R500, R200) and the cosmic baryon fraction)
- **matplotlib** (Dependency of hydromass used for visualizing the baryon fraction and decomposition profiles)

## Examples

```
from hydromass import decompose; from astropy.cosmology import Planck15; decompose(mymodel, cosmo=Planck15, silent=False)
```

## Evaluation signals

- The total baryon fraction profile must reach the cosmic value at R200 for the analyzed systems.
- Beyond R200 the baryon fraction may slightly exceed the cosmic value; the excess must correspond to only 10-20% nonthermal pressure support — larger excesses indicate a problem in the reconstruction or nonthermal modeling.
- The decomposition output must be consistent with the companion finding that stars contribute only 1-2% of the total gravitational field inside R200 (stellar dominance only inside ~0.02 R500).
- Posterior convergence of the hydromass reconstruction should be checked via its diagnostic tools (posterior predictive checks and WAIC).
- The written baryon_decomposition_profiles.csv must contain radial profiles with monotone cumulative gas/stellar mass components and a physically valid f_b(r) ≤ total baryon fraction in the outskirts.

## Limitations

- The interpretation of baryon fraction exceeding the cosmic value beyond R200 is only 'possibly indicating' nonthermal support — it is not a definitive measurement of nonthermal pressure.
- The latest hydromass version is not stated; the PyPI release may not include the newest features, so verify feature availability before running the nonthermal pressure marginalization.
- The method relies on the hydrostatic equilibrium assumption; systematics in the mass reconstruction propagate directly into the baryon fraction profile.
- The article's discussion/reproducibility details were unavailable (only reference metadata and no changelog), so some limitations of the comparison could not be extracted.

## Evidence

- [abstract] Baryon fraction reaching cosmic value at R200 with 10-20% nonthermal support: "The total baryon fraction reaches the cosmic value at R 200 and slightly exceeds it beyond this point, possibly indicating a mild level of nonthermal pressure support (10 − 20%) in cluster outskirts"
- [methods] hydromass data loading workflow in Jupyter: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
- [methods] Hamiltonian Monte Carlo via PyMC3: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [methods] Baryonic vs dark matter decomposition capability: "Decomposition of the hydrostatic mass profile into baryonic and dark matter components"
- [readme] Nonthermal pressure support modeling in hydromass: "Non-thermal pressure modeling and marginalization"
