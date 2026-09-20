---
name: multi-instrument-lightcurve-joint-fitting
description: Use when a planet (or multi-planet system) has transit light curves from
  more than one instrument (e.g. CHEOPS plus TESS sectors) and you need precise planetary
  radii and orbital periods before downstream modelling such as internal-structure
  inference.
when_to_use_negative:
- Only single-instrument photometry is available and no joint constraint across instruments
  is needed — the discovery-paper values may suffice, but note TESS-only radii for
  small, low-per-transit-SNR planets with large data gaps can deviate >3σ from the
  true value.
- The analysis goal is internal-structure/formation inference rather than radius/period
  retrieval — use the downstream plaNETic Bayesian internal-structure framework instead,
  feeding it the radii from this fit.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CHEOPS
  role: Primary high-precision photometric instrument providing 9.6 days of transit
    light curves that anchor the joint fit
- name: TESS
  role: Two sectors of photometry modelled jointly with CHEOPS to constrain periods
    and radii
- name: Python
  role: Environment for detrending light curves and running the joint transit model;
    plaNETic requires a Python 3.9 virtual environment
- name: plaNETic
  role: Downstream consumer of the fitted radii for Bayesian internal structure modelling
    (not part of the photometric fit itself)
  repo: github.com/joannegger/plaNETic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:d73e096a7d3f54a492c0813b0945bd824865ec227f4c8d63c950585466b579f3
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-instrument-lightcurve-joint-fitting

## Summary

Jointly model transit photometry from multiple space telescopes — here 9.6 days of CHEOPS observations of HIP 29442 (TOI-469) combined with two TESS sectors — to derive per-planet radii and orbital periods with posterior uncertainties, improving precision over single-instrument fits.

## When to use

Use when a planet (or multi-planet system) has transit light curves from more than one instrument (e.g. CHEOPS plus TESS sectors) and you need precise planetary radii and orbital periods before downstream modelling such as internal-structure inference.

## When NOT to use

- Only single-instrument photometry is available and no joint constraint across instruments is needed — the discovery-paper values may suffice, but note TESS-only radii for small, low-per-transit-SNR planets with large data gaps can deviate >3σ from the true value.
- The analysis goal is internal-structure/formation inference rather than radius/period retrieval — use the downstream plaNETic Bayesian internal-structure framework instead, feeding it the radii from this fit.

## Inputs

- CHEOPS light curves (~9.6 days total observing time) for HIP 29442 from the public space-telescope archive
- Two sectors of TESS photometry for HIP 24942 (TOI-469) from the TESS archive
- Prior/discovery-paper transit ephemerides and per-planet parameters for initialization

## Outputs

- Planetary radii with posterior uncertainties (3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, d)
- Orbital periods (~13.6, 3.5, and 6.4 days for planets b, c, d)
- Machine-readable fit results, e.g. hip29442_photometry_fit.csv

## How to apply

Retrieve the light curves from the space-telescope archives: 9.6 days of CHEOPS data and two TESS sectors of the target (HIP 29442). Normalize and detrend each light curve individually, removing instrumental systematics and long-term stellar trends while preserving the transit signals. Fit a single joint transit model across the CHEOPS and both TESS datasets simultaneously so that shared parameters (periods, ephemerides) are constrained by all data while per-instrument systematics are handled separately, yielding planetary radii and periods with posterior uncertainties. Critically compare retrieved radii against published discovery-paper values, flagging any >3σ deviations — for small, low-SNR planets the TESS-only radius can be biased, as happened for planet d of HIP 29442. Write the fitted values and uncertainties to a machine-readable output such as hip29442_photometry_fit.csv for use in later internal-structure modelling (e.g. with plaNETic).

## Related tools

- **CHEOPS** (Primary high-precision photometric instrument providing 9.6 days of transit light curves that anchor the joint fit)
- **TESS** (Two sectors of photometry modelled jointly with CHEOPS to constrain periods and radii)
- **Python** (Environment for detrending light curves and running the joint transit model; plaNETic requires a Python 3.9 virtual environment)
- **plaNETic** (Downstream consumer of the fitted radii for Bayesian internal structure modelling (not part of the photometric fit itself)) — github.com/joannegger/plaNETic

## Examples

```
git clone plaNETic && conda create -n planetic python=3.9 && pip install .
```

## Evaluation signals

- Retrieved radii agree within uncertainties with 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d.
- Retrieved orbital periods are near 13.6, 3.5, and 6.4 days for planets b, c, and d.
- Radius of planet d is compared against the discovery-paper median; a >3σ deviation is an expected and informative flag caused by low per-transit SNR and large gaps in TESS photometry of small planets.
- All fitted values are written to hip29442_photometry_fit.csv with per-planet posterior uncertainties.
- Detrended light curves preserve transit depths (no over-fitting of instrumental systematics that would bias radius estimates).

## Limitations

- Caution is required when using TESS photometry for small planets: the radius of planet d deviates by more than 3σ from the discovery paper's median value due to low per-transit SNR and large gaps in TESS coverage.
- Detrending must remove instrumental systematics and long-term stellar trends without destroying transit signals; aggressive detrending of shallow transits biases radii.
- Joint-fit precision depends on CHEOPS coverage; here only 9.6 days of CHEOPS time were available, so per-transit SNR of the smallest planets limits the radius precision.

## Evidence

- [abstract] Joint radii/periods from CHEOPS + two TESS sectors: "We observed HIP 29442 with CHEOPS for a total of 9.6 days, which we modelled jointly with two sectors of TESS data to derive planetary radii"
- [abstract] Reported radii and periods: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
- [abstract] Caution on TESS-only radii for small planets: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [intro] Downstream internal-structure modelling context: "plaNETic is a neural network-based Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth."
- [readme] Validation example in repo README: "To ensure the code is running correctly, we recommend running it for TOI-238 b. To do that, navigate into 'run_grid/TOI-238_Egger+' and run the executable 'run_grid_TOI-238.py'"
