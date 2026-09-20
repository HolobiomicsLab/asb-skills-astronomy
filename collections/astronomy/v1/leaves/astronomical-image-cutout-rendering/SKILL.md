---
name: astronomical-image-cutout-rendering
description: 'Use when interactive inspection of individual astronomical detections
  is needed: a catalog or Highcharts plot of point sources exists and, for each selected
  source, cutouts from same-field images in multiple filters must be rendered alongside
  a surface plot.'
when_to_use_negative:
- The data is not imagery or the goal is purely tabular/chart analysis — use the standard
  Toyz Highchart tile without a custom cutout tile
- Images cannot be served from the remote Toyz server (e.g. local-only tooling workflow),
  since the whole pipeline depends on server-side tasks returning cutout paths over
  the Toyz Websocket
- 'A ready-made viewer suffices: Astro-Toyz already provides an astro-viewer tile
  with WCS and region selection, so a fully custom tile is unnecessary'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3391
tools:
- name: Toyz
  role: Web framework providing the tile system, data sources, websocket task dispatch
    (Toyz.Core.Websocket.send_task), image viewer, and event-handler registry used
    to link the Highcharts selection to the custom tile
  repo: https://github.com/fred3m/toyz
- name: toyz-template
  role: Starting repository forked and installed (python setup.py install or via Toyz
    Paths) to create the custom point-source inspection tile
  repo: https://github.com/fred3m/toyz-template
- name: Astro-Toyz
  role: Addon package whose astro-viewer tile supports surface plots, pixel distribution
    histograms, and WCS on top of the Toyz image viewer
  repo: https://github.com/fred3m/astro-toyz
- name: Highcharts
  role: Interactive plotting tile whose point-selection events propagate to all tiles
    sharing the same data source, triggering cutout loading
- name: scipy
  role: scipy.ndimage.zoom used for scaling so large images can be rendered as tiled
    surface plots of raw pixel values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:f2acd12aea7af3ca0c9ddd91a48f6c5faf5f8425d7569caa9f802fbdc3cc9aa6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Astro-Toyz
  - Highcharts
  - Toyz
  - scipy
  - toyz-template
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# astronomical-image-cutout-rendering

## Summary

Build a custom Toyz workspace tile that loads cutout images (from multi-filter DECam-style exposures) and a surface plot of raw pixel values for point sources selected in a linked Highcharts plot, so detections can be inspected and rejected if they are artifacts.

## When to use

Use when interactive inspection of individual astronomical detections is needed: a catalog or Highcharts plot of point sources exists and, for each selected source, cutouts from same-field images in multiple filters must be rendered alongside a surface plot.

## When NOT to use

- The data is not imagery or the goal is purely tabular/chart analysis — use the standard Toyz Highchart tile without a custom cutout tile
- Images cannot be served from the remote Toyz server (e.g. local-only tooling workflow), since the whole pipeline depends on server-side tasks returning cutout paths over the Toyz Websocket
- A ready-made viewer suffices: Astro-Toyz already provides an astro-viewer tile with WCS and region selection, so a fully custom tile is unnecessary

## Inputs

- Same-field astronomical images (e.g. DECam FITS) taken in three different filters
- Source catalog exposed as a Toyz data source and rendered as a Highcharts plot
- Point-selection events (single clicks or drag-select) from the Highcharts tile
- Toyz Template repository (toyz_template module) as the starting codebase

## Outputs

- Custom Toyz workspace tile displaying cutout images for each selected point source
- Surface plot of raw pixel values centered on the selected point
- Server-side tasks.py function returning cutout image paths for the selected source
- Removable tile content for detections judged to be artifacts

## How to apply

Register a custom Toy by forking the Toyz Template repository and either running 'python setup.py install' for the 'toyz_template' module or adding its local directory via the Toyz Paths control group (toyz_name='toyz_template'). Define the tile's GUI declaratively with the Toyz JSON parameter parser (parameter objects with type, prop, css, func, and lbl fields, wrapped in div/conditional containers) and register Toyz event handlers so point-selection events from a Highcharts tile sharing the same data source propagate to the custom tile. Implement the server-side task in a module ending in 'tasks.py' with the required signature (toyz_settings, tid, params), where tid carries user_id, session_id, and request_id, and have it return cutout image paths for the selected source. Dispatch the job client-side via Toyz.Core.Websocket.send_task with a JSON task object {module, task, parameters}, render the returned cutouts in the viewer, and compute the surface plot of raw pixel values centered on the selected point, using scipy.ndimage.zoom-based scaling to tile large images.

## Related tools

- **Toyz** (Web framework providing the tile system, data sources, websocket task dispatch (Toyz.Core.Websocket.send_task), image viewer, and event-handler registry used to link the Highcharts selection to the custom tile) — https://github.com/fred3m/toyz
- **toyz-template** (Starting repository forked and installed (python setup.py install or via Toyz Paths) to create the custom point-source inspection tile) — https://github.com/fred3m/toyz-template
- **Astro-Toyz** (Addon package whose astro-viewer tile supports surface plots, pixel distribution histograms, and WCS on top of the Toyz image viewer) — https://github.com/fred3m/astro-toyz
- **Highcharts** (Interactive plotting tile whose point-selection events propagate to all tiles sharing the same data source, triggering cutout loading)
- **scipy** (scipy.ndimage.zoom used for scaling so large images can be rendered as tiled surface plots of raw pixel values)

## Examples

```
git clone git://github.com/fred3m/toyz.git && cd toyz-template && python setup.py install
```

## Evaluation signals

- Selecting a point (or dragging over points) on the Highcharts chart loads the matching cutouts and surface plot in the custom tile, confirming the shared-data-source event linkage
- Each set of displayed images corresponds to one selected source and to the same field taken in three different filters
- The tasks.py function accepts the required signature (toyz_settings, tid, params) with tid carrying user_id, session_id, and request_id, and returns valid cutout image paths
- The client task JSON sent via Toyz.Core.Websocket.send_task contains exactly the {module, task, parameters} keys
- Artifactory detections can be removed from the tile without breaking subsequent selections

## Limitations

- Very large images require tiled rendering with scipy.ndimage.zoom-based scaling; naive full-image surface plots will not scale
- The template repository documentation is sparse — 'In the future there will be more thorough documentation and a getting started tutorial', so setup relies on the paper's workflow
- Tile behavior depends on Toyz event propagation across tiles connected to the same data source; unlinked tiles will not receive point-selection events

## Evidence

- [abstract] quickly view very large images (> 2 Gb) taken with DECam and other cameras with a large FOV: "quickly view very large images (> 2 Gb) taken with DECam and other cameras with a large FOV"
- [methods] Toyz Paths registration of toyz_template: "This time click the ``+`` sign in the ``Toyz Paths`` control group. In the ``toyz_name`` field type
``toyz_template``"
- [methods] load image dialog: "Clicking the **load image** button open up a dialog that allows
the user to choose an image type."
- [readme] surface plots via Astro-Toyz: "With the [Astro-Toyz](https://github.com/fred3m/astro-toyz) package you can
view surface plots, pixel distribution histograms, and WCS."
- [readme] custom pages connecting plots to images: "Custom web pages can be written to connect interactive plots to surface plots and
images from different data sources"
