---
name: exoplanet-cloud-composition-profile-simulation
description: Use when a study asks how cloud composition and extinction in an exoplanet
  atmosphere change with bulk elemental composition (e.g., metal-rich vs C-rich bulk),
  and requires a fast 1D cloud model with condensed-species profiles per altitude
  layer.
when_to_use_negative:
- The goal is a full multi-dimensional, kinetic cloud-formation simulation rather
  than a 1D, physically consistent but simplified transport model — ExoLyn is a 'golden
  mean' approach balancing consistency with computational efficiency, not a replacement
  for complex cloud models.
- The task requires only gas-phase chemistry or spectra without cloud condensation
  — use a spectral tool like petitRADTRANS alone instead of running the cloud solver.
- The bulk composition requires reactions or species not present in reactions.txt/gibbs.txt
  and you cannot supply the needed Gibbs formation energy data from janaf.nist.gov.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ExoLyn
  role: 1D cloud model solving transport of cloud particles and vapor under self-consistently
    calculated condensation rates; produces the cloud composition profile
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Computes solid opacities from the converged solid cloud results to derive
    the extinction effect of cloud layers (e.g., the Fe cloud under C-rich composition)
- name: petitRADTRANS
  role: Can be combined with ExoLyn output to generate transmission or emission spectra
    of the modeled clouds
- name: sol.f90 (Fortran solver)
  role: Custom Fortran program that numerically solves the linear systems; much faster
    than the numpy solver
  repo: github.com/helonghuangastro/exolyn
- name: relaxation.py / parameters.py / read.py / init.py
  role: 'Python driver scripts: run the model, load parameters.txt, ingest reactions.txt
    and gibbs.txt, and initialize domain size and initial equilibrium concentrations'
  repo: github.com/helonghuangastro/exolyn
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:357ff236ee21f20936849a1aad2ae66fb5fa62c7e2be5f87273a96da459b77ba
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
  - relaxation.py / parameters.py / read.py / init.py
  - sol.f90 (Fortran solver)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# exoplanet-cloud-composition-profile-simulation

## Summary

Run the ExoLyn 1D multispecies cloud model to simulate cloud-particle and vapor transport with self-consistently computed condensation rates, producing altitude-resolved cloud composition profiles for hot-Jupiter-like atmospheres under different bulk elemental compositions (metal-rich, C-rich), and optionally derive cloud opacities with optool.

## When to use

Use when a study asks how cloud composition and extinction in an exoplanet atmosphere change with bulk elemental composition (e.g., metal-rich vs C-rich bulk), and requires a fast 1D cloud model with condensed-species profiles per altitude layer.

## When NOT to use

- The goal is a full multi-dimensional, kinetic cloud-formation simulation rather than a 1D, physically consistent but simplified transport model — ExoLyn is a 'golden mean' approach balancing consistency with computational efficiency, not a replacement for complex cloud models.
- The task requires only gas-phase chemistry or spectra without cloud condensation — use a spectral tool like petitRADTRANS alone instead of running the cloud solver.
- The bulk composition requires reactions or species not present in reactions.txt/gibbs.txt and you cannot supply the needed Gibbs formation energy data from janaf.nist.gov.

## Inputs

- parameters.txt configuration file (python-like syntax; hot-Jupiter-like, metal-rich, or C-rich bulk elemental composition)
- reactions.txt chemical reaction list
- gibbs.txt Gibbs formation energy data for all molecules involved in the reactions
- previously compiled sol.f90 Fortran solver (built via build.sh)

## Outputs

- Converged cloud composition profile (dominant condensed species as a function of altitude), e.g. exported to cloud_composition_profile.csv or c_rich_cloud_profile.csv
- Composition table per atmospheric layer with dominant condensed species per layer
- Opacity/extinction record of the cloud layer (via optool coupling)
- Plots of the cloud structure (draw.py)

## How to apply

