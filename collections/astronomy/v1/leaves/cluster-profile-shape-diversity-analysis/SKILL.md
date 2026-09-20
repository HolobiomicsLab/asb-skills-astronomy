---
name: cluster-profile-shape-diversity-analysis
description: Use when you have recovered (nonparametric) mass profiles for multiple
  clusters and need to test whether a parametric NFW model exhausts their shape variation,
  e.g. residual analysis over the X-COP 12-cluster sample.
when_to_use_negative:
- The input profiles are themselves parametric NFW fits, so residuals against another
  NFW fit would be circular — use the nonparametric log-normal mixture reconstruction
  output instead.
- The goal is a mean/profile-average comparison with NFW or Einasto (the article already
  shows <10% average deviation); this skill targets per-cluster residual structure,
  not ensemble-averaged agreement.
- Hydrostatic equilibrium cannot be assumed, e.g. major mergers requiring nonthermal
  pressure support beyond the mild 10-20% marginalization hydromass provides.
tools:
- name: hydromass
  role: Provides the parametric NFW mass-model fitting machinery and the nonparametric
    log-normal mixture reconstruction whose output profiles are compared; Bayesian
    optimization via PyMC3
  repo: https://github.com/domeckert/hydromass
- name: Python
  role: Implementation environment for the residual computation and per-cluster summary
    table (Python 3, tested on 3.6+)
  repo: https://github.com/domeckert/hydromass
- name: numpy
  role: Array computation for radial grids, residual arithmetic, and uncertainty propagation
- name: scipy
  role: Numerical routines underpinning the fitting and residual statistics (hydromass
    dependency)
- name: astropy
  role: Cosmology/unit handling when converting radii and masses in the residual analysis
    (hydromass dependency)
- name: matplotlib
  role: Visualization of per-cluster residual curves and shape-diversity diagnostics
    (hydromass dependency)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:a0fbbb250f1749e0460edc2e89ce9bf0dccc74cbd5fa8b0f5793fa00c3e5d507
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cluster-profile-shape-diversity-analysis

## Summary

Quantifies how much shape diversity exists among individually recovered hydrostatic mass profiles of galaxy clusters beyond what an NFW parametric fit can represent, by computing per-cluster fractional residuals (M_recovered - M_NFW)/M_NFW. It operationalizes the X-COP finding that NFW describes average profiles to <10% deviation while individual profiles show residual structure.

## When to use

Use when you have recovered (nonparametric) mass profiles for multiple clusters and need to test whether a parametric NFW model exhausts their shape variation, e.g. residual analysis over the X-COP 12-cluster sample.

## When NOT to use

- The input profiles are themselves parametric NFW fits, so residuals against another NFW fit would be circular — use the nonparametric log-normal mixture reconstruction output instead.
- The goal is a mean/profile-average comparison with NFW or Einasto (the article already shows <10% average deviation); this skill targets per-cluster residual structure, not ensemble-averaged agreement.
- Hydrostatic equilibrium cannot be assumed, e.g. major mergers requiring nonthermal pressure support beyond the mild 10-20% marginalization hydromass provides.

## Inputs

- Recovered hydrostatic mass profiles for the 12 X-COP clusters (from the nonparametric log-normal mixture reconstruction)
- Radial grids and posterior uncertainty arrays for each recovered profile
- NFW parametric model specification (scale radius/concentration and normalization mass) fitted per cluster via hydromass

## Outputs

- Per-cluster table mass_profile_residuals.csv with exactly 12 rows (one per X-COP cluster)
- Fractional residual curves (M_recovered - M_NFW)/M_NFW per radial bin, with propagated uncertainties
- Per-cluster shape-diversity statistics: residual dispersion over the fitted radial range and maximum absolute deviation

## How to apply

Load the recovered hydrostatic mass profiles (from the nonparametric log-normal mixture reconstruction in hydromass) together with their radial grids and posterior uncertainties. For each cluster, fit the NFW parametric mass model — characterized by a scale radius/concentration and a normalization mass — using hydromass's parametric mass-model machinery, which supports NFW, Einasto and other popular models with automatic or custom priors. In each radial bin compute the fractional residual (M_recovered - M_NFW)/M_NFW, propagating posterior uncertainties of both the recovered profile and the NFW fit. Summarize per-cluster shape diversity via the dispersion of residuals over the fitted radial range and the maximum absolute deviation, then assemble a per-cluster table (mass_profile_residuals.csv) for flagging clusters whose residual structure indicates diversity beyond the NFW form.

## Related tools

- **hydromass** (Provides the parametric NFW mass-model fitting machinery and the nonparametric log-normal mixture reconstruction whose output profiles are compared; Bayesian optimization via PyMC3) — https://github.com/domeckert/hydromass
- **Python** (Implementation environment for the residual computation and per-cluster summary table (Python 3, tested on 3.6+)) — https://github.com/domeckert/hydromass
- **numpy** (Array computation for radial grids, residual arithmetic, and uncertainty propagation)
- **scipy** (Numerical routines underpinning the fitting and residual statistics (hydromass dependency))
- **astropy** (Cosmology/unit handling when converting radii and masses in the residual analysis (hydromass dependency))
- **matplotlib** (Visualization of per-cluster residual curves and shape-diversity diagnostics (hydromass dependency))

## Examples

```
pip install hydromass
```

## Evaluation signals

- mass_profile_residuals.csv contains exactly 12 rows, one per X-COP cluster.
- Reported fractional deviations are consistent with the finding that NFW deviations are below 10% over a wide radial range.
- Residual uncertainties are propagated from the posteriors of both the recovered profile and the NFW fit, not point estimates alone.
- Clusters whose residual dispersion or maximum absolute deviation indicates shape diversity beyond the NFW form are explicitly flagged rather than averaged away.
- Posterior predictive checks / WAIC diagnostics from hydromass support the goodness of fit of the nonparametric reconstruction feeding the residuals.

## Limitations

- The average NFW agreement (<10%) masks per-cluster shape diversity, so ensemble-level metrics alone understate residuals — per-cluster statistics are required.
- Recovered profiles assume hydrostatic equilibrium; beyond R200 the total baryon fraction slightly exceeds the cosmic value, possibly indicating 10-20% nonthermal pressure support in cluster outskirts.
- The exact hydromass version used is not stated; PyPI may not carry the latest features, so some reconstruction options may be missing.

## Evidence

- [abstract] the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [abstract] there appears to be more diversity in the shape of individual profiles than can be captured by the NFW model: "there appears to be more diversity in the shape of individual profiles than can be captured by the NFW model"
- [readme] Parametric mass model reconstruction including Navarro-Frenk-White, Einasto and several other popular mass models, with automatic or custom priors: "Parametric mass model reconstruction including Navarro-Frenk-White, Einasto and several other popular mass models, with automatic or custom priors"
- [methods] Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
