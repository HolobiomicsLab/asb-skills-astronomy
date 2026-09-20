---
name: planetary-layer-mass-fraction-prediction
description: Use when you need interior structure posteriors (layer mass fractions
  and thicknesses) for a low-mass exoplanet from only mass, radius, and equilibrium
  temperature, and require inference in under a second on CPU without a dedicated
  interior model.
when_to_use_negative:
- 'The target planet is not in the low-mass regime: the model is trained only on synthetic
  planets below 25 Earth masses'
- You only have radii or atmosphere data but no mass and equilibrium temperature —
  ExoMDN requires mass, radius, and equilibrium temperature as inputs
- You need a physical forward interior model with self-consistent thermodynamics rather
  than a fast machine-learning posterior approximation
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3810
tools:
- name: ExoMDN
  role: Pre-trained mixture density network model that maps mass, radius, and equilibrium
    temperature to a posterior over layer mass fractions and thicknesses
  repo: https://github.com/philippbaumeister/ExoMDN
- name: tensorflow
  role: Backend (version 2.11) used to run the MDN forward pass on CPU for posterior
    prediction
- name: Python
  role: Language for the notebooks (introduction.ipynb) and package utilities used
    to load models and run interior predictions
- name: keras-mdn-layer
  role: MDN layer for Keras used by ExoMDN's network architecture
  repo: https://github.com/cpmpercussion/keras-mdn-layer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:9d67343feff264ec837f9d35c6ef7645f8cf6abda3d2b30e4bb32c266d3589e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/philippbaumeister/ExoMDN
  tools:
  - ExoMDN
  - NumPy
  - Python
  - TensorFlow
  - keras-mdn-layer
  - tensorflow
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3810
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# planetary-layer-mass-fraction-prediction

## Summary

Predict a full posterior over interior layer mass fractions and layer thicknesses (iron core, silicate mantle, water/high-pressure ice, H/He atmosphere) for a low-mass (<25 Earth mass) exoplanet using the pre-trained ExoMDN mixture density network, given mass, radius, and equilibrium temperature as inputs.

## When to use

Use when you need interior structure posteriors (layer mass fractions and thicknesses) for a low-mass exoplanet from only mass, radius, and equilibrium temperature, and require inference in under a second on CPU without a dedicated interior model.

## When NOT to use

- The target planet is not in the low-mass regime: the model is trained only on synthetic planets below 25 Earth masses
- You only have radii or atmosphere data but no mass and equilibrium temperature — ExoMDN requires mass, radius, and equilibrium temperature as inputs
- You need a physical forward interior model with self-consistent thermodynamics rather than a fast machine-learning posterior approximation

## Inputs

- Planetary mass, radius, and equilibrium temperature (observable vector) for one low-mass exoplanet
- Pre-trained ExoMDN mixture density network models from Zenodo DOI 10.5281/zenodo.7670706

## Outputs

- Posterior distribution (mixture-density samples/summary statistics) over mass fractions of the iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere
- Posterior distribution over layer thicknesses for the same four layers
- Wall-clock runtime measurement of the prediction

## How to apply

Install the exomdn package from the repository (Python 3.7+, tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1) via conda env create -f environment.yml followed by pip install -e ., or pip install -r requirements.txt; note ExoMDN does not work well with Python above 3.11. Load the pre-trained MDN models from the Zenodo release (DOI 10.5281/zenodo.7670706) using the utilities demonstrated in introduction.ipynb. Assemble the input observable vector of mass, radius, and equilibrium temperature for one low-mass (<25 Earth mass) planet, run the MDN forward pass with TensorFlow 2.11 on CPU, and back-transform the log-ratio-transformed outputs to physical mass fractions and thicknesses for the four modeled layers. Because the MDN was trained on more than 5.6 million synthetic planets, account for observational uncertainties by repeating predictions with inputs sampled from within the measurement uncertainties.

## Related tools

- **ExoMDN** (Pre-trained mixture density network model that maps mass, radius, and equilibrium temperature to a posterior over layer mass fractions and thicknesses) — https://github.com/philippbaumeister/ExoMDN
- **tensorflow** (Backend (version 2.11) used to run the MDN forward pass on CPU for posterior prediction)
- **Python** (Language for the notebooks (introduction.ipynb) and package utilities used to load models and run interior predictions)
- **keras-mdn-layer** (MDN layer for Keras used by ExoMDN's network architecture) — https://github.com/cpmpercussion/keras-mdn-layer

## Examples

```
conda env create -f environment.yml; conda activate exomdn; pip install -e .; jupyter notebook introduction.ipynb
```

## Evaluation signals

- Wall-clock prediction runtime is below one second on a standard Intel i5 CPU
- Posterior covers all four modeled layers: iron core, silicate mantle, water/high-pressure ice, and H/He atmosphere
- Back-transformed layer mass fractions are physically valid (log-ratio outputs correctly transformed back to fractions summing to a physical composition)
- Predicted posterior is consistent with the input observables (mass, radius, equilibrium temperature) within observational uncertainties
- Environment matches pinned versions: tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1

## Limitations

- Model is valid only for low-mass planets (<25 Earth masses); it was trained on synthetic planets below 25 Earth masses
- Does not work well with Python versions above 3.11 due to tensorflow incompatibilities
- Observational uncertainties are handled only by repeated predictions from within the uncertainties, not by a full joint uncertainty propagation

## Evidence

- [abstract] sub-second CPU inference claim: "Given mass, radius, and equilibrium temperature, we show that ExoMDN can deliver a full posterior distribution of mass fractions and thicknesses of each planetary layer in under a second on a"
- [abstract] log-ratio transformation step: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
- [abstract] training data domain: "The model is trained on a large dataset of more than 5.6 million synthetic planets below 25 Earth masses consisting of an iron core, a silicate mantle, a water and high-pressure ice layer, and a H/He"
- [abstract] uncertainty handling: "Observational uncertainties can be easily accounted for through repeated predictions from within the uncertainties."
- [methods] installation procedure: "Create a new conda environment named ***exomdn*** from the `environment.yml` file"
- [readme] getting-started notebook: "To get started check out `introduction.ipynb`. More in-depth examples can be found in the *more_examples* directory and"
- [readme] Python version constraint: "Currently, due to incompatibility with higher tensorflow versions, ExoMDN does not work well with Python versions above 3.11"
