---
name: poisson-noise-signal-synthesis
description: 'Use when validating or reproducing Marz-style spectral smoothing: you
  need a noisy exponential-decay test signal with known ground truth to show that
  the exponential decay filter preserves peak locations better than a moving-average
  (boxcar) smoother.'
when_to_use_negative:
- You are smoothing real observed quasar spectra for redshift matching rather than
  validating the smoothing method against known ground truth
- The goal is Marz's automatic cross-correlation redshifting (modified autoz template
  matching), not the preprocessing/filter demonstration
- Your signal has Gaussian-dominated noise or a flat continuum where peak-recovery
  behaviour of the exponential decay filter is not the quantity under test
tools:
- name: Marz
  role: Source of the exponential decay mean filter (window width 7 pixels, decay
    factor 0.9, Eq. 2) whose peak-recovery behaviour the synthetic Poisson-noised
    signal demonstrates
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:5c606036d4f53dfa690803fa0e1948a9039ac52338922b01b046b7e5ebd66e8d
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
yes_rate: 0.8461538461538461
n_claims: 13
---

# poisson-noise-signal-synthesis

## Summary

Synthesise a synthetic exponential-decay spectral signal corrupted with per-pixel independent Poisson noise, as Marz uses to demonstrate that its rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9) recovers true signal peaks where boxcar smoothing does not (Figure 2).

## When to use

Use when validating or reproducing Marz-style spectral smoothing: you need a noisy exponential-decay test signal with known ground truth to show that the exponential decay filter preserves peak locations better than a moving-average (boxcar) smoother.

## When NOT to use

- You are smoothing real observed quasar spectra for redshift matching rather than validating the smoothing method against known ground truth
- The goal is Marz's automatic cross-correlation redshifting (modified autoz template matching), not the preprocessing/filter demonstration
- Your signal has Gaussian-dominated noise or a flat continuum where peak-recovery behaviour of the exponential decay filter is not the quantity under test

## Inputs

- Synthetic exponential-decay spectral signal (per-pixel flux array with known peak location)
- Poisson noise realisation applied independently to each pixel
- Filter parameters: window width of 7 pixels and decay factor of 0.9 (Marz Eq. 2)

## Outputs

- Noisy synthetic spectrum (Poisson-noised flux per pixel)
- Exponential-decay smoothed flux per pixel (Marz filter, Eq. 2)
- Boxcar (moving average) smoothed flux per pixel
- Comparison plot (PNG) showing true signal, exponential-decay smoothed output, and boxcar-smoothed output, plus filter-strength versus pixel-offset curve

## How to apply

Generate a synthetic exponential-decay spectral signal whose peak location is known by construction, then corrupt it with independent Poisson noise drawn per pixel to emulate a low-S/N observed spectrum such as those from the AAOmega spectrograph. Implement Marz's rolling-point exponential decay mean filter with a window width of 7 pixels and a decay factor of 0.9 following Eq. 2 of the Marz paper, and apply it to the noisy signal. In parallel, apply a standard boxcar (moving average) smoother of comparable width as the baseline. Plot the true signal, the exponentially decayed smoothed output, and the boxcar output together, plus the filter-strength versus pixel-offset curve, and save the comparison as a PNG; the exponential filter should track the true peak while the boxcar output is depressed at the peak.

## Related tools

- **Marz** (Source of the exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) whose peak-recovery behaviour the synthetic Poisson-noised signal demonstrates) — https://github.com/samreay/Marz

## Examples

```
noisy = np.random.poisson(signal_true); smoothed = exp_decay_filter(noisy, window=7, decay=0.9); boxcar = np.convolve(noisy, np.ones(7)/7, mode='same')
```

## Evaluation signals

- The saved comparison plot exists and is a valid PNG file
- The exponentially decayed smoothed output recovers the original signal peak location
- The boxcar-smoothed output is depressed at the true peak (fails to recover peak location), matching the Figure 2 qualitative behaviour
- The filter parameters used match Marz Eq. 2: window width of 7 pixels and decay factor of 0.9
- The filter-strength versus pixel-offset curve is plotted alongside the signal comparison

## Limitations

- The demonstration is qualitative (peak recovered where boxcar output is depressed), matching Figure 2's behaviour, not a quantitative accuracy benchmark
- Marz was built in Javascript without significant supporting scientific libraries — 'the mathematical library support for Javascript is still in its infancy' — so reproductions typically reimplement Eq. 2 in a scientific language rather than reuse the JS code
- No version numbers are stated for the external libraries Marz depends on (AngularJS, Bootstrap, AngularUI, fitsjs, etc.), so exact dependency behaviour is not pinned

## Evidence

- [results] Marz applies a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) to quasar spectra: [evidence span withheld — anchor status: not_found]
- [methods] Generate a synthetic exponential-decay spectral signal and corrupt it with independent Poisson noise per pixel to emulate a noisy observed spectrum.: [evidence span withheld — anchor status: not_found]
- [results] as illustrated on a Poisson-noised exponential-decay signal in Figure 2, the smoothed output recovers the original signal peak location, whereas boxcar smoothing does not: [evidence span withheld — anchor status: not_found]
