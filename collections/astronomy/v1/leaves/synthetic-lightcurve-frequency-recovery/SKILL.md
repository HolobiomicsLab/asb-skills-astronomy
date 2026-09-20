---
name: synthetic-lightcurve-frequency-recovery
description: Use when benchmarking a period-finding/frequency-identification algorithm
  against a baseline on synthetic time series with a known injected primary frequency,
  SNR, and number of observations.
when_to_use_negative:
- Input data is real observational light curves without a known ground-truth frequency
  — the skill's validation logic depends on recovering an injected true frequency
- The signal lies outside the searched frequency range (0.001–5); FINKER and the baseline
  cannot recover frequencies beyond the grid
- The analysis targets multi-band photometry or different kernel/bandwidth configurations
  rather than reproducing the paper's single-band gaussian/local_constant/custom-bandwidth
  setup
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3318
tools:
- name: FINKER
  role: Nonparametric kernel regression frequency search over the folded light curve,
    selecting the optimal frequency by minimising squared residuals
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Execution environment for the synthetic light-curve generator, the FINKER
    API, and the conditional-entropy baseline implementation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:8f0d43a61940cb209f5eb48bad01a9055d1c90cd05d17cb2bd93ef215183b67e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - Python
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# synthetic-lightcurve-frequency-recovery

## Summary

Recover a known true frequency from a synthetic astronomical light curve by running FINKER's nonparametric kernel regression frequency search and, in parallel, the conditional-entropy baseline on the same curve, then compare both estimates against the ground-truth frequency (e.g. 0.6).

## When to use

Use when benchmarking a period-finding/frequency-identification algorithm against a baseline on synthetic time series with a known injected primary frequency, SNR, and number of observations.

## When NOT to use

- Input data is real observational light curves without a known ground-truth frequency — the skill's validation logic depends on recovering an injected true frequency
- The signal lies outside the searched frequency range (0.001–5); FINKER and the baseline cannot recover frequencies beyond the grid
- The analysis targets multi-band photometry or different kernel/bandwidth configurations rather than reproducing the paper's single-band gaussian/local_constant/custom-bandwidth setup

## Inputs

- Synthetic light curve parameters (n_points=100, time=10, freq_primary=0.6, snr_primary=7, baseline_magnitude=17.0)
- Time-magnitude observation arrays (t_observed, y_observed) with measurement uncertainties
- Frequency search grid: np.linspace(0.001, 5, 50000)

## Outputs

- FINKER best frequency estimate with associated frequency error (best_freq, freq_err, result_dict)
- Conditional-entropy baseline best frequency estimate
- Structured JSON record (e.g. conditional_entropy_result.json) containing the estimated frequency, true frequency, and FINKER comparison

## How to apply

Generate a synthetic light curve with a known true frequency (true_freq = 0.6) via generator.generate_synthetic_light_curve with n_points=100, time=10, snr_primary=7, baseline_magnitude=17.0. Define the search grid as freq_list = np.linspace(0.001, 5, 50000). Run the FINKER search with finker.parallel_nonparametric_kernel_regression using kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', and n_jobs=-2. Independently apply the conditional-entropy period-finding algorithm to the same time-magnitude pairs over the identical frequency grid, phase-folding and minimising the conditional-entropy statistic. Write the estimated frequencies for both methods (and the true frequency) to a structured JSON record (e.g. conditional_entropy_result.json) and validate that each estimate falls within 0.001–5 and is comparable to 0.6 and to each other.

## Related tools

- **FINKER** (Nonparametric kernel regression frequency search over the folded light curve, selecting the optimal frequency by minimising squared residuals) — github.com/FiorenSt/FINKER
- **Python** (Execution environment for the synthetic light-curve generator, the FINKER API, and the conditional-entropy baseline implementation)

## Examples

```
from FINKER.utils_FINKER import *; finker = FINKER(); best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t_observed, y_observed=y_observed, uncertainties=uncertainties, freq_list=np.linspace(0.001,5,50000), kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- conditional_entropy_result.json exists and contains an estimated-frequency field
- Estimated frequency falls within the searched frequency range 0.001–5
- Estimated frequency is comparable to the true frequency 0.6 and to the FINKER estimate on the same curve
- FINKER call returns three outputs (best_freq, freq_err, result_dict) with kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom'
- Both methods were run on the identical frequency grid of 50000 points from 0.001 to 5

## Limitations

- Performance comparison depends on the conditional-entropy baseline being run over the same frequency grid as FINKER; mismatched grids invalidate the comparison
- The synthetic setup uses only 100 points over time=10 with snr_primary=7; results may not transfer to different sampling, SNR, or curve complexity
- No discussion-section content is available in the article, so limitations and reproducibility gaps for this workflow are not documented by the authors

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [abstract] compared it with an established period-finding algorithm, conditional entropy: "compared it with an established period-finding algorithm, conditional entropy."
- [results] generator call with parameters: "n_points=100,
    time=10,
    freq_primary=true_freq,
    snr_primary=7,
    baseline_magnitude=17.0,"
- [results] frequency grid: "freq_list = np.linspace(0.001,5,50000)"
- [results] kernel and regression settings: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
- [readme] README usage example: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(
    t_observed=t_observed,"
