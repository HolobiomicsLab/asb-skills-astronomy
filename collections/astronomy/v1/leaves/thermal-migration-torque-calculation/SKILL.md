---
name: thermal-migration-torque-calculation
description: Use when you need migration torque coefficients (CI_jm17_iso, CI_jm17_tot,
  CI_p10, CL) for an embedded BH migrator as a function of radius in a solved pAGN
  AGN disk model.
when_to_use_negative:
- You need the bolometric luminosity of the disk (use calculate_lum_sum with Eq. 47
  from Thompson et al. 2005 instead)
- You are working with a Thompson et al. 2005 disk and only need accretion rates (use
  tho.plot_mdot() rather than torque coefficients)
- Input is already a finished torque-coefficient table and the task is only to re-plot
  or re-save results
edam_operation: http://edamontology.org/operation_0334
tools:
- name: pagn
  role: Solves the Sirko & Goodman 2003 AGN disk equations (SirkoAGN, solve_disk)
    supplying the temperature/density profiles the torque coefficients are evaluated
    on
  repo: https://gitlab.com/aatrani/pagn
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) over log-radius for the surface-density
    and thermal gradients required by the torque prescriptions
- name: numpy
  role: Array handling and writing the radius-plus-coefficients table with numpy.savetxt
    to torque_coefficients.csv
- name: matplotlib
  role: Quick-check plots of key disk parameters to verify the Sirko & Goodman model
    ran correctly before torque evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:aff6ab44b2be51ef8a3df2af2f725744b80b426469b5073fa30864ba7a95b8e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_0334
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# thermal-migration-torque-calculation

## Summary

Computes migration torques on a stellar-mass black hole embedded in a Sirko & Goodman AGN disk using pAGN disk profiles plus the Jiménez & Masset 2017, Paardekooper et al. 2010, and Masset 2017 coefficient prescriptions assembled via the Grishin et al. 2023 equations.

## When to use

Use when you need migration torque coefficients (CI_jm17_iso, CI_jm17_tot, CI_p10, CL) for an embedded BH migrator as a function of radius in a solved pAGN AGN disk model.

## When NOT to use

- You need the bolometric luminosity of the disk (use calculate_lum_sum with Eq. 47 from Thompson et al. 2005 instead)
- You are working with a Thompson et al. 2005 disk and only need accretion rates (use tho.plot_mdot() rather than torque coefficients)
- Input is already a finished torque-coefficient table and the task is only to re-plot or re-save results

## Inputs

- AGN disk use-case parameters: central BH mass (e.g. Mbh=1e6 solar masses), le=0.5, alpha=0.01, X=0.7, b=0, opacity='combined'
- Embedded migrator properties: BH mass (e.g. 50 solar masses), mass ratio q=5e-6
- Solved pAGN Sirko disk radial profiles (radius, Omega, temperature, density, scale height, sound speed, optical depth, Q) from solve_disk(N=1e4)

## Outputs

- CSV table (e.g. torque_coefficients.csv, via numpy.savetxt) with a radius column (R/ct.pc) and the four coefficient columns CI_jm17_iso, CI_jm17_tot, CI_p10, CL
- Total migration torque Gamma_tot = Gamma_therm + Gamma_I_jm_tot per disk radius

## How to apply

Instantiate Sirko.SirkoAGN with the use-case parameters (Mbh, le=0.5, alpha=0.01, X=0.7, b=0, opacity='combined') and call solve_disk(N=1e4) to obtain radial profiles of Omega, temperature, density, scale height, sound speed, optical depth, and Q. Interpolate the log-radius surface-density (Sigmalog10_spline) and thermal gradients with scipy's UnivariateSpline using k=3, s=0.005, ext=0 (requires scipy >= 1.10.0). Evaluate the thermal and isothermal coefficient prescriptions CI_jm17_iso, CI_jm17_tot (Jiménez & Masset 2017), CI_p10 (Paardekooper et al. 2010), and CL (Masset 2017) following the equations of Grishin et al. 2023, for the chosen embedded BH mass (e.g. 50 solar masses, q=5e-6 around a 1e6 solar-mass central BH) at each disk radius. Assemble radius (R/ct.pc) and coefficient values into a table and write it with numpy.savetxt to a CSV such as torque_coefficients.csv; the total torque is Gamma_tot = Gamma_therm + Gamma_I_jm_tot.

## Related tools

- **pagn** (Solves the Sirko & Goodman 2003 AGN disk equations (SirkoAGN, solve_disk) supplying the temperature/density profiles the torque coefficients are evaluated on) — https://gitlab.com/aatrani/pagn
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) over log-radius for the surface-density and thermal gradients required by the torque prescriptions)
- **numpy** (Array handling and writing the radius-plus-coefficients table with numpy.savetxt to torque_coefficients.csv)
- **matplotlib** (Quick-check plots of key disk parameters to verify the Sirko & Goodman model ran correctly before torque evaluation)

## Examples

```
sk = Sirko.SirkoAGN(Mbh=1e6*ct.MSun, le=0.5, alpha=0.01, b=0); sk.solve_disk(N=1e4)
```

## Evaluation signals

- Solved Sirko & Goodman disk shows temperature decreasing outward from ~1e5-1e6 K
- h/r stays below one in the inner disk regions
- Output table contains the radius column (R/ct.pc) plus all four coefficient columns (CI_jm17_iso, CI_jm17_tot, CI_p10, CL)
- Disk solved with N=1e4 grid points; 10^4 tends to be a sufficient resolution for most Mbh values
- Spline interpolation runs under scipy >= 1.10.0 with k=3, s=0.005, ext=0

## Limitations

- UnivariateSpline-based gradient computation requires scipy ver 1.10.0
- Overflow in the exponential Planck factor is avoided by masking exp_factor < 40 (large values give integral value zero and are safely ignored)
- The GitHub repository DariaGangardt/pAGN is archived and no longer maintained; the project moved to GitLab (gitlab.com/aatrani/pagn)

## Evidence

- [readme] pagn module solves SG03/TT05 AGN disk models: [evidence span withheld — anchor status: not_found]
- [methods] Sirko disk instantiation and solve: "sk = Sirko.SirkoAGN(Mbh=Mbh, le=le, Mdot=Mdot, alpha=alpha, X=X, 
                    b=b, opacity = Opacity)
sk.solve_disk(N=1e4)"
- [results] Spline interpolation over log radius: "Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)"
- [results] Torque equations source: "We use the equations from
`Grishin et al. 2023 <https://arxiv.org/pdf/2307.07546.pdf>`__"
- [results] Migration torque use case for 50 solar-mass BH: "looking at the migration torques a 50 M\ :math:`_\odot` BH in an AGN would
experience while orbiting a central BH"
- [readme] Repository archived notice: "> ⚠️ **Repository status:** This repository is archived and no longer maintained."
