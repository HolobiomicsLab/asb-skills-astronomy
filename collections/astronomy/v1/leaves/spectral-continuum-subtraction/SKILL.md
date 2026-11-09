---
name: spectral-continuum-subtraction
description: Use when preprocessing 1D FITS spectra (e.g. from the AAOmega spectrograph)
  before cross-correlation redshifting, so that continuum-dominated flux is removed
  and only feature information enters the autoz cross-correlation.
when_to_use_negative:
- The goal is photometric calibration or absolute flux measurement — the boxcar-median
  subtraction discards continuum flux by design.
- Input spectra already carry a separate continuum-free or pre-normalised product;
  re-applying taper/boxcar subtraction could distort feature amplitudes.
- The spectrum has extremely low signal-to-noise where the continuum estimate from
  a boxcar-median filter is unreliable and manual redshifting in Marz's interface
  is the intended fallback.
tools:
- name: Marz
  role: Implements the dual general/quasar preprocessing paths (cosine taper, boxcar-median
    continuum subtraction, variance widening, RMS normalisation) feeding its autoz-modified
    cross-correlation matcher; usable via web app or CLI.
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Underlying cross-correlation algorithm whose Fourier-space matching requires
    the RMS-normalised, continuum-subtracted spectra this skill produces.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:4fe208d95377b5b753b19df03c5383afa3b6af80bd876369984f1b5cfc053799
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-continuum-subtraction

## Summary

Preprocessing step from Marz's autoz-modified redshift pipeline in which continuum emission is removed from 1D FITS spectra (via cosine tapering and a boxcar-median filter) so that subsequent Fourier-space cross-correlation against stellar and galaxy templates is dominated by absorption/emission features rather than the smooth spectral shape.

## When to use

Use when preprocessing 1D FITS spectra (e.g. from the AAOmega spectrograph) before cross-correlation redshifting, so that continuum-dominated flux is removed and only feature information enters the autoz cross-correlation.

## When NOT to use

- The goal is photometric calibration or absolute flux measurement — the boxcar-median subtraction discards continuum flux by design.
- Input spectra already carry a separate continuum-free or pre-normalised product; re-applying taper/boxcar subtraction could distort feature amplitudes.
- The spectrum has extremely low signal-to-noise where the continuum estimate from a boxcar-median filter is unreliable and manual redshifting in Marz's interface is the intended fallback.

## Inputs

- 1D FITS spectra in Marz's well-defined input format (e.g. from the AAOmega spectrograph)
- variance/error array accompanying each spectrum

## Outputs

- continuum-subtracted, cosine-tapered spectrum per input
- widened variance array matching the tapered spectrum
- RMS-normalised spectrum ready for Fourier-space cross-correlation

## How to apply

Load each input FITS spectrum and preprocess it along Marz's dual general/quasar paths. Apply a cosine taper to the spectrum to suppress edge artifacts, then subtract the continuum using a boxcar-median filter, which removes the smooth low-frequency component while retaining narrow spectral features. Widen the variance array to match the tapered spectrum, and RMS-normalise the resulting spectrum so templates and data are comparable in Fourier space. This normalised, continuum-free spectrum is the input to the modified autoz cross-correlation against the twelve Marz stellar and galaxy templates (drawn from WiggleZ and SDSS template samples). Rationale: cross-correlation peak normalisation and redshift quality depend on the continuum being removed; the taper and median-boxcar prevent edge discontinuities and large-scale slopes from generating spurious correlation peaks.

## Related tools

- **Marz** (Implements the dual general/quasar preprocessing paths (cosine taper, boxcar-median continuum subtraction, variance widening, RMS normalisation) feeding its autoz-modified cross-correlation matcher; usable via web app or CLI.) — https://github.com/samreay/Marz
- **autoz** (Underlying cross-correlation algorithm whose Fourier-space matching requires the RMS-normalised, continuum-subtracted spectra this skill produces.)

## Examples

```
./marz.sh <FITSFilePath>
```

## Evaluation signals

- The processed spectrum should have a near-zero smooth baseline: a re-applied boxcar-median filter on the output should recover an approximately flat continuum.
- No strong edge discontinuities should remain — the cosine taper should suppress flux jumps at both ends of the wavelength range.
- Variance array length must match the tapered spectrum length after widening.
- RMS-normalised output should have unit root-mean-square flux, making cross-correlation peak values comparable across spectra and templates.
- Downstream check: the top-10 cross-correlation peaks per spectrum should be finite (Marz's xcor normalisation returns a factor of 1 for single-peak results instead of NaN), and refined redshifts should survive heliocentric and CMB corrections to land in a physically plausible range.

## Limitations

- Marz was built around AAOmega spectra in a well-defined input format; spectra deviating from that format may not preprocess correctly even though the tool has been generalised.
- The preprocessing runs on dual general/quasar paths — choosing the wrong path for a target type can degrade correlation peaks.
- Automatic matching can mis-assign redshifts for low signal-to-noise spectra; Marz provides manual redshifting (cycling automatic results, manual template comparison, marking features) for spectra not matched correctly.
- Continuum subtraction via boxcar-median filter can distort very broad features (e.g. wide QSO emission), which the quasar path partially addresses.

## Evidence

- [other] apply a cosine taper, subtract the continuum via a boxcar-median filter, widen the variance array, and RMS-normalise the resulting spectrum: [evidence span withheld — anchor status: not_found]
