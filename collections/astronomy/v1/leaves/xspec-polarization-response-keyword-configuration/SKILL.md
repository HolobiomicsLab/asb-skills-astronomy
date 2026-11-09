---
name: xspec-polarization-response-keyword-configuration
description: Use when binned ixpeobssim PHA1/PHA1Q/PHA1U Stokes spectra must be fed
  into XSPEC for spectro-polarimetric fitting and the per-file XFLT0001 Stokes keyword
  (Stokes:0/1/2) needs to be set or verified.
when_to_use_negative:
- The input is a PCUBE or PMAPCUBE polarization cube from xpbin rather than PHA1/PHA1Q/PHA1U
  spectral binned files
- The target is ordinary (non-polarization) spectral fitting where no I/Q/U trio is
  being combined in XSPEC
- The events have not yet been binned — this skill presumes xpbin has already produced
  the three flavor files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ixpeobssim
  role: Produces the PHA1, PHA1Q and PHA1U Stokes spectral binned files via the xpbin
    facility and writes the OGIP FITS tables
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XSPEC
  role: Consumes the I, Q and U Stokes spectra (keyed by XFLT0001 Stokes:0/1/2) for
    spectro-polarimetric fitting with models like powerlaw * polconst
- name: Python
  role: Used for FITS keyword manipulation and driving the ixpeobssim pipeline calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:e51f2e44af7f11495e37807e0536be6968d6ec6f5d75b5265aa2d5c10edaef6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - XSPEC
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# xspec-polarization-response-keyword-configuration

## Summary

Configures the three ixpeobssim PHA1-flavor Stokes products (I, Q, U) with the XSPEC polarization response conventions, ensuring each SPECTRUM extension carries the correct XFLT0001 Stokes keyword so that XSPEC applies the polarization correction during spectro-polarimetric fitting.

## When to use

Use when binned ixpeobssim PHA1/PHA1Q/PHA1U Stokes spectra must be fed into XSPEC for spectro-polarimetric fitting and the per-file XFLT0001 Stokes keyword (Stokes:0/1/2) needs to be set or verified.

## When NOT to use

- The input is a PCUBE or PMAPCUBE polarization cube from xpbin rather than PHA1/PHA1Q/PHA1U spectral binned files
- The target is ordinary (non-polarization) spectral fitting where no I/Q/U trio is being combined in XSPEC
- The events have not yet been binned — this skill presumes xpbin has already produced the three flavor files

## Inputs

- Level-2 FITS event list(s) from the ixpeobssim simulation pipeline containing event-by-event Q and U Stokes parameters and W_MOM event weights
- Binned ixpeobssim Stokes spectral products: *.pha1.fits, *.pha1q.fits, *.pha1u.fits (OGIP-modeled FITS binary tables)

## Outputs

- Three XSPEC-ready Stokes spectral files with XFLT0001 set to Stokes:0, Stokes:1 and Stokes:2 respectively
- XSPEC spectro-polarimetric fit results (e.g. from a powerlaw * polconst model)

## How to apply

First bin the level-2 FITS event list with xpbin using the PHA1, PHA1Q and PHA1U algorithm values, producing *.pha1.fits, *.pha1q.fits and *.pha1u.fits OGIP-modeled binary tables; the I spectrum is a standard PHA1 file following the OGIP spectral file format, while Q and U spectra contain each event's weighted q_i and u_i value per PI bin. Then set the XFLT0001 keyword of each file's SPECTRUM extension to Stokes:0 for the I file, Stokes:1 for the Q file, and Stokes:2 for the U file, since XSPEC uses this keyword to apply the polarization correction. Finally load the three spectra together in XSPEC and fit them with a spectro-polarimetric model such as powerlaw * polconst. Correct keyword configuration is what makes XSPEC interpret the trio as I/Q/U components of a single polarization measurement rather than ordinary spectra.

## Related tools

- **ixpeobssim** (Produces the PHA1, PHA1Q and PHA1U Stokes spectral binned files via the xpbin facility and writes the OGIP FITS tables) — https://github.com/lucabaldini/ixpeobssim
- **XSPEC** (Consumes the I, Q and U Stokes spectra (keyed by XFLT0001 Stokes:0/1/2) for spectro-polarimetric fitting with models like powerlaw * polconst)
- **Python** (Used for FITS keyword manipulation and driving the ixpeobssim pipeline calls)

## Examples

```
pipeline.xpbin(*file_list, algorithm='PHA1Q'); hdr['XFLT0001'] = 'Stokes:1'
```

## Evaluation signals

- All three output FITS files (*.pha1.fits, *.pha1q.fits, *.pha1u.fits) exist after the binning run
- The I spectrum is a standard PHA1 file following the OGIP spectral file format
- XFLT0001 of the SPECTRUM extension equals Stokes:0 for the I file, Stokes:1 for the Q file, and Stokes:2 for the U file
- XSPEC accepts the three spectra jointly and applies the polarization correction when fitting the powerlaw * polconst model
- Each PI bin of the Q/U spectra reflects event-weighted q_i/u_i contributions (epsilon_k = (1/T) * sum(weights)/sum(squared weights)) rather than raw counts

## Limitations

- The non-normalized PHA1QN/PHA1UN binning algorithms are a separate flavor; using them changes how the spectra must be interpreted in XSPEC
- The keyword convention described is specific to the ixpeobssim/IXPE OGIP FITS conventions and the XSPEC polarization correction; other missions or formats may not honor XFLT0001 Stokes keywords
- ixpeobssim is modular and primarily IXPE-specific — adapting it to other polarization-capable missions requires verifying the response-keyword interoperation

## Evidence

- [methods] binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
- [methods] we feed the I, Q and U Stokes spectra into XSPEC and fit them with a ``powerlaw * polconst`` spectro-polarimetric model: "we feed the I, Q and U Stokes spectra into XSPEC and fit them with a ``powerlaw * polconst`` spectro-polarimetric model"
- [readme] facilitating the interoperation with the standard visualization and analysis tools traditionally in use by the X-ray community: "facilitating the interoperation with the standard visualization and analysis tools traditionally in use by the X-ray community"
- [methods] ixpeobssim comes with its own set of facilities for binning event lists in several different flavors: "ixpeobssim comes with its own set of facilities for binning event lists in several different flavors"
