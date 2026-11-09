---
name: nested-sampling-posterior-extraction
description: Use when a Bayesian multi-messenger analysis (e.g., GW170817 strain +
  AT2017gfo photometry + GRB170817A afterglow) requires sampling a joint posterior
  and reporting asymmetric credible intervals on parameters like R_1.4, ejecta masses,
  or inclination.
when_to_use_negative:
- The target quantity is a point estimate or a model-selection problem that does not
  require full posterior sampling
- Only a single messenger's data are available and a-posteriori combination of separately
  analyzed messengers is acceptable, rather than a simultaneous joint-likelihood analysis
- Computational budget cannot accommodate nested sampling of a high-dimensional joint
  posterior; likelihood-free inference alternatives exist in NMMA for kilonova light-curve
  estimation
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: NMMA
  role: Framework whose nmma-analysis and lightcurve-analysis entry points assemble
    the joint GW + kilonova + GRB likelihood and priors before sampling
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Inference backend of NMMA through which the nested samplers are invoked
- name: dynesty
  role: Nested sampler used to draw the joint posterior samples; cited via Zenodo
    10.5281/zenodo.3348367
  repo: https://zenodo.org/records/17268284
- name: PyMultiNest
  role: Alternative sampler that can be selected for the analysis
- name: UltraNest
  role: Alternative robust general-purpose Bayesian inference engine usable as the
    sampler
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:bad964b0d835574f886035047cde651b90f8feac2ee3bde7fc8d6c0b09c86720
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - PyMultiNest
  - UltraNest
  - bilby
  - dynesty
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nested-sampling-posterior-extraction

## Summary

Run nested sampling (dynesty, via bilby as the backend of the NMMA framework) over a joint GW + kilonova + GRB afterglow likelihood and extract 90% credible-interval posteriors such as the radius of a 1.4 solar-mass neutron star, R_1.4 = 11.98 (+0.35/-0.40) km.

## When to use

Use when a Bayesian multi-messenger analysis (e.g., GW170817 strain + AT2017gfo photometry + GRB170817A afterglow) requires sampling a joint posterior and reporting asymmetric credible intervals on parameters like R_1.4, ejecta masses, or inclination.

## When NOT to use

- The target quantity is a point estimate or a model-selection problem that does not require full posterior sampling
- Only a single messenger's data are available and a-posteriori combination of separately analyzed messengers is acceptable, rather than a simultaneous joint-likelihood analysis
- Computational budget cannot accommodate nested sampling of a high-dimensional joint posterior; likelihood-free inference alternatives exist in NMMA for kilonova light-curve estimation

## Inputs

- Gravitational-wave likelihood products for GW170817 generated with the nmma-generation entry point
- AT2017gfo multi-band kilonova light-curve photometry
- GRB170817A afterglow light-curve data set (Troja et al.)
- Equation-of-state prior sets based on chiral EFT calculations
- Posterior sample files (e.g., gw_kn_only_posterior_samples.json) for cross-configuration comparison

## Outputs

- Joint posterior samples for EOS parameters (R_1.4), kilonova ejecta masses (dynamical and wind), jet energy, and binary inclination
- 90% credible intervals, e.g., R = 11.98 (+0.35/-0.40) km for the 1.4 solar-mass neutron-star radius
- Corner-style posterior comparison figure (e.g., posterior_comparison_gw_kn_vs_joint.png) quantifying credible-interval width changes
- Fitted kilonova and GRB afterglow light curves

## How to apply

Configure the likelihood within NMMA, which uses bilby as its inference backend, and select dynesty as the nested sampler (PyMultiNest and UltraNest are supported alternatives). Combine likelihood products generated with nmma-generation (GW), lightcurve-analysis (kilonova photometry), and the synthetic Gaussian jet-model GRB afterglow likelihood, with chiral-EFT-based EOS prior sets included. Run nmma-analysis to sample the joint posterior, then marginalize to the target parameter (e.g., R_1.4) and report the median with asymmetric credible-interval uncertainties such as +0.35/-0.40 km. Compare marginal posteriors across analysis configurations (GW+KN-only vs. GW+KN+GRB) by overlaying them corner-style and quantifying credible-interval width changes; note that because the ratio xi between jet energy E0 and disk mass mdisk is a free parameter, no significant EOS constraint is imposed by E0.

## Related tools

- **NMMA** (Framework whose nmma-analysis and lightcurve-analysis entry points assemble the joint GW + kilonova + GRB likelihood and priors before sampling) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Inference backend of NMMA through which the nested samplers are invoked)
- **dynesty** (Nested sampler used to draw the joint posterior samples; cited via Zenodo 10.5281/zenodo.3348367) — https://zenodo.org/records/17268284
- **PyMultiNest** (Alternative sampler that can be selected for the analysis)
- **UltraNest** (Alternative robust general-purpose Bayesian inference engine usable as the sampler)

## Examples

```
nmma-analysis --sampler dynesty --grb-datafile GRB170817A_afterglow.dat --prompt-lightcurve-m banana --samples joint_posterior_samples.json
```

## Evaluation signals

- The recovered R(1.4 solar-mass) must reproduce the reported 90% credible interval R = 11.98 (+0.35/-0.40) km
- Both the kilonova AT2017gfo and the GRB afterglow GRB170817A light curves must be well described by the fitted model
- GW+KN-only posteriors should be statistically consistent with the joint (GW+KN+GRB) posteriors, with only a minimal-yet-noticeable narrowing of credible-interval widths
- The GW+KN-only R_1.4 credible interval must overlap the joint R_1.4 = 11.98 (+0.35/-0.40) km estimate
- Posterior-sample files (e.g., gw_kn_only_posterior_samples.json) and comparison figures (posterior_comparison_gw_kn_vs_joint.png) are produced with the expected marginals

## Limitations

- Adding GRB170817A information yields only a minimal, yet noticeable, improvement in parameter uncertainties over the GW+KN-only analysis
- No significant constraint on the EOS is imposed by the jet energy E0 because the ratio xi between E0 and the disk mass mdisk is treated as a free parameter
- Additional uncertainties arise from limited physics input in the kilonova and semi-analytic GRB models

## Evidence

- [abstract] R_1.4 credible-interval target: [evidence span withheld — anchor status: not_found]
- [methods] backend and sampler: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [methods] dynesty citation: [evidence span withheld — anchor status: not_found]
- [results] GRB improvement finding: "the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added."
- [results] free xi parameter: "no significant constraint on the EOS is imposed by the jet energy E0 as the ratio ξ between it, E0, and the disk mass mdisk is taken as a free parameter"
- [results] model fit validation: "We find that both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis."
