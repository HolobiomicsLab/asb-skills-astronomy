---
name: supernova-spectral-classification
description: Use when a low-resolution (e.g. SEDm/IFS) extracted transient spectrum
  needs automated SN typing, or when quantifying classification confidence via rlap
  and top-10 template agreement.
when_to_use_negative:
- The spectrum is a host-galaxy spectrum or a residual host component — HyperGal's
  residual cannot be used for a posteriori host measurements such as redshift or local
  Halpha flux
- The spectrum is too contaminated to extract (e.g. an unusable transient near the
  host core); extraction quality (contrast, spectral RMS) must be assessed before
  typing is meaningful
- A redshift prior must be explicit — HyperGal's SNID typing confidence is quoted
  without considering any prior on the redshift or phase of the SN, so this skill
  is not a substitute for host-redshift-based classification
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: SNID
  role: Template-matching spectral classifier run on continuum-normalized extracted
    spectra; returns best-fit type, redshift, phase, and rlap; same classifier as
    used in ZTF
- name: HyperGal
  role: Scene-modeling pipeline that produces the low-contamination transient spectra
    fed to SNID typing
  repo: https://github.com/JeremyLezmy/hypergal
- name: pysedm
  role: Standard extraction pipeline whose output spectra serve as the baseline comparison
    for SNID typing efficiency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:39b4c2f262b9ec710cd0c5baf1e66abda989fbd9d2bb8a0221e640b5047dd42a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - HyperGal
  - SNID
  - pysedm
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# supernova-spectral-classification

## Summary

Classifies extracted transient spectra by supernova type (Ia, Ibc, II) using the SNID template-matching classifier, recording the best-fit type, redshift, phase, and rlap confidence score. HyperGal applies stricter-than-default confidence criteria (rlap ≥ 6 and 50% top-10 model type agreement) to flag uncertain typings.

## When to use

Use when a low-resolution (e.g. SEDm/IFS) extracted transient spectrum needs automated SN typing, or when quantifying classification confidence via rlap and top-10 template agreement.

## When NOT to use

- The spectrum is a host-galaxy spectrum or a residual host component — HyperGal's residual cannot be used for a posteriori host measurements such as redshift or local Halpha flux
- The spectrum is too contaminated to extract (e.g. an unusable transient near the host core); extraction quality (contrast, spectral RMS) must be assessed before typing is meaningful
- A redshift prior must be explicit — HyperGal's SNID typing confidence is quoted without considering any prior on the redshift or phase of the SN, so this skill is not a substitute for host-redshift-based classification

## Inputs

- Extracted SN spectrum from a simulated or observed SEDm data cube (HyperGal or standard PSF extraction)
- Continuum-normalized spectrum over the 4000–8000 Å typing range
- Injected (simulated) template type for verification of recovered classifications

## Outputs

- Best-match SN type, redshift, phase, and rlap per spectrum
- High-confidence flag (rlap >= 5 / rlap_min = 6) and top-10 agreement check per spectrum
- Per-type typing success rates and SN Ia false-positive rate aggregated per contrast bin
- CSV of per-spectrum records, e.g. snid_classification_results.csv

## How to apply

Continuum-normalize the extracted spectrum over the 4000–8000 Å typing range (e.g. with a 5th-order polynomial fit), then run SNID — the same classifier used in ZTF — and record the best-fit template's type, redshift, phase, and rlap. Apply HyperGal's confidence criteria: the best-fit model must have rlap ≥ 6 (rather than the default 5, which the paper still cites as the high-confidence threshold), and at least 50% of the top-10 SNID models must be of the same type as the best fit. If either criterion fails, classify the spectrum as 'uncertain' rather than forcing a type. Compare each recovered type to the injected (or known) template type to compute per-type typing success rates, and probe the False Positive Rate for SN Ia classification (core-collapse SNe typed as Ia). Write per-spectrum records (type, redshift, phase, rlap, verification flag) to a CSV such as snid_classification_results.csv.

## Related tools

- **SNID** (Template-matching spectral classifier run on continuum-normalized extracted spectra; returns best-fit type, redshift, phase, and rlap; same classifier as used in ZTF)
- **HyperGal** (Scene-modeling pipeline that produces the low-contamination transient spectra fed to SNID typing) — https://github.com/JeremyLezmy/hypergal
- **pysedm** (Standard extraction pipeline whose output spectra serve as the baseline comparison for SNID typing efficiency)

## Examples

```
run_hypergal.py -t ZTF20abhrmxh  # then SNID on the extracted spectrum: continuum-normalize over 4000-8000 Å, require rlap_min=6 and >=50% top-10 type agreement, else flag 'uncertain'
```

## Evaluation signals

- Every simulated spectrum has a recorded best-match type, redshift, phase, and rlap
- Each recovered type is checked against the injected template type; per-type agreement is computed for Ia, Ibc, and II
- The rlap >= 5 high-confidence flag is populated for every spectrum, and 'uncertain' is assigned whenever rlap_min = 6 or the 50% top-10 agreement criterion fails
- Typing success rates and the SN Ia false-positive rate (FPR) are comparable to the paper's reported efficiencies (e.g. ~95% Ia success, FPR below 2%, 10% more SNe Ia classified than standard extraction)

## Limitations

- The rlap-based confidence does not consider any prior on the redshift or the phase of the SN
- HyperGal typing depends on an external host-galaxy redshift measurement — 'the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy'
- Typing success rates depend on the SN-host contrast distribution, which was drawn from real SEDm observations and the SN-host distance was marginalized uniformly between 0 and 5 arcsec rather than explicitly modeled

## Evidence

- [methods] Stricter SNID confidence criteria: "The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one"
- [methods] Uncertain classification fallback: "If one of these criteria is not met, the spectrum is classified as “uncertain”."
- [methods] SNID as the ZTF classifier: "The test on the simulated cubes is per- formed using the same classifier as in ZTF, i.e. SNID"
- [methods] FPR metric definition: "To probe the critical contamination of the SN Ia sample by core-collapse SNe, the False Positive Rate (FPR) for SN Ia is examined"
- [abstract] Typing performance headline result: "From estimated contrast distribution of real SEDm observations, we show that HyperGal correctly classifies ~95% of SNe Ia."
