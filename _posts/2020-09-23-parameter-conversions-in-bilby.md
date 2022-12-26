---
layout: single
classes: wide
title: Sampling custom parameters in bilby
date: 2020-09-23 13:24 -0500
author_profile: true
read_time: true
comments: false
share: true
related: false
usemathjax: true
---
This is a post about parameter conversions in [bilby](https://lscsoft.docs.ligo.org/bilby/), which is
stochastic sampling library used for gravitational-wave (GW) parameter estimation. Details about bilby can
be found in the [Ashton et. al.](https://doi.org/10.3847/1538-4365/ab06fc) paper and related references.
This post is a detail that may be useful for some GW practitioners familiar with bilby. It's not
meant to be useful generally. 

In some cases, one may need to re-express the GW waveform differently compared to the standard 15-dimensional
parameterization for binary black holes (BBHs), or 17-dimensional parameterization for binary neutron star (BNS)
mergers. This can be the case, for example, where there are some physically motivated constraining relations
between some parameters. As an example, consider the parameterization reported in
[our paper](https://doi.org/10.1103/PhysRevD.104.083528) of applying neutron star universal relations to
measure the Hubble constant, $$H_0$$. I won't go into the motivation for the parameterization and how
certain universal relations in NSs help in reducing the dimensionality of the problem, but the bottomline
is - the tidal parameters, $$\bar{\lambda}_{1,2}$$, can re-expressed in terms of a universal constant
referred to as $$\bar{\lambda}_0^{(0)}$$ and the redshift, $$z$$, of the source, i.e.,
\begin{align}
    \bar{\lambda}_{1,2} = \bar{\lambda}\_{1,2}(\bar{\lambda}_0^{(0)}, z). \nonumber
\end{align}
Hence, instead of sampling over the 15 BBH parameter plus $$\{\bar{\lambda}_1, \bar{\lambda}_2\}$$, one may
wish to sample on BBH parameters $$+\;\{\bar{\lambda}_0^{(0)}, z\}$$, with appropriate priors on the latter
two parameters. This can be done by overriding the source model as follows:
```python
def lambda_0_z_lal_binary_neutron_star(
        frequency_array, chirp_mass, mass_ratio, luminosity_distance, a_1, tilt_1,
        phi_12, a_2, tilt_2, phi_jl, theta_jn, phase, lambda_0_0, z,
        **kwargs):
    """Source model to sample lambda_0_0 and redshift directly"""
    # extract the individual masses
    total_mass = bilby.gw.conversion.chirp_mass_and_mass_ratio_to_total_mass(
        chirp_mass, mass_ratio
    )
    mass_1 = total_mass / (1 + mass_ratio)
    mass_2 = total_mass - mass_1
    # convert to source-frame
    mass_1_source = mass_1 / (1 + z)
    mass_2_source = mass_2 / (1 + z)
    # reference mass value used in Phys. Rev. D 104, 083528.
    M0 = 1.4
    # compute the tidal parameters from universal relations
    lambda_1 = get_lambda_from_mass(mass_1_source, lambda_0_0, M0=M0)
    lambda_2 = get_lambda_from_mass(mass_2_source, lambda_0_0, M0=M0)
    # return the base waveform model source
    return bilby.gw.source.lal_binary_neutron_star(
        frequency_array, mass_1, mass_2, luminosity_distance, a_1, tilt_1,
        phi_12, a_2, tilt_2, phi_jl, theta_jn, phase, lambda_1, lambda_2,
        **kwargs
    )
```
The important points are:
- The function signature takes in the relevant parameters - note arguments include `lambda_0_0` and `z`,
  which are different from the individual tidal deformability parameters `lambda_1` and `lambda_2`.
- Perform the conversion to the standard GW parameters. This will be specific to the problem. In our case,
  we convert to source-frame parameters using the redshift, and then use universal relations to calculate
  `lambda_1` and `lambda_2`.
- Return the stock frequency domain source model, `lal_binary_neutron_star` in this case.

This new custom source model can be easily added to the waveform generator instantiation. For example,
```python
waveform_generator = bilby.gw.WaveformGenerator(
    frequency_domain_source_model=lambda_0_z_lal_binary_neutron_star,
    # other args and kwargs
)
```

## Using in `parallel_bilby`
For running in an HPC setting, it might be useful to create a module out of the source
models, which can be used by [bilby_pipe](https://git.ligo.org/lscsoft/bilby_pipe) and
[parallel_bilby](https://git.ligo.org/lscsoft/parallel_bilby).
For example, I created [this helper library](https://github.com/deepchatterjeeligo/tidal-cosmology-conversions)
to store the relevant source models for the work mentioned above. For the runs in the paper,
- I would specify the source model as,
```
frequency-domain-source-model=t_cosmo.source.lambda_0_z_lal_binary_neutron_star
```
- My prior file would have lines like,
```
lambda_0_0 = Uniform(0, 1000, name="lambda_0_0", latex_label="\bar{\lambda}_0^{(0)}")
z = DeltaFunction(0.01, name="redshift", latex_label="$z$")
```
along with the other parameters.

Hope this can serve as a reference if you wish to sample on custom
parameters in bilby.
