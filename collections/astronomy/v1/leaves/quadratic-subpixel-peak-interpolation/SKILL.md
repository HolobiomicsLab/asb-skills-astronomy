---
name: quadratic-subpixel-peak-interpolation
description: Use when cross-correlation of preprocessed spectra against templates
  produces discrete peak values on a binned (Fourier-space) redshift grid, and redshift
  estimates must be refined beyond the bin resolution before applying heliocentric
  and CMB corrections.
when_to_use_negative:
- The spectrum has not been cross-correlated against templates yet — there are no
  correlation peaks to interpolate around
- The analysis goal is interactive, human-guided redshifting (manual template comparison
  or feature marking), where automatic peak ranking and refinement are not the primary
  mechanism
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Marz
  role: Redshifting web/CLI application whose automatic matching pipeline ranks correlation
    peaks and applies the quadratic sub-pixel fit to the top 10
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Modified cross-correlation algorithm underlying Marz's matching that generates
    the peaks subsequently refined by quadratic interpolation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:da787382c04010347e7ef0b77af4bb8e91eb958e5685952c784157210520de02
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# quadratic-subpixel-peak-interpolation

## Summary

Refinement step in Marz's autoz cross-correlation pipeline: after ranking all template correlation peaks together, a quadratic fit is applied around each of the ten highest correlation peaks to determine local maxima at sub-pixel precision, yielding more accurate redshift candidates.

## When to use

Use when cross-correlation of preprocessed spectra against templates produces discrete peak values on a binned (Fourier-space) redshift grid, and redshift estimates must be refined beyond the bin resolution before applying heliocentric and CMB corrections.

## When NOT to use

- The spectrum has not been cross-correlated against templates yet — there are no correlation peaks to interpolate around
- The analysis goal is interactive, human-guided redshifting (manual template comparison or feature marking), where automatic peak ranking and refinement are not the primary mechanism

## Inputs

- Preprocessed, RMS-normalised FITS spectra (cosine-tapered, continuum-subtracted, variance-widened)
- Cross-correlation peak values from all twelve Marz stellar and galaxy templates, ranked together
- Fourier-space correlation arrays around each of the top 10 peaks

## Outputs

- Sub-pixel refined redshift candidates for the top 10 correlation peaks per spectrum
- Heliocentric- and CMB-corrected redshifts written to a per-spectrum CSV output file

## How to apply

First preprocess the spectrum along Marz's dual general/quasar paths (cosine taper, continuum subtraction via boxcar-median filter, variance widening, RMS normalisation), then cross-correlate in Fourier space against the twelve Marz stellar and galaxy templates using the modified autoz algorithm. Collect correlation peaks from all templates and sort them together so peaks compete globally rather than per-template. Retain only the ten highest correlation value peaks and, for each, fit a quadratic function to the correlation values around the discrete peak to locate the local maximum at sub-pixel precision. Convert each refined peak position into a redshift, then apply heliocentric and CMB corrections before writing the peaks and corrected redshifts to the per-spectrum CSV output. When only a single peak exists, apply the corrected xcor normalisation factor of 1 instead of returning NaN so the peak can still be refined and reported.

## Related tools

- **Marz** (Redshifting web/CLI application whose automatic matching pipeline ranks correlation peaks and applies the quadratic sub-pixel fit to the top 10) — https://github.com/samreay/Marz
- **autoz** (Modified cross-correlation algorithm underlying Marz's matching that generates the peaks subsequently refined by quadratic interpolation)

## Examples

```
./marz.sh <FITSFilePath>
```

## Evaluation signals

- Exactly ten peaks (or fewer if the correlation surface yields fewer valid peaks) are retained per spectrum, ranked by correlation value across all templates jointly
- Each retained peak's redshift is refined off the integer Fourier-bin grid via a quadratic fit rather than reported at the raw bin centre
- Results with only one correlation peak report a normalisation factor of 1, not NaN, so the peak still yields a usable redshift
- Final written redshifts include heliocentric and CMB corrections applied after sub-pixel refinement
- Per-spectrum CSV output contains the highest correlation peaks paired with their corrected redshifts

## Limitations

- Only the ten highest correlation peaks are refined; a true redshift signal ranked below tenth place will not be considered
- Quadratic sub-pixel accuracy depends on the quality of the correlation peak shape; noisy, low signal-to-noise spectra may yield biased peak positions
- The xcor normalisation edge case (single-peak results) originally returned NaN before the factor-of-1 fix, so unpatched versions would fail to produce a redshift for such spectra

## Evidence

- [other] Cross-correlation peaks from all templates are sorted together, and the ten highest correlation value peaks have a quadratic fit applied around the peak for sub-pixel determination of local maxima.: [evidence span withheld — anchor status: not_found]
