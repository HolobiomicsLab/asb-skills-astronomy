---
name: planet-composition-prior-configuration
description: Decision support for choosing and running plaNETic prior options (A1–A3,
  B1–B3) when modelling internal structure of small planets (0.5–15 Mearth) from observed
  masses/radii and stellar parameters.
when_to_use_negative:
- Planet masses fall outside the 0.5–15 Earth-mass range the trained BICEPS-based
  neural networks support
- Goal is a single fixed-composition answer rather than comparing how alternative
  water-abundance and Si/Mg/Fe priors shape the posterior
- No GPU is available and run time is unacceptable (CPU-only execution drastically
  increases computation time)
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: plaNETic
  role: Neural-network Bayesian framework whose run_grid executables perform full
    grid accept-reject sampling under prior options A1–A3 and B1–B3
  repo: github.com/joannegger/plaNETic
- name: BICEPS
  role: Interior forward model on which the trained DNN surrogates used for sampling
    are based
- name: TensorFlow
  role: Runtime for the DNN surrogates; requires GPU access for efficient execution
- name: Python 3.9
  role: Virtual-environment Python version required for installing and running plaNETic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:6cc37de47f232ce375cd18805ca50d345fdc23ba86dca4062f54bcf75dd42aa5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BICEPS
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
yes_rate: 0.5833333333333334
n_claims: 12
---

# planet-composition-prior-configuration

## Summary

Configure and run the six prior options (A1–A3, B1–B3) of the plaNETic Bayesian internal-structure framework, which vary the water-abundance prior (formation inside vs. outside the water iceline) and the planetary Si/Mg/Fe ratio prior (stellar, iron-enriched, or free) to probe how prior assumptions shape inferred compositions.

## When to use

Decision support for choosing and running plaNETic prior options (A1–A3, B1–B3) when modelling internal structure of small planets (0.5–15 Mearth) from observed masses/radii and stellar parameters.

## When NOT to use

- Planet masses fall outside the 0.5–15 Earth-mass range the trained BICEPS-based neural networks support
- Goal is a single fixed-composition answer rather than comparing how alternative water-abundance and Si/Mg/Fe priors shape the posterior
- No GPU is available and run time is unacceptable (CPU-only execution drastically increases computation time)

## Inputs

- 'stellar_planetary_parameters.csv' with observed host-star and planet properties (e.g. CHEOPS+TESS radii, ESPRESSO/HIRES RV masses)
- Executable 'run_grid_xxx.py' adapted from the 'TOI-469_Egger+' or 'TOI-238_Egger+' template
- Trained plaNETic DNN surrogates of the BICEPS interior model

## Outputs

- 'xxx_posterior.npy' files in the 'posteriors' subfolder, one per prior option, each a 3D numpy array of shape (1, N, 28) with N varying by prior option
- Corresponding prior sample arrays of shape (1, M, 28)

## How to apply

Create a subfolder in 'run_grid' mirroring the 'TOI-469_Egger+' or 'TOI-238_Egger+' template, containing a 'stellar_planetary_parameters.csv' with the host star and all planets' observed properties and an adapted 'run_grid_xxx.py' executable. Run it with Python 3.9 in a GPU-enabled TensorFlow environment so the full grid accept-reject sampling executes under all six prior options using the trained DNN surrogates of the BICEPS interior model, with the default 10,000 sampled stars and 10,000 planets per star. Options A1–A3 and B1–B3 encode different combinations of water-prior (inside vs. outside iceline) and Si/Mg/Fe-ratio priors (stellar vs. iron-enriched vs. free), so comparing posteriors across options reveals how formation-location assumptions affect the inferred structure. Batch size in 'compute_radius' (line 1308 of plaNETic.py) can be tuned for performance, and run time scales linearly with planet number (~10–15 min per planet/prior on an Apple M4 Pro).

## Related tools

- **plaNETic** (Neural-network Bayesian framework whose run_grid executables perform full grid accept-reject sampling under prior options A1–A3 and B1–B3) — github.com/joannegger/plaNETic
- **BICEPS** (Interior forward model on which the trained DNN surrogates used for sampling are based)
- **TensorFlow** (Runtime for the DNN surrogates; requires GPU access for efficient execution)
- **Python 3.9** (Virtual-environment Python version required for installing and running plaNETic)

## Examples

```
cd run_grid/TOI-238_Egger+ && python run_grid_TOI-238.py
```

## Evaluation signals

- Each of the six prior options (A1–A3, B1–B3) produced an 'xxx_posterior.npy' file in the 'posteriors' subfolder
- Each output .npy loads as a numpy array with trailing dimension 28, matching the reference TOI-238 b Option A1 shape (1, 771909, 28)
- Validation run of 'run_grid_TOI-238.py' completes all six options at ~10–15 minutes each on a GPU machine
- Posterior sample counts N differ per prior option (e.g. (1, 771909, 28) for A1 vs. (1, 1941179, 28) for A2 in the reference run), confirming priors were actually varied

## Limitations

- Trained DNNs may be missing because of GitHub's Large File Storage issue; contact the authors for the files
- Running on CPUs only drastically increases computation time and is not recommended
- Reducing the default 10,000 sampled stars or 10,000 planets per star decreases computation time but leads to smaller posterior distributions
- Observed parameters may be compatible with multiple scenarios (e.g. H/He-envelope vs. water-rich sub-Neptunes), so prior choice can dominate interpretation

## Evidence

- [intro] water and Si/Mg/Fe prior options: "it allows for different choices in priors concerning the expected abundance of water (formation inside vs. outside of iceline) and the planetary Si/Mg/Fe ratios"
- [intro] grid sampling setup: [evidence span withheld — anchor status: not_found]
- [methods] run_grid folder structure: "create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
- [readme] batch-size tuning: "changing the batch size when running the neural network (in the function 'compute_radius', line 1308 of 'plaNETic.py') might improve the performance"
- [readme] default sampling sizes: "reducing the number of sampled stars (default: 10'000) and planets per sampled star (default: 10'000)"
