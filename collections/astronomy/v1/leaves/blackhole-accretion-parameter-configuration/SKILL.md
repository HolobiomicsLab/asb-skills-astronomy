---
name: blackhole-accretion-parameter-configuration
description: 'Use when setting up a parametric 1D AGN disk profile in pagn: you must
  choose Mbh, epsilon, m, xi, Rin, Rout, Mdot_out, alpha, le, b, and the opacity scheme
  (''combined'' or custom) before calling solve_disk.'
when_to_use_negative:
- The disk is already solved and the task is post-processing only, e.g. integrating
  the bolometric luminosity with calculate_lum_sum — no re-configuration is needed.
- You need stellar-mass BH evolution inside a pre-computed AGN disk (migration torques
  via Grishin et al. 2023 equations) rather than setting up the disk parameters themselves.
tools:
- name: pagn
  role: Provides the ThompsonAGN and SirkoAGN classes whose constructor arguments
    accept the configured disk parameters; installed via pip install pagn
  repo: https://gitlab.com/aatrani/pagn
- name: Python
  role: Language in which pagn models are instantiated and parameterized
- name: numpy
  role: Used alongside pagn for grid handling after parameter configuration (e.g.
    log-spaced wavelength grids)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:3078165570de96de433bb97539675b3961d9c8c9f57fef2a0ec7c2082acbee8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - pagn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# blackhole-accretion-parameter-configuration

## Summary

Configures the physical input parameters of pAGN's AGN disk models (ThompsonAGN and SirkoAGN) — black hole mass, accretion rates, alpha viscosity, opacity choice, and inner/outer radii — prior to solving the disk equations.

## When to use

Use when setting up a parametric 1D AGN disk profile in pagn: you must choose Mbh, epsilon, m, xi, Rin, Rout, Mdot_out, alpha, le, b, and the opacity scheme ('combined' or custom) before calling solve_disk.

## When NOT to use

- The disk is already solved and the task is post-processing only, e.g. integrating the bolometric luminosity with calculate_lum_sum — no re-configuration is needed.
- You need stellar-mass BH evolution inside a pre-computed AGN disk (migration torques via Grishin et al. 2023 equations) rather than setting up the disk parameters themselves.

## Inputs

- Central black hole mass Mbh in physical units (e.g. 1e8 MSun as Mbh*ct.MSun)
- Thompson parameters: epsilon = 1e-3, m = 0.2, xi = 1.0, Rin = 3 Schwarzschild radii
- Thompson outflow scalings: Mdot_out (e.g. 1.5e-2 MSun/yr) and Rout (e.g. Rs*1e7)
- Sirko parameters: le = 0.5, alpha = 0.01, b = 0, Mdot, X
- Opacity specification: 'combined' keyword or a custom opacity array in pagn's required format

## Outputs

- Configured pagn model object (Thompson.ThompsonAGN or Sirko.SirkoAGN) ready for solve_disk(N=1e4)
- Solved 1D disk profiles (temperature, surface density, accretion rate) over a 1e4-point radial grid

## How to apply

Instantiate the model class from pagn with the documented parameter values: ThompsonAGN defaults are Mbh = 1e8 MSun, epsilon = 1e-3, m = 0.2, xi = 1.0, Rin = 3 Schwarzschild radii, opacity = 'combined', with Mdot_out = 1.5e-2 MSun/yr and Rout = Rs*1e7 used in the paper's Thompson configuration; SirkoAGN is configured with le = 0.5, alpha = 0.01, b = 0. Masses and rates must be passed in physical units (e.g. Mbh*ct.MSun, Mdot_out*ct.MSun/ct.yr). A custom opacity table can be supplied, but the opacity values must be provided in the specific format pagn requires. After configuration, solve the disk with solve_disk(N=1e4), since 10^4 grid points are documented as sufficient resolution for most Mbh values. Decision point: use Sirko & Goodman 2003 or Thompson et al. 2005 depending on which disk physics (self-gravity/outflow treatment) the study requires.

## Related tools

- **pagn** (Provides the ThompsonAGN and SirkoAGN classes whose constructor arguments accept the configured disk parameters; installed via pip install pagn) — https://gitlab.com/aatrani/pagn
- **Python** (Language in which pagn models are instantiated and parameterized)
- **numpy** (Used alongside pagn for grid handling after parameter configuration (e.g. log-spaced wavelength grids))

## Examples

```
from pagn import Thompson; tho = Thompson.ThompsonAGN(Mbh=1e8*ct.MSun, epsilon=1e-3, m=0.2, xi=1.0, Rin=3, opacity='combined'); tho.solve_disk(N=1e4)
```

## Evaluation signals

- The configured Thompson disk, when solved and integrated over wavelengths 1e-8 to 1e-3 m, reproduces the reference bolometric luminosity of 177303419977.49695 solar luminosities.
- The configured Sirko & Goodman disk reproduces the reference luminosity of 1054693353944.121 solar luminosities.
- solve_disk was called with N=1e4 grid points, the documented sufficient resolution for most Mbh values.
- Plotting key disk parameters (plot_mdot and related quick-check plots) after configuration shows the expected disk structure for the chosen parameters.
- Results can be written to a txt file via pagn's save() method for downstream verification.

## Limitations

- The repository is archived and no longer maintained; the project has moved to https://gitlab.com/aatrani/pagn.
- Custom opacity values must be provided in a specific format required by pagn; incorrect formatting will not work.
- N=10^4 grid points are described as sufficient 'for most Mbh values', implying very large black hole masses may need higher resolution.
- The discussion section contained no limitations text; no further reproducibility caveats were documented.

## Evidence

- [methods] Thompson constructor parameters: "tho = Thompson.ThompsonAGN(Mbh = mbh, sigma = sigma, epsilon = epsilon, m = m, xi = xi,
                      Mdot_out= Mdot_out, Rout = Rout, Rin = Rin, opacity =opacity)"
- [methods] Sirko constructor parameters: "sk = Sirko.SirkoAGN(Mbh=Mbh, le=le, Mdot=Mdot, alpha=alpha, X=X, 
                    b=b, opacity = Opacity)"
- [results] Paper's Thompson configuration values: "Mdot_out = 1.5e-2
            obj = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout = rout, Mdot_out=Mdot_out*ct.MSun/ct.yr)"
- [methods] Custom opacity format requirement: "Both SirkoAGN and ThompsonAGN can take in a custom opacity. The opacity values have to be provided in a specific format for this to work."
- [readme] pagn package purpose: "providing fully evolved AGN disk models through parametric 1D profiles (e.g. temperature, density)"
