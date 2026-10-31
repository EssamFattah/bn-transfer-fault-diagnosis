# Transfer Learning for Bayesian Network Fault Diagnosis

## Overview

I built this project to explore whether a Bayesian network used for fault diagnosis on one machine can be adapted to a similar machine when only a limited amount of new data is available.

The idea is that a new machine might share most of its components and dependencies with an existing one, but have different fault rates or less reliable sensors. Learning everything from scratch could waste useful information, while reusing the old model without changes could lead to inaccurate diagnoses.

I wanted to investigate when transferring knowledge from an existing model is useful, and how much the new model should rely on that knowledge.

## What I investigated

I compared three approaches:

- **Source only:** Reuse the original machine's Bayesian network without changing its probabilities.
- **From scratch:** Learn the probabilities using only data from the new machine.
- **Transfer learning:** Learn from the new machine's data while using the original model's probabilities as prior information.

I tested these approaches with different amounts of training data and under two scenarios:

- **Mild change:** The new machine behaves relatively similarly to the original.
- **Strong change:** The new machine has more substantial differences in fault rates and sensor reliability.

The experiment looks at two things: how accurately each approach estimates the new machine's probabilities, and how well it actually diagnoses faults.

## Results

The experiments show that transferring knowledge can be very useful when target data is limited, but the amount of benefit depends on how different the new machine is.

### 1. Estimating the new machine's probabilities

Transfer learning consistently reduced parameter estimation error compared with learning from scratch.

With only 25 training records:

| Scenario | From scratch | Transfer | Transfer error relative to scratch |
|---|---:|---:|---:|
| Mild change | 0.01007 | 0.00161 | 16% |
| Strong change | 0.01224 | 0.00268 | 22% |

<img width="1290" height="440" alt="image" src="https://github.com/user-attachments/assets/2278663b-76ba-4c37-a2ab-d5036f9578ed" />

In other words, transfer reduced parameter error by around 84% in the mild scenario and 78% in the strong scenario.

The advantage remained as more data became available, although the difference gradually narrowed. With 5,000 records, transfer still achieved lower parameter error than learning from scratch in both scenarios.

### 2. Diagnosing faults

Better parameter estimates did not translate into equally large improvements in diagnostic accuracy.

With only 25 training records:

| Scenario | Method | Top-1 accuracy | Top-3 accuracy |
|---|---|---:|---:|
| Mild | From scratch | 75.1% | 94.0% |
| Mild | Transfer | **78.6%** | **96.8%** |
| Strong | From scratch | 62.0% | 91.1% |
| Strong | Transfer | 61.0% | **92.3%** |

<img width="1290" height="790" alt="image" src="https://github.com/user-attachments/assets/7e17ce4d-8c1d-41fc-8577-349020c9c211" />

For the mild scenario, transfer improved top-1 accuracy by 3.5 percentage points and top-3 accuracy by 2.8 points. Both improvements were statistically clear across the experiment's seeds.

For the strong scenario, the top-1 difference was within statistical noise, but transfer did improve top-3 accuracy.

As more training data became available, the difference between transfer and learning from scratch became very small. With 1,000 records, both approaches were close to the true model's diagnostic accuracy.

### 3. What happens when the old model is reused unchanged?

This was one of the clearest results.

When the new machine differs substantially from the original, using the original model without adapting its probabilities can be extremely unreliable.

| Scenario | Source-only top-1 | True model top-1 | Impossible cases |
|---|---:|---:|---:|
| Mild change | 59.7% | 78.8% | 17.1% |
| Strong change | 8.9% | 69.1% | 64.4% |

The impossible cases occur when the original model assigns zero probability to observations that can occur on the new machine, partly because its original sensor tables contain hard zeros.

This highlights why simply reusing an existing model can be risky, even when the underlying network structure remains the same.

### 4. How much should the old model be trusted?

I also tested different prior strengths to investigate how much influence the original model should have during transfer.

The best tested prior strength depended on how different the new machine was:

| Scenario | Best tested prior strength | Parameter error |
|---|---:|---:|
| Mild change | 100 | 0.00042 |
| Strong change | 10 | 0.00166 |

<img width="1290" height="440" alt="image" src="https://github.com/user-attachments/assets/6245d393-df1f-403a-aa4e-1ee2f2508ee6" />

With a stronger shift, relying too heavily on the original model became counterproductive. In the strong scenario, a prior strength of 1,000 produced higher parameter error than learning from scratch.

This is an important part of the experiment: transferring knowledge is not just about reusing an old model, but also about deciding how much of that knowledge should be retained.

## Methodology

I used the `win95pts` printer-troubleshooting Bayesian network from the `bnlearn` repository. It contains 76 nodes representing possible faults, symptoms and diagnostic tests.

Since I didn't have real fault data from two machine variants, I generated synthetic versions of the original network. Both variants preserve the same graph structure but differ in their fault probabilities and sensor reliability.

For parameter estimation, I compared the learned conditional probability tables against the known probabilities of the target network.

For diagnosis, I generated test cases with exactly one active fault and hid the fault variables from the models. Each model then ranked the possible faults using exact posterior probabilities. Performance was measured using top-1 and top-3 accuracy.

Transfer learning was implemented using Dirichlet pseudo-counts. The source model's probabilities were treated as additional prior information, with the prior strength controlling how much influence they had compared with the new observations.

Each experiment was repeated over 10 independent seeds, with 95% confidence intervals used to assess uncertainty. Comparisons between methods used the same training data and test cases.

## Tools

- Python
- pgmpy
- NumPy and pandas
- Matplotlib
- SciPy
- Jupyter Notebook

## Limitations and next steps

This is a synthetic proof of concept rather than a validated system for real industrial equipment.

A few limitations are worth noting:

- Both machine variants share the same network structure. The experiment only investigates transferring probabilities, not transferring the graph itself.
- The synthetic changes are controlled perturbations. Real machines may experience more structured changes in fault rates and sensor reliability.
- Training data is fully observed, whereas real maintenance records often contain missing information.
- Diagnosis assumes exactly one active fault and observes all symptoms and test results. Real fault diagnosis may involve multiple simultaneous faults and incomplete observations.
- The best prior strength is selected using knowledge of the true target model. In practice, this would need to be estimated using methods such as cross-validation on the available target data.

Possible extensions include learning with incomplete observations, adapting the network structure, and investigating how to choose prior strength without access to the true target model.

## Status

This project is a synthetic proof of concept exploring parameter transfer in Bayesian networks. It helped me investigate the relationship between prior knowledge, limited training data and diagnostic performance, as well as the risks of transferring knowledge between systems that are not identical.
