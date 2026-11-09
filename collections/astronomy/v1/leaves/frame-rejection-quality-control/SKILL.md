---
name: frame-rejection-quality-control
description: Use when loading S2D spectrograph frames into the SBART DataClass before
  building stellar templates, and you need to discard low-SNR orders, frames with
  warning keywords, or flux outliers.
when_to_use_negative:
- Input is already a CCF RV time-series from the ESO official pipeline rather than
  raw S2D frames needing frame-level screening
- Goal is analyzing planetary or activity signals from an existing RV time-series,
  which the paper leaves as future work and does not involve frame rejection
- Frames lack header keywords or SNR information, making the warning-KW and order-wise
  SNR conditions inoperative
tools:
- name: SBART
  role: Pipeline whose DataClass ingests S2D frames and applies the frame-rejection
    conditions (order-wise SNR check, warning KW flags, MAD flux-outlier flagging)
    before template construction
  repo: github.com/iastro-pt/sBART
- name: ESPRESSO
  role: Instrument whose GTO S2D frames are the data being screened
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:99db45eb5bce5bdce20c6e7b247052c14fb018986886c411ba251dd4dd6b650a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ESO official pipeline
  - ESPRESSO
  - S-BART
  - SBART
  - Telfit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# frame-rejection-quality-control

## Summary

Pre-template frame filtering for S-BART RV extraction: ESPRESSO/HARPS S2D frames are rejected before stellar template construction using order-wise SNR checks, pipeline warning-keyword flags, and MAD-based flux-outlier detection, with optional sigma-clipping of RVs against the DRS estimate.

## When to use

Use when loading S2D spectrograph frames into the SBART DataClass before building stellar templates, and you need to discard low-SNR orders, frames with warning keywords, or flux outliers.

## When NOT to use

- Input is already a CCF RV time-series from the ESO official pipeline rather than raw S2D frames needing frame-level screening
- Goal is analyzing planetary or activity signals from an existing RV time-series, which the paper leaves as future work and does not involve frame rejection
- Frames lack header keywords or SNR information, making the warning-KW and order-wise SNR conditions inoperative

## Inputs

- ESPRESSO or HARPS S2D spectral frames (per-order spectra) for a target (e.g., 33 ESPRESSO GTO targets)
- Pipeline header keywords (KW flags) carried per frame
- Order-wise SNR values per frame/order
- DRS RV estimates used for sigma-clipping

## Outputs

- Filtered set of accepted frames (with frameIDs) passed to stellar template construction
- Rejected-frame records with rejection reason (SNR, warning keyword, MAD flux outlier, sigma-clip vs DRS RV)

## How to apply

Load the ESPRESSO GTO S2D frames (e.g., the 33 GTO targets) into the SBART DataClass and enable the rejection conditions so that each frame is screened before template construction: (1) an order-wise SNR condition that rejects frames/orders failing the SNR check, (2) rejection of frames carrying warning KW flags from the pipeline headers, and (3) MAD-based flagging of flux outliers. Optionally add sigma-clipping of RVs based on the DRS estimate for greater control in the selection of available frames. The rationale is that the stellar template is built from all retained observations, so any frame with corrupted orders or anomalous flux would bias the common-RV-shift fit against each template; the changelog shows these conditions were added explicitly as frame-selection controls. The NORMALIZATION_MODE setting governs how retained stellar spectra are normalized, so it must be set consistently across frames.

## Related tools

- **SBART** (Pipeline whose DataClass ingests S2D frames and applies the frame-rejection conditions (order-wise SNR check, warning KW flags, MAD flux-outlier flagging) before template construction) — github.com/iastro-pt/sBART
- **ESPRESSO** (Instrument whose GTO S2D frames are the data being screened)

## Evaluation signals

- Accepted-frame set should exclude every frame flagged by the warning KW conditions, the order-wise SNR check, and MAD flux-outlier detection
- After rejection, the stellar template should be built from the remaining observations and the RV fit should converge via the Laplace-approximation sampler
- Downstream check: S-BART M-type RVs should reach a median uncertainty of ~15 cm/s over 309 observations of 16 targets
- Sigma-clipping against the DRS estimate should remove RV outliers relative to the ESO CCF pipeline values
- Comparison against ESO CCF RVs should reproduce the ~10% (M-type) and ~4% (K-type) median RV scatter decrease

## Limitations

- Rejection relies on warning KW flags existing in the frame headers; frames without such keywords cannot be screened this way
- Telfit telluric templates have a fallback for reference frames for which GDAS profiles cannot be downloaded, so telluric handling after frame acceptance may degrade
- The changelog provides these rejection conditions as release notes without detailed thresholds; exact SNR cutoffs must be set by the user
- Full analysis of planetary and activity signals in the datasets was deferred to future work, so QC validation is limited to RV scatter/uncertainty metrics

## Evidence

- [other] frame-rejection conditions: [evidence span withheld — anchor status: not_found]
- [methods] warning KW flags: "Allow to reject Frames based on warning KW flags that might exist"
- [methods] order-wise SNR: "Added a new condition to check order-wise SNR"
- [methods] MAD flux outliers: "Allow to use MAD to flag flux outliers."
- [methods] sigma-clipping vs DRS: "Sigma-clipping RVs based on the DRS estimate"
