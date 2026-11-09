---
name: equal-arrival-time-flux-integration
description: Use when a joint GW + kilonova + GRB-afterglow analysis (e.g., GW170817/AT2017gfo/GRB170817A)
  needs a model afterglow lightcurve from a structured jet to fit observed afterglow
  flux-density data.
when_to_use_negative:
- The electromagnetic data are kilonova (e.g., AT2017gfo) photometry only — use the
  kilonova lightcurve models, not the afterglow jet model
- The analysis targets only the gravitational-wave signal or the EOS constraint without
  any GRB afterglow component
- A fully numerical (e.g., hydrodynamical) afterglow simulation is required rather
  than NMMA's semi-analytic Gaussian jet model
tools:
- name: NMMA
  role: Hosts the afterglow module implementing the Gaussian jet structure, blast-wave
    evolution with lateral expansion, equal-arrival-time flux integration, and the
    lightcurve-generation entry point writing afterglow_lightcurve.dat
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend used by NMMA for the Bayesian analysis in which the synthetic afterglow
    lightcurve is fitted to the GRB170817A data
  repo: https://arxiv.org/pdf/1811.02042
- name: dynesty
  role: Nested sampler (one of several supported backends) used for the joint inference
    that consumes the generated afterglow lightcurve
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:3defad21d098bf9de8f0f34c0eb454f0ffeab40bb0617f9fed5e9b32c3088af3
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
yes_rate: 0.75
n_claims: 8
---

# equal-arrival-time-flux-integration

## Summary

Generates a synthetic GRB afterglow lightcurve in NMMA by evolving a Gaussian-structured relativistic jet blast wave (E(θ)=E0·exp(−0.5(θ/θc)²)) with lateral expansion and integrating emission over equal-arrival-time surfaces, producing a time/flux-density table used to fit the GRB170817A afterglow data.

## When to use

Use when a joint GW + kilonova + GRB-afterglow analysis (e.g., GW170817/AT2017gfo/GRB170817A) needs a model afterglow lightcurve from a structured jet to fit observed afterglow flux-density data.

## When NOT to use

- The electromagnetic data are kilonova (e.g., AT2017gfo) photometry only — use the kilonova lightcurve models, not the afterglow jet model
- The analysis targets only the gravitational-wave signal or the EOS constraint without any GRB afterglow component
- A fully numerical (e.g., hydrodynamical) afterglow simulation is required rather than NMMA's semi-analytic Gaussian jet model

## Inputs

- Jet-structure parameters: E0 (isotropic-equivalent energy scale), theta_c (Gaussian core angle), xi (free ratio linking E0 to disk mass mdisk)
- Microphysical afterglow parameters of the relativistic thin-shell blast wave
- Observing time grid and observing bands/frequencies for the afterglow flux-density calculation
- GRB afterglow light-curve data set (e.g., the data set collected in Troja et al.) for downstream fitting

## Outputs

- afterglow_lightcurve.dat: a time / flux-density table of the synthetic Gaussian jet-model afterglow lightcurve

## How to apply

Implement the Gaussian angular jet structure in the NMMA afterglow module with energy per solid angle E(theta) = E0*exp(-0.5*(theta/theta_c)^2), tying the jet power to the leftover disk mass mdisk through the free ratio xi. Numerically evolve the relativistic thin-shell blast wave, including lateral (sideways) expansion as the jet decelerates. For each observing time, integrate the emission over the equal-arrival-time surface, accounting for light-travel-time delays across the emitting surface, to obtain the observed flux density F_nu. Run NMMA's lightcurve-generation entry point over the observing time grid to write afterglow_lightcurve.dat, which is then analyzed with the synthetic Gaussian jet-model lightcurve in the joint inference (via bilby-backed samplers such as dynesty).

## Related tools

- **NMMA** (Hosts the afterglow module implementing the Gaussian jet structure, blast-wave evolution with lateral expansion, equal-arrival-time flux integration, and the lightcurve-generation entry point writing afterglow_lightcurve.dat) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend used by NMMA for the Bayesian analysis in which the synthetic afterglow lightcurve is fitted to the GRB170817A data) — https://arxiv.org/pdf/1811.02042
- **dynesty** (Nested sampler (one of several supported backends) used for the joint inference that consumes the generated afterglow lightcurve) — https://zenodo.org/records/17268284

## Examples

```
jupyter notebook tutorials/tutorial-lightcurve_simulation.ipynb
```

## Evaluation signals

- afterglow_lightcurve.dat exists and parses as a two-column time/flux-density table covering the requested observing time grid
- The synthetic lightcurve qualitatively reproduces the GRB170817A afterglow behavior that the analysis reports as well described (rise, peak, and power-law decay)
- The jet energy follows the prescribed Gaussian form E(theta) = E0*exp(-0.5*(theta/theta_c)^2) and the jet power is linked to mdisk via the free ratio xi
- Lightcurve changes as expected when lateral expansion is toggled, since deceleration with sideways expansion is explicitly modeled
- The fitted lightcurve is consistent with the Troja et al. GRB afterglow data set used in the joint GW170817 + AT2017gfo + GRB170817A analysis

## Limitations

- The GRB model is semi-analytic; limited physics input in semi-analytic GRB models is an additional uncertainty of the framework
- No significant constraint on the EOS is imposed by the jet energy E0 because the ratio xi between E0 and the disk mass mdisk is a free parameter
- Reliable astrophysical interpretation of future BNS detections requires further advance of astrophysical transient models beyond the parameter-estimation infrastructure

## Evidence

- [results] GRB afterglow light-curve data analyzed with a synthetic Gaussian jet-model lightcurve: [evidence span withheld — anchor status: not_found]
- [results] Afterglow and kilonova both well described: "We find that both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis."
- [discussion] Semi-analytic GRB model uncertainty: "additional uncertainties in our work lie in limited physics input in kilonova and semi-analytic GRB and models"
- [results] xi free parameter prevents EOS constraint from E0: "no significant constraint on the EOS is imposed by the jet energy E0 as the ratio ξ between it, E0, and the disk mass mdisk is taken as a free parameter"
- [readme] Lightcurve simulation tutorial in the repository: "A tutorial on how to produce simulations of lightcurves is given here"
