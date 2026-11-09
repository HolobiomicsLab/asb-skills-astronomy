---
name: power-spectrum-estimation-from-particle-snapshots
description: Use when you have particle snapshots from cosmological N-body runs (e.g.
  512 Mpc/h box, N=1024^3) and need a matter power spectrum on specified k-bins, or
  a cross-code P ratio (e.g. P_CONCEPT/P_GADGET) up to k ~ 13 h/Mpc.
when_to_use_negative:
- The goal is linear-theory validation rather than snapshot analysis — use CLASS (or
  CONCEPT's built-in linear comparison) instead of estimating P(k) from particles.
- The simulation is strongly clustered and high-resolution — the article reports CONCEPT
  is slower than GADGET for higher-resolution runs (k_Nyquist ≳ 5 Mpc/h), so pipeline
  performance assumptions change.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: CONCEPT 1.0
  role: simulation code whose FFTW-based pipeline measures the matter power spectrum
    from particle snapshots; also the reference code in the ratio
  repo: github.com/jmd-dk/concept
- name: GADGET-4
  role: comparison code whose TreePM (orders 2, 5) and FMM-PM (order 5, ± randomised
    displacements) runs supply spectra for the relative comparison
  repo: github.com/jmd-dk/concept
- name: GADGET-2
  role: comparison code for the Figure 5 cross-code P(k) agreement test
  repo: github.com/jmd-dk/concept
- name: FFTW
  role: underlying MPI-parallelised real 3D in-place FFT library used by CONCEPT's
    power-spectrum measurement
- name: NumPy
  role: post-processing of measured spectra and formation of ratio tables
  repo: github.com/numpy/numpy
- name: class
  role: linear Einstein-Boltzmann reference against which the measured linear spectrum
    is designed to agree
  repo: github.com/lesgourg/class_public
- name: OpenMPI
  role: MPI framework used to launch CONCEPT and run the parallel measurement
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:abe8220f2d2843934a9f4e8c1ca7043e770542966090caf7497b2da1571ff2f8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CONCEPT 1.0
  - FFTW
  - GADGET-2
  - GADGET-4
  - NumPy
  - OpenMPI
  - Python
  - class
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# power-spectrum-estimation-from-particle-snapshots

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Estimate the matter power spectrum P(k) from N-body particle snapshots (CONCEPT 1.0 or GADGET-2/4 outputs) using CONCEPT's FFTW-based measurement pipeline with NumPy post-processing, then form cross-code ratios on identical k-bins to quantify code agreement.

## When to use

Use when you have particle snapshots from cosmological N-body runs (e.g. 512 Mpc/h box, N=1024^3) and need a matter power spectrum on specified k-bins, or a cross-code P ratio (e.g. P_CONCEPT/P_GADGET) up to k ~ 13 h/Mpc.

## When NOT to use

- The goal is linear-theory validation rather than snapshot analysis — use CLASS (or CONCEPT's built-in linear comparison) instead of estimating P(k) from particles.
- The simulation is strongly clustered and high-resolution — the article reports CONCEPT is slower than GADGET for higher-resolution runs (k_Nyquist ≳ 5 Mpc/h), so pipeline performance assumptions change.

## Inputs

- particle snapshots from CONCEPT 1.0 runs (e.g. 2048/1024/512/256 Mpc/h boxes, N=1024^3, launched via OpenMPI)
- particle snapshots from GADGET-2 / GADGET-4 runs (TreePM multipole orders 2 and 5; FMM-PM order 5 with and without randomised displacements)
- matched simulation parameter files from the concept1.0-data repository
- identical k-bin definitions shared across all codes being compared

## Outputs

- per-run matter power spectra P(k) tabulated versus wavenumber on identical k-bins (probing up to k ~ 13 h/Mpc in the 256 Mpc/h box)
- cross-code ratio spectra, e.g. relative_spectra_cond_concept_vs_gadget4.csv or power_spectrum_ratios.csv, with maximum deviation per scale regime

## How to apply

Run the CONCEPT and GADGET simulations on identical initial conditions using the deposited parameter files from github.com/Aarhus Cosmology/concept1.0-data, then estimate each run's matter power spectrum from the particle snapshots using CONCEPT's FFTW-based measurement pipeline (MPI-parallelised, real, 3D, in-place transformations), with NumPy for post-processing. Bin both codes' spectra on identical k-bins so the ratio P_concept/P_gadget is well-defined pointwise; for the 256 Mpc/h box the smallest probed scale is k ~ 13 h/Mpc (the box's Nyquist scale). Evaluate the ratio per scale regime: acceptance is well below the per mille level at large and intermediate scales and of order 1% at the smallest probed scales. For gadget-4 order-5 FMM-PM runs, check specifically at z = 10 for a several-per-mille power drop relative to CONCEPT, and re-run with randomised displacements enabled to remove it while leaving z = 0 behaviour practically unchanged. As an early-time sanity check, the measured linear power spectrum is designed to agree with the Einstein-Boltzmann code class.

## Related tools

- **CONCEPT 1.0** (simulation code whose FFTW-based pipeline measures the matter power spectrum from particle snapshots; also the reference code in the ratio) — github.com/jmd-dk/concept
- **GADGET-4** (comparison code whose TreePM (orders 2, 5) and FMM-PM (order 5, ± randomised displacements) runs supply spectra for the relative comparison) — github.com/jmd-dk/concept
- **GADGET-2** (comparison code for the Figure 5 cross-code P(k) agreement test) — github.com/jmd-dk/concept
- **FFTW** (underlying MPI-parallelised real 3D in-place FFT library used by CONCEPT's power-spectrum measurement)
- **NumPy** (post-processing of measured spectra and formation of ratio tables) — github.com/numpy/numpy
- **class** (linear Einstein-Boltzmann reference against which the measured linear spectrum is designed to agree) — github.com/lesgourg/class_public
- **OpenMPI** (MPI framework used to launch CONCEPT and run the parallel measurement)

## Examples

```
python -c "import numpy as np; np.savetxt('relative_spectra_cond_concept_vs_gadget4.csv', np.c_[k, P_concept/P_gadget4], header='k,P_concept/P_gadget4')"
```

## Evaluation signals

- Cross-code ratio P_CONCEPT/P_GADGET is well below the per mille level at large and intermediate scales (matching Tables 1–3 / Figures 5–6).
- Deviation at the smallest probed scale (k ~ 13 h/Mpc, Nyquist of the 256 Mpc/h box) is of the order of 1%.
- Both codes' spectra are computed on identical k-bins before ratioing — no interpolation-induced artefacts in the ratio.
- For gadget-4 order-5 FMM-PM at z = 10, the several-per-mille power drop versus CONCEPT disappears when randomised displacements are enabled, while z = 0 ratios stay practically the same.
- The measured linear power spectrum at early times agrees with a class-generated linear spectrum.

## Limitations

- CONCEPT uses the standard slab-decomposition of FFTW, which limits the number of CPU cores participating in the long-range force computation and thus the achievable scale of the measurement pipeline.
- GADGET format snapshot output is described as limited in CONCEPT, which constrains interoperable snapshot handling.
- Agreement at the smallest probed scales degrades to ~1% (near the Nyquist scale of the smallest box), so per-mille claims only hold at large/intermediate k.
- CONCEPT performance for power-spectrum work depends on clustering state: several times faster than GADGET-2 for k_Nyquist < 5 Mpc/h but slower at higher resolutions.

## Evidence

- [abstract] FFTW pipeline detail: [evidence span withheld — anchor status: not_found]
- [other] cross-code ratio method: [evidence span withheld — anchor status: not_found]
- [abstract] slab-decomposition limit: [evidence span withheld — anchor status: not_found]
