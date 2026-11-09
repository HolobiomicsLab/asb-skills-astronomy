---
name: highcharts-point-selection-linking
description: Use when a custom Toyz tile must display cutout images and surface plots
  for point sources selected (clicked or drag-boxed) in a Highcharts plot sharing
  the same data source.
when_to_use_negative:
- The plot and image tiles do not share the same data source, so point-selection events
  cannot propagate between tiles
- The goal is static visualization of a catalog or image without interactive selection
  — the default Toyz Highchart or image viewer alone suffices
- The remote server cannot run custom task modules (no tasks.py module registered),
  since cutout generation requires a server-side task dispatch
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Toyz
  role: Web framework providing the tile system, JSON parameter parser, event-handler
    registry, and Websocket task dispatch used to link chart selections to the custom
    tile
  repo: https://github.com/fred3m/toyz
- name: toyz-template
  role: Template repository forked and registered as a custom Toy ('toyz_template')
    to create the point-source inspection tile
  repo: https://github.com/fred3m/toyz-template
- name: Highcharts
  role: Interactive plotting library whose point-selection events on a shared data
    source trigger loading of cutouts in the custom tile
- name: scipy
  role: scipy.ndimage.zoom-based scaling used to tile and render surface plots of
    raw pixel values around the selected source
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:eca9046fcf3139ac4bf8bfb8519657656deb92ec66d2d370076f536df75eca15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Toyz
  - scipy
  - toyz-template
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# highcharts-point-selection-linking

## Summary

Builds a custom Toyz workspace tile that reacts to point selections made in a Highcharts chart, loading cutout images (same field in three filters) and a surface plot of raw pixel values centered on each selected point source.

## When to use

Use when a custom Toyz tile must display cutout images and surface plots for point sources selected (clicked or drag-boxed) in a Highcharts plot sharing the same data source.

## When NOT to use

- The plot and image tiles do not share the same data source, so point-selection events cannot propagate between tiles
- The goal is static visualization of a catalog or image without interactive selection — the default Toyz Highchart or image viewer alone suffices
- The remote server cannot run custom task modules (no tasks.py module registered), since cutout generation requires a server-side task dispatch

## Inputs

- Point selections (click or drag) on a Highcharts plot bound to a Toyz data source
- Images of the same astronomical field taken in three different filters (e.g. DECam FITS cutouts)
- Task parameters JSON object {module, task, parameters}

## Outputs

- Custom workspace tile displaying the cutout images for the selected source
- Surface plot of raw pixel values centered on the selected point
- Cutout image paths returned by the server-side task function

## How to apply

Fork the Toyz Template repository and register it as a custom Toy, either by running 'python setup.py install' to put 'toyz_template' on the python path or via the Toyz Paths control group with toyz_name='toyz_template' pointing at the local directory. Define the tile's GUI declaratively with the Toyz JSON parameter parser, using parameter objects with type, prop, css, func, and lbl fields wrapped in div/conditional containers. Register Toyz event handlers on the tile so point-selection events emitted by a Highcharts tile connected to the same data source propagate to the custom tile. Implement the server-side task in a module ending in 'tasks.py' with the signature (toyz_settings, tid, params) — where tid carries user_id, session_id, and request_id — returning cutout image paths for the selected source. Dispatch from the client via Toyz.Core.Websocket.send_task with the JSON task object {module, task, parameters}, then render the returned cutouts and compute the surface plot of raw pixel values, using scipy.ndimage.zoom-based scaling when tiling large (> 2 Gb DECam-class) images.

## Related tools

- **Toyz** (Web framework providing the tile system, JSON parameter parser, event-handler registry, and Websocket task dispatch used to link chart selections to the custom tile) — https://github.com/fred3m/toyz
- **toyz-template** (Template repository forked and registered as a custom Toy ('toyz_template') to create the point-source inspection tile) — https://github.com/fred3m/toyz-template
- **Highcharts** (Interactive plotting library whose point-selection events on a shared data source trigger loading of cutouts in the custom tile)
- **scipy** (scipy.ndimage.zoom-based scaling used to tile and render surface plots of raw pixel values around the selected source)

## Examples

```
cd toyz_template && python setup.py install
```

## Evaluation signals

- Selecting a point (or dragging over points) on the Highcharts chart loads the matching cutouts in the custom tile for all three filters
- A selection on one chart propagates to all tiles connected to the same data source, confirming the event-handler wiring
- The server task returns cutout image paths and the tid round-trips user_id, session_id, and request_id
- The surface plot is centered on the selected point with correct raw pixel values after scipy.ndimage.zoom-based scaling
- Each loaded cutout set can be removed from the tile when the detection is judged an artifact

## Limitations

- Very large images (> 2 Gb, e.g. DECam) require tiled rendering with scipy.ndimage.zoom scaling rather than direct display
- The template documentation is sparse — the toyz-template README notes that 'there will be more thorough documentation and a getting started tutorial for creating your own toyz' in the future
- Selected sources must be manually inspected; cutout sets corresponding to artifacts must be removed by hand

## Evidence

- [readme] Custom web pages can be written to connect interactive plots to surface plots and images from different data sources: [evidence span withheld — anchor status: external_doc]
- [abstract] create their own visualization tools that can be added on as extensions to the default Toyz framework: "create their own visualization tools that can be added on as extensions to the default Toyz framework"
- [methods] To create a new Highchart first add a data source: "To create a new Highchart first add a data source (see :ref:`add_data_source` for more)."
- [readme] Interactive plots using Highcharts (custom js plot libraries can also be used): [evidence span withheld — anchor status: external_doc]
