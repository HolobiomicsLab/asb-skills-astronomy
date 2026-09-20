---
name: thompson-agn-disk-solution
description: Use when you need a fully evolved 1D Thompson et al. 2005 AGN disk model
  around a supermassive BH (e.g. 1e6 MSun) with specified Mdot_out and Rout, e.g.
  as input to embedded-object torque or luminosity calculations.
when_to_use_negative:
- When the target disk model is Sirko & Goodman 2003 rather than Thompson et al. 2005
  — use the Sirko.SirkoAGN class (configured e.g. with le=0.5, alpha=0.01, b=0) instead
- When a multi-dimensional or time-dependent disk simulation is needed — pagn provides
  only parametric 1D steady-state profiles
- When the environment has scipy < 1.10.0 and the workflow requires the UnivariateSpline(k=3,
  s=0.005) gradient interpolation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Python module providing Thompson.ThompsonAGN with solve_disk(N=1e4) and plot_mdot()
    to solve the Thompson et al. 2005 AGN disk equations
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Array handling for radial grids and profile arrays (e.g. np.logspace grids)
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) for interpolating surface-density and
    thermal gradients over log-radius; requires scipy ver 1.10.0
- name: matplotlib
  role: Plotting key disk parameters and the accretion-rate diagnostic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:bd690c1d5b0d2a3274a874fafa58619974f80cf136686e866c6d25367755dc19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# thompson-agn-disk-solution

## Summary

Solve the Thompson et al. 2005 AGN disk equations with the pagn Python module to obtain 1D radial profiles (density, temperature, scale height, opacity) and verify sub-critical accretion, as a base for downstream tasks like migration-torque calculations.

## When to use

Use when you need a fully evolved 1D Thompson et al. 2005 AGN disk model around a supermassive BH (e.g. 1e6 MSun) with specified Mdot_out and Rout, e.g. as input to embedded-object torque or luminosity calculations.

## When NOT to use

- When the target disk model is Sirko & Goodman 2003 rather than Thompson et al. 2005 — use the Sirko.SirkoAGN class (configured e.g. with le=0.5, alpha=0.01, b=0) instead
- When a multi-dimensional or time-dependent disk simulation is needed — pagn provides only parametric 1D steady-state profiles
- When the environment has scipy < 1.10.0 and the workflow requires the UnivariateSpline(k=3, s=0.005) gradient interpolation

## Inputs

- Central black-hole mass Mbh in physical units (e.g. 1e6 * ct.MSun)
- Outer mass inflow rate Mdot_out (e.g. 1.5e-2 MSun/yr)
- Outer disk radius Rout (e.g. Rs * 1e7)
- Optional: custom opacity table in the specific format pagn requires
- Optional solver parameters: sigma, epsilon, m, xi, Rin

## Outputs

- 1D radial profiles of surface density, temperature, scale height, and opacity on the N-point grid
- Accretion-rate diagnostic plot (tho.plot_mdot()) with Mdot_out vs Mdot_c check
- Saved parameter table (txt/CSV, e.g. torque_coefficients.csv with radius, Sigma, T, torque coefficients)
- Optional bolometric luminosity (e.g. calculate_lum_sum over a wavelength grid)

## How to apply

Instantiate Thompson.ThompsonAGN with Mbh in physical units (e.g. 1e6 MSun), Mdot_out (e.g. 1.5e-2 MSun/yr) and Rout (e.g. Rs*1e7), then call tho.solve_disk(N=1e4) — 1e4 grid points are stated to be sufficient resolution for most Mbh values. Run tho.plot_mdot() as a diagnostic and confirm Mdot_out is below the critical accretion rate Mdot_c evaluated at r = Rout before trusting the solution. Optionally interpolate the resulting surface-density and thermal gradients over log-radius with scipy.interpolate.UnivariateSpline(k=3, s=0.005, ext=0), which requires scipy version 1.10.0 or later. Results (radius, surface density, temperature, torque coefficients) can be written to CSV or saved via the pagn save() helper.

## Related tools

- **pagn** (Python module providing Thompson.ThompsonAGN with solve_disk(N=1e4) and plot_mdot() to solve the Thompson et al. 2005 AGN disk equations) — https://gitlab.com/aatrani/pagn
- **numpy** (Array handling for radial grids and profile arrays (e.g. np.logspace grids))
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) for interpolating surface-density and thermal gradients over log-radius; requires scipy ver 1.10.0)
- **matplotlib** (Plotting key disk parameters and the accretion-rate diagnostic)

## Examples

```
from pagn import Thompson; tho = Thompson.ThompsonAGN(Mbh=1e6*ct.MSun, Mdot_out=1.5e-2*ct.MSun/ct.yr, Rout=Rs*1e7); tho.solve_disk(N=1e4); tho.plot_mdot()
```

## Evaluation signals

- Accretion-rate diagnostic shows Mdot_out below the critical accretion rate Mdot_c evaluated at r = Rout
- Disk solution converges with N=1e4 grid points without solver errors
- Output CSV (e.g. torque_coefficients.csv) contains finite values across the full radial grid
- Profiles are physically sensible over the radial range Rout down to Rin (positive surface density, monotonic-in-range temperature)
- Spline interpolation (k=3, s=0.005, ext=0) reproduces the tabulated Sigma(r) and thermal-gradient values without extrapolation warnings

## Limitations

- The GitHub repository (DariaGangardt/pAGN) is archived and no longer maintained; development moved to GitLab (https://gitlab.com/aatrani/pagn)
- Spline functions require scipy version 1.10.0 or later
- Custom opacity values must be provided in a specific format for ThompsonAGN/SirkoAGN to accept them
- Large exponential Planck factors cause overflow errors; values with exp_factor >= 40 are masked since their integral contribution is zero
- pagn provides only 1D parametric disk profiles, not self-consistent 2D/3D or time-dependent disk evolution

## Evidence

- [methods] ThompsonAGN instantiation parameters: "tho = Thompson.ThompsonAGN(Mbh = mbh, sigma = sigma, epsilon = epsilon, m = m, xi = xi, Mdot_out= Mdot_out, Rout = Rout, Rin = Rin, opacity =opacity)"
- [methods] Solver resolution guidance: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [results] scipy version requirement: "UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [readme] Repository status and migration: "This repository is archived and no longer maintained. The project has moved to GitLab: - **Code repository:** https://gitlab.com/aatrani/pagn"
- [readme] Purpose of the module: "`pagn` is a Python module to solve the equations from [Sirko and Goodman 2003](https://academic.oup.com/mnras/article/341/2/501/1105444) and [Thompson et al. 2005]"
