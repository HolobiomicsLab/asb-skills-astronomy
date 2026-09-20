---
name: cloud-condensation-thermodynamic-equilibrium-simulation
description: Use when you need to predict the compositional and vertical structure
  of condensation clouds in a 1D planetary atmosphere for a given bulk elemental composition,
  especially for hot-Jupiter, sub-Neptune, or self-luminous planet retrieval contexts.
when_to_use_negative:
- You need full 3D or microphysically complex cloud treatment rather than a fast 1D
  model balanced for retrieval efficiency
- Your chemical network involves species or reactions whose Gibbs formation energies
  are not available in gibbs.txt and cannot be sourced (e.g., from janaf.nist.gov)
- You only need opacities or spectra without computing the cloud condensation/transport
  structure first
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ExoLyn
  role: 1D multispecies cloud model that solves cloud-particle and vapor transport
    equations under self-consistently calculated condensation rates; the core simulation
    engine
  repo: github.com/helonghuangastro/exolyn
- name: Fortran (sol.f90)
  role: Custom Fortran program that numerically solves the linear systems for cloud-particle
    and vapor transport, much faster than the numpy solver
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates solid opacities from the converged solid cloud results to derive
    cloud-layer extinction
- name: petitRADTRANS
  role: Generates transmission or emission spectra from the ExoLyn cloud results
- name: numpy
  role: Alternative (slower) linear-system solver backend referenced in the package
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:924d2720c0bd70dd4d25b41814fce2963f647423c2f31bcc605af56bedef06fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Fortran (sol.f90)
  - Python
  - bash
  - numpy
  - optool
  - petitRADTRANS
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# cloud-condensation-thermodynamic-equilibrium-simulation

## Summary

Simulate 1D cloud formation in exoplanet atmospheres with ExoLyn, which solves the transport equation of cloud particles and vapor under self-consistently calculated condensation rates to predict layered cloud composition (e.g., Mg-silicates over Fe, SiO2 on metal-rich, Fe on C-rich planets) in seconds rather than with expensive complex cloud models.

## When to use

Use when you need to predict the compositional and vertical structure of condensation clouds in a 1D planetary atmosphere for a given bulk elemental composition, especially for hot-Jupiter, sub-Neptune, or self-luminous planet retrieval contexts.

## When NOT to use

- You need full 3D or microphysically complex cloud treatment rather than a fast 1D model balanced for retrieval efficiency
- Your chemical network involves species or reactions whose Gibbs formation energies are not available in gibbs.txt and cannot be sourced (e.g., from janaf.nist.gov)
- You only need opacities or spectra without computing the cloud condensation/transport structure first

## Inputs

- parameters.txt (python-like configuration: bulk elemental composition, reactions list, atmospheric/domain parameters)
- reactions.txt (chemical reaction definitions for gas and solid species)
- gibbs.txt (Gibbs formation energy data for all molecules involved in the reactions, e.g., from janaf.nist.gov)

## Outputs

- Converged cloud composition profile (dominant condensed species per atmospheric layer as a function of altitude, e.g., exported as cloud_composition_profile.csv or c_rich_cloud_profile.csv)
- Solid opacities/extinction record when coupled to optool (e.g., Fe cloud layer extinction)
- Transmission or emission spectra when coupled to petitRADTRANS

## How to apply

Compile the Fortran linear-system solver by running 'bash build.sh' on the ExoLyn package, then execute 'python3 relaxation.py'. parameters.py reads parameters.txt (a python-like file where expressions may reuse previously defined parameters); read.py ingests chemical data (reactions.txt and gibbs.txt); init.py determines domain size and initial equilibrium concentrations; and sol.f90 solves the linear systems for cloud-particle and vapor transport with self-consistently computed condensation rates. To change the scenario, edit reactions and bulk elemental composition in parameters.txt, ensuring all gas and solid species are listed there and that gibbs.txt contains Gibbs formation energy data for every molecule in the reactions (available from janaf.nist.gov). Run to convergence for the target composition (e.g., metal-rich for SiO2-dominated clouds, C-rich for Fe clouds with strong extinction), and optionally couple converged solid cloud results to optool for solid opacities or petitRADTRANS for transmission/emission spectra.

## Related tools

- **ExoLyn** (1D multispecies cloud model that solves cloud-particle and vapor transport equations under self-consistently calculated condensation rates; the core simulation engine) — github.com/helonghuangastro/exolyn
- **Fortran (sol.f90)** (Custom Fortran program that numerically solves the linear systems for cloud-particle and vapor transport, much faster than the numpy solver) — github.com/helonghuangastro/exolyn
- **optool** (Calculates solid opacities from the converged solid cloud results to derive cloud-layer extinction)
- **petitRADTRANS** (Generates transmission or emission spectra from the ExoLyn cloud results)
- **numpy** (Alternative (slower) linear-system solver backend referenced in the package)

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- relaxation.py completes with a converged solution from sol.f90 (faster than the numpy solver path)
- Exported composition profile CSV exists and the dominant condensed species matches expectations for the chosen bulk composition: SiO2 for metal-rich, Fe for C-rich, layered Mg-silicates over Fe for hot-Jupiter-like composition
- All gas and solid species in the setup are contained in parameters.txt, and gibbs.txt contains Gibbs formation energies for every molecule in the reactions
- MgSiO3/Mg2SiO4 components produce the expected 8–10 μm absorption feature in downstream opacity/spectrum calculations
- Results are consistent with more complex cloud formation models and obtained within seconds

## Limitations

- The model is 1D; it balances physical consistency with computational efficiency rather than capturing full multidimensional cloud dynamics
- Requires manual curation of reactions.txt and gibbs.txt; missing Gibbs data (janaf.nist.gov) or omitted species in parameters.txt will break the setup — species order in parameters.txt does not matter, but completeness does
- Cloud structure is sensitive to bulk elemental composition; extrapolating a hot-Jupiter-like configuration to C-rich or metal-rich regimes requires explicit parameter changes in parameters.txt

## Evidence

- [abstract] ExoLyn solves the transport equation of cloud particles and vapor under cloud condensation rates that are self-consistently calculated from thermodynamics: "ExoLyn solves the transport equation of cloud particles and vapor under cloud condensation rates that are self-consistently calculated from thermodynamics"
- [abstract] C-rich/metal-rich composition dependence: "SiO2-dominated clouds form on metal-rich planets and Fe clouds with a strong extinction effect form on C-rich planets"
- [readme] code structure: "**sol.f90** is a custom fortran program to numerically solve linear systems. It's much faster than the numpy solver."
- [readme] adding reactions: "Please also implement gibbs formation energy data for all the molecules involved in the reaction in `gibbs.txt`. You may find these data on janaf.nist.gov."
- [readme] customization: "You can change parameters by modifying parameters.txt. This is a python-like .txt file. You can write down the expressions without calculating yourself."
