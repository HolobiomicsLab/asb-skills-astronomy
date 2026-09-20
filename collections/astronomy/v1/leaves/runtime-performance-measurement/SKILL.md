---
name: runtime-performance-measurement
description: Use when a paper or README states a concrete runtime figure for an example
  script and the claim needs independent verification, e.g. vis-r's minimal example
  completing its MCMC fit in under 10 s.
when_to_use_negative:
- Input is a production-scale dataset rather than the bundled example — vis-r's claimed
  runtimes apply to the minimal example on a laptop, and other targets will differ
  in size
- The goal is comparing samplers rather than verifying a runtime claim — the stan
  and emcee versions differ in independent samples per step ('stan will yield many
  more independent samples for a given number of steps'), so wall-clock time alone
  is not a fair sampler-quality metric
- The workflow is too slow to warrant the method (e.g. high-resolution ALMA imaging)
  — the article's motivation is that image-based methods are computationally expensive,
  so timing those would measure the wrong thing
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: vis-r
  role: Package being timed; installed from GitHub so the emcee-based minimal example
    can be executed end-to-end
  repo: https://github.com/drgmk/vis-r
- name: Python
  role: Interpreter used to execute the example script (`python hello_world.py`) while
    wall-clock time is measured
- name: emcee
  role: MCMC sampler driving the fit whose runtime is measured in the emcee version
    of vis-r
- name: conda
  role: Environment creation pinning scipy=1.11.4 (frank wants scipy<1.12) to reproduce
    the documented install
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:379fcba8da351ea8512466795bf5d08d577976fcf600543504118bbfeaecc75e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - Python
  - conda
  - emcee
  - vis-r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# runtime-performance-measurement

## Summary

Measures the end-to-end wall-clock runtime of a packaged example workflow — here the vis-r minimal example script (examples/hello_world.py, an emcee-based MCMC fit) — to validate a performance claim such as 'runs in less than 10 s on an M2 Mac'.

## When to use

Use when a paper or README states a concrete runtime figure for an example script and the claim needs independent verification, e.g. vis-r's minimal example completing its MCMC fit in under 10 s.

## When NOT to use

- Input is a production-scale dataset rather than the bundled example — vis-r's claimed runtimes apply to the minimal example on a laptop, and other targets will differ in size
- The goal is comparing samplers rather than verifying a runtime claim — the stan and emcee versions differ in independent samples per step ('stan will yield many more independent samples for a given number of steps'), so wall-clock time alone is not a fair sampler-quality metric
- The workflow is too slow to warrant the method (e.g. high-resolution ALMA imaging) — the article's motivation is that image-based methods are computationally expensive, so timing those would measure the wrong thing

## Inputs

- Packaged minimal example script (examples/hello_world.py from github.com/drgmk/vis-r)
- Bundled example visibilities shipped with the repository's examples folder
- Reference runtime threshold from the paper/README (e.g. 'less than 10 s on an M2 Mac')

## Outputs

- Measured end-to-end wall-clock runtime of the full example fit
- Fitted radial profile parameters and posterior/sampling outputs from the emcee-based fit
- Results file recording runtime and fitted parameters (e.g. hello_world_results)

## How to apply

Install the software exactly as documented — for vis-r, create a conda environment on arm64 macOS with `CONDA_SUBDIR=osx-arm64 conda create -n vis-r -c conda-forge python emcee matplotlib scipy=1.11.4 ...` and `pip install https://github.com/drgmk/vis-r/archive/refs/heads/main.zip` — so the measurement reproduces the reference configuration (M2 Mac). Locate the minimal example script and its bundled example visibilities in the repository's examples folder, execute the full script with Python, and time the entire end-to-end fit (including the emcee sampling), not just a sub-step. Extract the fitted radial-profile parameters and posterior outputs the script writes, and record both the fitted parameter values and the measured wall-clock runtime to a results file (e.g. hello_world_results). Judge success by comparing the recorded runtime against the reported threshold (less than 10 s) and confirming the fitted parameters are present in the output file.

## Related tools

- **vis-r** (Package being timed; installed from GitHub so the emcee-based minimal example can be executed end-to-end) — https://github.com/drgmk/vis-r
- **Python** (Interpreter used to execute the example script (`python hello_world.py`) while wall-clock time is measured)
- **emcee** (MCMC sampler driving the fit whose runtime is measured in the emcee version of vis-r)
- **conda** (Environment creation pinning scipy=1.11.4 (frank wants scipy<1.12) to reproduce the documented install)

## Examples

```
python hello_world.py  # in the vis-r examples folder; measure wall-clock time end-to-end
```

## Evaluation signals

- Recorded wall-clock runtime is below the reported threshold (< 10 s)
- hello_world_results file exists and contains both the runtime and fitted parameter values
- Fitted radial-profile parameters and posterior/sampling outputs are present in the script's outputs (emcee-based fit)
- Runtime is measured over the full script execution (`python hello_world.py`), not a partial step

## Limitations

- The < 10 s figure is specific to an M2 Mac with the documented conda environment (scipy=1.11.4); other hardware or dependency versions will give different runtimes
- The claim covers only the minimal example script; production fits with larger `--sz` averaging or longer emcee runs, or the stan version's Pathfinder pre-fit, are not covered by this timing
- This skill captures a single runtime datum, not a benchmark distribution; repeated timings would be needed for statistical claims

## Evidence

- [results] less than 10 s on an M2 Mac: "# minimal example script, this runs in less than 10s on an M2 Mac"
- [intro] run the example with Python: "(try `python hello_world.py` in the examples folder)"
- [readme] install with conda and pip: "Install the most recent commit on an M2 Mac with `conda`,
which will create the shell command `vis-r` (`frank` wants scipy<1.12)"
- [abstract] one to two orders of magnitude faster: "Combined, these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [readme] stan vs emcee sampling differences: "Both default to the same number of 
warmup/sampling steps, but `stan` will yield many more independent samples
for a given number of steps."
