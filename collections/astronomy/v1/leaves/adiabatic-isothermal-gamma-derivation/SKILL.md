---
name: adiabatic-isothermal-gamma-derivation
description: Use when migration torques must be computed for a black-hole migrator
  embedded in a solved AGN disk (Sirko & Goodman or Thompson et al. models) and the
  local surface-density, temperature, and pressure gradients feeding Jiménez & Masset
  2017, Masset 2017, and Paardekooper et al.
when_to_use_negative:
- The disk model has not yet been solved — the gradients require converged radius-resolved
  profiles (surface density, temperature, pressure) from solve_disk; deriving gamma
  from analytic guesses is out of scope
- The analysis targets torque formulas not expressed through Grishin et al. 2023 equations
  or not built on the Jiménez & Masset 2017 / Masset 2017 / Paardekooper et al. 2010
  coefficients
- A custom opacity is supplied in an incompatible format — both SirkoAGN and ThompsonAGN
  require opacity values in a specific format, and an incorrectly formatted opacity
  would corrupt the profiles feeding the gradients
tools:
- name: pagn
  role: Provides the SirkoAGN and ThompsonAGN disk solvers whose solved profiles (surface
    density, temperature, pressure) supply the inputs to the spline-based gradient
    and gamma derivations
  repo: https://gitlab.com/aatrani/pagn
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) is used to build differentiable interpolants
    of log surface density, log temperature, and log pressure over log radius; requires
    scipy ver 1.10.0
- name: numpy
  role: Numerical evaluation of the gradient functions and torque coefficients over
    the disk grid
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:22d98eb4a7141277bca8e69e9df8b9ffd9c9ad6e3a7146cc34791fe77b13b113
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# adiabatic-isothermal-gamma-derivation

## Summary

Reconstructs the thermal-torque coefficient module (gamma_thermal, CI_jm17_iso, CI_jm17_tot, CI_p10, CL) that computes adiabatic, isothermal, and thermal gamma factors from disk gradients of a solved pAGN AGN disk, feeding the migration torque formulas of Grishin et al. 2023.

## When to use

Use when migration torques must be computed for a black-hole migrator embedded in a solved AGN disk (Sirko & Goodman or Thompson et al. models) and the local surface-density, temperature, and pressure gradients feeding Jiménez & Masset 2017, Masset 2017, and Paardekooper et al. 2010 torque coefficients are needed.

## When NOT to use

- The disk model has not yet been solved — the gradients require converged radius-resolved profiles (surface density, temperature, pressure) from solve_disk; deriving gamma from analytic guesses is out of scope
- The analysis targets torque formulas not expressed through Grishin et al. 2023 equations or not built on the Jiménez & Masset 2017 / Masset 2017 / Paardekooper et al. 2010 coefficients
- A custom opacity is supplied in an incompatible format — both SirkoAGN and ThompsonAGN require opacity values in a specific format, and an incorrectly formatted opacity would corrupt the profiles feeding the gradients

## Inputs

- Solved pAGN AGN disk object (SirkoAGN or ThompsonAGN) with radius-resolved surface density, temperature, and pressure profiles from solve_disk(N=1e4)
- Migrator parameters: black-hole mass (e.g. 50 solar masses), central SMBH mass (e.g. 1e6 solar masses), mass ratio q (e.g. 5e-6)
- Torque-coefficient formulas from Jiménez & Masset 2017, Masset 2017, and Paardekooper et al. 2010 as invoked by Grishin et al. 2023

## Outputs

- Gradient functions dSigmadR, dTdR, dPdR obtained by differentiating UnivariateSpline interpolants (k=3, s=0.005, ext=0) over log10(R)
- Torque coefficients gamma_thermal, CI_jm17_iso, CI_jm17_tot, CI_p10, CL evaluated at the migrator's orbital radius in both disk models
- Comparison against the reference torque values of the pAGN use-case runs

