---
name: core-mantle-thickness-constraint-analysis
description: Use when you have an ExoMDN-trained interior inference setup (mass, radius,
  equilibrium temperature) plus a candidate additional observable like fluid Love
  number k2 with a known measurement uncertainty, and you want to quantify the resulting
  constraint on layer thicknesses.
when_to_use_negative:
- No additional observable beyond mass, radius, and equilibrium temperature is available
  — plain ExoMDN inference will not reduce the interior-structure degeneracy
- The target planet is above ~25 Earth masses, outside the range of the >5.6 million
  synthetic training planets below 25 Earth masses
- A dedicated interior model forward calculation is required per sample rather than
  a sub-second statistical inference
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ExoMDN
  role: Provides the trained mixture density network models and fast interior inference
    (<1 s per prediction) whose posteriors are aggregated to quantify k2-driven constraints
  repo: https://github.com/philippbaumeister/ExoMDN
- name: TensorFlow
  role: Backend (tensorflow=2.11) on which the ExoMDN MDN models run for repeated
    predictions
  repo: https://github.com/philippbaumeister/ExoMDN
- name: tensorflow-probability
  role: Pinned dependency (tensorflow-probability=0.15) of the ExoMDN environment
    used for the probabilistic MDN outputs
- name: keras-mdn-layer
  role: MDN layer for Keras used by ExoMDN's model architecture
  repo: https://github.com/cpmpercussion/keras-mdn-layer
- name: NumPy
  role: Used for drawing repeated observable sets within the k2 uncertainty ranges
    and aggregating posteriors
- name: Matplotlib
  role: Renders the core/mantle thickness posterior constraint plot (e.g. k2_earth_analog_thickness_posterior.png)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:05a8056aa60503286257d21c7da200fbc59b8184fbbc8014591fe441dea7a2a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/philippbaumeister/ExoMDN
  tools:
  - ExoMDN
  - Matplotlib
  - NumPy
  - Python
  - TensorFlow
  - keras-mdn-layer
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# core-mantle-thickness-constraint-analysis

## Summary

Quantifies how well an additional observable—the fluid Love number k2 measured with a stated accuracy—reduces the degeneracy of exoplanet interior structures inferred with ExoMDN, e.g. constraining core and mantle thickness of an Earth analog to ~13% of true values for a 10% k2 measurement.

## When to use

Use when you have an ExoMDN-trained interior inference setup (mass, radius, equilibrium temperature) plus a candidate additional observable like fluid Love number k2 with a known measurement uncertainty, and you want to quantify the resulting constraint on layer thicknesses.

## When NOT to use

- No additional observable beyond mass, radius, and equilibrium temperature is available — plain ExoMDN inference will not reduce the interior-structure degeneracy
- The target planet is above ~25 Earth masses, outside the range of the >5.6 million synthetic training planets below 25 Earth masses
- A dedicated interior model forward calculation is required per sample rather than a sub-second statistical inference

## Inputs

- Trained ExoMDN mixture density network models (loaded per introduction.ipynb)
- Planet observable vector: mass, radius, equilibrium temperature (e.g. Earth analog values)
- Synthetic fluid Love number k2 treated as an additional observable, with a stated measurement uncertainty (e.g. 10%)

## Outputs

- Aggregated posterior distributions of core and mantle thickness across repeated predictions
- Posterior spread of core/mantle thickness expressed as a fraction of true values (~13% for 10% k2 accuracy)
- Constraint plot, e.g. k2_earth_analog_thickness_posterior.png rendered with Matplotlib

## How to apply

Load the trained ExoMDN mixture density network models following introduction.ipynb, then define the target planet's observable vector (e.g. an Earth analog's mass, radius, equilibrium temperature) and append a synthetic fluid Love number k2 as an additional observable. Account for the k2 measurement uncertainty (e.g. 10%) by repeatedly drawing observable sets from within the uncertainty ranges and running a fast MDN prediction for each draw, mirroring ExoMDN's approach of accounting for observational uncertainties through repeated predictions from within the uncertainties. Invert the log-ratio-transformed model outputs to obtain posterior distributions of layer mass fractions and thicknesses, then aggregate the core and mantle thickness posteriors across all repeated predictions. Finally, express the posterior spread of core and mantle thickness as a fraction of the true values (expect ~13% for a 10%-accurate k2 on an Earth analog) and render the constraint plot (e.g. k2_earth_analog_thickness_posterior.png) with Matplotlib.

## Related tools

- **ExoMDN** (Provides the trained mixture density network models and fast interior inference (<1 s per prediction) whose posteriors are aggregated to quantify k2-driven constraints) — https://github.com/philippbaumeister/ExoMDN
- **TensorFlow** (Backend (tensorflow=2.11) on which the ExoMDN MDN models run for repeated predictions) — https://github.com/philippbaumeister/ExoMDN
- **tensorflow-probability** (Pinned dependency (tensorflow-probability=0.15) of the ExoMDN environment used for the probabilistic MDN outputs)
- **keras-mdn-layer** (MDN layer for Keras used by ExoMDN's model architecture) — https://github.com/cpmpercussion/keras-mdn-layer
- **NumPy** (Used for drawing repeated observable sets within the k2 uncertainty ranges and aggregating posteriors)
- **Matplotlib** (Renders the core/mantle thickness posterior constraint plot (e.g. k2_earth_analog_thickness_posterior.png))

## Examples

```
conda env create -f environment.yml; conda activate exomdn; pip install -e .; jupyter notebook introduction.ipynb
```

## Evaluation signals

- Aggregated core and mantle thickness posteriors show spread of approximately 13% of the true values when k2 is measured with 10% accuracy on an Earth analog
- Posteriors are obtained after inverting the log-ratio-transformed MDN outputs back to layer mass fractions and thicknesses
- Each repeated prediction uses an observable set drawn from within the stated k2 uncertainty range (10% here)
- A full interior prediction completes in under a second on a standard CPU (e.g. Intel i5), so repeated-draw aggregation remains fast
- Final constraint plot matches the reported k2_earth_analog_thickness_posterior.png result

## Limitations

- ExoMDN is trained on synthetic planets below 25 Earth masses with iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere; planets outside this compositional or mass domain are unsupported
- Due to incompatibility with higher TensorFlow versions, ExoMDN does not work well with Python versions above 3.11
- The k2 constraint result (~13%) applies to the specific Earth-analog case and 10% accuracy; other planets or uncertainty levels require re-running the repeated-draw experiment
- Results depend on synthetic k2 values; real k2 measurements with different error characteristics may yield different constraints

## Evidence

- [abstract] Measuring k2 with an accuracy of 10% can constrain the thickness of core and mantle of an Earth analog to approximately 13% of the true values.: "measuringk2with an accuracy of 10% can constrain the thickness of core and mantle of an Earth analog to ≈13% of the true values."
- [abstract] Observational uncertainties handled by repeated predictions.: "Observational uncertainties can be easily accounted for through repeated predictions from within the uncertainties."
- [abstract] Log-ratio transformations convert interior structure data for the MDN.: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
- [methods] Repository setup via conda and editable pip install.: "Create a new conda environment named ***exomdn*** from the `environment.yml` file"
- [readme] Trained models and Python notebooks for running interior predictions.: [evidence span withheld — anchor status: not_found]
- [abstract] Sub-second full posterior inference from mass, radius, equilibrium temperature.: "Given mass, radius, and equilibrium temperature, we show that ExoMDN can deliver a full posterior distribution of mass fractions and thicknesses of each planetary layer in under a second"
