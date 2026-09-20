---
name: planetary-internal-structure-posterior-inference
description: Use when you have observed host-star and planetary parameters (mass,
  radius, period) for small exoplanets (0.5–15 Mearth) and need Bayesian posteriors
  of internal structure and composition under multiple prior configurations.
when_to_use_negative:
- The planet's mass is outside the 0.5–15 Mearth range the framework was designed
  for
- No GPU is available and long runtimes are unacceptable — CPU-only operation 'drastically
  increase[s] the computation time and is not recommended'
- The goal is photoevaporation/upper-atmosphere modelling itself rather than internal-structure
  inference — that downstream stage consumes these posteriors but requires hydrodynamic
  models instead
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: plaNETic
  role: Neural network-based Bayesian internal structure modelling framework run via
    the run_grid executables to produce internal-structure posteriors
  repo: github.com/joannegger/plaNETic
- name: BICEPS
  role: Interior model on which the surrogate neural networks were trained
- name: Python 3.9
  role: Virtual-environment runtime required to install and execute plaNETic
- name: TensorFlow
  role: Runs the trained DNN surrogate on GPU (Metal device on Apple Silicon) during
    accept-reject sampling
- name: CHEOPS
  role: Photometry (9.6 days of observations of HIP 29442) jointly modelled with TESS
    to derive the planetary radii that constrain the inference
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:df93e9a4d8350086f009f254b07e3682ad29ffe02a398613b76cb016cda73a90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BICEPS
  - CHEOPS
  - Python
  - Python 3.9
  - TensorFlow
  - plaNETic
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# planetary-internal-structure-posterior-inference

## Summary

Infers posteriors of a planet's internal structure (core/iron/silicate/water/H-He envelope fractions) from observed planetary and stellar parameters using plaNETic, a neural network-based Bayesian framework with full grid accept-reject sampling over BICEPS-trained surrogate networks.

## When to use

Use when you have observed host-star and planetary parameters (mass, radius, period) for small exoplanets (0.5–15 Mearth) and need Bayesian posteriors of internal structure and composition under multiple prior configurations.

## When NOT to use

- The planet's mass is outside the 0.5–15 Mearth range the framework was designed for
- No GPU is available and long runtimes are unacceptable — CPU-only operation 'drastically increase[s] the computation time and is not recommended'
- The goal is photoevaporation/upper-atmosphere modelling itself rather than internal-structure inference — that downstream stage consumes these posteriors but requires hydrodynamic models instead

## Inputs

- stellar_planetary_parameters.csv with observed properties of the host star and planets (e.g. radii derived from CHEOPS + TESS joint modelling, published ESPRESSO/HIRES RVs)
- run_grid_xxx.py executable adapted to the target system
- trained neural-network surrogate DNNs from the BICEPS interior model (bundled with the plaNETic repository)

## Outputs

- xxx_posterior.npy files in a 'posteriors' subfolder, one per planet and prior option, each of shape (1, N, 28)
- prior samples of shape (1, N, 28) printed by the executable for each prior option

## How to apply

Create a Python 3.9 virtual environment, git clone plaNETic and run 'pip install .', ensuring TensorFlow accesses a GPU (on Apple Silicon verify the Metal device is set). Create a subfolder in 'run_grid' (e.g. 'TOI-238_Egger+') containing 'stellar_planetary_parameters.csv' with the observed host-star and planet properties, plus an executable 'run_grid_xxx.py'; the full grid accept-reject sampling runs by default over 10'000 sampled stars x 10'000 planets per star with progress loops of 9429 posterior and 939 prior iterations per prior option. Choose among the supported priors for water abundance (formation inside vs. outside of the iceline) and planetary Si/Mg/Fe ratios (stellar vs. iron-enriched vs. free), running one inference per option (A1–A3, B1–B3). The executable writes 'xxx_posterior.npy' files into a 'posteriors' subfolder, each with shape (1, N, 28); for multiplanetary systems all planets are modelled simultaneously with runtime scaling linearly with planet count. If performance matters, tune the batch size in 'compute_radius' (line 1308 of 'plaNETic.py'), or reduce the number of sampled stars/planets per star at the cost of smaller posteriors.

## Related tools

- **plaNETic** (Neural network-based Bayesian internal structure modelling framework run via the run_grid executables to produce internal-structure posteriors) — github.com/joannegger/plaNETic
- **BICEPS** (Interior model on which the surrogate neural networks were trained)
- **Python 3.9** (Virtual-environment runtime required to install and execute plaNETic)
- **TensorFlow** (Runs the trained DNN surrogate on GPU (Metal device on Apple Silicon) during accept-reject sampling)
- **CHEOPS** (Photometry (9.6 days of observations of HIP 29442) jointly modelled with TESS to derive the planetary radii that constrain the inference)

## Examples

```
python run_grid_TOI-238.py
```

## Evaluation signals

- Each prior option reports a posterior array of shape (1, N, 28) and a prior array of shape (1, N, 28)
- Wall-clock runtime for one planet and one prior option falls in the roughly 10–15 minute range (observed 11–17 min on an M4 Pro, e.g. Option A1: 9429 iterations at 11.47it/s taking 13:41)
- The 'posteriors' subfolder contains 'xxx_posterior.npy' files for every modelled planet and prior option
- The validation run on TOI-238 b completes without plaNETic failing to find the trained DNNs (which would indicate a GitHub Large File Storage issue)
- Iteration rate per option is consistent with GPU usage (on the order of 9–14 it/s for posterior loops, ~1600–2000 it/s for prior loops on an M4 Pro)

## Limitations

- A GPU is effectively required; CPU-only runs drastically increase computation time
- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage — contact the authors for the data
- Reducing the number of sampled stars (default 10'000) or planets per star (default 10'000) shrinks the posterior distributions
- Windows installation is possible but more tedious; a support protocol must be requested from the authors
- Batch size in compute_radius (line 1308 of plaNETic.py) may need machine-specific tuning for performance

## Evidence

- [readme] It uses a full grid accept-reject sampling algorithm with neural networks trained on the interior model of the BICEPS code as a fast surrogate for the forward model.: [evidence span withheld — anchor status: not_found]
- [intro] Different prior choices for water and Si/Mg/Fe ratios: "it allows for different choices in priors concerning the expected abundance of water (formation inside vs. outside of iceline) and the planetary Si/Mg/Fe ratios"
- [methods] Installation procedure: "To install the code, create a new virtual environment with Python 3.9, then git clone plaNETic and run 'pip install .'"
- [readme] run_grid folder structure and outputs: "create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
- [readme] Runtime guidance: "the inference process for one planet and prior option takes around 10-15 minutes on our machine"
- [readme] GPU requirement: "plaNETic needs access to a GPU to run efficiently. While it is in principle possible to run the code on CPUs only, this will drastically increase the computation time"
- [readme] Example validation output: "Shape of posteriors:
(1, 771909, 28)"
