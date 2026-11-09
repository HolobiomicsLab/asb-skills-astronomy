---
name: gravitational-wave-bayesian-parameter-estimation
description: Use when you have gravitational-wave strain data plus electromagnetic
  counterpart observations (kilonova photometry, GRB afterglow light curves) and need
  joint posterior samples over EOS/ejecta parameters, or when comparing posteriors
  from nested analyses (e.g. GW+KN-only vs GW+KN+GRB).
when_to_use_negative:
- The analysis goal is a single-messenger analysis only (e.g. kilonova light-curve
  fitting without GW data) — then lightcurve-analysis alone suffices and the GW likelihood
  machinery is unnecessary.
- 'You need tight EOS constraints driven by the jet energy: the paper finds ''no significant
  constraint on the EOS is imposed by the jet energy E0'' because the ratio ξ between
  E0 and disk mass is a free parameter.'
- Kilonova model physics fidelity (not parameter-estimation infrastructure) is the
  limiting factor — the paper flags limited physics input in kilonova and semi-analytic
  GRB models as a dominant uncertainty.
edam_operation: http://edamontology.org/operation_3659
tools:
- name: NMMA
  role: Framework performing the simultaneous GW+KN+GRB Bayesian analysis and EOS
    constraints
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Inference backend used by NMMA for likelihood evaluation and parameter estimation
- name: dynesty
  role: Nested sampler used to draw posterior samples from the joint likelihood
  repo: https://zenodo.org/records/17268284
- name: nmma-analysis
  role: NMMA CLI entry point configuring and running the GW (and joint) analysis
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: lightcurve-analysis
  role: NMMA CLI entry point running the kilonova light-curve inference component
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:fe6c8836d685aabf1b6790635374911c5220430892d020d5436985e81c1ad340
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  - lightcurve-analysis
  - nmma-analysis
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gravitational-wave-bayesian-parameter-estimation

## Summary

Joint Bayesian parameter estimation of binary-neutron-merger observables (GW strain, kilonova light curves, GRB afterglow) using NMMA's nmma-analysis/lightcurve-analysis entry points with bilby as backend and dynesty nested sampling, yielding posteriors for ejecta masses, jet energy, inclination, and the neutron-star radius R_1.4.

## When to use

Use when you have gravitational-wave strain data plus electromagnetic counterpart observations (kilonova photometry, GRB afterglow light curves) and need joint posterior samples over EOS/ejecta parameters, or when comparing posteriors from nested analyses (e.g. GW+KN-only vs GW+KN+GRB).

## When NOT to use

- The analysis goal is a single-messenger analysis only (e.g. kilonova light-curve fitting without GW data) — then lightcurve-analysis alone suffices and the GW likelihood machinery is unnecessary.
- You need tight EOS constraints driven by the jet energy: the paper finds 'no significant constraint on the EOS is imposed by the jet energy E0' because the ratio ξ between E0 and disk mass is a free parameter.
- Kilonova model physics fidelity (not parameter-estimation infrastructure) is the limiting factor — the paper flags limited physics input in kilonova and semi-analytic GRB models as a dominant uncertainty.

## Inputs

- GW170817 gravitational-wave strain data
- AT2017gfo kilonova multi-band light-curve photometry
- Troja et al. GRB afterglow light-curve data set (for the joint GW+KN+GRB analysis)
- EOS sets from chiral EFT and radio/X-ray measurements of isolated NSs (optional prior constraints)

## Outputs

- Posterior samples for R_1.4, dynamical and wind ejecta masses, jet energy E0, and binary inclination (e.g. gw_kn_only_posterior_samples.json)
- Corner-style posterior comparison figure (e.g. posterior_comparison_gw_kn_vs_joint.png) with quantified credible-interval width changes

## How to apply

Load the GW170817 strain data and AT2017gfo kilonova multi-band photometry (and, for the joint case, the Troja et al. GRB afterglow data set analyzed with a synthetic Gaussian jet-model lightcurve). Configure the NMMA joint analysis by invoking the nmma-analysis entry point for the GW likelihood and lightcurve-analysis for the kilonova, with bilby as the inference backend. Run nested sampling of the likelihood with the dynesty sampler to obtain posterior samples for R_1.4, dynamical and wind ejecta masses, jet energy, and binary inclination. To compare analyses (e.g. GW+KN-only vs GW+KN+GRB), overlay R_1.4 and ejecta-parameter marginals in a corner-style figure and quantify credible-interval width changes; note that the GRB contributes mainly through its inclination constraint, which propagates to the EOS via kilonova ejecta mass measurements, so the improvement is expected to be minimal yet noticeable.

## Related tools

- **NMMA** (Framework performing the simultaneous GW+KN+GRB Bayesian analysis and EOS constraints) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Inference backend used by NMMA for likelihood evaluation and parameter estimation)
- **dynesty** (Nested sampler used to draw posterior samples from the joint likelihood) — https://zenodo.org/records/17268284
- **nmma-analysis** (NMMA CLI entry point configuring and running the GW (and joint) analysis) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **lightcurve-analysis** (NMMA CLI entry point running the kilonova light-curve inference component) — https://github.com/nuclear-multimessenger-astronomy/nmma

## Examples

```
nmma-analysis --sampler dynesty --outdir gw_kn_only --label gw_kn_only
```

## Evaluation signals

- GW+KN-only R_1.4 credible interval must overlap the joint-analysis estimate R_1.4 = 11.98 (+0.35/-0.40) km
- GW+KN-only posteriors should be statistically consistent with the joint GW+KN+GRB posteriors (adding GRB170817A yields only a minimal, yet noticeable, uncertainty reduction)
- Both the kilonova AT2017gfo and the GRB afterglow GRB170817A light curves should be well described by the fitted models
- Posterior sample file (e.g. gw_kn_only_posterior_samples.json) must contain samples for all target parameters including R_1.4, ejecta masses, and inclination
- Credible-interval widths from the joint analysis should be equal to or narrower than the GW+KN-only widths, with quantified reduction reported

## Limitations

- Adding GRB170817A information yields only a minimal, yet noticeable, improvement in parameter uncertainties — do not expect large posterior sharpening from the GRB alone.
- No significant constraint on the EOS is imposed by the jet energy E0 because the ratio ξ between E0 and the disk mass mdisk is treated as a free parameter.
- Additional uncertainties arise from limited physics input in the kilonova models and semi-analytic GRB models.
- No version number, repository URL, or DOI for the publicly available NMMA framework is provided in the discussion text.

## Evidence

- [abstract] R_1.4 joint posterior estimate: [evidence span withheld — anchor status: not_found]
- [results] GRB adds only modest improvement: "the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added."
- [results] GRB inclination drives EOS constraint: "the GRB’s inclination measurement imposes a constraint on the EOS via the kilonova eject masses measurements."
- [readme] bilby backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [readme] sampler citation guidance: "Based on the sampler used for the analysis, please also cite the associated papers:"
