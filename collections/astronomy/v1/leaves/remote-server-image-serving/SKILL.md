---
name: remote-server-image-serving
description: Use when a user needs to view or interact with a very large (>2 Gb) FITS
  image stored on a remote server or cluster directly in a browser, without downloading
  the file or installing local visualization software.
when_to_use_negative:
- The image is small enough to load locally, or the data are already stored on the
  user's own machine — Toyz's advantage is serving data that remains on the remote
  server.
- The goal is catalog/data-frame analysis rather than image viewing — use a Toyz data
  source with Highcharts plots instead of the 'large image' viewer type.
- The user needs a fully installed desktop astronomy visualization tool with local
  files; the article notes existing tools must run on a local machine with locally
  stored data, which is precisely the situation this skill avoids but also its scope
  limit.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3315
tools:
- name: Toyz
  role: Web application framework whose image viewer tiles the large remote FITS image
    and serves it to the browser via ssh port forwarding
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Runtime in which Toyz is installed and launched ('python setup.py install',
    'toyz --port=8888')
- name: scipy
  role: Dependency required for tiled-image scaling in the Toyz viewer
- name: numpy
  role: Scientific image/data processing dependency used by Toyz for reading image
    data
- name: Dark Energy Camera (DECam)
  role: Instrument that produced the >2 Gb FITS images motivating and validating this
    tiled remote-viewing workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:f27b8d4a315e8a0544a8016ac83a8279979357a2b80a4166e0099a8533eccc82
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Dark Energy Camera (DECam)
  - Python
  - Toyz
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# remote-server-image-serving

## Summary

Serve very large (>2 Gb) FITS astronomical images (e.g. from DECam) from a remote server to a user's web browser via Toyz's tiled/mosaic rendering, so no local storage or software installation is needed.

## When to use

Use when a user needs to view or interact with a very large (>2 Gb) FITS image stored on a remote server or cluster directly in a browser, without downloading the file or installing local visualization software.

## When NOT to use

- The image is small enough to load locally, or the data are already stored on the user's own machine — Toyz's advantage is serving data that remains on the remote server.
- The goal is catalog/data-frame analysis rather than image viewing — use a Toyz data source with Highcharts plots instead of the 'large image' viewer type.
- The user needs a fully installed desktop astronomy visualization tool with local files; the article notes existing tools must run on a local machine with locally stored data, which is precisely the situation this skill avoids but also its scope limit.

## Inputs

- Very large (>2 Gb) FITS image file (e.g. DECam or other large-FOV camera image) stored on the remote server
- Toyz configuration directory with server/port settings
- User login credentials (default admin/admin on first run)

## Outputs

- Browser-rendered tiled/mosaic view of the remote FITS image (only viewed sections loaded)
- Saved Toyz workspace viewer tile recording scale and location of the viewer window for each frame

## How to apply

Install Toyz from the cloned source repository (github.com/fred3m/toyz) with 'python setup.py install', adding optional dependencies via 'pip install -e .[all]' — scipy must be present for tiled-image scaling. Create a configuration directory (e.g. ~/toyz), run first-time setup by typing 'toyz' there, then launch with 'toyz --port=8888'; connect remotely using ssh port forwarding ('ssh -L8888:localhost:8888 user@hostname'). Log in with the default admin/admin account and change the password in Account Settings, then add a Viewer tile in a workspace. Click 'load image', choose the 'large image' type, and select the >2 Gb FITS file on the server so it opens as mosaic tiles that load only viewed sections. Navigate, zoom (rectangular zoom tool), and scale the tiled image, saving the viewer tile so the workspace records the scale and location of each viewer frame.

## Related tools

- **Toyz** (Web application framework whose image viewer tiles the large remote FITS image and serves it to the browser via ssh port forwarding) — https://github.com/fred3m/toyz
- **Python** (Runtime in which Toyz is installed and launched ('python setup.py install', 'toyz --port=8888'))
- **scipy** (Dependency required for tiled-image scaling in the Toyz viewer)
- **numpy** (Scientific image/data processing dependency used by Toyz for reading image data)
- **Dark Energy Camera (DECam)** (Instrument that produced the >2 Gb FITS images motivating and validating this tiled remote-viewing workflow)

## Examples

```
git clone git://github.com/fred3m/toyz.git && cd toyz && pip install -e .[all]; toyz --port=8888
```

## Evaluation signals

- The >2 Gb FITS file is opened as a set of mosaic tiles and only the viewed sections load, rather than the full image being read into memory.
- Tiles render smoothly across the large image while navigating, using the rectangular zoom tool, and rescaling in the browser.
- The viewer tile, when saved, records the information needed to load each viewer frame plus the scale and location of the viewer window.
- The remote session is reachable through ssh port forwarding ('ssh -L8888:localhost:8888 user@hostname') with the server launched on the matching port ('toyz --port=8888').
- The default admin/admin password has been changed in Account Settings, confirming first-time setup completed correctly.

## Limitations

- scipy must be installed for tiled-image scaling of large images to work.
- The default admin/admin credentials exist on first run; all users (especially admin) must change the password in Account Settings.
- Astro-Toyz is still undergoing active development; features like displaying source catalogs on the image viewer were planned but not yet present.
- Existing astronomy visualization tools must run on a local machine with data stored locally — which is no longer feasible for instruments like DECam generating over 1 Tb of data products per night — motivating, but also delimiting, the remote approach.

## Evidence
