---
title: Calculating Excited States of Point Defects in VASP
date: 2025-04-13
categories: [Computational Materials Science]
tags: [Defect, VASP]     # TAG names should always be lowercase
author: Shibu
description: Tutorial on Excited State calculations of point defect
toc: true # default is true
pin: false
comments: true
math: true
---

## Shining Light on Defects: Calculating Excited States of Point Defects in VASP

Point defects, the atomic-scale imperfections in crystals, are often the stars of the show when it comes to a material's optical and electronic behavior. They can absorb light, emit light (luminescence), trap charges, and mediate energy transfer. Think of color centers in diamonds (like the NV center), dopants in semiconductors, or recombination centers affecting solar cell efficiency.

Understanding these processes requires going beyond the ground state and exploring the *excited states* of the defect system. While VASP is a powerful tool for ground-state defect calculations, simulating their excited states presents a significant leap in complexity.

This post delves into the common approaches and critical considerations when tackling excited-state calculations for point defects specifically within the VASP framework.

**Why Study Excited States of Defects?**

*   **Optical Properties:** Predict absorption and emission energies (photoluminescence), explaining colors and fluorescence. Calculate oscillator strengths to estimate transition probabilities.
*   **Charge Trapping & Recombination:** Understand how defects capture electrons or holes, and how these excited carriers relax (radiatively or non-radiatively). This is crucial for LEDs, lasers, scintillators, and solar cells.
*   **Defect Levels:** Determine the precise energy positions of defect states relative to the band edges *in their excited configurations*.
*   **Photochemistry:** Model defect-assisted photocatalysis or photodegradation mechanisms.
*   **Quantum Information:** Characterize defects like the NV center in diamond for quantum bit applications, which relies heavily on its optical spin readout mechanism (involving excited states).

**The Added Layers of Challenge:**

Calculating excited states is inherently harder than ground-state DFT. For defects in VASP, we face combined challenges:

1.  **Standard DFT Limitations:** Ground-state DFT functionals aren't designed for excited states. TD-DFT, while common, has known issues (charge transfer, double excitations).
2.  **Defect Localization:** Defect states are often localized, but standard DFT functionals (like PBE) can artificially delocalize them, leading to incorrect energies and character.
3.  **Supercell Effects:** Finite-size errors from periodic boundary conditions persist. Applying charge corrections and potential alignment becomes more complex when dealing with excited states.
4.  **Strong Correlation:** Some defects might involve strongly correlated d or f electrons, requiring methods like DFT+U, which needs careful application in excited-state calculations.
5.  **Distinguishing States:** Defect-related excitations can mix or overlap with bulk transitions in the supercell, making interpretation difficult.

**Common Approaches in VASP:**

VASP offers several routes, each with pros and cons:

1.  **Delta Self-Consistent Field (ΔSCF):**
    *   **Concept:** Perform two separate SCF calculations: one for the ground state and one constrained to represent the excited state. The energy difference `E(excited) - E(ground)` gives the excitation energy.
    *   **How in VASP:**
        *   Calculate the relaxed ground state.
        *   Start a new calculation from the ground state `WAVECAR`.
        *   Constrain orbital occupations to force the system into an approximate excited state configuration. This often involves moving an electron from an occupied state (e.g., a defect state in the gap) to an unoccupied state (e.g., conduction band minimum or another defect state).
        *   Use tags like `FERWE` / `FERDO` to manually set occupations, `NUPDOWN` to fix total magnetic moment (useful for triplet states), or `I_CONSTRAINED_M` for specific atomic magnetic moments. Use `ISMEAR = -2` with a small `SIGMA`.
    *   **Pros:** Conceptually straightforward, uses standard VASP SCF engine. Allows for ionic relaxation *in the excited state* (by setting `NSW > 0` in the constrained calculation), enabling calculation of emission energies and Stokes shifts. Can sometimes yield surprisingly good results for the lowest excitation of a given spin/symmetry.
    *   **Cons:** Prone to "variational collapse" where the SCF converges back to the ground state (use `NELMIN` to force more SCF steps initially). Identifying the correct constraint isn't always obvious. Not systematic – hard to know if you found the *lowest* excited state or just *an* excited state. Orthogonality to the ground state isn't strictly enforced. Best for single-electron excitations.

