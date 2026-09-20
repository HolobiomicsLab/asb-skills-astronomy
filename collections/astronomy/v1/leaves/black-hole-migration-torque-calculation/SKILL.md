---
name: black-hole-migration-torque-calculation
description: Use when you need radial torque profiles (isothermal + thermal) for an
  embedded BH migrator in a parametric 1D AGN disk model, e.g. a 50 Msun BH with q=5e-6
  orbiting a 1e6 Msun central BH.
when_to_use_negative:
- 'The Thompson solution is super-critical: if tho.plot_mdot() shows Mdot_out above
  Mdot_c at r=Rout, the disk model is not valid for this workflow'
- You need a 2D or 3D hydrodynamic disk simulation rather than a parametric 1D AGN
  disk profile
- You want to model the disk with a custom opacity format that does not follow the
  specific format pagn requires for custom opacity tables
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Python module solving the Sirko & Goodman 2003 and Thompson et al. 2005 AGN
    disk equations to produce the radial profiles feeding the torque calculation
  repo: https://gitlab.com/aatrani/pagn
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) interpolates surface-density and thermal
    gradients over log-radius before torque evaluation
- name: matplotlib
  role: Used for plotting disk parameters and the accretion-rate diagnostic (tho.plot_mdot())
    and for plotting Gamma_tot versus radius
- name: numpy
  role: Array handling for the radial grid and disk profiles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:7e933e6e0d6b4bd02d17840c3ee9a7438f79e7fc0fcf9ae10001d7d1bd5e78e6
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
yes_rate: 0.7777777777777778
n_claims: 18
---

# black-hole-migration-torque-calculation

## Summary

Computes migration torques on a stellar-mass black hole embedded in an AGN disk by solving the Thompson et al. 2005 (or Sirko & Goodman 2003) disk equations with the pagn Python module and evaluating the Grishin et al. 2023 torque formulae over the disk's radial grid.

## When to use

Use when you need radial torque profiles (isothermal + thermal) for an embedded BH migrator in a parametric 1D AGN disk model, e.g. a 50 Msun BH with q=5e-6 orbiting a 1e6 Msun central BH.

## When NOT to use

- The Thompson solution is super-critical: if tho.plot_mdot() shows Mdot_out above Mdot_c at r=Rout, the disk model is not valid for this workflow
- You need a 2D or 3D hydrodynamic disk simulation rather than a parametric 1D AGN disk profile
- You want to model the disk with a custom opacity format that does not follow the specific format pagn requires for custom opacity tables

## Inputs

- AGN disk model parameters: central Mbh (e.g. 1e6 MSun), Mdot_out (e.g. 1.5e-2 MSun/yr), Rout (e.g. Rs*1e7)
- pagn ThompsonAGN (or SirkoAGN with le=0.5, alpha=0.01, b=0) object configuration
- Embedded BH mass (e.g. 50 MSun) defining the mass ratio q = 5e-6
- Grishin et al. 2023 torque equations for a locally thermally unsupported disk

## Outputs

- 1D radial profiles of density, temperature, scale height, and opacity from solve_disk(N=1e4)
- Torque coefficients Gamma_tot = Gamma_therm + Gamma_I_jm_tot versus radius (in units of Rs)
- torque_coefficients.csv containing radius, surface density, temperature, and torque coefficients

## How to apply

Instantiate ThompsonAGN with the central BH mass (e.g. Mbh=1e6 MSun), outer boundary conditions (Mdot_out=1.5e-2 MSun/yr, Rout=Rs*1e7), then run tho.solve_disk(N=1e4) to obtain 1D radial profiles of density, temperature, scale height, and opacity. Run tho.plot_mdot() to verify that Mdot_out is below the critical accretion rate Mdot_c at r=Rout — a sub-critical disk is required for a valid solution. Interpolate the surface-density and thermal gradients over log-radius using scipy.interpolate.UnivariateSpline(k=3, s=0.005, ext=0), which requires SciPy version 1.10.0 or later. Evaluate the Grishin et al. 2023 torque equations at each grid radius for the embedded BH (mass ratio q=5e-6), summing the Jiménez & Masset 2017 isothermal torque and the Masset 2017 thermal torque to get Gamma_tot = Gamma_therm + Gamma_I_jm_tot. Write radius, surface density, temperature, and torque coefficients to a CSV such as torque_coefficients.csv for downstream plotting versus radius in units of Rs.

## Related tools

- **pagn** (Python module solving the Sirko & Goodman 2003 and Thompson et al. 2005 AGN disk equations to produce the radial profiles feeding the torque calculation) — https://gitlab.com/aatrani/pagn
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) interpolates surface-density and thermal gradients over log-radius before torque evaluation)
- **matplotlib** (Used for plotting disk parameters and the accretion-rate diagnostic (tho.plot_mdot()) and for plotting Gamma_tot versus radius)
- **numpy** (Array handling for the radial grid and disk profiles)

## Examples

```
from pagn import Thompson; tho = Thompson.ThompsonAGN(Mbh=1e6*ct.MSun, Mdot_out=1.5e-2*ct.MSun/ct.yr, Rout=Rs*1e7); tho.solve_disk(N=1e4); tho.plot_mdot()
```

## Evaluation signals

- The accretion-rate diagnostic (tho.plot_mdot()) confirms Mdot_out is below the critical accretion rate Mdot_c evaluated at r = Rout
- torque_coefficients.csv contains finite torque coefficient values across the full radial grid
- Disk profiles were solved with N=1e4 grid points, described as sufficient resolution for most Mbh values
- Spline interpolation of log-radius gradients uses k=3, s=0.005, ext=0 and requires scipy >= 1.10.0
- Total torque equals the sum of the Jiménez & Masset 2017 isothermal torque and the Masset 2017 thermal torque

## Limitations

- The GitHub repository DariaGangardt/pAGN is archived and no longer maintained; the project has moved to gitlab.com/aatrani/pagn
- UnivariateSpline-based torque workflows require scipy version 1.10.0 or later
- Custom opacity tables must be provided in a specific format for ThompsonAGN and SirkoAGN to accept them
- Overflow in the exponential Planck factor is avoided by masking exp_factor < 40; large values give an integral of zero and are safely ignored

## Evidence

- [methods] tho = Thompson.ThompsonAGN(Mbh = mbh, sigma = sigma, epsilon = epsilon, m = m, xi = xi, Mdot_out= Mdot_out, Rout = Rout, Rin = Rin, opacity =opacity): "tho = Thompson.ThompsonAGN(Mbh = mbh, sigma = sigma, epsilon = epsilon, m = m, xi = xi, Mdot_out= Mdot_out, Rout = Rout, Rin = Rin, opacity =opacity)"
- [results] Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0): "Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [results] We use the equations from Grishin et al. 2023: "We use the equations from
`Grishin et al. 2023 <https://arxiv.org/pdf/2307.07546.pdf>`__"
- [results] looking at the migration torques a 50 M_sun BH in an AGN would experience while orbiting a central BH: "looking at the migration torques a 50 M\ :math:`_\odot` BH in an AGN would
experience while orbiting a central BH"
- [methods] one can also plot the accretion rate in the Thompson et al. disk: tho.plot_mdot(): "one can also plot the accretion rate in the Thompson et al. disk:

.. code:: ipython3

    tho.plot_mdot()"
- [readme] This repository is archived and no longer maintained. The project has moved to GitLab: "This repository is archived and no longer maintained.  
> The project has moved to GitLab:"
