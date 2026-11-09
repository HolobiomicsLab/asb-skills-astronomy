---
name: renormalization-group-operator-mixing-comparison
description: Use when translating high-energy couplings of a vector (e.g. axial-vector)
  mediator to low-energy DM-nucleon couplings, and you need to assess whether operator
  mixing shifts derived direct detection bounds (e.g. by a factor of two or more for
  SD cross sections).
when_to_use_negative:
- The mediator is not a vector-mediator simplified model, or the effective interaction
  is not describable by dimension-6 operators coupled to SM fields
- You need couplings only at a single scale with no interest in operator mixing or
  the ratio of running-included to running-neglected bounds
- You need an implementation in a language other than Mathematica or Python (the README
  notes the code is only written in these two languages)
tools:
- name: runDM
  role: Evolves the couplings of DM to the SM from the mediator mass scale to low
    energy, taking into account the mixing of all dimension-6 operators, and extracts
    the direct-detection operator coefficients (low-energy couplings to quarks, protons
    and neutrons)
  repo: https://github.com/bradkav/runDM
- name: Jupyter Notebook
  role: Used to execute the runDM example workflow (runDM-examples.ipynb) in Python
  repo: http://nbviewer.jupyter.org/github/bradkav/runDM/blob/master/python/runDM-examples.ipynb
- name: NRopsDD
  role: Optional interface for obtaining limits on general models from the runDM output
    coefficients
  repo: http://www.marcocirelli.net/NROpsDD.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:4424814c75e5c8e1a7a5e45bc10877206b69c4d4f97db9a27397c08d50a34828
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bradkav/runDM
  tools:
  - Jupyter Notebook
  - NRopsDD
  - runDM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# renormalization-group-operator-mixing-comparison

## Summary

Quantifies how RG running and dimension-6 operator mixing alter direct detection bounds in vector-mediator DM simplified models, by evolving couplings from the mediator mass scale to the nuclear energy scale with runDM and comparing running-included versus running-neglected spin-dependent couplings to protons and neutrons.

## When to use

Use when translating high-energy couplings of a vector (e.g. axial-vector) mediator to low-energy DM-nucleon couplings, and you need to assess whether operator mixing shifts derived direct detection bounds (e.g. by a factor of two or more for SD cross sections).

## When NOT to use

- The mediator is not a vector-mediator simplified model, or the effective interaction is not describable by dimension-6 operators coupled to SM fields
- You need couplings only at a single scale with no interest in operator mixing or the ratio of running-included to running-neglected bounds
- You need an implementation in a language other than Mathematica or Python (the README notes the code is only written in these two languages)

## Inputs

- Vector-mediator benchmark point: mediator mass and high-energy couplings of the mediator to SM fields
- List of dimension-6 operator coefficients at the mediator mass scale
- runDM Python installation (clone of github.com/bradkav/runDM) with runDM-examples.ipynb

## Outputs

- Low-energy couplings to up, down and strange quarks and to protons and neutrons, with running included and with running neglected
- coupling_ratios.csv: tabulated running-included / running-neglected SD coupling ratios for protons and neutrons (pandas DataFrame)
- coupling_comparison.png: matplotlib plot comparing running-included and running-neglected couplings/ratios
- Derived SD DM-nucleon cross-section bounds under both treatments

## How to apply

Install runDM from github.com/bradkav/runDM (or Zenodo doi:10.5281/zenodo.823249) and work through the provided Python example script or Jupyter notebook runDM-examples.ipynb. Define the mediator benchmark by specifying the mediator mass and the couplings of the mediator to SM fields at high energy. Evolve the dimension-6 operator couplings from the mediator mass scale down to the nuclear energy scale taking into account the mixing of all dimension-6 operators, extracting low-energy couplings to up, down and strange quarks and to protons and neutrons. Repeat the evaluation with running neglected so couplings are obtained without operator mixing effects. Compute the ratio of running-included to running-neglected spin-dependent couplings for protons and neutrons, tabulate the results (e.g. with pandas as coupling_ratios.csv), and plot both sets with matplotlib (coupling_comparison.png). Validate by checking that the shift in spin-dependent couplings corresponds to a change in derived SD DM-nucleon cross-section bounds by a factor of two or more, matching the paper's axial-vector-mediator finding.

## Related tools

- **runDM** (Evolves the couplings of DM to the SM from the mediator mass scale to low energy, taking into account the mixing of all dimension-6 operators, and extracts the direct-detection operator coefficients (low-energy couplings to quarks, protons and neutrons)) — https://github.com/bradkav/runDM
- **Jupyter Notebook** (Used to execute the runDM example workflow (runDM-examples.ipynb) in Python) — http://nbviewer.jupyter.org/github/bradkav/runDM/blob/master/python/runDM-examples.ipynb
- **NRopsDD** (Optional interface for obtaining limits on general models from the runDM output coefficients) — http://www.marcocirelli.net/NROpsDD.html

## Examples

```
git clone https://github.com/bradkav/runDM; jupyter notebook runDM/python/runDM-examples.ipynb
```

## Evaluation signals

- The ratio of running-included to running-neglected spin-dependent couplings for protons and neutrons is finite, non-unity, and tabulated for the same benchmark mediator mass and high-energy couplings in coupling_ratios.csv
- The derived SD DM-nucleon cross-section bounds differ between the running-included and running-neglected cases by a factor of two or more for the axial-vector mediator, consistent with the paper's reported finding
- The low-energy output includes couplings to up, down and strange quarks and to protons and neutrons, as runDM is designed to extract
- coupling_comparison.png renders both coupling sets (running included vs neglected) on comparable axes for the same benchmark
- The correct runDM version number is cited as 'runDM (Version X.X) [Computer software], doi:10.5281/zenodo.823249'

## Limitations

- The code is written only in Mathematica and Python; other-language implementations must be requested from the authors ('if you want it in Fortran, you better be ready to offer something in return')
- Running-induced shifts can change bounds by several orders of magnitude in some cases, so conclusions are highly sensitive to whether operator mixing is included; neglecting it can be badly wrong rather than a small correction
- The physics is documented in Appendix B of arXiv:1605.04917; consult it for details of the RG evolution between the mediator mass and nuclear energy scales

## Evidence

- [abstract] Inclusion of RG running via operator mixing shifts SD bounds: "For an axial-vector mediator, the running can alter the derived bounds on the spin-dependent DM-nucleon cross section by a factor of two or more."
- [readme] Scope of the runDM calculation: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy, taking into account the mixing of"
- [abstract] Magnitude of mixing effects on bounds: "In some cases the inclusion of these effects changes the bounds by several orders of magnitude, as a consequence of operator mixing which generates new interactions at low energy."
- [methods] Python entry points for the workflow: "For the Python code, we provide an example script as well as Jupyter Notebook."
- [intro] Direct-detection coefficients to extract: "extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and strange quarks and to protons and neutrons"
