# Transfer Learning for Bayesian Network Fault Diagnosis

## Overview

I built this project to explore whether a Bayesian network used for fault diagnosis on one machine can be adapted to a similar machine when only a limited amount of new data is available.

The idea is that a new machine might share most of its components and dependencies with an existing one, but have different fault rates or less reliable sensors. In that situation, learning everything from scratch might waste useful information, while reusing the old model without changes might lead to inaccurate diagnoses.

I wanted to investigate when transferring knowledge from an existing model is useful, and how much the new model should rely on that knowledge.

## What I'm investigating

The experiment compares three approaches:

- **Source only:** Reuse the original machine's Bayesian network without changing its probabilities.
- **From scratch:** Learn the probabilities using only data from the new machine.
- **Transfer learning:** Learn from the new machine's data while using the original model's probabilities as prior information.

I compare these approaches with different amounts of training data to investigate how the available data affects their performance.

## Methodology

I use the `win95pts` printer-troubleshooting Bayesian network from the `bnlearn` repository. The network represents relationships between possible faults, symptoms and diagnostic tests.

Since I don't have real fault data from two machine variants, I create synthetic versions of the original network. The variants preserve the same network structure but differ in their fault probabilities and the reliability of their observations.

I investigate two scenarios:

- **Mild change:** The new machine behaves similarly to the original.
- **Strong change:** The new machine has more substantial differences in fault rates and sensor reliability.

The experiment evaluates both how accurately each approach estimates the new machine's probabilities and how effectively it identifies faults from observed symptoms and test results. I also investigate how the strength of the source model's prior information affects performance.

## Tools

- Python
- pgmpy
- NumPy and pandas
- Matplotlib
- SciPy
- Jupyter Notebook

## Limitations and next steps

This is a synthetic proof of concept rather than a validated system for real industrial equipment. Both machine variants share the same network structure, and the simulated training data is more complete than real maintenance records are likely to be.

My next steps are to validate the experimental results, investigate the conditions under which transfer learning helps or hurts, and explore how the approach could be adapted to real fault-diagnosis data. Other possible extensions include incomplete observations and changes to the network structure itself.

## Status

This project is a work in progress. I'm using it to develop my understanding of Bayesian networks, parameter estimation and transfer learning, with a focus on their potential application to machine fault diagnosis.
