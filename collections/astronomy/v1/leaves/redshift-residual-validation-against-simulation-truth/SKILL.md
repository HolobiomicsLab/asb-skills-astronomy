---
name: redshift-residual-validation-against-simulation-truth
description: Use when you have simulated SEDm cubes with known input host redshifts
  and want to verify that a redshift measured from the HyperGal-isolated host component
  is consistent with simulation truth.
when_to_use_negative:
- 'Real (non-simulated) observations: there is no z_input truth value, so residual
  validation against simulation truth cannot be performed'
- 'Hosts that already have an external spectroscopic redshift (e.g. from SDSS, precision
  sigma_z ~ 1e-5): the pipeline-internal redshift at sigma_z ~ 5e-3 offers no advantage'
- Hosts whose emission lines (e.g. Halpha) fall outside the [4000,8000] Å SEDm wavelength
  domain at the redshift of interest
edam_operation: http://edamontology.org/operation_2945
tools:
- name: HyperGal
  role: Provides the fitted scene model from which the host hyperspectral component
    is isolated from the SN and diffuse background
  repo: https://github.com/JeremyLezmy/hypergal
- name: shapely
  role: Defines the elliptical aperture geometry used to integrate the isolated host
    component over selected spaxels
  repo: https://github.com/shapely/shapely
- name: SNID
  role: Source of the sigma_z ~ 5e-3 precision threshold used as the residual consistency
    criterion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:48de13e17ca1c1b35ece1e3581429313b24957d58e83bcb390ae596db94b4710
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - HyperGal
  - SNID
  - shapely
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Redshift residual validation against simulation truth

## Summary

Validates an in-pipeline host redshift estimate — obtained from the host component isolated by HyperGal's fitted scene model — by comparing it to the known simulation input redshift within the SNID precision sigma_z ~ 5e-3. This operationalizes the paper's sanctioned future-work extension that replaces HyperGal's external-redshift input.

## When to use

Use when you have simulated SEDm cubes with known input host redshifts and want to verify that a redshift measured from the HyperGal-isolated host component is consistent with simulation truth.

## When NOT to use

- Real (non-simulated) observations: there is no z_input truth value, so residual validation against simulation truth cannot be performed
- Hosts that already have an external spectroscopic redshift (e.g. from SDSS, precision sigma_z ~ 1e-5): the pipeline-internal redshift at sigma_z ~ 5e-3 offers no advantage
- Hosts whose emission lines (e.g. Halpha) fall outside the [4000,8000] Å SEDm wavelength domain at the redshift of interest

## Inputs

- Fitted HyperGal scene model for each simulated SEDm cube (host, SN point-source, and diffuse background components)
- Co-registered pre-transient PS1 (Pan-STARRS) image of the host field
- Source segmentation map of the host
- Simulation input (truth) host redshift z_input

## Outputs

- 1D integrated host spectrum over the [4000,8000] Å SEDm domain per simulation
- Per-simulation host redshift estimate z_est from emission-line positions (e.g. Halpha)
- Per-simulation consistency flag based on |z_est - z_input| <= 5e-3
- Aggregate residual distribution z_est - z_input across the simulated cube sample

## How to apply

Load the fitted HyperGal scene model for each simulated SEDm cube and separate the host galaxy hyperspectral component from the SN point-source and diffuse background components. Define a large elliptical aperture around the host on the co-registered PS1 image using source segmentation and shapely ellipse geometry, then integrate the isolated host component over that aperture to yield a 1D spectrum over the [4000,8000] Å SEDm domain. Detect emission lines (e.g. Halpha) in the integrated spectrum and convert their observed positions into a host redshift estimate z_est per simulation. Flag each result as consistent when the residual |z_est - z_input| falls within the SNID precision sigma_z ~ 5e-3 quoted in the discussion for SNID-derived redshifts (coarser than the SDSS spectroscopic precision of sigma_z ~ 1e-5). The rationale is that the paper lists the need for an external host redshift as HyperGal's most limiting constraint, so demonstrating recovery of z from the extracted host component justifies an in-pipeline redshift.

## Related tools

- **HyperGal** (Provides the fitted scene model from which the host hyperspectral component is isolated from the SN and diffuse background) — https://github.com/JeremyLezmy/hypergal
- **shapely** (Defines the elliptical aperture geometry used to integrate the isolated host component over selected spaxels) — https://github.com/shapely/shapely
- **SNID** (Source of the sigma_z ~ 5e-3 precision threshold used as the residual consistency criterion)

## Examples

```
flag_consistent = abs(z_est - z_input) <= 5e-3  # SNID sigma_z threshold quoted in the HyperGal discussion
```

## Evaluation signals

- Main spectral features of the integrated host spectrum appear at the input redshift (e.g. z = 0.045 recovered for SN ZTF20aamifit)
- Residual |z_est - z_input| <= 5e-3 (SNID precision threshold) for each flagged-consistent simulation
- An emission line (e.g. Halpha) is confidently detected in the integrated [4000,8000] Å host spectrum before redshift conversion
- The residual distribution z_est - z_input across simulated cubes is centered on zero without systematic offsets
- Integrated spectrum is computed over a defined elliptical aperture on the co-registered PS1 image, not over the full cube or the SN spaxels

## Limitations

- The paper notes this capability is not yet part of HyperGal: 'it effectively precludes the use of the residual host component for any a posteriori measurements, e.g. redshift or local measurement of Halpha flux' — this skill is the future-work extension, not a documented production feature
- Monochromatic galaxy amplitudes G(lambda) in the 3D fit correct redshift offsets and emission-line fluxes only to first order, and the effect's dependence on the relative stellar/gaseous distribution 'has not been studied extensively for Hyper Gal'
- The SN-host distance parameter was marginalized uniformly between 0 and 5 arcsec rather than explicitly modeled, which may affect how well the host component is isolated
- The sigma_z ~ 5e-3 tolerance is much coarser than external spectroscopic redshifts (SDSS, sigma_z ~ 1e-5 for z < 0.1)

## Evidence

- [discussion] external redshift constraint: "the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy"
- [discussion] residual host component unusable a posteriori: [evidence span withheld — anchor status: not_found]
- [discussion] SNID-derived redshift precision context: "the remaining 50% of SNe have a redshift deduced from a preliminary extraction of the SN spectrum"
- [discussion] distance marginalization: [evidence span withheld — anchor status: not_found]
- [readme] scene modeler description: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
