---
name: tabular-data-source-registration
description: Use when you need to make a local or remote CSV/text table available
  to a Toyz web workspace as a data source before creating Highcharts plots or linked-selection
  tiles bound to its columns.
when_to_use_negative:
- Input is imagery (e.g., FITS images from DECam) — use the Toyz image viewer / astro-toyz
  astro-viewer tile instead
- You only need static offline plotting in a local Python session without the Toyz
  web framework
- The data source is not needed by any chart — a Highcharts tile cannot be configured
  until a data source exists, so registering an unused source adds clutter
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: Toyz
  role: Web application framework in which the data source is registered via the Add
    Data Source dialog and consumed by Highcharts tiles
  repo: https://github.com/fred3m/toyz
- name: pandas
  role: Backend reader used by Toyz to open the registered CSV/text tabular data file
- name: Highcharts
  role: Interactive plot tile that binds series x/y columns to the registered data
    source
- name: astro-toyz
  role: Optional addon adding support for astropy Tables as a Data Source
  repo: https://github.com/fred3m/astro-toyz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:15c289a623d2202a9c0c0362980b4403db6277a9529af66353e636e4c7f625c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Highcharts
  - Toyz
  - astro-toyz
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tabular-data-source-registration

## Summary

Register a tabular file (e.g., CSV/text table readable with python, numpy, or pandas) as a named data source in a Toyz workspace so that Highcharts tiles and other visualization tools can bind series to its columns and share linked selections.

## When to use

Use when you need to make a local or remote CSV/text table available to a Toyz web workspace as a data source before creating Highcharts plots or linked-selection tiles bound to its columns.

## When NOT to use

- Input is imagery (e.g., FITS images from DECam) — use the Toyz image viewer / astro-toyz astro-viewer tile instead
- You only need static offline plotting in a local Python session without the Toyz web framework
- The data source is not needed by any chart — a Highcharts tile cannot be configured until a data source exists, so registering an unused source adds clutter

## Inputs

- Tabular data file on the Toyz server (CSV or text table readable by pandas/numpy)
- Toyz workspace session with an authenticated user account (default admin)
- Registered data source name chosen in the Add Data Source dialog

## Outputs

- Named data source in the Toyz workspace available to Highcharts tiles via the data source select box
- Persisted workspace (e.g., linked_selection_workspace) with chart settings bound to the source

## How to apply

Launch the Toyz web application (default port 8888; use `toyz --port=<remote_port>` with ssh -L port forwarding for remote servers) and log in — change the default admin password in Account Settings first. In the workspace, open the Add Data Source dialog and register the tabular data file (CSV/text table) as a named data source; Toyz opens data files 'using python, numpy, or pandas (astropy in the future)'. Critically, this registration must precede chart creation: 'To create a new Highchart first add a data source'. Once registered, all plots generated from that data source are linked, so a point selected in one Highcharts tile propagates to every other chart bound to the same source; persist this by saving the workspace (e.g., linked_selection_workspace).

## Related tools

- **Toyz** (Web application framework in which the data source is registered via the Add Data Source dialog and consumed by Highcharts tiles) — https://github.com/fred3m/toyz
- **pandas** (Backend reader used by Toyz to open the registered CSV/text tabular data file)
- **Highcharts** (Interactive plot tile that binds series x/y columns to the registered data source)
- **astro-toyz** (Optional addon adding support for astropy Tables as a Data Source) — https://github.com/fred3m/astro-toyz

## Examples

```
toyz --port=8888  # then in the web UI: Add Data Source dialog -> register my_table.csv before adding any Highcharts tile
```

## Evaluation signals

- The data source appears in the data source select box when configuring a Highcharts series in the Edit Chart dialog
- A Highcharts tile can be saved with x and y columns chosen from the registered source (charts cannot be created without a prior data source)
- Workspace is saved (e.g., linked_selection_workspace) and reloading it restores both tiles with their chart settings intact
- Selecting a point in one Highcharts chart selects the same point(s) in every other chart bound to the same data source (recorded in selection_propagation_report)

## Limitations

- Astro-Toyz support for astropy Tables as a data source was still under active development at time of writing
- Data must reside on (or be accessible from) the Toyz server; files are opened from server files via the load/open dialogs, not uploaded ad hoc
- Default admin credentials ship with the app and must be changed before registering data on a shared server

## Evidence

- [readme] Open a data file from the server using python, numpy, or pandas: [evidence span withheld — anchor status: external_doc]
- [readme] Interactive plots using Highcharts: [evidence span withheld — anchor status: external_doc]
- [other] Toyz task card: linked selection across plots from the same data source: [evidence span withheld — anchor status: not_found]
