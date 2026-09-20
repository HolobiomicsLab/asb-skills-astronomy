---
name: migration-torque-validation-against-reference
description: Use when a re-implemented migration torque-coefficient module must be
  checked for correctness against published pAGN reference outputs before it is trusted
  for AGN-disk migration studies.
when_to_use_negative:
- The disk model has not yet been solved with pAGN solve_disk (no radial profiles
  exist to differentiate for gradients)
- The migrator/central-object configuration differs from the published use case (e.g.
  different mass ratio than q=5e-6) and no reference outputs exist for that configuration
- You only need the disk structure itself (temperature, density profiles) rather than
  migration torque coefficients — use the pAGN disk-solving skill instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Provides the solved AGN disk models and the published reference use-case torque
    values used for validation
  repo: https://gitlab.com/aatrani/pagn
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) differentiations supply the dSigmadR,
    dTdR, dPdR gradients feeding the torque coefficients
- name: numpy
  role: Array handling of radial profiles and spline inputs/outputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:99231e4888b129fc01b16f2f93cb8ae69f786920b5287e8c01599150d59c819d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# migration-torque-validation-against-reference

## Summary

Validates a reconstructed thermal-torque coefficient module (gamma_thermal, CI_jm17_iso, CI_jm17_tot, CI_p10, CL) by confirming it runs standalone and reproduces the published pAGN use-case torque values for a 50 solar-mass BH migrator around a 1e6 solar-mass SMBH with mass ratio q = 5e-6.

## When to use

Use when a re-implemented migration torque-coefficient module must be checked for correctness against published pAGN reference outputs before it is trusted for AGN-disk migration studies.

## When NOT to use

- The disk model has not yet been solved with pAGN solve_disk (no radial profiles exist to differentiate for gradients)
- The migrator/central-object configuration differs from the published use case (e.g. different mass ratio than q=5e-6) and no reference outputs exist for that configuration
- You only need the disk structure itself (temperature, density profiles) rather than migration torque coefficients — use the pAGN disk-solving skill instead

## Inputs

- Reconstructed torque-coefficient module (gamma_thermal, CI_jm17_iso, CI_jm17_tot, CI_p10, CL functions)
- Solved pAGN disk profiles (surface density, temperature, pressure vs radius) from Sirko & Goodman and Thompson models
- Reference torque values from the pAGN use-case runs (50 Msun BH, Mbh=1e6 Msun, q=5e-6)

## Outputs

- Standalone-execution pass/fail (script_runs)
- Computed torque coefficients compared to reference values (output_matches_reference)

## How to apply

First verify the module executes standalone as a script (script_runs check). Then evaluate it on the pAGN reference use case: a 50 solar-mass black-hole migrator orbiting a 1e6 solar-mass central SMBH (mass ratio q = 5e-6) in disks solved with Sirko & Goodman (le=0.5, alpha=0.01, b=0) and Thompson (Rout=Rs*1e7, Mdot_out=1.5e-2 MSun/yr) models via solve_disk(N=1e4). Compute gamma_thermal, CI_jm17_iso, CI_jm17_tot, CI_p10, and CL from disk gradients obtained by differentiating UnivariateSpline (k=3, s=0.005, ext=0, scipy >= 1.10.0) interpolants of log10 Sigma and log10 T over log10 R. Compare computed torque values against the reference values of the pAGN use-case runs (output_matches_reference); the source states no numeric tolerance, so agreement is judged directly against the published use-case outputs.

## Related tools

- **pagn** (Provides the solved AGN disk models and the published reference use-case torque values used for validation) — https://gitlab.com/aatrani/pagn
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) differentiations supply the dSigmadR, dTdR, dPdR gradients feeding the torque coefficients)
- **numpy** (Array handling of radial profiles and spline inputs/outputs)

## Examples

```
from pagn import Sirko; sk = Sirko.SirkoAGN(Mbh=1e6*ct.MSun, le=0.5, alpha=0.01, b=0); sk.solve_disk(N=1e4); gam = gamma_thermal(sk, q=5e-6)
```

## Evaluation signals

- script_runs: the reconstructed module executes standalone without errors
- output_matches_reference: computed torque values reproduce the reference values of the pAGN use-case runs (no numeric tolerance stated; judged against published use-case outputs)
- Gradients must come from UnivariateSpline(k=3, s=0.005, ext=0) over log10(R), matching the source spline specification
- Environment check: scipy >= 1.10.0 required for the spline behavior used in the reference implementation
- Both disk models (Sirko & Goodman and Thompson) must be validated, not just one

## Limitations

- The source states no numeric tolerance for agreement with reference values; correctness is judged qualitatively against published use-case outputs
- The GitHub repository is archived and no longer maintained; the project has moved to gitlab.com/aatrani/pagn
- Spline gradients require scipy >= 1.10.0; older versions may behave differently
- solve_disk(N=1e4) is stated to be sufficient resolution 'for most Mbh values' — other SMBH masses may need different N

## Evidence

- [results] Torque calculations used the equations of Grishin et al. 2023: "We use the equations from `Grishin et al. 2023 <https://arxiv.org/pdf/2307.07546.pdf>`__"
- [results] Migration torques computed for 50 Msun BH with q=5e-6: "looking at the migration torques a 50 M\ :math:`_\odot` BH in an AGN would experience while orbiting a central BH"
- [results] Spline interpolation specification: "Sigmalog10_spline = UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)"
- [results] Sirko & Goodman use-case configuration: [evidence span withheld — anchor status: not_found]
- [methods] Disk solved with 1e4 grid points: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [readme] Repository archived, moved to GitLab: "This repository is archived and no longer maintained. The project has moved to GitLab"
- [other] Validation of task_008: script_runs and output_matches_reference with no numeric tolerance: [evidence span withheld — anchor status: not_found]
