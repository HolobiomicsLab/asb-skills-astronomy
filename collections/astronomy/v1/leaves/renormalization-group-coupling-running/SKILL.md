---
name: renormalization-group-coupling-running
description: Use when you have high-energy couplings of DM to SM fields in a vector-mediator
  simplified model plus a mediator mass, and you need low-energy direct-detection
  operator coefficients or RG-mixing-corrected cross section bounds.
when_to_use_negative:
- The model is not a simplified model with a vector mediator (runDM handles the running
  of DM couplings to SM fields specifically in vector-mediator simplified models).
- You only need UV-level couplings and will not translate them to low energy — RG
  running is unnecessary and operator mixing would obscure the comparison.
- You need direct-detection limits on general models rather than the running/extraction
  step itself — the README points to interfacing with the NRopsDD code for that purpose.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0601
tools:
- name: runDM
  role: 'Primary tool: evolves DM–SM mediator couplings between high energy and low
    energy with dimension-6 operator mixing, and extracts direct-detection operator
    coefficients'
  repo: https://github.com/bradkav/runDM
- name: Python
  role: Language implementation of runDM used with an example script and Jupyter Notebook
    to run the evolution and extraction
  repo: https://github.com/bradkav/runDM
- name: Jupyter Notebook
  role: Example notebook (runDM-examples.ipynb) demonstrating loading high-energy
    couplings and running the evolution/extraction
  repo: https://github.com/bradkav/runDM
- name: Mathematica
  role: Alternative runDM implementation with an example notebook, including an interface
    example to NRopsDD
  repo: https://github.com/bradkav/runDM
- name: NRopsDD
  role: Downstream code interfaced from the Mathematica example to obtain limits on
    general models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:63027492541757cc50d41aea750a6264f09a507cb5867aab4fc1550f66a7fc33
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# renormalization-group-coupling-running

## Summary

Evolve dark matter–Standard Model mediator couplings between the mediator mass scale and the nuclear energy scale using runDM, accounting for mixing of all dimension-6 operators, and extract the low-energy operator coefficients relevant for direct detection (couplings to up, down and strange quarks and to protons and neutrons).

## When to use

Use when you have high-energy couplings of DM to SM fields in a vector-mediator simplified model plus a mediator mass, and you need low-energy direct-detection operator coefficients or RG-mixing-corrected cross section bounds.

## When NOT to use

- The model is not a simplified model with a vector mediator (runDM handles the running of DM couplings to SM fields specifically in vector-mediator simplified models).
- You only need UV-level couplings and will not translate them to low energy — RG running is unnecessary and operator mixing would obscure the comparison.
- You need direct-detection limits on general models rather than the running/extraction step itself — the README points to interfacing with the NRopsDD code for that purpose.

## Inputs

- Mediator mass
- High-energy couplings of the mediator to SM fields
- runDM example inputs from runDM-examples.ipynb (high-energy mediator couplings and mediator mass)

## Outputs

- Low-energy evolved couplings of DM to SM fields
- Direct-detection operator coefficients: low-energy couplings to up, down and strange quarks and to protons and neutrons (e.g. written to low_energy_couplings.csv)

## How to apply

Specify the mediator mass and the high-energy couplings of the mediator to SM fields as inputs to runDM, which evolves the couplings between the mediator mass scale and the nuclear energy scale while taking into account the mixing of all dimension-6 operators. Use the Python implementation (example script or the runDM-examples.ipynb Jupyter Notebook) or the Mathematica notebook; physics details are in Appendix B of arXiv:1605.04917. After evolution, run runDM's extraction step to obtain the operator coefficients relevant for direct detection: low-energy couplings to up, down and strange quarks and to protons and neutrons. To validate a reimplementation, run the deposited runDM code on the same example inputs as reference, then compare all u, d, s, proton and neutron coefficients value-by-value within numerical tolerance (an output_matches_reference check).

## Related tools

- **runDM** (Primary tool: evolves DM–SM mediator couplings between high energy and low energy with dimension-6 operator mixing, and extracts direct-detection operator coefficients) — https://github.com/bradkav/runDM
- **Python** (Language implementation of runDM used with an example script and Jupyter Notebook to run the evolution and extraction) — https://github.com/bradkav/runDM
- **Jupyter Notebook** (Example notebook (runDM-examples.ipynb) demonstrating loading high-energy couplings and running the evolution/extraction) — https://github.com/bradkav/runDM
- **Mathematica** (Alternative runDM implementation with an example notebook, including an interface example to NRopsDD) — https://github.com/bradkav/runDM
- **NRopsDD** (Downstream code interfaced from the Mathematica example to obtain limits on general models)

## Examples

```
python runDM-examples.ipynb  # or: from runDM import runDM; couplings_low = runDM.getCouplings(couplings_high, mMed); coeffs = runDM.extractCoefficients(couplings_low)
```

## Evaluation signals

- Standalone low-energy couplings match the runDM reference output value-by-value within numerical tolerance for all u, d, s quark and proton/neutron coefficients (output_matches_reference check).
- All five coefficient sets (up, down, strange, proton, neutron) are present in the output; missing any one indicates an incomplete extraction step.
- RG running effects reproduce the expected physical signature: operator mixing generates new low-energy interactions, and bounds can shift by several orders of magnitude for mediator mass (factor of two or more for axial-vector spin-dependent DM-nucleon cross sections).
- The evolution spans the correct scale range, from the mediator mass scale down to the nuclear energy scale.
- The code version used is recorded, since the README requires citing runDM 'making sure to include the correct version number' (Zenodo doi:10.5281/zenodo.823249).

## Limitations

- runDM is restricted to simplified models with vector mediators and dimension-6 operator mixing; physics details are documented in Appendix B of arXiv:1605.04917 and in doc/runDM-manual.pdf.
- No changelog was found in the repository, so version tracking must rely on the Zenodo DOI and manual version numbering.
- The code is currently available only in Mathematica and Python; other-language ports depend on author availability ('if you want it in Fortran, you better be ready to offer something in return').

## Evidence

- [intro] runDM is a tool for calculating the running of the couplings of Dark Matter (DM) to the Standard Model (SM) in simplified models with vector mediators.: "`runDM` is a tool for calculating the running of the couplings of Dark Matter (DM) to the Standard Model (SM) in simplified models with vector mediators."
- [readme] By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy"
- [readme] extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons: "The code can also be used to extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons."
- [methods] For the Python code, we provide an example script as well as Jupyter Notebook.: "For the Python code, we provide an example script as well as Jupyter Notebook."
- [abstract] In some cases the inclusion of these effects changes the bounds by several orders of magnitude, as a consequence of operator mixing which generates new interactions at low energy.: "In some cases the inclusion of these effects changes the bounds by several orders of magnitude, as a consequence of operator mixing which generates new interactions at low energy."
