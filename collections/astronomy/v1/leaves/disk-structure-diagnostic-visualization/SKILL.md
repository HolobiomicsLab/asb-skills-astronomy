---
name: disk-structure-diagnostic-visualization
description: Use when a pagn SirkoAGN or ThompsonAGN disk has just been solved and
  you need to verify the solution ran correctly before using the profiles (temperature,
  density) downstream.
when_to_use_negative:
- You need quantitative model outputs (e.g. bolometric luminosity via calculate_lum_sum,
  or splined surface-density gradients) rather than a qualitative correctness check
- The disk object has not yet been solved with solve_disk — plotting before solving
  yields no profiles to inspect
- You need a custom diagnostic beyond the key parameters pagn plots (e.g. per-wavelength
  spectral quantities with UnivariateSpline interpolation)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Provides SirkoAGN.solve_disk and the .plot() diagnostic method for key disk
    parameters
  repo: https://gitlab.com/aatrani/pagn
- name: matplotlib
  role: Rendering backend for the diagnostic panel of disk parameters
- name: numpy
  role: Numerical arrays underpinning the solved disk profiles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:9e4620a7364e371ef49c0f8832d00d86966ba4d1b5c39b0eb4a1e9daf8982a18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# disk-structure-diagnostic-visualization

## Summary

A quick-check diagnostic for the pagn Sirko & Goodman AGN disk model: after solve_disk, plot key disk parameters (temperature profile, h/r ratio, and the inner/outer regime switch line) to verify the solution is physically sensible.

## When to use

Use when a pagn SirkoAGN or ThompsonAGN disk has just been solved and you need to verify the solution ran correctly before using the profiles (temperature, density) downstream.

## When NOT to use

- You need quantitative model outputs (e.g. bolometric luminosity via calculate_lum_sum, or splined surface-density gradients) rather than a qualitative correctness check
- The disk object has not yet been solved with solve_disk — plotting before solving yields no profiles to inspect
- You need a custom diagnostic beyond the key parameters pagn plots (e.g. per-wavelength spectral quantities with UnivariateSpline interpolation)

## Inputs

- A solved pagn Sirko.SirkoAGN object (post solve_disk(N=1e4)) with parametric 1D profiles of temperature, density, and h/r
- Configuration parameters: Mbh, le or Mdot, alpha, X, b, opacity (built-in or custom opacity table in the required format)

## Outputs

- A diagnostic matplotlib figure of key disk parameters with a vertical line marking the inner/outer regime switch
- A pass/fail sanity verdict: temperature decreasing outward from 10^5–10^6 K and h/r < 1 in the inner disk

## How to apply

After configuring Sirko.SirkoAGN (e.g. Mbh=1e8 MSun, le=0.5 with Mdot=None, alpha=0.01, X=0.7, b=0, opacity='combined') and running sk.solve_disk(N=1e4) — the paper notes 10^4 grid points is sufficient resolution for most Mbh values — call sk.plot() to render a panel of key disk parameters, including a vertical line marking the switch between the inner and outer disk regimes. Judge the output valid only if the plotted temperature decreases monotonically with distance from the central black hole, starts at 10^5–10^6 K near the inner edge, and the h/r ratio stays below one in the inner disk regions, matching the paper's quick-check criteria. For the Thompson et al. model, the analogous diagnostic is tho.plot_mdot(), which visualizes the accretion rate through the disk.

## Related tools

- **pagn** (Provides SirkoAGN.solve_disk and the .plot() diagnostic method for key disk parameters) — https://gitlab.com/aatrani/pagn
- **matplotlib** (Rendering backend for the diagnostic panel of disk parameters)
- **numpy** (Numerical arrays underpinning the solved disk profiles)

## Examples

```
from pagn import Sirko; sk = Sirko.SirkoAGN(Mbh=1e8*ct.MSun, le=0.5, Mdot=None, alpha=0.01, X=0.7, b=0, opacity='combined'); sk.solve_disk(N=1e4); sk.plot()
```

## Evaluation signals

- Plotted temperature decreases with distance from the central black hole
- Inner-disk temperature falls in the 10^5–10^6 K range at the start of the profile
- h/r ratio stays below one in the inner disk regions (thin-disk assumption holds)
- The vertical line marking the inner/outer regime switch appears at a plausible radius in the figure
- Solution was computed at N=1e4 grid points, stated as sufficient for most Mbh values

## Limitations

- The diagnostic is qualitative; it confirms the model ran correctly but does not validate quantitative outputs like luminosities
- The GitHub repository DariaGangardt/pAGN is archived and unmaintained; the project moved to GitLab (gitlab.com/aatrani/pagn)
- Custom opacity values must be provided in a specific format for SirkoAGN/ThompsonAGN to accept them

## Evidence

- [methods] To quickly check that the Sirko & Goodman model ran correctly, we can plot some key parameters: "To quickly check that the Sirko & Goodman model ran correctly, we can plot some key parameters"
- [methods] sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [readme] pagn is a Python module to solve the equations from Sirko and Goodman 2003 and Thompson et al. 2005, providing fully evolved AGN disk models through parametric 1D profiles: [evidence span withheld — anchor status: not_found]
