---
name: photoevaporation-model-intercomparison
description: Use when a confirmed planet's radius, mass, density, and host-star irradiation
  properties are available and you need a present-day atmospheric mass-loss estimate
  whose robustness against photoevaporation model choice must be tested.
when_to_use_negative:
- The goal is only a single mass-loss rate for a population study without assessing
  model-choice dependence — running one model suffices and the intercomparison adds
  no value
- Planet parameters (radius, mass, host irradiation) are not yet constrained from
  a joint fit, so neither model can be meaningfully evaluated
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Python
  role: Language in which both photoevaporation models are executed on the identical
    planet/star parameter set to compute the mass-loss rates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:1faa059d0a1bc94b79da1ad8b7e9ce01a8d5ca087e909da319542b611fa6e907
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# photoevaporation-model-intercomparison

## Summary

Estimates the present-day atmospheric mass-loss rate of a transiting planet by running two independent photoevaporation models on the same inferred planet/star parameter set and contrasting their predictions, exposing the strong dependence of the rate on model choice.

## When to use

Use when a confirmed planet's radius, mass, density, and host-star irradiation properties are available and you need a present-day atmospheric mass-loss estimate whose robustness against photoevaporation model choice must be tested.

## When NOT to use

- The goal is only a single mass-loss rate for a population study without assessing model-choice dependence — running one model suffices and the intercomparison adds no value
- Planet parameters (radius, mass, host irradiation) are not yet constrained from a joint fit, so neither model can be meaningfully evaluated

## Inputs

- Jointly inferred planet parameters: Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3 (from Bayesian fit of HARPS RVs, TESS photometry, and ground-based transits)
- Host-star properties needed for the irradiation calculation
- Two photoevaporation models runnable in Python on the identical parameter set

## Outputs

- Present-day atmospheric mass-loss rate from model 1 with propagated uncertainty (e.g. 0.013 ± 0.008 M⊕ Gyr−1)
- Present-day atmospheric mass-loss rate from model 2 with propagated uncertainty (e.g. 0.17 ± 0.12 M⊕ Gyr−1)
- Explicit statement of the model-choice dependence between the two predictions

## How to apply

Load the jointly inferred planetary parameters (for TOI-5005 b: Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3) together with the host-star properties needed for the irradiation calculation. Run the first photoevaporation model in Python on this parameter set to compute the present-day atmospheric mass-loss rate, then run a second, alternative photoevaporation model on the identical parameter set to obtain a contrasting prediction. Contrast the two rates and propagate the planetary parameter uncertainties (e.g. via the quoted ± values) to assign an uncertainty to each rate. Decision point: if the two rates disagree beyond their uncertainties, report the model-choice dependence rather than a single rate — as the paper does with 0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1.

## Related tools

- **Python** (Language in which both photoevaporation models are executed on the identical planet/star parameter set to compute the mass-loss rates)

## Evaluation signals

- Reproduced rates must agree with the reported values of 0.013 ± 0.008 M⊕/Gyr and 0.17 ± 0.12 M⊕/Gyr within their quoted uncertainties
- Both models must be run on the identical parameter set (same Rp, Mp, density, and stellar inputs) so the only difference is the model choice
- Uncertainties on each rate must be derived by propagating the planetary parameter uncertainties, not assumed
- The contrast between the two rates should exceed their individual uncertainties for the model-dependence claim to hold

## Limitations

- The reported contrast (0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1) demonstrates that predictions are strongly model-dependent, so a single-model rate should not be over-interpreted
- The mass-loss estimate inherits the precision of the joint Bayesian fit (e.g. Mp = 32.7 ± 5.9 M⊕), limiting the accuracy of each propagated uncertainty

## Evidence

- [abstract] found contrasting predictions depending on the choice of photoevaporation model (0.013 ± 0.008 M ⊕ Gyr −1 vs. 0.17 ± 0.12 M ⊕ Gyr −1 ): "found contrasting predictions depending on the choice of photoevaporation model (0.013 ± 0.008 M ⊕ Gyr −1 vs. 0.17 ± 0.12 M ⊕ Gyr −1 )"
- [abstract] We also estimated the present-day atmospheric mass-loss rate of TOI-5005 b: "We also estimated the present-day atmospheric mass-loss rate of TOI-5005 b"
- [abstract] We confirm TOI-5005 b to be a transiting super-Neptune with a radius of R p = 6.25 ± 0.24 R ⊕ ... and a mass of M p = 32.7 ± 5.9 M ⊕: [evidence span withheld — anchor status: not_found]
