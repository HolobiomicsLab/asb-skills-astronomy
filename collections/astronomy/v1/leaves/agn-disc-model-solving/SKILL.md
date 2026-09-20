---
name: agn-disc-model-solving
description: Use when a 1D radial profile (Teff, Sigma, h/r, opacity, Mdot(r)) of
  an AGN disk is needed from the Sirko & Goodman 2003 or Thompson et al. 2005 models
  as input to luminosity, torque, or compact-object-formation analyses.
when_to_use_negative:
- A multi-dimensional, time-dependent, or self-gravitating disk model is required
  — pagn provides steady-state 1D parametric profiles of the Sirko & Goodman and Thompson
  et al. equations only.
- Only a raw opacity table needs processing rather than a solved disk — pagn accepts
  custom opacity only in a specific required format and is not a general opacity tool.
- 'Migration torques are the final goal without a solved disk: the torque workflow
  (UnivariateSpline gradient interpolation, Grishin et al. 2023 equations) presupposes
  completed solve_disk output.'
edam_operation: http://edamontology.org/operation_0242
tools:
- name: pagn
  role: Python module that implements and solves the Sirko & Goodman 2003 and Thompson
    et al. 2005 AGN disk equations via SirkoAGN/ThompsonAGN.solve_disk(N=1e4)
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Provides the numeric arrays behind the solved 1D radial profiles and the Mdot(R)
    normalization
- name: matplotlib
  role: Renders the quick-check parameter plot (sk.plot) and the accretion-rate diagnostic
    (tho.plot_mdot) used to validate the solved disk
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) interpolates surface-density and thermal
    gradients over log-radius after the disk is solved (requires scipy ver 1.10.0)
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:8fd7a35459a9e401d2d134be6b5fde824518ca78a6c9c04edcf4fdab87cc0981
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_0242
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# agn-disc-model-solving

## Summary

Solves the 1D steady-state AGN accretion-disk equations of Sirko & Goodman (2003) and Thompson et al. (2005) using the pagn Python module, producing fully evolved parametric radial profiles (temperature, density, scale height, opacity, accretion rate) that feed downstream analyses such as bolometric luminosity sums and migration-torque calculations.

## When to use

Use when a 1D radial profile (Teff, Sigma, h/r, opacity, Mdot(r)) of an AGN disk is needed from the Sirko & Goodman 2003 or Thompson et al. 2005 models as input to luminosity, torque, or compact-object-formation analyses.

## When NOT to use

- A multi-dimensional, time-dependent, or self-gravitating disk model is required — pagn provides steady-state 1D parametric profiles of the Sirko & Goodman and Thompson et al. equations only.
- Only a raw opacity table needs processing rather than a solved disk — pagn accepts custom opacity only in a specific required format and is not a general opacity tool.
- Migration torques are the final goal without a solved disk: the torque workflow (UnivariateSpline gradient interpolation, Grishin et al. 2023 equations) presupposes completed solve_disk output.

## Inputs

- Model configuration: SirkoAGN(Mbh, le, Mdot=None, alpha, X, b, opacity) — e.g. Mbh=1e8 MSun, le=0.5, alpha=0.01, X=0.7, b=0, opacity='combined'
- Model configuration: ThompsonAGN(Mbh, sigma, epsilon, m, xi, Mdot_out, Rout, Rin, opacity) — e.g. sigma=300 km/s with Mbh from the M-sigma relation 2e8*(sigma/200)^4 MSun, epsilon=1e-3, m=0.2, xi=1.0, Mdot_out=1.5e-2 MSun/yr, Rout=Rs*1e7
- Grid resolution argument N (documented sufficient value: 1e4)

## Outputs

- Solved disk object with 1D radial profiles of surface density, temperature, scale height, opacity, and accretion rate Mdot(R)
- Diagnostic figure from sk.plot() (key disk parameters with the inner/outer regime switch marked) or tho.plot_mdot() (accretion-rate profile)
- Boundary values: Mdot_c at Rout, Mdot_out, and Mdot at Rin (e.g. 4.815066 Msun/yr = 1.831417e-01 Mdot_Edd for the Thompson M-sigma configuration)
- Saved parameter file via save(obj, filename) written to a txt file

