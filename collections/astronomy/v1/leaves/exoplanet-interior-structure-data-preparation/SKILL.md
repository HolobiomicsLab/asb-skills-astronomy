---
name: exoplanet-interior-structure-data-preparation
description: Use when preparing compositional interior-structure labels (layer mass
  fractions constrained to sum to one) as training targets for an MDN that predicts
  planetary layer fractions and thicknesses from mass, radius, and equilibrium temperature.
when_to_use_negative:
- Targets are unconstrained real-valued quantities rather than simplex-constrained
  compositional fractions, so no log-ratio closure handling is needed
- Planets are outside the training domain (above ~25 Earth masses), since ExoMDN was
  trained on synthetic planets below 25 Earth masses
- You only need to run interior predictions with the released trained models rather
  than retrain — use introduction.ipynb instead of the data-preparation/training pipeline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3474
tools:
- name: ExoMDN
  role: Provides the training routines (more_examples/model_training_demo.ipynb) and
    example training data used for the data-preparation and MDN training pipeline
  repo: https://github.com/philippbaumeister/ExoMDN
- name: keras-mdn-layer
  role: Keras MDN layer used to build the mixture density network whose labels are
    prepared by the log-ratio transformation
  repo: https://github.com/cpmpercussion/keras-mdn-layer
- name: tensorflow
  role: Framework (version 2.11) in which the MDN is built and trained on the transformed
    data
- name: tensorflow-probability
  role: Version 0.15 dependency supporting the probabilistic mixture output and negative
    log-likelihood training
- name: numpy
  role: Used with pandas to load and transform the example training data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:1d13e376b7e525b18b18e17c15f81097db06b3e050a7aad22c33db8689d5110f
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
  - numpy
  - scikit-learn
  - tensorflow
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# exoplanet-interior-structure-data-preparation

## Summary

Prepares synthetic exoplanet interior-structure data (iron core, silicate mantle, water/high-pressure ice, and H/He atmosphere mass fractions and layer thicknesses) into a form a mixture density network can handle by applying log-ratio transformations to the compositional labels.

## When to use

Use when preparing compositional interior-structure labels (layer mass fractions constrained to sum to one) as training targets for an MDN that predicts planetary layer fractions and thicknesses from mass, radius, and equilibrium temperature.

## When NOT to use

- Targets are unconstrained real-valued quantities rather than simplex-constrained compositional fractions, so no log-ratio closure handling is needed
- Planets are outside the training domain (above ~25 Earth masses), since ExoMDN was trained on synthetic planets below 25 Earth masses
- You only need to run interior predictions with the released trained models rather than retrain — use introduction.ipynb instead of the data-preparation/training pipeline

## Inputs

- CSV/table of synthetic planet interior structures (layer mass fractions and thicknesses) from the ExoMDN data directory
- Per-planet observables: mass, radius, equilibrium temperature
- Compositional labels: iron core, silicate mantle, water/high-pressure ice, H/He atmosphere mass fractions (summing to one)

## Outputs

- Log-ratio-transformed training labels ready for MDN training
- Trained MDN model artifact (e.g. trained_model.h5)
- Per-epoch training loss plot (e.g. training_loss.png)

## How to apply

Load the example training data from the data directory of the ExoMDN repository using pandas/numpy. Apply log-ratio transformations to the compositional labels — the iron core, silicate mantle, water/high-pressure ice, and H/He atmosphere mass fractions — because fractions are simplex-constrained and log-ratio transformation converts them into a form the MDN can easily handle. Keep mass, radius, and equilibrium temperature as the network inputs, and set up the MDN in TensorFlow 2.11 with tensorflow-probability 0.15 using the Keras MDN layer from github.com/cpmpercussion/keras-mdn-layer. Train on a sample of the more than 5.6 million synthetic planets below 25 Earth masses by minimizing the negative log-likelihood of the mixture output, following the routine in more_examples/model_training_demo.ipynb, then save the trained model (e.g. trained_model.h5) and record per-epoch training loss.

## Related tools

- **ExoMDN** (Provides the training routines (more_examples/model_training_demo.ipynb) and example training data used for the data-preparation and MDN training pipeline) — https://github.com/philippbaumeister/ExoMDN
- **keras-mdn-layer** (Keras MDN layer used to build the mixture density network whose labels are prepared by the log-ratio transformation) — https://github.com/cpmpercussion/keras-mdn-layer
- **tensorflow** (Framework (version 2.11) in which the MDN is built and trained on the transformed data)
- **tensorflow-probability** (Version 0.15 dependency supporting the probabilistic mixture output and negative log-likelihood training)
- **numpy** (Used with pandas to load and transform the example training data)

## Examples

```
conda activate exomdn; pip install -e .; jupyter notebook more_examples/model_training_demo.ipynb
```

## Evaluation signals

- Transformed compositional labels no longer lie on the simplex (mass fractions originally summing to one), which is required for MDN handling
- Training loss decreases monotonically over epochs and is recorded per epoch (e.g. training_loss.png)
- The saved model (e.g. trained_model.h5) reloads successfully and, given mass, radius, and equilibrium temperature, delivers a full posterior distribution of layer mass fractions and thicknesses
- Training used a sample of the more than 5.6 million synthetic planets below 25 Earth masses, matching the paper's dataset scale
- Model runs a full interior inference in under a second on a standard Intel i5 CPU

## Limitations

- ExoMDN does not work well with Python versions above 3.11 due to incompatibility with higher tensorflow versions; pin tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1
- Model validity is limited to low-mass planets — training data cover synthetic planets below 25 Earth masses
- Observational uncertainties must be handled by repeated predictions from within the uncertainties, not by the network directly

## Evidence

- [abstract] Log-ratio transformations employed for MDN input preparation: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
- [abstract] Training dataset scale and composition: "The model is trained on a large dataset of more than 5.6 million synthetic planets below 25 Earth masses"
- [methods] Pinned dependency versions for reproducible MDN training: "Currently, due to incompatibility with higher tensorflow versions, ExoMDN does not work well with Python versions above 3.11"
- [readme] Training routines and example data location: "We also make available the training routines in `more_examples/model_training_demo.ipynb`."
- [methods] MDN layer source for the mixture network: "We are using the MDN layer for Keras by https://github.com/cpmpercussion/keras-mdn-layer"
