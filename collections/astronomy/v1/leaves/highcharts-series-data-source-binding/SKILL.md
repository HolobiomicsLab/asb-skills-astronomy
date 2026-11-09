---
name: highcharts-series-data-source-binding
description: Use when you need browser-based interactive scatter/line plots from a
  shared tabular dataset on a remote server, and want selecting points in one chart
  to highlight the same points in every chart derived from that source.
when_to_use_negative:
- You need static publication-quality figures rather than interactive browser plots
  with linked selection
- Your data is a large FITS image rather than a tabular catalog — the image viewer
  tile, not a Highchart data-source binding, is the appropriate Toyz tool
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Toyz
  role: Web application framework hosting the workspace, data source registration,
    Highcharts tiles, and linked-selection behavior
  repo: https://github.com/fred3m/toyz
- name: Highcharts
  role: JavaScript plotting library rendering the interactive series inside Toyz chart
    tiles
- name: pandas
  role: Reads the registered data file (open a data file from the server using python,
    numpy, or pandas) that backs the data source
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:79d1384769b57abf9ea0187a0ffb890b0df0db5c5b067ac053ba3f2d7fa4bac2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Toyz
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# highcharts-series-data-source-binding

## Summary

Builds interactive Highcharts tiles in a Toyz workspace whose series are bound to a registered tabular data source (e.g., a CSV/text table), so all charts sharing that source are linked and point selections propagate across them.

## When to use

Use when you need browser-based interactive scatter/line plots from a shared tabular dataset on a remote server, and want selecting points in one chart to highlight the same points in every chart derived from that source.

## When NOT to use

- You need static publication-quality figures rather than interactive browser plots with linked selection
- Your data is a large FITS image rather than a tabular catalog — the image viewer tile, not a Highchart data-source binding, is the appropriate Toyz tool

## Inputs

- Tabular data file (e.g., CSV or text table) registered as a Toyz data source
- Toyz web application running on a remote server (default port 8888) with an authenticated user session

## Outputs

- One or more interactive Highcharts tiles with series bound to the shared data source
- Saved workspace file (e.g., linked_selection_workspace) preserving chart tiles and settings
- Selection propagation report recording whether selected point(s) in one chart appear selected in sibling charts

## How to apply

First register a tabular data file (CSV/text table) as a data source via the Add Data Source dialog, because in Toyz a data source must exist before a Highchart can be created. Create a Highcharts tile, open the Edit Chart dialog, and add a series bound to the registered data source, choosing the x and y columns from the data source select box; repeat for a second tile using the same source. Save the workspace (e.g., linked_selection_workspace) so both tiles persist with their chart settings. Then click-drag to select a point or region in one chart and verify the same point(s) are selected in every other chart connected to that data source — Toyz links all plots generated from the same data source.

## Related tools

- **Toyz** (Web application framework hosting the workspace, data source registration, Highcharts tiles, and linked-selection behavior) — https://github.com/fred3m/toyz
- **Highcharts** (JavaScript plotting library rendering the interactive series inside Toyz chart tiles)
- **pandas** (Reads the registered data file (open a data file from the server using python, numpy, or pandas) that backs the data source)

## Examples

```
toyz --port=8888
```

## Evaluation signals

- Both chart tiles persist after saving the workspace (linked_selection_workspace) and reload with their series/column bindings intact
- Selecting a single point in the first chart marks the identical point selected in the second chart bound to the same data source
- Click-drag selection of a region propagates the full set of selected points to every sibling chart
- Charts built from a different data source do NOT show the propagated selection (linkage is per-source)
- Selection propagation outcomes are recorded in selection_propagation_report

## Limitations

- A Highchart cannot be created until the data source is added; skipping data source registration breaks chart setup
- Linkage depends on charts sharing the same data source — charts from different sources are not linked
- Requires the Toyz web application to be running and reachable (e.g., via ssh port forwarding to the remote port)

## Evidence

- [methods] Data source must precede Highchart creation: "To create a new Highchart first add a data source (see :ref:`add_data_source` for more)."
- [readme] Highcharts provide interactive plots in Toyz: [evidence span withheld — anchor status: external_doc]
