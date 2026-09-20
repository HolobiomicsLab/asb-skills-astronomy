---
name: ssh-port-forwarding-configuration
description: Use when data (e.g. >2 Gb DECam FITS images) resides on a remote server
  and a browser-based viewer must be reached from a local machine without local storage
  or software installation.
when_to_use_negative:
- Data and Toyz are already on the local machine — a direct 'toyz --port=8888' launch
  and localhost URL suffice without an SSH tunnel
- A shared deployment already exposes Toyz over HTTPS on the institutional network,
  making manual port forwarding redundant
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Toyz
  role: Web application launched on the remote server ('toyz --port=<remote_port>')
    that the SSH tunnel exposes to the local browser
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Runtime environment in which Toyz is installed and launched on the remote
    server
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:09662a0d47866226f5e47ddfe6ba0f5adbcdd2846aac3a4cdf2df7f22d19ad0e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Python
  - Toyz
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ssh-port-forwarding-configuration

## Summary

Configure an SSH tunnel from a local machine to a remote server so the Toyz web application (serving large DECam FITS images) can be accessed in a local browser without installing software or moving data.

## When to use

Use when data (e.g. >2 Gb DECam FITS images) resides on a remote server and a browser-based viewer must be reached from a local machine without local storage or software installation.

## When NOT to use

- Data and Toyz are already on the local machine — a direct 'toyz --port=8888' launch and localhost URL suffice without an SSH tunnel
- A shared deployment already exposes Toyz over HTTPS on the institutional network, making manual port forwarding redundant

## Inputs

- Remote server hostname and SSH username
- Toyz application port (e.g. 8888) set via 'toyz --port=<remote_port>'
- Local terminal with SSH access to the remote server

## Outputs

- An SSH tunnel mapping a local port to the remote Toyz port
- Browser access to the Toyz web application (login as admin/admin, then workspace with Viewer tile)

## How to apply

After launching Toyz on the remote server with 'toyz --port=<remote_port>' from the configuration directory, open a local terminal and set up port forwarding with 'ssh -L<local_port>:localhost:<remote_port> <username>@<hostname>', choosing the same port number on both sides (e.g. 8888). Then point the local browser at the forwarded local port to reach the Toyz web application running on the server that stores the data. This keeps all data on the remote server, matching Toyz's design goal of platform-independent, browser-based access without installing software locally.

## Related tools

- **Toyz** (Web application launched on the remote server ('toyz --port=<remote_port>') that the SSH tunnel exposes to the local browser) — https://github.com/fred3m/toyz
- **Python** (Runtime environment in which Toyz is installed and launched on the remote server)

## Examples

```
ssh -L8888:localhost:8888 user@hostname  # then, on the server: toyz --port=8888
```

## Evaluation signals

- The local browser reaches the Toyz login page at http://localhost:<local_port> once the tunnel is active
- Login with the default admin/admin account succeeds, confirming the tunnel reaches the remote Toyz server (password then changed in Account Settings)
- The remote_port passed to 'toyz --port=' matches the <remote_port> in the 'ssh -L' command, and the local port forwards to it
- A >2 Gb FITS image loads as mosaic tiles in the viewer over the tunnel, confirming data stays on the remote server

## Limitations

- Tunnel must remain active; closing the local SSH session severs browser access to the Toyz workspace
- Port conflicts: both the local and remote ports must be free and consistent between the 'ssh -L' command and 'toyz --port='

## Evidence
