---
name: log-radius-spline-differentiation
description: Use when migration-torque or thermodynamic gradient calculations need
  continuous dSigmadR, dTdR, dPdR functions evaluated at arbitrary migrator radii
  from discrete AGN-disk profiles (e.g. pAGN Sirko & Goodman or Thompson solved disks).
when_to_use_negative:
- The disk profiles are not yet solved or the arrays are noisy/unconverged — spline
  derivatives will inherit spurious structure; solve_disk must run first
- You need gradients from a disk model not represented as smooth 1D radial profiles
  (e.g. multidimensional simulations), where log-log cubic splines with s=0.005 are
  not validated
- scipy version is below 1.10.0, since the spline construction requires it
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SciPy
  role: Provides UnivariateSpline (k=3, s=0.005, ext=0) used to build and differentiate
    the log-radius interpolants; requires version >= 1.10.0
- name: pagn
  role: Supplies the solved Sirko & Goodman and Thompson AGN-disk profiles (surface
    density, temperature, pressure vs radius) that the splines are fit to
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Used for the log10 transformations of radius and profile arrays prior to spline
    fitting
- name: Python
  role: Implementation language for the gradient/torque-coefficient module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:12922a97a5e6c3008ae7946b25c77be2f638cf6ec0b6e9b2247fed257d2f139f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# log-radius-spline-differentiation

## Summary

Fits cubic-spline interpolants with scipy's UnivariateSpline (k=3, s=0.005, ext=0) of log10 surface density, log10 temperature, and log10 pressure against log10 radius over a solved pAGN AGN-disk model, then differentiates the splines to obtain the dSigmadR, dTdR, and dPdR gradient functions needed by migration torque-coefficient formulas.

## When to use

Use when migration-torque or thermodynamic gradient calculations need continuous dSigmadR, dTdR, dPdR functions evaluated at arbitrary migrator radii from discrete AGN-disk profiles (e.g. pAGN Sirko & Goodman or Thompson solved disks).

## When NOT to use

- The disk profiles are not yet solved or the arrays are noisy/unconverged — spline derivatives will inherit spurious structure; solve_disk must run first
- You need gradients from a disk model not represented as smooth 1D radial profiles (e.g. multidimensional simulations), where log-log cubic splines with s=0.005 are not validated
- scipy version is below 1.10.0, since the spline construction requires it

## Inputs

- Solved pAGN disk object (SirkoAGN or ThompsonAGN) with radial arrays of surface density, temperature, and pressure from solve_disk(N=1e4)
- Radial grid (radius array) over which log10(R) is taken

## Outputs

- UnivariateSpline interpolant objects for log10(Sigma), log10(T), log10(P) versus log10(R)
- Derivative functions dSigmadR, dTdR, dPdR evaluable at any migrator orbital radius
- Local disk gradients consumed by adiabatic/isothermal/thermal gamma torque-coefficient expressions

## How to apply

First solve the AGN disk with pAGN (e.g. Sirko.SirkoAGN or Thompson.ThompsonAGN, then obj.solve_disk(N=1e4)) to obtain discrete radial profiles of surface density (Sigma = 2*rho*h), temperature, and pressure. Compute log10 of the radius grid and of each profile, then fit scipy.interpolate.UnivariateSpline with k=3, s=0.005, ext=0 to each log-log series; scipy >= 1.10.0 is required. Differentiate each spline object to produce dSigmadR, dTdR, and dPdR as functions of radius. Feed these local gradients into the adiabatic, isothermal, and thermal gamma expressions of Grishin et al. 2023, which combine the Jimenez & Masset 2017, Masset 2017, and Paardekooper et al. 2010 torque-coefficient formulas. The smoothing factor s=0.005 is a deliberate choice for the pAGN use-case grids, so re-tune it if the disk resolution or model changes.

## Related tools

- **SciPy** (Provides UnivariateSpline (k=3, s=0.005, ext=0) used to build and differentiate the log-radius interpolants; requires version >= 1.10.0)
- **pagn** (Supplies the solved Sirko & Goodman and Thompson AGN-disk profiles (surface density, temperature, pressure vs radius) that the splines are fit to) — https://gitlab.com/aatrani/pagn
- **numpy** (Used for the log10 transformations of radius and profile arrays prior to spline fitting)
- **Python** (Implementation language for the gradient/torque-coefficient module)

## Examples

```
from scipy.interpolate import UnivariateSpline; Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)
```

## Evaluation signals

- The standalone torque-coefficient script executes end-to-end without errors (script_runs check)
- Computed torque values (gamma_thermal, CI_jm17_iso, CI_jm17_tot, CI_p10, CL) match the published pAGN use-case reference outputs for the 50 solar-mass migrator around a 1e6 solar-mass SMBH with q = 5e-6
- Splines are constructed with exactly k=3, s=0.005, ext=0 and ext=0 guarantees extrapolation outside the fitted radius range returns the nearest value rather than NaN
- Differentiated gradients are finite and smooth across the disk radial extent for both the Sirko & Goodman (le=0.5, alpha=0.01, b=0) and Thompson (Rout=Rs*1e7, Mdot_out=1.5e-2 MSun/yr) configurations

## Limitations

- The spline fitting requires scipy version 1.10.0 or newer; older versions will not reproduce the interpolants
- No numeric tolerance is stated for the torque validation — agreement is judged qualitatively against the published pAGN use-case outputs, so exact numerical equivalence cannot be guaranteed across environments
- The repository DariaGangardt/pAGN on GitHub is archived and unmaintained; development moved to gitlab.com/aatrani/pagn

## Evidence

- [results] Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0: "Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [results] clear_output(obj.solve_disk(N=1e4)): "clear_output(obj.solve_disk(N=1e4))"
- [results] We use the equations from Grishin et al. 2023: "We use the equations from
`Grishin et al. 2023 <https://arxiv.org/pdf/2307.07546.pdf>`__"
- [results] looking at the migration torques a 50 solar-mass BH in an AGN would experience while orbiting a central BH: "looking at the migration torques a 50 M\ :math:`_\odot` BH in an AGN would
experience while orbiting a central BH"
- [readme] pagn is a Python module to solve the equations from Sirko and Goodman 2003 and Thompson et al. 2005: "`pagn` is a Python module to solve the equations from [Sirko and Goodman 2003](https://academic.oup.com/mnras/article/341/2/501/1105444) and [Thompson et al."
