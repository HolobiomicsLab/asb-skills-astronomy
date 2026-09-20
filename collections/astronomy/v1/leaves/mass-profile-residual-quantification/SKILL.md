---
name: mass-profile-residual-quantification
description: Use when you have nonparametric hydrostatic mass profiles (e.g. from
  hydromass log-normal mixture reconstruction of X-ray/SZ cluster data) and need to
  test whether an NFW parametric fit captures each cluster's radial mass shape.
when_to_use_negative:
- The goal is the average NFW concentration–mass relation or ΛCDM comparison rather
  than per-cluster residual structure
- Only parametric (NFW/Einasto) fits exist and no nonparametrically recovered profiles
  are available to compare against
- The mass profiles come from a model other than hydrostatic-equilibrium reconstruction
  of X-ray/SZ data (e.g. lensing-only profiles), where residual interpretation differs
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: hydromass
  role: Provides the parametric NFW mass-model fitting machinery and the nonparametric
    log-normal mixture reconstruction whose recovered profiles are compared
  repo: https://github.com/domeckert/hydromass
- name: numpy
  role: Numerical computation of fractional residuals and uncertainty propagation
    over radial bins
- name: scipy
  role: Scientific computation supporting residual statistics and NFW model evaluation
    (hydromass dependency)
- name: astropy
  role: Cosmology and unit handling for radial grids and mass normalizations (hydromass
    dependency)
- name: matplotlib
  role: Visualization of per-cluster residual curves (hydromass dependency)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:48604c221605fef503a21da0a5d6175d41dff7a7a2b8217aba909db3e30ea0cb
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
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mass-profile-residual-quantification

## Summary

Quantifies how well an NFW parametric model reproduces individually recovered hydrostatic mass profiles by computing fractional residuals (M_recovered - M_NFW)/M_NFW per radial bin, exposing per-cluster shape diversity beyond the NFW form.

## When to use

Use when you have nonparametric hydrostatic mass profiles (e.g. from hydromass log-normal mixture reconstruction of X-ray/SZ cluster data) and need to test whether an NFW parametric fit captures each cluster's radial mass shape.

## When NOT to use

- The goal is the average NFW concentration–mass relation or ΛCDM comparison rather than per-cluster residual structure
- Only parametric (NFW/Einasto) fits exist and no nonparametrically recovered profiles are available to compare against
- The mass profiles come from a model other than hydrostatic-equilibrium reconstruction of X-ray/SZ data (e.g. lensing-only profiles), where residual interpretation differs

## Inputs

- 12 recovered hydrostatic mass profiles from hydromass non-parametric log-normal mixture reconstruction (per-cluster radial grids with posterior uncertainties)
- NFW parametric mass-model fits (concentration/scale radius and normalization mass) per cluster

## Outputs

- Per-cluster fractional residual curves (M_recovered - M_NFW)/M_NFW vs radius with propagated uncertainties
- Summary table mass_profile_residuals.csv with per-cluster residual dispersion and maximum absolute deviation (12 rows, one per X-COP cluster)

## How to apply

Load the 12 recovered hydrostatic mass profiles with their radial grids and posterior uncertainties, then fit an NFW model to each cluster using hydromass's parametric mass-model machinery (scale radius/concentration plus normalization mass). For each radial bin compute the fractional residual (M_recovered - M_NFW)/M_NFW, propagating posterior uncertainties from both the recovered profile and the NFW fit. Summarize per-cluster shape diversity via the dispersion of residuals over the fitted radial range and the maximum absolute deviation, then assemble all statistics into a single table saved as mass_profile_residuals.csv. Interpretation follows the paper's benchmark: NFW deviations below 10% over a wide radial range indicate an excellent average description, while residual structure in individual clusters flags diversity the NFW form cannot capture.

## Related tools

- **hydromass** (Provides the parametric NFW mass-model fitting machinery and the nonparametric log-normal mixture reconstruction whose recovered profiles are compared) — https://github.com/domeckert/hydromass
- **numpy** (Numerical computation of fractional residuals and uncertainty propagation over radial bins)
- **scipy** (Scientific computation supporting residual statistics and NFW model evaluation (hydromass dependency))
- **astropy** (Cosmology and unit handling for radial grids and mass normalizations (hydromass dependency))
- **matplotlib** (Visualization of per-cluster residual curves (hydromass dependency))

## Examples

```
res = (M_rec - M_nfw) / M_nfw; np.save('residuals.npy', res); pd.DataFrame(summary).to_csv('mass_profile_residuals.csv')
```

## Evaluation signals

- mass_profile_residuals.csv contains exactly 12 rows, one per X-COP cluster
- Reported fractional deviations are below 10% over the wide radial range, consistent with the paper's finding
- Residual uncertainties reflect propagation of both the recovered-profile posteriors and the NFW fit posteriors
- Any cluster with residual structure exceeding the <10% benchmark is flagged as showing shape diversity beyond the NFW form
- Residual curves are evaluated over the same fitted radial range for all clusters so dispersions are comparable

## Limitations

- The NFW deviations being <10% is an average statement; individual clusters can exhibit more shape diversity than NFW captures
- Residual analysis inherits hydrostatic-equilibrium assumptions; unmodeled nonthermal pressure support (10–20% in cluster outskirts per the paper's baryon-fraction finding) can bias recovered profiles
- hydromass's latest PyPI release may lag the GitHub version, so some features may be missing

## Evidence

- [abstract] NFW excellent average but per-cluster diversity: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [abstract] per-cluster shape diversity beyond NFW: "there appears to be more diversity in the shape of individual profiles than can be captured by the NFW model"
- [methods] parametric NFW machinery in hydromass: "Parametric mass model reconstruction including Navarro-Frenk-White, Einasto and several other popular mass models, with automatic or custom priors"
- [methods] nonparametric reconstruction source profiles: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
- [readme] hydromass availability and docs: "A Python package for the reconstruction of hydrostatic mass profiles and deprojection of thermodynamic profile from X-ray and/or Sunyaev-Zeldovich data."
