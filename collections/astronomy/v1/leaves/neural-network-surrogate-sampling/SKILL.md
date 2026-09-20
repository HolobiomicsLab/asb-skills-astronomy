---
name: neural-network-surrogate-sampling
description: Use when you have observed stellar and planetary parameters (e.g. CHEOPS/TESS
  radii plus ESPRESSO/HIRES RVs) for planets of 0.5–15 M⊕ and need internal-structure
  posteriors without running the full BICEPS interior model directly.
when_to_use_negative:
- Planet masses fall outside the validated 0.5–15 Earth-mass range of the trained
  BICEPS-surrogate networks
- You need physical forward-model outputs (e.g. detailed composition profiles) rather
  than fast posterior sampling — run BICEPS directly instead
- 'No GPU is available and computation time cannot be tolerated: CPU-only runs ''drastically
  increase the computation time and is not recommended'''
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: plaNETic
  role: Neural-network-based Bayesian internal structure modelling framework that
    performs the full grid accept-reject sampling with DNN surrogates
  repo: https://github.com/joannegger/plaNETic
- name: BICEPS
  role: Interior model on whose outputs the surrogate neural networks were trained
- name: TensorFlow
  role: Deep-learning backend executing the DNN surrogates; requires GPU access for
    efficient runtime
- name: Python 3.9
  role: Runtime environment version required for installing and running plaNETic ('pip
    install .' after git clone)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:bbc86b80a1046140e8d4eb6437669709c92353d83abddac70b20905aba2ff3c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannegger/plaNETic
  tools:
  - BICEPS
  - Python
  - Python 3.9
  - TensorFlow
  - plaNETic
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# neural-network-surrogate-sampling

## Summary

Infers Bayesian posteriors of a small exoplanet's internal structure by running full grid accept-reject sampling through trained DNN surrogates of the BICEPS interior model (via the plaNETic framework), replacing slow physical forward models with fast neural network inference over six prior options (A1–A3, B1–B3).

## When to use

Use when you have observed stellar and planetary parameters (e.g. CHEOPS/TESS radii plus ESPRESSO/HIRES RVs) for planets of 0.5–15 M⊕ and need internal-structure posteriors without running the full BICEPS interior model directly.

## When NOT to use

- Planet masses fall outside the validated 0.5–15 Earth-mass range of the trained BICEPS-surrogate networks
- You need physical forward-model outputs (e.g. detailed composition profiles) rather than fast posterior sampling — run BICEPS directly instead
- No GPU is available and computation time cannot be tolerated: CPU-only runs 'drastically increase the computation time and is not recommended'

## Inputs

- 'stellar_planetary_parameters.csv' with observed host-star and planet properties (radii from joint CHEOPS+TESS modelling, masses/RVs from ESPRESSO and HIRES)
- Executable 'run_grid_xxx.py' adapted from a template subfolder (e.g. 'TOI-469_Egger+')
- Trained DNN surrogate networks shipped with plaNETic (trained on the BICEPS interior model)

## Outputs

- 'xxx_posterior.npy' files in a 'posteriors' subfolder, one per prior option (A1–A3, B1–B3), each a 3D numpy array of shape (1, N, 28)
- Corresponding prior samples of matching trailing dimension 28 (e.g. (1, 117814, 28) for TOI-238 b Option A1)

## How to apply

Create a new subfolder in 'run_grid' mirroring the 'TOI-469_Egger+' or 'TOI-238_Egger+' template, containing a 'stellar_planetary_parameters.csv' with the observed host-star and planet properties and an adapted executable 'run_grid_xxx.py'. Run the executable with Python 3.9 in a GPU-enabled environment (TensorFlow must see the GPU; on Apple Silicon follow the metal-device setup), which performs full grid accept-reject sampling using the trained DNN surrogates with the default 10,000 sampled stars and 10,000 planets per star, under six prior options covering water-abundance priors (inside vs. outside of iceline) and Si/Mg/Fe ratio priors (stellar vs. iron-enriched vs. free). The run generates 'xxx_posterior.npy' files in a 'posteriors' subfolder; in multiplanetary systems all planets are modelled simultaneously with computation time scaling linearly with planet count. If runtime is excessive, tune the batch size in the 'compute_radius' function at line 1308 of 'plaNETic.py', or reduce the number of sampled stars/planets (at the cost of smaller posteriors). Validate against the reference TOI-238 b run, whose Option A1 posterior has shape (1, 771909, 28).

## Related tools

- **plaNETic** (Neural-network-based Bayesian internal structure modelling framework that performs the full grid accept-reject sampling with DNN surrogates) — https://github.com/joannegger/plaNETic
- **BICEPS** (Interior model on whose outputs the surrogate neural networks were trained)
- **TensorFlow** (Deep-learning backend executing the DNN surrogates; requires GPU access for efficient runtime)
- **Python 3.9** (Runtime environment version required for installing and running plaNETic ('pip install .' after git clone))

## Examples

```
conda create -n planetic python=3.9; git clone https://github.com/joannegger/plaNETic && pip install .; cd run_grid/TOI-238_Egger+ && python run_grid_TOI-238.py
```

## Evaluation signals

- Each of the six prior options (A1–A3, B1–B3) produced a 'xxx_posterior.npy' file in the 'posteriors' subfolder
- Each output loads as a numpy array with trailing dimension of 28, matching the reference (1, 771909, 28) pattern from the TOI-238 b Option A1 run
- Reference validation run 'run_grid_TOI-238.py' completes with progress bars (~9429 iterations) and prints expected posterior/prior shapes for all six options
- TensorFlow reports GPU access (e.g. 'Metal device set to: Apple M4 Pro') before sampling starts
- Inference per planet and prior option takes roughly 10–15 minutes on the reference machine; large deviations suggest CPU-only execution or mis-tuned batch size

## Limitations

- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage; contact the authors while a permanent storage solution is pending
- CPU-only execution drastically increases computation time and is not recommended
- Reducing the number of sampled stars (default 10,000) or planets per star (default 10,000) shrinks the posterior distributions
- Caution is required when using TESS photometry for small planets with low per-transit SNR and large gaps — planet d's radius deviated by more than 3σ from the discovery paper's median value, and such input errors propagate into the posteriors

## Evidence

- [intro] It uses a full grid accept-reject sampling algorithm with neural networks trained on the interior model of the BICEPS code as a fast surrogate for the forward model.: [evidence span withheld — anchor status: not_found]
- [intro] bayesian internal structure modelling framework for small exoplanets: "Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth"
- [readme] create a new subfolder in run_grid with parameter file and executable: "To infer the internal structure of the planets in an observed planetary system, create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
- [methods] changing the batch size when running the neural network might improve performance: "changing the batch size when running the neural network (in the function 'compute_radius', line 1308 of 'plaNETic.py') might improve the performance"
- [readme] the computation time can be decreased by reducing the number of sampled stars: "the computation time can be decreased by reducing the number of sampled stars (default: 10'000) and planets per sampled star (default: 10'000). However, this will lead to smaller posterior"
- [readme] GitHub Large File Storage issue for trained DNNs: "If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage."
- [readme] Shape of posteriors reference output: "Shape of posteriors:
(1, 771909, 28)"
