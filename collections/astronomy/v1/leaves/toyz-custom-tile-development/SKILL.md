---
name: toyz-custom-tile-development
description: Use when you need a browser-based tile that reacts to point selections
  made in a Highcharts chart and loads matching image cutouts (e.g. DECam multi-filter
  frames of the same field) for visual inspection.
when_to_use_negative:
- You only need a static image viewer or standard Highchart without custom interaction
  — default Toyz tiles suffice without writing a custom module.
- Your images and catalogs are stored locally and you want a desktop tool; Toyz is
  designed for data stored on a remote server, accessed via a web browser.
- You need fully non-interactive batch processing rather than human-in-the-loop source
  inspection.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3315
tools:
- name: Toyz
  role: Web application framework providing the workspace, JSON parameter parser,
    event system, Websocket task dispatch, and image viewer on which the custom tile
    is built
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Language for the custom Toy module, including the tasks.py server-side task
    function and setup.py installation
  repo: https://github.com/fred3m/toyz-template
- name: Highcharts
  role: Interactive plotting library whose point-selection events are linked to the
    custom tile via the shared data source
  repo: https://github.com/fred3m/toyz
- name: numpy
  role: Scientific processing dependency used in Toyz for handling image/catalog data
- name: scipy
  role: scipy.ndimage.zoom is used for scaling cutouts when computing tiled surface
    plots of raw pixel values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:a4ee0ef6c0d43f17de7643413fa518161ca40a002a519d2e64ac331401203565
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Python
  - Toyz
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# toyz-custom-tile-development

## Summary

Build a custom Toyz workspace tile that loads images and surface plots of point sources selected in a linked Highcharts plot, using the Toyz Template repository as the scaffolding for declarative GUI parameters, event handlers, and server-side tasks.

## When to use

Use when you need a browser-based tile that reacts to point selections made in a Highcharts chart and loads matching image cutouts (e.g. DECam multi-filter frames of the same field) for visual inspection.

## When NOT to use

- You only need a static image viewer or standard Highchart without custom interaction — default Toyz tiles suffice without writing a custom module.
- Your images and catalogs are stored locally and you want a desktop tool; Toyz is designed for data stored on a remote server, accessed via a web browser.
- You need fully non-interactive batch processing rather than human-in-the-loop source inspection.

## Inputs

- Forked Toyz Template repository (github.com/fred3m/toyz-template)
- Images of the same field in three different filters (e.g. DECam FITS frames)
- Highcharts plot tile connected to a shared data source of point sources
- Custom module 'tasks.py' file on the Toyz server

## Outputs

- Custom workspace tile displaying image cutouts per selected source
- Surface plot of raw pixel values centered on the selected point
- Removable tile entries for detections judged to be artifacts

## How to apply

Fork the Toyz Template repository and register it as a custom Toy, either by running 'python setup.py install' for the 'toyz_template' module or by adding it via the Toyz Paths control group (toyz_name='toyz_template', path to the local directory). Define the tile's GUI controls declaratively with the Toyz JSON parameter parser, using parameter objects with type, prop, css, func, and lbl fields wrapped in div/conditional containers. Register Toyz event handlers on the tile so point-selection events from a Highcharts tile sharing the same data source propagate to the custom tile. Implement the server-side task in a module ending in 'tasks.py' with the signature (toyz_settings, tid, params), where tid carries user_id, session_id, and request_id, and return cutout image paths for the selected source; dispatch the job from the client via Toyz.Core.Websocket.send_task with {module, task, parameters}. Render the returned cutouts and compute the surface plot of raw pixel values centered on the selected point, using scipy.ndimage.zoom-based scaling for tiled large images.

## Related tools

- **Toyz** (Web application framework providing the workspace, JSON parameter parser, event system, Websocket task dispatch, and image viewer on which the custom tile is built) — https://github.com/fred3m/toyz
- **Python** (Language for the custom Toy module, including the tasks.py server-side task function and setup.py installation) — https://github.com/fred3m/toyz-template
- **Highcharts** (Interactive plotting library whose point-selection events are linked to the custom tile via the shared data source) — https://github.com/fred3m/toyz
- **numpy** (Scientific processing dependency used in Toyz for handling image/catalog data)
- **scipy** (scipy.ndimage.zoom is used for scaling cutouts when computing tiled surface plots of raw pixel values)

## Examples

```
python setup.py install  # register the forked toyz_template module, or add it via the Toyz Paths control group (toyz_name='toyz_template')
```

## Evaluation signals

- Selecting a point (or dragging over points) on the Highcharts chart loads the matching cutouts in the custom tile for all three filters.
- The surface plot shown is centered on the selected point and reflects raw pixel values of the selected source.
- The server-side task returns cutout image paths for the selected source, with tid fields (user_id, session_id, request_id) correctly propagated.
- Point-selection events from the Highcharts tile propagate to all tiles connected to the same data source.
- Tile entries can be removed when the detection is judged to be an artifact.

## Limitations

- The tile is tied to Toyz's linked-tile behavior: it only receives selections from Highcharts tiles sharing the same data source.
- Very large images require tiled rendering with scipy.ndimage.zoom-based scaling rather than direct display.
- The Toyz Template README notes more thorough documentation and a getting-started tutorial for creating new toyz are still forthcoming.

## Evidence

- [other] custom tile displays image and surface plot: [evidence span withheld — anchor status: not_found]
- [methods] template registration: [evidence span withheld — anchor status: not_found]
- [other] task function signature: [evidence span withheld — anchor status: not_found]
- [readme] README template purpose: [evidence span withheld — anchor status: not_found]
- [readme] custom tiles in README: [evidence span withheld — anchor status: external_doc]
