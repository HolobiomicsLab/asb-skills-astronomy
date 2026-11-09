---
name: multimessenger-bayesian-parameter-estimation
description: Use when you must jointly infer compact-binary-merger source parameters
  from combined gravitational-wave strain data, kilonova photometry (e.g. AT2017gfo),
  and GRB afterglow light curves (e.g. GRB170817A, Troja et al. data set) rather than
  combining single-messenger posteriors a posteriori.
when_to_use_negative:
- When you only need to combine existing single-messenger posteriors a posteriori,
  the a posteriori combining route the paper contrasts itself against is simpler than
  this simultaneous framework
- When no electromagnetic counterpart (kilonova or GRB afterglow) data exist for the
  event — GW-only analyses do not require the joint likelihood machinery
- 'When the goal is an EOS constraint from jet energy E0 alone: because the ratio
  ξ between E0 and the disk mass mdisk is a free parameter, no significant EOS constraint
  is imposed by E0'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: NMMA
  role: Framework providing the nmma-generation, nmma-analysis, and lightcurve-analysis
    entry points for the simultaneous multi-messenger joint analysis
  repo: github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend used by NMMA for Bayesian inference of the kilonova light curves
- name: dynesty
  role: Nested sampler used to sample the joint posterior
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:a2fd56790c4a0bf59273265bd50b27bbaf7f99dd7f264823dbf803efabf950cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# multimessenger-bayesian-parameter-estimation

## Summary

Joint Bayesian inference of gravitational-wave, kilonova, and GRB afterglow data within the NMMA framework to constrain neutron-star properties, yielding for GW170817/AT2017gfo/GRB170817A a radius of a 1.4 M⊙ neutron star of R = 11.98 (+0.35/−0.40) km.

## When to use

Use when you must jointly infer compact-binary-merger source parameters from combined gravitational-wave strain data, kilonova photometry (e.g. AT2017gfo), and GRB afterglow light curves (e.g. GRB170817A, Troja et al. data set) rather than combining single-messenger posteriors a posteriori.

## When NOT to use

- When you only need to combine existing single-messenger posteriors a posteriori, the a posteriori combining route the paper contrasts itself against is simpler than this simultaneous framework
- When no electromagnetic counterpart (kilonova or GRB afterglow) data exist for the event — GW-only analyses do not require the joint likelihood machinery
- When the goal is an EOS constraint from jet energy E0 alone: because the ratio ξ between E0 and the disk mass mdisk is a free parameter, no significant EOS constraint is imposed by E0

## Inputs

- GW170817 gravitational-wave strain data / likelihood products
- AT2017gfo kilonova photometric light-curve data
- GRB170817A afterglow light-curve data set (Troja et al.)
- Chiral-EFT-based EOS prior sets

## Outputs

- Joint posterior samples over GW, kilonova, and GRB jet-model parameters
- Posterior distribution of R(1.4 M⊙), reported as a 90% credible interval (11.98 +0.35/−0.40 km)
- Fitted kilonova and GRB afterglow light-curve models

## How to apply

Build the GW likelihood products with nmma-generation, run kilonova light-curve inference on AT2017gfo photometry with lightcurve-analysis (bilby backend), and model the GRB170817A afterglow (Troja et al. data set) with a synthetic Gaussian jet-model lightcurve. Then execute the simultaneous joint analysis with nmma-analysis, incorporating chiral-EFT-based EOS prior sets and sampling the joint posterior with the dynesty nested sampler. Extract the R(1.4 M⊙) posterior from the joint samples; the GRB inclination constraint propagates to the EOS via kilonova ejecta-mass measurements. Judge success by whether the recovered 90% credible interval reproduces R = 11.98 (+0.35/−0.40) km and whether both the AT2017gfo and GRB afterglow light curves are well described by the fitted model.

## Related tools

- **NMMA** (Framework providing the nmma-generation, nmma-analysis, and lightcurve-analysis entry points for the simultaneous multi-messenger joint analysis) — github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend used by NMMA for Bayesian inference of the kilonova light curves)
- **dynesty** (Nested sampler used to sample the joint posterior) — https://zenodo.org/records/17268284

## Examples

```
nmma-analysis --scheduler cores --detector H1 L1 V1 --prior-settings EOS --alignment ZeroSpin --eos-set ces --injection-detection-file GW170817.dat --samples joint_posterior.h5
```

## Evaluation signals

- Recovered R(1.4 M⊙) must reproduce the reported 90% credible interval 11.98 +0.35/−0.40 km
- The kilonova AT2017gfo light curve must be well described by the fitted model
- The GRB170817A afterglow light curve must be well described by the fitted Gaussian jet model
- Adding GRB170817A to the GW+kilonova analysis should yield a minimal yet noticeable improvement in parameter uncertainties
- Sampler diagnostics: posterior sampled with dynesty via the bilby backend, matching the article's analysis configuration

## Limitations

- Additional uncertainties come from limited physics input in the kilonova and semi-analytic GRB models
- No significant EOS constraint is imposed by the jet energy E0 since the ratio ξ between E0 and disk mass mdisk is a free parameter
- Reliable interpretation of future BNS detections requires further advancement of astrophysical transient models beyond parameter-estimation infrastructure
- Kilonova model systematics require separate treatment (data-driven systematics-error modeling cited in the README)
