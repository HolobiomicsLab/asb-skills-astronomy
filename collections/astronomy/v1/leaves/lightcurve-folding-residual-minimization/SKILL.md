---
name: lightcurve-folding-residual-minimization
description: Use when an unevenly sampled astronomical time series (e.g. ~100 points
  over 10 time units) needs its primary frequency recovered, and a residual-minimising
  nonparametric kernel regression over a frequency grid is preferred over conditional
  entropy period-finding.
when_to_use_negative:
- The goal is comparative benchmarking against another period-finding method rather
  than frequency recovery — although FINKER was compared with conditional entropy,
  the skill itself only runs the kernel-regression search
- The data are already folded or the frequency is known; folding/residual search is
  only needed to identify an unknown frequency
- Multi-band processing is required without preparing per-band inputs — multiband
  support exists but the single-band call shown takes single t_observed/y_observed/uncertainties
  arrays
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: FINKER
  role: Provides parallel_nonparametric_kernel_regression, which folds data at trial
    frequencies and selects best_freq by minimising squared residuals; installed via
    pip install FINKER
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Hosts the FINKER API calls, NumPy frequency-grid construction (np.linspace),
    and JSON serialisation of results
- name: NumPy
  role: Constructs the frequency search grid freq_list = np.linspace(0.001, 5, 50000)
    and supplies observation arrays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:c11a5b496856a9d8b001000f5827f28d237b0a03e7303ecdfde1122c7bac65de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# lightcurve-folding-residual-minimization

## Summary

Identify the dominant frequency of an astronomical light curve by running FINKER's parallel nonparametric kernel regression on datasets folded across a frequency grid, selecting the frequency that minimises squared residuals.

## When to use

Use when an unevenly sampled astronomical time series (e.g. ~100 points over 10 time units) needs its primary frequency recovered, and a residual-minimising nonparametric kernel regression over a frequency grid is preferred over conditional entropy period-finding.

## When NOT to use

- The goal is comparative benchmarking against another period-finding method rather than frequency recovery — although FINKER was compared with conditional entropy, the skill itself only runs the kernel-regression search
- The data are already folded or the frequency is known; folding/residual search is only needed to identify an unknown frequency
- Multi-band processing is required without preparing per-band inputs — multiband support exists but the single-band call shown takes single t_observed/y_observed/uncertainties arrays

## Inputs

- t_observed: numpy array of observation timestamps
- y_observed: numpy array of magnitudes/fluxes
- uncertainties: numpy array of per-point measurement uncertainties
- freq_list: numpy array of trial frequencies, e.g. np.linspace(0.001, 5, 50000)

## Outputs

- best_freq: the optimal frequency minimising squared residuals (float)
- freq_err: estimated frequency error (float)
- result_dict: dictionary including the selected kernel bandwidth
- JSON record (e.g. finker_frequency_search_results.json) with fields best_freq, freq_err, bandwidth

## How to apply

Generate or load a light curve with known timestamps, magnitudes and measurement uncertainties, then build a frequency grid with np.linspace(0.001, 5, 50000). Instantiate a FINKER object and call parallel_nonparametric_kernel_regression with kernel_type='gaussian', regression_type='local_constant', and bandwidth_method='custom', passing n_jobs=-2 for parallel search; the method folds the data at each trial frequency and applies nonparametric kernel regression, inherently weighting by measurement uncertainties. Retrieve best_freq, freq_err and the result dictionary, and persist the three values (best_freq, freq_err, bandwidth) as a JSON record. FINKER selects the optimal frequency by minimising squared residuals and can work with as few as 10 observations.

## Related tools

- **FINKER** (Provides parallel_nonparametric_kernel_regression, which folds data at trial frequencies and selects best_freq by minimising squared residuals; installed via pip install FINKER) — github.com/FiorenSt/FINKER
- **Python** (Hosts the FINKER API calls, NumPy frequency-grid construction (np.linspace), and JSON serialisation of results)
- **NumPy** (Constructs the frequency search grid freq_list = np.linspace(0.001, 5, 50000) and supplies observation arrays)

## Examples

```
from FINKER.utils_FINKER import *; best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t_observed, y_observed=y_observed, uncertainties=uncertainties, freq_list=np.linspace(0.001,5,50000), show_plot=False, kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- Recorded best_freq recovers the known true frequency (e.g. 0.6) within the reported freq_err
- Output JSON contains exactly the fields best_freq, freq_err and bandwidth
- The returned frequency falls inside the searched grid np.linspace(0.001, 5, 50000)
- The call used kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom'; any deviation should be flagged
- Best-frequency selection corresponds to the minimum of the squared-residual curve across trial frequencies

## Limitations

- Discussion section is empty (only a 'No changelog found.' placeholder), so limitations and reproducibility gaps cannot be extracted from the article
- With very sparse data the method is designed to work 'even with as few as 10 observations', but robustness at extreme sparsity beyond the paper's claims is unverified here
- bandwidth_method='custom' fixes the bandwidth for computational efficiency; Silverman's rule, Scott's rule and Adaptive bandwidth are alternatives that may be needed for varying data complexity

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [results] Frequency search grid and parameters: "freq_list = np.linspace(0.001,5,50000)"
- [results] FINKER call signature: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
- [results] Custom bandwidth and kernel settings: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
- [abstract] Uncertainty weighting claim: "This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis."
- [readme] README usage example: [evidence span withheld — anchor status: external_doc]
