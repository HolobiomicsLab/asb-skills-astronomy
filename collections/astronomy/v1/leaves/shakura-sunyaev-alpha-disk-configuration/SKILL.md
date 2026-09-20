---
name: shakura-sunyaev-alpha-disk-configuration
description: Use when a 1D steady AGN accretion-disk profile (temperature, density,
  scale height) must be generated under the alpha-disk prescription for embedded-object
  torque or luminosity calculations.
when_to_use_negative:
- The problem requires a supply/accretion-rate profile with an outer boundary condition
  — use Thompson.ThompsonAGN (Rout=Rs*1e7, Mdot_out=1.5e-2 Msun/yr) and its plot_mdot()
  instead
- The target is a time-dependent or multi-dimensional disk simulation rather than
  a parametric 1D steady-state profile
tools:
- name: pagn
  role: Provides Sirko.SirkoAGN for the b=0 alpha-disk configuration and solve_disk/plot
    methods
  repo: https://gitlab.com/aatrani/pagn
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) over log-radius for gradient interpolation
    in torque calculations (requires scipy >= 1.10.0)
- name: numpy
  role: Saves the radius/coefficient table with numpy.savetxt
- name: matplotlib
  role: Renders the quick-check diagnostic panel of key disk parameters
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:d3dc51c325534073e974e520e173de646bab72b352b42d5ab0186bba7e8c8d03
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
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# shakura-sunyaev-alpha-disk-configuration

## Summary

Configure and solve the Sirko & Goodman (2003) alpha-disk AGN model in pagn by instantiating SirkoAGN with the Shakura-Sunyaev b=0 setting (alpha=0.01, le=0.5, X=0.7, opacity='combined') and solving on a 1e4-point radial grid.

## When to use

Use when a 1D steady AGN accretion-disk profile (temperature, density, scale height) must be generated under the alpha-disk prescription for embedded-object torque or luminosity calculations.

## When NOT to use

- The problem requires a supply/accretion-rate profile with an outer boundary condition — use Thompson.ThompsonAGN (Rout=Rs*1e7, Mdot_out=1.5e-2 Msun/yr) and its plot_mdot() instead
- The target is a time-dependent or multi-dimensional disk simulation rather than a parametric 1D steady-state profile

## Inputs

- Central black-hole mass Mbh (e.g. 1e8 Msun, converted via pagn's ct.MSun)
- Eddington ratio le (0.5 in the paper's use case), with Mdot=None to derive Mdot from le
- Viscosity parameter alpha=0.01, hydrogen fraction X=0.7, b=0 (alpha-disk case)
- Opacity table choice ('combined' or a custom opacity in the required format)

## Outputs

- Radial profiles of radius, Omega, temperature, density, scale height, sound speed, optical depth, and Toomre Q on a 1e4-point grid
- Diagnostic figure of key disk parameters from sk.plot() with the regime-switch vertical line
- Optional tabulation of radius (R/ct.pc) and torque coefficients via numpy.savetxt

## How to apply

Instantiate Sirko.SirkoAGN with the b=0 (alpha-disk) configuration: Mbh in grams via ct.MSun, le=0.5 with Mdot=None so the accretion rate is computed from the Eddington ratio, alpha=0.01, X=0.7 hydrogen fraction, and opacity='combined'. Call solve_disk(N=1e4), the resolution stated to be sufficient for most Mbh values, to obtain radial profiles of radius, Omega, temperature, density, scale height, sound speed, optical depth, and Q. For downstream torque work, interpolate the log-radius surface-density and thermal gradients with scipy's UnivariateSpline (k=3, s=0.005, ext=0), which requires scipy >= 1.10.0. Verify the solve by calling sk.plot(): the temperature must decrease outward from ~1e5–1e6 K and the h/r ratio must stay below one in the inner disk regions, with a vertical line marking the inner/outer regime switch.

## Related tools

- **pagn** (Provides Sirko.SirkoAGN for the b=0 alpha-disk configuration and solve_disk/plot methods) — https://gitlab.com/aatrani/pagn
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) over log-radius for gradient interpolation in torque calculations (requires scipy >= 1.10.0))
- **numpy** (Saves the radius/coefficient table with numpy.savetxt)
- **matplotlib** (Renders the quick-check diagnostic panel of key disk parameters)

## Examples

```
sk = Sirko.SirkoAGN(Mbh=1e8*ct.MSun, le=0.5, Mdot=None, alpha=0.01, X=0.7, b=0, opacity='combined'); sk.solve_disk(N=1e4); sk.plot()
```

## Evaluation signals

- Plotted temperature decreases outward starting at ~1e5–1e6 K (paper's quick-check criterion)
- h/r ratio stays below one in the inner disk regions
- sk.plot() shows a vertical line marking the switch between inner and outer regimes
- Solve completed at N=1e4 grid points, stated as sufficient resolution for most Mbh values
- For torque use cases, the output table contains the radius column plus all four coefficient columns (CI_jm17_iso/tot, CI_p10, CL)

## Limitations

- Overflow in the exponential Planck factor is avoided by masking exp_factor < 40; large values give an integral of zero and are safely ignored
- Spline interpolation requires scipy version >= 1.10.0
- The GitHub repository is archived and unmaintained; development has moved to GitLab (gitlab.com/aatrani/pagn)

## Evidence

- [methods] methods: "To quickly check that the Sirko & Goodman model ran correctly, we can plot some key parameters"
- [results] results: "obj = Sirko.SirkoAGN(Mbh=Mbh*ct.MSun, le=le, alpha=alpha, b=0)"
- [readme] readme: "providing fully evolved AGN disk models through parametric 1D profiles (e.g. temperature, density)"
- [methods] methods-resolution: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
