---
name: cluster-gravitational-acceleration-decomposition
description: 'Use when testing whether the spiral-galaxy radial acceleration relation
  extends to galaxy-cluster scale: you have X-ray/SZ cluster data, hydrostatic mass
  profile reconstructions, and need log(g_obs) vs log(g_bar) comparison against the
  RAR.'
when_to_use_negative:
- The analysis target is spiral galaxies themselves — the RAR is the reference relation
  in galaxy disks; this skill tests its departure at cluster scale, not its measurement
  in spirals.
- Only a parametric NFW/Einasto mass fit is available without baryonic component decomposition
  (gas, BCG, satellite galaxies), since g_bar(r) cannot be computed from the total
  field alone.
- The clusters lack the deep X-ray and Sunyaev–Zel'dovich data needed for the hydromass
  joint hydrostatic reconstruction.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: hydromass
  role: Load public X-COP data products and perform non-parametric log-normal mixture
    hydrostatic mass reconstruction and baryonic/dark-matter decomposition
  repo: https://github.com/domeckert/hydromass
- name: Python
  role: Implementation environment (Python 3, tested on 3.6+) for the reconstruction
    and acceleration-relation computation
- name: numpy
  role: Numerical computation of g_obs(r) and g_bar(r) profiles and log-transformations
- name: matplotlib
  role: Plot log(g_obs) vs log(g_bar) with the RAR reference line and save acceleration_relation.png
- name: astropy
  role: hydromass dependency used for cosmological/physical unit handling in mass
    profile reconstruction
- name: scipy
  role: hydromass dependency for the Bayesian reconstruction routines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:38bdf67a43d7cd6bc7b433c4f32c0f4160228118a3f14b88a2b661535f3cd9df
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
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cluster-gravitational-acceleration-decomposition

## Summary

Reconstructs hydrostatic total gravitational field profiles of galaxy clusters (e.g., the 12-cluster X-COP sample) and decomposes them into baryonic contributions (gas, brightest cluster galaxy, satellite galaxies) to test whether the spiral-galaxy radial acceleration relation (RAR) holds at cluster scale. The article shows the observed-vs-baryonic acceleration relation exhibits a complex, non-power-law shape that strongly departs from the RAR.

## When to use

Use when testing whether the spiral-galaxy radial acceleration relation extends to galaxy-cluster scale: you have X-ray/SZ cluster data, hydrostatic mass profile reconstructions, and need log(g_obs) vs log(g_bar) comparison against the RAR.

## When NOT to use

- The analysis target is spiral galaxies themselves — the RAR is the reference relation in galaxy disks; this skill tests its departure at cluster scale, not its measurement in spirals.
- Only a parametric NFW/Einasto mass fit is available without baryonic component decomposition (gas, BCG, satellite galaxies), since g_bar(r) cannot be computed from the total field alone.
- The clusters lack the deep X-ray and Sunyaev–Zel'dovich data needed for the hydromass joint hydrostatic reconstruction.

## Inputs

- Public X-COP data products (X-ray and Sunyaev–Zel'dovich observations for 12 galaxy clusters)
- Non-parametric hydrostatic reconstruction outputs from hydromass: total mass/gravitational field profiles M_tot(<r)
- Baryonic component contributions: gas, brightest cluster galaxy, and satellite galaxy profiles (where available)
- Spiral-galaxy radial acceleration relation reference curve

## Outputs

- g_obs(r) = G*M_tot(<r)/r^2 acceleration profiles per cluster
- g_bar(r) summed baryonic acceleration profiles per cluster
- Log-log scatter plot log(g_obs) vs log(g_bar) for all 12 clusters with RAR overlay, saved as acceleration_relation.png

## How to apply

Load the public X-COP data products with the hydromass package in Python, which reconstructs hydrostatic mass profiles by jointly modeling X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure via non-parametric log-normal mixture deprojection (PyMC3 Hamiltonian Monte Carlo). Compute g_obs(r) = G*M_tot(<r)/r^2 from each cluster's reconstructed mass profile. Compute g_bar(r) from the summed baryonic components — gas, and where available brightest cluster galaxy and satellite galaxies (stellar content dominates inside ~0.02 R500 but is only 1–2% of the field inside R200). Plot log(g_obs) against log(g_bar) for all 12 clusters in matplotlib, overlaying the spiral-galaxy RAR as the reference line, and save as acceleration_relation.png. Decision point: verify the mean curve is complex (non-power-law) and departs from the RAR at cluster accelerations; sanity-check the underlying mass profiles against NFW (deviations <10% over a wide radial range) and the baryon fraction against the cosmic value at R200.

## Related tools

- **hydromass** (Load public X-COP data products and perform non-parametric log-normal mixture hydrostatic mass reconstruction and baryonic/dark-matter decomposition) — https://github.com/domeckert/hydromass
- **Python** (Implementation environment (Python 3, tested on 3.6+) for the reconstruction and acceleration-relation computation)
- **numpy** (Numerical computation of g_obs(r) and g_bar(r) profiles and log-transformations)
- **matplotlib** (Plot log(g_obs) vs log(g_bar) with the RAR reference line and save acceleration_relation.png)
- **astropy** (hydromass dependency used for cosmological/physical unit handling in mass profile reconstruction)
- **scipy** (hydromass dependency for the Bayesian reconstruction routines)

## Examples

```
g_obs = G * M_tot / r**2; g_bar = M_gas + M_BCG + M_sat; plt.scatter(np.log10(g_bar), np.log10(g_obs)); plt.savefig('acceleration_relation.png')
```

## Evaluation signals

- The saved acceleration_relation.png shows a complex, non-power-law mean curve that strongly departs from the spiral-galaxy RAR at cluster accelerations (the acceptance check).
- Recovered mass profiles match the NFW model to better than ~10% deviation over a wide radial range before acceleration computation.
- The total baryon fraction reaches the cosmic value at R200 (mildly exceeding it beyond), confirming the baryonic decomposition is consistent.
- g_obs(r) values reproduce G*M_tot(<r)/r^2 from each cluster's hydrostatic mass profile over two decades in radius.
- Stellar components (BCG + satellites) dominate only inside ~0.02 R500 and account for 1–2% of the field inside R200, as expected.

## Limitations

- Version of hydromass not stated; the PyPI release may not contain the latest features (per the article's methods/README).
- The baryon fraction slightly exceeds the cosmic value beyond R200, possibly indicating mild nonthermal pressure support (10–20%) in cluster outskirts, which biases the purely hydrostatic g_obs(r).
- No discussion-section limitations or reproducibility details were extractable from the source content.

## Evidence

- [abstract] the relation between observed and baryonic acceleration exhibits a complex shape that strongly departs from the radial acceleration relation in spiral galaxies: "the relation between observed and baryonic acceleration exhibits a complex shape that strongly departs from the radial acceleration relation in spiral galaxies"
- [abstract] 12 clusters with deep X-ray and SZ data over two decades in radius: "we use a set of 12 galaxy clusters with available deep X-ray and Sunyaev–Zel’dovich data to constrain the shape of the gravitational field with an unprecedented level of precision over two decades in"
- [abstract] baryonic decomposition into gas, BCG, and satellite galaxies: "we disentangle the gravitational field into the contribution of baryonic components (gas, brightest cluster galaxy, and satellite galaxies) and that of dark matter"
- [methods] loading public X-COP data products in a Jupyter notebook: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
- [methods] decomposition of hydrostatic mass profile into baryonic and dark matter components: "Decomposition of the hydrostatic mass profile into baryonic and dark matter components"
- [readme] nonparametric log-normal mixture reconstruction feature: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
