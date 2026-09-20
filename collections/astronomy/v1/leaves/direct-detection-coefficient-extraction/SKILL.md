---
name: direct-detection-coefficient-extraction
description: 'Use when you have runDM-evolved low-energy couplings for a vector mediator
  and need the dimension-6 operator coefficients for direct detection: couplings to
  u, d, s quarks and to protons and neutrons.'
when_to_use_negative:
- The model is not a simplified vector-mediator DM model — runDM's evolution and extraction
  are specific to dimension-6 operators in vector-mediator simplified models
- You need couplings at a scale other than the low-energy/nuclear direct-detection
  scale or operators beyond dimension-6
- You only need high-energy couplings translated between scales without the direct-detection
  quark/nucleon coefficient extraction
edam_operation: http://edamontology.org/operation_2480
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: runDM
  role: Performs the RG evolution mixing all dimension-6 operators and provides the
    routine that extracts the direct-detection operator coefficients (u, d, s quark
    and proton/neutron couplings); implemented in Python and Mathematica
  repo: https://github.com/bradkav/runDM/
- name: Python
  role: Language of the example script and Jupyter Notebook used to run the evolution
    and extraction, and to write low_energy_couplings.csv
- name: Jupyter Notebook
  role: runDM-examples.ipynb supplies the reference example inputs (mediator mass,
    high-energy couplings) and reference outputs for validating the extraction
  repo: http://nbviewer.jupyter.org/github/bradkav/runDM/blob/master/python/runDM-examples.ipynb
- name: NRopsDD
  role: Downstream code the extracted coefficients can be interfaced with to obtain
    limits on general models
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:c0dd383e7cf2901a45168dd587eb6b527773b1cf8cff1ae315aad3dc1634ad16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bradkav/runDM
  tools:
  - Jupyter Notebook
  - NRopsDD
  - Python
  - runDM
  edam_operation: http://edamontology.org/operation_2480
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# direct-detection-coefficient-extraction

## Summary

Extracts the operator coefficients relevant for direct detection from runDM's RG-evolved vector-mediator couplings, mapping them onto low-energy couplings to up, down and strange quarks and to protons and neutrons for use in direct-detection limit calculations.

## When to use

Use when you have runDM-evolved low-energy couplings for a vector mediator and need the dimension-6 operator coefficients for direct detection: couplings to u, d, s quarks and to protons and neutrons.

## When NOT to use

- The model is not a simplified vector-mediator DM model — runDM's evolution and extraction are specific to dimension-6 operators in vector-mediator simplified models
- You need couplings at a scale other than the low-energy/nuclear direct-detection scale or operators beyond dimension-6
- You only need high-energy couplings translated between scales without the direct-detection quark/nucleon coefficient extraction

## Inputs

- Mediator mass and high-energy couplings of the mediator to SM fields (as in the runDM example script / runDM-examples.ipynb)
- runDM-evolved low-energy coupling matrix (output of the RG evolution over dimension-6 operators)

## Outputs

- Direct-detection operator coefficients: low-energy couplings to up, down and strange quarks and to protons and neutrons
- Table of extracted coefficients (e.g. low_energy_couplings.csv)

## How to apply

First obtain the mediator mass and high-energy couplings of the mediator to SM fields (e.g. from the example inputs in runDM-examples.ipynb), then run the runDM RG evolution down to the nuclear scale, which takes into account the mixing of all dimension-6 operators. Next implement or invoke runDM's extraction routine that maps the evolved couplings onto the direct-detection operator coefficients — the low-energy couplings to up, down and strange quarks and to protons and neutrons. Write the extracted coefficients to a table (e.g. low_energy_couplings.csv) so they can feed direct-detection limit code such as NRopsDD. Validate by comparing the standalone extraction value-by-value against runDM's reference output on the same inputs, accepting the implementation when all u, d, s, proton and neutron coefficients match within numerical tolerance.

## Related tools

- **runDM** (Performs the RG evolution mixing all dimension-6 operators and provides the routine that extracts the direct-detection operator coefficients (u, d, s quark and proton/neutron couplings); implemented in Python and Mathematica) — https://github.com/bradkav/runDM/
- **Python** (Language of the example script and Jupyter Notebook used to run the evolution and extraction, and to write low_energy_couplings.csv)
- **Jupyter Notebook** (runDM-examples.ipynb supplies the reference example inputs (mediator mass, high-energy couplings) and reference outputs for validating the extraction) — http://nbviewer.jupyter.org/github/bradkav/runDM/blob/master/python/runDM-examples.ipynb
- **NRopsDD** (Downstream code the extracted coefficients can be interfaced with to obtain limits on general models)

## Examples

```
python runDM-examples.py  # specify mediator mass and high-energy SM couplings, evolve to low energy, write low_energy_couplings.csv
```

## Evaluation signals

- Standalone extracted coefficients match runDM's reference output (from the example script / runDM-examples.ipynb) value-by-value for all u, d, s, proton and neutron coefficients within numerical tolerance (output_matches_reference check)
- Reproducing the deposit's example notebook outputs for the same input mediator mass and couplings
- The output table (low_energy_couplings.csv) has entries for each of the five coefficient types: up, down, strange quarks, protons, neutrons
- Including RG running/mixing effects visibly changes the couplings relative to unrun values — literature states bounds can shift by several orders of magnitude for vector mediators and by a factor of two or more for axial-vector spin-dependent cross sections, so a null effect should be suspicious

## Limitations

- Physics details are documented only in Appendix B of arXiv:1605.04917 and in doc/runDM-manual.pdf; consult these for the operator basis and thresholds
- The correct Zenodo version number must be cited (doi:10.5281/zenodo.823249) when using the code in published work
- No changelog is present in the repo discussion, so version-to-version behavioural changes are not documented

## Evidence

- [readme] The code can also be used to extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons.: "The code can also be used to extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons."
- [intro] By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy"
- [methods] For the Python code, we provide an example script as well as Jupyter Notebook.: "For the Python code, we provide an example script as well as Jupyter Notebook."
- [readme] Further details about the physics behind the code can be found in Appendix B of [arXiv:1605.04917](http://arxiv.org/abs/1605.04917).: "Further details about the physics behind the code can be found in Appendix B of [arXiv:1605.04917](http://arxiv.org/abs/1605.04917)."
