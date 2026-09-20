---
name: accretion-rate-profile-extraction
description: Use when a solved pAGN disk object is available and you need Mdot(R),
  the outer-boundary Mdot_c/Mdot_out values, or the inner-boundary Mdot at Rin to
  judge whether the disk can fuel a bright AGN.
when_to_use_negative:
- You need a 2D or time-dependent AGN disk model — pAGN provides only parametric 1D
  radial profiles (temperature, density, Mdot)
- The disk has not been solved yet; solve_disk(N=1e4) must run before Mdot(R) can
  be extracted
- You need migration-torque calculations rather than accretion-rate profiles — those
  use the separate Grishin et al. 2023 torque equations in pAGN
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: pagn
  role: Provides Thompson.ThompsonAGN / Sirko.SirkoAGN models, solve_disk(N=1e4),
    plot_mdot(), and save() for extracting and persisting Mdot(R)
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Numerical handling of the radial grid and Mdot(R) arrays during extraction
    and normalization
- name: matplotlib
  role: Visualization of the accretion-rate profile via tho.plot_mdot() or custom
    plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:67281c3bf7c7bda0d21dc71db186841346212db47b567da30b62dcbafc14796e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# accretion-rate-profile-extraction

## Summary

Extract the radially varying accretion-rate profile Mdot(R) from a solved pAGN AGN disk model (Thompson et al. 2005 or Sirko & Goodman 2003), normalize by the Eddington accretion rate, and report boundary values at Rin and Rout.

## When to use

Use when a solved pAGN disk object is available and you need Mdot(R), the outer-boundary Mdot_c/Mdot_out values, or the inner-boundary Mdot at Rin to judge whether the disk can fuel a bright AGN.

## When NOT to use

- You need a 2D or time-dependent AGN disk model — pAGN provides only parametric 1D radial profiles (temperature, density, Mdot)
- The disk has not been solved yet; solve_disk(N=1e4) must run before Mdot(R) can be extracted
- You need migration-torque calculations rather than accretion-rate profiles — those use the separate Grishin et al. 2023 torque equations in pAGN

## Inputs

- Instantiated pAGN disk object (Thompson.ThompsonAGN or Sirko.SirkoAGN) with configuration parameters (Mbh, sigma, epsilon, m, xi, Mdot_out, Rout, Rin, opacity)
- Grid resolution N (documented choice: 1e4 grid points)
- Eddington accretion rate for normalization (≈26.29148 Msun/yr for the documented Thompson configuration)

## Outputs

- Radial accretion-rate profile Mdot(R) from the solved disk object
- Boundary values: Mdot_c at r = Rout, Mdot_out, and Mdot at r = Rin (in Msun/yr and Mdot_Edd units)
- Saved text file of key model parameters (e.g. thompson_mdot.txt) and optional plot via tho.plot_mdot()

## How to apply

Instantiate the disk model (e.g. Thompson.ThompsonAGN with sigma = 300 km/s, Mbh from the M-sigma relation as 2e8 MSun * (sigma/200 km/s)^4, epsilon = 1e-3, m = 0.2, xi = 1.0, Mdot_out = 1.5e-2 MSun/yr) and solve on a grid with solve_disk(N=1e4), since 10^4 points is documented as sufficient resolution for most Mbh values. Extract the Mdot(R) array from the solved object and normalize by the Eddington accretion rate (Mdot_Edd ≈ 26.29148 Msun/yr for the documented configuration). Optionally inspect the profile with tho.plot_mdot() and persist key parameters with the pagn save() helper to a txt file such as thompson_mdot.txt. Validate by checking that Mdot at r = Rin matches the reference 4.815066 Msun/yr (= 1.831417e-01 Mdot_Edd) within numerical tolerance and that Mdot_out (~8.311237e+03 Msun/yr) falls below Mdot_c (~9.606143e+03 Msun/yr) at the outer boundary.

## Related tools

- **pagn** (Provides Thompson.ThompsonAGN / Sirko.SirkoAGN models, solve_disk(N=1e4), plot_mdot(), and save() for extracting and persisting Mdot(R)) — https://gitlab.com/aatrani/pagn
- **numpy** (Numerical handling of the radial grid and Mdot(R) arrays during extraction and normalization)
- **matplotlib** (Visualization of the accretion-rate profile via tho.plot_mdot() or custom plots)

## Examples

```
from pagn import Thompson; tho = Thompson.ThompsonAGN(Mbh=2e8*1.989e30, sigma=3e5, epsilon=1e-3, m=0.2, xi=1.0, Mdot_out=1.5e-2*1.989e30/3.154e7); tho.solve_disk(N=1e4); tho.plot_mdot()
```

## Evaluation signals

- Mdot at r = Rin must equal the documented reference 4.815066 Msun/yr = 1.831417e-01 Mdot_Edd within numerical tolerance
- Mdot_out (~8.311237e+03 Msun/yr) must fall below Mdot_c at the outer boundary (~9.606143e+03 Msun/yr)
- Inner accretion rate should be ~5 Msun/yr, deemed sufficient to fuel a bright AGN
- N = 1e4 grid points yields a converged profile; significantly lower N should be checked against the reference boundary values
- For Sirko & Goodman disks, a quick visual check of key parameters via the package's plotting routine confirms the model ran correctly

## Limitations

- The GitHub repository (DariaGangardt/pAGN) is archived and no longer maintained; the project moved to https://gitlab.com/aatrani/pagn
- 10^4 grid points is documented as sufficient for most Mbh values, but convergence should be verified for extreme black-hole masses
- If a custom opacity is supplied to ThompsonAGN/SirkoAGN, values must be provided in the specific format the package expects or the solve will fail

## Evidence

- [methods] Solve the disk equations with tho.solve_disk(N=1e4), using 10^4 grid points as a sufficient resolution.: "tho.solve_disk(N=1e4) #10^4 tends to be a sufficient resolution for most Mbh values"
- [results] Documented Thompson configuration yields Mdot_Edd ≈ 26.29148 Msun/yr and inner-boundary reference values.: [evidence span withheld — anchor status: not_found]
- [results] Thompson disk instantiation uses Mdot_out = 1.5e-2 MSun/yr passed with ct units.: "obj = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout = rout, Mdot_out=Mdot_out*ct.MSun/ct.yr)"
- [methods] The package provides plotting and saving helpers for the accretion rate and key parameters.: "one can also plot the accretion rate in the Thompson et al. disk:

.. code:: ipython3

    tho.plot_mdot()"
- [readme] The repository is archived and maintenance moved to GitLab.: "This repository is archived and no longer maintained.  
> The project has moved to GitLab:
> - **Code repository:** https://gitlab.com/aatrani/pagn"
