---
name: dimension-6-operator-mixing-analysis
description: Use when you have a vector-mediator simplified model defined by the mediator
  mass and high-energy couplings to SM fields, and you need low-energy couplings to
  u, d, s quarks and protons/neutrons for direct detection.
when_to_use_negative:
- The model is not a vector-mediator simplified DM model (e.g. scalar or fermion mediators)
  — runDM's RG evolution is built for dimension-6 operators in vector-mediator simplified
  models
- You already have the low-energy couplings and only need direct-detection limits
  — use a direct-detection code (e.g. interface to NRopsDD) rather than re-running
  the evolution
- You need RG running between arbitrary scales in a full UV-complete model rather
  than the mediator-mass-to-nuclear-scale running of vector-mediator couplings
edam_operation: http://edamontology.org/operation_2476
edam_topics: []
tools:
- name: runDM
  role: Computes the running of DM-SM couplings in vector-mediator simplified models,
    mixing all dimension-6 operators between the mediator mass scale and the nuclear
    scale
  repo: https://github.com/bradkav/runDM
- name: Python
  role: Language of the runDM example script and Jupyter Notebook used to run the
    evolution and extract couplings
- name: Jupyter Notebook
  role: Interactive interface (runDM-examples.ipynb) for specifying inputs and running
    the RG evolution
- name: Mathematica
  role: Alternative implementation; example notebook provided, including interfacing
    with NRopsDD
- name: NRopsDD
  role: Optional downstream interface for obtaining limits on general models from
    the runDM low-energy coefficients
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:b60f025f78885036ca655d3dc2dfd141585be5234c65cc5493e8f5b03a6aeff0
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
  edam_operation: http://edamontology.org/operation_2476
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dimension-6-operator-mixing-analysis

## Summary

Evolve vector-mediator couplings from the mediator mass scale down to the nuclear energy scale using runDM, which runs the high-energy SM couplings while taking into account the mixing of all dimension-6 operators, and extract the low-energy direct-detection operator coefficients.

## When to use

Use when you have a vector-mediator simplified model defined by the mediator mass and high-energy couplings to SM fields, and you need low-energy couplings to u, d, s quarks and protons/neutrons for direct detection.

## When NOT to use

- The model is not a vector-mediator simplified DM model (e.g. scalar or fermion mediators) — runDM's RG evolution is built for dimension-6 operators in vector-mediator simplified models
- You already have the low-energy couplings and only need direct-detection limits — use a direct-detection code (e.g. interface to NRopsDD) rather than re-running the evolution
- You need RG running between arbitrary scales in a full UV-complete model rather than the mediator-mass-to-nuclear-scale running of vector-mediator couplings

## Inputs

- Mass of the vector mediator
- High-energy couplings of the mediator to SM fields (SM quarks and leptons)
- runDM example Python script / runDM-examples.ipynb (or Mathematica example notebook)

## Outputs

- Low-energy couplings of the mediator to up, down and strange quarks
- Low-energy couplings to protons and neutrons (direct-detection operator coefficients)
- Table of low-energy couplings (e.g. low_energy_couplings.csv)

## How to apply

Obtain runDM (MIT-licensed, on Zenodo doi:10.5281/zenodo.823249, written in both Mathematica and Python) and load its example Python script or runDM-examples.ipynb notebook. Specify the mass of the mediator and the couplings of the mediator to SM fields at high energy as inputs, then execute the renormalization-group evolution to the low-energy (nuclear) scale; the rationale is that SM-loop effects mix all dimension-6 operators, generating new interactions at low energy that can shift direct detection bounds by several orders of magnitude. Extract the resulting operator coefficients relevant for direct detection, namely the low-energy couplings to up, down and strange quarks and to protons and neutrons, and save them as a table (e.g. low_energy_couplings.csv). Validate by reproducing the outputs of the deposit's example script/notebook for the same input mediator mass and couplings; for general-model limits you can additionally interface with the NRopsDD code. Physics details are documented in Appendix B of arXiv:1605.04917 and in doc/runDM-manual.pdf.

## Related tools

- **runDM** (Computes the running of DM-SM couplings in vector-mediator simplified models, mixing all dimension-6 operators between the mediator mass scale and the nuclear scale) — https://github.com/bradkav/runDM
- **Python** (Language of the runDM example script and Jupyter Notebook used to run the evolution and extract couplings)
- **Jupyter Notebook** (Interactive interface (runDM-examples.ipynb) for specifying inputs and running the RG evolution)
- **Mathematica** (Alternative implementation; example notebook provided, including interfacing with NRopsDD)
- **NRopsDD** (Optional downstream interface for obtaining limits on general models from the runDM low-energy coefficients)

## Examples

```
import runDM; couplings_low = runDM.RunCouplings(mediator_mass, couplings_high)  # see runDM-examples.ipynb / example script
```

## Evaluation signals

- Reproducing the outputs of runDM-examples.ipynb / the example script for the same input mediator mass and couplings matches the extracted low-energy couplings
- The run accounts for the mixing of all dimension-6 operators (not a single-operator approximation)
- Extracted coefficients cover all direct-detection-relevant couplings: up, down, strange quarks and protons and neutrons
- The Zenodo DOI (10.5281/zenodo.823249) and correct version number are cited when runDM is used
- Sanity check that operator mixing generates non-zero low-energy couplings in channels absent at high energy, as expected from SM-loop effects

## Limitations

- Omitting RG running can change direct detection bounds on the mediator mass by several orders of magnitude, and for an axial-vector mediator the spin-dependent DM-nucleon cross-section bounds by a factor of two or more — results without running are not comparable
- The code is only provided in Mathematica and Python; other-language implementations require contacting the authors
- Physics assumptions are described in Appendix B of arXiv:1605.04917; the tool targets vector-mediator simplified models
- No changelog found for the repository

## Evidence

- [readme] Tool computes low-energy couplings including all dimension-6 operator mixing: "the code can be used to calculate the couplings at low energy, taking into account the mixing of all dimension-6 operators"
- [readme] Tool inputs: mediator mass and high-energy couplings: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy"
- [methods] Example script and notebook provided for Python: "For the Python code, we provide an example script as well as Jupyter Notebook."
- [abstract] Operator mixing shifts direct detection bounds by orders of magnitude: "In some cases the inclusion of these effects changes the bounds by several orders of magnitude, as a consequence of operator mixing which generates new interactions at low energy."
- [abstract] Running between mediator mass and nuclear scales: "We account for the evolution of the couplings between the energy scale of the mediator mass and the nuclear energy scale."
- [methods] Interface with NRopsDD for limits on general models: "an example of how to interface with the [NRopsDD code](http://www.marcocirelli.net/NROpsDD.html) for obtaining limits on general models"
