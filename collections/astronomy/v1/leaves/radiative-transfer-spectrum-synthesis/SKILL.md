---
name: radiative-transfer-spectrum-synthesis
description: Use when you have a converged 1D atmosphere/cloud profile (e.g. layered
  Mg-silicate clouds over an iron cloud from an ExoLyn relaxation run) and need to
  synthesize a mid-infrared spectrum to check or constrain cloud particle composition.
when_to_use_negative:
- You need the cloud structure itself rather than a spectrum — run the ExoLyn relaxation
  model instead (this skill consumes its converged profile).
- The atmosphere is cloud-free gas-only chemistry, where the cloud opacity contribution
  via optool is unnecessary.
- The wavelength range of interest lies outside the mid-infrared and does not sample
  the 8–10 μm Mg-silicate absorption diagnostic.
edam_operation: http://edamontology.org/operation_0303
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3321
tools:
- name: ExoLyn
  role: Produces the converged 1D layered cloud profile (Mg-silicate clouds over iron
    cloud) that supplies compositions and size distributions to the spectrum synthesis
  repo: https://github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates solid opacities from the cloud particle compositions and size distributions
- name: petitRADTRANS
  role: Generates transmission or emission spectra using the combined gas + cloud
    opacity set
- name: Python 3
  role: Driver for the ExoLyn relaxation run (relaxation.py) and glue code for the
    opacity/spectrum workflow
- name: numpy
  role: Numerical array handling in the pipeline (the custom Fortran solver sol.f90
    is used instead of the numpy solver for the linear systems)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:fbc1438ddd6abe952ad75bd5ab3c35f374ec763f5445e1aac13fc48e096d66ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/helonghuangastro/exolyn
  tools:
  - ExoLyn
  - Python
  - Python 3
  - numpy
  - optool
  - petitRADTRANS
  edam_operation: http://edamontology.org/operation_0303
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# radiative-transfer-spectrum-synthesis

## Summary

Generates a transmission or emission spectrum of a cloudy exoplanet atmosphere by combining a converged ExoLyn cloud profile with solid opacities from optool and gas+cloud opacities in petitRADTRANS, so that compositional diagnostics such as the 8–10 μm Mg-silicate feature can be reproduced and compared to observations.

## When to use

Use when you have a converged 1D atmosphere/cloud profile (e.g. layered Mg-silicate clouds over an iron cloud from an ExoLyn relaxation run) and need to synthesize a mid-infrared spectrum to check or constrain cloud particle composition.

## When NOT to use

- You need the cloud structure itself rather than a spectrum — run the ExoLyn relaxation model instead (this skill consumes its converged profile).
- The atmosphere is cloud-free gas-only chemistry, where the cloud opacity contribution via optool is unnecessary.
- The wavelength range of interest lies outside the mid-infrared and does not sample the 8–10 μm Mg-silicate absorption diagnostic.

## Inputs

- Converged ExoLyn atmosphere profile with layered Mg-silicate clouds over an iron cloud (from task_001-style relaxation run)
- Cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and size distributions
- ExoLyn configuration inputs: parameters.txt, reactions.txt, gibbs.txt (Gibbs formation energies from janaf.nist.gov)

## Outputs

- spectrum.dat: two-column (or more) wavelength and transit-radius/flux table
- Synthetic spectrum covering the mid-infrared including the 8–10 μm region

## How to apply

Start from the converged ExoLyn atmosphere profile produced by the upstream relaxation run (compiled with `bash build.sh`, executed with `python3 relaxation.py`, parameters set in parameters.txt). Extract the cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and their size distributions from the profile, then feed them to optool to calculate solid opacities. Assemble the combined gas + cloud opacity set in petitRADTRANS and run it in transmission (or emission) mode on a wavelength grid that extends into the mid-infrared so the 8–10 μm region is sampled. Write the resulting spectrum as a wavelength vs transit-radius/flux table (e.g. spectrum.dat). The rationale is that MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm, so the synthesized spectrum directly tests whether the modeled cloud composition matches the reported diagnostic.

## Related tools

- **ExoLyn** (Produces the converged 1D layered cloud profile (Mg-silicate clouds over iron cloud) that supplies compositions and size distributions to the spectrum synthesis) — https://github.com/helonghuangastro/exolyn
- **optool** (Calculates solid opacities from the cloud particle compositions and size distributions)
- **petitRADTRANS** (Generates transmission or emission spectra using the combined gas + cloud opacity set)
- **Python 3** (Driver for the ExoLyn relaxation run (relaxation.py) and glue code for the opacity/spectrum workflow)
- **numpy** (Numerical array handling in the pipeline (the custom Fortran solver sol.f90 is used instead of the numpy solver for the linear systems))

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- spectrum.dat exists and is a valid two-column (or more) wavelength vs transit-radius/flux table
- The MgSiO3/Mg2SiO4 absorption feature falls in the 8–10 μm range as reported in the paper
- The synthesized spectrum is consistent with the layered cloud structure (magnesiumsilicate-dominated clouds on top of an iron cloud) from the converged ExoLyn run
- Solid opacities are computed for all modeled cloud components (MgSiO3, Mg2SiO4, Fe) with their size distributions, not just a single species

## Limitations

- The ExoLyn 1D cloud model is a balance of physical consistency with computational efficiency ('golden mean'), not a full cloud formation model; results are consistent with more complex models but obtained in only a few seconds.
- Solid species must all be listed in parameters.txt and Gibbs formation energies must be supplied in gibbs.txt (from janaf.nist.gov) if reactions are customized — missing entries will break the run.
- petitRADTRANS must be run on a wavelength grid that covers the mid-infrared; a grid truncated below 8 μm will miss the diagnostic feature entirely.

## Evidence

- [abstract] Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities: "Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities"
- [abstract] petitRADTRANS spectra: "and with petitRADTRANS to generate transmission or emission spectra"
- [abstract] 8–10 μm feature: "MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm"
- [abstract] layered clouds: "the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud"
- [readme] compile and run: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [readme] run relaxation: "After compilation, you can run the code by `python3 relaxation.py`"
