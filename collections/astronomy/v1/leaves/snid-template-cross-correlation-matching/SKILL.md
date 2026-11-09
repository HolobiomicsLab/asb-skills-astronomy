---
name: snid-template-cross-correlation-matching
description: Use when a extracted 1D supernova spectrum (e.g. from SEDm/IFS data)
  must be spectroscopically typed (Ia, Ibc, II) with redshift and phase estimates,
  and a quantitative confidence measure such as rlap is needed to flag uncertain classifications.
when_to_use_negative:
- The input is imaging or photometry rather than a reduced 1D spectrum — SNID cross-correlation
  requires a wavelength-calibrated spectrum
- The spectrum is heavily host-contaminated and has not been scene-modeled (e.g. raw
  pysedm extraction near the host core), since host contamination can drive an incorrect
  typing
- A precise host redshift is required as output — SNID redshift precision (~5e-3 from
  preliminary SN spectrum extraction) is coarser than dedicated spectral host-redshift
  measurements
tools:
- name: SNID
  role: Template cross-correlation classifier that returns best-fit type, redshift,
    phase, and rlap for each normalized transient spectrum; same classifier as used
    in ZTF
- name: HyperGal
  role: Scene modeler producing the host-decontaminated extracted SN spectra that
    are subsequently typed by SNID
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:2ae3ac6d41aedf01115f242fd8c8348945660fd8a13cc34b0a14de6328679f46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - HyperGal
  - SNID
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# snid-template-cross-correlation-matching

## Summary

Classifies transient spectra by running SNID template cross-correlation over a continuum-normalized spectrum (4000-8000 Å) and scoring the best-fit template by its rlap parameter, treating rlap ≥ 5 (or the stricter rlap_min = 6 with ≥50% top-10 model agreement used in HyperGal) as a high-confidence typing.

## When to use

Use when a extracted 1D supernova spectrum (e.g. from SEDm/IFS data) must be spectroscopically typed (Ia, Ibc, II) with redshift and phase estimates, and a quantitative confidence measure such as rlap is needed to flag uncertain classifications.

## When NOT to use

- The input is imaging or photometry rather than a reduced 1D spectrum — SNID cross-correlation requires a wavelength-calibrated spectrum
- The spectrum is heavily host-contaminated and has not been scene-modeled (e.g. raw pysedm extraction near the host core), since host contamination can drive an incorrect typing
- A precise host redshift is required as output — SNID redshift precision (~5e-3 from preliminary SN spectrum extraction) is coarser than dedicated spectral host-redshift measurements

## Inputs

- Extracted 1D transient spectrum from SEDm IFU cubes (HyperGal scene-modeling or standard PSF extraction output)
- SNID spectral template library
- Typing wavelength range 4000-8000 Å

## Outputs

- Per-spectrum best-fit SN type, redshift, phase, and rlap value
- High-confidence flag (rlap ≥ 5, or rlap ≥ 6 plus ≥50% top-10 agreement)
- 'Uncertain' label for spectra failing the confidence criteria
- Typing success rates per SN type (Ia, Ibc, II) per contrast bin and SN Ia false-positive rate (e.g. snid_classification_results.csv)

## How to apply

Continuum-normalize the extracted spectrum (e.g. 5th-order polynomial over the 4000-8000 Å typing range) and run SNID — the same classifier used in ZTF — recording the best-fit template's type, redshift, phase, and rlap. Quantify classification confidence with the rlap parameter: rlap ≥ 5 indicates high confidence without any prior on redshift or phase. HyperGal's validation on 5000 simulated cubes applies stricter criteria: minimum rlap_min = 6 for the best-fit model, plus at least 50% of the top-10 models sharing the best model's type; if any criterion fails, the spectrum is labeled 'uncertain'. Compare each recovered type to the injected/expected template type, then aggregate typing success rates per contrast bin and the SN Ia false-positive rate (core-collapse SNe typed as Ia).

## Related tools

- **SNID** (Template cross-correlation classifier that returns best-fit type, redshift, phase, and rlap for each normalized transient spectrum; same classifier as used in ZTF)
- **HyperGal** (Scene modeler producing the host-decontaminated extracted SN spectra that are subsequently typed by SNID) — https://github.com/JeremyLezmy/hypergal

## Evaluation signals

- Every simulated spectrum yields a recorded best-match type, redshift, phase, and rlap
- The rlap ≥ 5 high-confidence flag (or stricter rlap_min = 6 with ≥50% top-10 agreement) is populated for each spectrum
- Recovered SN types match the injected template types; per-type success rates reproduce paper values (e.g. ~95% SNe Ia; >96% for contrast 0.2 < c < 0.3)
- SN Ia false-positive rate (core-collapse SNe classified as Ia) stays below 2% for contrast c > 0.1
- Spectra failing the rlap or top-10 agreement criteria are consistently labeled 'uncertain'

## Limitations

- In HyperGal's validation the true SN-host distance distribution was not explicitly modeled but marginalized uniformly between 0 and 5 arcsec
- The paper's stricter criteria (rlap_min = 6, 50% top-10 agreement) differ from the generic rlap ≥ 5 threshold; results are threshold-dependent
- Confidence is assessed without any prior on redshift or phase of the SN, which can inflate uncertainty for degenerate cases
- Typing efficiency is contrast-dependent: near the host core, host contamination can lead to an incorrect typing and bias cosmological analyses

## Evidence

- [methods] The test on the simulated cubes is performed using the same classifier as in ZTF, i.e. SNID: "The test on the simulated cubes is per- formed using the same classifier as in ZTF, i.e. SNID"
- [methods] Stricter SNID confidence criteria in HyperGal: rlap_min = 6 and 50% top-10 model agreement: "The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one"
- [methods] Uncertain classification when criteria are not met: "If one of these criteria is not met, the spectrum is classified as “uncertain”."
- [abstract] HyperGal correctly classifies ~95% of SNe Ia and FPR below 2%: "From estimated contrast distribution of real SEDm observations, we show that HyperGal correctly classifies ~95% of SNe Ia."
- [readme] HyperGal is the pipeline producing the extracted spectra typed by SNID in the SEDm/ZTF context: "Pipeline for supernovae extraction and classification within the SEDm/ZTF project. Please have a look at [the _HyperGal_ paper](https://arxiv.org/abs/2209.10882)."
