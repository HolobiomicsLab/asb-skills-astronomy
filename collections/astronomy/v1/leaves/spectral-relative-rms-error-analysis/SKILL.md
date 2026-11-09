---
name: spectral-relative-rms-error-analysis
description: 'Use when validating a simulated SEDm cube extraction where a known input
  spectrum was injected: comparing extracted vs. input spectra to characterize extraction
  error as a function of SN-host contrast.'
when_to_use_negative:
- No ground-truth input spectrum exists (real observations without simulation injection)
  — the metric requires the known injected spectrum
- You need a posteriori host measurements (e.g. redshift or local Halpha flux) from
  the residual host component, which HyperGal's fitting effectively precludes
- Evaluating typing performance rather than extraction accuracy — use SNID classification
  metrics (rlap_min=6, 50% top-10 agreement, FPR) instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3399
tools:
- name: HyperGal
  role: Scene-modeling pipeline that produces the extracted transient spectrum fed
    into the RMS metric
  repo: https://github.com/JeremyLezmy/hypergal
- name: iminuit
  role: Used for the per-metaslice 2D weighted chi-square minimization whose output
    spectrum is evaluated by the RMS metric
  repo: https://github.com/scikit-hep/iminuit
- name: SEDm (SEDmachine)
  role: Integral field spectrograph whose simulated cubes (input vs. extracted spectra)
    are compared by this metric
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:cbc06c4365fb2843436c70e4ade2933521b15c4fc89f9853676b5653c90d3ba2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - HyperGal
  - HyperGal (hypergal)
  - SEDm (SEDmachine)
  - iminuit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Spectral Relative RMS Error Analysis

## Summary

Quantifies the accuracy of a transient spectrum extracted from an SEDm data cube by computing the continuum-divided spectral relative RMS between the simulation input spectrum and the extracted spectrum, averaged over the [4000, 8000] Å domain.

## When to use

Use when validating a simulated SEDm cube extraction where a known input spectrum was injected: comparing extracted vs. input spectra to characterize extraction error as a function of SN-host contrast.

## When NOT to use

- No ground-truth input spectrum exists (real observations without simulation injection) — the metric requires the known injected spectrum
- You need a posteriori host measurements (e.g. redshift or local Halpha flux) from the residual host component, which HyperGal's fitting effectively precludes
- Evaluating typing performance rather than extraction accuracy — use SNID classification metrics (rlap_min=6, 50% top-10 agreement, FPR) instead

## Inputs

- Extracted transient spectrum from a HyperGal 3D linear amplitude fit (host galaxy, SN, background polynomial components)
- Known input spectrum injected into the simulated SEDm cube (synthetic SEDm cube with point source at fixed contrast c)
- Wavelength domain restricted to [4000, 8000] Å

## Outputs

- Continuum-divided spectral relative RMS value per simulated observation (Eq. 14)
- Spectral RMS distribution across contrast bins, e.g. median/percentile RMS vs. SN-host contrast c
- Validation verdict: fraction of extractions below the ~5% accuracy threshold

## How to apply

After HyperGal's progressive scene fitting (per-metaslice 2D weighted chi-square minimization with iminuit, chromatic 1D PSF/ADR fit, and final 3D linear amplitude fit over host, transient, and background components), take the extracted transient spectrum and the known injected input spectrum. Compute the spectral relative RMS between the two, dividing each by its continuum (continuum-divided variant) and averaging over the [4000, 8000] Å domain (Eq. 14 of the paper). Bin or plot the resulting RMS values in contrast bins (continuum divided) to evaluate how extraction accuracy degrades with SN-host contrast c. A successful application for high-contrast cases (c > 0.5) yields continuum-normalized relative RMS below ~5%, consistent with the regime where HyperGal reaches the 5% level for 60% of simulated observations.

## Related tools

- **HyperGal** (Scene-modeling pipeline that produces the extracted transient spectrum fed into the RMS metric) — https://github.com/JeremyLezmy/hypergal
- **iminuit** (Used for the per-metaslice 2D weighted chi-square minimization whose output spectrum is evaluated by the RMS metric) — https://github.com/scikit-hep/iminuit
- **SEDm (SEDmachine)** (Integral field spectrograph whose simulated cubes (input vs. extracted spectra) are compared by this metric)

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh; # then compare extracted vs input spectra over [4000, 8000] Å via continuum-divided relative RMS (Eq. 14)
```

## Evaluation signals

- Continuum-normalized spectral relative RMS for c > 0.5 injections is below ~5%
- RMS values are computed and averaged over the full [4000, 8000] Å domain (Eq. 14 definition)
- RMS distribution binned by contrast shows monotonic degradation with decreasing SN-host contrast
- Fraction of extractions reaching the 5% level across the simulation set is around 60%, consistent with the paper's reported regime
- Comparison against a standard (isolated point-source) extraction shows HyperGal RMS systematically lower in host-contamination regimes

## Limitations

- Requires a known injected input spectrum, so it is only applicable to simulated cubes; ~30% of real observed SNe show severe host contamination and cannot be directly validated this way
- The contrast distribution relies on the ZTF Cosmology SN Ia DR2 paper (Rigault et al. in prep), which is not yet available, complicating direct reproduction of the contrast sampling
- The SN-host distance distribution was marginalized uniformly between 0 and 5 arcsec rather than explicitly modeled, so contrast-conditioned RMS results inherit that simplification
- Metric is sensitive to residual host contamination near the host core where the pysedm-style isolated point-source assumption breaks down

## Evidence

- [methods] Compute the continuum-divided spectral relative RMS (Eq. 14) between the injected input spectrum and the extracted spectrum, averaged over the [4000, 8000] Å domain.: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
- [results] HyperGal reaches the 5% extraction accuracy level for 60% of simulated observations.: [evidence span withheld — anchor status: not_found]
- [results] The final 3D linear fit adjusts only component amplitudes after morphology is fixed.: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [abstract] The full extraction pipeline is validated on a large simulated set.: "The full procedure is validated on 5000 simulated cubes."
- [readme] HyperGal is a chromatic scene modeler using pre-transient images for the host hyperspectral model.: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a"
