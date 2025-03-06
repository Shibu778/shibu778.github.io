---
title: Quantum Error Mitigation
date: 2025-03-07
categories: [Quantum Technology]
tags: [Quantum Algorithm, Quantum Error Mitigation]     # TAG names should always be lowercase
author: Shibu
description: Simple explanation of quantum error mitigation
toc: true # default is true
pin: false
comments: true
---

# Brief Intro to Quantum Computing
Quantum computers leverage the unique properties of quantum systems to process information. However, these quantum systems are delicate and prone to errors. In classical computing, information is stored as bits (sequences of 0s and 1s) and processed using digital gates. Similarly, quantum information is stored in qubits. Unlike classical bits, qubits can exist in two distinct classical states, 0 and 1, as well as in a superposition of both states due to their quantum mechanical nature.

A qubit in superposition represents a state where there is a certain probability of it being 0 and a certain probability of it being 1, with the total probability summing up to 1. This superposition state is different from a classical probabilistic state because it can exhibit interference phenomena, as demonstrated in the double-slit experiment. Additionally, multiple qubits can interact to form entangled states.

The properties of superposition and entanglement in qubits are predicted and demonstrated to be useful in solving complex problems that are very difficult for classical computers to handle. Just as classical computers use gates to transform bits and perform computations, quantum computers use quantum gates to manipulate qubits. The results of these computations are obtained by measuring the quantum states of the qubits, which collapses the quantum state into one of the finite set of classical states.

That's a brief overview of qubits, quantum information processing, and quantum computers.

# Intro to Quantum Error Mitigation
Quantum systems that exhibit qubits are extremely delicate and can gradually lose their quantum coherence due to interactions with the environment. This leads to reduced fidelity in information processing, known as errors. Additionally, quantum gates are not perfect and can introduce errors during operations. To maintain the fidelity of quantum information processing, these errors need to be reduced and corrected.

Quantum error mitigation aims to reduce the errors incurred during quantum computations. It is also known as approximate error suppression or approximate quantum error correction. These errors are due to quantum hardware, not software. Error mitigation differs from error correction, which aims to completely remove the impact of errors on quantum computations.

There are several quantum error mitigation techniques, such as zero-noise extrapolation and probabilistic error cancellation.

## Zero Noise Extrapolation
Zero-noise extrapolation (ZNE) is a technique used in quantum computing to mitigate errors without requiring additional quantum resources. Quantum systems are inherently noisy, and this noise can affect the accuracy of computations. ZNE aims to reduce the impact of this noise by extrapolating the results of quantum computations to a "zero-noise" limit.

The process involves running the same quantum computation multiple times with varying levels of artificially increased noise. By analyzing how the results change with different noise levels, it is possible to estimate what the result would be if there were no noise at all. This is done through a mathematical extrapolation process.

For example, if you have a quantum circuit, you can intentionally increase the noise in the system by applying additional gate operations or by adjusting the parameters of the quantum gates. After running the circuit at different noise levels, you collect the results and use extrapolation techniques to predict the outcome in the absence of noise.

Zero-noise extrapolation is particularly useful because it does not require additional qubits or complex error correction codes. Instead, it leverages the existing quantum hardware and provides a practical way to improve the fidelity of quantum computations.

**Limitations:** ZNE has limitations, including challenges in scalability for larger systems, sensitivity to extrapolation errors, increased computational costs, and dependence on precise noise modeling. Despite these challenges, ZNE remains a practical approach for improving the fidelity of quantum computations, especially in near-term quantum devices where full error correction is not yet feasible.

# Reference
1. [About Error Mitigation — Mitiq 0.43.0 documentation](https://mitiq.readthedocs.io/en/stable/guide/error-mitigation.html)
