---
name: binary-black-hole-merger-rate-normalization
description: Use when comparing how COMPAS model assumptions (e.g. common-envelope
  survival of Hertzsprung-gap donors) change the number of merging BBHs at a fixed
  metallicity such as 10% or 5% solar, normalized per 1e9 solar masses of star formation.
when_to_use_negative:
- When an accurate cosmic (Gpc−3 per year) merger rate is needed — that requires convolving
  metallicity-specific redshift-dependent star formation rate with the time-delay
  distribution, which this normalized-count comparison does not do
- When analyzing individual observed gravitational-wave events (e.g. GW150914 progenitor
  masses) rather than population-level merger counts across models
- When varying multiple population-synthesis parameters jointly — the article's approach
  varies individual parameters independently relative to the Fiducial model
edam_operation: http://edamontology.org/operation_0306
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate the Fiducial and optimistic-CE
    BBH populations and their merger-time outputs
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: SSE
  role: Hurley et al. analytical single-star evolution fits underlying the stellar
    evolution prescriptions in COMPAS (grid extends to 50 Msun and is extrapolated
    above)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:88ba2bcc0753950133430c6e97e087805873e5d08672ef9b605aba0234aae936
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - SSE
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# binary-black-hole-merger-rate-normalization

## Summary

A method for quantifying binary black hole (BBH) merger counts from COMPAS binary population synthesis by filtering systems that merge within a Hubble time at fixed metallicity, normalizing per 1e9 solar masses of star formation, and comparing model variants (e.g. Fiducial vs optimistic common-envelope).

## When to use

Use when comparing how COMPAS model assumptions (e.g. common-envelope survival of Hertzsprung-gap donors) change the number of merging BBHs at a fixed metallicity such as 10% or 5% solar, normalized per 1e9 solar masses of star formation.

## When NOT to use

- When an accurate cosmic (Gpc−3 per year) merger rate is needed — that requires convolving metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which this normalized-count comparison does not do
- When analyzing individual observed gravitational-wave events (e.g. GW150914 progenitor masses) rather than population-level merger counts across models
- When varying multiple population-synthesis parameters jointly — the article's approach varies individual parameters independently relative to the Fiducial model

## Inputs

- COMPAS simulation output (Fiducial run) for BBH progenitors at 10% and 5% solar metallicity
- COMPAS simulation output (optimistic-CE run, HG donors survive common envelope) at the same metallicities
- Model assumptions: Kroupa IMF up to 100 Msun, uniform mass ratio, flat-in-log semimajor axis, circular (zero-eccentricity) binaries

## Outputs

- Normalized counts of merging BBHs per 1e9 solar masses of star formation at each metallicity and model (Fiducial: ~2e4 at 10% solar, ~3e4 at 5% solar)
- Optimistic/Fiducial total BBH merger-rate ratios (~3 at 10% Zsun, ~2 at 5% Zsun)
- Maximum total BBH mass per model at Z=0.002 (~50 Msun Fiducial vs ~60 Msun optimistic CE)
- ce_model_comparison.csv with model, metallicity, and value columns

## How to apply

Run COMPAS binary population synthesis under the Fiducial model and the model variant of interest (e.g. optimistic CE, in which Hertzsprung-gap donors survive common envelope), drawing primary masses from a Kroupa IMF up to 100 Msun, flat mass ratios, flat-in-log semimajor axes, and circular binaries. Filter each run for BBHs that merge within a Hubble time at the target metallicity (10% and 5% solar) and normalize counts per 1e9 solar masses of star formation; the Fiducial references are ~2e4 at 10% solar and ~3e4 at 5% solar. Compute the variant/Fiducial total BBH merger-rate ratio at each metallicity — the optimistic CE model gives ~3x at 10% Zsun and ~2x at 5% Zsun — and, at Z=0.002, the maximum total BBH mass in each model (Fiducial ~50 Msun rising to ~60 Msun under optimistic CE). Write the tabulated ratios and maximum masses (with model, metallicity, and value columns) to a CSV with pandas. Rationale: normalizing per stellar mass formed makes the metallicity-specific yields directly comparable across model assumptions; note that converting to a cosmic rate requires convolution with the metallicity-specific redshift-dependent star formation rate and the time-delay distribution, which this skill does not perform.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate the Fiducial and optimistic-CE BBH populations and their merger-time outputs) — http://github.com/TeamCOMPAS/COMPAS
- **SSE** (Hurley et al. analytical single-star evolution fits underlying the stellar evolution prescriptions in COMPAS (grid extends to 50 Msun and is extrapolated above))

## Examples

```
import pandas as pd; ce_model_comparison = pd.DataFrame({'model': [...], 'metallicity': [...], 'value': [...]}); ce_model_comparison.to_csv('ce_model_comparison.csv')
```

## Evaluation signals

- Optimistic/Fiducial total BBH merger-rate ratio is ~3 at Z=10%Zsun and ~2 at Z=5%Zsun, matching the published values
- Fiducial counts reproduce ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar and ~3e4 at 5% solar
- Maximum total BBH mass at Z=0.002 rises from ~50 (Fiducial) to ~60 (optimistic) solar masses
- ce_model_comparison.csv has model, metallicity, and value columns and matches the tabulated ratios and masses
- If rescaled, the formation rate lands near ~300 Gpc−3 per year at z=0, bracketed by the LIGO empirical estimate of 9–240 Gpc−3 per year

## Limitations

- Only individual parameters are varied independently relative to the Fiducial model; full exploration of the multidimensional parameter space (e.g. alpha, fLBV) is deferred
- An accurate merger-rate calculation requires convolution of the metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which was not done; the comparison to LIGO's 9–240 Gpc−3 per year is approximate
- The single-star (SSE) model grid extends only to 50 solar masses and is extrapolated above that limit
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model

## Evidence

- [discussion] Fiducial normalization at 10% solar metallicity: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] Optimistic CE vs Fiducial rate ratio: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙ and ∼2 times higher at Z=5%Z⊙"
- [discussion] Why a cosmic rate conversion is out of scope: "An accurate calculation of the merger rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution"
- [intro] Tool provenance for the populations being compared: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [readme] Where simulation results are published for retrieval: "We make the results of our simulations available at http://www.sr.bham.ac.uk/compas/."
