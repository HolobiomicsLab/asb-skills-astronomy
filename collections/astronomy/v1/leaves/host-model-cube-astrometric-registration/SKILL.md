---
name: host-model-cube-astrometric-registration
description: Use when a square-pixel photometric host model (e.g. PS1, 0.50 arcsec
  pixels after 2x2 rebinning) must be remapped onto mismatched integral-field-spectrograph
  spaxels (SEDm hexagonal 0.558 arcsec spaxels) per monochromatic slice.
when_to_use_negative:
- The host and instrument data already share the same pixel grid and WCS, so direct
  resampling or WCS reprojection suffices instead of polygon-overlap projection
- No pre-transient photometric imaging of the host exists, so a hyperspectral host
  model cube cannot be built in the first place
- The target is an isolated point source far from the host, where the standard PSF
  extraction assumption holds and host projection is unnecessary
tools:
- name: HyperGal
  role: Provides the scene-modeling framework and modules (hypergal.photometry.astrometry,
    hypergal.spectroscopy.sedmtools, hypergal.utils.geometry, hypergal.scene.host)
    used to build the two grids and accumulate overlap-weighted fluxes
  repo: https://github.com/Jeremy_Lezmy/Hyper_Gal
- name: Shapely
  role: Planar geometric object library used for the polygon-intersection overlap
    computation between square model pixels and hexagonal spaxels
  repo: https://github.com/shapely/shapely
- name: GeoPandas
  role: Access layer for shapely geometry operations used in the nearest-neighbor
    polygon-intersection overlap scheme
  repo: https://github.com/geopandas/geopandas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:b217024ae01d2536ceed1a62d95cb141842ad229e3947c3ccd5f038feb2da005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Jeremy_Lezmy/Hyper_Gal
  tools:
  - GeoPandas
  - HyperGal
  - Shapely
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# host-model-cube-astrometric-registration

## Summary

Projects the PS1-derived hyperspectral host galaxy model cube onto the SEDm hexagonal spaxel grid by geometrically overlapping the two polygonal pixel/spaxel grids, anchored at the transient position (equivalent to a nearest-neighbor interpolation scheme).

## When to use

Use when a square-pixel photometric host model (e.g. PS1, 0.50 arcsec pixels after 2x2 rebinning) must be remapped onto mismatched integral-field-spectrograph spaxels (SEDm hexagonal 0.558 arcsec spaxels) per monochromatic slice.

## When NOT to use

- The host and instrument data already share the same pixel grid and WCS, so direct resampling or WCS reprojection suffices instead of polygon-overlap projection
- No pre-transient photometric imaging of the host exists, so a hyperspectral host model cube cannot be built in the first place
- The target is an isolated point source far from the host, where the standard PSF extraction assumption holds and host projection is unnecessary

## Inputs

- SED-fit hyperspectral host galaxy model cube (output of hypergal.spectroscopy.sedfitting)
- PS1 pre-transient photometric images of the host galaxy (0.50 arcsec pixels after 2x2 rebinning)
- Transient position used as the spatial/astrometric anchor
- SEDm spaxel grid parameters (0.558 arcsec hexagonal spaxels, model-to-spaxel scale ratio 2.230)

## Outputs

- projected_model_cube.fits — projected host model cube on the SEDm spaxel grid
- spaxel_overlap_weights.csv — per-spaxel overlap-weight table

## How to apply

Load the SED-fit host galaxy model cube (product of hypergal.spectroscopy.sedfitting) and select a monochromatic (meta)slice. Build the square PS1-derived model pixel grid in the SEDm focal-plane frame using hypergal.photometry.astrometry, with the transient position as the spatial anchor, and build the hexagonal SEDm spaxel grid (0.558 arcsec spaxels; model-to-spaxel scale ratio 2.230) with hypergal.spectroscopy.sedmtools. Compute nearest-neighbor polygon-intersection overlap fractions between model pixels and spaxels via shapely geometry operations accessed through geopandas (wrapped in hypergal.utils.geometry), then accumulate overlap-weighted model pixel fluxes per spaxel following hypergal.scene.host conventions. The rationale is that mismatched geometries (square PS1 pixels vs hexagonal SEDm spaxels) preclude direct resampling, so exact polygon overlap conserves flux while the transient position fixes the astrometric registration.

## Related tools

- **HyperGal** (Provides the scene-modeling framework and modules (hypergal.photometry.astrometry, hypergal.spectroscopy.sedmtools, hypergal.utils.geometry, hypergal.scene.host) used to build the two grids and accumulate overlap-weighted fluxes) — https://github.com/Jeremy_Lezmy/Hyper_Gal
- **Shapely** (Planar geometric object library used for the polygon-intersection overlap computation between square model pixels and hexagonal spaxels) — https://github.com/shapely/shapely
- **GeoPandas** (Access layer for shapely geometry operations used in the nearest-neighbor polygon-intersection overlap scheme) — https://github.com/geopandas/geopandas

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- Every spaxel's overlap weights are non-negative and sum to <= 1
- The projected slice conserves the input model flux within the observed spaxel footprint (qualitative QC; the paper reports no numeric threshold for this intermediate step)
- Projected cube is written as projected_model_cube.fits and the weight table as spaxel_overlap_weights.csv with per-spaxel weights
- The model pixel grid is anchored such that the transient position aligns across the PS1 and SEDm frames (checked via hypergal.photometry.astrometry registration)
- Model-to-spaxel scale consistency: PS1 pixels 0.50 arcsec (after 2x2 rebinning) vs SEDm 0.558 arcsec hexagonal spaxels, ratio 2.230

## Limitations

- The paper reports no numeric threshold for flux conservation at this intermediate step, so validation is a qualitative QC pass on weight normalization and flux conservation
- The projection assumes the transient position is a reliable spatial anchor; astrometric error in the transient position propagates directly into the host model registration
- HyperGal overall requires an external host-galaxy redshift measurement as a priori input, which constrains the upstream SED-fit cube this projection consumes

## Evidence

- [readme] The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [methods] Geometric overlap of polygonal grids anchored at transient position: [evidence span withheld — anchor status: not_found]
- [abstract] Pre-transient photometric images as host model source: "uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
