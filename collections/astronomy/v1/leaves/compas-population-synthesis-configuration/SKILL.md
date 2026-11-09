---
name: compas-population-synthesis-configuration
description: Use when running population synthesis with COMPAS to evolve massive stellar
  binaries into double compact objects, or when testing whether an alternative physics
  prescription (e.g. supernova engine) changes evolutionary pathways for heavy BBH
  mergers.
when_to_use_negative:
- When the science goal requires a full multidimensional exploration of the model
  parameter space (e.g. alpha, fLBV jointly) rather than independent one-at-a-time
  variations — the article explicitly defers that to future studies.
- 'When accurate merger-rate prediction over cosmic time is required: this skill yields
  metallicity-specific yields, but an accurate rate needs convolution of the metallicity-specific
  redshift-dependent star formation rate with the delay-time distribution, which is
  not part of this configuration.'
- When stellar rotation, tides, or wind accretion onto the companion must be modeled
  — these are not included in the Fiducial model.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to evolve binaries from zero-age
    main sequence to two compact remnants, with easily adjustable evolution prescriptions
    and model parameters such as the supernova engine
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: STROOPWAFEL
  role: COMPAS's importance sampling algorithm used to efficiently sample the merging-BBH
    channel at low metallicities
- name: SSE
  role: Analytical single-star evolution fits (Hurley et al.) used by COMPAS for single
    stellar evolution, extrapolated above the original 50 Msun grid limit
- name: BSE
  role: Reference binary population synthesis approach on which COMPAS's methodology
    is broadly based
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:ba1d10a5015535140ad4498210e4b47f9b5692ee2d2817c122f77ab97196d843
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSE
  - COMPAS
  - SSE
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# compas-population-synthesis-configuration

## Summary

Configure and run COMPAS rapid binary population synthesis with a specified fiducial setup (Kroupa IMF up to 100 Msun, uniform mass ratio, flat-in-log semi-major axis, zero eccentricity, all stars in binaries) and controlled single-parameter variations (e.g. swapping the Fryer 'delayed' supernova engine for the 'rapid' model) to test robustness of binary black hole formation pathways.

## When to use

Use when running population synthesis with COMPAS to evolve massive stellar binaries into double compact objects, or when testing whether an alternative physics prescription (e.g. supernova engine) changes evolutionary pathways for heavy BBH mergers.

## When NOT to use

- When the science goal requires a full multidimensional exploration of the model parameter space (e.g. alpha, fLBV jointly) rather than independent one-at-a-time variations — the article explicitly defers that to future studies.
- When accurate merger-rate prediction over cosmic time is required: this skill yields metallicity-specific yields, but an accurate rate needs convolution of the metallicity-specific redshift-dependent star formation rate with the delay-time distribution, which is not part of this configuration.
- When stellar rotation, tides, or wind accretion onto the companion must be modeled — these are not included in the Fiducial model.

## Inputs

- COMPAS input configuration with initial distributions (Kroupa IMF m1 <= 100 Msun, uniform mass ratio, flat-in-log semi-major axis, zero eccentricity, binary fraction 1)
- Metallicity grid values (e.g. 10% and 5% solar metallicity)
- Chosen supernova prescription ('delayed' fiducial vs 'rapid' Fryer et al. model)
- STROOPWAFEL importance-sampling configuration for the merging-BBH channel

## Outputs

- COMPAS population output of evolved binaries with two compact remnants
- Set of double compact objects merging within a Hubble time, with component black hole masses and fallback fractions
- Distribution of BBH formation pathways (mass transfer, common envelope, remnant masses) per supernova prescription
- Merger yield, e.g. BBHs merging per 1e9 solar masses of star formation at each metallicity

## How to apply

Set up COMPAS with the Fiducial-model initial distributions: primary masses drawn from a Kroupa IMF up to 100 Msun, a uniform (flat) mass-ratio distribution, a flat-in-the-log semi-major-axis distribution, initial eccentricity of zero, and the assumption that all stars are in binaries. Evolve systems from zero-age main sequence to two compact remnants, using the STROOPWAFEL importance sampling algorithm when targeting the merging-BBH channel. To test prescription robustness (as in task_008), switch only one parameter at a time relative to the Fiducial model — e.g. change the supernova engine from the 'delayed' model to the 'rapid' model of Fryer et al. — and rerun at low metallicity (10% and 5% solar). Extract all double compact objects that merge within a Hubble time from the COMPAS output, recording component black hole masses and fallback fractions, then compare pathway distributions (mass transfer, common envelope, remnant masses) against the fiducial run. The rationale is that varying individual parameters independently isolates the effect of each uncertain physical process.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to evolve binaries from zero-age main sequence to two compact remnants, with easily adjustable evolution prescriptions and model parameters such as the supernova engine) — http://github.com/TeamCOMPAS/COMPAS_
- **STROOPWAFEL** (COMPAS's importance sampling algorithm used to efficiently sample the merging-BBH channel at low metallicities)
- **SSE** (Analytical single-star evolution fits (Hurley et al.) used by COMPAS for single stellar evolution, extrapolated above the original 50 Msun grid limit)
- **BSE** (Reference binary population synthesis approach on which COMPAS's methodology is broadly based)

## Examples

```
./COMPAS --initial-mass-max 100 --metallicity 0.001 --SN-engine rapid --importance-sampling STROOPWAFEL
```

## Evaluation signals

- Robustness check: both the 'rapid' and 'delayed' Fryer prescriptions predict high-mass BH formation through almost complete fallback, with comparable BBH merger counts — a significant change in pathway distributions would signal a misconfiguration.
- Yield sanity check: Fiducial model forms ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar metallicity, rising to ~3e4 at 5% solar metallicity; results far outside this range warrant investigation.
- Single-variable discipline: exactly one physics parameter changed per run relative to Fiducial; if multiple parameters are changed simultaneously, pathway attribution is invalid.
- Extraction completeness: all extracted double compact objects merge within a Hubble time, and each carries component BH masses and fallback fractions.
- Sampling correctness: STROOPWAFEL importance sampling is active for the merging-BBH channel at 10% and 5% solar metallicity runs.

## Limitations

- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit.
- Stellar rotation and tides are not included in the Fiducial model; wind accretion by a companion is not accounted for.
- Only individual parameters were varied; full exploration of the multidimensional model parameter space (e.g. alpha, fLBV) is deferred to future studies.
- The merger-rate comparison to LIGO is approximate: an accurate calculation requires convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution, which was not done.

## Evidence

- [discussion] Kroupa IMF up to 100 Msun with all stars in binaries: [evidence span withheld — anchor status: not_found]
- [discussion] All stars in binaries assumption: "using the Kroupa31 initial mass function (IMF), a uniform mass ratio distribution and assuming that all stars are in binaries"
- [intro] COMPAS is designed for easily adjustable prescriptions: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [discussion] One-at-a-time parameter variation approach: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model."
- [discussion] Rapid vs delayed supernova robustness finding: "using the ‘rapid' model of Fryer et al.38 does not significantly alter the typical evolutionary pathways for forming heavy BBHs"
- [readme] STROOPWAFEL citation trigger: "If you use COMPAS's importance sampling algorithm STROOPWAFEL, please cite"
