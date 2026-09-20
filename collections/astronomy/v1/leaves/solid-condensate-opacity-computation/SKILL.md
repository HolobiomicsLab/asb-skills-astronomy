---
name: solid-condensate-opacity-computation
description: Use when you have a converged multispecies cloud profile (layered Mg-silicate
  clouds over an Fe cloud) and need solid opacities and a mid-infrared spectrum, especially
  to reproduce or interpret the 8–10 μm MgSiO3/Mg2SiO4 absorption feature.
when_to_use_negative:
- You only need the cloud profile itself (compositions, condensation rates, layered
  structure) — use the ExoLyn relaxation run directly instead of computing opacities
- The analysis target is gas-phase chemistry only, with no cloud/condensate opacity
  contribution to the spectrum
- The wavelength region of interest does not extend to the mid-infrared where the
  Mg-silicate feature lies
tools:
- name: ExoLyn
  role: Produces the converged 1D cloud profile (layered Mg-silicate clouds over an
    iron cloud) whose particle compositions and size distributions are the input to
    the opacity calculation
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates the solid opacities from the cloud particle compositions fed in
    from the ExoLyn profile
- name: petitRADTRANS
  role: Assembles the combined gas + cloud opacity set and generates the transmission
    or emission spectrum over the mid-infrared wavelength grid
- name: Python 3 / numpy
  role: Drives the ExoLyn relaxation workflow and the post-processing/spectrum table
    writing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:27a8f1bfab79d555285b674fc5bcc5686418f05b46011c6918ef857a72980eef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Python
  - Python 3 / numpy
  - numpy
  - optool
  - petitRADTRANS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# solid-condensate-opacity-computation

## Summary

Compute solid (condensate) opacities for cloud particle compositions — e.g. MgSiO3, Mg2SiO4, and Fe — from an ExoLyn cloud profile using optool, then combine them with gas opacities in petitRADTRANS to generate transmission/emission spectra. This is how the 8–10 μm Mg-silicate absorption feature, which constrains cloud particle composition, is reproduced.

## When to use

Use when you have a converged multispecies cloud profile (layered Mg-silicate clouds over an Fe cloud) and need solid opacities and a mid-infrared spectrum, especially to reproduce or interpret the 8–10 μm MgSiO3/Mg2SiO4 absorption feature.

## When NOT to use

- You only need the cloud profile itself (compositions, condensation rates, layered structure) — use the ExoLyn relaxation run directly instead of computing opacities
- The analysis target is gas-phase chemistry only, with no cloud/condensate opacity contribution to the spectrum
- The wavelength region of interest does not extend to the mid-infrared where the Mg-silicate feature lies

## Inputs

- Converged ExoLyn atmosphere profile (layered Mg-silicate clouds over an iron cloud) from the upstream relaxation run
- Cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and their size distributions
- petitRADTRANS-compatible gas opacity set for the atmospheric column

## Outputs

- Solid opacities for the condensate species computed by optool
- Combined gas + cloud opacity set for the atmospheric column
- Spectrum table (spectrum.dat) with wavelength and transit-radius/flux columns covering the mid-infrared

## How to apply

Load the converged ExoLyn atmosphere profile produced by the upstream relaxation run, extracting the cloud particle compositions (MgSiO3, Mg2SiO4, Fe) and their size distributions per layer. Feed these compositions into optool to calculate the solid opacities, since ExoLyn is explicitly designed to be combined with optool for this step. Assemble the combined gas + cloud opacity set in petitRADTRANS for the atmospheric column, then run petitRADTRANS in transmission (or emission) mode over a wavelength grid that covers the mid-infrared so the 8–10 μm region is sampled. Write the result as a wavelength and transit-radius/flux table (e.g. spectrum.dat). Rationale: MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm, so the wavelength grid must extend into the mid-infrared for the feature to be detectable in the output spectrum.

## Related tools

- **ExoLyn** (Produces the converged 1D cloud profile (layered Mg-silicate clouds over an iron cloud) whose particle compositions and size distributions are the input to the opacity calculation) — github.com/helonghuangastro/exolyn
- **optool** (Calculates the solid opacities from the cloud particle compositions fed in from the ExoLyn profile)
- **petitRADTRANS** (Assembles the combined gas + cloud opacity set and generates the transmission or emission spectrum over the mid-infrared wavelength grid)
- **Python 3 / numpy** (Drives the ExoLyn relaxation workflow and the post-processing/spectrum table writing)

## Examples

```
bash build.sh; python3 relaxation.py
```

## Evaluation signals

- spectrum.dat exists and is a valid two-column (or more) wavelength/transit-radius or flux table
- The MgSiO3/Mg2SiO4 absorption feature in the generated spectrum falls in the 8–10 μm range
- The opacity set contains contributions from all condensate species in the ExoLyn profile (MgSiO3, Mg2SiO4, Fe), not just gas opacity
- The wavelength grid of the petitRADTRANS run covers the mid-infrared so the 8–10 μm region is actually sampled

## Limitations

- ExoLyn is a 1D cloud model balancing physical consistency with computational efficiency; results are consistent with more complex cloud formation models but obtained in only a few seconds, which implies simplifications relative to full kinetic cloud models
- The MgSiO3/Mg2SiO4 8–10 μm feature depends on the cloud particle compositions and size distributions from the upstream profile; errors there propagate to the opacity/spectrum
- Extending ExoLyn to new chemistry requires supplying gibbs formation energy data for all molecules involved (available on janaf.nist.gov) and ensuring all gas and solid species are listed in parameters.txt

## Evidence

- [abstract] Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities: "Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities"
- [abstract] MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm: "MgSiO3 and Mg2SiO4 components give rise to an absorption feature at 8–10 μm"
- [abstract] and with petitRADTRANS to generate transmission or emission spectra: "and with petitRADTRANS to generate transmission or emission spectra"
- [readme] After downloading the code, you can compile the fortran part by running `bash build.sh`: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [abstract] the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud: "the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud"
