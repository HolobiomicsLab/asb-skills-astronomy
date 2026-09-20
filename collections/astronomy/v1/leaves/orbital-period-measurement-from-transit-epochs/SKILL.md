---
name: orbital-period-measurement-from-transit-epochs
description: Use when raw or detrended light curves from one or more photometric instruments
  show transits of multiple planets and you need to measure orbital periods and transit
  depths (radii) jointly rather than per-dataset, e.g. CHEOPS + TESS multi-sector
  photometry of a system like HIP 29442 (TOI-469).
when_to_use_negative:
- Only single-sector TESS photometry of small planets with low per-transit SNR and
  large data gaps is available — the article shows the recovered radius of planet
  d deviates >3σ from the discovery-paper median in exactly this situation
- Radial velocities alone (ESPRESSO/HIRES) without transit photometry — periods from
  RVs alone do not constrain transit-derived radii
- The goal is internal-structure inference rather than transit ephemeris/depth measurement
  — use the downstream plaNETic framework instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: CHEOPS
  role: Source of 9.6 days of high-precision photometry used to measure transit epochs,
    depths, and periods
- name: TESS
  role: Two additional sectors of photometry modelled jointly with CHEOPS to derive
    planetary radii and periods
- name: Python
  role: Implementation environment for detrending and the joint transit fit (plaNETic
    runs under Python 3.9)
  repo: https://github.com/joannegger/plaNETic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:c40f144adcd4ded806fb4ad2c08c51a69671f119afec10559731aac2ba9097df
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
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# orbital-period-measurement-from-transit-epochs

## Summary

Jointly fit a photometric transit model across multiple space-telescope datasets (here 9.6 days of CHEOPS plus two TESS sectors) to derive per-planet orbital periods and radii with posterior uncertainties for a multiplanet system.

## When to use

Use when raw or detrended light curves from one or more photometric instruments show transits of multiple planets and you need to measure orbital periods and transit depths (radii) jointly rather than per-dataset, e.g. CHEOPS + TESS multi-sector photometry of a system like HIP 29442 (TOI-469).

## When NOT to use

- Only single-sector TESS photometry of small planets with low per-transit SNR and large data gaps is available — the article shows the recovered radius of planet d deviates >3σ from the discovery-paper median in exactly this situation
- Radial velocities alone (ESPRESSO/HIRES) without transit photometry — periods from RVs alone do not constrain transit-derived radii
- The goal is internal-structure inference rather than transit ephemeris/depth measurement — use the downstream plaNETic framework instead

## Inputs

- CHEOPS light curves (~9.6 days total observing time) for HIP 29442
- Two TESS sectors of photometry for HIP 29442
- Host-star and planetary parameter table (e.g. stellar_planetary_parameters.csv) for prior constraints

## Outputs

- Per-planet orbital periods with posterior uncertainties (13.6, 3.5, 6.4 days for b, c, d)
- Per-planet planetary radii with posterior uncertainties (3.410 ± 0.046, 1.551 ± 0.045, 1.538 ± 0.049 R⊕)
- Fitted-values table (e.g. hip29442_photometry_fit.csv)

## How to apply

Retrieve the CHEOPS light curves (9.6 days total observing time) and the two TESS sectors for the target from their public archives. Normalize and detrend each light curve, removing instrumental systematics and long-term stellar trends while preserving transit signals. Fit a joint transit model across all datasets simultaneously (rather than independently), so ephemerides and depth ratios are constrained by every transit epoch, and derive per-planet radii and orbital periods with posterior uncertainties. Validate the recovered periods (13.6, 3.5, and 6.4 days for planets b, c, d of HIP 29442) and radii (3.410 ± 0.046, 1.551 ± 0.045, 1.538 ± 0.049 R⊕) against published values, and write fitted values to an output table such as hip29442_photometry_fit.csv.

## Related tools

- **CHEOPS** (Source of 9.6 days of high-precision photometry used to measure transit epochs, depths, and periods)
- **TESS** (Two additional sectors of photometry modelled jointly with CHEOPS to derive planetary radii and periods)
- **Python** (Implementation environment for detrending and the joint transit fit (plaNETic runs under Python 3.9)) — https://github.com/joannegger/plaNETic

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('hip29442_photometry_fit.csv'); assert all(abs(df['period_days'] - [13.6, 3.5, 6.4]) < 0.1)"
```

## Evaluation signals

- Recovered radii agree within uncertainties with the reported values 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, d
- Recovered orbital periods fall near 13.6, 3.5, and 6.4 days for planets b, c, d
- Light-curve detrending preserves transit signals — no systematic attenuation of transit depths in any instrument's residuals
- Fitted values written to hip29442_photometry_fit.csv with per-planet posterior uncertainties attached
- Any deviation of a small-planet radius (like planet d) from the discovery-paper median is explainable by low per-transit SNR and TESS data gaps, not by a modelling error

## Limitations

- TESS photometry of small planets with low per-transit SNR and large gaps can yield radii deviating by more than 3σ from discovery-paper medians — caution is required
- CHEOPS coverage is limited (9.6 days total), so period determination relies on combining epochs across instruments
- The downstream plaNETic framework needs GPU access to run efficiently; CPU-only execution drastically increases computation time, and trained DNNs may be missing due to GitHub's Large File Storage issue

## Evidence

- [abstract] Joint CHEOPS+TESS modelling yields radii and periods: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
- [abstract] Joint modelling of CHEOPS and two TESS sectors: "We observed HIP 29442 with CHEOPS for a total of 9.6 days, which we modelled jointly with two sectors of TESS data to derive planetary radii"
- [abstract] TESS caution for small planets: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [readme] plaNETic runs on Python 3.9 after pip install: "To install the code, create a new virtual environment with Python 3.9, then git clone plaNETic and run 'pip install .'"
