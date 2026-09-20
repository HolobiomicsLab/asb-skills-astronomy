---
name: ifu-hexagonal-spaxel-grid-geometry
description: Use when resampling a square-pixel photometric model cube (e.g. PS1 0.50
  arcsec pixels after 2x2 rebinning) onto a hexagonal IFS spaxel grid (e.g. SEDm 0.558
  arcsec spaxels) per monochromatic slice, with mismatched pixel geometries and a
  transient position as spatial anchor.
when_to_use_negative:
- The target's data are already on the SEDm spaxel grid and only require fitting,
  not re-projection (e.g. during the ultimate 3D fit of the galaxy model, point source,
  and diffuse background)
- The input is a rectangular-pixel IFS cube rather than a hexagonal spaxel grid —
  polygon-overlap weights against hexagons are unnecessary
- The host galaxy is unresolved or absent; the projection anchors on the transient
  position and would have no meaningful extended host model to distribute
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Shapely
  role: Planar geometric operations (polygon intersection/overlap fractions) between
    square model pixels and hexagonal spaxels
  repo: https://github.com/shapely/shapely
- name: GeoPandas
  role: Vectorized access to shapely geometry operations for computing per-spaxel
    overlap weight tables
  repo: https://github.com/geopandas/geopandas
- name: HyperGal
  role: 'Hosts the wrapping modules used in the projection: hypergal.photometry.astrometry
    (PS1 grid framing), hypergal.spectroscopy.sedmtools (hexagonal spaxel grid), hypergal.utils.geometry
    (overlap weights), hypergal.scene.host (projected cube conventions)'
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:a9a275e8684c798200e671b45d82c535c4095da4b8a6c21e0295a3182ec9df10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shapely/shapely
  tools:
  - GeoPandas
  - HyperGal
  - Shapely
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ifu-hexagonal-spaxel-grid-geometry

## Summary

Projects a PS1-derived hyperspectral host galaxy model cube onto the SEDm hexagonal spaxel grid (0.558 arcsec spaxels) by computing polygon-intersection overlap fractions between square model pixels and hexagonal spaxels, anchored at the transient position. This is the geometric backbone of HyperGal's scene-modeling projection step.

## When to use

Use when resampling a square-pixel photometric model cube (e.g. PS1 0.50 arcsec pixels after 2x2 rebinning) onto a hexagonal IFS spaxel grid (e.g. SEDm 0.558 arcsec spaxels) per monochromatic slice, with mismatched pixel geometries and a transient position as spatial anchor.

## When NOT to use

- The target's data are already on the SEDm spaxel grid and only require fitting, not re-projection (e.g. during the ultimate 3D fit of the galaxy model, point source, and diffuse background)
- The input is a rectangular-pixel IFS cube rather than a hexagonal spaxel grid — polygon-overlap weights against hexagons are unnecessary
- The host galaxy is unresolved or absent; the projection anchors on the transient position and would have no meaningful extended host model to distribute

## Inputs

- SED-fit host galaxy model cube (output of hypergal.spectroscopy.sedfitting)
- Monochromatic (meta)slice of the hyperspectral host model
- PS1-derived model pixel grid parameters (0.50 arcsec pixels after 2x2 rebinning)
- Transient position (spatial anchor for astrometric alignment, via hypergal.photometry.astrometry)
- SEDm spaxel grid geometry (0.558 arcsec hexagonal spaxels; model-to-spaxel scale ratio 2.230)

## Outputs

- projected_model_cube.fits (overlap-weighted projected host model cube on the SEDm spaxel grid)
- spaxel_overlap_weights.csv (per-spaxel overlap-weight table)

## How to apply

Load the SED-fit host galaxy model cube (product of hypergal.spectroscopy.sedfitting) and select a monochromatic (meta)slice. Build the square PS1-derived model pixel grid (0.50 arcsec pixels after 2x2 rebinning) in the SEDm focal-plane frame using hypergal.photometry.astrometry, anchored at the transient position, then build the hexagonal SEDm spaxel grid (0.558 arcsec spaxels; model-to-spaxel scale ratio 2.230) using hypergal.spectroscopy.sedmtools. Compute polygon-intersection overlap fractions between model pixels and spaxels with shapely geometry operations accessed through geopandas, as wrapped in hypergal.utils.geometry — this is effectively equivalent to a nearest-neighbor interpolation scheme. Accumulate overlap-weighted model pixel fluxes per spaxel to produce the projected host model slice/cube following hypergal.scene.host conventions, writing projected_model_cube.fits and the per-spaxel overlap-weight table spaxel_overlap_weights.csv.

## Related tools

- **Shapely** (Planar geometric operations (polygon intersection/overlap fractions) between square model pixels and hexagonal spaxels) — https://github.com/shapely/shapely
- **GeoPandas** (Vectorized access to shapely geometry operations for computing per-spaxel overlap weight tables) — https://github.com/geopandas/geopandas
- **HyperGal** (Hosts the wrapping modules used in the projection: hypergal.photometry.astrometry (PS1 grid framing), hypergal.spectroscopy.sedmtools (hexagonal spaxel grid), hypergal.utils.geometry (overlap weights), hypergal.scene.host (projected cube conventions)) — https://github.com/JeremyLezmy/hypergal

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- Every spaxel's overlap weights are non-negative
- Every spaxel's overlap weights sum to <= 1
- The projected slice conserves the input model flux within the observed spaxel footprint (qualitative QC — the paper reports no numeric threshold for this intermediate step)
- The model-to-spaxel scale ratio matches the expected 2.230 for PS1 0.50 arcsec pixels vs SEDm 0.558 arcsec spaxels
- The projected cube is registered such that the transient position anchors the alignment between the PS1 pixel grid and the SEDm spaxel grid

## Limitations

- The projection is effectively equivalent to a nearest-neighbor interpolation scheme, so sub-spaxel spatial structure of the host model is not smoothly preserved
- The paper reports no numeric threshold for flux conservation at this intermediate step, so validation is only a qualitative QC pass on weight normalization and flux conservation
- HyperGal requires an external redshift measurement of the host galaxy as input, which constrains the downstream use of the projected model

## Evidence

- [methods] HyperGal projects the host model onto the SEDm spaxel grid by geometrically overlapping the two polygonal spaxel grids, using the transient position as the spatial anchor; this is effectively equivalent to a nearest-neighbor interpolation scheme.: [evidence span withheld — anchor status: not_found]
- [abstract] The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations: "projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [readme] uses pre-transient photometric images to generate a hyperspectral model of the host galaxy: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
