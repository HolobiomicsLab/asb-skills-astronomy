---
name: toyz-workspace-configuration
description: Use when building a Toyz workspace with two or more interactive Highcharts
  tiles bound to the same registered tabular data source (e.g., a CSV/text table)
  and you need linked point/region selection across all charts.
when_to_use_negative:
- You need a single static plot with no cross-chart selection linking — a plain matplotlib/pandas
  plot suffices and the linked-selection machinery adds no value
- The task is viewing large astronomical images (e.g., > 2 Gb DECam FITS images) rather
  than linked tabular Highcharts — use the Toyz image viewer or Astro-Toyz tile instead
- No Toyz server is available or the data cannot be registered as a workspace data
  source, since a data source must exist before any Highchart can be created
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Toyz
  role: Web application framework providing the workspace, data source registration,
    Highcharts tiles, and linked selection across charts
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Underlying language in which Toyz runs and through which the server-side data
    sources are served
  repo: https://github.com/fred3m/toyz
- name: pandas
  role: Used by Toyz to open tabular data files registered as data sources
- name: Highcharts
  role: JavaScript plotting library behind the interactive Highcharts tiles whose
    selections are linked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:996c2c875c2cc0958d1b9173cdc5c1865305aa8047064a49861e479964f78b13
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Python
  - Toyz
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# toyz-workspace-configuration

## Summary

Configure a Toyz web workspace in which multiple interactive Highcharts tiles are bound to a single shared tabular data source, so that point selections in one plot propagate to all plots generated from that source. This exploits Toyz's linked-selection behavior across charts in the same workspace.

## When to use

Use when building a Toyz workspace with two or more interactive Highcharts tiles bound to the same registered tabular data source (e.g., a CSV/text table) and you need linked point/region selection across all charts.

## When NOT to use

- You need a single static plot with no cross-chart selection linking — a plain matplotlib/pandas plot suffices and the linked-selection machinery adds no value
- The task is viewing large astronomical images (e.g., > 2 Gb DECam FITS images) rather than linked tabular Highcharts — use the Toyz image viewer or Astro-Toyz tile instead
- No Toyz server is available or the data cannot be registered as a workspace data source, since a data source must exist before any Highchart can be created

## Inputs

- Tabular data file (CSV/text table) registered as a Toyz data source
- Toyz web application running on a server (remote access via ssh port forwarding)
- Admin or user account credentials for the Toyz workspace

## Outputs

- Saved Toyz workspace (e.g., linked_selection_workspace) with two Highcharts tiles bound to the shared data source
- selection_propagation_report recording whether selections propagate between linked charts

## How to apply

Launch the Toyz web application on a specified port (default 8888) and log in — ideally after changing the default admin password in Account Settings, as the documentation strongly recommends. Register the tabular file as a data source via the Add Data Source dialog before creating any chart, since the Toyz docs state a data source must be added first. Create a Highcharts tile, open the Edit Chart dialog, and add a series bound to the registered data source, picking x and y columns from the data source select box; repeat for a second tile using the same data source. Save the workspace (e.g., linked_selection_workspace) so both tiles persist with their chart settings. Validate by selecting a point (or click-dragging a region for multiple points) in the first chart and inspecting the second chart's selection state via the Toyz web client API, recording results in a selection_propagation_report.

## Related tools

- **Toyz** (Web application framework providing the workspace, data source registration, Highcharts tiles, and linked selection across charts) — https://github.com/fred3m/toyz
- **Python** (Underlying language in which Toyz runs and through which the server-side data sources are served) — https://github.com/fred3m/toyz
- **pandas** (Used by Toyz to open tabular data files registered as data sources)
- **Highcharts** (JavaScript plotting library behind the interactive Highcharts tiles whose selections are linked)

## Examples

```
toyz --port=8888
```

## Evaluation signals

- Both Highcharts tiles appear in the same saved workspace with series bound to the registered data source, with x and y columns chosen from the data source select box
- Selecting a single point in the first chart programmatically selects the identical point in the second chart (verified via the Toyz web client API)
- Click-drag selection of a region of points in one chart selects the same collection of points in every other chart from that source
- Workspace persists after save as linked_selection_workspace with chart settings intact on reload
- selection_propagation_report records positive propagation for every chart connected to the shared data source

## Limitations

- A data source must be added before creating a Highchart; you cannot bind a chart series without a prior Add Data Source step
- Linked selection only holds among plots generated from the same data source — charts on different sources will not propagate selections
- The default admin password must be changed (Account Settings), and remote access typically requires ssh port forwarding to the port the toyz application was launched on
- Highcharts is the default plotting library; custom js plot libraries can be substituted but their selection behavior may differ

## Evidence

- [full_text] Toyz links all plots generated from the same data source: [evidence span withheld — anchor status: not_found]
- [readme] Interactive plots using Highcharts: [evidence span withheld — anchor status: external_doc]
- [methods] Launch on a specified port: [evidence span withheld — anchor status: not_found]
