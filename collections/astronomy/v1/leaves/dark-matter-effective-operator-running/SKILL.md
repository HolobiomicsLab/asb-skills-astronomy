---
name: dark-matter-effective-operator-running
description: Use when you need low-energy DM-nucleon couplings or direct detection
  bounds for a vector-mediator simplified model defined at high energy, or when translating
  LHC mediator mass limits into direct detection cross-section bounds.
when_to_use_negative:
- The model is not a simplified model with a vector mediator (runDM does not cover
  other mediator types)
- You already have direct-detection operator coefficients at the nuclear scale and
  only need, e.g., the direct detection likelihood — the running step is unnecessary
- You need limits on general non-simplified models — use the NRopsDD interface workflow
  instead
tools:
- name: runDM
  role: Core code that performs the RG evolution of DM-SM couplings with all dimension-6
    operator mixing and extracts direct-detection operator coefficients; available
    in Mathematica and Python
  repo: https://github.com/bradkav/runDM/
- name: Python
  role: Implementation language of runDM's example script and Jupyter Notebook used
    to run the evolution and export low-energy couplings
- name: Jupyter Notebook
  role: Interactive interface to runDM examples (runDM-examples.ipynb) used as the
    reference output for validation
- name: Mathematica
  role: Alternative runDM implementation with an example notebook and the interface
    to NRopsDD for limits on general models
- name: NRopsDD
  role: External code that runDM interfaces with for obtaining limits on general models
    from the extracted low-energy couplings
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:8f73daca96594c9c497066bfe447e5bc66c02da9995b914702e313edf2d93a9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bradkav/runDM
  tools:
  - Jupyter Notebook
  - Mathematica
  - NRopsDD
  - Python
  - runDM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# dark-matter-effective-operator-running

## Summary

Evolve the couplings of Dark Matter to Standard Model fields in vector-mediator simplified models from the mediator mass scale down to the nuclear energy scale, accounting for the mixing of all dimension-6 operators, then extract the low-energy operator coefficients (couplings to u, d, s quarks and to protons/neutrons) used in direct detection.

## When to use

Use when you need low-energy DM-nucleon couplings or direct detection bounds for a vector-mediator simplified model defined at high energy, or when translating LHC mediator mass limits into direct detection cross-section bounds.

## When NOT to use

- The model is not a simplified model with a vector mediator (runDM does not cover other mediator types)
- You already have direct-detection operator coefficients at the nuclear scale and only need, e.g., the direct detection likelihood — the running step is unnecessary
- You need limits on general non-simplified models — use the NRopsDD interface workflow instead

## Inputs

- Mass of the vector mediator (high-energy scale)
- High-energy couplings of the mediator to SM fields (dimension-6 operator coefficients)
- Choice of running-included vs. running-neglected treatment

## Outputs

- Low-energy couplings of the mediator to up, down and strange quarks
- Low-energy couplings to protons and neutrons (direct-detection operator coefficients)
- Comparison table of running-included vs. running-neglected couplings for axial-vector benchmarks

## How to apply

Specify the mediator mass and the high-energy couplings of the mediator to SM fields as inputs to runDM (Python or Mathematica implementation). Run the renormalization-group evolution between the mediator mass scale and the nuclear energy scale with the full dimension-6 operator mixing matrix included, since operator mixing generates new interactions at low energy that can shift bounds by orders of magnitude. Extract the operator coefficients relevant for direct detection — the low-energy couplings to up, down and strange quarks and to protons and neutrons — from the evolved couplings. For axial-vector mediators, compute both running-included and running-neglected couplings and compare: the running can alter spin-dependent DM-nucleon cross-section bounds by a factor of two or more. Validate by reproducing the outputs of the deposited example script and runDM-examples.ipynb notebook for the same input mediator mass and couplings.

## Related tools

- **runDM** (Core code that performs the RG evolution of DM-SM couplings with all dimension-6 operator mixing and extracts direct-detection operator coefficients; available in Mathematica and Python) — https://github.com/bradkav/runDM/
- **Python** (Implementation language of runDM's example script and Jupyter Notebook used to run the evolution and export low-energy couplings)
- **Jupyter Notebook** (Interactive interface to runDM examples (runDM-examples.ipynb) used as the reference output for validation)
- **Mathematica** (Alternative runDM implementation with an example notebook and the interface to NRopsDD for limits on general models)
- **NRopsDD** (External code that runDM interfaces with for obtaining limits on general models from the extracted low-energy couplings)

## Evaluation signals

- Reproduce the outputs of the deposited example script / runDM-examples.ipynb for the same input mediator mass and couplings
- Standalone extraction of low-energy u, d, s quark and proton/neutron couplings matches the runDM reference output within numerical tolerance (output_matches_reference check)
- For axial-vector benchmarks, the ratio of running-included to running-neglected spin-dependent couplings corresponds to a change in direct detection bounds by a factor of two or more
- All dimension-6 operators are included in the evolution (operator mixing present) — omitted-mixing runs should be flagged as running-neglected and treated separately
- Correct Zenodo/GitHub version of runDM is cited (doi:10.5281/zenodo.823249) with the right version number

## Limitations

- Physics details are documented in Appendix B of arXiv:1605.04917 and the manual in doc/runDM-manual.pdf — the README does not expose internal parameters
- Running-induced shifts can change bounds by several orders of magnitude, so running-neglected results are not reliable estimates of uncertainty
- No changelog is available for the repository, making version-diff checks between releases impossible from the README alone

## Evidence

- [readme] By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy, taking into account the mixing of all dimension-6 operators.: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy"
- [intro] The code can also be used to extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons.: "extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons"
- [abstract] For an axial-vector mediator, the running can alter the derived bounds on the spin-dependent DM-nucleon cross section by a factor of two or more.: "For an axial-vector mediator, the running can alter the derived bounds on the spin-dependent DM-nucleon cross section by a factor of two or more."
- [methods] For the Python code, we provide an example script as well as Jupyter Notebook. A static version of the notebook can be viewed [here](http://nbviewer.jupyter.org/github/bradkav/runDM/blob/master/python/runDM-examples.ipynb).: "For the Python code, we provide an example script as well as Jupyter Notebook."
