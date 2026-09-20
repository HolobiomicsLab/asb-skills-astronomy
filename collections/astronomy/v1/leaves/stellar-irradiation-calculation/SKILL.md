---
name: stellar-irradiation-calculation
description: Use when a confirmed planet has well-constrained radius, mass, and density
  posteriors from a joint Bayesian fit, and a present-day atmospheric mass-loss rate
  must be estimated that requires host-star irradiation as input.
when_to_use_negative:
- When the goal is TESS photometric contamination analysis (flux fractions, CROWDSAP/FLFRCSAP,
  dilution of transit signals) — that is the TESS-cont skill, not irradiation-driven
  mass-loss estimation
- When the planet's radius, mass, and density are not yet constrained by a joint Bayesian
  fit, since the irradiation and mass-loss inputs would be too uncertain to propagate
edam_operation: http://edamontology.org/operation_0334
tools:
- name: Python
  role: Language used to run both photoevaporation models on the identical inferred
    planet and stellar parameter set and to propagate parameter uncertainties to the
    mass-loss rates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:2025aab8bd13dc1f46fc20a5c26174ba3cbc45c8b99fbe026ea4d532d8540941
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# stellar-irradiation-calculation

## Summary

Computes the present-day stellar irradiation environment of TOI-5005 b from its inferred planet parameters (Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕) and host-star properties, as the input needed to estimate the present-day atmospheric mass-loss rate with photoevaporation models.

## When to use

Use when a confirmed planet has well-constrained radius, mass, and density posteriors from a joint Bayesian fit, and a present-day atmospheric mass-loss rate must be estimated that requires host-star irradiation as input.

## When NOT to use

- When the goal is TESS photometric contamination analysis (flux fractions, CROWDSAP/FLFRCSAP, dilution of transit signals) — that is the TESS-cont skill, not irradiation-driven mass-loss estimation
- When the planet's radius, mass, and density are not yet constrained by a joint Bayesian fit, since the irradiation and mass-loss inputs would be too uncertain to propagate

## Inputs

- Planet posterior parameters from the joint Bayesian fit: Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3
- Host-star properties required for the irradiation calculation (stellar mass, radius, and flux/irradiation-relevant parameters)
- Two independent photoevaporation model implementations runnable in Python

## Outputs

- Present-day atmospheric mass-loss rate from model 1 with propagated uncertainty: 0.013 ± 0.008 M⊕ Gyr−1
- Present-day atmospheric mass-loss rate from model 2 with propagated uncertainty: 0.17 ± 0.12 M⊕ Gyr−1
- A model-comparison statement quantifying the dependence of the mass-loss prediction on the choice of photoevaporation model

## How to apply

Load the inferred planet parameters from the joint Bayesian fit of 38 HARPS radial velocities, three TESS sectors, and two PEST/TRAPPIST-South transits (Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3), together with the host-star properties needed for the irradiation calculation. Run a first photoevaporation model on this planet-and-star parameter set in Python to obtain the present-day atmospheric mass-loss rate, then run a second, alternative photoevaporation model on the identical parameter set. Contrast the two rates — for TOI-5005 b they diverge by roughly an order of magnitude (0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1) — and propagate the planetary parameter uncertainties to quote an uncertainty on each rate. The rationale is that single-model mass-loss estimates can be strongly model-dependent, so the model spread itself must be reported as a systematic.

## Related tools

- **Python** (Language used to run both photoevaporation models on the identical inferred planet and stellar parameter set and to propagate parameter uncertainties to the mass-loss rates)

## Evaluation signals

- Reproduced rates agree with the reported 0.013 ± 0.008 M⊕/Gyr and 0.17 ± 0.12 M⊕/Gyr within their quoted uncertainties
- Both photoevaporation models were run on the identical planet and stellar parameter set (same Rp, Mp, density, and host-star inputs)
- Uncertainties on each rate are derived by propagating the planetary parameter uncertainties (Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕)
- The two rates differ by roughly an order of magnitude, consistent with the reported strong model-choice dependence

## Limitations

- The mass-loss prediction depends strongly on the choice of photoevaporation model; a single-model estimate is not robust for TOI-5005 b
- The uncertainty on the higher rate (0.17 ± 0.12 M⊕ Gyr−1) is large relative to its central value, limiting discriminative power between evolution scenarios
- Results inherit any systematics in the joint Bayesian fit of HARPS radial velocities, TESS photometry, and ground-based transits

## Evidence

- [abstract] We also estimated the present-day atmospheric mass-loss rate of TOI-5005 b: "We also estimated the present-day atmospheric mass-loss rate of TOI-5005 b"
- [abstract] Contrasting photoevaporation model predictions: "found contrasting predictions depending on the choice of photoevaporation model (0.013 ± 0.008 M ⊕ Gyr −1 vs. 0.17 ± 0.12 M ⊕ Gyr −1 )"
- [other] Planet parameters from the joint Bayesian fit feed the irradiation and mass-loss calculation: [evidence span withheld — anchor status: not_found]
