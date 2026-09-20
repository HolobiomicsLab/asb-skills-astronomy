---
name: elliptical-aperture-photometry
description: Use when you have a separated host galaxy component (cube or spaxel map)
  plus co-registered reference imaging, and need an integrated 1D host spectrum for
  redshift or line-flux measurements.
when_to_use_negative:
- The residual host component has not been separated from the SN point source and
  diffuse background — integrating the full scene would bias line positions and fluxes
  with SN light.
- The host is so compact that a large elliptical aperture would mostly capture background/SN
  flux; aperture-fitting a point-source extractor output is the wrong frame.
- You need spatially resolved line maps rather than an integrated measurement — this
  skill deliberately collapses the host into one 1D spectrum.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: shapely
  role: Constructs the elliptical aperture geometry around the segmented host for
    spaxel selection
  repo: https://github.com/shapely/shapely
- name: geopandas
  role: Handles geometric/spatial operations on the aperture and segmentation regions
  repo: https://github.com/geopandas/geopandas
- name: HyperGal
  role: Provides the fitted scene model from which the host hyperspectral component
    is isolated before aperture integration
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:abddc34b58da028d2e927c855510355a89505736e1e75789769d5e6cec1a814f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shapely/shapely
  tools:
  - HyperGal
  - geopandas
  - shapely
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# elliptical-aperture-photometry

## Summary

Defines a source-shaped elliptical aperture around a host galaxy on co-registered imaging (e.g. PS1) and integrates the isolated HyperGal host hyperspectral component over it to produce a 1D integrated host spectrum for subsequent measurements such as emission-line redshifts.

## When to use

Use when you have a separated host galaxy component (cube or spaxel map) plus co-registered reference imaging, and need an integrated 1D host spectrum for redshift or line-flux measurements.

## When NOT to use

- The residual host component has not been separated from the SN point source and diffuse background — integrating the full scene would bias line positions and fluxes with SN light.
- The host is so compact that a large elliptical aperture would mostly capture background/SN flux; aperture-fitting a point-source extractor output is the wrong frame.
- You need spatially resolved line maps rather than an integrated measurement — this skill deliberately collapses the host into one 1D spectrum.

## Inputs

- Fitted HyperGal scene model for an SEDm cube with separated host, SN point-source, and diffuse background components
- Co-registered PS1 (Pan-STARRS) reference image of the host field
- Source segmentation map of the reference image

## Outputs

- 1D integrated host spectrum over the [4000,8000] Å SEDm domain
- Emission-line detections (e.g. Halpha) and derived host redshift estimate per simulation/target

## How to apply

Run source segmentation on the co-registered reference image (e.g. PS1) to locate the host, then build an elliptical aperture with shapely ellipse geometry sized to the segmented source. Integrate the isolated HyperGal host component — the fitted galaxy model with SN point-source and diffuse background removed — over the aperture spaxels, restricted to the [4000,8000] Å SEDm domain, to yield a 1D integrated host spectrum. The large elliptical aperture captures extended host flux while excluding residual transient contamination. Detect emission lines (e.g. Halpha) in the integrated spectrum and compare inferred redshifts to input simulation values; residuals within |z_est - z_input| ~ 5e-3 (the SNID-derived redshift precision) indicate success.

## Related tools

- **shapely** (Constructs the elliptical aperture geometry around the segmented host for spaxel selection) — https://github.com/shapely/shapely
- **geopandas** (Handles geometric/spatial operations on the aperture and segmentation regions) — https://github.com/geopandas/geopandas
- **HyperGal** (Provides the fitted scene model from which the host hyperspectral component is isolated before aperture integration) — https://github.com/JeremyLezmy/hypergal

## Examples

```
from shapely.geometry import Point; ellipse = Point(sn_x, sn_y).buffer(1).exterior; host_spec = host_cube[:, apert_mask].sum(axis=1)  # integrate host component over elliptical aperture on [4000,8000] A
```

## Evaluation signals

- Main spectral features of the integrated host spectrum appear at the known/input host redshift (e.g. z = 0.045 for SN ZTF20aamifit).
- Residual |z_est - z_input| per simulation falls within the SNID-derived redshift precision sigma_z ~ 5e-3.
- Emission lines (e.g. Halpha) are detected in the integrated spectrum within the [4000,8000] Å SEDm domain.
- Aperture, built from PS1 segmentation with shapely ellipse geometry, contains the segmented host spaxels while excluding the SN point-source component.
- Integrated flux scales sensibly with the number of summed spaxels and does not show SN-dominated continuum features.

## Limitations

- The paper notes the residual host component 'effectively precludes the use of the residual host component for any a posteriori measurements, e.g. redshift or local measurement of Halpha flux' in the standard setup, so this in-pipeline use is a proposed future-work extension rather than a sanctioned feature.
- HyperGal requires an external host-galaxy redshift measurement as input, so an in-pipeline redshift estimate must still be validated against simulation inputs.
- The aperture is defined on the co-registered PS1 image; mis-registration or poor segmentation of low-surface-brightness hosts will distort the ellipse and bias the integrated spectrum.

## Evidence

- [methods] Define a large elliptical aperture around the host on the co-registered PS1 image using source segmentation and shapely ellipse geometry.: [evidence span withheld — anchor status: not_found]
- [results] host spectrum isolated by HyperGal and integrated over spaxels shows features at input redshift: [evidence span withheld — anchor status: not_found]
- [discussion] SNID redshift precision threshold: "the remaining 50% of SNe have a redshift deduced from a preliminary extraction of the SN spectrum"
- [discussion] a posteriori measurement restriction: [evidence span withheld — anchor status: not_found]
- [readme] shapely README description: [evidence span withheld — anchor status: external_doc]
