---
name: transit-depth-radius-retrieval
description: Use when photometric time series from two or more missions (e.g. CHEOPS
  + TESS) contain transit signals of small planets (≲3.5 R⊕) whose radii need per-transit-SNR-aware
  joint retrieval with uncertainty validation.
when_to_use_negative:
- The target's transits are already characterized with high per-transit SNR from a
  single instrument — the joint multi-sector fit is motivated by small, low-SNR planets.
- Only RVs (ESPRESSO/HIRES) are available and no transit photometry exists — transit-depth
  retrieval cannot produce radii without light curves.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CHEOPS
  role: 'Primary high-precision photometry source: 9.6 days of light curves jointly
    modelled with TESS to derive planetary radii'
- name: TESS
  role: Two sectors of photometry modelled jointly with CHEOPS to derive planetary
    radii and periods
- name: Python
  role: Implementation environment for detrending and the joint transit fit; downstream
    plaNETic runs on Python 3.9
  repo: https://github.com/joannegger/plaNETic
- name: plaNETic
  role: 'Downstream consumer of the retrieved radii: neural network-based Bayesian
    internal structure modelling for planets 0.5–15 Mearth'
  repo: https://github.com/joannegger/plaNETic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:6d760038740d50f5921508c344e197342cc0ca70e448285141c4992f637cd6b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannegger/plaNETic
  tools:
  - CHEOPS
  - Python
  - TESS
  - plaNETic
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# transit-depth-radius-retrieval

## Summary

Jointly model multi-instrument transit photometry (e.g. 9.6 days of CHEOPS light curves plus two TESS sectors) to retrieve per-planet transit depths, radii, and orbital periods with posterior uncertainties. Required because TESS-only radii for small, low-SNR planets can be biased (planet d of HIP 29442 deviates >3σ from the discovery-paper median).

## When to use

Use when photometric time series from two or more missions (e.g. CHEOPS + TESS) contain transit signals of small planets (≲3.5 R⊕) whose radii need per-transit-SNR-aware joint retrieval with uncertainty validation.

## When NOT to use

- The target's transits are already characterized with high per-transit SNR from a single instrument — the joint multi-sector fit is motivated by small, low-SNR planets.
- Only RVs (ESPRESSO/HIRES) are available and no transit photometry exists — transit-depth retrieval cannot produce radii without light curves.

## Inputs

- CHEOPS light curves (~9.6 days of observing for HIP 29442)
- Two TESS sectors of light curves for the same target
- Prior transit ephemerides / discovery-paper parameter medians for validation

## Outputs

- Per-planet radii with posterior uncertainties (R⊕), e.g. 3.410±0.046, 1.551±0.045, 1.538±0.049 for HIP 29442 b/c/d
- Orbital periods (e.g. 13.6, 3.5, 6.4 days for planets b, c, d)
- Fitted-parameter table (e.g. hip29442_photometry_fit.csv)

## How to apply

Retrieve the CHEOPS light curves (9.6 days total for HIP 29442) and both TESS sectors from public space-telescope archives, then normalize and detrend each light curve to remove instrumental systematics and long-term stellar trends while preserving transit signals. Fit a single joint transit model across all datasets simultaneously so per-planet radii and periods are constrained by every transit, deriving posterior uncertainties per planet. For HIP 29442 this yields radii of 3.410±0.046, 1.551±0.045, and 1.538±0.049 R⊕ at periods of 13.6, 3.5, and 6.4 days for planets b, c, d. Explicitly compare small-planet radii against discovery-paper values: planet d deviates by >3σ due to low per-transit SNR and large gaps in TESS photometry, so flag such deviations rather than silently accepting a TESS-dominated fit. Write fitted values (e.g. to hip29442_photometry_fit.csv) before feeding the radii into downstream internal-structure modelling such as plaNETic.

## Related tools

- **CHEOPS** (Primary high-precision photometry source: 9.6 days of light curves jointly modelled with TESS to derive planetary radii)
- **TESS** (Two sectors of photometry modelled jointly with CHEOPS to derive planetary radii and periods)
- **Python** (Implementation environment for detrending and the joint transit fit; downstream plaNETic runs on Python 3.9) — https://github.com/joannegger/plaNETic
- **plaNETic** (Downstream consumer of the retrieved radii: neural network-based Bayesian internal structure modelling for planets 0.5–15 Mearth) — https://github.com/joannegger/plaNETic

## Examples

```
python run_joint_transit_fit.py --target HIP29442 --instruments cheops,tess --n_sectors 2 --output hip29442_photometry_fit.csv
```

## Evaluation signals

- Retrieved radii agree within uncertainties with the reference values 3.410±0.046, 1.551±0.045, and 1.538±0.049 R⊕ for planets b, c, d.
- Retrieved orbital periods fall near 13.6, 3.5, and 6.4 days for planets b, c, and d respectively.
- Small-planet radius (planet d) is explicitly compared against the discovery-paper median; a >3σ deviation is expected and must be flagged, not treated as a fitting failure.
- All fitted values are written to the output CSV (hip29442_photometry_fit.csv) with per-planet posterior uncertainties.
- Detrended light curves retain transit signals — normalization must not flatten in-transit flux.

## Limitations

- TESS photometry for small planets with low per-transit SNR and large observing gaps can yield radii deviating >3σ from discovery values (as for HIP 29442 planet d) — caution is required.
- CHEOPS coverage is limited (~9.6 days), constraining which transits of longer-period planets are captured.

## Evidence

- [abstract] Joint multi-instrument fit yields radii and periods: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
- [abstract] Joint modelling workflow with CHEOPS and TESS: "We observed HIP 29442 with CHEOPS for a total of 9.6 days, which we modelled jointly with two sectors of TESS data to derive planetary radii"
- [abstract] >3σ deviation for small planet d from TESS: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [full_text] Validation targets radii within uncertainties: [evidence span withheld — anchor status: not_found]
- [abstract] Downstream radii feed internal-structure modelling: "combining them with published RVs from ESPRESSO and HIRES provides us with ideal conditions to investigate the internal structure and formation pathways"
