---
name: kernel-bandwidth-selection
description: Use when performing FINKER-style frequency identification on astronomical
  time series with measurement uncertainties and you must choose kernel_type and bandwidth_method
  before running parallel_nonparametric_kernel_regression over a frequency grid.
when_to_use_negative:
- The data are already folded or a period/frequency estimate is already available
  and only needs refinement
- The analysis is multi-band and you have not accounted for measurement-uncertainty
  weighting that FINKER's kernel regression inherently incorporates
- The dataset is not a time series with uncertainties (e.g. an already-processed feature
  table); kernel-regression folding of the data at trial frequencies is meaningless
edam_operation: http://edamontology.org/operation_0438
tools:
- name: FINKER
  role: Provides parallel_nonparametric_kernel_regression with kernel functions and
    bandwidth calculation (Gaussian kernel, custom fixed bandwidth, Silverman's rule,
    Scott's rule, Adaptive bandwidth) used to select the optimal frequency
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Used to build the frequency grid with np.linspace and call the FINKER search
    (from FINKER.utils_FINKER import *)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:545058f14622d9f63db54a8959c6f15959f07b4a582c13fc943e39683c7551f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - Python
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kernel-bandwidth-selection

## Summary

Selects an appropriate kernel and bandwidth (Gaussian kernel with a custom fixed bandwidth, Silverman's rule, Scott's rule, or Adaptive bandwidth) for FINKER's nonparametric kernel regression, which folds a light curve at trial frequencies and picks the frequency minimising squared residuals.

## When to use

Use when performing FINKER-style frequency identification on astronomical time series with measurement uncertainties and you must choose kernel_type and bandwidth_method before running parallel_nonparametric_kernel_regression over a frequency grid.

## When NOT to use

- The data are already folded or a period/frequency estimate is already available and only needs refinement
- The analysis is multi-band and you have not accounted for measurement-uncertainty weighting that FINKER's kernel regression inherently incorporates
- The dataset is not a time series with uncertainties (e.g. an already-processed feature table); kernel-regression folding of the data at trial frequencies is meaningless

## Inputs

- Numpy arrays of observation times (t_observed), magnitudes/y_observed, and measurement uncertainties
- Frequency grid to search (numpy array, e.g. np.linspace(0.001, 5, 50000))
- Choice of kernel_type ('gaussian'), regression_type ('local_constant' or 'local_linear'), and bandwidth_method ('custom', Silverman, Scott, or Adaptive)

## Outputs

- best_freq: optimal frequency minimising squared residuals over the grid
- freq_err: reported frequency error on the estimate
- result_dict: dictionary including the selected kernel bandwidth

## How to apply

Instantiate a FINKER() object and pass the observation times, magnitudes and uncertainties to parallel_nonparametric_kernel_regression, setting kernel_type='gaussian' and regression_type='local_constant' (or 'local_linear') with bandwidth_method='custom' for computational efficiency, or 'Silverman's rule', 'Scott's rule', or 'Adaptive' when a data-driven bandwidth is preferred. Search a dense frequency grid, e.g. np.linspace(0.001, 5, 50000), because the regression is re-run on folded datasets at each trial frequency and the optimal frequency is the one minimising squared residuals. The custom fixed bandwidth is the paper's default for efficiency; alternative rules trade speed for adaptivity to varying dataset complexity. Retrieve best_freq and freq_err from the returned values/result_dict and check that a known true frequency (e.g. 0.6 in the article's synthetic test with n_points=100, time=10, snr_primary=7) is recovered within freq_err.

## Related tools

- **FINKER** (Provides parallel_nonparametric_kernel_regression with kernel functions and bandwidth calculation (Gaussian kernel, custom fixed bandwidth, Silverman's rule, Scott's rule, Adaptive bandwidth) used to select the optimal frequency) — github.com/FiorenSt/FINKER
- **Python** (Used to build the frequency grid with np.linspace and call the FINKER search (from FINKER.utils_FINKER import *))

## Examples

```
best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t, y_observed=y, uncertainties=unc, freq_list=freq, kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- The returned best_freq recovers a known true frequency (e.g. 0.6 in the article's synthetic test) within the reported freq_err
- The result dictionary contains the selected kernel bandwidth value
- bandwidth_method is one of the implemented options: 'custom', Silverman's rule, Scott's rule, or Adaptive bandwidth
- Frequency search covered a dense grid (e.g. 50000 points from 0.001 to 5) so the squared-residual minimum was sampled finely
- The optimum corresponds to a minimum in squared residuals across folded datasets at the trial frequencies

## Limitations

- The custom fixed bandwidth choice prioritises computational efficiency; adaptive alternatives may be needed for datasets of varying complexity
- No discussion-section content was available ('_No changelog found._'), so limitations and reproducibility gaps cannot be extracted from the article
- Performance claims rest on comparison with the conditional entropy period-finding algorithm on synthetic light curves (n_points=100, time=10, snr_primary=7)

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [results] freq_list = np.linspace(0.001,5,50000): "freq_list = np.linspace(0.001,5,50000)"
- [results] kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom': "kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom'"
- [readme] Includes Gaussian kernel and implements a custom fixed bandwidth for computational efficiency. Multiple other options are available: Silverman's rule, Scott's rule, and an Adaptive bandwidth.: [evidence span withheld — anchor status: external_doc]
- [results] best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
