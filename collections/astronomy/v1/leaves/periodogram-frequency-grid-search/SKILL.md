---
name: periodogram-frequency-grid-search
description: Use when you need to identify a significant frequency in an astronomical
  light curve (possibly multiband, with measurement uncertainties, and with as few
  as ~10-100 observations) by scanning a dense frequency grid with a kernel-regression-based
  period-finding method.
when_to_use_negative:
- When the goal is to compare against or benchmark the conditional entropy period-finding
  algorithm rather than run the kernel-regression search itself.
- When the data are not sparse irregularly sampled time series — with densely sampled,
  evenly spaced data a standard FFT-based periodogram may suffice.
- 'When no frequency grid can be specified: the skill requires an explicit freq_list
  of candidate frequencies to scan.'
edam_operation: http://edamontology.org/operation_0395
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: FINKER
  role: Provides the parallel nonparametric kernel regression frequency search executed
    over the frequency grid
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Runtime environment in which FINKER utilities (FINKER.utils_FINKER) are imported
    and the search is invoked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:c06191a687aef08a0851e904af7e9634cf85134006701c525a89cd86016f237b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - Python
  edam_operation: http://edamontology.org/operation_0395
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# periodogram-frequency-grid-search

## Summary

Runs FINKER's parallel nonparametric kernel regression frequency search over an explicitly defined frequency grid (e.g. np.linspace(0.001, 5, 50000)) on astronomical time series, returning the best frequency and its error by minimising squared residuals of the kernel fit on folded data.

## When to use

Use when you need to identify a significant frequency in an astronomical light curve (possibly multiband, with measurement uncertainties, and with as few as ~10-100 observations) by scanning a dense frequency grid with a kernel-regression-based period-finding method.

## When NOT to use

- When the goal is to compare against or benchmark the conditional entropy period-finding algorithm rather than run the kernel-regression search itself.
- When the data are not sparse irregularly sampled time series — with densely sampled, evenly spaced data a standard FFT-based periodogram may suffice.
- When no frequency grid can be specified: the skill requires an explicit freq_list of candidate frequencies to scan.

## Inputs

- numpy arrays of observation times (t_observed), magnitudes/fluxes (y_observed), and measurement uncertainties (uncertainties)
- frequency search grid as a numpy array (e.g. freq_list = np.linspace(0.001, 5, 50000))
- optional multiband light-curve data (FINKER facilitates multi-band data analysis)

## Outputs

- best_freq: the optimal frequency selected by minimising squared residuals over the grid
- freq_err: uncertainty estimate on the recovered frequency
- result_dict: per-frequency regression results (returned alongside best_freq and freq_err)

## How to apply

Instantiate a FINKER object and call parallel_nonparametric_kernel_regression with the observation times, magnitudes, and uncertainties as numpy arrays, passing an explicit freq_list such as np.linspace(0.001, 5, 50000). Configure the search with kernel_type='gaussian', regression_type='local_constant', and bandwidth_method='custom' (the custom fixed bandwidth is used for computational efficiency; Silverman's, Scott's, and adaptive options are alternatives), and set n_jobs=-2 for parallel evaluation. FINKER applies nonparametric kernel regression to the data folded at each candidate frequency and selects the optimal frequency by minimising squared residuals; the weighting system inherently accounts for measurement uncertainties. The call returns (best_freq, freq_err, result_dict); for validation on synthetic data, generate a curve with a known true_freq (e.g. 0.6, n_points=100, time=10, snr_primary=7, baseline_magnitude=17.0) and check best_freq lies within freq_err of the truth.

## Related tools

- **FINKER** (Provides the parallel nonparametric kernel regression frequency search executed over the frequency grid) — github.com/FiorenSt/FINKER
- **Python** (Runtime environment in which FINKER utilities (FINKER.utils_FINKER) are imported and the search is invoked)

## Examples

```
best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t, y_observed=y, uncertainties=err, freq_list=np.linspace(0.001,5,50000), kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- best_freq lies within the returned freq_err of a known injected true frequency (e.g. true_freq=0.6 in the paper's synthetic test)
- result_dict contains residual information for every frequency in freq_list (50000 grid points for the reference setup)
- the recovered frequency remains stable across varying SNR levels for a fixed curve setup (n_points=100, time=10), consistent with reported noise resilience
- the run completes with n_jobs=-2 parallelism and the chosen bandwidth_method ('custom') without errors for small-N inputs
- for multiband inputs, the uncertainty-weighted fit returns a single consistent best_freq across bands

## Limitations

- Frequency recovery is bounded by the supplied freq_list range and resolution (0.001–5 with 50000 points in the reference test); frequencies outside the grid cannot be found.
- The 'custom' bandwidth method is a fixed bandwidth chosen for computational efficiency; Silverman's, Scott's, and adaptive alternatives may be needed for other data regimes.
- The article's discussion section is empty aside from a 'No changelog found.' placeholder, so no limitations or reproducibility gaps are documented in the source itself.

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [results] freq_list = np.linspace(0.001,5,50000): [evidence span withheld — anchor status: not_found]
- [results] kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom': [evidence span withheld — anchor status: not_found]
- [readme] Includes Gaussian kernel and implements a custom fixed bandwidth for computational efficiency. Multiple other options are available: Silverman's rule, Scott's rule, and an Adaptive bandwidth.: [evidence span withheld — anchor status: not_found]
- [abstract] It exhibits resilience against noise and adapts well to datasets with varying complexity.: "It exhibits resilience against noise and adapts well to datasets with varying complexity."
