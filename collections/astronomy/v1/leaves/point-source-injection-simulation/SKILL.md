---
name: point-source-injection-simulation
description: Use when you need to quantify the accuracy of a transient spectrum extractor
  (e.g. HyperGal vs. pysedm) on IFS cubes by injecting sources of known contrast and
  spectrum, then measuring recovery error per contrast bin.
when_to_use_negative:
- The goal is classification/typing performance rather than extraction accuracy —
  use SNID classification with rlap_min = 6 and the 50% top-10 model agreement criterion
  instead
- The transient is treated as an isolated point source with no structured host background
  — the standard PSF extraction path (e.g. pysedm) applies, and injection-based scene-modeling
  validation is unnecessary
- No pre-transient photometric host reference is available to build the hyperspectral
  galaxy model required as the injection scene
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HyperGal
  role: Scene-modeling pipeline whose extraction accuracy is being validated via injection
    and relative-RMS recovery
  repo: https://github.com/JeremyLezmy/hypergal
- name: iminuit
  role: Minimizer for the per-metaslice 2D weighted chi-square fit of SN position,
    PSF, and background parameters during the injection test
  repo: https://github.com/scikit-hep/iminuit
- name: SEDm (SEDmachine)
  role: Integral field spectrograph whose synthetic cubes receive the injected point
    source and onto which the scene model is projected
- name: CIGALE
  role: SED fitter used as physically-motivated spectral interpolator to generate
    the host-galaxy hyperspectral model used as the injection background
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:489c672f7b33275f94c1083dd53cf6ceceaf1cae62e0b96d9832eaa4458384dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - HyperGal (hypergal)
  - SEDm (SEDmachine)
  - iminuit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# point-source-injection-simulation

## Summary

Validation-by-injection scheme for scene-modeling spectrum extraction: inject a point source with a known input spectrum into a synthetic SEDm data cube at a controlled SN/host contrast c, run the extraction pipeline, and quantify recovered-spectrum fidelity via continuum-divided spectral relative RMS. It isolates extraction accuracy from classifier behavior by comparing extracted vs. injected spectra.

## When to use

Use when you need to quantify the accuracy of a transient spectrum extractor (e.g. HyperGal vs. pysedm) on IFS cubes by injecting sources of known contrast and spectrum, then measuring recovery error per contrast bin.

## When NOT to use

- The goal is classification/typing performance rather than extraction accuracy — use SNID classification with rlap_min = 6 and the 50% top-10 model agreement criterion instead
- The transient is treated as an isolated point source with no structured host background — the standard PSF extraction path (e.g. pysedm) applies, and injection-based scene-modeling validation is unnecessary
- No pre-transient photometric host reference is available to build the hyperspectral galaxy model required as the injection scene

## Inputs

- Projected hyperspectral host-galaxy model cube (from CIGALE SED fitting of pre-transient photometric images, e.g. PS1)
- Known input transient spectrum for the injected point source
- Synthetic SEDm (SEDmachine) data cube with a target contrast c (c > 0.5 for the 5%-level validation regime)
- Wavelength domain [4000, 8000] Å for RMS evaluation

## Outputs

- Extracted transient spectrum from the 3D linear amplitude fit (host galaxy, transient, background components)
- Continuum-divided spectral relative RMS (Eq. 14) between input and extracted spectra
- Spectral RMS distribution binned by contrast (continuum divided)

## How to apply

Build a projected hyperspectral host-galaxy model and inject a point source with a known input spectrum into a synthetic SEDm cube at a fixed contrast c > 0.5. Run the extraction pipeline under test — for HyperGal: per-metaslice 2D weighted chi-square minimization with iminuit over the SN position using the constrained chromatic Gaussian+Moffat PSF, differential PSF, and 6-parameter background model, followed by the chromatic 1D fit of the PSF/ADR models across wavelength, then the final 3D linear amplitude fit over host galaxy, transient, and background components. Compute the continuum-divided spectral relative RMS (Eq. 14) between the injected input spectrum and the extracted spectrum, averaged over the [4000, 8000] Å domain, and bin results by contrast. Decision point: for c > 0.5 the continuum-normalized spectral relative RMS should fall below ~5%, consistent with the regime where HyperGal reaches the 5% level for 60% of simulated observations.

## Related tools

- **HyperGal** (Scene-modeling pipeline whose extraction accuracy is being validated via injection and relative-RMS recovery) — https://github.com/JeremyLezmy/hypergal
- **iminuit** (Minimizer for the per-metaslice 2D weighted chi-square fit of SN position, PSF, and background parameters during the injection test) — https://github.com/scikit-hep/iminuit
- **SEDm (SEDmachine)** (Integral field spectrograph whose synthetic cubes receive the injected point source and onto which the scene model is projected)
- **CIGALE** (SED fitter used as physically-motivated spectral interpolator to generate the host-galaxy hyperspectral model used as the injection background)

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- Continuum-normalized spectral relative RMS for the c > 0.5 case falls below ~5% (Eq. 14 metric averaged over [4000, 8000] Å)
- Extracted spectrum closely tracks the known injected input spectrum after continuum division across the [4000, 8000] Å domain
- Final 3D fit only adjusts scaling amplitudes of host galaxy, SN, and background polynomial components, with morphology fixed by the earlier 2D and 1D fits
- RMS distribution computed per contrast bin shows monotonic degradation as contrast c decreases
- Injection test reproduced across the validated 5000 simulated cubes population

## Limitations

- The true distance distribution between the SN and its host was not explicitly modeled — it was marginalized uniformly between 0 and 5 arcsec
- HyperGal requires an external redshift measurement of the host galaxy as a priori input
- Interaction of G(lambda) with emission line position/intensity depends on the relative stellar/gaseous distribution and has not been studied extensively
- Only ~50% of the ZTF sample has spectral host redshifts; the rest rely on SNID estimates with coarser precision (~5e-3)

## Evidence

- [methods] Compute the continuum-divided spectral relative RMS (Eq. 14) between the injected input spectrum and the extracted spectrum: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
- [methods] Contrast regime and 5% extraction accuracy: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline validated on simulated cubes: "The full procedure is validated on 5000 simulated cubes."
- [readme] Scene model projection and fitting: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [abstract] CIGALE-based host model generation: "it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
