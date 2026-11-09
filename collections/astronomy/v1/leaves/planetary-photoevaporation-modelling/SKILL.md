---
name: planetary-photoevaporation-modelling
description: Use when a confirmed planet's radius, mass, density, and host-star irradiation
  properties are available and a present-day atmospheric escape rate is required,
  especially to test sensitivity to the choice of photoevaporation model.
when_to_use_negative:
- The planet's radius and mass are not yet constrained by a joint fit (e.g. candidate
  with no RV mass), so no reliable irradiation/escape calculation can be anchored
- The goal is flux-contamination assessment of TESS photometry rather than atmospheric
  escape — TESS-cont addresses crowding metrics (CROWDSAP, FLFRCSAP), not mass loss
tools:
- name: Python
  role: Language used to run both photoevaporation models on the identical planet/stellar
    parameter set and propagate uncertainties
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:2702ab7ec873dec82c45299bfcf1dbdc402da3909b9191f1edac91d99c3073af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# planetary-photoevaporation-modelling

## Summary

Estimates the present-day atmospheric mass-loss rate of a confirmed transiting planet by running two independent photoevaporation models on the same set of planet (Rp, Mp, ρp) and host-star irradiation parameters, then contrasting the predictions to expose model-choice dependence.

## When to use

Use when a confirmed planet's radius, mass, density, and host-star irradiation properties are available and a present-day atmospheric escape rate is required, especially to test sensitivity to the choice of photoevaporation model.

## When NOT to use

- The planet's radius and mass are not yet constrained by a joint fit (e.g. candidate with no RV mass), so no reliable irradiation/escape calculation can be anchored
- The goal is flux-contamination assessment of TESS photometry rather than atmospheric escape — TESS-cont addresses crowding metrics (CROWDSAP, FLFRCSAP), not mass loss

## Inputs

- Jointly inferred planet parameters: radius Rp = 6.25 ± 0.24 R⊕, mass Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3
- Host-star properties required for the stellar irradiation calculation (e.g. luminosity/effective temperature)
- Two photoevaporation model implementations runnable in Python

## Outputs

- Present-day atmospheric mass-loss rate from model 1: 0.013 ± 0.008 M⊕ Gyr−1
- Present-day atmospheric mass-loss rate from model 2: 0.17 ± 0.12 M⊕ Gyr−1
- Uncertainty estimates obtained by propagating the planetary parameter uncertainties

## How to apply

First assemble the inferred planetary parameters from the joint Bayesian fit — for TOI-5005 b, Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3 — together with the host-star properties needed for the irradiation calculation. Run the first photoevaporation model in Python on this parameter set to compute the present-day mass-loss rate, then run the second, alternative model on the identical parameter set. Contrast the two rates (for TOI-5005 b: 0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1) and propagate the planetary parameter uncertainties to quote an uncertainty on each rate. The rationale is that model choice can dominate the prediction, so a single-model estimate understates the true uncertainty.

## Related tools

- **Python** (Language used to run both photoevaporation models on the identical planet/stellar parameter set and propagate uncertainties)

## Evaluation signals

- Reproduced rates agree with the reported values of 0.013 ± 0.008 M⊕/Gyr and 0.17 ± 0.12 M⊕/Gyr within their quoted uncertainties
- Both models were run on the identical parameter set (Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, ρp = 0.74 ± 0.16 g cm−3) — verify inputs match
- Uncertainties on each rate are quoted and traceable to the planetary parameter uncertainties from the Bayesian fit
- The strong model-choice dependence (roughly an order-of-magnitude difference between the two rates) is reproduced and reported

## Limitations

- The prediction depends strongly on the choice of photoevaporation model — contrasting predictions (0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1) mean a single-model result should not be over-interpreted
- Estimates are for the present-day epoch only; the article's finding does not establish the full evolutionary history

## Evidence

- [abstract] Contrasting atmospheric mass-loss predictions depending on photoevaporation model: "found contrasting predictions depending on the choice of photoevaporation model (0.013 ± 0.008 M ⊕ Gyr −1 vs. 0.17 ± 0.12 M ⊕ Gyr −1 )"
- [abstract] Estimation of present-day atmospheric mass-loss rate: "We also estimated the present-day atmospheric mass-loss rate of TOI-5005 b"
- [abstract] Planet parameters anchoring the photoevaporation calculation: [evidence span withheld — anchor status: not_found]
