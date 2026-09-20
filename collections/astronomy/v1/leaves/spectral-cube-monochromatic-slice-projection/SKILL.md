---
name: spectral-cube-monochromatic-slice-projection
description: Use when a square-pixelated hyperspectral model cube (e.g. from photometric
  SED fitting) must be regridded onto a hexagonal IFS spaxel grid with mismatched
  pixel geometries, using polygon overlap fractions rather than resampling.
when_to_use_negative:
- When the target grid has the same pixel geometry and sampling as the model cube
  (no polygon overlap needed; simple array copy suffices)
- When sub-spaxel accuracy beyond nearest-neighbor weighting is required — the overlap
  scheme is only equivalent to nearest-neighbor interpolation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Shapely
  role: Planar geometry operations used to compute polygon-intersection overlap fractions
    between model pixels and SEDm spaxels
  repo: https://github.com/shapely/shapely
- name: GeoPandas
  role: Wraps shapely geometry objects for spatial overlap computation between the
    two grids
  repo: https://github.com/geopandas/geopandas
- name: HyperGal
  role: Provides hypergal.photometry.astrometry, hypergal.spectroscopy.sedmtools,
    hypergal.utils.geometry, and hypergal.scene.host modules that implement the projection
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:232fdb190dcb6ed2dea25eb0e7b5d59deeb6c1ac9a7c734ebec5dd65711bda03
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
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spectral-cube-monochromatic-slice-projection

## Summary

Projects a monochromatic slice of a hyperspectral host-galaxy model cube (PS1-derived, square 0.50 arcsec pixels after 2x2 rebinning) onto the SEDm hexagonal spaxel grid (0.558 arcsec spaxels) by geometric polygon-intersection overlap, equivalent to a nearest-neighbor interpolation, anchored at the transient position.

## When to use

Use when a square-pixelated hyperspectral model cube (e.g. from photometric SED fitting) must be regridded onto a hexagonal IFS spaxel grid with mismatched pixel geometries, using polygon overlap fractions rather than resampling.

## When NOT to use

- When the target grid has the same pixel geometry and sampling as the model cube (no polygon overlap needed; simple array copy suffices)
- When sub-spaxel accuracy beyond nearest-neighbor weighting is required — the overlap scheme is only equivalent to nearest-neighbor interpolation

## Inputs

- SED-fit host galaxy model cube (output of hypergal.spectroscopy.sedfitting)
- Monochromatic (meta)slice of the model cube
- PS1-derived square pixel grid parameters (0.50 arcsec pixels after 2x2 rebinning)
- SEDm hexagonal spaxel grid parameters (0.558 arcsec spaxels; scale ratio 2.230)
- Transient position (spatial anchor)

## Outputs

- projected_model_cube.fits (projected host model cube/slice)
- spaxel_overlap_weights.csv (per-spaxel overlap-weight table)

## How to apply

Load the SED-fit host galaxy model cube (product of hypergal.spectroscopy.sedfitting) and select a monochromatic (meta)slice. Build the square PS1-derived model pixel grid (0.50 arcsec pixels) in the SEDm focal-plane frame using hypergal.photometry.astrometry, anchored at the transient position, and build the hexagonal SEDm spaxel grid (0.558 arcsec spaxels; model-to-spaxel scale ratio 2.230) with hypergal.spectroscopy.sedmtools. Compute nearest-neighbor polygon-intersection overlap fractions between model pixels and spaxels via shapely geometry operations accessed through geopandas, wrapped in hypergal.utils.geometry, then accumulate overlap-weighted model pixel fluxes per spaxel following hypergal.scene.host conventions. The transient position serves as the spatial anchor for aligning the two grids. Outputs are written to projected_model_cube.fits and spaxel_overlap_weights.csv.

## Related tools

- **Shapely** (Planar geometry operations used to compute polygon-intersection overlap fractions between model pixels and SEDm spaxels) — https://github.com/shapely/shapely
- **GeoPandas** (Wraps shapely geometry objects for spatial overlap computation between the two grids) — https://github.com/geopandas/geopandas
- **HyperGal** (Provides hypergal.photometry.astrometry, hypergal.spectroscopy.sedmtools, hypergal.utils.geometry, and hypergal.scene.host modules that implement the projection) — https://github.com/JeremyLezmy/hypergal

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh  # pipeline includes projection of the model cube onto the SEDm spaxel grid (outputs projected_model_cube.fits, spaxel_overlap_weights.csv)
```

## Evaluation signals

- Every spaxel's overlap weights are non-negative
- Each spaxel's overlap weights sum to <= 1
- Projected slice conserves the input model flux within the observed spaxel footprint (qualitative QC; no numeric threshold published)
- Projected geometry matches the SEDm 0.558 arcsec hexagonal spaxel layout anchored at the transient position

## Limitations

- The paper reports no numeric threshold for flux conservation at this intermediate step, so validation is a qualitative QC pass
- Overlap weighting is effectively a nearest-neighbor interpolation scheme, not a flux-conserving resampling with sub-spaxel precision

## Evidence

- [methods] geometric polygon overlap projection: [evidence span withheld — anchor status: not_found]
- [methods] nearest-neighbor equivalence: [evidence span withheld — anchor status: not_found]
- [abstract] projection onto observation space: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [readme] tool roles from README: "it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
