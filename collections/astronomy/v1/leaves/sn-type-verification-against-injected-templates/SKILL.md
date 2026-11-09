---
name: sn-type-verification-against-injected-templates
description: Use when validating an automated spectral classification pipeline on
  simulated data where the true SN type (Ia, Ibc, II) is known per spectrum — e.g.
  computing typing efficiency per contrast bin and the core-collapse-as-Ia false-positive
  rate.
when_to_use_negative:
- Spectra from real observations where no injected template type exists — this skill
  is a simulation-based ground-truth comparison, not a standalone typing tool
- 'Redshift or phase estimation tasks: rlap-based acceptance here intentionally does
  not consider any prior on the redshift or phase of the SN'
- Classification of host-galaxy spectra or residual host components, which the HyperGal
  residuals preclude using for a posteriori measurements
tools:
- name: SNID
  role: Spectral classifier applied to each normalized extracted spectrum; produces
    best-fit type, redshift, phase, and rlap confidence score used for verification
- name: HyperGal
  role: Source of the extracted transient spectra being typed and verified against
    injected templates
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:828859db29fff0367350723d722bd4ff9fd12ad8103a50b28eda3841f0212bb8
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# SN type verification against injected templates

## Summary

Verification procedure for SNID spectral typing of extracted transient spectra: each spectrum's recovered best-fit type (with rlap confidence score) is compared to the template type injected into simulated SEDm cubes, yielding per-type typing success rates and a SN Ia false-positive rate for the HyperGal extraction pipeline.

## When to use

Use when validating an automated spectral classification pipeline on simulated data where the true SN type (Ia, Ibc, II) is known per spectrum — e.g. computing typing efficiency per contrast bin and the core-collapse-as-Ia false-positive rate.

## When NOT to use

- Spectra from real observations where no injected template type exists — this skill is a simulation-based ground-truth comparison, not a standalone typing tool
- Redshift or phase estimation tasks: rlap-based acceptance here intentionally does not consider any prior on the redshift or phase of the SN
- Classification of host-galaxy spectra or residual host components, which the HyperGal residuals preclude using for a posteriori measurements

## Inputs

- Extracted transient spectra from simulated SEDm cubes (HyperGal and standard-extraction outputs)
- Injected template SN types per simulated spectrum (ground truth: Ia, Ibc, II)
- Contrast bin assignment per simulated spectrum

## Outputs

- Per-spectrum SNID records (type, redshift, phase, rlap, verification flag) written to snid_classification_results.csv
- Per-type typing success rates aggregated per contrast bin
- SN Ia false-positive rate (core-collapse SNe classified as Ia)
- 'uncertain' flags for spectra failing the rlap/top-10 agreement criteria

## How to apply

Run SNID (the same classifier used in ZTF) on each extracted spectrum from the simulated cubes, recording the best-fit template's type, redshift, phase, and rlap. Continuum-normalize spectra over the 4000–8000 Å typing range before classification. Apply the paper's confidence criteria — minimum rlap on the best-fit model and agreement among the top-10 SNID models; the paper sets rlap_min = 6 (stricter than the default rlap ≥ 5) and requires ≥ 50% of the top-10 models to share the best-fit type, flagging failures as 'uncertain'. Compare each recovered type to the injected template type per spectrum and per SN type, aggregate per contrast bin, and compute typing success rates plus the SN Ia false-positive rate (core-collapse SNe classified as Ia). Validation succeeds only if every spectrum has a recorded type/redshift/phase/rlap and the rlap high-confidence flag is populated.

## Related tools

- **SNID** (Spectral classifier applied to each normalized extracted spectrum; produces best-fit type, redshift, phase, and rlap confidence score used for verification)
- **HyperGal** (Source of the extracted transient spectra being typed and verified against injected templates) — https://github.com/JeremyLezmy/hypergal

## Evaluation signals

- Every simulated spectrum has a recorded best-match type, redshift, phase, and rlap in snid_classification_results.csv
- The rlap ≥ 5 high-confidence flag is populated for each spectrum (with the paper's stricter rlap_min = 6 applied to the best-fit model)
- Each recovered type matches the injected template type check per spectrum and per SN type (Ia, Ibc, II)
- Typing success rates comparable to the paper's reported per-type efficiencies, e.g. >96% Ia success for 0.2 < c < 0.3
- SN Ia false-positive rate below 2% for c > 0.1, consistent with the paper's reported FPR

## Limitations

- Stricter criteria (rlap_min = 6 rather than 5, plus 50% top-10 model agreement) mark spectra 'uncertain', reducing classified counts relative to default SNID usage
- The true SN-host distance distribution is not explicitly modeled (marginalized uniformly between 0 and 5 arcsec), which affects contrast-bin aggregation fidelity
- HyperGal requires an external host-galaxy redshift measurement as a priori input; the remaining 50% of ZTF SNe rely on coarser SNID redshift estimates (~5e-3 precision)
- Typing performance is evaluated under an assumed contrast distribution; core-collapse contrast distributions are only assumed similar, and the ZTF DR2 contrast-distribution source (Rigault et al.) was in preparation

## Evidence

- [methods] The test on the simulated cubes is per- formed using the same classifier as in ZTF, i.e. SNID: "The test on the simulated cubes is per- formed using the same classifier as in ZTF, i.e. SNID"
- [methods] rlap criteria: "The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one"
- [methods] uncertain flagging: "If one of these criteria is not met, the spectrum is classified as “uncertain”."
- [methods] FPR examination: "To probe the critical contamination of the SN Ia sample by core-collapse SNe, the False Positive Rate (FPR) for SN Ia is examined"
- [abstract] validation scope: "The full procedure is validated on 5000 simulated cubes."
