---
name: grb-afterglow-jet-model-fitting
description: Use when a GRB afterglow light-curve data set (e.g., multi-band GRB170817A
  afterglow measurements) must be modeled with a structured-jet afterglow model and
  incorporated into joint multi-messenger Bayesian inference.
when_to_use_negative:
- When only the gravitational-wave signal or the kilonova AT2017gfo is available and
  no GRB afterglow data set exists to constrain the jet
- When a high-fidelity GRB afterglow model is required — the article notes that 'limited
  physics input in semi-analytic GRB models' is an additional uncertainty
edam_operation: http://edamontology.org/operation_0334
tools:
- name: NMMA
  role: Framework providing the afterglow module (Gaussian jet structure, blast-wave
    evolution, F_nu integration) and the lightcurve-generation / lightcurve-analysis
    / nmma-analysis entry points
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend through which NMMA runs the afterglow light-curve inference on the
    GRB170817A data
- name: dynesty
  role: Nested sampler used to sample the afterglow and joint posteriors
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:fbfc527dcddf003392f3d471c57809b045594f707a5de71a5f9ea6495fd11e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: http://edamontology.org/operation_0334
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# grb-afterglow-jet-model-fitting

## Summary

Fits GRB afterglow light-curve data (e.g., GRB170817A from the Troja et al. data set) with a synthetic Gaussian jet-model lightcurve inside the NMMA framework, so the afterglow can enter a simultaneous GW + kilonova + GRB analysis.

## When to use

Use when a GRB afterglow light-curve data set (e.g., multi-band GRB170817A afterglow measurements) must be modeled with a structured-jet afterglow model and incorporated into joint multi-messenger Bayesian inference.

## When NOT to use

- When only the gravitational-wave signal or the kilonova AT2017gfo is available and no GRB afterglow data set exists to constrain the jet
- When a high-fidelity GRB afterglow model is required — the article notes that 'limited physics input in semi-analytic GRB models' is an additional uncertainty

## Inputs

- GRB170817A afterglow light-curve data set collected in Troja et al. (time, flux density, band)
- Gaussian jet-model parameters: E0, theta_c, free ratio xi tying E0 to disk mass mdisk
- Kilonova/posterior-linked parameters: leftover disk mass mdisk from the joint analysis

## Outputs

- Synthetic Gaussian jet-model afterglow lightcurve table (afterglow_lightcurve.dat: time / flux density)
- Posterior samples for the afterglow parameters from lightcurve-analysis (posterior_samples.hdf5)
- Best-fit GRB afterglow lightcurve with credible bands overlaid on observations (bestfit_lightcurves.png)

## How to apply

Implement the Gaussian angular jet structure in the NMMA afterglow module, with energy per solid angle E(theta) = E0*exp(-0.5*(theta/theta_c)^2) and the jet power tied to the leftover disk mass mdisk through the free ratio xi, so no direct EOS constraint is imposed by E0. Evolve the relativistic thin-shell blast wave numerically, including lateral (sideways) expansion of the jet as it decelerates. Compute the observed flux density F_nu by integrating the emission over the equal-arrival-time surface, accounting for light-travel-time delays across the emitting surface. Run NMMA's lightcurve-generation entry point to produce the synthetic afterglow lightcurve over the observing time grid (written to afterglow_lightcurve.dat), then fit the Troja et al. GRB170817A afterglow data set via lightcurve-analysis with the bilby backend and dynesty sampler. Finally, combine the afterglow likelihood with the GW and kilonova likelihoods in nmma-analysis for the joint posterior; the inclination measurement from the GRB is what feeds into the EOS constraint via the kilonova ejecta masses.

## Related tools

- **NMMA** (Framework providing the afterglow module (Gaussian jet structure, blast-wave evolution, F_nu integration) and the lightcurve-generation / lightcurve-analysis / nmma-analysis entry points) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend through which NMMA runs the afterglow light-curve inference on the GRB170817A data)
- **dynesty** (Nested sampler used to sample the afterglow and joint posteriors)

## Evaluation signals

- afterglow_lightcurve.dat exists and parses as a time/flux-density table over the observing time grid
- The fitted synthetic Gaussian jet-model lightcurve qualitatively matches the GRB170817A afterglow behavior reported as 'well described'
- The best-fit afterglow lightcurve with credible bands reproduces the observed GRB170817A afterglow in the joint-analysis comparison plot
- xi is treated as a free parameter; verify the posterior of E0 alone does not spuriously constrain the EOS
- Adding GRB170817A to the GW170817 + AT2017gfo analysis yields a minimal yet noticeable improvement in parameter uncertainties

## Limitations

- Limited physics input in the semi-analytic GRB model is an acknowledged additional uncertainty
- No constraint on the EOS is imposed by the jet energy E0 because the ratio xi between E0 and mdisk is a free parameter
- Reliable interpretation requires astrophysical transient models beyond the parameter-estimation infrastructure itself

## Evidence

- [results] GRB afterglow analyzed with synthetic Gaussian jet model: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve described before57,58"
- [results] xi free parameter means no EOS constraint from E0: "no significant constraint on the EOS is imposed by the jet energy E0 as the ratio ξ between it, E0, and the disk mass mdisk is taken as a free parameter"
- [results] GRB contribution to joint analysis: "the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added."
- [results] afterglow well described by fitted model: "both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis"
- [readme] NMMA lightcurve tutorial: "A tutorial on how to produce simulations of lightcurves is given here"