Clone ExoLyn and compile the Fortran linear-system solver with 'bash build.sh' (sol.f90 is much faster than the numpy fallback). Edit parameters.txt — a python-like file where expressions can reuse previously defined parameters — to switch the hot-Jupiter-like setup to the target bulk elemental composition, and verify reactions.txt and gibbs.txt contain all gas and solid species, sourcing Gibbs formation energies from janaf.nist.gov where needed. Run 'python3 relaxation.py': parameters.py loads parameters.txt, read.py ingests reactions.txt and gibbs.txt, init.py determines domain size and initial equilibrium concentrations, and sol.f90 solves the linear systems for cloud-particle and vapor transport under self-consistently calculated condensation rates. For the metal-rich case, validate that the converged profile is SiO2-dominated; for the C-rich case, couple the converged solids to optool to compute solid opacities and confirm Fe is the dominant cloud material with strong extinction. Expected results for a hot-Jupiter run are layered clouds with magnesiumsilicates on top of an iron cloud, converging in only a few seconds.

## Related tools

- **ExoLyn** (1D cloud model solving transport of cloud particles and vapor under self-consistently calculated condensation rates; produces the cloud composition profile) — github.com/helonghuangastro/exolyn
- **optool** (Computes solid opacities from the converged solid cloud results to derive the extinction effect of cloud layers (e.g., the Fe cloud under C-rich composition))
- **petitRADTRANS** (Can be combined with ExoLyn output to generate transmission or emission spectra of the modeled clouds)
- **sol.f90 (Fortran solver)** (Custom Fortran program that numerically solves the linear systems; much faster than the numpy solver) — github.com/helonghuangastro/exolyn
- **relaxation.py / parameters.py / read.py / init.py** (Python driver scripts: run the model, load parameters.txt, ingest reactions.txt and gibbs.txt, and initialize domain size and initial equilibrium concentrations) — github.com/helonghuangastro/exolyn

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- cloud_composition_profile.csv exists and the dominant condensed species in the profile is SiO2 for the metal-rich bulk composition run
- c_rich_cloud_profile.csv shows Fe as the dominant cloud material per layer with a strong extinction effect in the optool-derived opacity record
- A standard hot-Jupiter-like run reproduces layered clouds: magnesiumsilicate-dominated clouds on top of an iron cloud
- The run converges in only a few seconds (efficiency invariant of ExoLyn vs complex cloud formation models)
- parameters.txt, reactions.txt, and gibbs.txt jointly contain all gas and solid species of the setup, with Gibbs formation energies present for every molecule in each reaction

## Limitations

- ExoLyn is a 1D model balancing physical consistency with computational efficiency; its results are consistent with more complex cloud formation models but it does not replace them in fidelity.
- Changing the setup requires manual curation: reactions must be edited in parameters.txt and Gibbs formation energy data for all involved molecules must be implemented in gibbs.txt, sourced from janaf.nist.gov.
- The order of species in parameters.txt does not matter, but all gas and solid species of the setup must be present, otherwise the run is incomplete.

## Evidence

- [abstract] ExoLyn solves the transport equation of cloud particles and vapor under cloud condensation rates that are self-consistently calculated from thermodynamics: "ExoLyn solves the transport equation of cloud particles and vapor under cloud condensation rates that are self-consistently calculated from thermodynamics"
- [abstract] SiO2-dominated clouds form on metal-rich planets and Fe clouds with strong extinction form on C-rich planets: "SiO2-dominated clouds form on metal-rich planets and Fe clouds with a strong extinction effect form on C-rich planets"
- [abstract] Layered hot-Jupiter clouds: magnesiumsilicates on top of iron cloud: "the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud"
- [methods] Compile the fortran code with build.sh: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [methods] sol.f90 is much faster than the numpy solver: "**sol.f90** is a custom fortran program to numerically solve linear systems. It's much faster than the numpy solver."
- [readme] Gibbs formation energy data source: "Please also implement gibbs formation energy data for all the molecules involved in the reaction in `gibbs.txt`. You may find these data on janaf.nist.gov."
- [abstract] Consistency with complex models in a few seconds: "This finding is consistent with more complex cloud formation models but can be obtained with ExoLyn in only a few seconds"
