---
name: bayesian-nested-sampling-eos-inference
description: Use when you need posterior EOS samples for dense neutron-star matter
  that fuse nuclear-physics EOS bands with model-based high-density extensions (e.g.,
  pQCD constraints at ~40-50 n_sat), and you must sample the EOS parameter space with
  nested sampling.
when_to_use_negative:
- The analysis targets kilonova light-curve or GRB afterglow parameter estimation
  rather than the EOS — NMMA exposes separate entry points (lightcurve-analysis, nmma-analysis)
  for that purpose
- The EOS input is already fully constrained and causal/stable up to the target density,
  so no high-density speed-of-sound extension or pQCD likelihood is needed
- The analysis is a parameter-estimation run on GW strain or EM light curves (e.g.,
  simulating light curves with nmma-generation) rather than EOS inference
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: NMMA
  role: 'Framework hosting the EOS inference pipeline: combine-EOS entry point merges
    per-constraint EOS samples; speed-of-sound extension and pQCD likelihood are embedded
    in its EOS analysis'
  repo: github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend of NMMA that defines prior distributions (including speed-of-sound
    priors) and likelihood terms (including the pQCD constraint) for the Bayesian
    EOS model
  repo: https://bilby-dev.github.io/bilby/citing-bilby.html
- name: dynesty
  role: Nested sampler run via the bilby backend to explore the EOS parameter space
    and estimate posteriors/evidences
  repo: https://zenodo.org/records/17268284
- name: UltraNest
  role: Alternative robust Bayesian inference engine that can be selected as the sampler
    instead of dynesty
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:b2f067543178ce888f444fed5aec892c72b5e8c84143ace41bb48da322f46bee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMMA
  - UltraNest
  - bilby
  - dynesty
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bayesian-nested-sampling-eos-inference

## Summary

Infers neutron-star equation-of-state posteriors by combining chiral-EFT/nuclear-physics constraints (valid up to ~1.5 n_sat), a speed-of-sound parameterization for the supranuclear extension, and optional pQCD likelihood terms, using Bayesian nested sampling with dynesty via the bilby backend inside the NMMA framework.

## When to use

Use when you need posterior EOS samples for dense neutron-star matter that fuse nuclear-physics EOS bands with model-based high-density extensions (e.g., pQCD constraints at ~40-50 n_sat), and you must sample the EOS parameter space with nested sampling.

## When NOT to use

- The analysis targets kilonova light-curve or GRB afterglow parameter estimation rather than the EOS — NMMA exposes separate entry points (lightcurve-analysis, nmma-analysis) for that purpose
- The EOS input is already fully constrained and causal/stable up to the target density, so no high-density speed-of-sound extension or pQCD likelihood is needed
- The analysis is a parameter-estimation run on GW strain or EM light curves (e.g., simulating light curves with nmma-generation) rather than EOS inference

## Inputs

- Per-constraint chiral-EFT/nuclear-physics EOS posterior sample files from the upstream NMMA EOS analysis (combine-EOS entry point input)
- pQCD thermodynamic band (pressure vs. energy density at ~40-50 n_sat) used as a likelihood term
- Prior distributions for speed-of-sound extension parameters, defined in bilby
- Optional multi-messenger data (GW170817, AT2017gfo, GRB170817A) whose constraints feed the EOS posterior

## Outputs

- eos_samples_pqcd_constrained.dat: causal, thermodynamically stable posterior EOS samples
- Derived neutron-star properties per retained EOS (e.g., radius of a 1.4 solar-mass neutron star, R_1.4)
- Posterior distribution and evidence from the nested sampling run

## How to apply

Load the per-constraint EOS posterior sample files from the upstream NMMA EOS analysis and merge them with the combine-EOS entry point of NMMA. Extend each EOS above the nuclear-physics band (which ends at ~1.5 n_sat from Quantum Monte Carlo with chiral EFT interactions) using the speed-of-sound parameterization, drawing speed-of-sound values from prior distributions defined in bilby, and add the pQCD constraint as a likelihood term comparing candidate EOS pressure at ~40-50 n_sat against the pQCD thermodynamic band. Run nested sampling with the dynesty sampler through the bilby backend inside NMMA (alternative samplers PyMultiNest and UltraNest are supported) to obtain the constrained EOS posterior. Screen every posterior EOS for causality (speed of sound not exceeding the speed of light) and thermodynamic stability (monotonically increasing pressure with energy density) up to the pQCD matching density, rejecting violating samples, then save survivors and derived neutron-star properties to eos_samples_pqcd_constrained.dat. Validate against the published NMMA benchmark radius R_1.4 ≈ 11.98 (+0.35/-0.40) km.

## Related tools

- **NMMA** (Framework hosting the EOS inference pipeline: combine-EOS entry point merges per-constraint EOS samples; speed-of-sound extension and pQCD likelihood are embedded in its EOS analysis) — github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend of NMMA that defines prior distributions (including speed-of-sound priors) and likelihood terms (including the pQCD constraint) for the Bayesian EOS model) — https://bilby-dev.github.io/bilby/citing-bilby.html
- **dynesty** (Nested sampler run via the bilby backend to explore the EOS parameter space and estimate posteriors/evidences) — https://zenodo.org/records/17268284
- **UltraNest** (Alternative robust Bayesian inference engine that can be selected as the sampler instead of dynesty)

## Evaluation signals

- Every retained EOS satisfies causality (speed of sound ≤ speed of light) and thermodynamic stability (monotonically increasing pressure with energy density) at all densities up to the pQCD matching density (~40-50 n_sat)
- Resulting radius of a 1.4 solar-mass neutron star remains consistent with the published NMMA estimate R_1.4 = 11.98 (+0.35/-0.40) km
- Output file eos_samples_pqcd_constrained.dat is produced containing only the surviving causal, stable EOS samples and their derived neutron-star properties
- EOS samples below the nuclear-physics band reproduce the upstream chiral-EFT/QMC constraints (valid to ~1.5 n_sat) after combine-EOS merging
- The nested sampling run (dynesty via bilby) completes and yields a posterior distribution and evidence without sampler failures

## Limitations

- The nuclear-physics EOS constraints from Quantum Monte Carlo with chiral EFT interactions are only valid up to 1.5 n_sat; the high-density regime relies on the speed-of-sound model and remains asymptotically unconstrained unless an additional input such as pQCD is supplied
- Additional uncertainties arise from limited physics input in the kilonova and semi-analytic GRB models that feed the multi-messenger constraints
- No version number, repository URL, or DOI for the NMMA framework is given in the article's discussion text, complicating exact-reproducibility checks of the sampling run

## Evidence

- [methods] dynesty as nested sampler: [evidence span withheld — anchor status: not_found]
- [readme] bilby as NMMA backend defining priors and likelihoods: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [abstract] R_1.4 validation benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] NMMA combines multi-messenger signals with nuclear-physics EOS constraints: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows. In addition, our framework allows us to incorporate constraints from nuclear-physics"
