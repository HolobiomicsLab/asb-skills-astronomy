---
name: spectral-continuum-normalization
description: Use when an extracted SEDm transient spectrum (from HyperGal scene modeling
  or standard PSF extraction) must be prepared for SNID template typing, or when extraction
  quality is scored via continuum-divided spectral RMS over 4000–8000 Å.
when_to_use_negative:
- When the spectrum feeds host-galaxy hyperspectral modeling (CIGALE SED fitting),
  where the galaxy continuum shape carries the physical information and must not be
  divided out
- When the goal is absolute flux or photometric consistency, since continuum division
  discards the red continuum level and paper RMS metrics are explicitly continuum-divided
- When the extracted spectrum does not span the 4000–8000 Å domain, so no reliable
  5th-order continuum fit can be anchored
edam_operation: http://edamontology.org/operation_3222
tools:
- name: SNID
  role: Consumes the continuum-normalized spectrum for template cross-correlation
    typing, returning type, redshift, phase, and rlap
- name: HyperGal
  role: Produces the extracted transient spectra (chromatic scene model of host +
    point source + background) that this normalization prepares for typing
  repo: https://github.com/JeremyLezmy/hypergal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:6bcaa2c1cc8f4f738ba6b88f2a8f75caf466505420d2f5ab6e7b3111699b9b85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - HyperGal
  - SNID
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spectral-continuum-normalization

## Summary

Removes the low-order continuum shape from an extracted transient spectrum (5th-order polynomial over the 4000–8000 Å typing range) so it can be matched against SNID templates, and yields continuum-divided spectra used to quantify extraction quality via relative RMS.

## When to use

Use when an extracted SEDm transient spectrum (from HyperGal scene modeling or standard PSF extraction) must be prepared for SNID template typing, or when extraction quality is scored via continuum-divided spectral RMS over 4000–8000 Å.

## When NOT to use

- When the spectrum feeds host-galaxy hyperspectral modeling (CIGALE SED fitting), where the galaxy continuum shape carries the physical information and must not be divided out
- When the goal is absolute flux or photometric consistency, since continuum division discards the red continuum level and paper RMS metrics are explicitly continuum-divided
- When the extracted spectrum does not span the 4000–8000 Å domain, so no reliable 5th-order continuum fit can be anchored

## Inputs

- Extracted transient spectrum from the HyperGal hyperspectral scene-modeling fit of an SEDm cube
- Extracted transient spectrum from the standard PSF extraction (pysedm-style) of the same SEDm cube
- Simulation input (injected) SN spectra for relative-RMS comparison
- Wavelength grid covering at least the 4000–8000 Å typing range

## Outputs

- Continuum-normalized (continuum-divided) transient spectrum ready for SNID cross-correlation
- 5th-order polynomial continuum fit coefficients over 4000–8000 Å
- Spectral relative RMS between input and extracted spectra, averaged over [4000, 8000] Å
- Per-spectrum SNID records (type, redshift, phase, rlap) written to snid_classification_results.csv

## How to apply

Fit a 5th-order polynomial to the continuum of the extracted SN spectrum over the 4000–8000 Å typing range — the same domain SNID uses — and divide the spectrum by this fit so the continuum is flattened to unity. Feed the normalized spectrum to SNID (the same classifier used in ZTF), recording the best-fit template's type, redshift, phase, and rlap. Assess extraction fidelity by computing the spectral relative RMS between simulation input and extracted spectra on the continuum-divided data, averaged over [4000, 8000] Å, and bin these RMS distributions by host-transient contrast. The rationale is that SNID cross-correlates against template spectra where continuum shape is not informative for typing, while contrast-binned continuum-divided RMS isolates line-region recovery, which drives typing efficiency (e.g. >96% SN Ia success for 0.2 < c < 0.3).

## Related tools

- **SNID** (Consumes the continuum-normalized spectrum for template cross-correlation typing, returning type, redshift, phase, and rlap)
- **HyperGal** (Produces the extracted transient spectra (chromatic scene model of host + point source + background) that this normalization prepares for typing) — https://github.com/JeremyLezmy/hypergal

## Examples

```
import numpy as np; m=(wave>=4000)&(wave<=8000); c=np.polyfit(wave[m], flux[m], 5); spec_norm=flux/np.polyval(c, wave)
```

## Evaluation signals

- After normalization, the continuum over the 4000–8000 Å fit range is flattened (divided spectrum oscillates around unity) without clipping narrow emission/absorption features
- Every simulated spectrum yields a recorded best-match type/redshift/phase/rlap downstream, with the rlap ≥ 5 high-confidence flag populated
- Continuum-divided relative RMS over [4000, 8000] Å decreases with increasing contrast bin, matching the paper's per-contrast RMS distributions
- Typing success rates and SN Ia false-positive rate computed on normalized spectra reproduce the reported efficiencies (>96% Ia success for 0.2 < c < 0.3; FPR < 2% for c > 0.1)
- HyperGal-normalized spectra show systematically lower continuum-divided RMS and ~10% higher SN Ia typing success than standard-extraction spectra at matched contrast

## Limitations

- Continuum division discards absolute flux and continuum-shape information, so the normalized product cannot support photometric or a posteriori host measurements (e.g. redshift or local Halpha flux)
- The fixed 5th-order fit over a restricted 4000–8000 Å window can absorb very broad SN features or fail if the usable wavelength coverage shrinks at low contrast / near the host core
- Normalization alone does not resolve typing uncertainty: spectra failing rlap criteria (rlap_min = 6 with 50% top-10 model agreement in the paper's stricter setup) must still be flagged 'uncertain'

## Evidence

- [methods] SNID classifier context: "The test on the simulated cubes is per- formed using the same classifier as in ZTF, i.e. SNID"
- [methods] RMS metric definition: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
- [methods] Continuum-divided RMS distributions: "Spectral RMS distribution in contrast bins (continuum divided)"
- [methods] Stricter confidence criteria on normalized SNID output: "The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one"
- [readme] Pipeline context for extraction+classification: "Pipeline for supernovae extraction and classification within the SEDm/ZTF project."
