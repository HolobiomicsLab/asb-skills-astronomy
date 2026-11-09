---
name: exoplanet-transmission-spectrum-generation
description: Use when an atmospheric column with layered Mg-silicate clouds atop an
  iron cloud must be converted into a synthetic spectrum, and the 8–10 μm Mg-silicate
  absorption feature must be reproduced or checked against observed mid-infrared data.
when_to_use_negative:
- When the atmosphere is cloud-free and only gas opacities are needed — no optool
  solid opacities or cloud size distributions are involved
- When the goal is the cloud profile itself rather than a spectrum — the ExoLyn relaxation
  run (task upstream of this skill) already produces that in seconds
tools:
- name: ExoLyn
  role: Produces the converged 1D layered cloud atmosphere profile (Mg-silicate clouds
    atop an iron cloud) that is the input column for the spectral calculation
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates the solid opacities from the cloud particle compositions and size
    distributions
- name: petitRADTRANS
  role: Generates the transmission or emission spectrum from the combined gas + cloud
    opacity set
- name: numpy
  role: Numerical handling of the wavelength grid, opacity arrays, and spectrum output
    table in the Python workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:be5985360dc2b485e4ec9116a409f2a01a7fc72a372176aef4b541c8e67e699d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Python
  - numpy
  - optool
  - petitRADTRANS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# exoplanet-transmission-spectrum-generation

## Summary

Generates an exoplanet transmission or emission spectrum from a converged ExoLyn cloud-atmosphere profile by combining solid opacities (via optool) with gas opacities in petitRADTRANS, so that the 8–10 μm MgSiO3/Mg2SiO4 absorption feature constrains cloud particle composition.

## When to use

Use when an atmospheric column with layered Mg-silicate clouds atop an iron cloud must be converted into a synthetic spectrum, and the 8–10 μm Mg-silicate absorption feature must be reproduced or checked against observed mid-infrared data.

## When NOT to use

- When the atmosphere is cloud-free and only gas opacities are needed — no optool solid opacities or cloud size distributions are involved
- When the goal is the cloud profile itself rather than a spectrum — the ExoLyn relaxation run (task upstream of this skill) already produces that in seconds

## Inputs

- Converged ExoLyn atmosphere profile (layered Mg-silicate clouds over an iron cloud) from the upstream relaxation run
- Cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and their size distributions
- Gas opacity set for the atmospheric column
- Mid-infrared-covering wavelength grid (must include the 8–10 μm range)

## Outputs

- Synthetic transmission or emission spectrum written as spectrum.dat (a wavelength and transit-radius/flux table)
- Mid-infrared spectral segment containing the 8–10 μm Mg-silicate absorption feature

## How to apply

Start from the converged atmosphere profile produced by the upstream ExoLyn relaxation run (compile the Fortran part with `bash build.sh`, then run `python3 relaxation.py`), which yields layered Mg-silicate (MgSiO3, Mg2SiO4) clouds over an iron cloud along with particle size distributions. Feed the cloud particle compositions and size distributions into optool to calculate the solid opacities. Assemble the combined gas + cloud opacity set in petitRADTRANS for the atmospheric column, then run petitRADTRANS in transmission or emission mode on a wavelength grid that samples the mid-infrared so the 8–10 μm region is covered. Write the result as a wavelength vs transit-radius/flux table (spectrum.dat) and verify the Mg-silicate absorption falls in the 8–10 μm range, since MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm.

## Related tools

- **ExoLyn** (Produces the converged 1D layered cloud atmosphere profile (Mg-silicate clouds atop an iron cloud) that is the input column for the spectral calculation) — github.com/helonghuangastro/exolyn
- **optool** (Calculates the solid opacities from the cloud particle compositions and size distributions)
- **petitRADTRANS** (Generates the transmission or emission spectrum from the combined gas + cloud opacity set)
- **numpy** (Numerical handling of the wavelength grid, opacity arrays, and spectrum output table in the Python workflow)

## Examples

```
python3 relaxation.py
```

## Evaluation signals

- spectrum.dat exists and is a valid two-column (or more) wavelength / transit-radius or flux table
- The MgSiO3/Mg2SiO4 absorption feature falls within the 8–10 μm range as reported in the paper
- The modeled cloud structure matches the article's finding of magnesium-silicate-dominated clouds layered on top of an iron cloud
- The wavelength grid actually samples the mid-infrared (8–10 μm points present in spectrum.dat)
- The full relaxation-plus-spectrum chain completes on the order of seconds, consistent with ExoLyn's efficiency claim

## Limitations

- ExoLyn is a 1D cloud model that balances physical consistency with computational efficiency (a 'golden mean'), so detailed cloud microphysics beyond its self-consistent condensation/transport treatment is not captured
- The Fortran solver must be compiled first (bash build.sh) before running; skipping compilation causes failure
- Custom reactions require Gibbs formation energy data for all molecules in gibbs.txt (available on janaf.nist.gov) and all gas and solid species must be listed in parameters.txt
- Only an 8–10 μm constraint is validated here; feature behavior outside that range must be checked separately

## Evidence

- [abstract] MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm: "MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm"
- [abstract] optool + petitRADTRANS combination: "Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities"
- [abstract] petitRADTRANS spectral modes: "and with petitRADTRANS to generate transmission or emission spectra"
- [methods] custom Fortran solver used by the upstream run: "**sol.f90** is a custom fortran program to numerically solve linear systems"
- [readme] running the upstream ExoLyn relaxation: "After compilation, you can run the code by `python3 relaxation.py`"
