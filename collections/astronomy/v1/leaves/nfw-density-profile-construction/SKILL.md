---
name: nfw-density-profile-construction
description: Use when you need a spherically symmetric gas density profile for a halo
  of known virial mass (e.g. 1e10 Msun at z=0, Δv=200) to feed into a sphere solver
  with an isotropic background field.
when_to_use_negative:
- The geometry is planar — use the semi-infinite slab solvers instead of a spherical
  profile
- The radiation field is a central point source rather than an isotropic background
  — use the Sphere_Prime / point-source sphere case
- You need non-equilibrium ionization evolution, which rabacus does not provide (planned
  future improvement)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: rabacus
  role: Provides the NFW halo construction utilities and the Sphere_Bgnd solver that
    consumes the built profile
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Imposes physical units on all halo profile variables so they are compatible
    with rabacus solver inputs
  repo: https://github.com/python-quantities/python-quantities
- name: Python
  role: High-level language in which the profile construction and solver calls are
    scripted
  repo: https://bitbucket.org/galtay/rabacus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:f9847c766837010a8923fab3da4614683fb94055f6201d73c0bed4c5448bec54
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Quantities
  - numpy
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# nfw-density-profile-construction

## Summary

Constructs a spherical NFW dark-matter halo density profile parameterized by virial mass, overdensity Δv=200, and a concentration–mass relation, as the input geometry for rabacus Sphere_Bgnd radiative-transfer solutions on a cosmological gaseous halo.

## When to use

Use when you need a spherically symmetric gas density profile for a halo of known virial mass (e.g. 1e10 Msun at z=0, Δv=200) to feed into a sphere solver with an isotropic background field.

## When NOT to use

- The geometry is planar — use the semi-infinite slab solvers instead of a spherical profile
- The radiation field is a central point source rather than an isotropic background — use the Sphere_Prime / point-source sphere case
- You need non-equilibrium ionization evolution, which rabacus does not provide (planned future improvement)

## Inputs

- Halo virial mass (e.g. 1e10 Msun)
- Overdensity parameter Δv (e.g. 200)
- Redshift (e.g. z=0)
- Concentration relation (c0=3.5, Eqs. 41-43)
- Quantities-enabled unit system (python-quantities)

## Outputs

- NFW radial density profile object with Quantities units, suitable as input to rabacus Sphere_Bgnd solvers

## How to apply

Build the NFW profile for a halo of virial mass 1e10 Msun with overdensity Δv=200 at z=0, using the c0=3.5 concentration relation given in Eqs. 41-43 of the rabacus paper. Express every profile quantity (radius, density, column density) with Quantities units, since all variables in rabacus carry units through the Quantities Python package. Once the density profile object is constructed, pass it to the Sphere_Bgnd solver together with an initialized HM12_Photorates_Table to obtain hydrogen and helium ionization, equilibrium temperature, and recombination-radiation radial profiles. The rationale is that the sphere case in rabacus requires a well-posed spherically symmetric density distribution, and the NFW form is the standard cosmological halo model the paper uses for its gaseous-halo example.

## Related tools

- **rabacus** (Provides the NFW halo construction utilities and the Sphere_Bgnd solver that consumes the built profile) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Imposes physical units on all halo profile variables so they are compatible with rabacus solver inputs) — https://github.com/python-quantities/python-quantities
- **Python** (High-level language in which the profile construction and solver calls are scripted) — https://bitbucket.org/galtay/rabacus

## Examples

```
import rabacus as ra; nfw = ra.nfw.NFW(Mh=1e10*pq.Msun, yv=200.0, c0=3.5, z=0.0); r = nfw.r_vir * np.linspace(0.01, 1.0, 512)
```

## Evaluation signals

- All profile arrays carry Quantities dimensionality (e.g. density in mass/volume, radius in kpc) and units convert cleanly with Quantities
- The profile integrates to the specified overdensity Δv=200 within the virial radius at the chosen redshift
- Concentration computed from the c0=3.5 relation (Eqs. 41-43) matches the analytic expectation for 1e10 Msun at z=0
- Feeding the profile into Sphere_Bgnd yields converged radial ionization and temperature profiles for all five background models (grey_fixT_B through hm12_evoT_rt)

## Limitations

- Metal line cooling and non-equilibrium solvers are not yet included in rabacus (planned future improvements)
- The sphere solution assumes a spherically symmetric gas distribution with an isotropic background — clumpy or aspherical halos are out of scope
- Equilibrium temperatures only; time-dependent thermal evolution is not computed

## Evidence
