---
name: figure-reproduction-comparison-plotting
description: Use when a paper's figure demonstrates a quantitative or qualitative
  algorithm property (e.g. peak recovery under smoothing) that can be validated by
  regenerating the synthetic inputs and overlaying original, method, and baseline
  curves in one plot.
when_to_use_negative:
- Input is real observed data (e.g. AAOmega FITS spectra) whose redshift you actually
  need — use Marz's matching workflow instead of a figure-reproduction exercise
- The figure depends on proprietary data or templates not reconstructable from the
  paper's description
- The claim is a statistical aggregate over a survey (e.g. runz 54% vs Marz 91% matching
  performance), not a single-figure algorithm behaviour
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Marz
  role: Source application whose Eq. 2 exponential decay smoothing filter (window
    7, decay 0.9) is re-implemented to reproduce Figure 2; open-source web/CLI tool
    for redshifting AAOmega spectra
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:bcdf10524ac9e391f6e362a1482ababe8e7541007c0ad1f9875d7cdab4f1981f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# figure-reproduction-comparison-plotting

## Summary

Reproduce a paper figure by re-implementing the described algorithm (here, Marz's rolling-point exponential decay mean filter with window width 7 and decay factor 0.9) on synthetic data and plotting the result against a baseline method to visually verify the claimed behaviour.

## When to use

Use when a paper's figure demonstrates a quantitative or qualitative algorithm property (e.g. peak recovery under smoothing) that can be validated by regenerating the synthetic inputs and overlaying original, method, and baseline curves in one plot.

## When NOT to use

- Input is real observed data (e.g. AAOmega FITS spectra) whose redshift you actually need — use Marz's matching workflow instead of a figure-reproduction exercise
- The figure depends on proprietary data or templates not reconstructable from the paper's description
- The claim is a statistical aggregate over a survey (e.g. runz 54% vs Marz 91% matching performance), not a single-figure algorithm behaviour

## Inputs

- Synthetic exponential-decay spectral signal with independent Poisson noise per pixel
- Algorithm parameters from the paper (window width 7 pixels, decay factor 0.9 per Marz Eq. 2)
- Baseline smoother specification (boxcar moving average of comparable width)

## Outputs

- PNG comparison plot showing true signal, exponential-decay smoothed output, and boxcar-smoothed output
- Filter-strength versus pixel-offset curve
- Per-pixel smoothed flux arrays for both smoothers

## How to apply

First, reconstruct the synthetic test case exactly as described in the paper: for Marz Figure 2 this is an exponential-decay spectral signal corrupted with independent Poisson noise per pixel to emulate a noisy observed spectrum. Next, implement the paper's method precisely with its stated parameters — Eq. 2 of Marz specifies a rolling-point exponential decay mean filter with window width 7 pixels and decay factor 0.9 — and a baseline competitor (a standard boxcar moving average of comparable width) on the same noisy input. Record smoothed flux per pixel for both. Plot the true signal, the exponentially decayed smoothed output, and the boxcar-smoothed output together, plus the filter-strength versus pixel-offset curve, so the key qualitative claim (exponential decay recovers the true signal peak while boxcar output is depressed) is directly visible. Save the plot as a PNG and verify the file exists and is a valid PNG before declaring the figure reproduced.

## Related tools

- **Marz** (Source application whose Eq. 2 exponential decay smoothing filter (window 7, decay 0.9) is re-implemented to reproduce Figure 2; open-source web/CLI tool for redshifting AAOmega spectra) — https://github.com/samreay/Marz

## Examples

```
python reproduce_fig2.py --window 7 --decay 0.9 --baseline boxcar --output fig2_comparison.png
```

## Evaluation signals

- Comparison plot PNG exists on disk and is a valid PNG file
- Exponential-decay smoothed output peaks at (or tracks) the true synthetic signal peak location
- Boxcar-smoothed output shows a depressed peak at the same location, matching the qualitative behaviour reported in Figure 2
- Filter implementation uses the exact stated parameters: window width 7 pixels and decay factor 0.9 (Eq. 2)
- All three curves (true signal, exponential-decay, boxcar) plus the filter-strength versus pixel-offset curve appear in the single figure

## Limitations

- Validation is qualitative (visual peak recovery on one synthetic case), not a statistical error metric across many trials
- The synthetic Poisson-noised exponential-decay signal is a simplification of real quasar spectra; behaviour on noisy AAOmega spectrograph data with broad emission features may differ
- Marz was developed without significant supporting scientific libraries and mathematical library support for Javascript was still in its infancy, so exact numerical agreement with the original implementation may require care

## Evidence
