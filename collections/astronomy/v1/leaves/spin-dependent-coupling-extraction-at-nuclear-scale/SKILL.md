---
name: spin-dependent-coupling-extraction-at-nuclear-scale
description: Use when a simplified model with a vector mediator requires spin-dependent
  DM-nucleon coupling extraction at low energy, and RG running/operator mixing between
  the mediator mass scale and nuclear scale must be quantified for direct detection
  bounds.
when_to_use_negative:
- The mediator is not a vector mediator / the model is not expressible in terms of
  dimension-6 operators mixing under runDM's RG (e.g. purely leptophilic operators
  outside runDM's basis).
- You only need bounds at the mediator mass scale with no RG evolution to the nuclear
  energy scale — running extraction is unnecessary there.
- A fully general effective theory beyond vector-mediator simplified models is needed;
  the paper instead suggests interfacing with NRopsDD for limits on general models.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: runDM
  role: Evolves the dimension-6 operator couplings from the mediator mass scale to
    the nuclear energy scale with full operator mixing and extracts low-energy couplings
    to quarks, protons and neutrons
  repo: https://github.com/bradkav/runDM
- name: Python
  role: Implementation language for the runDM example script used to run the evolution
    and extraction workflow
- name: Jupyter Notebook
  role: Interactive interface (runDM-examples.ipynb) used to execute the running-included
    vs. running-neglected comparison
- name: NRopsDD
  role: Optional interface for obtaining limits on general models beyond the axial-vector
    benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/jhep08(2016)111
    title: 'You can hide but you have to run: direct detection with vector mediators'
  source_content_hash: sha256:436a564e514d4bbb5830ece133490e70cfffaeb753ddc1ea7ca5b0114a35730b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bradkav/runDM
  tools:
  - Jupyter Notebook
  - NRopsDD
  - Python
  - runDM
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spin-dependent-coupling-extraction-at-nuclear-scale

## Summary

Evolve dimension-6 operator couplings of an axial-vector mediator from the mediator mass scale down to the nuclear energy scale with runDM (including operator mixing), then extract low-energy spin-dependent couplings to protons and neutrons both with and without running to quantify the shift in direct detection bounds.

## When to use

Use when a simplified model with a vector mediator requires spin-dependent DM-nucleon coupling extraction at low energy, and RG running/operator mixing between the mediator mass scale and nuclear scale must be quantified for direct detection bounds.

## When NOT to use

- The mediator is not a vector mediator / the model is not expressible in terms of dimension-6 operators mixing under runDM's RG (e.g. purely leptophilic operators outside runDM's basis).
- You only need bounds at the mediator mass scale with no RG evolution to the nuclear energy scale — running extraction is unnecessary there.
- A fully general effective theory beyond vector-mediator simplified models is needed; the paper instead suggests interfacing with NRopsDD for limits on general models.

## Inputs

- Mediator mass of the axial-vector benchmark point
- High-energy couplings of the mediator to SM fields
- runDM Python example script / Jupyter notebook (runDM-examples.ipynb)

## Outputs

- Low-energy spin-dependent couplings to protons and neutrons with running included (cond-running-included)
- Low-energy spin-dependent couplings with running neglected (cond-running-neglected)
- Ratio table of running-included vs. running-neglected spin-dependent couplings (coupling_ratios.csv)
- Comparison plot of the two sets of couplings/ratios (coupling_comparison.png)

## How to apply

Specify an axial-vector mediator benchmark via its mass and the high-energy couplings of the mediator to SM fields. Use runDM to evolve the dimension-6 operator couplings from the mediator mass scale down to the nuclear energy scale, taking into account the mixing of all dimension-6 operators, and extract the low-energy couplings to up, down and strange quarks and to protons and neutrons (running included). Repeat the same extraction with running neglected so couplings are evaluated without operator mixing effects. Compute the ratio of running-included to running-neglected spin-dependent couplings for protons and neutrons, tabulate them (e.g. with pandas in coupling_ratios.csv), and plot the two sets with matplotlib. Validate that the running-induced shift in spin-dependent couplings corresponds to a change in direct detection bounds by a factor of two or more, matching the paper's axial-vector finding.

## Related tools

- **runDM** (Evolves the dimension-6 operator couplings from the mediator mass scale to the nuclear energy scale with full operator mixing and extracts low-energy couplings to quarks, protons and neutrons) — https://github.com/bradkav/runDM
- **Python** (Implementation language for the runDM example script used to run the evolution and extraction workflow)
- **Jupyter Notebook** (Interactive interface (runDM-examples.ipynb) used to execute the running-included vs. running-neglected comparison)
- **NRopsDD** (Optional interface for obtaining limits on general models beyond the axial-vector benchmark)

## Examples

```
python runDM-examples.py  # or: jupyter notebook runDM-examples.ipynb
```

## Evaluation signals

- The ratio of running-included to running-neglected spin-dependent couplings for protons and neutrons must be tabulated for each benchmark (coupling_ratios.csv exists and is non-empty).
- Validation threshold: the running-induced shift in the derived spin-dependent couplings must correspond to a change in direct detection bounds by a factor of two or more for the axial-vector mediator, matching the paper's reported finding.
- A comparison plot (coupling_comparison.png) showing both the running-included and running-neglected couplings/ratios is produced with matplotlib.
- The evolution explicitly accounts for the mixing of all dimension-6 operators, not just a subset (running-neglected run must show couplings without operator mixing effects).
- The extracted low-energy coefficients are for the operators relevant for direct detection: couplings to up, down and strange quarks and to protons and neutrons.

## Limitations

- The factor-of-two-or-more alteration applies to the axial-vector mediator case; the paper notes other cases where running changes bounds by several orders of magnitude, so magnitude is model-dependent.
- The code is only provided in Mathematica and Python; other-language ports are not available (the authors jokingly note Fortran would require compensation).
- The workflow covers vector-mediator simplified models via dimension-6 operators; general models require interfacing with NRopsDD instead.
- No changelog was found in the repository, so version-specific behavior should be verified; cite the exact runDM version number when publishing.

## Evidence

- [readme] By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy: "By specifying the mass of the mediator and the couplings of the mediator to SM fields at high energy, the code can be used to calculate the couplings at low energy"
- [readme] operator mixing: "taking into account the mixing of all dimension-6 operators. The code can also be used to extract the operator coefficients relevant for direct detection, namely low energy couplings to up, down and"
- [abstract] axial-vector running effect: "For an axial-vector mediator, the running can alter the derived bounds on the spin-dependent DM-nucleon cross section by a factor of two or more."
- [methods] python notebook availability: "For the Python code, we provide an example script as well as Jupyter Notebook."
- [methods] NRopsDD interface: [evidence span withheld — anchor status: not_found]
