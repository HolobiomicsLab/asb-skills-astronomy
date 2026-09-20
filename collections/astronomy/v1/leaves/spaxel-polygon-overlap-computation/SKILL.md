---
name: spaxel-polygon-overlap-computation
description: Use when a monochromatic slice of a host galaxy model cube built on square
  PS1 pixels (0.50 arcsec after 2x2 rebinning) must be resampled onto the mismatched
  hexagonal SEDm spaxel grid for scene-model fitting.
when_to_use_negative:
- The transient has no pre-transient photometric host images, so no PS1-derived hyperspectral
  host model exists to project
- You need sub-spaxel-accurate PSF-interpolated projection rather than the nearest-neighbor
  polygon-overlap approximation
- The instrument grid is not the SEDm hexagonal 0.558 arcsec spaxel layout (scale
  ratio 2.230 assumed)
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Shapely
  role: Planar geometry engine used to compute polygon intersections between square
    model pixels and hexagonal spaxels
  repo: https://github.com/shapely/shapely
- name: GeoPandas
  role: Pandas-based wrapper through which the shapely polygon-overlap operations
    are accessed (as wrapped in hypergal.utils.geometry)
  repo: https://github.com/geopandas/geopandas
- name: HyperGal
  role: Scene-modeling package supplying hypergal.photometry.astrometry, hypergal.spectroscopy.sedmtools,
    hypergal.utils.geometry, and hypergal.scene.host conventions for the projection
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:96432dc848451264602897f6fd036d57adbf6324487410ff4379266ac94a5af1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shapely/shapely
  tools:
  - GeoPandas
  - HyperGal
  - Shapely
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spaxel-polygon-overlap-computation

## Summary

Projects a PS1-derived hyperspectral host galaxy model cube onto the SEDm hexagonal spaxel grid (0.558 arcsec spaxels) by computing polygon-intersection overlap fractions between square model pixels and hexagonal spaxels, effectively a nearest-neighbor spatial projection anchored at the transient position.

## When to use

Use when a monochromatic slice of a host galaxy model cube built on square PS1 pixels (0.50 arcsec after 2x2 rebinning) must be resampled onto the mismatched hexagonal SEDm spaxel grid for scene-model fitting.

## When NOT to use

- The transient has no pre-transient photometric host images, so no PS1-derived hyperspectral host model exists to project
- You need sub-spaxel-accurate PSF-interpolated projection rather than the nearest-neighbor polygon-overlap approximation
- The instrument grid is not the SEDm hexagonal 0.558 arcsec spaxel layout (scale ratio 2.230 assumed)

## Inputs

- SED-fit host galaxy model cube from hypergal.spectroscopy.sedfitting (monochromatic (meta)slice)
- PS1-derived model pixel grid (0.50 arcsec pixels after 2x2 rebinning) registered via hypergal.photometry.astrometry
- SEDm hexagonal spaxel grid (0.558 arcsec spaxels) from hypergal.spectroscopy.sedmtools
- Transient position used as the spatial anchor

## Outputs

- projected_model_cube.fits — overlap-weighted projected host model cube on the SEDm spaxel grid
- spaxel_overlap_weights.csv — per-spaxel table of model-pixel overlap weights

## How to apply

Load the SED-fit host model cube (product of hypergal.spectroscopy.sedfitting) and select a monochromatic (meta)slice. Build the square PS1-derived pixel grid (0.50 arcsec pixels after 2x2 rebinning) in the SEDm focal-plane frame with hypergal.photometry.astrometry, anchored at the transient position, and the hexagonal SEDm spaxel grid (0.558 arcsec spaxels, model-to-spaxel scale ratio 2.230) with hypergal.spectroscopy.sedmtools. Compute nearest-neighbor polygon-intersection overlap fractions between model pixels and spaxels using shapely geometry operations accessed through geopandas, as wrapped in hypergal.utils.geometry. Accumulate overlap-weighted model pixel fluxes per spaxel following hypergal.scene.host conventions, then write the projected cube to projected_model_cube.fits and the per-spaxel weight table to spaxel_overlap_weights.csv.

## Related tools

- **Shapely** (Planar geometry engine used to compute polygon intersections between square model pixels and hexagonal spaxels) — https://github.com/shapely/shapely
- **GeoPandas** (Pandas-based wrapper through which the shapely polygon-overlap operations are accessed (as wrapped in hypergal.utils.geometry)) — https://github.com/geopandas/geopandas
- **HyperGal** (Scene-modeling package supplying hypergal.photometry.astrometry, hypergal.spectroscopy.sedmtools, hypergal.utils.geometry, and hypergal.scene.host conventions for the projection) — https://github.com/JeremyLezmy/hypergal

## Examples

```
import geopandas as gpd; overlaps = gpd.overlay(model_pixels_gdf, spaxels_gdf, how='intersection'); overlaps['w'] = overlaps.area / overlaps['area_spaxel']
```

## Evaluation signals

- Every spaxel's overlap weights are non-negative and sum to <= 1
- The projected slice conserves the input model flux within the observed spaxel footprint (qualitative QC pass; the paper reports no numeric threshold for this intermediate step)
- projected_model_cube.fits and spaxel_overlap_weights.csv are produced with consistent per-spaxel entries
- Model-to-spaxel scale ratio is 2.230 (0.50 arcsec PS1 pixels vs 0.558 arcsec SEDm spaxels), confirming correct grid registration
- Projection is anchored at the transient position — misregistration should show as displaced host morphology in the projected slice

## Limitations

- The projection is effectively equivalent to a nearest-neighbor interpolation scheme, so it does not smooth or interpolate flux across spaxel boundaries beyond overlap weighting
- No numeric flux-conservation threshold is reported for this intermediate step; acceptance is a qualitative QC pass
- The projected host model depends on external astrometry of the transient position; any astrometric offset propagates directly into the projected cube

## Evidence

- [full_text] geometric overlap with transient anchor: [evidence span withheld — anchor status: not_found]
- [full_text] grid parameters: [evidence span withheld — anchor status: not_found]
- [readme] projection onto SEDm space: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [full_text] geopandas/shapely geometry operations: [evidence span withheld — anchor status: not_found]
