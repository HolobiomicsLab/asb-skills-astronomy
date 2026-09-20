---
name: pixel-value-surface-plot-visualization
description: Use when point sources selected in a Highcharts scatter plot over a catalog
  must be inspected visually via image cutouts and surface plots of raw pixel values
  (e.g. vetting DECam detections across filters).
when_to_use_negative:
- You only need static plots or non-interactive vetting — a standard Toyz Highcharts
  or image-viewer tile without custom event wiring suffices.
- Data are small local tables/images where a local tool (e.g. Ginga or matplotlib)
  avoids the server/websocket round-trip entirely.
- The analysis target is catalog-level statistics rather than pixel-level inspection
  of individual detections.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3399
tools:
- name: Toyz
  role: Web framework hosting the custom tile, JSON parameter parser, event system,
    and websocket task dispatch
  repo: https://github.com/fred3m/toyz
- name: toyz-template
  role: Template repository forked and registered as a custom Toy to build the inspection
    tile
  repo: https://github.com/fred3m/toyz-template
- name: Highcharts
  role: Interactive plot tile emitting point-selection events linked to the custom
    tile via a shared data source
- name: scipy
  role: scipy.ndimage.zoom used for scaling tiled large images in the surface plot
    computation
- name: numpy
  role: Array handling of raw pixel values for the surface plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:28797183bd9bf5d2358c2bc9d403697a55d3432ea5a9b82bab8570139dcc4f64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Toyz
  - numpy
  - scipy
  - toyz-template
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pixel-value-surface-plot-visualization

## Summary

Build a custom Toyz workspace tile that, for each point source selected in a linked Highcharts plot, loads multi-filter image cutouts and renders a surface plot of the raw pixel values centered on the selected detection, enabling visual vetting of spurious detections.

## When to use

Use when point sources selected in a Highcharts scatter plot over a catalog must be inspected visually via image cutouts and surface plots of raw pixel values (e.g. vetting DECam detections across filters).

## When NOT to use

- You only need static plots or non-interactive vetting — a standard Toyz Highcharts or image-viewer tile without custom event wiring suffices.
- Data are small local tables/images where a local tool (e.g. Ginga or matplotlib) avoids the server/websocket round-trip entirely.
- The analysis target is catalog-level statistics rather than pixel-level inspection of individual detections.

## Inputs

- Astronomical images of the same field in three different filters (e.g. DECam FITS images, potentially > 2 Gb)
- A catalog/data source of detected sources shared by the Highcharts plot and the custom tile
- User point-selection (click or drag) events from a Highcharts tile

## Outputs

- Image cutouts of the selected source from each filter, displayed in the custom tile
- A surface plot of raw pixel values centered on the selected point
- Removal of cutouts for sources the user flags as detection artifacts

## How to apply

Fork or obtain the Toyz Template repository and register it as a custom Toy either with 'python setup.py install' or via the Toyz Paths control group (toyz_name='toyz_template', path to the local directory). Define the tile's GUI declaratively with the Toyz JSON parameter parser (parameter objects with type, prop, css, func, lbl fields, wrapped in div/conditional containers), and register event handlers so point-selection events emitted by a Highcharts tile sharing the same data source propagate to the custom tile. Implement the server-side task in a module ending in 'tasks.py' with signature (toyz_settings, tid, params) — tid carries user_id, session_id, request_id — returning cutout image paths for the selected source in each of the three filters. Dispatch from the client via Toyz.Core.Websocket.send_task with a JSON task object {module, task, parameters}, render the returned cutouts, and compute the surface plot of raw pixel values centered on the selected point, scaling large tiled images with scipy.ndimage.zoom-based interpolation.

## Related tools

- **Toyz** (Web framework hosting the custom tile, JSON parameter parser, event system, and websocket task dispatch) — https://github.com/fred3m/toyz
- **toyz-template** (Template repository forked and registered as a custom Toy to build the inspection tile) — https://github.com/fred3m/toyz-template
- **Highcharts** (Interactive plot tile emitting point-selection events linked to the custom tile via a shared data source)
- **scipy** (scipy.ndimage.zoom used for scaling tiled large images in the surface plot computation)
- **numpy** (Array handling of raw pixel values for the surface plot)

## Examples

```
Toyz.Core.Websocket.send_task({module: 'toyz_template', task: 'get_cutouts', parameters: {source_id: src_id, filters: ['g','r','z']}})
```

## Evaluation signals

- Selecting a single point (or dragging over multiple points) on the Highcharts chart loads matching cutouts and the surface plot in the custom tile.
- Cutouts for all three filters of the same field appear for each selected source, and each set can be removed when the detection is an artifact.
- The server-side task returns paths for every selected source and the tile renders them without websocket/task errors.
- Large tiled images render correctly with the scipy.ndimage.zoom-based scaling applied.
- Removing a flagged detection deletes its cutout set from the tile without breaking the link to remaining points.

## Limitations

- Requires the custom module to be registered correctly (via python setup.py install or the Toyz Paths control group) or events/tasks will not dispatch.
- The surface plot is of raw pixel values, so detector artifacts in uncalibrated pixels will be visible in the plot.
- Very large images (> 2 Gb DECam frames) depend on tiling plus scipy.ndimage.zoom-based scaling for interactive performance.

## Evidence

- [discussion] The custom tile (Figure 7) displays an image and surface plot from images of the same field taken in three different filters: [evidence span withheld — anchor status: not_found]
- [discussion] Implement the server-side task function in a module ending in 'tasks.py' with the required signature (toyz_settings, tid, params): [evidence span withheld — anchor status: not_found]
- [intro] modules for scientific image and data processing like numpy (Walt et al., 201): [evidence span withheld — anchor status: not_found]
- [abstract] Toyz can quickly view very large images (> 2 Gb) taken with DECam and other large-FOV cameras: [evidence span withheld — anchor status: not_found]
