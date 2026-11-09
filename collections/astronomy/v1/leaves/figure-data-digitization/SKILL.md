---
name: figure-data-digitization
description: Use when a paper reports a quantitative relation only as a scatter plot
  (e.g. brightest masked-source flux-density vs proportional RMS drop in Figs. 4-5)
  with no machine-readable table, and the points must be extracted for refitting or
  sub-sample comparison.
when_to_use_negative:
- The data already exist as a machine-readable table or CSV — digitizing the figure
  would only add pixel-level error.
- The figure lacks calibrated quantitative axes or is a schematic illustration with
  no tick positions to anchor the mapping.
- Sub-pixel precision is required for downstream quantitative claims — manual digitization
  accuracy is limited to what the tick calibration and marker size allow.
tools:
- name: Python
  role: Language for the plotting/digitization script that calibrates pixel-to-data
    axes and extracts scatter-point coordinates
- name: automated-radio-imaging-paper
  role: Source repository supplying the figure image files for Figures 4 and 5
  repo: https://github.com/timstaley/automated-radio-imaging-paper
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:2574b70ad62f45b037036adb5659a5300262dbb8e426fbda5bc917687ca45f64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/automated-radio-imaging-paper
  tools:
  - Python
  - automated-radio-imaging-paper
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# figure-data-digitization

## Summary

Recover quantitative scatter-point coordinates from published raster figures (here, Figures 4 and 5 of Staley & Anderson 2015) by calibrating pixel-to-data axes against known tick positions, then use the digitized sample for statistical tests such as fitting trends of proportional RMS-estimate reduction versus brightest masked-source flux.

## When to use

Use when a paper reports a quantitative relation only as a scatter plot (e.g. brightest masked-source flux-density vs proportional RMS drop in Figs. 4-5) with no machine-readable table, and the points must be extracted for refitting or sub-sample comparison.

## When NOT to use

- The data already exist as a machine-readable table or CSV — digitizing the figure would only add pixel-level error.
- The figure lacks calibrated quantitative axes or is a schematic illustration with no tick positions to anchor the mapping.
- Sub-pixel precision is required for downstream quantitative claims — manual digitization accuracy is limited to what the tick calibration and marker size allow.

## Inputs

- Raster figure image files (Figures 4 and 5) from github.com/timstaley/automated-radio-imaging-paper
- Known axis tick positions of each figure panel (for pixel-to-data calibration)
- Labels of annotated outlier points: SWIFT_554620, XTEJ1908+094, PTF09AXC
- Observation-duration metadata for applying the >3.5 h cut

## Outputs

- digitized_figures_4_5.csv containing digitized (brightest masked-source flux, proportional RMS drop) coordinates with outlier tags
- Two linear trend fits: full-sample fit and >3.5 h sub-sample fit of proportional RMS drop vs brightest masked-source flux

## How to apply

Load the figure image files from the paper's source repository (github.com/timstaley/automated-radio-imaging-paper) and, in a Python plotting/digitization script, calibrate the pixel-to-data coordinate mapping using the known axis tick positions of each figure panel. Extract the (brightest masked-source flux, proportional RMS drop) coordinate of every scatter point and tag the labelled outliers SWIFT_554620, XTEJ1908+094 and PTF09AXC. Fit a linear trend of proportional RMS drop versus brightest masked-source flux on the full digitized sample, then apply the >3.5 h observation-duration cut, refit the trend on the retained sub-sample, and compare correlation tightness between the two samples. Write the digitized coordinates and both trend fits to digitized_figures_4_5.csv. The rationale is that the >3.5 h cut removes short-duration observations that scatter the relation, exposing the underlying flux-density vs RMS-estimate-reduction trend claimed in the paper.

## Related tools

- **Python** (Language for the plotting/digitization script that calibrates pixel-to-data axes and extracts scatter-point coordinates)
- **automated-radio-imaging-paper** (Source repository supplying the figure image files for Figures 4 and 5) — https://github.com/timstaley/automated-radio-imaging-paper

## Examples

```
git clone https://github.com/timstaley/automated-radio-imaging-paper
```

## Evaluation signals

- Digitized coordinates for the three labelled outliers (SWIFT_554620, XTEJ1908+094, PTF09AXC) reproduce their relative positions in the original figures
- Axis calibration maps the known tick positions back to their stated data values with no offset
- The >3.5 h duration-cut sub-sample yields a visibly tighter flux–RMS-reduction relation than the full sample, matching the paper's claimed qualitative sharpening
- digitized_figures_4_5.csv contains both the full-sample and >3.5 h sub-sample trend fits alongside the point coordinates

## Limitations

- Digitized coordinates carry pixel-level uncertainty inherited from raster resolution and the tick-based calibration; results are approximate.
- Only observations longer than 3.5 hours show a clear relation — the full-sample fit is scattered and any conclusions depend on the duration cut.
- Only figures hosted in the paper source repository (github.com/timstaley/automated-radio-imaging-paper) can be digitized; figures not included there are unavailable.

## Evidence

- [intro] Python is the implementation language for the digitization tooling: [evidence span withheld — anchor status: not_found]
- [readme] The paper (and its figures) is the published source being digitized: [evidence span withheld — anchor status: not_found]
- [other] The >3.5 h cut sharpens the digitized flux–RMS-reduction relation: [evidence span withheld — anchor status: not_found]
