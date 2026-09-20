---
name: monte-carlo-input-sampling
description: Use when an exoplanet's observed mass, radius, and equilibrium temperature
  carry stated measurement uncertainties and the goal is a propagated, uncertainty-aware
  posterior of interior structure (iron core, silicate mantle, water/ice, H/He) rather
  than a single-point prediction.
when_to_use_negative:
- Target planet exceeds the training domain of ExoMDN (trained on synthetic planets
  below 25 Earth masses) — sampled inputs would fall outside model validity
- Observables carry no stated uncertainties, so a single ExoMDN point prediction suffices
  and repeated input sampling adds no information
- Additional observables such as the fluid Love number k2 are available; the paper
  shows k2 reduces interior degeneracy more directly than input resampling
edam_operation: http://edamontology.org/operation_2426
tools:
- name: ExoMDN
  role: Mixture density network model run once per sampled (mass, radius, equilibrium
    temperature) triple to predict log-ratio-transformed layer mass fractions and
    thicknesses
  repo: github.com/philippbaumeister/ExoMDN
- name: keras-mdn-layer
  role: Keras MDN layer underlying ExoMDN; provides the mixture parameterization and
    sample_from_output used to draw interior-structure samples from each prediction
  repo: github.com/cpmpercussion/keras-mdn-layer
- name: tensorflow 2.11
  role: Backend on which the ExoMDN MDN is executed for each repeated prediction
- name: numpy
  role: Random sampling of the observable inputs from distributions defined by the
    stated uncertainties; aggregation of repeated predictions
- name: pandas
  role: Persisting the per-sample prediction records into posterior_samples.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:a7cbf10d7ed0435a50ab8f67d525427ebee0ef0ad98e1c7832f590ec8ac0d3cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoMDN
  - Python
  - TensorFlow
  - keras-mdn-layer
  - numpy
  - pandas
  - tensorflow 2.11
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# monte-carlo-input-sampling

## Summary

Propagates observational uncertainties in exoplanet observables (mass, radius, equilibrium temperature) into an interior-structure posterior by drawing repeated random samples of the inputs and running ExoMDN's mixture density network on each sampled triple. This replaces a single-point prediction with an uncertainty-aware posterior of layer mass fractions and thicknesses.

## When to use

Use when an exoplanet's observed mass, radius, and equilibrium temperature carry stated measurement uncertainties and the goal is a propagated, uncertainty-aware posterior of interior structure (iron core, silicate mantle, water/ice, H/He) rather than a single-point prediction.

## When NOT to use

- Target planet exceeds the training domain of ExoMDN (trained on synthetic planets below 25 Earth masses) — sampled inputs would fall outside model validity
- Observables carry no stated uncertainties, so a single ExoMDN point prediction suffices and repeated input sampling adds no information
- Additional observables such as the fluid Love number k2 are available; the paper shows k2 reduces interior degeneracy more directly than input resampling

## Inputs

- Observed planet mass with stated uncertainty (e.g. <10% for the characterized sample)
- Observed planet radius with stated uncertainty (e.g. <5% for the characterized sample)
- Equilibrium temperature with stated uncertainty
- Trained ExoMDN mixture density network models (trained on >5.6 million synthetic planets below 25 Earth masses)

## Outputs

- Combined uncertainty-aware posterior distribution of mass fractions per layer (iron core, silicate mantle, water/high-pressure ice, H/He atmosphere)
- Combined posterior distribution of layer thicknesses
- posterior_samples.csv with one record per sampled prediction

## How to apply

Load the target planet's observed mass, radius, and equilibrium temperature with their stated uncertainties from the example planet parameters used in the ExoMDN introduction.ipynb setup. Draw repeated random samples of the three observables from distributions defined by those uncertainties, using the characterized exoplanet sample (mass uncertainty below 10%, radius uncertainty below 5%) as reference scales for plausible magnitudes. Run the ExoMDN mixture density network — a TensorFlow 2.11 model built on the keras-mdn-layer — on each sampled input triple to predict log-ratio-transformed mass fractions and thicknesses of the iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere; each prediction completes in under a second on a standard Intel i5 CPU. Aggregate the repeated predictions into a combined uncertainty-aware posterior distribution for each layer property, e.g. persisting one record per sampled prediction (posterior_samples.csv). Rationale: the MDN returns a full conditional distribution per point prediction, so sampling the inputs and pooling predictions propagates measurement error through the non-linear interior inference without retraining.

## Related tools

- **ExoMDN** (Mixture density network model run once per sampled (mass, radius, equilibrium temperature) triple to predict log-ratio-transformed layer mass fractions and thicknesses) — github.com/philippbaumeister/ExoMDN
- **keras-mdn-layer** (Keras MDN layer underlying ExoMDN; provides the mixture parameterization and sample_from_output used to draw interior-structure samples from each prediction) — github.com/cpmpercussion/keras-mdn-layer
- **tensorflow 2.11** (Backend on which the ExoMDN MDN is executed for each repeated prediction)
- **numpy** (Random sampling of the observable inputs from distributions defined by the stated uncertainties; aggregation of repeated predictions)
- **pandas** (Persisting the per-sample prediction records into posterior_samples.csv)

## Examples

```
jupyter notebook introduction.ipynb
```

## Evaluation signals

- posterior_samples.csv exists and contains exactly one record per drawn input sample
- Posterior spread of layer mass fractions and thicknesses is consistent with the propagated observational uncertainties (wider than a single-point MDN posterior, scaling with the mass <10% / radius <5% input ranges)
- Each ExoMDN prediction completes in under a second on a standard Intel i5 CPU, so a repeated-prediction run of N samples finishes in roughly N seconds
- Per-sample outputs invert the log-ratio transformation to physically valid mass fractions across the iron core, silicate mantle, water/high-pressure ice, and H/He layers
- Pooled posterior matches the abstract's stated behavior: 'Observational uncertainties can be easily accounted for through repeated predictions from within the uncertainties.'

## Limitations

- ExoMDN is trained on synthetic planets below 25 Earth masses; predictions outside this mass regime are unsupported
- Environment constraints: ExoMDN does not work well with Python versions above 3.11 and currently requires tensorflow=2.11 and tensorflow-probability=0.15
- Input resampling propagates observational error but does not by itself resolve interior degeneracy; the paper notes the fluid Love number k2 is needed to significantly reduce it (a 10% k2 accuracy constrains Earth-analog core/mantle thickness to ~13%)

## Evidence

- [abstract] Observational uncertainties accounted for by repeated sampling: "Observational uncertainties can be easily accounted for through repeated predictions from within the uncertainties."
- [abstract] Full posterior per prediction in under a second: [evidence span withheld — anchor status: not_found]
- [abstract] Reference uncertainty scales of the characterized sample: "We used ExoMDN to characterize the interiors of 22 confirmed exoplanets with mass and radius uncertainties below 10 and 5%, respectively"
- [abstract] Log-ratio transformation of interior structure data: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
- [readme] Entry-point notebook for running predictions: "To get started check out `introduction.ipynb`. More in-depth examples can be found in the *more_examples* directory"
- [intro] Training-domain limit of the model: "The model is trained on more than 5.6 million synthetic planet interior structures."
