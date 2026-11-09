---
name: cross-code-simulation-comparison
description: Use when two simulation codes' power spectra are available as |code_A/code_B
  − 1| versus k curves and you need to locate the wavenumber where they diverge beyond
  a threshold (e.g. 1‰) and determine whether that scale is resolution-dependent.
when_to_use_negative:
- You need absolute power spectrum amplitudes or cosmological parameter fits rather
  than code-to-code relative agreement
- Only a single box size is available — the resolution-independence test requires
  comparing k_div × L_box / (24 × 2π) across at least two L_box values
- The comparison involves codes other than CONCEPT and GADGET without deposited |code_A/code_B
  − 1| curves in the same format
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: CONCEPT 1.0
  role: Cosmological N-body code whose power spectrum is compared against GADGET-2
    in the relative power spectrum analysis
  repo: github.com/jmd-dk/concept
- name: GADGET-2
  role: Reference N-body code used as the baseline for the cross-code power spectrum
    comparison
- name: NumPy
  role: Used to load the deposited relative power spectra and scan/interpolate the
    |CONCEPT/GADGET-2 − 1| curves for k_div
  repo: github.com/numpy/numpy
- name: Python
  role: Language in which the k_div extraction and tabulation workflow is implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:2737bd6721eb6e56571d2c4371247980852708f1226c8371b3cd210a58232efc
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CONCEPT 1.0
  - GADGET-2
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-code-simulation-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantify agreement between CONCEPT 1.0 and GADGET-2 cosmological N-body simulations by analyzing their relative power spectrum |CONCEPT/GADGET-2 − 1| versus wavenumber k, extracting the divergence wavenumber k_div where deviation first exceeds a 1‰ (0.001) threshold and testing whether it scales with simulation resolution (k_div ≈ 24 × 2π/L_box) rather than an absolute scale.

## When to use

Use when two simulation codes' power spectra are available as |code_A/code_B − 1| versus k curves and you need to locate the wavenumber where they diverge beyond a threshold (e.g. 1‰) and determine whether that scale is resolution-dependent.

## When NOT to use

- You need absolute power spectrum amplitudes or cosmological parameter fits rather than code-to-code relative agreement
- Only a single box size is available — the resolution-independence test requires comparing k_div × L_box / (24 × 2π) across at least two L_box values
- The comparison involves codes other than CONCEPT and GADGET without deposited |code_A/code_B − 1| curves in the same format

## Inputs

- Relative power spectra |CONCEPT/GADGET-2 − 1| as a function of wavenumber k, one curve per box size (2048, 1024, 512, 256 Mpc/h), from the github.com/Aarhus Cosmology/concept1.0-data deposit
- Box sizes L_box for the four simulations
- Deviation threshold (0.001, i.e. 1 per mille)

## Outputs

- k_div_results.csv tabulating per box size: measured k_div, predicted cutoff 24 × 2π/L_box, and relative deviation
- Dimensionless ratios k_div × L_box / (24 × 2π) for the four box sizes
- Statement of whether k_div scales with simulation resolution or with an absolute scale

## How to apply

Load the deposited relative power spectra for each simulated box size (2048, 1024, 512, 256 Mpc/h) from the concept1.0-data GitHub deposit using NumPy. For each box size, scan the |CONCEPT/GADGET-2 − 1| curve and record k_div, the first wavenumber where the deviation exceeds the 0.001 (per mille) threshold, interpolating linearly between sampled k values. For each box, compute the predicted cutoff 24 × 2π/L_box and the dimensionless ratio k_div × L_box / (24 × 2π) to test whether divergence tracks the grid resolution rather than an absolute scale. Tabulate k_div, the predicted 24 × 2π/L_box, and the relative deviation for the four boxes and write k_div_results.csv. Rationale: the paper reports agreement well below per mille at large/intermediate scales and ~1% at k ~ 13 h/Mpc, so the per mille threshold is a sensitive criterion for locating code-divergence onset, and the resolution-scaled cutoff follows from N-body grid artifacts rather than physics.

## Related tools

- **CONCEPT 1.0** (Cosmological N-body code whose power spectrum is compared against GADGET-2 in the relative power spectrum analysis) — github.com/jmd-dk/concept
- **GADGET-2** (Reference N-body code used as the baseline for the cross-code power spectrum comparison)
- **NumPy** (Used to load the deposited relative power spectra and scan/interpolate the |CONCEPT/GADGET-2 − 1| curves for k_div) — github.com/numpy/numpy
- **Python** (Language in which the k_div extraction and tabulation workflow is implemented)

## Examples

```
import numpy as np; k, dev = np.loadtxt('relative_power_spectrum_2048Mpc.h.txt', unpack=True); k_div = np.interp(0.001, dev, k); print(k_div, 24*2*np.pi/2048)
```

## Evaluation signals

- For each of the four box sizes, the dimensionless ratio k_div × L_box / (24 × 2π) should be close to unity within the per mille–percent accuracy at which the paper reports code agreement
- All four box sizes (2048, 1024, 512, 256 Mpc/h) must yield k_div consistent with the same resolution-scaled cutoff ≈ 24 × 2π/L_box
- k_div results must reproduce the reported behavior: agreement well below per mille at large/intermediate scales and ~1% at k ~ 13 h/Mpc
- k_div_results.csv must contain, per box size, the measured k_div, the predicted 24 × 2π/L_box, and the relative deviation with no missing box sizes

## Limitations

- The per mille threshold marks onset of divergence; beyond it the codes can differ by ~1% at the smallest scales probed (k ~ 13 h/Mpc), so k_div alone does not characterize the magnitude of divergence at high k
- CONCEPT 1.0 was found several times faster than GADGET-2 only for low/medium resolutions (k_Nyquist ≲ 5 Mpc/h), so re-running simulations to extend the comparison is expensive at high resolution
- The conclusion k_div ≈ 24 × 2π/L_box is grounded in these four box sizes and these two codes; extrapolation to other codes or resolutions assumes the same grid-artifact origin

## Evidence

- [abstract] simulation parameter files for recreating the simulations producing this data — are made publicly available: [evidence span withheld — anchor status: not_found]
- [abstract] For the code comparisons, we use the well-known gadget-2 code (Springel 2005b) as well as its newer incarnation gadget-4: [evidence span withheld — anchor status: not_found]
