---
name: cpu-runtime-benchmarking
description: Use when a trained model's claimed inference speed must be validated
  on commodity CPU hardware, e.g. verifying that one ExoMDN forward pass on (mass,
  radius, equilibrium temperature) yields a full four-layer interior posterior in
  under a second.
when_to_use_negative:
- The target planet is above 25 Earth masses — the MDN was trained only on synthetic
  planets below 25 Earth masses, so runtime and posterior results would be out of
  model scope
- The goal is GPU or distributed-hardware benchmarking — the paper's sub-second claim
  and validation threshold are defined specifically for a standard Intel i5 CPU with
  TensorFlow 2.11
- A dedicated physical interior structure model is required for the benchmark (e.g.
  self-consistent thermodynamic forward modeling) rather than an ML surrogate's inference
  latency
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ExoMDN
  role: Pre-trained mixture density network models and notebooks (introduction.ipynb)
    used to run the timed interior prediction
  repo: github.com/philippbaumeister/ExoMDN
- name: tensorflow
  role: Runtime engine (pinned to tensorflow=2.11) executing the MDN forward pass
    on CPU during the wall-clock benchmark
- name: keras-mdn-layer
  role: MDN layer for Keras underlying the ExoMDN model architecture being benchmarked
  repo: github.com/cpmpercussion/keras-mdn-layer
- name: conda
  role: Reproducible environment creation (conda env create -f environment.yml) to
    ensure the pinned dependency versions the runtime claim depends on
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:a7a5c578d020d0b1dc3774d197347a0479de483ded50a61c6f3b5ab3e0ae3c7d
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
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cpu-runtime-benchmarking

## Summary

Benchmarks the wall-clock CPU runtime of a trained machine-learning inference pipeline, here ExoMDN's mixture density network, against a concrete latency target (sub-second full interior posterior on a standard Intel i5).

## When to use

Use when a trained model's claimed inference speed must be validated on commodity CPU hardware, e.g. verifying that one ExoMDN forward pass on (mass, radius, equilibrium temperature) yields a full four-layer interior posterior in under a second.

## When NOT to use

- The target planet is above 25 Earth masses — the MDN was trained only on synthetic planets below 25 Earth masses, so runtime and posterior results would be out of model scope
- The goal is GPU or distributed-hardware benchmarking — the paper's sub-second claim and validation threshold are defined specifically for a standard Intel i5 CPU with TensorFlow 2.11
- A dedicated physical interior structure model is required for the benchmark (e.g. self-consistent thermodynamic forward modeling) rather than an ML surrogate's inference latency

## Inputs

- Exoplanet observable vector: mass, radius, and equilibrium temperature for one low-mass (<25 Earth mass) planet
- Pre-trained ExoMDN mixture density network models from Zenodo DOI 10.5281/zenodo.7670706

## Outputs

- Full posterior distribution of mass fractions and thicknesses for the iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere
- Measured wall-clock runtime of a single MDN prediction on a standard Intel i5 CPU
- Combined posterior and runtime record saved as interior_posterior_prediction.csv

## How to apply

Set up the pinned environment first — conda env create -f environment.yml from the ExoMDN repo, conda activate exomdn, pip install -e . — because the runtime claim only holds for tensorflow=2.11 with tensorflow-probability=0.15 and scikit-learn=1.1.1 on CPU. Load the pre-trained MDN models from the Zenodo release (DOI 10.5281/zenodo.7670706) using the utilities shown in introduction.ipynb, assemble the observable vector for one low-mass (<25 Earth mass) planet as mass, radius, and equilibrium temperature, and run the MDN forward pass in TensorFlow 2.11 on CPU. Back-transform the log-ratio-transformed outputs to physical mass fractions and thicknesses for the iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere. Measure the wall-clock time of the prediction (not model loading) on a standard Intel i5, checking against the paper's threshold of under one second. Record the posterior samples/summary statistics and the measured runtime to interior_posterior_prediction.csv.

## Related tools

- **ExoMDN** (Pre-trained mixture density network models and notebooks (introduction.ipynb) used to run the timed interior prediction) — github.com/philippbaumeister/ExoMDN
- **tensorflow** (Runtime engine (pinned to tensorflow=2.11) executing the MDN forward pass on CPU during the wall-clock benchmark)
- **keras-mdn-layer** (MDN layer for Keras underlying the ExoMDN model architecture being benchmarked) — github.com/cpmpercussion/keras-mdn-layer
- **conda** (Reproducible environment creation (conda env create -f environment.yml) to ensure the pinned dependency versions the runtime claim depends on)

## Examples

```
conda env create -f environment.yml && conda activate exomdn && pip install -e . && jupyter notebook introduction.ipynb
```

## Evaluation signals

- Measured wall-clock runtime of a single MDN prediction is below one second on a standard Intel i5 CPU
- Back-transformed posterior covers all four modeled layers: iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere
- Log-ratio-transformed MDN outputs are successfully back-transformed to physical mass fractions and thicknesses before reporting
- Runtime benchmark was run under the pinned environment (tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1, Python 3.7–3.11) on CPU
- Output file interior_posterior_prediction.csv contains both posterior samples/summary statistics and the recorded runtime

## Limitations

- The sub-second runtime and posterior coverage are only demonstrated for low-mass planets (<25 Earth masses), matching the >5.6 million synthetic training planets
- ExoMDN does not work well with Python versions above 3.11 due to incompatibility with higher tensorflow versions, constraining the benchmark environment
- The benchmark measures a point prediction; observational uncertainties require repeated predictions from within the uncertainties, which multiplies runtime accordingly

## Evidence

- [abstract] Sub-second posterior claim on commodity CPU: "Given mass, radius, and equilibrium temperature, we show that ExoMDN can deliver a full posterior distribution of mass fractions and thicknesses of each planetary layer in under a second on a"
- [readme] No dedicated interior model needed: "ExoMDN is capable of providing a full inference of the interior structure of low-mass exoplanets in under a second"
- [methods] Pinned conda environment for reproducible benchmark: "Create a new conda environment named ***exomdn*** from the `environment.yml` file"
- [abstract] Training-domain limit for benchmark inputs: "The model is trained on a large dataset of more than 5.6 million synthetic planets below 25 Earth masses"
- [readme] Python version constraint on benchmark setup: "Currently, due to incompatibility with higher tensorflow versions, ExoMDN does not work well with Python versions above 3.11"
- [abstract] Log-ratio transform requiring back-transformation after prediction: "We employ log-ratio transformations to convert the interior structure data into a form that the MDN can easily handle."
