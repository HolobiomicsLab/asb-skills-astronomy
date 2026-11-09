---
name: training-loss-convergence-monitoring
description: Use when training an MDN on compositional data (e.g., iron core, silicate
  mantle, water/high-pressure ice, H/He mass fractions) and you must verify the mixture
  negative log-likelihood loss decreases over epochs before trusting or saving the
  trained model.
when_to_use_negative:
- When pre-trained ExoMDN models suffice — the repository already provides trained
  models and Python notebooks (introduction.ipynb) for running interior predictions
  without retraining
- When the target planets exceed the training domain (25 Earth masses) or their interior
  structure requires layers outside the iron core / silicate mantle / water-high-pressure
  ice / H/He atmosphere model
- When the task is plain regression with a single-valued output — an MDN and its mixture
  loss are unnecessary overhead
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: keras-mdn-layer
  role: Provides the Keras MDN layer and get_mixture_loss_func used to define the
    negative log-likelihood loss whose per-epoch values are monitored
  repo: https://github.com/cpmpercussion/keras-mdn-layer
- name: ExoMDN
  role: Repository supplying the training routines (more_examples/model_training_demo.ipynb),
    example training data, and the exomdn package installed via pip install -e .
  repo: https://github.com/philippbaumeister/ExoMDN
- name: tensorflow 2.11
  role: Backend used to build and fit the MDN and record the training history; pinned
    by the ExoMDN environment.yml
- name: tensorflow-probability 0.15
  role: Dependency supporting the probabilistic mixture output and loss computation
    during training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:f8047398e81b1711a23aa028aed91c1f8387087315a1d847c052f6032665b72b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cpmpercussion/keras-mdn-layer
  tools:
  - ExoMDN
  - NumPy
  - Python
  - TensorFlow
  - keras-mdn-layer
  - scikit-learn
  - tensorflow 2.11
  - tensorflow-probability 0.15
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# training-loss-convergence-monitoring

## Summary

Monitors the per-epoch negative log-likelihood training loss of the ExoMDN mixture density network (Keras MDN layer, TensorFlow 2.11) while it learns planetary layer mass fractions and thicknesses from more than 5.6 million synthetic planets, verifying convergence before saving the model artifact.

## When to use

Use when training an MDN on compositional data (e.g., iron core, silicate mantle, water/high-pressure ice, H/He mass fractions) and you must verify the mixture negative log-likelihood loss decreases over epochs before trusting or saving the trained model.

## When NOT to use

- When pre-trained ExoMDN models suffice — the repository already provides trained models and Python notebooks (introduction.ipynb) for running interior predictions without retraining
- When the target planets exceed the training domain (25 Earth masses) or their interior structure requires layers outside the iron core / silicate mantle / water-high-pressure ice / H/He atmosphere model
- When the task is plain regression with a single-valued output — an MDN and its mixture loss are unnecessary overhead

## Inputs

- Training data from the data directory of the ExoMDN repository: synthetic planet interior structures (more than 5.6 million planets below 25 Earth masses) with iron core, silicate mantle, water/high-pressure ice, and H/He atmosphere mass fractions plus layer thicknesses
- Log-ratio-transformed compositional labels paired with inputs of mass, radius, and equilibrium temperature

## Outputs

- Per-epoch training loss record/plot (e.g., training_loss.png) showing decreasing negative log-likelihood
- Trained MDN model artifact (e.g., trained_model.h5) that, given mass, radius, and equilibrium temperature, outputs a full posterior distribution of mass fractions and thicknesses of each planetary layer

## How to apply

After applying log-ratio transformations to the compositional interior-structure labels, build the network with the Keras MDN layer (github.com/cpmpercussion/keras-mdn-layer) so its final layer is mdn.MDN(OUTPUT_DIMS, N_MIXES) and compile it with mdn.get_mixture_loss_func(OUTPUT_DIMS, N_MIXES), which implements the negative log-likelihood of a mixture of multivariate normal distributions with diagonal covariance (Bishop PRML Eq. 2.43). Fit with model.fit on the transformed synthetic-planet training set, capturing the returned history object so per-epoch loss values are recorded (e.g., plotted as training_loss.png). Monitor that the loss decreases steadily across epochs; a flat or diverging loss indicates the log-ratio preprocessing, number of mixture components, or learning rate needs revision. Once the loss has converged, save the model artifact (e.g., trained_model.h5) and validate that reloading it reproduces a full posterior over layer mass fractions and thicknesses. Follow the reference routine in more_examples/model_training_demo.ipynb of the ExoMDN repository, using tensorflow=2.11 and tensorflow-probability=0.15 for reproducibility.

## Related tools

- **keras-mdn-layer** (Provides the Keras MDN layer and get_mixture_loss_func used to define the negative log-likelihood loss whose per-epoch values are monitored) — https://github.com/cpmpercussion/keras-mdn-layer
- **ExoMDN** (Repository supplying the training routines (more_examples/model_training_demo.ipynb), example training data, and the exomdn package installed via pip install -e .) — https://github.com/philippbaumeister/ExoMDN
- **tensorflow 2.11** (Backend used to build and fit the MDN and record the training history; pinned by the ExoMDN environment.yml)
- **tensorflow-probability 0.15** (Dependency supporting the probabilistic mixture output and loss computation during training)

## Examples

```
model.compile(loss=mdn.get_mixture_loss_func(OUTPUT_DIMS, N_MIXES), optimizer=keras.optimizers.Adam()); history = model.fit(x=x_train, y=y_train)
```

## Evaluation signals

- The recorded per-epoch training loss (training_loss.png) decreases over epochs rather than plateauing high or diverging
- The loss being minimized is the mixture negative log-likelihood generated by mdn.get_mixture_loss_func(OUTPUT_DIMS, N_MIXES) with output dimensions and mixture count matching the layer-quantity labels
- The saved model artifact (trained_model.h5) reloads successfully with the custom MDN layer and loss function
- Given mass, radius, and equilibrium temperature, the reloaded model delivers a full posterior distribution of the mass fractions and thicknesses of each planetary layer
- The training environment matches the pinned versions: tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1

## Limitations

- ExoMDN does not work well with Python versions above 3.11 due to incompatibility with higher tensorflow versions
- The model is only trained on synthetic planets below 25 Earth masses with a four-layer composition (iron core, silicate mantle, water/high-pressure ice, H/He atmosphere); structures outside this setup are out of scope
- The keras-mdn-layer loss assumes a mixture of multivariate normal distributions with a diagonal covariance matrix, which constrains the posterior shapes it can represent

## Evidence

- [methods] MDN loss definition: [evidence span withheld — anchor status: external_doc]
- [methods] MDN layer usage and fitting: [evidence span withheld — anchor status: external_doc]
- [readme] Training routines location: "We also make available the training routines in `more_examples/model_training_demo.ipynb`."
- [abstract] Log-ratio preprocessing: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
- [abstract] Training data scale: "The model is trained on a large dataset of more than 5.6 million synthetic planets below 25 Earth masses"
