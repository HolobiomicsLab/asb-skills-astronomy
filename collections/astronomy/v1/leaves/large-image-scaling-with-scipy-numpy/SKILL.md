---
name: large-image-scaling-with-scipy-numpy
description: Use when a user needs to view or navigate a very large (>2 Gb) FITS image
  stored on a remote server in a browser, without downloading the full file or installing
  local visualization software.
when_to_use_negative:
- The image is small enough to load fully in a local viewer (e.g. DS9 or Ginga), where
  tiled remote rendering adds needless setup
- The data are catalogs or tables rather than images — use Toyz's data-source/Highchart
  plotting workflow instead of the image viewer
- No server-side access is available and the image must be analyzed entirely offline
  on a local machine
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Toyz
  role: Web application framework providing the browser-based image viewer with tiled
    mosaic rendering of large FITS files
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Language in which Toyz is implemented and run on the remote server hosting
    the data
  repo: https://github.com/fred3m/toyz
- name: numpy
  role: Scientific image and data processing module underpinning image handling and
    tiled-image scaling
- name: scipy
  role: Required for tiled-image scaling of the large-image tiles in the Toyz viewer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:5085192666aa843ad669477474e71051e4b501c7b35603256eed900cc158a074
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Python
  - Toyz
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# large-image-scaling-with-scipy-numpy

## Summary

Render very large (>2 Gb) FITS astronomical images, such as those from DECam, in a web browser via Toyz's tiled mosaic viewer, using numpy and scipy on the server for tiled-image scaling so only viewed sections are loaded.

## When to use

Use when a user needs to view or navigate a very large (>2 Gb) FITS image stored on a remote server in a browser, without downloading the full file or installing local visualization software.

## When NOT to use

- The image is small enough to load fully in a local viewer (e.g. DS9 or Ginga), where tiled remote rendering adds needless setup
- The data are catalogs or tables rather than images — use Toyz's data-source/Highchart plotting workflow instead of the image viewer
- No server-side access is available and the image must be analyzed entirely offline on a local machine

## Inputs

- Very large (>2 Gb) FITS astronomical image stored on the remote server (e.g. DECam or other large-FOV camera image)
- Toyz configuration directory (e.g. ~/toyz) with user accounts set up

## Outputs

- Browser-rendered tiled (mosaic) view of the FITS image, with zoom and scaling applied only to viewed tiles
- Saved viewer tile in the Toyz workspace recording the information to load each viewer frame plus the scale and location of the viewer window

## How to apply

Install Toyz from the cloned source repository (github.com/fred3m/toyz) with 'python setup.py install' plus optional dependencies via 'pip install -e .[all]', ensuring scipy is present since it performs the tiled-image scaling; numpy supplies the underlying image/data processing. Create a configuration directory (e.g. ~/toyz), run first-time setup by typing 'toyz' inside it, then launch the application with 'toyz --port=8888' and connect remotely through ssh port forwarding ('ssh -L8888:localhost:8888 user@hostname'). Log in with the default admin account and immediately change the password in Account Settings. In a workspace, create a Viewer tile, click 'load image', choose the 'large image' image type, and select the >2 Gb FITS file on the server so it is opened as a set of mosaic tiles that only load viewed sections. Navigate, zoom with the rectangular zoom tool, and adjust scaling in the browser; the server-side scipy/numpy scaling means only the tiles in the viewer window are computed and transmitted.

## Related tools

- **Toyz** (Web application framework providing the browser-based image viewer with tiled mosaic rendering of large FITS files) — https://github.com/fred3m/toyz
- **Python** (Language in which Toyz is implemented and run on the remote server hosting the data) — https://github.com/fred3m/toyz
- **numpy** (Scientific image and data processing module underpinning image handling and tiled-image scaling)
- **scipy** (Required for tiled-image scaling of the large-image tiles in the Toyz viewer)

## Examples

```
toyz --port=8888
```

## Evaluation signals

- The >2 Gb FITS file opens in the browser as mosaic tiles without loading the full image or storing it locally
- Tiles render smoothly while navigating and using the rectangular zoom tool across the large image
- Scaling changes applied in the viewer respond correctly (scipy present), rather than failing to rescale tiles
- Saving the viewer tile records the information needed to load each viewer frame plus the scale and location of the viewer window
- Connection was established via ssh port forwarding and the chosen port (e.g. 8888), and the default admin password was changed in Account Settings

## Limitations

- Existing astronomy visualization tools that must run on a local machine with data stored locally are not feasible for instruments like DECam generating over 1 Tb of data products per night — this skill requires server-side deployment
- scipy must be installed for tiled-image scaling; without it the large-image tiles will not scale correctly
- The default admin/admin account must be changed to something more secure before exposing the viewer to real users

## Evidence
