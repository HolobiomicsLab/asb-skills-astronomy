---
name: exponential-decay-peak-recovery-analysis
description: Use when preprocessing noisy (e.g. Poisson-noised) quasar or AAOmega
  spectra prior to autoz cross-correlation matching, and a smoother must preserve
  broad emission-feature peak locations rather than depress them as a moving average
  would.
when_to_use_negative:
- When peak-position preservation is irrelevant and a simple moving average suffices,
  since the exponential-decay weighting adds no benefit over boxcar
- When the goal is general redshift assignment rather than demonstrating/verifying
  the smoothing step itself — use Marz's full autoz cross-correlation matching instead
- When spectra are already smooth or high signal-to-noise, where the Poisson-noise
  emulation and peak-recovery demonstration are moot
tools:
- name: Marz
  role: Source implementation of the rolling-point exponential decay mean filter (Eq.
    2) applied to quasar spectra before cross-correlation redshifting
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:5f7c14ec6fd76d654c5a18a6c54ee8a682cf9d5fffcfa38d0af6207ab3a0f076
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
yes_rate: 0.875
n_claims: 8
---

# exponential-decay-peak-recovery-analysis

## Summary

Reproduces Marz's quasar-spectrum preprocessing smoother: a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) and demonstrates via a Poisson-noised synthetic exponential-decay signal that it recovers the true peak location where an equal-width boxcar smoother does not (Figure 2).

## When to use

Use when preprocessing noisy (e.g. Poisson-noised) quasar or AAOmega spectra prior to autoz cross-correlation matching, and a smoother must preserve broad emission-feature peak locations rather than depress them as a moving average would.

## When NOT to use

- When peak-position preservation is irrelevant and a simple moving average suffices, since the exponential-decay weighting adds no benefit over boxcar
- When the goal is general redshift assignment rather than demonstrating/verifying the smoothing step itself — use Marz's full autoz cross-correlation matching instead
- When spectra are already smooth or high signal-to-noise, where the Poisson-noise emulation and peak-recovery demonstration are moot

## Inputs

- Synthetic exponential-decay spectral signal corrupted with independent per-pixel Poisson noise
- Quasar (QSO) spectra in FITS format as handled by Marz
- Filter parameters: window width 7 pixels, decay factor 0.9 (Eq. 2 of Marz)

## Outputs

- Per-pixel exponentially decayed smoothed flux array
- Boxcar-smoothed baseline flux array of comparable width
- Comparison plot (true signal vs exponential-decay output vs boxcar output) with filter-strength versus pixel-offset curve, saved as PNG

## How to apply

Generate a synthetic exponential-decay spectral signal and corrupt each pixel with independent Poisson noise to emulate an observed spectrum. Implement the rolling-point exponential decay mean filter following Eq. 2 of the Marz paper, using a window width of 7 pixels and a decay factor of 0.9, applying exponentially decaying weights within each window so recent pixels dominate while peak position is preserved. Apply the filter to the noisy signal and, as a baseline, apply a boxcar (moving average) smoother of comparable width to the same data. Plot the true signal, the exponentially decayed smoothed output, and the boxcar-smoothed output together, plus the filter-strength versus pixel-offset curve, and save the comparison as a PNG. The rationale, from Marz's design for OzDES/2dFLenS AAOmega spectra, is that broad quasar emission features fed to the modified autoz cross-correlation matcher must retain their peak positions for reliable redshift matching.

## Related tools

- **Marz** (Source implementation of the rolling-point exponential decay mean filter (Eq. 2) applied to quasar spectra before cross-correlation redshifting) — https://github.com/samreay/Marz

## Examples

```
import numpy as np; w = 0.9 ** np.arange(6, -1, -1); smoothed = np.array([np.dot(noisy[max(0,i-6):i+1], w[-(i+1):]) / w[-(i+1):].sum() for i in range(len(noisy))])
```

## Evaluation signals

- The saved figure file exists and is a valid PNG
- The exponentially decayed smoothed output's peak is located at the true signal's peak pixel, while the boxcar-smoothed output's peak is depressed/shifted
- The filter-strength versus pixel-offset curve follows the exponential-decay form implied by decay factor 0.9 over a 7-pixel window
- The smoothed output qualitatively tracks the original signal, matching the behaviour reported in Figure 2 of the Marz paper

## Limitations

- The peak-recovery advantage is demonstrated on a synthetic Poisson-noised exponential-decay signal (Figure 2), not on a systematic evaluation over real quasar samples
- Marz implements the smoothing in client-side Javascript where, as the paper notes, mathematical library support 'is still in its infancy', so no versioned scientific libraries underpin the filter
- Specific parameters (7-pixel window, 0.9 decay) were chosen for quasar emission features; other object types or pixel scales may need retuning

## Evidence

- [full_text] Marz applies a rolling-point exponential decay mean filter (window width 7 pixels, decay factor 0.9, Eq. 2) to quasar spectra: [evidence span withheld — anchor status: not_found]
- [full_text] the smoothed output recovers the original signal peak location, whereas boxcar smoothing does not: [evidence span withheld — anchor status: not_found]
- [abstract] autoz cross-correlation matching context: "Behind the scenes, a modified version of the autoz cross-correlation algorithm is used to match input spectra against a variety of stellar and galaxy templates"
