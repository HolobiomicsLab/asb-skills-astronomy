---
name: benchmark-coupling-table-generation
description: Use when quantifying how RG running and dimension-6 operator mixing change
  spin-dependent DM-nucleon couplings or direct detection bounds for a vector-mediator
  simplified model at a chosen benchmark (mediator mass + high-energy couplings).
when_to_use_negative:
- The model is not a simplified vector-mediator model with dimension-6 operators (runDM's
  scope is vector mediators; for limits on general models the article points to the
  NRopsDD interface instead)
- You only need couplings at a single fixed scale without assessing the running-induced
  shift — the with/without-running comparison table is the point of this skill
tools:
- name: runDM
  role: Evolves the dimension-6 operator couplings from the mediator mass scale to
    the nuclear energy scale, with and without operator mixing, and extracts low-energy
    couplings to quarks and nucleons
  repo: https://github.com/bradkav/runDM
- name: Python
  role: Language for executing the runDM example script, computing coupling ratios,
    and writing the CSV/plot outputs
  repo: https://github.com/bradkav/runDM
- name: Jupyter Notebook
  role: Interactive environment for the provided runDM-examples.ipynb workflow
  repo: https://github.com/bradkav/runDM
- name: Mathematica
  role: Alternative implementation of runDM for the same coupling evolution, with
    an example notebook
  repo: https://github.com/bradkav/runDM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:bcde8d18ed411d377354dd8278a3064981e6c57718aa52e1f9386bac7acf80cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bradkav/runDM
  tools:
  - Jupyter Notebook
  - Mathematica
  - Python
  - runDM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# benchmark-coupling-table-generation

## Summary

Generates benchmark tables comparing low-energy spin-dependent DM-nucleon couplings computed with versus without RG running for an axial-vector vector mediator, using runDM to evolve dimension-6 operator couplings from the mediator mass scale down to the nuclear energy scale.

## When to use

Use when quantifying how RG running and dimension-6 operator mixing change spin-dependent DM-nucleon couplings or direct detection bounds for a vector-mediator simplified model at a chosen benchmark (mediator mass + high-energy couplings).

## When NOT to use

- The model is not a simplified vector-mediator model with dimension-6 operators (runDM's scope is vector mediators; for limits on general models the article points to the NRopsDD interface instead)
- You only need couplings at a single fixed scale without assessing the running-induced shift — the with/without-running comparison table is the point of this skill

## Inputs

- Axial-vector mediator benchmark point: mediator mass and high-energy couplings of the mediator to SM fields
- runDM Python package installed from github.com/bradkav/runDM (Zenodo deposit doi:10.5281/zenodo.823249)
- runDM example script / Jupyter notebook (runDM-examples.ipynb) as starting template

## Outputs

- coupling_ratios.csv — pandas table of running-included / running-neglected ratios of spin-dependent couplings to protons and neutrons
- coupling_comparison.png — matplotlib plot comparing running-included vs running-neglected couplings and their ratios
- Low-energy operator coefficients: couplings to up, down and strange quarks and to protons and neutrons at the nuclear energy scale

## How to apply

Install runDM (github.com/bradkav/runDM, Zenodo doi:10.5281/zenodo.823249) and start from the provided Python example script or runDM-examples.ipynb. Define an axial-vector mediator benchmark by specifying the mediator mass and the high-energy couplings of the mediator to SM fields. Run the evolution from the mediator mass scale to the nuclear energy scale twice: once taking into account the mixing of all dimension-6 operators (running included) and once neglecting operator mixing (running neglected). Extract the low-energy couplings to protons and neutrons relevant for the spin-dependent cross section from both runs, compute the running-included/neglected ratio, tabulate with pandas in coupling_ratios.csv, and visualize with matplotlib in coupling_comparison.png. Validate against the paper's reported finding that running alters the axial-vector spin-dependent bounds by a factor of two or more.

## Related tools

- **runDM** (Evolves the dimension-6 operator couplings from the mediator mass scale to the nuclear energy scale, with and without operator mixing, and extracts low-energy couplings to quarks and nucleons) — https://github.com/bradkav/runDM
- **Python** (Language for executing the runDM example script, computing coupling ratios, and writing the CSV/plot outputs) — https://github.com/bradkav/runDM
- **Jupyter Notebook** (Interactive environment for the provided runDM-examples.ipynb workflow) — https://github.com/bradkav/runDM
- **Mathematica** (Alternative implementation of runDM for the same coupling evolution, with an example notebook) — https://github.com/bradkav/runDM

## Examples

```
jupyter nbconvert --to notebook --execute runDM-examples.ipynb
```

## Evaluation signals

- coupling_ratios.csv exists and contains running-included/neglected ratios for both proton and neutron spin-dependent couplings at the benchmark
- The derived shift in spin-dependent couplings corresponds to a change in direct detection bounds by a factor of two or more, matching the paper's axial-vector-mediator finding
- The two runDM runs differ only in operator-mixing treatment (all dimension-6 operators mixed vs couplings evaluated without mixing), so the ratio isolates the running effect
- Extracted coefficients include the full set runDM is designed to output: low-energy couplings to up, down and strange quarks and to protons and neutrons
- coupling_comparison.png renders the paired coupling values and ratios from the CSV without mismatched rows

## Limitations

- runDM covers running of DM-SM couplings in simplified models with vector mediators only (dimension-6 operator mixing); other mediator types require different machinery
- For limits on general (non-simplified) models, runDM alone is insufficient — the README points to interfacing with the NRopsDD code
- The code is available only in Mathematica and Python; the README notes other-language requests require contacting the authors

## Evidence

- [abstract] For an axial-vector mediator, the running can alter the derived bounds on the spin-dependent DM-nucleon cross section by a factor of two or more.: "For an axial-vector mediator, the running can alter the derived bounds on the spin-dependent DM-nucleon cross section by a factor of two or more."
- [readme] By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy"
- [readme] calculate the couplings at low energy, taking into account the mixing of all dimension-6 operators: "calculate the couplings at low energy, taking into account the mixing of all dimension-6 operators"
- [intro] extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons: "extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons"
- [methods] For the Python code, we provide an example script as well as Jupyter Notebook.: "For the Python code, we provide an example script as well as Jupyter Notebook."
