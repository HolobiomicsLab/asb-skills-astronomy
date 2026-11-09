---
name: instrument-response-convolution
description: Use when generating simulated IXPE event lists from a source model (e.g.
  a power-law spectrum with constant polarization degree/angle) plus IRF definitions
  identified by IRFNAME/CALDB keywords, and the count spectrum must be produced by
  convolution with the instrument response.
when_to_use_negative:
- Input is already an observed (level-1/level-2) IXPE FITS event list from the HEASARC
  rather than a photon source model — the convolution step is a forward-simulation
  stage, not an analysis of existing data.
- The goal is binning or analysis of existing event lists (e.g. PCUBE, PMAPCUBE, PHA1/PHA1Q/PHA1U
  products via xpbin) rather than simulating a new observation.
- The mission is not IXPE and no compatible polarization-capable instrument response
  is available; the framework is IXPE-specific, though its modular design makes adaptation
  to other missions possible.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ixpeobssim
  role: Simulation framework that convolves the photon spectrum with the instrument
    response functions and samples event pdfs to produce FITS event lists
  repo: https://github.com/lucabaldini/ixpeobssim
- name: Python
  role: Implementation language; one-dimensional pdfs are sampled via interpolated
    splines built from the cumulative function
- name: CALDB
  role: Source of the instrument response functions referenced via the CALDB keyword
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:970dcda08b6b8e85743c70e1f1db0fab65b3378f85b48630563a85fa8dbb824f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - CALDB
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# instrument-response-convolution

## Summary

Folding a photon source model through IXPE instrument response functions (IRFs) to compute the count spectrum as a function of energy and time/phase, then sampling one-dimensional pdfs via interpolated splines built from the cumulative function as the percent point function (ppf) — the first stage of the ixpeobssim simulation flow for a single on-axis model component.

## When to use

Use when generating simulated IXPE event lists from a source model (e.g. a power-law spectrum with constant polarization degree/angle) plus IRF definitions identified by IRFNAME/CALDB keywords, and the count spectrum must be produced by convolution with the instrument response.

## When NOT to use

- Input is already an observed (level-1/level-2) IXPE FITS event list from the HEASARC rather than a photon source model — the convolution step is a forward-simulation stage, not an analysis of existing data.
- The goal is binning or analysis of existing event lists (e.g. PCUBE, PMAPCUBE, PHA1/PHA1Q/PHA1U products via xpbin) rather than simulating a new observation.
- The mission is not IXPE and no compatible polarization-capable instrument response is available; the framework is IXPE-specific, though its modular design makes adaptation to other missions possible.

## Inputs

- ixpeobssim configuration file defining the source model (e.g. ixpeobssim/config/toy_point_source.py with a power-law spectrum and constant polarization degree and angle)
- Instrument response functions (IRFs) identified by the IRFNAME/CALDB keywords
- Photon spectrum of the source model (energy and time/phase dependence)

## Outputs

- Per-component partial event lists later merged and sorted in time
- Consolidated FITS event list with EVENTS, GTI and MONTE_CARLO extensions (columns TIME, ENERGY, X/Y, Q, U, W_MOM; TSTART/TSTOP in MET, MJDREF 01 Jan 2017)
- Simulated count spectrum as a function of energy and time (or phase)

## How to apply

Load the ixpeobssim configuration file (e.g. ixpeobssim/config/toy_point_source.py) together with the IRF identified by the IRFNAME/CALDB keywords. Compute the count spectrum as a function of energy and time (or phase) by convolving the input photon spectrum with the instrument response functions. Sample one-dimensional pdfs by calculating the cumulative function on a suitable regular grid and building an interpolated spline representing the percent point function (ppf). Extract the photoelectron emission angle for each event from the proper azimuthal distribution, populating the DETPHI/PHI, Q, U and W_MOM event columns. Downstream stages (vignetting, fiducial-area and dead-time filtering) discard events, and the merged event list is finally sorted in time and written as a FITS file with EVENTS, GTI and MONTE_CARLO extensions.

## Related tools

- **ixpeobssim** (Simulation framework that convolves the photon spectrum with the instrument response functions and samples event pdfs to produce FITS event lists) — https://github.com/lucabaldini/ixpeobssim
- **Python** (Implementation language; one-dimensional pdfs are sampled via interpolated splines built from the cumulative function)
- **CALDB** (Source of the instrument response functions referenced via the CALDB keyword)

## Examples

```
xpsimulation.py --config ixpeobssim/config/toy_point_source.py --irfname ixpe_svpr2v1
```

## Evaluation signals

- The output FITS file opens with astropy and contains an EVENTS extension with the documented columns (TIME, ENERGY, X/Y, Q, U, W_MOM).
- Header keywords conform to the documented definitions: TSTART/TSTOP in MET, MJDREF 01 Jan 2017, and TELAPSE = TSTOP - TSTART.
- The simulated count spectrum is statistically consistent with the input photon spectrum folded through the IRFs.
- The vignetting stage correctly discards a random fraction of events with the proper dependence on the off-axis angle.
- Events outside the fiducial area of the readout chip and events occurring during the readout of an earlier event (dead-time) are absent from the final event list.

## Limitations

- The simulation produces event lists containing a strict superset of the information in publicly released IXPE data products, so downstream analyses should not assume exact parity with official HEASARC level-1/level-2 files.
- Instrumental corrections are applied via random event-discard stages (vignetting, fiducial area, dead-time), which introduces statistical, not deterministic, corrections.
- Adapting the framework to other missions with polarization capabilities is 'potentially straightforward' due to modularity but the implementation is largely IXPE-specific.

## Evidence

- [abstract] Simulation of a single model component starts with calculating the count spectrum from the photon spectrum convolved with the instrument response.: "the basic flow of the simulation for a single model component starts with the calculation of the count spectrum, as a function of energy and time (or phase), given the photon spectrum"
- [abstract] One-dimensional pdf sampling via ppf splines.: "we calculate the cumulative function on a suitable, regular grid and use the values to build an interpolated spline representing the percent point function (ppf)"
- [abstract] Merging, sorting, and dedicated filtering stages before writing the final event list.: "the partial event lists are then merged and sorted in time. All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written"
- [readme] ixpeobssim produces realistic simulated observations as FITS event lists.: "it is designed to produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data"
- [abstract] Photoelectron emission angle extraction once the response is applied.: "we also have all the necessary ingredients to extract the photo-electron emission angle, according to the proper azimuthal distribution"
