---
name: synthetic-lightcurve-generation
description: Use when a joint GW170817/AT2017gfo/GRB170817A analysis requires a forward-modeled
  afterglow lightcurve from a structured (Gaussian) jet, e.g. to fit the Troja et
  al. GRB afterglow data set alongside GW and kilonova data.
when_to_use_negative:
- The task is Bayesian parameter estimation on an already-generated lightcurve (use
  NMMA's nmma-analysis / lightcurve-analysis entry points, not the generation step)
- The signal is a kilonova such as AT2017gfo — this skill produces the semi-analytic
  GRB afterglow lightcurve, not a kilonova model
- High-fidelity jet physics is required — the paper itself flags 'limited physics
  input in ... semi-analytic GRB models' as an additional uncertainty
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: NMMA
  role: Hosts the afterglow module implementing the Gaussian jet structure, thin-shell
    blast-wave evolution with lateral expansion, equal-arrival-time surface integration,
    and the lightcurve-generation entry point
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:1f6454c66c3583312e82d05da398e7dd7a07e34604052ee99f08826767ac627e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# synthetic-lightcurve-generation

## Summary

Generate a synthetic GRB afterglow lightcurve with NMMA's Gaussian angular jet structure, so that the GRB170817A afterglow data can be included in a simultaneous GW + kilonova + afterglow Bayesian analysis.

## When to use

Use when a joint GW170817/AT2017gfo/GRB170817A analysis requires a forward-modeled afterglow lightcurve from a structured (Gaussian) jet, e.g. to fit the Troja et al. GRB afterglow data set alongside GW and kilonova data.

## When NOT to use

- The task is Bayesian parameter estimation on an already-generated lightcurve (use NMMA's nmma-analysis / lightcurve-analysis entry points, not the generation step)
- The signal is a kilonova such as AT2017gfo — this skill produces the semi-analytic GRB afterglow lightcurve, not a kilonova model
- High-fidelity jet physics is required — the paper itself flags 'limited physics input in ... semi-analytic GRB models' as an additional uncertainty

## Inputs

- GRB170817A afterglow light-curve data set collected in Troja et al. (time, flux-density measurements)
- Gaussian jet-structure parameters: E0 (on-axis jet energy), theta_c (opening-angle core), theta (viewing angle), mdisk (disk mass) and free ratio xi linking E0 to mdisk
- Observing time grid for the synthetic lightcurve

## Outputs

- afterglow_lightcurve.dat — a time / flux-density table over the observing time grid
- Synthetic Gaussian jet-model lightcurve usable as the afterglow model in the NMMA joint analysis

## How to apply

Implement the Gaussian angular jet structure in the NMMA afterglow module with energy per solid angle E(theta) = E0*exp(-0.5*(theta/theta_c)^2), tying the jet power to the leftover disk mass mdisk through the free ratio xi. Evolve the relativistic thin-shell blast wave numerically, including lateral (sideways) expansion as the jet decelerates. Compute the observed flux density F_nu by integrating emission over the equal-arrival-time surface, accounting for light-travel-time delays across the emitting surface. Run NMMA's lightcurve-generation entry point over the observing time grid to write afterglow_lightcurve.dat, then verify the file parses as a time/flux-density table and qualitatively reproduces the GRB170817A afterglow behavior reported as well described in the analysis.

## Related tools

- **NMMA** (Hosts the afterglow module implementing the Gaussian jet structure, thin-shell blast-wave evolution with lateral expansion, equal-arrival-time surface integration, and the lightcurve-generation entry point) — https://github.com/nuclear-multimessenger-astronomy/nmma

## Examples

```
nmma-generation  # NMMA lightcurve-generation entry point producing afterglow_lightcurve.dat; see tutorials/tutorial-lightcurve_simulation.ipynb
```

## Evaluation signals

- afterglow_lightcurve.dat exists and parses as a time / flux-density table
- The synthetic Gaussian jet-model lightcurve qualitatively matches the GRB170817A afterglow behavior that the joint analysis reports as well described
- The energy profile satisfies E(theta) = E0*exp(-0.5*(theta/theta_c)^2) with E0 tied to mdisk via the free ratio xi
- The blast-wave evolution includes lateral (sideways) expansion during deceleration
- F_nu is integrated over the equal-arrival-time surface with light-travel-time delays accounted for

## Limitations

- Semi-analytic GRB models carry limited physics input, an acknowledged additional uncertainty in the framework
- No significant constraint on the EOS is imposed by the jet energy E0 because the ratio xi between E0 and the disk mass mdisk is a free parameter
- Reliable astrophysical interpretation of future BNS detections requires astrophysical transient models to advance beyond the parameter-estimation infrastructure presented

## Evidence

- [other] Gaussian jet energy profile with xi: [evidence span withheld — anchor status: not_found]
- [results] Synthetic Gaussian jet-model lightcurve for afterglow data: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve described before"
- [methods] NMMA entry points: "Entry points
- `nmma-generation`
- `nmma-analysis`"
- [results] xi as free parameter limits EOS constraint: "no significant constraint on the EOS is imposed by the jet energy E0 as the ratio ξ between it, E0, and the disk mass mdisk is taken as a free parameter"
- [discussion] Semi-analytic GRB model uncertainty: "additional uncertainties in our work lie in limited physics input in kilonova and semi-analytic GRB and models"
- [readme] Lightcurve simulation tutorial: "A tutorial on how to produce simulations of lightcurves is given here"
