---
name: bayesian-posterior-sampling
description: Use when you have observed masses (0.5–15 M⊕) and radii plus stellar
  parameters for small exoplanets and need internal-structure posteriors under multiple
  water-abundance and Si/Mg/Fe ratio prior choices.
when_to_use_negative:
- Planet masses are outside the 0.5–15 Earth-mass range the trained DNN surrogates
  were built for (e.g. giant planets)
- Only CPU is available and fast turnaround is required — the README states CPU-only
  runs 'drastically increase the computation time and is not recommended'
- You need to derive planetary radii themselves from photometry — this skill consumes
  already-fit parameters (radius/RV fitting is a separate step)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: plaNETic
  role: Neural network-based Bayesian internal structure modelling framework that
    runs the full grid accept-reject sampling using BICEPS-trained DNN surrogates
  repo: github.com/joannegger/plaNETic
- name: BICEPS
  role: Interior forward model on which the neural network surrogates are trained
- name: Python 3.9
  role: Runtime environment for installing and executing plaNETic (virtualenv + pip
    install)
- name: TensorFlow
  role: Runs the trained DNN surrogates; requires GPU access, batch size tunable in
    compute_radius (line 1308 of plaNETic.py)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:7b1c3c233eea56f8bbbd0fe345973fd54b6850f5f0309c007acbaf4aae3f276f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BICEPS
  - Python
  - Python 3.9
  - TensorFlow
  - plaNETic
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# bayesian-posterior-sampling

## Summary

Runs full grid accept-reject sampling with neural network surrogates (plaNETic, trained on the BICEPS interior model) to compute Bayesian posteriors of a small planet's internal structure from observed stellar/planetary parameters.

## When to use

Use when you have observed masses (0.5–15 M⊕) and radii plus stellar parameters for small exoplanets and need internal-structure posteriors under multiple water-abundance and Si/Mg/Fe ratio prior choices.

## When NOT to use

- Planet masses are outside the 0.5–15 Earth-mass range the trained DNN surrogates were built for (e.g. giant planets)
- Only CPU is available and fast turnaround is required — the README states CPU-only runs 'drastically increase the computation time and is not recommended'
- You need to derive planetary radii themselves from photometry — this skill consumes already-fit parameters (radius/RV fitting is a separate step)

## Inputs

- stellar_planetary_parameters.csv with observed host-star and planet properties (masses, radii, stellar parameters)
- run_grid_xxx.py executable adapted from the TOI-469_Egger+ or TOI-238_Egger+ template
- GPU-enabled Python 3.9 environment with TensorFlow and trained BICEPS DNN surrogates

## Outputs

- xxx_posterior.npy files (one per prior option A1–A3, B1–B3) in a 'posteriors' subfolder, each a 3D numpy array of shape (1, N, 28), e.g. (1, 771909, 28) for TOI-238 b Option A1
- prior samples of shape (1, N, 28) for comparison against posteriors

## How to apply

Install plaNETic in a Python 3.9 virtual environment (git clone, 'pip install .'), ideally with GPU access to TensorFlow. Create a subfolder in 'run_grid' mirroring the 'TOI-469_Egger+' template, containing 'stellar_planetary_parameters.csv' with the host star and all planets' observed properties (e.g. CHEOPS+TESS radii and ESPRESSO/HIRES RVs for HIP 29442 b, c, d), plus an executable 'run_grid_xxx.py'. Adapt and run the executable: it performs full grid accept-reject sampling with the default 10,000 sampled stars and 10,000 planets per star across six prior options (A1–A3, B1–B3) covering water abundance inside vs. outside the iceline and stellar vs. iron-enriched vs. free Si/Mg/Fe ratios. Each option generates an 'xxx_posterior.npy' file in a 'posteriors' subfolder; in multiplanet systems all planets are modelled simultaneously with computation time scaling linearly with planet count.

## Related tools

- **plaNETic** (Neural network-based Bayesian internal structure modelling framework that runs the full grid accept-reject sampling using BICEPS-trained DNN surrogates) — github.com/joannegger/plaNETic
- **BICEPS** (Interior forward model on which the neural network surrogates are trained)
- **Python 3.9** (Runtime environment for installing and executing plaNETic (virtualenv + pip install))
- **TensorFlow** (Runs the trained DNN surrogates; requires GPU access, batch size tunable in compute_radius (line 1308 of plaNETic.py))

## Examples

```
cd run_grid/TOI-238_Egger+ && python run_grid_TOI-238.py
```

## Evaluation signals

- Each of the six prior options (A1–A3, B1–B3) produced an 'xxx_posterior.npy' file in the 'posteriors' subfolder
- Each output .npy loads as a numpy array with trailing dimension 28, matching the reference TOI-238 b Option A1 shape (1, 771909, 28)
- Prior arrays are also generated with the same 28-column trailing shape (e.g. (1, 117814, 28) for TOI-238 b A1)
- The TOI-238 b validation run (run_grid_TOI-238.py) completes in roughly 10–15 minutes per planet per prior option on a GPU machine
- Posterior row counts differ meaningfully between prior options (e.g. 16,505 for B1 vs 1,941,179 for A2 in the reference run), confirming priors were actually varied

## Limitations

- If plaNETic cannot find the trained DNNs, this is a GitHub Large File Storage issue; contact the authors for the trained network files
- GPU access is effectively required; CPU-only execution drastically increases computation time
- Posterior sample sizes depend on the sampled-star/planet counts (defaults 10,000 each); reducing them yields smaller posterior distributions
- Only apply to small exoplanets with masses between 0.5 and 15 Earth masses

## Evidence

- [intro] full grid accept-reject sampling with DNN surrogates: [evidence span withheld — anchor status: not_found]
- [intro] prior choices: "it allows for different choices in priors concerning the expected abundance of water (formation inside vs. outside of iceline) and the planetary Si/Mg/Fe ratios (stellar vs. iron-enriched vs. free)"
- [readme] run_grid setup: "create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
- [readme] output files: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [readme] mass range: "plaNETic is a neural network-based Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth."
- [readme] batch size tuning: "changing the batch size when running the neural network (in the function 'compute_radius', line 1308 of 'plaNETic.py') might improve the performance"
- [readme] runtime: "the inference process for one planet and prior option takes around 10-15 minutes on our machine"
