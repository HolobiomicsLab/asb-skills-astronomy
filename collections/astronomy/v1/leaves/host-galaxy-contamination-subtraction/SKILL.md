---
name: host-galaxy-contamination-subtraction
description: Use when a transient spectrum from an IFS cube is contaminated by host-galaxy
  light — e.g. the transient lies near the host core (~30% of SEDm observations show
  severe contamination) and an isolated point-source extractor would yield biased
  or unclassifiable spectra.
when_to_use_negative:
- The transient is an isolated point source far from its host core — the standard
  pysedm extraction intrinsically assumes an isolated point source and is sufficient
- No external host-galaxy redshift measurement is available — HyperGal's most limiting
  constraint is the need for an external redshift measurement of the host galaxy
- The residual host component is needed for a posteriori measurements (e.g. redshift
  or local Halpha flux) — the modeling precludes such use
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: HyperGal
  role: Chromatic scene modeler that fits host galaxy, point-source transient, and
    background components to the SEDm cube and subtracts the host contribution
  repo: https://github.com/JeremyLezmy/hypergal
- name: CIGALE
  role: SED fitter used as a physically-motivated spectral interpolator to convert
    pre-transient host photometry into the hyperspectral galaxy cube
- name: iminuit
  role: Performs the per-metaslice 2D weighted chi-square minimization of scene parameters
    over the SN position
  repo: https://github.com/scikit-hep/iminuit
- name: pysedm
  role: Baseline SEDm data-reduction pipeline whose isolated point-source extraction
    is the comparison standard for host contamination handling
- name: SNID
  role: Classifies the extracted transient spectrum to verify that host subtraction
    enables correct typing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:bc927bd7f2b6bfed3a4adbd0ef33960403386a747223e1213de41e9188d82102
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - HyperGal (hypergal)
  - SNID
  - iminuit
  - pysedm
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Host-galaxy contamination subtraction via hyperspectral scene modeling

## Summary

Subtracts structured host-galaxy light from a low-resolution integral-field spectrograph cube (SEDm) to recover a clean transient spectrum, using a hyperspectral host model built from pre-transient photometry (CIGALE) plus chromatic PSF/ADR and background components fitted with progressive 2D/1D/3D stages.

## When to use

Use when a transient spectrum from an IFS cube is contaminated by host-galaxy light — e.g. the transient lies near the host core (~30% of SEDm observations show severe contamination) and an isolated point-source extractor would yield biased or unclassifiable spectra.

## When NOT to use

- The transient is an isolated point source far from its host core — the standard pysedm extraction intrinsically assumes an isolated point source and is sufficient
- No external host-galaxy redshift measurement is available — HyperGal's most limiting constraint is the need for an external redshift measurement of the host galaxy
- The residual host component is needed for a posteriori measurements (e.g. redshift or local Halpha flux) — the modeling precludes such use

## Inputs

- SEDm data cube (spectro-spatial observation space) of the transient field
- Pre-transient multi-band photometric images of the host galaxy (e.g. PS1) for B_gal estimation
- Host-galaxy redshift measurement (external prior)
- CIGALE-fitted hyperspectral host galaxy model cube
- Estimated sky background level B_sky from two sky levels

## Outputs

- Extracted transient (SN) spectrum with host-galaxy and background components subtracted
- Fitted scene component amplitudes G(lambda) for host, SN, and background polynomial
- Wavelength-dependent PSF/ADR parameters from the chromatic 1D fit
- Continuum-divided spectral relative RMS (Eq. 14) over [4000, 8000] Å as an accuracy metric
- SNID classification of the extracted spectrum (rlap_min = 6, 50% top-10 model agreement)

## How to apply

First build a projected host galaxy hyperspectral model from pre-transient photometric images (PS1 for B_gal, plus an estimated B_sky from two sky levels), using CIGALE as a physically-motivated spectral interpolator, and inject it with a point-source and a 6-parameter diffuse background component into SEDm observation space. Then run a per-metaslice 2D weighted chi-square minimization (iminuit) over the SN position with a constrained chromatic Gaussian+Moffat PSF, a differential PSF, and the background model, followed by a chromatic 1D fit of the PSF/ADR models across wavelength to refine wavelength-dependent point-spread parameters. Finally solve a 3D linear amplitude fit over all monochromatic slices in which only the scaling amplitudes of the host galaxy, SN, and background polynomial components remain free — morphology is already fixed by the earlier 2D and 1D fits — and take the SN component amplitudes as the extracted transient spectrum. Quantify extraction accuracy with the continuum-divided spectral relative RMS (Eq. 14) between input and extracted spectra averaged over [4000, 8000] Å; HyperGal reaches the 5% level for 60% of simulated observations.

## Related tools

- **HyperGal** (Chromatic scene modeler that fits host galaxy, point-source transient, and background components to the SEDm cube and subtracts the host contribution) — https://github.com/JeremyLezmy/hypergal
- **CIGALE** (SED fitter used as a physically-motivated spectral interpolator to convert pre-transient host photometry into the hyperspectral galaxy cube)
- **iminuit** (Performs the per-metaslice 2D weighted chi-square minimization of scene parameters over the SN position) — https://github.com/scikit-hep/iminuit
- **pysedm** (Baseline SEDm data-reduction pipeline whose isolated point-source extraction is the comparison standard for host contamination handling)
- **SNID** (Classifies the extracted transient spectrum to verify that host subtraction enables correct typing)

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- Continuum-normalized spectral relative RMS over [4000, 8000] Å for contrast c > 0.5 simulations should fall below ~5% (HyperGal reaches the 5% level for 60% of simulated observations)
- SNID typing with strict criteria (rlap_min = 6 and ≥50% top-10 model agreement) should classify the extracted spectrum; uncertain if either criterion fails
- SN Ia classification rate on the real SEDm contrast distribution should approach ~95% with a false positive rate below 2%
- Comparison against the standard (pysedm-style) extraction should show ~10% more correctly classified SNe Ia
- In the final 3D fit, only scaling amplitudes of host/SN/background vary; if morphology parameters shift, the 2D/1D fit stages were not converged before the 3D fit

## Limitations

- Requires an external host-galaxy redshift measurement as input — the most limiting constraint
- The true SN–host distance distribution was not explicitly modeled (marginalized uniformly between 0 and 5 arcsec)
- The interaction of monochromatic galaxy amplitudes G(lambda) with emission line positions/intensities depends on the stellar vs. gaseous component distribution and has not been studied extensively
- The residual host component cannot be used for a posteriori measurements such as redshift or local Halpha flux
- For ~50% of ZTF SNe lacking spectral host redshifts, redshift comes from a preliminary SN extraction with coarser precision (~5e-3)

## Evidence

- [abstract] uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator: "uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
- [abstract] The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [methods] of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
- [discussion] corrected to first order by the monochromatic galaxy amplitudes G(lambda) during the ultimate 3D fit: [evidence span withheld — anchor status: not_found]
- [readme] HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
