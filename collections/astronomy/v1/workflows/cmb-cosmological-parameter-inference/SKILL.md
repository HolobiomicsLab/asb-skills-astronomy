---
name: cmb-cosmological-parameter-inference-workflow
description: 'Use when observed CMB likelihood data and Lambda-CDM priors must be
  assembled into a callable CAMB likelihood, sampled for cosmological parameters,
  cleared of burn-in, diagnosed for autocorrelation, and summarized as physical constraints.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - wmap-likelihood-chain-configuration
  - cmb-likelihood-module-integration
  - likelihood-chain-architecture-implementation
  - plugin-module-registration-pattern
  - cosmological-parameter-mcmc-sampling
  - cosmological-parameter-sampler-setup
  - burn-in-determination-via-standard-error-comparison
  - monte-carlo-standard-error-analysis
  - cross-sampler-convergence-comparison
  - mcmc-autocorrelation-time-analysis
  - mcmc-convergence-diagnostic-tabulation
  - sampler-stopping-criterion-computation
  - lambda-cdm-parameter-inference
  - mcmc-ensemble-sampling-cosmological-parameters
  - posterior-summary-statistics-computation
  member_tools:
  - CAMB
  - CosmoHammer
  - CosmoHammerPlugins
  - Python
  - WMAP
  - acor
  - emcee
  - mpi4py
  - CosmoMC
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/astronomy/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# CMB Lambda-CDM Parameter Inference

## Summary

Configure a callable CMB likelihood, sample it while recomputing theory predictions, determine the burn-in, diagnose the autocorrelation of the chains, and report posterior constraints.


## When to use

Use when observed CMB likelihood data and Lambda-CDM priors must be assembled into a callable CAMB likelihood, sampled for cosmological parameters, cleared of burn-in, diagnosed for autocorrelation, and summarized as physical constraints.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — configure_cmb_likelihood

**Goal:** assemble a callable CMB theory-and-likelihood chain

**EDAM operation:** http://edamontology.org/operation_2426

**Inputs:** likelihood-data/observed-cmb, parameter-table/cosmology-priors, configuration/cmb-chain · **Outputs:** likelihood-model/callable-cmb

**Candidate leaf skills:** `wmap-likelihood-chain-configuration` (primary), `cmb-likelihood-module-integration`, `likelihood-chain-architecture-implementation`, `plugin-module-registration-pattern`

**Tools (primary):** CAMB, CosmoHammer, CosmoHammerPlugins, Python, WMAP, acor, emcee, mpi4py


**Grounding:** 1 KB(s); DOIs: 10.1016/j.ascom.2013.06.003

### Stage 2 — sample_parameter_posterior

**Goal:** sample the CMB likelihood over the bounded cosmological parameter space

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** likelihood-model/callable-cmb, parameter-table/cosmology-priors · **Outputs:** posterior-samples/cmb-cosmology

**Candidate leaf skills:** `cosmological-parameter-mcmc-sampling` (primary), `cosmological-parameter-sampler-setup`

**Tools (primary):** CAMB, CosmoHammer, CosmoMC, Python, acor, emcee

**Other candidate tools:** CosmoHammerPlugins, mpi4py

**Grounding:** 1 KB(s); DOIs: 10.1016/j.ascom.2013.06.003

### Stage 3 — determine_burn_in

**Goal:** derive a defensible burn-in length from the initialisation bias of the chains

**EDAM operation:** http://edamontology.org/operation_2238

**Inputs:** posterior-samples/cmb-cosmology · **Outputs:** burn-in-table/cmb-mcmc

**Candidate leaf skills:** `burn-in-determination-via-standard-error-comparison` (primary), `monte-carlo-standard-error-analysis`, `cross-sampler-convergence-comparison`

**Tools (primary):** CosmoHammer, CosmoMC, Python, acor, emcee


**Grounding:** 1 KB(s); DOIs: 10.1016/j.ascom.2013.06.003

### Stage 4 — diagnose_autocorrelation

**Goal:** quantify autocorrelation and effective sampling after the burn-in

**EDAM operation:** http://edamontology.org/operation_2426

**Inputs:** posterior-samples/cmb-cosmology, burn-in-table/cmb-mcmc · **Outputs:** diagnostic-report/cmb-mcmc

**Candidate leaf skills:** `mcmc-autocorrelation-time-analysis` (primary), `mcmc-convergence-diagnostic-tabulation`, `sampler-stopping-criterion-computation`

**Tools (primary):** CAMB, CosmoHammer, CosmoMC, Python, acor, emcee

**Other candidate tools:** CosmoHammerPlugins

**Grounding:** 1 KB(s); DOIs: 10.1016/j.ascom.2013.06.003

### Stage 5 — summarize_constraints

**Goal:** convert converged CMB posterior samples into physical constraints

**EDAM operation:** http://edamontology.org/operation_3658

**Inputs:** posterior-samples/cmb-cosmology, diagnostic-report/cmb-mcmc · **Outputs:** parameter-table/cmb-cosmology-posterior

**Candidate leaf skills:** `lambda-cdm-parameter-inference` (primary), `mcmc-ensemble-sampling-cosmological-parameters`, `posterior-summary-statistics-computation`

**Tools (primary):** CAMB, CosmoHammer, CosmoHammerPlugins, CosmoMC, Python, acor, emcee


**Grounding:** 1 KB(s); DOIs: 10.1016/j.ascom.2013.06.003

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
