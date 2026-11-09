---
name: binary-black-hole-mass-ratio-sampling
description: 'Use when validating a gravitational-wave surrogate model over a parameter
  range: you need the magnitude of surrogate error (Eq. 31) for randomly drawn q in
  [1,2] and the q where the largest error occurs.'
when_to_use_negative:
- The task is to generate a precessing-spin waveform rather than quantify error over
  mass ratio — use the precessing models (NRSur7dq4, SEOBNRv4PHMSur) instead of this
  nonspinning error sweep
- You need a waveform for one specific q rather than an error profile over a random
  sample of q — a single surrogate evaluation suffices
- Mass ratios outside [1,2] or beyond a model's training domain (e.g. q>8 for NRHybSur3dq8)
  are required — the surrogate is not trained there
tools:
- name: gwsurrogate
  role: Downloads catalog surrogate data (gwsurrogate.catalog.pull), loads the model
    (gwsurrogate.LoadSurrogate), and evaluates the surrogate at sampled (q, chiA,
    chiB) to return time array and waveform modes
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: NumPy
  role: Draws the 1,000 uniformly random mass ratios from [1,2] and provides the C-API
    headers gwsurrogate's C-extensions need at build time
- name: gwtools
  role: Computes pointwise amplitude and phase differences of surrogate waveforms
    against the reference EOB family; required dependency of gwsurrogate
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: pytest
  role: Runs the package's regression tests to cross-check stored waveform outputs
    of the validation
  repo: https://github.com/sxs-collaboration/gwsurrogate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:277948db8f8585195dd1f5d42f415c12a49b66a82f1a256247ec16351c9321a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - NumPy
  - Python
  - gwsurrogate
  - gwtools
  - pytest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# binary-black-hole-mass-ratio-sampling

## Summary

Randomly samples mass ratios q in [1,2] and evaluates a loaded gwsurrogate surrogate model (e.g. the EOB test case) at each sample to quantify surrogate error (Eq. 31) and locate the worst-case mass ratio, as done for the Fig. 9 validation.

## When to use

Use when validating a gravitational-wave surrogate model over a parameter range: you need the magnitude of surrogate error (Eq. 31) for randomly drawn q in [1,2] and the q where the largest error occurs.

## When NOT to use

- The task is to generate a precessing-spin waveform rather than quantify error over mass ratio — use the precessing models (NRSur7dq4, SEOBNRv4PHMSur) instead of this nonspinning error sweep
- You need a waveform for one specific q rather than an error profile over a random sample of q — a single surrogate evaluation suffices
- Mass ratios outside [1,2] or beyond a model's training domain (e.g. q>8 for NRHybSur3dq8) are required — the surrogate is not trained there

## Inputs

- A gwsurrogate surrogate model (e.g. the EOB-LAL surrogate from PhysRevX.4.031006) with downloaded catalog data
- Mass-ratio sampling range q in [1,2], sample size 1,000, zero spins (nonspinning fiducial family)
- Waveform settings: L=16,384 time samples, dt sized to cover the full waveform, f_low=0

## Outputs

- Per-sample surrogate error values (Eq. 31) for the 1,000 drawn q values
- Pointwise amplitude and phase difference series per parameter point (via gwtools)
- Aggregated table surrogate_validation_results.csv with max error, max amplitude/phase error, and the q of largest error

## How to apply

Pull the surrogate data with gwsurrogate.catalog.pull and load it once via gwsurrogate.LoadSurrogate so fitted coefficients and the empirical-interpolant basis are in memory. Draw 1,000 mass-ratio values uniformly at random from [1,2] with numpy.random, holding spins at zero for the nonspinning fiducial EOB family. Evaluate the loaded surrogate at each sampled q via sur(q, chiA, chiB, dt, f_low) with L=16,384 time samples (dt sized to cover the full waveform, f_low=0 to return the full surrogate), the time array being in units of total mass M. Compute the surrogate reconstruction error against the reference EOB family plus pointwise amplitude and phase differences with gwtools utilities. Aggregate max error, max pointwise amplitude/phase error, and the argmax q into surrogate_validation_results.csv; the passing profile is error uniformly below 1e-7 across the 1,000 draws with the largest error at q = 1.068 (amplitude/phase at or below ~1e-3).

## Related tools

- **gwsurrogate** (Downloads catalog surrogate data (gwsurrogate.catalog.pull), loads the model (gwsurrogate.LoadSurrogate), and evaluates the surrogate at sampled (q, chiA, chiB) to return time array and waveform modes) — https://github.com/sxs-collaboration/gwsurrogate
- **NumPy** (Draws the 1,000 uniformly random mass ratios from [1,2] and provides the C-API headers gwsurrogate's C-extensions need at build time)
- **gwtools** (Computes pointwise amplitude and phase differences of surrogate waveforms against the reference EOB family; required dependency of gwsurrogate) — https://bitbucket.org/chadgalley/gwtools/
- **pytest** (Runs the package's regression tests to cross-check stored waveform outputs of the validation) — https://github.com/sxs-collaboration/gwsurrogate

## Examples

```
sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(q, [0,0,0], [0,0,0], dt=0.1, f_low=0)
```

## Evaluation signals

- Surrogate error (Eq. 31) is uniformly below 1e-7 across all 1,000 sampled mass ratios
- The largest surrogate error occurs at q = 1.068 (Fig. 9 reference value)
- Pointwise amplitude and phase errors stay at or below ~1e-3
- Stored waveform outputs match the gwsurrogate package's pytest regression tests
- Output table surrogate_validation_results.csv contains max error, max pointwise amplitude/phase error, and argmax q for all 1,000 draws

## Limitations

- The surrogate retains the fiducial waveform family's accuracy only within the trained parameter domain; results here cover the nonspinning EOB family with q in [1,2]
- C-extensions require NumPy's C-API headers at build time; a Python 3.11 / numpy<2.0 conda environment may be needed for source builds
- The empirical-interpolant basis ties evaluation cost to O(mL + m*c_fit) for L time samples — L=16,384 must be covered by the chosen dt

## Evidence

- [abstract] Online evaluation cost: [evidence span withheld — anchor status: not_found]
- [methods] Sampling and evaluation via gwsurrogate: "t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low, precessing_opts=precessing_opts)"
- [readme] Load the surrogate model once: "sur = gwsurrogate.LoadSurrogate('NRSur7dq4')"
- [readme] Catalog data download and empirical-interpolant basis availability: "gwsurrogate.catalog.pull('NRSur7dq4')       # This can take a few minutes"
- [readme] gwtools is a hard dependency for waveform comparison utilities: "If you are installing gwsurrogate with pip you will automatically get gwtools"
