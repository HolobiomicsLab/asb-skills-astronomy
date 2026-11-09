---
name: stokes-spectrum-binning
description: Use when you have simulated or observed IXPE event lists (FITS format,
  one per detector unit) and need binned Stokes I/Q/U spectra (PHA1/PHA1Q/PHA1U) for
  spectro-polarimetric fitting or ensemble statistical analysis.
when_to_use_negative:
- You need spatially resolved polarization maps or modulation cubes rather than energy
  spectra — use the PCUBE or PMAPCUBE binning algorithms instead of the PHA1 family.
- The input is already a binned PHA1/PHA1Q/PHA1U product — rebinning binned files
  is not the intended flow; binning operates on event lists.
- You need background-subtracted or rescaled products — background subtraction is
  a separate downstream step applied to binned polarization data, not part of the
  xpbin binning itself.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ixpeobssim
  role: Provides the xpbin application that bins event lists into PHA1/PHA1Q/PHA1U
    Stokes spectra within the simulation-and-analysis pipeline
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XSPEC
  role: Downstream consumer of the binned I/Q/U Stokes spectra, fitting them with
    spectro-polarimetric models such as 'powerlaw * polconst'
- name: HEASOFT
  role: Standard X-ray analysis stack with which ixpeobssim's binned products interoperate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:382e4ed37f24204b2d7b0829ba62ab3d3c7f922b1c81279ee7d20169f6889614
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - HEASOFT
  - Python
  - XSPEC
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# stokes-spectrum-binning

## Summary

Binning IXPE FITS event lists into Stokes I, Q, and U spectra using the ixpeobssim xpbin application with the PHA1, PHA1Q, and PHA1U algorithms, producing spectro-polarimetric products suitable for XSPEC fitting.

## When to use

Use when you have simulated or observed IXPE event lists (FITS format, one per detector unit) and need binned Stokes I/Q/U spectra (PHA1/PHA1Q/PHA1U) for spectro-polarimetric fitting or ensemble statistical analysis.

## When NOT to use

- You need spatially resolved polarization maps or modulation cubes rather than energy spectra — use the PCUBE or PMAPCUBE binning algorithms instead of the PHA1 family.
- The input is already a binned PHA1/PHA1Q/PHA1U product — rebinning binned files is not the intended flow; binning operates on event lists.
- You need background-subtracted or rescaled products — background subtraction is a separate downstream step applied to binned polarization data, not part of the xpbin binning itself.

## Inputs

- IXPE FITS event lists per detector unit (DU), either observed or produced by ixpeobssim simulation
- Binning algorithm selection (PHA1, PHA1Q, PHA1U) passed to xpbin via command-line arguments

## Outputs

- PHA1 binned intensity spectrum (Stokes I) per realization and detector
- PHA1Q binned Q Stokes spectrum
- PHA1U binned U Stokes spectrum
- Stokes spectra formatted for downstream XSPEC spectro-polarimetric fitting

## How to apply

Run the xpbin application on each per-detector FITS event list, invoking it once per flavor: 'PHA1' for the intensity (Stokes I) spectrum, 'PHA1Q' for the Q Stokes spectrum, and 'PHA1U' for the U Stokes spectrum. This step typically follows ensemble generation (e.g., pipeline.generate_ensamble with size=1000 and duration=50000 s for a toy source model), where standard_ensamble_processing runs xpbin on each realization automatically. The binned Stokes spectra are designed to interoperate with XSPEC, where I, Q, and U are fitted jointly with a spectro-polarimetric model such as 'powerlaw * polconst'. The output products are small relative to the event lists, so intermediate event files can be deleted after binning to leave only the binned products on disk. For related polarization products, xpbin also supports PCUBE modulation cubes and PMAPCUBE maps, but those are separate algorithms from the PHA1-family spectral binning.

## Related tools

- **ixpeobssim** (Provides the xpbin application that bins event lists into PHA1/PHA1Q/PHA1U Stokes spectra within the simulation-and-analysis pipeline) — https://github.com/lucabaldini/ixpeobssim
- **XSPEC** (Downstream consumer of the binned I/Q/U Stokes spectra, fitting them with spectro-polarimetric models such as 'powerlaw * polconst')
- **HEASOFT** (Standard X-ray analysis stack with which ixpeobssim's binned products interoperate)

## Examples

```
ixpeobssim.pipeline.bootstrap_pipeline('toy_pollin'); pipeline.generate_ensamble(size=1000., duration=50000.)  # standard_ensamble_processing then runs xpbin with PHA1, PHA1Q, PHA1U
```

## Evaluation signals

- Each event list realization yields all three binned flavors (PHA1, PHA1Q, PHA1U) — an ensemble is complete only when every realization has the full set.
- The binned Q and U spectra must be consistent with the I spectrum's energy binning so they can be jointly loaded into XSPEC as a spectro-polarimetric dataset.
- For Monte Carlo ensembles (e.g., 1000 realizations of a 50 ks observation), the collected per-realization Stokes estimates should support meaningful pull-distribution / posterior parameter analysis.
- The binned products should interoperate with standard X-ray tools (HEASOFT/XSPEC) without format errors.
- Cleanup invariant: after processing, the large intermediate event files are deleted and only the binned products remain on disk.

## Limitations

- The binned products are only as good as the instrument response and source model used upstream; the simulation convolves ground truth with the instrument response, and filtering stages (vignetting, chip fiducial area, deadtime) shape the event lists before binning.
- ixpeobssim event lists are a strict superset of the publicly released IXPE data products, so binning behavior calibrated on simulated lists may need checking against official HEASARC products.
- The framework is IXPE-specific in much of its implementation; adapting it to other polarization-capable missions is possible but not automatic.

## Evidence

- [methods] PHA1/PHA1Q/PHA1U binning via xpbin: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
- [methods] Ensemble generation gauging statistical properties: "generate a large number of different realizations of the same observation to gauge the statistical properties"
- [methods] XSPEC fitting of Stokes spectra: "we feed the I, Q and U Stokes spectra into XSPEC and fit them with a ``powerlaw * polconst`` spectro-polarimetric model"
- [readme] Event lists as FITS superset of public products: "it is designed to produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information"
- [methods] Binning facilities exist for event lists: "ixpeobssim comes with its own set of facilities for binning event lists in several different flavors"
