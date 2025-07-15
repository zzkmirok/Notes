DAY 2
==================================
Python stack in AMS
===================================
# Atomic simulation environment (ASE)

an example: import ase.caslculators.emt.EMT (a force field in ASE)

## Use cases
- ML potential

## Problems and difficulties

- Parallelization difficult to make user-friendly, AMS Driver should run in serial.

## ASE vs MLPotential (which included in AMS)

- MLPotential includes pretrained model.
- Indeed a wrapper of ASE.


## Example: Harmonic Spring Calculator

- torch.nn.functional.pdist calculate pair-wise distances
- from ase.build import molecule::

    Invoke a molecule with str

## conda manage different python venv

> Using Loggers for debugging:
>   logger.debug()


PLAMS Python library for workflows
===================================================

> quite like cty
> 1. generate from SMILES
> 2. settings
> 3. job creation and running