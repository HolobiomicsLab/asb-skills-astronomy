---
name: eos-posterior-comparison-across-analyses
description: Use when you have NMMA posterior samples from a multi-messenger fit (e.g.,
  GW170817 strain + AT2017gfo photometry) and want to test whether adding another
  messenger (GRB170817A) shifts or tightens posteriors on R_1.4 and ejecta parameters.
when_to_use_negative:
- You only have gravitational-wave data with no kilonova photometry — the comparison
  hinges on the GW+KN baseline that couples inclination to ejecta masses.
- The two posterior sets come from analyses with different priors, likelihoods, or
  samplers, making a credible-interval comparison invalid.
- You only have published point estimates (e.g., a single R_1.4 value) rather than
  posterior sample files in JSON/hdf5 format.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: NMMA
  role: Framework providing the GW+KN-only likelihood and the joint GW+KN+GRB posterior
    samples being compared
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: nmma-analysis
  role: NMMA entry point used to run the GW component of the restricted GW+KN-only
    analysis
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: lightcurve-analysis
  role: NMMA entry point used to run the kilonova light-curve component of the GW+KN-only
    analysis
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Inference backend underlying the NMMA likelihood and posterior-sampling setup
- name: dynesty
  role: Nested sampler used with bilby to draw the GW+KN-only posterior samples
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:25ad3adf40ae962f02700a42b008c5c91a012f9b8e4524af540bf71499ca5fc3
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# eos-posterior-comparison-across-analyses

## Summary

Compares posterior samples of equation-of-state-relevant parameters (R_1.4, dynamical and wind ejecta masses, jet energy, inclination) between a GW+KN-only NMMA analysis and a joint GW+KN+GRB analysis, quantifying how adding GRB170817A information changes credible-interval widths.

## When to use

Use when you have NMMA posterior samples from a multi-messenger fit (e.g., GW170817 strain + AT2017gfo photometry) and want to test whether adding another messenger (GRB170817A) shifts or tightens posteriors on R_1.4 and ejecta parameters.

## When NOT to use

- You only have gravitational-wave data with no kilonova photometry — the comparison hinges on the GW+KN baseline that couples inclination to ejecta masses.
- The two posterior sets come from analyses with different priors, likelihoods, or samplers, making a credible-interval comparison invalid.
- You only have published point estimates (e.g., a single R_1.4 value) rather than posterior sample files in JSON/hdf5 format.

## Inputs

- GW170817 gravitational-wave strain data
- AT2017gfo kilonova multi-band light-curve photometry
- joint GW+KN+GRB posterior samples (JSON result files from a prior joint NMMA run)
- NMMA analysis configuration selecting a GW+KN-only likelihood (no GRB data)

## Outputs

- gw_kn_only_posterior_samples.json (dynesty posterior samples for R_1.4, ejecta masses, inclination)
- posterior_comparison_gw_kn_vs_joint.png (corner-style overlay of marginal posteriors)
- quantified credible-interval width changes per parameter

## How to apply

First run the restricted GW+kilonova analysis by invoking the nmma-analysis and lightcurve-analysis entry points with bilby as the inference backend and the dynesty nested sampler on the GW170817 strain data and AT2017gfo multi-band light-curve photometry, sampling the joint likelihood for R_1.4, dynamical/wind ejecta masses, and binary inclination; save the samples to gw_kn_only_posterior_samples.json. Then load the joint GW+KN+GRB posterior samples and overlay the R_1.4 and ejecta-parameter marginals in a corner-style comparison figure, quantifying the change in credible-interval widths. The rationale is that the GRB constrains the EOS indirectly through its inclination measurement, so the GW+KN-only result is the correct baseline for judging the GRB's marginal value. Validate that the two posterior sets are statistically consistent and that the GW+KN-only R_1.4 credible interval overlaps the joint estimate R_1.4 = 11.98 (+0.35/-0.40) km.

## Related tools

- **NMMA** (Framework providing the GW+KN-only likelihood and the joint GW+KN+GRB posterior samples being compared) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **nmma-analysis** (NMMA entry point used to run the GW component of the restricted GW+KN-only analysis) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **lightcurve-analysis** (NMMA entry point used to run the kilonova light-curve component of the GW+KN-only analysis) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Inference backend underlying the NMMA likelihood and posterior-sampling setup)
- **dynesty** (Nested sampler used with bilby to draw the GW+KN-only posterior samples) — https://zenodo.org/records/17268284

## Evaluation signals

- The GW+KN-only R_1.4 credible interval overlaps the joint-analysis estimate of 11.98 (+0.35/-0.40) km.
- The GW+KN-only posteriors are statistically consistent with the joint GW+KN+GRB posteriors across R_1.4, ejecta masses, and inclination.
- The reduction in credible-interval width from adding GRB170817A is small but nonzero, matching the 'minimal, yet, noticeable' expectation.
- The corner-style figure successfully overlays both posterior sets' R_1.4 and ejecta-parameter marginals without sampling artifacts.
- The dynesty nested-sampling run under bilby converged (no evidence-ratio or sampling-failure warnings in the result files).

## Limitations

- No significant constraint on the EOS is imposed by the jet energy E0 because the ratio ξ between E0 and the disk mass mdisk is treated as a free parameter.
- Additional uncertainties arise from limited physics input in the kilonova and semi-analytic GRB models used in the likelihoods.
- The improvement from adding GRB170817A information is by construction small; this skill quantifies marginal gains, not large shifts in constraints.
- The discussion text provides no version number or DOI for the NMMA release used, so reproducibility of the exact posteriors requires care.

## Evidence

- [results] the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added.: "the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added."
- [abstract] R_1.4 joint estimate: [evidence span withheld — anchor status: not_found]
- [readme] bilby backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [results] GRB inclination constrains EOS: "the inclination constraint from the GRB plays a role in the constraint on EOS"
- [methods] dynesty sampler citation: [evidence span withheld — anchor status: not_found]