## How to apply

Install pagn from PyPI (pip install pagn) and import the desired model: Sirko.SirkoAGN for Sirko & Goodman (configured with Mbh, le Eddington ratio with Mdot=None, alpha, X=0.7, b=0 for the alpha-disk case, opacity='combined') or Thompson.ThompsonAGN for Thompson et al. (configured with Mbh, sigma, epsilon=1e-3, m=0.2, xi=1.0, and either explicit Mdot_out/Rout/Rin or package defaults). Call obj.solve_disk(N=1e4) — 10^4 grid points is the documented sufficient resolution for most Mbh values. For Thompson disks, validate the outer boundary with tho.plot_mdot(): Mdot_out must fall below the critical accretion rate Mdot_c evaluated at r=Rout (e.g. Mdot_c ≈ 9.606143e+03 Msun/yr vs Mdot_out ≈ 8.311237e+03 Msun/yr for the M-sigma-derived configuration). For Sirko & Goodman, run the quick-check diagnostic sk.plot() and accept the solution only if temperature decreases outward starting at 10^5–10^6 K and h/r stays below one in the inner disk. Optionally persist key parameters to a txt file via the save method before downstream use.

## Related tools

- **pagn** (Python module that implements and solves the Sirko & Goodman 2003 and Thompson et al. 2005 AGN disk equations via SirkoAGN/ThompsonAGN.solve_disk(N=1e4)) — https://gitlab.com/aatrani/pagn
- **numpy** (Provides the numeric arrays behind the solved 1D radial profiles and the Mdot(R) normalization)
- **matplotlib** (Renders the quick-check parameter plot (sk.plot) and the accretion-rate diagnostic (tho.plot_mdot) used to validate the solved disk)
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) interpolates surface-density and thermal gradients over log-radius after the disk is solved (requires scipy ver 1.10.0))

## Examples

```
pip install pagn; from pagn import Sirko; sk = Sirko.SirkoAGN(Mbh=1e8, le=0.5, Mdot=None, alpha=0.01, X=0.7, b=0, opacity='combined'); sk.solve_disk(N=1e4)
```

## Evaluation signals

- solve_disk(N=1e4) completes without errors and yields finite 1D profiles across the radial grid.
- Thompson validity check: tho.plot_mdot() shows Mdot_out below Mdot_c evaluated at r=Rout (e.g. 8.311237e+03 vs 9.606143e+03 Msun/yr).
- Sirko & Goodman quick-check: plotted temperature decreases outward from 10^5–10^6 K and h/r stays below one in the inner disk regions.
- Boundary-value agreement: Mdot at Rin matches the documented 4.815066 Msun/yr (= 1.831417e-01 Mdot_Edd) within numerical tolerance for the Thompson M-sigma configuration.
- Downstream sanity: bolometric luminosity via calculate_lum_sum over 1e-8–1e-3 m matches the reference 177303419977.49695 solar luminosities for the Thompson default disk.

## Limitations

- The GitHub repository is archived and unmaintained; the project moved to GitLab (https://gitlab.com/aatrani/pagn) for the latest version and issues.
- 10^4 grid points is stated as sufficient resolution 'for most Mbh values' — very large or small Mbh configurations may need a different N.
- Custom opacity tables must be provided in a specific required format; incorrect formatting will break instantiation.
- Gradient interpolation for migration torques requires scipy ver 1.10.0 or later for UnivariateSpline.

## Evidence

- [methods] solve_disk at documented resolution: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [readme] README scope statement: "providing fully evolved AGN disk models through parametric 1D profiles (e.g. temperature, density)"
- [results] Thompson instantiation and solution: "obj = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout = rout, Mdot_out=Mdot_out*ct.MSun/ct.yr)"
- [methods] Sirko & Goodman quick-check diagnostic: "To quickly check that the Sirko & Goodman model ran correctly, we can plot some key parameters"
- [methods] Boundary accretion rates: "one can also plot the accretion rate in the Thompson et al. disk"
