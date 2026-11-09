---
name: exoplanet-transit-photometry-modelling
description: Use when multi-instrument transit photometry (e.g. CHEOPS visits plus
  two TESS sectors) is available for a multi-planet system and per-planet radii and
  orbital periods must be derived jointly with uncertainties.
when_to_use_negative:
- The goal is internal-structure or formation-history inference rather than transit
  parameter retrieval — use the downstream plaNETic/BICEPS internal structure modelling
  framework with the derived radii instead.
- Only TESS photometry of a small planet with low per-transit SNR and large photometric
  gaps is available — the article explicitly cautions against relying on such TESS-only
  radius retrievals.
- The system has no transits (e.g. RV-only planets) — transit photometry modelling
  does not apply.
tools:
- name: CHEOPS
  role: High-precision space photometry source providing 9.6 days of transit observations
    of HIP 29442
- name: TESS
  role: Source of the two photometric sectors modelled jointly with the CHEOPS data
    to derive planetary radii
- name: Python
  role: Implementation language for the joint transit fitting and detrending workflow
    (plaNETic ecosystem requires Python 3.9)
  repo: https://github.com/joannegger/plaNETic
- name: plaNETic
  role: 'Downstream consumer: takes the retrieved planetary and stellar parameters
    to compute internal-structure posteriors via full grid accept-reject sampling'
  repo: https://github.com/joannegger/plaNETic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:a20b24df3b34dbbfc767c6ac224b33304369f9cd612673ea8504f219979574ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannegger/plaNETic
  tools:
  - CHEOPS
  - Python
  - TESS
  - plaNETic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# exoplanet-transit-photometry-modelling

## Summary

Jointly model transit light curves from a high-precision photometer (CHEOPS, 9.6 days on HIP 29442) with multiple TESS sectors to derive per-planet radii (R⊕) and orbital periods with posterior uncertainties, and flag TESS-driven deviations for small, low-SNR planets.

## When to use

Use when multi-instrument transit photometry (e.g. CHEOPS visits plus two TESS sectors) is available for a multi-planet system and per-planet radii and orbital periods must be derived jointly with uncertainties.

## When NOT to use

- The goal is internal-structure or formation-history inference rather than transit parameter retrieval — use the downstream plaNETic/BICEPS internal structure modelling framework with the derived radii instead.
- Only TESS photometry of a small planet with low per-transit SNR and large photometric gaps is available — the article explicitly cautions against relying on such TESS-only radius retrievals.
- The system has no transits (e.g. RV-only planets) — transit photometry modelling does not apply.

## Inputs

- CHEOPS light curves for HIP 29442 (9.6 days total observing time) from the public space-telescope archive
- Two sectors of TESS light curves for HIP 29442 from the TESS archive
- Host-star and planetary prior parameters (e.g. stellar_planetary_parameters.csv as used by the associated plaNETic workflow)

## Outputs

- Per-planet transit-fit posteriors: radii in R⊕ and orbital periods in days (here 3.410±0.046, 1.551±0.045, 1.538±0.049 R⊕; 13.6, 3.5, 6.4 d)
- Detrended, normalized light curves with instrumental systematics removed
- Fitted values table (hip29442_photometry_fit.csv)
- Deviation assessment of retrieved radii vs. discovery-paper medians (e.g. >3σ flag for planet d)

## How to apply

Retrieve the CHEOPS light curves (9.6 days total on HIP 29442/TOI-469) and the two TESS sectors from their public space-telescope archives. Normalize and detrend each light curve, removing instrumental systematics and long-term stellar trends while preserving the transit signals of planets b, c, and d. Fit a single joint transit model across the CHEOPS and both TESS datasets simultaneously (rather than per-instrument) to combine per-transit information and obtain posterior uncertainties on each planet's radius and orbital period. Compare the retrieved radius of planet d against the discovery-paper median to flag expected >3σ deviations arising from low per-transit SNR and large gaps in TESS photometry of small planets. Validate by requiring the fitted radii to agree within uncertainties with 3.410±0.046, 1.551±0.045, and 1.538±0.049 R⊕ (planets b, c, d) and periods near 13.6, 3.5, and 6.4 days, then export all fitted values (e.g. to hip29442_photometry_fit.csv).

## Related tools

- **CHEOPS** (High-precision space photometry source providing 9.6 days of transit observations of HIP 29442)
- **TESS** (Source of the two photometric sectors modelled jointly with the CHEOPS data to derive planetary radii)
- **Python** (Implementation language for the joint transit fitting and detrending workflow (plaNETic ecosystem requires Python 3.9)) — https://github.com/joannegger/plaNETic
- **plaNETic** (Downstream consumer: takes the retrieved planetary and stellar parameters to compute internal-structure posteriors via full grid accept-reject sampling) — https://github.com/joannegger/plaNETic

## Evaluation signals

- Retrieved radii agree within uncertainties with 3.410±0.046, 1.551±0.045, and 1.538±0.049 R⊕ for planets b, c, and d
- Retrieved orbital periods are consistent with ~13.6, 3.5, and 6.4 days for planets b, c, and d
- All fitted values are written to hip29442_photometry_fit.csv with posterior uncertainties attached
- Radius of planet d is checked against the discovery-paper median and the expected >3σ deviation is flagged rather than treated as a fit failure
- Light curves are verified to retain transit signals after detrending (systematics and long-term stellar trends removed without erasing transits)

## Limitations

- For planet d the radius deviates by more than 3σ from the discovery paper's median value; caution is required when using TESS photometry for small planets with low per-transit SNR and large gaps.
- Joint-fit quality depends on total CHEOPS baseline (here only 9.6 days); limited observing time increases radius uncertainties.
- The downstream plaNETic code requires GPU access for efficiency; CPU-only runs drastically increase computation time, and missing trained DNNs due to GitHub Large File Storage issues block execution.

## Evidence

- [abstract] Joint modelling of 9.6 days CHEOPS data with two TESS sectors yields radii and periods: "We observed HIP 29442 with CHEOPS for a total of 9.6 days, which we modelled jointly with two sectors of TESS data"
- [abstract] TESS small-planet caution (>3σ deviation for planet d): "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [abstract] Reported radii and periods with uncertainties: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
- [methods] Python 3.9 installation for the associated tooling: "To install the code, create a new virtual environment with Python 3.9, then git clone plaNETic and run 'pip install .'"
- [readme] Parameter file format for downstream internal-structure modelling of the derived parameters: "Parameter file 'stellar_planetary_parameters.csv' with the observed properties of the host star and all planets in the system that you would like to model"
