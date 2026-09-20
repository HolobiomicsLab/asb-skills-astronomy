---
name: exoplanet-interior-posterior-inference
description: Use when a low-mass (<25 Earth mass) exoplanet needs full-posterior interior
  characterization from only mass, radius, and equilibrium temperature observables,
  including uncertainty propagation via repeated predictions within observational
  error ranges.
when_to_use_negative:
- The planet exceeds the training domain (above ~25 Earth masses) — the model was
  trained only on synthetic planets below 25 Earth masses
- No mass and radius measurements are available with usable precision (the characterized
  sample required mass and radius uncertainties below 10% and 5%, respectively)
- A dedicated, fully self-consistent interior structure model calculation is required
  rather than a fast machine-learning inference
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ExoMDN
  role: Mixture density network model performing the interior posterior inference
    from mass, radius, and equilibrium temperature
  repo: https://github.com/philippbaumeister/ExoMDN
- name: tensorflow
  role: Runs the MDN forward pass (version 2.11 pinned in environment.yml) on CPU
    for posterior prediction
- name: tensorflow-probability
  role: Probabilistic support (version 0.15) for the mixture-density posterior handling
- name: keras-mdn-layer
  role: MDN layer for Keras used by ExoMDN, providing the mixture output layer and
    sample_from_output sampling
  repo: https://github.com/cpmpercussion/keras-mdn-layer
- name: numpy
  role: Numerical handling of log-ratio back-transformation and posterior sample aggregation
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202346216
    title: 'ExoMDN: Rapid characterization of exoplanet interior structures with mixture
      density networks'
  source_content_hash: sha256:733ef12ed87d88f1ce359110141ad103259fdf792e593250bdcc2c667fce05f4
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
  - conda
  - keras-mdn-layer
  - numpy
  - pandas
  - pip
  - tensorflow
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# exoplanet-interior-posterior-inference

## Summary

Rapid Bayesian-style inference of exoplanet interior structure using ExoMDN mixture density networks: given mass, radius, and equilibrium temperature, it returns a full posterior over mass fractions and layer thicknesses (iron core, silicate mantle, water/high-pressure ice, H/He atmosphere) in under a second on a standard Intel i5 CPU.

## When to use

Use when a low-mass (<25 Earth mass) exoplanet needs full-posterior interior characterization from only mass, radius, and equilibrium temperature observables, including uncertainty propagation via repeated predictions within observational error ranges.

## When NOT to use

- The planet exceeds the training domain (above ~25 Earth masses) — the model was trained only on synthetic planets below 25 Earth masses
- No mass and radius measurements are available with usable precision (the characterized sample required mass and radius uncertainties below 10% and 5%, respectively)
- A dedicated, fully self-consistent interior structure model calculation is required rather than a fast machine-learning inference

## Inputs

- Planetary mass, radius, and equilibrium temperature (with stated uncertainties)
- Trained ExoMDN mixture density network models (Zenodo DOI 10.5281/zenodo.7670706)
- Optional: synthetic fluid Love number k2 as an additional observable

## Outputs

- Full posterior distribution of mass fractions and thicknesses for iron core, silicate mantle, water/high-pressure ice layer, and H/He atmosphere
- Aggregated uncertainty-aware posteriors (e.g., posterior_samples.csv, exoplanet_characterization.csv, interior_posterior_prediction.csv)
- Posterior constraint plots (e.g., k2_earth_analog_thickness_posterior.png via Matplotlib)

## How to apply

Create the exomdn conda environment from environment.yml (Python 3.7+, tensorflow=2.11, tensorflow-probability=0.15, scikit-learn=1.1.1) and install the package with pip install -e ., then load the trained MDN models from Zenodo (10.5281/zenodo.7670706) following introduction.ipynb. Assemble the observable vector (mass, radius, equilibrium temperature) and run the MDN forward pass on CPU; the model outputs a mixture-density posterior over log-ratio-transformed layer mass fractions and thicknesses, which must be back-transformed to physical values for the four modeled layers. To propagate observational uncertainties, draw repeated samples of the observables from within the stated uncertainty ranges (reference scales: mass <10%, radius <5%) and aggregate the repeated predictions into a combined uncertainty-aware posterior. Optionally append the fluid Love number k2 as an additional observable to reduce interior degeneracy — a 10%-accurate k2 constrains core and mantle thickness of an Earth analog to ~13% of true values.

## Related tools

- **ExoMDN** (Mixture density network model performing the interior posterior inference from mass, radius, and equilibrium temperature) — https://github.com/philippbaumeister/ExoMDN
- **tensorflow** (Runs the MDN forward pass (version 2.11 pinned in environment.yml) on CPU for posterior prediction)
- **tensorflow-probability** (Probabilistic support (version 0.15) for the mixture-density posterior handling)
- **keras-mdn-layer** (MDN layer for Keras used by ExoMDN, providing the mixture output layer and sample_from_output sampling) — https://github.com/cpmpercussion/keras-mdn-layer
- **numpy** (Numerical handling of log-ratio back-transformation and posterior sample aggregation)

## Examples

```
conda env create -f environment.yml && conda activate exomdn && pip install -e . && jupyter notebook introduction.ipynb
```

## Evaluation signals

- Single-prediction wall-clock runtime is below one second on a standard Intel i5 CPU
- Posterior covers all four modeled layers: iron core, silicate mantle, water/high-pressure ice, and H/He atmosphere
- Back-transformed mass fractions are physically valid (fractions sum appropriately after log-ratio inversion) and layer thicknesses are positive
- For the k2 experiment, aggregated core and mantle thickness posteriors constrain values to ~13% of the true values at 10% k2 accuracy
- Characterization table for the confirmed-exoplanet sample contains exactly 22 rows with posteriors for all four layers, consistent with reported structures for GJ 1214 b and TRAPPIST-1 planets

## Limitations

- Model is trained only on synthetic planets below 25 Earth masses; applicability to higher-mass planets is unsupported
- ExoMDN does not work well with Python versions above 3.11 due to tensorflow incompatibility
- Observational uncertainty handling relies on repeated predictions from within uncertainty ranges, which multiplies inference cost by the number of draws

## Evidence

- [abstract] Given mass, radius, and equilibrium temperature, we show that ExoMDN can deliver a full posterior distribution of mass fractions and thicknesses of each planetary layer in under a second: "Given mass, radius, and equilibrium temperature, we show that ExoMDN can deliver a full posterior distribution of mass fractions and thicknesses of each planetary layer in under a second"
- [intro] trained on more than 5.6 million synthetic planet interior structures: "The model is trained on more than 5.6 million synthetic planet interior structures."
- [methods] create conda environment and install package: "Create a new conda environment named ***exomdn*** from the `environment.yml` file"
- [abstract] observational uncertainties via repeated predictions: "Observational uncertainties can be easily accounted for through repeated predictions from within the uncertainties."
- [readme] under-a-second inference claim: "ExoMDN is capable of providing a full inference of the interior structure of low-mass exoplanets in 
under a second without the need for a dedicated interior model."
