---
name: binary-evolution-model-variation
description: Use when a BBH formation pathway claimed under a Fiducial COMPAS model
  needs robustness testing against an alternative prescription (e.g. supernova engine,
  CE treatment, alpha_lambda, f_LBV mass loss), with outputs compared pathway-by-pathway.
when_to_use_negative:
- The goal is full exploration of the multi-dimensional model parameter space (e.g.
  varying alpha, f_LBV jointly) — the paper explicitly defers that to future studies
  and only varies parameters independently
- The question requires an absolute merger-rate prediction convolved with cosmic star
  formation history rather than a relative fiducial-vs-varied pathway comparison
- The evolution of systems depends on physics not present in this Fiducial model —
  stellar rotation and tides, wind accretion by the companion, and the SSE single-star
  grid above 50 Msun (which is extrapolated) are not handled
tools:
- name: COMPAS
  role: Rapid binary population synthesis code whose adjustable evolution prescriptions
    are varied one at a time relative to the Fiducial model and rerun at 10%/5% solar
    metallicity
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS importance-sampling algorithm used to efficiently sample the merging-BBH
    channel when rerunning the varied model
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:5c2451e55a8a865421f17a00393142f6e7837a3abad589cd815fdcac51142fb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# binary-evolution-model-variation

## Summary

A one-parameter-at-a-time sensitivity analysis for binary population synthesis: rerun a COMPAS population with a single physics prescription (e.g. the 'rapid' vs 'delayed' Fryer supernova engine) changed relative to the Fiducial model, then compare formation pathways and merger counts to judge robustness of the channel.

## When to use

Use when a BBH formation pathway claimed under a Fiducial COMPAS model needs robustness testing against an alternative prescription (e.g. supernova engine, CE treatment, alpha_lambda, f_LBV mass loss), with outputs compared pathway-by-pathway.

## When NOT to use

- The goal is full exploration of the multi-dimensional model parameter space (e.g. varying alpha, f_LBV jointly) — the paper explicitly defers that to future studies and only varies parameters independently
- The question requires an absolute merger-rate prediction convolved with cosmic star formation history rather than a relative fiducial-vs-varied pathway comparison
- The evolution of systems depends on physics not present in this Fiducial model — stellar rotation and tides, wind accretion by the companion, and the SSE single-star grid above 50 Msun (which is extrapolated) are not handled

## Inputs

- Fiducial COMPAS population-synthesis configuration (Kroupa IMF, m1 ≤ 100 Msun, uniform mass ratio, flat-in-log a, e = 0, all stars in binaries)
- Alternative physics prescription(s) to swap in, e.g. 'rapid' vs 'delayed' Fryer supernova engine
- Target metallicities (e.g. 10% and 5% solar), sampled with STROOPWAFEL for the merging-BBH channel
- COMPAS output files with double compact objects, including component masses and fallback fractions

## Outputs

- Merging-BBH sample (mergers within a Hubble time) under the varied model, with component BH masses and fallback fractions
- Side-by-side comparison of formation-pathway distributions (mass transfer, common envelope, remnant masses) between fiducial and varied models
- Qualitative verdict on whether the changed prescription significantly alters typical evolutionary pathways and total merger counts (e.g. counts of ∼2×10^4 merging BBHs per 10^9 Msun of star formation at 10% solar metallicity as the fiducial reference point)

## How to apply

Start from the Fiducial COMPAS setup — Kroupa IMF with primary masses up to 100 Msun, uniform mass ratio, flat-in-log semi-major axis, zero eccentricity, all stars in binaries — and change exactly one prescription at a time (e.g. switch the supernova engine from 'delayed' to the 'rapid' model of Fryer et al.). Rerun the population synthesis at the relevant metallicities (10% and 5% solar metallicity) using the STROOPWAFEL importance-sampling algorithm restricted to the merging-BBH channel, then extract all double compact objects merging within a Hubble time, recording component black hole masses and fallback fractions. Compare the distribution of formation pathways (mass transfer, common envelope, remnant masses, fallback) and total merger counts against the fiducial output; the change is judged insignificant if the same pathway dominates (e.g. heavy-BH formation through almost complete fallback) and merger counts stay comparable. The rationale, per the paper, is the common approach of varying individual parameters independently and assessing their impact relative to the Fiducial model, reserving full multi-dimensional parameter exploration for later studies.

## Related tools

- **COMPAS** (Rapid binary population synthesis code whose adjustable evolution prescriptions are varied one at a time relative to the Fiducial model and rerun at 10%/5% solar metallicity) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS importance-sampling algorithm used to efficiently sample the merging-BBH channel when rerunning the varied model)

## Evaluation signals

- Exactly one prescription/parameter differs from the Fiducial run per comparison (one-at-a-time variation, as in the paper's approach)
- Both 'delayed' and 'rapid' runs predict high-mass BH formation through almost complete fallback, with comparable BBH merger counts — this confirms pathway robustness
- Only double compact objects merging within a Hubble time are retained from the COMPAS output
- STROOPWAFEL importance sampling was applied and restricted to the merging-BBH channel at both 10% and 5% solar metallicity
- Merger-count ratios against the Fiducial model are reported (cf. paper's benchmarks: optimistic CE ∼3x higher at 10% Zsun; alpha_lambda 0.1→0.01 drops the rate by ∼2x)

## Limitations

- Only individual parameters are varied independently; full exploration of the parameter space (e.g. alpha, f_LBV jointly) is deferred to future studies with COMPAS
- Merger-rate comparisons are approximate: an accurate rate requires convolution of metallicity-specific redshift-dependent star formation rate with the time delay distribution, which was not done
- The Fiducial model omits stellar rotation and tides, wind accretion by the companion, and extrapolates the SSE single-star grid above 50 solar masses
- Ability of a single formation channel to explain all observed events remains untested pending future GW observations

## Evidence

- [discussion] varying individual parameters independently and assessing their impact relative to the Fiducial model: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model."
- [discussion] rapid Fryer supernova model does not alter heavy-BBH pathways: [evidence span withheld — anchor status: not_found]
- [readme] COMPAS designed with easily adjustable prescriptions: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [discussion] full parameter-space exploration deferred to future studies: "We leave a full exploration of this parameter space for future studies with COMPAS"
