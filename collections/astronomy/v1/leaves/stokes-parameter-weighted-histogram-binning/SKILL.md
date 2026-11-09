---
name: stokes-parameter-weighted-histogram-binning
description: Use when you have an ixpeobssim-simulated or IXPE level-2 FITS event
  list and need to produce Stokes I/Q/U count spectra (PHA1 flavors) as XSPEC input
  for spectro-polarimetric fitting.
when_to_use_negative:
- You need polarization maps or sky-binned polarization products rather than spectra
  — use xpbin with PCUBE or PMAPCUBE instead of the PHA1 flavors.
- You want normalized Q/U spectra rather than weighted count-spectrum flavors — use
  the PHA1QN/PHA1UN binning algorithms instead.
tools:
- name: ixpeobssim
  role: Provides the xpbin facility and pipeline.xpbin call that bin event lists into
    PHA1/PHA1Q/PHA1U Stokes spectral FITS files
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XSPEC
  role: Consumes the I, Q and U Stokes spectra (via the XFLT0001 Stokes keyword) for
    spectro-polarimetric fitting, e.g. powerlaw * polconst
- name: Python
  role: Language in which the ixpeobssim pipeline and binning workflow are invoked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:93a40f1170d65297ce8ea5895cf050412c73e7bc7d35a851192e54b808bd6828
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - XSPEC
  - ixpeobssim
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# stokes-parameter-weighted-histogram-binning

## Summary

Bins an IXPE level-2 event list (with event-by-event Q and U Stokes parameters and W_MOM weights) into I, Q and U Stokes PHA spectra via ixpeobssim's xpbin PHA1/PHA1Q/PHA1U algorithms, producing OGIP FITS files ready for XSPEC spectro-polarimetric fitting.

## When to use

Use when you have an ixpeobssim-simulated or IXPE level-2 FITS event list and need to produce Stokes I/Q/U count spectra (PHA1 flavors) as XSPEC input for spectro-polarimetric fitting.

## When NOT to use

- You need polarization maps or sky-binned polarization products rather than spectra — use xpbin with PCUBE or PMAPCUBE instead of the PHA1 flavors.
- You want normalized Q/U spectra rather than weighted count-spectrum flavors — use the PHA1QN/PHA1UN binning algorithms instead.

## Inputs

- ixpeobssim level-2 FITS event list(s) with event-by-event Q and U Stokes parameters and W_MOM event weights
- algorithm choice among PHA1, PHA1Q, PHA1U (or normalized PHA1QN/PHA1UN variants)

## Outputs

- *.pha1.fits — standard OGIP PHA1 (Stokes I) count spectrum
- *.pha1q.fits — Q Stokes spectrum
- *.pha1u.fits — U Stokes spectrum
- XFLT0001 keyword set to Stokes:0/1/2 in each SPECTRUM extension

## How to apply

Load the level-2 FITS event list(s), which contain the event-by-event Q and U Stokes parameters and the W_MOM event weights. Run the ixpeobssim pipeline call pipeline.xpbin(*file_list, algorithm='PHA1') to bin the events in pulse-invariant (PI) space into a weighted count spectrum, where each PI-bin content is scaled by epsilon_k = (1/T) * (sum of weights)/(sum of squared weights). Repeat the binning with algorithm='PHA1Q' and 'PHA1U' so each event contributes its weighted q_i and u_i value to the corresponding PI bin. Write the three OGIP-modeled FITS binary tables (*.pha1.fits, *.pha1q.fits, *.pha1u.fits) and set the SPECTRUM extension XFLT0001 keyword to Stokes:0, Stokes:1 and Stokes:2 for the I, Q and U files respectively, so that XSPEC applies the polarization correction when fitting a model such as powerlaw * polconst.

## Related tools

- **ixpeobssim** (Provides the xpbin facility and pipeline.xpbin call that bin event lists into PHA1/PHA1Q/PHA1U Stokes spectral FITS files) — https://github.com/lucabaldini/ixpeobssim
- **XSPEC** (Consumes the I, Q and U Stokes spectra (via the XFLT0001 Stokes keyword) for spectro-polarimetric fitting, e.g. powerlaw * polconst)
- **Python** (Language in which the ixpeobssim pipeline and binning workflow are invoked)

## Examples

```
pipeline.xpbin(*file_list, algorithm='PHA1')
```

## Evaluation signals

- The three output FITS files (*.pha1.fits, *.pha1q.fits, *.pha1u.fits) exist after the xpbin run.
- The I spectrum is a standard PHA1 file following the OGIP spectral file format.
- XFLT0001 of the SPECTRUM extension is Stokes:0 for the I file, Stokes:1 for the Q file, and Stokes:2 for the U file.
- The spectra load in XSPEC and can be fit jointly with a spectro-polarimetric model such as powerlaw * polconst.

## Limitations

- The XFLT0001 Stokes keyword must be set correctly per flavor; if it is missing or wrong, XSPEC will not apply the polarization correction.
- The normalized variants PHA1QN/PHA1UN are separate algorithms — running plain PHA1Q/PHA1U will not produce normalized Q/U spectra.

## Evidence

- [methods] PHA1/PHA1Q/PHA1U invocation: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
- [methods] XSPEC feeding: "we feed the I, Q and U Stokes spectra into XSPEC and fit them with a ``powerlaw * polconst`` spectro-polarimetric model"
- [methods] Binning facilities: "ixpeobssim comes with its own set of facilities for binning event lists in several different flavors"
- [abstract] Event list outputs: "it is designed to produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data"
- [readme] README tool description: "ixpeobssim is a simulation and analysis framework specifically developed for the Imaging X-ray Polarimetry Explorer (IXPE)."
