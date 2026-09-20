---
name: interactive-plot-selection-propagation-verification
description: Use when multiple interactive Highcharts tiles in a Toyz workspace share
  one registered tabular data source and you must confirm that selecting a point in
  one chart selects the same points in all other charts from that source.
when_to_use_negative:
- Plots are generated from different data sources — Toyz links only charts from the
  same source
- Static or non-interactive plot images where no selection state can be inspected
  via the web client API
- Single-chart analysis where no second chart exists to verify propagation against
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Toyz
  role: Web application framework hosting the Highcharts tiles, data source registration,
    and the web client API used to inspect selection state
  repo: https://github.com/fred3m/toyz
- name: Highcharts
  role: Interactive plot library whose tiles carry the linked selection state under
    test
- name: Python
  role: Language in which Toyz is implemented and through which the web client API
    is exercised
  repo: https://github.com/fred3m/toyz
- name: pandas
  role: Backend for opening tabular data files as Toyz data sources
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:4f940ba3c3a76ff1dcb40526c26d476a931251500a3aa98643ca11b446b04f8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Python
  - Toyz
  - pandas
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# interactive-plot-selection-propagation-verification

## Summary

Verifies that Toyz's linked-selection behavior propagates point or region selections from one Highcharts plot to every other plot generated from the same registered data source, using the Toyz web client API to inspect the second chart's selection state.

## When to use

Use when multiple interactive Highcharts tiles in a Toyz workspace share one registered tabular data source and you must confirm that selecting a point in one chart selects the same points in all other charts from that source.

## When NOT to use

- Plots are generated from different data sources — Toyz links only charts from the same source
- Static or non-interactive plot images where no selection state can be inspected via the web client API
- Single-chart analysis where no second chart exists to verify propagation against

## Inputs

- Tabular data file (e.g. CSV/text table) registered as a Toyz data source
- Two or more Highcharts tiles in a saved Toyz workspace bound to the same data source
- Admin login credentials for the Toyz web application

## Outputs

- selection_propagation_report recording whether selections propagate across linked charts
- Persisted Toyz workspace (linked_selection_workspace) with both chart settings

## How to apply

Launch the Toyz web application on its default port (8888), log in as admin, and register a tabular data file (e.g. a CSV/text table) via the Add Data Source dialog. Create two Highcharts tiles, each with a series bound to the same data source, choosing x and y columns from the data source select box in the Edit Chart dialog; a data source must be added before creating any Highchart. Save the workspace (e.g. linked_selection_workspace) so both tiles persist with their chart settings. Select a point — or click-drag a region for multiple points — in the first chart, then programmatically inspect the second chart's selection state via the Toyz web client API and record whether the same point(s) are selected there in a selection_propagation_report. The check passes only if selection of a point on one chart selects the same point(s) on every other chart connected to the same data source.

## Related tools

- **Toyz** (Web application framework hosting the Highcharts tiles, data source registration, and the web client API used to inspect selection state) — https://github.com/fred3m/toyz
- **Highcharts** (Interactive plot library whose tiles carry the linked selection state under test)
- **Python** (Language in which Toyz is implemented and through which the web client API is exercised) — https://github.com/fred3m/toyz
- **pandas** (Backend for opening tabular data files as Toyz data sources)

## Examples

```
toyz --port=8888
```

## Evaluation signals

- Selecting a single point in chart 1 results in the same point being selected in chart 2 per the web client API
- Click-drag region selection in chart 1 selects the identical set of points in chart 2
- Both Highcharts tiles are demonstrably bound to the same registered data source before testing
- selection_propagation_report records the propagation behavior for every chart connected to the shared source
- Workspace saves and reloads with both tiles and their chart settings intact

## Limitations

- Linked selection only applies between plots generated from the same data source
- Astro-Toyz interactive features such as region selection are still under active development
- The web application is intended for remote/browser use; local-machine rendering of very large catalogs is out of scope

## Evidence

- [full_text] finding: [evidence span withheld — anchor status: not_found]
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [readme] readme-astro: [evidence span withheld — anchor status: not_found]
