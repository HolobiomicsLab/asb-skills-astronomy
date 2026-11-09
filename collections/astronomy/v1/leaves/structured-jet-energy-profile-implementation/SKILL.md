---
name: structured-jet-energy-profile-implementation
description: Use when modeling GRB afterglow emission with an angular jet structure
  — e.g., fitting the GRB170817A afterglow lightcurve (Troja et al. data set) jointly
  with GW and kilonova data in NMMA, or generating a synthetic Gaussian-jet lightcurve
  table.
when_to_use_negative:
- Modeling the kilonova AT2017gfo lightcurve itself — that uses NMMA's kilonova models
  (e.g., SVD-based surrogates via create-svdmodel), not the structured GRB jet model
- Attempting EOS constraints from jet energy E0 alone — since the ratio xi between
  E0 and mdisk is a free parameter, E0 does not constrain the EOS
- Prompt-emission (gamma-ray spike) GRB modeling — this skill covers the afterglow
  blast-wave lightcurve only
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NMMA
  role: Framework whose afterglow module hosts the Gaussian jet structure, the thin-shell
    blast-wave evolution, and the lightcurve-generation entry point (nmma-generation)
    that writes afterglow_lightcurve.dat
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend used by NMMA for the Bayesian sampling of the afterglow parameters
    (e.g., E0, theta_c, inclination) against the GRB170817A afterglow data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:db46f409fcf89789dc39d513680e4a0d5acb875eebd393f7f9ec6b04074e7985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structured-jet-energy-profile-implementation

## Summary

Implements a structured (Gaussian) relativistic jet energy profile in NMMA's afterglow module to generate synthetic GRB afterglow lightcurves, in which the jet energy E0 is tied to the leftover disk mass mdisk through the free ratio xi. This is the model used to fit the GRB170817A afterglow in the joint GW170817 + AT2017gfo + afterglow analysis.

## When to use

Use when modeling GRB afterglow emission with an angular jet structure — e.g., fitting the GRB170817A afterglow lightcurve (Troja et al. data set) jointly with GW and kilonova data in NMMA, or generating a synthetic Gaussian-jet lightcurve table.

## When NOT to use

- Modeling the kilonova AT2017gfo lightcurve itself — that uses NMMA's kilonova models (e.g., SVD-based surrogates via create-svdmodel), not the structured GRB jet model
- Attempting EOS constraints from jet energy E0 alone — since the ratio xi between E0 and mdisk is a free parameter, E0 does not constrain the EOS
- Prompt-emission (gamma-ray spike) GRB modeling — this skill covers the afterglow blast-wave lightcurve only

## Inputs

- GRB170817A afterglow light-curve data set collected in Troja et al. (multi-band flux densities vs. time)
- Jet-structure parameters: on-axis energy E0, Gaussian core angle theta_c, and the free ratio xi linking E0 to the leftover disk mass mdisk
- Observer time grid and observing frequencies for the synthetic lightcurve
- NMMA afterglow module (Gaussian jet model) configuration for the afterglow calculation

## Outputs

- afterglow_lightcurve.dat — synthetic Gaussian jet-model lightcurve as a time/flux-density table
- Posterior constraints on afterglow parameters (jet energy E0, viewing angle/inclination) from the joint GW + kilonova + afterglow analysis

## How to apply

Implement the Gaussian angular jet structure in the NMMA afterglow module with energy per solid angle E(theta) = E0*exp(-0.5*(theta/theta_c)^2), tying the jet power to the leftover disk mass mdisk via Eq. 18 through the free ratio xi. Evolve the relativistic thin-shell blast wave numerically, including lateral (sideways) expansion of the jet as it decelerates. Compute the observed flux density F_nu by integrating the emission over the equal-arrival-time surface, accounting for light-travel-time delays across the emitting surface. Run NMMA's lightcurve-generation entry point (nmma-generation) over the observing time grid to write the synthetic lightcurve to afterglow_lightcurve.dat. When fitting the GRB170817A afterglow, treat E0 as a sampled parameter but keep xi free — this is why E0 imposes no EOS constraint, while the resulting viewing-angle (inclination) constraint feeds back into the EOS via kilonova ejecta masses.

## Related tools

- **NMMA** (Framework whose afterglow module hosts the Gaussian jet structure, the thin-shell blast-wave evolution, and the lightcurve-generation entry point (nmma-generation) that writes afterglow_lightcurve.dat) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend used by NMMA for the Bayesian sampling of the afterglow parameters (e.g., E0, theta_c, inclination) against the GRB170817A afterglow data)

## Evaluation signals

- afterglow_lightcurve.dat exists, parses as a time/flux-density table over the requested observing time grid
- The synthetic Gaussian jet-model lightcurve qualitatively matches the GRB170817A afterglow behavior the paper reports as well described
- The angular profile follows E(theta) = E0*exp(-0.5*(theta/theta_c)^2) and the E0–mdisk coupling reproduces the free-ratio xi parameterization of Eq. 18
- The computed F_nu shows the expected signatures of lateral expansion and equal-arrival-time integration (light-travel-time delays across the emitting surface)
- In the joint fit, the GRB inclination constraint acts on the EOS through kilonova ejecta masses, while E0 itself yields no EOS constraint (xi free)

## Limitations

- Limited physics input in the semi-analytic GRB model is an acknowledged additional uncertainty of the framework
- The jet energy E0 cannot constrain the EOS because the ratio xi between E0 and mdisk is treated as a free parameter
- Validation of the synthetic lightcurve against GRB170817A is qualitative rather than a strict quantitative benchmark

## Evidence

- [results] GRB afterglow data modeled with Gaussian jet: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve described before57,58"
- [results] E0 imposes no EOS constraint (free xi): "no significant constraint on the EOS is imposed by the jet energy E0 as the ratio ξ between it, E0, and the disk mass mdisk is taken as a free parameter"
- [results] Afterglow well described by the analysis: "We find that both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis."
- [discussion] Semi-analytic GRB model uncertainty: "additional uncertainties in our work lie in limited physics input in kilonova and semi-analytic GRB and models"
- [readme] NMMA lightcurve simulation tutorial: "A tutorial on how to produce simulations of lightcurves is given here [tutorial-lightcurve_simulation.ipynb]"
