---
name: radial-temperature-profile-validation
description: Use when a Sirko.SirkoAGN (or Thompson) disk model in pagn has been solved
  (e.g. solve_disk(N=1e4)) and you need to confirm the solution ran correctly via
  the paper's quick-check criteria before further analysis.
when_to_use_negative:
- The disk object has not yet been solved with solve_disk — the diagnostic only validates
  a completed solution
- You need quantitative validation against published luminosities (e.g. bolometric
  luminosity via calculate_lum_sum) rather than a qualitative visual sanity check
- The target is a disk model outside the pagn Sirko & Goodman / Thompson et al. implementations,
  whose parameter conventions the diagnostic assumes
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Provides Sirko.SirkoAGN / Thompson.ThompsonAGN with solve_disk(N=1e4) and
    the built-in plot() / plot_mdot() quick-check diagnostics
  repo: https://gitlab.com/aatrani/pagn
- name: matplotlib
  role: Rendering backend for the diagnostic plots of key disk parameters
- name: numpy
  role: Numerical arrays supporting the disk profiles being plotted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:0035f92ca4aeeeaafa8d854306594ee75772f186b91febc17491de7fc29506be
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
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# radial-temperature-profile-validation

## Summary

A quick-check diagnostic for the pagn AGN disk solvers: after solving the Sirko & Goodman (2003) disk equations, plot key disk parameters and verify the temperature profile decreases outward from ~10^5–10^6 K while the h/r ratio stays below unity in the inner disk.

## When to use

Use when a Sirko.SirkoAGN (or Thompson) disk model in pagn has been solved (e.g. solve_disk(N=1e4)) and you need to confirm the solution ran correctly via the paper's quick-check criteria before further analysis.

## When NOT to use

- The disk object has not yet been solved with solve_disk — the diagnostic only validates a completed solution
- You need quantitative validation against published luminosities (e.g. bolometric luminosity via calculate_lum_sum) rather than a qualitative visual sanity check
- The target is a disk model outside the pagn Sirko & Goodman / Thompson et al. implementations, whose parameter conventions the diagnostic assumes

## Inputs

- Solved pagn Sirko.SirkoAGN or Thompson.ThompsonAGN object (after solve_disk(N=1e4))
- Model configuration: Mbh, le (e.g. 0.5), alpha (e.g. 0.01), X=0.7, b=0, opacity='combined'
- numpy and matplotlib imported for rendering/inspection

## Outputs

- Diagnostic matplotlib figure of key disk parameters (temperature profile, h/r ratio, regime-switch vertical line)
- For Thompson disks: accretion-rate plot via tho.plot_mdot()
- Optional txt file of key AGN model parameters via the save() method

## How to apply

First instantiate and solve the model as in the paper's methods, e.g. Sirko.SirkoAGN(Mbh=1e8 MSun, le=0.5 with Mdot=None so it is computed from le, alpha=0.01, X=0.7, b=0, opacity='combined'), then call sk.solve_disk(N=1e4), the resolution stated to be sufficient for most Mbh values. Render the built-in diagnostic with sk.plot(), which plots key disk parameters including a vertical line marking the switch between the inner and outer disk regimes. Judge the output correct only if the plotted temperature decreases monotonically with distance from the central black hole, starting at 10^5–10^6 K, and the h/r ratio remains below one in the inner disk regions. For the Thompson et al. (2005) model, an analogous sanity check is tho.plot_mdot() to inspect the accretion rate profile. Results can also be saved to a txt file for record-keeping.

## Related tools

- **pagn** (Provides Sirko.SirkoAGN / Thompson.ThompsonAGN with solve_disk(N=1e4) and the built-in plot() / plot_mdot() quick-check diagnostics) — https://gitlab.com/aatrani/pagn
- **matplotlib** (Rendering backend for the diagnostic plots of key disk parameters)
- **numpy** (Numerical arrays supporting the disk profiles being plotted)

## Examples

```
from pagn import Sirko; sk = Sirko.SirkoAGN(Mbh=1e8, le=0.5, alpha=0.01, X=0.7, b=0, opacity='combined'); sk.solve_disk(N=1e4); sk.plot()
```

## Evaluation signals

- Plotted temperature decreases with distance from the central BH, starting at 10^5–10^6 K
- h/r ratio stays below one in the inner disk regions
- Vertical line marking the switch between inner and outer regimes is present in the diagnostic figure
- Model was solved with N=1e4 grid points, the resolution the paper states is sufficient for most Mbh values
- For Thompson disks, the plot_mdot() accretion-rate profile is physically consistent with Mdot_out = 1.5e-2 MSun/yr and Rout = Rs*1e7

## Limitations

- N=1e4 'tends to be' sufficient resolution for most Mbh values — very high or very low Mbh may need adjustment
- The GitHub repository is archived and no longer maintained; the project moved to GitLab (gitlab.com/aatrani/pagn)
- The diagnostic is a quick visual check; quantitative checks (e.g. bolometric luminosity sums) require separate steps, and UnivariateSpline-based work needs scipy version 1.10.0

## Evidence

- [methods] quick-check: "To quickly check that the Sirko & Goodman model ran correctly, we can plot some key parameters"
- [methods] resolution: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [methods] solve signature: "sk = Sirko.SirkoAGN(Mbh=Mbh, le=le, Mdot=Mdot, alpha=alpha, X=X, b=b, opacity = Opacity)"
- [readme] archival note: "This repository is archived and no longer maintained.
> The project has moved to GitLab:"
- [methods] thompson mdot plot: "one can also plot the accretion rate in the Thompson et al. disk:

.. code:: ipython3

    tho.plot_mdot()"
