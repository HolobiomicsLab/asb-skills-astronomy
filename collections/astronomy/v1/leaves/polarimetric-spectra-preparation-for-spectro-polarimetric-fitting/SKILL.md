---
name: polarimetric-spectra-preparation-for-spectro-polarimetric-fitting
description: Use when you have ixpeobssim-simulated (or released) IXPE FITS event
  lists containing event-by-event Q/U Stokes parameters and W_MOM weights, and need
  Stokes spectra ready for XSPEC spectro-polarimetric fitting.
when_to_use_negative:
- Input is already a binned polarization product (e.g. a PCUBE or PMAPCUBE from xpbin's
  --algorithm PCUBE/PMAPCUBE switches) rather than an event list to be binned in PI
  space
- The analysis goal is a normalized QN/UN Stokes profile rather than PHA1/PHA1Q/PHA1U
  spectra for XSPEC fitting (use the PHA1QN/PHA1UN algorithms instead)
- You need raw, unbinned event-by-event Stokes information rather than spectra suitable
  for feeding into XSPEC
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ixpeobssim
  role: Provides the xpbin facility and pipeline.xpbin call used to bin event lists
    into PHA1/PHA1Q/PHA1U Stokes spectra
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XSPEC
  role: Consumes the I, Q and U Stokes spectra for spectro-polarimetric fitting (e.g.
    powerlaw * polconst)
- name: Python
  role: Environment in which the ixpeobssim pipeline and xpbin binning are invoked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:41ecb1b89a80f14bcc34a6bf694b25f078e326cb91355ee275631ab88ee6bfb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - XSPEC
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# polarimetric-spectra-preparation-for-spectro-polarimetric-fitting

## Summary

Bins IXPE level-2 event lists into I, Q and U Stokes spectra via ixpeobssim's xpbin with the PHA1, PHA1Q and PHA1U algorithms, and tags the OGIP-format FITS spectra with the Stokes XFLT0001 keyword so XSPEC can perform spectro-polarimetric fitting.

## When to use

Use when you have ixpeobssim-simulated (or released) IXPE FITS event lists containing event-by-event Q/U Stokes parameters and W_MOM weights, and need Stokes spectra ready for XSPEC spectro-polarimetric fitting.

## When NOT to use

- Input is already a binned polarization product (e.g. a PCUBE or PMAPCUBE from xpbin's --algorithm PCUBE/PMAPCUBE switches) rather than an event list to be binned in PI space
- The analysis goal is a normalized QN/UN Stokes profile rather than PHA1/PHA1Q/PHA1U spectra for XSPEC fitting (use the PHA1QN/PHA1UN algorithms instead)
- You need raw, unbinned event-by-event Stokes information rather than spectra suitable for feeding into XSPEC

## Inputs

- Simulated or released IXPE level-2 FITS event list(s) with event-by-event Q and U Stokes parameters and W_MOM event weights
- Choice of xpbin algorithm ('PHA1', 'PHA1Q', 'PHA1U')

## Outputs

- OGIP-format PHA1 weighted count spectrum (*.pha1.fits) with XFLT0001 = Stokes:0
- Q Stokes spectrum (*.pha1q.fits) with XFLT0001 = Stokes:1
- U Stokes spectrum (*.pha1u.fits) with XFLT0001 = Stokes:2

## How to apply

Load the simulated level-2 FITS event list(s), which contain event-by-event Q and U Stokes parameters and the W_MOM event weights, and invoke the core pipeline call pipeline.xpbin(*file_list, algorithm='PHA1') to bin events in pulse-invariant (PI) space into a weighted count spectrum, where each PI-bin content is scaled by epsilon_k = (1/T) * (sum of weights)/(sum of squared weights). Repeat the binning with the PHA1Q and PHA1U algorithm values so each event contributes its weighted q_i and u_i value to the corresponding PI bin, yielding the Q and U Stokes spectra. Write the three flavors to OGIP-modeled FITS binary tables (*.pha1.fits, *.pha1q.fits, *.pha1u.fits), setting the XFLT0001 keyword of the SPECTRUM extension to Stokes:0, Stokes:1 and Stokes:2 for the I, Q and U files respectively, so XSPEC applies the polarization correction. The three spectra can then be fed into XSPEC and fit with a spectro-polarimetric model such as powerlaw * polconst.

## Related tools

- **ixpeobssim** (Provides the xpbin facility and pipeline.xpbin call used to bin event lists into PHA1/PHA1Q/PHA1U Stokes spectra) — https://github.com/lucabaldini/ixpeobssim
- **XSPEC** (Consumes the I, Q and U Stokes spectra for spectro-polarimetric fitting (e.g. powerlaw * polconst))
- **Python** (Environment in which the ixpeobssim pipeline and xpbin binning are invoked)

## Examples

```
pipeline.xpbin(*file_list, algorithm='PHA1')  # then repeat with algorithm='PHA1Q' and algorithm='PHA1U'
```

## Evaluation signals

- All three output FITS files (*.pha1.fits, *.pha1q.fits, *.pha1u.fits) exist after the xpbin runs
- The I spectrum is a standard PHA1 file following the OGIP spectral file format
- XFLT0001 keyword of the SPECTRUM extension is Stokes:0 in the I file, Stokes:1 in the Q file, and Stokes:2 in the U file
- The Q and U spectra reflect per-event weighted q_i/u_i contributions in each PI bin, consistent with the W_MOM weighting scheme
- The three spectra load in XSPEC and support a spectro-polarimetric model fit (e.g. powerlaw * polconst) with the polarization correction applied via the Stokes keyword

## Limitations

- The weighting scheme scales each PI bin by epsilon_k = (1/T) * (sum of weights)/(sum of squared weights), so the binned spectra are weighted count spectra, not plain counts; downstream statistics must account for this
- The Stokes XFLT0001 keyword convention (Stokes:0/1/2) must be set correctly per flavor or XSPEC will not apply the polarization correction
- ixpeobssim is specific to IXPE data conventions, although the modular implementation could be adapted to other polarization-capable missions

## Evidence

- [methods] xpbin with PHA1/PHA1Q/PHA1U algorithms produces the three Stokes spectral flavors: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
- [methods] Stokes spectra are fitted in XSPEC with a spectro-polarimetric model: "we feed the I, Q and U Stokes spectra into XSPEC and fit them with a ``powerlaw * polconst`` spectro-polarimetric model"
- [methods] ixpeobssim has dedicated binning facilities for event lists: "ixpeobssim comes with its own set of facilities for binning event lists in several different flavors"
- [abstract] Simulations produce FITS event lists that are supersets of released IXPE products: "produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data"
- [readme] ixpeobssim supports interoperation with standard X-ray analysis tools: "facilitating the interoperation with the standard visualization and analysis tools traditionally in use by the X-ray community"
