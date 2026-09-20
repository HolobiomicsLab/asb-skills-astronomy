---
name: uncertainty-weighted-lightcurve-regression
description: Use when the input is an astronomical time series (t_observed, y_observed,
  uncertainties) and the goal is to recover periodic frequencies — even with sparse
  data or multi-band observations — by minimising squared residuals over a trial frequency
  grid.
when_to_use_negative:
- The dataset has no associated measurement uncertainties and uncertainty weighting
  is essential to the analysis
- The signal is aperiodic/non-variable, so folding at trial frequencies and residual
  minimisation is not meaningful
- Only a single best period with no uncertainty/error estimate (freq_err) is needed
  and the kernel-regression machinery is overkill
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0593
tools:
- name: FINKER
  role: Core library providing parallel_nonparametric_kernel_regression, Gaussian
    kernel, local_constant/local_linear regression, custom/Silverman/Scott/Adaptive
    bandwidths, and multiband processing
  repo: github.com/FiorenSt/FINKER
- name: NumPy
  role: Builds the trial frequency grid (np.linspace(0.001, 5, 50000)) and provides
    the observation arrays passed to FINKER
- name: Python
  role: Execution environment; FINKER is importable via 'from FINKER.utils_FINKER
    import *' and installable with pip
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:6e1c73460522df67088f3f552fbc2baa1df9b02107704de837c2307055f24e75
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# uncertainty-weighted-lightcurve-regression

## Summary

Frequency identification in astronomical time series by nonparametric kernel regression (FINKER): fold the light curve at trial frequencies, fit a Gaussian-kernel local-constant (or local-linear) regression weighted by measurement uncertainties, and select the frequency minimising squared residuals.

## When to use

Use when the input is an astronomical time series (t_observed, y_observed, uncertainties) and the goal is to recover periodic frequencies — even with sparse data or multi-band observations — by minimising squared residuals over a trial frequency grid.

## When NOT to use

- The dataset has no associated measurement uncertainties and uncertainty weighting is essential to the analysis
- The signal is aperiodic/non-variable, so folding at trial frequencies and residual minimisation is not meaningful
- Only a single best period with no uncertainty/error estimate (freq_err) is needed and the kernel-regression machinery is overkill

## Inputs

- numpy array t_observed (observation times)
- numpy array y_observed (magnitudes/fluxes)
- numpy array uncertainties (per-point measurement uncertainties used as regression weights)
- numpy array freq_list of trial frequencies (e.g. np.linspace(0.001, 5, 50000))

## Outputs

- best_freq (optimal frequency minimising squared residuals)
- freq_err (frequency uncertainty estimate)
- result_dict (table of trial frequency vs squared residual, savable as result_dict.csv)

## How to apply

Instantiate a FINKER() object and call parallel_nonparametric_kernel_regression with numpy arrays t_observed, y_observed, uncertainties and a trial frequency grid (the paper uses freq_list = np.linspace(0.001, 5, 50000)). Configure kernel_type='gaussian', regression_type='local_constant' (local_linear also available), and bandwidth_method='custom' fixed bandwidth for computational efficiency (Silverman's rule, Scott's rule, and Adaptive bandwidth are alternatives), with n_jobs=-2 for parallel search. At each trial frequency the folded data are fit with the kernel regression, weighting observations by their measurement uncertainties. The function returns best_freq, freq_err, and result_dict mapping each trial frequency to its squared residual; the optimal frequency is the grid point minimising squared residuals. Validate by checking that the minimum of the squared-residual curve coincides with the known/expected frequency within grid resolution, as in the paper's synthetic test recovering true_freq=0.6.

## Related tools

- **FINKER** (Core library providing parallel_nonparametric_kernel_regression, Gaussian kernel, local_constant/local_linear regression, custom/Silverman/Scott/Adaptive bandwidths, and multiband processing) — github.com/FiorenSt/FINKER
- **NumPy** (Builds the trial frequency grid (np.linspace(0.001, 5, 50000)) and provides the observation arrays passed to FINKER)
- **Python** (Execution environment; FINKER is importable via 'from FINKER.utils_FINKER import *' and installable with pip)

## Examples

```
from FINKER.utils_FINKER import *; finker = FINKER(); best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t_observed, y_observed=y_observed, uncertainties=uncertainties, freq_list=freq, kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- The minimum of the squared-residual-vs-frequency curve coincides with the injected/expected true frequency within grid resolution (paper: 0.6 recovered in the synthetic test)
- best_freq lies within the trial freq_list grid (0.001–5 in the paper's test)
- result_dict contains a squared-residual entry for every trial frequency (50000 rows), matching the frequency grid length
- freq_err is returned and finite alongside best_freq
- Re-running with the paper's settings (kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom') reproduces the synthetic recovery

## Limitations

- The discussion section content is absent ('No changelog found.' placeholder), so no extracted limitations or reproducibility gaps from the article itself
- Only a single synthetic test (true_freq=0.6, n_points=100, time=10, snr_primary=7, baseline_magnitude=17.0) is shown in the results section as validation
- Performance claims (superior accuracy, robustness vs conditional entropy) come from the abstract; specific failure modes/edge cases are not documented in the provided context

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [abstract] This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis.: "This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis."
- [results] kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom',: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
- [results] freq_list = np.linspace(0.001,5,50000): "freq_list = np.linspace(0.001,5,50000)"
- [readme] Includes Gaussian kernel and implements a custom fixed bandwidth for computational efficiency. Multiple other options are available: Silverman's rule, Scott's rule, and an Adaptive bandwidth.: [evidence span withheld — anchor status: external_doc]
- [results] best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
