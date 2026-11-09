---
name: relativistic-blastwave-dynamics-integration
description: Use when reconstructing or extending the GRB afterglow forward model
  for GRB170817A-style data — i.e. when a structured-jet synthetic lightcurve is needed
  as the EM afterglow component of a simultaneous NMMA analysis.
when_to_use_negative:
- The target is a kilonova (AT2017gfo-like thermal) lightcurve rather than a relativistic
  afterglow — use NMMA's kilonova/SVD models and the lightcurve_simulation tutorial
  instead.
- The analysis goal is to constrain the equation of state directly through the jet
  energy E0 — this is not possible here because the ratio xi between E0 and mdisk
  is a free parameter.
- High-fidelity hydrodynamical jet simulations are required; this skill uses a semi-analytic
  Gaussian jet model with known limited physics input.
tools:
- name: NMMA
  role: 'Hosts the afterglow module: Gaussian jet structure, blast-wave evolution,
    equal-arrival-time flux integration, and the lightcurve-generation entry points
    that write afterglow_lightcurve.dat'
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend through which the synthetic afterglow lightcurve enters the Bayesian
    joint analysis of GW, kilonova, and afterglow data
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: dynesty
  role: Nested sampler used to fit the afterglow model parameters within NMMA's analysis
    entry points
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:6e4a2f2d4b785cb1da092b3e7d5938fdb96411482f01ba5c42f96a348aa82197
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# relativistic-blastwave-dynamics-integration

## Summary

Forward-models GRB afterglow emission by numerically evolving a relativistic thin-shell blast wave driven by a Gaussian angular jet structure, integrating emission over equal-arrival-time surfaces to produce a synthetic lightcurve for joint GW/kilonova/afterglow inference in NMMA.

## When to use

Use when reconstructing or extending the GRB afterglow forward model for GRB170817A-style data — i.e. when a structured-jet synthetic lightcurve is needed as the EM afterglow component of a simultaneous NMMA analysis.

## When NOT to use

- The target is a kilonova (AT2017gfo-like thermal) lightcurve rather than a relativistic afterglow — use NMMA's kilonova/SVD models and the lightcurve_simulation tutorial instead.
- The analysis goal is to constrain the equation of state directly through the jet energy E0 — this is not possible here because the ratio xi between E0 and mdisk is a free parameter.
- High-fidelity hydrodynamical jet simulations are required; this skill uses a semi-analytic Gaussian jet model with known limited physics input.

## Inputs

- Gaussian jet parameters: E0 (energy normalization), theta_c (opening angle), xi (free E0-to-mdisk ratio, Eq. 18), observer inclination
- Microphysical blast-wave parameters for the thin-shell evolution
- Observing time grid for the afterglow lightcurve (Troja et al. GRB afterglow data set epochs)

## Outputs

- afterglow_lightcurve.dat — a time/flux-density table of the synthetic Gaussian jet-model afterglow lightcurve
- Synthetic lightcurve consumed by nmma-analysis/lightcurve-analysis for joint GW + kilonova + afterglow inference

## How to apply

Implement the Gaussian angular jet structure with energy per solid angle E(theta) = E0*exp(-0.5*(theta/theta_c)^2), tying the jet power to the leftover disk mass mdisk through the free ratio xi (Eq. 18). Evolve the relativistic thin-shell blast wave numerically, including lateral (sideways) expansion as the jet decelerates, since lateral spreading shapes the post-peak decline of the GRB170817A afterglow. Compute the observed flux density F_nu by integrating emission over the equal-arrival-time surface, explicitly accounting for light-travel-time delays across the emitting surface. Run NMMA's lightcurve-generation entry point over the observing time grid and write the result to afterglow_lightcurve.dat, which enters the joint analysis of GW170817, AT2017gfo, and the afterglow via the Troja et al. data set.

## Related tools

- **NMMA** (Hosts the afterglow module: Gaussian jet structure, blast-wave evolution, equal-arrival-time flux integration, and the lightcurve-generation entry points that write afterglow_lightcurve.dat) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend through which the synthetic afterglow lightcurve enters the Bayesian joint analysis of GW, kilonova, and afterglow data) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **dynesty** (Nested sampler used to fit the afterglow model parameters within NMMA's analysis entry points) — https://github.com/nuclear-multimessenger-astronomy/nmma

## Examples

```
jupyter notebook tutorials/tutorial-lightcurve_simulation.ipynb
```

## Evaluation signals

- afterglow_lightcurve.dat exists, parses as a time/flux-density table, and covers the requested observing time grid
- The synthetic Gaussian jet-model lightcurve qualitatively matches the GRB170817A afterglow behavior that the analysis reports as well described
- The F_nu computation reflects light-travel-time delays across the equal-arrival-time surface (peak time consistent with the chosen theta_c and E0)
- Lateral expansion is active in the blast-wave evolution (post-peak decay steepens, as expected for a decelerating structured jet)
- Sampling over xi confirms it remains unconstrained/free, with the afterglow contributing primarily an inclination constraint

## Limitations

- Limited physics input in the semi-analytic GRB model is an acknowledged additional uncertainty in the framework.
- No significant constraint on the EOS is imposed by the jet energy E0 because the ratio xi between E0 and the disk mass mdisk is taken as a free parameter.
- The afterglow constrains the EOS only indirectly: the GRB's inclination measurement imposes a constraint on the EOS via the kilonova ejecta-mass measurements, and the improvement on parameter uncertainties when adding GRB170817A is minimal yet noticeable.

## Evidence

- [results] Gaussian jet-model lightcurve: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve"
- [methods] blast-wave evolution: [evidence span withheld — anchor status: not_found]
- [readme] lightcurve simulation tutorial: "A tutorial on how to produce simulations of lightcurves is given here"
- [results] free xi parameter: "no significant constraint on the EOS is imposed by the jet energy E0 as the ratio ξ between it, E0, and the disk mass mdisk is taken as a free parameter"
