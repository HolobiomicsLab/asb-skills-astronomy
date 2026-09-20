---
name: fits-event-file-construction
description: Use when simulating X-ray polarimetric observations that must be written
  as FITS event lists compatible with the IXPE data products (e.g. from ixpeobssim
  source models and IRFs).
when_to_use_negative:
- The task is analyzing existing IXPE data products rather than producing simulated
  event lists (use the binning applications like xpbin instead)
- The target mission format is not IXPE-like FITS event lists without adaptation,
  since the modular implementation must be adapted for missions other than IXPE
edam_operation: http://edamontology.org/operation_3679
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ixpeobssim
  role: Simulation framework that generates the event lists, applies the vignetting/fiducial-area/dead-time
    filtering stages, and writes the consolidated FITS file
  repo: https://github.com/lucabaldini/ixpeobssim
- name: astropy
  role: Used in validation to open the output FITS file and check the EVENTS extension,
    columns, and time keywords
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:28c0ac564f03aa2a7a5b4e88efb4c618738c425372af304c11e1370e255b2096
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - astropy
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_3679
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fits-event-file-construction

## Summary

Constructs consolidated IXPE-format FITS event lists by merging per-component partial event lists, applying instrumental correction stages, and writing EVENTS/GTI/MONTE_CARLO extensions with documented header keywords and columns.

## When to use

Use when simulating X-ray polarimetric observations that must be written as FITS event lists compatible with the IXPE data products (e.g. from ixpeobssim source models and IRFs).

## When NOT to use

- The task is analyzing existing IXPE data products rather than producing simulated event lists (use the binning applications like xpbin instead)
- The target mission format is not IXPE-like FITS event lists without adaptation, since the modular implementation must be adapted for missions other than IXPE

## Inputs

- ixpeobssim configuration file (e.g. ixpeobssim/config/toy_point_source.py) with IRFNAME/CALDB keywords
- Instrument response functions (IRFs)
- Photon spectrum model (e.g. power-law with constant polarization degree and angle)
- Partial per-component event lists from the ROI

## Outputs

- Consolidated FITS event list with EVENTS, GTI and MONTE_CARLO extensions
- Event columns TIME, ENERGY, X/Y, Q, U, W_MOM and header keywords TSTART/TSTOP (MET), TELAPSE, MJDREF 01 Jan 2017

## How to apply

Starting from a configuration file such as ixpeobssim/config/toy_point_source.py (power-law spectrum, constant polarization degree and angle) and the IRF identified by IRFNAME/CALDB, compute the count spectrum in energy and time (or phase) by convolving the photon spectrum with the instrument response, sampling 1D pdfs through interpolated splines built from the cumulative function as the ppf. Extract the photoelectron emission angle from the proper azimuthal distribution to populate DETPHI/PHI, Q, U and W_MOM columns. Merge the partial per-component event lists and sort in time, then run the dedicated filtering stages: vignetting (randomly discard events with the proper off-axis dependence), fiducial-area (discard events projected outside the readout-chip fiducial area), and dead-time (discard events during readout of an earlier event). Write the final consolidated event list as a FITS file with EVENTS, GTI and MONTE_CARLO extensions, TSTART/TSTOP in MET with MJDREF 01 Jan 2017, and TIME, ENERGY, X/Y, Q, U, W_MOM columns.

## Related tools

- **ixpeobssim** (Simulation framework that generates the event lists, applies the vignetting/fiducial-area/dead-time filtering stages, and writes the consolidated FITS file) — https://github.com/lucabaldini/ixpeobssim
- **astropy** (Used in validation to open the output FITS file and check the EVENTS extension, columns, and time keywords)

## Evaluation signals

- The output FITS file opens with astropy and contains an EVENTS extension with the documented columns (TIME, ENERGY, X/Y, Q, U, W_MOM)
- Header keywords are correct: TSTART/TSTOP in MET, TELAPSE = TSTOP - TSTART, MJDREF 01 Jan 2017
- The simulated count spectrum is statistically consistent with the input photon spectrum folded through the IRFs
- All events lie within the fiducial area of the readout chip and none occur during the readout of an earlier event (dead-time filter applied)
- The file contains a strict superset of the information included in the publicly released IXPE data products

## Limitations

- Much of the framework is specific to IXPE; adapting it to other missions with polarization capabilities requires work on the modular implementation
- The vignetting stage only approximates the vignetting function statistically, by randomly discarding a fraction of events with the proper off-axis angle dependence
- Dead-time losses arise because events occurring during the readout of an earlier event are discarded

## Evidence

- [abstract] the partial event lists are then merged and sorted in time. All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written: "the partial event lists are then merged and sorted in time. All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written"
- [abstract] events with a projected position on the focal plane lying outside the fiducial area of the readout chip are discarded: "events with a projected position on the focal plane lying outside the fiducial area of the readout chip are discarded"
- [readme] Given a source model and the response functions of the telescopes, it is designed to produce realistic simulated observations, in the form of event lists in FITS format: "Given a source model and the response functions of the telescopes, it is designed to produce realistic simulated observations, in the form of event lists in FITS format"
- [abstract] the correct vignetting function is obtained by randomly discarding a fraction of the events with the proper dependence on the off-axis angle: "the correct vignetting function is obtained by randomly discarding a fraction of the events with the proper dependence on the off-axis angle"
