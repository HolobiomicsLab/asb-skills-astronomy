---
name: exoplanet-cloud-condensation-modeling
description: Use when you need vertical compositional structure of clouds (e.g., layered
  magnesiumsilicate over iron) in a hot-Jupiter, sub-Neptune, or self-luminous planet
  atmosphere, and require the result in seconds for joint gas+cloud retrieval.
when_to_use_negative:
- The task requires full microphysical cloud formation detail or particle size-distribution
  dynamics that only more complex cloud formation models provide, rather than a computationally
  efficient 1D approximation
- You need spectra or opacities alone with no cloud-structure calculation — use petitRADTRANS
  or optool directly instead
- The atmosphere's chemical network involves molecules whose Gibbs formation energies
  are unavailable on janaf.nist.gov and cannot be supplied in gibbs.txt
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3826
tools:
- name: ExoLyn
  role: 1D cloud model that solves transport equations of cloud particles and vapor
    under self-consistently calculated condensation rates to produce layered cloud
    composition profiles
  repo: github.com/helonghuangastro/exolyn
- name: sol.f90 (Fortran solver)
  role: Custom Fortran program to numerically solve linear systems at each relaxation
    step; much faster than the numpy solver
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates solid opacities from ExoLyn cloud composition output
- name: petitRADTRANS
  role: Generates transmission or emission spectra from the ExoLyn cloud structure
- name: numpy
  role: Alternative (slower) solver backend for the linear systems in place of sol.f90
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:7c108d8ad1286c986f88bac9998438cf7b462373ce448933c1ad6117e5bb3a9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Python
  - bash
  - numpy
  - optool
  - petitRADTRANS
  - sol.f90 (Fortran solver)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3826
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# exoplanet-cloud-condensation-modeling

## Summary

Model multispecies cloud formation in 1D exoplanet atmospheres with ExoLyn, a 1D cloud model that solves coupled transport equations for cloud particles and vapor under self-consistently calculated condensation rates, producing vertical cloud compositional profiles at retrieval-compatible computational cost.

## When to use

Use when you need vertical compositional structure of clouds (e.g., layered magnesiumsilicate over iron) in a hot-Jupiter, sub-Neptune, or self-luminous planet atmosphere, and require the result in seconds for joint gas+cloud retrieval.

## When NOT to use

- The task requires full microphysical cloud formation detail or particle size-distribution dynamics that only more complex cloud formation models provide, rather than a computationally efficient 1D approximation
- You need spectra or opacities alone with no cloud-structure calculation — use petitRADTRANS or optool directly instead
- The atmosphere's chemical network involves molecules whose Gibbs formation energies are unavailable on janaf.nist.gov and cannot be supplied in gibbs.txt

## Inputs

- parameters.txt (python-like configuration file defining reactions, gas and solid species, and model parameters)
- reactions.txt (chemical reaction list)
- gibbs.txt (Gibbs formation energies for all molecules involved in the reactions, sourced from janaf.nist.gov)
- 1D atmospheric temperature-pressure profile

## Outputs

- Steady-state condensed-species mass and number densities per altitude/pressure level (e.g., cloud_composition_profile.csv)
- Vertical cloud composition profile showing layered structure (e.g., MgSiO3/Mg2SiO4 layer over an Fe cloud)
- Optional plots via draw.py
- Solid opacities via optool and transmission/emission spectra via petitRADTRANS when coupled

## How to apply

Compile the Fortran linear solver with `bash build.sh` so sol.f90 (faster than the numpy solver) is available for solving the linear systems at each relaxation step. Configure chemistry by editing parameters.txt (python-like syntax, # for comments) to set reactions and gas/solid species lists, and ensure Gibbs formation energies for all molecules in reactions.txt are present in gibbs.txt, sourced from janaf.nist.gov. Run the relaxation driver with `python3 relaxation.py`, which reads parameters.txt via parameters.py, loads reactions.txt and gibbs.txt via read.py, and initializes domain size and initial equilibrium concentrations via init.py. ExoLyn then solves the coupled transport equations for cloud particles and vapor under cloud condensation rates self-consistently calculated from thermodynamics. Once converged, tabulate steady-state condensed-species mass/number densities per altitude/pressure level (optionally save as cloud_composition_profile.csv) and visualize with draw.py; for spectra, combine ExoLyn output with optool for solid opacities and petitRADTRANS for transmission or emission spectra.

## Related tools

- **ExoLyn** (1D cloud model that solves transport equations of cloud particles and vapor under self-consistently calculated condensation rates to produce layered cloud composition profiles) — github.com/helonghuangastro/exolyn
- **sol.f90 (Fortran solver)** (Custom Fortran program to numerically solve linear systems at each relaxation step; much faster than the numpy solver) — github.com/helonghuangastro/exolyn
- **optool** (Calculates solid opacities from ExoLyn cloud composition output)
- **petitRADTRANS** (Generates transmission or emission spectra from the ExoLyn cloud structure)
- **numpy** (Alternative (slower) solver backend for the linear systems in place of sol.f90)

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- Output profile reproduces the reported layered hot-Jupiter structure: a magnesiumsilicate-dominated cloud (MgSiO3/Mg2SiO4) on top of an iron cloud
- The solution converges under the relaxation driver, with steady-state condensed-species mass/number densities tabulated per altitude/pressure level
- run completed with the sol.f90 Fortran solver available (built via build.sh) rather than falling back to the slower numpy solver
- Results are consistent with more complex cloud formation models while obtained in only a few seconds
- For spectral validation, MgSiO3 and Mg2SiO4 components produce the expected absorption feature at 8–10 μm

## Limitations

- It is a 1D model balancing physical consistency with computational efficiency ('golden mean'), not a full multi-dimensional microphysical treatment
- Gibbs formation energies must be manually supplied in gibbs.txt for every molecule in the reactions; missing JANAF data blocks the chemistry setup
- Cloud composition depends on planet metallicity assumptions: SiO2-dominated clouds form on metal-rich planets and Fe clouds with strong extinction form on C-rich planets, so a hot-Jupiter validation case may not transfer directly

## Evidence

- [methods] Compile the Fortran solver: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [readme] Configure chemistry in parameters.txt and gibbs.txt: "Please also implement gibbs formation energy data for all the molecules involved in the reaction in `gibbs.txt`. You may find these data on janaf.nist.gov."
- [readme] Run the relaxation driver: "After compilation, you can run the code by `python3 relaxation.py`"
- [abstract] ExoLyn core physics: "ExoLyn solves the transport equation of cloud particles and vapor under cloud condensation rates that are self-consistently calculated from thermodynamics"
- [abstract] Expected layered output: "the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud"
