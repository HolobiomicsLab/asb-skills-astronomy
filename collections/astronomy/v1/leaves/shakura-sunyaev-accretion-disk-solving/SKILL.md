---
name: shakura-sunyaev-accretion-disk-solving
description: Use when you need a parametric 1D AGN disk model (temperature, surface
  density, accretion-rate profiles) for a given black-hole mass, Eddington ratio le,
  and viscosity alpha — e.g. as input for stellar/binary evolution in AGN disks or
  migration-torque calculations.
when_to_use_negative:
- You need time-dependent or multi-dimensional disk evolution — pAGN provides only
  parametric 1D steady-state profiles
- The disk physics you need is not covered by the Sirko & Goodman 2003 or Thompson
  et al. 2005 formulations (pAGN solves only these two models)
tools:
- name: pagn
  role: Python module providing SirkoAGN and ThompsonAGN disk solvers, plotting helpers
    (plot_mdot), save(), and calculate_lum_sum
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Numerical arrays for radial grids and temperature/opacity arrays fed to solve_disk
    and calculate_lum_sum
- name: Python
  role: Implementation language for the pAGN module and driver scripts
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005) for interpolating surface density and thermal
    gradients over log radius
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:f0d1f1f3790d1470cc8f2e8539627ada82c3597d8edc64cfa7a488506d80bcac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - numpy
  - pagn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# shakura-sunyaev-accretion-disk-solving

## Summary

Solve 1D steady-state AGN accretion-disk equations (Shakura–Sunyaev style, as formulated by Sirko & Goodman 2003 and Thompson et al. 2005) with the pAGN Python module, producing fully evolved radial profiles (temperature, density, accretion rate) and derived quantities such as bolometric luminosity and migration torques.

## When to use

Use when you need a parametric 1D AGN disk model (temperature, surface density, accretion-rate profiles) for a given black-hole mass, Eddington ratio le, and viscosity alpha — e.g. as input for stellar/binary evolution in AGN disks or migration-torque calculations.

## When NOT to use

- You need time-dependent or multi-dimensional disk evolution — pAGN provides only parametric 1D steady-state profiles
- The disk physics you need is not covered by the Sirko & Goodman 2003 or Thompson et al. 2005 formulations (pAGN solves only these two models)

## Inputs

- Black-hole mass Mbh (e.g. 1e8 solar masses via ct.MSun)
- Eddington ratio le (e.g. 0.5) or an explicit accretion rate Mdot
- Viscosity parameter alpha (e.g. 0.01), hydrogen fraction X=0.7, b=0, opacity option ('combined' or a custom opacity in the required format)
- For Thompson et al. model: sigma, epsilon, m, xi, Mdot_out (e.g. 1.5e-2 MSun/yr), Rout (e.g. Rs*1e7), Rin
- Radial grid size N (1e4 recommended)

## Outputs

- Solved 1D AGN disk profiles: effective temperature, surface density, accretion rate vs. radius
- Bolometric luminosity in solar luminosities (e.g. 1054693353944.121 for the canonical Sirko & Goodman 2003 disk)
- Saved parameter table (txt file via save(obj, filename))
- Optional migration torque profiles for embedded migrators (using Grishin et al. 2023 equations)

## How to apply

Install the package (pip install pagn) and instantiate a model: Sirko.SirkoAGN(Mbh=Mbh*ct.MSun, le=le, Mdot=None, alpha=alpha, X=0.7, b=b, opacity='combined') for the Sirko & Goodman 2003 model, or Thompson.ThompsonAGN(Mbh, sigma, epsilon, m, xi, Mdot_out, Rout, Rin, opacity) for Thompson et al. 2005 (e.g. Rout=Rs*1e7, Mdot_out=1.5e-2 MSun/yr). When le is given, leave Mdot=None so the accretion rate is derived from the Eddington ratio. Solve the disk with sk.solve_disk(N=1e4); 10^4 radial grid points is documented as sufficient resolution for most Mbh values. Validate by plotting key parameters (sk.plot_mdot() for Thompson) and, for the canonical Sirko & Goodman configuration (Mbh=1e8 MSun, le=0.5, alpha=0.01, b=0), confirm the bolometric luminosity computed via calculate_lum_sum (approximating the Eq. 47 double integral from Thompson et al. 2005 as discrete sums over wavelength and radius) reproduces 1054693353944.121 solar luminosities. Results can be saved to txt via the save(obj, filename) helper.

## Related tools

- **pagn** (Python module providing SirkoAGN and ThompsonAGN disk solvers, plotting helpers (plot_mdot), save(), and calculate_lum_sum) — https://gitlab.com/aatrani/pagn
- **numpy** (Numerical arrays for radial grids and temperature/opacity arrays fed to solve_disk and calculate_lum_sum)
- **Python** (Implementation language for the pAGN module and driver scripts)
- **SciPy** (UnivariateSpline (k=3, s=0.005) for interpolating surface density and thermal gradients over log radius)

## Examples

```
from pagn import Sirko; import pagn.constants as ct; sk = Sirko.SirkoAGN(Mbh=1e8*ct.MSun, le=0.5, alpha=0.01, b=0); sk.solve_disk(N=1e4)
```

## Evaluation signals

- Reproduce the reference Sirko & Goodman 2003 bolometric luminosity of 1054693353944.121 solar luminosities for le=0.5, alpha=0.01, N=1e4 within numerical tolerance
- Reference Thompson et al. 2005 luminosity check: 1773034194977.49695... i.e. 177303419977.49695 solar luminosities for Rout=Rs*1e7, Mdot_out=1.5e-2 MSun/yr
- Solution converges with N=1e4 radial grid points (documented as sufficient resolution for most Mbh values)
- Quick visual check: plotted key disk parameters (temperature, surface density) show smooth, physically sensible radial profiles
- Masked Planck exponential factors (exp_factor < 40) contain no overflow errors in calculate_lum_sum

## Limitations

- The GitHub repository DariaGangardt/pAGN is archived and no longer maintained; the project moved to https://gitlab.com/aatrani/pagn — use GitLab for the latest version
- Spline interpolation of surface density and thermal gradients requires scipy version 1.10.0
- Large exponential Planck factors cause overflow errors in the luminosity sum; values with exp_factor >= 40 are safely ignored since they contribute zero to the integral
- Custom opacity must be provided in a specific format required by both SirkoAGN and ThompsonAGN

## Evidence

- [abstract] pAGN module for the Python programming language, which is the first public implementation of these accretion-disc models: "Our findings are implemented in the pAGN module for the Python programming language, which is the first public implementation of these accretion-disc models."
- [results] pagn solves Sirko & Goodman and Thompson models: "``pagn`` is a Python module to solve the equations presented in Sirko & Goodman (2003) and Thompson et al. (2005)"
- [methods] solve_disk with 1e4 grid points: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [methods] bolometric luminosity via Eq. 47 sums: [evidence span withheld — anchor status: not_found]
- [results] scipy version constraint for splines: "UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [results] canonical Sirko & Goodman luminosity result: "Luminosity of Sirko & Goodman 2003 disk:  1054693353944.121  solar luminosities"
- [readme] pAGN provides 1D disk profiles: "providing fully evolved AGN disk models through parametric 1D profiles (e.g. temperature, density)"