2.  **Time-Dependent DFT (TD-DFT) / Linear Response:**
    *   **Concept:** Calculates the response of the system to a time-dependent perturbation (like light). Excitation energies appear as poles in the frequency-dependent response function (e.g., the dielectric function).
    *   **How in VASP:**
        *   Typically via calculating the frequency-dependent dielectric matrix (`LOPTICS = .TRUE.`). This uses the Casida formalism or similar linear-response approaches within DFT.
        *   Alternatively, the Sternheimer equation (`IBRION = 7` or `8`) can be used for TD-DFT calculations of response properties.
        *   Using hybrid functionals (like HSE06 via `LHFCALC = .TRUE.`) is often *essential* to get qualitatively correct starting electronic structure (band gap, defect levels) before applying TD-DFT.
    *   **Pros:** More systematic than ΔSCF for absorption spectra. Provides multiple excitation energies and oscillator strengths simultaneously. Standard approach for optical absorption.
    *   **Cons:** Inherits limitations of the underlying DFT functional (especially band gap errors with LDA/GGA, charge-transfer excitation errors). Standard VASP implementation calculates *vertical* excitations (at fixed ground-state geometry). Can be computationally expensive, especially with hybrids and many frequency points (`NOMEGA`). Defect states might be hard to isolate in the calculated spectrum, requiring analysis of state character. Doesn't easily allow for geometry relaxation in the excited state.

3.  **GW Approximation + Bethe-Salpeter Equation (GW-BSE):**
    *   **Concept:** A more advanced, many-body perturbation theory approach. First, `GW` calculations correct the DFT eigenvalues to yield more accurate quasiparticle energies (improving band gaps and defect level positions). Then, the `BSE` is solved using the GW quasiparticles to include electron-hole interactions (excitonic effects), yielding the optical spectrum.
    *   **How in VASP:** Requires a multi-step workflow: DFT ground state -> GW calculation (`ALGO = GW` variants, needs many empty bands `NBANDS`) -> BSE calculation (`ALGO = BSE`, often combined with `LOPTICS = .TRUE.`).
    *   **Pros:** Considered the state-of-the-art within VASP for optical absorption spectra in solids. Provides accurate quasiparticle levels and explicitly includes excitonic effects, which can be crucial for defects.
    *   **Cons:** *Very* computationally demanding, especially for large defect supercells required to minimize spurious interactions. Convergence with respect to parameters (k-points, `NBANDS`, energy cutoffs) is complex and must be carefully checked. Setup is significantly more involved than TD-DFT or ΔSCF. Primarily gives absorption spectra (vertical excitations).

**Key VASP Considerations for Defect Excited States:**

*   **Functional:** Standard LDA/PBE often gives poor starting points (band gap, defect level position/localization). Using HSE06 (`LHFCALC = .TRUE.`) is highly recommended for TD-DFT and ΔSCF, and often used as a starting point for GW. DFT+U (`LDAU = .TRUE.`, etc.) might be needed if d/f electrons are involved in the defect.
*   **Supercell Size:** Still crucial! Finite-size effects might differ between ground and excited states. Convergence testing is essential.
*   **K-points:** Need a mesh converged for the *defect supercell*. Gamma-only (`1x1x1`) is rarely sufficient, even for large cells, particularly for response calculations (TD-DFT, GW-BSE).
*   **Geometry:** Decide if you need vertical excitations (absorption, fixed geometry) or relaxed excited-state geometry (emission, ΔSCF allows this). Calculate the Stokes shift (`E_abs - E_em`).
*   **Charged Defects:** Combining charged defect formalisms (NELECT, corrections) with excited-state methods is tricky. How potential alignment (`ΔV`) and charge corrections (`E_corr`) apply needs careful consideration, potentially requiring specialized schemes. This is an advanced topic.
*   **Analysis:** Don't just look at energies! Analyze the character of the excited state: Which orbitals contribute? Is it localized on the defect? Is there charge transfer? VASP outputs (like `PROJCAR`, `PARCHG` visualization) can help. In TD-DFT/BSE spectra, identify peaks corresponding to defect transitions vs. bulk transitions.

**Conclusion:**

Calculating excited states of point defects in VASP is a challenging but rewarding endeavor, pushing the boundaries of standard DFT simulations. Methods like ΔSCF, TD-DFT, and GW-BSE offer different levels of accuracy and computational cost. Success requires careful planning, choice of method and functional, rigorous convergence testing (especially supercell size and k-points), and detailed analysis of the results. Be prepared for significantly higher computational costs compared to ground-state calculations, and always critically evaluate the results in the context of the chosen method's known limitations.


### Further Reading & References:

Calculating excited states, especially for defects within a periodic DFT framework like VASP, involves combining several advanced concepts. These references provide background on the methods and specific applications:

**Reviews and Foundational Concepts:**

