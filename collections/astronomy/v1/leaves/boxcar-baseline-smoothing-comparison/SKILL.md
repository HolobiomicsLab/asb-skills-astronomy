---
name: boxcar-baseline-smoothing-comparison
description: Use when smoothing noisy astronomical spectra (e.g. quasar spectra before
  cross-correlation redshifting) where preserving the position of broad emission features
  matters, or when deciding between an exponential-decay rolling filter and a boxcar
  baseline.
when_to_use_negative:
- The goal is redshift classification or template matching itself (use Marz's autoz
  cross-correlation workflow; smoothing is only a preprocessing demonstration)
- The input is already a smoothed or high signal-to-noise spectrum where peak suppression
  by boxcar averaging is not a concern
- The signal is not peak-like (e.g. pure continuum estimation), since the skill's
  validation criterion is specifically peak-location recovery
edam_operation: http://edamontology.org/operation_3214
tools:
- name: Marz
  role: Source implementation whose Eq. 2 rolling-point exponential decay mean filter
    (window 7 pixels, decay 0.9) is reconstructed and compared against a boxcar smoother
    before autoz cross-correlation matching
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:27c663fb72eca8c47e58703df1d37412cae408ded581633c94f50309d6d5d648
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  edam_operation: http://edamontology.org/operation_3214
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# boxcar-baseline-smoothing-comparison

## Summary

Reconstructs Marz's preprocessing smoother for quasar spectra — a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) — and demonstrates against a standard boxcar (moving average) smoother that the exponential decay filter recovers the true signal peak location on Poisson-noised synthetic data (Figure 2 behaviour), while boxcar smoothing depresses the peak.

## When to use

Use when smoothing noisy astronomical spectra (e.g. quasar spectra before cross-correlation redshifting) where preserving the position of broad emission features matters, or when deciding between an exponential-decay rolling filter and a boxcar baseline.

## When NOT to use

- The goal is redshift classification or template matching itself (use Marz's autoz cross-correlation workflow; smoothing is only a preprocessing demonstration)
- The input is already a smoothed or high signal-to-noise spectrum where peak suppression by boxcar averaging is not a concern
- The signal is not peak-like (e.g. pure continuum estimation), since the skill's validation criterion is specifically peak-location recovery

## Inputs

- Synthetic exponential-decay spectral signal corrupted with independent per-pixel Poisson noise (proxy for a noisy quasar spectrum)
- Noise-free true signal (for peak-recovery validation)
- Quasar spectra in generic FITS format as ingested by Marz before cross-correlation matching

## Outputs

- Per-pixel smoothed flux array from the exponential decay filter (window=7, decay=0.9)
- Per-pixel smoothed flux array from the boxcar (moving average) baseline of comparable width
- Comparison PNG plot: true signal vs exponential-decay-smoothed vs boxcar-smoothed, plus filter-strength vs pixel-offset curve

## How to apply

Generate a synthetic exponential-decay spectral signal and corrupt each pixel with independent Poisson noise to emulate an observed spectrum. Implement the rolling-point exponential decay mean filter with a window width of 7 pixels and a decay factor of 0.9 following Eq. 2 of the Marz paper, recording the smoothed flux per pixel. Apply a standard boxcar (moving average) smoother of comparable width to the same noisy signal as the baseline. Plot the true signal, the exponentially decayed smoothed output, and the boxcar-smoothed output together, plus the filter-strength versus pixel-offset curve, showing that the exponential filter recovers the original peak location while the boxcar output is depressed at the peak. Save the comparison as a PNG and validate that the figure exists, is a valid PNG, and that the exponential filter's peak matches the true signal peak where the boxcar does not.

## Related tools

- **Marz** (Source implementation whose Eq. 2 rolling-point exponential decay mean filter (window 7 pixels, decay 0.9) is reconstructed and compared against a boxcar smoother before autoz cross-correlation matching) — https://github.com/samreay/Marz

## Examples

```
ed = exp_decay_filter(noisy, window=7, decay=0.9); box = np.convolve(noisy, np.ones(7)/7, mode='same'); plot_comparison(true_signal, ed, box, out='fig2_comparison.png')
```

## Evaluation signals

- The output figure file exists and is a valid PNG
- The exponential-decay-smoothed output's peak pixel equals the true signal's peak pixel (peak recovered)
- The boxcar-smoothed output's peak is depressed relative to the true signal peak at the same location, reproducing the qualitative Figure 2 behaviour
- The exponential filter uses exactly the Marz parameters: window width 7 pixels and decay factor 0.9 per Eq. 2
- The filter-strength versus pixel-offset curve is plotted alongside the three spectra

## Limitations

- The peak-recovery advantage is demonstrated on a synthetic Poisson-noised exponential-decay signal, not on real observed quasar spectra, so real-data behaviour is an extrapolation
- Marz was developed in Javascript 'without significant supporting scientific libraries', and 'the mathematical library support for Javascript is still in its infancy', so numerical behaviour may depend on the in-app DSP implementation rather than vetted scientific libraries
- The paper does not state version numbers for external libraries (AngularJS, Bootstrap, fitsjs, Digital Signal Processing package), complicating exact reproduction of the original filter implementation

## Evidence

- [other] Marz applies a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) to quasar spectra: [evidence span withheld — anchor status: not_found]
- [other] the smoothed output recovers the original signal peak location, whereas boxcar smoothing does not: [evidence span withheld — anchor status: not_found]
- [abstract] a modified version of the autoz cross-correlation algorithm is used to match input spectra against a variety of stellar and galaxy templates: "a modified version of the autoz cross-correlation algorithm is used to match input spectra against a variety of stellar and galaxy templates"
- [discussion] the mathematical library support for Javascript is still in its infancy: "the mathematical library support for Javascript is still in its in- fancy"
