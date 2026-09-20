---
name: stellar-lightcurve-detrending
description: Use when combining photometry from multiple instruments (e.g. CHEOPS
  visits plus several TESS sectors) to fit planetary transits, especially when small
  planets have low per-transit SNR and instrumental trends could bias radius posteriors.
when_to_use_negative:
- When only TESS photometry is available for a small planet with low per-transit SNR
  and large data gaps — the paper warns this can bias the radius by >3σ relative to
  discovery values
- When the analysis target is internal-structure modelling from already-published
  radii (use plaNETic-style Bayesian internal structure modelling instead of re-deriving
  radii from photometry)
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: CHEOPS
  role: Source of high-precision space photometry (9.6 days for HIP 29442) that is
    detrended and jointly modelled
- name: TESS
  role: Two sectors of archival photometry combined with CHEOPS in the joint transit
    fit
- name: plaNETic
  role: Downstream Bayesian internal-structure modelling framework that consumes the
    derived planetary radii
  repo: github.com/joannegger/plaNETic
- name: Python
  role: Implementation language for the detrending and joint transit fitting workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:7b89c033f8e9e57527771d908e77c20cb98a85b51d2b6b1f6fcbff15762c4d01
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHEOPS
  - Python
  - TESS
  - plaNETic
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# stellar-lightcurve-detrending

## Summary

Normalize and detrend space-telescope light curves (CHEOPS and TESS) by removing instrumental systematics and long-term stellar trends while preserving transit signals, so that a joint multi-instrument transit fit recovers unbiased planetary radii.

## When to use

Use when combining photometry from multiple instruments (e.g. CHEOPS visits plus several TESS sectors) to fit planetary transits, especially when small planets have low per-transit SNR and instrumental trends could bias radius posteriors.

## When NOT to use

- When only TESS photometry is available for a small planet with low per-transit SNR and large data gaps — the paper warns this can bias the radius by >3σ relative to discovery values
- When the analysis target is internal-structure modelling from already-published radii (use plaNETic-style Bayesian internal structure modelling instead of re-deriving radii from photometry)

## Inputs

- CHEOPS light curves (e.g. ~9.6 days of observations of HIP 29442) from the public space-telescope archive
- TESS light curves for two sectors of the same target
- List of target planets and expected orbital periods for transit masking/window selection

## Outputs

- Detrended, normalized light curves per instrument/visit suitable for joint transit fitting
- Joint-fit posterior values of per-planet radii (R⊕) and orbital periods (days) with uncertainties, e.g. written to hip29442_photometry_fit.csv

## How to apply

Retrieve the CHEOPS light curves (e.g. 9.6 days of observing for HIP 29442) and the relevant TESS sectors from their public space-telescope archives, then normalize and detrend each light curve individually, removing instrumental systematics and long-term stellar trends while taking care to preserve the transit signals of the target planets. Fit a joint transit model across the CHEOPS and both TESS datasets simultaneously rather than per-instrument, so per-planet radii and orbital periods are constrained by all transits at once. For small planets with few transits (here planet d, period 6.4 days), expect larger uncertainties and validate against discovery-paper values; the paper found planet d's radius deviating by more than 3σ from the discovery median when TESS coverage was gappy. Write the fitted radii and periods (here 3.410±0.046, 1.551±0.045, 1.538±0.049 R⊕ and 13.6, 3.5, 6.4 days) with posterior uncertainties to an output file such as hip29442_photometry_fit.csv.

## Related tools

- **CHEOPS** (Source of high-precision space photometry (9.6 days for HIP 29442) that is detrended and jointly modelled)
- **TESS** (Two sectors of archival photometry combined with CHEOPS in the joint transit fit)
- **plaNETic** (Downstream Bayesian internal-structure modelling framework that consumes the derived planetary radii) — github.com/joannegger/plaNETic
- **Python** (Implementation language for the detrending and joint transit fitting workflow)

## Examples

```
python fit_transits.py --cheops cheops_lc/ --tess tess_s1,tess_s2 --planets b,c,d --output hip29442_photometry_fit.csv
```

## Evaluation signals

- Retrieved radii agree within uncertainties with published joint-fit values 3.410±0.046, 1.551±0.045, and 1.538±0.049 R⊕ for planets b, c, d
- Recovered orbital periods are near 13.6, 3.5, and 6.4 days
- Radius of planet d may deviate by >3σ from the discovery-paper median; flag and explain such a deviation rather than forcing agreement
- Transit depths are visibly preserved after detrending (no over-fitting of in-transit points)
- All fitted values are written to hip29442_photometry_fit.csv

## Limitations

- Caution is required when using TESS photometry for small planets with low per-transit SNR and large gaps; the planet d radius deviated by more than 3σ from the discovery paper's median value
- Total CHEOPS coverage was limited (~9.6 days), constraining the number of transits per planet and hence radius precision

## Evidence

- [abstract] Joint modelling of CHEOPS and two TESS sectors to derive planetary radii: "We observed HIP 29442 with CHEOPS for a total of 9.6 days, which we modelled jointly with two sectors of TESS data"
- [other] Detrend each light curve while preserving transits: [evidence span withheld — anchor status: not_found]
- [abstract] TESS caution for small planets: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [abstract] Radii and periods recovered by joint fit: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