1.  **First-principles calculations of point defects in solids**
    *   C. Freysoldt, B. Grabowski, T. Hickel, J. Neugebauer, G. Kresse, A. Janotti, and C. G. Van de Walle
    *   *Reviews of Modern Physics* **86**, 253 (2014)
    *   *Comment:* While primarily focused on ground states, this essential review covers defect formalism, charged defects, corrections, and the underlying DFT challenges, providing crucial context.

2.  **Excited states from time-dependent density functional theory**
    *   M. E. Casida (Chapter in *Recent Advances in Density Functional Methods, Part I*)
    *   *World Scientific*, pp 155-192 (1995)
    *   *Comment:* Foundational work on the Casida formalism for TD-DFT linear response, which underlies many implementations, including aspects of VASP's `LOPTICS` calculations.

3.  **Electronic excitations: density-functional versus many-body Green's-function approaches**
    *   G. Onida, L. Reining, and A. Rubio
    *   *Reviews of Modern Physics* **74**, 601 (2002)
    *   *Comment:* A comprehensive review comparing TD-DFT and Many-Body Perturbation Theory (GW-BSE) for electronic excitations, primarily in solids. Excellent for understanding the theory behind GW-BSE.

**ΔSCF / Constrained DFT for Defect Excitations:**

4.  **First-principles theory of nonradiative carrier capture at point defects**
    *   A. Alkauskas, Q. Yan, and C. G. Van de Walle
    *   *Physical Review B* **90**, 075202 (2014)
    *   *Comment:* While focused on non-radiative capture, this paper extensively discusses the methodology of using constrained DFT (related to ΔSCF) to calculate configuration coordinate diagrams, including excited state energies and relaxation for defects.

5.  **Tutorial: Defects in semiconductors—Combining experiment and theory**
    *   A. Alkauskas and C. G. Van de Walle (Section V.C on Optical Properties)
    *   *Journal of Applied Physics* **129**, 111101 (2021)
    *   *Comment:* This tutorial includes sections clearly explaining the use of ΔSCF/constrained DFT for calculating optical absorption and emission energies (Stokes shift) for defects.

**GW-BSE for Defect Excitations:**

6.  **Accurate defect levels and optical transitions using the GW-BSE formalism**
    *   M. Råsander and A. Pasquarello
    *   *Physical Review B* **93**, 195207 (2016)
    *   *Comment:* Demonstrates the application of GW-BSE for calculating defect levels and optical properties, comparing with experimental data.

7.  **Excitation Spectra of Point Defects from Optimal Tuning of screened Range-separated Hybrid Functionals**
    *   D. A. Vakhrushev, M. Råsander, and A. Pasquarello
    *   *The Journal of Physical Chemistry C* **123**, 10522 (2019)
    *   *Comment:* Explores the use of tuned functionals as starting points and compares with GW-BSE for defect excitations. Highlights the importance of the starting point.

**Functionals and Corrections (Relevance to Excited States):**

8.  **Hybrid functionals based on a screened Coulomb potential**
    *   J. Heyd, G. E. Scuseria, and M. Ernzerhof
    *   *The Journal of Chemical Physics* **118**, 8207 (2003); *ibid.* **124**, 219906 (2006) (Erratum)
    *   *Comment:* The HSE functional, crucial for getting reasonable starting band gaps and defect level positions before attempting TD-DFT or GW-BSE calculations in VASP.

9.  **Fully Ab Initio Finite-Size Corrections for Charged-Defect Supercell Calculations**
    *   C. Freysoldt, J. Neugebauer, and C. G. Van de Walle
    *   *Physical Review Letters* **102**, 016402 (2009)
    *   *Comment:* Describes the FNV correction scheme for charged ground states. Applying corrections to *excited* charged states requires care and is an advanced topic, but understanding the ground state corrections is the first step.

10. **Electron-energy-loss spectra and the structural stability of nickel oxide: An LSDA+U study**
    *   S. L. Dudarev, G. A. Botton, S. Y. Savrasov, C. J. Humphreys, and A. P. Sutton
    *   *Physical Review B* **57**, 1505 (1998)
    *   *Comment:* Details the DFT+U method, potentially needed if strongly correlated electrons are involved in the defect states, although its use in excited states needs careful validation.

**VASP Manual:**

11. **The VASP Manual (online or PDF)**
    *   Available from the VASP website (requires license).
    *   *Comment:* The ultimate reference for VASP tags mentioned (e.g., `LOPTICS`, `ALGO = GW/BSE`, `LHFCALC`, `FERWE`/`FERDO`, `I_CONSTRAINED_M`, `IBRION=7,8`, `NBANDS`, `NOMEGA`). Check the specific sections on Response Functions, GW, BSE, and constrained calculations.
