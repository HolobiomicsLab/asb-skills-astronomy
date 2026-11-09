---
name: chord-diagram-correlation-layout
description: Use when a tabular dataset (CSV/TXT with named columns, NA for missing)
  has been correlated (Pearson, Spearman, or MIC) and pairwise variable relationships
  need a circular visual diagnostic.
when_to_use_negative:
- The goal is to inspect the correlation structure as a rectangular matrix with dendrogram
  ordering — use the heatmap (pheatmap) or distogram diagnostic instead of the chord
  layout
- The relationships between variables should be shown as a network of vertices with
  weighted edges — use the qgraph correlation graph (Spring or Circular layout) instead
- The data are not a numeric correlation matrix of variables (e.g. the task is principal
  component contribution bars, where the nightingale chart is the appropriate diagnostic)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: AMADA
  role: Shiny application that computes the correlation matrix (Pearson, Spearman,
    or MIC) and renders the chord diagram diagnostic via its interface tabs
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: circlize
  role: R package used to draw the chord diagram with ribbons around a circular layout,
    thickness weighted by pairwise correlation
- name: shiny
  role: Web framework hosting the AMADA app; the chord diagram is rendered through
    the Shiny tabs and updated with the 'Make it so' button
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:58b6c4dffb60a233d8bcc29eb4b2c889e28c67021f23ccf54a1c567f8ae593c2
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - ape
  - circlize
  - corrplot
  - fpc
  - shiny
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chord-diagram-correlation-layout

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Render pairwise variable correlations of a high-dimensional astronomical dataset as a chord diagram (ribbons around a circular layout) using the circlize R package inside the AMADA Shiny app, with ribbon thickness weighted by pairwise correlation and colour intensity spanning fully anti-correlated to fully correlated variables.

## When to use

Use when a tabular dataset (CSV/TXT with named columns, NA for missing) has been correlated (Pearson, Spearman, or MIC) and pairwise variable relationships need a circular visual diagnostic.

## When NOT to use

- The goal is to inspect the correlation structure as a rectangular matrix with dendrogram ordering — use the heatmap (pheatmap) or distogram diagnostic instead of the chord layout
- The relationships between variables should be shown as a network of vertices with weighted edges — use the qgraph correlation graph (Spring or Circular layout) instead
- The data are not a numeric correlation matrix of variables (e.g. the task is principal component contribution bars, where the nightingale chart is the appropriate diagnostic)

## Inputs

- Tabular dataset in CSV/TXT format with named columns separated by spaces; missing data marked as NA (e.g. SDSS Type Ia/II supernova host-galaxy properties, Guo et al. 2011 mock galaxy catalog, or ZENS catalog of 141 galaxy groups at 0.05 < z < 0.0585)
- Pairwise correlation matrix computed by AMADA using the user-selected correlation method (Pearson, Spearman, or Maximum Information Coefficient)

## Outputs

- Chord diagram figure: circular layout of variables connected by ribbons whose thickness is weighted by pairwise correlation and whose colour intensity spans fully anti-correlated to correlated
- One of the six AMADA visualization diagnostic figure files (heatmap, distogram, dendrogram, graph, chord diagram, nightingale chart) checked for existence and non-emptiness

## How to apply

Import the dataset into AMADA via the 'Import Dataset' panel (CSV/TXT, named columns separated by spaces, missing values marked NA), then choose the correlation method (Pearson, Spearman, or Maximum Information Coefficient) on the left control panel and click 'Make it so' to update the plots. AMADA computes the pairwise correlation matrix upstream (part of the hierarchical clustering/PCA analysis for different choices of correlation matrices) and feeds it to the chord-diagram diagnostic built with the circlize R package. The chord diagram draws ribbons around a circular layout of the variables, where each ribbon's thickness is weighted by the pairwise correlation and the colour intensity spans the range from fully anti-correlated to correlated. Select the chord diagram colour scheme from the 'Chord diagram colour' control option before updating. Validate that the chord-diagram figure renders as a non-empty, readable image consistent with the correlation matrix; the AMADA paper reports no quantitative acceptance threshold, so successful rendering of the diagnostic from the shared input is the acceptance criterion.

## Related tools

- **AMADA** (Shiny application that computes the correlation matrix (Pearson, Spearman, or MIC) and renders the chord diagram diagnostic via its interface tabs) — https://github.com/RafaelSdeSouza/AMADA
- **circlize** (R package used to draw the chord diagram with ribbons around a circular layout, thickness weighted by pairwise correlation)
- **shiny** (Web framework hosting the AMADA app; the chord diagram is rendered through the Shiny tabs and updated with the 'Make it so' button)

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The chord diagram figure file exists and is non-empty and readable, matching the described AMADA diagnostic (paper reports no quantitative acceptance threshold; acceptance is successful rendering of all six diagnostics from the shared input)
- The ribbon thickness in the rendered chord diagram is visibly weighted by the pairwise correlations of the chosen correlation method (Pearson, Spearman, or MIC)
- The colour intensity of the ribbons spans the full range from fully anti-correlated to correlated variable pairs
- The displayed chord diagram updates after the user changes the 'Chord diagram colour' scheme and clicks 'Make it so'
- The underlying correlation matrix matches the dataset imported in the 'Import Dataset' panel (verified at the bottom of that panel) with missing values handled as NA

## Limitations

- Data must be imported as CSV/TXT with named columns separated by spaces; missing data must be marked as NA — other formats will not import properly
- The paper reports no quantitative acceptance threshold for the diagnostic figures, so correctness must be judged by visual/rendering checks
- Correlation method choice (Pearson, Spearman, MIC) changes the ribbons' weights, so chord diagrams from different methods are not directly comparable

## Evidence

- [intro] chord diagram rendering weights and colour scale: [evidence span withheld — anchor status: not_found]
