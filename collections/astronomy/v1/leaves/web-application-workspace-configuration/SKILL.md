---
name: web-application-workspace-configuration
description: Use when you need to stand up a browser-based Toyz workspace on a remote
  server that stores your data — e.g. to view large FITS images or build tiled viewer/chart
  workspaces without installing local software.
when_to_use_negative:
- When the data and visualization tools can run entirely locally and no remote-server,
  browser-based access is required — the article notes local-machine tools are only
  infeasible for very large remote datasets (e.g. DECam nights producing over 1 Tb).
- When the goal is batch data reduction or scripting rather than interactive, browser-based
  visualization and workspace configuration.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3452
tools:
- name: Toyz
  role: Web application framework whose first-time setup, port launch, and workspace
    tile configuration this skill performs
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Runtime environment in which Toyz is installed via 'python setup.py install'
    and 'pip install -e .[all]'
- name: scipy
  role: Optional dependency required for tiled-image scaling of large images in the
    viewer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:50ecc2aba6b41a61731d2649a5105f0c36f717223c260728aa1c934926e9cb38
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Python
  - Toyz
  - scipy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# web-application-workspace-configuration

## Summary

Configure and launch the Toyz web application so that users can access a workspace of viewer and analysis tiles in a browser against data stored on a remote server. This covers first-time setup, port forwarding, authentication hardening, and workspace tile creation.

## When to use

Use when you need to stand up a browser-based Toyz workspace on a remote server that stores your data — e.g. to view large FITS images or build tiled viewer/chart workspaces without installing local software.

## When NOT to use

- When the data and visualization tools can run entirely locally and no remote-server, browser-based access is required — the article notes local-machine tools are only infeasible for very large remote datasets (e.g. DECam nights producing over 1 Tb).
- When the goal is batch data reduction or scripting rather than interactive, browser-based visualization and workspace configuration.

## Inputs

- Toyz source repository (github.com/fred3m/toyz)
- Configuration directory (e.g. ~/toyz) for first-time setup
- SSH credentials and hostname for the remote server storing the data
- Port number for the web application (local and remote)

## Outputs

- Running Toyz web application on the specified port
- Authenticated user session with a changed admin password
- Saved Toyz workspace containing configured tiles (e.g. Viewer tile) with recorded frame, scale, and window location

## How to apply

Install Toyz from the cloned source repository (github.com/fred3m/toyz) with 'python setup.py install', adding optional dependencies via 'pip install -e .[all]' — scipy must be present for tiled-image scaling of large images. Create a configuration directory (e.g. ~/toyz), navigate into it and run 'toyz' to trigger the first-time setup, then launch the application with 'toyz --port=<remote_port>'. For a remote server, connect using ssh port forwarding ('ssh -L<local_port>:localhost:<remote_port> <username>@<hostname>') so the browser reaches the server-side app as localhost. Log in with the default admin/admin account and immediately change the password in Account Settings, since all users — especially admin — are strongly recommended to secure it. Finally, open a workspace and add the needed tiles (e.g. a Viewer tile), saving the workspace so it records how to load each viewer frame plus the scale and location of the viewer window.

## Related tools

- **Toyz** (Web application framework whose first-time setup, port launch, and workspace tile configuration this skill performs) — https://github.com/fred3m/toyz
- **Python** (Runtime environment in which Toyz is installed via 'python setup.py install' and 'pip install -e .[all]')
- **scipy** (Optional dependency required for tiled-image scaling of large images in the viewer)

## Examples

```
mkdir ~/toyz; cd ~/toyz; toyz; toyz --port=8888; ssh -L8888:localhost:8888 user@hostname
```

## Evaluation signals

- First-time setup completes without errors when 'toyz' is typed from the configuration directory.
- The application is reachable in a local browser at the forwarded port (e.g. localhost:8888) via ssh port forwarding.
- Login with the default admin/admin succeeds, and the password has been changed in Account Settings.
- A new Viewer tile can be created and saved in a workspace.
- The saved workspace records the information to load each viewer frame plus the scale and location of the viewer window.

## Limitations

- The default admin/admin account is insecure; all users, especially admin, must change the password before real use.
- Remote access requires ssh port forwarding to the specified port; without it the browser cannot reach the server-side application.
- scipy must be installed for tiled-image scaling of very large images; missing optional dependencies degrade the viewer.

## Evidence

scientists by allowing them to interact with their data via a web browser and run
scripts on a remote server"
