---
name: log-ratio-output-backtransformation
description: Use when an ExoMDN mixture density network has produced log-ratio-transformed
  outputs and they must be converted back to physical layer mass fractions and thicknesses
  of a low-mass exoplanet interior.
when_to_use_negative:
- The planet is above 25 Earth masses — ExoMDN was trained only on synthetic planets
  below 25 Earth masses
- No mass, radius, and equilibrium temperature observables are available for the target
  planet
- Outputs are already physical layer mass fractions/thicknesses and no log-ratio back-transformation
  is needed
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ExoMDN
  role: Pre-trained mixture density network whose log-ratio outputs are back-transformed
    to physical interior structure
  repo: github.com/philippbaumeister/ExoMDN
- name: tensorflow 2.11
  role: Runs the MDN forward pass on CPU to obtain the mixture-density posterior
- name: numpy
  role: Array manipulation for assembling observables and transforming model outputs
- name: keras-mdn-layer
  role: MDN layer for Keras used by ExoMDN to output mixture-density parameters
  repo: https://github.com/cpmpercussion/keras-mdn-layer
- name: conda
  role: Creates and activates the exomdn environment (environment.yml) used to run
    the prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:94d4400f4e5d2a8f97f9ca72901764cbebc4edf08787eb55c12ec7c81b9efcd7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoMDN
  - NumPy
  - Python
  - TensorFlow
  - conda
  - keras-mdn-layer
  - numpy
  - tensorflow 2.11
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# log-ratio-output-backtransformation

## Summary

Back-transforms the log-ratio-transformed mixture-density outputs of ExoMDN into physical layer mass fractions and thicknesses for the four modeled layers (iron core, silicate mantle, water/high-pressure ice, H/He atmosphere), enabling a full interior posterior from mass, radius, and equilibrium temperature.

## When to use

Use when an ExoMDN mixture density network has produced log-ratio-transformed outputs and they must be converted back to physical layer mass fractions and thicknesses of a low-mass exoplanet interior.

## When NOT to use

- The planet is above 25 Earth masses — ExoMDN was trained only on synthetic planets below 25 Earth masses
- No mass, radius, and equilibrium temperature observables are available for the target planet
- Outputs are already physical layer mass fractions/thicknesses and no log-ratio back-transformation is needed

## Inputs

- Trained ExoMDN mixture density network models (Zenodo DOI 10.5281/zenodo.7670706)
- Input observable vector: planetary mass, radius, and equilibrium temperature for a low-mass (<25 Earth mass) exoplanet
- Raw MDN outputs: mixture-density parameters over log-ratio-transformed layer mass fractions and thicknesses

## Outputs

- Full posterior distribution of physical layer mass fractions (iron core, silicate mantle, water/high-pressure ice, H/He atmosphere)
- Posterior distribution of layer thicknesses for each of the four layers
- Recorded wall-clock runtime of the CPU prediction (e.g. interior_posterior_prediction.csv with posterior summaries and runtime)

## How to apply

Load the pre-trained ExoMDN models from the Zenodo release (DOI 10.5281/zenodo.7670706) using the model-loading utilities shown in introduction.ipynb of the exomdn package (installed via conda activate exomdn and pip install -e .). Assemble the input observable vector for one low-mass (<25 Earth mass) planet from its mass, radius, and equilibrium temperature. Run the MDN forward pass with tensorflow 2.11 on CPU; the network outputs the posterior over log-ratio-transformed layer mass fractions and thicknesses, because log-ratio transformations are employed to put the compositional data into a form the MDN can handle. Back-transform the outputs to physical fractions and thicknesses for the iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere, then verify the posterior covers all four layers and that a prediction completes in under a second on a standard Intel i5 CPU.

## Related tools

- **ExoMDN** (Pre-trained mixture density network whose log-ratio outputs are back-transformed to physical interior structure) — github.com/philippbaumeister/ExoMDN
- **tensorflow 2.11** (Runs the MDN forward pass on CPU to obtain the mixture-density posterior)
- **numpy** (Array manipulation for assembling observables and transforming model outputs)
- **keras-mdn-layer** (MDN layer for Keras used by ExoMDN to output mixture-density parameters) — https://github.com/cpmpercussion/keras-mdn-layer
- **conda** (Creates and activates the exomdn environment (environment.yml) used to run the prediction)

## Examples

```
conda activate exomdn && pip install -e . && jupyter notebook introduction.ipynb
```

## Evaluation signals

- Wall-clock prediction runtime is below one second on a standard Intel i5 CPU
- Back-transformed posterior covers all four modeled layers: iron core, silicate mantle, water/high-pressure ice, and H/He atmosphere
- Layer mass fractions are valid compositions (summing to 1) after the inverse log-ratio transformation
- Thicknesses and mass fractions are physically plausible (non-negative, within the ranges of the 5.6-million-planet training set)
- Repeated predictions from within observational uncertainties reproduce the expected posterior spread

## Limitations

- ExoMDN was trained only on planets below 25 Earth masses, so predictions for higher-mass planets are outside the training domain
- ExoMDN does not work well with Python versions above 3.11 due to incompatibility with higher tensorflow versions; requires tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1
- Observational uncertainties are not propagated directly; they must be accounted for through repeated predictions from within the uncertainties

## Evidence

- [abstract] Log-ratio transformations are used to make interior structure data tractable for the MDN: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
- [abstract] Full posterior of mass fractions and thicknesses delivered in under a second: "Given mass, radius, and equilibrium temperature, we show that ExoMDN can deliver a full posterior distribution of mass fractions and thicknesses of each planetary layer in under a second on a"
- [readme] README states the capability and speed of interior inference: "Given mass, radius, and equilibrium temperature, ExoMDN is capable of providing a full inference of the interior structure of low-mass exoplanets in under a second"
- [methods] Starting point for model loading and running predictions: "To get started check out `introduction.ipynb`."
