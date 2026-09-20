---
name: stokes-parameter-event-binning
description: Use when you have an ixpeobssim-simulated (or IXPE level-2) FITS event
  list containing photoelectron emission angles and need to recover polarization degree/angle
  by computing Stokes parameters in energy bins.
when_to_use_negative:
- You need spatially resolved polarization maps rather than source-averaged Stokes
  parameters — use the PMAPCUBE binning algorithm instead of PCUBE
- You only need spectral (non-polarimetric) products — the plain PHA1 binning flavor
  is the right choice
- The input is raw level-1 track data rather than a reconstructed level-2 event list
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: ixpeobssim
  role: Provides the xpbin application with the PCUBE binning algorithm that computes
    Stokes parameters, MDP, polarization degree and angle from the event list
  repo: https://github.com/lucabaldini/ixpeobssim
- name: Python
  role: Runtime environment in which ixpeobssim and its xpbin binning facilities are
    executed
- name: XSPEC
  role: Optional downstream fitter for PHA1/PHA1Q/PHA1U Stokes spectra with spectro-polarimetric
    models such as powerlaw * polconst
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:a8651e7803c9c36285e57dd435089ad38fc84b9c55cf06c152370c72a060f50d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - XSPEC
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# stokes-parameter-event-binning

## Summary

Binning ixpeobssim level-2 FITS event lists into polarization cubes with xpbin's PCUBE algorithm, which computes the Stokes parameters (I, Q, U), modulation factor-corrected polarization degree and angle, MDP, and their uncertainties in energy bins.

## When to use

Use when you have an ixpeobssim-simulated (or IXPE level-2) FITS event list containing photoelectron emission angles and need to recover polarization degree/angle by computing Stokes parameters in energy bins.

## When NOT to use

- You need spatially resolved polarization maps rather than source-averaged Stokes parameters — use the PMAPCUBE binning algorithm instead of PCUBE
- You only need spectral (non-polarimetric) products — the plain PHA1 binning flavor is the right choice
- The input is raw level-1 track data rather than a reconstructed level-2 event list

## Inputs

- ixpeobssim level-2 FITS event list (from xpobssim) containing photoelectron emission angles, energies, and detector coordinates
- Instrument response files (modulation factor, effective area) needed to convert modulation to polarization

## Outputs

- Polarization cube (PCUBE binned file) with Stokes parameters I, Q, U per energy bin
- Measured polarization degree and angle with statistical uncertainties per energy bin
- Minimum detectable polarization (MDP) per energy bin
- Optionally, PHA1/PHA1Q/PHA1U (or PHA1QN/PHA1UN) Stokes spectra for XSPEC fitting

## How to apply

Run the xpbin application on the level-2 FITS event list produced by xpobssim, passing the --algorithm PCUBE command-line switch so the Stokes parameters are computed from the azimuthal distribution of the photoelectron emission angle. The PCUBE output (the polarization cube) contains, per energy bin, the Stokes parameters I, Q, and U, the minimum detectable polarization (MDP), and the polarization degree and angle with associated statistical uncertainties. Because the measured azimuthal modulation is diluted by the instrument response, convert the modulation to a physical polarization using the modulation factor — the modulation factor serves the purpose of converting the azimuthal modulation measured by the detector into the actual source polarization. For spectro-polarimetric work, the companion PHA1/PHA1Q/PHA1U (or normalized PHA1QN/PHA1UN) bin flavors produce Stokes spectra that can be fed into XSPEC and fitted with a 'powerlaw * polconst' spectro-polarimetric model. Finally, extract the measured polarization degree and angle from the polarization cube and compare them against the input model values within their statistical uncertainties.

## Related tools

- **ixpeobssim** (Provides the xpbin application with the PCUBE binning algorithm that computes Stokes parameters, MDP, polarization degree and angle from the event list) — https://github.com/lucabaldini/ixpeobssim
- **Python** (Runtime environment in which ixpeobssim and its xpbin binning facilities are executed)
- **XSPEC** (Optional downstream fitter for PHA1/PHA1Q/PHA1U Stokes spectra with spectro-polarimetric models such as powerlaw * polconst)

## Examples

```
xpbin --algorithm PCUBE event_list.fits
```

## Evaluation signals

- The PCUBE output contains Stokes parameters I, Q, U plus MDP, polarization degree and angle per energy bin with associated uncertainties
- Recovered polarization degree and angle are consistent with the known constant input values within their statistical uncertainties
- Over many realizations (e.g. the 1000-realization ensemble benchmark), fit-parameter pulls are distributed as standard normal variables with zero mean and unit standard deviation
- Binned files are produced by xpbin invoked with the correct --algorithm switch (PCUBE for polarization cubes, PHA1/PHA1Q/PHA1U for Stokes spectra)

## Limitations

- PCUBE yields source-averaged (or region-averaged) polarization per energy bin; spatially resolved polarization requires the PMAPCUBE algorithm instead
- The measured azimuthal modulation must be corrected by the modulation factor to yield the true source polarization; skipping this correction gives a biased polarization degree
- Results are statistical: validation of the recovery requires an ensemble of realizations (e.g. 1000 observations of 50 ks) to confirm pulls are standard-normal distributed

## Evidence

- [methods] PCUBE binning is invoked via xpbin command-line switch: "generated by |xpbin| run with the ``--algorithm PCUBE`` command-line switch"
- [methods] xpbin computes Stokes parameters, MDP, polarization degree and angle: [evidence span withheld — anchor status: not_found]
- [results] Modulation factor converts measured azimuthal modulation to source polarization: [evidence span withheld — anchor status: not_found]
- [methods] ixpeobssim has its own binning facilities for event lists: "ixpeobssim comes with its own set of facilities for binning event lists in several different flavors"
- [methods] PHA1/PHA1Q/PHA1U bin flavors for Stokes spectra: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
