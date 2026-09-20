---
name: spectral-smoothing-filter-implementation
description: Use when preprocessing Poisson-noised quasar/astronomical spectra prior
  to autoz cross-correlation matching, and you need a smoothing filter that preserves
  emission-line peak locations rather than flattening them.
when_to_use_negative:
- Spectra are already smooth or high signal-to-noise where an extra 7-pixel filter
  would degrade rather than preserve cross-correlation features
- The analysis goal is feature suppression (e.g. continuum fitting) where deliberately
  depressing peaks is desired, since the exponential decay filter is chosen specifically
  to recover peak locations
- The input is not a 1D spectrum (e.g. image cubes or 2D frames), where the rolling-point
  1D filter and its 7-pixel window are ill-defined
tools:
- name: Marz
  role: Source of the exponential decay mean filter (Eq. 2, window 7 pixels, decay
    0.9) used in spectral preprocessing before autoz cross-correlation matching
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:4a364e4a4b7998799d14960c14d7d651a8afc05ab92a9dc749500e24d0166ab2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-smoothing-filter-implementation

## Summary

Reproduces Marz's preprocessing smoother for quasar spectra: a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) that preserves broad emission-feature peak positions, benchmarked against a boxcar (moving average) smoother of comparable width which depresses the peak.

## When to use

Use when preprocessing Poisson-noised quasar/astronomical spectra prior to autoz cross-correlation matching, and you need a smoothing filter that preserves emission-line peak locations rather than flattening them.

## When NOT to use

- Spectra are already smooth or high signal-to-noise where an extra 7-pixel filter would degrade rather than preserve cross-correlation features
- The analysis goal is feature suppression (e.g. continuum fitting) where deliberately depressing peaks is desired, since the exponential decay filter is chosen specifically to recover peak locations
- The input is not a 1D spectrum (e.g. image cubes or 2D frames), where the rolling-point 1D filter and its 7-pixel window are ill-defined

## Inputs

- Synthetic exponential-decay spectral signal corrupted with per-pixel independent Poisson noise (emulating an observed quasar spectrum)
- Noisy 1D quasar spectrum in a generic FITS file (Marz requires generic FITS files)
- Filter parameters: window width 7 pixels, decay factor 0.9 (Eq. 2 of the Marz paper)

## Outputs

- Per-pixel smoothed flux array from the exponential decay filter
- Per-pixel smoothed flux array from a comparable-width boxcar (moving average) baseline
- Comparison plot (PNG) of true signal vs. exponentially decayed output vs. boxcar output, plus a filter-strength vs. pixel-offset curve

## How to apply

Generate a synthetic exponential-decay spectral signal and corrupt each pixel with independent Poisson noise to emulate an observed OzDES/AAOmega quasar spectrum. Implement the rolling-point exponential decay mean filter following Eq. 2 of the Marz paper with a window width of 7 pixels and decay factor 0.9, weighting more recent (closer) pixels more heavily so the filter strength decays as 0.9 raised to the pixel offset. Apply the filter per pixel to record smoothed flux, then run a standard boxcar moving average of comparable width on the same noisy signal as the baseline. Plot the true signal, exponentially decayed output, and boxcar output together — the exponential decay filter recovers the original peak location while the boxcar output is depressed at the peak — and add a filter-strength versus pixel-offset curve. This matters because Marz smooths spectra so broad emission features survive before the modified autoz cross-correlation matching against stellar and galaxy templates.

## Related tools

- **Marz** (Source of the exponential decay mean filter (Eq. 2, window 7 pixels, decay 0.9) used in spectral preprocessing before autoz cross-correlation matching) — https://github.com/samreay/Marz

## Examples

```
import numpy as np; true = np.exp(-x/5); noisy = np.random.poisson(true); smoothed = np.convolve(noisy, 0.9**np.arange(7), mode='same'); boxcar = np.convolve(noisy, np.ones(7)/7, mode='same')
```

## Evaluation signals

- The saved figure file exists and is a valid PNG
- argmax of the exponential-decay-smoothed output equals the pixel index of the true (noise-free) signal peak
- The boxcar-smoothed baseline shows a depressed peak value at the true peak location, reproducing the qualitative behaviour of Figure 2
- The filter-strength versus pixel-offset curve follows a 0.9^offset decay across the 7-pixel window, normalising to the Eq. 2 weighting
- Smoothed output tracks the true signal closely outside the peak (low residual variance relative to the noisy input)

## Limitations

- The 7-pixel window and 0.9 decay factor are the parameters Marz uses for its quasar preprocessing; other window widths or decay factors were not validated in the paper's demonstration
- The Figure 2 demonstration is on a synthetic Poisson-noised exponential-decay signal, so validation is qualitative (peak recovered vs. peak depressed) rather than a quantitative error metric
- Marz was developed without significant supporting scientific libraries, and mathematical library support for Javascript is still in its infancy; no version numbers are stated for the external libraries used (AngularJS, Bootstrap, AngularUI, fitsjs)

## Evidence

- [full_text] Marz applies a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) to quasar spectra: [evidence span withheld — anchor status: not_found]
- [full_text] the smoothed output recovers the original signal peak location, whereas boxcar smoothing does not: [evidence span withheld — anchor status: not_found]
