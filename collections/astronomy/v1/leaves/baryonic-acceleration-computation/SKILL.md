---
name: baryonic-acceleration-computation
description: Use when cluster-scale hydrostatic mass profiles and baryonic component
  profiles are available (e.g. X-COP products via hydromass) and you need g_bar(r)
  from baryonic components to compare against g_obs(r), as in a radial acceleration
  relation test.
when_to_use_negative:
- The target is spiral-galaxy kinematic RAR data (e.g. SPARC rotation curves) rather
  than cluster-scale hydrostatic reconstructions — the relation itself is what is
  being tested at cluster scale and it strongly departs there
- No disentangled baryonic components (gas, BCG, satellite galaxies) are available,
  only the total hydrostatic mass profile, so g_bar(r) cannot be constructed
- The goal is fitting parametric mass models (NFW, Einasto) rather than computing
  acceleration profiles from an existing reconstruction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: hydromass
  role: Python package used to load the public X-COP data products and obtain the
    nonparametric hydrostatic mass profiles and baryonic/dark matter decomposition
    needed to compute g_obs(r) and g_bar(r)
  repo: https://github.com/domeckert/hydromass
- name: numpy
  role: Dependency of hydromass used for numerical computation of the acceleration
    profiles
- name: scipy
  role: Dependency of hydromass used in the reconstruction and profile computation
    workflow
- name: astropy
  role: Dependency of hydromass providing constants/units for computing g_obs(r) =
    G*M_tot(<r)/r^2
- name: matplotlib
  role: Dependency of hydromass used to plot log(g_obs) against log(g_bar) with the
    RAR reference line
- name: pymc3
  role: Dependency of hydromass supplying the Hamiltonian Monte Carlo Bayesian optimization
    behind the nonparametric mass reconstruction
- name: pyproffit
  role: Dependency of hydromass for profile/surface brightness analysis feeding the
    reconstruction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:733bb1560e76c738cabc31cd4fe6a2be108d5427dc4a05aa49b7a19e331fb790
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# baryonic-acceleration-computation

## Summary

Computes baryonic gravitational acceleration g_bar(r) for galaxy clusters by summing the gravitational field contributions of gas, brightest cluster galaxy, and satellite galaxies from hydrostatic reconstruction outputs, for direct comparison against the total observed acceleration g_obs(r) = G*M_tot(<r)/r^2 (e.g. to test the radial acceleration relation at cluster scale).

## When to use

Use when cluster-scale hydrostatic mass profiles and baryonic component profiles are available (e.g. X-COP products via hydromass) and you need g_bar(r) from baryonic components to compare against g_obs(r), as in a radial acceleration relation test.

## When NOT to use

- The target is spiral-galaxy kinematic RAR data (e.g. SPARC rotation curves) rather than cluster-scale hydrostatic reconstructions — the relation itself is what is being tested at cluster scale and it strongly departs there
- No disentangled baryonic components (gas, BCG, satellite galaxies) are available, only the total hydrostatic mass profile, so g_bar(r) cannot be constructed
- The goal is fitting parametric mass models (NFW, Einasto) rather than computing acceleration profiles from an existing reconstruction

## Inputs

- Public X-COP data products for the 12 galaxy cluster sample (X-ray and Sunyaev-Zeldovich)
- Nonparametric hydrostatic reconstruction outputs: total mass profiles M_tot(<r) from hydromass
- Baryonic component profiles: gas, brightest cluster galaxy, and satellite galaxy contributions (available for a subset of systems)
- Spiral-galaxy radial acceleration relation reference curve

## Outputs

- Radial profiles g_obs(r) = G*M_tot(<r)/r^2 per cluster
- Radial profiles g_bar(r) from summed baryonic component contributions per cluster
- Log(g_obs) vs log(g_bar) scatter/mean curve plot (e.g. acceleration_relation.png) overlaid with the spiral-galaxy RAR reference line

## How to apply

