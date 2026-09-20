---
name: disk-boundary-condition-configuration
description: Use when reproducing or parameterizing a Thompson et al. (2005) style
  AGN disk in pagn and you must choose Rout, Mdot_out (in MSun/yr), and Rin before
  calling solve_disk, or when validating inner-boundary accretion rates against reference
  values.
when_to_use_negative:
- You need a Sirko & Goodman (2003) disk model, which is configured with le, alpha,
  and b (e.g. le=0.5, alpha=0.01, b=0) rather than Rout/Mdot_out boundary conditions.
- You require time-dependent or multi-dimensional AGN disk evolution; pagn provides
  parametric 1D profiles only.
- Rout and Mdot_out are already prescribed by the target configuration and you only
  need downstream products like luminosities or migration torques.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Python module providing Thompson.ThompsonAGN, whose constructor accepts Mdot_out,
    Rout, and Rin, and solve_disk(N=1e4) to solve the disk equations
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Numerical arrays for the disk grid and for extracting/normalizing the Mdot(R)
    profile
- name: matplotlib
  role: Visualizing the accretion-rate profile via tho.plot_mdot()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:f7f812e2bfacff380b1b307ec9b531c7a3b021f36ecb766b22cfb6e780335890
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# disk-boundary-condition-configuration

## Summary

Configures the outer boundary conditions (outer radius Rout and outer accretion rate Mdot_out) needed to instantiate and solve the Thompson et al. (2005) AGN disk model with the pagn Python package, producing a fully solved 1D disk profile with well-defined inner-boundary Mdot.

## When to use

Use when reproducing or parameterizing a Thompson et al. (2005) style AGN disk in pagn and you must choose Rout, Mdot_out (in MSun/yr), and Rin before calling solve_disk, or when validating inner-boundary accretion rates against reference values.

## When NOT to use

- You need a Sirko & Goodman (2003) disk model, which is configured with le, alpha, and b (e.g. le=0.5, alpha=0.01, b=0) rather than Rout/Mdot_out boundary conditions.
- You require time-dependent or multi-dimensional AGN disk evolution; pagn provides parametric 1D profiles only.
- Rout and Mdot_out are already prescribed by the target configuration and you only need downstream products like luminosities or migration torques.

## Inputs

- Black-hole mass Mbh (or sigma = 300 km/s for the M-sigma scaling 2e8 MSun * (sigma/200 km/s)^4)
- Outer radius Rout (e.g. Rout = Rs*1e7)
- Outer accretion rate Mdot_out = 1.5e-2 in MSun/yr, converted with ct.MSun/ct.yr
- Model parameters epsilon = 1e-3, m = 0.2, xi = 1.0; optional custom opacity table
- Grid resolution N = 1e4 for solve_disk

## Outputs

- Solved Thompson et al. AGN disk object with radially varying Mdot(R) profile
- Boundary values: Mdot_c at Rout (~9.606143e+03 MSun/yr), Mdot_out (~8.311237e+03 MSun/yr), Mdot at Rin (~4.815066 MSun/yr = 1.831417e-01 Mdot_Edd)
- Optional plots via tho.plot_mdot() and a saved parameter file (e.g. thompson_mdot.txt)

## How to apply

Instantiate Thompson.ThompsonAGN with sigma = 300 km/s, Mbh from the M-sigma relation (2e8 MSun * (sigma/200 km/s)^4), epsilon = 1e-3, m = 0.2, xi = 1.0, and pass the outer accretion rate Mdot_out = 1.5e-2 converted to physical units as Mdot_out*ct.MSun/ct.yr together with Rout (e.g. Rout = Rs*1e7); leaving Mdot_out/Rout/Rin as None lets the package defaults and scalings apply. Solve the disk on a grid with tho.solve_disk(N=1e4), since 10^4 grid points are documented as sufficient resolution for most Mbh values. Extract the radially varying Mdot(R) and normalize by the Eddington accretion rate (Mdot_Edd ~ 26.29148 MSun/yr for this configuration). Judge correctness by checking the outer-boundary hierarchy (Mdot_out ~8.311237e+03 MSun/yr falls below Mdot_c ~9.606143e+03 MSun/yr at Rout) and by matching the inner-boundary value Mdot(Rin) ~4.815066 MSun/yr (= 1.831417e-01 Mdot_Edd) within numerical tolerance.

## Related tools

- **pagn** (Python module providing Thompson.ThompsonAGN, whose constructor accepts Mdot_out, Rout, and Rin, and solve_disk(N=1e4) to solve the disk equations) — https://gitlab.com/aatrani/pagn
- **numpy** (Numerical arrays for the disk grid and for extracting/normalizing the Mdot(R) profile)
- **matplotlib** (Visualizing the accretion-rate profile via tho.plot_mdot())

## Examples

```
tho = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout=rout, Mdot_out=1.5e-2*ct.MSun/ct.yr); tho.solve_disk(N=1e4)
```

## Evaluation signals

- Mdot at r = Rin matches the reference value 4.815066 MSun/yr (= 1.831417e-01 Mdot_Edd) within numerical tolerance.
- Mdot_out (~8.311237e+03 MSun/yr) is strictly below Mdot_c (~9.606143e+03 MSun/yr) at the outer boundary r = Rout.
- Inner accretion rate is ~5 MSun/yr, deemed sufficient to fuel a bright AGN.
- solve_disk was run with N=1e4 grid points, the documented sufficient resolution for most Mbh values.
- Mdot(R) normalized by Mdot_Edd (~26.29148 MSun/yr for this configuration) is finite and monotonic across the solved grid.

## Limitations

- The GitHub repository (DariaGangardt/pAGN) is archived and unmaintained; the project has moved to https://gitlab.com/aatrani/pagn.
- If Mdot_out/Rout/Rin are passed as None, package defaults and scalings are used, so boundary values depend on the internal M-sigma/default logic rather than user-supplied conditions.
- 10^4 grid points are 'sufficient for most Mbh values', implying other mass regimes may need different resolution.

## Evidence

- [results] Thompson disk was configured with Rout = Rs*1e7 and Mdot_out = 1.5e-2 MSun/yr: "Mdot_out = 1.5e-2
            obj = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout = rout, Mdot_out=Mdot_out*ct.MSun/ct.yr)"
- [methods] solve_disk with N=1e4 grid points: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [readme] pagn solves Sirko & Goodman 2003 and Thompson et al. 2005 disk models: "`pagn` is a Python module to solve the equations from [Sirko and Goodman 2003](https://academic.oup.com/mnras/article/341/2/501/1105444) and [Thompson et al. 2005]"
- [methods] plot_mdot visualizes the Thompson disk accretion rate: "one can also plot the accretion rate in the Thompson et al. disk:

.. code:: ipython3

    tho.plot_mdot()"
- [abstract] pagn is the first public implementation of these accretion-disc models: "Our findings are implemented in the pAGN module for the Python programming language, which is the first public implementation of these accretion-disc models."
