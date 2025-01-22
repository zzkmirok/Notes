Unified Free Energy Dynamics (UFED)
===================================

## [2012-Chen, M. et al. Tuckerman, M.E. - Heating and flooding A unified approach for rapid generation of free energy surface](https://pubs.aip.org/aip/jcp/article/137/2/024102/316029/Heating-and-flooding-A-unified-approach-for-rapid)

The mentioned scheme thus strategically combines the high-temperature adiabatic dynamics of temperature-accelerated molecular dynamics/ driven Adiabatic Free Energy Dynamics (TAMD/d-AFED), the biasing potential of Metadynamics, and the use of the force as in Adaptive Biasing Force (ABF), umbrella integration or the recent and very powerful single-sweep method.

> *Note*: The above paragraph points out several modern CV-based enhanced sampling methods.

This is an extended phase-space approach by coupling a fictitious particle to each CV of interest and adding two key ingredients.

- Enforce adiabatic decoupling between fictious and physical particles.
- Sample the thermodynamic force on the fictitious particles to reconstruct the FES by a posteriori numerical integration.

### Methodology:

Mainly based on TAMD/d-AFED scheme, following steps:

1. Replace the dirac function in the probability function based on canonical partition function by the limit of a Gaussian distribution (large mean, small variances).
2. Introducing a high temperature and adiabatic decoupling in the extended phase-space variables.
3. Achieve Efficiency by working with the free energy gradient or mean force on CV. Even at higher temperature this also works for sampling accelerations.
4. Apply bias to the extended phase-space variables, which helps reduce the frequency of required visits to a given free energy basin. In contrast to this, the bias in metadynamics is applied directly to the CVs.
5. In practice, bias are mapped onto a regular grid and employ an interpolation algorithm to calculate the bias and its derivatives at arbitrary positions in the extended phase space. Free Energy Surface are also expended  in a basis function set, turning minimization of an objective function of FES to solving only a linear system.

**Advantages**

1. The extended variables can evolve at a temperature higher than that of the physical system, which accelerates the sampling as in the original d-AFED method.
2. A history-dependent biasing potential can be imposed on the extended variables to penalize regions already sampled.

$\implies$

- Faster convergence speed, more robust than metadynamics in terms of bias accumulation speed and
  less sensitive to degeneracies in the FES.
- Simpler to implement than ABF because the use of an extended phase space makes the construction of the force more straightforward for any set of CVs.
- A promising method to accurately generate FES when higher dimensional FESs are
  required.

In the literature, there is a comment on the high dimensional FES:

> The CV space is not uniformly populated but rather has a “sponge-like” structure with high dimensional but well localized free energy basins separated by elongated transition paths of lower effective dimensionality.

> *Note*: This infers an intuition when dealing with high dimensional FES.