Load the reconstructed hydrostatic mass profile M_tot(<r) and the disentangled baryonic components for each of the 12 X-COP clusters using the hydromass package in Python (which depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit). Compute the observed acceleration as g_obs(r) = G*M_tot(<r)/r^2 from the total hydrostatic mass profile. Build the baryonic acceleration g_bar(r) from the summed contributions of gas and, where available for the subset of systems, the brightest cluster galaxy and satellite galaxies — note stellar content dominates the field only inside ~0.02 R500 and contributes just 1-2% of the total field inside R200, so stellar components matter mostly at small radii. Plot log(g_obs) against log(g_bar) for all 12 clusters (e.g. with matplotlib), overlaying the spiral-galaxy radial acceleration relation as the reference line, and inspect whether the mean curve is a simple power law or a complex shape departing from the RAR.

## Related tools

- **hydromass** (Python package used to load the public X-COP data products and obtain the nonparametric hydrostatic mass profiles and baryonic/dark matter decomposition needed to compute g_obs(r) and g_bar(r)) — https://github.com/domeckert/hydromass
- **numpy** (Dependency of hydromass used for numerical computation of the acceleration profiles)
- **scipy** (Dependency of hydromass used in the reconstruction and profile computation workflow)
- **astropy** (Dependency of hydromass providing constants/units for computing g_obs(r) = G*M_tot(<r)/r^2)
- **matplotlib** (Dependency of hydromass used to plot log(g_obs) against log(g_bar) with the RAR reference line)
- **pymc3** (Dependency of hydromass supplying the Hamiltonian Monte Carlo Bayesian optimization behind the nonparametric mass reconstruction)
- **pyproffit** (Dependency of hydromass for profile/surface brightness analysis feeding the reconstruction)

## Examples

```
import astropy.constants as c; g_obs = (c.G * M_tot(r) / r**2).to('m s**-2'); g_bar = M_gas(r)+M_bcg(r)+M_sat(r)  # then matplotlib loglog(g_obs, g_bar)
```

## Evaluation signals

- The plotted log(g_obs)-log(g_bar) relation for the 12 X-COP clusters exhibits a complex, non-power-law shape that strongly departs from the spiral-galaxy RAR at cluster accelerations
- Stellar (BCG + satellite galaxy) contributions dominate the baryonic field only inside ~0.02 R500 and account for only 1-2% of the total field inside R200, so gas dominates g_bar at large radii
- The nonparametric reconstructed mass profiles are NFW-like, with deviations of less than 10% over a wide radial range, bounding the acceleration profiles used
- The total baryon fraction approaches the cosmic value at R200, providing a consistency check on the baryonic mass budget summed into g_bar
- g_obs(r) must be computed from the same hydrostatic reconstruction (M_tot(<r)/r^2) as the baryonic decomposition, ensuring the two are directly comparable

## Limitations

- The hydrostatic equilibrium assumption underlies the reconstruction; the article indicates a mild level of nonthermal pressure support (10-20%) in cluster outskirts, since the total baryon fraction slightly exceeds the cosmic value beyond R200
- Baryonic disentanglement (gas, brightest cluster galaxy, satellite galaxies) is available only for a subset of systems, so g_bar is not uniformly complete across the sample
- There is more diversity in the shape of individual mass profiles than can be captured by the NFW model, so single-model accelerations may misrepresent individual clusters
- The version of hydromass is not stated; the PyPI release may not include all features

## Evidence

- [abstract] The relation between observed and baryonic acceleration strongly departs from the radial acceleration relation in spiral galaxies and does not hold at galaxy-cluster scale.: "the relation between observed and baryonic acceleration exhibits a complex shape that strongly departs from the radial acceleration relation in spiral galaxies"
- [abstract] For a subset of systems, the gravitational field is disentangled into baryonic components (gas, brightest cluster galaxy, satellite galaxies) and dark matter.: "we disentangle the gravitational field into the contribution of baryonic components (gas, brightest cluster galaxy, and satellite galaxies) and that of dark matter"
- [methods] Decomposition of the hydrostatic mass profile into baryonic and dark matter components is a core capability of the hydromass package.: "Decomposition of the hydrostatic mass profile into baryonic and dark matter components"
- [methods] hydromass depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit.: "``hydromass`` depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit."
- [abstract] The stellar content dominates the gravitational field inside ∼0.02 R500 but is responsible for only 1–2% of the total gravitational field inside R200.: "The stellar content dominates the gravitational field inside ∼0.02 R 500 but is responsible for only 1–2% of the total gravitational field inside R 200"
- [methods] The user can easily load the public X-COP data products and apply the reconstruction tools directly within a Jupyter notebook.: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
