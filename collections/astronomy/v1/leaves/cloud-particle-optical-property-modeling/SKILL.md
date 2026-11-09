---
name: cloud-particle-optical-property-modeling
description: Use when a transmission/emission spectrum must be generated from a converged
  cloud-atmosphere profile and checked for the 8–10 μm MgSiO3/Mg2SiO4 absorption feature,
  or when cloud particle composition is to be constrained from a mid-infrared spectral
  feature.
when_to_use_negative:
- Cloud-free atmospheres or retrievals where cloud condensate chemistry (Mg-silicate/Fe/SiO2
  condensation reactions) is irrelevant — the whole chain is built around cloud particle
  opacities
- Studies of cloud species not covered by the existing reactions/gibbs data and for
  which Gibbs formation energies cannot be supplied in gibbs.txt
- 'Non-1D or time-dependent cloud modeling needs: ExoLyn is deliberately a ''golden
  mean'' 1D model balancing physical consistency with computational efficiency, not
  a full cloud microphysics code'
tools:
- name: ExoLyn
  role: Solves transport of cloud particles and vapor to produce the layered (Mg-silicate
    over Fe) cloud composition and size distributions that seed the opacity calculation
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates solid opacities from the ExoLyn cloud particle compositions (MgSiO3,
    Mg2SiO4, Fe) and size distributions
- name: petitRADTRANS
  role: Assembles the combined gas + cloud opacity set and generates transmission
    or emission spectra over the mid-IR wavelength grid
- name: sol.f90 (Fortran solver)
  role: Custom Fortran linear-system solver used inside the ExoLyn relaxation run;
    much faster than the numpy solver
  repo: github.com/helonghuangastro/exolyn
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:be6699093b72514887fc7902b9db5b077cf94dd7c1b998024315c214bf77d678
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Python
  - numpy
  - optool
  - petitRADTRANS
  - sol.f90 (Fortran solver)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cloud-particle-optical-property-modeling

## Summary

Generates an exoplanet atmosphere spectrum whose cloud signature reproduces the 8–10 μm Mg-silicate absorption by chaining ExoLyn (multispecies cloud profile), optool (solid opacities), and petitRADTRANS (transmission/emission radiative transfer). It exists to constrain cloud particle composition (MgSiO3, Mg2SiO4, Fe) directly from observed spectra.

## When to use

Use when a transmission/emission spectrum must be generated from a converged cloud-atmosphere profile and checked for the 8–10 μm MgSiO3/Mg2SiO4 absorption feature, or when cloud particle composition is to be constrained from a mid-infrared spectral feature.

## When NOT to use

- Cloud-free atmospheres or retrievals where cloud condensate chemistry (Mg-silicate/Fe/SiO2 condensation reactions) is irrelevant — the whole chain is built around cloud particle opacities
- Studies of cloud species not covered by the existing reactions/gibbs data and for which Gibbs formation energies cannot be supplied in gibbs.txt
- Non-1D or time-dependent cloud modeling needs: ExoLyn is deliberately a 'golden mean' 1D model balancing physical consistency with computational efficiency, not a full cloud microphysics code

## Inputs

- Converged ExoLyn atmosphere profile (layered Mg-silicate clouds over an iron cloud) from the upstream relaxation run
- Cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and size distributions
- ExoLyn configuration files: parameters.txt, reactions.txt, gibbs.txt (Gibbs formation energy data, e.g. from janaf.nist.gov)
- Wavelength grid covering the mid-infrared (must sample the 8–10 μm region)

## Outputs

- Solid opacities for the cloud species calculated by optool
- Combined gas + cloud opacity set for the atmospheric column in petitRADTRANS
- spectrum.dat: a two-column (or more) table of wavelength and transit radius/flux showing the 8–10 μm Mg-silicate absorption feature

## How to apply

First produce the layered cloud structure (magnesium-silicate-dominated clouds on top of an iron cloud) with the upstream ExoLyn relaxation run: compile the Fortran solver with `bash build.sh`, then run `python3 relaxation.py` with parameters.txt, reactions.txt, and gibbs.txt in place. Take the cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and their size distributions from the converged profile and pass them to optool to calculate the solid opacities. Assemble the combined gas + cloud opacity set in petitRADTRANS for the atmospheric column and run it in transmission (or emission) mode over a wavelength grid that covers the mid-infrared so the 8–10 μm region is sampled. Write the result as a wavelength and transit-radius/flux table to spectrum.dat; the rationale is that MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm, so its presence verifies both the cloud composition and the opacity chain.

## Related tools

- **ExoLyn** (Solves transport of cloud particles and vapor to produce the layered (Mg-silicate over Fe) cloud composition and size distributions that seed the opacity calculation) — github.com/helonghuangastro/exolyn
- **optool** (Calculates solid opacities from the ExoLyn cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and size distributions)
- **petitRADTRANS** (Assembles the combined gas + cloud opacity set and generates transmission or emission spectra over the mid-IR wavelength grid)
- **sol.f90 (Fortran solver)** (Custom Fortran linear-system solver used inside the ExoLyn relaxation run; much faster than the numpy solver) — github.com/helonghuangastro/exolyn

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- spectrum.dat exists and is a two-column (or more) wavelength and transit-radius/flux table
- The MgSiO3/Mg2SiO4 absorption feature in the generated spectrum falls in the 8–10 μm range
- optool opacities are produced for all three solid species (MgSiO3, Mg2SiO4, Fe), not a subset
- The petitRADTRANS wavelength grid actually samples the mid-infrared 8–10 μm region
- The ExoLyn relaxation run completes in only a few seconds (efficiency invariant of the 'golden mean' model)

## Limitations

- ExoLyn is a 1D cloud model balancing physical consistency with computational efficiency; it is consistent with, but not a replacement for, more complex cloud formation models
- Adding new reactions requires editing parameters.txt AND implementing Gibbs formation energy data for all involved molecules in gibbs.txt (data available on janaf.nist.gov) — missing data will silently break the chemistry
- The Fortran part must be compiled with build.sh before running; the numpy solver fallback is much slower
- The diagnostic 8–10 μm feature is specific to MgSiO3/Mg2SiO4 clouds; metal-rich planets form SiO2-dominated clouds and C-rich planets form Fe clouds with strong extinction, changing the spectral signature

## Evidence

- [abstract] MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm: "MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm"
- [abstract] ExoLyn combined with optool and petitRADTRANS: "Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities"
- [readme] Compile the Fortran solver before running: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [methods] Custom Fortran linear-system solver: "**sol.f90** is a custom fortran program to numerically solve linear systems."
- [readme] New species require Gibbs energy data in gibbs.txt: "Please also implement gibbs formation energy data for all the molecules involved in the reaction in `gibbs.txt`."
