---
name: fits-image-tiled-rendering
description: Use when a user needs to inspect or navigate a >2 Gb DECam or other large-FOV
  FITS image stored remotely, without local storage, local software installation,
  or loading the whole image into memory.
when_to_use_negative:
- The image is small or already stored locally, where a local astronomy viewer (e.g.
  DS9-style tools) is adequate
- The task is catalog/plot analysis rather than image viewing — use Toyz data sources
  and Highcharts workflows instead
- The user cannot run scripts or port-forward to the server that stores the data
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Toyz
  role: Web application framework whose image viewer opens the >2 Gb FITS file as
    mosaic tiles served from the remote server
  repo: https://github.com/fred3m/toyz
- name: scipy
  role: Required dependency for scaling tiled images in the Toyz image viewer
- name: Astro-Toyz
  role: Addon astro-viewer tile built on the Toyz image viewer with WCS and region-selection
    support for FITS images
  repo: https://github.com/fred3m/astro-toyz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:52d39bc83bca183bf3ae71d443259822c8f261b7635b1b6d749a9eee8c84d2a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Astro-Toyz
  - Toyz
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# fits-image-tiled-rendering

## Summary

Browser-based viewing of very large (>2 Gb) FITS astronomical images stored on a remote server using Toyz's tiled (mosaic) image viewer, which only loads the viewed sections rather than the full image.

## When to use

Use when a user needs to inspect or navigate a >2 Gb DECam or other large-FOV FITS image stored remotely, without local storage, local software installation, or loading the whole image into memory.

## When NOT to use

- The image is small or already stored locally, where a local astronomy viewer (e.g. DS9-style tools) is adequate
- The task is catalog/plot analysis rather than image viewing — use Toyz data sources and Highcharts workflows instead
- The user cannot run scripts or port-forward to the server that stores the data

## Inputs

- Large (>2 Gb) FITS image file stored on the remote server (e.g. a DECam exposure)
- Toyz configuration directory (e.g. ~/toyz)
- SSH credentials and port number for remote access

## Outputs

- Tiled (mosaic) rendering of the FITS image in the user's web browser
- Saved Toyz workspace viewer tile recording image-load information and viewer window scale/location

## How to apply

Install Toyz from the cloned source (github.com/fred3m/toyz) with 'python setup.py install' plus optional dependencies via 'pip install -e .[all]', making sure scipy is present since it is needed for tiled-image scaling. Create a configuration directory (e.g. ~/toyz), run first-time setup by typing 'toyz', then launch the server with 'toyz --port=8888' and reach it remotely through ssh port forwarding ('ssh -L8888:localhost:8888 user@hostname'). Log in with the default admin account (admin/admin), change the password in Account Settings, and add a new Viewer tile in a workspace. Click 'load image', choose the 'large image' image type, and select the >2 Gb FITS file on the server so it is opened as a set of mosaic tiles; only viewed sections are loaded. Navigate, zoom with the rectangular zoom tool, and rescale the image, then save the viewer tile so the workspace records the load information plus the viewer window's scale and location.

## Related tools

- **Toyz** (Web application framework whose image viewer opens the >2 Gb FITS file as mosaic tiles served from the remote server) — https://github.com/fred3m/toyz
- **scipy** (Required dependency for scaling tiled images in the Toyz image viewer)
- **Astro-Toyz** (Addon astro-viewer tile built on the Toyz image viewer with WCS and region-selection support for FITS images) — https://github.com/fred3m/astro-toyz

## Examples

```
toyz --port=8888
```

## Evaluation signals

- Tiles render smoothly when navigating and zooming across the full large image in the browser
- Only viewed sections of the image are loaded as mosaic tiles, never the full >2 Gb image
- Saving the viewer tile persists the information needed to load each viewer frame plus the scale and location of the viewer window
- Access works remotely through ssh port forwarding without any local installation of viewing software

## Limitations

- scipy must be installed, otherwise tiled-image scaling will not work
- Requires the data to reside on a server the user can run Toyz on and reach via port forwarding; local-only data defeats the design
- Astro-Toyz is still under active development (e.g. displaying source catalogs on the image viewer is listed as future work)

## Evidence

- [abstract] abstract: "quickly view very large images (> 2 Gb) taken with DECam and other cameras with a large FOV"
- [methods] methods: "Clicking the **load image** button open up a dialog that allows the user to choose an image type."
- [readme] readme: "Toyz was initially created as a way to view and interact with FITS images stored on a remote server"