## How to apply

First solve the AGN disk with pAGN's SirkoAGN or ThompsonAGN classes using solve_disk(N=1e4) to obtain surface density, temperature, and pressure profiles versus radius. Then build cubic-spline interpolants with UnivariateSpline(k=3, s=0.005, ext=0) fitted over log10 radius against the log10 surface density (Sigma = 2*rho*h) and log10 temperature, and differentiate the splines to obtain dSigmadR, dTdR, and dPdR gradient functions (this requires scipy >= 1.10.0). Combine these local gradients at the migrator's orbital radius into the adiabatic, isothermal, and thermal gamma expressions, then implement the torque-coefficient formulas of Jiménez & Masset 2017, Masset 2017, and Paardekooper et al. 2010 as invoked through the equations of Grishin et al. 2023. Validate on the pAGN use case: a 50 solar-mass BH migrator orbiting a 1e6 solar-mass SMBH with mass ratio q = 5e-6 in both the Sirko & Goodman (le=0.5, alpha=0.01, b=0) and Thompson (Rout = Rs*1e7, Mdot_out = 1.5e-2 MSun/yr) disk configurations.

## Related tools

- **pagn** (Provides the SirkoAGN and ThompsonAGN disk solvers whose solved profiles (surface density, temperature, pressure) supply the inputs to the spline-based gradient and gamma derivations) — https://gitlab.com/aatrani/pagn
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) is used to build differentiable interpolants of log surface density, log temperature, and log pressure over log radius; requires scipy ver 1.10.0)
- **numpy** (Numerical evaluation of the gradient functions and torque coefficients over the disk grid)

## Examples

```
from pagn import Sirko; from scipy.interpolate import UnivariateSpline; sk = Sirko.SirkoAGN(Mbh=1e6*ct.MSun, le=0.5, alpha=0.01, b=0); sk.solve_disk(N=1e4); Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)
```

## Evaluation signals

- script_runs: the module executes standalone without errors on the solved Sirko & Goodman and Thompson disk objects
- output_matches_reference: computed torque values reproduce the reference values of the pAGN use-case runs (50 solar-mass BH, 1e6 solar-mass SMBH, q = 5e-6); no numeric tolerance is stated, so agreement is judged against the published use-case outputs
- Spline interpolants are constructed with the exact parameters k=3, s=0.005, ext=0 over log10(R) and the environment satisfies scipy >= 1.10.0
- Disk solutions converged with N=1e4 grid points, the resolution stated to be sufficient for most Mbh values
- Gradient functions (dSigmadR, dTdR, dPdR) evaluated at the migrator's orbital radius yield finite, physically sensible values in both disk models

## Limitations

- The spline functions require scipy ver 1.10.0 — older scipy versions are a hard compatibility constraint
- The source states no numeric tolerance for torque validation, so agreement must be judged directly against the published use-case outputs rather than a defined epsilon
- The GitHub repository DariaGangardt/pAGN is archived and no longer maintained; the project has moved to GitLab (gitlab.com/aatrani/pagn) and should be obtained from there
- Custom opacities are only accepted in a specific format, and misformatted opacity tables would propagate into the disk profiles and hence the derived gradients

## Evidence

- [results] Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0: "Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [results] Migration torques were computed for a 50 solar-mass BH migrator in AGN disks using a central BH of 1e6 solar masses with mass ratio q = 5e-6: "looking at the migration torques a 50 M\ :math:`_\odot` BH in an AGN would
experience while orbiting a central BH"
- [results] Torque calculations used the equations of Grishin et al. 2023: "We use the equations from
`Grishin et al. 2023 <https://arxiv.org/pdf/2307.07546.pdf>`__"
- [readme] pagn provides fully evolved AGN disk models through parametric 1D profiles such as temperature and density: [evidence span withheld — anchor status: not_found]
- [methods] 10^4 tends to be a sufficient resolution for most Mbh values: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
