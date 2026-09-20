---
name: false-positive-rate-evaluation
description: Use when evaluating a supernova spectral classifier's contamination of
  the SN Ia sample, e.g. counting core-collapse SNe (II, Ibc) typed as Ia across contrast
  bins for an extraction method like HyperGal.
when_to_use_negative:
- When the goal is extraction fidelity (spectral relative RMS) rather than classification
  contamination — use the spectral RMS metric over [4000,8000] Å instead
- When evaluating SN Ia typing success rate (true positives) rather than core-collapse
  contamination of the Ia sample
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SNID
  role: Spectral classifier run on each extracted spectrum to assign type/redshift/phase/rlap
    used for the FPR computation
  repo: https://github.com/JeremyLezmy/hypergal
- name: HyperGal
  role: Scene-modeling extraction pipeline whose typed spectra are evaluated for the
    false positive rate
  repo: https://github.com/JeremyLezmy/hyper_gal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:b2dfaebcd451561cec6df3a7ea61820e91cbb0d1e3435d1ff6cb815e369a2a64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - HyperGal
  - SNID
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# false-positive-rate-evaluation

## Summary

Quantifies the SN Ia false-positive rate — core-collapse SNe misclassified as type Ia — by running SNID typing on spectra (e.g. HyperGal vs. standard extraction outputs over 5000 simulated SEDm cubes) and comparing recovered types against injected template types.

## When to use

Use when evaluating a supernova spectral classifier's contamination of the SN Ia sample, e.g. counting core-collapse SNe (II, Ibc) typed as Ia across contrast bins for an extraction method like HyperGal.

## When NOT to use

- When the goal is extraction fidelity (spectral relative RMS) rather than classification contamination — use the spectral RMS metric over [4000,8000] Å instead
- When evaluating SN Ia typing success rate (true positives) rather than core-collapse contamination of the Ia sample

## Inputs

- Extracted transient spectra from simulated SEDm cubes (HyperGal and standard-extraction outputs)
- Injected/true SN template types per simulated spectrum (Ia, Ibc, II)
- Host-transient contrast values for binning

## Outputs

- Per-spectrum records (type, redshift, phase, rlap, uncertainty flag, true-vs-recovered type agreement), e.g. snid_classification_results.csv
- False positive rate for SN Ia classification, aggregated per contrast bin

## How to apply

Run SNID on each extracted spectrum (continuum-normalized over the 4000–8000 Å typing range) and record the best-fit type, redshift, phase, and rlap. Apply the paper's confidence criteria — best-fit rlap ≥ 5 (the paper uses a stricter rlap_min = 6) and at least 50% agreement among the top-10 SNID models — flagging spectra failing either as 'uncertain'. For each spectrum, compare the recovered SNID type to the injected template type, then define false positives as core-collapse SNe (types II or Ibc) classified as SN Ia. Aggregate per contrast bin (c) and compute the FPR; in HyperGal's validation on 5000 simulated cubes the FPR is below 2% for c > 0.1, half that of the standard extraction method.

## Related tools

- **SNID** (Spectral classifier run on each extracted spectrum to assign type/redshift/phase/rlap used for the FPR computation) — https://github.com/JeremyLezmy/hypergal
- **HyperGal** (Scene-modeling extraction pipeline whose typed spectra are evaluated for the false positive rate) — https://github.com/JeremyLezmy/hyper_gal

## Evaluation signals

- Every simulated spectrum has a recorded best-match type/redshift/phase/rlap and a populated rlap ≥ 5 high-confidence flag
- Each recovered SNID type is checked against the injected template type; core-collapse SNe typed as Ia are counted as false positives
- FPR for SN Ia is below 2% for contrast c > 0.1 (matching the paper's reported HyperGal result)
- Spectra failing the confidence criteria (rlap < threshold or <50% top-10 model type agreement) are labeled 'uncertain', not silently counted
- Typing success rates per contrast bin (e.g. >96% Ia success for 0.2 < c < 0.3) are comparable to the paper's reported efficiencies

## Limitations

- SN-host distance distribution was not explicitly modeled (marginalized uniformly between 0 and 5 arcsec), so contrast-bin FPRs carry that simplification
- HyperGal requires an external host-galaxy redshift measurement as a priori input, which affects which spectra can be typed at all
- Results depend on the assumed contrast distribution of real SEDm observations (ZTF Cosmology DR2, in prep), which is not yet published

## Evidence

- [methods] To probe the critical contamination of the SN Ia sample by core-collapse SNe, the False Positive Rate (FPR) for SN Ia is examined: "To probe the critical contamination of the SN Ia sample by core-collapse SNe, the False Positive Rate (FPR) for SN Ia is examined"
- [methods] The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one: "The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one"
- [methods] If one of these criteria is not met, the spectrum is classified as “uncertain”.: "If one of these criteria is not met, the spectrum is classified as “uncertain”."
- [abstract] The false positive rate is less than 2%, half as much as the standard extraction method.: "The false positive rate is less than 2%, half as much as the standard extraction method."
- [abstract] The full procedure is validated on 5000 simulated cubes.: "The full procedure is validated on 5000 simulated cubes."
