---
name: common-envelope-survival-model-comparison
description: Use when comparing COMPAS population-synthesis runs that differ only
  in CE survival treatment (pessimistic vs optimistic, HG donors surviving) to quantify
  the effect on merging-BBH counts per 1e9 solar masses of star formation at specific
  metallicities.
when_to_use_negative:
- The analysis goal is sensitivity to a different parameter (e.g. αλ from 0.1 to 0.01,
  or LBV wind mass-loss rate 1.5e-4 to 1e-3 Msun/yr) rather than the CE survival prescription
- An absolute cosmological merger rate is needed — this skill reports per-1e9-Msun
  star-formation counts, not the redshift-dependent convolved rate
- The comparison involves binary_c, StarTrack, or BSE codes rather than COMPAS Fiducial
  vs optimistic-CE output
edam_operation: http://edamontology.org/operation_0306
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate the Fiducial and optimistic-CE
    BBH populations being compared
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: SSE
  role: Analytical single-star evolution fits (Hurley et al.) underlying COMPAS's
    stellar evolution in both models
- name: pandas
  role: Writes the ratio and maximum-mass table to ce_model_comparison.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:6a86a61fac65bea4471e132632b7cb58eb670e19f0f632f5c25a61fe36166d53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - SSE
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# common-envelope-survival-model-comparison

## Summary

Quantifies how switching COMPAS's common-envelope prescription from the Fiducial (pessimistic) model to the optimistic CE model — in which Hertzsprung-gap donors survive common envelope — changes total binary black hole merger rates and maximum BBH masses at fixed metallicity.

## When to use

Use when comparing COMPAS population-synthesis runs that differ only in CE survival treatment (pessimistic vs optimistic, HG donors surviving) to quantify the effect on merging-BBH counts per 1e9 solar masses of star formation at specific metallicities.

## When NOT to use

- The analysis goal is sensitivity to a different parameter (e.g. αλ from 0.1 to 0.01, or LBV wind mass-loss rate 1.5e-4 to 1e-3 Msun/yr) rather than the CE survival prescription
- An absolute cosmological merger rate is needed — this skill reports per-1e9-Msun star-formation counts, not the redshift-dependent convolved rate
- The comparison involves binary_c, StarTrack, or BSE codes rather than COMPAS Fiducial vs optimistic-CE output

## Inputs

- Published COMPAS simulation output for the Fiducial (pessimistic CE) run (www.sr.bham.ac.uk/compas/)
- Published COMPAS simulation output for the optimistic-CE run in which Hertzsprung-gap donors survive common envelope
- Metallicity subsets: Z=10% solar, Z=5% solar, and Z=0.002

## Outputs

- Optimistic/Fiducial total BBH merger-rate ratios (~3 at 10% Zsun, ~2 at 5% Zsun)
- Maximum total BBH mass per model at Z=0.002 (Fiducial ~50 Msun vs optimistic ~60 Msun)
- ce_model_comparison.csv with model, metallicity, and value columns (written via pandas)

## How to apply

Retrieve the published COMPAS Fiducial and optimistic-CE simulation results from http://www.sr.bham.ac.uk/compas/. For each model, filter for binary black holes that merge within a Hubble time at Z=10% solar and Z=5% solar, normalizing counts per 1e9 solar masses of star formation (Fiducial references: ~2e4 at 10% solar, ~3e4 at 5% solar). Compute the optimistic/Fiducial total BBH merger-rate ratio at each metallicity and compare the maximum total BBH mass at Z=0.002 between models. Expect the optimistic treatment to raise the rate ~3x at 10% Zsun and ~2x at 5% Zsun, because letting HG donors survive CE retains systems that the Fiducial model would merge into one star. Write ratios and maximum masses (with model, metallicity, and value columns) to ce_model_comparison.csv using pandas.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate the Fiducial and optimistic-CE BBH populations being compared) — http://github.com/TeamCOMPAS/COMPAS_
- **SSE** (Analytical single-star evolution fits (Hurley et al.) underlying COMPAS's stellar evolution in both models)
- **pandas** (Writes the ratio and maximum-mass table to ce_model_comparison.csv)

## Examples

```
pd.DataFrame(rows, columns=['model','metallicity','value']).to_csv('ce_model_comparison.csv', index=False)
```

## Evaluation signals

- Tabulated optimistic/Fiducial rate ratio is ~3 at Z=10%Zsun and ~2 at Z=5%Zsun
- Fiducial counts match ~2e4 merging BBHs per 1e9 Msun star formation at 10% solar and ~3e4 at 5% solar
- Maximum total BBH mass at Z=0.002 rises from ~50 Msun (Fiducial) to ~60 Msun (optimistic)
- ce_model_comparison.csv contains model, metallicity, and value columns and only the two CE models being compared
- Both models are filtered identically: only BBHs merging within a Hubble time, normalized per 1e9 solar masses of star formation

## Limitations

- Only individual parameters are varied independently relative to the Fiducial model; full exploration of the multidimensional parameter space (e.g. alpha, fLBV) is deferred to future studies
- The merger-rate comparison to LIGO is approximate; an accurate rate requires convolution of metallicity-specific redshift-dependent star formation rate with the time delay distribution, which was not done
- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model

## Evidence

- [discussion] optimistic rate ratio: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙ and ∼2 times higher at Z=5%Z⊙"
- [discussion] fiducial normalization counts: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [methods] COMPAS availability: "Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_"
- [discussion] published results repository: "We make the results of our simulations available at http://www.sr.bham.ac.uk/compas/."
- [readme] what COMPAS evolves: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants."
