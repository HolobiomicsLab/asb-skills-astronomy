---
name: metallicity-dependent-merger-rate-normalization
description: Use when converting a simulated population of double compact objects
  (e.g. from COMPAS) into per-metallicity merger yields and a rescaled Gpc^-3 per
  year rate to compare with gravitational-wave observations.
when_to_use_negative:
- 'When an accurate cosmological merger rate is required: the paper notes this needs
  convolution of metallicity-specific redshift-dependent star formation rate with
  the time delay distribution, which this simple rescaling omits'
- When the formation channel or binary physics differs substantially from the Fiducial
  model (e.g. optimistic common-envelope treatment, alpha_lambda=0.01, or enhanced
  LBV winds), since yields shift by factors of ~2-10
- When targeting neutron-star or white-dwarf compact-object channels rather than binary
  black holes
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3360
tools:
- name: COMPAS
  role: Rapid binary population synthesis code that evolves the initial binary population
    through mass transfer and common envelope to double compact objects and provides
    GW merger times for rate normalization
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS importance-sampling algorithm usable to efficiently sample rare merging-BBH
    progenitors before normalizing rates
  repo: http://github.com/TeamCOMPAS/COMPAS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:4460c840289eae5038654e27b3404caf040d6896f9cc3fe89697947affe47503
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metallicity-dependent-merger-rate-normalization

## Summary

Normalizes COMPAS-derived merging binary black hole yields per unit star formation (1e9 solar masses) at discrete low metallicities (10% and 5% solar) and rescales them to a volumetric BBH merger rate at z=0, benchmarked against LIGO's empirical rate estimate.

## When to use

Use when converting a simulated population of double compact objects (e.g. from COMPAS) into per-metallicity merger yields and a rescaled Gpc^-3 per year rate to compare with gravitational-wave observations.

## When NOT to use

- When an accurate cosmological merger rate is required: the paper notes this needs convolution of metallicity-specific redshift-dependent star formation rate with the time delay distribution, which this simple rescaling omits
- When the formation channel or binary physics differs substantially from the Fiducial model (e.g. optimistic common-envelope treatment, alpha_lambda=0.01, or enhanced LBV winds), since yields shift by factors of ~2-10
- When targeting neutron-star or white-dwarf compact-object channels rather than binary black holes

## Inputs

- COMPAS population output at 10% solar metallicity (Z=0.002): double compact objects with masses and GW merger times
- COMPAS population output at 5% solar metallicity (Z=0.001): double compact objects with masses and GW merger times
- Star-formation mass basis (per 1e9 solar masses) used for yield normalization

## Outputs

- bbh_merger_rates.csv with per-metallicity merging-BBH yields and rescaled volumetric rates
- Merging-BBH counts per 1e9 solar masses of star formation at 10% and 5% solar metallicity
- Volumetric BBH formation rate at z=0 in Gpc^-3 per year

## How to apply

Evolve the initial binary population with COMPAS under the Fiducial model (delayed Fryer supernova, alpha_lambda=0.1, fLBV=1.5, Kroupa IMF to 100 Msun, flat mass-ratio, flat-in-log semimajor axis, zero eccentricity) at 10% and 5% solar metallicity. Compute the gravitational-wave merger time for each double compact object and select only binary black holes that merge within a Hubble time. Normalize the merging-BBH counts per 1e9 solar masses of star formation at each metallicity (fiducial expectation ~2e4 at 10% Zsun and ~3e4 at 5% Zsun), then rescale to a volumetric BBH formation rate at z=0 (~300 Gpc^-3 per year) for comparison with the LIGO empirical estimate of 9-240 Gpc^-3 per year. Write per-metallicity yields and rescaled rates to a tabular output such as bbh_merger_rates.csv. Note that a truly accurate rate would require convolving metallicity-specific redshift-dependent star formation with the time-delay distribution, which this normalization approximates.

## Related tools

- **COMPAS** (Rapid binary population synthesis code that evolves the initial binary population through mass transfer and common envelope to double compact objects and provides GW merger times for rate normalization) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS importance-sampling algorithm usable to efficiently sample rare merging-BBH progenitors before normalizing rates) — http://github.com/TeamCOMPAS/COMPAS

## Examples

```
./COMPAS --population-model --metallicity 0.002 --kick-magnitude-distribution MAXWELLIAN --fryer-supernova-engine DELAYED --common-envelope-alpha 1.0 --common-envelope-lambda 0.1 --LBV-mass-loss 1.5
```

## Evaluation signals

- Merging-BBH yield per 1e9 solar masses of star formation should be ~2e4 at 10% solar metallicity and ~3e4 at 5% solar metallicity under the Fiducial model
- Rescaled BBH formation rate at z=0 should be ~300 Gpc^-3 per year, comparable to (within roughly an order of magnitude above) the LIGO empirical estimate of 9-240 Gpc^-3 per year
- Yields should increase as metallicity decreases from 10% to 5% solar (2e4 to 3e4), reflecting stronger line-driven winds at higher Z
- Only double compact objects with gravitational-wave merger times shorter than a Hubble time should enter the normalized counts
- Robustness check: switching to the rapid Fryer supernova model should not significantly alter the yields for heavy BBHs

## Limitations

- An accurate merger rate requires convolution of metallicity-specific redshift-dependent star formation rate with the time delay distribution; the ~300 Gpc^-3 per year figure is approximate
- Merger-rate comparison with LIGO is rough: the fiducial ~300 Gpc^-3 per year exceeds the empirical LIGO estimate of 9-240 Gpc^-3 per year
- Rates are highly sensitive to binary physics parameters: optimistic common envelope raises rates ~3x (10% Zsun) / ~2x (5% Zsun); alpha_lambda=0.01 lowers them ~2x / ~4x; enhanced LBV winds cut LVT151012-like mergers ~10x at 5% Zsun
- The single-star (SSE) model grid extends only to 50 solar masses and is extrapolated above that limit, affecting massive progenitors
- Stellar rotation, tides, and wind accretion are not included in the Fiducial model

## Evidence

- [discussion] Fiducial yields per unit star formation: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] Rescaled rate vs LIGO: [evidence span withheld — anchor status: not_found]
- [discussion] Need for full rate calculation: "An accurate calculation of the merger rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution"
- [discussion] Initial distribution assumptions: [evidence span withheld — anchor status: not_found]
- [intro] COMPAS role: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [readme] Zenodo data sharing of settings and outputs: "make your input settings and output data publicly available by uploading it to the [COMPAS Zenodo community]"
