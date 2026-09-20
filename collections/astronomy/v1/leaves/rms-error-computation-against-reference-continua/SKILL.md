---
name: rms-error-computation-against-reference-continua
description: Use when benchmarking spectrum-normalisation output against known reference
  continua — e.g. deposited test-star spectra (O to G type) or synthetic spectra with
  trusted continua — to reproduce an RMS-error table.
when_to_use_negative:
- When no reference continua exist for the spectra — the RMS can only be computed
  against trusted continua, not against data-normalised pseudo-continua
- When spectra have wavelengths in nanometers without adjusting the sampling (default
  0.05 assumes angstroms), since the normalisation and hence the RMS comparison would
  be invalid
tools:
- name: SUPPNet
  role: Produces the normalised flux that is compared to the reference continua via
    RMS error
  repo: https://github.com/RozanskiT/suppnet.git
- name: Python
  role: Runtime environment in which SUPPNet is installed as a module and RMS computation
    is performed
- name: anaconda
  role: Python distribution providing the suppnet-env environment required to run
    SUPPNET before computing RMS values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202141480
    title: 'SUPPNet: Neural network for stellar spectrum normalisation'
  source_content_hash: sha256:5d5e394ab9a562963c9d9c5d563987824a22670095adefc7417d4f54bba93c97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RozanskiT/suppnet.git
  tools:
  - Python
  - SUPPNet
  - anaconda
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rms-error-computation-against-reference-continua

## Summary

Quantifies normalisation accuracy by computing the root mean squared (RMS) error between SUPPNet-normalised flux and reference continua, per spectrum and window-averaged, mirroring the paper's reported metrics (0.0128 over 3900-7000 Å, 0.0081 over 4200-7000 Å, ~0.0050 for synthetic spectra).

## When to use

Use when benchmarking spectrum-normalisation output against known reference continua — e.g. deposited test-star spectra (O to G type) or synthetic spectra with trusted continua — to reproduce an RMS-error table.

## When NOT to use

- When no reference continua exist for the spectra — the RMS can only be computed against trusted continua, not against data-normalised pseudo-continua
- When spectra have wavelengths in nanometers without adjusting the sampling (default 0.05 assumes angstroms), since the normalisation and hence the RMS comparison would be invalid

## Inputs

- Headerless two-column spectra (first column wavelengths in angstroms, second column flux) for O to G type test stars
- Synthetic test spectra with known reference continua
- Reference continuum arrays for each test spectrum
- SUPPNet-normalised flux (same sampling as input, --sampling 0.05 default)

## Outputs

- Per-spectrum RMS error values over the 3900-7000 Å window
- Per-spectrum RMS error values over the 4200-7000 Å window
- Window-averaged RMS table mirroring the paper's reported metrics, with discrepancy flags

## How to apply

Normalise each headerless two-column spectrum (wavelengths in angstroms, flux) with SUPPNET in unsupervised batch mode, e.g. 'SUPPNET --quiet --sampling 0.05 --weights active', using --weights synth as a variant for synthetic spectra. For each normalised spectrum, compute the RMS error between the SUPPNet-normalised flux and the reference continuum over two windows: 3900-7000 Å and 4200-7000 Å (the narrower window excludes the blue region where errors are larger). Repeat for synthetic test spectra, where the paper reports RMS of the order of 0.0050. Aggregate per-spectrum and window-averaged RMS values into a table mirroring the reported metrics; validate against the expected values (0.0128 and 0.0081 over test stars) and flag discrepancies in the output table.

## Related tools

- **SUPPNet** (Produces the normalised flux that is compared to the reference continua via RMS error) — https://github.com/RozanskiT/suppnet.git
- **Python** (Runtime environment in which SUPPNet is installed as a module and RMS computation is performed)
- **anaconda** (Python distribution providing the suppnet-env environment required to run SUPPNET before computing RMS values)

## Examples

```
SUPPNET --quiet --sampling 0.05 --weights active test_star_spectrum.txt
```

## Evaluation signals

- Aggregate RMS over the test stars must equal 0.0128 in the 3900-7000 Å range
- Aggregate RMS over the test stars must equal 0.0081 in the 4200-7000 Å range
- RMS on synthetic spectra must be of the order of 0.0050; otherwise flag the discrepancy in the output table
- Output table structure mirrors the paper's reported per-spectrum and window-averaged metrics

## Limitations

- The published validation values (0.0128, 0.0081) apply to high-resolution spectra of O to G type stars; other spectral types or resolutions may deviate and must be flagged
- The narrower 4200-7000 Å window excludes the 3900-4200 Å region, where normalisation errors are evidently larger (RMS 0.0128 vs 0.0081), so window choice materially affects the reported value
- The synth weights set is trained only on synthetic spectra and the README notes it 'shouldn't be used' in general, limiting the synthetic-spectrum benchmark variant

## Evidence

- [abstract] RMS error of 0.0128 in the spectral range 3900-7000 Å and 0.0081 in the range 4200-7000 Å for high-resolution spectra of O to G type stars: "gives a root mean squared (RMS) error over the set of test stars equal to 0.0128 in the spectral range from 3900 Å to 7000 Å and 0.0081 in the range from 4200 Å to 7000 Å"
- [abstract] RMS of the order of 0.0050 for synthetic spectra: "Experiments with synthetic spectra give a RMS of the order of 0.0050"
- [readme] Input spectra are headerless two-column files: "Spectra that you are working with should't have header: the forst column should contain wavelengths in angstroms"
- [readme] Batch normalisation command used before RMS computation: "Normalisation of group of spectra without any supervision: SUPPNET --quiet [--sampling RESAMPLING_STEP=0.05]"
