---
name: accretion-rate-boundary-check
description: Use when a ThompsonAGN disk has been configured with a user-supplied
  Mdot_out and Rout, and downstream work (e.g. migration-torque profiles) requires
  proof that Mdot_out < Mdot_c at the outer boundary.
when_to_use_negative:
- The disk was built with Sirko.SirkoAGN, which is parameterized by Mdot/le rather
  than an outer Mdot_out boundary — this Thompson-specific check does not apply
- You only need disk luminosities or temperature profiles and never impose a boundary
  Mdot_out, so no sub-criticality question arises
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Provides Thompson.ThompsonAGN and its plot_mdot() diagnostic used to visualize
    the accretion-rate profile and verify the sub-critical boundary condition
  repo: https://gitlab.com/aatrani/pagn
- name: matplotlib
  role: Rendering backend for the accretion-rate plot produced by plot_mdot()
- name: Python
  role: Execution environment for the pAGN check workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:ee7b11ebc90889c9f8eeff61b5f1f926d3e4185324c5d2529e61f9c9c44cbc2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# accretion-rate-boundary-check

## Summary

A validation step for pAGN Thompson et al. 2005 disk models that plots and inspects the radial accretion-rate profile (tho.plot_mdot()) to confirm the imposed outer mass flux Mdot_out is below the critical accretion rate Mdot_c evaluated at r = Rout, ensuring the disk is in the sub-critical regime the model assumes.

## When to use

Use when a ThompsonAGN disk has been configured with a user-supplied Mdot_out and Rout, and downstream work (e.g. migration-torque profiles) requires proof that Mdot_out < Mdot_c at the outer boundary.

## When NOT to use

- The disk was built with Sirko.SirkoAGN, which is parameterized by Mdot/le rather than an outer Mdot_out boundary — this Thompson-specific check does not apply
- You only need disk luminosities or temperature profiles and never impose a boundary Mdot_out, so no sub-criticality question arises

## Inputs

- Solved pAGN Thompson.ThompsonAGN object (post tho.solve_disk(N=1e4)) with radial profiles of density, temperature, scale height, and opacity
- Outer boundary mass flux Mdot_out in MSun/yr and outer radius Rout in units of Rs used at instantiation

## Outputs

- Accretion-rate radial profile plot (tho.plot_mdot())
- Boolean/qualitative verdict that Mdot_out < Mdot_c evaluated at r = Rout (sub-critical regime confirmed)

## How to apply

After instantiating ThompsonAGN (e.g. Mbh = 1e6 MSun, Mdot_out = 1.5e-2 MSun/yr, Rout = Rs*1e7) and solving the disk with tho.solve_disk(N=1e4), call tho.plot_mdot() to display the accretion-rate profile across the radial grid. Read off the critical accretion rate Mdot_c at r = Rout and compare it against the imposed outer boundary value Mdot_out: the model is valid only if Mdot_out lies below Mdot_c at that radius. If the check fails, the supplied Mdot_out is inconsistent with the disk solution and the configuration (Mdot_out or Rout) must be revised before proceeding to torque or luminosity calculations. This diagnostic is a gate: downstream steps such as spline interpolation of surface-density gradients and Grishin et al. 2023 torque computation are only trustworthy on a sub-critical disk.

## Related tools

- **pagn** (Provides Thompson.ThompsonAGN and its plot_mdot() diagnostic used to visualize the accretion-rate profile and verify the sub-critical boundary condition) — https://gitlab.com/aatrani/pagn
- **matplotlib** (Rendering backend for the accretion-rate plot produced by plot_mdot())
- **Python** (Execution environment for the pAGN check workflow)

## Examples

```
from pagn import Thompson; tho = Thompson.ThompsonAGN(Mbh=1e6*ct.MSun, Mdot_out=1.5e-2*ct.MSun/ct.yr, Rout=Rs*1e7); tho.solve_disk(N=1e4); tho.plot_mdot()
```

## Evaluation signals

- Mdot_out (1.5e-2 MSun/yr in the reference configuration) must be strictly below Mdot_c evaluated at r = Rout (Rs*1e7)
- The accretion-rate profile from tho.plot_mdot() must be finite across the full N=1e4 radial grid with no overflow artifacts
- The check must pass before any downstream torque-coefficient CSV is accepted; torque_coefficients.csv must contain finite values only on a validated disk

## Limitations

- The check is specific to the Thompson et al. 2005 model where Mdot_out and Rout are user-supplied boundary conditions; it does not generalize to Sirko & Goodman configurations
- 10^4 grid points is stated to be sufficient resolution for most Mbh values, but very different Mbh choices may warrant a denser grid before trusting the Mdot_c comparison
- The GitHub repository DariaGangardt/pAGN is archived and unmaintained; current code lives at gitlab.com/aatrani/pagn

## Evidence

- [results] Run the accretion-rate diagnostic and confirm Mdot_out is sub-critical: [evidence span withheld — anchor status: not_found]
- [results] Thompson disk configured with Rout and Mdot_out: [evidence span withheld — anchor status: not_found]
- [methods] plot accretion rates in the Thompson et al. disk: "one can also plot the accretion rate in the Thompson et al. disk:

.. code:: ipython3

    tho.plot_mdot()"
- [methods] solve_disk resolution guidance: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
