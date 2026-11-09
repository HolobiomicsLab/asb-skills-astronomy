---
name: agn-disc-structure-modelling
description: Use when a task requires a fully evolved 1D AGN disc model (radial temperature,
  density, surface-density, scale-height, optical-depth profiles) for a given central
  BH mass, Eddington ratio or accretion rate, and viscosity alpha.
when_to_use_negative:
- The problem requires multi-dimensional disc simulations or comparisons with more
  complex disc models rather than parametric 1D profiles
- The target system is not an AGN accretion disc (e.g. a standard thin alpha-disc
  outside the SG/T05 model assumptions such as self-gravity regime handled here)
- The task needs the disc already solved — e.g. luminosity or torque computation should
  consume the solve_disk output rather than re-solving the structure
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pagn
  role: Python module that solves the Sirko & Goodman 2003 and Thompson et al. 2005
    AGN disc equations via SirkoAGN/ThompsonAGN and solve_disk(N=1e4)
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Numerical arrays for the radial grid and profile quantities returned by the
    solved disc
- name: matplotlib
  role: Plot key disc parameters to check the Sirko & Goodman model ran correctly
    (e.g. plot_mdot)
- name: SciPy
  role: UnivariateSpline (k=3, s=0.005, ext=0) interpolation of log-radius surface-density
    and thermal gradients for downstream torque work; requires scipy >= 1.10.0
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:7a7b701767d8c22a13a1492ddf992f39c69e63a75d8dfe8b15ec3e6017e98959
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - matplotlib
  - numpy
  - pagn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# agn-disc-structure-modelling

## Summary

Solve the 1D steady-state AGN accretion-disc equations of Sirko & Goodman (2003) or Thompson et al. (2005) with the pagn Python module, producing parametric radial profiles of temperature, density, scale height, optical depth, and Q that serve as the basis for luminosity and migration-torque calculations.

## When to use

Use when a task requires a fully evolved 1D AGN disc model (radial temperature, density, surface-density, scale-height, optical-depth profiles) for a given central BH mass, Eddington ratio or accretion rate, and viscosity alpha.

## When NOT to use

- The problem requires multi-dimensional disc simulations or comparisons with more complex disc models rather than parametric 1D profiles
- The target system is not an AGN accretion disc (e.g. a standard thin alpha-disc outside the SG/T05 model assumptions such as self-gravity regime handled here)
- The task needs the disc already solved — e.g. luminosity or torque computation should consume the solve_disk output rather than re-solving the structure

## Inputs

- Central black-hole mass (e.g. Mbh=1e8*ct.MSun or 1e6 Msun)
- Eddington ratio le (e.g. 0.5) or explicit Mdot accretion rate (Thompson: Mdot_out e.g. 1.5e-2 MSun/yr)
- Viscosity parameter alpha (e.g. 0.01), hydrogen fraction X=0.7, opacity choice ('combined' or custom opacity table)
- Number of radial grid points N (e.g. 1e4)
- Thompson-specific parameters: Rout (e.g. Rs*1e7), Rin, sigma, epsilon, m, xi

## Outputs

- Radial profiles of radius, Omega, temperature, density, surface density, scale height, sound speed, optical depth, accretion rate, and Toomre Q
- Saved parameter file (e.g. sirko_luminosity.txt) via the save(obj, filename) routine
- Quick-look diagnostic plots of key disc parameters

## How to apply

Install pagn from PyPI and instantiate the appropriate model: Sirko.SirkoAGN(Mbh, le, Mdot=None, alpha, X=0.7, b=0, opacity='combined') for Sirko & Goodman 2003, or Thompson.ThompsonAGN(Mbh, Rout, Mdot_out, ...) for Thompson et al. 2005; if Mdot is None the accretion rate is derived from the Eddington ratio le. Solve the disc equations with solve_disk(N=1e4) — 10^4 radial grid points is documented as sufficient resolution for most Mbh values. Optionally supply a custom opacity in the required format, then plot key parameters (e.g. plot_mdot() for the Thompson disk) to verify temperature decreases outward from ~1e5–1e6 K and h/r stays below one in the inner disk. Results can be persisted with the save(obj, filename) routine, and the solved profiles (radius, Omega, temperature, density, scale height, sound speed, optical depth, Q) feed downstream skills such as bolometric luminosity summation and migration-torque coefficient evaluation.

## Related tools

- **pagn** (Python module that solves the Sirko & Goodman 2003 and Thompson et al. 2005 AGN disc equations via SirkoAGN/ThompsonAGN and solve_disk(N=1e4)) — https://gitlab.com/aatrani/pagn
- **numpy** (Numerical arrays for the radial grid and profile quantities returned by the solved disc)
- **matplotlib** (Plot key disc parameters to check the Sirko & Goodman model ran correctly (e.g. plot_mdot))
- **SciPy** (UnivariateSpline (k=3, s=0.005, ext=0) interpolation of log-radius surface-density and thermal gradients for downstream torque work; requires scipy >= 1.10.0)

## Examples

```
from pagn import Sirko; import pagn.constants as ct; sk = Sirko.SirkoAGN(Mbh=1e8*ct.MSun, le=0.5, Mdot=None, alpha=0.01, X=0.7, b=0, opacity='combined'); sk.solve_disk(N=1e4)
```

## Evaluation signals

- Solved Sirko & Goodman disc shows temperature decreasing outward from ~1e5–1e6 K
- Scale height ratio h/r stays below one in the inner disk regions
- Luminosity derived from the solved Sirko & Goodman disk reproduces 1054693353944.121 solar luminosities within numerical tolerance
- Output parameter file contains the expected key AGN model parameters (radius, Omega, temperature, density, scale height, sound speed, optical depth, Q)
- N=1e4 radial grid points, documented as sufficient resolution for most Mbh values, produces a converged solution

## Limitations

- 10^4 grid points is a documented guideline ('sufficient for most Mbh values'); very large or small Mbh may require grid-size adjustment
- Custom opacity tables must be provided in a specific format for SirkoAGN and ThompsonAGN
- The GitHub repository is archived and unmaintained; the project moved to GitLab (gitlab.com/aatrani/pagn)
- Torque-adjacent spline interpolation requires scipy >= 1.10.0

## Evidence

- [readme] pagn is a Python module to solve the equations from Sirko and Goodman 2003 and Thompson et al. 2005, providing fully evolved AGN disk models through parametric 1D profiles: "`pagn` is a Python module to solve the equations from [Sirko and Goodman 2003]"
- [methods] solve_disk with 1e4 grid points: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [methods] SirkoAGN instantiation parameters: "sk = Sirko.SirkoAGN(Mbh=Mbh, le=le, Mdot=Mdot, alpha=alpha, X=X, 
                    b=b, opacity = Opacity)"
- [results] Sirko & Goodman disk configured with le=0.5, alpha=0.01, b=0: "obj = Sirko.SirkoAGN(Mbh=Mbh*ct.MSun, le=le, alpha=alpha, b=0)"
- [results] scipy >= 1.10.0 required for spline interpolation: "UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [readme] Repository archived, moved to GitLab: "> ⚠️ **Repository status:** This repository is archived and no longer maintained."
