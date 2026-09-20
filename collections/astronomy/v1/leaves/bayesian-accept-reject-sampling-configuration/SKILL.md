---
name: bayesian-accept-reject-sampling-configuration
description: Use when observed stellar and planetary parameters (e.g. mass, radius,
  period for small planets of 0.5–15 Earth masses) must be converted into internal-structure
  posteriors, or when benchmarking runtime across the A1–A3/B1–B3 prior options of
  plaNETic.
when_to_use_negative:
- The planet's mass is outside the 0.5–15 Earth-mass range the BICEPS-trained networks
  were trained for
- Only CPU access is available and the ~10–15 min GPU runtime budget is unacceptable,
  since CPU-only execution drastically increases computation time and is not recommended
- The goal is photochemical or atmospheric observation modelling rather than internal-structure
  inference
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: plaNETic
  role: Framework providing the full grid accept-reject sampler, prior options, and
    run_grid templates (TOI-238_Egger+, TOI-469_Egger+)
  repo: https://github.com/joannegger/plaNETic
- name: Python 3.9
  role: Virtual-environment interpreter used to install and execute the run_grid executable
- name: TensorFlow
  role: Runs the trained neural-network surrogate for the forward model; needs GPU
    access (Metal device on Apple Silicon)
- name: BICEPS
  role: Interior model whose training data underlie the neural-network surrogate used
    in accept-reject sampling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:58501572d094b94679f78e10cf024be8a9d196a231a62bc7fa872a2c283bf347
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# bayesian-accept-reject-sampling-configuration

## Summary

Configure and run plaNETic's full grid accept-reject sampling with trained BICEPS neural-network surrogates to compute posteriors of a planet's internal structure (28 parameters per sample) from observed stellar/planetary properties, across selectable water-abundance and Si/Mg/Fe prior options.

## When to use

Use when observed stellar and planetary parameters (e.g. mass, radius, period for small planets of 0.5–15 Earth masses) must be converted into internal-structure posteriors, or when benchmarking runtime across the A1–A3/B1–B3 prior options of plaNETic.

## When NOT to use

- The planet's mass is outside the 0.5–15 Earth-mass range the BICEPS-trained networks were trained for
- Only CPU access is available and the ~10–15 min GPU runtime budget is unacceptable, since CPU-only execution drastically increases computation time and is not recommended
- The goal is photochemical or atmospheric observation modelling rather than internal-structure inference

## Inputs

- stellar_planetary_parameters.csv with observed properties of the host star and planets (modeled after run_grid/TOI-238_Egger+)
- Executable run_grid_xxx.py adapted from the template examples
- Choice of prior option: A1–A3 (water abundance, formation inside vs. outside of iceline) and/or B1–B3 (Si/Mg/Fe ratios: stellar vs. iron-enriched vs. free)
- Trained DNN surrogate networks shipped with plaNETic (trained on the BICEPS interior model)

## Outputs

- xxx_posterior.npy files in a 'posteriors' subfolder, with posterior array of shape (1, N, 28)
- Prior array of shape (1, N, 28) printed per option
- Optional runtime_table.csv compiling per-option wall-clock time and iteration rate for A1–A3 and B1–B3

## How to apply

Create a Python 3.9 virtual environment, git clone plaNETic, and 'pip install .', ensuring TensorFlow can access a GPU (on Apple Silicon verify the Metal device is set, e.g. 'Metal device set to: Apple M4 Pro'). Create a subfolder in 'run_grid' modeled on 'TOI-238_Egger+' containing a 'stellar_planetary_parameters.csv' with the host star and planet observed properties, plus an executable 'run_grid_xxx.py'. Run the executable so it performs the full grid accept-reject sampling over the default 10'000 sampled stars x 10'000 planets per star (progress loops of 9429 posterior and 939 prior iterations per prior option), selecting among prior options A1–A3 (water inside vs. outside iceline) and B1–B3 (Si/Mg/Fe: stellar, iron-enriched, free). The run writes 'xxx_posterior.npy' files in a 'posteriors' subfolder with posterior and prior shapes (1, N, 28). For benchmarking, record wall-clock time and iteration rate per option and compile into runtime_table.csv; expect ~10–15 min per planet and prior option on an M4 Pro, with computation time scaling linearly with the number of planets.

## Related tools

- **plaNETic** (Framework providing the full grid accept-reject sampler, prior options, and run_grid templates (TOI-238_Egger+, TOI-469_Egger+)) — https://github.com/joannegger/plaNETic
- **Python 3.9** (Virtual-environment interpreter used to install and execute the run_grid executable)
- **TensorFlow** (Runs the trained neural-network surrogate for the forward model; needs GPU access (Metal device on Apple Silicon))
- **BICEPS** (Interior model whose training data underlie the neural-network surrogate used in accept-reject sampling)

## Examples

```
cd run_grid/TOI-238_Egger+ && python run_grid_TOI-238.py
```

## Evaluation signals

- Each prior option reports a posterior array of shape (1, N, 28) and a prior array of shape (1, N, 28)
- Measured wall-clock time for one planet and one prior option falls within the reported ~10–15 min range (observed 11:17–16:54 on an M4 Pro, at 9.29–13.92 it/s over 9429 iterations)
- Posterior iteration loops complete at 9429 iterations and prior loops at 939 iterations, matching the README example output
- xxx_posterior.npy files are generated in the 'posteriors' subfolder for each run
- Per-option posterior sample counts (e.g. 771909 for A1, 16505 for B1) are nonzero and option-dependent as in the README example

## Limitations

- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage; contact the authors for the files
- Running on CPUs only drastically increases computation time and is not recommended; a GPU is needed for efficient runs
- Reducing the default 10'000 sampled stars x 10'000 planets per star decreases computation time but leads to smaller posterior distributions
- Changing the batch size in 'compute_radius' (line 1308 of 'plaNETic.py') may be needed to improve performance on some machines
- Computation time scales linearly with the number of planets modelled simultaneously in multiplanetary systems

## Evidence

- [intro] full grid accept-reject sampling with NN surrogate: [evidence span withheld — anchor status: not_found]
- [intro] prior choices: "it allows for different choices in priors concerning the expected abundance of water (formation inside vs. outside of iceline) and the planetary Si/Mg/Fe ratios"
- [methods] installation and run steps: "To install the code, create a new virtual environment with Python 3.9, then git clone plaNETic and run 'pip install .'"
- [readme] runtime and scaling: "the inference process for one planet and prior option takes around 10-15 minutes on our machine. In multiplanetary systems, all planets are modelled simultaneously, with the computation time scaling"
- [readme] output shapes and GPU: "Shape of posteriors:
(1, 771909, 28)"
