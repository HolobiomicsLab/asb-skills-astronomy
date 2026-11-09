---
name: exoplanet-parameter-uncertainty-propagation
description: Use when a derived planetary quantity (e.g. atmospheric mass-loss rate)
  must be computed from fitted planet/star parameters and reported with a propagated
  uncertainty, especially when comparing alternative physical models on identical
  inputs.
when_to_use_negative:
- The analysis goal is quantifying TESS aperture flux contamination (CROWDSAP/FLFRCSAP
  metrics) — that is the TESS-cont workflow, not uncertainty propagation of derived
  planet quantities
- Only a single photoevaporation model is available, so the model-dependence contrast
  that motivates this skill cannot be assessed
- No fitted parameter uncertainties (1σ intervals or posterior samples) exist for
  the planet/star inputs, making propagation of uncertainties meaningless
tools:
- name: Python
  role: Runtime for executing the two photoevaporation models on the fitted TOI-5005
    b parameters and for Monte Carlo propagation of the quoted uncertainties
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:7d42ae0f977f70cfdc866f4369365f1f32c5db32982b7db7babfb2095d668a11
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
yes_rate: 1.0
n_claims: 7
---

# exoplanet-parameter-uncertainty-propagation

## Summary

Estimate a derived planetary quantity — here, the present-day atmospheric mass-loss rate of TOI-5005 b — by running photoevaporation models on the jointly fitted planet and stellar parameters and propagating the quoted 1σ uncertainties (Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, ρp = 0.74 ± 0.16 g cm−3) onto the output rate.

## When to use

Use when a derived planetary quantity (e.g. atmospheric mass-loss rate) must be computed from fitted planet/star parameters and reported with a propagated uncertainty, especially when comparing alternative physical models on identical inputs.

## When NOT to use

- The analysis goal is quantifying TESS aperture flux contamination (CROWDSAP/FLFRCSAP metrics) — that is the TESS-cont workflow, not uncertainty propagation of derived planet quantities
- Only a single photoevaporation model is available, so the model-dependence contrast that motivates this skill cannot be assessed
- No fitted parameter uncertainties (1σ intervals or posterior samples) exist for the planet/star inputs, making propagation of uncertainties meaningless

## Inputs

- Posterior/fitted planet parameters from the joint Bayesian fit: Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3
- Host-star properties needed for the irradiation calculation (stellar flux/irradiation at the planet)
- Two alternative photoevaporation model implementations runnable in Python

## Outputs

- Present-day atmospheric mass-loss rate per model with propagated uncertainty (expected: 0.013 ± 0.008 and 0.17 ± 0.12 M⊕ Gyr−1)
- Contrast statement quantifying the dependence of the prediction on the choice of photoevaporation model

## How to apply

Load the inferred TOI-5005 b parameters from the joint Bayesian fit of 38 HARPS radial velocities, three sectors of TESS photometry, and two PEST/TRAPPIST-South transits (Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕, mean density 0.74 ± 0.16 g cm−3), together with the host-star properties needed for the irradiation calculation. Sample the planetary parameter uncertainties (e.g. Monte Carlo draws from the quoted 1σ intervals, preserving the Rp–Mp–density consistency) and run the first photoevaporation model in Python on each sample to obtain the present-day atmospheric mass-loss rate distribution; repeat with the second, alternative photoevaporation model on the identical parameter set. Report each rate as a central value with propagated uncertainty and contrast the two predictions — the key scientific output is the model-choice dependence (0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1), not a single consensus rate.

## Related tools

- **Python** (Runtime for executing the two photoevaporation models on the fitted TOI-5005 b parameters and for Monte Carlo propagation of the quoted uncertainties)

## Evaluation signals

- Reproduced rates agree with the reported values of 0.013 ± 0.008 M⊕/Gyr and 0.17 ± 0.12 M⊕/Gyr within their quoted uncertainties
- Both photoevaporation models are run on the identical parameter set, so differences in predicted rates are attributable to model choice rather than input differences
- Propagated 1σ intervals on each rate must bracket the published uncertainty ranges (±0.008 and ±0.12 M⊕ Gyr−1)
- Density values recomputed from sampled Rp and Mp draws remain consistent with the fitted 0.74 ± 0.16 g cm−3, confirming physically consistent sampling

## Limitations

- The prediction depends strongly on the choice of photoevaporation model — the two models yield contrasting rates, so no single validated rate exists and results must always be reported per model
- Uncertainty propagation is only as complete as the quoted 1σ parameter uncertainties; systematic model-choice uncertainty is not captured by propagating planetary parameters alone
- The skill is demonstrated for one target (TOI-5005 b, a super-Neptune with Zplanet = 0.76); transfer to other planets requires re-deriving irradiation and planetary inputs

## Evidence

- [abstract] Contrasting photoevaporation model predictions: "found contrasting predictions depending on the choice of photoevaporation model (0.013 ± 0.008 M ⊕ Gyr −1 vs. 0.17 ± 0.12 M ⊕ Gyr −1 )"
- [abstract] Fitted planet parameters used as inputs: [evidence span withheld — anchor status: not_found]
- [abstract] Mean density constraint for sampling consistency: "corresponds to a mean density of ρ p = 0.74 ± 0.16 g cm −3"
- [abstract] Mass-loss estimation is part of the paper workflow: "We also estimated the present-day atmospheric mass-loss rate of TOI-5005 b"
